# Lumen M0.1 --- Servire Runtime Configuration and Bootstrap Implementation Plan

**Date:** 2026-09-16\
**Status:** **IMPLEMENTED AND TESTED — 2026-09-17**
**Scope:** Servire package only

## Terminology

For this implementation:

-   **Lumen** means the complete system, including Servire and every
    other Lumen service.
-   **Servire** means the Servire service specifically.
-   **the Stack** means all managed Lumen services other than Servire,
    considered collectively.
-   Individual services are referred to by their service names, for
    example Praebere, Moderari, Pontis or Nuntius.

## Objective

Complete the Servire-owned bootstrap and external-runtime-configuration
work that can be implemented within the Servire package before changing
Praebere and Moderari.

The implementation establishes MongoDB as Servire's bootstrap
dependency, makes Servire authoritative for the Ollama endpoint, updates
the External Dependencies UI, integrates Ollama into the existing Verify
flow, makes Servire a Nuntius participant, and prepares per-service
runtime configuration distribution during Stack startup.

Illuminates.One authorization, installation identity, Dockerisation and
HAProxy are outside this implementation pass.

## Implementation Order

### 1. MongoDB Bootstrap

Servire starts independently of the Stack.

On Servire startup:

1.  Load MongoDB connection details from Servire `config.yaml`.
2.  Attempt the MongoDB connection.
3.  If MongoDB is available, continue Servire bootstrap.
4.  If MongoDB is unavailable, Servire remains running and its UI
    remains accessible.

When MongoDB is unavailable:

-   Servire enters a restricted bootstrap state.
-   Verify is unavailable.
-   Stack Start is unavailable.
-   No Stack services are started.
-   The Servire tab displays a clear, human-readable MongoDB unavailable
    message.
-   Other component tabs remain visible and retain their existing
    unavailable presentation.
-   A **Retry** action is provided.

Retry:

-   retries only the MongoDB bootstrap connection;
-   does not restart Servire;
-   does not start the Stack;
-   leaves the failure state visible if MongoDB remains unavailable;
-   continues normal Servire bootstrap if MongoDB becomes available.

Credentials must not be exposed in the UI or error messages.

### 2. Load Authoritative Ollama Configuration

After MongoDB bootstrap succeeds, Servire loads the persisted Ollama
Host/Port from MongoDB.

If no persisted Ollama configuration exists, Servire uses the default
endpoint from Servire `config.yaml`.

The initial M0.1 default is:

``` text
localhost:11434
```

The loaded/default value becomes Servire's current Ollama configuration
and is displayed in the Servire UI.

Servire owns the authoritative external Ollama endpoint. Praebere
continues to own provider/model discovery and lifecycle, while Moderari
continues to own inference.

### 3. External Dependencies UI

Update Servire's **External Dependencies** UI.

Remove **Pi** from External Dependencies.

Add configurable Ollama Host/Port fields and retain MongoDB
availability:

``` text
External Dependencies

Ollama
  Host:  [ localhost ]
  Port:  [ 11434     ]    [ Apply ]

  State: Not verified

MongoDB
  State: Available
```

Pi remains part of Lumen and remains the Pontis-spawned ACP/tool
provider. It is simply not an external dependency configured by the
researcher through Servire.

### 4. Ollama Apply and Staging

While Servire is running and the Stack is stopped, the Ollama Host and
Port fields are editable.

**Apply**:

1.  validates the Host/Port values syntactically;
2.  stages the proposed Ollama endpoint in Servire;
3.  resets/invalidates the existing Verify state.

Apply does **not**:

-   contact the Ollama endpoint;
-   persist the proposed endpoint to MongoDB;
-   send the endpoint to Praebere or Moderari.

After Apply, the operator must run the existing Verify process before
the Stack can be started.

### 5. Ollama Verification

Extend Servire's existing Verify flow to perform a lightweight
reachability check against the staged/current Ollama endpoint.

This check establishes only that something is reachable at the
configured endpoint.

It does **not** establish that:

-   the endpoint is a valid Ollama model provider;
-   required models exist;
-   model discovery succeeds;
-   provider reconciliation succeeds.

Those remain Praebere responsibilities during Stack startup.

### 6. Persist Ollama After Successful Ollama Reachability

If the Ollama portion of Verify succeeds:

1.  persist the staged Host/Port to MongoDB;
2.  make that endpoint Servire's authoritative Ollama configuration;
3.  mark the Ollama dependency appropriately in the Servire UI;
4.  continue the remaining existing Verify checks.

Persistence is tied to successful Ollama reachability, **not** to
successful completion of the entire Verify process.

If an unrelated Verify check subsequently fails, the successfully
validated Ollama configuration remains persisted.

### 7. Failed Ollama Verification

If the staged/current Ollama endpoint fails the lightweight reachability
check:

-   do not persist the staged endpoint;
-   retain the previous persisted Ollama configuration in MongoDB;
-   show Ollama as unavailable;
-   fail the overall Verify operation;
-   leave Stack Start disabled.

The operator can correct the Host/Port, Apply again, and run Verify
again.

### 8. Preserve Existing Verify and Start Semantics

The existing Servire Verify process remains the gate for Stack Start.

Successful overall Verify requires:

-   MongoDB to remain available;
-   the configured Ollama endpoint to pass Servire's lightweight
    reachability check;
-   all other existing required Servire validation checks to pass.

Only successful overall Verify enables Stack Start.

Stopping the Stack retains the existing behaviour whereby Verify must be
run again before another Stack Start.

### 9. Lock Ollama Configuration While the Stack Is Running

Once Stack startup begins and while the Stack is running:

-   Ollama Host/Port are displayed but not editable;
-   Apply is disabled.

M0.1 does not support hot Ollama endpoint reconfiguration.

Changing Ollama requires:

``` text
Stop Stack
    |
    v
Edit Ollama Host / Port
    |
    v
Apply
    |
    v
Verify
    |
    v
Start Stack
```

### 10. Make Servire a Nuntius Participant

During Stack startup, Nuntius starts according to the existing Servire
dependency/startup ordering.

Once Nuntius is successfully started and healthy, Servire
declares/registers itself with Nuntius using the existing Nuntius
service declaration, registration and routing mechanism.

No Servire-specific side channel is introduced.

The implementation must allow Servire to:

-   originate `\obt` runtime-configuration messages;
-   receive responses/acknowledgements;
-   participate without affecting existing service-to-service `\obt`
    behaviour.

### 11. Service-Requested Runtime Configuration During Stack Startup

Servire does **not** track which services require Ollama and does not
proactively distribute the Ollama endpoint to particular consumers.

Servire is the authoritative holder of external runtime configuration.
Each service owns knowledge of the infrastructure configuration it
requires and requests that configuration from Servire during its own
startup, once it can communicate through Nuntius.

Conceptually:

``` text
\obt servire ollama
```

The exact `\obt` request and response schema must reuse the existing
Nuntius declaration, routing and acknowledgement conventions found in
the current code.

For Praebere:

``` text
Servire starts Praebere
        |
        v
Praebere starts sufficiently
to communicate through Nuntius
        |
        v
\obt servire ollama
        |
        v
Servire returns authoritative
persisted/verified Ollama endpoint
        |
        v
Praebere connects / reconciles /
discovers models
        |
        +-- SUCCESS --> Praebere startup complete
        |                 |
        |                 v
        |            continue Stack startup
        |
        +-- FAILURE --> startup failure
                          |
                          v
                   Servire performs normal
                   Stack startup rollback
```

Moderari follows the same request pattern: once it can communicate
through Nuntius during startup, it requests `\obt servire ollama`,
receives the authoritative endpoint, adopts it for inference, and only
then completes its startup.

A service is not considered to have completed startup merely because its
process has launched. Where a service requires Servire-owned runtime
configuration, obtaining and successfully applying that configuration is
part of the service's startup.

Servire's lightweight Verify does not replace Praebere's authoritative
provider validation. If Praebere cannot successfully initialise against
the Servire-provided persisted Ollama connection, Stack startup has
failed and Servire invokes its existing startup rollback.

This request model is deliberately generic. A future service requiring
Servire-owned infrastructure information can request the configuration
it needs without Servire gaining service-specific knowledge of its
consumers.

The responsibility boundary is:

-   **Servire** owns, persists and serves authoritative external runtime
    configuration.
-   **Individual services** know which external runtime configuration
    they require and request it from Servire during startup.
-   **Nuntius** transports the `\obt` request and response.
-   **Servire's existing startup management** determines whether each
    managed service ultimately becomes operational and whether Stack
    startup continues or rolls back.

### 12. Temporary Servire-Only Provider Boundary

During this initial Servire-only implementation, Praebere and Moderari
do not yet request the new Servire-owned Ollama runtime configuration.

Servire should implement and test its side of the generic
configuration-provider contract:

-   Servire registration/declaration with Nuntius;
-   receipt and routing of a request addressed to Servire;
-   recognition of an Ollama configuration request;
-   construction of the authoritative Ollama configuration response from
    the persisted/verified Servire state;
-   response routing back through Nuntius;
-   appropriate handling of invalid or unavailable configuration
    requests.

Servire must **not** contain temporary knowledge that Praebere and
Moderari are the Ollama consumers and must not proactively send Ollama
configuration to either service.

Until Praebere and Moderari are changed, the absence of requests from
those services is expected and must not itself become a fatal Stack
startup condition.

When Praebere is implemented next, it will request its required Ollama
configuration from Servire during startup. Successful Ollama
validation/reconciliation then becomes part of Praebere startup and a
real Servire startup gate. Failure must exercise Servire's existing
Stack rollback.

When Moderari is subsequently implemented, it will independently request
the Ollama configuration it requires and successful adoption becomes
part of Moderari startup.

No temporary Servire-side consumer list or later conversion from push to
pull should be required.

### 13. Lifecycle and State Handling

Servire must maintain clear state for:

-   MongoDB bootstrap;
-   persisted Ollama endpoint;
-   staged Ollama endpoint;
-   Ollama verification;
-   Nuntius registration;
-   availability of Servire-owned runtime configuration for `\obt`
    requests.

Servire does **not** maintain state describing which services consume
Ollama or which services should receive particular infrastructure
configuration. Consumer requirements belong to the consuming services.

Required lifecycle behaviour includes:

-   restarting Servire performs MongoDB bootstrap again;
-   restarting Servire reloads the persisted Ollama endpoint;
-   after successful Verify, the persisted/verified Ollama endpoint is
    available to Lumen services requesting it through Nuntius;
-   stopping the Stack unlocks Ollama editing;
-   stopping the Stack invalidates Verify according to existing Servire
    behaviour;
-   changing and applying an Ollama endpoint stages the new value and
    invalidates Verify;
-   failed Ollama verification does not replace the previously persisted
    endpoint;
-   successful Ollama reachability persists the new endpoint and makes
    it the authoritative value returned by subsequent
    `\obt servire ollama` requests;
-   Stack startup does not require Servire to enumerate or proactively
    configure Ollama consumers;
-   a managed service requiring Servire-owned runtime configuration
    requests it as part of its own startup;
-   failure of a service to complete its required
    configuration/initialisation is handled through Servire's existing
    startup failure and rollback behaviour;
-   existing Servire Start/Stop/Restart and dependency management remain
    intact.

### 14. Tests

The Servire implementation should cover at least:

-   MongoDB available during bootstrap;
-   MongoDB unavailable during bootstrap;
-   MongoDB Retry while still unavailable;
-   MongoDB restored followed by successful Retry;
-   default Ollama configuration when MongoDB contains no persisted
    value;
-   persisted Ollama configuration reload;
-   removal of Pi from External Dependencies;
-   Ollama Host/Port UI and dependency state;
-   Apply stages without contacting Ollama;
-   Apply stages without persisting to MongoDB;
-   Apply invalidates Verify;
-   failed Ollama Verify leaves the previous persisted value unchanged;
-   successful Ollama reachability persists the staged endpoint;
-   later unrelated Verify failure does not undo the persisted Ollama
    endpoint;
-   successful overall Verify enables Stack Start;
-   Ollama configuration is locked during Stack startup/running;
-   stopping the Stack restores editability and requires Verify before
    restart;
-   Servire registration with Nuntius;
-   per-service `\obt` construction/routing for Praebere;
-   per-service `\obt` construction/routing for Moderari;
-   existing Stack dependency ordering;
-   existing startup rollback;
-   no regression of existing Servire Start/Stop/Restart behaviour.

## Resulting Servire Flow

``` text
Servire starts
     |
     v
MongoDB bootstrap
     |
     +-- FAIL --> Restricted Servire UI --> Retry
     |
     v
Load persisted Ollama endpoint
     |
     +-- none --> config.yaml default
     |
     v
Normal Servire UI
     |
Edit -> Apply
       [stage only; invalidate Verify]
     |
     v
Verify
     |
     +--> lightweight Ollama reachability
     |
     +-- FAIL --> do not persist; Verify fails
     |
     +-- SUCCESS --> persist endpoint to MongoDB
                     make authoritative
                     continue remaining Verify checks
     |
     v
Overall Verify successful
     |
     v
Start Stack
     |
     v
Nuntius starts
     |
     v
Servire registers
     |
     v
Consumer service starts sufficiently
to communicate through Nuntius
     |
     v
Consumer requests required configuration
e.g. \obt servire ollama
     |
     v
Servire returns authoritative endpoint
     |
     v
Consumer completes its own initialisation
     |
     +-- SUCCESS --> continue Stack startup
     |
     +-- FAILURE --> Servire normal startup rollback
```

## Implementation Boundary

This pass deliberately does **not** implement:

-   Praebere's request/reconciliation changes;
-   Moderari's request/inference-endpoint changes;
-   Illuminates.One authorization;
-   installation identity/key management;
-   Dockerisation;
-   HAProxy.

The intended next implementation order is:

**Servire → Praebere → Moderari → final Servire acknowledgement/rollback
enforcement.**

Once these three service changes are complete and tested together, work
can proceed into the Docker/distribution stages.
