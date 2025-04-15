# Oracle APEX HTML & CSS Coding Standards Cookbook

## Table of Contents
- [Introduction](#introduction)
- [General Principles](#general-principles)
- [HTML Standards](#html-standards)
  - [Document Structure](#document-structure)
  - [Semantics](#semantics)
  - [Accessibility](#accessibility)
  - [Forms](#forms)
  - [Template Directives](#template-directives)
- [CSS Standards](#css-standards)
  - [Naming Conventions](#naming-conventions)
  - [Organization](#organization)
  - [Responsive Design](#responsive-design)
  - [Universal Theme Guidelines](#universal-theme-guidelines)
- [Performance Optimization](#performance-optimization)
- [Common Patterns and Components](#common-patterns-and-components)
- [Code Examples](#code-examples)
- [Testing and Validation](#testing-and-validation)
- [Additional Resources](#additional-resources)

## Introduction

This document outlines the coding standards and best practices for HTML and CSS development within Oracle APEX applications. Following these guidelines ensures consistency, maintainability, and optimal performance across all APEX applications.

Oracle APEX is a low-code development platform that allows developers to build scalable, secure enterprise apps with world-class features. While APEX abstracts many frontend concerns, customizing the HTML and CSS is often necessary for advanced UIs. This cookbook provides standards to ensure these customizations align with Oracle best practices.

## General Principles

### Code Readability and Maintainability

- Write code that is easy to read, understand, and maintain
- Use consistent indentation (2 spaces recommended)
- Keep lines under 100 characters when possible
- Use meaningful names for classes, IDs, and variables
- Include comments for complex sections or non-obvious code
- Organize files logically and consistently

### Oracle APEX Compatibility

- Always consider how custom code interacts with APEX's framework
- Avoid modifying core APEX functionality directly
- Test custom code against multiple APEX versions when possible
- Follow Oracle's Universal Theme design principles

### Version Control Practices

- Document all HTML/CSS changes within version control commits
- Include ticket/issue references in commit messages
- Create separate branches for major UI changes
- Regularly merge from main branch to prevent style conflicts

## HTML Standards

### Document Structure

#### Template Structure

```html
<!-- Recommended APEX page template structure -->
<header class="t-Header" id="apex-page-header">
  <!-- Header content -->
</header>

<div class="t-Body">
  <div class="t-Body-main">
    <div class="t-Body-title" id="apex-page-title">
      <!-- Page title -->
    </div>
    <div class="t-Body-content" id="apex-page-content">
      <!-- Main content -->
    </div>
  </div>
  <div class="t-Body-side" id="apex-page-sidebar">
    <!-- Sidebar content -->
  </div>
</div>

<footer class="t-Footer" id="apex-page-footer">
  <!-- Footer content -->
</footer>
```

#### Indentation and Formatting

- Use 2-space indentation for nested elements
- Place each block-level element on a new line
- Keep inline elements together when they form a logical unit
- Use lowercase for element names, attributes, and values

```html
<!-- Good -->
<div class="t-Region">
  <div class="t-Region-header">
    <h2 class="t-Region-title">Region Title</h2>
  </div>
  <div class="t-Region-body">
    <p>Region content goes here. <a href="#link">This is an inline link</a> that stays with its text.</p>
  </div>
</div>

<!-- Bad -->
<DIV class="t-Region">
<div class="t-Region-header"><h2 class="t-Region-title">Region Title</h2></div>
<div class="t-Region-body">
<p>Region content goes here.
<a href="#link">This is an inline link</a> that stays with its text.</p></div></DIV>
```

### Semantics

#### Use Semantic HTML5 Elements

- Prefer semantic elements over generic divs and spans when appropriate
- Use the following elements for their intended purpose:
  - `<header>`, `<footer>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`
  - `<h1>` through `<h6>` for heading hierarchy
  - `<p>` for paragraphs
  - `<ul>`, `<ol>`, `<li>` for lists
  - `<figure>` and `<figcaption>` for images with captions
  - `<table>` for tabular data (not for layout)

```html
<!-- Good -->
<section class="t-Region">
  <header class="t-Region-header">
    <h2 class="t-Region-title">Customer Details</h2>
  </header>
  <div class="t-Region-body">
    <article class="customer-info">
      <h3>Contact Information</h3>
      <p>Primary contact details for the customer.</p>
    </article>
  </div>
</section>

<!-- Bad -->
<div class="t-Region">
  <div class="t-Region-header">
    <div class="t-Region-title">Customer Details</div>
  </div>
  <div class="t-Region-body">
    <div class="customer-info">
      <div>Contact Information</div>
      <div>Primary contact details for the customer.</div>
    </div>
  </div>
</div>
```

#### ID and Class Naming

- Use IDs sparingly and only for unique elements
- Prefix custom IDs with application or component namespace to prevent conflicts
- For classes, follow Oracle's naming conventions:
  - APEX Universal Theme uses `t-[Component]` for its components
  - Use `a-[Component]` for application-specific components
  - Use `u-[Utility]` for utility classes

```html
<!-- Good -->
<div id="app123-customer-details" class="a-CustomerCard t-Region">
  <div class="t-Region-header">
    <h2 class="t-Region-title">Customer Details</h2>
  </div>
  <div class="t-Region-body u-alignCenter">
    <!-- Content -->
  </div>
</div>

<!-- Bad -->
<div id="customer-details" class="card region">
  <div class="header">
    <h2 class="title">Customer Details</h2>
  </div>
  <div class="body center">
    <!-- Content -->
  </div>
</div>
```

### Accessibility

#### Basic Requirements

- Use proper heading structure (h1-h6) in sequential order
- Include descriptive alt text for all images
- Ensure sufficient color contrast (WCAG AA compliance minimum)
- Support keyboard navigation for all interactive elements
- Add ARIA roles and attributes where necessary

```html
<!-- Good -->
<img src="logo.png" alt="Oracle Corporation Logo">
<button aria-label="Close dialog" class="t-Button t-Button--noLabel t-Button--icon">
  <span class="t-Icon fa fa-times" aria-hidden="true"></span>
</button>

<!-- Bad -->
<img src="logo.png">
<button class="t-Button t-Button--noLabel t-Button--icon">
  <span class="t-Icon fa fa-times"></span>
</button>
```

#### APEX-Specific Accessibility

- Use APEX's built-in accessibility features:
  - Page and region landmarks
  - Skip navigation links
  - Accessible form validation

```html
<!-- Add landmarks to custom regions -->
<div class="t-Region" role="region" aria-label="Customer Information">
  <!-- Region content -->
</div>

<!-- Ensure form elements have proper labels -->
<div class="t-Form-fieldContainer">
  <label for="P1_CUSTOMER_NAME" class="t-Form-label">Customer Name</label>
  <div class="t-Form-inputContainer">
    <input type="text" id="P1_CUSTOMER_NAME" name="P1_CUSTOMER_NAME" required aria-required="true">
    <span class="t-Form-help">Enter the full customer name</span>
  </div>
</div>
```

### Forms

#### Structure and Layout

- Follow the Oracle Universal Theme form conventions
- Use appropriate input types for data validation
- Include proper labels for all form elements
- Group related form elements with fieldset and legend
- Provide clear validation messages

```html
<div class="t-Form-fieldContainer">
  <div class="t-Form-labelContainer">
    <label for="P1_EMAIL" class="t-Form-label">Email Address</label>
  </div>
  <div class="t-Form-inputContainer">
    <div class="t-Form-itemWrapper">
      <input type="email" id="P1_EMAIL" name="P1_EMAIL" class="text_field apex-item-text" required>
    </div>
    <span class="t-Form-help">Enter a valid email address</span>
  </div>
</div>
```

#### Validation and Error Handling

- Use HTML5 validation attributes (required, pattern, min, max, etc.)
- Add ARIA attributes for custom validation
- Use Oracle's standard error display patterns

```html
<div class="t-Form-fieldContainer t-Form-fieldContainer--floatingLabel is-required is-error">
  <div class="t-Form-labelContainer">
    <label for="P1_USERNAME" class="t-Form-label">Username</label>
  </div>
  <div class="t-Form-inputContainer">
    <div class="t-Form-itemWrapper">
      <input type="text" id="P1_USERNAME" name="P1_USERNAME" 
             class="text_field apex-item-text" 
             required 
             pattern="[A-Za-z0-9_]{3,16}" 
             aria-invalid="true">
    </div>
    <div class="t-Form-error">
      Username must be 3-16 characters and contain only letters, numbers, and underscores.
    </div>
  </div>
</div>
```

### Template Directives

- Use APEX substitution strings consistently
- Follow Oracle's conventions for template directives
- Document custom substitution strings

```html
<!-- APEX substitution strings -->
<div class="t-Region">
  <div class="t-Region-header">
    <h2 class="t-Region-title">#TITLE#</h2>
  </div>
  <div class="t-Region-body">
    #BODY#
  </div>
  <div class="t-Region-buttons t-Region-buttons--bottom">
    #PREVIOUS##NEXT#
  </div>
</div>
```

## CSS Standards

### Naming Conventions

#### BEM-Inspired Approach for Custom Components

Oracle's approach is inspired by BEM (Block, Element, Modifier) methodology:

- Blocks: Main components (e.g., `t-Button`)
- Elements: Parts of blocks (e.g., `t-Button-label`)
- Modifiers: Variations (e.g., `t-Button--hot`)

For custom components:

```css
/* Block */
.a-Dashboard {}

/* Element */
.a-Dashboard-panel {}

/* Modifier */
.a-Dashboard--compact {}
.a-Dashboard-panel--highlighted {}
```

#### Universal Theme Class Prefixes

- `t-`: Theme components (Oracle-provided)
- `a-`: Application-specific components
- `u-`: Utility classes
- `js-`: JavaScript hooks (no styling)

```css
/* Oracle Theme component */
.t-Button {}

/* Application component */
.a-ReportCard {}

/* Utility class */
.u-textCenter {}

/* JavaScript hook (no styling) */
.js-showHideRegion {}
```

### Organization

#### File Structure

- Organize CSS into logical files:
  - `base.css`: Reset and base styles
  - `layout.css`: Grid and structural elements
  - `components.css`: Reusable UI components
  - `utilities.css`: Helper classes
  - `theme-overrides.css`: Customizations to Universal Theme

#### Within File Organization

- Group related styles together
- Order properties consistently (recommended: alphabetical)
- Add comments to delineate sections

```css
/* ----- CUSTOMER CARD COMPONENT ----- */

/* Card container */
.a-CustomerCard {
  background-color: #ffffff;
  border: 1px solid #e6e6e6;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  margin-bottom: 1rem;
  padding: 1.25rem;
}

/* Card header */
.a-CustomerCard-header {
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 1rem;
  padding-bottom: 0.75rem;
}

/* Card title */
.a-CustomerCard-title {
  color: #333333;
  font-size: 1.25rem;
  font-weight: 500;
  margin: 0;
}

/* Status modifier */
.a-CustomerCard--active {
  border-left: 3px solid #4cd964;
}

.a-CustomerCard--inactive {
  border-left: 3px solid #ff3b30;
}
```

### Responsive Design

#### Mobile-First Approach

- Start with styles for mobile devices
- Add complexity for larger screens with media queries
- Use Oracle's breakpoint variables

```css
/* Base styles (mobile first) */
.a-Dashboard-panel {
  width: 100%;
  margin-bottom: 1rem;
}

/* Tablet and above */
@media (min-width: 768px) {
  .a-Dashboard-panel {
    width: 48%;
    margin-right: 2%;
    float: left;
  }
  
  .a-Dashboard-panel:nth-child(2n) {
    margin-right: 0;
  }
}

/* Desktop and above */
@media (min-width: 1024px) {
  .a-Dashboard-panel {
    width: 31.333%;
    margin-right: 3%;
  }
  
  .a-Dashboard-panel:nth-child(2n) {
    margin-right: 3%;
  }
  
  .a-Dashboard-panel:nth-child(3n) {
    margin-right: 0;
  }
}
```

#### Breakpoints

Use Oracle's standard breakpoints:

```css
/* Extra small devices (phones) */
/* Base styles - no media query needed */

/* Small devices (tablets) */
@media (min-width: 576px) { /* ... */ }

/* Medium devices (desktops) */
@media (min-width: 768px) { /* ... */ }

/* Large devices (large desktops) */
@media (min-width: 992px) { /* ... */ }

/* Extra large devices */
@media (min-width: 1200px) { /* ... */ }
```

### Universal Theme Guidelines

#### Theme Customization

- Use Theme Roller for global theme adjustments when possible
- Override styles in a structured manner
- Target Universal Theme classes carefully

```css
/* Properly override a Universal Theme component */
.t-Button.t-Button--hot {
  background-color: #e85d88; /* Custom hot button color */
}

/* Add custom styles to existing components */
.t-Region.a-Dashboard-region {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s ease;
}

.t-Region.a-Dashboard-region:hover {
  box-shadow: 0 6px 10px rgba(0, 0, 0, 0.15);
}
```

#### Theme Integration

- Extend rather than override the Universal Theme
- Use Oracle's CSS custom properties (variables) where available
- Follow Oracle's component patterns for new elements

```css
/* Using Universal Theme CSS variables */
.a-CustomAlert {
  background-color: var(--ut-component-background-color);
  border: 1px solid var(--ut-component-border-color);
  border-radius: var(--ut-component-border-radius);
  color: var(--ut-component-text-color);
  padding: var(--ut-component-padding-y) var(--ut-component-padding-x);
}

.a-CustomAlert--danger {
  background-color: var(--ut-palette-danger);
  color: var(--ut-palette-danger-contrast);
}
```

## Performance Optimization

### CSS Optimization

- Minimize CSS specificity issues
- Avoid deeply nested selectors (3 levels max)
- Use classes over IDs for styling
- Avoid universal selectors (`*`) in complex selectors
- Limit use of expensive properties (box-shadow, text-shadow, gradients, etc.)

```css
/* Good - Flat hierarchy */
.t-Region .a-Dashboard-metric {
  color: #333;
}

/* Bad - Deeply nested and inefficient */
.t-Body .t-Region .t-Region-body .a-Dashboard .a-Dashboard-panel .a-Dashboard-metric {
  color: #333;
}
```

### Loading Strategies

- Load critical CSS inline for faster rendering
- Defer non-critical CSS
- Follow Oracle's recommendations for file organization

```html
<!-- Critical CSS inline in head -->
<style>
  /* Only essential styles needed for initial render */
  .t-PageBody {
    background-color: #f5f5f5;
    font-family: 'Oracle Sans', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  }
  
  .t-Header {
    background-color: #1f4e79;
    color: white;
  }
</style>

<!-- Non-critical CSS loaded normally -->
<link rel="stylesheet" href="#APP_IMAGES#css/app.min.css">
```

## Common Patterns and Components

### Standard Component Extensions

Document how to extend common Oracle APEX components while maintaining compatibility:

#### Cards

```html
<div class="t-Card a-ProjectCard">
  <div class="t-Card-wrap">
    <div class="t-Card-icon"><span class="fa fa-project-diagram"></span></div>
    <div class="t-Card-titleWrap">
      <h3 class="t-Card-title">Project Name</h3>
    </div>
    <div class="t-Card-body">
      <div class="a-ProjectCard-metrics">
        <span class="a-ProjectCard-metric">
          <span class="a-ProjectCard-metricValue">42</span>
          <span class="a-ProjectCard-metricLabel">Tasks</span>
        </span>
        <span class="a-ProjectCard-metric">
          <span class="a-ProjectCard-metricValue">78%</span>
          <span class="a-ProjectCard-metricLabel">Complete</span>
        </span>
      </div>
    </div>
    <div class="t-Card-actionWrap">
      <a href="#" class="t-Button t-Button--small">View Details</a>
    </div>
  </div>
</div>
```

#### Custom Region Styling

```css
/* Extending a standard region */
.t-Region.a-MetricsRegion {
  background-color: #f9f9f9;
}

.a-MetricsRegion .t-Region-header {
  border-bottom-color: #e0e0e0;
}

.a-MetricsRegion .t-Region-title {
  font-size: 1.2rem;
  text-transform: uppercase;
  letter-spacing: 1px;
}
```

## Code Examples

### Basic Page Template Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>#TITLE#</title>
  #APEX_CSS#
  #THEME_CSS#
  #TEMPLATE_CSS#
  #THEME_STYLE_CSS#
  #APPLICATION_CSS#
  #PAGE_CSS#
  #HEAD#
  #APEX_JAVASCRIPT#
  #THEME_JAVASCRIPT#
  #TEMPLATE_JAVASCRIPT#
  #APPLICATION_JAVASCRIPT#
  #PAGE_JAVASCRIPT#
</head>
<body class="t-PageBody t-PageBody--hideLeft t-PageBody--hideActions no-anim #PAGE_CSS_CLASSES#" #TEXT_DIRECTION# #ONLOAD#>
  <div class="t-Body">
    <header class="t-Header" id="t_Header">
      <div class="t-Header-branding">
        <div class="t-Header-controls">
          <button class="t-Button t-Button--icon t-Button--header t-Button--headerTree" aria-label="#EXPAND_COLLAPSE_NAV_LABEL#" title="#EXPAND_COLLAPSE_NAV_LABEL#" id="t_Button_navControl" type="button"><span class="t-Icon fa fa-bars" aria-hidden="true"></span></button>
        </div>
        <div class="t-Header-logo">
          <a href="#HOME_LINK#" class="t-Header-logo-link">#LOGO#</a>
        </div>
        <div class="t-Header-navBar">#NAVIGATION_BAR#</div>
      </div>
      <div class="t-Header-nav">#TOP_GLOBAL_NAVIGATION_LIST##REGION_POSITION_06#</div>
    </header>
    <div class="t-Body-main">
      <div class="t-Body-title" id="t_Body_title">#REGION_POSITION_01#</div>
      <div class="t-Body-content" id="t_Body_content">
        <main id="main" class="t-Body-mainContent">
          #SUCCESS_MESSAGE##NOTIFICATION_MESSAGE##GLOBAL_NOTIFICATION#
          <div class="t-Body-fullContent">#REGION_POSITION_08#</div>
          <div class="t-Body-contentInner">#BODY#</div>
        </main>
        <footer class="t-Footer" id="t_Footer">
          <div class="t-Footer-body">
            <div class="t-Footer-content">#REGION_POSITION_05#</div>
            <div class="t-Footer-apex">
              <div class="t-Footer-version">#APP_VERSION#</div>
              <div class="t-Footer-customize">#CUSTOMIZE#</div>
              <div class="t-Footer-srMode">#SCREEN_READER_TOGGLE#</div>
            </div>
          </div>
          <div class="t-Footer-top">
            <a href="#top" class="t-Footer-topButton" id="t_Footer_topButton"><span class="a-Icon icon-up-chevron"></span></a>
          </div>
        </footer>
      </div>
    </div>
  </div>
  #FORM_CLOSE#
  #DEVELOPER_TOOLBAR#
  #APEX_JAVASCRIPT#
  #GENERATED_CSS#
  #THEME_JAVASCRIPT#
  #APPLICATION_JAVASCRIPT#
  #PAGE_JAVASCRIPT#
  #GENERATED_JAVASCRIPT#
</body>
</html>
```

### Custom Component Example

```html
<!-- Custom metric card component -->
<div class="a-MetricCard">
  <div class="a-MetricCard-icon">
    <span class="fa fa-users"></span>
  </div>
  <div class="a-MetricCard-content">
    <span class="a-MetricCard-value">1,254</span>
    <span class="a-MetricCard-label">Total Users</span>
  </div>
  <div class="a-MetricCard-change a-MetricCard-change--positive">
    <span class="a-MetricCard-changeIcon fa fa-arrow-up"></span>
    <span class="a-MetricCard-changeValue">12.3%</span>
  </div>
</div>
```

```css
/* Styling for the custom component */
.a-MetricCard {
  background-color: #ffffff;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  margin-bottom: 1rem;
  overflow: hidden;
  padding: 1.5rem;
  position: relative;
}

.a-MetricCard-icon {
  align-items: center;
  background-color: rgba(0, 0, 0, 0.05);
  border-radius: 50%;
  display: flex;
  font-size: 1.5rem;
  height: 3rem;
  justify-content: center;
  margin-right: 1rem;
  width: 3rem;
}

.a-MetricCard-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.a-MetricCard-value {
  font-size: 1.75rem;
  font-weight: 600;
  line-height: 1;
  margin-bottom: 0.25rem;
}

.a-MetricCard-label {
  color: #777777;
  font-size: 0.875rem;
}

.a-MetricCard-change {
  align-items: center;
  display: flex;
  position: absolute;
  right: 1.5rem;
  top: 1.5rem;
}

.a-MetricCard-change--positive {
  color: #4cd964;
}

.a-MetricCard-change--negative {
  color: #ff3b30;
}

.a-MetricCard-changeIcon {
  font-size: 0.75rem;
  margin-right: 0.25rem;
}

.a-MetricCard-changeValue {
  font-size: 0.875rem;
  font-weight: 500;
}
```

## Testing and Validation

### Code Quality Tools

- Use CSS linters to maintain code quality
- Validate HTML against W3C standards
- Setup continuous integration for automated testing

### Browser Testing

- Test across standard browsers:
  - Chrome (latest 2 versions)
  - Firefox (latest 2 versions)
  - Safari (latest 2 versions)
  - Edge (latest version)
- Test responsive layouts at standard breakpoints
- Validate for accessibility using tools like Axe or WAVE

### Performance Testing

- Run Lighthouse audits for performance metrics
- Measure and optimize Critical Rendering Path
- Test load times on various network conditions

## Additional Resources

### Oracle Documentation

- [Oracle APEX Universal Theme Documentation](https://apex.oracle.com/pls/apex/apex_pm/r/ut/getting-started)
- [Oracle JavaScript Extension Toolkit (JET)](https://www.oracle.com/webfolder/technetwork/jet/index.html)

### Community Resources

- [Oracle APEX Community](https://apex.oracle.com/en/community/)
- [Oracle Technology Network for APEX](https://community.oracle.com/tech/developers/categories/application_express)

### Accessibility Guidelines

- [Web Content Accessibility Guidelines (WCAG) 2.1](https://www.w3.org/TR/WCAG21/)
- [Oracle's Accessibility Guidelines](https://www.oracle.com/corporate/accessibility/)

---

## Contribution Guidelines

Contributions to this coding standards document are welcome. Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature-name`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin feature/your-feature-name`)
5. Create a new Pull Request

## License

This coding standards document is licensed under [Your License Choice]. See the LICENSE file for details.

---

*Last updated: April 16, 2025*
