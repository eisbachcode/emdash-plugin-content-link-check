# emdash-plugin-content-link-check

The EmDash content-link-check plugin by Eisbachcode (`@eisbachcode/emdash-plugin-content-link-check`), not released yet.

Before editing this plugin, read `skills/creating-plugins/SKILL.md` completely. Codex discovers the same directory through `.agents/skills`; Claude discovers it through `.claude/skills` and reads these instructions through `.claude/CLAUDE.md`.
Keep `emdash-plugin.jsonc` aligned with the runtime implementation, declare every capability and host the plugin uses, and run the generated validation, typecheck, test, and build scripts after changes.

## Toolchain

`emdash` is a peer with a floor and no ceiling (`>=1.0.1`, the same as
`env:emdash` in the manifest). Never write the floor as `>=1.0.0` or
`^1.0.0`: npm carries an accidental, deprecated `emdash@1.0.0` published
in April 2026, five months before the real 1.0. The dev dependency stays
below the next major until the toolchain is moved on purpose. Built with
`@emdash-cms/plugin-cli@0.13.1` and `@emdash-cms/plugin-test@0.2.6` (EmDash 1.0.1);
`scripts/compat-matrix.sh` runs the suite against later EmDash releases.
A plain `pnpm install` is enough.

## Conventions

- Tabs. English in code, comments and docs.
- ESM: internal imports carry `.js`; `import type` for types.
- The `emdash` peer range has a floor and no ceiling, and the floor matches
  `release.requires["env:emdash"]` in the manifest. Compatibility with later
  EmDash releases is checked by `scripts/compat-matrix.sh`, not promised by
  the range.
- Read EmDash APIs from the published release or from `upstream/main`, and
  say which. A fork's `main` drifts behind without anyone noticing.
- Anything that talks HTTP takes an injected `fetch`. Whole sync ticks go
  through the test host's `host.http.respond()`.
- A sandboxed invocation gets ten subrequests and every `ctx` call is one.
  Plugins with a budget test count each invocation's worst case there; run
  it after any change that adds a `ctx` call.
- A test must be able to fail on a real regression. Do not assert a config
  literal back at itself or restate the implementation.

## Checks

```sh
pnpm install
pnpm typecheck
pnpm test        # emdash-plugin validate, then vitest
pnpm build
./scripts/compat-matrix.sh 1.0.1   # suites against other EmDash releases
```

## Releases

Versions come from changesets: add one with `pnpm changeset` for every
change to a published package, written for someone upgrading.

Until its first release the package stays `"private": true`. A release
workflow publishes a public package whose version is missing from npm,
changeset or not, so a public package would go out with its first merge to
`main`. The release PR removes the flag and the README's status note, and
adds the package's first changeset. That first publish is manual (`pnpm publish`):
npm's trusted-publisher setting can only be added to a package that exists.

Listing images go in `images/` and are declared under
`release.artifacts.screenshots` in the manifest. Never use a `screenshots/`
folder: the registry bundle takes it whole and refuses a bundle over 256 KB
or an image over 128 KB. The registry also refuses a manifest `description`
over 140 graphemes, which `emdash-plugin validate` does not check.

Never name a script `publish`, `version` or `prepare`: npm and pnpm
run scripts with those names on their own during a publish or a version
bump. `emdash-plugin init` generates `"publish": "emdash-plugin publish"`,
which would push to the EmDash registry after every `npm publish`; here it
is `registry:publish`, and `prepublishOnly` builds before any publish.

`.github/workflows/release.yml` does the npm side: with changesets on main
it opens a "Version Packages" pull request, and merging that publishes
through npm trusted publishing. npm trusts that workflow by its file name,
so renaming it breaks publishing until the setting on npmjs.com changes too.
The npm publish job runs in the `npm` environment, which the npm trusted
publisher requires.

The plugin is not in the EmDash registry yet, and nothing here publishes to
it. Listing it is a separate decision: `pnpm release:setup` generates
`emdash-release.yml`, which `release.yml` then calls after an npm publish
(the analytics plugin's repository has the finished pair, including the
`github.event_name` fix the generated plan step needs), and `pnpm exec
emdash-plugin profile setup` creates the registry profile. Do neither without
that decision.

The repository installs with pnpm 11; `allowBuilds` in `pnpm-workspace.yaml` lets esbuild
and workerd run their install scripts, which the test host needs.
