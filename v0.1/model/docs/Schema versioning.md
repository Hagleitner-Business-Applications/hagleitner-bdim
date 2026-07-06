# Schema Versioning

The BDIM uses a single, monotonically increasing version (e.g. `v0.1`, `v0.2`, `v1.0`). Each version lives in its own folder, and that version is part of every schema's `$id`:

```
https://schema.hagleitner.com/bdim/v0.1/<area>/<name>.schema.json
```

A new version is published only for breaking changes. Non-breaking changes are made in place within the current version.

In addition to the version, every schema carries an `x-revision` date that is updated on every in-place edit. The version identifies the contract; the revision identifies the specific state of that contract. Consumers can compare the `x-revision` of a cached schema against the one served by the schema server to detect updates without diffing the document.

Once a version is published, its `$id`s are **permanent**. Consumers may cache and pin them.

## When to edit in place vs. bump

Edit the current version in place for changes that cannot invalidate existing documents or mislead existing consumers:

- Adding an optional property.
- Loosening a constraint (wider `pattern`, higher `maxLength`, additional `enum` value, …).
- Documentation, examples, `title` / `description`.

Bump to a new version for any breaking change:

- Adding a required property.
- Removing or renaming a property.
- Tightening a constraint (narrower type, stricter `pattern`, lower `maxLength`, removing an `enum` value, …).
- Changing the semantics of an existing property.

Rule of thumb: if an existing valid document could become invalid, or an existing consumer could misinterpret the data, it's a breaking change — bump.

## When to update `x-revision`

Update `x-revision` to today's date (UTC, `YYYY-MM-DD`) on every in-place edit that changes the **contract or its documented meaning**:

- Any of the in-place changes listed above (new optional property, loosened constraint, …).
- Material changes to `description`, `title`, or examples that affect how a consumer interprets the schema.

Do **not** bump `x-revision` for changes that don't affect consumers:

- Whitespace, formatting, or key reordering.
- Internal comments or repository-only metadata (e.g. `x-status`).
- Edits to files outside the schema (docs, build scripts).

When a version is bumped (folder copy to `v0.2/`), reset `x-revision` in every file of the new folder to the bump date.

## How to bump

1. Copy the current version folder: `v0.1/` → `v0.2/`.
2. Rewrite every `$id` in the new folder from `.../bdim/v0.1/...` to `.../bdim/v0.2/...`.
3. Leave `v0.1/` frozen — no further edits except critical doc fixes.
4. Make the breaking changes in `v0.2/`.

## Conventions that keep this cheap

- Always use **relative `$ref`s** within a version (e.g. `../components/basetypes.schema.json`). A version bump then needs only the folder copy + `$id` rewrite — no `$ref` edits.
- The `$id` filename must match the actual file path on disk.
- Never reuse a published `$id` for a schema with different semantics.
