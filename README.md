# IKE Example Workspace

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Documentation](https://img.shields.io/badge/docs-ike.network%2Fike--example--ws-blue)](https://ike.network/ike-example-ws/)
[![IKE Network](https://img.shields.io/badge/IKE-Network-green)](https://ike.network/)

Workspace aggregator that orchestrates three consumer-side
subprojects (`doc-example`, `example-project`, `its`/`ike-example-its`)
against the IKE foundation repos (`ike-tooling`, `ike-docs`,
`ike-platform`) consumed from Nexus. Clone this repo and run
`mvn ws:init` to pull down the subprojects and build the cascade
as a single reactor.

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
git clone https://github.com/IKE-Network/ike-example-ws.git
cd ike-example-ws

mvn ws:init          # clone subprojects per workspace.yaml
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
`ws:init` clones them into this directory; `ws:release-publish`
releases them in dependency order.

| Repo | Artifact | Purpose |
|---|---|---|
| [doc-example](https://github.com/IKE-Network/doc-example) | `network.ike.examples:doc-example` | Doc-only template — `<packaging>pom</packaging>` with `adoc` source classifier |
| [example-project](https://github.com/IKE-Network/example-project) | `network.ike.examples:example-project` | Hybrid Java+docs template — `<packaging>jar</packaging>` with optional `adoc` classifier when `src/docs/asciidoc/` exists |
| [ike-example-its](https://github.com/IKE-Network/ike-example-its) | `network.ike.examples:ike-example-its` | Optional integration-test harness clones into `./its/`; a file-activated profile picks it up when present. Split from this workspace in [`#343`](https://github.com/IKE-Network/ike-issues/issues/343) so the IT harness can evolve on its own cadence. |

See [`workspace.yaml`](workspace.yaml) for version pins and dependency
declarations.

## Release Cascade

```
ike-tooling → ike-docs → ike-platform → [ workspace: { doc-example, example-project, ike-example-its } → ike-example-ws ]
```

`ike-tooling` is bootstrapped out-of-band (it releases itself using
the plugin it produces). The foundation tier is then orchestrated
by `cascade-foundation-publish` (in `ike-platform`'s
`ike-workspace-maven-plugin`), which walks `ike-tooling → ike-docs →
ike-platform` and then invokes the workspace's own
`ws:release-publish` to release the three subprojects in dependency
order, finishing with the workspace root.

## Integration Tests

See [`its/README.md`](its/README.md) for the IT suite layout
and how to add new end-to-end tests.

## Links

- **Documentation:** [`https://ike.network/ike-example-ws/`](https://ike.network/ike-example-ws/)
- **Subproject sites** (each publishes its own top-level gh-pages):
  - [`doc-example`](https://ike.network/doc-example/) — documentation-only example
  - [`example-project`](https://ike.network/example-project/) — Java + docs example
  - [`ike-example-its`](https://ike.network/ike-example-its/) — integration test harness
- **Foundation repos** (consumed from Nexus, not workspace subprojects):
  [`ike-tooling`](https://ike.network/ike-tooling/) ·
  [`ike-docs`](https://ike.network/ike-docs/) ·
  [`ike-platform`](https://ike.network/ike-platform/)
- **Issues:** [`IKE-Network/ike-issues`](https://github.com/IKE-Network/ike-issues) (cross-project tracker)
- **Source:** [`IKE-Network/ike-example-ws`](https://github.com/IKE-Network/ike-example-ws)

## History

Split from the archived `ike-pipeline` repo to resolve a Maven
extension-plugin reactor-load cycle. See
[`IKE-Network/ike-issues#216`](https://github.com/IKE-Network/ike-issues/issues/216).
