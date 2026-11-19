# [FONTOS Backend] Developer Guide

This guide is for backend developers who want to use the `tsb.fontos.*` framework modules (Fontos-Framework-Backend) to build Spring Boot backend services for port/terminal operating systems (TOS), following the conventions used across the `Fontos-Sample-Backend` reference applications.

It is the backend companion to the `[tsb-fontos-ui] Developer Guide` (frontend/React).

> **Scope note**: This guide was assembled from the current source tree of `Fontos-Framework-Backend` / `Fontos-Sample-Backend` and from the internal how-to decks (`적용방법`) listed in the Appendix. Some slide screenshots were partially obscured or cut off at capture time — those spots are marked `(source unclear — verify against the repository)`. Treat this document as a starting point and diff it against the actual source before publishing internally.

## Table of Contents

1. [Introduction](#1-introduction)
2. [Development Environment Setup](#2-development-environment-setup)
   - 2.1 [Toolchain Installation](#21-toolchain-installation)
   - 2.2 [Git / Azure DevOps Workflow](#22-git--azure-devops-workflow)
   - 2.3 [Importing and Running a Project in STS](#23-importing-and-running-a-project-in-sts)
   - 2.4 [TFS vs Git, for .NET Developers](#24-tfs-vs-git-for-net-developers)
3. [Getting Started](#3-getting-started)
   - 3.1 [Prerequisites](#31-prerequisites)
   - 3.2 [Multi-module Gradle Setup](#32-multi-module-gradle-setup)
   - 3.3 [Reference Project Skeleton](#33-reference-project-skeleton)
   - 3.4 [Core Configuration Files](#34-core-configuration-files)
4. [CATOS+ Project Architecture & Conventions](#4-catos-project-architecture--conventions)
   - 4.1 [Layered Architecture](#41-layered-architecture)
   - 4.2 [Terminology: FONTOS.NET vs FONTOS.J2EE](#42-terminology-fontosnet-vs-fontosj2ee)
   - 4.3 [Naming Rules & Conventions](#43-naming-rules--conventions)
   - 4.4 [SQL Query Conventions](#44-sql-query-conventions)
   - 4.5 [Dto (Data Transfer Object) Projects](#45-dto-data-transfer-object-projects)
   - 4.6 [Biz (Business) Projects](#46-biz-business-projects)
   - 4.7 [Ct (Client) Projects](#47-ct-client-projects)
   - 4.8 [Ex (External Interface) Projects](#48-ex-external-interface-projects)
   - 4.9 [Java Coding Style Guide](#49-java-coding-style-guide)
5. [REST API Foundations](#5-rest-api-foundations)
   - 5.1 [Base Controller / Service Classes](#51-base-controller--service-classes)
   - 5.2 [Standard Response Wrappers](#52-standard-response-wrappers)
   - 5.3 [Exception Handling](#53-exception-handling)
   - 5.4 [Pagination](#54-pagination)
   - 5.5 [API Documentation (SpringDoc / OpenAPI / Swagger)](#55-api-documentation-springdoc--openapi--swagger)
6. [Security](#6-security)
   - 6.1 [JWT-based Authentication & Authorization](#61-jwt-based-authentication--authorization)
   - 6.2 [SQL Injection Validator](#62-sql-injection-validator)
   - 6.3 [Password Policy Validator](#63-password-policy-validator)
   - 6.4 [CATOS API URI / Access-Control Convention](#64-catos-api-uri--access-control-convention)
   - 6.5 [Multi-Factor Authentication (TOTP/OTP)](#65-multi-factor-authentication-totpotp)
   - 6.6 [SSO & Token Lifecycle Management](#66-sso--token-lifecycle-management)
7. [WebSocket / STOMP Messaging](#7-websocket--stomp-messaging)
   - 7.1 [Concepts](#71-concepts)
   - 7.2 [Server Configuration](#72-server-configuration)
   - 7.3 [Securing WebSocket/STOMP Connections with JWT](#73-securing-websocketstomp-connections-with-jwt)
   - 7.4 [Controllers and Message Mapping](#74-controllers-and-message-mapping)
   - 7.5 [Pub/Sub Patterns](#75-pubsub-patterns)
8. [TCP / Socket Messaging](#8-tcp--socket-messaging)
   - 8.1 [Netty-based TCP Client (`tsb.fontos.net`)](#81-netty-based-tcp-client-tsbfontosnet)
   - 8.2 [Wiring a Client End to End](#82-wiring-a-client-end-to-end)
   - 8.3 [Calling the TCP Client from a REST Controller](#83-calling-the-tcp-client-from-a-rest-controller)
   - 8.4 [`tsb.fontos.cnet` — Annotation-driven TCP Server/Client](#84-tsbfontoscnet--annotation-driven-tcp-serverclient)
9. [SOAP Web Services](#9-soap-web-services)
   - 9.1 [Project Layout & Versions](#91-project-layout--versions)
   - 9.2 [Generating Java from WSDL](#92-generating-java-from-wsdl)
   - 9.3 [Implementing the Service](#93-implementing-the-service)
   - 9.4 [Publishing the CXF Endpoint](#94-publishing-the-cxf-endpoint)
   - 9.5 [SOAP Client](#95-soap-client)
   - 9.6 [Composing Multiple SOAP Modules into One WAR](#96-composing-multiple-soap-modules-into-one-war)
   - 9.7 [SOAP Client — Advanced Configuration (WSDL Codegen, mTLS, SoapAction)](#97-soap-client--advanced-configuration-wsdl-codegen-mtls-soapaction)
10. [Report Generation](#10-report-generation)
    - 10.1 [JasperReports](#101-jasperreports)
    - 10.2 [OpenHTMLtoPDF](#102-openhtmltopdf)
11. [Excel Export](#11-excel-export)
    - 11.1 [Backend Setup](#111-backend-setup)
    - 11.2 [Frontend Integration](#112-frontend-integration)
    - 11.3 [Per-cell Coloring](#113-per-cell-coloring)
12. [Frontend Log Forwarding](#12-frontend-log-forwarding)
    - 12.1 [Backend Setup](#121-backend-setup)
    - 12.2 [Frontend Integration](#122-frontend-integration)
13. [Scheduling & Task Monitoring](#13-scheduling--task-monitoring)
    - 13.1 [Simple `@Scheduled` Jobs](#131-simple-scheduled-jobs)
    - 13.2 [Quartz-based Scheduling](#132-quartz-based-scheduling)
    - 13.3 [Task Monitoring (Micrometer)](#133-task-monitoring-micrometer)
    - 13.4 [Manual Scheduling (`TSchedulingConfigurer` / `TManualScheduler`)](#134-manual-scheduling-tschedulingconfigurer--tmanualscheduler)
14. [Build & Deployment](#14-build--deployment)
    - 14.1 [Per-site Build Configuration](#141-per-site-build-configuration)
    - 14.2 [WAR Packaging](#142-war-packaging)
    - 14.3 [WildFly / JBoss WAR Deployment & JNDI DataSource](#143-wildfly--jboss-war-deployment--jndi-datasource)
    - 14.4 [External Configuration File Resolution](#144-external-configuration-file-resolution)
    - 14.5 [Legacy Gradle Profile-based Resource Folders](#145-legacy-gradle-profile-based-resource-folders)
    - 14.6 [SSL/TLS Configuration](#146-ssltls-configuration)
    - 14.7 [Bundling the FONTOS Frontend Build into the Backend WAR](#147-bundling-the-fontos-frontend-build-into-the-backend-war)
15. [Data Synchronization (Observer/Callback Pattern)](#15-data-synchronization-observercallback-pattern)
16. [Database & Datasource Configuration](#16-database--datasource-configuration)
    - 16.1 [Multi-Dialect Mapper Switching](#161-multi-dialect-mapper-switching)
    - 16.2 [Multi-Datasource Routing](#162-multi-datasource-routing)
17. [Web Serial (COM Port) Communication (Frontend Reference)](#17-web-serial-com-port-communication-frontend-reference)
    - 17.1 [Overview](#171-overview)
    - 17.2 [Test Environment Setup](#172-test-environment-setup)
    - 17.3 [FONTOS React Integration](#173-fontos-react-integration)
18. [Migrating to JDK 17 / Spring Boot 3.x (V2)](#18-migrating-to-jdk-17--spring-boot-3x-v2)
    - 18.1 [Migration Baseline](#181-migration-baseline)
    - 18.2 [Dev Environment & Build Setup](#182-dev-environment--build-setup)
    - 18.3 [Library Version Matrix](#183-library-version-matrix)
    - 18.4 [Jakarta EE9 Namespace Rename](#184-jakarta-ee9-namespace-rename)
    - 18.5 [Spring Security 5→6 DSL Migration](#185-spring-security-56-dsl-migration)
    - 18.6 [Reporting Module Compatibility](#186-reporting-module-compatibility)
19. [Real-World Business Component Walkthrough (Controller, Service, Dao, Mapper)](#19-real-world-business-component-walkthrough-controller-service-dao-mapper)
    - 19.1 [Two Deployment Patterns Seen in Production](#191-two-deployment-patterns-seen-in-production)
    - 19.2 [Full-Stack Example: the `SingleGrid` Feature (Single-Module Pattern)](#192-full-stack-example-the-singlegrid-feature-single-module-pattern)
    - 19.3 [Ct-Project Controllers Confirmed in Production: `catos.ct.ip`](#193-ct-project-controllers-confirmed-in-production-catosctip)
20. [Best Practices & Security Notes](#20-best-practices--security-notes)
21. [Appendix](#21-appendix)
    - 21.1 [Module / Dependency Reference](#211-module--dependency-reference)
    - 21.2 [Source Locations](#212-source-locations)

---

## 1. Introduction

`tsb.fontos.*` is the backend framework family that FONTOS-based terminal/port applications are built on. It provides Spring Boot building blocks for REST APIs, JWT authentication/authorization, input validation, WebSocket/STOMP messaging, raw TCP socket messaging, SOAP web services, report generation (PDF/Excel), and scheduling — so that individual product teams don't re-implement this plumbing per project.

### Key Features

- **REST foundation**: base controller/service/DAO classes, standard success/error response envelopes, centralized exception handling (`tsb.fontos.core`)
- **Security**: stateless JWT authentication with automatic refresh, URI-pattern based authorization, SQL Injection input validation, configurable password policy (`tsb.fontos.core.security`, `tsb.fontos.security`)
- **Realtime messaging**: STOMP-over-WebSocket wrapper with pluggable handshake/channel security hooks (`tsb.fontos.net.ws`)
- **TCP/Socket integration**: a Netty-based async/sync TCP message client for talking to legacy terminal message servers (C3IT/XIS-style), plus an annotation-driven "Spring MVC over TCP" framework (`tsb.fontos.net`, `tsb.fontos.cnet`)
- **SOAP interoperability**: Apache CXF-based SOAP server/client scaffolding with WSDL-to-Java code generation (used in `Fontos-Sample-Backend`)
- **Reporting**: JasperReports and HTML-to-PDF (`openhtmltopdf`) report export, barcode/QR code generation
- **Excel export**: server-driven grid-to-Excel export with per-cell color rules
- **Scheduling & monitoring**: `@Scheduled`/Quartz job templates with Micrometer-based task metrics

### Architecture

The framework is split into independently versioned Gradle modules living in the **`Fontos-Framework-Backend`** repository. Application/service teams consume them as project dependencies from their own repository (e.g. **`Fontos-Sample-Backend`**, which also doubles as the "how do I use this" reference implementation referenced throughout this guide).

```
Fontos-Framework-Backend/
├── tsb.fontos.core/                  # common utilities, base REST/DAO/service classes,
│   │                                  # exception handling, JWT/password/SQL-injection primitives
│   └── src/main/java/tsb/fontos/core/
│       ├── security/                  # JwtSecurityProperties, PasswordEncoder, TPasswordPolicy*, sql/*
│       ├── web/, service/, dao/, http/rest/, exception/
│       └── ...
├── tsb.fontos.security/               # JWT filters/managers, URI-pattern authorization, OTP
│   └── src/main/java/tsb/fontos/security/
│       ├── jwt/ (+mng/), filter/, authentication/, access/, web/configuration/
├── tsb.fontos.net/                    # Netty TCP client + WebSocket/STOMP wrapper
│   └── src/main/java/tsb/fontos/net/
│       ├── msg/                       # TCP client (SimpleNetMessageClient, NettyTcpClient, ...)
│       └── ws/                        # STOMP config/security/controller base classes
├── tsb.fontos.cnet/                   # "Spring MVC"-style annotation routing over raw TCP sockets
├── tsb.fontos.report.jasper/          # JasperReports PDF export
├── tsb.fontos.report.openhtmltopdf/   # Thymeleaf + Jsoup + OpenHTMLtoPDF HTML→PDF export, QR code
└── tsb.fontos.core.micrometer/        # scheduler/task Prometheus metrics via Micrometer
```

Consuming applications (see `Fontos-Sample-Backend/tsb.fontos.sample`) reference these modules either as published artifacts or, in local development, as sibling Gradle projects wired through `settings.gradle` (see [3.2](#32-multi-module-gradle-setup)).

Common versions across the framework (as of this writing): **Java 8**, **Spring Boot 2.6.7**, Gradle with `io.spring.dependency-management 1.0.11.RELEASE`.

---

## 2. Development Environment Setup

This section covers setting up a Windows development machine from scratch for CATOS+/FONTOS-J2EE backend work — tool installation, Azure DevOps-hosted Git access, and importing/running a project in Spring Tool Suite (STS). If your machine is already set up, skip to [3. Getting Started](#3-getting-started).

### 2.1 Toolchain Installation

All installers are pulled from the shared tools drop (`FTP: /1.TSB_PROGRAM/[TOOL]DEVELOP_FOR_JAVA/`):

| Tool | Version | Notes |
|---|---|---|
| Java | JDK 1.8 (8u202) | Last JDK 8 update available without an Oracle commercial license — install this exact build, not a newer 8u release |
| Spring Tool Suite | 4.13.0.RELEASE (Eclipse 4.22.0) | The team's standard IDE for backend work |
| Lombok | 1.18.22 | Must be installed *into* the STS install, not just added as a dependency |
| Git | 2.35.1.2 | Client used for Azure DevOps repositories |

Install order and steps:

1. **JDK 8u202** — download `jdk-8u202-windows-x64.zip` from Oracle's Java SE 8 archive and install.
2. **Spring Tool Suite** — extract `spring-tool-suite-4-4.13.0.RELEASE-e4.22.0-win32.win32.x86_64.self-extracting.zip`, then from a command prompt in that folder run:

   ```
   java -jar spring-tool-suite-4-4.13.0.RELEASE-e4.22.0-win32.win32.x86_64.self-extracting.jar
   ```

3. **Lombok** — extract `lombok-1.18.22.zip`, then from a command prompt in that folder run:

   ```
   java -jar lombok-1.18.22.jar
   ```

   The Lombok installer's UI needs to be pointed at your STS install: choose **Specific location...** and select `SpringToolSuite4.exe` under the STS installation path so Lombok patches that specific Eclipse/STS instance (without this step, `@Getter`/`@Setter`/`@Slf4j`/etc. annotations used throughout the framework won't be recognized by the IDE, even though the project still compiles from the command line).
4. **Git** — extract and run the `Git-2.35.1.2-64-bit` installer.

### 2.2 Git / Azure DevOps Workflow

Source is hosted on Azure DevOps at `https://tds.tsb.co.kr/Product-JAVA`. Git access requires a Personal Access Token (PAT), not your regular login password.

1. **Generate a PAT**: log in at the URL above → top-right profile icon → **Security** → **Personal Access Tokens** → **New Token**. Scope: all/full access; expiration: the longest option available. Copy the generated token immediately and store it somewhere safe (e.g. a password manager) — Azure DevOps will not show it again.
2. **Create a workspace root folder** to hold all repositories, e.g. `D:\WorkSpace\Product-JAVA`, and point STS's workspace at that folder.
3. **Enable the Git views**: `Window > Show View > Other...` → add **Git Repositories** (shows all local repos registered in the workspace — conceptually similar to TFS Source Control) and **Git Staging** (shows pending changes for the currently-selected project — similar to TFS Pending Changes).
4. **Create the local repository folder structure** as `{Project Name}\{Repository Name}`, matching Azure DevOps's own project/repo hierarchy, e.g.:

   ```
   Tsb-Fontos-Framework\Fontos-Framework-Backend
   Tsb-Catos-Cm\Catos-Cm-Backend
   Tsb-Catos-Plus\Catos-Plus-Backend
   ```

5. **Clone each repository**: `Git Repositories` view → **Clone a Git Repository** → paste the repo's URI → under **Authentication**, set **User** to your work email and **Password** to the PAT from step 1 (check **Store in Secure Store** so STS remembers it) → select the **master** branch (CATOS's source-control policy uses a single `master` branch only — no long-lived feature branches) → set the destination directory to the matching folder created in step 4.
6. Repeat for every repository the project needs (e.g. `Tsb-Catos-Cm/Catos-Cm-Backend`, `Tsb-Catos-Plus/Catos-Plus-Backend`, plus `Tsb-Fontos-Framework/Fontos-Framework-Backend` from [1. Introduction](#1-introduction)). Successfully cloned repos appear listed under the Git Repositories view.
7. **Keep sources current**: select one or more repositories in the Git Repositories view → right-click → **Pull**.

### 2.3 Importing and Running a Project in STS

1. **Import (Gradle)**: `File > Import... > Gradle > Existing Gradle Project`, then point it at one project folder within a cloned repository (e.g. `catos.biz.common` under `Catos-Plus-Backend`).
2. Because FONTOS-J2EE projects are Gradle multi-module builds, each project's `settings.gradle` declares the sibling projects it depends on (see [3.2](#32-multi-module-gradle-setup)) — importing `catos.biz.common` therefore automatically pulls in every project it depends on as additional Eclipse projects in the same import step.
3. **Set the per-project JDK**: right-click the imported project (e.g. `tsb.fontos.core`) → **Properties** → **Java Build Path** → **Libraries** → select the **JRE System Library** entry → set it to **jdk 1.8.0_202**.
4. **Set the Gradle JVM globally**: `Window > Preferences > Gradle` → set **Java Home** to the `jdk 1.8.0_202` install directory.
5. **Refresh Gradle Project**: right-click the project → **Gradle > Refresh Gradle Project**. A project freshly cloned from Git only has Gradle's dependency *declarations* — the actual library JARs aren't downloaded yet, so this step is required to pull them from the remote Maven/Gradle repositories into your local Gradle cache.
6. **Run**:
   - **Build/Run**: Gradle Tasks view → `{Project} > application > bootRun`.
   - **Debug**: right-click the project → **Debug As > Spring Boot App**.

### 2.4 TFS vs Git, for .NET Developers

A quick orientation for developers coming from FONTOS.NET's TFS-based workflow:

| Item | TFS (TFVC) | Git |
|---|---|---|
| Version control model | Centralized (CVCS) | Distributed (DVCS) |
| Local commits | Not possible | Possible |
| Branching | Relatively heavyweight/slow | Lightweight/fast |
| Network dependency | Must sync with the server | Most work happens locally |
| Sharing work-in-progress | Shelvesets | Not directly supported |

| TFS concept | Git equivalent |
|---|---|
| Workspace (excluding pending changes) | Working tree |
| Workspace (including pending changes) | Stage/index |
| TFS Server | Local repository *and* remote repository (two distinct concepts in Git) |

Git's `pull`/`push` against a remote server approximate TFS's centralized model, and `commit` + `push` let you update both the local and remote repository in one flow.

---

## 3. Getting Started

### 3.1 Prerequisites

- JDK 1.8
- Gradle (wrapper provided per project)
- MySQL / MSSQL / Oracle (MyBatis is the ORM used throughout the samples)
- Tomcat or WildFly for WAR deployment (see [14. Build & Deployment](#14-build--deployment))

### 3.2 Multi-module Gradle Setup

In local development, a consuming project's `settings.gradle` references the framework modules by relative path so you can edit framework and application code side-by-side without publishing snapshots. This block is skipped automatically in CI (`GRADLE_USER_HOME` is set there, so the packaged artifacts are used instead):

```gradle
rootProject.name = 'tsb.fontos.sample'

if (System.getenv("GRADLE_USER_HOME") == null) {
    println "** project reference DEV TOOL"

    include ':tsb.fontos.core'
    project(':tsb.fontos.core').projectDir =
        new File(settingsDir, '../../../Tsb-Fontos-Framework/Fontos-Framework-Backend/tsb.fontos.core')

    include ':tsb.fontos.security'
    project(':tsb.fontos.security').projectDir =
        new File(settingsDir, '../../../Tsb-Fontos-Framework/Fontos-Framework-Backend/tsb.fontos.security')

    include ':tsb.fontos.net'
    project(':tsb.fontos.net').projectDir =
        new File(settingsDir, '../../../Tsb-Fontos-Framework/Fontos-Framework-Backend/tsb.fontos.net')

    include ':tsb.fontos.report.jasper'
    project(':tsb.fontos.report.jasper').projectDir =
        new File(settingsDir, '../../../Tsb-Fontos-Framework/Fontos-Framework-Backend/tsb.fontos.report.jasper')
}
```

> The relative path above (`../../../Tsb-Fontos-Framework/...`) assumes a specific sibling-checkout layout on the author's machine. Adjust it to wherever `Fontos-Framework-Backend` is actually checked out in your workspace — on the current dev environment it is simply a sibling folder, e.g. `../../Fontos-Framework-Backend/...`.

In `build.gradle`, add the module(s) your service actually needs as project dependencies (local dev) and their transitive third-party libraries:

```gradle
if (System.getenv("GRADLE_USER_HOME") == null) {
    println "** project dependencies DEV TOOL"
    implementation project(":tsb.fontos.core")
    implementation project(":tsb.fontos.security")
    implementation project(":tsb.fontos.net")
    // add report/jasper, report/openhtmltopdf, cnet, core.micrometer as needed
}

// Security / JWT
implementation 'org.springframework.boot:spring-boot-starter-security'
compileOnly 'io.jsonwebtoken:jjwt:0.9.1'   // NOTE: compileOnly in tsb.fontos.security —
                                            // consuming apps must add jjwt as a runtime dependency themselves
implementation 'com.konghq:unirest-java:3.7.02'

// Encryption (Jasypt, used for encrypted config values)
implementation 'com.github.ulisesbocchio:jasypt-spring-boot-starter:3.0.4'
implementation 'org.bouncycastle:bcprov-jdk16:1.45'
```

### 3.3 Reference Project Skeleton

`Fontos-Sample-Backend/tsb.fontos.sample` is the canonical starter layout. New services typically copy this structure:

```
tsb.fontos.sample/
├── build.gradle, settings.gradle, build_external.gradle, build.properties
├── buildConfigs/{tomcat,wildfly,wildfly.terminal_id}/   # per-site property overlays, see §14
├── src/main/java/tsb/fontos/sample/
│   ├── SampleApplication.java          # @SpringBootApplication, extends TBaseResourceAppInitializer
│   ├── SampleServletInitializer.java   # WAR bootstrap (SpringBootServletInitializer)
│   ├── aop/                            # ApplicationContextProvider, LogInterceptor, CertificationInterceptor
│   ├── config/                         # OpenApiConfig, SwaggerConfig, WebConfig, WebSecurityConfig, ...
│   ├── controller/  → board/, jwt/, jwt/sso/, login/, report/{jasper,openhtmltopdf}/, excel/, web/, xml/, opt/
│   ├── dao/, item/, param/, service/    # layered MyBatis DAO → Service → Controller
│   ├── datasouce/                       # multi-datasource routing (RoutingDataSource, DataSourceType)
│   ├── ftp/, mail/, oracle/, encryption/ # Jasypt AES/DES config
│   ├── scheduler/                       # simple @Scheduled examples, see §13
│   ├── security/                        # JwtAuthenticationSecurityConfiguration, sqlInjection/, passwordValidation/
│   └── ws/                              # STOMP WebSocket sample, see §7
├── src/main/resources/
│   ├── application.properties           # ~350 lines, profile-agnostic defaults
│   ├── SL/module-config.properties      # feature on/off toggles, messages, logback
│   ├── mapper/{mysql,mssql}/…           # MyBatis XML mappers
│   └── application-context.xml          # legacy XML bean config, still present alongside Java config
└── src/main/webapp/WEB-INF/             # jboss-web.xml, jboss-deployment-structure.xml (WildFly)
```

`SampleApplication.java` shows the standard entry-point shape:

```java
package tsb.fontos.sample;

import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

import tsb.fontos.core.security.TDEVEnableWebSecurity;
import tsb.fontos.core.web.TBaseResourceAppInitializer;

@SpringBootApplication
@EnableAspectJAutoProxy
@ComponentScan
@TDEVEnableWebSecurity   // simple @EnableWebSecurity wrapper from tsb.fontos.core.security
public class SampleApplication extends TBaseResourceAppInitializer {
    // TApplicationResouceConfigInit.loadApplicationProperty(...) is invoked during startup
    // to resolve app-config.properties / application.properties per the active site profile.
}
```

### 3.4 Core Configuration Files

Framework-level path/behavior configuration is centralized in `app-config.properties`, which is resolved via `env.appconfig.*` keys pointing at the actual property/context/module files (these can point to `classpath:` or `file:` locations depending on the deployment target):

```properties
env.appconfig.app-property=classpath:application.properties
env.appconfig.app-context=classpath:application-context.xml
env.appconfig.module-info=classpath:module-Info.properties
env.appconfig.encryptor=tsb.fontos.core.encryption.JasyptConfigAES

env.module.database-info=database-Info.properties
env.module.log-info=logback-spring.xml
env.module.msg-info=messages
env.module.config-info=module-config.properties
env.module.local-cache=classpath:localCacheinfo.xml

# routing.datasouce.policy is "param" or "session"
routing.datasouce.policy=param
routing.datasouce.session.name=dataSouceType
routing.datasouce.parameter.name=getDataSouceType
```

HTTP request/response tracing is a commonly-used framework toggle worth knowing early:

```properties
# ==== HTTP REQUEST, RESPONSE Tracing SETTING [START]=====
fontos.http.tracing.enabled=true
fontos.http.tracing.patterns-included=/**
fontos.http.tracing.patterns-excluded=/api/test1,/api/test2
fontos.http.tracing.request-enable=true
fontos.http.tracing.response-enable=true
fontos.http.tracing.max-body-size=1048576
# ==== HTTP REQUEST, RESPONSE Tracing SETTING [STOP]=====
```

---

## 4. CATOS+ Project Architecture & Conventions

This section documents **CATOS+**, the actual product built on top of the `tsb.fontos.*` framework covered elsewhere in this guide. Where the earlier sections describe framework building blocks, this section describes the product-level layering, naming rules, and coding conventions that CATOS+ backend projects (`catos.*`) are expected to follow. It assumes working familiarity with Java, Spring Boot, MyBatis, and Gradle.

### 4.1 Layered Architecture

CATOS+ separates **Business** logic from **Client** (per-site/per-module web entry points) so that the same business logic can be reused by multiple clients instead of being duplicated per site:

- **Business** projects are split by domain area: `Common`, `Quay`, `Yard`, `Gate`, `Rail`.
- Each **Client** project references the Business project(s) it needs as a library dependency.

```
                              ┌────────┐   ┌──────┐
                              │   DB   │   │ C3IT │
                              └───┬────┘   └──┬───┘
                    ┌─────────────┴─────┬─────┴─────────────┐
                    │                                        │
     ┌──────────────▼───────────────┐      ┌─────────────────▼──────────────┐
     │ yq.war                       │      │ ta.war                          │
     │  FONTOS                      │      │  FONTOS                         │
     │    └─ CATOS CM               │      │    └─ CATOS CM                  │
     │         ├─ Biz Base          │      │         ├─ Biz Base             │
     │         ├─ Biz Common ⇄ Yard │      │         ├─ Biz Common ⇄ Quay    │
     │         └─ Ct Yq             │      │         └─ Ct Ta                │
     └──────────────┬───────────────┘      └─────────────────┬──────────────┘
       ┌─────────────┼─────────────┐          ┌───────────────┼───────────────┐
   Ct Yq1         Ct Yq2        Ct Yq3     Ct Ta1          Ct Ta2          Ct Ta3
  (Frontend)     (Frontend)   (Frontend)  (Frontend)      (Frontend)      (Frontend)
```

Each deployable WAR (e.g. `yq.war` for Yard/Quay, `ta.war` for Terminal Appointment — naming is site/module specific) stacks: the `tsb.fontos.*` framework at the bottom, **CATOS CM** (the shared CATOS common module) above it, one or more **Biz** projects for the domain areas that WAR needs, and one **Ct** project that exposes those Biz services as REST endpoints. Multiple frontend applications (`Ct Yq1`, `Ct Yq2`, `Ct Yq3`, ...) then all talk to that single Ct/WAR deployment, sharing the same business logic instead of each reimplementing it. All WARs ultimately share the same database and, where relevant, the same external terminal message server (`C3IT` — see [8. TCP / Socket Messaging](#8-tcp--socket-messaging)).

### 4.2 Terminology: FONTOS.NET vs FONTOS.J2EE

For teams moving from the older .NET stack to the Java stack:

| Item | FONTOS.NET | FONTOS.J2EE |
|---|---|---|
| Programming language | C# (.NET Framework 4.7.2) | Java (JDK 1.8 8u202) |
| Framework | Spring.NET | Spring Boot (2.6.7) |
| ORM | iBATIS | MyBatis |
| SCM | TFS (MS Azure) | Git (MS Azure) |
| In-code querying | LINQ | Stream |

**Java 8** was chosen because it supports most of the language features .NET developers already rely on (lambdas, Stream API) and is the last JDK 8 update usable without an Oracle commercial license. **Spring Boot 2.6.7** is the newest version that still targets Java 8; it brings an embedded web server (Tomcat by default), Gradle-based build/dependency management, auto-configuration, and easy RESTful API support (`@RestController`/`@RequestMapping`) plus scheduling (`@EnableScheduling`/`@Scheduled`).

**iBATIS → MyBatis**, side by side:

| Item | iBATIS | MyBatis |
|---|---|---|
| If | `<isEqual>`, `<isNotEmpty>`, `<isNotNull>`, ... | `<if test="condition">` |
| Choose/When/Otherwise | not supported | `<choose><when><otherwise>` |
| Loop | `<iterate>` | `<foreach>` |

**LINQ → Stream**, the common operations:

| Stream | LINQ | Description |
|---|---|---|
| `.stream()` | `.AsEnumerable()` | Convert a collection into a stream |
| `.parallelStream()` | `.AsParallel()` | Process in parallel |
| `.filter(predicate)` | `.Where(predicate)` | Keep elements matching a condition |
| `.map(Function)` | `.Select(Func)` | Transform each element |
| `.flatMap(Function)` | `.SelectMany(Func)` | Flatten nested collections |
| `.sorted()` | `.OrderBy(Func)` | Ascending sort |
| `.sorted(Comparator.reverseOrder())` | `.OrderByDescending(Func)` | Descending sort |
| `.distinct()` | `.Distinct()` | Remove duplicates |
| `.limit(n)` | `.Take(n)` | Take the first N |
| `.skip(n)` | `.Skip(n)` | Skip the first N |
| `.count()` | `.Count()` | Element count |
| `.max(Comparator)` / `.min(Comparator)` | `.Max(Func)` / `.Min(Func)` | Max/min |
| `.mapToInt().sum()` / `.average()` | `.Sum(Func)` / `.Average(Func)` | Sum/average |
| `.anyMatch(predicate)` / `.allMatch(predicate)` | `.Any(predicate)` / `.All(predicate)` | Any/all match |
| `.findFirst()` | `.FirstOrDefault()` | First element |
| not supported | `.LastOrDefault()` | Last element |
| `.collect(Collectors.toList())` | `.ToList()` | Materialize to a list |
| `.collect(Collectors.joining(","))` | `.Aggregate((x, y) => x + "," + y)` | Join to a string |
| `.forEach(Consumer)` | `.ForEach(Action)` | Iterate with a side effect |

### 4.3 Naming Rules & Conventions

**Package naming** follows `{Product Name}.{Group Name}.{Module ID}`:

- `{Product Name}`: `catos`
- `{Group Name}`: `ct` / `biz` / `dto` / `ex`
- `{Module ID}`: `common`, `quay`, `yard`, `ta`, `yq`, ... (site/domain specific)

Examples: `catos.ct.ta`, `catos.ct.yq`, `catos.biz.common`, `catos.biz.quay`, `catos.dto.common`, `catos.dto.quay`, `catos.ex.conpas`, `catos.ex.kline`.

**Casing** — Camel Case and Pascal Case are both used, but for different things than in .NET:

| Item | FONTOS.NET | FONTOS.J2EE |
|---|---|---|
| Class | Pascal Case | Pascal Case |
| Method | Pascal Case | Camel Case |
| Property | Pascal Case | Camel Case |
| Field | Camel Case | Camel Case |

(Camel Case: first word lowercase, subsequent words capitalized, e.g. `exampleMethodName`. Pascal Case: every word capitalized, e.g. `ExampleMethodName`.)

### 4.4 SQL Query Conventions

Because CATOS+ must run against multiple database engines (Oracle, MS-SQL), SQL is written in **uppercase**, using **ANSI JOIN** syntax rather than each vendor's own legacy join shorthand (e.g. Oracle's `(+)` outer-join operator):

| Oracle-style join | ANSI JOIN |
|---|---|
| `SELECT E.NAME, D.DEPT_NAME`<br>`FROM EMPLOYEES E, DEPARTMENTS D`<br>`WHERE E.DEPT_ID = D.DEPT_ID (+)` | `SELECT E.NAME, D.DEPT_NAME`<br>`FROM EMPLOYEES E`<br>`LEFT OUTER JOIN DEPARTMENTS D`<br>`ON E.DEPT_ID = D.DEPT_ID` |

### 4.5 Dto (Data Transfer Object) Projects

- Created as `catos.dto.{business}`; roughly equivalent to the `Common` project in FONTOS.NET.
- Holds only `Item`/`Param` classes shared across Biz projects — kept in a separate project specifically so that Biz projects don't have to reference each other directly to share these types, which would otherwise create circular project references.
- Classes are normally fields-only (occasionally a helper method); use Lombok's `@Getter`/`@Setter` instead of hand-writing accessors.
- **Item** — carries a result value out of a DB call/function; extends the framework's `CTBaseItem`:

```java
package catos.dto.sample.item.grid;

import lombok.Getter;
import lombok.Setter;
import catos.cm.core.item.common.CTBaseItem;
import tsb.fontos.core.util.TStringUtil;

@Getter
@Setter
public class SingleGridItem extends CTBaseItem {
    private static final long serialVersionUID = 1L;

    private String trailerUid = TStringUtil.Empty;
    private String trailerNo = TStringUtil.Empty;
    private String trucker = TStringUtil.Empty;
    private int tareWgt;
    private int maxWgt;
    private int loadableWgt;
    private int axleQty;
    private boolean unusedChk;
    private String suspendFrom = TStringUtil.Empty;
    private String suspendTo = TStringUtil.Empty;
    private String remark = TStringUtil.Empty;
}
```

- **Param** — carries parameters *into* a DB call/function; extends the framework's `CTBaseParam`. Comma-separated filter fields commonly expose a derived `getXxxList()` convenience method:

```java
package catos.dto.sample.param.grid;

import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

import lombok.Getter;
import lombok.Setter;
import catos.cm.core.param.common.CTBaseParam;
import tsb.fontos.core.util.TStringUtil;

@Getter
@Setter
public class SingleGridParam extends CTBaseParam {
    private static final long serialVersionUID = 1L;

    private String trailerNos;
    private String truckers;

    public List<String> getTrailerNoList() {
        if (TStringUtil.isNullOrEmpty(trailerNos) == true) {
            return null;
        } else {
            return Stream.of(trailerNos.split(",")).map(p -> p.trim()).collect(Collectors.toList());
        }
    }

    public List<String> getTruckerList() {
        if (TStringUtil.isNullOrEmpty(truckers) == true) {
            return null;
        } else {
            return Stream.of(truckers.split(",")).map(p -> p.trim()).collect(Collectors.toList());
        }
    }
}
```

### 4.6 Biz (Business) Projects

- Holds Service/Dao/Mapper — i.e. the project that actually implements CATOS business logic; roughly equivalent to the `Service` project in FONTOS.NET.
- FONTOS.NET's Service lived *inside* each Client project, which meant it couldn't be shared across clients. CATOS+ fixes this by giving each business domain (`Common`/`Quay`/`Yard`/`Gate`/`Rail`) its own standalone Biz project, created as `catos.biz.{business}`.
- Biz projects sometimes need to call each other, but direct project-to-project references would create circular dependencies. The fix is a shared **`Biz.Base`** project that holds only Service *interfaces* — Biz projects depend on `Biz.Base`'s interfaces, never on another Biz project's concrete `Service` class directly. At runtime, when the Ct project starts up, Spring's component scan discovers the concrete `Service` implementations and injects them wherever a `Biz.Base` interface is wired in — so the indirection is resolved by Spring, not by a direct compile-time project reference.

```java
package catos.biz.base_sample.service.grid;

import catos.cm.core.item.common.CTBaseResultItem;
import catos.dto.sample.item.grid.SingleGridItem;
import catos.dto.sample.param.grid.SingleGridParam;
import tsb.fontos.core.item.BaseItemsList;

public interface ISingleGridService {
    BaseItemsList<SingleGridItem> inquirySingleGrid(SingleGridParam param);
    CTBaseResultItem saveSingleGrid(SingleGridItem item);
    CTBaseResultItem deleteSingleGrid(SingleGridItem item);
}
```

- `Biz.{Business}` projects are created as `catos.biz.{business}`.
- **Service**: annotate the implementing class `@Service` so Spring registers it as a bean. Implements the actual business logic. Exceptions are generally *not* caught locally — let them propagate (or explicitly `throw`) so the Ct project's controller layer can handle them centrally (see [4.7](#47-ct-client-projects)). Any method that hits the DAO layer needs an explicit transaction annotation, e.g. `@Transactional(propagation = Propagation.REQUIRED)`.
- **`@Transactional`** — declarative transaction management; an AOP-driven annotation ensuring a set of DB operations either all succeed or all roll back:

| Propagation | Description |
|---|---|
| `REQUIRED` (default) | Join the current transaction if one exists, otherwise start a new one |
| `REQUIRES_NEW` | Always start a new transaction |
| `NESTED` | Start a nested transaction (uses a savepoint) |
| `SUPPORTS` | Join if a transaction exists, otherwise run without one |
| `NOT_SUPPORTED` | Run without a transaction |
| `NEVER` | Throw if a transaction exists |
| `MANDATORY` | Require an existing transaction, throw if none |

**Inquiry** (read-only) methods use `Propagation.NEVER` since no transaction is needed:

```java
// excerpt from SingleGridServiceImpl implements ISingleGridService
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import tsb.fontos.core.item.BaseItemsList;
import catos.dto.sample.item.grid.SingleGridItem;
import catos.dto.sample.param.grid.SingleGridParam;

@Transactional(propagation = Propagation.NEVER)
public BaseItemsList<SingleGridItem> inquirySingleGrid(SingleGridParam param) {
    return this.truckMasterDao.inquirySingleGrid(param);
}
```

**Save** (insert/update) methods use `Propagation.REQUIRED` and return a `CTBaseResultItem`. To abort the operation for a validation failure, either throw a `BizException` with a message id (which the framework's `CTBaseRestController` catches and turns into a `ResultStatus.FAIL` response) or set the result status directly on `CTBaseResultItem`:

```java
// excerpt from SingleGridServiceImpl implements ISingleGridService (same imports as inquirySingleGrid above,
// plus the two used here):
import catos.cm.core.item.common.CTBaseResultItem;
import catos.cm.core.item.common.CTBaseResultItem.ResultStatus;
import tsb.fontos.core.exception.BizException;

@Transactional(propagation = Propagation.REQUIRED)
public CTBaseResultItem saveSingleGrid(SingleGridItem item) {
    CTBaseResultItem resultItem = new CTBaseResultItem();
    // ...
    if (count > 0) {
        throw new BizException("MSG_BIZSL_00003", new Object[] { "Trailer No.", item.getTrailerNo() });
    }
    // ...
    resultItem.setResultStatus(ResultStatus.SUCCESS);
    resultItem.setResultObject(item);
    return resultItem;
}
```

**Delete** methods follow the same `Propagation.REQUIRED` + `CTBaseResultItem` pattern as Save.

**`CTBaseResultItem`** — the standard result envelope for insert/update/delete/validation operations:

```java
package catos.cm.core.item.common;

import lombok.Getter;
import lombok.Setter;
import tsb.fontos.core.item.BaseDataItem;

@Getter
@Setter
public class CTBaseResultItem extends BaseDataItem {
    private static final long serialVersionUID = 1L;
    private String resultType;         // optional extra code/id
    private ResultStatus resultStatus;
    private Object resultObject;       // optional result payload
    private String msgId;              // message id to show the user
    private Object[] msgParam;         // parameters for that message
}
```

| Result Status | Description |
|---|---|
| `SUCCESS` (default) | The requested operation succeeded |
| `WARNING` | Succeeded, but a warning should be shown |
| `CONFIRM` | Needs user confirmation before proceeding |
| `FAIL` | The requested operation failed |

**Dao (Data Access Object)** — an interface annotated `@Mapper` so Spring registers it as a MyBatis mapper bean. Each Dao interface corresponds 1:1 with a MyBatis mapper XML file; method name, parameter type, and return type must match the mapper XML's statement `id`:

```java
// Dao — inside an interface such as:
// package catos.biz.sample.service.grid.dao;
// import catos.dto.sample.item.grid.SingleGridItem;
public int countDuplicatedTrailerNo(SingleGridItem item);
```
```xml
<!-- Mapper -->
<select id="countDuplicatedTrailerNo"
        parameterType="catos.dto.sample.item.grid.SingleGridItem"
        resultType="int">
    SELECT COUNT(*) COUNT
    FROM TB_TRAILER
    WHERE TRAILER_NO = #{trailerNo}
</select>
```

**Mapper** files live under `src/main/resources/mapper/{DB Type}/{Category}/{Mapper Name}.xml` — a full pair is maintained for **both** `oracle` and `mssql`:

```
src/main/resources/mapper/
├── mssql/grid/SingleGridMap.xml
└── oracle/grid/SingleGridMap.xml
```

The mapper's `namespace` is what binds it to its Dao interface:

```xml
<mapper namespace="catos.biz.sample.service.grid.dao.SingleGridDao">
```

Key tags: `<resultMap>` (map result columns to a class's properties), `<select>`/`<insert>`/`<update>`/`<delete>` (the four statement types), `parameterType` (Java type of the parameter passed in), `resultType` (Java type the SQL result maps to).

### 4.7 Ct (Client) Projects

- Created as `catos.ct.{Module ID}`; the web server that serves the Frontend and exposes the REST API it consumes — conceptually the "Client" project from FONTOS.NET, but also doubles as the Spring Boot/library configuration root for the deployable.

| | Web server | API server |
|---|---|---|
| Purpose | Render/serve web pages (HTML/CSS/JS) | Serve data (JSON/XML) and run logic |
| Main use | User interface | Serving Frontend apps, other clients |
| Response format | Static/dynamic web content | Data responses |

- References the Dto/Biz projects it needs as library dependencies, and calls into their Services to do the actual work. Business logic should be pushed down into a Biz project's Service rather than implemented directly in Ct, unless it's genuinely Ct-specific (not shared/common CATOS business logic).
- Requests from the Frontend arrive through REST Controllers. CATOS+ uses **POST for everything**, regardless of whether the operation is a read or a write, because most requests need to pass a full object in the body.

**Configuration files** (in `resources/` or `resources/{Module ID}/`):

| File | Purpose |
|---|---|
| `application.properties` | Spring Boot + library configuration; also carries CATOS-specific runtime settings |
| `app-config.properties` | Build-related configuration |
| `application-context.xml` | Library/bean-scan/MyBatis map scan configuration |
| `module-Info.properties` | Version/title metadata for the client |
| `sql-mapper-config.xml` | MyBatis-related configuration |

Key `application.properties` entries specific to CATOS:

| Item | Description |
|---|---|
| `catos.version` | Version of the production line this client targets (e.g. `8.1`/`7.7`) |
| `app.productCode` | Site code this client runs for (e.g. `PCT`/`MMC`) — equivalent to `Product` in .NET's `app.config` |
| `app.dbType` | DB engine this client runs against (`oracle`/`mssql`) |

**`Application.java`** — the entry point class; its annotations control whether security is enabled:

| Annotation | Description |
|---|---|
| `@TApiEnableWebSecurity` | Apply security configuration |
| `@TDEVEnableWebSecurity` | Disable security — developer-mode only |

**`ApplicationRunner.java`** — runs once the project has fully started; used for post-startup configuration, and `@PreDestroy` handles cleanup on shutdown (e.g. closing a TCP connection to a C3IT-style message server):

```java
// excerpt from ApplicationRunner.java
import javax.annotation.PreDestroy;

@PreDestroy
public void onDestroy() throws Exception {
    if (this.simpleNetMessageClient != null) {
        this.simpleNetMessageClient.shutdown();
    }
}
```

**Package layout inside a Ct project:**

- `Common` — Item/Param classes specific to this client (most Item/Param types live in the Dto project instead; this package is normally limited to client-specific compile options/configuration).
- `Service` — same purpose as a Biz project's Service, but only for logic that's genuinely specific to this one client.
- `Controller` — REST Controllers that receive Frontend requests and call into the needed Biz Service(s). Extend `CTBaseRestController` for common `BizException` handling. Return `TRestResponse<ItemType>` (single `dataItem` or `dataItems` list, with `errorInfo` carrying exception details on failure).

| Annotation | Description |
|---|---|
| `@RestController` | Registers the class as a REST controller bean in the Spring context |
| `@RequestMapping("/grid")` | Sets the class-level URI path (e.g. `http://localhost:8080/grid/`) |
| `@RequiredArgsConstructor` | Lombok — generates a constructor for all `final` fields at compile time, so Spring can inject them |
| `@PostMapping("/inquiry-single-grid")` | Maps a method to POST at the given sub-path |
| `@RequestBody SingleGridParam param` | Binds the HTTP request body to a parameter object |

```java
package catos.ct.sample.controller.grid;

import java.util.List;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import lombok.RequiredArgsConstructor;
import catos.cm.core.controller.base.CTBaseRestController;
import catos.cm.core.item.common.CTBaseResultItem;
import catos.dto.sample.item.grid.SingleGridItem;
import catos.dto.sample.param.grid.SingleGridParam;
import tsb.fontos.core.http.rest.TRestResponse;
import catos.biz.base_sample.service.grid.ISingleGridService;

@RequiredArgsConstructor
@RestController
@RequestMapping("/grid")
public class SingleGridController extends CTBaseRestController {
    private final ISingleGridService singleGridService;

    @PostMapping("/inquiry-single-grid")
    public TRestResponse<SingleGridItem> inquirySingleGrid(@RequestBody SingleGridParam param) {
        List<SingleGridItem> itemList = this.singleGridService.inquirySingleGrid(param);
        return new TRestResponse<SingleGridItem>(itemList);
    }

    @PostMapping("/save-single-grid")
    public TRestResponse<CTBaseResultItem> saveSingleGrid(@RequestBody SingleGridItem item) {
        TRestResponse<CTBaseResultItem> responseItem = new TRestResponse<CTBaseResultItem>();
        CTBaseResultItem resultItem = this.singleGridService.saveSingleGrid(item);
        responseItem.setDataItem(resultItem);
        return responseItem;
    }
}
```

**Scheduler** — `@EnableScheduling` on a configuration class activates scheduling; `@Scheduled` on a method sets when it runs (CATOS uses cron expressions):

| Item | Description |
|---|---|
| `fixedRate` | Run on a fixed interval measured from the previous run's *start* (ms) |
| `fixedDelay` | Run a fixed delay after the previous run *finished* |
| `cron` | Precise scheduling via a cron expression |

`@ConditionalOnProperty` lets a scheduler be turned on/off purely via configuration, no code change needed — when the property doesn't match, Spring simply excludes that bean from the context. Example (`LogScheduler`, gated by `scheduler.log.enable`, firing every 30 seconds per `scheduler.log.cron`, pushing a WebSocket notification and writing a log line):

```java
package catos.ct.sample.scheduler.log;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import tsb.fontos.core.scheduler.TScheduler;
import tsb.fontos.net.ws.item.LogItem;

@Configuration
@EnableScheduling
@ConditionalOnProperty(name = "scheduler.log.enable", havingValue = "true")
@RequiredArgsConstructor
@Slf4j
public class LogScheduler extends TScheduler {
    private final SimpMessagingTemplate simpMessagingTemplate;

    @Scheduled(cron = "${scheduler.log.cron}")
    public void log() {
        LogItem logItem = new LogItem();
        logItem.setTitle("[Ct.Sample] LogScheduler");
        logItem.setMessage(LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
        this.simpMessagingTemplate.convertAndSend("/notifySample", logItem);
        log.debug("[Ct.Sample] LogScheduler - log");
    }
}
```

```properties
# application.properties
scheduler.log.enable=true
```
```properties
# module-config.properties
scheduler.log.cron=*/30 * * * * *
```

**WebSocket** — since plain HTTP only lets the backend respond to a request it already received, CATOS uses WebSocket (see [7. WebSocket / STOMP Messaging](#7-websocket--stomp-messaging) for the full framework mechanics) whenever the backend needs to push to the Frontend unprompted (e.g. broadcasting). `SimpMessagingTemplate` is the class used to send — see `LogScheduler` above for a usage example. Configured via `application.properties`; set `stomp.ws.enable=false` to disable WebSocket entirely for a client that doesn't need it (it defaults to enabled/`true` if unset):

```properties
# STOMP WebSocket [START] ===================================
#stomp.ws.enable=false
stomp.ws.message-size-limit=65536
stomp.ws.send-time-limit=1000000
stomp.ws.send-buffer-size-limit=524288
stomp.ws.endpoint=/endpoint
stomp.ws.simple-broker=/notifySample
stomp.ws.application-destination-prefixes=/pub
stomp.ws.user-destination-prefix=/user
stomp.ws.allowed-origin-patterns=*
# STOMP WebSocket [END] =======================================
```

### 4.8 Ex (External Interface) Projects

- Created as `catos.ex.{Business or Site code}`; used to interface with external (customer-side) systems.
- Common/shared CATOS business logic still belongs in a Biz project — Ex is reserved for integration logic that's specific to one external partner/site.
- Overall project structure mirrors a Ct project (see [4.7](#47-ct-client-projects)) — `catos.ex.sample` has reference examples for both the sending side (a Scheduler) and the receiving side (a Controller).
- **`CTRestTemplateUtil`** — a utility for calling another server over REST. Configure a `RestTemplateParam` (URI/path/method/headers/body); set `logTitle` if the call should be logged.

```java
// excerpt from a catos.ex.sample scheduler/controller class
import org.springframework.http.HttpHeaders;
import org.springframework.web.bind.annotation.RequestMethod;

import catos.dto.sample.item.grid.SingleGridItem;
import catos.dto.sample.param.grid.SingleGridParam;
import catos.ex.sample.config.SampleModuleConfiguration;
import catos.ex.sample.util.CTRestTemplateUtil;
import catos.ex.sample.util.RestTemplateParam;
import tsb.fontos.core.http.rest.TRestResponse;
import tsb.fontos.security.authentication.JwtAccessToken;

try {
    HttpHeaders headers = new HttpHeaders();
    headers.add(JwtAccessToken.REQUEST_HEADER_AUTH, this.jwtTokenRepository.getMasterToken());

    SingleGridParam param = new SingleGridParam();

    RestTemplateParam restParam = new RestTemplateParam();
    restParam.setUri(SampleModuleConfiguration.getInstance().getUriClientSample());
    restParam.setPath("/grid/inquiry-single-grid");
    restParam.setRequestMethod(RequestMethod.POST);
    restParam.setHeaders(headers);
    restParam.setBody(param);
    restParam.setResponseType(TRestResponse.class);
    restParam.setResponseSubType(SingleGridItem.class);

    TRestResponse<SingleGridItem> responseItem =
        (TRestResponse<SingleGridItem>) CTRestTemplateUtil.request(restParam);

    if (responseItem != null && responseItem.getDataItems() != null) {
        log.debug("[Ex.Sample] inquirySingleGrid : " + responseItem.getDataItems().size());
    } else {
        log.debug("[Ex.Sample] inquirySingleGrid : Fail");
    }
} catch (Exception ex) {
    log.debug("[Ex.Sample] inquirySingleGrid : Exception");
}
```

### 4.9 Java Coding Style Guide

Source: internal `Java Coding Style Guide v1.0` (2022-06-07). This closely mirrors Google's Java Style Guide (translated), and covers file structure and formatting beyond the naming rules in [4.3](#43-naming-rules--conventions). Treat 4.3 and this subsection together as the full style reference.

**File header template.** Every source file should open with a license/copyright block, then package, then imports, then the class — each section separated by one blank line:

```java
/**
 * CONFIDENTIAL AND PROPRIETARY SOURCE CODE OF TOTAL SOFT BANK
 * LIMITED
 *
 * Copyright (C) 2005-2022 TOTAL SOFT BANK LIMITED. All Rights
 * Reserved. Use of this source code is subject to the terms of
 * the applicable license agreement.
 *
 * The copyright notice(s) in this source code does not indicate
 * the actual or intended publication of this source code.
 *
 *
 * ----------------------------------
 * CHANGE REVISION
 * ----------------------------------
 * DATE            AUTHOR              REVISION
 * 2019.04.15      Lim JC 1.0          First release.
 */
package tsb.fontos.core;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

> The style guide's own package example is `tsb.fontos.core` — it predates (or is independent of) this guide's `catos.{ct|biz|dto|ex}.{module}` product-layer convention (§4.1–4.8). Use the `catos.*` scheme for CATOS+ product code; this header/import/format guidance still applies regardless of package root.

**File basics.** UTF-8 encoding; file name matches the class name exactly (case-sensitive) with a `.java` extension; only the ASCII space character (0x20) is used for whitespace — **tabs are never used for indentation**.

**Imports.** No wildcard (`*`) imports — always import the specific type. Group static imports separately from non-static imports, with one blank line between the two groups; no further ordering rule is specified within each group.

```java
import static java.lang.Integer.SIZE;
import static java.lang.Math.random;

import tsb.fontos.core.obj.TsbBaseObject;
import org.springframework.boot.SpringApplication;
import java.io.File;
```

**Class structure.** One class per `.java` file. Order members logically (don't just append new methods at the end); overloaded constructors/methods with the same name must be grouped together with nothing else interleaved between overloads.

**Braces (K&R style).** Braces are always used, even for single-statement `if`/`while`/`for` bodies:

```java
// good
if (a > b) {
    result = true;
} else {
    result = false;
}
```

Rules: no line break before an opening brace; line break after an opening brace; line break before a closing brace; line break after a closing brace **except** when followed by `else` or a comma; nothing else may share the line with an opening or closing brace. Empty blocks may close immediately (`void doNothing() {}`) **except** as part of a multi-block statement (`if`/`else`, `try`/`catch`/`finally`) — an empty `catch` block must still be written as `catch (Exception e) {\n}`, not collapsed to `{}`.

**Indentation & line length.** Block indentation is **2 spaces**. Line-wrap continuation indentation is a *different* rule — **4+ spaces** (these are not the same value; don't conflate them). Column limit is **100 characters**, except for unbreakable lines (long Javadoc URLs, package/import statements, shell command lines meant for copy-paste). One statement per line. When wrapping: break after a comma; break before a non-assignment operator; break after the assignment operator (`=`); wrapped lines get at least 4 spaces of indent, and consecutive wrapped lines match the first wrapped line's indent.

**Whitespace.** One blank line between consecutive members/initializers. No space after a method name (`doSomething(a, b)`, not `doSomething (a, b)`); no space after array brackets (`array[i]`); no space between a cast and its operand — **`(Application)getObject(a, b)`, not `(Application) getObject(a, b)`** (this is stricter than common Google-style Java, which normally keeps a space after a cast — verify this is intentional company policy before enforcing it in review). Space on both sides of binary operators (`a = b + c`); space after commas/semicolons; space after `if`/`while`/`for`/`switch`/`catch` keywords. Column-aligning variable declarations with padding spaces is **not recommended**.

**Constants.** `ALL_UPPERCASE` with words separated by `_` (e.g. `PRODUCT_NAME`) — this is the one naming rule not already covered by the class/method/field/parameter conventions in §4.3.

**Specific constructs:**
- **Enums** follow normal class rules; a line break after the comma following a constant is optional — `private enum Answer { YES, NO, MAYBE }` and the one-per-line form are both acceptable.
- **Variable declarations**: one variable per statement (`int a; int b;`, not `int a, b;`), except a `for` loop header may declare multiple (`for (int a = 1, b = 2; a < 10; a++)`). Declare locals as close as possible to first use, not at the top of the block.
- **Arrays**: Java-style declaration only (`int[] array`, never `int array[]`); block-style initializers are allowed in any comma/line-break layout.
- **Switch statements**: indent like any other block; an intentional fall-through must be marked with a `// fall through` comment; a `default` clause is mandatory even if empty.
- **Annotations**: one per line normally; a single parameterless annotation may instead sit inline before the declaration (`@Nullable public void foo() {}`); multiple annotations on one line are allowed for fields (`@Partial @Mock DataLoader loader;`).
- **Numeric literals**: `long` literals use an uppercase `L` suffix (`1000000000L`), never lowercase `l` (easily confused with `1`).

**Known gaps in the source style guide** (not omissions in this write-up — verify with the team before enforcing): the "Modifier order" section has only a heading with no actual ordering list defined; there is no Eclipse formatter profile, Checkstyle config, or other IDE tooling referenced anywhere; and the guide does not cover exception-handling or logging conventions, or Javadoc tag conventions, at all.

---

## 5. REST API Foundations

### 5.1 Base Controller / Service Classes

`tsb.fontos.core` supplies a thin, opinionated layer over Spring MVC/MyBatis:

- `ITsbRestController` (marker interface, used as an AOP pointcut) → `TBaseRestController implements ITsbRestController` — extend this for new REST controllers; it gives you the framework's common helpers and plugs your controller into the shared AOP aspects (`TRestExceptionAspect`, `TControllerPagingAspect`, `TDaoNewVersionAspect`).
- `ITsbService` → `BaseService implements ITsbService` (abstract) — implement `getDao()`; you get CRUD dispatch (`list/one/string/count/insert/update/delete`), reflection-based dynamic DAO invocation, and optimistic-locking support for free:

```java
package tsb.fontos.core.service;

import tsb.fontos.core.dao.BaseDao;
import tsb.fontos.core.item.BaseDataItem;

public abstract class BaseService implements ITsbService {
    public abstract BaseDao getDao();

    public int process(BaseDataItem item) {
        switch (item.getOpCode()) {
            case CREATE: return this.getDao().insert(item);
            case UPDATE: return this.getDao().update(item);
            case DELETE: return this.getDao().delete(item);
            default: return 0;
        }
    }
}
```

- `TBaseRestService` — a `BaseService` specialization used for simple query-only services (see the Excel export example in §11.2, `SingleGridService extends TBaseRestService`).

### 5.2 Standard Response Wrappers

- `ITRestResponse` → `TRestNormalResponse<T>` (success code + result payload), `TRestErrorResponse` (error code + `ErrorMsgItem`)
- `TRestResponse<T> extends BaseRestResponse<T>` — adds paging support (single item / list / `TPageInfo`-paginated, via multiple constructors)

### 5.3 Exception Handling

- `BaseException extends RuntimeException` — carries an `errorId`, `MessageFormat`-style dynamic parameters, and a cause chain (`getCauses()`)
- `BizException extends BaseException` — business-rule exception; `getMessage()` is reformatted to `[<ClassName>] BizException occurred!! ErrorId: ...` for consistent log grepping
- `TExceptionResponeHandler` — static helper used by the JWT filter/entry point/access-denied handler to write a standard JSON error body directly onto `HttpServletResponse` at the servlet-filter level (i.e., before Spring MVC exception handling would normally kick in)

### 5.4 Pagination

List endpoints page results via **PageHelper**, wrapped by the framework's own `TPageHelper`/`TPageInfo` types — don't call `com.github.pagehelper.PageHelper` directly in application code.

**Dependencies:**

```gradle
implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.2'
implementation 'com.github.pagehelper:pagehelper:5.3.0'
```

**MyBatis config** — register `PageInterceptor` in the existing `<plugins>` block of `sql-mapper-config.xml`, after any existing interceptors (e.g. `tsb.fontos.sample.aop.LogInterceptor`):

```xml
<plugin interceptor="com.github.pagehelper.PageInterceptor">
    <property name="offsetAsPageNum" value="true" />
    <property name="rowBoundsWithCount" value="true" />
    <property name="pageSizeZero" value="false" />
    <property name="reasonable" value="false" />
</plugin>
```

| Property | Default | Meaning |
|---|---|---|
| `helperDialect` | `auto` | DB dialect (`mysql`, `oracle`, `mariadb`, `postgresql`, `sqlserver`, `db2`, `h2`, ...); `auto` detects from the JDBC URL |
| `offsetAsPageNum` | `false` | Treat `RowBounds.offset` as a page number |
| `rowBoundsWithCount` | `false` | Also run a `count(*)` query when using RowBounds |
| `pageSizeZero` | `false` | `pageSize = 0` returns all results, unpaged |
| `reasonable` | `false` | Auto-correct an out-of-range page number (`< 1` → 1, `> pages` → last page) |
| `supportMethodsArguments` | `false` | Allow Mapper methods to take `pageNum`/`pageSize` as direct arguments |
| `params` | `count=countSql` | Parameter name used with `startPage(Object params)` |
| `autoRuntimeDialect` | `false` | Detect DB type at runtime — useful with multi-datasource setups (§16) |
| `closeConn` | `true` | Close the Connection after the count query |

**Controller/Service pattern:**

```java
// excerpt from a Controller in tsb.fontos.sample.controller.board
import java.util.List;

import javax.servlet.http.HttpServletRequest;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import tsb.fontos.core.item.TPageInfo;
import tsb.fontos.core.pagehelper.TPageHelper;
import tsb.fontos.sample.item.board.BoardItem;
import tsb.fontos.sample.param.board.BoardParam;

@RequestMapping(value = "/boardPage", method = RequestMethod.GET)
public List<BoardItem> boardPage(HttpServletRequest request, BoardParam param) {
    TPageHelper.startPage(param.getPageNum(), param.getPageSize()); // paging setup
    TPageInfo pageItem = TPageHelper.of(service.selectBoardAll(param), true); // Mapper call
    List<BoardItem> list = pageItem.getList();
    // to also return paging metadata: return new TRestResponse<>(pageItem, list);
    return list;
}
```

`TPageHelper.startPage(pageNum, pageSize)` must be called immediately before the Mapper/Service call it applies to (PageHelper works by intercepting the *next* MyBatis query on the current thread). `TPageInfo` exposes at least `getList()` and `getPageSize()`. It's also acceptable to call `startPage` in the Service layer instead of the Controller.

```xml
<select id="selectBoardAll" resultMap="boardResult">
    SELECT ID, TITLE, CONTENT FROM BOARD
    <if test="title != null and title != ''">
        WHERE TITLE LIKE '%${title}%'
    </if>
    ORDER BY ID ASC
</select>
```

> **Security note**: `LIKE '%${title}%'` uses raw string substitution (`${title}`), not a bind parameter (`#{title}`) — this is SQL-injectable if `title` isn't validated upstream (see [6.2](#62-sql-injection-validator)). Prefer binding `title` with `#{title}` and building the `%...%` wildcarding in Java/service code, or strictly validate/escape the input if `${}` interpolation is unavoidable for this pattern.

The request-side paging param only needs `pageNum`/`pageSize` getters (e.g. on `BoardParam`); the response-side paging shape (`TPageInfo`, full `TRestResponse` field list) should be confirmed against the actual framework source if you need more than `getList()`/`getPageSize()`.

### 5.5 API Documentation (SpringDoc / OpenAPI / Swagger)

```gradle
implementation 'org.springdoc:springdoc-openapi-ui:1.6.11'
```

```properties
springdoc.version=v1.0.0
springdoc.packages-to-scan=tsb.fontos.sample.controller
springdoc.packages-to-exclude=tsb.fontos.sample.test01,tsb.fontos.sample.test02
springdoc.paths-to-exclude=/two/**,/hello**, /test**/**
springdoc.swagger-ui.disable-swagger-default-url=true
springdoc.default-consumes-media-type=application/json;charset=UTF-8
springdoc.default-produces-media-type=application/json;charset=UTF-8
springdoc.swagger-ui.tagsSorter=alpha
springdoc.swagger-ui.operationsSorter=alpha
springdoc.swagger-ui.enabled=true
springdoc.api-docs.path=/api-docs/json
springdoc.api-docs.groups.enabled=true
```

UI is served at `/swagger-ui.html` (or `/swagger-ui/index.html`); reference: [springdoc.org](https://springdoc.org/#properties).

**Document metadata + JWT bearer scheme**, registered as an `OpenAPI` bean:

```java
package tsb.fontos.sample.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpHeaders;

import io.swagger.v3.oas.models.Components;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import io.swagger.v3.oas.models.security.SecurityRequirement;
import io.swagger.v3.oas.models.security.SecurityScheme;

@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI openAPI(@Value("${springdoc.version}") String version) {
        Info info = new Info()
            .title("Example API Docs")
            .version(version)
            .description("Report any issues or errors found.")
            .license(new License().name("Apache 2.0").url("http://springdoc.org"))
            .contact(new Contact().name("...").email("...").url("http://www.tsb.co.kr/"));

        SecurityScheme bearerAuth = new SecurityScheme()
            .type(SecurityScheme.Type.HTTP)
            .scheme("bearer")
            .bearerFormat("JWT")
            .in(SecurityScheme.In.HEADER)
            .name(HttpHeaders.AUTHORIZATION);

        SecurityRequirement addSecurityItem = new SecurityRequirement();
        addSecurityItem.addList("JWT");

        return new OpenAPI()
            .components(new Components().addSecuritySchemes("JWT", bearerAuth))
            .addSecurityItem(addSecurityItem)
            .info(info);
    }
}
```

**Annotating DTOs/params** with `@Schema` documents each field (description/example/required); a class's rendered schema in Swagger UI may include more fields than are declared in its own body if it extends a base DTO (e.g. `BoardItem extends BaseDataItem` — most rendered fields come from `BaseDataItem`, not the subclass).

**Common-response annotation pattern** — define one composite annotation per module for shared response codes instead of repeating `@ApiResponse` on every endpoint:

```java
package tsb.fontos.sample.config;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import io.swagger.v3.oas.annotations.media.ArraySchema;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import tsb.fontos.core.http.rest.TRestErrorResponse;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@ApiResponses(value = {
    @ApiResponse(responseCode = "401", description = "Unauthorized",
        content = @Content(array = @ArraySchema(schema = @Schema(implementation = TRestErrorResponse.class))))
})
public @interface ApiDocumentResponse { }
```

```java
package tsb.fontos.sample.controller.board;

import java.util.List;

import javax.servlet.http.HttpServletRequest;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import tsb.fontos.core.http.rest.TRestNormalResponse;
import tsb.fontos.sample.item.board.BoardItem;
import tsb.fontos.sample.param.board.SimpleBoardParam;

@RestController
@RequestMapping(value = "/board/v1")
@Tag(name = "Board API controller", description = "Board API controller desc")
@RequiredArgsConstructor
@ApiDocumentResponse
public class SwaggerDocumentationTestController {
    @GetMapping(value = "/commonboard")
    @Operation(summary = "Board list", description = "Fetch the board list")
    public TRestNormalResponse<List<BoardItem>> boardAllByNewResponse(HttpServletRequest request, SimpleBoardParam param) {
        // ...
    }
}
```

**Excluding endpoints from documentation:** `@Hidden` on a Controller class or individual method excludes just that target; for bulk exclusion, `springdoc.packages-to-exclude` / `springdoc.paths-to-exclude` (shown above) exclude by package or URL pattern without touching the endpoint code itself.

Annotation reference: `@Tag` (marks a class as a Swagger resource), `@Parameter`/`@Parameters` (single/multiple operation parameters), `@Schema` (model field metadata, including `accessMode = READ_ONLY` for response-only fields), `@Operation(summary=..., description=...)`, `@ApiResponse`/`@ApiResponses`, `@Hidden` (exclude from docs).

---

## 6. Security

### 6.1 JWT-based Authentication & Authorization

#### What is a JWT (JSON Web Token)?

A JWT is a `.`-delimited, Base64-encoded, stateless authentication token made of three parts: **Header** (signing algorithm, e.g. `HS256`), **Payload** (claims — user id, roles, expiry, etc.), and **Signature** (integrity check). No server-side session is required; the client carries its own auth state.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiJ1c2VySWQiLCJuYW1lIjoiSm9obiBEb2UiLCJpYXQiOjE2ODAwMDAwMDB9.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### Core classes

| Class | Package | Purpose |
|---|---|---|
| `JwtSecurityProperties` | `tsb.fontos.core.security` | `@ConfigurationProperties(prefix="fontos.jwt.security")` — access/refresh/SSO expiry + secret key |
| `JwtCookieProperties` | `tsb.fontos.core.security` | `@ConfigurationProperties(prefix="fontos.jwt.cookie")` — cookie name/path/maxAge/secure/httpOnly/sameSite |
| `JwtConfig` | `tsb.fontos.security.jwt` | Secret key constants + `@Bean JwtManager jwtManager()` |
| `JwtManager` (`extends JwtBaseManager` → `AbstractJwtManager`) | `tsb.fontos.security.jwt` | Low-level token build/parse (`Jwts.builder()...signWith(HS256, key)`, `getClaims(...)`) |
| `JwtTokenProvider` (`extends AbstractJwtTokenProvider`) | `tsb.fontos.security.jwt` | Application-facing API: `createToken`, `resolveToken`, `validateToken` / `validateTokenThrowException`, `isExpiringSoon` |
| `JwtRestApiAuthenticationToken` (`extends AbstractJwtAuthenticationToken`) | `tsb.fontos.security.authentication` | Builder-style claim holder passed into `createToken(...)` |
| `ApiJwtAuthenticateFilter` (`extends OncePerRequestFilter`) | `tsb.fontos.security.filter` | Parses/validates the incoming JWT and populates `SecurityContextHolder` |
| `ApiJwtRefreshFilter` (`extends OncePerRequestFilter`) | `tsb.fontos.security.filter` | Silently reissues a token nearing expiry, returned via the `x-access-token` response header |
| `AbstractJwtAuthenticationManager` | `tsb.fontos.security.jwt.mng` | Login/logout/refresh/SSO token lifecycle orchestration |
| `TApiEnableWebSecurity` | `tsb.fontos.security.web.configuration` | `@EnableWebSecurity` + `@Import(TApiWebSecurityConfiguration.class)` convenience annotation |

#### Generating a token

```java
// excerpt from a login Controller (this.jwtTokenProvider : tsb.fontos.security.jwt.JwtTokenProvider is an injected field)
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import org.springframework.web.bind.annotation.PostMapping;

import tsb.fontos.security.authentication.JwtRestApiAuthenticationToken;
import tsb.fontos.security.constant.TRoleConstants;
import tsb.fontos.sample.item.auth.UserInfo;
import tsb.fontos.sample.item.auth.UsernameToken;

@PostMapping("/auth/jwt/login")
public UsernameToken tokenByUserName(UserInfo param) {
    JwtRestApiAuthenticationToken token = new JwtRestApiAuthenticationToken();

    token.setSubject(param.getUserName());                 // required
    token.setIssuer("MODULE CODE");
    token.setAudience("WEBUSER");
    token.setIssuedAt(new Date());
    token.setExpire(new Date(System.currentTimeMillis()
            + (1000 * param.getExpireSeconds())));          // required

    token.setId("API 000123421 ID");
    token.setUsername("User Name");
    token.setIssuedType("service");

    List<String> roles = new ArrayList<>();
    roles.add(TRoleConstants.ROLE_USER);
    roles.add(TRoleConstants.ROLE_MANAGER);
    roles.add("SUPER_USER");
    roles.add("STEVEDORING_USER");
    token.setRoles(roles);                                   // required

    String tokenStr = this.jwtTokenProvider.createToken(token);

    UsernameToken usernameToken = new UsernameToken();
    usernameToken.setAccessToken(tokenStr);
    return usernameToken;
}
```

`subject`, `expire`, and `roles` are mandatory; everything else is optional/custom.

#### Validating a token

```java
// excerpt from JwtTokenProvider (tsb.fontos.security.jwt) — jwtMng : tsb.fontos.security.jwt.JwtManager
import java.util.Date;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.ExpiredJwtException;

public boolean validateTokenThrowException(String jwtToken) {
    Claims claims = jwtMng.getClaims(jwtToken);
    if (claims.getExpiration().before(new Date())) {
        throw new ExpiredJwtException(null, claims, "JWT token is expired.");
    }
    return true;
    // also throws SignatureException, MalformedJwtException, UnsupportedJwtException,
    // IllegalArgumentException — see JwtTokenProvider Javadoc in the framework source
}
```

#### Automatic refresh

`ApiJwtRefreshFilter` runs immediately after `ApiJwtAuthenticateFilter` and, when the current token's remaining validity drops below a configurable threshold, transparently issues a new one via a `JwtRefreshHandler` implementation and returns it in the `x-access-token` response header:

```java
package tsb.fontos.webprototype.security.jwt;

import java.util.Date;

import lombok.extern.slf4j.Slf4j;
import tsb.fontos.security.authentication.JwtAccessToken;
import tsb.fontos.security.authentication.JwtRestApiAuthenticationToken;
import tsb.fontos.security.filter.JwtRefreshHandler;
import tsb.fontos.security.jwt.JwtTokenProvider;

@Slf4j
public class SimpleJwtRefreshHandler implements JwtRefreshHandler {
    private JwtTokenProvider jwtTokenProvider;

    public SimpleJwtRefreshHandler(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @Override
    public String generateToken(JwtAccessToken token) {
        JwtRestApiAuthenticationToken newtoken = new JwtRestApiAuthenticationToken();
        newtoken.setSubject(token.getSubject());
        newtoken.setIssuer(token.getIssuer());
        newtoken.setAudience(token.getAudience());
        newtoken.setIssuedAt(new Date());
        newtoken.setExpire(new Date(System.currentTimeMillis() + (1000 * 60)));
        newtoken.setId(token.getId());
        newtoken.setUsername(token.getUsername());
        newtoken.setIssuedType(token.getIssuedType());
        newtoken.setRoles(token.getRoles());
        return this.jwtTokenProvider.createToken(newtoken);
    }
}

// registration — excerpt from a WebSecurityConfigurerAdapter-style @Configuration class
// import tsb.fontos.security.filter.ApiJwtRefreshFilter;
public ApiJwtRefreshFilter apiJwtRefreshFilter(JwtTokenProvider jwtTokenProvider) {
    ApiJwtRefreshFilter refreshFilter =
        new ApiJwtRefreshFilter(jwtTokenProvider, new SimpleJwtRefreshHandler(jwtTokenProvider));
    refreshFilter.setRefreshThresholdMinute(20); // reissue when <= 20 min remain
    return refreshFilter;
}
```

On the frontend, `TRemoteConfig` needs `accessTokenScheme = AppConstant.AUTH_SCHEME_BEARER` and `accessTokenRefreshHeadName = AppConstant.TOKEN_REFRESH_HEADER` so the framework's HTTP client automatically swaps in the refreshed token from the `x-access-token` response header.

#### Wiring it into Spring Security

```java
// excerpt from a WebSecurityConfigurerAdapter in tsb.fontos.webprototype.security.jwt
// (JwtRefreshAuthenticationAndAuthorizationMngSecurityConfiguration)
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.access.intercept.FilterSecurityInterceptor;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import tsb.fontos.security.filter.ApiJwtAuthenticateFilter;
import tsb.fontos.security.filter.UriPatternAuthorizationManagerManagerFilter;

@Override
public void configure(HttpSecurity http) throws Exception {
    http.httpBasic().disable()
        .csrf().disable()
        .cors().configurationSource(configurationSource()).and()
        .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        .and().authorizeRequests()
        .antMatchers(HttpMethod.POST, "/auth/jwt/login").permitAll()
        .antMatchers(HttpMethod.POST, "/auth/jwt/verifyToken").permitAll()
        .antMatchers("/**/auth/hello**").authenticated()
        .antMatchers(HttpMethod.GET, "/jwt/**").permitAll()
        .antMatchers(HttpMethod.POST, "/jwt/**").permitAll()
        .anyRequest().authenticated()
        .and()
        .addFilterBefore(apiJwtAuthenticateFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class)
        .addFilterAfter(apiJwtRefreshFilter(jwtTokenProvider), ApiJwtAuthenticateFilter.class)
        .addFilterAfter(new UriPatternAuthorizationManagerManagerFilter(
                uriPatternAuthorizationManager()
                    .addMapping("/**/api/**/inquiry**", "INQUIRY")
                    .addMapping("/**/api/**/save**", "SAVE")
                    .addMapping("/**/api/**/insert**", "INSERT")
                    .addMapping("/**/api/**/delete**", "DELETE")
                    .addControlMapping("/**/api/**/control/**")),
                FilterSecurityInterceptor.class)
        .exceptionHandling().authenticationEntryPoint(jwtAuthenticationEntryPoint())
        .accessDeniedHandler(apiAccessDeniedHandler());
}
```

#### Authorization (action-level permission check)

`UriPatternAuthorizationManagerManagerFilter` maps request URIs to logical "actions" (`INQUIRY`/`SAVE`/`INSERT`/`DELETE`/`CONTROL`), then delegates to a `BaseUriPatternActionPermissionChecker` implementation you provide:

```java
package tsb.fontos.webprototype.security.jwt;

import java.util.List;

import javax.servlet.http.HttpServletRequest;

import tsb.fontos.security.authorization.BaseUriPatternActionPermissionChecker;
import tsb.fontos.security.authorization.TUriPatternAuthorizationManager;
import tsb.fontos.security.authorization.UriPatternActionItem;
import tsb.fontos.security.authorization.UriPatternActionParam;
import tsb.fontos.security.profile.TDefaultUser;

public class SimpleUriPatternActionPermissionChecker extends BaseUriPatternActionPermissionChecker {

    @Override
    public boolean hasActionPermission(HttpServletRequest request, TDefaultUser user, UriPatternActionParam param) {
        List<UriPatternActionItem> accessRoleItems =
            store.SearchAccessRole(user.getUserId(), pgmCode, param.getApiName());

        return accessRoleItems.stream()
            .filter(item -> user.getUserId().equals(item.getUserId()))
            .filter(item -> param.getApiName().equals(item.getApiName()))
            .filter(item -> user.getRoles().contains(item.getRole()))
            .anyMatch(item -> hasPermissionForAction(item, param.getActionType()));
    }
}

public TUriPatternAuthorizationManager uriPatternAuthorizationManager() {
    return new TUriPatternAuthorizationManager(new SimpleUriPatternActionPermissionChecker());
}
```

**End-to-end flow:**

```
Client --(Authorization: Bearer <token>)--> ApiJwtAuthenticateFilter --(fail)--> 401 (ApiAccessDeniedHandler)
                                                  | success
                                                  v
                                          ApiJwtRefreshFilter  [reissues token if near expiry, via x-access-token header]
                                                  |
                                                  v
                                UriPatternAuthorizationManagerManagerFilter  [resolves URI -> action, e.g. INQUIRY]
                                                  |
                                                  v
                                          ActionPermissionChecker --(fail)--> 403
                                                  | success
                                                  v
                                              Controller
```

Reference implementation package: `tsb.fontos.webprototype.security.jwt` in `Fontos-Sample-WebPrototype` (`JwtRefreshAuthenticationAndAuthorizationMngSecurityConfiguration`, `SimpleJwtRefreshHandler`, `SimpleUriPatternActionPermissionChecker`, `WebJwtSecurityBeanConfiguration`) and `tsb.fontos.sample.security.*` in `Fontos-Sample-Backend`.

### 6.2 SQL Injection Validator

Package: `tsb.fontos.core.security.sql`.

**Purpose**: detect and reject dangerous SQL-Injection-shaped input on selected DTO fields at Spring's data-binding stage, log the offending value server-side only, and return a generic, safe error to the client.

1. Mark the DTO class and the risky field(s):

```java
package tsb.fontos.sample.item.sqlinjection;

import lombok.Getter;
import lombok.Setter;
import tsb.fontos.core.security.sql.CheckSqlInjection;
import tsb.fontos.core.security.sql.SqlInjectionTarget;

@Getter
@Setter
@CheckSqlInjection
public class SqlInjectionTestItem {
    @SqlInjectionTarget
    private String containerNo;
    private String vesselCode; // not validated — no annotation
}
```

2. Register the validator via a `ControllerAdvice` that extends the framework base class:

```java
package tsb.fontos.sample.security.sqlinjection;

import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.InitBinder;

import tsb.fontos.core.security.sql.BaseSqlInjectionControllerAdvice;
import tsb.fontos.core.security.sql.SqlInjectionSpringValidator;

@ControllerAdvice
public class SecureSqlInjectionControllerAdvice extends BaseSqlInjectionControllerAdvice {
    private final SqlInjectionSpringValidator validator = new SqlInjectionSpringValidator();

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        binder.addValidators(validator);
    }
}
```

3. Add a dedicated exception handler that logs details server-side but returns a minimal message to the client:

```java
package tsb.fontos.sample.security.sqlinjection;

import java.util.HashMap;
import java.util.Map;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import lombok.extern.slf4j.Slf4j;
import tsb.fontos.core.security.sql.SqlInjectionException;

@Slf4j
@ControllerAdvice
public class SqlInjectionExceptionHandler {
    @ExceptionHandler(SqlInjectionException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ResponseBody
    public Map<String, Object> handleSqlInjection(SqlInjectionException ex) {
        log.warn("SQL Injection detected. Field='{}', Value='{}'", ex.getFieldName(), ex.getFieldValue());

        Map<String, Object> result = new HashMap<>();
        result.put("code", "SQL_INJECTION_DETECTED");
        result.put("message", "Invalid input detected."); // never leak the raw value
        result.put("field", ex.getFieldName());
        return result;
    }
}
```

4. Apply `@Validated` on the controller parameter:

```java
package tsb.fontos.sample.controller.sqlinjection;

import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import tsb.fontos.sample.item.sqlinjection.SqlInjectionTestItem;

@RestController
public class SqlInjectionValidationTestController {
    @GetMapping("/sqlInjectionValidation")
    public SqlInjectionTestItem sqlInjectionValidation(@Validated SqlInjectionTestItem item) {
        return item;
    }
}
```

**How the check works**: `TSimpleSqlInjectionValidator` runs a single case-insensitive regex over target fields that blocks quotes, `OR`/`AND` tautologies, `UNION SELECT`/`DROP TABLE`/`DELETE FROM`/`SLEEP(`, SQL comment markers (`--`, `/* */`), `;`, and even `javascript:` (basic XSS). `AnnotationSqlInjectionValidator` walks the object graph reflectively (nested objects, collections, `String[]`) applying it to every `@SqlInjectionTarget` field.

> **Known limitation**: the regex blocks single **and** double quotes outright, so legitimate free-text input containing an apostrophe (e.g. a shipper name like `O'Brien Logistics`) will also be rejected. Reserve `@SqlInjectionTarget` for identifiers/codes (container numbers, vessel codes, search keywords), not free-text fields, or extend the validator if apostrophes must be allowed.

### 6.3 Password Policy Validator

Package: `tsb.fontos.core.security` (separate from the `sql` sub-package above).

Configure the policy externally:

```properties
fontos.security.password-policy.minimum-length=8
fontos.security.password-policy.upper-credit=1
fontos.security.password-policy.lower-credit=1
fontos.security.password-policy.digit-credit=1
fontos.security.password-policy.special-credit=1
```

Register the beans:

```java
package tsb.fontos.sample.security.passwordvalidation;

import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import tsb.fontos.core.security.TPasswordPolicy;
import tsb.fontos.core.security.TPasswordPolicyProperties;
import tsb.fontos.core.security.TPasswordPolicyProvider;
import tsb.fontos.core.security.TPasswordPolicyValidator;

@Configuration
@EnableConfigurationProperties(TPasswordPolicyProperties.class)
public class PasswordPolicyConfiguration {

    @Bean
    public TPasswordPolicyProvider passwordPolicyProvider(TPasswordPolicyProperties props) {
        return () -> new TPasswordPolicy(
                props.getMinimumLength(), props.getUpperCredit(),
                props.getLowerCredit(), props.getDigitCredit(), props.getSpecialCredit());
    }

    @Bean
    public TPasswordPolicyValidator passwordPolicyValidator(TPasswordPolicyProvider provider) {
        return new TPasswordPolicyValidator(provider);
    }
}
```

Use it:

```java
package tsb.fontos.sample.controller.passwordvalidation;

import java.util.Collections;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import tsb.fontos.core.security.TPasswordPolicyValidator;

@RestController
public class PasswordPolicyValidationTestController {
    @Autowired
    private TPasswordPolicyValidator passwordPolicyValidator;

    @GetMapping("/passwordValidator/{password}")
    public ResponseEntity<?> validatePassword(@PathVariable String password) {
        List<String> errors = passwordPolicyValidator.validateAll(password, "");
        if (errors.isEmpty()) {
            return ResponseEntity.ok(Collections.singletonMap("message", "Password is valid."));
        }
        return ResponseEntity.badRequest().body(Collections.singletonMap("errors", errors));
    }
}
```

`TPasswordPolicyProvider` is a strategy interface — implement it differently per tenant/module if password rules should vary by customer.

> **Security review recommended**: the framework's `PasswordEncoder` (`tsb.fontos.core.security`) supports two modes — an AES/CBC/PKCS5Padding mode that derives both the key **and** the IV from an MD5 digest of a configured secret, and a legacy time-based character-shuffling mode used when no key is configured. MD5-derived key/IV material and the legacy obfuscation scheme are both weak by modern standards. If this component is used for anything beyond backward-compatible decryption of legacy data, have your security lead evaluate replacing it with a vetted password hash (BCrypt/Argon2) for storage and a standard authenticated cipher (AES-GCM with a properly derived key/IV) for reversible encryption needs.

### 6.4 CATOS API URI / Access-Control Convention

This is an organizational convention (from `CATOS API Security for FONTOS-J2EE`) for naming and gating REST endpoints consistently across CATOS+/FONTOS services, independent of the JWT mechanics above.

**Naming rule**: `{Screen Title}` + action suffix (`/create`, `/update`, `/delete`); drop "List" from list-screen names; see restfulapi.net's resource-naming guide for edge cases.

**URI grouping (middle segment)**:

| Segment | Meaning |
|---|---|
| `/public` | No login required — must be explicitly whitelisted |
| `/common` | Login required, no per-role authorization (yet) — anticipate future authorization needs |
| `/vessel`, `/gate`, ... | Domain-grouped, authorization-checked endpoints |

**Full URI format**: `version/middle-segment/API-name`, e.g. `v1/common/portcode`, `v1/stowagebayplan`, `v1/trucklist`.

**Additional conventions**:
- Non-CRUD actions off a main screen: `{API Name}/control/{Control API Name}` — checked against `TB_BASICDEFINE_CONTROL.APINAME` and `TB_ACCESSROLE_CONTROL.VALUE`.
- Screen-scoped lookups: `{API Name}/searchXXX` — checked against `TB_ACCESS_ROLE.CINQUIRY`.
- A control action that needs a main screen's own service must go through a *control-specific* URI (e.g. `portcode/countrycode/search`), not the main screen's own `search` URI, to keep authorization scoping correct.

**Implementation**: add an `API_NAME` column to `TB_BASIC_DEFINE`; on each request, resolve `PGM_CODE`/`SUBMENU` from `TB_BASIC_DEFINE` via the URI's API name, then check the caller's permission against `TB_STAFF` → `TB_ACCESS_ROLE` / `TB_ACCESSROLE_CONTROL`. `TB_ACCESS_ROLE.CGROUP` is `'S'` for a staff-specific row or `'G'` for a group/role-level row (used when `TB_STAFF.USER_GROUP` is set).

### 6.5 Multi-Factor Authentication (TOTP/OTP)

TOTP-based MFA is layered on top of the ID/password login flow. First-time setup: after ID/PW succeeds, the server generates a **20-byte random secret key** tied to the account, renders it as an `otpauth://` QR code (`otpauth://totp/AccountName?secret=SecretKey&issuer=YourApp`), the user scans it with an authenticator app (Google/Microsoft Authenticator, Authy), and confirms with one initial OTP entry. From then on, every login requires ID/PW **and** the current 30-second OTP code from the app.

```
Login attempt → ID/PW check → (first time only: generate 20-byte secret → show QR → user scans
→ user enters initial OTP → verify → MFA enabled) → subsequent logins: app generates OTP →
user enters OTP → server verifies → login success
```

**Data model**: the common database gains an MFA-method / target-user flag alongside existing user/module-access tables, plus a per-user issued-secret-key table (20-byte random key, as above). MFA is being rolled out per module — at the time this deck was written it was confirmed for CATOS but still **under discussion** for MOST/PLUS, so don't assume it's mandatory across every module.

**QR code generation libraries** (different per client stack):
- Java/web backend: `implementation 'net.glxn:qrgen:1.4'`
- C#/WinForms client: `C1.Win.C1BarCode.2.dll` (ComponentOne barcode control)

**Reference/sample locations** (demo-only — no framework `OtpService`/`Hotp`/`Totp` class bodies are shown in the source deck, only these sample call sites): backend `tsb.fontos.webprototype.controller.otp.SampleTOtpController.java` / `OtpController.java`; React frontend `OptView.tsx`; C# client `Tsb.Fontos.Sample.Client.Web.Opt.MakeOptCodeView.cs`. If you need the actual key-generation/QR-generation/OTP-validation implementation, pull it from the framework source directly — this deck only documents the flow and demo call sites.

> **Security flag**: the source training materials for this feature include screenshots of a running demo UI with a real-looking plaintext Base32 TOTP secret and its `otpauth://` URL visible on-screen. Never reuse a demo secret from a screenshot as a real value, and if you need a demo screenshot for internal docs, regenerate it with an obviously fake placeholder secret.

### 6.6 SSO & Token Lifecycle Management

This subsection extends [6.1](#61-jwt-based-authentication--authorization) with the CATOS HOME → web-module single-sign-on flow and a DB-backed token lifecycle (issuance/revocation) layer. It also introduces the concrete SSO-specific framework types: `JwtSSOTokenProvider`, `JwtCookieProperties`, `TokenStorageManager` (`HybridTokenStorage` / `InMemoryTokenStorage` implementations), alongside the `JwtTokenProvider`/`JwtSecurityProperties`/`JwtAccessToken`/`TDefaultUser`/`TJwtTokenItem` types already covered in 6.1.

**Flow overview.** CATOS HOME (the native C# desktop shell) shows module icons (MOST, PLUS, EDI, ...) for whatever a user's account has access to, per the common `TB_STAFF`/module-access tables. Double-clicking a module icon generates a **fresh, ~1-minute-expiry JWT** and opens `http(s)://<module>/token/{token}` in the browser; the target module's backend validates the token, establishes its own session (an SSO cookie), and redirects to its own home page. **A new token is issued on every icon double-click** — tokens are not reused across launches. Each module must implement its own token-based auth-handling URL; there's no shared "gateway" endpoint across modules.

**1. Issuing the SSO token (CATOS HOME, C#)** — builds a short-lived (default 30s, configurable) JWT with `sub`=user ID, `iss`=`"CATOSHOME"`, `aud`=target module code (e.g. `MOST`), plus a random `jti` for replay protection:

```csharp
public string GenerateJwtToken()
{
    string issuerVal = "CATOSHOME";
    string userId = this.tbxUserID.Text;
    string securityKey = JwtConfig.SSO_Secret_Key; // [core-provided default secret key]
    string moudleName = this.tbxModuleName.Text;
    int expirationTime = 30;

    var payload = new {
        sub = userId, iss = issuerVal, aud = moudleName,
        iat = DateTimeOffset.UtcNow.ToUnixTimeSeconds(),
        exp = DateTimeOffset.UtcNow.AddSeconds(expirationTime).ToUnixTimeSeconds(),
        jti = JwtSSOKeyGeneration.GenerateRandomKey(20),
        username = "User Name", issuedtype = "SSO", roles = "ROLE_SSO" // roles are NOT passed by CATOSHOME
    };

    JWTHelper jwtHelper = new JWTHelper();
    return jwtHelper.makeJWTToken(securityKey, header, payload);
}
```

Required per-module inputs: `userId` (CATOS HOME login user), `moduleName` (registered in DB), `securityKey` (module-registered secret key, from DB), `expirationTime` (short-lived, 30s–1min recommended). Reference C# namespace layout: `Tsb.Fontos.Core.Security.{Audit, Authentication, Authorization, Encryption, Hash, Jwt}`, with `Jwt` containing `JwtConfig.cs` / `JWTHelper.cs` / `JwtSSOKeyGeneration.cs`.

**2. Backend configuration** (`application.properties`) — separate expiry/secret for access vs. refresh vs. SSO tokens, plus per-cookie settings:

```properties
fontos.jwt.security.access.expiry-seconds=3600
fontos.jwt.security.access.secret-key=${JWT_ACCESS_SECRET_KEY}
fontos.jwt.security.access.refresh-threshold-seconds=1200
fontos.jwt.security.refresh.expiry-seconds=14400
fontos.jwt.security.sso.expiry-seconds=60
fontos.jwt.security.sso.secret-key=${JWT_SSO_SECRET_KEY}

fontos.jwt.cookie.refresh.name=REFRESH_TOKEN
fontos.jwt.cookie.refresh.path=/jwt
fontos.jwt.cookie.refresh.max-age=43200
fontos.jwt.cookie.refresh.secure=true
fontos.jwt.cookie.refresh.http-only=false
fontos.jwt.cookie.refresh.same-site=Lax

fontos.jwt.cookie.sso.name=SSO_TOKEN
fontos.jwt.cookie.sso.path=/
fontos.jwt.cookie.sso.max-age=60
fontos.jwt.cookie.sso.secure=true
fontos.jwt.cookie.sso.http-only=false
fontos.jwt.cookie.sso.same-site=Lax
```

> **Security flags**: (1) the source training slide hardcodes literal secret-key values for both `...access.secret-key` and `...sso.secret-key` directly in `application.properties` — never do this; externalize via environment variable, secrets manager, or Jasypt-encrypted property, as shown above with placeholders. (2) `http-only=false` on both cookies is called out **in the source material itself** as a deliberate (but insecure) demo setting, presumably so client-side JS can read the cookie — `http-only=true` is the secure default; treat `false` as a documented trade-off, not something to carry into production without a specific reason.

**3. Refresh token generation** — clones the access token's claims with a new issuance/expiry and a new `jti`:

```java
package tsb.fontos.webprototype.security.jwt;

import java.util.Date;
import java.util.concurrent.TimeUnit;

import tsb.fontos.core.security.JwtSecurityProperties;
import tsb.fontos.security.authentication.JwtAccessToken;
import tsb.fontos.security.authentication.JwtRestApiAuthenticationToken;
import tsb.fontos.security.filter.JwtRefreshHandler;
import tsb.fontos.security.jwt.JwtSSOKeyGeneration;
import tsb.fontos.security.jwt.JwtTokenProvider;

public class SimpleJwtRefreshHandler implements JwtRefreshHandler {
    private JwtTokenProvider jwtTokenProvider;
    private JwtSecurityProperties jwtSecurityProperties;

    @Override
    public String generateToken(JwtAccessToken token) {
        JwtRestApiAuthenticationToken newtoken = new JwtRestApiAuthenticationToken();
        newtoken.setSubject(token.getSubject());
        newtoken.setIssuer(token.getIssuer());
        newtoken.setAudience(token.getAudience());
        newtoken.setIssuedAt(new Date());
        newtoken.setExpire(new Date(System.currentTimeMillis()
                + TimeUnit.SECONDS.toMillis(jwtSecurityProperties.getAccess().getExpirySeconds())));
        newtoken.setId(JwtSSOKeyGeneration.generateRandomKey(20));
        newtoken.setUsername(token.getUsername());
        newtoken.setIssuedType(token.getIssuedType());
        newtoken.setTerminalCode(token.getTerminalCode());
        newtoken.setRoles(token.getRoles());
        return this.jwtTokenProvider.createToken(newtoken);
    }
}
```

**4. User validation and token generation** — a sample `TokenUserValidator` (DB lookup stub) and `AbstractTokenGenerator` subclass that builds both access + refresh tokens for a validated user:

```java
package tsb.fontos.webprototype.security.jwt;

import java.util.Arrays;

import org.springframework.stereotype.Component;

import tsb.fontos.security.constant.TRoleConstants;
import tsb.fontos.security.jwt.mng.TokenUserValidator;
import tsb.fontos.security.profile.TDefaultUser;

@Component
public class SampleTokenUserValidator implements TokenUserValidator {
    @Override
    public TDefaultUser validateUser(String userId) {
        SampleUser user = new SampleUser(); // extends TDefaultUser, adds terminalCode
        user.setUserId(userId);
        user.setTerminalCode("TerminalCode");
        user.setRoles(Arrays.asList(TRoleConstants.ROLE_USER, TRoleConstants.ROLE_MANAGER));
        return user;
    }
    // overload: validateUser(TUserAuthRequest authRequest) — same shape, from posted credentials
}
```

```java
package tsb.fontos.webprototype.security.jwt;

import java.util.Date;
import java.util.concurrent.TimeUnit;

import tsb.fontos.core.security.JwtSecurityProperties;
import tsb.fontos.security.authentication.JwtAccessToken;
import tsb.fontos.security.authentication.JwtRestApiAuthenticationToken;
import tsb.fontos.security.authentication.TJwtTokenItem;
import tsb.fontos.security.jwt.JwtSSOKeyGeneration;
import tsb.fontos.security.jwt.JwtTokenProvider;
import tsb.fontos.security.jwt.mng.AbstractTokenGenerator;
import tsb.fontos.security.profile.TDefaultUser;

public class SampleTokenGenerator extends AbstractTokenGenerator {
    private JwtTokenProvider jwtTokenProvider;
    private JwtSecurityProperties jwtSecurityProperties;

    @Override
    public TJwtTokenItem createToken(TDefaultUser user) {
        SampleUser customUser = (SampleUser) user;
        JwtRestApiAuthenticationToken accessToken = new JwtRestApiAuthenticationToken();
        accessToken.setIssuer("TSB Issuer");
        accessToken.setSubject(user.getUserId());
        accessToken.setAudience("API Web SVR Audience");
        accessToken.setId(JwtSSOKeyGeneration.generateRandomKey(20));
        accessToken.setIssuedType(JwtAccessToken.ISSUED_TYPE_USER);
        accessToken.setTerminalCode(customUser.getTerminalCode());
        accessToken.setExpire(new Date(System.currentTimeMillis()
                + TimeUnit.SECONDS.toMillis(jwtSecurityProperties.getAccess().getExpirySeconds())));
        accessToken.setRoles(user.getRoles());
        String accessTokenStr = jwtTokenProvider.createToken(accessToken);

        JwtRestApiAuthenticationToken refreshToken = accessToken.clone();
        refreshToken.setIssuedAt(new Date());
        refreshToken.setExpire(new Date(System.currentTimeMillis()
                + TimeUnit.SECONDS.toMillis(jwtSecurityProperties.getRefresh().getExpirySeconds())));
        String refreshTokenStr = jwtTokenProvider.createToken(refreshToken);

        TJwtTokenItem item = new TJwtTokenItem();
        item.setUserName(user.getUserId());
        item.setTokenType(JwtAccessToken.BEARER_TYPE);
        item.setAccessToken(accessTokenStr);
        item.setRefreshToken(refreshTokenStr);
        return item;
    }
}
```

**5. Wiring it together** — two Spring `@Bean` configuration classes, one for plain SSO, one when DB-backed token lifecycle management is also enabled (the latter adds a `tokenLifecycleManager` to every filter):

```java
package tsb.fontos.webprototype.security.jwt;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import tsb.fontos.core.security.JwtCookieProperties;
import tsb.fontos.core.security.JwtSecurityProperties;
import tsb.fontos.security.jwt.JwtSSOTokenProvider;
import tsb.fontos.security.jwt.JwtTokenProvider;
import tsb.fontos.security.jwt.mng.RefreshTokenCacheManager;
import tsb.fontos.security.jwt.mng.RefreshTokenCookieCacheManager;
import tsb.fontos.security.jwt.mng.SSOTokenCacheManager;
import tsb.fontos.security.jwt.mng.SSOTokenCookieCacheManager;
import tsb.fontos.security.jwt.mng.SimpleJwtAuthenticationManager;
import tsb.fontos.security.jwt.mng.TokenUserValidator;
import tsb.fontos.security.web.configuration.JwtRefreshAuthenticationAndAuthorizationSSOSecurityConfiguration;

@Configuration
public class WebJwtSecuritySSOBeanConfiguration {
    @Bean
    public JwtRefreshAuthenticationAndAuthorizationSSOSecurityConfiguration webSecurityConfigure(
            JwtTokenProvider jwtTokenProvider, JwtSecurityProperties jwtSecurityProperties) {
        return new JwtRefreshAuthenticationAndAuthorizationSSOSecurityConfiguration(jwtTokenProvider, jwtSecurityProperties);
    }
    @Bean
    public JwtTokenProvider jwtTokenProvider(JwtSecurityProperties p) {
        return new JwtTokenProvider(p.getAccess().getSecretKey(), p.getAccess().getExpirySeconds());
    }
    @Bean
    public JwtSSOTokenProvider jwtSSOTokenProvider(JwtSecurityProperties p) {
        return new JwtSSOTokenProvider(p.getSso().getSecretKey(), p.getSso().getExpirySeconds());
    }
    @Bean
    public SimpleJwtAuthenticationManager jwtTokenService(JwtTokenProvider jwtTokenProvider,
            JwtSSOTokenProvider jwtSSOTokenProvider, TokenUserValidator userValidator,
            SampleTokenGenerator tokenGenerator, SSOTokenCacheManager ssoCache, RefreshTokenCacheManager refreshCache) {
        SimpleJwtAuthenticationManager mgr = new SimpleJwtAuthenticationManager(jwtTokenProvider, userValidator, tokenGenerator);
        mgr.setJwtSSOTokenProvider(jwtSSOTokenProvider);
        mgr.setSSoTokenCacheManager(ssoCache);
        mgr.setRefreshTokenCacheManager(refreshCache);
        return mgr;
    }
    @Bean
    public SSOTokenCacheManager ssoTokenCacheManager(JwtCookieProperties p) { return new SSOTokenCookieCacheManager(p); }
    @Bean
    public RefreshTokenCacheManager refreshTokenCacheManager(JwtCookieProperties p) { return new RefreshTokenCookieCacheManager(p); }
}
```

The token-lifecycle-managed variant (`WebJwtSecurityTokenMngBeanConfiguration`) additionally wires a `TokenStorageManager` bean — `HybridTokenStorage` (DB-backed, via `SampleTokenStoreService`, with an in-process cache size e.g. `1000`) or `InMemoryTokenStorage` (no DB) — and passes it into both `jwtTokenService` and the auth/refresh filters.

**6. Security filter chain** — same shape as [6.1](#61-jwt-based-authentication--authorization)'s filter registration, gating everything except `/jwt/**` endpoints and applying the `UriPatternAuthorizationManagerManagerFilter` for action-level checks:

```java
// excerpt from the same configure(HttpSecurity) method shown in §6.1 — same imports apply
.antMatchers("/**/auth/hello**").authenticated()
.antMatchers(HttpMethod.GET, "/jwt/**").permitAll()
.antMatchers(HttpMethod.POST, "/jwt/**").permitAll()
.anyRequest().authenticated()
.and()
.addFilterBefore(apiJwtAuthenticateFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class)
.addFilterAfter(apiJwtRefreshFilter(jwtTokenProvider), ApiJwtAuthenticateFilter.class)
.addFilterAfter(new UriPatternAuthorizationManagerManagerFilter(
        uriPatternAuthorizationManager()
            .addMapping("/**/api/**/inquiry**", "INQUIRY")
            .addMapping("/**/api/**/save**", "SAVE")
            .addMapping("/**/api/**/insert**", "INSERT")
            .addMapping("/**/api/**/delete**", "DELETE")
            .addControlMapping("/**/api/**/control/**")),
        FilterSecurityInterceptor.class)
.exceptionHandling().authenticationEntryPoint(jwtAuthenticationEntryPoint())
    .accessDeniedHandler(apiAccessDeniedHandler());
```

Excluded paths (login/sso/refresh/logout) are registered once via a shared `ExcludedPathMatcher` and reused by both the auth filter and the refresh filter.

**CORS** must allow credentials for the SSO login/refresh endpoints so the browser will accept the Set-Cookie response:

```java
// excerpt from a CorsConfigurationSource @Bean method
import java.util.Arrays;

import org.springframework.web.cors.CorsConfiguration;
// source : org.springframework.web.cors.UrlBasedCorsConfigurationSource

CorsConfiguration specificConfig = new CorsConfiguration();
specificConfig.setAllowedOrigins(Arrays.asList("http://localhost:3000"));
specificConfig.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE"));
specificConfig.setAllowCredentials(true); // required for cookie issuance on these specific paths
source.registerCorsConfiguration("/jwt/login", specificConfig);
source.registerCorsConfiguration("/jwt/logout", specificConfig);
source.registerCorsConfiguration("**/catoshome/sso/access/**", specificConfig);
source.registerCorsConfiguration("**/jwt/ssotokenbyusernameByPost", specificConfig);
```

**7. Accepting the SSO token from CATOS HOME** — validates issuer/audience/user, then caches it (as a cookie/session) and redirects into the app:

```java
// excerpt from a Controller in tsb.fontos.webprototype.controller.login
import java.io.IOException;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.MalformedJwtException;
import io.jsonwebtoken.SignatureException;
import io.jsonwebtoken.UnsupportedJwtException;
import tsb.fontos.security.authentication.JwtAccessToken;
import tsb.fontos.security.jwt.NotSupportSSOJwtToken;

@GetMapping("/sso/catoshome/access/{token}")
public void ssoLoginAccessFromCatosHome(HttpServletRequest req, HttpServletResponse res, @PathVariable String token) throws IOException {
    try {
        JwtAccessToken tokenInfo = jwtSSOTokenProvider.getTokenInfo(token);
        if (!checkIssuedType.equals(tokenInfo.getIssuedType())) {
            throw new NotSupportSSOJwtToken("IssuedType is not supported: " + tokenInfo.getIssuedType());
        }
        // ... check issuer / audience / user ID as appropriate for this module
        jwtTokenService.storeCachedSSOToken(req, res, tokenInfo.getSubject(), token);
        res.sendRedirect(req.getContextPath() + "/index.html");
    } catch (NotSupportSSOJwtToken | ExpiredJwtException | UnsupportedJwtException | MalformedJwtException | SignatureException | IllegalArgumentException e) {
        jwtTokenService.removeCachedSSOToken(req, res);
        res.sendRedirect(req.getContextPath() + "/index.html");
    }
}
```

Cookie-based auto-login (`/jwt/cached/login`) checks the SSO cookie first and falls back to the refresh cookie if both are present. Standard login/logout endpoints (`/jwt/login`, `/jwt/logout`) round out the controller surface. On the frontend, an `ILoginStrategy` implementation (e.g. `SampleLoginStrategyByHttpOnlyCookie.ts`) calls the cached-token endpoint on app start, verifies the token, and — if valid — requests a fresh access token and stores it for subsequent API calls.

**8. Token lifecycle (issuance/revocation) storage** — when `TokenStorageManager` is DB-backed, two tables track issued and revoked tokens:

```sql
CREATE TABLE TEST_EA_ISSUED_TOKENS (
    JTI VARCHAR2(128) PRIMARY KEY, USER_ID VARCHAR2(128) NOT NULL, MODULE VARCHAR2(64),
    TOKEN VARCHAR2(2000), ISSUED_TYPE VARCHAR2(32),
    ISSUED_AT TIMESTAMP DEFAULT CURRENT_TIMESTAMP, EXPIRES_AT TIMESTAMP, CLIENT_INFO VARCHAR2(1000)
);
CREATE TABLE TEST_EA_REVOKED_TOKENS (
    JTI VARCHAR2(128) PRIMARY KEY, REVOKED_AT TIMESTAMP DEFAULT CURRENT_TIMESTAMP, REASON VARCHAR2(256)
);
```

Revocation reasons used in practice: `LOGOUT`, `ADMIN_KICK`, `ABUSE`, `LOCATION_CHANGE`. `SampleTokenStoreService`/`SampleTokenStoreDao` (MyBatis) provide `insertIssuedToken`/`updateIssuedToken`/`selectIssuedTokenByJti`/`deleteIssuedTokenByJti`/`insertRevokedToken`/`isTokenRevoked`/`selectAllIssuedToken`/`selectAllRevokedToken`.

**Method reference** (`TokenAuthenticationManager`-shaped interface, concrete impl shown as `SimpleJwtAuthenticationManager`):

| Method | Purpose |
|---|---|
| `loginByUserInfo(req, res, TUserAuthRequest)` | Authenticate by ID/password, issue JWT |
| `loginBySsoToken(req, res, String ssoToken)` | Authenticate + issue JWT via an externally-supplied SSO token |
| `loginByCachedToken(req, res)` | Auto-login: tries the cached SSO token first, falls back to the cached refresh token |
| `logOut(req, res, TDefaultUser)` | Revoke access token, delete refresh token, clear caches |
| `createToken(TDefaultUser)` | Build a `TJwtTokenItem` (access + refresh) for a validated user |
| `validateToken(String)` | Validate a JWT |
| `store/remove/retrieveCachedSSOToken(...)`, `store/remove/retrieveCachedRefreshToken(...)` | Cookie-cache management for each token type |

---

## 7. WebSocket / STOMP Messaging

### 7.1 Concepts

WebSocket gives a persistent, bidirectional TCP-based connection (HTML5 standard) so the server can push data to clients without waiting for a request — unlike plain HTTP request/response. **STOMP** (Simple Text Oriented Messaging Protocol) runs on top of WebSocket and standardizes the message format/commands (`CONNECT`, `CONNECTED`, `SUBSCRIBE`, `SEND`, `MESSAGE`, ...), letting you use `@MessageMapping`-style endpoints instead of a raw `WebSocketHandler`.

```
>>> SUBSCRIBE
id:sub-0
destination:/sub/chat/

>>> SEND
destination:/pub/message
content-length:50

Message sent: {channelId: '1', userId: '1', message: 'hello'}

<<< MESSAGE
destination:/sub/chat/1
content-type:application/json
content-length:43
```

Publish (`pub`) traffic goes to `/app`-style destinations routed to `@MessageMapping` handler methods; subscribe (`sub`) traffic goes through the SimpleBroker, which tracks subscriptions in memory and fans out `MESSAGE` frames.

### 7.2 Server Configuration

The framework base class lives in `tsb.fontos.net.ws.config.BaseStompWebSocketMessageBrokerConfigurer` (`implements WebSocketMessageBrokerConfigurer`). Extend it and register your STOMP endpoint:

```java
package tsb.fontos.sample.ws.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

import tsb.fontos.net.ws.config.BaseStompWebSocketMessageBrokerConfigurer;
import tsb.fontos.net.ws.config.StompWebSocketProperties;
import tsb.fontos.security.jwt.JwtTokenProvider;

@Configuration
@Import(StompWebSocketProperties.class)
@EnableWebSocketMessageBroker
public class SampleStompWebSocketConfiguration extends BaseStompWebSocketMessageBrokerConfigurer {

    @Autowired private StompWebSocketProperties stompWebSocketProperties;
    @Autowired private JwtTokenProvider jwtTokenProvider;

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        super.configureMessageBroker(registry); // broker/app/user destination prefixes from properties
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint(stompWebSocketProperties.getEndpoint())
                .setAllowedOriginPatterns(stompWebSocketProperties.getAllowedOriginPatternsArray())
                .withSockJS();
    }
}
```

Properties (`@ConfigurationProperties(prefix = "stomp.ws")`):

```properties
stomp.ws.message-size-limit=65536
stomp.ws.send-time-limit=1000000
stomp.ws.send-buffer-size-limit=524288
stomp.ws.endpoint=/ws
stomp.ws.simple-broker=/sub,/user
stomp.ws.application-destination-prefixes=/pub
stomp.ws.user-destination-prefix=/user
stomp.ws.allowed-origin-patterns=*
```

Front-end (React + `@stomp/stompjs`):

```json
{ "dependencies": { "@stomp/stompjs": "^7.0.0" } }
```

```ts
private client = new Client({
  brokerURL: "ws://localhost:8080/ws",
  reconnectDelay: 5000,
  connectHeaders: { userId: "userId" },
  onConnect: () => { this.subscribeToChannel(this.state.channel); },
  onDisconnect: () => { console.log("Disconnected"); },
});
```

### 7.3 Securing WebSocket/STOMP Connections with JWT

There are two distinct security checkpoints, both pluggable via strategy interfaces on the framework's base classes:

1. **Handshake-level** (`HandshakeInterceptor` via `BaseHandshakeAuthInterceptor` / `IWebSocketHandshakeSecurityHandler`) — runs once, during the HTTP→WebSocket upgrade, before STOMP even starts.
2. **STOMP channel-level** (`BaseSecurityChannelInterceptor` / `IWebSocketSecurityHandler`) — runs per STOMP frame (`CONNECT`, `SUBSCRIBE`, `SEND`, ...).

```java
package tsb.fontos.sample.ws.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

import tsb.fontos.net.ws.config.BaseStompWebSocketMessageBrokerConfigurer;
import tsb.fontos.net.ws.config.StompWebSocketProperties;
import tsb.fontos.net.ws.handshake.SampleWebSocketHandshakeAuthInterceptor;
import tsb.fontos.sample.ws.security.SampleStompJwtAuthenticationHandler;
import tsb.fontos.sample.ws.security.SampleStompSubProtocolErrorHandler;
import tsb.fontos.sample.ws.security.SampleStompWsSecurityChannelInterceptor;
import tsb.fontos.sample.ws.security.SampleWebSocketHandshakeSecurityHandler;

@Configuration
@Import(StompWebSocketProperties.class)
@EnableWebSocketMessageBroker
public class SampleStompWebSocketConfiguration extends BaseStompWebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        SampleWebSocketHandshakeAuthInterceptor handshakeInterceptor =
            new SampleWebSocketHandshakeAuthInterceptor(
                new SampleWebSocketHandshakeSecurityHandler(jwtTokenProvider));

        registry.addEndpoint(stompWebSocketProperties.getEndpoint())
                .addInterceptors(handshakeInterceptor)
                .setAllowedOriginPatterns(stompWebSocketProperties.getAllowedOriginPatternsArray())
                .withSockJS();

        registry.setErrorHandler(new SampleStompSubProtocolErrorHandler());
    }

    @Bean
    public SampleStompWsSecurityChannelInterceptor securityChannelInterceptor() {
        SampleStompWsSecurityChannelInterceptor interceptor = new SampleStompWsSecurityChannelInterceptor();
        interceptor.setConnectSecurityHandler(new SampleStompJwtAuthenticationHandler(jwtTokenProvider));
        return interceptor;
    }
}
```

Handshake-level check (rejects the upgrade outright if the token is missing/invalid):

```java
package tsb.fontos.sample.ws.security;

import java.util.Map;

import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.http.server.ServletServerHttpRequest;
import org.springframework.util.StringUtils;
import org.springframework.web.socket.WebSocketHandler;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import tsb.fontos.net.ws.constant.WebSocketConstants;
import tsb.fontos.net.ws.security.IWebSocketHandshakeSecurityHandler;
import tsb.fontos.net.ws.util.JwtTokenUtil;
import tsb.fontos.security.authentication.JwtTokenItem;
import tsb.fontos.security.jwt.JwtTokenProvider;

@Slf4j
@RequiredArgsConstructor
public class SampleWebSocketHandshakeSecurityHandler implements IWebSocketHandshakeSecurityHandler {
    private final JwtTokenProvider jwtTokenProvider;

    @Override
    public void checkSecurity(ServerHttpRequest request, ServerHttpResponse response,
                               WebSocketHandler handler, Map<String, Object> attributes) throws Exception {
        String authHeader = "";
        if (request instanceof ServletServerHttpRequest) {
            authHeader = ((ServletServerHttpRequest) request).getServletRequest()
                .getParameter(WebSocketConstants.JWT_TOKEN_PARAM_KEY);
        }
        if (!StringUtils.hasText(authHeader)) throw new Exception("Token missing");

        JwtTokenItem tokenItem = JwtTokenUtil.resolveToken(authHeader);
        if (!jwtTokenProvider.validateToken(tokenItem.getToken())) {
            throw new Exception("Invalid or expired token");
        }
    }
}
```

STOMP CONNECT-level check (registers the authenticated `Principal` for the session):

```java
package tsb.fontos.sample.ws.security;

import org.springframework.messaging.simp.stomp.StompCommand;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.util.StringUtils;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import tsb.fontos.net.ws.security.IWebSocketSecurityHandler;
import tsb.fontos.net.ws.util.JwtTokenUtil;
import tsb.fontos.security.authentication.JwtAccessToken;
import tsb.fontos.security.authentication.JwtTokenItem;
import tsb.fontos.security.jwt.JwtTokenProvider;

@Slf4j
@RequiredArgsConstructor
public class SampleStompJwtAuthenticationHandler implements IWebSocketSecurityHandler {
    private final JwtTokenProvider jwtTokenProvider;

    @Override
    public boolean checkSecurity(StompHeaderAccessor headers) {
        StompCommand command = headers.getCommand();
        if (StompCommand.CONNECT.equals(command)) {
            validateAndRegisterUser(headers);
        } else if (!StompCommand.DISCONNECT.equals(command)) {
            validatePrincipal(headers); // SUBSCRIBE/SEND must already carry a principal
        }
        return true;
    }

    private void validateAndRegisterUser(StompHeaderAccessor headers) {
        String rawHeader = headers.getFirstNativeHeader(JwtAccessToken.REQUEST_HEADER_AUTH);
        if (!StringUtils.hasText(rawHeader)) throw new AccessDeniedException("Authorization header is missing");

        JwtTokenItem tokenItem = JwtTokenUtil.resolveToken(rawHeader);
        if (!jwtTokenProvider.validateToken(tokenItem.getToken())) {
            throw new AccessDeniedException("Token is invalid or expired");
        }
        JwtAccessToken tokenInfo = jwtTokenProvider.getTokenInfo(tokenItem.getToken());
        headers.setUser(new SampleStompUserPrincipal(tokenInfo.getUsername(),
                headers.getFirstNativeHeader("userId"), headers.getFirstNativeHeader("roomId")));
    }
}
```

Client side, the token is passed both as a handshake query parameter and as a STOMP `CONNECT` header:

```ts
const socket = new SockJS(baseURL + this.END_POINT + "?token=" + remoteConfig.accessToken);
this._stompClient = Stomp.over(socket);
this._stompClient.connect(
  { Authorization: `${remoteConfig.accessToken}`, userId: this.state.userId, roomId: this.state.roomId },
  (frame) => { this.setState({ isConnected: true }); },
  (error) => { console.error("STOMP connection failed:", error); }
);
```

**Security responsibility summary:**

| Stage | Responsibility | Component |
|---|---|---|
| Handshake | JWT validation, allow/deny the upgrade | `*WebSocketHandshakeSecurityHandler` |
| Connect | JWT auth, principal registration | `*StompJwtAuthenticationHandler` |
| Subscribe | Topic access check | `SessionSubscribeEvent` listener |
| Send | Message-send permission check | `*StompJwtAuthenticationHandler` |
| Disconnect | Session/resource cleanup | `SessionDisconnectEvent` listener |
| Connection lifecycle | Connect/disconnect bookkeeping | `*WebSocketHandlerDecorator` |

### 7.4 Controllers and Message Mapping

Extend `BaseStompWebSocketController` (from `tsb.fontos.net`), which exposes a ready-to-use `SimpMessagingTemplate`:

```java
// tsb.fontos.net.ws.controller.BaseStompWebSocketController
package tsb.fontos.net.ws.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.simp.SimpMessagingTemplate;

public class BaseStompWebSocketController {
    @Autowired private SimpMessagingTemplate simpMessagingTemplate;
    protected SimpMessagingTemplate GetSimpMessagingTemplate() { return simpMessagingTemplate; }
}
```

```java
package tsb.fontos.sample.ws.controller;

import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.web.bind.annotation.RestController;

import lombok.extern.slf4j.Slf4j;
import tsb.fontos.net.ws.controller.BaseStompWebSocketController;
import tsb.fontos.sample.ws.item.BoundaryItem;
import tsb.fontos.sample.ws.item.YBayBoundaryResponse;

@RestController
@Slf4j
public class WebSocketTestController extends BaseStompWebSocketController {

    @MessageMapping("/BoundaryItem")
    public void boundaryItem(BoundaryItem boundaryItem, SimpMessageHeaderAccessor accessor) {
        YBayBoundaryResponse response = new YBayBoundaryResponse();
        response.setType("boundaryItem");
        response.setBoundaryItem(boundaryItem);
        this.GetSimpMessagingTemplate().convertAndSend("/sub/ybay/" + boundaryItem.getChannel(), response);
    }
}
```

New-connection and subscribe events are observed with `@EventListener`:

```java
// excerpt from a @Component/@Configuration class (same package as WebSocketTestController)
import org.springframework.context.event.EventListener;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.web.socket.messaging.SessionConnectEvent;

@EventListener
public synchronized void newConnectionCheck(SessionConnectEvent e) {
    StompHeaderAccessor accessor = StompHeaderAccessor.wrap(e.getMessage());
    log.debug("new connection: sessionId={}, userId={}",
              accessor.getSessionId(), accessor.getFirstNativeHeader("userId"));
}
```

### 7.5 Pub/Sub Patterns

**Channel-based** — clients subscribe to a shared topic keyed by an arbitrary channel id, and the server fans updates out to that topic:

```java
// excerpt from a controller extending BaseStompWebSocketController
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;

import tsb.fontos.sample.ws.item.ContainerItem;
import tsb.fontos.sample.ws.item.YBayContainerResponse;

@MessageMapping("/ContainerItem")
public void containerItem(ContainerItem item, SimpMessageHeaderAccessor accessor) {
    YBayContainerResponse response = new YBayContainerResponse();
    response.setType("containerItem");
    response.setContainerItem(item);
    this.GetSimpMessagingTemplate().convertAndSend("/sub/ybay/" + item.getChannel(), response);
}
```

**User-targeted** — `convertAndSendToUser(...)` routes to `/user/{destination-prefix}/...` (prefix set via `setUserDestinationPrefix`/`stomp.ws.user-destination-prefix`); pass a `MessageHeaders` built from the target `sessionId` when addressing a specific session rather than a logical username:

```java
// excerpt from the same controller as containerItem() above
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.messaging.simp.SimpMessageType;

private MessageHeaders createHeaders(String sessionId) {
    SimpMessageHeaderAccessor accessor = SimpMessageHeaderAccessor.create(SimpMessageType.MESSAGE);
    if (sessionId != null) accessor.setSessionId(sessionId);
    accessor.setLeaveMutable(true);
    return accessor.getMessageHeaders();
}

this.GetSimpMessagingTemplate().convertAndSendToUser(
    accessor.getSessionId(), "/sub/ybay/" + channelId, response, createHeaders(accessor.getSessionId()));
```

On new subscription, it's common to replay any buffered state for that channel so a late-joining client isn't starting from a blank state (`SessionSubscribeEvent` listener → look up existing items for the channel → `convertAndSend` them back).

---

## 8. TCP / Socket Messaging

### 8.1 Netty-based TCP Client (`tsb.fontos.net`)

Used to talk to legacy terminal message servers (the sample naming — `C3IT`, `XIS` — reflects real upstream systems FONTOS integrates with) over a line/tag-delimited raw TCP protocol, with Netty underneath.

```
TCP Server(s): C3IT / XIS / ...
        │  (raw socket)
        ▼
┌─────────────────────────── TCP Client [FONTOS] ───────────────────────────┐
│ Inbound → Inbound Channel Handler → Receive Message Handler → Message Queue│
│    Message Queue ⇄ Event Thread (queue polling) / Event Thread (sync wait) │
│    → Message Parser → Async Receive | Sync Receive                        │
│    (reply) → Message Builder (object→string) → Send Message Handler       │
│    → Outbound Channel Handler → Outbound                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

Key classes (`tsb.fontos.net.msg.*`):

- `SimpleNetMessageClient` (`extends AbstractNetMessageClientOperatoins`) — public API: `start()/stop()/shutdown()`, `sendMessage(msg, syncType, waitTimeout, msgObjectTypes...)`, convenience `sendSyncMessage`/`sendASyncMessage`. Sync sends block on a `SimpleCallableSyncMessage` keyed by a generated sync id until the matching response arrives.
- `NettyTcpClient` (`implements MessageChannelClient`) — Netty `Bootstrap` + `NioEventLoopGroup`, `SO_KEEPALIVE`, auto-reconnect via a `Timer`/`TimerTask` on `reconnectionTryInterval`.
- `NetMessageClientFactory.build(NetMessgeClinetOperationSetting)` — static factory that assembles a `SimpleNetMessageClient` from your handlers.
- `NetMessageClientConfig` — `name/host/port/syncMessageWaitTimeout(ms, default 5000)/reconnectionTryInterval(ms)/tls`; defaults to `127.0.0.1:1004`.

### 8.2 Wiring a Client End to End

1. Handle every inbound message (raw, unfiltered) — implement `NetInboundMessageHandler`:

```java
package tsb.fontos.sample.tcpclient.c3it.msg.handler;

import lombok.extern.slf4j.Slf4j;
import tsb.fontos.net.msg.NetInboundMessageHandler;

@Slf4j
public class CatosReceiveMessageHandler implements NetInboundMessageHandler {
    @Override
    public void MessageProcessHandler(Object receiveMsg) {
        if (receiveMsg != null) log.debug(">> received: {}", receiveMsg);
    }
}
```

2. Handle only "interesting" (already object-mapped) messages — implement `NetReceiveMessageProcessHandler`:

```java
package tsb.fontos.sample.tcpclient.c3it.msg.handler;

import lombok.extern.slf4j.Slf4j;
import tsb.fontos.net.msg.NetReceiveMessageProcessHandler;

@Slf4j
public class CatosReceiveMessageProcessHandler implements NetReceiveMessageProcessHandler {
    @Override
    public void MessageProcess(Object sender) {
        if (sender != null) log.debug("[MessageProcess] {}", sender);
    }
}
```

3. Build the login/authentication message sent on connect — extend `BaseNetSecurityMessageHandler`:

```java
package tsb.fontos.sample.tcpclient.c3it.msg.handler;

import tsb.fontos.net.msg.BaseNetSecurityMessageHandler;
import tsb.fontos.net.msg.NetMessageBroker;
import tsb.fontos.sample.tcpclient.c3it.msg.objects.MsgC21;

public class CatosNetSecurityMessageHandler extends BaseNetSecurityMessageHandler {
    public CatosNetSecurityMessageHandler(NetMessageBroker messageBroker) { super(messageBroker); }

    @Override
    public String GetLoginMessage() {
        MsgC21 loginMsg = new MsgC21();
        loginMsg.setEquipmentNo("^M01");
        loginMsg.setUserID("<terminal-user-id>");
        loginMsg.setPassword("<per-environment secret — load from vault/env, never hardcode>");
        loginMsg.setDBMSString("<per-environment DB string>");
        loginMsg.setMMode("CN");
        loginMsg.setLDAPAuthority("N");
        return this.getMessageBroker().SendBroker(loginMsg).toString();
    }
}
```

> The original slide deck's example hardcodes a sample user id, an encoded password string, and a DB connection string directly in this class. **Do not copy that pattern.** Load per-environment credentials from configuration/secret storage (e.g. Jasypt-encrypted properties or a vault), never as string literals in source.

4. Handle heartbeat/signal replies — extend `BaseNetSignalMessageHandler`; 5. Observe connect/disconnect — implement `MessageChannelStatusListener`; 6. Configure the endpoint via `@ConfigurationProperties`:

```java
package tsb.fontos.sample.tcpclient.c3it.msg.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

import lombok.Getter;
import lombok.Setter;
import tsb.fontos.net.msg.NetMessageClientConfig;

@ConfigurationProperties("msg.client")
@Getter @Setter
public class CatosMessageClientConfig {
    private NetMessageClientConfig catos;
    private NetMessageClientConfig xis;
}
```

```properties
msg.client.catos.name=C3IT
msg.client.catos.host=127.0.0.1
msg.client.catos.port=6778
msg.client.catos.syncMessageWaitTimeout=5000
msg.client.catos.reconnectionTryInterval=5
```

7. Assemble and start the client (typically in a `@Configuration`/`@Component` bean method):

```java
package tsb.fontos.sample.tcpclient.c3it.msg.config;

import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import tsb.fontos.net.msg.IBaseMsgObjectSend;
import tsb.fontos.net.msg.MessageHandleStore;
import tsb.fontos.net.msg.NetMessageClientFactory;
import tsb.fontos.net.msg.NetMessageClientOperatoins;
import tsb.fontos.net.msg.NetMessgeClinetOperationSetting;
import tsb.fontos.net.msg.TNetMessageClientOperatoins;
import tsb.fontos.sample.tcpclient.c3it.msg.broker.C3ITMessageBroker;
import tsb.fontos.sample.tcpclient.c3it.msg.handler.CatosMessageClientChannelStatusListener;
import tsb.fontos.sample.tcpclient.c3it.msg.handler.CatosNetSecurityMessageHandler;
import tsb.fontos.sample.tcpclient.c3it.msg.handler.CatosNettyMessageClientChannelInitializer;
import tsb.fontos.sample.tcpclient.c3it.msg.handler.CatosReceiveMessageHandler;
import tsb.fontos.sample.tcpclient.c3it.msg.handler.CatosReceiveMessageProcessHandler;
import tsb.fontos.sample.tcpclient.c3it.msg.handler.CatosSignalMessageHandler;
import tsb.fontos.sample.tcpclient.c3it.msg.handler.SimpleNetMessageClientPoolHandler;
import tsb.fontos.sample.tcpclient.c3it.msg.objects.RstC01;
import tsb.fontos.sample.tcpclient.c3it.msg.objects.RstC21;

@Component
public class SampleTcpClientOperationInit {

    @Bean
    public NetMessageClientOperatoins tNetMessageClientOperatoins(CatosMessageClientConfig msgClientConfig) {
        MessageHandleStore msgHdlStore = new MessageHandleStore();
        CatosReceiveMessageProcessHandler msgProcessHdl = new CatosReceiveMessageProcessHandler();
        msgHdlStore.addMsgProcessDic(RstC01.class, msgProcessHdl);
        msgHdlStore.addMsgProcessDic(RstC21.class, msgProcessHdl);

        C3ITMessageBroker messageBroker = new C3ITMessageBroker();
        messageBroker.setMessageHandleStore(msgHdlStore);

        NetMessgeClinetOperationSetting env = new NetMessgeClinetOperationSetting();
        env.setConnectConfig(msgClientConfig.getCatos())
           .setChannelConfig(new CatosNettyMessageClientChannelInitializer(
                   new SimpleNetMessageClientPoolHandler(new CatosReceiveMessageHandler(), messageBroker),
                   new CatosSignalMessageHandler()))
           .setChannelLiftcycle(new CatosMessageClientChannelStatusListener())
           .setMessageProcessHandler(new CatosReceiveMessageHandler())
           .setMessageBroker(messageBroker)
           .setMessageSecurityHandler(new CatosNetSecurityMessageHandler(messageBroker));

        TNetMessageClientOperatoins<IBaseMsgObjectSend> clientCtx = NetMessageClientFactory.Build(env);
        try {
            clientCtx.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return clientCtx;
    }
}
```

### 8.3 Calling the TCP Client from a REST Controller

```java
// excerpt from tsb.fontos.sample.tcpclient.controller.HelloController
// (msgClientContext : tsb.fontos.net.msg.TNetMessageClientOperatoins, injected)
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import tsb.fontos.sample.tcpclient.c3it.msg.objects.MsgC21;
import tsb.fontos.sample.tcpclient.c3it.msg.objects.RstC01;
import tsb.fontos.sample.tcpclient.c3it.C3ITInfConstant;

@RequestMapping(value = "/C01", method = RequestMethod.GET)
public String rstC01() {
    MsgC21 msg = new MsgC21();
    msg.setEquipmentNo(C3ITInfConstant.CHAR_DTL_CARET + "Test Equip Name");
    // ... set remaining fields
    Object result = msgClientContext.SendSyncMessage(msg, RstC01.class); // blocks for the reply
    return result.toString();
}

@RequestMapping(value = "/C011", method = RequestMethod.GET)
public String rstC012Async() {
    MsgC21 msg = new MsgC21();
    // ... set fields
    msgClientContext.SendASyncMessage(msg); // fire-and-forget
    return "sendSuccess...";
}
```

Reference project: `Fontos-Sample-Backend/tsb.fontos.sample.tcpclient` (`c3it.msg.{broker,codec,config,handler,objects}`, `controller/HelloController.java`).

### 8.4 `tsb.fontos.cnet` — Annotation-driven TCP Server/Client

A separate, heavier-weight module that re-implements much of Spring MVC's request-mapping machinery (`@CnetController`, `@CnetRequestMapping`, argument resolvers, `HttpMessageConverter`-style JSON/XML/Gson converters) directly over raw Netty sockets, including a Tomcat-`Lifecycle`-style container lifecycle (`CnetLifecycle`) and session/connection management (`CnetConnectionManager`, `CnetSessionManager`) with pub/sub-style broadcast targeting (`CnetInterestedMessage`). Use `tsb.fontos.net` (§8.1–8.3) for straightforward client integrations against an existing message server; reach for `cnet` only if you need to expose your own annotation-routed TCP **server** with Spring-MVC-like ergonomics.

---

## 9. SOAP Web Services

### 9.1 Project Layout & Versions

The reference layout splits a **service module** (WSDL + generated stubs + business logic, no `main()`) from a **web app module** (CXF config + `@SpringBootApplication`):

```
/soap-service-module         # tsb.fontos.sample.ws.egs / .erp / soapServer's egs/erp/jpbi/slhms packages
 └─ src/main/java/...        # WSDL-generated interfaces/DTOs + <Xxx>SoapService implementation
 └─ src/main/resources/wsdl  # *.wsdl contracts
/soap-web-app                # tsb.fontos.sample.ws.composite / soapServer
 └─ src/main/java/...        # CxfConfig / SoapServiceConfig + Spring Boot entry point
```

| Component | Version |
|---|---|
| Java | 1.8 |
| Spring Boot | 2.x |
| Apache CXF | 3.4.10 |
| JAX-WS | 2.3.x |
| JAXB | 2.3.x |
| ORM | MyBatis |

### 9.2 Generating Java from WSDL

Add the codegen dependencies (kept out of the runtime classpath via a dedicated `jaxws` configuration):

```gradle
configurations {
    jaxws                  // wsimport-only deps, not on the runtime classpath
    compileOnly { extendsFrom annotationProcessor }
}

dependencies {
    implementation 'org.apache.cxf:cxf-spring-boot-starter-jaxws:3.4.10'
    implementation 'javax.xml.bind:jaxb-api:2.3.1'
    implementation 'com.sun.xml.bind:jaxb-impl:2.3.1'
    implementation 'com.sun.xml.bind:jaxb-core:2.3.0.1'
    implementation 'com.sun.xml.ws:jaxws-rt:2.3.2'
    jaxws 'com.sun.xml.ws:jaxws-tools:2.2.10'
}
```

Add a Gradle task that invokes `wsimport` (via the Ant DSL) against your WSDL:

```gradle
task egsGenerateSoapSources {
    ext.GEN_DEST_DIR = file("${projectDir}/src/main/java")
    ext.GEN_PACKAGE  = "tsb.fontos.sample.ws.egs.soap"
    ext.GEN_WSDL     = file("${projectDir}/src/main/resources/wsdl/EGSInterface.wsdl")

    doLast {
        ant {
            taskdef(name: 'wsimport', classname: 'com.sun.tools.ws.ant.WsImport',
                    classpath: configurations.jaxws.asPath)
            wsimport(wsdl: GEN_WSDL, package: GEN_PACKAGE, sourcedestdir: GEN_DEST_DIR,
                     keep: true, extension: true, verbose: true, xnocompile: true) {
                xjcarg(value: "-XautoNameResolution")
            }
        }
    }
}
```

Run it from Eclipse via **Run → Run Configurations → Gradle Task**, targeting `egsGenerateSoapSources` with the module's working directory, then refresh the project — generated JAXB stubs (`ObjectFactory`, request/response DTOs, the port interface) land under the configured package.

### 9.3 Implementing the Service

```java
package tsb.fontos.sample.ws.egs;

import java.util.List;

import javax.jws.WebService;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import tsb.fontos.sample.item.board.AcknowledgementItem;
import tsb.fontos.sample.item.board.CmBoardItem;
import tsb.fontos.sample.item.board.MessageError;
import tsb.fontos.sample.item.board.MessageHeader;
import tsb.fontos.sample.param.board.CmBoardParam;
import tsb.fontos.sample.service.board.CmBoardService;
import tsb.fontos.sample.ws.egs.soap.EgsContainerListOffTakerIdParm;
import tsb.fontos.sample.ws.egs.soap.IEGSWebService;

@WebService(serviceName = "EGSWebServiceService",
            portName = "EGSWebServicePort",
            targetNamespace = "http://egs.spj.webservice.itos.tsb/",
            endpointInterface = "tsb.fontos.sample.ws.egs.soap.IEGSWebService")
@Component
public class EgsSoapService implements IEGSWebService {

    @Autowired private CmBoardService service;

    @Override
    public AcknowledgementItem containerOffTakeId(EgsContainerListOffTakerIdParm request) {
        List<CmBoardItem> boardItems = service.selectCmBoardAll(new CmBoardParam());

        MessageHeader header = new MessageHeader();
        header.setMsgId("MSG-0001 --> " + (boardItems != null ? boardItems.size() : 0));
        header.setMsgType("RESPONSE");
        header.setTransCode("OK");

        MessageError error = new MessageError();
        error.setCode(0);
        error.setMessage("SUCCESS");

        AcknowledgementItem ack = new AcknowledgementItem();
        ack.setMessageHeader(header);
        ack.setMessageError(error);
        return ack;
    }
}
```

`serviceName`/`portName`/`endpointInterface` must match the `<wsdl:service>`/`<wsdl:port>`/`<wsdl:portType>` definitions in the WSDL contract.

### 9.4 Publishing the CXF Endpoint

```java
package tsb.fontos.sample.ws.composite;

import javax.xml.bind.JAXBException;
import javax.xml.ws.Endpoint;

import org.apache.cxf.Bus;
import org.apache.cxf.ext.logging.LoggingFeature;
import org.apache.cxf.jaxb.JAXBDataBinding;
import org.apache.cxf.jaxws.EndpointImpl;
import org.apache.cxf.transport.servlet.CXFServlet;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import tsb.fontos.sample.ws.egs.EgsSoapService;

@Configuration
public class SoapServiceConfig {

    @Autowired private Bus bus;

    @Bean
    public Endpoint egsEndpoint(EgsSoapService egsWebService) throws JAXBException {
        EndpointImpl endpoint = new EndpointImpl(bus, egsWebService);
        endpoint.getFeatures().add(new LoggingFeature()); // request/response XML logging — useful for debugging
        endpoint.setWsdlLocation("classpath:wsdl/EGSInterface.wsdl");
        endpoint.setDataBinding(new JAXBDataBinding(tsb.fontos.sample.ws.egs.soap.ObjectFactory.class));
        endpoint.publish("/egswebservice"); // full path: /soap-api/egswebservice
        return endpoint;
    }

    @Bean
    public ServletRegistrationBean<CXFServlet> cxfServlet() {
        return new ServletRegistrationBean<>(new CXFServlet(), "/soap-api/*");
    }
}
```

Resulting endpoints: `http://localhost:8080/soap-api/egswebservice` (SOAP), `http://localhost:8080/soap-api/egswebservice?wsdl` (WSDL browsing).

### 9.5 SOAP Client

Generated JAX-WS stubs can be called directly (no Spring wiring needed for a simple client):

```java
// classes generated by wsimport under the configured GEN_PACKAGE, e.g.:
import tsb.fontos.sample.ws.gen.userservice.GetUserRequest;
import tsb.fontos.sample.ws.gen.userservice.GetUserResponse;
import tsb.fontos.sample.ws.gen.userservice.UserEndpointInterface;
import tsb.fontos.sample.ws.gen.userservice.UserService;

UserService service = new UserService();
UserEndpointInterface port = service.getUserServicePort();
GetUserRequest request = new GetUserRequest();
request.setUserId("U12345");
GetUserResponse response = port.getUser(request);
```

For consuming **externally-hosted** SOAP services (rather than services generated in-repo), the framework also offers a Spring-WS style helper: `tsb.fontos.sample.webservice.SoapGatewaySupport extends WebServiceGatewaySupport`, wrapping `WebServiceTemplate` + `SoapActionCallback`. This module is also where mTLS (client-certificate) SOAP integrations against external partner endpoints are wired up — see `webservice/noatum/` for a real-world example pattern (client-cert config against a customer's SOAP gateway).

> If you copy the `noatum`-style mTLS example, double-check `application.properties` for any keystore password or endpoint credential that may have been left in place from the original integration — those must be per-environment secrets sourced from your secret store, never committed as plaintext.

### 9.6 Composing Multiple SOAP Modules into One WAR

`tsb.fontos.sample.ws.composite` shows how to combine independently-built SOAP service modules (`tsb.fontos.sample.ws.egs`, `tsb.fontos.sample.ws.erp`) into a single deployable: it `@ComponentScan`s both its own package and the sibling modules' packages, and its `SoapServiceConfig` registers one CXF `Endpoint` bean per service. The individual `ws.egs`/`ws.erp` modules keep their own `Application.java` commented out — they're meant to be consumed as libraries only, never run standalone.

### 9.7 SOAP Client — Advanced Configuration (WSDL Codegen, mTLS, SoapAction)

This expands [9.5](#95-soap-client) with the full client-project setup used for real external SOAP integrations (worked example: a customer web-service integration referred to as "Noatum").

**Dependencies and WSDL code generation.** A SOAP client project needs the web-services starter plus a dedicated `jaxws` configuration used only by a custom Gradle codegen task (not a runtime dependency):

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web-services'
    annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
    compileOnly 'org.projectlombok:lombok'
    jaxws 'com.sun.xml.ws:jaxws-tools:2.2.10'
    implementation project(":tsb.fontos.core")
}

task wsimportNoatum {
    ext.GEN_DEST_DIR = file("${projectDir}/src/main/java")
    ext.GEN_PACKAGE = "tsb.fontos.sample.webservice.ws.gen.noatum"
    ext.GEN_WSDL = "${projectDir}/src/main/resources/wsdl/noatumtermcastellonWSDL.xml"
    doLast {
        ant {
            sourceSets.main.output.classesDirs.singleFile.mkdirs()
            file(GEN_DEST_DIR).mkdirs()
            taskdef(name: 'wsimport', classname: 'com.sun.tools.ws.ant.WsImport', classpath: configurations.jaxws.asPath)
            wsimport(keep: true, sourcedestdir: GEN_DEST_DIR, package: GEN_PACKAGE, wsdl: GEN_WSDL,
                     extension: "true", encoding: "utf-8", xnocompile: "true") {
                xjcarg(value: "-XautoNameResolution")
            }
        }
    }
}
```

Run one such task per WSDL (via STS/Eclipse Run Configurations → Gradle Task, e.g. `wsimportnoatum`), then refresh the project — generated sources land under `<package>.ws.gen.<serviceName>`.

**Connection properties**, bound via a `@ConfigurationProperties` POJO per external service:

```java
package tsb.fontos.sample.webservice.noatum;

import org.springframework.boot.context.properties.ConfigurationProperties;

import lombok.Getter;
import lombok.Setter;

@Getter @Setter
@ConfigurationProperties(prefix = "ws.noatum")
public class NoatumWebserviceProperties {
    private String sslKeyStore, sslKeyStorePassword, sslKeyStoreType;
    private String sslTrustStore, sslTrustStorePassword, sslTrustStoreType;
    private String soapAddressLocation, soapDefaultAction;
}
```

```properties
ws.noatum.ssl-key-store=security/client_noatumtermcastellon.p12
ws.noatum.ssl-key-store-password=${WS_NOATUM_KEYSTORE_PASSWORD}
ws.noatum.ssl-key-store-type=JKS
ws.noatum.ssl-trust-store=security/noatumtermcastellonTrustCacerts
ws.noatum.ssl-trust-store-password=${WS_NOATUM_TRUSTSTORE_PASSWORD}
ws.noatum.ssl-trust-store-type=JKS
ws.noatum.soap-address-location=https://<partner-host>/msg-broker-ejb/MsgWSService/MsgWS
ws.noatum.soap-default-action="http://ws.utils.indra.com"
```

> **Security flag**: the source training deck hardcodes real-looking plaintext values for `ws.noatum.ssl-key-store-password` and `ws.noatum.ssl-trust-store-password` directly in `application.properties`. Never commit literal keystore/truststore passwords — externalize them (env var / secrets manager / Jasypt), as shown above with placeholders.

**The `SoapGatewaySupport` wrapper.** Client calls go through the framework's own `SoapGatewaySupport` class (found standalone in the sample project, package root `tsb.fontos.sample.webservice`) — **not** Spring-WS's `WebServiceGatewaySupport` directly, though it wraps `WebServiceTemplate.marshalSendAndReceive(...)` internally (confirmed via a stack trace showing `SoapGatewaySupport.callSoapService(SoapGatewaySupport.java:25)` → `WebServiceTemplate.marshalSendAndReceive`). It exposes two overloads:

- `callSoapService(String address, Object request)` → `JAXBElement<T>` — for WSDLs with no explicit `soapAction`.
- `callSoapService(String address, String soapAction, Object request)` → `T` directly — for WSDLs whose operation declares `<soap:operation soapAction="...">`; build the soap-action string from config (e.g. `soapProperties.getSoapDefaultAction() + "/Add"`).

```java
package tsb.fontos.sample.webservice.noatum;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.transport.http.HttpComponentsMessageSender;

import org.apache.http.client.HttpClient;
import tsb.fontos.core.security.ssl.SSLConfig;
import tsb.fontos.core.security.ssl.THttpsClientFactory;
import tsb.fontos.sample.webservice.SoapGatewaySupport;

@Configuration
@EnableConfigurationProperties(NoatumWebserviceProperties.class)
public class NoatumSoapConfiguration {
    @Autowired NoatumWebserviceProperties noatumWebserviceProperties;

    @Bean
    public Jaxb2Marshaller marshaller() {
        Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
        marshaller.setPackagesToScan("tsb.fontos.sample.webservice.ws.gen.noatum");
        return marshaller;
    }

    @Bean(name = "noatumSoapGatewaySupport")
    public SoapGatewaySupport noatumSoapGatewaySupport(Jaxb2Marshaller marshaller) {
        SoapGatewaySupport client = new SoapGatewaySupport();
        client.setMarshaller(marshaller);
        client.setUnmarshaller(marshaller);
        client.setMessageSender(noatumHttpsComponentsMessageSender()); // mTLS, below
        return client;
    }

    @Bean
    public HttpComponentsMessageSender noatumHttpsComponentsMessageSender() {
        ClassPathResource keyRs = new ClassPathResource(noatumWebserviceProperties.getSslKeyStore());
        ClassPathResource trustRs = new ClassPathResource(noatumWebserviceProperties.getSslTrustStore());

        SSLConfig sslConfig = new SSLConfig();
        sslConfig.setKeyStore(keyRs);
        sslConfig.setKeyPwd(noatumWebserviceProperties.getSslKeyStorePassword());
        sslConfig.setKeyStoreType(noatumWebserviceProperties.getSslKeyStoreType());
        sslConfig.setTrustStore(trustRs);
        sslConfig.setTrustPwd(noatumWebserviceProperties.getSslTrustStorePassword());
        sslConfig.setTrustStoreType(noatumWebserviceProperties.getSslTrustStoreType());

        HttpClient httpClient = THttpsClientFactory.build().createHttpsClient(sslConfig, 1000); // 1000ms timeout
        HttpComponentsMessageSender sender = new HttpComponentsMessageSender();
        sender.setHttpClient(httpClient);
        return sender;
    }
}
```

`THttpsClientFactory`/`SSLConfig` are framework helpers (`tsb.fontos.core`) for building an Apache `HttpClient` configured for mutual-TLS; their own source isn't shown in the training material, only the call sites above.

**Calling it** — inject the qualified client bean, build the request, unwrap the response:

```java
package tsb.fontos.sample.webservice.noatum;

import javax.xml.bind.JAXBElement;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

import tsb.fontos.sample.webservice.SoapGatewaySupport;
import tsb.fontos.sample.webservice.ws.gen.noatum.GetMessages;
import tsb.fontos.sample.webservice.ws.gen.noatum.GetMessagesResponse;
import tsb.fontos.sample.webservice.ws.gen.noatum.MessageListResultWS;

@Component
public class SampleNoatumService {
    @Autowired NoatumWebserviceProperties noatumWebserviceProperties;
    @Autowired @Qualifier("noatumSoapGatewaySupport") private SoapGatewaySupport noatumSoapGatewaySupport;

    public MessageListResultWS getMessages() {
        GetMessages request = new GetMessages();
        request.setFolder("outbox");
        JAXBElement<GetMessagesResponse> resp = noatumSoapGatewaySupport.callSoapService(
                noatumWebserviceProperties.getSoapAddressLocation(), request);
        return resp.getValue().getResult(); // map into the return type as appropriate
    }
}
```

```java
package tsb.fontos.sample.controller.webservice;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import lombok.RequiredArgsConstructor;
import tsb.fontos.core.http.rest.TRestResponse;
import tsb.fontos.sample.param.board.WebBoardParam;
import tsb.fontos.sample.webservice.noatum.SampleNoatumService;
import tsb.fontos.sample.webservice.ws.gen.noatum.MessageListResultWS;

@RestController
@RequiredArgsConstructor
public class NoatumWebServiceController {
    private final SampleNoatumService service;

    @GetMapping("/api/v1/getMessages")
    public TRestResponse<MessageListResultWS> getMessages(WebBoardParam param) {
        return new TRestResponse<>(service.getMessages());
    }
}
```

**Wiring the client project into a consuming Web Application** — reference it as a sibling Gradle project (same DEV-TOOL pattern as [3.2](#32-multi-module-gradle-setup)) and add its connection properties either to `application.properties` directly or to `Module-config.properties`:

```gradle
// settings.gradle
include ':tsb.fontos.sample.webservice'
project(':tsb.fontos.sample.webservice').projectDir = new File(settingsDir, '../../../Tsb-Fontos-Sample/Fontos-Sample-Backend/tsb.fontos.sample.webservice')
```

```gradle
// consuming app's build.gradle
implementation project(":tsb.fontos.sample.webservice")
```

**Known gotchas:**

1. **Building a TrustStore from a browser-exported certificate**: export the endpoint's cert from the browser (padlock → connection details → export), then `keytool -importcert -alias <alias> -file <cert>.crt -keystore <truststore-name> -storepass <pwd>`.
2. **Missing `@XmlRootElement` on wsimport-generated classes** — a request type generated by `wsimport` that isn't annotated causes `MarshallingFailureException: ... missing an @XmlRootElement annotation` at `Jaxb2Marshaller.marshal`. Fix by manually adding `@XmlRootElement(name = "<wsdlElementName>")` to the generated class, matching the element name in the WSDL/XSD.
3. **Explicit `soapAction` in the WSDL** — if `<wsdl:operation><soap:operation soapAction="...">` is present, you must pass that soapAction string explicitly via the 3-arg `callSoapService(address, soapAction, request)` overload rather than relying on automatic SOAPAction dispatch.

---

## 10. Report Generation

### 10.1 JasperReports

Module: `tsb.fontos.report.jasper`. Dependencies:

```gradle
implementation 'net.sf.jasperreports:jasperreports:6.5.1'
implementation 'com.lowagie:itext:2.1.7'
// optional, for barcodes/QR codes:
implementation 'net.sf.barcode4j:barcode4j:2.1'
implementation 'net.sourceforge.barbecue:barbecue:1.5-beta1'
implementation 'com.google.zxing:javase:3.3.0'
```

Configure where `.jrxml`/`.jasper` template files live (classpath or filesystem, with an optional `catos.home`-style system-property root for per-environment overrides):

```properties
env.report.jasper.path-prefix=classpath:
env.report.jasper.root-path=
env.report.jasper.config-path=report/sl/jasper

## Alternative: filesystem-based, e.g. launched with -Dcatos.home=D:/Temp/catosWebAppConfigHome
# env.report.jasper.path-prefix=file:
# env.report.jasper.root-path=systemProperty:catos.home
# env.report.jasper.config-path=report/sl/jasper
```

Extend `BaseReportJasperController` and build a `ReportJasperItem`:

```java
package tsb.fontos.sample.controller.report.jasper;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

import tsb.fontos.report.jasper.controller.BaseReportJasperController;
import tsb.fontos.report.jasper.item.ReportJasperItem;
import tsb.fontos.report.jasper.item.SubReportJasperItem;
import tsb.fontos.sample.item.report.Employee;
import tsb.fontos.sample.param.board.BoardParam;
import tsb.fontos.sample.service.report.JasperService;

@Controller
public class JasperController extends BaseReportJasperController {

    @Autowired private JasperService jasperService;

    @GetMapping("reportJasperDefaultSampleByPreview")
    public ResponseEntity<byte[]> reportJasperDefaultSample(BoardParam param) throws Exception {
        List<Employee> result = jasperService.selectEmployeeItems(param);

        ReportJasperItem reportItem = new ReportJasperItem("employee-rpt-database.jrxml", "employee-rpt-database.pdf");
        reportItem.addParameter("createdBy", "www.tsb.co.kr1").addParameter("qrCode1", "12345678");
        reportItem.setDataCollection(result);

        return this.exportReportToPreviewPdf(reportItem);
    }

    // Master/sub (nested) report
    @GetMapping("reportJasperMasterDetailSampleByPreview")
    public ResponseEntity<byte[]> reportJasperMasterDetailSampleByPreview(BoardParam param) throws Exception {
        List<Employee> master = jasperService.selectEmployeeMasterItems(param);
        List<Employee> sub = jasperService.selectEmployeeSubItems(param);

        ReportJasperItem reportItem = new ReportJasperItem("employee-rpt-databaseMain.jrxml", "employee-rpt-database.pdf");
        reportItem.addParameter("createdBy", "www.tsb.co.kr1").addParameter("qrCode1", "12345678");
        reportItem.addParameter("PREADVICE_DATA", sub);
        reportItem.addSubReportItem(new SubReportJasperItem("PREADVICE_SUBREPORT1", "employee-rpt-databaseSub.jrxml"));
        reportItem.setDataCollection(master);

        return this.exportReportToPreviewPdf(reportItem);
    }
}
```

**Images** (logo, dynamic images) can be passed as a URL, a classpath resource, or via the framework's own `ReportImageLoader`, then bound to a JasperReports `Image` component whose *Expression Class* is `java.awt.Image`:

```java
// excerpt continuing a JasperController method
import javax.imageio.ImageIO;

import org.springframework.core.io.ClassPathResource;

import tsb.fontos.report.jasper.util.ReportImageLoader;

reportItem.addParameter("IMAGE_URL_1", baseUrl + "/image/log/tsb_log.jpg"); // URL
reportItem.addParameter("IMAGE_URL_2",
    ImageIO.read(new ClassPathResource("public/image/log/tsb_log.jpg").getInputStream())); // classpath
reportItem.addParameter("IMAGE_URL_3", ReportImageLoader.Load("tsb_log.jpg")); // framework loader
```

**PDF permissions/encryption** via `JasperPermissionItem`:

```java
// excerpt continuing a JasperController method
import tsb.fontos.report.jasper.item.JasperPermissionItem;

reportItem.setPermission(new JasperPermissionItem.Builder()
        .userPassword("1234")
        .allowPrinting(true)
        .allowCopy(true)
        .build());
```

**iReport Designer** (template authoring tool) needs **JDK 7u21** specifically (iReport 5.5 is the last free version and only runs on JDK 7) — set `jdkhome` in `ireport.conf` accordingly. Missing font handling: install the font via iReport's Font Installation wizard, export it as a font-extension `.jar`, drop it under `libs/`, reference it from `build.gradle` (`implementation files('libs/arial-font-extension.jar')`), and set `net.sf.jasperreports.awt.ignore.missing.font=true` so a still-missing glyph doesn't hard-fail rendering.

Example resource/source layout:

```
src/main/resources/report.sl.jasper/
  employee-rpt-database.{jasper,jrxml}
  employee-rpt-databaseMain.jrxml
  employee-rpt-databaseSub.jrxml
  barcode-test-rpt.{jasper,jrxml}
  tsb_log.jpg
src/main/java/.../controller/report/jasper/JasperController.java
src/main/java/.../dao/report/jasper/
```

### 10.2 OpenHTMLtoPDF

Module: `tsb.fontos.report.openhtmltopdf` — an alternative to JasperReports for teams that would rather author report layouts as Thymeleaf HTML templates than `.jrxml` files.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
implementation 'org.jsoup:jsoup:1.16.2'
implementation 'com.openhtmltopdf:openhtmltopdf-pdfbox:1.0.10'
implementation 'org.apache.pdfbox:pdfbox:2.0.24'
implementation 'net.sourceforge.barbecue:barbecue:1.5-beta1'  // barcodes
implementation 'net.glxn:qrgen:1.4'                            // QR codes
```

`ReportOpenHtmlToPdfHandler` renders the Thymeleaf template, forces XHTML-strict output via Jsoup, then converts the resulting W3C DOM to PDF with `openhtmltopdf`'s `PdfRendererBuilder`:

```java
// excerpt from a class in tsb.fontos.report.openhtmltopdf
import java.io.ByteArrayOutputStream;
import java.io.IOException;

import org.jsoup.Jsoup;
import org.jsoup.helper.W3CDom;
import org.jsoup.nodes.Document;

import com.openhtmltopdf.pdfboxout.PdfBoxRenderer;
import com.openhtmltopdf.pdfboxout.PdfRendererBuilder;

import tsb.fontos.report.openhtmltopdf.config.AppReportOpenhtmlToPdfPathInfo;
import tsb.fontos.report.openhtmltopdf.item.ReportOpenHtmlToPdfItem;

public byte[] exportReportToPdf(ReportOpenHtmlToPdfItem reportItem) throws IOException {
    String inputHTML = this.processToHtml(reportItem); // Thymeleaf render
    Document doc = Jsoup.parse(inputHTML, "UTF-8");
    doc.outputSettings().syntax(Document.OutputSettings.Syntax.xml);

    try (ByteArrayOutputStream os = new ByteArrayOutputStream()) {
        PdfRendererBuilder builder = new PdfRendererBuilder();
        builder.toStream(os);
        builder.withW3cDocument(new W3CDom().fromJsoup(doc),
                "file:///" + AppReportOpenhtmlToPdfPathInfo.getEnvReportOpenhtmlToPdfBasePath() + "/");
        PdfBoxRenderer renderer = builder.buildPdfRenderer();
        renderer.layout();
        renderer.createPDF();
        return os.toByteArray();
    }
}
```

`BaseReportOpenHtmlToPdfController` mirrors the Jasper controller's preview/download split. QR codes are generated with a thin wrapper over `qrgen`:

```java
import net.glxn.qrgen.core.image.ImageType;
import net.glxn.qrgen.javase.QRCode;

public static byte[] getQRCode(String text, int width, int height) {
    return QRCode.from(text).withSize(width, height).to(ImageType.PNG).stream().toByteArray();
}
```

---

## 11. Excel Export

### 11.1 Backend Setup

Module dependency (Apache POI, version-pinned to avoid transitive conflicts):

```gradle
implementation 'org.apache.poi:poi:5.2.3'
implementation 'org.apache.poi:poi-ooxml:5.2.3'
implementation 'org.apache.xmlbeans:xmlbeans:5.1.1'
implementation 'org.apache.commons:commons-compress:1.21'
implementation 'org.apache.commons:commons-collections4:4.4'
```

```properties
fontos.file.export.enabled=true
fontos.file.export.excel-temp-dir=/var/fontos/excel_export/tempfile
```

Controller — resolve the export via `ExportProviderFactory`, stream the generated file back as an attachment:

```java
package tsb.fontos.sample.controller.excel;

import java.io.FileNotFoundException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Path;
import java.nio.file.Paths;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.util.UriUtils;

import lombok.RequiredArgsConstructor;
import tsb.fontos.core.export.ExportFileItem;
import tsb.fontos.core.export.ExportProviderFactory;
import tsb.fontos.core.param.FileExportParm;
import tsb.fontos.core.web.TBaseRestController;

@RestController
@RequiredArgsConstructor
public class SampleExcelExportController extends TBaseRestController {

    @Autowired(required = false)
    private ExportProviderFactory exportProviderFactory;

    @PostMapping("/export/excel-export-download")
    public ResponseEntity<Resource> exportTest06(HttpServletRequest request, HttpServletResponse response,
                                                   @RequestBody FileExportParm param) throws Exception {
        ExportFileItem fileItem = exportProviderFactory.getExcel().export(request, response, param);
        Path filePath = Paths.get(fileItem.getFullName()).normalize();
        Resource resource = new UrlResource(filePath.toUri());
        if (!resource.exists()) throw new FileNotFoundException("File not found: " + fileItem.getName());

        String encodedFilename = UriUtils.encode(fileItem.getName(), StandardCharsets.UTF_8);
        response.setHeader("Access-Control-Expose-Headers", "Content-Disposition");
        return ResponseEntity.ok()
            .contentType(MediaType.APPLICATION_OCTET_STREAM)
            .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + encodedFilename + "\"")
            .body(resource);
    }
}
```

### 11.2 Frontend Integration

Service/DAO pair on the frontend (`tsb-fontos-core`), pointed at the backend endpoint above:

```typescript
export default class SampleExportGridDataService extends BaseService implements IExportGridDataService {
    constructor(public dao: IExportGridDataDao) { super(); }
    public async exportAndDownload(param: FileExportParm): Promise<DownloadFileItem> {
        return this.dao.exportAndDownload(param);
    }
}

export default class SampleExportGridDataDao extends HttpWebDaoBindingSupport implements IExportGridDataDao {
    public async exportAndDownload(param: FileExportParm): Promise<DownloadFileItem> {
        const request: TRequest = { serverKey: BizRemoteServerKeys.Default, url: "/export/excel-export-download", datas: param };
        return this.postDownloadFile(request, RemoteServerCfgHdl.getInstance().getConfig(BizRemoteServerKeys.Default));
    }
}
```

Register the service under the exact framework-expected name, `CoreServerName.CORE_EXPORT_GRID_DATA_SVR`:

```typescript
case CoreServerName.CORE_EXPORT_GRID_DATA_SVR:
    svrObj = new SampleExportGridDataService(new SampleExportGridDataDao()) as T;
    break;
```

Trigger it from a grid screen via `ExportGridService.downloadExcel(...)`:

```typescript
downloadExcelAtServer = async () => {
  const excelService = new ExportGridService();
  await excelService.downloadExcel({
    gridRef: this._grid,
    title: "Main Title",
    fileName: `BOARD_TSB_${ExportUtil.getFormattedCurrentDate()}`,
    searchConditions: ExportConvert.convertToSearchConditionItems(this._controller.getSearchConditions()),
    dataLocation: "singleGridService.selectSingleGrid", // BeanName.MethodName
    fileFormat: FileExportConstants.EXPORT_FILE_FORMAT_XLSX,
  });
};
```

`dataLocation` must resolve to a `@Service`-annotated bean method that returns the export rows, e.g.:

```java
// illustrative — see §19.2 for this same bean name backed by real production code
package tsb.fontos.sample.service.singlegrid;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import tsb.fontos.core.service.TBaseRestService;
import tsb.fontos.sample.dao.singlegrid.SingleGridDao;
import tsb.fontos.sample.item.singlegrid.SingleGridItem;
import tsb.fontos.sample.param.singlegrid.SingleGridParam;

@Service("singleGridService")
@Transactional(readOnly = true)
public class SingleGridService extends TBaseRestService {
    @Autowired private final SingleGridDao singleGridDao;

    @Transactional(propagation = Propagation.NEVER)
    public List<SingleGridItem> selectSingleGrid(SingleGridParam param) {
        return singleGridDao.selectSingleGrid(param);
    }
}
```

Row/column caps are enforced by a singleton config (`tsb.fontos.core.export.ExportDataConfig`, defaults 500 columns / 5000 rows) — export requests outside these bounds raise an exception.

### 11.3 Per-cell Coloring

Set `colorRule` on the grid header schema for the column that should carry per-row color; the framework then looks for `{dataField}Backcolor`/`{dataField}Forecolor` fields on the export item:

```json
{ "text": "Unno", "dataField": "unno", "colorRule": "ABC" }
```

```java
// illustrative — matches the real field pair confirmed in §19.2's SingleGridItem
import tsb.fontos.core.item.BaseDataItem;

public class SingleGridItem extends BaseDataItem {
    private String unno;
    private String unnoBackcolor = "12648384";
    private String unnoForecolor = "16777158";
}
```

---

## 12. Frontend Log Forwarding

FONTOS Core ships a ready-made endpoint that lets a frontend application forward its own runtime log messages (errors, warnings, etc.) to the backend, where they land in the same Logback pipeline as server-side logs — useful for centralizing SPA error reports without standing up a separate logging service.

### 12.1 Backend Setup

Enable the feature and name the logger it should write through:

```properties
fontos.frontend.log.enabled=true
fontos.frontend.log.name=FRONTEND_LOG
```

Add a dedicated appender/logger pair in `logback-spring.xml` using that same name — this is what actually determines where the forwarded messages are written:

```xml
<!-- [START] SPA(Web Page) LOG SETTING -->
<appender name="ROLLING_FRONTEND" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>C:\log\mylog_sl_frontend.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- rollover daily -->
        <fileNamePattern>C:\log\fontoslog\mylog_frontend-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
            <!-- or whenever the file size reaches 1MB -->
            <maxFileSize>20MB</maxFileSize>
        </timeBasedFileNamingAndTriggeringPolicy>
    </rollingPolicy>
    <encoder>
        <pattern>[%-5level] %d{HH:mm:ss.SSS} %logger{36} - %msg%n</pattern>
    </encoder>
</appender>

<logger name="FRONTEND_LOG" level="DEBUG" additivity="false">
    <appender-ref ref="ROLLING_FRONTEND"/>
</logger>
<!-- [END] SPA(Web Page) LOG SETTING -->
```

> The `name` on `<logger>` must exactly match `fontos.frontend.log.name` from `application.properties` — that's the wiring between the two files.

The incoming log payload is described by a small framework DTO and level enum (`tsb.fontos.core.log`):

```java
package tsb.fontos.core.log;

import java.util.Map;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class SimpleFrontendLogItem {
    private SimpleFrontendLogLevel logLevel; // must match a Logback output level for it to actually get written
    private Map<String, Object> logData;     // arbitrary payload forwarded from the frontend
}

public enum SimpleFrontendLogLevel {
    DEBUG("DEBUG"), INFO("INFO"), WARN("WARN"), ERROR("ERROR"), TRACE("TRACE");

    private final String level;
    SimpleFrontendLogLevel(String level) { this.level = level; }
    public String getLevel() { return level; }
}
```

Expose the endpoint by extending the framework's `BaseFrontendLogController`:

```java
package tsb.fontos.sample.controller.log;

import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import tsb.fontos.core.http.rest.TRestResponse;
import tsb.fontos.core.item.BaseDataItem;
import tsb.fontos.core.log.BaseFrontendLogController;
import tsb.fontos.core.log.SimpleFrontendLogItem;

public class SimpleFrontendLogController extends BaseFrontendLogController {

    @Value("${fontos.frontend.log.name}")
    private String loggerName;

    @RequestMapping(value = "${fontos.frontend.log.urlpath:/frontendLog}", method = RequestMethod.GET)
    public TRestResponse<BaseDataItem> frontendLogByGet(HttpServletRequest request,
                                                          @RequestBody SimpleFrontendLogItem logItem) throws Exception {
        this.writeLogger(request, logItem);
        return new TRestResponse<>(new BaseDataItem());
    }

    @RequestMapping(value = "${fontos.frontend.log.urlpath:/frontendLog}", method = RequestMethod.POST)
    public TRestResponse<BaseDataItem> frontendLogByPost(HttpServletRequest request,
                                                           @RequestBody SimpleFrontendLogItem logItem) throws Exception {
        this.writeLogger(request, logItem);
        return new TRestResponse<>(new BaseDataItem());
    }

    @Override
    protected String GetLoggerName() {
        return this.loggerName;
    }
}
```

Both `GET` and `POST` are provided; the default request path is **`/frontendLog`** (overridable via `fontos.frontend.log.urlpath`). Unless there's a specific reason not to, backend services should keep using the FONTOS-provided path rather than defining a custom one, so every application's frontend logs land in the same place.

### 12.2 Frontend Integration

The frontend side lives in `tsb-fontos-core`'s `logger/remote` package (`FrontendLogItem`, `FrontendLogLevel`, `IFrontendLogItem`, `IFrontendLogService`) plus `GeneralLogger`, and is wired up in an application the same way other core-provided services are (compare with the Excel export wiring in [§11.2](#112-frontend-integration)):

1. **Define the client-side log item** — the fields your app actually wants to forward (kept separate from the framework's own `FrontendLogItem`, see the mapping step below):

```typescript
import { BaseDataItem } from "tsb-fontos-core";

export default class FrontendToRemoteLogItem {
  message!: string;
  pgmCode!: string;
}
```

2. **Add a DAO** that posts to the framework's endpoint — reuse `/frontendLog` unless there's a specific reason to deviate:

```typescript
export default class RemoteSendLogDao extends HttpWebDaoBindingSupport {
  public async writeLog(item: IFrontendLogItem): Promise<CTResponse<CTRestResponse<CTBaseResultItem>>> {
    try {
      let requestParam = new CTRequest();
      requestParam.serverKey = CTRemoteServerKeys.Default;
      requestParam.url = "/frontendLog";
      requestParam.datas = item;

      let responseItem = await this.postRestResponse(CTBaseResultItem, requestParam);
      return responseItem;
    } catch (ex) {
      GeneralLogger.error(ex);
      throw ex;
    }
  }
}
```

3. **Add a service** that converts your app's log item into the shape FONTOS Core (the backend) expects:

```typescript
export default class RemoteSendLogService extends BaseService implements IFrontendLogService {
  public dao: RemoteSendLogDao;

  constructor(dao: RemoteSendLogDao) {
    super();
    this.dao = dao;
  }

  public async writeLog(logLevel: FrontendLogLevel, logItem: SimpleFrontendLogItem): Promise<BaseDataItem> {
    try {
      let rlogItem: FrontendLogItem = new FrontendLogItem();
      rlogItem.logLevel = logLevel;
      rlogItem.logData = {
        "pgmCode": logItem.pgmCode,
        "message": logItem.message,
      };

      this.dao.writeLog(rlogItem);
      return {} as any;
    } catch (ex) {
      GeneralLogger.error(ex);
      throw ex;
    }
  }
}
```

4. **Register the service** under the exact framework-expected name, `CoreServerName.CORE_REMOTE_LOG_SVR` (same pattern as the Excel export service registration in [§11.2](#112-frontend-integration)):

```typescript
case CoreServerName.CORE_REMOTE_LOG_SVR:
  svrObj = new RemoteSendLogService(new RemoteSendLogDao()) as T;
  break;
```

5. **Send a log** from anywhere in the app via `GeneralLogger.remoteWriteLog(...)`:

```tsx
sendLog = async () => {
  const logitem: FrontendToRemoteLogItem = new FrontendToRemoteLogItem();
  logitem.message = this.state.errMsg;
  logitem.pgmCode = "PGM-CODE-SAMPLE";

  GeneralLogger.remoteWriteLog(FrontendLogLevel.ERROR, logitem);
};
```

`GeneralLogger.remoteWriteLog` resolves the registered `CORE_REMOTE_LOG_SVR` service under the hood, maps your app-specific log item into the framework's `FrontendLogItem`, and posts it to `/frontendLog` — where it's written into the `FRONTEND_LOG` logger configured in [§12.1](#121-backend-setup).

---

## 13. Scheduling & Task Monitoring

### 13.1 Simple `@Scheduled` Jobs

The lightweight path: standard Spring `@Scheduled`, gated by a feature-toggle property so it can be turned off per environment without a redeploy:

```properties
sample.scheduleOne.enable=true
sample.scheduleOne.cron=*/1 * * * * *
```

```java
package tsb.fontos.sample.scheduler;

import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import tsb.fontos.core.micrometer.annotation.TSchedulerTaskMonitor;

@Component
@ConditionalOnProperty(prefix = "sample", name = "scheduleOne.enable", havingValue = "true")
public class SampleOneScheduler {
    @TSchedulerTaskMonitor(name = "sampleOneJob") // see §13.3
    @Scheduled(cron = "${sample.scheduleOne.cron}")
    public void run() { /* ... */ }
}
```

The framework provides a dedicated `TScheduler` base class for this pattern — extend it (rather than a bare `@Component`) so annotation-based jobs share the framework's own scheduler wiring:

```java
package tsb.fontos.sample.scheduler;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;

import lombok.extern.slf4j.Slf4j;
import tsb.fontos.core.scheduler.TScheduler;
import tsb.fontos.sample.param.board.BoardParam;
import tsb.fontos.sample.service.board.BoardService;

@Slf4j
@Configuration
@ConditionalOnProperty(name = "sample.scheduleOne.enable", havingValue = "true")
@EnableScheduling
public class SampleOneScheduler extends TScheduler {
    @Autowired private BoardService service;

    @Scheduled(cron = "${sample.scheduleOne.cron}")
    // alternatives (only one active at a time): fixedDelayString / fixedRateString
    public void taskWorkOne() {
        int maxId = service != null ? service.selectMaxBoardId(new BoardParam()) : 0;
        log.info("maxId : " + maxId);
    }
}
```

`module-config.properties` convention: one module-wide `<module>.scheduleConfig.enable` flag gates every schedule in that module; each job then has its own `<module>.<jobName>.enable` flag plus three mutually-exclusive trigger properties (only one uncommented at a time, matching whichever `@Scheduled` attribute is active in code):

```properties
sample.scheduleConfig.enable=false   # false disables ALL schedules in this module regardless of per-job flags

sample.scheduleOne.enable=true
sample.scheduleOne.cron=*/1 * * * * *
#sample.scheduleOne.fixedDelayString=1000
#sample.scheduleOne.fixedRateString=2000
```

### 13.2 Quartz-based Scheduling

For heavier/CPU-bound batch jobs, the dedicated `tsb.fontos.sample.schedule` module shows a full Quartz setup: register `JobDetail`/`Trigger` beans, extend `QuartzJobBean` for the job itself, and isolate heavy jobs onto their own bounded thread pool so they can't starve lighter scheduled work:

```java
package tsb.fontos.sample.schedule;

import org.quartz.CronScheduleBuilder;
import org.quartz.JobBuilder;
import org.quartz.JobDetail;
import org.quartz.Trigger;
import org.quartz.TriggerBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configuration
public class QuartzJobRegister {
    @Bean
    public JobDetail normalJobOneDetail() {
        return JobBuilder.newJob(NormalQuartzJobOne.class).withIdentity("normalJobOne").storeDurably().build();
    }

    @Bean
    public Trigger normalJobOneTrigger(JobDetail normalJobOneDetail) {
        return TriggerBuilder.newTrigger().forJob(normalJobOneDetail)
                .withSchedule(CronScheduleBuilder.cronSchedule("0/2 * * * * ?")).build();
    }
}

@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean("heavyJobExecutor")
    public ThreadPoolTaskExecutor heavyJobExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(2); // cap concurrent heavy jobs regardless of Quartz thread pool size
        return executor;
    }
}
```

### 13.3 Task Monitoring (Micrometer)

Module `tsb.fontos.core.micrometer` adds Prometheus-backed metrics for scheduled/task methods without any manual instrumentation:

```properties
fontos.management.endpoints.scheduler.enabled=true
```

```java
// excerpt from an @Aspect class in tsb.fontos.core.micrometer
import java.util.concurrent.TimeUnit;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;

import io.micrometer.core.instrument.Timer;
import tsb.fontos.core.micrometer.annotation.TSchedulerTaskMonitor;

@Around("@annotation(tSchedulerTaskMonitor)")
public Object monitorTaskExecution(ProceedingJoinPoint joinPoint, TSchedulerTaskMonitor tSchedulerTaskMonitor) throws Throwable {
    String taskName = tSchedulerTaskMonitor.name();
    Timer taskTimer = getOrCreateTimer(taskName);
    long start = System.nanoTime();
    try {
        Object result = joinPoint.proceed();
        getOrCreateSuccessCounter(taskName).increment();
        return result;
    } catch (Exception e) {
        getOrCreateFailureCounter(taskName).increment();
        throw e;
    } finally {
        taskTimer.record(System.nanoTime() - start, TimeUnit.NANOSECONDS);
    }
}
```

Annotate any scheduled/task method with `@TSchedulerTaskMonitor(name = "...")` (see §13.1) to get `scheduler.task.timer`, `scheduler.task.success`, and `scheduler.task.failure` metrics, tagged by task name, exposed through Actuator/Prometheus. `TThreadPoolTaskSchedulerMetrics` additionally exposes pool-level gauges (`scheduler.active_tasks`, `scheduler.thread_pool_size`, etc.) for a given `ThreadPoolTaskScheduler`.

### 13.4 Manual Scheduling (`TSchedulingConfigurer` / `TManualScheduler`)

When the schedule interval itself needs to come from a runtime source (e.g. looked up from the database rather than a static property), use manual registration instead of `@Scheduled`. A config class extends `TSchedulingConfigurer` and overrides `DoAddSchedule(...)` to register one or more `Runnable` tasks against a `ScheduledTaskRegistrar`:

```java
package tsb.fontos.sample.scheduler;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;

import lombok.extern.slf4j.Slf4j;
import tsb.fontos.core.scheduler.TSchedulingConfigurer;
import tsb.fontos.sample.param.board.BoardParam;
import tsb.fontos.sample.service.board.BoardService;

@Slf4j
@Configuration
@ConditionalOnProperty(name = "sample.scheduleConfig.enable", havingValue = "true")
@EnableScheduling
public class SampleSchedulerConfig extends TSchedulingConfigurer {
    @Autowired private ApplicationContext context;
    @Autowired private BoardService service;

    @Override
    public void DoAddSchedule(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        // if schedule intervals live in the DB, look them up here (framework does not do this for you)
        if (service != null) {
            log.info("maxId : " + service.selectMaxBoardId(new BoardParam()));
        }

        CustomSampleOneScheduler customOne = context.getBean(CustomSampleOneScheduler.class);
        scheduledTaskRegistrar.addFixedRateTask(createRunnable(customOne), "*/4 * * * * *");

        CustomSampleTwoScheduler customTwo = context.getBean(CustomSampleTwoScheduler.class);
        scheduledTaskRegistrar.addFixedDelayTask(createRunnable(customTwo), 3000);
    }
}
```

Each task class extends `TManualScheduler`, implements its work method, and overrides `getTaskMethodName()` to tell the framework (via reflection) which method to invoke:

```java
package tsb.fontos.sample.scheduler;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import lombok.extern.slf4j.Slf4j;
import tsb.fontos.core.scheduler.TManualScheduler;
import tsb.fontos.sample.param.board.BoardParam;
import tsb.fontos.sample.service.board.BoardService;

@Slf4j
@Component(value = "customSampleOneScheduler")
public class CustomSampleOneScheduler extends TManualScheduler {
    @Autowired private BoardService service;

    public void taskWorkOne() {
        if (service != null) {
            log.info("list size : " + service.selectBoardAll(new BoardParam()).size());
        }
    }

    @Override
    public String getTaskMethodName() {
        return "taskWorkOne";
    }
}
```

**Calling another module's REST API from a scheduled task** — a common pattern for cross-module polling jobs, using the framework's `RestTemplate`-based helpers and response wrappers:

```java
// excerpt continuing a TManualScheduler subclass with REST-client helper methods
import java.util.List;

import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriComponentsBuilder;

import tsb.fontos.core.http.rest.TRestErrorResponse;
import tsb.fontos.core.http.rest.TRestNormalResponse;
import tsb.fontos.sample.item.board.BoardItem;

public void taskWorkGet() {
    String url = baseUrl + "/boardAllByNewResponse";
    UriComponentsBuilder uriBuilder = UriComponentsBuilder.fromHttpUrl(url)
            .queryParam("title", "12").queryParam("id", "1223");

    RestTemplate restTemplate = new RestTemplate(this.getClientHttpRequestFactory(10000)); // 10s timeout
    TRestErrorResponse errorResponse = null;
    TRestNormalResponse<List<BoardItem>> response = null;
    try {
        response = restTemplate.getForObject(uriBuilder.toUriString(),
                (new TRestNormalResponse<List<BoardItem>>()).getClass(), new HttpEntity<>(new HttpHeaders()));
    } catch (Exception e) {
        errorResponse = this.ParseErrorInfo(e);
    }
    if (errorResponse != null) {
        log.error(errorResponse.getResult().getMsg());
        return;
    }
    // ... process response.getResult()
}
```

`this.getClientHttpRequestFactory(timeoutMs)` and `this.ParseErrorInfo(e)` imply this scheduler extends a framework base class providing REST-client helpers (not named in the source material — confirm the exact base class against the framework source if you need to extend it yourself).

---

## 14. Build & Deployment

### 14.1 Per-site Build Configuration

Each deployable web-app project keeps a `buildConfigs/` folder (name is fixed — the tooling looks for it literally) with one subfolder per deployment target, each carrying its own copy of the environment-sensitive files:

```
tsb.fontos.sample/
└── buildConfigs/
    ├── tomcat/
    │   ├── app-config.properties
    │   ├── application.properties
    │   ├── application-context.xml
    │   ├── envbuild.gradle
    │   └── sql-mappeer-config.xml
    ├── wildfly/
    └── wildfly.terminal_id/
```

`build.gradle` pulls in an `envbuild.gradle` from the selected site folder:

```gradle
plugins {
    id 'org.springframework.boot' version '2.6.7'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id 'war'
}
apply from: './build_external.gradle'
```

`envbuild.gradle` is where you override dependency versions, add WAS-specific drivers, or exclude libraries that conflict with the target application server's own classpath:

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web:2.5.6'
    implementation 'com.oracle.ojdbc:ojdbc8:19.3.0.0'
}
configurations.all {
    exclude group: 'org.hibernate.validator', module: 'hibernate-validator'
    exclude group: 'com.zaxxer', module: 'HikariCP'
    exclude group: 'org.apache.tomcat.embed', module: 'tomcat-embed-jasper'
}
```

`build.properties` selects which site folder is active and which files from it to apply:

```properties
# DEV: local development; otherwise the folder name under buildConfigs/
deployment.env=DEV
#deployment.env=tomcat
#deployment.env=wildfly

deployment.env.buildFiles=envbuild.gradle
deployment.env.configFiles=app-config.properties,application.properties,application-context.xml,sql-mappeer-config.xml
```

Multiple files can be comma-separated in either property if a site needs more than one override file.

### 14.2 WAR Packaging

Standard Spring Boot WAR setup — `war` plugin + `SpringBootServletInitializer` subclass — deployable to Tomcat or WildFly:

```gradle
bootWar {
    archiveName("${p_artifact}-${p_version}.war")
}
```

```java
package tsb.fontos.sample;

import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

public class SampleServletInitializer extends SpringBootServletInitializer {
    // calls TApplicationResouceConfigInit.Load(...) during WAR bootstrap,
    // mirroring what SampleApplication.main() does for the embedded-server dev mode
}
```

WildFly-specific deployment descriptors (`jboss-web.xml`, `jboss-deployment-structure.xml`) live under `src/main/webapp/WEB-INF/` and are only relevant when `deployment.env=wildfly*`.

### 14.3 WildFly / JBoss WAR Deployment & JNDI DataSource

Deploying a Spring Boot–based FONTOS project to WildFly (tested against WildFly 11, Java 8) needs a bit more than the generic WAR packaging in [14.2](#142-war-packaging): even though the app is Spring Boot–based, WildFly deployment isn't fundamentally different from a classic WAR — add the WAS-specific config files below and deploy as usual (`war`, `bootWar`, and `bootJar` artifacts all work with the right settings).

**1. Add `jboss-deployment-structure.xml`** under `src/main/webapp/WEB-INF/` — required for WildFly to control class loading correctly (fixes issues such as log file creation failing under the WAS classloader).

**2. JNDI datasource wiring** — keep a separate Spring context file for the JNDI variant rather than editing the default one:

```properties
# app-config.properties — point at the JNDI-flavored context instead of the classpath default
env.appconfig.app-context=classpath:application-context-jndi.xml
```

```properties
# database-Info.properties (or equivalent env config)
custom.one.datasourcename=java:jboss/jdbc/if
```

```properties
# application.properties
spring.datasource.jndi-name=${custom.one.datasourcename}
```

**3. Gradle dependency exclusions** — avoid classpath conflicts between the app's embedded libraries and WildFly's own (HikariCP, hibernate-validator, springdoc, and embedded-Tomcat's Jasper in particular cause real problems):

```gradle
configurations.all {
    exclude group: 'org.hibernate.validator', module: 'hibernate-validator'
    exclude group: 'org.springdoc', module: 'springdoc-openapi-ui'
    exclude group: 'com.zaxxer', module: 'HikariCP'
    // org.apache.tomcat.embed:tomcat-embed-jasper can also cause
    // NoClassDefFoundError: org/apache/juli/logging/LogFactory — exclude if hit
}
```

**4. WAS-level settings**: Spring Boot 5+ can throw `ModuleNotFoundException: jdk.unsupported` on WildFly — fix by editing `<WILDFLY_HOME>/modules/system/layers/base/jdk/unsupported/main/module.xml` and restarting the server. The JNDI binding itself (name + target datasource) must also be configured on the WildFly side, independent of the app's own properties, for the `java:jboss/jdbc/if` lookup to resolve.

### 14.4 External Configuration File Resolution

A module's runtime config files (DB connection, logging, messages, module-level properties) are located via `app-config.properties`, which supports **three interchangeable path-resolution modes** — the same property keys, different `path-prefix`/`root-path` values, letting ops relocate config outside the packaged JAR/WAR without a rebuild:

```properties
##### Mode 1: classpath-based (config packaged inside the artifact) #####
env.appconfig.path-prefix=classpath:
env.appconfig.root-path=
env.appconfig.config-path=SL   # the module ID, e.g. from module-Info.properties

##### Mode 2: file, absolute path #####
#env.appconfig.path-prefix=file:
#env.appconfig.root-path=D:\\Temp\\catosWebAppConfigHome
#env.appconfig.config-path=SL

##### Mode 3: file, path from a JVM system property #####
#env.appconfig.path-prefix=file:
#env.appconfig.root-path=systemProperty:catos.home
#env.appconfig.config-path=SL

env.appconfig.app-property=classpath:application.properties
env.appconfig.app-context=classpath:application-context.xml
env.appconfig.module-info=classpath:module-Info.properties

env.module.database-info=database-Info.properties
env.module.log-info=logback-spring.xml
env.module.msg-info=messages
env.module.config-info=module-config.properties
```

Mode 3 is activated with a JVM system property at launch, letting the same artifact point at different config roots per environment without touching build output:

```
java -jar tsb-fontos-sample-0.1.0.jar -Dcatos.home=D:\Temp\catosWebAppConfigHome
```

`module-Info.properties` identifies the module itself (`module.info.module.id=SL`, plus product id/name/title/version) — `module.info.module.id` is what `env.appconfig.config-path` typically points at, so the config folder name matches the module.

**Loading it at startup** — both the bootJar (`main()`) and WAR (`SpringBootServletInitializer`) entry points call the same loader, via a base class:

```java
package tsb.fontos.sample;

import org.springframework.boot.SpringApplication;

import tsb.fontos.core.web.TApplicationResoucedConfigInit;
import tsb.fontos.core.web.TBaseResourceAppInitializer;

public class SampleApplication extends TBaseResourceAppInitializer {
    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(SampleApplication.class);
        springApplication.addInitializers(applicationContext -> {
            TApplicationResoucedConfigInit.loadApplicationProperty(applicationContext);
        });
        springApplication.run(args);
    }
}
```

```java
package tsb.fontos.sample;

import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

import tsb.fontos.core.web.TApplicationResoucedConfigInit;

public class SampleServletInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        TApplicationResoucedConfigInit.Load();
        SpringApplicationBuilder appBuilder = application.sources(SampleApplication.class);
        appBuilder.application().addInitializers(applicationContext ->
                TApplicationResoucedConfigInit.LoadApplicationProperty(applicationContext));
        return appBuilder;
    }
}
```

(Class name transcribed as `TApplicationResoucedConfigInit` in the source material — likely a typo for "Resource"; confirm the exact spelling against the framework source before using it.)

**Reading a resolved property in code** — three equivalent ways:

```java
// excerpt from any Spring-managed bean
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.env.Environment;

import tsb.fontos.core.app.TApplicationContextProvider;

@Value("${test.aaa}") private String tempstr;                      // field injection
String tempstr2 = TApplicationContextProvider.getProperty("test.aaa"); // static helper
@Autowired private Environment env;
String tempstr3 = env.getProperty("test.aaa");                      // Environment reference
```

> Note the interaction with [14.3](#143-wildfly--jboss-war-deployment--jndi-datasource): the WildFly guide overrides `env.appconfig.app-context` to `classpath:application-context-jndi.xml`, while the default here is `classpath:application-context.xml` — treat this as a per-environment override of the same key, not a separate mechanism.

### 14.5 Legacy Gradle Profile-based Resource Folders

An older/alternate mechanism for per-site config exists alongside the `buildConfigs/{site}/` convention in [14.1](#141-per-site-build-configuration) — **don't mix the two** in the same project; pick one. This one selects sibling `src/main/resources-{profile}` folders via a Gradle project property at build time:

```gradle
sourceSets {
    ext.profile = (!project.hasProperty('profile') || !profile) ? 'local' : profile
    main {
        resources {
            srcDirs "src/main/resources", "src/main/resources-${profile}"
        }
    }
}
tasks {
    processResources {
        duplicatesStrategy = org.gradle.api.file.DuplicatesStrategy.INCLUDE // last srcDir wins on same-named files
    }
}
```

Build/run with `-Pprofile=<name>` (e.g. as a Gradle Task program argument in STS: `-Pprofile=site2`), which merges `src/main/resources-site2/*` over the defaults in `src/main/resources/*`. Only the config files that actually vary per site need to exist under `resources-{profile}` — everything else falls through to the shared `resources` folder.

> This predates (or is a simpler alternative to) the `buildConfigs/{site}/` + `envbuild.gradle` + `build.properties` convention in §14.1 — treat it as legacy/alternate, not something to combine with the newer convention in the same project.

### 14.6 SSL/TLS Configuration

Setting up HTTPS for local/dev testing, across three deployment targets (embedded Spring Boot/Tomcat, standalone Tomcat, and JEUS). All examples below use developer-test certificates — regenerate real certificates (CA-signed, not self-signed) for anything beyond local testing.

**Generating a test keystore/certificate** (`keytool`, bundled with the JDK):

```
keytool -genkey -alias ssltest -storetype PKCS12 -keyalg RSA -keystore ".keystore" -validity 4000
keytool -export -alias ssltest -keystore ".keystore" -rfc -file "cert.cer"
keytool -import -alias ssltest -file "cert.cer" -keystore "cert.crt"
```

**Spring Boot** (`application.properties`):

```properties
server.ssl.key-store: /path/to/.keystore
server.ssl.key-store-password: ${SSL_KEYSTORE_PASSWORD}
server.ssl.keyStoreType: PKCS12
server.ssl.keyAlias: ssltest
```

The same settings can instead be passed as JVM system properties (IDE Run Configuration VM args, or a `java -jar` command line) — useful when the keystore path/password shouldn't live in a checked-in properties file:

```
-Dserver.ssl.key-store=file:/path/to/keystore1.p12
-Dserver.ssl.key-store-password=${SSL_KEYSTORE_PASSWORD}
-Dserver.ssl.keyStoreType=PKCS12
-Dserver.ssl.keyAlias=ssltest
```

**Tomcat** (`conf/server.xml` + `conf/web.xml`):

```xml
<!-- server.xml -->
<Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol"
    maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
    keystoreFile="/path/to/.keystore" keystorePass="${SSL_KEYSTORE_PASSWORD}"
    clientAuth="false" sslProtocol="TLS" />
```

```xml
<!-- web.xml — force all URLs to redirect to HTTPS -->
<security-constraint>
    <web-resource-collection>
        <web-resource-name>Protected Context</web-resource-name>
        <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <user-data-constraint>
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>
</security-constraint>
```

**JEUS** (`domain.xml`, per-listener) — prefer paths relative to the domain's `config/ssl/` folder over absolute paths, for portability across environments:

```xml
<listener>
    <name>http-server</name>
    <listen-address>0.0.0.0</listen-address>
    <listen-port>8808</listen-port>
    <ssl>
        <client-auth>Unnecessary</client-auth>
        <keystore-file>ssl/.keystore</keystore-file>
        <keystore-pass>${SSL_KEYSTORE_PASSWORD}</keystore-pass>
        <keystore-keypassword>${SSL_KEYSTORE_KEY_PASSWORD}</keystore-keypassword>
        <keystore-type>JKS</keystore-type>
        <truststore-file>ssl/glv_rcasp_crt.crt</truststore-file>
        <truststore-pass>${SSL_TRUSTSTORE_PASSWORD}</truststore-pass>
        <truststore-type>JKS</truststore-type>
    </ssl>
</listener>
```

Also configurable via the JEUS WebAdmin console (`http://<host>:<port>/webadmin/applications` → Listeners → SSL properties), where password fields are masked in the UI.

> **Security flag**: the internal training material for this topic reuses **one hardcoded keystore/truststore password across all of the Spring Boot, Tomcat, and JEUS examples** (7+ occurrences). Never inline a real password in documentation or config examples — use a placeholder (as shown above) and source the real value from an environment variable, secrets manager, or Jasypt-encrypted property. This applies even to "dev/test" credentials once they're committed anywhere shared.

A self-signed dev certificate will trigger a browser trust warning (`NET::ERR_CERT_AUTHORITY_INVALID` in Chrome) — expected for local testing; proceed past the warning locally, but use a CA-signed certificate before any non-local use.

### 14.7 Bundling the FONTOS Frontend Build into the Backend WAR

This subsection covers the backend-side half of frontend/backend deployment integration — the frontend build process and React runtime details (context-path handling, `REACT_APP_API_URL`) belong to the companion `[tsb-fontos-ui] Developer Guide`, not here.

**Background**: a FONTOS frontend build (Create React App) can be deployed with either a hardcoded, environment-specific absolute path (old approach: `PUBLIC_URL=/tsb-fontos-webprototype` baked in at build time, requiring a rebuild per deployment context path) or a relative-path build (`"homepage": "."` in `package.json`, so all static asset paths resolve relative to whatever context path the browser is actually loaded from). The relative-path approach means **the same frontend build artifact can be bundled under any backend WAR context path without a frontend rebuild** — this is the deployment-relevant payoff of a frontend-side setting.

**1. Point the backend build at the frontend build output** — add a `build.properties` file to the backend project (values are local-workspace paths in the example; adjust per developer/CI machine):

```properties
frontend.packageDir=E:\\React_Workspace\\...\\Fontos-Framework_Template\\packages
frontend.buildDir=E:\\React_Workspace\\...\\Fontos-Framework_Template\\packages\\Fontos-Sample-Frontend\\build

frontend.build.batchFile=buildWebPage.bat
frontend.build.args1=E:
frontend.build.args2=E:\\React_Workspace\\...\\Fontos-Framework_Template\\packages
frontend.build.args3=fontos-sample-frontend
frontend.build.args4=buildProd_SITE01
```

**2. A batch script that shells out to the yarn workspace build** (`buildWebPage.bat`, kept at the backend project root, generic enough to reuse per module by just changing `build.properties`):

```bat
@echo on
@echo "arg 1=>" %1
@echo "arg 2=>" %2
@echo "arg 3=>" %3
@echo "arg 4=>" %4

%1
cd %2
yarn workspace %3 %4
pause
```

**3. `build.gradle` reads those properties and defines two Gradle tasks** — one to build the frontend from source and copy it in, one to just re-copy an already-built frontend:

```gradle
Properties properties = new Properties()
properties.load(project.file('build.properties').newDataInputStream())

def frontendPackageDir = properties.getProperty('frontend.packageDir')
def frontendBuildDir = properties.getProperty('frontend.buildDir')
def frontendBuildBatchFile = properties.getProperty('frontend.build.batchFile')
def frontendBuildArgs1 = properties.getProperty('frontend.build.args1')
def frontendBuildArgs2 = properties.getProperty('frontend.build.args2')
def frontendBuildArgs3 = properties.getProperty('frontend.build.args3')
def frontendBuildArgs4 = properties.getProperty('frontend.build.args4')

def buildWebpageFiles = {
    def batchFile = "${project.projectDir}\\${frontendBuildBatchFile}"
    if (!file(batchFile).exists()) {
        println "[WARNING] The webpage build batch file does not exist. path: ${batchFile}"
        return
    }
    exec {
        commandLine 'cmd', '/c', "${batchFile}", "${frontendBuildArgs1}", "${frontendBuildArgs2}",
                "${frontendBuildArgs3}", "${frontendBuildArgs4}"
    }
}

def copyWebpageFiles = {
    def webPageBuildDir = file("${frontendBuildDir}").getAbsolutePath()
    if (!file(webPageBuildDir).exists()) {
        println "[WARNING] The webpage build folder does not exist. path: ${webPageBuildDir}"
        return
    }
    def targetDir = file("${project.projectDir}\\build\\resources\\main\\public")
    if (!targetDir.exists()) targetDir.mkdirs()
    copy {
        from fileTree(dir: webPageBuildDir)
        into targetDir
    }
}

task includeOnlyCopyWebPage {
    doLast { copyWebpageFiles() } // frontend already built — just (re-)copy it in
}

task includeBuildAndCopyWebPage {
    doFirst { buildWebpageFiles() } // build the frontend from source first...
    doLast { copyWebpageFiles() }   // ...then copy the build output in
}
```

The key line is `into "${project.projectDir}/build/resources/main/public"` — this is Spring Boot's classpath static-resource root, so whatever lands there is served as static content and packaged straight into the backend's WAR/JAR.

**4. Build the WAR** via an Eclipse/STS Gradle Task run configuration with Gradle Tasks set to `includeBuildAndCopyWebPage bootWar` (rebuild frontend + repackage) or `includeOnlyCopyWebPage bootWar` (frontend already built, just repackage) — output lands in `build/libs/<artifact>.war` as usual (see [14.2](#142-war-packaging)).

**5. Serve the frontend's `index.html` at the WAR's root path** with a minimal controller:

```java
package tsb.fontos.webprototype.controller;

import java.io.IOException;

import javax.servlet.http.HttpServletResponse;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class IndexController {
    @GetMapping(value = "/")
    public void index(HttpServletResponse response) throws IOException {
        response.sendRedirect("./index.html");
    }
}
```

---

## 15. Data Synchronization (Observer/Callback Pattern)

This is **not** a WebSocket/push mechanism or a DB-cache-invalidation layer — it's a lightweight, in-process **observer/callback (publish–subscribe) utility**, letting one part of the app notify other registered components of a data change within the same JVM, without polling. Framework types: `DataSyncAgent` (key/String-based) and `DataSyncNotifiedHandler`/`DataSyncNotifiedEventArgs` (the listener contract), plus a "group" style built around a custom subclass (e.g. `SampleGroupDataSync`) for typed change-event objects instead of a String key.

**1. Implement the listener** — override `handle`, reading the changed value via `getTargetDataToSync()`:

```java
package tsb.fontos.sample.controller.observer;

import lombok.extern.slf4j.Slf4j;
import tsb.fontos.core.sync.DataSyncNotifiedEventArgs;
import tsb.fontos.core.sync.DataSyncNotifiedHandler;

@Slf4j
public class SampleNormalDataSyncNotifiedHandler implements DataSyncNotifiedHandler {
    public static final String NORMAL_TEST01_DATA_CHANGED = "NORMAL_TEST01_DATA";

    @Override
    public void handle(Object sender, DataSyncNotifiedEventArgs e) {
        Object targetData = e.getTargetDataToSync();
        if (targetData != null) {
            log.debug(targetData.toString());
        }
    }
}
```

**2. Register it** — in a `@Component`-annotated config class, declare `@Bean` factory methods that construct the agent and register the listener:

```java
package tsb.fontos.sample.controller.observer;

import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import tsb.fontos.core.sync.DataSyncAgent;

@Component
public class SampleDataSyncAgentInit {
    @Bean
    public SampleGroupDataSync groupDataSync() {
        SampleGroupDataSync dataSync = new SampleGroupDataSync();
        dataSync.addCallbackListener(new SampleGroupDataSyncNotifiedHandler()); // group/listener-object style
        return dataSync;
    }

    @Bean
    public DataSyncAgent normalDataSyncAgent() {
        DataSyncAgent dataSync = new DataSyncAgent();
        dataSync.addCallbackInfo(SampleNormalDataSyncNotifiedHandler.NORMAL_TEST01_DATA_CHANGED,
                new SampleNormalDataSyncNotifiedHandler()); // key/String-based style
        return dataSync;
    }
}
```

**3. Trigger a notification** — `@Autowired`-inject the registered bean wherever the data change originates, then call `notifyToSync(key, data)` (keyed style) or `notifyToViewSync(changeEventParam)` (group style):

```java
package tsb.fontos.sample.controller.observer;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import tsb.fontos.core.sync.DataSyncAgent;
import tsb.fontos.core.web.TBaseRestController;

@RestController
public class DataSyncTestOneController extends TBaseRestController {
    @Autowired private SampleGroupDataSync groupDataSync;
    @Autowired private DataSyncAgent normalDataSyncAgent;

    @GetMapping("/dataSync-group/{message}")
    public String groupDataSync(@PathVariable String message) {
        SampleChangedDataParam param = new SampleChangedDataParam(SampleDataChangeType.VOYAGE_CHANGED, this, message);
        groupDataSync.notifyToViewSync(param); // use the injected bean — see caution below
        return message;
    }

    @GetMapping("/dataSync-normal/{message}")
    public String notifyNormal(@PathVariable String message) {
        normalDataSyncAgent.notifyToSync(SampleNormalDataSyncNotifiedHandler.NORMAL_TEST01_DATA_CHANGED, message);
        return message;
    }
}
```

No application/YAML configuration is involved — this mechanism is wired entirely through `@Component`/`@Bean`/`@Autowired` and plain Java calls. Package convention: `tsb.fontos.sample.controller.observer` in the sample project.

---

## 16. Database & Datasource Configuration

### 16.1 Multi-Dialect Mapper Switching

For projects that must support more than one database dialect (e.g. MySQL vs. MSSQL) from the same codebase, MyBatis mapper XML files are split into sibling source folders per dialect — `mapper.mssql.<module>`, `mapper.mysql.<module>` — and a single property picks which folder set is scanned:

```properties
# application.properties
#app.dbType=mysql
app.dbType=mssql
```

```xml
<!-- application-context.xml -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:sql-mapper-config.xml" />
    <property name="mapperLocations" value="classpath:mapper/${app.dbType}/**/*.xml" />
</bean>
```

Related beans: `tsb.fontos.core.dao.TSqlSessionInfo` (wraps a `SqlSessionTemplate`), `tsb.fontos.core.dao.TSqlSessionManager` (property `targetSqlSessions`, a map from a session key to a `TSqlSessionInfo`, keyed either by plain `String` or by a `DataSourceType` enum). This is purely a *mapper-selection* switch — see [16.2](#162-multi-datasource-routing) for routing actual connections/transactions across multiple databases.

> This is the only concrete mechanism in the source training material about switching database dialects — it does not cover JDBC driver class names, Connector/J version pinning, or connection-string tuning; confirm those separately against your actual driver dependency (see the module reference in the Appendix).

### 16.2 Multi-Datasource Routing

For multi-terminal/multi-tenant deployments where a request must be routed to a different physical database depending on context (session attribute, request param, etc.), the framework provides a routing `DataSource` plus an AOP-driven key-decision mechanism.

> The framework's actual class/property names in this package consistently use the misspellings **"Datasouce"/"DataSouce"/"Decistion"** (missing "r" in "source"; extra "t" in "decision") — preserve these verbatim in code, since they reflect the real API, not a documentation typo.

```xml
<bean id="dataSource" class="tsb.fontos.core.datasouce.TRoutingDataSource">
    <property name="targetDataSources">
        <map key-type="java.lang.String">
            <entry key="ONE" value-ref="oneDataSource" />
            <entry key="TWO" value-ref="twoDataSource" />
        </map>
    </property>
    <property name="defaultTargetDataSource" ref="oneDataSource" />
</bean>

<bean id="oneRoutingDataSouceKeyDecistion" class="tsb.fontos.sample.config.TOneRoutingDataSouceKeyDecistion" />

<bean id="oneRoutingDsAdvisor" class="tsb.fontos.core.datasouce.TRoutingDataSouceDecisionAspect">
    <property name="defaultDataSourceKey" value="ONE"/>
    <property name="routingDataSouceKeyDecision" ref="oneRoutingDataSouceKeyDecistion"/>
    <property name="decistionFilterAnnotation" value="org.springframework.stereotype.Service"/>
</bean>

<aop:config>
    <aop:aspect ref='oneRoutingDsAdvisor'>
        <aop:pointcut id="point1" expression="execution(* tsb.fontos.sample.service.one.board.**.*(..))" />
        <aop:before method="beforeMethod" pointcut-ref="point1" />
        <aop:after method="afterMethod" pointcut-ref="point1" />
        <aop:around method="aroundMethod" pointcut-ref="point1" />
    </aop:aspect>
</aop:config>
```

The routing key is resolved per-request by a `TBaseRoutingDataSouceKeyDecistion` subclass — check the session first, then an explicit param, then fall back to the default:

```java
package tsb.fontos.sample.config;

import javax.servlet.http.HttpSession;

import tsb.fontos.core.datasouce.IDataSourceParam;
import tsb.fontos.core.datasouce.TBaseRoutingDataSouceKeyDecistion;
import tsb.fontos.core.util.TStringUtil;
import tsb.fontos.sample.constant.DataSourceKeys;

public class TOneRoutingDataSouceKeyDecistion extends TBaseRoutingDataSouceKeyDecistion {
    @Override
    public String getDefaultDataSourceKey() {
        return DataSourceKeys.ONE;
    }

    @Override
    public String getDataSourceKey(HttpSession session, Object arg) {
        if (session != null) {
            Object v = session.getAttribute("DataSouceType");
            if (v != null) return v.toString();
        }
        if (arg instanceof IDataSourceParam) {
            IDataSourceParam param = (IDataSourceParam) arg;
            if (!TStringUtil.isNullOrWhiteSpace(param.getDataSouceType())) {
                return param.getDataSouceType();
            }
        }
        return this.getDefaultDataSourceKey();
    }
}
```

Pin a `@Service` to a specific routing key with the custom `@TRDataSource` annotation:

```java
package tsb.fontos.sample.service.board;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import tsb.fontos.core.datasouce.annotation.TRDataSource;
import tsb.fontos.core.service.TBaseRestService;
import tsb.fontos.sample.constant.DataSourceKeys;
import tsb.fontos.sample.dao.board.BoardDao;

@Service
@TRDataSource(DataSourceKeys.ONE)
public class BoardService extends TBaseRestService {
    @Autowired private final BoardDao boardDao;
}
```

**Transaction manager choice depends on the deployment shape** — this is the key decision table for this topic:

| Scenario | Terminal scope | Cross-DB transaction needed? | Transaction manager |
|---|---|---|---|
| Multi-terminal, different DBs | Multi | Yes | `JTA DB Transaction` (JTA/XA distributed TX) |
| Single-terminal, different DBs, cross-DB TX required | Single | Yes | `JTA DB Transaction` |
| Single-terminal, different DBs, no cross-DB TX | Single | No | `DataSourceTransactionManager` (plain Spring local TX per DataSource) |

In the no-cross-DB-transaction case, only the primary DataSource is read/write — secondary DataSources are typically treated as read-only. Use `JTA DB Transaction` only when a single business operation must commit/rollback atomically across more than one physical database; otherwise prefer the simpler `DataSourceTransactionManager` per DataSource.

---

## 17. Web Serial (COM Port) Communication (Frontend Reference)

> **Scope note**: this is entirely a **browser-native Web Serial API** topic (client-side JavaScript, `navigator.serial`) — there is no backend Java library, Gradle dependency, or server-side serial-port class involved. It's included here for completeness since it appears in the same internal training series and one variant integrates with the FONTOS React frontend, but it is not a backend development topic.

### 17.1 Overview

The [Web Serial API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Serial_API) lets a web page request direct access to a serial (COM) port from the browser, without a native driver or plugin. Browser support (Chrome 89+, Edge 89+, Opera 75+) requires enabling **Experimental Web Platform features** at `chrome://flags/` for local testing in some Chrome versions; Firefox and Safari (desktop and mobile) do not support it at all, and no mobile browser does.

### 17.2 Test Environment Setup

1. **Virtual serial port bridge** — install a tool such as HHD Software's "Free Virtual Serial Ports" (or the commercial "Virtual Serial Port Tools") to create a `COM1 <-> COM2` loopback bridge on a single dev machine (no physical hardware needed).
2. **RS232 terminal utility** — install "Hercules SETUP utility" (freeware) and open it on `COM2` with settings `Baud: 9600, Data size: 8, Parity: none, Handshake: OFF, Mode: Free`; use it as a dummy device to send/receive raw text against the browser side on `COM1`.
3. **Generic HTML/JS example**: [UnJavaScripter/web-serial-example](https://github.com/UnJavaScripter/web-serial-example) on GitHub — a minimal reference implementation with no framework dependency.

### 17.3 FONTOS React Integration

The FONTOS sample frontend includes a "Serial Web" menu item/component with **Receive**/**Send** fields and an on/off COM-port toggle. To run it locally:

```
yarn install
yarn workspace fontos-sample-frontend start
```

Then in the browser, click the on/off button to trigger the native port-picker dialog (`navigator.serial.requestPort()`), select the bridged COM port, and use the Send/Receive fields — verify round-trip communication against the Hercules utility bound to the other end of the bridge. On the backend project side, a static `serial/` folder (`dist/app.js`, `dist/serial-handler.js`, `index.html`, `styles.css`) is deployed under `src/main/resources/public/` for the plain HTML/JS variant, served at a URL like `http://localhost:8088/serial/index.html`.

---

## 18. Migrating to JDK 17 / Spring Boot 3.x (V2)

> **Scope note**: this is a forward-looking migration track, not the current baseline. Everything else in this guide documents the framework's current, in-production baseline (**Java 8, Spring Boot 2.6.7**, per [1. Introduction](#1-introduction)). This section documents a separate, parallel effort to migrate to **Java 17 / Spring Boot 3.5.13**, tracked in newly created `-V2`-suffixed Git repositories (`Fontos-Framework-Backend-V2`, `Fontos-Sample-Backend-V2`, `Fontos-Sample-WebPrototype-V2`, etc.) alongside — not replacing — the existing repos. Confirm with the EA team whether a given project should be built against the V1 (Java 8) or V2 (Java 17) baseline before applying anything below.

### 18.1 Migration Baseline

| Component | Existing (V1) | Migrated (V2) |
|---|---|---|
| Java | JDK 8 | **JDK 17+** |
| Spring Boot | 2.6.7 | **3.5.13** |
| Spring Framework | 5.x | 6.x |
| Spring Security | 5.x | 6.x |
| Servlet API | `javax.servlet` | `jakarta.servlet` |
| Tomcat | 9.x | 10.x |
| Gradle wrapper | (see [3.2](#32-multi-module-gradle-setup)) | **8.14.4** |
| `io.spring.dependency-management` | 1.0.11.RELEASE | **1.1.7** |

The module list itself is unchanged by the migration (confirmed against the V2 Eclipse project import: `tsb.fontos.core`, `tsb.fontos.security`, `tsb.fontos.net`, `tsb.fontos.cnet`, `tsb.fontos.report.jasper`, `tsb.fontos.report.openhtmltopdf`, `tsb.fontos.core.micrometer`, plus the `tsb.fontos.sample.*` family) — this is purely a toolchain/library-version migration, not a restructuring of the multi-module layout in [3.2](#32-multi-module-gradle-setup).

> **Gap vs. a typical JDK 8→17 migration**: the source migration procedure this section is based on does **not** cover `javax.xml.bind`/JAXB removal, JPMS/module-system issues (`--add-opens`/`--add-exports`, illegal-reflective-access), or JDK17 GC/JVM tuning flags — despite "JDK17" being in its title, it is functionally a Spring Boot 2→3 / Spring Security 5→6 / Jakarta EE9 migration guide, with JDK 17 mostly showing up as a compiler-level requirement plus two library-compatibility callouts (below). If your project uses JAXB (relevant to [9. SOAP Web Services](#9-soap-web-services), which generates JAXB-annotated classes from WSDL) or has other `javax.*` → JDK-removed-API dependencies, budget separate investigation time — it isn't covered by this migration material.

### 18.2 Dev Environment & Build Setup

1. Install JDK 17 (e.g. from [jdk.java.net](https://jdk.java.net/java-se-ri/17)); in Eclipse/STS, add it under **Installed JREs** and set **Compiler Compliance Level = 17** (check "Use `--release` option").
2. Pull the `-V2` repositories requested from the team's repo-provisioning process, and import as **Existing Gradle Projects**.
3. Replace the Gradle wrapper with **8.14.4** (copy the unzipped `gradle/`, `gradlew`, `gradlew.bat` into the project root).
4. Update `settings.gradle`'s sibling-project `projectDir` references to point at the new `-V2` repo paths (same DEV-TOOL pattern as [3.2](#32-multi-module-gradle-setup), just repointed):

```gradle
include ':tsb.fontos.core'
project(':tsb.fontos.core').projectDir = new File(settingsDir, '../../../Tsb-Fontos-Framework/Fontos-Framework-Backend-V2/tsb.fontos.core')
```

5. **`build.gradle` plugin block** — Spring Boot 3.5.13, dependency-management 1.1.7, target Java 17:

```gradle
plugins {
    id 'java'
    id 'war'
    id 'maven-publish'
    id 'org.springframework.boot' version '3.5.13'
    id 'io.spring.dependency-management' version '1.1.7'
    id 'org.sonarqube' version '3.4.0.2513'
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
    options.compilerArgs.add("-parameters")
}
```

6. `bootJar`/`bootWar`/`war` packaging (compare with [14.2](#142-war-packaging)):

```gradle
bootJar { archiveBaseName = p_artifact; archiveVersion = p_version }
bootWar { archiveBaseName = p_artifact; archiveVersion = p_version; archiveExtension = 'war' }
war     { archiveBaseName = p_artifact; archiveVersion = p_version }
```

### 18.3 Library Version Matrix

| Category | Library | V1 | V2 | Note |
|---|---|---|---|---|
| Database | MyBatis (Spring Boot starter) | 2.2.2 | **3.0.4** | Jakarta EE–compatible version |
| | Oracle `ojdbc8` | 19.3.0.0 | 19.3.0.0 | Unchanged |
| | MySQL Connector | 5.1.31 / 8.x | **8.0.33** | Latest driver recommended |
| Security | JWT (jjwt) | `io.jsonwebtoken:jjwt:0.9.1` (single jar) | **`jjwt-api`/`jjwt-impl`/`jjwt-jackson`:0.12.6** | Split into 3 artifacts — pin all three to the same version |
| | OAuth2 | 2.3.4.RELEASE | Spring Boot starter | `spring-boot-starter-oauth2-client` |
| Reporting | JasperReports | 5.6.1 | **6.21.3** | 5.x throws reflection errors under JDK 17 |
| | PDF engine | iText 2.1.7 | **OpenPDF 1.3.39** | Replaces iText as the PDF engine dependency of JasperReports |
| Monitoring | Spring Boot Admin Client | 2.6.7 | **3.4.5** | Must align with the Boot 3.x BOM |
| Cloud | Spring Cloud | 2021.0.1 | **2024.0.0** | Boot 3.x requires Spring Cloud 2022.x+ |
| Utility | Guava | 11.0.2 | recommend **33.x+** | A `NoSuchMethodError` at build time is a strong signal this is overdue |

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.4'
    implementation 'org.springframework.boot:spring-boot-starter-aop'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'com.github.pagehelper:pagehelper:5.3.0'
    runtimeOnly 'com.microsoft.sqlserver:mssql-jdbc:12.6.0.jre8'
    runtimeOnly 'mysql:mysql-connector-java:8.0.33'
    implementation 'org.springdoc:springdoc-openapi-ui:1.6.11'
    implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6'
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
    implementation 'io.jsonwebtoken:jjwt-api:0.12.6'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.12.6'
    runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.12.6'
    implementation 'com.github.ulisesbocchio:jasypt-spring-boot-starter:3.0.4'
    implementation('net.sf.jasperreports:jasperreports:6.21.3') { exclude group: 'com.lowagie', module: 'itext' }
    implementation 'com.github.librepdf:openpdf:1.3.39'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'io.micrometer:micrometer-registry-prometheus'
    implementation 'de.codecentric:chaos-monkey-spring-boot:2.5.3'
    // ... plus the framework's existing tsb.fontos.* project dependencies, per 3.2
}
```

### 18.4 Jakarta EE9 Namespace Rename

Spring Boot 3.x applies Jakarta EE 9, which renames the `javax.*` namespace to `jakarta.*` for the packages the framework actually uses:

| Old | New |
|---|---|
| `javax.servlet` | `jakarta.servlet` |
| `javax.validation` | `jakarta.validation` |
| `javax.persistence` | `jakarta.persistence` |

```java
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpSession;
import jakarta.validation.Valid;
```

This is a mechanical but pervasive find/replace across every controller, filter, and DAO that imports these packages. As noted in [18.1](#181-migration-baseline), this migration material doesn't address `javax.xml.bind` (JAXB) — check the [SOAP Web Services](#9-soap-web-services) module separately, since wsimport-generated classes depend on JAXB annotations that were removed from the JDK itself (not just renamed) starting at JDK 11.

### 18.5 Spring Security 5→6 DSL Migration

This directly extends [6.1](#61-jwt-based-authentication--authorization)/[6.6](#66-sso--token-lifecycle-management) — Spring Security 6's lambda-based DSL replaces the old fluent/chained style, and `antMatchers` is renamed to `requestMatchers`:

```java
// V1 (Spring Security 5.x) — imports as in §6.1's configure(HttpSecurity) example
import org.springframework.http.HttpMethod;
import org.springframework.security.config.http.SessionCreationPolicy;

http
    .httpBasic().disable()
    .csrf().disable()
    .cors().configurationSource(configurationSourceByProxy()).and()
    .sessionManagement()
        .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
    .and()
    .authorizeRequests()
        .antMatchers(HttpMethod.GET, "/jwt/**").permitAll()
        .antMatchers(HttpMethod.POST, "/jwt/**").permitAll();

// V2 (Spring Security 6.x) — additionally needs:
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;

http
    .httpBasic(AbstractHttpConfigurer::disable)
    .csrf(AbstractHttpConfigurer::disable)
    .cors(cors -> cors.configurationSource(configurationSourceByProxy()))
    .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
    .authorizeHttpRequests(auth -> auth
        .requestMatchers(HttpMethod.GET, "/jwt/**").permitAll()
        .requestMatchers(HttpMethod.POST, "/jwt/**").permitAll());
```

`WebSecurity.antMatchers` is renamed the same way:

```java
// import org.springframework.security.config.annotation.web.builders.WebSecurity; (both V1 and V2)

// V1
@Override
public void configure(WebSecurity web) {
    web.ignoring().antMatchers("/swagger-ui/**");
}

// V2
@Override
public void configure(WebSecurity web) {
    web.ignoring().requestMatchers("/swagger-ui/**");
}
```

**Filter registration** moves toward `@Bean`-based registration instead of directly instantiating and wiring a filter with constructor arguments, and the URI-pattern authorization filter should prefer `AuthorizationManager` over a direct `FilterSecurityInterceptor` reference:

```java
// V1
.addFilterBefore(apiJwtAuthenticateFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class)

// V2 — expose the filter as a bean instead
@Bean
public ApiJwtAuthenticateFilter apiJwtAuthenticateFilter() { /* ... */ }
```

**Modernized collection literals** — since V2 requires Java 17+, prefer `List.of(...)`/`Set.of(...)`/`Map.of(...)` over `Arrays.asList(...)` in new/touched code:

```java
// V1: import java.util.Arrays;
globalConfig.setAllowedOrigins(Arrays.asList("http://localhost:3000"));
// V2: import java.util.List; (Arrays import no longer needed for this line)
globalConfig.setAllowedOrigins(List.of("http://localhost:3000"));
```

**JWT library split** — `io.jsonwebtoken:jjwt:0.9.1` (a single jar) becomes three coordinates in 0.12.6 (`jjwt-api`, `jjwt-impl`, `jjwt-jackson` — see [18.3](#183-library-version-matrix)); pin all three to the same version, since `JwtTokenProvider`/`JwtSSOTokenProvider` and related classes in [6.1](#61-jwt-based-authentication--authorization)/[6.6](#66-sso--token-lifecycle-management) depend on the JJWT API surface, which changed between the two major versions — expect compile errors, not just a version bump, when upgrading this dependency.

### 18.6 Reporting Module Compatibility

[10. Report Generation](#10-report-generation)'s JasperReports-based examples need the version 6.21.3 upgrade in [18.3](#183-library-version-matrix) before running under JDK 17 — version 5.x is reported to throw reflection-related errors on JDK 17. The upgrade also swaps the PDF engine dependency from iText 2.1.7 to OpenPDF 1.3.39; if any report template or export code references iText types directly (rather than going through JasperReports' own API), that code will need to be ported to the OpenPDF equivalents as part of this migration.

---

## 19. Real-World Business Component Walkthrough (Controller, Service, Dao, Mapper)

Sections [4.5](#45-dto-data-transfer-object-projects)–[4.8](#48-ex-external-interface-projects) documented the CATOS+ Dto/Biz/Ct/Ex product-layer split from internal training material, using an illustrative (non-production) `SingleGrid`/trailer-management example. This section cross-checks that documentation against three real, currently-active FONTOS-core backend repositories reviewed directly (not from slide screenshots) while preparing this guide:

| Repository (this machine) | Package root | Role |
|---|---|---|
| `Fontos-Sample-WebPrototype` (`tsb.fontos.webprototype`) | `tsb.fontos.webprototype` | The framework's own sample/reference backend app — already used elsewhere in this guide (JWT/SSO, WebSocket examples) |
| `Most-Jpb` (`most.jpb`) | `most.jpb` | A real production module built on the FONTOS core framework, deployed at a live site (`buildConfigs`-style per-site config, see [14.1](#141-per-site-build-configuration)) |
| `Catos-Plus-Backend-MSSQL` (`catos.ct.ip`) | `catos.ct.ip` | A real CATOS+ **Ct** project (see [4.7](#47-ct-client-projects)) for an IP (in-port/terminal) module, deployed to multiple sites (`buildConfigs/{mmc_fz,mmc_jpb,mmc_odd,wmct}/`) |

> **Naming collision, not a business relationship**: both the illustrative example in [4.5](#45-dto-data-transfer-object-projects)/[4.7](#47-ct-client-projects) and the real code below are called `SingleGrid`. They are unrelated features that happen to share a generic name for "a single flat editable grid" — the illustrative example models trailer/trucker master data (`trailerNo`, `trucker`, `tareWgt`), while the real production feature below models IMDG (dangerous-goods) class reference data (`TB_IMDGDATA`: UN number, IMDG class, packing group, marine pollutant flag, etc.). Don't conflate the two when cross-referencing.

### 19.1 Two Deployment Patterns Seen in Production

**Pattern 1 — single-module** (`tsb.fontos.webprototype`, `most.jpb`): Controller, Service, Dao, Item, Param, and mapper XML all live under one project's own package tree, feature-by-feature — e.g. `most.jpb.{controller|service|dao|item|param}.singlegrid`. There is no separate Dto/Biz split; the Dao interface is bound directly to its mapper XML by namespace, in the same project that also holds the Controller. This is the right fit for a standalone module that has no sibling client needing to reuse its business logic.

**Pattern 2 — multi-module CATOS+ split** (`catos.ct.ip`): confirmed live in this project's `build.gradle`, which depends on separate Gradle subprojects rather than holding Service/Dao/Mapper itself:

```gradle
dependencies {
    // ...
    implementation project(":tsb.fontos.core")
    implementation project(":tsb.fontos.security")
    implementation project(":tsb.fontos.net")
    implementation project(":tsb.fontos.report.jasper")

    implementation project(":catos.cm.core")       // CATOS Common Module
    implementation project(":catos.cm.security")
    implementation project(":catos.cm.net")

    implementation project(":catos.biz.common")    // Biz projects, one per domain
    implementation project(":catos.biz.billing")
    implementation project(":catos.biz.gate")
    implementation project(":catos.biz.quay")
    implementation project(":catos.biz.rail")
    implementation project(":catos.biz.yard")
}
```

This is a real confirmation of the layering diagram in [4.1](#41-layered-architecture): `catos.ct.ip` (the Ct project) sits on top of six real `catos.biz.*` domain modules and the `catos.cm.*` Common Module, which in turn sit on the `tsb.fontos.*` framework. The domain list actually used in production (`common`, `billing`, `gate`, `quay`, `rail`, `yard`) is close to, but not identical to, the abstract `Common`/`Quay`/`Yard`/`Gate`/`Rail` list in [4.1](#41-layered-architecture) — `billing` is a real additional domain not previously listed. `catos.ct.ip`'s own Controller package also has an `inf` (interface/messaging monitoring) subpackage with no matching `catos.biz.inf` dependency, suggesting that slice of logic is either handled inside `catos.biz.common` or is genuinely Ct-local per [4.7](#47-ct-client-projects)'s "unless it's genuinely Ct-specific" carve-out — not confirmed further here.

### 19.2 Full-Stack Example: the `SingleGrid` Feature (Single-Module Pattern)

The `SingleGrid` feature (Controller + Service + Dao + Item + Param + per-dialect Mapper XML) appears essentially identically in both `tsb.fontos.webprototype` (Oracle dialect) and `most.jpb` (MSSQL dialect) — strong evidence this began life as the framework's own starter template and was carried into the real production module with no business customization. Walking through it layer by layer, using the actual `most.jpb` source (the `tsb.fontos.webprototype` version is byte-for-byte the same except for the package name):

**Item** — the data-carrying class, extending the framework's `BaseDataItem`:

```java
package most.jpb.item.singlegrid;

import lombok.Getter;
import lombok.Setter;
import tsb.fontos.core.item.BaseDataItem;

@Getter
@Setter
public class SingleGridItem extends BaseDataItem {
    private static final long serialVersionUID = 1L;
    private String unid;
    private String unno;
    private String imdgClass;
    private String packingGrp;
    private String properShipNm;
    private String subsidiaryRisk;
    private String limitedQty;
    private String marinePollut;
    private String ems;
    private String flashPoint;
    private String extendGrp;
    private String fe;
    private String staffCd;
    private Boolean dummyChk;

    private String unnoBackcolor = "12648384";
    private String unnoForecolor = "16777158";
}
```

**Param** — the search/request parameter, extending `BaseBizParm`; note the nested `TempSortedColumnItem` list for grid-side custom column sorting:

```java
package most.jpb.param.singlegrid;

import java.util.List;

import lombok.Getter;
import lombok.Setter;
import tsb.fontos.core.param.BaseBizParm;
import most.jpb.item.singlegrid.TempSortedColumnItem;

@Getter
@Setter
public class SingleGridParam extends BaseBizParm {
    private static final long serialVersionUID = 1L;
    private String imdgClass;
    private List<TempSortedColumnItem> sortColumns;
}
```

**Dao** — a MyBatis mapper interface extending `ITsbRestDao`; each `@Mapper`-annotated method name matches a `<select>`/`<insert>`/`<update>`/`<delete>` statement `id` in the paired XML, exactly as documented in [4.6](#46-biz-business-projects):

```java
package most.jpb.dao.singlegrid;

import java.util.List;

import org.apache.ibatis.annotations.Mapper;

import com.github.pagehelper.Page;

import tsb.fontos.core.annotation.TNewVersionDecision;
import tsb.fontos.core.annotation.TNewVersionDecisionType;
import tsb.fontos.core.dao.ITsbRestDao;
import most.jpb.item.singlegrid.SingleGridItem;
import most.jpb.param.singlegrid.SingleGridParam;

public interface SingleGridDao extends ITsbRestDao {
    @Mapper
    List<SingleGridItem> selectSingleGrid(SingleGridParam parm);

    @Mapper
    List<SingleGridItem> checkExist(SingleGridItem item);

    @Mapper
    Page<SingleGridItem> selectSingleGridPage(SingleGridParam parm);

    @TNewVersionDecision(decision = TNewVersionDecisionType.SUPPORT)
    @Mapper
    int insertSingleGrid(SingleGridItem item);

    @TNewVersionDecision(decision = TNewVersionDecisionType.SUPPORT)
    @Mapper
    int updateSingleGrid(SingleGridItem item);

    @Mapper
    int deleteSingleGrid(SingleGridItem item);
}
```

**Mapper XML** — this is the one file that genuinely differs per DB dialect, confirming the [16.1](#161-multi-dialect-mapper-switching) `mapper/{dbType}/**/*.xml` convention against real code. The `most.jpb` (MSSQL) version at `mapper/mssql/singlegrid/SingleGridMapper.xml`:

```xml
<mapper namespace="most.jpb.dao.singlegrid.SingleGridDao">
    <resultMap id="singleGridResult" type="most.jpb.item.singlegrid.SingleGridItem">
        <result property="unid" column="UNID" />
        <result property="unno" column="UNNO" />
        <result property="imdgClass" column="CLASS" />
        <!-- ... remaining columns omitted for brevity ... -->
    </resultMap>

    <select id="selectSingleGrid" resultMap="singleGridResult">
        SELECT DISTINCT UNID, UNNO, CLASS, PACKING_GRP, PROPER_SHIP_NM, ...
        FROM TB_IMDGDATA
        WHERE 1=1
        <if test="imdgClass != null and imdgClass != ''">
            AND (CLASS LIKE #{imdgClass} + '%')   <!-- MSSQL: '+' string concatenation -->
        </if>
    </select>

    <insert id="insertSingleGrid">
        INSERT INTO TB_IMDGDATA (UNID, UNNO, CLASS, ..., UPDATE_TIME, CLEAR_LIVING_QUARTER)
        VALUES (#{unid}, #{unno}, #{imdgClass}, ..., SYSDATETIME(), CASE WHEN #{dummyChk} = 1 THEN 'Y' ELSE 'N' END)
    </insert>
    <!-- update / delete / checkExist follow the same shape -->
</mapper>
```

The `tsb.fontos.webprototype` (Oracle) sibling at `mapper/oracle/singlegrid/SingleGridMapper.xml` is identical except for exactly the two dialect-specific tokens: string concatenation (`||` in Oracle vs. `+` in MSSQL) and the current-timestamp function (`SYSDATE` in Oracle vs. `SYSDATETIME()` in MSSQL). Everything else — column list, `WHERE` clause, `<if>` conditions, the resultMap — is character-for-character the same. This is the cleanest possible real-world confirmation of [16.1](#161-multi-dialect-mapper-switching): the dialect boundary is *exactly* the mapper XML, nothing else.

**Service** — `@Service` bean, class-level `@Transactional(readOnly = true)` with per-method overrides:

```java
package most.jpb.service.singlegrid;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import com.github.pagehelper.Page;

import lombok.RequiredArgsConstructor;
import tsb.fontos.core.exception.BizFileDonwloadException;
import tsb.fontos.core.service.TBaseRestService;
import most.jpb.dao.singlegrid.SingleGridDao;
import most.jpb.item.singlegrid.SingleGridItem;
import most.jpb.param.singlegrid.SingleGridParam;

@Service(value = "singleGridService")
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class SingleGridService extends TBaseRestService {

    @Autowired
    private final SingleGridDao singleGridDao;

    @Transactional(propagation = Propagation.NEVER)
    public List<SingleGridItem> selectSingleGrid(SingleGridParam param) {
        if ("222".equals(param.getImdgClass())) {
            throw new BizFileDonwloadException("ED0001", "ED0001 MESSAGE...", new Object[] {});
        }
        return singleGridDao.selectSingleGrid(param);
    }

    @Transactional(propagation = Propagation.NEVER)
    public Page<SingleGridItem> selectSingleGridPage(SingleGridParam param) {
        return singleGridDao.selectSingleGridPage(param);
    }

    @Transactional(propagation = Propagation.NEVER)
    public Boolean checkExist(SingleGridItem item) {
        return singleGridDao.checkExist(item).size() > 0;
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public int insertSingleGrid(SingleGridItem item) {
        return singleGridDao.insertSingleGrid(item);
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public int updateSingleGrid(SingleGridItem item) {
        return singleGridDao.updateSingleGrid(item);
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public int deleteSingleGrid(SingleGridItem item) {
        return singleGridDao.deleteSingleGrid(item);
    }
}
```

This confirms the read/`NEVER` vs. write/`REQUIRES_NEW` propagation split is a real, deliberately-chosen pattern (identical in both repositories) — not just the illustrative `NEVER`-for-reads example already shown in [4.6](#46-biz-business-projects). Writes here use `REQUIRES_NEW` (always start an independent transaction) rather than [4.6](#46-biz-business-projects)'s illustrative `REQUIRED` (join if one exists) — both are valid choices from the propagation table in [4.6](#46-biz-business-projects); `REQUIRES_NEW` means each save/insert/update/delete commits (or rolls back) on its own, independent of whatever transaction context the caller happens to be in.

**Controller** — `@RestController extends TBaseRestController`, using the paging annotation from [5.4](#54-pagination):

```java
package most.jpb.controller.singlegrid;

import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import lombok.RequiredArgsConstructor;
import tsb.fontos.core.annotation.TPagingDecision;
import tsb.fontos.core.annotation.TPagingDecisionType;
import tsb.fontos.core.http.rest.TRestResponse;
import tsb.fontos.core.item.TPageInfo;
import tsb.fontos.core.pagehelper.TPageHelper;
import tsb.fontos.core.web.TBaseRestController;
import most.jpb.item.singlegrid.SingleGridItem;
import most.jpb.param.singlegrid.SingleGridParam;
import most.jpb.service.singlegrid.SingleGridService;

@RestController
@RequestMapping("/")
@RequiredArgsConstructor
public class SingleGridController extends TBaseRestController {

    private final SingleGridService singleGridService;

    @TPagingDecision(required = true, decision = TPagingDecisionType.BY_PARAMTER)
    @RequestMapping(value = "/selectSingleGrid", method = RequestMethod.GET)
    public TRestResponse<SingleGridItem> selectSingleGrid(HttpServletRequest request, SingleGridParam param) {
        if (param.getImdgClass() == "") return null;
        List<SingleGridItem> result = param.isPaging()
                ? singleGridService.selectSingleGridPage(param)
                : singleGridService.selectSingleGrid(param);
        TPageInfo<SingleGridItem> pageInfo = TPageHelper.of(result, param.isPaging());
        return new TRestResponse<SingleGridItem>(pageInfo);
    }

    @PostMapping("/saveSingleGrid")
    public int saveSingleGrid(HttpServletRequest request, @RequestBody SingleGridItem item) {
        item.setNewVersion(new Date());
        if (singleGridService.checkExist(item)) {
            return singleGridService.updateSingleGrid(item);
        } else {
            insertSingleGrid(request, item);
            return 1;
        }
    }
    // insertSingleGrid / insertSingleGridBatch / updateSingleGrid / deleteSingleGrid follow the same shape
}
```

> ```java
> if (param.getImdgClass() == null || param.getImdgClass().isEmpty()) return null;
> ```

### 19.3 Ct-Project Controllers Confirmed in Production: `catos.ct.ip`

`catos.ct.ip` has no Service/Dao/Mapper of its own for domain business logic (per [19.1](#191-two-deployment-patterns-seen-in-production), that lives in the `catos.biz.*` projects it depends on) — its Controllers stay thin and delegate to injected `catos.biz`/`catos.cm` Services and `tsb.fontos.security` framework classes. `IpSSOController` is the real production counterpart to the illustrative `SampleSSOLoginController` flow already documented in [6.6](#66-sso--token-lifecycle-management):

```java
package catos.ct.ip.controller.common.security;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import catos.cm.security.item.UserInfoItem;
import catos.cm.security.param.AuthenticationParam;
import catos.cm.security.service.AuthenticationService;
import lombok.RequiredArgsConstructor;
import tsb.fontos.core.exception.BizException;
import tsb.fontos.security.authentication.JwtAccessToken;
import tsb.fontos.security.authentication.TJwtTokenItem;
import tsb.fontos.security.jwt.JwtSSOTokenProvider;
import tsb.fontos.security.jwt.mng.SimpleJwtAuthenticationManager;

// Need to refactoring ([developer initials redacted])
@RequiredArgsConstructor
@Controller
@RequestMapping("/init/ip-sso")
public class IpSSOController {
    private final SimpleJwtAuthenticationManager simpleJwtAuthenticationManager;
    private final JwtSSOTokenProvider jwtSSOTokenProvider;
    private final AuthenticationService authenticationService;

    @GetMapping({"/login"})
    public String loginGET(HttpServletRequest request, HttpServletResponse response, @RequestParam String token) {
        TJwtTokenItem tokenItem = this.ssoLogin(request, response, token);
        if (tokenItem != null) {
            return "redirect:/";
        } else {
            throw new BizException("MSG_CTCM_00234");
        }
    }

    @PostMapping({"/login"})
    public String loginPOST(HttpServletRequest request, HttpServletResponse response, @RequestBody AuthenticationParam param) {
        if (param == null) {
            throw new BizException("MSG_CTCM_00234");
        }

        TJwtTokenItem tokenItem = this.ssoLogin(request, response, param.getToken());
        if (tokenItem != null) {
            return "redirect:/";
        } else {
            throw new BizException("MSG_CTCM_00234");
        }
    }

    private TJwtTokenItem ssoLogin(HttpServletRequest request, HttpServletResponse response, String token) {
        try {
            JwtAccessToken ssoTokenItem = this.jwtSSOTokenProvider.getTokenInfo(token);
            if (ssoTokenItem == null) return null;

            AuthenticationParam param = new AuthenticationParam();
            param.setUserId(ssoTokenItem.getSubject());
            param.setUserName(ssoTokenItem.getSubject());
            param.setDecryPassword(true);

            UserInfoItem userItem = this.authenticationService.inquiryUserInfo(param);
            if (userItem == null) return null;

            AuthenticationParam tokenPararm = new AuthenticationParam();
            tokenPararm.setUserId(userItem.getUserId());
            tokenPararm.setUserName(userItem.getUserId());
            tokenPararm.setPassword(userItem.getPassword());

            TJwtTokenItem tokenItem2 = this.simpleJwtAuthenticationManager.loginByUserInfo(request, response, tokenPararm);
            return tokenItem2;
        } catch (Exception e) {
            return null;
        }
    }
}
```

This is a verified, byte-for-byte match (imports included; only the author-initials comment is redacted) for a direct production implementation of the [6.6](#66-sso--token-lifecycle-management) flow: an incoming SSO token is decoded via `JwtSSOTokenProvider.getTokenInfo()`, the subject is used to look up the local user via the injected `AuthenticationService` (a `catos.cm.security` Service — package confirmed by the real `import`, not implemented in this project), and a local session token is then minted via `SimpleJwtAuthenticationManager.loginByUserInfo()`. Note `catos.cm.security.*` (not `catos.biz.*`) is where the shared Common-Module security Services actually live — a real, previously-undocumented package family alongside the `catos.biz.*` domain Services from [19.1](#191-two-deployment-patterns-seen-in-production). The neighboring `IpSecurityController`/`IpTotpController` in the same package similarly confirm [6.1](#61-jwt-based-authentication--authorization)'s JWT login flow and [6.5](#65-multi-factor-authentication-totpotp)'s TOTP flow are real, deployed code paths, not just training-deck samples — both were reviewed but are not reproduced here for brevity since they follow the same shape as [6.1](#61-jwt-based-authentication--authorization)/[6.5](#65-multi-factor-authentication-totpotp). (`IpSecurityController` alone injects eight real Services/framework classes — `AuthenticationService`, `CmAccessRoleService`, `SimpleJwtAuthenticationManager`, `JwtTokenProvider`, `TMailHandler` — across nine endpoints: token issue/cache-refresh/revoke, password change/forgotten-password-by-email via `TMailHandler`, and access-role/menu/control inquiry.)

> Both `IpSecurityController` and `IpSSOController` carry the source comment `// Need to refactoring ([developer initials redacted])` — a real author-attributed technical-debt marker left in production code. The developer's initials are redacted here per this guide's standing PII policy (see [21.2](#212-source-locations)); the comment itself is worth knowing about if you touch this controller, since it signals the original author already flagged it as needing cleanup.

**Takeaway — which pattern to use for new work:** follow Pattern 1 ([19.1](#191-two-deployment-patterns-seen-in-production), single-module) for a standalone module with one client. Follow Pattern 2 (the full Dto/Biz/Ct split from [4.5](#45-dto-data-transfer-object-projects)–[4.8](#48-ex-external-interface-projects), confirmed real in [19.1](#191-two-deployment-patterns-seen-in-production)/[19.3](#193-ct-project-controllers-confirmed-in-production-catosctip)) as soon as the business logic needs to be shared by more than one Ct/client project — retrofitting from Pattern 1 to Pattern 2 later means extracting Service/Dao/Mapper out of the single module into a new `catos.biz.*` project, which is far more work than starting with the split.

---

## 20. Best Practices & Security Notes

This section consolidates warnings surfaced while assembling this guide. Treat all of them as action items for a security/architecture review, not just documentation trivia.

1. **Never hardcode credentials, tokens, or connection strings in source**, even in "sample"/"education" modules — several reference examples in `Fontos-Sample-Backend` do this (a demo JWT signing key in a SOAP sample's `JwtTokenProvider`, a TCP-client login example with an inline username/password/DB string). Treat every one of those as an anti-pattern to avoid, not a template to copy. Load secrets from environment variables, a vault, or Jasypt-encrypted properties instead.
2. **Audit for accidentally-committed real secrets before using `Fontos-Sample-Backend` as an onboarding reference.** During this review, at least one real-looking third-party API key and one real-looking customer integration credential were found committed in plaintext inside sample module `application.properties`/service files. This guide intentionally does not reproduce those values. Have the repository owner rotate/revoke them and scrub git history, then scan the rest of the sample tree for similar leftovers before pointing new hires at it.
3. **`PasswordEncoder`'s AES mode derives both key and IV from a single MD5 digest** — acceptable only for backward-compatibility with legacy encrypted data, not for new sensitive-data encryption. Prefer a vetted password hash (BCrypt/Argon2) for stored credentials and AES-GCM with properly-derived key material for reversible encryption.
4. **The SQL Injection validator's regex blocks all quote characters**, which will false-positive on legitimate free-text containing an apostrophe. Scope `@SqlInjectionTarget` to identifiers/codes/search terms, not free-text fields.
5. **`compileOnly 'io.jsonwebtoken:jjwt:0.9.1'` in `tsb.fontos.security`** means consuming applications must explicitly add `jjwt` (or an equivalent) as a real runtime dependency — it will not come along transitively.
6. **Dependency hygiene**: several modules still pin old third-party versions (e.g. Guava 12.0.1, MySQL connector 5.1.31, BouncyCastle `bcprov-jdk16:1.45`). Check these against current CVE advisories before using a module as-is in a new, security-sensitive service.
7. **GDPR-relevant integrations** (e.g. any SOAP/mTLS integration with an EU customer/partner endpoint) should have their credential handling and data flows reviewed for GDPR compliance separately from the functional integration work — this guide only documents the technical wiring, not the compliance posture.
8. As with all AI-assisted output, **review this guide and any code drawn from it internally before it is used in production or shared externally** — several source slides had screenshots with cropped/obscured code, marked inline above; verify those spots against the live repository.

---

## 21. Appendix

### 21.1 Module / Dependency Reference

| Module | Artifact | Key third-party deps |
|---|---|---|
| `tsb.fontos.core` | `tsb-fontos-core` | spring-boot-starter-web/security/aop, MyBatis 3.5.9, PageHelper 5.3.0, Guava 12.0.1, Apache POI 5.2.3, Jasypt 3.0.4, spring-boot-starter-mail, commons-net 3.6 (FTP/SFTP) |
| `tsb.fontos.security` | `tsb-fontos-security` | spring-boot-starter-security/web/aop, `io.jsonwebtoken:jjwt:0.9.1` (**compileOnly**), PageHelper, Guava, mysql-connector-java 5.1.31 (runtimeOnly) |
| `tsb.fontos.net` | `tsb-fontos-net` | spring-boot-starter-websocket, `io.netty:netty-all:4.1.97.Final`, spring-boot-starter-web/security/aop |
| `tsb.fontos.cnet` | `tsb-fontos-cnet` | netty-all, spring-boot-starter-data-jdbc/oauth2-client/oauth2-resource-server, mybatis-spring-boot-starter 2.3.1, jackson-dataformat-xml, gson 2.9.0 |
| `tsb.fontos.report.jasper` | `tsb-fontos-report-jasper` | jasperreports 5.6.1 or 6.21.0 (two build variants), itext 2.1.7, optional barcode4j/barbecue/zxing |
| `tsb.fontos.report.openhtmltopdf` | `tsb-fontos-report-openhtmltopdf` | spring-boot-starter-thymeleaf, jsoup 1.16.2, openhtmltopdf-pdfbox 1.0.10, pdfbox 2.0.24, barbecue, qrgen |
| `tsb.fontos.core.micrometer` | `tsb-fontos-core-micrometer` | spring-boot-actuator 2.6.7, micrometer-registry-prometheus |

Common baseline across all modules: **Java 8**, **Spring Boot 2.6.7**, `io.spring.dependency-management 1.0.11.RELEASE`. See [18. Migrating to JDK 17 / Spring Boot 3.x (V2)](#18-migrating-to-jdk-17--spring-boot-3x-v2) for the parallel migration-track version matrix.

### 21.2 Source Locations

- Framework: `Fontos-Framework-Backend` (this machine: `C:\Users\lee.hs\git\Fontos-Framework-Backend`)
- Reference/sample applications: `Fontos-Sample-Backend` (`https://tds.tsb.co.kr/Product-JAVA/Tsb-Fontos-Sample/_git/Fontos-Sample-Backend`)
- Frontend prototype used in several security/WebSocket examples, and (per [19. Real-World Business Component Walkthrough](#19-real-world-business-component-walkthrough-controller-service-dao-mapper)) also a full single-module FONTOS backend sample app in its own right: `Fontos-Sample-WebPrototype` (`https://tds.tsb.co.kr/Product-JAVA/Tsb-Fontos-Sample/_git/Fontos-Sample-WebPrototype`; this machine: `C:\Users\lee.hs\git\Fontos-Sample-WebPrototype\tsb.fontos.webprototype`)
- Frontend reference app used in the log-forwarding, Excel-export, and Web Serial examples: `Fontos-Sample-Frontend` (`https://tds.tsb.co.kr/Product-JAVA/Tsb-Fontos-Sample/_git/Fontos-Sample-Frontend`)
- FONTOS Framework template/common repos referenced alongside the Frontend repo: `Fontos-Framework_Template`, `Fontos-Framework-Frontend`, `Catos-Cm-Frontend` (all under `https://tds.tsb.co.kr/Product-JAVA/...`)
- A real single-module production project analyzed in [19. Real-World Business Component Walkthrough](#19-real-world-business-component-walkthrough-controller-service-dao-mapper) (this machine): `Most-Jpb` (`C:\Users\lee.hs\git\Most-Jpb\most.jpb`)
- CATOS+ product repositories referenced in [4. CATOS+ Project Architecture & Conventions](#4-catos-project-architecture--conventions) and, for `Catos-Plus-Backend-MSSQL` specifically, analyzed directly in [19. Real-World Business Component Walkthrough](#19-real-world-business-component-walkthrough-controller-service-dao-mapper) (this machine): `Catos-Cm-Backend`, `Catos-Plus-Backend-MSSQL` (`C:\Users\lee.hs\git\Catos-Plus-Backend-MSSQL\catos.ct.ip`), `Catos-Sample-Backend`, `Catos-Sample-Frontend` (under `C:\Users\lee.hs\git\`)
- V2 (JDK17/Spring Boot 3.x) migration repositories, see [18. Migrating to JDK 17 / Spring Boot 3.x (V2)](#18-migrating-to-jdk-17--spring-boot-3x-v2): `Fontos-Framework-Backend-V2`, `Fontos-Kpa-Backend-V2`, `Fontos-Sample-Backend-V2`, `Fontos-Sample-WebPrototype-V2`, `C3IT-Gradle-Migration-Backend-V2`
- Companion frontend guide: `[tsb-fontos-ui] Developer Guide` (`DEVELOPER_GUIDE.md`, attached alongside this document)

Reference decks/documents consolidated into this guide (internal, for traceability — not for redistribution outside the team):
Spring Boot JASPER REPORT 기능 사용방법 (2024.02.20); Spring Boot SoapServer 기능 사용방법 (2025.11.01); FONTOS J2EE WebSocket STOMP + Spring Security 적용방법 (V0.1, 2025.11.29); FONTOS J2EE SQL Injection Validator 적용방법 (V0.1, 2025.11.24); FONTOS J2EE 비밀번호 정책 Validator 적용방법 (V0.1, 2025.11.24); FONTOS J2EE JWT기반 API 인증및인가 적용방법 (V1.2, 2025.05.21); FONTOS J2EE BACKEND EXCEL EXPORT 적용방법 (V0.3, 2025.08.23); FontosWeb TCP Client 개발가이드 (2024.10.21); FontosWeb WebSocket 가이드 (2024.09.05); CATOS API Security for FONTOS-J2EE (v1.1, 2025.04.09); Fontos BACKEND WAR 빌드환경구성 (2024.12.16); SpringBoot FRONTEND LOG 서버전달 사용방법 (2024.11.26); CATOS+ Development Environment Install Guide for FONTOS-J2EE (v1.0, 2025.03.23); CATOS+ Development Guide (Backend) for FONTOS-J2EE (v1.0, 2025.03.23); MFA OTP 적용 방안 (2026.01.20); FONTOS J2EE JWT기반 API SSO 및 토큰관리 적용방법 (V0.2, 2026.06.30); FONTOS J2EE 데이터 페이징 적용방법 (V0.1, 2025.05.26); Fontos J2EE for WildFly 배포 가이드 (undated); DataSync 사용방법 (2024.10.05); Mysql JDBC Driver (2024.09.26 — actually documents `app.dbType` mapper-folder switching, not JDBC driver versions); Web ComPort Serial 통신테스트 (Ver.01, 2024.08.20); WebApp 외부설정파일 Application 저장방법 (2024.08.03); Spring Boot SoapClient 기능 사용방법 (2024.05.11); WebApp Profile build (2023.02.21); WebApp 멀티데이터소스 설정방법 (2023.02.21); Web Application Java SpringDoc/Swagger 적용 가이드 (V0.2, 2023.02.17); WebApp Task Scheduler 추가방법 (2023.02.21); Web Application SSL 적용 방법 (V0.1, 2022.11.09); Java Coding Style Guide (v1.0, 2022.06.07); FONTOS V2 JDK17/Spring Boot 3.x 마이그레이션 절차서 (V0.1, 2025.06.27); Fontos FRONTEND WEB 배포환경 설정 방법 (2024.10.22, rev. 2026.03.30).

> **Note on this round's inputs**: one file referenced in an earlier request — *FONTOS2.0INST Development Environment Install Guide for FONTOS for Java_v1.0_Kor_20220504.docx* (intended for a gap-analysis against [2. Development Environment Setup](#2-development-environment-setup)) — was never found among the attached files and still hasn't been processed. Please re-attach it if you'd like that comparison done. Separately, an Excel Export deck attached in an earlier round (`FONTOS_J2EE_BACKEND_EXCEL_EXPORT_V0.3_20250823.pptx`) was a byte-identical duplicate (matching MD5) of a file already incorporated into [11. Excel Export](#11-excel-export), so it was skipped rather than reprocessed.
>
> **This round**: the JDK17/Spring Boot 3.x migration deck's embedded Git-repo-tracking spreadsheet and the frontend deployment deck's footer both contained internal staff names/contact emails — these were not reproduced anywhere in this guide, per the standing data-handling policy.
>
> **Latest round**: [19. Real-World Business Component Walkthrough](#19-real-world-business-component-walkthrough-controller-service-dao-mapper) was written directly from real source code in three device-connected repositories (`Fontos-Sample-WebPrototype`/`tsb.fontos.webprototype`, `Most-Jpb`/`most.jpb`, `Catos-Plus-Backend-MSSQL`/`catos.ct.ip`) rather than from a training deck. No hardcoded secrets, internal hostnames, or customer data were found in the files reviewed for this section. One developer's initials, left in an inline `// Need to refactoring (...)` comment in `catos.ct.ip`'s `IpSecurityController`/`IpSSOController`, were redacted per the standing PII policy. §19 does not cover every file in these three repositories — only the `SingleGrid` feature (Pattern 1) and the security/SSO controllers (Pattern 2) were reviewed in depth; treat the two deployment patterns identified as representative, not as an exhaustive audit of either codebase.
