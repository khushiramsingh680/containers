+++
title = "Projects"
weight = 12
+++

## Projects
- [Use Nexus as a Docker Registry](#task-01-nexus-as-a-container-registry)

## Task 01: Nexus as a Container Registry
**Objectives:** Install Nexus as a Docker image, login, and push images.

### Solution
---
- Step 1: Run Nexus in Docker  
```sh
mkdir -p ~/nexus-data && chmod 777 ~/nexus-data  
docker run -d --name nexus -p 8081:8081 -p 5000:5000 -v ~/nexus-data:/nexus-data sonatype/nexus3  
```
- Access Nexus UI: http://localhost:8081  
- Default admin credentials: admin / password in ~/nexus-data/admin.password  

- Step 2: Create a Private Docker Registry in Nexus  
```sh
1. Login to Nexus UI (http://localhost:8081)  
2. Go to Administration → Repositories → Create repository  
3. Select docker (hosted)  
4. Name it docker-hosted  
5. Set HTTP Port to 5000  
6. Save  

```
- Nexus now acts as a Docker registry at http://localhost:5000  

- Step 3: Configure Docker to Trust Nexus Registry  
```sh
Edit /etc/docker/daemon.json:  
{ "insecure-registries": ["localhost:5000"] }  
```
### Restart Docker: sudo systemctl restart docker  

- Step 4: Login to Nexus Docker Registry  
docker login localhost:5000 (enter Nexus admin credentials)  

- Step 5: Tag and Push an Image to Nexus  
```txt
docker pull alpine:latest  
docker tag alpine:latest localhost:5000/my-alpine:1.0  
docker push localhost:5000/my-alpine:1.0  
```
- Step 6: Verify Image in Nexus  
```sh
Go to Nexus UI → Browse → Repositories → docker-hosted  
You should see my-alpine:1.0 successfully pushed  
```
- Popular Docker Registries:  

## Docker Hub (public)
-  Harbor (open-source private),
-  JFrog Artifactory (enterprise),
-  GitLab Container Registry (integrated),
-  AWS ECR 
-  Azure ACR
-  Google Artifact Registry/GCR
