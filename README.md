# Sample repository for the reproduction of a Gradle bug involving composite builds and platform dependencies

## Description

Composite builds including projects with platform dependencies may fail with a circular task dependency error if the platform contains a constraint on the included project.

Relevant Gradle issue: https://github.com/gradle/gradle/issues/13979

## Steps to reproduce

* Clone this repository
* Perform a **composite build** by running:

```bash
./gradlew clean build --include-build library-project
```

* Observe the build failing with a circular dependency on task `:library-project:compileJava`

### Dependency graph

```
compileClasspath - Compile classpath for source set 'main'.
\--- com.example:library-project:1.0.0
     \--- com.example:platform-project:1.0.0
          \--- com.example:library-project:1.0.0 (c)

(c) - dependency constraint
```

### Dependency setup

```gradle
// Dependencies in project under test
dependencies {
   implementation 'com.example:library-project:1.0.0'
}

// Dependencies in 'library-project'
dependencies {
    api platform('com.example:platform-project:1.0.0')
}

// Dependencies in 'platform-project'
dependencies {
    constraints {
		api 'com.example:library-project:1.0.0'
    }
}
```


## Expected Behavior

Expecting build to complete with `BUILD SUCCESSFUL` the same way it does for...

* `./gradlew build` builds, that relying on using published dependencies from the same `./platform-project` and `./library-project` projects.
* `./gradlew build --include-build library-project --include-build platform-project` composite builds that also includes the platform dependency in the composite build.

## Current Behavior

Running `./gradlew build --include-build library-project` yields the following error:

```
FAILURE: Build failed with an exception.

* What went wrong:
Circular dependency between the following tasks:
:library-project:compileJava
\--- :library-project:compileJava (*)

(*) - details omitted (listed previously)
```

Excerpt of stack-trace:

```
at org.gradle.execution.plan.DefaultExecutionPlan.onOrderingCycle(DefaultExecutionPlan.java:464)
at org.gradle.execution.plan.DefaultExecutionPlan.determineExecutionPlan(DefaultExecutionPlan.java:294)
at org.gradle.execution.taskgraph.DefaultTaskExecutionGraph.ensurePopulated(DefaultTaskExecutionGraph.java:311)
at org.gradle.execution.taskgraph.DefaultTaskExecutionGraph.populate(DefaultTaskExecutionGraph.java:158)
at org.gradle.initialization.DefaultTaskExecutionPreparer.prepareForTaskExecution(DefaultTaskExecutionPreparer.java:41)
at org.gradle.initialization.BuildOperatingFiringTaskExecutionPreparer$CalculateTaskGraph.populateTaskGraph(BuildOperatingFiringTaskExecutionPreparer.java:117)
(...)
```

## Context

The error occurs when a composite build includes a project that inherits dependency constraints on itself from a platform dependency.

Performing a build relying only published artifacts of each individual project with no composite build involved works without an error.

Interestingly, including the platform project in the composite build will also resolves the error.

The error makes testing modules with such platform dependency setups hard to test using composite builds.

## Environment:

Build scan URL: https://scans.gradle.com/s/7r6x47jakglra

Versions:

```
------------------------------------------------------------
Gradle 6.5.1
------------------------------------------------------------

Build time:   2020-06-30 06:32:47 UTC
Revision:     66bc713f7169626a7f0134bf452abde51550ea0a

Kotlin:       1.3.72
Groovy:       2.5.11
Ant:          Apache Ant(TM) version 1.10.7 compiled on September 1 2019
JVM:          1.8.0_212 ( 25.212-b03)
OS:           Windows 10 10.0 amd64
```
