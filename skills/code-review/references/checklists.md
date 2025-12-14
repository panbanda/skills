# Code Review Checklists

Detailed checklists by category. Use during Step 2 of the review process.

## Security (OWASP-Based)

### Input Validation
- [ ] All user inputs validated and sanitized
- [ ] Input length limits enforced
- [ ] Character whitelisting where appropriate
- [ ] File upload validation (type, size, content)

### Injection Prevention
- [ ] SQL: Parameterized queries only, no string concatenation
- [ ] Command: No shell execution with user input, or properly escaped
- [ ] XSS: Output encoding for HTML/JS/CSS contexts
- [ ] LDAP/XML/Path: Proper escaping or parameterization

### Authentication & Authorization
- [ ] Authentication required for sensitive operations
- [ ] Authorization checked at every access point
- [ ] Session tokens properly generated and validated
- [ ] Password handling uses secure hashing (bcrypt, argon2)

### Secrets & Sensitive Data
- [ ] No hardcoded credentials, API keys, or secrets
- [ ] Secrets loaded from environment or secret manager
- [ ] Sensitive data not logged or exposed in errors
- [ ] PII handled according to data protection requirements

### Access Control
- [ ] Principle of least privilege followed
- [ ] Direct object references validated against user permissions
- [ ] Admin functions properly protected
- [ ] Rate limiting on sensitive endpoints

## Performance

### Algorithmic Complexity
- [ ] No O(n^2) or worse in hot paths
- [ ] Appropriate data structures for access patterns
- [ ] Pagination for large result sets

### Database
- [ ] No N+1 query patterns
- [ ] Queries use appropriate indexes
- [ ] Large queries limited or paginated
- [ ] Transactions scoped minimally

### Memory & Resources
- [ ] No memory leaks (closures, event listeners, caches)
- [ ] Large allocations avoided in loops
- [ ] Streams used for large data processing
- [ ] Resources properly closed/disposed

### Caching
- [ ] Appropriate caching for expensive operations
- [ ] Cache invalidation strategy defined
- [ ] Cache keys properly scoped

### Concurrency
- [ ] Race conditions addressed
- [ ] Deadlock potential considered
- [ ] Thread-safe data structures where needed
- [ ] Async operations properly awaited

## Quality

### Testing
- [ ] Unit tests for new/changed logic
- [ ] Edge cases covered
- [ ] Error paths tested
- [ ] Integration tests for API changes
- [ ] Test assertions meaningful (not just "no exception")

### Error Handling
- [ ] Errors caught at appropriate level
- [ ] Error messages helpful but not leaking internals
- [ ] Fallback behavior defined for failures
- [ ] Cleanup happens in error paths

### Logging & Observability
- [ ] Appropriate log levels used
- [ ] Structured logging where applicable
- [ ] Request IDs/correlation IDs propagated
- [ ] Metrics for key operations

### Edge Cases
- [ ] Null/undefined handled
- [ ] Empty collections handled
- [ ] Boundary conditions tested
- [ ] Concurrent access considered

## Complexity

### Metrics Thresholds
- [ ] Cyclomatic complexity <15 per function
- [ ] Cognitive complexity <15 per function
- [ ] Function length <50 lines (ideally <30)
- [ ] Nesting depth <4 levels
- [ ] File length <500 lines

### Simplification Opportunities
- [ ] Early returns to reduce nesting
- [ ] Guard clauses for preconditions
- [ ] Complex conditionals extracted to named functions
- [ ] State machines for complex state logic

## Maintainability

### Naming
- [ ] Names describe purpose, not implementation
- [ ] Consistent naming conventions
- [ ] No abbreviations except well-known ones
- [ ] Boolean names indicate true condition

### Single Responsibility
- [ ] Functions do one thing
- [ ] Classes have single reason to change
- [ ] Files have cohesive purpose

### Comments
- [ ] Comments explain "why" not "what"
- [ ] No commented-out code
- [ ] TODO/FIXME comments have context/ticket reference
- [ ] Complex algorithms documented

### Code Organization
- [ ] Related code grouped together
- [ ] Public API at top, private implementation below
- [ ] Imports organized and minimal

## Design

### SOLID Principles
- [ ] Single Responsibility: One reason to change
- [ ] Open/Closed: Extensible without modification
- [ ] Liskov Substitution: Subtypes substitutable
- [ ] Interface Segregation: Specific interfaces
- [ ] Dependency Inversion: Depend on abstractions

### Coupling & Cohesion
- [ ] Low coupling between modules
- [ ] High cohesion within modules
- [ ] Dependencies explicit and injected
- [ ] Circular dependencies avoided

### API Design
- [ ] Consistent with existing patterns
- [ ] Backward compatible or migration provided
- [ ] Errors clearly documented
- [ ] Return types predictable

### Architectural Patterns
- [ ] Follows established patterns in codebase
- [ ] New patterns justified and documented
- [ ] Separation of concerns maintained
