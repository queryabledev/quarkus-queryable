---
name: quarkus-queryable-lifecycle
description: Add or troubleshoot Quarkus Queryable REST lifecycle listeners for validation, auditing, and post-commit behavior in consuming applications.
---

# Quarkus Queryable REST lifecycle

These instructions describe Queryable 4.0.3. Inspect the application's installed version and existing listeners, transactions, security and tests before making changes.

Create an `@ApplicationScoped` class in the application's `service.rs` package implementing `QueryableRsListener<Entity>`. The entity type is inferred, including through generic superclasses. Use callback annotations from `io.quarkiverse.queryable.runtime`, not similarly named JPA annotations:

- Fetch: `@PreFetch`, `@PostFetch`; list: `@PreList`, `@PostList`.
- Create: `@PrePersist`, `@PostPersist`; update: `@PreUpdate`, `@PostUpdate`; delete: `@PreDelete`, `@PostDelete`.

A callback may take no argument, the entity, its identifier, `List<Entity>` for `@PostList`, or `QueryableEvent<Entity>` for operation metadata. Check which values are available for the operation before accessing event optionals.

Pre-write callbacks run before persistence and may mutate or reject the entity. For request validation, throw an appropriate Jakarta REST exception such as `BadRequestException`; unmapped runtime exceptions are server errors. For auditing, inject `SecurityIdentity` and set server-owned fields; restore immutable creation fields during updates.

```java
@ApplicationScoped
public class ProductRsListener implements QueryableRsListener<Product> {
    @PrePersist
    @PreUpdate
    void validate(Product product) {
        if (product.name == null || product.name.isBlank()) {
            throw new BadRequestException("name is required");
        }
    }
}
```

An invalid POST or PUT returns HTTP 400 without performing the write. Use a Jakarta REST `ExceptionMapper` if the application requires a specific error body.

Post-write callbacks run after transaction commit via JTA synchronization. Their exceptions are logged and cannot roll back the committed transaction. Design external side effects accordingly; do not promise atomic delivery from a callback alone.

Test the consuming application's HTTP operations and persistence: verify accepted and rejected writes, audit fields and rollback behavior as applicable, and observe post-commit effects after a successful commit. Use normal Quarkus security policies for authorization; listeners do not replace route protection.
