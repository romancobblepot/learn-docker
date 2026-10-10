# Checkpoint 3.3: Docker Compose Multi-Container Apps

## Purpose

Verify you can use Docker Compose to manage multi-container applications.

## Verification Questions

<details>
<summary>Question 1: What does Docker Compose do?</summary>

**Answer:** Manages multi-container applications with a single YAML configuration file. Handles starting, stopping, networking, and volumes for all services together.
</details>

<details>
<summary>Question 2: How do containers communicate in Compose?</summary>

**Answer:** Compose creates a network and adds all services. Containers communicate using service names as hostnames (automatic DNS resolution).
</details>

<details>
<summary>Question 3: What does `depends_on` do?</summary>

**Answer:** Specifies startup order. Service B with `depends_on: [A]` waits for A to start first.
</details>

## Testing Checklist

**Create a test compose file:**

```yaml
# compose.yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  
  app:
    image: alpine
    command: sleep 3600
    depends_on:
      - web
```

**Test it:**

```bash
# Start services
docker compose up -d

# Verify running
docker compose ps

# Test communication
docker compose exec app ping web
# Should work!

# View logs
docker compose logs

# Stop everything
docker compose down
```

**Success criteria:**
- [ ] Both services start
- [ ] `docker compose ps` shows running services
- [ ] Services can communicate (ping works)
- [ ] Can access web service at localhost:8080
- [ ] `docker compose down` cleans up everything

## Common Issues

**Issue: "no configuration file provided"**
- **Solution:** Must have `compose.yaml` in current directory

**Issue: "service 'x' failed to build"**
- **Check:** Dockerfile exists if using `build:`
- **Check:** Image name correct if using `image:`

**Issue: Services can't communicate**
- **Check:** Service names correct (use in environment variables)
- **Check:** All services in same compose file (same network)

## Congratulations!

You've completed the entire Docker tutorial! 🎉

You now know how to:
- ✅ Run containers
- ✅ Build images
- ✅ Publish ports
- ✅ Persist data
- ✅ Orchestrate multi-container apps

**You're ready to use Docker in real projects!**
