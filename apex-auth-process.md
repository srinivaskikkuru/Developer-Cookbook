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

- **Primary Method**: Social Login With Azure Integration
- **Secondary Support**: SAML authentication with Azure Integration
- **Enhanced Security**: Two-factor authentication for all application is mandatory

### 1.2 Authentication Flow

1. User submits credentials in the APEX login form
2. APEX validates credentials against the chosen authentication scheme
3. On successful authentication, our custom post-authentication process executes
4. The post-authentication process:
   - Retrieves employee data from appropriate source (ERP or Active Directory)
   - Establishes user session with proper role assignments
   - Sets application items for authorization decisions

### 1.3 Post-Authentication Data Source Selection
 ** sql for referring ERP data source ** 
 Prerequisites : Either onPremisesSamAccountName(Username) or Email Address has to be set as * Username * in authentication for social login or SAML
```sql for referring ERP data source
DECLARE
    l_user_name VARCHAR2(200);
	l_email_address VARCHAR2(200);
	l_employee_name VARCHAR2(200);
	l_supervisor_name VARCHAR2(200);
	l_supervisor_email_address VARCHAR2(200);
	l_supervisor_user_name VARCHAR2(200);
	
BEGIN
    BEGIN
        SELECT
            upper(user_name) user_name,
			lower(email_address) email_address,
			employee_name,
			position,
			supervisor_name,
			supervisor_email_address,
			supervisor_user_name
        INTO l_user_name,
		l_email_address,
		l_employee_name,
		l_supervisor_name,
		l_supervisor_email_address,
		l_supervisor_user_name		
        FROM
            hr_active_employees_mv 
        WHERE
            lower(user_name) = lower(v('APP_USER'))
            OR lower(email_address) = lower(v('APP_USER'));
    EXCEPTION
        WHEN OTHERS THEN
            l_user_name := NULL;
    END;

    IF l_user_name IS NOT NULL THEN
        apex_application.g_user := l_user_name;
        dbms_session.set_context('APEX$SESSION', 'APP_USER', l_user_name);
		
		-- apex_util.set_session_state('A_EMAIL_ADDRESS', l_email_address);  -- Create Application Page Item with name : A_EMAIL_ADDRESS , Scope : Application , Session State Protection : Unresticted
		-- apex_util.set_session_state('A_EMPLOYEE_NAME', l_employee_name); -- Create Application Page Item with name : A_EMPLOYEE_NAME , Scope : Application , Session State Protection : Unresticted
		-- apex_util.set_session_state('A_SUPERVISOR_NAME', l_supervisor_name); -- Create Application Page Item with name : A_SUPERVISOR_NAME , Scope : Application , Session State Protection : Unresticted
		-- apex_util.set_session_state('A_SUPERVISOR_EMAIL_ADDRESS', l_supervisor_email_address); -- Create Application Page Item with name : A_SUPERVISOR_EMAIL_ADDRESS , Scope : Application , Session State Protection : Unresticted
		-- apex_util.set_session_state('A_SUPERVISOR_USER_NAME', l_supervisor_user_name); -- Create Application Page Item with name : A_SUPERVISOR_USER_NAME , Scope : Application , Session State Protection : Unresticted
		-- Incase If any other values are required from HR data, assign it here itself
	ELSE 
		RAISE_APPLICATION_ERROR(-20001, 'Username not found in any HR data source');
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        RAISE_APPLICATION_ERROR(-20001, 'User not found in any data source');
END;
```
 ** sql for referring ERP data source ** 
 Prerequisites : Either EmailAddress or AzureObjectId has to be set as * Username * in authentication for social login. Avoid SAML authentication when configuring the Azure MFA.
```sql for referring Active Directory data source
DECLARE
    l_user_name VARCHAR2(200);
	l_email_address VARCHAR2(200);
	l_employee_name VARCHAR2(200);
	l_supervisor_name VARCHAR2(200);
	l_supervisor_email_address VARCHAR2(200);
	l_supervisor_user_name VARCHAR2(200);
	
BEGIN
    BEGIN
        SELECT
            upper(user_name) user_name,
			lower(email_address) email_address,
			employee_name,
			position,
			supervisor_name,
			supervisor_email_address,
			supervisor_user_name
        INTO l_user_name,
		l_email_address,
		l_employee_name,
		l_supervisor_name,
		l_supervisor_email_address,
		l_supervisor_user_name		
        FROM
            TABLE ( soa_infra_util_pkg.get_ad_employee_details(v('APP_USER')); -- APP_USER will have email address or AzureObjectId
    EXCEPTION
        WHEN OTHERS THEN
            l_user_name := NULL;
    END;

    IF l_user_name IS NOT NULL THEN
        apex_application.g_user := l_user_name;
        dbms_session.set_context('APEX$SESSION', 'APP_USER', l_user_name);
		
		-- apex_util.set_session_state('A_EMAIL_ADDRESS', l_email_address);  -- Create Application Page Item with name : A_EMAIL_ADDRESS , Scope : Application , Session State Protection : Unresticted
		-- apex_util.set_session_state('A_EMPLOYEE_NAME', l_employee_name); -- Create Application Page Item with name : A_EMPLOYEE_NAME , Scope : Application , Session State Protection : Unresticted
		-- apex_util.set_session_state('A_SUPERVISOR_NAME', l_supervisor_name); -- Create Application Page Item with name : A_SUPERVISOR_NAME , Scope : Application , Session State Protection : Unresticted
		-- apex_util.set_session_state('A_SUPERVISOR_EMAIL_ADDRESS', l_supervisor_email_address); -- Create Application Page Item with name : A_SUPERVISOR_EMAIL_ADDRESS , Scope : Application , Session State Protection : Unresticted
		-- apex_util.set_session_state('A_SUPERVISOR_USER_NAME', l_supervisor_user_name); -- Create Application Page Item with name : A_SUPERVISOR_USER_NAME , Scope : Application , Session State Protection : Unresticted
		-- Incase If any other values are required from HR data, assign it here itself
	ELSE 
		RAISE_APPLICATION_ERROR(-20001, 'Username not found in any HR data source');
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        RAISE_APPLICATION_ERROR(-20001, 'User not found in any data source');
END;
```
## 2. Authorization Schema

### 2.1 Database Schema for RBAC

We'll create the following tables to implement role-based access control:

#### 2.1.1 Users Table

```sql
CREATE TABLE app_users (
    user_id      number
        NOT NULL ENABLE,
    user_name    VARCHAR2(100 BYTE),
    created_on   DATE,
    created_by   VARCHAR2(100 BYTE),
    updated_on   DATE,
    updated_by   VARCHAR2(100 BYTE),
    active       VARCHAR2(1 BYTE),
    full_name    VARCHAR2(400 BYTE),
    CONSTRAINT app_users_username_uk UNIQUE (user_name),
    CONSTRAINT app_users_id_uk primary key (user_id)
);

COMMENT ON TABLE users IS 'Central registry of application users';
```

#### 2.1.2 Roles Table

```sql

CREATE TABLE app_roles (
    role_id                 NUMBER(5, 0)
        NOT NULL ENABLE,
    role               VARCHAR2(100 BYTE),
    created_by         VARCHAR2(100 BYTE),
    created_on         DATE,
    last_modified_by   VARCHAR2(100 BYTE),
    last_modified_on   DATE,
    CONSTRAINT app_roles_pk PRIMARY KEY ( role_id ),
    CONSTRAINT app_roles_name_uk UNIQUE (role)
);

COMMENT ON TABLE roles IS 'Application roles for authorization';
```

#### 2.1.3 Permissions Table

```sql

CREATE TABLE app_permissions (
    permission_id               NUMBER(5, 0)
        NOT NULL ENABLE,
    permission       VARCHAR2(200 BYTE)
        NOT NULL ENABLE,
    component_name   VARCHAR2(100 BYTE),
    category         VARCHAR2(100 BYTE),
    page_id          VARCHAR2(100 BYTE),
    perm_code        VARCHAR2(5 BYTE),
    CONSTRAINT app_permissions_pk PRIMARY KEY ( permission_id ),
    CONSTRAINT app_permissions_key_uk UNIQUE (permission)
);
COMMENT ON TABLE permissions IS 'Granular permissions that can be assigned to roles';
```

#### 2.1.4 User-Role Mapping

```sql
CREATE TABLE app_user_roles (
    id            NUMBER(38, 0)
        NOT NULL ENABLE,
    user_id       number
        NOT NULL ENABLE,
    role_id       number
        NOT NULL ENABLE,
    assigned_on   DATE
        NOT NULL ENABLE,
    assigned_by   VARCHAR2(100 BYTE)
        NOT NULL ENABLE,
    valid_from    TIMESTAMP(6),
    valid_until   TIMESTAMP(6),
    CONSTRAINT user_roles_pk PRIMARY KEY ( id ),
    CONSTRAINT app_user_roles_uk UNIQUE (user_id, role_id),
    CONSTRAINT app_user_roles_user_fk FOREIGN KEY (user_id) REFERENCES app_users (user_id),
    CONSTRAINT app_user_roles_role_fk FOREIGN KEY (role_id) REFERENCES app_roles (role_id)
);
COMMENT ON TABLE user_roles IS 'Mapping between users and their assigned roles';
```

#### 2.1.5 Role-Permission Mapping

```sql

CREATE TABLE app_roles_permissions (
    id              NUMBER(5, 0)
        NOT NULL ENABLE,
    role_id         NUMBER(5, 0)
        NOT NULL ENABLE,
    permission_id   NUMBER(5, 0)
        NOT NULL ENABLE,
    created_by      VARCHAR2(500 BYTE),
    created_on      DATE,
    CONSTRAINT roles_permissions_pk PRIMARY KEY ( id ),
    CONSTRAINT app_role_perms_role_fk FOREIGN KEY (role_id) REFERENCES app_roles (role_id),
    CONSTRAINT app_role_perms_perm_fk FOREIGN KEY (permission_id) REFERENCES app_permissions (permission_id)
);

COMMENT ON TABLE role_permissions IS 'Mapping between roles and their assigned permissions';
```

### 2.2 Authorization Functions

#### 2.2.1 Load User Roles Function

```sql
DECLARE
    l_count NUMBER;
BEGIN
    SELECT
        COUNT(*)
    INTO l_count
    FROM
        app_user au,
        app_roles ar,
        app_user_roles aur
    WHERE
            aur.user_id = au.user_id
        AND aur.role_id = ar.role_id
        AND upper(ar.role) = upper('ADMIN') -- Role Name
        AND upper(au.user) = upper(v('APP_USER'));

    IF l_count > 0 THEN
        RETURN TRUE;
    ELSE
        RETURN FALSE;
    END IF;
EXCEPTION
    WHEN OTHERS THEN
        RETURN FALSE;
END;
```


## 3. Integration with APEX Application

### 3.1 Application Authentication Setup for Social Login

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


## 5. Initialization and Setup
Create a Apex form for handling the users, roles and permissions. 
* This administration page should have restriction and should be allowed to be accessed only by ** "IT Admin Role" **
1) User Page -> This page will display the list of users and allow the mapping for users and roles. This form will have the button to add new users.
2) Roles Page -> This page will display the list of roles and allow the mapping for roles and permissions.This form will have the button to add new roles.
3) Permission Page -> This page will display the list of permissions. This form will have the button to add new permissions.

## 6. Security Best Practices

1. **Session Management**:
   - Set appropriate session timeouts
   - Implement session fixation protection
   - Use secure cookies
   - Always add authentication and authorization to all the pages

2. **Audit Logging**:
   - Create audit tables where ever applicable.

3. **Error Handling**:
   - Create a custom error handler that doesn't expose sensitive information
   - Log all errors for investigation
   - Present user-friendly error messages

## 7. Monitoring and Maintenance
### Error Reporting Page
Create a page to review the logs for capture during the runtime.
* This page access is restricted to IT Admin role only


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

4. Initialize the system:
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

1. **Active Directory Connection Failures**:
   - Check network connectivity to AD server
   - Verify AD configuration parameters like client id, secret key and scope

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
