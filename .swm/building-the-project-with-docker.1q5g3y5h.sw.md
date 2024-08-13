---
title: Building the Project with Docker
---
This document provides a detailed walkthrough of how Docker is used in the ai-dial-core-demo project, focusing on the Dockerfile configuration.

<SwmSnippet path="/Dockerfile" line="1">

---

## Dockerfile Configuration - Stage 1: Dependency Caching

The Dockerfile begins with a multi-stage build. In the first stage, we use the `gradle:8.2.0-jdk17-alpine` image as a base and set the working directory to `/home/gradle/src`. The Gradle user home is set to `/cache` to cache dependencies. The `build.gradle` and `settings.gradle` files are copied into the container, and the `gradle --no-daemon build --stacktrace` command is run to pull and cache dependencies.

```
FROM gradle:8.2.0-jdk17-alpine as cache

WORKDIR /home/gradle/src
ENV GRADLE_USER_HOME /cache
COPY build.gradle settings.gradle ./
# just pull dependencies for cache
RUN gradle --no-daemon build --stacktrace
```

---

</SwmSnippet>

<SwmSnippet path="/Dockerfile" line="9">

---

## Dockerfile Configuration - Stage 2: Building the Application

In the second stage, we again use the `gradle:8.2.0-jdk17-alpine` image as a base. We copy the cached dependencies from the first stage into the Gradle home directory. The application source code is copied into the container with the correct ownership. The `gradle --no-daemon build --stacktrace -PdisableCompression=true -x test` command is run to build the application, skipping tests. The built application is then extracted into a new `/build` directory.

```
FROM gradle:8.2.0-jdk17-alpine as builder

COPY --from=cache /cache /home/gradle/.gradle
COPY --chown=gradle:gradle . /home/gradle/src

WORKDIR /home/gradle/src
RUN gradle --no-daemon build --stacktrace -PdisableCompression=true -x test
RUN mkdir /build && tar -xf /home/gradle/src/build/distributions/aidial-core*.tar --strip-components=1 -C /build
```

---

</SwmSnippet>

<SwmSnippet path="/Dockerfile" line="18">

---

## Dockerfile Configuration - Stage 3: Final Image

In the final stage, we use the `eclipse-temurin:17-jdk-alpine` image as a base. Several commands are run to update packages and fix known vulnerabilities. The `su-exec` package is installed for later use. Several environment variables are set to disable OpenTelemetry exporters and configure local storage directories. A non-root user `appuser` is created. The built application from the second stage is copied into the container with the correct ownership. The `docker-entrypoint.sh` script is copied into the container and made executable. A healthcheck is configured to periodically check the application's health endpoint. Ports 8080 and 9464 are exposed. The log and storage directories are created with the correct ownership. Finally, the `docker-entrypoint.sh` script is set as the entrypoint for the container.

```
FROM eclipse-temurin:17-jdk-alpine

# fix CVE-2023-5363
# TODO remove the fix once a new version is released
RUN apk update && apk upgrade --no-cache libcrypto3 libssl3
# fix CVE-2023-52425
RUN apk upgrade --no-cache libexpat
RUN apk add --no-cache su-exec

ENV OTEL_TRACES_EXPORTER="none"
ENV OTEL_METRICS_EXPORTER="none"
ENV OTEL_LOGS_EXPORTER="none"

# Local storage dir configured in the default aidial.settings.json
ENV STORAGE_DIR /app/data
ENV LOG_DIR /app/log

WORKDIR /app

RUN adduser -u 1001 --disabled-password --gecos "" appuser

```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYWktZGlhbC1jb3JlLWRlbW8lM0ElM0FTd2ltbS1EZW1v" repo-name="ai-dial-core-demo" doc-type="general-build-tool"><sup>Powered by [Swimm](/)</sup></SwmMeta>
