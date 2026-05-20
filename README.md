# IKE Workspace Example

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Documentation](https://img.shields.io/badge/docs-ike.network%2Fworkspace--example-blue)](https://ike.network/workspace-example/)
[![IKE Network](https://img.shields.io/badge/IKE-Network-green)](https://ike.network/)

> **Note (2026-05-20):** This workspace was previously named
> `ike-example-ws`, with subprojects `example-project` and `its`
> (the `ike-example-its` repo). All four were renamed under the
> canonical naming policy in IKE-Network/ike-issues#467 so the
> artifact ID, git repo name, on-disk directory, and
> workspace.yaml subproject key all match. GitHub redirects keep
> old clone URLs working.

Workspace aggregator that orchestrates three consumer-side
subprojects (`doc-example`, `project-example`,
`integration-tests-example`) against the IKE foundation repos
(`ike-tooling`, `ike-docs`, `ike-platform`) consumed from Nexus.
Clone this repo and run `mvn ws:scaffold-init` to pull down the
subprojects and build the cascade as a single reactor.

## What this workspace is for

- **Local development** of the consumer-side subprojects against
  released foundation versions in one editor / one command.
- **Coordinated releases** of the subprojects via `mvn ws:release-publish`.
- **End-to-end integration tests** that exercise external Nexus
  consumption of the post-`IKE-Network/ike-issues#321`
  classifier-canonical doc shape (`<packaging>pom</packaging>` +
  `<classifier>adoc</classifier>` source attachment).

## Bootstrap

```bash
git clone https://github.com/IKE-Network/workspace-example.git
cd workspace-example

mvn ws:scaffold-init # clone subprojects per workspace.yaml
mvn ws:overview      # print the workspace dashboard
mvn clean install    # build the full cascade
```

## Foundation repos (consumed from Nexus, NOT workspace subprojects)

These are the upstream IKE infrastructure projects this workspace
consumes at their released versions. They are intentionally **not**
managed by this workspace — they release on their own cadence via
their own `ike:release-publish`. See the comment block in
[`workspace.yaml`](workspace.yaml) for the rationale (self-bootstrap
for `ike-tooling`, parent-cycle avoidance for `ike-platform`).

| Repo | Artifact | Purpose |
|---|---|---|
| [ike-tooling](https://github.com/IKE-Network/ike-tooling) | `network.ike.tooling:*` | Release orchestration, BOM generation, AsciiDoc utilities; source of `ike-maven-plugin` |
| [ike-docs](https://github.com/IKE-Network/ike-docs) | `network.ike.docs:*` | AsciiDoc plumbing: `ike-doc-maven-plugin` (render goals), Koncept extension, fonts, DocBook XSL, shared resources |
| [ike-platform](https://github.com/IKE-Network/ike-platform) | `network.ike.platform:*` | Consumer-facing parent: `ike-parent`, `ike-bom`, `ike-workspace-maven-plugin` |

## Subprojects (managed by this workspace)

These are the consumer-side demos the workspace orchestrates.
`ws:scaffold-init` clones them into this directory; `ws:release-publish`
releases them in dependency order.

| Repo | Artifact | Purpose |
|---|---|---|
| [doc-example](https://github.com/IKE-Network/doc-example) | `network.ike.examples:doc-example` | Doc-only template — `<packaging>pom</packaging>` with `adoc` source classifier |
| [project-example](https://github.com/IKE-Network/project-example) | `network.ike.examples:project-example` | Hybrid Java+docs template — `<packaging>jar</packaging>` with optional `adoc` classifier when `src/docs/asciidoc/` exists |
| [integration-tests-example](https://github.com/IKE-Network/integration-tests-example) | `network.ike.examples:integration-tests-example` | Optional integration-test harness; a file-activated profile picks it up when present. Split from this workspace in [`#343`](https://github.com/IKE-Network/ike-issues/issues/343) so the IT harness can evolve on its own cadence. |

See [`workspace.yaml`](workspace.yaml) for version pins and dependency
declarations.

## Release Cascade

```
ike-tooling → ike-docs → ike-platform → [ workspace: { doc-example, project-example, integration-tests-example } → workspace-example ]
```

`ike-tooling` is bootstrapped out-of-band (it releases itself using
the plugin it produces). The foundation tier is then orchestrated
by `cascade-foundation-publish` (in `ike-platform`'s
`ike-workspace-maven-plugin`), which walks `ike-tooling → ike-docs →
ike-platform` and then invokes the workspace's own
`ws:release-publish` to release the three subprojects in dependency
order, finishing with the workspace root.

## Integration Tests

See [`integration-tests-example/README.md`](integration-tests-example/README.md)
for the IT suite layout and how to add new end-to-end tests.

## Doc as Code + LLM-Friendly

This workspace follows the IKE Network's doc-as-code philosophy:
build conventions, documentation standards, and AI-assistant
guidance live as versioned Markdown files in
[`ike-build-standards`](https://github.com/IKE-Network/ike-tooling/tree/main/ike-build-standards#readme)
and are unpacked into every consumer's `.claude/standards/` at
the `validate` phase. When a developer — or Claude itself —
opens any IKE project, the agent reads those standards and
applies them automatically; contributors don't have to memorize
the conventions.

The standards most directly relevant to a workspace consumer are
[`IKE-WORKSPACE.md`](https://github.com/IKE-Network/ike-tooling/blob/main/ike-build-standards/src/main/standards/IKE-WORKSPACE.md)
(workspace.yaml layout, `ws:*` goals) and
[`IKE-RELEASE.md`](https://github.com/IKE-Network/ike-tooling/blob/main/ike-build-standards/src/main/standards/IKE-RELEASE.md)
(cascade procedure). See the
[full inventory](https://github.com/IKE-Network/ike-tooling/tree/main/ike-build-standards#readme).

## Links

- **Documentation:** [`https://ike.network/workspace-example/`](https://ike.network/workspace-example/)
- **Subproject sites** (each publishes its own top-level gh-pages):
  - [`doc-example`](https://ike.network/doc-example/) — documentation-only example
  - [`project-example`](https://ike.network/project-example/) — Java + docs example
  - [`integration-tests-example`](https://ike.network/integration-tests-example/) — integration test harness
- **Foundation repos** (consumed from Nexus, not workspace subprojects):
  [`ike-tooling`](https://ike.network/ike-tooling/) ·
  [`ike-docs`](https://ike.network/ike-docs/) ·
  [`ike-platform`](https://ike.network/ike-platform/)
- **Build standards:** [`ike-build-standards`](https://ike.network/ike-tooling/ike-build-standards/)
- **Issues:** [`IKE-Network/ike-issues`](https://github.com/IKE-Network/ike-issues) (cross-project tracker)
- **Source:** [`IKE-Network/workspace-example`](https://github.com/IKE-Network/workspace-example)

## History

Split from the archived `ike-pipeline` repo to resolve a Maven
extension-plugin reactor-load cycle. See
[`IKE-Network/ike-issues#216`](https://github.com/IKE-Network/ike-issues/issues/216).
<!-- BEGIN ike-managed: developer-setup -->

## Developer Setup

New to IKE development? The
[Developer Environment guide](https://ike.network/ike-tooling/ike-build-standards/developer-environment.html)
covers IDE configuration, JDK 25 setup, and the tooling conventions
every IKE workspace expects — start there before your first build.
<!-- END ike-managed: developer-setup -->
