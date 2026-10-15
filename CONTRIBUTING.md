# Contributing

This repository is a plugin marketplace. Each plugin is a standalone directory
at the repository root, or under `third_party/`, with its own manifest.

## Before you start

Run every command from the **repository root** — the directory containing
`.cursor-plugin/`, `schemas/`, and `scripts/`. `scripts/validate-plugins.mjs`
resolves the repository root from its own location, while `npm install` places
installed packages under your current directory.

Use **Node 20**, the version the repository's current workflow pins.

## Validate your change

Run these two commands, in this order:

```sh
npm install --no-save ajv ajv-formats
node scripts/validate-plugins.mjs
```

`npm install --no-save` installs those two schema validators for this run only,
without recording them in a dependency manifest or lockfile and without adding a
dependency to any plugin.

A nonzero result must be corrected before publication. Do not open a pull
request while `node scripts/validate-plugins.mjs` prints `ERROR:` lines or exits
nonzero; each error names the manifest that failed.

## What validation checks

**Each plugin's manifest at `<plugin>/.cursor-plugin/plugin.json`.** For every
plugin the marketplace lists, the script loads the manifest from that plugin's
own `.cursor-plugin/` directory — for example
`teaching/.cursor-plugin/plugin.json` or
`third_party/gmail/.cursor-plugin/plugin.json` — and validates it against
`schemas/plugin.schema.json`. The manifest must carry a `name`, and unknown
top-level fields are rejected, so a typo'd or invented key fails the run. That
`name` must match the marketplace entry's `name`.

**The marketplace manifest at `.cursor-plugin/marketplace.json`.** The script
validates it against `schemas/marketplace.schema.json`. It must declare a `name`
and a `plugins` array, and each entry must have a `name` and a `source`. The
script then resolves every `source` to a real directory in the checkout, failing
if that directory or the plugin manifest inside it is missing.

**The schemas.** `schemas/marketplace.schema.json` and
`schemas/plugin.schema.json` are the source of truth for both checks, applied
with `ajv` and `ajv-formats`. To change what a manifest may contain, change the
schema and the affected manifests together.

## Adding a plugin

1. Create the plugin directory at the repository root, or under `third_party/`
   for externally maintained integrations.
2. Add a manifest at `<plugin>/.cursor-plugin/plugin.json` with at least a
   lowercase kebab-case `name` and a `description`.
3. Add an entry to `.cursor-plugin/marketplace.json` whose `name` matches the
   manifest and whose `source` is the plugin's directory path.
4. Run the two validation commands above and fix every reported error.

The same two commands are recorded in `.github/workflows/validate-plugins.yml`.
