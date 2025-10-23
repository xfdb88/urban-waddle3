# GitHub Copilot Instructions

This document provides guidelines for GitHub Copilot when working on this repository.

## Project Overview

This is a simple web-based project (urban-waddle3). Please consider these guidelines when generating code suggestions.

## Code Style and Standards

### General Guidelines
- Write clean, readable, and maintainable code
- Follow consistent naming conventions throughout the project
- Use meaningful variable and function names
- Keep functions small and focused on a single responsibility
- Add comments only when necessary to explain complex logic

### Python Guidelines (if applicable)
- Follow PEP 8 style guidelines
- Use 4 spaces for indentation
- Maximum line length of 88 characters (Black formatter standard)
- Use type hints where appropriate
- Use descriptive docstrings for classes and functions

### JavaScript/TypeScript Guidelines (if applicable)
- Use ES6+ features and modern JavaScript syntax
- Prefer `const` and `let` over `var`
- Use arrow functions where appropriate
- Follow consistent semicolon usage
- Use single quotes for strings unless interpolation is needed

### HTML/CSS Guidelines (if applicable)
- Use semantic HTML5 elements
- Write accessible markup (proper ARIA labels, alt text, etc.)
- Keep CSS organized and use meaningful class names
- Prefer CSS Grid or Flexbox for layouts
- Use CSS variables for consistent theming

## Security Best Practices

- **Never** commit secrets, API keys, or credentials to the repository
- Always validate and sanitize user input
- Use parameterized queries to prevent SQL injection
- Set proper security headers for web applications
- Implement proper authentication and authorization
- For cookies: set `httpOnly`, `secure`, `sameSite: strict`, and appropriate `maxAge`
- Keep dependencies up to date and scan for vulnerabilities regularly

## Testing

- Write tests for new features and bug fixes
- Aim for meaningful test coverage of critical paths
- Use descriptive test names that explain what is being tested
- Follow the Arrange-Act-Assert pattern in tests
- Keep tests independent and isolated

## Documentation

- Update README.md when adding new features or changing setup instructions
- Document public APIs and interfaces
- Include examples in documentation where helpful
- Keep documentation synchronized with code changes

## Dependencies

- Minimize external dependencies where possible
- Only add well-maintained and trusted libraries
- Document why specific dependencies are needed
- Keep dependencies updated to latest stable versions

## Git Practices

- Write clear, concise commit messages
- Keep commits focused on a single change or feature
- Reference issue numbers in commit messages when applicable

## Performance

- Consider performance implications of code changes
- Optimize for readability first, then performance if needed
- Profile before optimizing to identify actual bottlenecks
- Minimize network requests and database queries

## Accessibility

- Ensure all interactive elements are keyboard accessible
- Provide appropriate ARIA labels and roles
- Maintain sufficient color contrast
- Test with screen readers when making UI changes

## Error Handling

- Handle errors gracefully with appropriate messages
- Log errors with sufficient context for debugging
- Don't expose sensitive information in error messages
- Provide helpful error messages to users
