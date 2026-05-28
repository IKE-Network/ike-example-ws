---
date_published: 2026-05-27
date_modified: 2026-05-27
canonical_url: https://ike.network/integration-tests-example/dependency-info.html
---

# Maven Coordinates

## [Apache Maven](#apache-maven)

```
<dependency>
  <groupId>network.ike.examples</groupId>
  <artifactId>integration-tests-example</artifactId>
  <version>26-SNAPSHOT</version>
  <type>pom</type>
</dependency>
```

## [Apache Ivy](#apache-ivy)

```
<dependency org="network.ike.examples" name="integration-tests-example" rev="26-SNAPSHOT">
  <artifact name="integration-tests-example" type="pom" />
</dependency>
```

## [Groovy Grape](#groovy-grape)

```
@Grapes(
@Grab(group='network.ike.examples', module='integration-tests-example', version='26-SNAPSHOT')
)
```

## [Gradle/Grails](#gradle-grails)

```
implementation 'network.ike.examples:integration-tests-example:26-SNAPSHOT'
```

## [Scala SBT](#scala-sbt)

```
libraryDependencies += "network.ike.examples" % "integration-tests-example" % "26-SNAPSHOT"
```

## [Leiningen](#leiningen)

```
[network.ike.examples/integration-tests-example "26-SNAPSHOT"]
```
