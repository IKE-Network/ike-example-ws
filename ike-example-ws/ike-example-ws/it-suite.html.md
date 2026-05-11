---
date_published: 2026-05-10
date_modified: 2026-05-10
canonical_url: https://ike.network/ike-platform/ike-parent/ike-example-ws/it-suite.html
---

# Integration Test Suite

The `its/` reactor is the **executable specification** for cross-cascade behavior in the IKE Network release stack. It runs the five-repo (or six-repo, including this workspace) build chain the way an external consumer would — pulling artifacts from Nexus, scaffolding a fresh project layout, and asserting that the documented pipeline actually works end-to-end.

The IT harness lives in its own repo, [ike-example-its](https://github.com/IKE-Network/ike-example-its)[1] (extracted in #343), and is cloned into this workspace as an **optional** subproject:

```
cd ike-example-ws
mvn ws:init                  # clones doc-example, example-project, its
mvn verify -pl its           # run the IT suite
```

When `its/` is present, the workspace’s file-activated profile picks it up as a reactor module. When it’s absent — for a leaner build, or because a workflow doesn’t need IT runs — the workspace builds without it. The same harness can be checked out and run standalone (without a workspace) from its own repo.

## [#why-an-it-suite-at-this-layer](#why-an-it-suite-at-this-layer)Why an IT suite at this layer?

The three foundation reactors (ike-tooling, ike-docs, ike-platform) plus this workspace’s two consumers (doc-example, example-project) each have their own unit and integration tests inside their own modules. Those catch regressions inside a single reactor.

What they don’t catch:

- **Cross-reactor coordination bugs.** A change in `ike-tooling` that breaks `ike-doc-maven-plugin` (consumed by `ike-docs`) only surfaces when an actual consumer cascades through Nexus.
- **Newly-published artifact resolution.** A consumer building against `ike-platform v30` succeeds locally but breaks for a fresh clone if `ike-bom` v30 was never deployed correctly.
- **Scaffold + extension delivery.** The consumer-side `.mvn/extensions.xml` for `wagon-ssh-external` (#338) reaches external projects via the `ike:scaffold-publish` flow — that delivery only exercises in a fresh project tree.
- **Built-With supplement classifier delivery.** The `ike-build-standards:built-with:zip` classifier (#340) is unpacked at `initialize` phase by `maven-dependency-plugin` — works only when the dep declaration in `ike-parent` reaches the consumer’s effective POM.

Each of those is testable inside a fresh `maven-invoker-plugin` sandbox, and each represents a real regression the IKE Network has hit at least once during the v144→v150 cascade work.

## [#planned-it-inventory](#planned-it-inventory)Planned IT inventory

The cases below are listed in `ike-example-its’s `README.adoc` as the intended coverage. Implementations are pending — `src/it/` in that repo currently holds only `settings.xml`. Each case is small (a handful of files) and runs in seconds when the artifacts under test are present in the local repository.

| IT case | What it proves | Catches |
| --- | --- | --- |
| `src/it/doc-only/` | Minimal `<packaging>pom</packaging>` project with `src/docs/asciidoc/`. The doc-pipeline profile in `ike-parent` auto-activates on `src/docs/asciidoc/` existence; the `maven-assembly-plugin` execution attaches a `<classifier>adoc</classifier><type>zip</type>` artifact. | Doc pipeline regressions: profile activation, assembly descriptor paths, classifier name (post-#321 the canonical classifier is `adoc`, not the retired `ike-doc` packaging). |
| `src/it/java-plus-docs/` | `<packaging>jar</packaging>` consumer with both `src/main/java/` and `src/docs/asciidoc/`. A single `mvn verify` produces the JAR plus the `adoc`-classifier zip plus the rendered HTML/PDF outputs. | Multi-output projects: doc pipeline shouldn’t disable the default jar lifecycle, classifier collisions shouldn’t occur. |
| `src/it/bom-import/` | Consumer that imports `network.ike.platform:ike-bom` via `<scope>import</scope>` but does NOT inherit `ike-parent`. The BOM’s managed dependency versions flatten correctly at consumer build time. | Maven 4 consumer-POM constraint: BOM imports must produce a fully-resolved consumer POM at install time, with no unresolved property references reaching downstream consumers. |
| `src/it/workspace-create/` | Invokes `ws:create` from a scratch directory and asserts the generated workspace pom + workspace.yaml reference `network.ike.platform:ike-parent` at the released version (not legacy `local.aggregate` placeholder per #183). | `ws:create` template regressions; placeholder-vs-real-coordinates drift; ike-parent version pinning. |

Additional cases worth adding as the cascade matures (not yet in the README’s plan):

| IT case | What it would prove |
| --- | --- |
| `src/it/scaffold-extensions/` | A fresh consumer running `ike:scaffold-publish` ends up with `.mvn/extensions.xml` containing `wagon-ssh-external`. Catches regressions in the scaffold-zip → consumer-disk delivery for classified config (#338). |
| `src/it/built-with-classifier/` | External consumer’s `mvn site` produces `built-with.html` with the Curated narrative section pulled from the platform-wide `supplement.yaml` unpacked from `ike-build-standards:built-with:zip`. Catches regressions in the unpack-then-mojo-read pipeline for classifier-distributed content (#340 + its phase / filename follow-ups). |
| `src/it/sbom-shape/` | Verifies `target/bom.json` validates against the CycloneDX 1.6 schema and contains expected components for a known-shape consumer. Catches regressions in the SBOM generation that #333 set up. |
| `src/it/release-cascade/` | Heavier — exercises a full `ike-tooling → ike-docs → ike-platform → consumer` cascade in isolated local repos with synthetic versions. Catches inter-repo coordination bugs (the kind that hit during the v148/v149 cascades and required intermediate fixes). |

## [#how-to-run](#how-to-run)How to run

```
# From the workspace root, exercising the released artifacts
# already in the local repository (requires `its/` to be cloned;
# `mvn ws:init` does that, or `git clone .../ike-example-its.git its`):
mvn verify -pl its

# To force a clean Nexus pull (ignore local repo cache):
mvn verify -pl its -U

# To run a single case:
mvn invoker:run -pl its -Dinvoker.test=doc-only

# Or standalone, from the IT harness's own checkout:
cd ../ike-example-its
mvn verify
```

The reactor’s `pom.xml` declares `maven-invoker-plugin` 3.9.0 configured to clone each `src/it/<case>/` into `target/it/<case>/` and run `mvn verify` there in a fresh Maven environment.

## [#when-to-add-a-new-it-case](#when-to-add-a-new-it-case)When to add a new IT case

The bar for new cases is low: add one whenever you fix a bug that "should have been caught earlier" by an automated check. Examples from recent cascades:

- The empty-staging gh-pages publish bug (#334) — would have been caught by an IT that asserts a single-module project’s `target/staging/` is correctly handled by the publish path.
- The classifier filename mismatch in the v148 supplement delivery — would have been caught by an `src/it/built-with-classifier/` case.
- The version-nested staging recursion (#337) — would have been caught by a `release-cascade` IT exercising `4` substitution in `site.deploy.url`.
- The parent-artifactId staging nesting (#342, surfaced during the v150 workspace release) — would have been caught by a `release-cascade` IT that verifies the published gh-pages tree has its content at the expected URL, not at `<parentArtifactId>/<artifactId>/`.

Each "we hit this in the cascade and had to release another fix" moment is exactly the kind of thing this suite exists to catch preemptively. File a bug + add a regression IT, in that order.

## [#see-also](#see-also)See also

- [ike-example-ws overview](index.html)[2] — workspace landing page.
- [workspace.yaml — Annotated Tour](workspace-yaml.html)[3] — the manifest format that the IT suite assumes the workspace exercises.
- [maven-invoker-plugin](https://maven.apache.org/plugins/maven-invoker-plugin/)[4] — upstream documentation for the harness.
