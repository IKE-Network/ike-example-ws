# IKE Example Workspace — Claude Standards

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

This is **IKE Example Workspace** — a workspace aggregator for the
IKE five-repo split. It is not a component repo; it's a coordination
layer that orchestrates multi-repo releases and end-to-end integration
testing.

Clone this repo and run `mvn ws:init` to pull down all the component
repos declared in `workspace.yaml`. After the initial clone, the
top-level reactor automatically includes every repo whose `pom.xml`
exists on disk (file-activated profiles).

### Component Layout

```
ike-example-ws/
├── workspace.yaml              ← the manifest
├── pom.xml                     ← workspace aggregator
├── ike-docs/                   ← cloned by ws:init
├── ike-platform/               ← cloned by ws:init
├── doc-example/                ← cloned by ws:init
├── example-project/            ← cloned by ws:init
└── its/                        ← integration test suite (optional)
```

### Release Cascade

```
ike-tooling → [ike-docs → ike-platform → { doc-example, example-project }] → ike-example-ws
```

This workspace covers the bracketed portion. `ike-tooling` is
bootstrapped separately (see workspace.yaml comment for why).

## Key Workspace Commands

```bash
# Bootstrap: clone all component repos from their remotes
mvn ws:init

# Status
mvn ws:overview

# Full reactor build of all cloned repos
mvn clean install

# Preview a coordinated release
mvn ws:release -DdryRun=true

# Execute a coordinated release (pushes when ready)
mvn ws:release -Dpush=true

# Align inter-component dependency versions after a version change
mvn ws:align-publish

# Start a cross-repo feature branch
mvn ws:feature-start -Dfeature=my-feature
```

Never invoke raw `git` across these repos — always use the `ws:*`
goals so that all component repos stay coordinated.

## Integration Tests (`its/`)

The `its/` directory (when populated) holds end-to-end smoke tests
that verify the full cascade produces consumable artifacts — e.g.,
"after releasing v1, a fresh `mvn verify` on doc-example succeeds
against Nexus." Run with:

```bash
mvn verify -pl its
```

## Plugin Versions

The workspace `pom.xml` pins plugin versions at literals because
this POM doesn't inherit from ike-parent:

- `network.ike.tooling:ike-maven-plugin:125`
- `network.ike.platform:ike-workspace-maven-plugin:1`

Use `ws:align-publish` to keep these in sync with the subprojects.
