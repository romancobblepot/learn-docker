# Checkpoint 2.1: Your First Dockerfile

## Purpose

This checkpoint verifies that you've successfully created a Dockerfile, built your first custom image, and can run your application from that image. This is a critical skill - building images is how you package your own applications!

## Verification Questions

Test your understanding:

<details>
<summary>Question 1: What's the difference between RUN and CMD?</summary>

**Answer:**
- **`RUN`**: Executes commands **during the build process**
  - Example: `RUN npm install` installs dependencies when building the image
  - Creates a new layer in the image
  - Can have multiple `RUN` instructions

- **`CMD`**: Specifies the **default command to run when container starts**
  - Example: `CMD ["node", "src/index.js"]` starts your app
  - Only one `CMD` instruction (last one wins)
  - Can be overridden when running the container

**Analogy:**
- `RUN` = Setup tasks when building a house (install plumbing, wiring)
- `CMD` = What happens when someone moves in (turn on the lights)

**Why this matters:** Mixing them up is a common mistake! Installing dependencies with `CMD` won't work because CMD runs when the container starts, not during the build.
</details>

<details>
<summary>Question 2: Why do we copy package.json before copying the rest of the code?</summary>

**Answer:** **Layer caching optimization!**

**Bad order (slow rebuilds):**
```dockerfile
COPY . .           # Copies EVERYTHING including package.json
RUN npm install    # Installs dependencies
```
Problem: Any code change forces npm install to run again.

**Good order (fast rebuilds):**
```dockerfile
COPY package.json .   # Copy only dependency list
RUN npm install       # Install dependencies
COPY . .             # Copy the rest of the code
```
Benefit: Code changes don't invalidate the npm install cache!

**Why this matters:** On a real project with lots of dependencies, `npm install` might take 2-5 minutes. With proper layer ordering, it only runs when dependencies actually change.
</details>

<details>
<summary>Question 3: What does the "." (dot) mean in "docker build -t my-app ."?</summary>

**Answer:** The dot `.` is the **build context** - the directory Docker uses to find files to copy.

**Breaking down the command:**
```bash
docker build -t my-app .
```
- `docker build` - Command to build an image
- `-t my-app` - Tag (name) the image "my-app"
- `.` - Use current directory as build context

**Why this matters:**
- Docker can only COPY files from the build context
- If your Dockerfile is in `/home/user/app`, you need to run `docker build` from that directory
- Or specify the context: `docker build -t my-app /home/user/app`
</details>

<details>
<summary>Question 4: What's the difference between an image and a Dockerfile?</summary>

**Answer:**
- **Dockerfile**: Text file with instructions for building an image (the recipe)
- **Image**: The actual built package containing your app (the result)

**Analogy:**
- Dockerfile = Recipe card
- Image = The actual cake baked from that recipe
- Container = Eating/serving the cake

**Process:**
1. Write Dockerfile (instructions)
2. Run `docker build` (follow the recipe)
3. Get an image (the finished product)
4. Run container from image (use the product)

**Why this matters:** You share Dockerfiles with your team (so they can build), OR you share built images (ready to run). Both are useful!
</details>

## Implementation Check

Let's verify your Dockerfile and build worked correctly.

**Run these commands:**

```bash
# 1. Show your Dockerfile
cat Dockerfile

# 2. Verify image was built
docker images | grep my-todo-app

# 3. Check if container is running
docker ps | grep my-todo

# 4. Test the app
curl http://localhost:3000
```

**Expected results:**

**From `cat Dockerfile`:**
- [ ] Starts with `FROM node:18-alpine` or similar
- [ ] Has `WORKDIR /app`
- [ ] Copies package.json before other files
- [ ] Has `RUN npm install`
- [ ] Has `COPY . .` to copy application code
- [ ] Has `CMD` to start the application

**From `docker images`:**
- [ ] Shows `my-todo-app` image
- [ ] Has a TAG (e.g., "latest")
- [ ] Shows SIZE (should be ~125MB for Node.js app)

**From `docker ps`:**
- [ ] Container named `my-todo` is running
- [ ] STATUS shows "Up"
- [ ] PORTS shows `0.0.0.0:3000->3000/tcp`

**From `curl localhost:3000`:**
- [ ] Returns HTML (the todo app page)
- [ ] No "connection refused" error

## Common Issues

### Issue 1: "no such file or directory" when copying files

**Symptom:**
```
ERROR [4/6] COPY package.json .
failed to compute cache key: "/package.json" not found
```

**Cause:** The build context doesn't contain the file you're trying to copy.

**Solution:**

**Step 1: Check you're in the right directory**
```bash
# You should be in the app directory containing package.json
pwd
ls -la package.json
```

**Step 2: Build from the correct location**
```bash
# If package.json is in getting-started-app/
cd getting-started-app
docker build -t my-todo-app .
```

**Step 3: Check the COPY path in Dockerfile**
```dockerfile
# Correct:
COPY package.json .

# Wrong (if file is in root of build context):
COPY src/package.json .
```

---

### Issue 2: "command not found: npm" in container

**Symptom:**
Container starts but app doesn't work. Logs show:
```
docker logs my-todo
/bin/sh: npm: not found
```

**Cause:** Base image doesn't include Node.js/npm.

**Solution:**

**Check your FROM instruction:**
```dockerfile
# Correct - includes Node.js and npm:
FROM node:18-alpine

# Wrong - plain alpine doesn't have Node.js:
FROM alpine
```

**Rebuild with correct base image:**
```bash
docker build -t my-todo-app .
docker run -d -p 3000:3000 --name my-todo my-todo-app
```

---

### Issue 3: "port is already allocated"

**Symptom:**
```
docker: Error response from daemon: driver failed programming external connectivity: Bind for 0.0.0.0:3000 failed: port is already allocated.
```

**Cause:** Another container or process is using port 3000.

**Solution:**

**Option 1: Use a different port**
```bash
# Stop and remove old container
docker stop my-todo
docker rm my-todo

# Run on different port (e.g., 3001)
docker run -d -p 3001:3000 --name my-todo my-todo-app

# Access at http://localhost:3001
```

**Option 2: Remove conflicting container**
```bash
# Find what's using port 3000
docker ps | grep 3000

# Stop it
docker stop <container-name>
docker rm <container-name>

# Now run yours
docker run -d -p 3000:3000 --name my-todo my-todo-app
```

---

### Issue 4: Build succeeds but app doesn't work

**Symptom:**
- `docker build` completes successfully
- Container starts and shows "Up" in `docker ps`
- But http://localhost:3000 doesn't work or shows errors

**Cause:** The app might have errors, or CMD might be wrong.

**Solution:**

**Step 1: Check container logs**
```bash
docker logs my-todo
```

Look for error messages. Common issues:
- Missing files
- Wrong command in CMD
- App crashing on startup

**Step 2: Enter the container and debug**
```bash
# Get a shell inside the running container
docker exec -it my-todo sh

# Check if files are there
ls -la
cat src/index.js

# Try running the app manually
node src/index.js
```

**Step 3: Common fixes**

**If files are missing:**
```dockerfile
# Make sure you copied them
COPY . .
```

**If wrong startup command:**
```dockerfile
# Check CMD matches your app structure
CMD ["node", "src/index.js"]  # Adjust path as needed
```

**Rebuild and retry:**
```bash
docker stop my-todo
docker rm my-todo
docker build -t my-todo-app .
docker run -d -p 3000:3000 --name my-todo my-todo-app
```

---

### Issue 5: "npm ERR! code ELIFECYCLE" during build

**Symptom:**
```
RUN npm install
npm ERR! code ELIFECYCLE
npm ERR! errno 1
```

**Cause:** npm install failed, usually due to:
- Invalid package.json
- Dependency conflicts
- Network issues during build

**Solution:**

**Step 1: Check package.json is valid**
```bash
# Validate JSON syntax
cat package.json
```

**Step 2: Try installing locally first**
```bash
# On your machine (outside Docker)
npm install
```
If this fails, fix package.json first.

**Step 3: Clear npm cache and rebuild**
```bash
docker build --no-cache -t my-todo-app .
```

---

## Testing Checklist

Verify everything works:

**Build verification:**
- [ ] Dockerfile exists in project directory
- [ ] `docker build -t my-todo-app .` completes without errors
- [ ] `docker images` shows my-todo-app

**Run verification:**
- [ ] `docker run -d -p 3000:3000 --name my-todo my-todo-app` starts container
- [ ] `docker ps` shows container running (STATUS: Up)
- [ ] `docker logs my-todo` shows app started (no errors)

**App verification:**
- [ ] http://localhost:3000 loads the todo app
- [ ] Can add a todo item
- [ ] Can check off a todo
- [ ] No errors in browser console

**Understanding verification:**
- [ ] Can explain what each line in Dockerfile does
- [ ] Understand RUN vs CMD
- [ ] Understand why layer order matters

## Next Steps

Excellent work! You've successfully:
- ✅ Written your first Dockerfile
- ✅ Built a custom Docker image
- ✅ Run an application from your image
- ✅ Tested that it works

**Now you're ready for Part 2** where we'll optimize the build process to make rebuilds much faster using layer caching!

Continue with the lesson!
