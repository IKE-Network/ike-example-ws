---
date_published: 2026-05-10
date_modified: 2026-05-10
canonical_url: https://github.com/IKE-Network/example-project/index.html
---

# example-project

A standalone reference project demonstrating an IKE consumer that ships **both compiled Java code and rendered AsciiDoc documentation** through a single `mvn verify` cycle. Inherits `ike-parent` from `ike-platform` and exercises the doc pipeline contributed by `ike-docs`.

| Coordinate | Value |
| --- | --- |
| Group ID | `network.ike.examples` |
| Artifact | `example-project` |
| Packaging | `jar` (plus AsciiDoc doc-pipeline activated by `src/docs/asciidoc`) |
| Parent | `network.ike.platform:ike-parent` |
| Java version | 25 |

## [#role-in-the-ike-ecosystem](#role-in-the-ike-ecosystem)Role in the IKE Ecosystem

This project is the canonical template for an **IKE library that ships both code and docs**. The companion project [doc-example](https://ike.network/doc-example/)[1] is the template for a **doc-only** deliverable. Together they cover the two common shapes of IKE consumer:

| Template | When to copy from it |
| --- | --- |
| `example-project` | You’re shipping a JAR (a library, a CLI, a service) AND want rendered docs alongside it. Builds run a Java compile + test cycle plus the AsciiDoc → HTML/PDF pipeline. |
| [doc-example](https://ike.network/doc-example/)[1] | You’re shipping a published document as the primary deliverable. No Java compile path; uses `<packaging>pom</packaging>` and ships the source as the `adoc` classifier (post-`IKE-Network/ike-issues#321`). |

For the workspace-aggregator template (one repo orchestrating multiple consumers via `ws:*` goals), see [ike-example-ws](https://ike.network/ike-example-ws/)[2].

## [#release-cascade-position](#release-cascade-position)Release Cascade Position

```
ike-tooling -> ike-docs -> ike-platform -> [example-project, doc-example] -> ike-example-ws
```

`example-project` releases after `ike-platform` (whose `ike-parent` this project consumes) and after `ike-docs` (whose `ike-doc-maven-plugin` is declared as a regular managed plugin in `ike-parent` via `16`; the documentation pipeline activates through a profile when `src/docs/asciidoc/` exists). Earlier revisions used the `<extensions>true</extensions>` mechanism to register a custom `<packaging>ike-doc</packaging>` type at literal versions; that machinery was retired in [ike-issues#321](https://github.com/IKE-Network/ike-issues/issues/321)[3] in favor of a classifier-canonical doc shape (`<classifier>adoc</classifier><type>zip</type>`).

## [#project-structure](#project-structure)Project Structure

```
example-project/
├── pom.xml                              (1)
├── src/
│   ├── main/java/                       (2)
│   ├── test/java/                       (3)
│   ├── docs/asciidoc/                   (4)
│   │   ├── index.adoc                   (5)
│   │   └── chapters/
│   └── site/                            (6)
│       ├── asciidoc/index.adoc
│       ├── resources/css/site.css
│       └── site.xml
└── target/
    ├── example-project-1-SNAPSHOT.jar   (7)
    └── generated-docs/
        ├── html/                        (8)
        ├── pdf-prawn/                   (9)
        └── pdf-fop/                     (10)
```

Inherits `ike-parent` — no Java/build config needed locally. Java sources. Compiled with Java 25 via the parent’s `maven-compiler-plugin` configuration. JUnit 5 tests. Run by `maven-surefire-plugin` from the parent. The IKE doc pipeline source. Activated by the `doc-pipeline` profile in `ike-parent` — file-activated by the presence of this directory. Master document. Includes are conventionally under `chapters/`. The Maven Site Plugin source — what generates this site you’re reading. Compiled artifact. Rendered HTML documentation. Prawn PDF (when `-Dike.pdf.prawn` is passed). FOP PDF (when `-Dike.pdf.fop` is passed).  

## [#build-commands](#build-commands)Build Commands

```
# Compile + test + HTML docs (default):
mvn clean verify

# With Prawn PDF:
mvn clean verify -Dike.pdf.prawn

# With FOP PDF:
mvn clean verify -Dike.pdf.fop

# All renderers:
mvn clean verify -Dike.pdf.prawn -Dike.pdf.fop

# Generate this site (Maven Site Plugin):
mvn site
```

## [#inheritance-pattern](#inheritance-pattern)Inheritance Pattern

`example-project` inherits `ike-parent` directly:

```
<parent>
    <groupId>network.ike.platform</groupId>
    <artifactId>ike-parent</artifactId>
    <version>6</version>
</parent>
```

After inheriting, the project gets — for free — Java 25 compilation, GPG signing via Bouncy Castle, JaCoCo coverage, the AsciiDoc documentation pipeline, and dependency-version management for the IKE ecosystem. `ike-maven-plugin` and `ike-doc-maven-plugin` are declared as ordinary managed plugins (no `extensions=true`, post-#321) and resolve via property indirection (`154`, `16`).

The doc pipeline activates automatically when `src/docs/asciidoc/` exists. No `<plugins>` block in this project’s POM is needed for documentation — the parent profile handles it.

## [#what-to-copy-when-starting-a-new-project](#what-to-copy-when-starting-a-new-project)What to Copy When Starting a New Project

When creating a new IKE library, copy the following from this template:

| File / Directory | Purpose |
| --- | --- |
| `pom.xml` | Parent declaration, group/artifact coordinates, single-segment version strategy. Adjust group ID and artifact name only. |
| `src/main/java/` | Java sources. Replace with your own. |
| `src/test/java/` | JUnit 5 test sources. Replace with your own. |
| `src/docs/asciidoc/` | AsciiDoc documentation source. Edit `index.adoc` and add `chapters/` includes as needed. |
| `src/site/` | Maven Site Plugin source. Update `site.xml` (project name, repo URL, breadcrumb path) and `asciidoc/index.adoc` for your project. Keep the Forest-theme `site.css` and `ike-logo.svg` for visual consistency with the rest of the IKE ecosystem. |

## [#resources](#resources)Resources

| Resource | URL |
| --- | --- |
| Source repository | [https://github.com/IKE-Network/example-project](https://github.com/IKE-Network/example-project)[4] |
| Cross-project issue tracker | [https://github.com/IKE-Network/ike-issues](https://github.com/IKE-Network/ike-issues)[5] |
| IKE Network landing page | [https://ike.network/](https://ike.network/)[6] |
| IKE Platform (parent POM, BOM, workspace plugin) | [https://ike.network/ike-platform/](https://ike.network/ike-platform/)[7] |
| Sibling: doc-only template | [https://ike.network/doc-example/](https://ike.network/doc-example/)[1] |
| Sibling: workspace-aggregator template | [https://ike.network/ike-example-ws/](https://ike.network/ike-example-ws/)[2] |
