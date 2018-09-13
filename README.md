# Dependency Management with Gradle

What we are trying to solve are the following scenarios:
```
                                  +----+
                                  |    |
                                  | C  |
                            +---> |    |
                            |     +----+
                            |
           +----+      +----+
           |    |      |    |
           | A  +----> | B  |
           |    |      |    |     +----+
           +----+      +----+     |    |
                            |     |    |
                            |     | D  |
                            +---> |    |
                                  +----+

     Scenarios:

     - B, C and D pulled in to A's workspace ( transitive: true)
     - B pulled into A's workspace (transitive: false)
     - B, C and D pulled into A's workspace, but C is in a different version ( transitive: true,  and exclude )

```

## Setup

Each of the folders `A/`, `B/`, `C/` and `D/` represent one of the boxes or _components_ in the diagram above.

The dependency graph is such that `A` depends on `B` which depends on `C` and `D`.

This POC works entirely on the local machine, using only the local maven repository.

### Publications

First we need to publish our lowest level dependencies

#### D

`cd D && ./gradlew publishToMavenLocal && cd ..`

This will publish `D` in version `0.0.1-SNAPSHOT`.

#### C
_0.0.1-SNAPSHOT_:

`cd C && ./gradlew publishToMavenLocal && cd ..`

This will publish `C` in version `0.0.1-SNAPSHOT`.

_0.0.2-SNAPSHOT_:

Now change the contents of `C/src/C.c` to `2`, and the version in `C/gradle.properties` to `0.0.2`

`cd C && ./gradlew publishToMavenLocal && cd ..`

This will publish `C` in version `0.0.2-SNAPSHOT`

#### B

`cd B && ./gradlew publishToMavenLocal && cd ..`


## Scenarios

The different scenarios from the diagram is covered here with instructions on how to replicate them.

### B, C and D pulled into A's workspace

Make sure that `A/build.gradle` applies the file `transitivedeps.gradle`.

Running the command `./gradlew clean getDeps` in `A/` should establish `B`, `C` and `D` in the folder `A/build/dependencies`.

This can be seen from the dependency declaration in `A/transitivedeps.gradle`.

### B pulled into A's workspace

Make sure that `A/build.gradle` applies the file `deps.gradle`.

Running the command `./gradlew clean unzipDeps` in `A/` should establish `B`, in the folder `A/build/dependencies`.

This can be seen from the dependency declaration in `A/deps.gradle`.

### B, C and D pulled into A's workspace with a different version of C

Make sure that `A/build.gradle` applies the file `excludedeps.gradle`.

Running the command `./gradlew clean unzipDeps` in `A/` should establish `B`, `C` and `D` in the folder `A/build/dependencies`.

Look into the dependency in `A/build/dependencies/C` and notice that it is different from the version pulled in the first scenario.

This should match the output of the publication made in the setup of `C-0.0.2-SNAPSHOT`.

This can be seen from the dependency declaration in `A/excludedeps.gradle`.
