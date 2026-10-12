# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A multi-module Maven POM-only repository that serves as centralized dependency management for other Bliksoft Java projects. There is no application source code here — only POMs that declare compatible library versions for downstream projects to inherit via `<parent>` or `<dependencyManagement>`.

## Build Commands

```bash
mvn clean install       # Build and install to local repo
mvn clean verify        # Build with full verification
mvn deploy              # Deploy to Bliksoft private Maven repo
```

Releases go to `https://mvn.bliksoft.cz/private`, snapshots to `https://mvn.bliksoft.cz/private-snapshots`.

The root POM uses the Flatten Maven Plugin with a CI-friendly `${revision}` property. The resolved version is written to `.flattened-pom.xml` (git-ignored) at build time.

## Branch Strategy

Each branch targets a specific JDK version and produces its own artifact version:

| Branch | Artifact version |
|--------|-----------------|
| `java-8` | `1.8.x-SNAPSHOT` |
| `java-11` | `1.11.x-SNAPSHOT` |
| `java-17` | `1.17.x-SNAPSHOT` |
| `java-21` | `1.21.x-SNAPSHOT` |
| `java-25` | `1.25.x-SNAPSHOT` |

The `Y` of `1.XX.Y` is kept the same on all version branches (releases bump it everywhere), so an
application switches to another JDK line by changing only `XX` of the BOM version.

All changes must be propagated from `master` to the relevant version branches to ease merging. JDK-version-specific dependency version properties use a `-versionX` suffix (e.g., `-version8`, `-version11`).

## Module Overview

Sub-modules (`client`, `service`, `servicedef`, `jaxb`, `hibernate`, `log4j`, `slf4j`, `mysql`, `mariadb`, `ojdbc`, `httpclient`, `flyingsaucer`, `ooxml`, `freemarker`, `mail`, `junit`, `webcam`) are POM-only artifacts that group dependencies by concern. They contain no version declarations of their own — all versions are defined in the **root `pom.xml`**.

## Version Properties

All dependency versions live in the `<properties>` block of the root `pom.xml`. There are two kinds:

**Non-JDK-specific** — hardcoded inline in `<dependencyManagement>` (e.g. log4j, commons-*, httpclient5). Updated on `master` and merged to all version branches.

**JDK-specific** — declared as properties with a `-versionX` suffix (e.g. `-version8`, `-version11`). A value of `XX` means the dependency is not relevant for that branch (either unused or must be set in the appropriate branch). Updated directly on the target branch, never via master.

The suffix is the **minimum JDK of the version line** the property holds, so an older branch is never bumped to an incompatible line. When one artifact has several such lines (JavaFX: `org.openjfx-javafx-version11` = 17.0.x, `-version17` = 21.0.x, `-version23` = 25.x), `<dependencyManagement>` uses a selector property (`org.openjfx-javafx-version`) that each branch points at its line (`${org.openjfx-javafx-version17}`). New JDK-specific properties go inside the `do not reorder/reformat` block; the master → branch merge then conflicts on the neighbouring branch values - keep the branch values.

**`webcam` module** — `dependency-management-webcam` brings webcam-capture and OpenCV (bytedeco, with OpenBLAS and JavaCPP) including the natives of the build machine via OS-activated profiles (`-Djavacpp.platform=<platform>` selects another one; profiles of a dependency POM see system properties, not the consumer's POM properties). Per-platform packaging copies the classifier variants managed in the root POM.

## Making Changes

- **Non-JDK-specific version** (hardcoded in `<dependencyManagement>`): update on `master`, then merge `master` into all version branches.
- **JDK-specific version** (property with `-versionX` suffix): check out the target branch and update directly. Do not touch `XX` properties — they are intentional placeholders.
- Merges flow **master → version branches only**, never in reverse.
