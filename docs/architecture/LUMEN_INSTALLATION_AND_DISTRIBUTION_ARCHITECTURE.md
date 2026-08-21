# Lumen Installation and Distribution Architecture

**Status:** Proposed\
**Date:** 2026-08-10\
**Scope:** Lumen product installation, configuration, distribution,
upgrade, and runtime hand-off

## 1. Purpose

This document defines the proposed installation and distribution
architecture for **Lumen**.

As the platform evolves, **Lumen** becomes the name of the complete
installed product rather than the name of an individual orchestration
component. The current orchestrator becomes **Lumen Moderari**, while
the other architectural components are presented as **Lumen Services**.

The installation experience should therefore resemble a conventional
Windows application installer rather than requiring users to clone
repositories, create Python environments, manually load containers, edit
configuration files, or understand the internal service topology.

The intended user experience is:

> **Install Lumen. Configure the required AI and tool providers. Start
> Lumen.**

The installer is responsible for bringing the complete Lumen deployment
into a known-good initial state. Once installation is complete, **Lumen
Servire** assumes responsibility for ongoing operation and
configuration.

------------------------------------------------------------------------

## 2. Product Model

The externally visible product is:

**Lumen**

The internal architecture consists of independently responsible Lumen
Services, including:

  -----------------------------------------------------------------------
  Service                             Responsibility
  ----------------------------------- -----------------------------------
  **Moderari**                        Lumen orchestration

  **Pontis**                          Client/session and protocol bridge

  **Rogare**                          Lumen user console/client

  **Vestigare (Trace)**               Behavioural recording

  **Repetere (Replay)**               Deterministic replay and divergence
                                      detection

  **Fiducia**                         Replay scheduling and
                                      evidence-driven trust automation

  **Aestimare (Assess)**              Behavioural assessment

  **Servire**                         Operational control plane
  -----------------------------------------------------------------------

The term **Extensions** should be retired as the product architecture is
standardised.

These components are not separate products from the user's perspective.
They are services that collectively implement Lumen.

### Architectural Principle

> **Lumen is the product; Moderari, Pontis, Rogare, Vestigare, Repetere,
> Fiducia, Aestimare and Servire are Lumen Services.**

The separation remains important architecturally because each service
retains a single responsibility, but normal users should not have to
assemble those services themselves.

------------------------------------------------------------------------

## 3. Installation Experience

The primary Windows distribution should provide a conventional
installer:

`Lumen-Setup.exe`

A typical installation flow is:

1.  Welcome
2.  System and prerequisite validation
3.  Runtime configuration
4.  Lumen Services
5.  AI/model provider configuration
6.  Tool provider configuration
7.  Storage and networking
8.  Configuration review
9.  Installation
10. Health verification
11. Launch Lumen Servire

Most users should be able to accept sensible defaults.

An **Advanced Installation** path may expose additional service,
networking, storage, and runtime controls.

------------------------------------------------------------------------

## 4. Installer Responsibilities

The Lumen installer has one primary responsibility:

> **Bring Lumen into a known-good initial operating state.**

The installer should:

-   detect the host operating environment;
-   validate system prerequisites;
-   detect or install an appropriate container runtime;
-   validate available disk and memory;
-   validate required network ports;
-   load or retrieve the required Lumen container images;
-   create the Lumen installation directories;
-   collect initial provider configuration;
-   generate installation configuration;
-   generate deployment configuration;
-   create secrets where required;
-   start the Lumen Services;
-   perform service health checks;
-   validate configured providers;
-   create Windows application/start-menu integration;
-   launch Servire when installation completes.

The installer should **not** become the long-term Lumen management
interface.

------------------------------------------------------------------------

## 5. Servire Responsibility Boundary

After successful installation, **Lumen Servire** becomes the operational
owner of the installed Lumen environment.

Servire should provide facilities for:

-   service status;
-   service lifecycle management;
-   operational logs;
-   configuration management;
-   AI/model provider management;
-   tool provider management;
-   health monitoring;
-   runtime diagnostics;
-   updates;
-   future multi-stack management.

This creates a clean lifecycle boundary:

``` text
Installation / Upgrade
        │
        ▼
 Lumen Installer
        │
        ▼
 Known-good Lumen
        │
        ▼
     Servire
        │
        ▼
Ongoing Operations
```

Users should not normally rerun the installer merely to change a
provider or operational setting.

------------------------------------------------------------------------

## 6. Container-Based Distribution

Each Lumen Service should be distributed as a versioned container image.

For an offline or self-contained installer, images can be supplied as
compressed Docker image archives.

Example distribution:

``` text
Lumen-1.0.0/
│
├── Lumen-Setup.exe
│
├── images/
│   ├── lumen-moderari-1.0.0.tar.zst
│   ├── lumen-pontis-1.0.0.tar.zst
│   ├── lumen-rogare-1.0.0.tar.zst
│   ├── lumen-servire-1.0.0.tar.zst
│   ├── lumen-trace-1.0.0.tar.zst
│   ├── lumen-replay-1.0.0.tar.zst
│   ├── lumen-fiducia-1.0.0.tar.zst
│   └── lumen-assess-1.0.0.tar.zst
│
├── manifests/
│   └── lumen-stack.json
│
└── documentation/
```

Conceptually, installation performs:

``` text
Validate host
     │
     ▼
Install / validate container runtime
     │
     ▼
Load Lumen images
     │
     ▼
Generate configuration
     │
     ▼
Generate deployment
     │
     ▼
Start services
     │
     ▼
Health verification
     │
     ▼
Launch Servire
```

The user should not need to manually execute Docker commands.

------------------------------------------------------------------------

## 7. Distribution Modes

Two primary distributions should eventually be supported.

### 7.1 Lumen Web Installer

A relatively small `Lumen-Setup.exe` downloads the appropriate
signed/versioned Lumen service images during installation.

Advantages include:

-   smaller initial download;
-   latest compatible service payload;
-   easier patch distribution;
-   simpler release packaging.

### 7.2 Lumen Offline Installer

A self-contained installation package includes all required Lumen
service images.

This supports:

-   isolated networks;
-   air-gapped environments;
-   private infrastructure;
-   sovereign AI deployments;
-   controlled enterprise environments.

Offline installation is particularly aligned with Lumen's model-agnostic
and infrastructure-independent architecture.

------------------------------------------------------------------------

## 8. Container Runtime

The initial Windows installer should support Docker as the primary
container runtime.

Installation choices may include:

``` text
Container Runtime

● Install Docker Desktop
○ Use existing Docker installation
○ Advanced — specify Docker host
```

The installer should validate that the selected runtime is actually
operational before proceeding with Lumen deployment.

The architecture should not permanently couple Lumen to Docker Desktop.

The Lumen deployment abstraction should allow future support for:

-   Docker Desktop;
-   Docker Engine;
-   remote Docker hosts;
-   Podman;
-   Kubernetes;
-   other OCI-compatible environments.

This is one reason deployment configuration should be generated from a
Lumen-specific installation manifest rather than treating Docker Compose
as the canonical Lumen configuration.

------------------------------------------------------------------------

## 9. Service Installation Policy

For a normal installation, the recommended model is:

> **Install Lumen, including the complete supported Lumen Service set.**

Users should not ordinarily be asked to understand the dependency graph
and decide which architectural services they require.

The services are relatively lightweight deployment units whose
separation exists primarily to preserve architectural responsibilities.

Servire can expose whether individual capabilities are enabled or
disabled.

Example:

``` text
Lumen

Core
  Moderari       Running
  Pontis         Running
  Rogare         Running
  Servire        Running

Capabilities
  Trace          Enabled
  Replay         Enabled
  Fiducia        Disabled
  Assess         Disabled
```

An advanced installation mode may allow particular service containers to
be omitted where deployment constraints require it.

------------------------------------------------------------------------

## 10. AI / Model Provider Configuration

The installer should explicitly distinguish **model providers** from
**tool providers**.

A model provider supplies inference capability to Lumen.

Initial configuration could support:

``` text
AI Provider

○ Ollama
○ OpenAI-compatible provider
○ OpenAI
○ Anthropic
○ Other supported provider
```

Configuration may include:

``` text
Provider name:
Provider type:
Endpoint:
Model:
Authentication:
```

The installer should provide a **Test Connection** operation before
completing configuration.

### Multiple Providers

The configuration model should support multiple providers from the
beginning, even if the initial installer UI encourages configuration of
one default provider.

For example:

``` text
Configured Model Providers

Local Qwen       Ollama             Connected
Claude           Anthropic          Connected
Office LLM       OpenAI-compatible  Connected
```

There should not be an architectural assumption that one installation
has exactly one "Lumen model".

------------------------------------------------------------------------

## 11. Tool Provider Configuration

Tool providers are a separate concern from model providers.

The initial installer should present something similar to:

``` text
Tool Provider

● Install and configure Pi
○ Use an existing Pi installation
○ Configure another ACP tool provider
○ No tool provider
```

### 11.1 Install Pi

If the user chooses Pi, the installer should:

1.  detect whether Pi is already available;
2.  retrieve/install an approved compatible Pi version if required;
3.  determine/configure the executable location;
4.  configure Pontis for the ACP provider;
5.  validate that an ACP session can be established;
6.  enumerate available tools;
7.  report successful provider validation.

### 11.2 Existing Pi

Where Pi is already installed, configuration may request:

``` text
Pi executable:
Default working directory:

[Test Pi]
```

### 11.3 Alternative ACP Provider

Lumen should not architecturally treat Pi as a special case.

Pi is the first supported tool provider, not the definition of a tool
provider.

An alternative provider configuration may include:

``` text
Provider name:
Protocol: ACP
Executable:
Arguments:
Working directory:

[Test provider]
```

This preserves the intended Pontis abstraction and enables future
tool-provider substitution.

------------------------------------------------------------------------

## 12. Lumen Installation Manifest

The installer should create a canonical **Lumen installation manifest**.

For example:

``` yaml
lumen:
  installation_id: "<generated-id>"
  version: "1.0.0"

runtime:
  type: docker
  host: local

services:
  moderari:
    enabled: true
  pontis:
    enabled: true
  rogare:
    enabled: true
  trace:
    enabled: true
  replay:
    enabled: true
  fiducia:
    enabled: true
  assess:
    enabled: true
  servire:
    enabled: true

providers:
  models:
    - id: local-qwen
      type: ollama
      endpoint: http://host.docker.internal:11434
      model: qwen2.5-coder:14b

  tools:
    - id: pi
      type: acp
      provider: pi

storage:
  root: C:\ProgramData\Lumen

network:
  pontis: 11435
  moderari: 11436
  replay: 11437
  trace: 11438
```

The exact schema will evolve, but the architectural distinction is
important:

> **The Lumen manifest describes the desired Lumen installation.**

> **Docker Compose describes one mechanism used to deploy that
> installation.**

Docker Compose should therefore be generated deployment state rather
than the canonical product configuration.

This enables future generation of alternative deployment targets from
the same Lumen manifest.

------------------------------------------------------------------------

## 13. Installation Directory Structure

A Windows installation could use a structure such as:

``` text
C:\Program Files\Lumen\
    installer/
    tools/
    metadata/

C:\ProgramData\Lumen\
    config/
    data/
    logs/
    secrets/
    runtime/
```

The precise layout should follow Windows conventions and security
requirements.

The important principle is separation between:

-   installed executable/tooling assets;
-   persistent configuration;
-   persistent application data;
-   logs;
-   secrets;
-   disposable container/runtime state.

Container replacement or upgrade must not destroy user configuration or
Lumen data.

------------------------------------------------------------------------

## 14. Networking and Ports

The installer should validate Lumen's configured service ports before
deployment.

The current planned service allocation includes:

  Service       Port
  ---------- -------
  Pontis       11435
  Moderari     11436
  Replay       11437
  Trace        11438

The installer should:

-   detect conflicts;
-   propose alternatives where appropriate;
-   record the resulting assignments in the Lumen manifest;
-   generate the corresponding service configuration automatically.

Users should not need to edit multiple service configuration files when
a port changes.

------------------------------------------------------------------------

## 15. Lumen Installer Engine

A small **Lumen Installer Engine** should perform the actual
installation orchestration.

This is installation tooling, not another runtime Lumen Service.

Conceptually:

``` text
             Lumen-Setup.exe
                    │
                    ▼
          Lumen Installer Engine
                    │
       ┌────────────┼──────────────┐
       │            │              │
       ▼            ▼              ▼
 Runtime        Configuration     Payload
 Detection       Generation       Manager
       │            │              │
       ▼            ▼              ▼
 Container       lumen.yml      Lumen Images
 Runtime            │
                    ▼
             Deployment Generator
                    │
                    ▼
              Lumen Services
                    │
                    ▼
                 Servire
```

The installer engine could initially be implemented in Python and
packaged into a standalone executable.

The Windows setup/bootstrap layer can then concentrate on:

-   prerequisite handling;
-   Windows installation lifecycle;
-   elevation;
-   UI;
-   shortcuts;
-   uninstall registration;
-   upgrade integration.

This also keeps the core installation logic testable independently of
the Windows installer UI.

------------------------------------------------------------------------

## 16. Health Verification

Installation is not complete merely because containers started.

The installer should establish a **known-good installation** by
performing structured validation.

Validation should include, where applicable:

-   container/runtime availability;
-   all required Lumen Services started;
-   service health endpoints responding;
-   expected inter-service dependencies available;
-   persistent storage writable;
-   configured model provider reachable;
-   configured tool provider reachable;
-   ACP session establishment successful where configured;
-   expected tool discovery successful;
-   Servire reachable.

The final installation page should report meaningful results rather than
simply "Installation Complete".

Example:

``` text
Lumen Installation Verification

Moderari          Healthy
Pontis            Healthy
Rogare            Healthy
Trace             Healthy
Replay            Healthy
Servire           Healthy

Model Provider
Local Qwen        Connected

Tool Provider
Pi                Connected
12 tools detected

Lumen is ready.
```

------------------------------------------------------------------------

## 17. Upgrade Architecture

Lumen upgrades should preserve persistent configuration and data while
replacing disposable service containers.

Conceptually:

``` text
Discover compatible update
        │
        ▼
Download / load images
        │
        ▼
Validate compatibility
        │
        ▼
Create recovery point
        │
        ▼
Stop affected services
        │
        ▼
Perform required migrations
        │
        ▼
Start updated services
        │
        ▼
Health verification
        │
   ┌────┴────┐
   │         │
Success    Failure
   │         │
   ▼         ▼
Complete   Rollback
```

Updates should be version-aware across the complete Lumen service set.

A Lumen release manifest should define compatible versions of all
included services rather than independently installing arbitrary
"latest" service versions.

For example:

``` yaml
release: 1.3.0

services:
  moderari: 1.3.0
  pontis: 1.2.4
  rogare: 1.1.2
  trace: 1.4.1
  replay: 1.3.2
  fiducia: 1.0.0
  assess: 1.0.1
  servire: 1.5.0
```

This allows Lumen to be tested, released, installed, and rolled back as
a coherent product.

------------------------------------------------------------------------

## 18. Rollback

Upgrade safety should be designed from the beginning rather than added
later.

Before an upgrade, Lumen should retain sufficient state to restore:

-   the previous release manifest;
-   previous container image versions;
-   previous generated deployment configuration;
-   previous compatible configuration;
-   database/storage state where migrations require backup.

A failed post-upgrade health check should permit automatic or
user-approved rollback to the previous known-good Lumen release.

------------------------------------------------------------------------

## 19. Secrets

Authentication credentials and provider API keys must not be embedded
directly into container images or ordinary deployment manifests.

The installer should provide a secure secrets mechanism appropriate to
the deployment platform.

The Lumen manifest should refer to secrets symbolically rather than
containing exposed credential values wherever practical.

Servire should subsequently provide secure provider credential
management.

------------------------------------------------------------------------

## 20. Uninstallation

The uninstaller should distinguish between removing application/runtime
assets and destroying user data.

A normal uninstall should offer choices such as:

``` text
Remove Lumen

☑ Remove Lumen Services and application files
☑ Remove downloaded container images
☐ Remove Lumen configuration
☐ Remove Lumen data and recordings
```

Destructive removal of persistent Lumen data should require an explicit
user choice.

The container runtime itself should not automatically be removed merely
because Lumen is uninstalled, particularly where it may be shared by
other applications.

------------------------------------------------------------------------

## 21. Windows Installer Technology

A Windows bootstrapper technology such as **WiX Toolset / Burn** is a
strong candidate for the outer installer.

Its role would include:

-   creating `Lumen-Setup.exe`;
-   prerequisite detection;
-   prerequisite installation;
-   elevation;
-   installation registration;
-   upgrade handling;
-   uninstall handling;
-   invocation of the Lumen Installer Engine.

The final implementation technology should be selected during installer
development, but the architecture should keep Windows bootstrap concerns
separate from Lumen deployment logic.

------------------------------------------------------------------------

## 22. Future Cross-Platform Installation

Although the first polished installer is expected to target Windows, the
Lumen Installer Engine should avoid unnecessary Windows-specific
assumptions.

A future Linux installation could provide:

``` text
curl ... | lumen-install
```

or a native package/bootstrap mechanism while invoking the same
conceptual installer engine and Lumen manifest.

The same model could eventually support:

-   Windows workstation installations;
-   Linux workstations;
-   Linux servers;
-   remote Docker environments;
-   private clusters;
-   Kubernetes deployments;
-   air-gapped enterprise deployments.

The product remains **Lumen** regardless of the deployment mechanism.

------------------------------------------------------------------------

## 23. Architectural Invariants

The following principles should be treated as installation and
distribution invariants.

### 23.1 Lumen is the installed product

Individual architectural components are Lumen Services, not
independently presented end-user products.

### 23.2 Service boundaries remain intact

Simplifying installation must not collapse the single-responsibility
service architecture.

### 23.3 Installation and operation are separate responsibilities

The installer establishes a known-good deployment.

Servire operates it.

### 23.4 Model providers and tool providers are different concepts

Inference capability and tool capability must be independently
configurable.

### 23.5 Pi is a provider, not an architectural dependency

Pi may be the default/initial supported tool provider, but Pontis and
Lumen must permit alternative providers.

### 23.6 Lumen configuration is deployment-independent

The Lumen installation manifest is canonical configuration.

Docker Compose is generated deployment implementation.

### 23.7 Containers are disposable; Lumen state is persistent

Container replacement must not imply loss of configuration, evidence,
recordings, assessment data, or other persistent Lumen state.

### 23.8 Releases are coherent

A Lumen version represents a tested collection of compatible Lumen
Service versions.

### 23.9 Installation must be verifiable

"Containers started" is not equivalent to "Lumen installed
successfully."

A successful installation requires health and dependency verification.

### 23.10 Offline deployment is a first-class capability

The architecture should permit fully self-contained installation without
dependence on public cloud infrastructure.

------------------------------------------------------------------------

## 24. Proposed Initial Development Sequence

Installer work should be approached incrementally.

### Phase 1 --- Installation Contract

Define:

-   canonical service names;
-   Lumen release manifest;
-   Lumen installation manifest;
-   persistent directory structure;
-   service ports;
-   health-check contract;
-   provider configuration schema;
-   Docker Compose generation rules.

### Phase 2 --- Installer Engine

Build a command-line installer engine capable of:

-   validating Docker;
-   loading local Lumen image TARs;
-   creating configuration;
-   generating Compose;
-   starting Lumen;
-   performing health checks;
-   reporting installation state.

This can be developed and tested before building a graphical installer.

### Phase 3 --- Provider Setup

Add:

-   Ollama/OpenAI-compatible provider configuration;
-   provider connectivity testing;
-   Pi detection/installation;
-   ACP validation;
-   alternative tool-provider configuration.

### Phase 4 --- Windows Installer

Wrap the proven installer engine with a conventional Windows
installation experience.

### Phase 5 --- Offline Distribution

Produce a fully self-contained Lumen distribution containing the
complete tested service image set.

### Phase 6 --- Upgrade and Rollback

Introduce:

-   signed release manifests;
-   version discovery;
-   image replacement;
-   migration orchestration;
-   health-gated upgrade completion;
-   rollback.

### Phase 7 --- Servire Integration

Move all post-install configuration and operational control into
Servire, including provider changes and eventually Lumen update
management.

------------------------------------------------------------------------

## 25. Product Outcome

The desired outcome is that installation complexity does not expose
Lumen's internal implementation complexity.

A user should experience:

``` text
Install Lumen
     │
     ▼
Choose AI Provider
     │
     ▼
Choose Tool Provider
     │
     ▼
Install
     │
     ▼
Lumen verifies itself
     │
     ▼
Servire opens
```

Behind that simple workflow, Lumen can remain a distributed set of
independently responsible services.

This gives Lumen both properties simultaneously:

> **Architecturally modular. Operationally one product.**

That is the central objective of the Lumen installation and distribution
architecture.
