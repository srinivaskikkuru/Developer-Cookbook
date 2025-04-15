# Oracle SQL & PL/SQL Coding Standards Cookbook

![Oracle Database](https://img.shields.io/badge/Oracle-Database-F80000?style=for-the-badge&logo=oracle&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-Standards-blue?style=for-the-badge)
![PL/SQL](https://img.shields.io/badge/PL/SQL-Best_Practices-orange?style=for-the-badge)

## Table of Contents

- [Introduction](#introduction)
- [General Formatting Guidelines](#general-formatting-guidelines)
- [Naming Conventions](#naming-conventions)
- [SQL Coding Standards](#sql-coding-standards)
- [PL/SQL Coding Standards](#plsql-coding-standards)
- [Performance Considerations](#performance-considerations)
- [Error Handling](#error-handling)
- [Security Best Practices](#security-best-practices)
- [Code Documentation](#code-documentation)
- [Version Control Practices](#version-control-practices)
- [Testing Standards](#testing-standards)
- [Oracle-Specific Features](#oracle-specific-features)
- [Contributing](#contributing)
- [License](#license)

## Introduction

This cookbook provides standardized guidelines for writing SQL and PL/SQL code in Oracle environments. Following these standards ensures consistency, readability, maintainability, and performance across database projects. These standards align with Oracle's recommended practices and industry best practices for database development.

### Purpose

- Establish uniform coding practices across development teams
- Improve code readability and maintainability
- Enhance application performance and security
- Facilitate code reviews and knowledge sharing
- Reduce bugs and technical debt

### How to Use This Cookbook

This document serves as a reference guide for developers. Teams should review and adapt these standards to their specific project needs while maintaining the core principles.

## General Formatting Guidelines

### Indentation and Line Breaks

- Use consistent indentation of 2 or 4 spaces (not tabs)
- Limit line length to 80-120 characters
- Break long statements into multiple lines for readability
- Align similar elements vertically when breaking lines

```sql
-- Good practice
SELECT employee_id,
       first_name,
       last_name,
       salary
FROM   employees
WHERE  department_id = 10
AND    hire_date > TO_DATE('01-JAN-2020', 'DD-MON-YYYY');

-- Avoid
SELECT employee_id, first_name, last_name, salary FROM employees WHERE department_id = 10 AND hire_date > TO_DATE('01-JAN-2020', 'DD-MON-YYYY');
```

### Case Conventions

- Use UPPERCASE for all SQL keywords and Oracle built-in functions
- Use lowercase for schema objects, variables, and user-defined identifiers
- Be consistent throughout your codebase

```sql
-- Good practice
SELECT employee_id, last_name
FROM   employees
WHERE  department_id IN (SELECT department_id 
                         FROM   departments
                         WHERE  location_id = 1700);

-- Avoid mixing cases
Select Employee_Id, Last_Name
From   EMPLOYEES
where  Department_Id IN (select DEPARTMENT_ID 
                         from   Departments
                         Where  LOCATION_ID = 1700);
```

### Whitespace Usage

- Use spaces around operators for readability
- Include spaces after commas in lists
- Use parentheses to clearly show operation precedence
- Add blank lines to separate logical code blocks

```sql
-- Good practice (with appropriate whitespace)
UPDATE employees
SET    salary = salary * 1.1,
       commission_pct = commission_pct + 0.05
WHERE  department_id = 50
AND    hire_date < ADD_MONTHS(SYSDATE, -60);

-- Avoid (cramped without whitespace)
UPDATE employees SET salary=salary*1.1,commission_pct=commission_pct+0.05 WHERE department_id=50 AND hire_date<ADD_MONTHS(SYSDATE,-60);
```

### Comments

- Add header comments for all code modules
- Include inline comments for complex logic
- Use consistent comment formatting
- Comment why, not what (the code already shows what)

```sql
/*
* Purpose: Calculate employee bonuses based on department and performance
* Author:  Jane Smith
* Created: 15-APR-2025
* Modified: N/A
*/
SELECT e.employee_id,
       e.last_name,
       -- Higher bonus multiplier for sales department
       CASE WHEN e.department_id = 80 THEN e.salary * 0.15
            ELSE e.salary * 0.10
       END AS bonus_amount
FROM   employees e
WHERE  e.hire_date < ADD_MONTHS(SYSDATE, -12); -- Only for employees with 1+ year tenure
```

## Naming Conventions

### General Naming Guidelines

- Use meaningful, descriptive names
- Keep names concise but clear
- Use singular nouns for tables representing entities
- Avoid Oracle reserved words
- Use underscore (_) to separate words
- Limit name length to 30 characters (Oracle constraint)

### Database Object Naming Conventions

| Object Type | Prefix | Example | Notes |
|-------------|--------|---------|-------|
| Table | None | employees | Singular noun |
| View | v_ | v_employee_details | Describes the view's purpose |
| Materialized View | mv_ | mv_monthly_sales | Describes the contained data |
| Sequence | seq_ | seq_employee_id | Related to the table and column it serves |
| Index | idx_ | idx_emp_last_name | Indicates indexed column(s) |
| Primary Key Constraint | pk_ | pk_employees | Indicates table |
| Foreign Key Constraint | fk_ | fk_emp_dept | Indicates relationship |
| Unique Constraint | uk_ | uk_emp_email | Indicates unique column(s) |
| Check Constraint | ck_ | ck_salary_minimum | Describes the condition |
| Trigger | trg_ | trg_emp_audit | Describes the trigger's action |
| Stored Procedure | p_ | p_process_payroll | Verb indicating action |
| Function | f_ | f_calculate_bonus | Verb indicating action |
| Package | pkg_ | pkg_employee_mgmt | Describes the package's purpose |
| Type | typ_ | typ_employee_record | Describes the type's purpose |

### PL/SQL Variable Naming

| Variable Type | Convention | Example | Notes |
|---------------|------------|---------|-------|
| Constants | c_ (uppercase name) | c_MAX_SALARY | All caps after prefix |
| Global Variables | g_ | g_current_user | Indicates global scope |
| Parameters | p_ | p_employee_id | Indicates parameter status |
| Local Variables | l_ or descriptive name | l_total_salary | Clear name |
| Record Types | r_ | r_employee | Used for record variables |
| Cursors | cur_ | cur_departments | Describes data set |
| Boolean Variables | is_, has_, can_ | is_active, has_children | Indicates boolean nature |

## SQL Coding Standards

### SELECT Statements

- Always specify column names instead of using `SELECT *`
- Use table aliases for multi-table queries and qualify column names
- Place each selected column on a separate line for complex queries
- Include column aliases for calculated fields or when clarification is needed
- Order JOIN clauses consistently, typically with the largest table first
- Use ANSI JOIN syntax instead of Oracle's proprietary syntax

```sql
-- Good practice
SELECT e.employee_id,
       e.last_name,
       d.department_name,
       e.salary * 12 AS annual_salary
FROM   employees e
JOIN   departments d ON e.department_id = d.department_id
WHERE  e.hire_date > TO_DATE('01-JAN-2020', 'DD-MON-YYYY');

-- Avoid
SELECT *
FROM employees, departments
WHERE employees.department_id = departments.department_id
AND hire_date > TO_DATE('01-JAN-2020', 'DD-MON-YYYY');
```

### WHERE Clauses

- Place each condition on a separate line for complex conditions
- Use parentheses to clarify logical grouping
- Use IS NULL/IS NOT NULL instead of =NULL/!=NULL
- Use IN for multiple OR conditions on the same column
- Use BETWEEN for range queries when appropriate
- Use LIKE sparingly and consider performance implications

```sql
-- Good practice
SELECT employee_id, last_name
FROM   employees
WHERE  department_id IN (10, 20, 30)
AND    salary BETWEEN 5000 AND 10000
AND    commission_pct IS NOT NULL;

-- Avoid
SELECT employee_id, last_name
FROM   employees
WHERE  department_id = 10 OR department_id = 20 OR department_id = 30
AND    salary >= 5000 AND salary <= 10000
AND    commission_pct != NULL; -- This will not work as expected
```

### Data Manipulation Language (DML)

- Always specify column names in INSERT statements
- Include a WHERE clause in UPDATE and DELETE statements
- Use multi-table INSERT/UPDATE/DELETE with caution
- Consider using MERGE for conditional insert/update operations
- Use the RETURNING clause to get affected rows when appropriate

```sql
-- Good practice
INSERT INTO departments
  (department_id, department_name, location_id)
VALUES
  (280, 'Data Science', 1700);

UPDATE employees
SET    salary = salary * 1.1,
       last_update_date = SYSDATE
WHERE  department_id = 20;

-- Avoid
INSERT INTO departments
VALUES (280, 'Data Science', NULL, NULL, 1700);

UPDATE employees
SET salary = salary * 1.1; -- Missing WHERE clause!
```

### Transactions

- Keep transactions as short as possible
- Use explicit COMMIT and ROLLBACK statements
- Avoid mixing DDL and DML in the same transaction
- Consider using SAVEPOINT for complex transactions
- Always handle exceptions that could leave transactions open

```sql
-- Good practice
BEGIN
  INSERT INTO departments (department_id, department_name, location_id)
  VALUES (280, 'Data Science', 1700);
  
  UPDATE employees
  SET    department_id = 280
  WHERE  employee_id IN (107, 108, 109);
  
  COMMIT;
EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
    RAISE;
END;
```

## PL/SQL Coding Standards

### Package Structure

- Divide functionality into logical packages
- Include package specification headers with purpose and author
- Declare constants, types, and exceptions before procedures and functions
- Order package body elements to match the specification
- Implement private procedures and functions at the end

```sql
CREATE OR REPLACE PACKAGE pkg_employee_mgmt AS
  /*
  * Purpose: Employee management operations package
  * Author: John Smith
  * Created: 15-APR-2025
  */
  
  -- Constants
  c_DEFAULT_DEPT CONSTANT NUMBER := 10;
  
  -- Types
  TYPE t_emp_ids IS TABLE OF employees.employee_id%TYPE;
  
  -- Exceptions
  e_invalid_department EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_invalid_department, -20001);
  
  -- Public procedures and functions
  PROCEDURE p_hire_employee(
    p_first_name  IN VARCHAR2,
    p_last_name   IN VARCHAR2,
    p_email       IN VARCHAR2,
    p_department  IN NUMBER DEFAULT c_DEFAULT_DEPT,
    p_job_id      IN VARCHAR2,
    p_employee_id OUT NUMBER
  );
  
  FUNCTION f_calculate_bonus(
    p_employee_id IN NUMBER
  ) RETURN NUMBER;
  
END pkg_employee_mgmt;
```

### Procedure and Function Structure

- Include a clear header comment for each module
- Handle parameters consistently (IN, OUT, IN OUT)
- Document parameters with inline comments
- Add validation for all input parameters
- Implement proper error handling
- Return meaningful values from functions

```sql
CREATE OR REPLACE FUNCTION f_calculate_bonus(
  p_employee_id IN NUMBER   -- Employee ID to calculate bonus for
) RETURN NUMBER IS
  /*
  * Purpose: Calculate employee bonus based on salary and department
  * Returns: Bonus amount in same currency as salary
  */
  l_salary     NUMBER;
  l_dept_id    NUMBER;
  l_bonus_pct  NUMBER := 0.1;  -- Default bonus percentage
  l_bonus      NUMBER;
BEGIN
  -- Validate input
  IF p_employee_id IS NULL THEN
    RAISE_APPLICATION_ERROR(-20001, 'Employee ID cannot be null');
  END IF;
  
  -- Get employee details
  SELECT salary, department_id
  INTO   l_salary, l_dept_id
  FROM   employees
  WHERE  employee_id = p_employee_id;
  
  -- Determine bonus percentage based on department
  CASE l_dept_id
    WHEN 80 THEN l_bonus_pct := 0.15;  -- Sales
    WHEN 20 THEN l_bonus_pct := 0.12;  -- Research
    ELSE l_bonus_pct := 0.1;           -- Default
  END CASE;
  
  -- Calculate bonus
  l_bonus := l_salary * l_bonus_pct;
  
  RETURN l_bonus;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    RAISE_APPLICATION_ERROR(-20002, 'Employee ID ' || p_employee_id || ' not found');
  WHEN OTHERS THEN
    -- Log error
    log_error('f_calculate_bonus', SQLERRM, DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
    RAISE;
END f_calculate_bonus;
```

### Control Structures

- Use consistent indentation within control structures
- Add END IF, END LOOP with a comment indicating the matching start
- Keep CASE statements formatted consistently
- Avoid deep nesting of control structures
- Use EXIT WHEN in loops instead of IF-EXIT combinations

```sql
-- Good practice for IF statements
IF l_salary > 10000 THEN
  l_tax_rate := 0.3;
ELSIF l_salary > 5000 THEN
  l_tax_rate := 0.2;
ELSE
  l_tax_rate := 0.1;
END IF; -- salary check

-- Good practice for LOOPS
FOR i IN 1..l_count LOOP
  IF employees_arr(i).salary > 5000 THEN
    process_high_earner(employees_arr(i));
  END IF;
END LOOP; -- employee processing

-- Good practice for CASE statements
CASE l_department_id
  WHEN 10 THEN
    l_location := 'New York';
  WHEN 20 THEN
    l_location := 'Dallas';
  WHEN 30 THEN
    l_location := 'Chicago';
  ELSE
    l_location := 'Unknown';
END CASE; -- department location
```

### Exception Handling

- Handle exceptions at the appropriate level
- Use named exceptions for clarity
- Include specific error handling for expected errors
- Add generic error handling for unexpected errors
- Avoid suppressing errors without proper logging
- Consider rethrowing exceptions after handling

```sql
PROCEDURE p_update_employee(
  p_employee_id IN NUMBER,
  p_salary      IN NUMBER
) IS
  e_salary_too_high EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_salary_too_high, -20010);
  
  l_old_salary NUMBER;
  l_dept_id    NUMBER;
BEGIN
  -- Validate inputs
  IF p_employee_id IS NULL OR p_salary IS NULL THEN
    RAISE_APPLICATION_ERROR(-20001, 'Employee ID and salary cannot be null');
  END IF;
  
  -- Get current values
  BEGIN
    SELECT salary, department_id
    INTO   l_old_salary, l_dept_id
    FROM   employees
    WHERE  employee_id = p_employee_id;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
      RAISE_APPLICATION_ERROR(-20002, 'Employee ID ' || p_employee_id || ' not found');
  END;
  
  -- Validate salary change
  IF p_salary > l_old_salary * 1.2 THEN
    RAISE e_salary_too_high;
  END IF;
  
  -- Update salary
  UPDATE employees
  SET    salary = p_salary,
         last_update_date = SYSDATE
  WHERE  employee_id = p_employee_id;
  
  -- Log the change
  INSERT INTO salary_changes
    (employee_id, old_salary, new_salary, change_date)
  VALUES
    (p_employee_id, l_old_salary, p_salary, SYSDATE);
  
EXCEPTION
  WHEN e_salary_too_high THEN
    log_error('p_update_employee', 'Salary increase exceeds 20% limit', 
              'Employee: ' || p_employee_id || 
              ', Old: ' || l_old_salary || 
              ', New: ' || p_salary);
    RAISE_APPLICATION_ERROR(-20010, 'Salary increase exceeds policy limits');
    
  WHEN OTHERS THEN
    log_error('p_update_employee', SQLERRM, DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
    ROLLBACK;
    RAISE;
END p_update_employee;
```

## Performance Considerations

### Query Optimization

- Use appropriate indexes for frequently queried columns
- Avoid functions on indexed columns in WHERE clauses
- Use hints sparingly and only when necessary
- Consider execution plans for complex queries
- Limit the use of DISTINCT, ORDER BY, and GROUP BY when possible
- Use EXISTS instead of IN for subqueries when checking existence

```sql
-- Good practice (index-friendly)
SELECT employee_id, last_name
FROM   employees
WHERE  department_id = 50
AND    hire_date BETWEEN TO_DATE('01-JAN-2020', 'DD-MON-YYYY')
                     AND TO_DATE('31-DEC-2020', 'DD-MON-YYYY');

-- Avoid (not index-friendly)
SELECT employee_id, last_name
FROM   employees
WHERE  TO_CHAR(hire_date, 'YYYY') = '2020'
AND    department_id = 50;
```

### Bulk Operations

- Use bulk collect and forall for processing multiple rows
- Set appropriate collection sizes to avoid memory issues
- Consider using LIMIT clause with bulk collect
- Handle exceptions properly in bulk operations

```sql
-- Good practice (bulk operations)
DECLARE
  TYPE t_emp_rec IS RECORD (
    employee_id employees.employee_id%TYPE,
    salary      employees.salary%TYPE
  );
  TYPE t_emp_tab IS TABLE OF t_emp_rec;
  
  l_employees t_emp_tab;
BEGIN
  -- Bulk collect with limit
  SELECT employee_id, salary
  BULK COLLECT INTO l_employees
  FROM   employees
  WHERE  department_id = 50
  LIMIT  100;
  
  -- Process in bulk
  FORALL i IN 1..l_employees.COUNT
    UPDATE employee_history
    SET    yearly_salary = l_employees(i).salary * 12
    WHERE  employee_id = l_employees(i).employee_id
    AND    history_year = EXTRACT(YEAR FROM SYSDATE);
    
EXCEPTION
  WHEN OTHERS THEN
    log_error('bulk_update_process', SQLERRM, DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
    RAISE;
END;
```

### Resource Management

- Close cursors explicitly when done
- Use DETERMINISTIC keyword for cacheable functions
- Use NOCOPY hint for large IN OUT parameters
- Implement connection pooling at the application level
- Consider result cache for frequently accessed, rarely changing data

```sql
-- Good practice (DETERMINISTIC function)
CREATE OR REPLACE FUNCTION f_get_tax_rate(
  p_income IN NUMBER
) RETURN NUMBER DETERMINISTIC IS
  l_tax_rate NUMBER;
BEGIN
  IF p_income <= 10000 THEN
    l_tax_rate := 0.1;
  ELSIF p_income <= 50000 THEN
    l_tax_rate := 0.2;
  ELSE
    l_tax_rate := 0.3;
  END IF;
  
  RETURN l_tax_rate;
END f_get_tax_rate;

-- Good practice (NOCOPY for large parameters)
PROCEDURE p_process_large_data(
  p_data IN OUT NOCOPY BLOB
) IS
BEGIN
  -- Process the BLOB data
  DBMS_LOB.TRIM(p_data, 1000);
  -- Other operations...
END p_process_large_data;
```

## Error Handling

### Best Practices

- Create custom exceptions for specific error conditions
- Use error codes consistently across applications
- Implement error logging for all exceptions
- Include context information in error messages
- Handle errors at the appropriate level

```sql
-- Custom exceptions
DECLARE
  e_invalid_status EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_invalid_status, -20100);
  
  e_process_failed EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_process_failed, -20101);
BEGIN
  -- Application logic...
  IF l_status NOT IN ('OPEN', 'CLOSED', 'PENDING') THEN
    RAISE e_invalid_status;
  END IF;
  
EXCEPTION
  WHEN e_invalid_status THEN
    log_error('process_order', 'Invalid status: ' || l_status, p_order_id);
    RAISE_APPLICATION_ERROR(-20100, 'Invalid order status: ' || l_status);
    
  WHEN OTHERS THEN
    log_error('process_order', SQLERRM, 'Order ID: ' || p_order_id || 
              ', ' || DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
    RAISE e_process_failed;
END;
```

### Error Logging

- Implement a centralized error logging mechanism
- Log error code, message, call stack, and context
- Include timestamp and session information
- Consider severity levels for errors
- Implement monitoring for critical errors

```sql
-- Error logging procedure
PROCEDURE log_error(
  p_source   IN VARCHAR2,
  p_message  IN VARCHAR2,
  p_context  IN VARCHAR2 DEFAULT NULL
) IS
  PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN
  INSERT INTO error_logs (
    log_id,
    log_date,
    username,
    source,
    error_message,
    context_info,
    call_stack
  ) VALUES (
    error_log_seq.NEXTVAL,
    SYSTIMESTAMP,
    SYS_CONTEXT('USERENV', 'SESSION_USER'),
    p_source,
    SUBSTR(p_message, 1, 4000),
    p_context,
    DBMS_UTILITY.FORMAT_ERROR_BACKTRACE
  );
  
  COMMIT;
EXCEPTION
  WHEN OTHERS THEN
    -- Last resort if logging fails
    NULL;
END log_error;
```

## Security Best Practices

### SQL Injection Prevention

- Use bind variables for all dynamic SQL
- Validate and sanitize all user inputs
- Avoid executing user-provided SQL strings
- Implement proper error handling to prevent information disclosure
- Use DBMS_ASSERT for dynamic SQL when necessary

```sql
-- Good practice (using bind variables)
PROCEDURE p_get_employee_by_dept(
  p_dept_id IN NUMBER
) IS
  l_cursor SYS_REFCURSOR;
BEGIN
  OPEN l_cursor FOR
    'SELECT employee_id, last_name, salary 
     FROM   employees 
     WHERE  department_id = :dept_id'
    USING p_dept_id;
  
  -- Process cursor...
  CLOSE l_cursor;
END p_get_employee_by_dept;

-- Avoid (vulnerable to SQL injection)
PROCEDURE p_get_employee_by_dept_unsafe(
  p_dept_id IN VARCHAR2
) IS
  l_cursor SYS_REFCURSOR;
  l_query  VARCHAR2(1000);
BEGIN
  l_query := 'SELECT employee_id, last_name, salary 
              FROM   employees 
              WHERE  department_id = ' || p_dept_id;
              
  OPEN l_cursor FOR l_query;
  
  -- Process cursor...
  CLOSE l_cursor;
END p_get_employee_by_dept_unsafe;
```

### Privilege Management

- Apply principle of least privilege
- Use roles for managing permissions
- Avoid using SYS or SYSTEM accounts for application operations
- Create specific application accounts with limited privileges
- Audit sensitive operations

```sql
-- Good practice (creating application-specific users and roles)
CREATE ROLE hr_clerk_role;

GRANT SELECT ON employees TO hr_clerk_role;
GRANT SELECT ON departments TO hr_clerk_role;
GRANT EXECUTE ON pkg_employee_query TO hr_clerk_role;

CREATE USER hr_clerk IDENTIFIED BY password;
GRANT hr_clerk_role TO hr_clerk;
GRANT CREATE SESSION TO hr_clerk;

-- Set up auditing for sensitive operations
AUDIT UPDATE ON employees BY ACCESS;
AUDIT EXECUTE ON pkg_salary_update BY ACCESS;
```

### Sensitive Data Handling

- Encrypt sensitive data at rest and in transit
- Use Oracle's built-in encryption features
- Mask or redact sensitive data in query results
- Implement data access controls
- Log access to sensitive data

```sql
-- Transparent Data Encryption (TDE)
ALTER TABLE employees MODIFY (
  salary ENCRYPT USING 'AES256'
);

-- Column-level encryption
CREATE OR REPLACE FUNCTION f_encrypt_ssn(
  p_ssn IN VARCHAR2
) RETURN RAW IS
BEGIN
  RETURN DBMS_CRYPTO.ENCRYPT(
    src => UTL_I18N.STRING_TO_RAW(p_ssn, 'AL32UTF8'),
    typ => DBMS_CRYPTO.ENCRYPT_AES256 + DBMS_CRYPTO.CHAIN_CBC + DBMS_CRYPTO.PAD_PKCS5,
    key => UTL_I18N.STRING_TO_RAW(g_encryption_key, 'AL32UTF8')
  );
END f_encrypt_ssn;

-- Data Redaction policy
BEGIN
  DBMS_REDACT.ADD_POLICY(
    object_schema       => 'HR',
    object_name         => 'EMPLOYEES',
    policy_name         => 'REDACT_SALARIES',
    column_name         => 'SALARY',
    function_type       => DBMS_REDACT.PARTIAL,
    function_parameters => '9,1,*',
    expression          => 'SYS_CONTEXT(''USERENV'',''CLIENT_IDENTIFIER'') != ''HR_ADMIN'''
  );
END;
```

## Code Documentation

### Header Documentation

Include a standard header for all code objects:

```sql
/*
* Object Name: pkg_employee_mgmt
* Object Type: Package
* Description: Employee management operations including hiring, transfers, and terminations
* 
* Author:      Jane Smith
* Created:     15-APR-2025
* Modified:    N/A
* Modified By: N/A
* Version:     1.0
*
* Dependencies:
*   - employees table
*   - departments table
*   - locations table
*   - pkg_audit_log package
*
* Notes:
*   - Implements business rules HR-101 to HR-114
*   - Raises application errors -20100 through -20120
*/
```

### Inline Documentation

- Document complex logic with inline comments
- Explain business rules and assumptions
- Use consistent comment formatting
- Document exceptions and error handling
- Explain non-obvious code

```sql
-- Calculate prorated bonus
-- Formula: (months worked / 12) * standard bonus * performance factor
l_prorated_bonus := (l_months_worked / 12) * l_standard_bonus * l_perf_factor;

-- Apply bonus cap based on salary grade
-- HR Policy HR-105: Bonus cannot exceed 30% of base salary
IF l_prorated_bonus > (l_base_salary * 0.3) THEN
  l_prorated_bonus := l_base_salary * 0.3;
END IF;
```

### Parameter Documentation

Document all parameters for procedures and functions:

```sql
/**
* Transfers an employee to a new department
*
* @param p_employee_id  Employee ID to transfer
* @param p_new_dept_id  Target department ID
* @param p_new_job_id   Optional new job ID (NULL means keep current job)
* @param p_new_salary   Optional new salary (NULL means keep current salary)
* @param p_effective_date Transfer effective date (defaults to current date)
*
* @raises e_invalid_employee  When employee ID is invalid (-20101)
* @raises e_invalid_dept      When department ID is invalid (-20102)
* @raises e_invalid_job       When job ID is invalid (-20103)
*/
PROCEDURE p_transfer_employee(
  p_employee_id    IN  NUMBER,
  p_new_dept_id    IN  NUMBER,
  p_new_job_id     IN  VARCHAR2 DEFAULT NULL,
  p_new_salary     IN  NUMBER   DEFAULT NULL,
  p_effective_date IN  DATE     DEFAULT SYSDATE
);
```

## Version Control Practices

### Script Format

- Include header with version information
- Add change log within scripts
- Use consistent file naming conventions
- Include rollback sections where appropriate

```sql
/*
* Script: create_employee_history_table.sql
* Version: 1.0
* Author: John Smith
* Date: 15-APR-2025
*
* Change Log:
* 1.0 (15-APR-2025, John Smith) - Initial version
*/

-- Create table
CREATE TABLE employee_history (
  history_id      NUMBER PRIMARY KEY,
  employee_id     NUMBER NOT NULL,
  effective_date  DATE NOT NULL,
  end_date        DATE,
  department_id   NUMBER,
  job_id          VARCHAR2(10),
  salary          NUMBER(8,2),
  manager_id      NUMBER,
  CONSTRAINT fk_emphistory_employee FOREIGN KEY (employee_id)
    REFERENCES employees (employee_id)
);

-- Create index
CREATE INDEX idx_emphistory_empid ON employee_history (employee_id);

-- Create sequence
CREATE SEQUENCE seq_emphistory_id START WITH 1 INCREMENT BY 1;

-- Rollback section (commented out)
/*
DROP SEQUENCE seq_emphistory_id;
DROP TABLE employee_history;
*/
```

### Change Management

- Include clear deployment instructions
- Add dependencies between scripts
- Document required privileges
- Include verification steps
- Provide rollback procedures

```sql
/*
* Deployment Instructions:
* 1. Execute as SCHEMA_OWNER user
* 2. Run prerequisite script: create_base_tables.sql
* 3. Verify successful creation of objects
* 4. Grant permissions: GRANT SELECT ON employee_history TO reporting_user;
*
* Verification:
* 1. SELECT COUNT(*) FROM employee_history; -- Should return 0
* 2. INSERT test record and verify sequence works
*
* Rollback:
* 1. Run rollback section at end of script if needed
*/
```

## Testing Standards

### Unit Testing

- Create test cases for all procedures and functions
- Test normal execution paths
- Test boundary conditions
- Test error handling
- Document test cases

```sql
-- Test case for calculate_bonus function
CREATE OR REPLACE PROCEDURE test_calculate_bonus IS
  l_result NUMBER;
  l_expected NUMBER;
BEGIN
  -- Test case 1: Standard employee
  l_expected := 500;
  l_result := f_calculate_bonus(101);
  
  IF l_result = l_expected THEN
    DBMS_OUTPUT.PUT_LINE('Test case 1: PASSED');
  ELSE
    DBMS_OUTPUT.PUT_LINE('Test case 1: FAILED. Expected ' || 
                         l_expected || ', got ' || l_result);
  END IF;
  
  -- Test case 2: Sales employee (higher bonus)
  l_expected := 1500;
  l_result := f_calculate_bonus(145);
  
  IF l_result = l_expected THEN
    DBMS_OUTPUT.PUT_LINE('Test case 2: PASSED');
  ELSE
    DBMS_OUTPUT.PUT_LINE('Test case 2: FAILED. Expected ' ||
