---
name: quarkus-queryable
description: Configure and troubleshoot Quarkus Queryable generated REST collection endpoints, annotation-driven query parameters, and custom or Hibernate filters in consuming applications.
---

# Quarkus Queryable

Use this skill for applications consuming `dev.queryable:quarkus-queryable`. These instructions describe the 4.0.3 API; check the application's installed version and its documentation before relying on features that may differ. Inspect the application's entities, dependency management, security and existing tests before changing them.

## Set up and expose a collection

- Add the extension with a version compatible with the application, following its dependency conventions. The application must also choose a JDBC driver and JSON provider. Do not add another resource for the same generated route.
- Annotate a JPA Panache entity with `@QRs("/api/products")`, using imports from `io.quarkiverse.queryable.runtime`. Add field-level query annotations only for fields intended to be searchable. `@QOrderBy("name asc")` sets the default order.
- REST Data Panache provides CRUD and `/count`; Queryable supplies the collection GET. Filters affect the collection and its `listSize` header, not the separate `/count` or other CRUD endpoints.
- Apply normal Quarkus HTTP authorization to both the collection path and item paths. Query parameters, fixed conditions and logical deletion are not authorization controls.

For example, `@QLike` on `name` enables `GET /api/products?like.name=book`, and `@Q` on `price` enables `?from.price=10&to.price=20`. An annotation's configured name or prefix can change its parameter; check the actual entity before constructing a request. A `@QLogicalDelete` field defaults to active but `?obj.active=false` can request inactive records.

## Choose filters

| Annotation | Default query parameter | Meaning |
| --- | --- | --- |
| `@Q` | `obj.field` | Equality; `from.field` and `to.field` for supported comparable scalars |
| `@QLike` | `like.field` | Case-insensitive substring |
| `@QList` | `obj.field` | Comma-separated IN |
| `@QLikeList` | `like.field` | OR of comma-separated substrings |
| `@QNil` | `nil.field` | IS NOT NULL (legacy naming) |
| `@QNotNil` | `notNil.field` | IS NULL (legacy naming) |
| `@QLogicalDelete` | `obj.field` | Defaults to the true/active value |

Class-level `@Q` exposes supported persistent scalar and element-collection fields, including inherited fields; it can expose sensitive fields. Static, transient, embedded and association fields are not exposed automatically. Prefer field annotations for a restricted surface. A field-level `@Q` overrides that field's generated name and prefix; class-level attributes do not configure individual fields. Element collections use membership and do not support ranges.

Prefix supported parameters with `or.` to combine all such occurrences into one OR group, ANDed with ordinary filters. For example, `?obj.country=IT&or.obj.name=Alice&or.obj.name=Bob` matches the country and either name. Logical-delete, fixed and Hibernate-filter predicates remain in AND; unsupported or invalid `or.*` parameters return 400. Collection filtering and `listSize` agree before pagination.

For fixed predicates, date semantics, custom Criteria filters or Hibernate `@QFilter`, read [advanced filters](references/advanced-filters.md). For lifecycle hooks, use the separate `quarkus-queryable-lifecycle` skill if installed.

Test through the consuming application's normal Quarkus test setup: exercise matching and nonmatching values, combined filters, `listSize` with pagination, invalid inputs where relevant, and authorization when changed. Verify the separate `/count` behavior when a caller depends on filtered totals.
