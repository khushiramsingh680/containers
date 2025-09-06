+++
title = "Part 03"
weight = 3
+++


## Docker Architecture – Workflow
```txt
+----------------+      Unix Socket / REST API       +----------------+
|  Docker Client |  ---------------------------->   |  Docker Daemon |
|  (CLI / GUI)   |                                |  (dockerd)     |
+----------------+                                +----------------+
                                                        |
                                                        | Uses container runtime (containerd)
                                                        v
                                              +-----------------------+
                                              |  Container Runtime     |
                                              |  (containerd / CRI-O) |
                                              +-----------------------+
                                                        |
                                                        | Creates & manages
                                                        v
                                              +-----------------------+
                                              |  Containers           |
                                              |  (Running Apps)       |
                                              +-----------------------+
                                                        ^
                                                        | Images
                                                        |
                                              +-----------------------+
                                              | Docker Registry       |
                                              | (Docker Hub / Private)|
                                              +-----------------------+
```
```txt
Docker CLI
   |
   v
Docker Daemon (dockerd)
   |
   v
containerd  (high-level runtime)
   |
   v
runc       (low-level runtime, OCI-compliant)
   |
   v
Linux Kernel (namespaces, cgroups, filesystem)
```
## Components Explained

### 1. Docker Client (CLI)
- Interface to interact with Docker (`docker run`, `docker build`).
- Sends commands to Docker daemon via:
  - **Unix socket** (`/var/run/docker.sock`)
  - **TCP/REST API** (optional, for remote management)

### 2. Docker Daemon (`dockerd`)
- Background service managing containers, images, networks, and volumes.
- Receives commands from CLI.
- Delegates container creation to the container runtime.

### 3. Container Runtime
- Low-level manager responsible for running containers.
- Docker uses **containerd** by default.
- Handles:
  - Image unpacking
  - Container lifecycle
  - Storage and logging
- Communicates with daemon via **API or socket**.

### 4 Low-level Runtime

- The **low-level runtime** is responsible for actually creating and running containers on the host OS.
- **runc** is the default low-level runtime used by Docker, containerd, and CRI-O.
- Responsibilities of the low-level runtime:
  - Creating **namespaces** for process isolation
  - Managing **cgroups** for resource limits
  - Mounting **filesystems** for container file isolation
  - Directly interacting with the **Linux kernel** to run containers
- It implements the **OCI runtime specification**, ensuring containers are standardized and portable.
- High-level runtimes like **containerd** or **CRI-O** call the low-level runtime to execute containers.


### 5. Docker Registries
- Store and distribute container images.
- **Docker Hub**: Public registry.
- **Private registries**: Internal organization use.
- Commands: `docker pull`, `docker push`.

### 6. Communication Flow
1. User runs `docker run nginx`.
2. CLI sends request via **Unix socket** to daemon.
3. Daemon pulls image from **registry** (if needed).
4. Daemon delegates container creation to **containerd**.
5. containerd interacts with OS kernel to start the container.
6. Daemon sends container info back to CLI.

### 7. Socket Exposure
- **Unix socket**: `/var/run/docker.sock` (local access, secure)
- **TCP socket**: `tcp://0.0.0.0:2375` (remote access, insecure)
- Kubernetes CRI sockets:
  - CRI-O: `/var/run/crio/crio.sock`
  - containerd: `/run/containerd/containerd.sock`

