---
date_published: 2026-05-09
date_modified: 2026-05-09
canonical_url: https://github.com/IKE-Network/doc-example/dependency-info.html
---

# Maven Coordinates

## [Apache Maven](#apache-maven)

```
<dependency>
  <groupId>network.ike.examples</groupId>
  <artifactId>doc-example</artifactId>
  <version>13-SNAPSHOT</version>
  <type>pom</type>
</dependency>
```

## [Apache Ivy](#apache-ivy)

```
<dependency org="network.ike.examples" name="doc-example" rev="13-SNAPSHOT">
  <artifact name="doc-example" type="pom" />
</dependency>
```

## [Groovy Grape](#groovy-grape)

```
@Grapes(
@Grab(group='network.ike.examples', module='doc-example', version='13-SNAPSHOT')
)
```

## [Gradle/Grails](#gradle-grails)

```
implementation 'network.ike.examples:doc-example:13-SNAPSHOT'
```

## [Scala SBT](#scala-sbt)

```
libraryDependencies += "network.ike.examples" % "doc-example" % "13-SNAPSHOT"
```

## [Leiningen](#leiningen)

```
[network.ike.examples/doc-example "13-SNAPSHOT"]
```
