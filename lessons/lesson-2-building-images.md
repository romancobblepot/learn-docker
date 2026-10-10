# Lesson 2: Building Images - Package Your Own Apps

## Learning Objectives

By the end of this lesson, you will:

- **Understand** what Dockerfiles are and why they exist
- **Write** a Dockerfile from scratch for a real application
- **Build** your own Docker images
- **Optimize** builds to run faster (layer caching)
- **Reduce** image sizes with multi-stage builds

**Time**: 45-60 minutes

## Why Build Your Own Images?

In Lesson 1, you ran pre-built images from Docker Hub. That's great for getting started, but in real projects you need to **package your own applications**.

### The Problem

**Traditional way of deploying apps:**
1. Write deployment docs: "Install Node.js 18, then npm install, then..."
2. Teammate tries to follow docs → Gets errors
3. Docs are outdated or incomplete
4. Different versions on different machines
5. "It works on my machine!" 🤷

### The Docker Way

**Instead, you create a Dockerfile:**
1. Write instructions **once** in a Dockerfile
2. Build an image from it
3. Share that image
4. Everyone runs the **exact same environment**
5. No more "works on my machine" problems! ✅

**Think of a Dockerfile as:**
- A recipe for creating an image
- Step-by-step instructions Docker follows
- Automated setup that works every time

---

## What is a Dockerfile?

**Simple answer:** A text file with instructions for building a Docker image.

**What it contains:**
- Base image to start from (e.g., "Node.js 18 installed")
- Commands to run (e.g., "install dependencies")
- Files to copy (e.g., "copy my app code")
- How to start the app (e.g., "run npm start")

**Example Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "start"]
```

**What this does:**
1. Start with Node.js 18 (pre-installed)
2. Set working directory to /app
3. Copy package.json
4. Install dependencies
5. Copy rest of the code
6. Define startup command

---

## What We're Building

In this lesson, we'll package a real Node.js todo application:

**By the end, you'll:**
- Build an image containing the todo app
- Run your custom image as a container
- Access the app at http://localhost:3000
- Optimize the build process (make it faster)
- Reduce image size with multi-stage builds

Let's start!

---

## Part 1: Your First Dockerfile

### Step 1.1: Get the Sample Application

Let me download Docker's official getting-started todo app:

[Claude runs via Bash tool: `git clone https://github.com/docker/getting-started-app.git && cd getting-started-app && ls -la`]

**Here's what we have:**

```
Cloning into 'getting-started-app'...
remote: Enumerating objects: 100, done.
...
total 48
drwxr-xr-x  8 user user  256 Feb  4 11:00 .
drwxr-xr-x 15 user user  480 Feb  4 11:00 ..
-rw-r--r--  1 user user  257 Feb  4 11:00 package.json
-rw-r--r--  1 user user 1234 Feb  4 11:00 package-lock.json
drwxr-xr-x  5 user user  160 Feb  4 11:00 src
-rw-r--r--  1 user user  123 Feb  4 11:00 spec
```

**What's in this app:**
- `src/` - Node.js application code
- `package.json` - Dependencies list (React, Express, etc.)
- No Dockerfile yet - we'll create it!

### Step 1.2: Understand Dockerfile Instructions

Before writing our Dockerfile, let's understand the common instructions:

#### `FROM` - Start with a base image
```dockerfile
FROM node:18-alpine
```
- **What it does:** Specifies the base image to build on top of
- **Why use it:** Don't start from scratch - build on existing images
- **Example:** `node:18-alpine` includes Node.js 18 pre-installed

#### `WORKDIR` - Set the working directory
```dockerfile
WORKDIR /app
```
- **What it does:** Sets the directory for subsequent commands
- **Why use it:** Organizes files, avoids conflicts
- **Like:** Running `cd /app` but permanent

#### `COPY` - Copy files into the image
```dockerfile
COPY package.json .
```
- **What it does:** Copies files from your machine into the image
- **Why use it:** Your code needs to be in the image!
- **Format:** `COPY <source> <destination>`

#### `RUN` - Execute commands during build
```dockerfile
RUN npm install
```
- **What it does:** Runs commands while building the image
- **Why use it:** Install dependencies, compile code, setup tasks
- **Runs at:** Build time (not when container starts)

#### `EXPOSE` - Document which port the app uses
```dockerfile
EXPOSE 3000
```
- **What it does:** Documents that the app listens on port 3000
- **Why use it:** Helps others understand your image
- **Note:** Doesn't actually publish the port (use `-p` for that)

#### `CMD` - Default command to run
```dockerfile
CMD ["node", "src/index.js"]
```
- **What it does:** Specifies what runs when container starts
- **Why use it:** Tells Docker how to start your app
- **Format:** Array format `["command", "arg1", "arg2"]`

### Step 1.3: Create Your Dockerfile

Let me create a Dockerfile (no extension!) in the `getting-started-app` directory:

[Claude runs via Bash tool: Create Dockerfile with Write tool or cat << EOF]

**Here's the Dockerfile we created:**

```dockerfile
# Start from Node.js 18 on Alpine Linux (small base image)
FROM node:18-alpine

# Set working directory to /app
WORKDIR /app

# Copy package files
COPY package.json package-lock.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application
COPY . .

# App runs on port 3000
EXPOSE 3000

# Start the application
CMD ["node", "src/index.js"]
```

**Why this order?** (We'll optimize this in Part 2!)

### Step 1.4: Build Your Image

Now let me build an image from this Dockerfile:

[Claude runs via Bash tool: `docker build -t my-todo-app .`]

**Here's what happened during the build:**

```
[+] Building 45.2s (10/10) FINISHED
 => [internal] load build definition from Dockerfile          0.1s
 => [internal] load .dockerignore                             0.0s
 => [internal] load metadata for docker.io/library/node:18    1.2s
 => [1/5] FROM docker.io/library/node:18-alpine@sha256:...   15.3s
 => [internal] load build context                             0.2s
 => [2/5] WORKDIR /app                                        0.1s
 => [3/5] COPY package.json package-lock.json ./              0.1s
 => [4/5] RUN npm install                                     25.8s
 => [5/5] COPY . .                                            0.2s
 => exporting to image                                        2.2s
 => => exporting layers                                       2.1s
 => => writing image sha256:abc123...                         0.0s
 => => naming to docker.io/library/my-todo-app:latest        0.0s
```

**Notice:**
- Each `[X/5]` line is a **layer** being built
- `FROM` pulled the Node.js 18 base image (15.3s)
- `RUN npm install` took the longest (25.8s) - downloading dependencies
- **Total build time: 45.2 seconds**

**Breaking down the command:**
- `docker build` - Command to build an image
- `-t my-todo-app` - Tag (name) the image "my-todo-app"
- `.` - Build context (current directory containing Dockerfile)

**First build takes time** because it downloads the base image and installs dependencies.

### Step 1.5: Run Your Custom Image

Now let me run a container from our newly built image:

[Claude runs via Bash tool: `docker run -d -p 3000:3000 --name my-todo my-todo-app`]

**Output:** `abc123def456...` (container ID)

Let me verify it's running:

[Claude runs via Bash tool: `docker ps`]

**Output:**
```
CONTAINER ID   IMAGE         COMMAND                  STATUS         PORTS
abc123def456   my-todo-app   "docker-entrypoint.s…"   Up 3 seconds   0.0.0.0:3000->3000/tcp
```

Perfect! The container is running on port 3000.

**Command breakdown:**
- `-d` - Run in background
- `-p 3000:3000` - Map port 3000 (host) to 3000 (container)
- `--name my-todo` - Name the container "my-todo"
- `my-todo-app` - The image we just built

### Step 1.6: Test Your App

**Open your browser to:** http://localhost:3000

You should see the todo app! 🎉

**Try it:**
- Add a todo item: "Learn Docker"
- Check it off
- It works - you're running YOUR custom image!

### Checkpoint 2.1: Test Your Understanding of Dockerfiles

**Let me verify you understand what we just built.**

[Use AskUserQuestion tool with these questions:]

**Question 1**: What's the difference between `RUN` and `CMD`?
- A) They do the same thing
- B) `RUN` executes during build; `CMD` executes when container starts ✅
- C) `RUN` is for copying files; `CMD` is for commands

**Question 2**: Why do we use `COPY` twice in the Dockerfile?
- A) One for files, one for folders
- B) Copy package.json first (for caching), then copy rest of code ✅
- C) Docker requires it

**Question 3**: What does `-t my-todo-app` do in the build command?
- A) Creates a tag file
- B) Tests the build
- C) Tags (names) the image "my-todo-app" ✅

**Question 4**: What took the longest time during the build we just ran?
- A) `FROM node:18-alpine`
- B) `RUN npm install` (25.8 seconds) ✅
- C) `COPY . .`

**Question 5**: When you change `src/index.js`, what gets rebuilt?
- A) Only that specific layer
- B) Nothing - Docker caches everything
- C) All layers after the changed layer ✅

**Review answers, explain misconceptions. When ready, continue to Part 2 where we'll optimize the build!**

**After user answers correctly, pause and ask:**

"Great work! Before we move to Part 2, do you have any questions about:
- Dockerfile instructions (FROM, RUN, CMD, COPY, etc.)
- How to build images
- The todo app we just built

Or are you ready to learn how to make builds MUCH faster with layer caching?"

**Wait for user response. Answer any questions. Only proceed when they confirm they're ready.**

---

## Part 2: Optimizing Builds with Layer Caching

### The Problem: Slow Rebuilds

Let me demonstrate what happens when we make a small code change and rebuild. I'll add a comment to `src/index.js`:

[Claude runs via Bash tool: `echo "// bug fix" >> src/index.js`]

Now let's rebuild:

[Claude runs via Bash tool: `docker build -t my-todo-app .`]

**Here's what happened:**

```
[+] Building 28.4s (10/10) FINISHED
 => [1/5] FROM docker.io/library/node:18-alpine             0.0s
 => [2/5] WORKDIR /app                                      0.0s
 => [3/5] COPY package.json package-lock.json ./            0.1s
 => [4/5] RUN npm install                                   25.2s  ← SLOW!
 => [5/5] COPY . .                                          0.2s
 => exporting to image                                      2.9s
```

**Problem:**
- Changed one line of code in `src/index.js`
- Docker rebuilt `RUN npm install` (25.2 seconds!) even though dependencies didn't change
- This happens because `COPY . .` comes before we use the code, invalidating the cache

**Real-world impact:**
- You change code 50 times per day
- Each rebuild wastes ~25 seconds
- **That's 20+ minutes wasted per day!**

### How Layer Caching Works

**Docker caches each layer:**
- If instruction hasn't changed → Reuse cached layer (fast! ⚡)
- If instruction changed → Rebuild that layer and ALL layers after it (slow ⏱️)

**The trick:** Order matters!

**Current Dockerfile (not optimal):**
```dockerfile
COPY . .              # Copies EVERYTHING (including package.json)
RUN npm install       # Installs dependencies
```

**Problem:** Any code change invalidates the cache, forcing `npm install` to run again.

**Optimized Dockerfile:**
```dockerfile
COPY package.json .   # Copy dependencies list first
RUN npm install       # Install dependencies (cached unless package.json changes)
COPY . .              # Copy code (changes here don't invalidate npm install cache)
```

**Result:** Changing code doesn't force reinstalling dependencies!

### Step 2.1: Optimize Your Dockerfile

Good news: Our Dockerfile already has this optimization! Let me demonstrate the difference. I'll make a small code change:

[Claude runs via Bash tool: `echo "// small optimization test" >> src/index.js`]

Now rebuild:

[Claude runs via Bash tool: `docker build -t my-todo-app .`]

**Look at the output:**

```
[+] Building 2.1s (10/10) FINISHED
 => [1/5] FROM docker.io/library/node:18-alpine             0.0s
 => [2/5] WORKDIR /app                                      0.0s
 => [3/5] COPY package.json package-lock.json ./            0.0s
 => CACHED [4/5] RUN npm install                            0.0s  ← CACHED!
 => [5/5] COPY . .                                          0.2s
 => exporting to image                                      1.9s
```

**See "CACHED"?** Docker reused the `RUN npm install` layer!

**Performance comparison:**
- **Before optimization:** 28.4 seconds (reinstalled dependencies)
- **After optimization:** 2.1 seconds (reused cache)
- **13x faster!** ⚡

### Step 2.2: Use .dockerignore

**Problem:** `COPY . .` copies EVERYTHING, including files we don't need:
- `node_modules/` (we install it with `npm install` anyway)
- `.git/` (not needed in the image)
- Development files, logs, etc.

**Solution:** Create a `.dockerignore` file (like `.gitignore`). Let me create one:

[Claude runs via Bash tool: Create .dockerignore file with Write tool or echo]

**Here's what we added:**
```
node_modules
.git
.gitignore
.dockerignore
npm-debug.log
README.md
.env
```

**Benefits:**
- Faster builds (less to copy)
- Smaller build context
- More cache hits
- Avoid copying sensitive files

### Step 2.3: Rebuild and Compare

Let me rebuild with the `.dockerignore` in place:

[Claude runs via Bash tool: `docker build -t my-todo-app .`]

**Build time:** ~2 seconds (slightly faster since we're copying less)

The build context is smaller, making the `COPY . .` step faster!

### Checkpoint 2.2: Test Your Understanding of Layer Caching

**Let me verify you understand layer caching optimization.**

[Use AskUserQuestion tool with these questions:]

**Question 1**: Why is layer order important in a Dockerfile?
- A) Docker reads from bottom to top
- B) Order doesn't matter
- C) Changing a layer invalidates all layers AFTER it ✅

**Question 2**: Why do we copy package.json BEFORE copying the rest of the code?
- A) package.json is smaller
- B) Docker requires it
- C) Dependencies change rarely; code changes often - maximize cache hits ✅

**Question 3**: What does `.dockerignore` do?
- A) Deletes files from the image
- B) Excludes files from being copied into the image ✅
- C) Ignores Docker errors

**Question 4**: When we saw "CACHED" in the build output, what did that mean?
- A) The build failed
- B) Docker saved the output to disk
- C) Docker reused an existing layer without rebuilding ✅

**Question 5**: After adding `.dockerignore`, what improved?
- A) Runtime performance
- B) Build speed (less to copy) ✅
- C) Image security only

**Review answers, explain misconceptions, then continue to Part 3!**

**After user answers correctly, pause and ask:**

"Excellent! Before we move to Part 3, do you have any questions about:
- Layer caching and why order matters
- The .dockerignore file
- How we went from 28 seconds to 2 seconds

Or are you ready to learn about multi-stage builds for even smaller images?"

**Wait for user response. Answer any questions. Only proceed when they confirm they're ready.**

---

## Part 3: Multi-Stage Builds (Smaller Images)

### The Problem: Bloated Images

Right now, our image includes:
- Node.js runtime ✅ (needed)
- npm package manager ✅ (needed)
- Build tools (if we had any) ❌ (not needed in production)
- Development dependencies ❌ (not needed)

**Result:** Bigger images = Slower deployments, more security risks

### The Solution: Multi-Stage Builds

**Idea:** Use one stage to build, another stage to run.

**Multi-stage Dockerfile:**
```dockerfile
# Stage 1: Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
# If you had a build step (TypeScript, webpack), run it here

# Stage 2: Production stage
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production  # Only production dependencies!
COPY --from=builder /app/src ./src  # Copy only what's needed
EXPOSE 3000
CMD ["node", "src/index.js"]
```

**What this does:**
1. **Build stage** - Install all dependencies, build the app
2. **Production stage** - Start fresh, copy only production dependencies and built code
3. **Result** - Smaller final image (no dev dependencies, no build tools)

### When to Use Multi-Stage Builds

**Use multi-stage builds when:**
- Your app has a build step (TypeScript → JavaScript, React → static files)
- You have dev dependencies vs production dependencies
- You want smaller production images

**For compiled languages (Go, Rust, Java):**
- Build stage: Use compiler image (large)
- Production stage: Use minimal runtime image (tiny)
- Result: Massive size savings! (e.g., 1GB → 20MB)

### Step 3.1: Compare Image Sizes

Let me check the size of our current image:

[Claude runs via Bash tool: `docker images my-todo-app`]

**Here's what we have:**

```
REPOSITORY     TAG       IMAGE ID       CREATED         SIZE
my-todo-app    latest    abc123def456   2 minutes ago   125MB
```

**Our image is 125MB.**

For this Node.js app, multi-stage builds would provide modest savings (maybe 110MB). But let me show you where multi-stage builds REALLY shine:

**Example with a Go app:**
- Without multi-stage: 800MB (includes Go compiler, tools, source code)
- With multi-stage: 15MB (just the compiled binary in alpine)
- **98% smaller!** 🎉

**When multi-stage builds make the biggest difference:**
- Compiled languages (Go, Rust, Java, C++)
- Build tools (webpack, TypeScript compiler)
- Apps with complex build pipelines

Our simple Node.js app doesn't have a build step, so the savings are smaller. But now you know the technique for when you need it!

### Checkpoint 2.3: Test Your Understanding of Multi-Stage Builds

**Let me verify you understand multi-stage builds.**

[Use AskUserQuestion tool with these questions:]

**Question 1**: What's the main benefit of multi-stage builds?
- A) Faster builds
- B) Smaller final images (only runtime dependencies, not build tools) ✅
- C) Better security only

**Question 2**: When are multi-stage builds MOST useful?
- A) Simple scripts
- B) Static websites
- C) Compiled languages or apps with build steps ✅

**Question 3**: In a multi-stage build, what happens to the first stage?
- A) It's kept as a separate image
- B) It's discarded - only the final stage becomes the image ✅
- C) It's merged with the final stage

**Question 4**: For a Go app, why can multi-stage builds reduce size from 800MB to 15MB?
- A) Compression
- B) Final image only needs the compiled binary, not the Go compiler ✅
- C) Docker magic

**Review answers, explain misconceptions. Lesson 2 complete!**

**After user answers correctly, pause and ask:**

"Fantastic! Lesson 2 is complete. Before we finish, do you have any questions about:
- Writing Dockerfiles
- Layer caching and optimization
- Multi-stage builds
- Anything we covered in this lesson

Or are you ready to move on to Lesson 3 where we'll run applications with networking, volumes, and Docker Compose?"

**Wait for user response. Answer any questions. Only proceed when they confirm they're ready.**

---

## Lesson Completion

Congratulations! You've completed Lesson 2! 🎉

### What You Built

In this lesson, you:
- ✅ Created your first Dockerfile from scratch
- ✅ Built a custom Docker image for a Node.js app
- ✅ Ran your app from your custom image
- ✅ Optimized builds with layer caching
- ✅ Used .dockerignore to speed up builds
- ✅ Understood multi-stage builds for smaller images

### What You Learned

**Concepts:**
- Dockerfiles are recipes for building images
- Each instruction creates a layer
- Layer order matters for caching
- `RUN` runs at build time, `CMD` runs at container start
- Multi-stage builds separate build from runtime

**Commands:**
```bash
docker build -t <name> .        # Build an image
docker build -t <name>:<tag> .  # Build with specific tag
docker images                    # List images
docker rmi <image>              # Remove an image
```

**Dockerfile instructions:**
```dockerfile
FROM       # Base image
WORKDIR    # Set working directory
COPY       # Copy files into image
RUN        # Execute commands during build
EXPOSE     # Document port (doesn't publish)
CMD        # Default command when container starts
```

### Key Takeaways

**Practical skills you now have:**
1. ✅ Package any application as a Docker image
2. ✅ Share images so teammates run the exact same environment
3. ✅ Optimize builds for speed (layer caching)
4. ✅ Reduce image sizes (multi-stage builds)
5. ✅ No more "it works on my machine" problems!

**Reflection:** Think about what you learned - you can now package ANY application as a Docker image!

### Next Steps

In **Lesson 3**, you'll learn to:
- Publish ports so others can access your apps
- Persist data with volumes (so todos don't disappear!)
- Run multi-container applications (app + database)
- Use Docker Compose to manage everything easily

**Ready for Lesson 3?** Let me know when you want to continue!

### 🤓 Want to Go Deeper? (Optional)

**Curious about image layers and how they work internally?**

Check out `architecture/image-layers.md` for a deep dive into:
- Union filesystems (OverlayFS)
- How layers are stored and shared
- Copy-on-write mechanics

**But you don't need this to proceed!** The practical knowledge you have is enough.
