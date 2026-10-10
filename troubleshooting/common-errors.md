# Common Docker Errors and Solutions

This guide catalogs common errors you might encounter while learning Docker, organized by category. Each error includes the symptom, cause, and step-by-step solution.

---

## Container Errors

### Error: "docker: command not found"

**Symptom:**
```
bash: docker: command not found
```
or
```
zsh: command not found: docker
```

**Cause:**
- Docker Desktop is not installed
- Docker Desktop is not running
- Docker is not in your system PATH

**Solution:**

**Step 1: Check if Docker Desktop is installed**
- **macOS**: Look for Docker icon in menu bar (top-right)
- **Windows**: Look for Docker icon in system tray (bottom-right)
- **Linux**: Docker Engine should be installed

**Step 2: If not installed**
1. Download Docker Desktop: [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Install following the instructions for your OS
3. Start Docker Desktop

**Step 3: If installed but command not found**
1. Open Docker Desktop application
2. Wait for it to fully start (green icon)
3. Close and reopen your terminal
4. Verify: `docker --version`

---

### Error: "Cannot connect to the Docker daemon"

**Symptom:**
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock.
Is the docker daemon running?
```

**Cause:**
Docker Desktop is installed but not running, or the Docker daemon hasn't started.

**Solution:**

**macOS/Windows:**
1. Open Docker Desktop application
2. Wait 10-30 seconds for full startup
3. Look for green "running" indicator in icon
4. Verify with: `docker info`

**Linux:**
```bash
# Start Docker daemon
sudo systemctl start docker

# Enable Docker to start on boot
sudo systemctl enable docker

# Check status
sudo systemctl status docker
```

**Still not working?**
- Restart Docker Desktop completely
- Restart your computer
- Check Docker Desktop logs for errors

---

### Error: "port is already allocated"

**Symptom:**
```
docker: Error response from daemon: driver failed programming external connectivity:
Bind for 0.0.0.0:8080 failed: port is already allocated.
```

**Cause:**
Another container or application is already using port 8080 on your host.

**Solution:**

**Option 1 - Use a different host port:**
```bash
# Stop and remove existing container if it exists
docker stop <container-name>
docker rm <container-name>

# Run with different port (e.g., 8081)
docker run -d -p 8081:80 <image-name>
```
Access at http://localhost:8081

**Option 2 - Find and stop the conflicting process:**

**macOS/Linux:**
```bash
# Find what's using port 8080
lsof -i :8080

# Stop that process
kill <PID>
```

**Windows:**
```cmd
# Find what's using port 8080
netstat -ano | findstr :8080

# Stop that process
taskkill /PID <PID> /F
```

**Option 3 - Remove the conflicting container:**
```bash
# Find containers using that port
docker ps -a

# Remove the container
docker rm -f <container-name>
```

---

### Error: "container exits immediately"

**Symptom:**
Running `docker ps` shows no container, but `docker ps -a` shows:
```
CONTAINER ID   IMAGE    STATUS
abc123         nginx    Exited (0) 2 seconds ago
```

**Cause:**
The container's main process exited. Containers run as long as their main process (PID 1) is running. When it exits, the container stops.

**Common reasons:**
- Command completed successfully (e.g., `echo "hello"` then exits)
- Application crashed
- Misconfigured CMD/ENTRYPOINT

**Solution:**

**Step 1: Check the logs**
```bash
docker logs <container-name>
```
This shows why the container exited.

**Step 2: Common fixes**

**If the app crashed:**
- Fix the error shown in logs
- Rebuild image if needed

**If you used the wrong command:**
```bash
# Wrong: This exits immediately
docker run ubuntu echo "hello"

# Right: Keep container running with interactive shell
docker run -it ubuntu bash
```

**If running a background service:**
```bash
# Ensure CMD in Dockerfile doesn't exit
# For example, nginx should run in foreground:
CMD ["nginx", "-g", "daemon off;"]
```

---

### Error: "OCI runtime create failed"

**Symptom:**
```
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "app": executable file not found in $PATH
```

**Cause:**
The command specified in CMD or ENTRYPOINT doesn't exist in the container, or there's a configuration error.

**Solution:**

**Check the command:**
```bash
# If using custom image, verify the command exists
docker run -it <image> sh
which app
ls -la /path/to/app
```

**Common issues:**
1. **Typo in command**: Check Dockerfile CMD/ENTRYPOINT
2. **Wrong path**: Use absolute path (e.g., `/usr/bin/app` not `app`)
3. **Missing executable bit**: In Dockerfile, add:
   ```dockerfile
   RUN chmod +x /path/to/app
   ```
4. **Wrong architecture**: Image built for different CPU architecture

---

## Image Errors

### Error: "Unable to find image locally" followed by pull failure

**Symptom:**
```
Unable to find image 'myapp:latest' locally
docker: Error response from daemon: pull access denied for myapp, repository does not exist or may require 'docker login'
```

**Cause:**
- Image name typo
- Image doesn't exist on Docker Hub
- Private image requires authentication
- Network connectivity issues

**Solution:**

**Step 1: Verify image name**
```bash
# Search Docker Hub
docker search <image-name>
```

**Step 2: Check spelling and tag**
```bash
# Make sure tag exists
docker pull nginx:stable   # Good
docker pull nginx:stabel   # Typo - fails
```

**Step 3: If private image, login**
```bash
docker login
# Enter username and password
docker pull <your-username>/<image-name>
```

**Step 4: Check network**
```bash
# Test connectivity to Docker Hub
ping registry-1.docker.io
```

---

### Error: "manifest unknown" or "manifest not found"

**Symptom:**
```
Error response from daemon: manifest for nginx:invalidtag not found: manifest unknown: manifest unknown
```

**Cause:**
The specified tag doesn't exist for that image.

**Solution:**

**Step 1: Check available tags**
- Visit Docker Hub: https://hub.docker.com/_/nginx
- Look at "Tags" tab

**Step 2: Use valid tag or omit tag (uses `latest`)**
```bash
docker pull nginx          # Uses latest
docker pull nginx:1.25     # Specific version
docker pull nginx:stable   # Stable tag
```

---

### Error: "image with reference was found but does not match the specified platform"

**Symptom:**
```
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
```

**Cause:**
Image is built for a different CPU architecture (e.g., x86 vs ARM).

**Solution:**

**macOS Apple Silicon (M1/M2/M3):**
```bash
# Use --platform flag
docker pull --platform linux/amd64 <image-name>

# Or specify in docker run
docker run --platform linux/amd64 <image-name>
```

**Or use multi-platform images:**
Many official images support multiple architectures automatically (e.g., `nginx`, `ubuntu`).

---

### Error: "Cannot remove image, image is being used"

**Symptom:**
```
Error response from daemon: conflict: unable to remove repository reference "nginx" (must force) - container abc123 is using its referenced image
```

**Cause:**
Can't remove an image while containers (even stopped ones) exist from that image.

**Solution:**

**Option 1: Remove containers first**
```bash
# Find containers using this image
docker ps -a --filter ancestor=nginx

# Remove them
docker rm <container-id>

# Then remove image
docker rmi nginx
```

**Option 2: Force remove (stops and removes containers)**
```bash
docker rmi -f nginx
```
⚠️ **Warning**: This force-removes containers using this image!

---

## Networking Errors

### Error: "connection refused localhost:8080"

**Symptom:**
Browser shows "connection refused" when accessing http://localhost:8080, or:
```bash
curl localhost:8080
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

**Cause:**
- Container isn't running
- Port not published correctly
- Application inside container not listening
- Firewall blocking port

**Solution:**

**Step 1: Verify container is running**
```bash
docker ps
```
Check STATUS is "Up", not "Exited"

**Step 2: Verify port mapping**
```bash
docker ps
```
Look at PORTS column. Should show: `0.0.0.0:8080->80/tcp`

**If no port mapping shown:**
```bash
# Stop and remove container
docker stop <container>
docker rm <container>

# Run with correct port mapping
docker run -d -p 8080:80 <image>
```

**Step 3: Check application inside container**
```bash
# Check if app is listening
docker exec <container> netstat -tlnp

# Check app logs for errors
docker logs <container>
```

**Step 4: Check firewall (macOS/Linux)**
```bash
# macOS: Check System Preferences > Security & Privacy > Firewall
# Linux: Check iptables or ufw
sudo ufw status
```

---

### Error: "driver failed programming external connectivity"

**Symptom:**
```
Error response from daemon: driver failed programming external connectivity on endpoint <container>: Error starting userland proxy: listen tcp4 0.0.0.0:8080: bind: address already in use
```

**Cause:**
Same as "port is already allocated" - see above.

**Solution:**
See "port is already allocated" error above.

---

### Error: "network not found"

**Symptom:**
```
Error response from daemon: network my-network not found
```

**Cause:**
Referenced network doesn't exist (common when using `--network` flag or Docker Compose).

**Solution:**

**Create the network:**
```bash
docker network create my-network
```

**Or use existing network:**
```bash
# List networks
docker network ls

# Use an existing one
docker run --network bridge <image>
```

---

## General Errors

### Error: "permission denied" (Linux)

**Symptom:**
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Cause:**
On Linux, Docker requires root permissions or membership in the `docker` group.

**Solution:**

**Option 1: Add user to docker group (recommended)**
```bash
# Add current user to docker group
sudo usermod -aG docker $USER

# Log out and log back in (or restart)
# Then verify
docker ps
```

**Option 2: Use sudo (temporary)**
```bash
sudo docker ps
```

---

### Error: "no space left on device"

**Symptom:**
```
Error response from daemon: write /var/lib/docker/...: no space left on device
```

**Cause:**
Docker's storage is full (images, containers, volumes, build cache).

**Solution:**

**Clean up Docker resources:**
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove build cache
docker builder prune

# Or remove everything (careful!)
docker system prune -a --volumes
```

**Check disk usage:**
```bash
docker system df
```

---

## Getting More Help

If you encounter an error not listed here:

1. **Check Docker logs:**
   - **macOS**: Docker Desktop → Troubleshoot → View Logs
   - **Windows**: Docker Desktop → Troubleshoot → View Logs
   - **Linux**: `journalctl -u docker.service`

2. **Use `docker ai`:**
   ```bash
   docker ai "your error message or question"
   ```

3. **Check container logs:**
   ```bash
   docker logs <container-name>
   ```

4. **Inspect container:**
   ```bash
   docker inspect <container-name>
   ```

5. **Search official docs:**
   - [https://docs.docker.com/](https://docs.docker.com/)

6. **Search community:**
   - [https://forums.docker.com/](https://forums.docker.com/)
   - Stack Overflow: tag `docker`

---

**Back to Tutorial:** Return to the lesson and continue learning!
