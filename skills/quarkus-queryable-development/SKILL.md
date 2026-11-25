---
name: quarkus-queryable-development
description: Maintain Quarkus Queryable itself and update its consumer skills when extension changes materially affect application developers.
---

# Develop Quarkus Queryable

Use this skill when changing this extension's runtime, deployment processor, public API or consumer-facing documentation. It is for repository maintainers, not applications consuming the extension.

1. Follow the repository's `AGENTS.md`. Inspect the affected implementation, tests and `docs/modules/ROOT/pages/` before changing behavior.
2. Assess whether the change affects consumer setup, public annotations or APIs, generated endpoints, query semantics, lifecycle behavior, compatibility, security guidance or observable errors. Internal refactors that preserve the consumer contract normally need no skill change.
3. When consumer guidance changes, update the relevant documentation and the corresponding portable skill in the same change: `skills/quarkus-queryable/` for setup and filtering, or `skills/quarkus-queryable-lifecycle/` for listeners. Update linked references and examples as necessary. Verify behavior against implementation and integration or consumer tests; do not describe planned behavior as released. Keep version caveats accurate, especially for unreleased features or changed minimum versions.
4. Run focused tests appropriate to the change. For extension-wide changes, consider the repository's Maven integration tests and the standalone `consumer-tests` instructions in `docs/modules/ROOT/pages/testing.adoc`. Validate edited skills' frontmatter, names, relative links and installability; avoid coupling them to files outside their own skill directories.

If no skill update is warranted, say so briefly when reporting a meaningful platform change. Keep the skill concise and avoid making unrelated edits solely to synchronize wording.
