# workspace-reactor-example — Project Notes

<!-- Migrated from CLAUDE.md by ws:init.
     This file is for hand-authored, project-specific information.
     Commit this file to git. -->

# IKE Workspace Reactor Example — Claude Standards

## Initial Setup — ALWAYS DO THIS FIRST

Run `mvn validate` before any other work. This unpacks the current
build standards into `.claude/standards/`. Do not proceed without
this step.

If `mvn validate` fails because `ike-build-standards` is not in the
local repository, fetch it from Nexus (default) or install it from
the `ike-tooling` workspace.

After validate completes, read and follow these files in `.claude/standards/`:

- MAVEN.md — Maven 4 build standards (always read)
- IKE-MAVEN.md — IKE-specific Maven conventions (always read)
- IKE-WORKSPACE.md — Workspace manifest and `ws:*` goals (always read)

## Project Overview

This is **IKE Workspace Reactor Example** (originally `ike-example-ws`,
then `workspace-example`) — a workspace aggregator for the IKE
example template set. It is not a component repo; it's a
coordination layer that orchestrates multi-repo releases and
end-to-end integration testing. The `-wsr` suffix marks it as an
aggregator that is itself activated as a Maven reactor; see the
`arch-workspace-and-workspace-reactor` topic in `ike-lab-documents`.

Clone this repo and run `mvn ws:scaffold-init` to pull down all the component
repos declared in `workspace.yaml`. After the initial clone, the
top-level reactor automatically includes every repo whose `pom.xml`
exists on disk (file-activated profiles).

### Component Layout

```
workspace-reactor-example/
├── workspace.yaml                  ← the manifest
├── pom.xml                         ← workspace aggregator
├── .mvn/extensions.xml             ← ike-workspace-extension registration
├── doc-example/                    ← cloned by ws:scaffold-init
├── project-example/                ← cloned by ws:scaffold-init
└── integration-tests-example/      ← integration test suite (optional)
```

### Release Cascade

```
ike-tooling → ike-docs → ike-platform → { doc-example, project-example, integration-tests-example } → workspace-reactor-example
```

This workspace covers the consumer-side portion; the foundation
(ike-tooling, ike-docs, ike-platform, ike-workspace-extension,
ike-base-parent) is intentionally NOT under workspace orchestration
— see `workspace.yaml` comments for the rationale.

## Key Workspace Commands

```bash
# Bootstrap: clone all component repos from their remotes
mvn ws:scaffold-init

# Status
mvn ws:overview

# Full reactor build of all cloned repos
mvn clean install

# Preview a coordinated release
mvn ws:release-draft

# Execute a coordinated release (pushes when ready)
mvn ws:release-publish -Dpush=true

# Align inter-component dependency versions after a version change
mvn ws:align-publish

# Start a cross-repo feature branch
mvn ws:feature-start-publish -Dfeature=my-feature
```

Never invoke raw `git` across these repos — always use the `ws:*`
goals so that all component repos stay coordinated.

## Integration Tests (`integration-tests-example/`)

The `integration-tests-example/` directory (when populated) holds
end-to-end smoke tests that verify the full cascade produces
consumable artifacts — e.g., "after releasing v1, a fresh
`mvn verify` on doc-example succeeds against Nexus." Run with:

```bash
mvn verify -pl integration-tests-example
```

## Plugin Versions

The workspace `pom.xml` inherits from `ike-parent`, which manages
the versions of `ike-maven-plugin`, `ike-workspace-maven-plugin`,
and `ike-doc-maven-plugin`. No literal `<version>` declarations are
needed in this POM's `<build><plugins>` (and none should be added —
overriding the inherited versions defeats the purpose of the parent).

`ike-doc-maven-plugin` is declared in `ike-parent`'s
`<pluginManagement>` with `<extensions>true</extensions>` at a
literal version (Maven resolves extension plugins before property
interpolation), so the parent itself drives that pin.

Use `ws:align-publish` to keep `ike-parent`'s version in sync with
whatever release of `ike-platform` this workspace tracks.

## `.mvn/jvm.config` constraints

Maven's `.mvn/jvm.config` is parsed as raw JVM arguments — one token
per line, NO comment syntax. A `#` at column 0 is passed to the JVM
as if it were a main-class name, and IntelliJ will show:

```
Error: Could not find or load main class #
Caused by: java.lang.ClassNotFoundException: #
```

Do NOT add `#`-prefixed comments to `.mvn/jvm.config`. The standard
workspace-root content is a single line:

- `--sun-misc-unsafe-memory-access=allow` — suppresses the JFFI
  `sun.misc.Unsafe` deprecation warnings emitted by
  JRuby/AsciidoctorJ on Java 24+.

Also do NOT set `-Denv.PATH` or PATH-related options here or in
`MAVEN_OPTS`: PATH entries containing spaces (e.g. JetBrains
Toolbox) cause the JVM launcher to bail with the same
"Could not find or load main class" error for an unrelated reason.

`ws:scaffold-init` seeds this file correctly and seeds the same
content in each cloned subproject (never overwriting an existing
file).
