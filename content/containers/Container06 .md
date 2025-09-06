# Docker Security

Docker security is about protecting containers, images, the Docker daemon, host system, and networks from vulnerabilities or misuse. It requires best practices at all layers.

---

## 1. Image Security
- Use official/trusted images only.
- Keep images small & minimal (Alpine, distroless).
- Scan images for vulnerabilities:
  docker scan myimage:latest
- Enable Docker Content Trust (DCT) to sign & verify images:
  export DOCKER_CONTENT_TRUST=1

---

## 2. Container Security
- Run as non-root user:
  USER 1001   (in Dockerfile)
- Avoid --privileged containers (full host access).
- Use read-only filesystem:
  docker run --read-only nginx
- Limit resources (cgroups):
  docker run -m 512m --cpus="1.0" nginx
- Drop capabilities:
  docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

---

## 3. Docker Daemon Security
- Prefer rootless mode for Docker.
- Protect /var/run/docker.sock (limit access).
- Use TLS for Docker API:
  dockerd --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem -H=0.0.0.0:2376

---

## 4. Network Security
- Use user-defined networks for isolation.
- Encrypt overlay networks for multi-host.
- Restrict exposed ports (-p flag).
- Apply firewall rules (iptables, ufw).

---

## 5. Host Security
- Keep OS & Docker updated.
- Enable AppArmor or SELinux.
- Restrict root access on host.
- For strong isolation → run containers inside VMs.

---

## 6. Security Tools
- Docker Bench for Security – Audits configs:
  docker run -it --net host --pid host --cap-add audit_control \
    -v /var/lib:/var/lib -v /var/run/docker.sock:/var/run/docker.sock \
    -v /usr/lib/systemd:/usr/lib/systemd -v /etc:/etc \
    docker/docker-bench-security
- Trivy / Clair / Anchore – Image scanning.
- Falco – Runtime monitoring.

---

## 7. Seccomp Profiles
- Restrict Linux syscalls with seccomp.
- Example:
  docker run --security-opt seccomp=/path/to/profile.json nginx

---

## 8. Rootless Docker
- Run daemon & containers without root → reduces attack surface.
- Install:
  curl -fsSL https://get.docker.com/rootless | sh

---

## 9. Best Practices (Quick Checklist ✅)
- Use minimal, signed images
- Run containers as non-root
- Drop privileges & capabilities
- Use TLS for API
- Patch host & Docker regularly
- Scan images before use
- Monitor containers at runtime

---
