---
date_published: 2026-05-27
date_modified: 2026-05-27
canonical_url: https://ike.network/project-example/dependency-info.html
---

# Maven Coordinates

## [Apache Maven](#apache-maven)

```
<dependency>
  <groupId>network.ike.examples</groupId>
  <artifactId>project-example</artifactId>
  <version>34-SNAPSHOT</version>
</dependency>
```

## [Apache Ivy](#apache-ivy)

```
<dependency org="network.ike.examples" name="project-example" rev="34-SNAPSHOT">
  <artifact name="project-example" type="jar" />
</dependency>
```

## [Groovy Grape](#groovy-grape)

```
@Grapes(
@Grab(group='network.ike.examples', module='project-example', version='34-SNAPSHOT')
)
```

## [Gradle/Grails](#gradle-grails)

```
implementation 'network.ike.examples:project-example:34-SNAPSHOT'
```

## [Scala SBT](#scala-sbt)

```
libraryDependencies += "network.ike.examples" % "project-example" % "34-SNAPSHOT"
```

## [Leiningen](#leiningen)

```
[network.ike.examples/project-example "34-SNAPSHOT"]
```
