# Checkpoint 1.2: Understanding Images and Container Lifecycle

## Purpose

This checkpoint verifies that you understand Docker images, can pull and inspect images from Docker Hub, and can manage the complete container lifecycle. Images are the foundation of containers - understanding them deeply is crucial!

## Verification Questions

Test your conceptual understanding:

<details>
<summary>Question 1: What's the difference between an image and a container?</summary>

**Answer:**
- **Image**: A static, immutable template (like a blueprint or class definition)
  - Contains filesystem layers and metadata
  - Can't be modified (only create new images)
  - One image can create many containers

- **Container**: A running instance of an image (like an object instance)
  - Active process with its own writable layer
  - Has state (running, stopped, etc.)
  - Can be started, stopped, removed independently

**Analogy:**
- Image = Recipe (instructions, ingredients list)
- Container = The actual meal you cook from that recipe

**Why this matters:** Understanding this distinction helps you manage Docker resources. Removing a container doesn't remove the image. You can recreate containers from images anytime.
</details>

<details>
<summary>Question 2: Why are images composed of layers?</summary>

**Answer:** Layers provide three key benefits:

1. **Caching**: When building images, unchanged layers are reused → faster builds
2. **Sharing**: Multiple images can share the same base layers → saves disk space
3. **Efficiency**: When pulling/pushing images, only new/changed layers transfer → faster network operations

**Example:**
- Image A: Ubuntu base (100MB) + Python (50MB) + App 1 (10MB) = 160MB
- Image B: Ubuntu base (100MB) + Python (50MB) + App 2 (15MB) = 165MB
- **Total disk space**: Only 175MB (not 325MB) because Ubuntu and Python layers are shared!

**Why this matters:** Layer optimization is key to fast builds and small images (covered in Lesson 2).
</details>

<details>
<summary>Question 3: What does `docker pull nginx` actually download?</summary>

**Answer:**
It downloads the **nginx image** from Docker Hub to your local machine. Specifically:
- All the layers that compose the nginx image
- Metadata about the image (tags, labels, configuration)
- Default: pulls the `latest` tag (if no tag specified)

**Full name**: `docker pull nginx` is shorthand for:
```
docker.io/library/nginx:latest
```
- `docker.io` - Registry (Docker Hub)
- `library` - Namespace (official images)
- `nginx` - Repository (image name)
- `latest` - Tag (version)

**Why this matters:** Understanding image naming helps you pull specific versions (e.g., `nginx:1.25`) and use private registries.
</details>

<details>
<summary>Question 4: What happens to data inside a container when you remove it?</summary>

**Answer:**
**Data is lost** when you remove a container with `docker rm`.

**Important distinctions:**
- `docker stop` → Data preserved (container still exists, just stopped)
- `docker rm` → Data lost (container is deleted)
- Image layers remain untouched (can create new containers)

**Why this matters:** This is why we need **volumes** (Lesson 3) to persist data. Without volumes:
- Database data lost when container is removed
- Application logs lost when container is removed
- User uploads lost when container is removed

**Pro tip:** Use named volumes (`-v`) or bind mounts for data you want to keep!
</details>

<details>
<summary>Question 5: What's the difference between `docker run` and `docker exec`?</summary>

**Answer:**
- **`docker run`**: Creates a **new** container from an image and runs a command
  - Starts from image blueprint
  - Creates fresh container instance
  - Example: `docker run -it ubuntu bash`

- **`docker exec`**: Runs a command in an **existing running** container
  - Container must already be running
  - Doesn't create anything new
  - Example: `docker exec -it <container> bash`

**Analogy:**
- `docker run` = Building a new house from blueprints
- `docker exec` = Entering an existing house that's already built

**Why this matters:** Using `docker run` when you meant `docker exec` creates duplicate containers! Understanding this prevents resource waste.
</details>

## Implementation Check

Let's verify your understanding with hands-on commands.

**Challenge: Pull and inspect the nginx image**

Run these commands and share the output:

```bash
# 1. Pull nginx image
docker pull nginx

# 2. List your images
docker images

# 3. Inspect nginx layers
docker image history nginx

# 4. Get detailed nginx image info
docker inspect nginx
```

**Key things to check:**

From `docker pull nginx`:
- [ ] Shows "Pulling from library/nginx"
- [ ] Shows multiple layers being pulled (or "Already exists" if cached)
- [ ] Ends with "Status: Downloaded newer image..." or "Status: Image is up to date..."

From `docker images`:
- [ ] nginx image appears in the list
- [ ] Shows TAG (default: latest)
- [ ] Shows SIZE (typically 130-150MB)

From `docker image history nginx`:
- [ ] Shows multiple layers (10-15 rows)
- [ ] Shows commands that created each layer (COPY, RUN, ENV, CMD, etc.)
- [ ] Shows size of each layer

From `docker inspect nginx`:
- [ ] Returns JSON with image details
- [ ] Includes "Config" section with CMD, ENV, EXPOSE
- [ ] Includes "RootFS" section with layer information

## Hands-On Challenge

**Complete this challenge to demonstrate your understanding:**

1. **Pull the `alpine` image** (tiny Linux image)
   ```bash
   docker pull alpine
   ```

2. **Run an alpine container interactively:**
   ```bash
   docker run -it --name my-alpine alpine sh
   ```
   Inside the container, run:
   ```bash
   echo "Hello from Alpine!" > /tmp/test.txt
   cat /tmp/test.txt
   exit
   ```

3. **Verify container is stopped:**
   ```bash
   docker ps -a
   ```
   Look for `my-alpine` with STATUS "Exited"

4. **Restart the container and check the file still exists:**
   ```bash
   docker start my-alpine
   docker exec my-alpine cat /tmp/test.txt
   ```
   Should show "Hello from Alpine!"

5. **Remove the container:**
   ```bash
   docker stop my-alpine
   docker rm my-alpine
   ```

6. **Create a new container and verify the file is gone:**
   ```bash
   docker run -it --name my-alpine-2 alpine sh
   cat /tmp/test.txt   # Should show "No such file"
   exit
   ```

7. **Clean up:**
   ```bash
   docker rm my-alpine-2
   docker rmi alpine
   ```

**What this teaches:**
- Data persists when you stop/start a container
- Data is lost when you remove a container
- Each `docker run` creates a fresh container from the image

## Common Issues

### Issue 1: "manifest for nginx:latest not found"

**Symptom:** Error when pulling:
```
Error response from daemon: manifest for nginx:latest not found
```

**Cause:** Network issues, typo in image name, or Docker Hub connectivity problems.

**Solution:**
1. **Check image name spelling:**
   ```bash
   docker search nginx
   ```

2. **Try pulling with explicit tag:**
   ```bash
   docker pull nginx:stable
   ```

3. **Check Docker Hub directly:**
   - Visit [https://hub.docker.com/_/nginx](https://hub.docker.com/_/nginx)
   - Verify the image and tags exist

4. **Test Docker Hub connectivity:**
   ```bash
   docker pull hello-world
   ```

### Issue 2: "Error response from daemon: No such image"

**Symptom:** Error when running commands like `docker inspect nginx`:
```
Error: No such image: nginx
```

**Cause:** The image hasn't been pulled yet.

**Solution:**
```bash
# Pull the image first
docker pull nginx

# Then inspect it
docker inspect nginx
```

### Issue 3: "Cannot remove container, container is running"

**Symptom:** Error when trying to remove a container:
```
Error response from daemon: You cannot remove a running container. Stop the container before attempting removal or force remove
```

**Cause:** Container must be stopped before removal.

**Solution:**

**Option 1 - Stop then remove:**
```bash
docker stop <container-name>
docker rm <container-name>
```

**Option 2 - Force remove (stops and removes):**
```bash
docker rm -f <container-name>
```

### Issue 4: "image is being used by running container"

**Symptom:** Error when trying to remove an image:
```
Error response from daemon: conflict: unable to remove repository reference "nginx" (must force) - container abc123 is using its referenced image
```

**Cause:** Can't remove an image if containers (even stopped ones) exist from that image.

**Solution:**
1. **List containers using that image:**
   ```bash
   docker ps -a --filter ancestor=nginx
   ```

2. **Remove those containers:**
   ```bash
   docker rm <container-id>
   ```

3. **Then remove the image:**
   ```bash
   docker rmi nginx
   ```

**Force remove (dangerous, removes image and stops/removes containers):**
```bash
docker rmi -f nginx
```

### Issue 5: "docker exec" command not working

**Symptom:** Error like:
```
Error: No such container: <container-name>
```

**Cause:** Container doesn't exist, has wrong name, or is not running.

**Solution:**
1. **Check if container exists:**
   ```bash
   docker ps -a
   ```

2. **If container is stopped, start it first:**
   ```bash
   docker start <container-name>
   docker exec <container-name> <command>
   ```

3. **Use container ID instead of name:**
   ```bash
   docker ps
   docker exec <container-id> <command>
   ```

## Testing Checklist

Verify each of these works:

**Image Operations:**
- [ ] `docker pull nginx` downloads nginx image
- [ ] `docker images` shows nginx in list
- [ ] `docker image history nginx` shows layers
- [ ] `docker inspect nginx` returns JSON details

**Container Lifecycle:**
- [ ] `docker run -d --name test-nginx nginx` creates running container
- [ ] `docker ps` shows test-nginx running
- [ ] `docker logs test-nginx` shows nginx logs
- [ ] `docker exec test-nginx ls /etc/nginx` lists nginx config files
- [ ] `docker stop test-nginx` stops container
- [ ] `docker ps -a` shows test-nginx with "Exited" status
- [ ] `docker start test-nginx` starts container again
- [ ] `docker stop test-nginx && docker rm test-nginx` removes container
- [ ] `docker ps -a` no longer shows test-nginx

**Cleanup:**
- [ ] `docker rmi nginx` removes nginx image
- [ ] `docker images` no longer shows nginx

## Key Commands Summary

**Image commands:**
```bash
docker pull <image>           # Download image from registry
docker images                 # List local images
docker image ls               # Same as above
docker image history <image>  # Show image layers
docker inspect <image>        # Detailed image info (JSON)
docker rmi <image>            # Remove image
docker image prune            # Remove unused images
```

**Container lifecycle commands:**
```bash
docker run <image>            # Create and start container
docker ps                     # List running containers
docker ps -a                  # List all containers
docker start <container>      # Start stopped container
docker stop <container>       # Stop running container
docker restart <container>    # Restart container
docker rm <container>         # Remove container
docker logs <container>       # View container logs
docker exec <container> <cmd> # Execute command in running container
docker inspect <container>    # Detailed container info (JSON)
```

## Next Steps

Excellent work! You've successfully:
- ✅ Understood the difference between images and containers
- ✅ Pulled images from Docker Hub
- ✅ Inspected image layers
- ✅ Managed the complete container lifecycle (create, start, stop, remove)
- ✅ Used `docker exec` to run commands in running containers

**You're now ready for Lesson 2**, where you'll learn to **build your own custom images** using Dockerfiles!

In Lesson 2, you'll:
- Write a Dockerfile from scratch
- Build a custom image for a Node.js todo app
- Optimize builds with layer caching
- Create production-ready images with multi-stage builds

Continue when ready!
