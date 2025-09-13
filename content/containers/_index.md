+++ 
title = "Docker"
type = "home"
+++

-  [Please click Here to connect](https://jiomeetpro.jio.com/shortener?meetingId=6548552155&pwd=6gBTS)
## Docker – Table of Contents

## 1. Linux Kernel & Container Fundamentals
- **Namespaces** → PID, Mount, Network, IPC, UTS, User
- **cgroups** → CPU, Memory, I/O, PIDs limits
- **Filesystem isolation** with `chroot`
- Experimenting with `unshare` to create isolated environments
- How Docker and Podman use namespaces and cgroups internally

## 2. Introduction
- What is Docker?
- Benefits of containerization
- Docker vs Virtual Machines
- Containers vs Low-Level Runtimes (CRI-O, containerd)

## 3. Installation & Setup
- Install Docker on Linux, Windows, macOS
- Docker Desktop
- Docker Engine vs Docker Desktop
- Verify installation

## 4. Docker Architecture
- Docker Daemon
- Docker Client
- Docker Images
- Docker Containers
- Docker Registries (Docker Hub, private registries)
- Low-level container runtimes: CRI-O & containerd

## 5. Working with Docker Images
- Building images with Dockerfile
- Using prebuilt images
- Managing images (`docker pull`, `docker images`, `docker rmi`)
- Image layers & storage drivers

## 6. Working with Containers
- Running containers (`docker run`)
- Listing, starting, stopping, removing containers
- Detached mode & interactive mode
- Executing commands inside containers
- Understanding isolation using **namespaces**
- Resource control using **cgroups**
- Low-level experiments with `unshare` and `chroot`

## 7. Docker Networking
- Bridge network
- Host network
- Overlay network
- Custom networks
- Port mapping & exposing services
- Network namespaces overview

## 8. Docker Volumes & Storage
- Bind mounts vs volumes
- Creating and using volumes
- Sharing data between containers
- Backup & restore volumes
- Storage namespaces & container filesystem isolation

## 9. Docker Compose
- Introduction to Compose
- docker-compose.yml structure
- Multi-container applications
- Environment variables & scaling

## 10. Docker Security
- Best practices
- User namespaces
- Scanning images
- Secrets management
- Seccomp & AppArmor profiles

## 11. Docker in CI/CD
- Using Docker in pipelines
- Building and pushing images
- Deploying with Docker
- Integration with Kubernetes using CRI-O/containerd

## 12. Advanced Docker
- Multi-stage builds
- Health checks
- Resource limits (CPU & memory)
- Logging & monitoring
- Low-level runtime configuration

## 13. Troubleshooting
- Common errors
- Debugging containers
- Checking logs & events
- Using `docker inspect` and runtime debug tools

## 14. Docker vs Alternatives
- Podman
- LXC/LXD
- CRI-O & containerd


---
