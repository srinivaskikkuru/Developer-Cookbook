# Oracle APEX Development Standards Cookbook

![Oracle APEX Logo](https://oracle-apex.github.io/apex-frontend-boost/img/Oracle_APEX_Logo.png)

*A comprehensive guide for developers working with Oracle Application Express (APEX)*

Version 1.0.0 | April 2025

## Table of Contents

1. [Introduction](#introduction)
2. [General Development Guidelines](#general-development-guidelines)
3. [Naming Conventions](#naming-conventions)
4. [SQL and PL/SQL Standards](#sql-and-plsql-standards)
5. [Page and Region Design](#page-and-region-design)
6. [Interactive Reports and Grids](#interactive-reports-and-grids)
7. [JavaScript and jQuery Usage](#javascript-and-jquery-usage)
8. [CSS Styling](#css-styling)
9. [Security Best Practices](#security-best-practices)
10. [Performance Optimization](#performance-optimization)
11. [Version Control](#version-control)
12. [Documentation](#documentation)
13. [Testing](#testing)
14. [Deployment](#deployment)
15. [Maintenance](#maintenance)
16. [Appendix](#appendix)

## Introduction

This cookbook provides standardized guidelines for developing Oracle APEX applications according to Oracle's best practices. Following these standards will ensure consistency, maintainability, performance, and security across your APEX applications.

### Purpose

- Establish a uniform approach to APEX development
- Ensure application quality and maintainability
- Promote best practices in line with Oracle recommendations
- Serve as a reference for both new and experienced developers

### Scope

This document covers Oracle APEX versions 21.1 and above. Standards are based on Oracle's official documentation, industry best practices, and proven methodologies from experienced APEX developers.

## General Development Guidelines

### Application Structure

- **Workspace Organization**: 
  - Create separate workspaces for development, testing, and production environments
  - Maintain consistent application IDs across environments when possible

- **Application Separation**:
  - Use separate applications for distinct functional areas
  - Consider a modular design approach for complex systems

- **Development Methodology**:
  - Follow an iterative development approach
  - Establish clear milestones and delivery timelines

### Configuration Management

- **Application Definition**:
  - Define application properties consistently
  - Set appropriate application availability settings
  - Configure standardized error handling

- **Substitution Strings**:
  - Use application-level substitution strings for frequently used values
  - Follow the format `&STRING.` consistently

- **Build Options**:
  - Utilize build options for environment-specific features
  - Document all build options clearly

## Naming Conventions

Consistent naming conventions are essential for maintainable applications. Always prioritize clarity and descriptiveness over brevity.

### General Naming Rules

- Use clear, descriptive names that indicate purpose
- Maintain consistent capitalization (recommended: lowercase with underscores)
- Avoid abbreviations unless they are widely understood
- Prefix names according to object type
- Limit names to 30 characters when possible (for database compatibility)

### Specific Naming Conventions

| Object Type | Prefix | Example | Notes |
|-------------|--------|---------|-------|
| Page | `P` | `P10_EMPLOYEE_LIST` | Include functional description |
| Region | `R` | `R_EMPLOYEE_DETAILS` | Describe content, not location |
| Item | Context-specific | `P10_EMPLOYEE_ID` | Page number prefix |
| Button | `BTN` | `BTN_SAVE` | Action-oriented naming |
| Dynamic Action | `DA` | `DA_CONFIRM_DELETE` | Describe trigger and action |
| Process | `PROC` | `PROC_UPDATE_EMPLOYEE` | Indicate processing purpose |
| Application Item | `G` | `G_USER_ROLE` | For global items |
| List | `L` | `L_NAVIGATION_MENU` | Indicate list purpose |
| Template | - | `CORPORATE_HEADER` | Descriptive of style/purpose |
| Authorization Scheme | `AUTH` | `AUTH_ADMIN_ONLY` | Describe permissions granted |

### Database Objects

| Object Type | Prefix | Example | Notes |
|-------------|--------|---------|-------|
| Table | None | `EMPLOYEES` | Plural noun |
| View | `V_` | `V_EMPLOYEE_DETAILS` | Describe purpose |
| Procedure | `PROC_` | `PROC_UPDATE_EMPLOYEE` | Verb + noun |
| Function | `FUNC_` | `FUNC_GET_DEPARTMENT` | Verb + noun |
| Package | `PKG_` | `PKG_EMPLOYEE_API` | Group by functionality |
| Trigger | `TRG_` | `TRG_EMPLOYEES_BI` | Include table and timing |
| Sequence | `SEQ_` | `SEQ_EMPLOYEE_ID` | Associate with table |
| Index | `IDX_` | `IDX_EMPLOYEES_NAME` | Include indexed columns |
| Constraint | `PK_`, `FK_`, `UK_`, `CK_` | `PK_EMPLOYEES` | Indicate constraint type |

## SQL and PL/SQL Standards

### SQL Coding Standards

- **Formatting**:
  - Use consistent indentation (2 or 4 spaces)
  - Align keywords and clauses vertically
  - Place each major clause on a new line
  - Use uppercase for SQL keywords

```sql
SELECT e.employee_id,
       e.first_name,
       e.last_name,
       d.department_name
  FROM employees e
  JOIN departments d ON e.department_id = d.department_id
 WHERE e.hire_date > DATE '2020-01-01'
 ORDER BY e.last_name ASC,
          e.first_name ASC;
```

- **Optimization**:
  - Avoid `SELECT *` in production code
  - Use table aliases for multi-table queries
  - Include explicit joins instead of implicit joins
  - Use bind variables for dynamic values
  - Include appropriate indexing strategy

- **Readability**:
  - Add comments for complex queries
  - Use meaningful column aliases
  - Include appropriate line breaks

### PL/SQL Coding Standards

- **Package Structure**:
  - Group related functionality in packages
  - Create appropriate package specifications and bodies
  - Separate public and private methods clearly

- **Error Handling**:
  - Implement comprehensive exception handling
  - Log errors appropriately
  - Use named exceptions for clarity

- **Formatting**:
  - Use consistent indentation (2 or 4 spaces)
  - Include block labels where appropriate
  - Add comments for complex logic

```plsql
CREATE OR REPLACE PROCEDURE proc_update_employee (
  p_employee_id IN employees.employee_id%TYPE,
  p_first_name  IN employees.first_name%TYPE,
  p_last_name   IN employees.last_name%TYPE,
  p_email       IN employees.email%TYPE,
  p_success     OUT BOOLEAN
) AS
  v_count NUMBER;
BEGIN
  -- Check if employee exists
  SELECT COUNT(*)
    INTO v_count
    FROM employees
   WHERE employee_id = p_employee_id;
   
  IF v_count > 0 THEN
    -- Update existing employee
    UPDATE employees
       SET first_name = p_first_name,
           last_name = p_last_name,
           email = p_email,
           updated_date = SYSDATE
     WHERE employee_id = p_employee_id;
     
    p_success := TRUE;
  ELSE
    p_success := FALSE;
  END IF;
EXCEPTION
  WHEN OTHERS THEN
    p_success := FALSE;
    apex_error.add_error(
      p_message => 'Error updating employee: ' || SQLERRM,
      p_display_location => apex_error.c_inline_in_notification
    );
END proc_update_employee;
```

### APEX-Specific SQL/PL/SQL

- **Report Queries**:
  - Include pagination support (ROWNUM or ROW_NUMBER())
  - Add sorting capabilities
  - Implement appropriate security controls

- **Process Code**:
  - Validate inputs before processing
  - Use the APEX API for session state management
  - Implement appropriate error handling

- **Dynamic Actions**:
  - Use PL/SQL functions for complex validations
  - Return structured data (JSON) when working with JavaScript

## Page and Region Design

### Page Structure

- **Page Zero**:
  - Use for global elements (header, footer, navigation)
  - Avoid overloading with functionality

- **Page Hierarchy**:
  - Implement logical parent-child relationships
  - Use breadcrumbs to indicate hierarchy

- **Page Groups**:
  - Organize pages into functional groups
  - Use consistent naming for groups

### Region Organization

- **Layout**:
  - Use grid layouts for responsive design
  - Maintain consistent spacing and alignment
  - Group related regions logically

- **Region Types**:
  - Select appropriate region types for content
  - Use regions to organize page elements logically

- **Region Display Selector**:
  - Implement where multiple views of the same data are needed
  - Use consistent selector labels

### Best Practices

- Limit scrolling where possible
- Ensure consistent UI elements across pages
- Implement appropriate error and success messaging
- Use confirmation dialogs for destructive actions
- Maintain responsive design for mobile compatibility

## Interactive Reports and Grids

### Interactive Reports

- **Configuration**:
  - Define default report view with most relevant columns
  - Set appropriate pagination options (50-100 records per page)
  - Configure reasonable column widths

- **Column Settings**:
  - Use meaningful headings and tooltips
  - Configure appropriate formatting based on data type
  - Set correct sorting and filtering options

- **Saved Reports**:
  - Create standard default views as public saved reports
  - Use descriptive names for saved reports

### Interactive Grids

- **Edition Settings**:
  - Configure appropriate edition mode (row or cell)
  - Set validation rules for editable columns
  - Implement lost update detection

- **Performance**:
  - Limit the number of columns for better performance
  - Use appropriate lazy loading settings
  - Implement server-side validation for complex rules

- **Customization**:
  - Document any JavaScript customizations
  - Use declarative configuration before custom code

### Tabular Forms (Legacy)

- Consider migration to Interactive Grids for new development
- Apply similar standards as Interactive Grids where used

## JavaScript and jQuery Usage

### General Guidelines

- Include JavaScript only when necessary
- Use declarative APEX features before custom JavaScript
- Place code in external files for reusability
- Follow APEX JavaScript coding standards and patterns

### JavaScript Structure

- Namespace your JavaScript functions
- Use the APEX JavaScript API where available
- Encapsulate related functionality

```javascript
// Recommended structure
var MyApp = MyApp || {};

MyApp.employeeModule = {
    initPage: function() {
        // Initialize page-specific elements
        this.setupEventHandlers();
    },
    
    setupEventHandlers: function() {
        // Set up event handlers
        $("#my-button").on("click", this.handleButtonClick);
    },
    
    handleButtonClick: function(event) {
        // Handle button click
    }
};

// Document ready function
$(document).ready(function() {
    MyApp.employeeModule.initPage();
});
```

### Dynamic Actions vs. JavaScript

- Prefer Dynamic Actions for simple interactions
- Use JavaScript for complex, reusable functionality
- Document when custom JavaScript is required

### jQuery Usage

- Use APEX's included jQuery version
- Avoid conflicting with APEX's jQuery by using `apex.jQuery`
- Follow jQuery best practices for performance

## CSS Styling

### Theme Utilization

- Use the Universal Theme when possible
- Create custom themes based on Universal Theme
- Utilize template options before custom CSS

### Custom CSS Guidelines

- Place custom CSS in a dedicated file
- Use the CSS Cascade appropriately
- Follow BEM (Block, Element, Modifier) naming convention
- Implement responsive design principles

```css
/* Example of BEM naming convention */
.employee-card {
    /* Block: The main component */
    margin: 10px;
    padding: 15px;
}

.employee-card__header {
    /* Element: A part of the block */
    font-weight: bold;
}

.employee-card--featured {
    /* Modifier: Different state of the block */
    border: 2px solid #c74634;
}
```

### CSS Organization

- Group related styles together
- Include comments for complex selectors
- Minimize specificity to avoid conflicts

## Security Best Practices

### Authentication

- Use appropriate authentication schemes
- Implement multi-factor authentication where needed
- Configure proper session management

### Authorization

- Create granular authorization schemes
- Apply authorization at the appropriate level (application, page, component)
- Document authorization schemes thoroughly

### Data Protection

- Validate all inputs
- Escape outputs to prevent XSS
- Use bind variables to prevent SQL injection
- Implement appropriate data masking

### Session Management

- Configure appropriate session timeout
- Use secure session state protection
- Implement proper page access protection

## Performance Optimization

### Database Optimization

- Create appropriate indexes
- Optimize SQL queries
- Use efficient PL/SQL practices

### APEX Optimization

- Minimize page weight
- Use appropriate caching strategies
- Implement lazy loading where beneficial

### Page Loading Performance

- Reduce the number of page components
- Optimize JavaScript and CSS loading
- Use asynchronous processes for long-running operations

### Monitoring

- Implement appropriate logging
- Review APEX debug logs regularly
- Monitor database performance

## Version Control

### Export Strategy

- Export applications as separate files
- Include supporting objects
- Document export format and method

### Git Integration

- Structure repository logically
- Include meaningful commit messages
- Use branches for feature development

```
/project-root
  /app
    /f100  # Application 100
      application/
      pages/
      shared_components/
    /f200  # Application 200
  /supporting-objects
    /data
    /scripts
  /documentation
  README.md
  CHANGELOG.md
```

### Release Management

- Tag releases with version numbers
- Maintain a changelog
- Document deployment procedures

## Documentation

### Inline Documentation

- Include comments for complex code
- Document assumptions and constraints
- Add page and region headers with purpose descriptions

### External Documentation

- Maintain user guides
- Create technical documentation
- Document application architecture

### Standards Documentation

- Keep this cookbook updated
- Document exceptions to standards
- Include examples of best practices

## Testing

### Manual Testing

- Create test scripts for key functionality
- Include regression testing procedures
- Document test results

### Automated Testing

- Implement unit tests for PL/SQL
- Use UI testing tools where appropriate
- Automate regression testing

### User Acceptance Testing

- Define UAT procedures
- Document feedback collection methods
- Implement issue tracking

## Deployment

### Deployment Strategy

- Document deployment steps
- Use scripted deployments when possible
- Implement version checking

### Environment Management

- Maintain consistent environments
- Document environment differences
- Use build options for environment-specific features

### Post-Deployment Verification

- Create deployment checklists
- Implement smoke tests
- Monitor initial usage

## Maintenance

### Application Monitoring

- Review error logs regularly
- Monitor performance metrics
- Implement usage analytics

### Regular Updates

- Apply APEX patches
- Update dependent libraries
- Refresh documentation

### Refactoring

- Identify technical debt
- Schedule regular refactoring
- Maintain backwards compatibility

## Appendix

### Reference Materials

- [Oracle APEX Documentation](https://apex.oracle.com/en/learn/documentation/)
- [Oracle SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/index.html)
- [Oracle PL/SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/index.html)

### Checklists

#### Pre-Release Checklist

- [ ] All pages follow naming conventions
- [ ] SQL/PL/SQL code follows standards
- [ ] Security authorization schemes implemented
- [ ] Form validations complete
- [ ] Error handling implemented
- [ ] Documentation updated
- [ ] Performance tested
- [ ] Cross-browser compatibility verified
- [ ] Accessibility requirements met
- [ ] Unit tests pass

#### Code Review Checklist

- [ ] Naming conventions followed
- [ ] Code properly formatted
- [ ] Error handling implemented
- [ ] Security controls in place
- [ ] Performance considerations addressed
- [ ] Documentation complete
- [ ] No hardcoded values
- [ ] Appropriate use of APEX features

### Templates

#### Package Template

```plsql
CREATE OR REPLACE PACKAGE pkg_name AS
  /*
  * Package: PKG_NAME
  * Purpose: Brief description
  * Created: YYYY-MM-DD by [Author]
  */
  
  -- Public type definitions
  
  -- Public constant declarations
  
  -- Public variable declarations
  
  -- Public function and procedure declarations
  PROCEDURE procedure_name (
    p_param1 IN data_type,
    p_param2 OUT data_type
  );
  
  FUNCTION function_name (
    p_param IN data_type
  ) RETURN return_data_type;
  
END pkg_name;
/

CREATE OR REPLACE PACKAGE BODY pkg_name AS
  -- Private type definitions
  
  -- Private constant declarations
  
  -- Private variable declarations
  
  -- Private function and procedure implementations
  
  -- Public function and procedure implementations
  PROCEDURE procedure_name (
    p_param1 IN data_type,
    p_param2 OUT data_type
  ) AS
  BEGIN
    -- Implementation
  EXCEPTION
    WHEN OTHERS THEN
      -- Error handling
  END procedure_name;
  
  FUNCTION function_name (
    p_param IN data_type
  ) RETURN return_data_type AS
    v_result return_data_type;
  BEGIN
    -- Implementation
    RETURN v_result;
  EXCEPTION
    WHEN OTHERS THEN
      -- Error handling
      RETURN NULL;
  END function_name;
  
END pkg_name;
/
```

---

*This cookbook is maintained by [Your Team/Organization]. Last updated: April 2025.*
