---
date_published: 2026-05-26
date_modified: 2026-05-26
canonical_url: https://ike.network/ike-platform/ike-parent/workspace-reactor-example/index.html
---

# workspace-reactor-example

A workspace aggregator that orchestrates three consumer-side subprojects: `doc-example`, `project-example`, and `integration-tests-example`. Cloning this repo and running `mvn ws:scaffold-init` pulls all three down at the versions declared in `workspace.yaml`. A subsequent `mvn clean install` rebuilds them in dependency order from a single command.

Originally `ike-example-ws` (with subprojects `example-project` and `its`); renamed to `workspace-example` on 2026-05-20 under the canonical naming policy ([ike-issues#467](https://github.com/IKE-Network/ike-issues/issues/467)[1]): artifact ID, git repo name, on-disk directory, and workspace.yaml subproject key must all match. Renamed again to `workspace-reactor-example` on 2026-05-23 under the `-wsr` aggregator-reactor suffix policy — see the `arch-workspace-and-workspace-reactor` topic in `ike-lab-documents` for the vocabulary. GitHub redirects keep old clone URLs working.

The foundation repos `ike-tooling`, `ike-docs`, and `ike-platform` are **not** subprojects — they are consumed at their released versions from Nexus, not co-developed inside this workspace (see "Subprojects" below for the rationale). This is the canonical template for an **IKE workspace aggregator**.

| Coordinate | Value |
| --- | --- |
| Group ID | `network.ike.examples` |
| Artifact | `workspace-reactor-example` |
| Packaging | POM (workspace aggregator) |
| Parent | `network.ike.platform:ike-parent` |

## [#role-in-the-ike-ecosystem](#role-in-the-ike-ecosystem)Role in the IKE Ecosystem

`workspace-reactor-example` is a **coordination layer**, not a component repo. It holds no production artifacts and contributes no code to releases. Its job is to make multi-repo operations — cross-repo feature branches, coordinated releases, dependency-version alignment — work from one command rather than four.

The two component-repo templates it complements:

| Template | Role |
| --- | --- |
| [project-example](https://ike.network/project-example/)[2] | `JAR + docs` consumer template — single-repo project that ships compiled Java code AND rendered AsciiDoc docs. |
| [doc-example](https://ike.network/doc-example/)[3] | Doc-only consumer template — single-repo project whose primary deliverable is a published document. |
| **`workspace-reactor-example`** **(this site)** | Workspace-aggregator template — orchestrates multiple component repositories into a single coordinated build/release/feature cycle. |

## [#subprojects](#subprojects)Subprojects

`workspace.yaml` declares three consumer-side subprojects this workspace manages:

| Subproject | Live site |
| --- | --- |
| `doc-example` | [ike.network/doc-example/](https://ike.network/doc-example/)[3] — Doc-only consumer template. Demonstrates the canonical `<packaging>pom</packaging>` + `adoc` classifier shape for a module whose primary deliverable is a published document. |
| `project-example` | [ike.network/project-example/](https://ike.network/project-example/)[2] — Hybrid Java+docs consumer template. Demonstrates a `jar`-packaged module that also ships rendered AsciiDoc on the side. |
| `integration-tests-example` | [ike.network/integration-tests-example/](https://ike.network/integration-tests-example/)[4] — Integration-test harness. Exercises the full release cascade end-to-end against published Nexus artifacts (e.g., "after releasing `doc-example` v1, a fresh `mvn verify` against Nexus succeeds"). Lives in its own repo so it can iterate independently of the templates it tests (IKE-Network/ike-issues#343); cloned in here on demand by `ws:scaffold-init`. |

The foundation repos (`ike-tooling`, `ike-docs`, `ike-platform`) are **intentionally not** workspace subprojects — they are upstream infrastructure consumed at their released versions from Nexus, not co-developed inside this workspace. See the comment block in `workspace.yaml` for the rationale (self-bootstrap concerns for `ike-tooling`, parent-cycle concerns for `ike-platform`).

| Foundation repo | Live site |
| --- | --- |
| `ike-tooling` | [ike.network/ike-tooling/](https://ike.network/ike-tooling/)[5] — Release orchestration, BOM generation, AsciiDoc utilities; source of `ike-maven-plugin`. |
| `ike-docs` | [ike.network/ike-docs/](https://ike.network/ike-docs/)[6] — Doc plumbing: `ike-doc-maven-plugin`, `koncept-asciidoc-extension`, `minimal-fonts`, `docbook-xsl`, `ike-doc-resources`, `semantic-linebreak`. |
| `ike-platform` | [ike.network/ike-platform/](https://ike.network/ike-platform/)[7] — Consumer-facing parent: `ike-parent`, `ike-bom`, `ike-workspace-maven-plugin`. |

## [#release-cascade](#release-cascade)Release Cascade

```
ike-tooling -> ike-docs -> ike-platform -> { doc-example, project-example, integration-tests-example } -> [workspace-reactor-example]
```

`workspace-reactor-example` is the last link in the cascade. Its role is to verify — through its integration tests — that releasing the full cascade produces consumable artifacts on Nexus before any tags are cut on the component repos.

## [#layout](#layout)Layout

```
workspace-reactor-example/
├── workspace.yaml              (1)
├── pom.xml                     (2)
├── .mvn/extensions.xml         (3)
├── doc-example/                (4)
├── project-example/            (4)
├── integration-tests-example/  (5)
└── src/site/                   (6)
```

The workspace manifest. Declares each subproject’s repo URL, branch, version, group ID, and dependency relationships. Workspace aggregator. Declares `<subprojects>` directly at top level — the `ike-workspace-extension` (registered in `.mvn/extensions.xml`) prunes entries whose directory is not yet on disk so a fresh clone can bootstrap via `mvn ws:scaffold-init` before any subproject is cloned. See IKE-Network/ike-issues#460. Registers `network.ike.tooling:ike-workspace-extension` so Maven loads the extension on every build. The version literal is managed by `ws:scaffold-publish`. Worked example subprojects — cloned by `ws:scaffold-init` from `workspace.yaml`. The foundation (`ike-tooling`, `ike-docs`, `ike-platform`, `ike-workspace-extension`, `ike-base-parent`) is intentionally **not** a subproject — it’s consumed at its released versions from Maven Central. Optional: the IT harness (`integration-tests-example`) that exercises the full release cascade end-to-end (e.g., "after releasing `doc-example` v1, a fresh `mvn verify` succeeds against Nexus"). Lives in its own repo (#343), cloned in here on demand. Maven Site Plugin source — what generates this site you’re reading.  

## [#key-workspace-commands](#key-workspace-commands)Key Workspace Commands

```
# Bootstrap: clone all subproject repos from their remotes
mvn ws:scaffold-init

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

For the full reference of all 50 `ws:*` goals, see the [ws:* Goal Reference](https://ike.network/ike-platform/ike-workspace-maven-plugin/ws-goals.html)[8] on the IKE Platform site. For first-time setup of a workspace, see [Workspace Getting Started](https://ike.network/ike-platform/workspace-getting-started.html)[9].

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
2. **All subprojects track the workspace’s git branch.** No per-subproject branch opt-out. See [ike-issues#233](https://github.com/IKE-Network/ike-issues/issues/233)[10] for the alignment model.
3. **Component repos commit and push direct to `main`.** No PRs in the IKE-Network organization. Releases cut from `main` via `ws:release-publish`.

## [#not-published-to-maven-central](#not-published-to-maven-central)Not published to Maven Central

`workspace-reactor-example` is a reference template for an IKE workspace aggregator, not a library. It holds no production artifacts and is deliberately **not** published to Maven Central — nothing should ever declare a dependency on it. You consume it by copying its `workspace.yaml`, aggregator `pom.xml`, and `src/site/` into a workspace of your own. The IKE foundation (`ike-base-parent`, `ike-tooling`, `ike-docs`, `ike-platform`) is the part published to Central and meant to be inherited; see [the IKE Network landing page](https://ike.network/)[11] for the foundation/examples split.

## [#resources](#resources)Resources

| Resource | URL |
| --- | --- |
| Source repository | [https://github.com/IKE-Network/workspace-reactor-example](https://github.com/IKE-Network/workspace-reactor-example)[12] |
| Cross-project issue tracker | [https://github.com/IKE-Network/ike-issues](https://github.com/IKE-Network/ike-issues)[13] |
| IKE Network landing page | [https://ike.network/](https://ike.network/)[11] |
| IKE Platform (workspace plugin lives here) | [https://ike.network/ike-platform/](https://ike.network/ike-platform/)[7] |
| ws:* Goal Reference | [https://ike.network/ike-platform/ike-workspace-maven-plugin/ws-goals.html](https://ike.network/ike-platform/ike-workspace-maven-plugin/ws-goals.html)[8] |
| Workspace Getting Started | [https://ike.network/ike-platform/workspace-getting-started.html](https://ike.network/ike-platform/workspace-getting-started.html)[9] |
