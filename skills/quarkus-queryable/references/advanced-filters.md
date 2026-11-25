# Advanced filters (Queryable 4.0.3)

## Fixed conditions and dates

`@Q(condition = "PUBLIC", options = QOption.EXECUTE_ALWAYS)` applies typed equality on every collection query; `@QLike` supports the same option for a case-insensitive substring. The condition must be nonempty and valid. Supplied query parameters for that annotation, including ranges, are ignored. A condition without `EXECUTE_ALWAYS` is invalid. Options on `@QList`, `@QLikeList`, `@QNil` and `@QNotNil` are currently ignored.

For `java.util.Date`, date-only ranges use UTC day boundaries and are inclusive; date-only equality uses UTC midnight. Offset date-times represent exact instants; local date-times and supported legacy date-time formats use UTC. Invalid values return 400; URL-encode `+` and spaces.

For example, `from.createdAt=2026-03-10` starts at `2026-03-10T00:00:00.000Z`, and `to.createdAt=2026-03-10` ends at `2026-03-10T23:59:59.999Z`. Supported legacy forms are `yyyy-MM-dd HH:mm[:ss]`, `dd/MM/yyyy HH:mm[:ss]` and `dd/MM/yyyy, HH:mm[:ss]`.

## Custom Criteria filters

Put an application-scoped `QueryableFilter<Entity>` implementation in `service.filters` under the application's package convention. The entity type is inferred, including through generic superclasses. `contribute(QueryableFilterContext<Entity> context)` can use `param`, `params`, `criteriaBuilder`, `root` and `add` to build predicates. `orParams("address.city")` reads and marks all `or.address.city` occurrences as handled; validate them and use `addOr` for each predicate. Custom parameters are not automatically documented in OpenAPI.

```java
@ApplicationScoped
public class ProductFilter implements QueryableFilter<Product> {
    @Override
    public void contribute(QueryableFilterContext<Product> context) {
        String code = context.param("category.code");
        if (code != null) {
            context.add(context.criteriaBuilder().equal(
                    context.root().get("category").get("code"), code));
        }
    }
}
```

## Hibernate filters

`@QFilter` connects a REST parameter to an existing Hibernate `@FilterDef` and `@Filter`; Hibernate owns the SQL and aliases. Example:

```java
@FilterDef(name = "Product.minPrice", parameters = @ParamDef(name = "minimum", type = BigDecimal.class))
@Filter(name = "Product.minPrice", condition = "price >= :minimum")
@QFilter(name = "Product.minPrice", queryParam = "min.price", filterParam = "minimum")
```

The REST parameter activates the filter and is converted to the Hibernate parameter type; invalid values return 400. For multiple Hibernate parameters, repeat `@QFilter` with the same name; when activated, all parameters without resolvers are required. An already enabled filter is not overwritten and activation fails. `QOption.WITHOUT_PARAMETERS` permits activation by parameter presence, ignoring its value; omit `filterParam` and use one mapping for the filter. Hibernate parameters must be absent or have resolvers.

Filters apply to collection results and `listSize` and are cleaned up after use, including on failure. They do not filter `/count` or other CRUD operations. Use named SQL parameters for input; optional filters and fixed conditions do not provide authorization.
