---
date_published: 2026-05-19
date_modified: 2026-05-19
canonical_url: https://ike.network/integration-tests-example/dependency-info.html
---

# Maven Coordinates

## [Apache Maven](#apache-maven)

```
<dependency>
  <groupId>network.ike.examples</groupId>
  <artifactId>integration-tests-example</artifactId>
  <version>24-SNAPSHOT</version>
  <type>pom</type>
</dependency>
```

## [Apache Ivy](#apache-ivy)

```
<dependency org="network.ike.examples" name="integration-tests-example" rev="24-SNAPSHOT">
  <artifact name="integration-tests-example" type="pom" />
</dependency>
```

## [Groovy Grape](#groovy-grape)

```
@Grapes(
@Grab(group='network.ike.examples', module='integration-tests-example', version='24-SNAPSHOT')
)
```

## [Gradle/Grails](#gradle-grails)

```
implementation 'network.ike.examples:integration-tests-example:24-SNAPSHOT'
```

## [Scala SBT](#scala-sbt)

```
libraryDependencies += "network.ike.examples" % "integration-tests-example" % "24-SNAPSHOT"
```

## [Leiningen](#leiningen)

```
[network.ike.examples/integration-tests-example "24-SNAPSHOT"]
```
