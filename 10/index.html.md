---
date_published: 2026-05-11
date_modified: 2026-05-11
canonical_url: https://ike.network/ike-platform/ike-parent/ike-example-ws/index.html
---

# ike-example-ws

A workspace aggregator that orchestrates two consumer-side subprojects, `doc-example` and `example-project`. Cloning this repo and running `mvn ws:init` pulls those two down at the versions declared in `workspace.yaml`. A subsequent `mvn clean install` rebuilds them in dependency order from a single command.

The foundation repos `ike-tooling`, `ike-docs`, and `ike-platform` are **not** subprojects — they are consumed at their released versions from Nexus, not co-developed inside this workspace (see "Subprojects" below for the rationale). This is the canonical template for an **IKE workspace aggregator**.

| Coordinate | Value |
| --- | --- |
| Group ID | `local.aggregate` |
| Artifact | `ike-example-ws` |
| Packaging | POM (workspace aggregator) |
| Parent | `network.ike.platform:ike-parent` |

## [#role-in-the-ike-ecosystem](#role-in-the-ike-ecosystem)Role in the IKE Ecosystem

`ike-example-ws` is a **coordination layer**, not a component repo. It holds no production artifacts and contributes no code to releases. Its job is to make multi-repo operations — cross-repo feature branches, coordinated releases, dependency-version alignment — work from one command rather than four.

The two component-repo templates it complements:

| Template | Role |
| --- | --- |
| [example-project](https://ike.network/example-project/)[1] | `JAR + docs` consumer template — single-repo project that ships compiled Java code AND rendered AsciiDoc docs. |
| [doc-example](https://ike.network/doc-example/)[2] | Doc-only consumer template — single-repo project whose primary deliverable is a published document. |
| **`ike-example-ws`** **(this site)** | Workspace-aggregator template — orchestrates multiple component repositories into a single coordinated build/release/feature cycle. |

## [#subprojects](#subprojects)Subprojects

`workspace.yaml` declares two consumer-side subprojects this workspace manages:

| Subproject | Live site |
| --- | --- |
| `doc-example` | [ike.network/doc-example/](https://ike.network/doc-example/)[2] — Doc-only consumer template. Demonstrates the canonical `<packaging>pom</packaging>` + `adoc` classifier shape for a module whose primary deliverable is a published document. |
| `example-project` | [ike.network/example-project/](https://ike.network/example-project/)[1] — Hybrid Java+docs consumer template. Demonstrates a `jar`-packaged module that also ships rendered AsciiDoc on the side. |

The foundation repos (`ike-tooling`, `ike-docs`, `ike-platform`) are **intentionally not** workspace subprojects — they are upstream infrastructure consumed at their released versions from Nexus, not co-developed inside this workspace. See the comment block in `workspace.yaml` for the rationale (self-bootstrap concerns for `ike-tooling`, parent-cycle concerns for `ike-platform`).

| Foundation repo | Live site |
| --- | --- |
| `ike-tooling` | [ike.network/ike-tooling/](https://ike.network/ike-tooling/)[3] — Release orchestration, BOM generation, AsciiDoc utilities; source of `ike-maven-plugin`. |
| `ike-docs` | [ike.network/ike-docs/](https://ike.network/ike-docs/)[4] — Doc plumbing: `ike-doc-maven-plugin`, `koncept-asciidoc-extension`, `minimal-fonts`, `docbook-xsl`, `ike-doc-resources`, `semantic-linebreak`. |
| `ike-platform` | [ike.network/ike-platform/](https://ike.network/ike-platform/)[5] — Consumer-facing parent: `ike-parent`, `ike-bom`, `ike-workspace-maven-plugin`. |

## [#release-cascade](#release-cascade)Release Cascade

```
ike-tooling -> ike-docs -> ike-platform -> { example-project, doc-example } -> [ike-example-ws]
```

`ike-example-ws` is the last link in the cascade. Its role is to verify — through its integration tests — that releasing the full cascade produces consumable artifacts on Nexus before any tags are cut on the component repos.

## [#layout](#layout)Layout

```
ike-example-ws/
├── workspace.yaml              (1)
├── pom.xml                     (2)
├── ike-docs/                   (3)
├── ike-platform/               (3)
├── example-project/            (3)
├── doc-example/                (3)
├── its/                        (4)
└── src/site/                   (5)
```

The workspace manifest. Declares each subproject’s repo URL, branch, version, group ID, and dependency relationships. Workspace aggregator. Each subproject is in a file-activated profile so the reactor automatically includes only repos that are physically cloned. Cloned by `mvn ws:init` from `workspace.yaml`. Empty until then. Optional: integration test suite that exercises the full cascade end-to-end (e.g., "after releasing v1, a fresh `mvn verify` on doc-example succeeds against Nexus"). Maven Site Plugin source — what generates this site you’re reading.  

## [#key-workspace-commands](#key-workspace-commands)Key Workspace Commands

```
# Bootstrap: clone all subproject repos from their remotes
mvn ws:init

# Status: snapshot of workspace state, alignment, dirty repos
mvn ws:overview

# Full reactor build of all cloned subprojects
mvn clean install

# Daily-driver: pull, refresh-main, push across the workspace
mvn ws:sync

# Coordinated feature branch across all subprojects
mvn ws:feature-start-publish -Dfeature=my-feature
mvn ws:feature-finish-squash-publish -Dfeature=my-feature

# Coordinated multi-repo release
mvn ws:release-draft       # preview
mvn ws:release-publish     # execute
```

For the full reference of all 50 `ws:*` goals, see the [ws:* Goal Reference](https://ike.network/ike-platform/ike-workspace-maven-plugin/ws-goals.html)[6] on the IKE Platform site. For first-time setup of a workspace, see [Workspace Getting Started](https://ike.network/ike-platform/workspace-getting-started.html)[7].

## [#when-to-copy-this-template](#when-to-copy-this-template)When to Copy This Template

When creating a new IKE workspace aggregator (e.g., a workspace that holds a different set of subprojects, or one that adds new subprojects to the cascade), copy:

| File / Directory | Purpose |
| --- | --- |
| `workspace.yaml` | Declare your subprojects, their repo URLs, branches, and dependency relationships. The `ws:*` goals consult this file as source of truth. |
| `pom.xml` | Aggregator with file-activated profiles per subproject. Adjust the artifact ID and the profile blocks for your subproject set. |
| `src/site/` | Maven Site Plugin source. Update `site.xml` (workspace name, repo URL) and `asciidoc/index.adoc` for your project. Keep the Forest-theme `site.css` and `ike-logo.svg` for visual consistency with the rest of the IKE ecosystem. |
| `.mvn/jvm.config` | Single-line `--sun-misc-unsafe-memory-access=allow` to suppress JFFI deprecation warnings. NEVER add `#`-prefixed comments to this file — Maven parses each line as a raw JVM argument. |

## [#discipline-rules](#discipline-rules)Discipline Rules

1. **Use `ws:*` goals, never raw `git` for workspace ops.** The whole point of the workspace plugin is that one command stays coherent across all subprojects. Reaching into one subproject with raw git breaks the abstraction.
2. **All subprojects track the workspace’s git branch.** No per-subproject branch opt-out. See [ike-issues#233](https://github.com/IKE-Network/ike-issues/issues/233)[8] for the alignment model.
3. **Component repos commit and push direct to `main`.** No PRs in the IKE-Network organization. Releases cut from `main` via `ws:release-publish`.

## [#resources](#resources)Resources

| Resource | URL |
| --- | --- |
| Source repository | [https://github.com/IKE-Network/ike-example-ws](https://github.com/IKE-Network/ike-example-ws)[9] |
| Cross-project issue tracker | [https://github.com/IKE-Network/ike-issues](https://github.com/IKE-Network/ike-issues)[10] |
| IKE Network landing page | [https://ike.network/](https://ike.network/)[11] |
| IKE Platform (workspace plugin lives here) | [https://ike.network/ike-platform/](https://ike.network/ike-platform/)[5] |
| ws:* Goal Reference | [https://ike.network/ike-platform/ike-workspace-maven-plugin/ws-goals.html](https://ike.network/ike-platform/ike-workspace-maven-plugin/ws-goals.html)[6] |
| Workspace Getting Started | [https://ike.network/ike-platform/workspace-getting-started.html](https://ike.network/ike-platform/workspace-getting-started.html)[7] |
