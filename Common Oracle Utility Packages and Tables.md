# Common Oracle Utility Packages and Tables

*A centralized reference guide for shared database objects used across APEX applications*

Version 1.0.0 | April 2025

## Overview

This document catalogs the utility packages, tables, and other database objects that are maintained as shared resources across all applications in our Oracle ecosystem. Using these common utilities ensures consistency, reduces redundancy, and simplifies maintenance.

## How to Use This Guide

- **Development**: Reference this guide when building new functionality to avoid duplicating existing utilities
- **Maintenance**: Update this document when modifying or creating new shared objects
- **Onboarding**: Use as a reference for new team members to understand our common architecture

## Common Utility Packages

### 1. PKG_LOGGING

**Purpose**: Centralized error and event logging across all applications

**Location**: COMMON_SCHEMA

**Dependencies**: LOG_EVENTS table, LOG_ERRORS table

**Main Procedures/Functions**:

| Procedure/Function | Description | Example Usage |
|-------------------|-------------|--------------|
| PROC_LOG_ERROR | Logs error details with stack trace | `PKG_LOGGING.PROC_LOG_ERROR(p_error => SQLERRM, p_location => 'PKG_EMPLOYEE.PROC_UPDATE');` |
| PROC_LOG_EVENT | Logs application events | `PKG_LOGGING.PROC_LOG_EVENT(p_event_type => 'DATA_UPDATE', p_details => 'Employee record updated');` |
| FUNC_GET_ERRORS | Retrieves errors for a given time period | `SELECT * FROM TABLE(PKG_LOGGING.FUNC_GET_ERRORS(SYSDATE-7, SYSDATE));` |
| PROC_PURGE_LOGS | Removes logs older than specified date | `PKG_LOGGING.PROC_PURGE_LOGS(p_date => ADD_MONTHS(SYSDATE, -6));` |

**Creation Script**:

```sql
CREATE OR REPLACE PACKAGE pkg_logging AS
  PROCEDURE proc_log_error(
    p_error     IN VARCHAR2,
    p_location  IN VARCHAR2,
    p_user_id   IN NUMBER DEFAULT NVL(TO_NUMBER(SYS_CONTEXT('APEX$SESSION', 'APP_USER')), 0),
    p_app_id    IN NUMBER DEFAULT NVL(V('APP_ID'), 0)
  );
  
  PROCEDURE proc_log_event(
    p_event_type IN VARCHAR2,
    p_details    IN VARCHAR2,
    p_user_id    IN NUMBER DEFAULT NVL(TO_NUMBER(SYS_CONTEXT('APEX$SESSION', 'APP_USER')), 0),
    p_app_id     IN NUMBER DEFAULT NVL(V('APP_ID'), 0)
  );
  
  FUNCTION func_get_errors(
    p_start_date IN DATE,
    p_end_date   IN DATE
  ) RETURN SYS_REFCURSOR;
  
  PROCEDURE proc_purge_logs(
    p_date IN DATE
  );
END pkg_logging;
/
```

### 2. PKG_EMAIL

**Purpose**: Standardized email functionality for all applications

**Location**: COMMON_SCHEMA

**Dependencies**: EMAIL_TEMPLATES table

**Main Procedures/Functions**:

| Procedure/Function | Description | Example Usage |
|-------------------|-------------|--------------|
| PROC_SEND_EMAIL | Sends email with optional attachments | `PKG_EMAIL.PROC_SEND_EMAIL(p_to => 'user@example.com', p_subject => 'Notification', p_body => 'Hello');` |
| PROC_SEND_TEMPLATE | Sends email using a template | `PKG_EMAIL.PROC_SEND_TEMPLATE(p_template_code => 'WELCOME_EMAIL', p_to => 'user@example.com', p_params => params);` |
| FUNC_GET_TEMPLATE | Retrieves an email template | `v_template := PKG_EMAIL.FUNC_GET_TEMPLATE('PASSWORD_RESET');` |

**Creation Script**:

```sql
CREATE OR REPLACE PACKAGE pkg_email AS
  PROCEDURE proc_send_email(
    p_to        IN VARCHAR2,
    p_subject   IN VARCHAR2,
    p_body      IN CLOB,
    p_body_html IN CLOB DEFAULT NULL,
    p_from      IN VARCHAR2 DEFAULT 'system@example.com',
    p_cc        IN VARCHAR2 DEFAULT NULL,
    p_bcc       IN VARCHAR2 DEFAULT NULL
  );
  
  PROCEDURE proc_send_template(
    p_template_code IN VARCHAR2,
    p_to            IN VARCHAR2,
    p_params        IN apex_application_global.vc_arr2,
    p_from          IN VARCHAR2 DEFAULT 'system@example.com',
    p_cc            IN VARCHAR2 DEFAULT NULL,
    p_bcc           IN VARCHAR2 DEFAULT NULL
  );
  
  FUNCTION func_get_template(
    p_template_code IN VARCHAR2
  ) RETURN CLOB;
END pkg_email;
/
```

### 3. PKG_SECURITY

**Purpose**: Security-related utilities for authentication and authorization

**Location**: COMMON_SCHEMA

**Dependencies**: USER_ROLES table, ACCESS_CONTROL table

**Main Procedures/Functions**:

| Procedure/Function | Description | Example Usage |
|-------------------|-------------|--------------|
| FUNC_HAS_ROLE | Checks if user has a specific role | `IF PKG_SECURITY.FUNC_HAS_ROLE('ADMIN') THEN...` |
| FUNC_ENCRYPT | Encrypts sensitive data | `v_encrypted := PKG_SECURITY.FUNC_ENCRYPT(p_value);` |
| FUNC_DECRYPT | Decrypts encrypted data | `v_decrypted := PKG_SECURITY.FUNC_DECRYPT(p_encrypted);` |
| FUNC_HASH_PASSWORD | Creates secure password hash | `v_hash := PKG_SECURITY.FUNC_HASH_PASSWORD(p_password);` |
| PROC_AUDIT_ACCESS | Records access attempts | `PKG_SECURITY.PROC_AUDIT_ACCESS(p_resource => 'EMP_DATA', p_success => TRUE);` |

**Creation Script**:

```sql
CREATE OR REPLACE PACKAGE pkg_security AS
  FUNCTION func_has_role(
    p_role_code IN VARCHAR2,
    p_user_id   IN NUMBER DEFAULT NVL(TO_NUMBER(SYS_CONTEXT('APEX$SESSION', 'APP_USER')), 0)
  ) RETURN BOOLEAN;
  
  FUNCTION func_encrypt(
    p_value IN VARCHAR2
  ) RETURN RAW;
  
  FUNCTION func_decrypt(
    p_encrypted IN RAW
  ) RETURN VARCHAR2;
  
  FUNCTION func_hash_password(
    p_password IN VARCHAR2
  ) RETURN VARCHAR2;
  
  PROCEDURE proc_audit_access(
    p_resource IN VARCHAR2,
    p_success  IN BOOLEAN,
    p_user_id  IN NUMBER DEFAULT NVL(TO_NUMBER(SYS_CONTEXT('APEX$SESSION', 'APP_USER')), 0),
    p_app_id   IN NUMBER DEFAULT NVL(V('APP_ID'), 0)
  );
END pkg_security;
/
```

### 4. PKG_UTILS

**Purpose**: Generic utility functions used across applications

**Location**: COMMON_SCHEMA

**Dependencies**: None

**Main Procedures/Functions**:

| Procedure/Function | Description | Example Usage |
|-------------------|-------------|--------------|
| FUNC_FORMAT_PHONE | Formats phone numbers consistently | `v_formatted := PKG_UTILS.FUNC_FORMAT_PHONE('+1 (555) 123-4567');` |
| FUNC_VALIDATE_EMAIL | Validates email format | `IF PKG_UTILS.FUNC_VALIDATE_EMAIL('user@example.com') THEN...` |
| FUNC_TO_NUMBER | Safely converts string to number | `v_num := PKG_UTILS.FUNC_TO_NUMBER('123.45', 0);` |
| FUNC_DATE_DIFF | Gets difference between dates in specified unit | `v_days := PKG_UTILS.FUNC_DATE_DIFF(SYSDATE, SYSDATE-10, 'DAY');` |

**Creation Script**:

```sql
CREATE OR REPLACE PACKAGE pkg_utils AS
  FUNCTION func_format_phone(
    p_phone IN VARCHAR2
  ) RETURN VARCHAR2;
  
  FUNCTION func_validate_email(
    p_email IN VARCHAR2
  ) RETURN BOOLEAN;
  
  FUNCTION func_to_number(
    p_value    IN VARCHAR2,
    p_default  IN NUMBER DEFAULT NULL
  ) RETURN NUMBER;
  
  FUNCTION func_date_diff(
    p_date1 IN DATE,
    p_date2 IN DATE,
    p_unit  IN VARCHAR2 DEFAULT 'DAY'
  ) RETURN NUMBER;
END pkg_utils;
/
```

### 5. PKG_FILE_UTILS

**Purpose**: File management utilities

**Location**: COMMON_SCHEMA

**Dependencies**: FILES table, FILE_CATEGORIES table

**Main Procedures/Functions**:

| Procedure/Function | Description | Example Usage |
|-------------------|-------------|--------------|
| PROC_STORE_FILE | Stores file in database | `PKG_FILE_UTILS.PROC_STORE_FILE(p_filename => 'report.pdf', p_blob => l_blob);` |
| FUNC_GET_FILE | Retrieves file by ID | `v_file := PKG_FILE_UTILS.FUNC_GET_FILE(123);` |
| PROC_DELETE_FILE | Removes file from storage | `PKG_FILE_UTILS.PROC_DELETE_FILE(123);` |

**Creation Script**:

```sql
CREATE OR REPLACE PACKAGE pkg_file_utils AS
  PROCEDURE proc_store_file(
    p_filename     IN VARCHAR2,
    p_blob         IN BLOB,
    p_mime_type    IN VARCHAR2 DEFAULT 'application/octet-stream',
    p_category     IN VARCHAR2 DEFAULT 'GENERAL',
    p_description  IN VARCHAR2 DEFAULT NULL,
    p_file_id      OUT NUMBER
  );
  
  FUNCTION func_get_file(
    p_file_id IN NUMBER
  ) RETURN BLOB;
  
  PROCEDURE proc_delete_file(
    p_file_id IN NUMBER
  );
END pkg_file_utils;
/
```

## Common Tables

### 1. LOG_EVENTS

**Purpose**: Stores application event logs

**Schema**: COMMON_SCHEMA

**Columns**:

| Column Name | Data Type | Description |
|-------------|-----------|-------------|
| LOG_ID | NUMBER | Primary key |
| EVENT_DATE | TIMESTAMP | When the event occurred |
| APP_ID | NUMBER | Application ID |
| USER_ID | NUMBER | User ID |
| EVENT_TYPE | VARCHAR2(100) | Type of event (e.g., LOGIN, UPDATE) |
| DETAILS | CLOB | Event details |
| IP_ADDRESS | VARCHAR2(50) | User's IP address |
| SESSION_ID | VARCHAR2(255) | Session identifier |

**Creation Script**:

```sql
CREATE TABLE log_events (
    log_id       NUMBER GENERATED ALWAYS AS IDENTITY,
    event_date   TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
    app_id       NUMBER,
    user_id      NUMBER,
    event_type   VARCHAR2(100) NOT NULL,
    details      CLOB,
    ip_address   VARCHAR2(50),
    session_id   VARCHAR2(255),
    CONSTRAINT pk_log_events PRIMARY KEY (log_id)
);

CREATE INDEX idx_log_events_date ON log_events(event_date);
CREATE INDEX idx_log_events_app_user ON log_events(app_id, user_id);
```

### 2. LOG_ERRORS

**Purpose**: Stores application error logs

**Schema**: COMMON_SCHEMA

**Columns**:

| Column Name | Data Type | Description |
|-------------|-----------|-------------|
| ERROR_ID | NUMBER | Primary key |
| ERROR_DATE | TIMESTAMP | When the error occurred |
| APP_ID | NUMBER | Application ID |
| USER_ID | NUMBER | User ID |
| ERROR_MESSAGE | VARCHAR2(4000) | Error message |
| ERROR_BACKTRACE | CLOB | Error stack trace |
| ERROR_LOCATION | VARCHAR2(255) | Where error occurred |
| PAGE_ID | NUMBER | APEX page ID if applicable |

**Creation Script**:

```sql
CREATE TABLE log_errors (
    error_id         NUMBER GENERATED ALWAYS AS IDENTITY,
    error_date       TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
    app_id           NUMBER,
    user_id          NUMBER,
    error_message    VARCHAR2(4000),
    error_backtrace  CLOB,
    error_location   VARCHAR2(255),
    page_id          NUMBER,
    CONSTRAINT pk_log_errors PRIMARY KEY (error_id)
);

CREATE INDEX idx_log_errors_date ON log_errors(error_date);
CREATE INDEX idx_log_errors_app ON log_errors(app_id);
```

### 3. EMAIL_TEMPLATES

**Purpose**: Stores email templates for use across applications

**Schema**: COMMON_SCHEMA

**Columns**:

| Column Name | Data Type | Description |
|-------------|-----------|-------------|
| TEMPLATE_ID | NUMBER | Primary key |
| TEMPLATE_CODE | VARCHAR2(50) | Unique template identifier |
| SUBJECT | VARCHAR2(500) | Email subject line template |
| BODY_TEXT | CLOB | Plain text email body template |
| BODY_HTML | CLOB | HTML email body template |
| CREATED_BY | VARCHAR2(255) | Creator username |
| CREATED_DATE | DATE | Creation date |
| UPDATED_BY | VARCHAR2(255) | Last updater username |
| UPDATED_DATE | DATE | Last update date |

**Creation Script**:

```sql
CREATE TABLE email_templates (
    template_id     NUMBER GENERATED ALWAYS AS IDENTITY,
    template_code   VARCHAR2(50) NOT NULL,
    subject         VARCHAR2(500) NOT NULL,
    body_text       CLOB NOT NULL,
    body_html       CLOB,
    created_by      VARCHAR2(255),
    created_date    DATE DEFAULT SYSDATE,
    updated_by      VARCHAR2(255),
    updated_date    DATE,
    CONSTRAINT pk_email_templates PRIMARY KEY (template_id),
    CONSTRAINT uk_email_template_code UNIQUE (template_code)
);
```

### 4. APP_PARAMETERS

**Purpose**: Centralized application parameters/settings

**Schema**: COMMON_SCHEMA

**Columns**:

| Column Name | Data Type | Description |
|-------------|-----------|-------------|
| PARAM_ID | NUMBER | Primary key |
| APP_ID | NUMBER | Application ID (null for global) |
| PARAM_NAME | VARCHAR2(100) | Parameter name |
| PARAM_VALUE | VARCHAR2(4000) | Parameter value |
| PARAM_DESCRIPTION | VARCHAR2(1000) | Description of parameter |
| IS_ENCRYPTED | CHAR(1) | Y/N flag if value is encrypted |

**Creation Script**:

```sql
CREATE TABLE app_parameters (
    param_id          NUMBER GENERATED ALWAYS AS IDENTITY,
    app_id            NUMBER,
    param_name        VARCHAR2(100) NOT NULL,
    param_value       VARCHAR2(4000),
    param_description VARCHAR2(1000),
    is_encrypted      CHAR(1) DEFAULT 'N',
    CONSTRAINT pk_app_parameters PRIMARY KEY (param_id),
    CONSTRAINT uk_app_param UNIQUE (app_id, param_name),
    CONSTRAINT ck_is_encrypted CHECK (is_encrypted IN ('Y', 'N'))
);
```

### 5. USER_ROLES

**Purpose**: Maps users to roles for authorization

**Schema**: COMMON_SCHEMA

**Columns**:

| Column Name | Data Type | Description |
|-------------|-----------|-------------|
| USER_ROLE_ID | NUMBER | Primary key |
| USER_ID | NUMBER | User ID |
| ROLE_CODE | VARCHAR2(50) | Role identifier |
| ASSIGNED_DATE | DATE | When role was assigned |
| ASSIGNED_BY | NUMBER | User ID who assigned role |
| EXPIRY_DATE | DATE | When role expires (if applicable) |

**Creation Script**:

```sql
CREATE TABLE user_roles (
    user_role_id   NUMBER GENERATED ALWAYS AS IDENTITY,
    user_id        NUMBER NOT NULL,
    role_code      VARCHAR2(50) NOT NULL,
    assigned_date  DATE DEFAULT SYSDATE NOT NULL,
    assigned_by    NUMBER,
    expiry_date    DATE,
    CONSTRAINT pk_user_roles PRIMARY KEY (user_role_id),
    CONSTRAINT uk_user_role UNIQUE (user_id, role_code)
);

CREATE INDEX idx_user_roles_user ON user_roles(user_id);
```

### 6. FILES

**Purpose**: Stores uploaded files

**Schema**: COMMON_SCHEMA

**Columns**:

| Column Name | Data Type | Description |
|-------------|-----------|-------------|
| FILE_ID | NUMBER | Primary key |
| FILENAME | VARCHAR2(255) | Original filename |
| FILE_MIME_TYPE | VARCHAR2(255) | MIME type |
| FILE_CHARSET | VARCHAR2(128) | Character set if applicable |
| FILE_BLOB | BLOB | File contents |
| FILE_COMMENTS | VARCHAR2(4000) | Comments |
| TAGS | VARCHAR2(4000) | Tags for searching |
| CREATED_BY | NUMBER | User who created record |
| CREATED_DATE | DATE | Creation date |
| CATEGORY_ID | NUMBER | FK to FILE_CATEGORIES |

**Creation Script**:

```sql
CREATE TABLE files (
    file_id         NUMBER GENERATED ALWAYS AS IDENTITY,
    filename        VARCHAR2(255) NOT NULL,
    file_mime_type  VARCHAR2(255),
    file_charset    VARCHAR2(128),
    file_blob       BLOB NOT NULL,
    file_comments   VARCHAR2(4000),
    tags            VARCHAR2(4000),
    created_by      NUMBER,
    created_date    DATE DEFAULT SYSDATE,
    category_id     NUMBER,
    CONSTRAINT pk_files PRIMARY KEY (file_id)
);

CREATE INDEX idx_files_filename ON files(filename);
CREATE INDEX idx_files_category ON files(category_id);
```

## Common Views

### 1. V_USER_DETAILS

**Purpose**: Consolidated user information from multiple sources

**Schema**: COMMON_SCHEMA

**Base Tables**: USERS, USER_ROLES, DEPARTMENTS

**Creation Script**:

```sql
CREATE OR REPLACE VIEW v_user_details AS
SELECT 
    u.user_id,
    u.username,
    u.email,
    u.first_name,
    u.last_name,
    u.first_name || ' ' || u.last_name AS full_name,
    d.department_name,
    d.department_code,
    LISTAGG(r.role_code, ',') WITHIN GROUP (ORDER BY r.role_code) AS user_roles,
    u.created_date,
    u.last_login_date
FROM 
    users u
LEFT JOIN 
    departments d ON u.department_id = d.department_id
LEFT JOIN 
    user_roles ur ON u.user_id = ur.user_id
LEFT JOIN 
    roles r ON ur.role_code = r.role_code
WHERE 
    (ur.expiry_date IS NULL OR ur.expiry_date > SYSDATE)
GROUP BY
    u.user_id,
    u.username,
    u.email,
    u.first_name,
    u.last_name,
    d.department_name,
    d.department_code,
    u.created_date,
    u.last_login_date;
```

### 2. V_ERROR_SUMMARY

**Purpose**: Summarizes error logs for reporting

**Schema**: COMMON_SCHEMA

**Base Tables**: LOG_ERRORS

**Creation Script**:

```sql
CREATE OR REPLACE VIEW v_error_summary AS
SELECT 
    TRUNC(error_date) AS error_day,
    app_id,
    COUNT(*) AS error_count,
    COUNT(DISTINCT user_id) AS affected_users,
    MIN(error_date) AS first_occurrence,
    MAX(error_date) AS last_occurrence,
    LISTAGG(DISTINCT error_location, ', ') 
        WITHIN GROUP (ORDER BY error_location) AS error_locations
FROM 
    log_errors
WHERE 
    error_date > SYSDATE - 30
GROUP BY 
    TRUNC(error_date),
    app_id
ORDER BY 
    error_day DESC,
    app_id;
```

## Database Jobs

### 1. JOB_PURGE_LOGS

**Purpose**: Automatically purges old log records

**Schedule**: Runs weekly on Sunday at 1 AM

**Creation Script**:

```sql
BEGIN
    DBMS_SCHEDULER.CREATE_JOB (
        job_name        => 'JOB_PURGE_LOGS',
        job_type        => 'STORED_PROCEDURE',
        job_action      => 'PKG_LOGGING.PROC_PURGE_LOGS',
        start_date      => SYSTIMESTAMP,
        repeat_interval => 'FREQ=WEEKLY; BYDAY=SUN; BYHOUR=1',
        enabled         => TRUE,
        comments        => 'Weekly job to purge logs older than 6 months'
    );
    
    DBMS_SCHEDULER.SET_JOB_ARGUMENT_VALUE (
        job_name          => 'JOB_PURGE_LOGS',
        argument_position => 1,
        argument_value    => 'ADD_MONTHS(SYSDATE, -6)'
    );
END;
/
```

### 2. JOB_CLEANUP_FILES

**Purpose**: Removes temporary files

**Schedule**: Runs daily at 2 AM

**Creation Script**:

```sql
BEGIN
    DBMS_SCHEDULER.CREATE_JOB (
        job_name        => 'JOB_CLEANUP_FILES',
        job_type        => 'PLSQL_BLOCK',
        job_action      => 'BEGIN
                              DELETE FROM files 
                              WHERE category_id = (SELECT category_id FROM file_categories WHERE category_code = ''TEMP'')
                              AND created_date < SYSDATE - 7;
                              COMMIT;
                            END;',
        start_date      => SYSTIMESTAMP,
        repeat_interval => 'FREQ=DAILY; BYHOUR=2',
        enabled         => TRUE,
        comments        => 'Daily job to remove temporary files older than 7 days'
    );
END;
/
```

## Installation and Setup

### Dependencies

Before installing these utilities, ensure:

1. The COMMON_SCHEMA user exists with appropriate privileges
2. Oracle database version 19c or higher is used
3. APEX version 21.1 or higher is installed

### Installation Steps

1. **Create Tables**:
   Execute all table creation scripts in the order listed

2. **Create Views**:
   Execute all view creation scripts

3. **Create Packages**:
   Execute package specs first, then package bodies

4. **Create Jobs**:
   Execute job creation scripts

5. **Grant Permissions**:
   Grant appropriate access to application schemas

```sql
-- Example grants to an application schema
GRANT EXECUTE ON common_schema.pkg_logging TO app_schema;
GRANT EXECUTE ON common_schema.pkg_email TO app_schema;
GRANT EXECUTE ON common_schema.pkg_security TO app_schema;
GRANT EXECUTE ON common_schema.pkg_utils TO app_schema;
GRANT EXECUTE ON common_schema.pkg_file_utils TO app_schema;

GRANT SELECT ON common_schema.v_user_details TO app_schema;
GRANT SELECT ON common_schema.v_error_summary TO app_schema;
```

## Usage Examples

### Implementing Logging

```sql
-- In your application code
BEGIN
  -- Your business logic here
  UPDATE employees SET salary = salary * 1.1 WHERE department_id = 10;
  
  -- Log the event
  pkg_logging.proc_log_event(
    p_event_type => 'SALARY_UPDATE',
    p_details    => 'Applied 10% increase to Department 10'
  );
EXCEPTION
  WHEN OTHERS THEN
    -- Log the error
    pkg_logging.proc_log_error(
      p_error    => SQLERRM,
      p_location => 'PROC_UPDATE_SALARIES'
    );
    RAISE;
END;
```

### Sending Emails

```sql
DECLARE
  l_params apex_application_global.vc_arr2;
BEGIN
  -- Set template parameters
  l_params(1) := 'John Doe';
  l_params(2) := 'https://example.com/reset?token=abc123';
  
  -- Send email using template
  pkg_email.proc_send_template(
    p_template_code => 'PASSWORD_RESET',
    p_to            => 'john.doe@example.com',
    p_params        => l_params
  );
END;
```

### Using Security Functions

```sql
-- In an authorization scheme or process
DECLARE
  l_is_authorized BOOLEAN;
BEGIN
  -- Check if user has required role
  l_is_authorized := pkg_security.func_has_role('ADMIN');
  
  IF NOT l_is_authorized THEN
    -- Log unauthorized access attempt
    pkg_security.proc_audit_access(
      p_resource => 'ADMIN_CONSOLE',
      p_success  => FALSE
    );
    
    -- Raise appropriate error
    apex_error.add_error(
      p_message           => 'You are not authorized to access this resource',
      p_display_location  => apex_error.c_inline_in_notification
    );
  END IF;
END;
```

## Contributing

To add a new utility package or table to this shared repository:

1. Submit a request to the DBA team
2. Include full documentation and test cases
3. Ensure backwards compatibility
4. Follow the naming and coding standards in this document

## Contacts

For questions about these utilities, contact:

- **Database Administration**: dba-team@example.com
- **APEX Development Team**: apex-dev@example.com

---

*This document is maintained by the Database Team. Last updated: April 2025.*
