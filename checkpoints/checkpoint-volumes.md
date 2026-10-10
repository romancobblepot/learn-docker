# Checkpoint 3.2: Data Persistence with Volumes

## Purpose

Verify you understand volumes and can persist data across container restarts/recreations.

## Verification Questions

<details>
<summary>Question 1: What happens to container data when you remove a container?</summary>

**Answer:** Data is deleted UNLESS it's stored in a volume. Volumes persist data outside the container filesystem.
</details>

<details>
<summary>Question 2: Why use volumes instead of storing data in containers?</summary>

**Answer:**
- Data persists across container restarts/recreations
- Share data between containers
- Easier backups
- Better performance (especially Mac/Windows)
</details>

## Testing Checklist

**Test data persistence:**

```bash
# Create volume and container
docker volume create test-data
docker run -d --name test1 -v test-data:/data alpine sleep 3600

# Write data
docker exec test1 sh -c "echo 'Hello!' > /data/message.txt"

# Verify data exists
docker exec test1 cat /data/message.txt

# Remove container
docker rm -f test1

# New container, same volume
docker run --rm -v test-data:/data alpine cat /data/message.txt
# Should show: Hello!

# Cleanup
docker volume rm test-data
```

**Success criteria:**
- [ ] Data written to volume
- [ ] Data survives container removal
- [ ] New container can access the data
- [ ] Understand volume lifecycle

## Common Issues

**Issue: "volume not found"**
- **Solution:** Create it first: `docker volume create name`

**Issue: "volume is in use"**
- **Solution:** Stop containers using it first

**Issue: Data not persisting**
- **Check:** Volume mounted to correct path?
- **Check:** App writing to mounted path?

## Next Steps

Continue to Part 3: Multi-Container Apps with Docker Compose!
