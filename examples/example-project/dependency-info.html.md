---
date_published: 2026-05-09
date_modified: 2026-05-09
canonical_url: https://github.com/IKE-Network/example-project/dependency-info.html
---

# Maven Coordinates

## [Apache Maven](#apache-maven)

```
<dependency>
  <groupId>network.ike.examples</groupId>
  <artifactId>example-project</artifactId>
  <version>13-SNAPSHOT</version>
</dependency>
```

## [Apache Ivy](#apache-ivy)

```
<dependency org="network.ike.examples" name="example-project" rev="13-SNAPSHOT">
  <artifact name="example-project" type="jar" />
</dependency>
```

## [Groovy Grape](#groovy-grape)

```
@Grapes(
@Grab(group='network.ike.examples', module='example-project', version='13-SNAPSHOT')
)
```

## [Gradle/Grails](#gradle-grails)

```
implementation 'network.ike.examples:example-project:13-SNAPSHOT'
```

## [Scala SBT](#scala-sbt)

```
libraryDependencies += "network.ike.examples" % "example-project" % "13-SNAPSHOT"
```

## [Leiningen](#leiningen)

```
[network.ike.examples/example-project "13-SNAPSHOT"]
```
