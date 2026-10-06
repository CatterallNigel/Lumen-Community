# Lumen Researcher Foundation Build — Beta
## M0.1 Researcher Installation Quick Start

This guide contains the essential commands and operational information required to install and run the **Lumen Researcher Foundation Build — Beta (M0.1)**.

> **Important:** Lumen M0.1 is an authorised research distribution. Access to the Docker image and authorisation to operate an installation are separate controls.

---

## 1. Files required

Keep these two files together in the same installation directory:

- `compose-registry-m0.1.yaml`
- `haproxy.cfg`

Open **Command Prompt** or **PowerShell** in that directory before running the commands below.

Docker Desktop must be installed and running. Ollama must also be installed and running on the host machine before Lumen can use local models.

---

## 2. Log in to the Illuminates.One Docker Registry

Your Docker Registry username and password are supplied separately by Illuminates.One.

Run:

```cmd
docker login registry.illuminates.one
```

When prompted, enter the supplied username and password.

A successful login reports:

```text
Login Succeeded
```

The Registry credentials allow Docker to retrieve the Lumen image. They are **not** the Lumen installation authorisation.

Do not share your Registry credentials.

---

## 3. Install and start Lumen

From the directory containing `compose-registry-m0.1.yaml` and `haproxy.cfg`, run:

```cmd
docker compose -f compose-registry-m0.1.yaml up -d
```

Docker will pull the required images if they are not already present and create the persistent volumes required by this installation.

To see the running containers:

```cmd
docker compose -f compose-registry-m0.1.yaml ps
```

The expected containers are:

```text
lumen-m01
lumen-m01-mongodb
lumen-m01-haproxy
```

---

## 4. Open Servire

On the machine running Lumen, open:

```text
http://127.0.0.1:11439
```

On first use, Lumen will present the **Lumen M0.1 Research Licence**. The licence must be accepted before the installation can proceed.

Follow the Servire workflow to validate the external dependencies and start the Lumen Stack.

### Configure Ollama before clicking Validate Lumen

If the Docker Compose installation is healthy, MongoDB does not normally require any researcher configuration. **Ollama is the external dependency that needs particular attention before clicking `Validate Lumen`.**

Lumen M0.1 defaults the Ollama connection to:

```text
Host: host.docker.internal
Port: 11434
```

`host.docker.internal` allows Lumen running inside Docker to reach Ollama running on the host machine. With the default Ollama installation this corresponds to the host-side Ollama service on port `11434`.

If your Ollama host or port is different:

1. Change the Ollama URL/host and/or port in Servire.
2. Click **Apply**.
3. Then click **Validate Lumen**.

**Do not forget to click Apply after changing the Ollama settings.** Validation uses the applied configuration.

If **Validate Lumen** fails, click **Show validation details**. The validation details identify which check failed and provide the information needed to diagnose the configuration problem.

Only proceed to start the managed Lumen Stack once validation succeeds.

---

## 5. Stop and restart Lumen

To stop the installation without deleting its persistent state:

```cmd
docker compose -f compose-registry-m0.1.yaml stop
```

To start it again:

```cmd
docker compose -f compose-registry-m0.1.yaml start
```

Alternatively, to remove the containers while **preserving the installation volumes**:

```cmd
docker compose -f compose-registry-m0.1.yaml down
```

and recreate them with:

```cmd
docker compose -f compose-registry-m0.1.yaml up -d
```

Because the persistent volumes remain, this does not normally create a new Lumen installation identity.

---

## 6. IMPORTANT — do not delete the Lumen volumes unless you intend to reinstall

The Lumen installation identity and persistent installation state are stored outside the disposable application container.

Running:

```cmd
docker compose -f compose-registry-m0.1.yaml down -v
```

deletes the associated persistent volumes.

**This should not be used as a normal uninstall/restart command.**

Deleting those volumes and reinstalling Lumen creates a **new installation identity**.

The previous authorisation belongs to the previous installation identity. A newly created installation will therefore **not authenticate as the previously authorised installation**.

If the volumes have been deleted, or Lumen is deliberately being moved/reinstalled as a new installation, contact **Illuminates.One / Nigel Catterall** and request that the research authorisation be reset to permit the new installation.

Do this **before expecting the replacement installation to authorise successfully**.

Simply downloading the Docker image again does not transfer or restore the previous installation authorisation.

---

## 7. Updating/re-pulling the M0.1 image

The M0.1 research image is distributed from:

```text
registry.illuminates.one/lumen:m0.1
```

To explicitly pull it:

```cmd
docker pull registry.illuminates.one/lumen:m0.1
```

The frozen M0.1 image digest is:

```text
sha256:87e857e7a9115a12d72954bd5c17117f676945b885ad87e4a0fdfcb8d2bfc8d0
```

You can inspect the locally retrieved image with:

```cmd
docker image inspect registry.illuminates.one/lumen:m0.1
```

Do not modify or rebuild the supplied M0.1 image. The research distribution is intended to run from the published Illuminates.One image.

---

## 8. Useful diagnostic commands

Check container status:

```cmd
docker compose -f compose-registry-m0.1.yaml ps
```

View Lumen container output:

```cmd
docker logs lumen-m01
```

Follow Lumen output continuously:

```cmd
docker logs -f lumen-m01
```

View HAProxy output:

```cmd
docker logs lumen-m01-haproxy
```

View MongoDB output:

```cmd
docker logs lumen-m01-mongodb
```

If reporting a problem, please describe what you were doing, what you expected to happen, and what actually happened. Relevant Servire operational logs are also useful when available.

---

## 9. Important M0.1 notes

- This is the **Lumen Researcher Foundation Build — Beta**, release **M0.1**.
- It is a research distribution, not a production service.
- The supplied installation is intended for a single researcher on a single machine.
- Do not redistribute the research distribution or Registry credentials.
- Do not attempt to bypass or modify the Lumen authorisation mechanism.
- Lumen requires periodic authorisation with Illuminates.One in order to continue operating.
- Temporary Internet connectivity loss does not necessarily cause immediate loss of operation; Lumen's authorisation mechanism provides for temporary connectivity failures.
- Lumen must be able to connect to the Ollama endpoint configured in Servire. The M0.1 default is `host.docker.internal:11434`, which is appropriate when Ollama is running on the same host machine using its default port. If Ollama is running elsewhere, configure the appropriate host and port in Servire, click Apply, and then Validate Lumen.
- The Docker Registry login controls access to the published image. Lumen runtime authorisation is a separate mechanism.
- For M0.1, Servire log export is known to have limitations when Lumen is running in Docker. This does not prevent normal Lumen operation.

---

## 10. Normal shutdown

When finished, a normal shutdown that preserves the installation is:

```cmd
docker compose -f compose-registry-m0.1.yaml down
```

To run Lumen again later:

```cmd
docker compose -f compose-registry-m0.1.yaml up -d
```

**Do not add `-v` unless you intentionally want to destroy the installation's persistent state and understand that Illuminates.One will need to reset the authorisation before a newly created installation can be authorised.**

---

**Illuminates.One — Lumen Researcher Foundation Build — Beta (M0.1)**
