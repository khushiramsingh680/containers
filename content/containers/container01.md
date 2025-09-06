+++
title = "part 01"
weight = 1
+++

## **Topics**
### 1. **Linux Namespaces**
### 2. **Linux Cgroups**
### 3. **Unshare**
### 4. **chroot**

### Introduction: What Are Linux Namespaces?

By default, **all processes on a Linux system share the same namespaces** for things like process IDs, network, mounts, etc. This means:

- All processes see the same list of processes.
- All processes share the same network interfaces and IP addresses.
- All processes see the same filesystem mount points.
- All processes share the same hostname.

---

### Why Do We Need Namespaces?

Namespaces allow the Linux kernel to **isolate and virtualize system resources** so that a group of processes can have their own view of the system. This is key for:

- Containers (Docker, Kubernetes, OpenShift)
- Sandboxed environments
- Security isolation

---

### How to Create New Namespaces?

Linux provides the `unshare` command and APIs to create new namespaces. When a process is created with new namespaces, it gets its own isolated view of resources.

Containers like Docker also create new namespaces automatically when they launch.

---

### Types of Linux Namespaces and What They Isolate

| Namespace Type     | What It Isolates                    | Example of Isolation                         |
|--------------------|-----------------------------------|----------------------------------------------|
| **PID namespace**   | Process IDs                      | Separate process trees per container          |
| **Mount namespace** | Filesystem mount points           | Containers have separate filesystem views     |
| **Network namespace** | Network devices, IPs, ports      | Containers have their own network interfaces  |
| **UTS namespace**   | Hostname and domain name           | Containers have separate hostnames             |
| **IPC namespace**   | Interprocess communication         | Separate message queues and semaphores        |
| **User namespace**  | User and group IDs                 | Containers can remap users for security       |
| **Cgroup namespace**| View of cgroups                   | Isolated view of resource groups               |

---

### Hands-On Lab

### 1. Verify All Processes Share the Same Namespaces by Default

Run this on your current shell:

    echo "Current shell PID: $$"
    ls -l /proc/$$/ns/

You will see namespaces like `pid`, `net`, `mnt`, `uts`, etc. All processes started normally share these namespaces.

---

### 2. Create a New Shell with New Namespaces Using `unshare`

Run the following command to launch a new shell with new mount, UTS, IPC, network, and PID namespaces:

    sudo unshare --mount --uts --ipc --net --pid --fork /bin/bash

This starts a bash shell inside the new namespaces.  
Inside this new shell:

- The `hostname` is separate.
- Process IDs start fresh (your bash is PID 1).
- Mounted filesystems are isolated.
- Network interfaces are isolated.

---

### 3. Test Isolation Examples

- Change hostname:

      hostname my-container
      echo "New hostname: $(hostname)"

- Check process list (bash should be PID 1):

      ps aux

- Mount a new filesystem:

      mount -t tmpfs tmpfs /mnt
      mount | grep /mnt

- Check network interfaces:

      ip addr

Open another terminal and check the hostname, mounts, and processes there — they remain unaffected.

---

### 4. Exit New Namespace Shell

    exit

---

### Bonus: Explore Namespaces in Docker Containers

Docker automatically creates new namespaces for each container.

Start a container:

    docker run -it --rm alpine sh

Inside container:

    hostname
    ps aux
    ls -l /proc/self/ns/

On host, find the container’s PID:

    docker inspect --format '{{.State.Pid}}' <container_id_or_name>
    ls -l /proc/<container_pid>/ns/

---

### Summary

- Without `unshare` or container runtimes, all processes share the same namespaces.
- Namespaces allow resource and process isolation needed by containers.
- `unshare` can be used to experiment with namespaces manually.
- Containers (Docker, Kubernetes) use namespaces to create isolated environments for workloads.

---

## Introduction to cgroups (Control Groups)

While namespaces isolate *what* a process sees (e.g., process list, network), **cgroups control how much resources a process or group of processes can use**.

cgroups allow you to limit, prioritize, and account for CPU, memory, disk I/O, and network bandwidth for processes.

---

### How cgroups Work

- You create cgroups (control groups) and assign processes to them.
- You can configure limits, such as max CPU shares or memory limits, per cgroup.
- The Linux kernel enforces these limits, ensuring processes don’t exceed allocated resources.

---

### Common cgroups Controllers

| Controller | Resource Controlled             | Description                            |
|------------|-------------------------------|------------------------------------|
| `cpu`      | CPU scheduling and usage       | Limits CPU time for processes      |
| `memory`   | Memory usage                   | Limits RAM usage, triggers OOM     |
| `blkio`    | Block device I/O              | Controls disk read/write bandwidth |
| `net_cls`  | Network traffic classification | Controls and tags network traffic  |

---

### Hands-On Lab: Basic cgroups Usage

### 1. Check if cgroups are mounted

    mount | grep cgroup

You should see various cgroup controllers mounted, e.g., `/sys/fs/cgroup/cpu`, `/sys/fs/cgroup/memory`, etc.

---

### 2. Create a cgroup and limit CPU

    sudo mkdir /sys/fs/cgroup/cpu/mygroup
    echo 50000 | sudo tee /sys/fs/cgroup/cpu/mygroup/cpu.cfs_quota_us
    echo 100000 | sudo tee /sys/fs/cgroup/cpu/mygroup/cpu.cfs_period_us

This limits the cgroup to 50% of a CPU core.

---

### 3. Add a process to the cgroup

Get the PID of a process (e.g., a bash shell):

    pidof bash

Add the PID to the cgroup:

    echo <pid> | sudo tee /sys/fs/cgroup/cpu/mygroup/cgroup.procs

---

### 4. Verify the limits

Check the cgroup usage and limits in `/sys/fs/cgroup/cpu/mygroup`.

---

## How Namespaces and cgroups Work Together

- **Namespaces** isolate *what processes see* (virtualization of system resources).
- **cgroups** limit *how much resources* those processes can consume.

Together, they form the backbone of container runtime isolation and resource management.

---
## Common Docker Resource Control Flags

| Resource | Docker Flag           | Description                           |
|----------|----------------------|-------------------------------------|
| CPU      | `--cpus`, `--cpu-shares`, `--cpu-quota`, `--cpu-period` | Limit CPU usage                      |
| Memory   | `--memory`, `--memory-swap` | Limit container memory usage        |
| Block I/O | `--device-read-bps`, `--device-write-bps` | Limit disk read/write bandwidth     |
| PIDs     | `--pids-limit`       | Limit number of processes in container |

---

## CPU Limiting Examples

### Limit container to use 0.5 CPU cores

```bash
docker run --rm -it --cpus="0.5" ubuntu bash
```

### Set CPU shares (relative weight)
```sh
docker run --rm -it --cpu-shares=512 ubuntu bash
```

### Limit CPU usage using quota and period
```sh
Limit to 50% of one CPU:

docker run --rm -it --cpu-quota=50000 --cpu-period=100000 ubuntu bash
```

### Limit Number of Processes (PIDs)
```sh
docker run --rm -it --pids-limit=50 ubuntu bash
```
## Summary


- Namespaces provide resource isolation.
- cgroups provide resource control.
- Containers like Docker and Kubernetes use both heavily to provide secure, isolated, and resource-controlled environments.


# chroot vs unshare

| Aspect               | `chroot`                               | `unshare`                                   |
|----------------------|--------------------------------------|---------------------------------------------|
| **Purpose**          | Change the root directory (`/`) for a process to isolate filesystem view. | Create new namespaces to isolate various resources (PID, network, mount, etc.). |
| **Isolation type**   | Filesystem isolation only.            | Can isolate multiple resources: PID, network, mount, UTS, IPC, user, etc. |
| **Security**         | Limited; processes can escape `chroot` if privileged. | Much stronger isolation; used for containers and sandboxing. |
| **Usage**            | Run a command with a different root filesystem. | Run a command with new kernel namespaces. |
| **Example**          | `chroot /new/root /bin/bash`          | `unshare --mount --pid --net /bin/bash`     |
| **Scope**            | Filesystem namespace only.             | Multiple namespaces, depending on options. |
| **Container use**    | Sometimes used historically for jail-like environments, but insufficient alone. | Core kernel primitive for containers and sandboxing. |
| **Privilege**        | Usually requires root.                  | Usually requires root for some namespaces (e.g., network), but user namespaces allow unprivileged use. |

---

### Summary

- `chroot` changes **only the filesystem root** for a process.
- `unshare` creates **new namespaces**, providing isolation for many system resources, not just filesystem.
- `unshare` is much more powerful and is the basis for modern container runtimes.
- `chroot` can be part of isolation but is often not secure enough alone.



## Docker Examples: Running Processes in Shared Namespaces

---

## 1. Share PID Namespace Between Containers

Allows containers to see and interact with each other's processes.

```bash
# Start a container in detached mode
docker run -dit --name container1 ubuntu sleep 1000

# Start another container sharing PID namespace with container1
docker run -dit --name container2 --pid=container:container1 ubuntu sleep 1000

# Exec into container2 and see processes (should see container1's processes too)
docker exec -it container2 bash
ps aux
```

## 2. Share Network Namespace Between Containers
### Start container1 normally
```sh
docker run -dit --name net1 ubuntu sleep 1000
```

### Start container2 sharing network namespace of net1
```sh
docker run -dit --name net2 --net=container:net1 ubuntu sleep 1000
```
### Inside net2, check IP address (should be same as net1)
```sh
docker exec -it net2 ip addr
```

## 3. Share IPC Namespace Between Containers
```sh
docker run -dit --name ipc1 ubuntu sleep 1000
docker run -dit --name ipc2 --ipc=container:ipc1 ubuntu sleep 1000
```
### Check IPC namespace inode inside ipc2
```sh
docker exec ipc2 ls -l /proc/self/ns/ipc
```

### 4. Share UTS Namespace Between Containers
```sh
docker run -dit --name uts1 ubuntu sleep 1000
docker run -dit --name uts2 --uts=container:uts1 ubuntu sleep 1000
```

### Change hostname in uts1 container
```sh
docker exec uts1 hostname shared-host
```

### Check hostname in uts2 container (should be "shared-host")
```sh
docker exec uts2 hostname
```

## 5. Share Mount Namespace Between Containers (Experimental)
```sh
docker run -dit --name mount1 ubuntu sleep 1000
docker run -dit --name mount2 --mount=container:mount1 ubuntu sleep 1000
```

## 6. Share Host PID Namespace
```sh
docker run -it --pid=host ubuntu bash
```
### Now ps aux inside container shows all host processes
```sh
ps aux
```


## Tasks to do 
- [ ] Install docker 
