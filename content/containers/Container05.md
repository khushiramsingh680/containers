+++
title = "Container restart policies"
weight = 1
+++


## Docker Restart Policies & Types of Containers

---

## 1. Docker Restart Policies

Restart policies define how Docker should handle container restarts when they exit or the Docker daemon restarts.

### Types of Restart Policies

1. **no** (default)
   - Container does **not** restart automatically.
   - Example:
     ```bash
     docker run --restart=no nginx
     ```

2. **on-failure**
   - Restarts container **only if it exits with a non-zero exit code** (error).
   - Optionally, you can limit retries (e.g., `on-failure:5`).
   - Example:
     ```bash
     docker run --restart=on-failure:3 myapp
     ```

3. **always**
   - Always restarts the container, regardless of exit status.
   - If stopped manually, it will restart after Docker daemon restarts.
   - Example:
     ```bash
     docker run --restart=always redis
     ```

4. **unless-stopped**
   - Similar to `always`, but if you manually stop the container, it will not restart after daemon reboot.
   - Example:
     ```bash
     docker run --restart=unless-stopped postgres
     ```

---

## 2. Types of Containers

Docker containers can be categorized based on their purpose and usage:

### 2.1 System / Service Containers
- Long-running services (like databases, web servers, monitoring tools).
- Typically use restart policies (`always`, `unless-stopped`).
- Examples: `mysql`, `nginx`, `prometheus`.

### 2.2 Application Containers
- Encapsulate application code and dependencies.
- Often run a single app or microservice.
- Examples: `mycompany/api:latest`, `node:20-app`.

### 2.3 Ephemeral / Debug Containers
- Short-lived containers for testing, debugging, or one-off tasks.
- Do not usually need restart policies.
- Examples:
  ```bash
  docker run --rm -it ubuntu bash
```



# Docker Storage

Docker storage allows containers to store and share data beyond their ephemeral writable layer.  
By default, container data is lost when the container is removed.  
To persist or share data, Docker provides different storage mechanisms.

---

## 1. Types of Docker Storage

### 1.1 Container Writable Layer
- Every container gets a **thin writable layer** on top of the image layers.
- Temporary: deleted when the container is removed.
- Good for short-lived, non-persistent data.
- **Limitation**: tied to container lifecycle.

### 1.2 Volumes (Recommended)
- Managed by Docker (`/var/lib/docker/volumes/` on host).
- Independent of container lifecycle.
- Can be shared between multiple containers.
- Can be backed by plugins (e.g., NFS, cloud storage).
- Example:
  ```bash
  docker volume create mydata
  docker run -d -v mydata:/app/data nginx
```

### 1.3 Bind Mounts

- Maps a directory or file from the host filesystem into the container.
- Flexible: can use any path on host.
- Good for local development and sharing configs/logs.
- Example:
```sh
docker run -d -v /host/path:/container/path nginx
```

# Docker Volumes

Docker volumes provide a way to persist data outside of containers’ writable layers.  
They are managed by Docker and are the preferred mechanism for data persistence.

---

## 1. Why Use Volumes?
- Data persistence beyond container lifecycle
- Sharing data between multiple containers
- Easier backup and restore
- Better performance compared to bind mounts
- Managed by Docker (stored in `/var/lib/docker/volumes/`)

---

## 2. Types of Docker Storage
1. **Volumes**
   - Managed by Docker
   - Stored in `/var/lib/docker/volumes/`
   - Best for persistence
2. **Bind Mounts**
   - Maps host path → container path
   - Stored anywhere on host filesystem
   - Good for local development
3. **tmpfs Mounts**
   - Stored only in memory
   - Non-persistent
   - Good for sensitive or temporary data

---

## 3. Common Docker Volume Commands
- Create a Volume  
  `docker volume create myvolume`

- List Volumes  
  `docker volume ls`

- Inspect a Volume  
  `docker volume inspect myvolume`

- Remove a Volume  
  `docker volume rm myvolume`

- Remove All Unused Volumes  
  `docker volume prune`

---

## 4. Using Volumes with Containers
- Mount a Volume  
  `docker run -d --name mycontainer -v myvolume:/data busybox`

- Bind Mount Example  
  `docker run -d --name mybind -v /host/data:/container/data busybox`

- tmpfs Mount Example  
  `docker run -d --name mytmp --mount type=tmpfs,destination=/app/tmp busybox`

---

## 5. Backup & Restore Volumes
- Backup a Volume  
  `docker run --rm -v myvolume:/data -v $(pwd):/backup busybox tar cvf /backup/backup.tar /data`

- Restore a Volume  
  `docker run --rm -v myvolume:/data -v $(pwd):/backup busybox tar xvf /backup/backup.tar -C /`

---

## 6. Volume Drivers
Docker supports different volume drivers, e.g.:
- **local** (default, stores data on local filesystem)
- **nfs** (store data on remote NFS server)
- **azurefile**, **aws ebs**, **gcp-pd** (cloud storage integrations)

Example:  
`docker volume create --driver local mylocalvol`  
`docker volume create --driver vieux/sshfs -o sshcmd=user@host:/path -o password=pass mysshvolume`

---

## 7. Best Practices
- Use **named volumes** instead of anonymous volumes
- Prefer **volumes** over bind mounts for portability
- Use **tmpfs** for sensitive, ephemeral data
- Regularly prune unused volumes
- For production, consider **driver-based storage** (NFS, cloud)

---
# Docker Networking

Docker networking allows containers to communicate with each other, the host system, and external networks.  
It is essential for microservices and multi-container applications.

---

## 1. Default Networks
When Docker is installed, it creates these default networks:
- **bridge** (default for standalone containers)
- **host** (container shares host network stack)
- **none** (isolated, no networking)

Check networks:
`docker network ls`

Inspect a network:
`docker network inspect bridge`

---

## 2. Types of Docker Networks

1. **Bridge Network** (default)
   - Containers get their own IP
   - NAT used for external communication
   - Best for single-host setups
   - Example:  
     `docker run -d --name web --network bridge nginx`

2. **Host Network**
   - Container shares host’s network namespace
   - No port mapping needed
   - Higher performance, but less isolation
   - Example:  
     `docker run -d --network host nginx`

3. **None Network**
   - No external connectivity
   - Only loopback interface available
   - Useful for security and testing
   - Example:  
     `docker run -d --network none nginx`

4. **User-defined Bridge Network**
   - Allows custom isolated networks
   - Containers can talk by name
   - Example:  
     `docker network create mynetwork`  
     `docker run -d --name db --network mynetwork mysql`  
     `docker run -d --name app --network mynetwork nginx`

5. **Overlay Network** (Swarm/Kubernetes)
   - Multi-host networking
   - Uses VXLAN tunneling
   - Suitable for distributed apps
   - Example:  
     `docker network create -d overlay myoverlay`

6. **Macvlan Network**
   - Assigns MAC address from host’s network
   - Container appears as a physical device
   - Used for legacy applications
   - Example:  
     `docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 mymacvlan`

---

## 3. Common Networking Commands
- List networks  
  `docker network ls`

- Inspect a network  
  `docker network inspect <network_name>`

- Create a network  
  `docker network create mynet`

- Remove a network  
  `docker network rm mynet`

- Connect a container to a network  
  `docker network connect mynet mycontainer`

- Disconnect a container from a network  
  `docker network disconnect mynet mycontainer`

---

## 4. Port Mapping
By default, containers in bridge networks are isolated from the host.  
We use `-p` or `--publish` to map host ports.

- Example:  
  `docker run -d -p 8080:80 nginx`  
  (Maps host port 8080 → container port 80)

---

## 5. DNS & Service Discovery
- Docker provides built-in DNS for container name resolution.
- Containers in the same user-defined network can resolve each other by name.
- Example:  
  `ping db` from `app` container in the same network.

---

## 6. Best Practices
- Use **user-defined bridge networks** for multi-container apps.
- For multi-host deployments, use **overlay networks**.
- Use **macvlan** when containers need to appear as separate physical devices.
- Avoid exposing unnecessary ports to the host.
- Use `docker-compose` or Kubernetes for complex networking.

---
