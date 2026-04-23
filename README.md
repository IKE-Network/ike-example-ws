# IKE Example Workspace

Workspace aggregator for the IKE five-repo split: `ike-docs`,
`ike-platform`, `doc-example`, `example-project`. Clone this repo
and run `mvn ws:init` to pull down all component repos and build
them as a single reactor.

## What this workspace is for

- **Local development** across the whole cascade in one editor / one
  command.
- **Coordinated releases** via `mvn ws:release`.
- **End-to-end integration tests** that exercise the actual external
  consumption of `ike-doc` packaging from Nexus.

## Bootstrap

```bash
git clone https://github.com/IKE-Network/ike-example-ws.git
cd ike-example-ws

mvn ws:init          # clone component repos per workspace.yaml
mvn ws:overview      # print the workspace dashboard
mvn clean install    # build the full cascade
```

## Component Repos

| Repo | Artifact | Purpose |
|---|---|---|
| [ike-docs](https://github.com/IKE-Network/ike-docs) | `network.ike.docs:*` | AsciiDoc plumbing, `ike-doc-maven-plugin` |
| [ike-platform](https://github.com/IKE-Network/ike-platform) | `network.ike.platform:*` | `ike-parent`, `ike-bom`, workspace plugin |
| [doc-example](https://github.com/IKE-Network/doc-example) | `network.ike.examples:doc-example` | Doc-only template (`ike-doc` packaging) |
| [example-project](https://github.com/IKE-Network/example-project) | `network.ike.examples:example-project` | Java + docs template |

See [`workspace.yaml`](workspace.yaml) for version pins and dependency
declarations.

## Release Cascade

```
ike-tooling → ike-docs → ike-platform → { doc-example, example-project } → [this workspace's ITs]
```

`ike-tooling` is bootstrapped out-of-band (it releases itself using
the plugin it produces). Everything else is orchestrated through
`mvn ws:release`.

## Integration Tests

See [`its/README.adoc`](its/README.adoc) for the IT suite layout
and how to add new end-to-end tests.

## History

Split from the archived `ike-pipeline` repo to resolve a Maven
extension-plugin reactor-load cycle. See
[`IKE-Network/ike-issues#216`](https://github.com/IKE-Network/ike-issues/issues/216).
