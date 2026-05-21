---
date_published: 2026-05-20
date_modified: 2026-05-20
canonical_url: https://ike.network/ike-platform/ike-parent/workspace-example/dependency-info.html
---

# Maven Coordinates

## [Apache Maven](#apache-maven)

```
<dependency>
  <groupId>network.ike.examples</groupId>
  <artifactId>workspace-example</artifactId>
  <version>23</version>
  <type>pom</type>
</dependency>
```

## [Apache Ivy](#apache-ivy)

```
<dependency org="network.ike.examples" name="workspace-example" rev="23">
  <artifact name="workspace-example" type="pom" />
</dependency>
```

## [Groovy Grape](#groovy-grape)

```
@Grapes(
@Grab(group='network.ike.examples', module='workspace-example', version='23')
)
```

## [Gradle/Grails](#gradle-grails)

```
implementation 'network.ike.examples:workspace-example:23'
```

## [Scala SBT](#scala-sbt)

```
libraryDependencies += "network.ike.examples" % "workspace-example" % "23"
```

## [Leiningen](#leiningen)

```
[network.ike.examples/workspace-example "23"]
```
