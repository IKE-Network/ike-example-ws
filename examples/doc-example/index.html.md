---
date_published: 2026-05-09
date_modified: 2026-05-09
canonical_url: https://github.com/IKE-Network/doc-example/index.html
---

# doc-example

A standalone reference project demonstrating an IKE consumer whose **primary deliverable is a published document** rather than a JAR. Inherits `network.ike.platform:ike-parent` and exercises the full multi-renderer pipeline (HTML, Prawn PDF, FOP PDF, plus Prince/AH/ WeasyPrint/XEP toggleable on demand).

| Coordinate | Value |
| --- | --- |
| Group ID | `network.ike.examples` |
| Artifact | `doc-example` |
| Packaging | `pom` (post-`IKE-Network/ike-issues#321`) |
| Parent | `network.ike.platform:ike-parent` (v27) |
| Source classifier | `<classifier>adoc</classifier><type>zip</type>` |
| Render classifiers | `prince`, `fop`, `xep`, `pdf-default`, `html-single`, etc. |

## [#role-in-the-ike-ecosystem](#role-in-the-ike-ecosystem)Role in the IKE Ecosystem

This project is the canonical template for an **IKE deliverable that is itself a published document**. The companion project [example-project](https://ike.network/example-project/)[1] is the template for a **JAR + docs** deliverable. Together they cover the two common shapes of IKE consumer:

| Template | When to copy from it |
| --- | --- |
| `doc-example` | You’re shipping a published document as the primary deliverable — a guide, specification, manual, or reference. No Java compile path. Uses `<packaging>pom</packaging>` and ships the source as the `adoc` classifier. |
| [example-project](https://ike.network/example-project/)[1] | You’re shipping a JAR (a library, a CLI, a service) AND want rendered docs alongside it. |

For the workspace-aggregator template, see [ike-example-ws](https://ike.network/ike-example-ws/)[2].

## [#why-packagingpom-packaging-for-doc-only](#why-packagingpom-packaging-for-doc-only)Why `<packaging>pom</packaging>` for doc-only

The doc-pipeline activation in `ike-parent` is path-conditional on `<file><exists>src/docs/asciidoc</exists></file>`, **not** packaging- conditional. Any inheritor with that source directory gets:

- Asciidoctor rendering via `idoc:asciidoc` (HTML5, Prawn PDF, DocBook → XSL-FO chains).
- Multi-renderer PDF wrappers via `idoc:render-pdf` (Prince, Apache FOP, RenderX XEP, Antenna House, WeasyPrint).
- Source-payload attachment as `<classifier>adoc</classifier><type>zip</type>` by `maven-assembly-plugin`.
- Renderer-output attachments as additional classifiers (`prince`, `fop`, `xep`, `pdf-default`, `html-single`, …​).

For a doc-only module like `doc-example`, `<packaging>pom</packaging>` gives exactly what the project ships — no compile/test/jar phases, no empty primary jar, just the classifier attachments. This is the post-`IKE-Network/ike-issues#321` shape; earlier revisions used a custom `<packaging>ike-doc</packaging>` type with extension-realm machinery, which was retired in favor of the classifier-canonical shape. See the [ike-parent module page](https://ike.network/ike-platform/ike-parent/)[3] for the full design rationale.

## [#release-cascade-position](#release-cascade-position)Release Cascade Position

```
ike-tooling -> ike-docs -> ike-platform -> [doc-example, example-project] -> ike-example-ws
```

`doc-example` releases after `ike-platform` (whose `ike-parent` this project inherits) and after `ike-docs` (whose `ike-doc-maven-plugin` provides the `idoc:*` render goals declared at `14` in `ike-parent’s `<pluginManagement>`). The workspace-orchestrated release flow is `ws:release-publish` from `ike-example-ws`, which detects source-changed subprojects and releases them in topological order — see [ike-example-ws](https://ike.network/ike-example-ws/)[2].

## [#renderer-pipelines](#renderer-pipelines)Renderer Pipelines

`doc-example` exercises the full set of PDF renderers IKE supports. All start from the same AsciiDoc source under `src/docs/asciidoc/`.

| Renderer | Path | Activation |
| --- | --- | --- |
| HTML | `target/generated-docs/html/` | Default; always built. |
| Prawn PDF | `target/generated-docs/pdf-prawn/doc-example.pdf` | `-Dike.pdf.prawn`. Ruby-based, fast, sensible defaults. |
| FOP PDF | `target/generated-docs/pdf-fop/doc-example.pdf` | `-Dike.pdf.fop`. Java-based, XSL-FO via DocBook intermediate. |
| Prince PDF | `target/generated-docs/pdf-prince/doc-example.pdf` | `-Dike.pdf.prince`. Commercial; CSS Paged Media. |
| Antenna House PDF | `target/generated-docs/pdf-ah/doc-example.pdf` | `-Dike.pdf.ah`. Commercial; CSS Paged Media. |
| WeasyPrint PDF | `target/generated-docs/pdf-weasyprint/doc-example.pdf` | `-Dike.pdf.weasyprint`. Open source; Python-based. |
| RenderX XEP PDF | `target/generated-docs/pdf-xep/doc-example.pdf` | `-Dike.pdf.xep`. Free personal license; XSL-FO via DocBook. |

For a deeper tour of each pipeline, see the renderer documentation on [the IKE Docs site](https://ike.network/ike-docs/)[4].

## [#project-structure](#project-structure)Project Structure

```
doc-example/
├── pom.xml                              (1)
├── src/
│   ├── docs/asciidoc/                   (2)
│   │   ├── index.adoc                   (3)
│   │   └── chapters/
│   └── site/                            (4)
│       ├── asciidoc/index.adoc
│       ├── resources/css/site.css
│       └── site.xml
└── target/
    ├── doc-example-1-adoc.zip           (5)
    └── generated-docs/
        ├── html/                        (6)
        ├── pdf-prawn/
        └── pdf-fop/
```

Inherits `network.ike.platform:ike-parent` with `<relativePath/>` (per `IKE-Network/ike-issues#323`). Declares `<packaging>pom</packaging>`. The IKE doc-pipeline source. The path-conditional `doc-pipeline` profile in `ike-parent` activates on this directory’s existence. Master document. Includes are conventionally under `chapters/`. The Maven Site Plugin source — what generates this site you’re reading. Distinct from `src/docs/asciidoc/` (the deliverable). `adoc`-classified source ZIP, attached by `maven-assembly-plugin` and deployed to Nexus. Rendered outputs; renderer-specific classifiers attach additional ZIPs to the published artifact set.  

## [#build-commands](#build-commands)Build Commands

```
# HTML only (default):
mvn clean verify

# With Prawn PDF:
mvn clean verify -Dike.pdf.prawn

# With FOP PDF:
mvn clean verify -Dike.pdf.fop

# All free renderers at once:
mvn clean verify -Dike.pdf.prawn -Dike.pdf.fop -Dike.pdf.weasyprint

# Generate this site (Maven Site Plugin):
mvn site
```

## [#inheritance-pattern](#inheritance-pattern)Inheritance Pattern

```
<parent>
    <groupId>network.ike.platform</groupId>
    <artifactId>ike-parent</artifactId>
    <version>27</version>
    <relativePath/>             <!-- force repo lookup; see ike-issues#323 -->
</parent>

<groupId>network.ike.examples</groupId>
<artifactId>doc-example</artifactId>
<version>1-SNAPSHOT</version>
<packaging>pom</packaging>
```

After inheriting, the project gets — for free — Java toolchain defaults (harmlessly inherited; no compile happens for `pom`-packaged modules), GPG signing via Bouncy Castle, the AsciiDoc documentation pipeline, and dependency-version management for the IKE ecosystem.

## [#what-to-copy-when-starting-a-new-project](#what-to-copy-when-starting-a-new-project)What to Copy When Starting a New Project

When creating a new IKE document project, copy the following:

| File / Directory | Purpose |
| --- | --- |
| `pom.xml` | Parent declaration (with `<relativePath/>`), `<packaging>pom</packaging>`, group/artifact coordinates. Adjust group ID and artifact name only. |
| `src/docs/asciidoc/` | Documentation source. Edit `index.adoc` and add `chapters/` includes as needed. This is the deliverable. |
| `src/site/` | Maven Site Plugin source. Update `site.xml` (project name, repo URL) and `asciidoc/index.adoc` for your project. The Forest-theme `site.css` and `ike-logo.svg` come from `ike-build-standards` — inherited automatically; only override locally if you need per-project branding. |

## [#resources](#resources)Resources

| Resource | URL |
| --- | --- |
| Source repository | [https://github.com/IKE-Network/doc-example](https://github.com/IKE-Network/doc-example)[5] |
| Cross-project issue tracker | [https://github.com/IKE-Network/ike-issues](https://github.com/IKE-Network/ike-issues)[6] |
| IKE Network landing page | [https://ike.network/](https://ike.network/)[7] |
| IKE Docs (renderer pipelines, `idoc:*` plugin) | [https://ike.network/ike-docs/](https://ike.network/ike-docs/)[4] |
| IKE Platform (parent POM, BOM, workspace plugin) | [https://ike.network/ike-platform/](https://ike.network/ike-platform/)[8] |
| Sibling: code+docs template | [https://ike.network/example-project/](https://ike.network/example-project/)[1] |
| Sibling: workspace-aggregator template | [https://ike.network/ike-example-ws/](https://ike.network/ike-example-ws/)[2] |
