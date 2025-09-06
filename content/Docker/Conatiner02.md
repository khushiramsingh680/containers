+++
title = "Container 02"
weight = 3
+++



## Open Container Initiative (OCI) and Container Standards

## What is OCI?

The **Open Container Initiative (OCI)** is an open governance project under the Linux Foundation started in 2015. It creates open standards for container formats and runtimes to ensure interoperability and portability across container platforms.

## Key OCI Specifications

1. **OCI Image Format Specification**  
   Defines a standard image format including layers, manifests, and metadata so images are portable across tools.

2. **OCI Runtime Specification**  
   Defines how to run containers using namespaces, cgroups, hooks, and other OS features.

3. **OCI Distribution Specification**  
   Defines how images are pushed and pulled from registries.

## Why OCI Matters

Before OCI, different tools had incompatible container formats and runtimes leading to vendor lock-in and fragmentation. OCI solves this by standardizing image and runtime formats.

## Related Concepts

- **Docker Image**: Initially a proprietary format; now OCI compatible.  
- **runc**: Reference OCI runtime implementation.  
- **containerd**: Daemon managing container lifecycle and image transfer, built on OCI specs.  
- **CRI-O**: Kubernetes container runtime that uses OCI standards.  
- **Kubernetes**: Container orchestration platform supporting OCI images and runtimes.

## Summary Table

| Term            | Description                                  |
|-----------------|----------------------------------------------|
| **OCI**         | Open standards for container images and runtimes. |
| **OCI Image Spec**   | Defines container image format.             |
| **OCI Runtime Spec** | Defines container runtime behavior.         |
| **Docker Image**     | Container image format compatible with OCI. |
| **runc**         | Reference OCI runtime implementation.         |
| **containerd**   | Container runtime daemon implementing OCI specs. |
| **CRI-O**        | Kubernetes runtime using OCI standards.        |


# Container Runtimes and Image Formats Explained

## Container Runtimes

A **container runtime** is the software responsible for running containers according to the container image and runtime specifications. It handles creating the container, setting up namespaces, cgroups, and other OS-level features.

### 1. runc

- **What is it?**  
  `runc` is the reference implementation of the [OCI Runtime Specification](https://github.com/opencontainers/runtime-spec).  
- **Role:**  
  It is a lightweight CLI tool to spawn and run containers using Linux namespaces, cgroups, and other kernel features.  
- **Usage:**  
  `runc` itself is mostly used as a low-level component by other container engines.  
- **Example:**  
  Docker and containerd use `runc` under the hood to create and run containers.

### 2. containerd

- **What is it?**  
  `containerd` is a daemon that manages the entire container lifecycle: image transfer, container execution, storage, and supervision.  
- **Role:**  
  Acts as an industry-standard container runtime daemon that uses `runc` for low-level container execution.  
- **Features:**  
  - Pull and manage container images.  
  - Manage container snapshots and storage.  
  - Manage container execution and lifecycle.  
- **Usage:**  
  Widely used by Docker (as its runtime backend) and Kubernetes (via CRI plugins).

### 3. CRI-O

- **What is it?**  
  CRI-O is an OCI-compliant container runtime specifically designed for Kubernetes.  
- **Role:**  
  Provides a lightweight runtime focused solely on Kubernetes' Container Runtime Interface (CRI).  
- **Features:**  
  - Supports OCI images and runtimes.  
  - Integrates tightly with Kubernetes.  
  - Avoids unnecessary components to reduce overhead.  
- **Usage:**  
  An alternative to `containerd` for Kubernetes environments.

---

## Container Image Formats: OCI vs Docker

### Docker Image Format

- Originally developed by Docker Inc.  
- Used proprietary format with JSON manifests and layers.  
- Early versions had some incompatibilities with other runtimes.

### OCI Image Format

- Created by the Open Container Initiative (OCI) to standardize container images.  
- Based on the Docker image format but with a well-defined spec for manifests, layers, and configuration.  
- Ensures that images built by any OCI-compliant tool can run anywhere.

### Key Differences

| Feature              | Docker Image Format                     | OCI Image Format                         |
|----------------------|---------------------------------------|-----------------------------------------|
| Origin               | Docker Inc.                           | Open Container Initiative (Linux Foundation) |
| Specification        | Initially proprietary                 | Open, community-driven specification     |
| Compatibility        | Supported by Docker and many runtimes| Supported by Docker, containerd, Podman, CRI-O |
| Focus                | Docker ecosystem                     | Interoperability and standardization    |

### Why It Matters

- The OCI image format promotes portability and interoperability across container tools and platforms.
- Most modern container tools now support OCI images as a standard.

---

## Summary Table

| Term          | Description                                                                                   |
|---------------|-----------------------------------------------------------------------------------------------|
| **runc**      | Reference OCI runtime; low-level container executor using Linux kernel features.              |
| **containerd**| Container runtime daemon managing lifecycle, image transfer, storage; uses `runc` internally.|
| **CRI-O**     | Lightweight Kubernetes-focused OCI runtime implementing CRI interface.                        |
| **Docker Image** | Original Docker container image format, now largely compatible with OCI images.             |
| **OCI Image** | Open standard container image format promoting portability and interoperability.             |

---

# Container Runtimes: High-Level vs Low-Level Explained

---

## What is a Container Runtime?

A container runtime is software that runs containers by managing container lifecycle, image handling, and execution.

---

## High-Level vs Low-Level Container Runtimes

| Aspect                  | High-Level Runtime                | Low-Level Runtime (OCI Runtime)     |
|-------------------------|---------------------------------|------------------------------------|
| **Role**                | Manages container lifecycle, image pull, storage, networking, etc. | Creates and runs containers using kernel features like namespaces and cgroups |
| **Examples**            | containerd, CRI-O, Docker Engine | runc, crun, kata-runtime            |
| **Responsibilities**    | - Pull and manage images<br>- Manage container lifecycle (start, stop)<br>- Handle storage and snapshots<br>- Provide API for orchestration tools | - Spawn container processes<br>- Set up namespaces, cgroups, capabilities<br>- Execute container according to OCI runtime spec |
| **Interaction**         | Calls low-level runtime to actually start containers | Runs as subprocess invoked by high-level runtime |
| **Usage in Kubernetes** | containerd and CRI-O implement Kubernetes CRI interface | Usually runc (or alternative OCI runtimes) used by containerd or CRI-O to execute containers |
| **Installation**        | Runs as daemon/service          | CLI tools or plugins                 |
| **Complexity**          | More complex, handles many aspects of container lifecycle | Lightweight, focused on execution only |

---

## How They Work Together

- The **high-level runtime** manages everything except the actual container start.
- When a container needs to start, the high-level runtime calls the **low-level runtime**.
- The **low-level runtime** (like `runc`) creates the container process using Linux kernel primitives (namespaces, cgroups).
- This separation enables modularity, flexibility, and easier maintenance.

---

## Examples

| High-Level Runtime | Low-Level Runtime (OCI Runtime)   | Notes                            |
|--------------------|----------------------------------|---------------------------------|
| Docker Engine      | runc                             | Docker uses containerd and runc |
| containerd         | runc or crun                    | Kubernetes often uses this combo |
| CRI-O              | runc or crun                    | Kubernetes-focused lightweight runtime |

---

## Why This Design?

- **Modularity:** Easier to swap out low-level runtimes without changing lifecycle management.
- **Standardization:** `runc` follows OCI spec as a reference runtime.
- **Performance & Security:** Alternative runtimes like `crun`, `kata-runtime`, or `gvisor` can be plugged in for specialized needs.

---

## Summary Table

| Component            | Description                                  |
|----------------------|----------------------------------------------|
| **High-Level Runtime**| Container lifecycle, image management, API  |
| **Low-Level Runtime** | Runs containers using kernel features        |
| **runc**             | Reference low-level OCI runtime               |
| **containerd**       | Popular high-level runtime daemon             |
| **CRI-O**            | Kubernetes-specific high-level runtime        |
| **crun**             | Lightweight alternative low-level runtime    |
| **kata-runtime**     | VM-based low-level runtime for enhanced security |

---


## Using Multiple Low-Level Container Runtimes

---

## Can We Have Multiple Low-Level Runtimes?

Yes, it is possible and common to have **multiple low-level runtimes** installed and configured on the same system.

---

## Why Use Multiple Low-Level Runtimes?

- Different workloads may require different runtimes for:
  - **Performance** (e.g., `crun` is faster and lighter than `runc`)
  - **Security** (e.g., `kata-runtime` provides VM-level isolation)
  - **Specialized environments** or sandboxing

---

## How to Configure Multiple Runtimes in containerd

In the containerd configuration file (`/etc/containerd/config.toml`), you can define multiple runtimes:

```toml
[plugins.cri.containerd.runtimes.runc]
  runtime_type = "io.containerd.runc.v2"
  runtime_engine = "/usr/bin/runc"

[plugins.cri.containerd.runtimes.crun]
  runtime_type = "io.containerd.runc.v2"
  runtime_engine = "/usr/bin/crun"

[plugins.cri.containerd.runtimes.kata]
  runtime_type = "io.containerd.kata.v2"
  runtime_engine = "/usr/bin/kata-runtime"
```
## Selecting Runtime Per Container or Pod

- When creating containers or pods, specify which runtime to use.
- In Kubernetes, this is done via RuntimeClass.
   ```sh
    apiVersion: node.k8s.io/v1
    kind: RuntimeClass
    metadata:
    name: kata
    handler: kata
   ```

## Commands to Verify Installed Container Runtimes

---

### Check installed low-level runtimes (common locations)

```bash
which runc
which crun
which kata-runtime
```
### Check running containerd service
```sh
systemctl status containerd
ps aux | grep containerd
```
### Verify configured runtimes in containerd config
```sh
cat /etc/containerd/config.toml | grep -A 5 runtimes
```
### Check default runtime Docker uses
```sh
docker info | grep -i runtime
```
### kubectl get runtimeclass
```sh
kubectl get runtimeclass
```
### Inspect container runtime for a specific pod
```sh
kubectl get pod <pod-name> -o jsonpath='{.spec.runtimeClassName}'
```

## Containers Under the Hood: The Core Concepts

### 1. Linux Namespaces – Isolation  
Namespaces provide **process isolation** by creating separate instances of global system resources for each container. This ensures that containers have their own view of:

- **PID namespace:** Separate process IDs inside each container (process 1 inside container ≠ host process 1)  
- **Mount namespace:** Isolated filesystem mount points  
- **Network namespace:** Independent network interfaces, IP addresses, and ports  
- **UTS namespace:** Separate hostname and domain name  
- **IPC namespace:** Isolated inter-process communication mechanisms (message queues, semaphores)  
- **User namespace:** Maps user and group IDs, allowing privilege isolation  
- **Cgroup namespace:** Isolated view of control groups (resource groups)

---

### 2. Control Groups (cgroups) – Resource Management  
Cgroups limit and prioritize the resources (CPU, memory, disk I/O, network) that containers can use. This prevents one container from hogging all system resources and impacting others.

- You can set limits like max CPU shares or memory usage  
- The kernel enforces these limits at runtime  
- Important for multi-tenant environments and resource fairness  

---

### 3. Union File Systems (OverlayFS, AUFS, etc.) – Efficient Storage  
Containers use union filesystems to build layered images efficiently:

- Base OS layers + application layers stacked without duplication  
- Containers share read-only layers; only container-specific changes are stored separately (copy-on-write)  
- This reduces storage space and speeds up container start times  

---

### 4. Container Runtime  
The container runtime (e.g., **containerd**, **CRI-O**, or Docker Engine) is responsible for:

- Creating namespaces and cgroups  
- Setting up network interfaces (via CNI plugins)  
- Mounting the filesystem layers  
- Starting and managing container processes  

---

### 5. Networking  
Containers get isolated network stacks:

- Virtual Ethernet interfaces (veth pairs) connect container namespace to host network bridge  
- IP addresses assigned per container  
- Network policies enforce traffic rules (firewalls, routing)  

---

### 6. Process Lifecycle  
Containers run as regular processes on the host but inside isolated namespaces and resource limits:

- PID 1 inside the container is typically the main application process  
- Signals and process control are managed inside namespaces, isolated from the host  

---

### Summary  
Containers combine Linux kernel features (namespaces + cgroups + union filesystems) with runtime tooling to create **lightweight, isolated, and resource-controlled environments** for running applications consistently anywhere.
