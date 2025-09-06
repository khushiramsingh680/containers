+++
title = "Part 04"
weight = 4
+++



## 1. Container Related commands Commands

### Lifecycle
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
docker run -d -p 8080:80 --name webserver nginx
docker start CONTAINER_NAME_OR_ID
docker stop CONTAINER_NAME_OR_ID
docker restart CONTAINER_NAME_OR_ID
docker rm CONTAINER_NAME_OR_ID
docker rm -f CONTAINER_NAME_OR_ID
docker pause CONTAINER_NAME
docker unpause CONTAINER_NAME
docker kill CONTAINER_NAME
```
### Monitoring & Inspection
```sh
docker ps
docker ps -a
docker inspect CONTAINER_NAME
docker logs CONTAINER_NAME
docker logs -f CONTAINER_NAME
docker stats
docker stats CONTAINER_NAME
docker exec -it CONTAINER_NAME bash
docker attach CONTAINER_NAME
docker top CONTAINER_NAME
docker exec CONTAINER_NAME env
docker inspect -f '{{ .Mounts }}' CONTAINER_NAME
```

### Management
```sh
docker network inspect NETWORK_NAME
docker network connect NETWORK_NAME CONTAINER_NAME
docker network disconnect NETWORK_NAME CONTAINER_NAME
docker run --network NETWORK_NAME IMAGE_NAME
```

### misc
```sh
docker run -it IMAGE_NAME bash
docker run -d IMAGE_NAME
docker run -d --name limited_container --memory="512m" --cpus="1.0" IMAGE_NAME
docker ps -q -f ancestor=IMAGE_NAME
```
- [ ] Install mysql in docker
- [ ] Install nginx
- [ ] Try to connect from nginx container

- [Proejct Link](https://github.com/khushiramsingh680/studentapp)
    - [ ] create build ( war file)
    - [ ] set up mysql
    - [ ]  Set up Tomcat
    - [ ] Test if the app is able to persist the data 

## Docker Images 



## 1. What is a Docker Image
- A **Docker image** is a **read-only template** used to create containers.
- It contains:
  - Application code
  - Runtime
  - Libraries
  - Dependencies
  - OS-level configurations (optional)
- Images are **composed of layers**, each layer representing a filesystem change.

---

## 2. Image Layers and Filesystem
- Docker images are built using **layers**, each layer represents a filesystem change.
- Layers are **stacked** on top of each other using **union filesystems**.
- **Union filesystem** allows layers to be combined into a single view for the container.
- Examples of union filesystems Docker uses:
  - `overlay2` (default on modern Linux)
  - `aufs` (older systems)
  - `btrfs`
  - `zfs`

### How Layers Work
1. **Base layer**: Usually an OS image like `ubuntu` or `alpine`.
2. **Intermediate layers**: Created by `RUN`, `COPY`, `ADD` commands in Dockerfile.
3. **Top layer (container layer)**: Writable layer when container is running.
4. **Caching**: Docker reuses unchanged layers to speed up builds.

---

## 3. Image Storage Purpose
- Images are stored **locally** to allow fast container creation without downloading every time.
- Stored in **Docker’s storage driver directory** (depends on filesystem/driver):
  - Default for Linux with `overlay2`: `/var/lib/docker/overlay2/`
- Each image layer is **read-only**; containers get a **writable top layer** on top.

---

## Comparison of Docker Storage Drivers / Filesystems

| Storage Driver | Default on | Features | Pros | Cons | Use Case |
|----------------|------------|---------|------|------|---------|
| **overlay2**   | Modern Linux | OverlayFS-based union filesystem | Fast, efficient, low storage overhead, supports copy-on-write | Requires modern kernel (≥ 4.0) | Recommended for most modern Docker setups |
| **aufs**       | Older Linux | Multi-layered union filesystem | Supports multiple read-only layers, writable top layer | Deprecated, slower than overlay2, not in mainline kernel | Legacy systems |
| **btrfs**      | Select distributions | Copy-on-write filesystem, snapshots | Snapshots & rollbacks, efficient storage, checksums for data integrity | Requires btrfs kernel support, more complex | Advanced use, snapshot & rollback capability |
| **zfs**        | Select distributions | Advanced filesystem with COW | Snapshots, clones, compression, large-scale datasets | Requires ZFS kernel support, more complex setup | Enterprise setups, large storage, snapshot & clone heavy workflows |
| **vfs**        | All Linux (fallback) | Simple, non-union filesystem | Simple, no kernel requirements, works everywhere | No copy-on-write, poor performance, large storage use | Testing, environments where no unionfs is available |

---

## Key Points
- **overlay2** is the modern default and most recommended for production Docker use.
- **aufs** is legacy and mostly phased out.
- **btrfs** and **zfs** provide advanced features like snapshots and cloning but require extra setup and kernel support.
- **vfs** is a simple fallback driver that doesn’t use copy-on-write; very slow and storage-heavy.
- All drivers (except vfs) support **union filesystem concepts**, enabling Docker image layers (read-only) and container writable layers efficiently.


## Copy-on-Write (CoW) in Docker

## What is Copy-on-Write?
Copy-on-Write (CoW) is a storage optimization technique used by Docker’s storage drivers (overlay2, aufs, btrfs, zfs).  
Instead of duplicating files/layers, Docker uses *references* to existing read-only image layers.  
A copy is only made **when a modification occurs**.

---

## How It Works
1. **Image Layers**:
   - Docker images are built in layers (e.g., base OS, updates, application code).
   - These layers are **read-only**.

2. **Container Creation**:
   - When a container starts, Docker adds a **thin writable layer** (container layer) on top of the image layers.
   - The container can write new files or modify existing ones in this writable layer.

3. **File Modification (Copy-on-Write Trigger)**:
   - If a process modifies a file from a lower (read-only) image layer:
     - The file is first **copied** into the container’s writable layer.
     - The process then modifies this copied version.
   - The original file in the image layer remains unchanged.

---

## Example
- Image has `/etc/config.txt` in a read-only layer.
- Container modifies `/etc/config.txt`.
- Docker:
  - Copies `/etc/config.txt` → to the writable container layer.
  - Applies the change only in the container.
- Result: Image layer is unchanged, container sees the modified file.

---

## Benefits of Copy-on-Write
- **Storage efficiency**: No duplication unless needed.
- **Fast container startup**: Containers share image layers.
- **Immutability**: Image layers are never altered after creation.
- **Isolation**: Each container has its own writable layer.

---

## Drawbacks of Copy-on-Write
- **Performance overhead**: First write involves copying.
- **Complexity**: File operations are slightly slower than native filesystem writes.
- **Driver dependency**: Behavior differs across overlay2, aufs, btrfs, zfs.

---

## Storage Drivers Using CoW
- **overlay2**: Most common; efficient CoW at the file level.
- **aufs**: Older; also uses CoW at the file level.
- **btrfs**: Native CoW filesystem with advanced features (snapshots, rollback).
- **zfs**: Native CoW filesystem with snapshots and clones.
- **vfs**: Does *not* support CoW (copies files fully → slow and storage-heavy).



## 4. Image Lifecycle Commands

### Pulling and Searching
```bash
# Pull an image from Docker Hub or private registry
docker pull IMAGE_NAME[:TAG]

# Search images on Docker Hub
docker search IMAGE_NAME
```

### Listing and Inspecting Images
```sh
# List all local images
docker images

# Inspect image details (layers, environment variables, entrypoint)
docker inspect IMAGE_NAME[:TAG]

# View history of image layers
docker history IMAGE_NAME[:TAG]

# Inspect image metadata
docker inspect IMAGE_NAME[:TAG]

# Inspect layers of an image
docker history IMAGE_NAME[:TAG]

# Check environment variables and entrypoint
docker inspect -f '{{ .Config.Env }}' IMAGE_NAME[:TAG]
docker inspect -f '{{ .Config.Entrypoint }}' IMAGE_NAME[:TAG]

```

### Tagging and Pushing images
```sh
# Tag a local image for a registry
docker tag IMAGE_NAME[:TAG] REGISTRY_URL/IMAGE_NAME[:TAG]

# Push image to registry
docker push REGISTRY_URL/IMAGE_NAME[:TAG]
```
### Removing Images
```sh
# Remove an image locally
docker rmi IMAGE_NAME[:TAG]

# Force remove image if in use
docker rmi -f IMAGE_NAME[:TAG]
```

### Converting Container Changes to Images
```sh
# Commit changes from a running container to a new image
docker commit CONTAINER_NAME NEW_IMAGE_NAME[:TAG]
```

### Working with Private Registries
```sh
# Tag an image for private registry
docker tag my-app:v1 myregistry.local:5000/my-app:v1

# Push image to private registry
docker push myregistry.local:5000/my-app:v1

# Pull image from private registry
docker pull myregistry.local:5000/my-app:v1
```

### Save and Load Docker Images as TAR Files
```sh
# Save nginx latest image
docker save -o nginx_latest.tar nginx:latest

# Save Ubuntu 22.04 image
docker save -o ubuntu_22.04.tar ubuntu:22.04
```
### Load Docker Image from a TAR File
```sh
# Load nginx image from tar
docker load -i nginx_latest.tar

# Load Ubuntu image from tar
docker load -i ubuntu_22.04.tar
```

## Task 
- Install nexus as a docker image
- Login to nexus 
- Push your images to nexus 




### Solution

### 🐳 Task: Install Nexus as a Docker Image and Push Custom Images

---

### 1️⃣ Run Nexus in Docker
```bash
# Create a persistent volume for Nexus data
mkdir -p ~/nexus-data && chmod 777 ~/nexus-data

# Run Sonatype Nexus 3 as a container
docker run -d --name nexus \
  -p 8081:8081 -p 5000:5000 \
  -v ~/nexus-data:/nexus-data \
  sonatype/nexus3
```
- Nexus UI → http://localhost:8081
- Default admin credentials → admin / password from ~/nexus-data/admin.password

### Create a Private Docker Registry in Nexus
```sh
Login to Nexus UI (http://localhost:8081)

Go to Administration → Repositories → Create repository

Select docker (hosted)

Name it (e.g., docker-hosted)

HTTP Port → 5000

Save
```
- Now Nexus is acting as a Docker registry at http://localhost:5000.

### Configure Docker to Trust Nexus Registry
```sh
Edit /etc/docker/daemon.json:

{
  "insecure-registries": ["localhost:5000"]
}
```

### Restart Docker:
```sh
sudo systemctl restart docker
```

###  Login to Nexus Docker Registry
```sh
docker login localhost:5000
# Enter Nexus admin credentials (or user credentials you created)
```

### Tag and Push an Image to Nexus
```sh
# Pull a base image
docker pull alpine:latest

# Tag the image for Nexus
docker tag alpine:latest localhost:5000/my-alpine:1.0

# Push to Nexus registry
docker push localhost:5000/my-alpine:1.0
```

### Verify Image in Nexus
```txt
Go to Nexus UI → Browse → Repositories → docker-hosted
```
- You should see my-alpine:1.0 pushed successfully.


### Docker Registries 
- Docker Hub (default public registry)
- Harbor (open-source private registry)
- JFrog Artifactory (enterprise registry)
- GitLab Container Registry (integrated with GitLab)
- AWS Elastic Container Registry (ECR)
- Azure Container Registry (ACR)
- Google Artifact Registry / Container Registry (GCR)