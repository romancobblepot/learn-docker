# Checkpoint 1.1: Your First Container

## Purpose

This checkpoint verifies that you've successfully run your first Docker container, can view running containers, and understand basic container operations. This is a critical foundation for everything that follows!

## Verification Questions

Test your conceptual understanding (try to answer before revealing):

<details>
<summary>Question 1: Why did we use the `-d` flag in `docker run -d`?</summary>

**Answer:** The `-d` flag runs the container in "detached" mode, meaning it runs in the background without blocking your terminal. Without `-d`, the container would run in the foreground and you'd see its output directly (and couldn't use your terminal for other commands until it stops).

**Why this matters:** For long-running services (web servers, databases), you want them in the background so you can continue using your terminal.
</details>

<details>
<summary>Question 2: What does `0.0.0.0:8080->80/tcp` mean in the PORTS column?</summary>

**Answer:** This is the port mapping:
- `0.0.0.0:8080` - The container is accessible on **all network interfaces** on the host at port **8080**
- `->` - Maps to
- `80/tcp` - Port **80** inside the container (using TCP protocol)

**In simple terms:** Requests to `localhost:8080` on your machine are forwarded to port `80` inside the container.

**Why this matters:** Containers have their own network namespace. To access services inside, you must publish (map) ports.
</details>

<details>
<summary>Question 3: What happens if you run the same `docker run` command again?</summary>

**Answer:** Docker will create a **second** container from the same image. Each `docker run` creates a new container instance. You'd have two separate containers running (with different container IDs and likely different names).

**Why this matters:** Understanding that `docker run` creates new containers (vs `docker start` which starts existing ones) is crucial for container lifecycle management.
</details>

<details>
<summary>Question 4: What's the difference between a container and an image?</summary>

**Answer:**
- **Image**: A static template/blueprint containing the filesystem and configuration (immutable)
- **Container**: A running instance of an image (active process with its own writable layer)

**Analogy:**
- Image = Class definition (in programming)
- Container = Object instance of that class

**Why this matters:** You can create many containers from one image. Deleting a container doesn't delete the image.
</details>

## Implementation Check

Let's verify your container is working correctly.

**Run these commands and share the output:**

```bash
# 1. List running containers
docker ps

# 2. Check if the container is accessible
curl http://localhost:8080
```

**Key things to check:**

From `docker ps` output:
- [ ] Container appears in the list
- [ ] STATUS shows "Up X seconds/minutes" (not "Exited")
- [ ] PORTS shows `0.0.0.0:8080->80/tcp` or similar
- [ ] NAMES shows a container name

From `curl localhost:8080` (or browser):
- [ ] You see HTML content or the welcome page loads
- [ ] No "connection refused" errors

## Common Issues

### Issue 1: "docker: command not found"

**Symptom:** Command fails with `docker: command not found` or `command not found: docker`

**Cause:** Docker Desktop is not installed, not running, or Docker is not in your system PATH.

**Solution:**
1. **Check if Docker Desktop is installed:**
   - macOS: Look for Docker icon in menu bar
   - Windows: Look for Docker icon in system tray
   - Linux: Docker Engine should be installed

2. **If installed but not running:**
   - Open Docker Desktop application
   - Wait for status to show "running" (green icon)
   - Retry the command

3. **If not installed:**
   - Download from [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
   - Install and start Docker Desktop
   - Verify with: `docker --version`

### Issue 2: "Cannot connect to the Docker daemon"

**Symptom:** Error message like:
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock.
Is the docker daemon running?
```

**Cause:** Docker Desktop is installed but not running.

**Solution:**
1. **Start Docker Desktop:**
   - macOS: Open Docker Desktop from Applications
   - Windows: Open Docker Desktop from Start menu
   - Linux: Run `sudo systemctl start docker`

2. **Wait for Docker to fully start** (can take 10-30 seconds)
   - Look for green "running" indicator
   - Icon should be solid (not animated)

3. **Verify Docker is running:**
   ```bash
   docker info
   ```
   If this works, Docker is running.

4. **Retry your command**

### Issue 3: "port is already allocated" or "address already in use"

**Symptom:** Error like:
```
docker: Error response from daemon: driver failed programming external connectivity:
Bind for 0.0.0.0:8080 failed: port is already allocated.
```

**Cause:** Another container or application is already using port 8080 on your host.

**Solution:**

**Option 1 - Use a different port:**
```bash
# Stop and remove the existing container if it exists
docker stop welcome-to-docker
docker rm welcome-to-docker

# Run with a different port (e.g., 8081)
docker run -d -p 8081:80 --name welcome-to-docker docker/welcome-to-docker
```
Then access at [http://localhost:8081](http://localhost:8081)

**Option 2 - Find and stop the conflicting process:**
```bash
# macOS/Linux: Find what's using port 8080
lsof -i :8080

# Windows: Find what's using port 8080
netstat -ano | findstr :8080
```
Then stop that process or container.

### Issue 4: "Unable to find image locally" but pull fails

**Symptom:** Error like:
```
Unable to find image 'docker/welcome-to-docker:latest' locally
Error response from daemon: Get "https://registry-1.docker.io/v2/": dial tcp: lookup registry-1.docker.io: no such host
```

**Cause:** Network connectivity issues or Docker Hub is unreachable.

**Solution:**
1. **Check internet connection:**
   ```bash
   ping google.com
   ```

2. **Check Docker Hub status:**
   - Visit [https://www.dockerstatus.com/](https://www.dockerstatus.com/)

3. **Try pulling a smaller image to test:**
   ```bash
   docker pull hello-world
   ```

4. **Check firewall/proxy settings:**
   - Corporate networks may block Docker Hub
   - You may need to configure Docker to use a proxy

5. **Restart Docker Desktop:**
   - Sometimes fixes transient network issues

### Issue 5: Container exits immediately

**Symptom:** `docker ps` shows no running containers, but `docker ps -a` shows container with STATUS "Exited (0)" or "Exited (1)"

**Cause:** For the `docker/welcome-to-docker` image, this shouldn't happen (it runs a web server). But generally, containers exit when their main process stops.

**Solution:**
1. **Check the logs to see why it exited:**
   ```bash
   docker logs <container-name>
   ```

2. **For this specific image, try:**
   ```bash
   # Remove the failed container
   docker rm <container-name>

   # Run again
   docker run -d -p 8080:80 docker/welcome-to-docker
   ```

3. **If still failing:**
   - Check Docker Desktop logs for errors
   - Restart Docker Desktop

## Testing Checklist

Verify each of these works:

- [ ] `docker --version` shows Docker version
- [ ] `docker run -d -p 8080:80 docker/welcome-to-docker` completes without errors
- [ ] `docker ps` shows the container with STATUS "Up"
- [ ] [http://localhost:8080](http://localhost:8080) loads the welcome page in browser
- [ ] `docker logs <container-name>` shows web server logs
- [ ] `docker stop <container-name>` stops the container successfully
- [ ] `docker ps` no longer shows the container (it's stopped)
- [ ] `docker ps -a` shows the container with STATUS "Exited"

## Next Steps

Excellent work! You've successfully:
- ✅ Run your first Docker container
- ✅ Viewed running containers with `docker ps`
- ✅ Accessed a containerized web application
- ✅ Viewed container logs
- ✅ Stopped a container

Now you're ready to move to **Part 2: Understanding Images**, where you'll learn what container images are and how they're composed of layers.

Continue with the lesson!
