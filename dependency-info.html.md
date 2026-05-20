---
date_published: 2026-05-19
date_modified: 2026-05-19
canonical_url: https://ike.network/ike-platform/ike-parent/workspace-example/dependency-info.html
---

# Maven Coordinates

## [Apache Maven](#apache-maven)

```
<dependency>
  <groupId>network.ike.examples</groupId>
  <artifactId>workspace-example</artifactId>
  <version>23-SNAPSHOT</version>
  <type>pom</type>
</dependency>
```

## [Apache Ivy](#apache-ivy)

```
<dependency org="network.ike.examples" name="workspace-example" rev="23-SNAPSHOT">
  <artifact name="workspace-example" type="pom" />
</dependency>
```

## [Groovy Grape](#groovy-grape)

```
@Grapes(
@Grab(group='network.ike.examples', module='workspace-example', version='23-SNAPSHOT')
)
```

## [Gradle/Grails](#gradle-grails)

```
implementation 'network.ike.examples:workspace-example:23-SNAPSHOT'
```

## [Scala SBT](#scala-sbt)

```
libraryDependencies += "network.ike.examples" % "workspace-example" % "23-SNAPSHOT"
```

## [Leiningen](#leiningen)

```
[network.ike.examples/workspace-example "23-SNAPSHOT"]
```
