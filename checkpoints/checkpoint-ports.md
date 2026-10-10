# Checkpoint 3.1: Port Publishing

## Purpose

Verify you understand port publishing and can access containerized applications from your host machine.

## Verification Questions

<details>
<summary>Question 1: What does `-p 3000:80` mean?</summary>

**Answer:** Forwards traffic from port 3000 on your host machine to port 80 inside the container. Access the container's service at http://localhost:3000.
</details>

<details>
<summary>Question 2: Why can't you access containers without publishing ports?</summary>

**Answer:** Containers have isolated network namespaces. They're in their own network bubble. Publishing ports creates a forwarding rule that breaks through this isolation.
</details>

## Testing Checklist

```bash
# Run nginx with published port
docker run -d -p 8080:80 --name test-nginx nginx

# Verify accessible
curl http://localhost:8080
# Should return HTML

# Clean up
docker stop test-nginx && docker rm test-nginx
```

**Success criteria:**
- [ ] Container starts successfully
- [ ] `docker ps` shows port mapping (0.0.0.0:8080->80/tcp)
- [ ] Can access via browser at localhost:8080
- [ ] `curl` returns HTML content

## Common Issues

**Issue: "port is already allocated"**
- **Solution:** Use different port: `-p 8081:80`

**Issue: "connection refused"**
- **Check:** Is container running? (`docker ps`)
- **Check:** Correct port mapping? (PORTS column)
- **Check:** App listening inside container? (`docker logs`)

## Next Steps

Continue to Part 2: Data Persistence with Volumes!
