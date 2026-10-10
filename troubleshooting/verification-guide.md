# Docker Setup Verification Guide

This guide helps you verify Docker is installed and working correctly.

## Quick Health Check

Run these commands in order:

```bash
# 1. Check Docker is installed
docker --version

# 2. Check Docker is running
docker info

# 3. Run test container
docker run hello-world

# 4. Check you can pull images
docker pull alpine

# 5. Verify networking
docker run --rm alpine ping -c 1 google.com
```

**If all pass: Docker is working! ✅**

## Diagnostic Commands

**Check Docker status:**
```bash
docker info
```
Shows: Server version, storage driver, network info

**Check resource usage:**
```bash
docker system df
```
Shows: Images, containers, volumes, build cache sizes

**View running containers:**
```bash
docker ps
```

**View all containers (including stopped):**
```bash
docker ps -a
```

**Check Docker Desktop (Mac/Windows):**
- Look for Docker icon in menu bar/system tray
- Should show "Docker Desktop is running"
- Green icon = good
- Animated icon = starting
- Red icon = problem

## Common Setup Issues

### Issue 1: Docker Desktop not starting

**Symptoms:**
- Icon shows "Starting..." forever
- `docker` commands hang or fail

**Solutions:**

**macOS:**
```bash
# Restart Docker Desktop
killall Docker && open -a Docker

# Check logs
cat ~/Library/Containers/com.docker.docker/Data/log/vm/dockerd.log
```

**Windows:**
```powershell
# Restart Docker Desktop
Stop-Process -Name "Docker Desktop" -Force
Start-Process "C:\Program Files\Docker\Docker\Docker Desktop.exe"
```

**Linux:**
```bash
# Restart Docker service
sudo systemctl restart docker
sudo systemctl status docker
```

### Issue 2: Permission denied (Linux)

**Symptom:**
```
permission denied while trying to connect to Docker daemon
```

**Solution:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Log out and log back in
# Or run:
newgrp docker

# Verify
docker run hello-world
```

### Issue 3: No space left on device

**Symptom:**
```
no space left on device
```

**Solution:**
```bash
# Check disk usage
docker system df

# Clean up
docker system prune -a  # Remove all unused resources

# Or clean specific things:
docker container prune  # Remove stopped containers
docker image prune -a   # Remove unused images
docker volume prune     # Remove unused volumes
```

### Issue 4: Network connectivity issues

**Symptom:**
- Can't pull images
- Container can't reach internet

**Solution:**

**Check DNS:**
```bash
# Test from container
docker run --rm alpine ping -c 1 8.8.8.8
docker run --rm alpine ping -c 1 google.com
```

**Check Docker Desktop DNS settings:**
- Settings → Network
- Try changing DNS to 8.8.8.8

**Corporate network/proxy:**
- Configure proxy in Docker Desktop settings
- Settings → Resources → Proxies

## Platform-Specific Notes

### macOS
- Docker Desktop required (no native Docker Engine)
- File sharing can be slow (use volumes for better performance)
- Resource limits in Settings → Resources

### Windows
- WSL 2 backend recommended
- Hyper-V required (Windows Pro/Enterprise)
- Or use WSL 2 (Windows Home)

### Linux
- Docker Engine installed directly
- No Docker Desktop (unless you want it)
- Best performance (native)

## Verification Checklist

Run through this checklist to ensure everything works:

**Installation:**
- [ ] Docker installed
- [ ] `docker --version` shows version
- [ ] `docker info` shows server info

**Basic operations:**
- [ ] Can run `docker run hello-world`
- [ ] Can pull images: `docker pull alpine`
- [ ] Can list containers: `docker ps`
- [ ] Can remove containers: `docker rm`

**Networking:**
- [ ] Can publish ports: `docker run -p 8080:80 nginx`
- [ ] Can access localhost:8080
- [ ] Containers can reach internet

**Storage:**
- [ ] Can create volumes: `docker volume create test`
- [ ] Can mount volumes: `docker run -v test:/data alpine`
- [ ] Can remove volumes: `docker volume rm test`

**Advanced:**
- [ ] Can build images: `docker build -t test .`
- [ ] Can use Docker Compose: `docker compose version`

**If all checked: You're ready to use Docker!** ✅

## Getting More Help

**Check Docker status:**
- [Docker Status Page](https://www.dockerstatus.com/)

**Documentation:**
- [Official Docker Docs](https://docs.docker.com/)
- [Docker Desktop Troubleshooting](https://docs.docker.com/desktop/troubleshoot/)

**Community:**
- [Docker Forums](https://forums.docker.com/)
- [Stack Overflow: docker tag](https://stackoverflow.com/questions/tagged/docker)

**Use Docker AI:**
```bash
docker ai "your question here"
```

**Back to Tutorial:** Return to your lesson!
