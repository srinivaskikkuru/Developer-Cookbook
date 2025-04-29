# Oracle APEX Authentication and Authorization Process

## Overview

This document outlines the authentication and authorization process for Oracle APEX applications, including:
- Authentication methodology
- Dual data source integration (ERP & Active Directory)
- Authorization schema with role-based access control (RBAC)
- Implementation scripts and best practices

## 1. Authentication Process

### 1.1 Authentication Methods

For Oracle APEX, we'll implement a robust authentication system using:

- **Primary Method**: Oracle APEX Authentication with custom post-authentication processing
- **Secondary Support**: LDAP Authentication for Active Directory integration
- **Enhanced Security**: Two-factor authentication for sensitive operations

### 1.2 Authentication Flow

1. User submits credentials in the APEX login form
2. APEX validates credentials against the chosen authentication scheme
3. On successful authentication, our custom post-authentication process executes
4. The post-authentication process:
   - Retrieves employee data from appropriate source (ERP or Active Directory)
   - Establishes user session with proper role assignments
   - Sets application items for authorization decisions

### 1.3 Post-Authentication Data Source Selection

```sql
CREATE OR REPLACE PROCEDURE select_user_data_source(
    p_username IN VARCHAR2
) AS
    v_user_exists_erp NUMBER;
    v_user_exists_ad NUMBER;
    v_user_data user_data_record; -- Custom record type defined elsewhere
BEGIN
    -- Check if user exists in ERP system
    SELECT COUNT(*) INTO v_user_exists_erp 
    FROM HR_DATA_SOURCE
    WHERE username = p_username;
    
    -- Check if user exists in Active Directory
    SELECT COUNT(*) INTO v_user_exists_ad
    FROM ACTIVE_DATA_SOURCE
    WHERE username = p_username;
    
    -- Decide which source to use based on business rules
    IF v_user_exists_erp = 1 AND (
        -- Business rules for when to prefer ERP data
        -- Example: Internal employee flag is set
        EXISTS (SELECT 1 FROM HR_DATA_SOURCE WHERE username = p_username AND internal_emp_flag = 'Y')
    ) THEN
        -- Get data from ERP
        SELECT employee_id, full_name, email, department, position
        INTO v_user_data.employee_id, v_user_data.full_name, v_user_data.email, 
             v_user_data.department, v_user_data.position
        FROM HR_DATA_SOURCE
        WHERE username = p_username;
        
        -- Set source flag
        v_user_data.data_source := 'ERP';
    ELSIF v_user_exists_ad = 1 THEN
        -- Get data from Active Directory
        SELECT employee_id, display_name, email, department, job_title
        INTO v_user_data.employee_id, v_user_data.full_name, v_user_data.email,
             v_user_data.department, v_user_data.position
        FROM ACTIVE_DATA_SOURCE
        WHERE username = p_username;
        
        -- Set source flag
        v_user_data.data_source := 'AD';
    ELSE
        -- Handle case where user doesn't exist in either source
        RAISE_APPLICATION_ERROR(-20001, 'User not found in any data source');
    END IF;
    
    -- Set session data for the application
    apex_util.set_session_state('USER_ID', v_user_data.employee_id);
    apex_util.set_session_state('USER_NAME', v_user_data.full_name);
    apex_util.set_session_state('USER_EMAIL', v_user_data.email);
    apex_util.set_session_state('USER_DEPARTMENT', v_user_data.department);
    apex_util.set_session_state('USER_POSITION', v_user_data.position);
    apex_util.set_session_state('USER_DATA_SOURCE', v_user_data.data_source);
    
    -- Load user roles (implemented in next section)
    load_user_roles(p_username);
END select_user_data_source;
/
```

## 2. Authorization Schema

### 2.1 Database Schema for RBAC

We'll create the following tables to implement role-based access control:

#### 2.1.1 Users Table

```sql
CREATE TABLE app_users (
    user_id VARCHAR2(100) PRIMARY KEY,
    username VARCHAR2(100) NOT NULL UNIQUE,
    email VARCHAR2(255),
    display_name VARCHAR2(255),
    data_source VARCHAR2(50) CHECK (data_source IN ('ERP', 'AD')),
    is_active CHAR(1) DEFAULT 'Y' CHECK (is_active IN ('Y', 'N')),
    created_date TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    last_login_date TIMESTAMP WITH TIME ZONE,
    CONSTRAINT app_users_username_uk UNIQUE (username)
);

COMMENT ON TABLE app_users IS 'Central registry of all application users';
```

#### 2.1.2 Roles Table

```sql
CREATE TABLE app_roles (
    role_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    role_name VARCHAR2(100) NOT NULL,
    role_description VARCHAR2(500),
    is_active CHAR(1) DEFAULT 'Y' CHECK (is_active IN ('Y', 'N')),
    created_date TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR2(100),
    CONSTRAINT app_roles_name_uk UNIQUE (role_name)
);

COMMENT ON TABLE app_roles IS 'Application roles for authorization';
```

#### 2.1.3 Permissions Table

```sql
CREATE TABLE app_permissions (
    permission_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    permission_name VARCHAR2(100) NOT NULL,
    permission_key VARCHAR2(50) NOT NULL,
    permission_description VARCHAR2(500),
    component_type VARCHAR2(50), -- PAGE, REGION, BUTTON, etc.
    component_id VARCHAR2(50),   -- Component identifier
    is_active CHAR(1) DEFAULT 'Y' CHECK (is_active IN ('Y', 'N')),
    created_date TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR2(100),
    CONSTRAINT app_permissions_key_uk UNIQUE (permission_key)
);

COMMENT ON TABLE app_permissions IS 'Granular permissions that can be assigned to roles';
```

#### 2.1.4 User-Role Mapping

```sql
CREATE TABLE app_user_roles (
    user_role_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id VARCHAR2(100) NOT NULL,
    role_id NUMBER NOT NULL,
    assigned_date TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    assigned_by VARCHAR2(100),
    is_active CHAR(1) DEFAULT 'Y' CHECK (is_active IN ('Y', 'N')),
    CONSTRAINT app_user_roles_uk UNIQUE (user_id, role_id),
    CONSTRAINT app_user_roles_user_fk FOREIGN KEY (user_id) REFERENCES app_users (user_id),
    CONSTRAINT app_user_roles_role_fk FOREIGN KEY (role_id) REFERENCES app_roles (role_id)
);

COMMENT ON TABLE app_user_roles IS 'Mapping between users and their assigned roles';
```

#### 2.1.5 Role-Permission Mapping

```sql
CREATE TABLE app_role_permissions (
    role_permission_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    role_id NUMBER NOT NULL,
    permission_id NUMBER NOT NULL,
    assigned_date TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    assigned_by VARCHAR2(100),
    is_active CHAR(1) DEFAULT 'Y' CHECK (is_active IN ('Y', 'N')),
    CONSTRAINT app_role_perms_uk UNIQUE (role_id, permission_id),
    CONSTRAINT app_role_perms_role_fk FOREIGN KEY (role_id) REFERENCES app_roles (role_id),
    CONSTRAINT app_role_perms_perm_fk FOREIGN KEY (permission_id) REFERENCES app_permissions (permission_id)
);

COMMENT ON TABLE app_role_permissions IS 'Mapping between roles and their assigned permissions';
```

### 2.2 Authorization Functions

#### 2.2.1 Load User Roles Function

```sql
CREATE OR REPLACE PROCEDURE load_user_roles(
    p_username IN VARCHAR2
) AS
    v_user_id VARCHAR2(100);
BEGIN
    -- Get user ID
    SELECT user_id INTO v_user_id
    FROM app_users
    WHERE username = p_username;
    
    -- Create a comma-separated list of role names for this user
    DECLARE
        v_roles VARCHAR2(4000);
    BEGIN
        SELECT LISTAGG(r.role_name, ',') WITHIN GROUP (ORDER BY r.role_name)
        INTO v_roles
        FROM app_roles r
        JOIN app_user_roles ur ON r.role_id = ur.role_id
        WHERE ur.user_id = v_user_id
        AND r.is_active = 'Y'
        AND ur.is_active = 'Y';
        
        -- Set session state for roles
        apex_util.set_session_state('USER_ROLES', v_roles);
    END;
    
    -- Create a collection of permissions for this user
    APEX_COLLECTION.CREATE_OR_TRUNCATE_COLLECTION('USER_PERMISSIONS');
    
    INSERT INTO apex_collections (
        collection_name,
        c001, -- permission_key
        c002  -- permission_name
    )
    SELECT DISTINCT
        p.permission_key,
        p.permission_name
    FROM app_permissions p
    JOIN app_role_permissions rp ON p.permission_id = rp.permission_id
    JOIN app_user_roles ur ON rp.role_id = ur.role_id
    WHERE ur.user_id = v_user_id
    AND p.is_active = 'Y'
    AND rp.is_active = 'Y'
    AND ur.is_active = 'Y';
    
    -- Update last login time
    UPDATE app_users
    SET last_login_date = CURRENT_TIMESTAMP
    WHERE user_id = v_user_id;
    
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        -- User not found in the authorization tables
        -- Handle based on your requirements (create new user entry or deny access)
        NULL;
END load_user_roles;
/
```

#### 2.2.2 Check Permission Function

```sql
CREATE OR REPLACE FUNCTION has_permission(
    p_permission_key IN VARCHAR2
) RETURN BOOLEAN IS
    v_count NUMBER;
BEGIN
    -- Check if the permission exists in the user's permissions collection
    SELECT COUNT(*)
    INTO v_count
    FROM apex_collections
    WHERE collection_name = 'USER_PERMISSIONS'
    AND c001 = p_permission_key;
    
    RETURN v_count > 0;
END has_permission;
/
```

## 3. Integration with APEX Application

### 3.1 Application Authentication Setup

1. Navigate to Shared Components > Authentication Schemes
2. Create a new Authentication Scheme:
   - Name: "Dual Source Authentication"
   - Scheme Type: "Custom"
   - Authentication Function:

```sql
FUNCTION authenticate(p_username IN VARCHAR2, p_password IN VARCHAR2) RETURN BOOLEAN IS
    v_result BOOLEAN;
BEGIN
    -- First try to authenticate against the ERP system
    BEGIN
        SELECT CASE WHEN COUNT(*) > 0 THEN TRUE ELSE FALSE END INTO v_result
        FROM HR_DATA_SOURCE
        WHERE username = p_username
        AND password_hash = DBMS_CRYPTO.HASH(
            UTL_RAW.CAST_TO_RAW(p_password || salt),
            DBMS_CRYPTO.HASH_SH256
        );
        
        IF v_result THEN
            RETURN TRUE;
        END IF;
    EXCEPTION
        WHEN OTHERS THEN
            NULL; -- Continue to AD authentication
    END;
    
    -- If ERP authentication failed, try Active Directory
    BEGIN
        -- Use APEX_LDAP package to authenticate against AD
        v_result := APEX_LDAP.AUTHENTICATE(
            p_username => p_username,
            p_password => p_password,
            p_ldap_host => 'ldap.example.com',
            p_ldap_port => 389,
            p_ldap_dn => 'cn=' || p_username || ',ou=users,dc=example,dc=com'
        );
        
        RETURN v_result;
    EXCEPTION
        WHEN OTHERS THEN
            RETURN FALSE;
    END;
END authenticate;
```

3. Set Post-Authentication Procedure:
   - Procedure Name: `select_user_data_source`

### 3.2 Authorization Implementation

#### 3.2.1 Create Authorization Scheme

1. Navigate to Shared Components > Authorization Schemes
2. Create a new Authorization Scheme for each permission:
   - Name: "Has Permission [PERMISSION_NAME]"
   - Scheme Type: "PL/SQL Function Returning Boolean"
   - PL/SQL Function Body:

```sql
RETURN has_permission('[PERMISSION_KEY]');
```

#### 3.2.2 Apply Authorization to Components

1. In Page Designer, select the component to restrict
2. In the Property Editor, under Security:
   - Authorization Scheme: Select the appropriate scheme

## 4. Data Synchronization Process

### 4.1 Synchronization Job for HR Data

```sql
CREATE OR REPLACE PROCEDURE sync_users_from_hr AS
BEGIN
    MERGE INTO app_users a
    USING (
        SELECT 
            employee_id,
            username,
            email,
            full_name as display_name,
            'ERP' as data_source
        FROM HR_DATA_SOURCE
        WHERE active_flag = 'Y'
    ) h
    ON (a.user_id = h.employee_id)
    WHEN MATCHED THEN
        UPDATE SET
            a.email = h.email,
            a.display_name = h.display_name,
            a.is_active = 'Y'
    WHEN NOT MATCHED THEN
        INSERT (user_id, username, email, display_name, data_source, is_active)
        VALUES (h.employee_id, h.username, h.email, h.display_name, h.data_source, 'Y');
    
    COMMIT;
END sync_users_from_hr;
/
```

### 4.2 Synchronization Job for Active Directory

```sql
CREATE OR REPLACE PROCEDURE sync_users_from_ad AS
BEGIN
    MERGE INTO app_users a
    USING (
        SELECT 
            employee_id,
            username,
            email,
            display_name,
            'AD' as data_source
        FROM ACTIVE_DATA_SOURCE
        WHERE active_flag = 'Y'
    ) ad
    ON (a.user_id = ad.employee_id)
    WHEN MATCHED THEN
        UPDATE SET
            a.email = ad.email,
            a.display_name = ad.display_name,
            a.is_active = 'Y'
    WHEN NOT MATCHED THEN
        INSERT (user_id, username, email, display_name, data_source, is_active)
        VALUES (ad.employee_id, ad.username, ad.email, ad.display_name, ad.data_source, 'Y');
    
    COMMIT;
END sync_users_from_ad;
/
```

### 4.3 Schedule Synchronization Jobs

```sql
BEGIN
    DBMS_SCHEDULER.CREATE_JOB (
        job_name        => 'SYNC_HR_USERS_JOB',
        job_type        => 'STORED_PROCEDURE',
        job_action      => 'sync_users_from_hr',
        start_date      => SYSTIMESTAMP,
        repeat_interval => 'FREQ=DAILY; BYHOUR=2',
        enabled         => TRUE,
        comments        => 'Daily synchronization of ERP users at 2 AM'
    );
    
    DBMS_SCHEDULER.CREATE_JOB (
        job_name        => 'SYNC_AD_USERS_JOB',
        job_type        => 'STORED_PROCEDURE',
        job_action      => 'sync_users_from_ad',
        start_date      => SYSTIMESTAMP,
        repeat_interval => 'FREQ=DAILY; BYHOUR=3',
        enabled         => TRUE,
        comments        => 'Daily synchronization of Active Directory users at 3 AM'
    );
END;
/
```

## 5. Initialization and Setup

### 5.1 Initial Roles Setup

```sql
-- Insert default roles
BEGIN
    INSERT INTO app_roles (role_name, role_description, created_by) 
    VALUES ('ADMIN', 'System Administrator with full access', 'SYSTEM');
    
    INSERT INTO app_roles (role_name, role_description, created_by) 
    VALUES ('USER', 'Standard user with basic access', 'SYSTEM');
    
    INSERT INTO app_roles (role_name, role_description, created_by) 
    VALUES ('MANAGER', 'Department manager with elevated privileges', 'SYSTEM');
    
    INSERT INTO app_roles (role_name, role_description, created_by) 
    VALUES ('AUDITOR', 'Read-only access for auditing purposes', 'SYSTEM');
    
    COMMIT;
END;
/
```

### 5.2 Initial Permissions Setup

```sql
-- Insert base permissions
BEGIN
    -- Admin permissions
    INSERT INTO app_permissions (permission_name, permission_key, permission_description, component_type, created_by)
    VALUES ('User Management', 'USER_MGMT', 'Manage users and their roles', 'PAGE', 'SYSTEM');
    
    INSERT INTO app_permissions (permission_name, permission_key, permission_description, component_type, created_by)
    VALUES ('Role Management', 'ROLE_MGMT', 'Manage roles and their permissions', 'PAGE', 'SYSTEM');
    
    -- User permissions
    INSERT INTO app_permissions (permission_name, permission_key, permission_description, component_type, created_by)
    VALUES ('View Dashboard', 'VIEW_DASHBOARD', 'Access to the main dashboard', 'PAGE', 'SYSTEM');
    
    INSERT INTO app_permissions (permission_name, permission_key, permission_description, component_type, created_by)
    VALUES ('Edit Profile', 'EDIT_PROFILE', 'Edit personal profile information', 'PAGE', 'SYSTEM');
    
    -- Manager permissions
    INSERT INTO app_permissions (permission_name, permission_key, permission_description, component_type, created_by)
    VALUES ('Department Reports', 'DEPT_REPORTS', 'Access department reports', 'PAGE', 'SYSTEM');
    
    -- Auditor permissions
    INSERT INTO app_permissions (permission_name, permission_key, permission_description, component_type, created_by)
    VALUES ('Audit Logs', 'VIEW_AUDIT', 'View system audit logs', 'PAGE', 'SYSTEM');
    
    COMMIT;
END;
/
```

### 5.3 Link Roles and Permissions

```sql
-- Assign permissions to roles
BEGIN
    -- Admin role permissions
    FOR r IN (SELECT role_id FROM app_roles WHERE role_name = 'ADMIN')
    LOOP
        FOR p IN (SELECT permission_id FROM app_permissions)
        LOOP
            INSERT INTO app_role_permissions (role_id, permission_id, assigned_by)
            VALUES (r.role_id, p.permission_id, 'SYSTEM');
        END LOOP;
    END LOOP;
    
    -- User role permissions
    FOR r IN (SELECT role_id FROM app_roles WHERE role_name = 'USER')
    LOOP
        FOR p IN (SELECT permission_id FROM app_permissions 
                 WHERE permission_key IN ('VIEW_DASHBOARD', 'EDIT_PROFILE'))
        LOOP
            INSERT INTO app_role_permissions (role_id, permission_id, assigned_by)
            VALUES (r.role_id, p.permission_id, 'SYSTEM');
        END LOOP;
    END LOOP;
    
    -- Manager role permissions
    FOR r IN (SELECT role_id FROM app_roles WHERE role_name = 'MANAGER')
    LOOP
        FOR p IN (SELECT permission_id FROM app_permissions 
                 WHERE permission_key IN ('VIEW_DASHBOARD', 'EDIT_PROFILE', 'DEPT_REPORTS'))
        LOOP
            INSERT INTO app_role_permissions (role_id, permission_id, assigned_by)
            VALUES (r.role_id, p.permission_id, 'SYSTEM');
        END LOOP;
    END LOOP;
    
    -- Auditor role permissions
    FOR r IN (SELECT role_id FROM app_roles WHERE role_name = 'AUDITOR')
    LOOP
        FOR p IN (SELECT permission_id FROM app_permissions 
                 WHERE permission_key IN ('VIEW_DASHBOARD', 'VIEW_AUDIT'))
        LOOP
            INSERT INTO app_role_permissions (role_id, permission_id, assigned_by)
            VALUES (r.role_id, p.permission_id, 'SYSTEM');
        END LOOP;
    END LOOP;
    
    COMMIT;
END;
/
```

## 6. Security Best Practices

1. **Password Policies**:
   - Enforce strong password requirements
   - Implement password expiration and history
   - Use salted hash storage

2. **Session Management**:
   - Set appropriate session timeouts
   - Implement session fixation protection
   - Use secure cookies

3. **Audit Logging**:
   - Create audit tables:

```sql
CREATE TABLE app_audit_logs (
    log_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id VARCHAR2(100),
    username VARCHAR2(100),
    action_type VARCHAR2(50),
    action_description VARCHAR2(4000),
    ip_address VARCHAR2(50),
    user_agent VARCHAR2(1000),
    action_timestamp TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

COMMENT ON TABLE app_audit_logs IS 'System audit trail for security monitoring';
```

4. **Error Handling**:
   - Create a custom error handler that doesn't expose sensitive information
   - Log all errors for investigation
   - Present user-friendly error messages

## 7. Monitoring and Maintenance

### 7.1 Failed Login Attempts Monitoring

```sql
CREATE TABLE app_failed_logins (
    attempt_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username VARCHAR2(100),
    ip_address VARCHAR2(50),
    user_agent VARCHAR2(1000),
    attempt_timestamp TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE PROCEDURE log_failed_login(
    p_username IN VARCHAR2,
    p_ip_address IN VARCHAR2,
    p_user_agent IN VARCHAR2
) AS
BEGIN
    INSERT INTO app_failed_logins (username, ip_address, user_agent)
    VALUES (p_username, p_ip_address, p_user_agent);
    
    -- Check for brute force attempts
    DECLARE
        v_count NUMBER;
    BEGIN
        SELECT COUNT(*)
        INTO v_count
        FROM app_failed_logins
        WHERE username = p_username
        AND attempt_timestamp > SYSTIMESTAMP - INTERVAL '30' MINUTE;
        
        IF v_count >= 5 THEN
            -- Lock the account if it exists
            UPDATE app_users
            SET is_active = 'N'
            WHERE username = p_username;
            
            -- Log the account lock
            INSERT INTO app_audit_logs (
                user_id,
                username,
                action_type,
                action_description,
                ip_address,
                user_agent
            ) VALUES (
                NULL,
                p_username,
                'ACCOUNT_LOCK',
                'Account locked due to multiple failed login attempts',
                p_ip_address,
                p_user_agent
            );
        END IF;
    END;
END log_failed_login;
/
```

### 7.2 Cleanup Jobs

```sql
BEGIN
    DBMS_SCHEDULER.CREATE_JOB (
        job_name        => 'CLEANUP_OLD_SESSIONS',
        job_type        => 'PLSQL_BLOCK',
        job_action      => 'BEGIN
                             DELETE FROM app_failed_logins
                             WHERE attempt_timestamp < SYSTIMESTAMP - INTERVAL ''90'' DAY;
                             
                             COMMIT;
                           END;',
        start_date      => SYSTIMESTAMP,
        repeat_interval => 'FREQ=DAILY',
        enabled         => TRUE,
        comments        => 'Daily cleanup of old failed login records'
    );
END;
/
```

## 8. Deployment Steps

1. Create the database schema objects:
   - Authorization tables (users, roles, permissions)
   - Mapping tables (user_roles, role_permissions)
   - Audit and monitoring tables

2. Create PL/SQL packages:
   - Authentication functions
   - User data source selection
   - Authorization functions

3. Configure APEX Application:
   - Set up authentication scheme
   - Create authorization schemes
   - Apply authorization to application components

4. Set up data synchronization:
   - Deploy synchronization procedures
   - Schedule synchronization jobs

5. Initialize the system:
   - Create default roles and permissions
   - Assign initial role-permission mappings
   - Create admin user(s)

6. Test and validate:
   - Test authentication against both data sources
   - Verify role-based access control
   - Confirm audit logging functionality

## 9. Troubleshooting

### 9.1 Authentication Issues

Common issues and resolutions:

1. **LDAP Connection Failures**:
   - Check network connectivity to AD server
   - Verify LDAP configuration parameters
   - Test LDAP bind credentials

2. **Data Source Selection Problems**:
   - Enable debug logging in the selection procedure
   - Verify data exists in both sources with expected format
   - Check for exceptions in the post-authentication process

### 9.2 Authorization Issues

1. **Missing Permissions**:
   - Check user-role assignments
   - Verify role-permission mappings
   - Inspect the USER_PERMISSIONS collection contents

2. **Authorization Scheme Failures**:
   - Review authorization function implementation
   - Check for SQL errors in the logs
   - Test the authorization function directly

## 10. References

- [Oracle APEX Security Best Practices](https://apex.oracle.com/pls/apex/r/apex_pm/apex/security-best-practices)
- [Oracle Database Security Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/index.html)
- [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)