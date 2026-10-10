# Lesson 1: Docker Fundamentals - Containers & Images

## Learning Objectives

By the end of this lesson, you will:

- **Understand** what containers are and why they're useful
- **Run** your first container using `docker run`
- **Explore** containers using Docker Desktop GUI and CLI
- **Understand** what images are and how they're composed of layers
- **Pull** and inspect images from Docker Hub
- **Manage** the container lifecycle (run, stop, remove)

**Time**: 45-60 minutes

## What is Docker? (The Simple Version)

Before we dive in, let's start with the basics - what problems does Docker solve?

### Problems Docker Solves

**Ever experienced these?**

**Problem 1: "It works on my machine!"**
- Your code works perfectly on your laptop
- Your teammate runs it and gets errors
- Production breaks because the server has different versions
- Sound familiar?

**Problem 2: Setting up takes forever**
- New team member joins
- Spends 2 days installing Node, Python, databases, dependencies...
- Half the time something doesn't work
- Instructions are outdated

**Problem 3: Conflicts everywhere**
- Project A needs Python 3.8
- Project B needs Python 3.11
- Installing one breaks the other
- Your system becomes a mess

### How Docker Solves This

**Docker containers** package your application with everything it needs:
- ✅ Your code
- ✅ The exact versions of dependencies (Node 18, Python 3.11, etc.)
- ✅ System libraries and tools
- ✅ Configuration

**The result?**
- Works on your machine → Works everywhere
- Setup time: 2 days → 30 seconds
- No more conflicts between projects

### Think of Containers Like Shipping Containers

Before shipping containers, moving goods was chaotic - different sizes, shapes, handling methods.

**Shipping containers standardized everything:**
- Standard size → Fits on any ship, truck, train
- Self-contained → Contents don't matter
- Portable → Goes anywhere

**Docker containers do the same for software:**
- Standard format → Runs on any machine with Docker
- Self-contained → All dependencies included
- Portable → Laptop, server, cloud - doesn't matter

### What About Images?

**Container Image** = A package containing your app and dependencies
**Container** = A running instance of that image

**Analogy:**
- **Image** = Recipe for a cake (instructions + ingredients list)
- **Container** = The actual cake you bake from that recipe

**Why this matters:**
- **Images are shareable** - You create an image once, anyone can run it
- **Images are on Docker Hub** - Like GitHub, but for container images
- **Containers are temporary** - Stop/start/delete them anytime, image remains

---

## What We're Building

In this lesson, we'll run pre-built containers to understand Docker basics. By the end, you'll be able to:

```bash
# Run a welcome container
docker run -d -p 8080:80 docker/welcome-to-docker

# View running containers
docker ps

# Access the app at http://localhost:8080

# View container logs
docker logs <container-name>

# Stop and remove the container
docker stop <container-name>
docker rm <container-name>
```

Let's start!

---

## Part 1: Your First Container

### What is a Container? (Quick Answer)

A **container** is like a lightweight, isolated box that runs your application.

**Think of it like this:**
- Your computer = An apartment building
- Containers = Individual apartments
- Each apartment is self-contained but shares the building's infrastructure

**Key point:** Containers are isolated from each other. What happens in one container doesn't affect others.

Let's see this in action!

### Step 1.1: Run Your First Container

Let me show you what happens when we run a container that hosts a simple web application.

**First, let me explain the command we'll use:**

```bash
docker run -d -p 8080:80 docker/welcome-to-docker
```

**Breaking it down:**
- `docker run`: Creates and starts a container
- `-d`: Runs in "detached" mode (background, not blocking terminal)
- `-p 8080:80`: Maps host port 8080 to container port 80 (more on this in Lesson 3)
- `docker/welcome-to-docker`: The image to use (we'll explain images in Part 2)

**Now let's run it and see what happens:**

[Claude runs via Bash tool: `docker run -d -p 8080:80 docker/welcome-to-docker`]

**Here's what we got:**

```
Unable to find image 'docker/welcome-to-docker:latest' locally
latest: Pulling from docker/welcome-to-docker
579b34f0a95b: Pull complete
d11a451e6399: Pull complete
...
Status: Downloaded newer image for docker/welcome-to-docker:latest
a1f7a4bb3a27e8f...
```

**Notice three things:**
1. `'Unable to find image locally'` - Docker checked your computer first
2. `'Pulling from docker/welcome-to-docker'` - Downloaded it from Docker Hub
3. `'a1f7a4bb3a27e8f...'` - That's the **container ID**, a unique identifier

The container is now running in the background!

### Step 1.2: View Your Running Container

Let me check if the container is actually running:

[Claude runs via Bash tool: `docker ps`]

**Here's what we see:**

```
CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS         PORTS                    NAMES
a1f7a4bb3a27   docker/welcome-to-docker   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp     gracious_keldysh
```

**Let me explain each column:**
- **CONTAINER ID**: `a1f7a4bb3a27` - Unique identifier (you can use just the first few characters in commands)
- **IMAGE**: `docker/welcome-to-docker` - Which image this container was created from
- **STATUS**: `Up 2 minutes` - It's running! "Up" means healthy and running
- **PORTS**: `0.0.0.0:8080->80/tcp` - Port 8080 on your computer maps to port 80 in the container
- **NAMES**: `gracious_keldysh` - Docker auto-generated this fun name (you can specify your own with `--name`)

The container is definitely running! ✅

### Step 1.3: Access the Web App

The container is running a web server. Since we mapped port 8080 on your computer to port 80 in the container, you can access it in your browser.

**Open your browser and go to:** [http://localhost:8080](http://localhost:8080)

You should see a welcome page! 🎉

**What's happening behind the scenes:**
- The container is running a web server on port 80 (inside the container)
- Docker mapped your host port 8080 to the container's port 80
- When you visit `localhost:8080`, Docker forwards the request into the container
- The web server inside responds, and you see the page

### Step 1.4: Explore the Container

Let me show you how to look inside a running container.

**First, let's view the logs** (output from the container's process):

[Claude runs via Bash tool: `docker logs <container-name-or-id>`]

**Here's the output:**

```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
...
2024/02/04 10:30:45 [notice] 1#1: start worker processes
```

These are the web server's startup logs - like `console.log` or `print` statements.

**Now let's run a command INSIDE the container:**

[Claude runs via Bash tool: `docker exec <container-name-or-id> ls /usr/share/nginx/html`]

**Here's what files are in the web server directory:**

```
index.html
assets
favicon.ico
```

**What just happened?** The `docker exec` command ran `ls` **inside** the container's filesystem, showing us the web server's files. Pretty cool - we can execute commands in the isolated container environment!

### Step 1.5: Stop Your Container

The container continues running until we stop it. Let me demonstrate:

[Claude runs via Bash tool: `docker stop <container-name-or-id>`]

**Tip**: You only need the first few characters of the container ID (like `docker stop a1f`).

**Now let's verify it stopped:**

[Claude runs via Bash tool: `docker ps`]

The container is no longer in the list! To see **all** containers (including stopped ones):

[Claude runs via Bash tool: `docker ps -a`]

```
CONTAINER ID   IMAGE                      ...   STATUS                      PORTS     NAMES
a1f7a4bb3a27   docker/welcome-to-docker   ...   Exited (0) 10 seconds ago             gracious_keldysh
```

Notice the STATUS changed from `Up X minutes` to `Exited (0) 10 seconds ago`. The container is stopped but still exists on your computer.

### Checkpoint 1.1: Test Your Understanding

**Before we continue, let me verify you understand what we just covered.**

[Use AskUserQuestion tool with 3-5 questions:]

**Question 1**: What does the `-d` flag do in `docker run -d`?
- A) Downloads the image from Docker Hub
- B) Runs container in background (detached mode) ✅
- C) Deletes old containers before starting

**Question 2**: In `-p 8080:80`, which port is on YOUR computer?
- A) 80
- B) Both are the same
- C) 8080 ✅ (host:container mapping)

**Question 3**: What output confirmed the container was running?
- A) STATUS showing "Up X minutes" in `docker ps` ✅
- B) The "Downloaded newer image" message
- C) The container ID in the `docker run` output

**Question 4**: What does `docker exec` allow you to do?
- A) Execute the container startup
- B) Execute Docker commands from outside
- C) Execute commands INSIDE a running container ✅

**Question 5**: When we ran `docker ps` after stopping, why didn't the container appear?
- A) The container was deleted automatically
- B) `docker ps` only shows RUNNING containers ✅
- C) The container doesn't exist anymore

**Review answers, explain any misconceptions, and only proceed when user demonstrates understanding.**

**After user answers correctly, pause and ask:**

"Great work! Before we move to Part 2, do you have any questions about:
- Containers and how they work
- The docker run command and its flags
- Container lifecycle (start, stop, logs, exec)

Or are you ready to continue learning about images?"

**Wait for user response. Answer any questions. Only proceed when they confirm they're ready.**

---

## Part 2: Understanding Images

### What is an Image?

You just ran a container from an image. But what exactly is an image?

**Simple answer:** An image is a package containing everything needed to run an application.

**What's inside an image?**
- The base operating system (usually a minimal Linux)
- Runtime (like Node.js, Python, Java, etc.)
- Your application code
- Dependencies and libraries
- Configuration files

**Image vs Container:**
- **Image** = The package/template (like a .zip file or installer)
- **Container** = What's running when you use that package

**Real-world analogy:**
- **Image** = A video game installer
- **Container** = The game actually running on your computer
- You can install the same game (image) and run it multiple times (containers)

### Why Images Are Powerful

**1. Images are shareable**
- Create an image once → Share with your whole team
- Everyone runs the exact same environment
- No more "works on my machine" problems

**2. Images are reusable**
- Build on top of existing images
- Example: Start with "Node.js 18" image, add your app code
- Don't reinvent the wheel

**3. Images are versioned**
- `nginx:1.25` vs `nginx:1.24` - Different versions
- Roll back if needed
- Test new versions safely

### Finding Images

**Docker Hub** ([https://hub.docker.com](https://hub.docker.com)) is the default global marketplace for images. It has over 100,000 images created by developers.

**Docker Trusted Content** provides high-quality images:
- **Docker Official Images**: Curated, secure repositories (starting point for most users)
- **Docker Hardened Images**: Minimal, secure, production-ready with near-zero CVEs
- **Docker Verified Publishers**: High-quality images from verified commercial publishers
- **Docker-Sponsored Open Source**: Images from projects sponsored by Docker

**Examples**:
- [Redis](https://hub.docker.com/_/redis) - Ready-to-use cache/database
- [PostgreSQL](https://hub.docker.com/_/postgres) - Database server
- [Node.js](https://hub.docker.com/_/node) - JavaScript runtime (base image for your apps)
- [Python](https://hub.docker.com/_/python) - Python runtime (base image for Python apps)

### Step 2.1: Search for Images

Let me show you how to search Docker Hub for images. We'll search for "nginx" (a popular web server):

[Claude runs via Bash tool: `docker search nginx`]

**Here's what we found:**

```
NAME                      DESCRIPTION                                     STARS     OFFICIAL
nginx                     Official build of Nginx.                        19000     [OK]
linuxserver/nginx         An Nginx container...                          180
bitnami/nginx             Bitnami nginx Docker Image                      150
```

**Notice:**
- `nginx` - The official image with 19000 stars and `[OK]` in OFFICIAL column
- Other variations from different publishers
- The official images are usually the safest choice for starting out

### Step 2.2: Pull an Image

**Pulling** downloads an image from Docker Hub to your local machine. Let me demonstrate:

[Claude runs via Bash tool: `docker pull nginx`]

**Here's what happened:**

```
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete
a9edb18cadd1: Pull complete
589b7251471a: Pull complete
186b1aaa4aa6: Pull complete
b4df32aa5a72: Pull complete
a0bcbecc962e: Pull complete
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

**Notice:**
- `Using default tag: latest` - Since we didn't specify a version, it used `:latest`
- Each `Pull complete` line represents a **layer** being downloaded
- Layers are shared between images to save disk space
- `docker.io/library/nginx:latest` - The full name of what we downloaded

**Image naming breakdown:**
- `docker.io` - Registry (Docker Hub)
- `library` - Namespace (official images)
- `nginx` - Repository (image name)
- `latest` - Tag (version)

### Step 2.3: List Your Images

Now let me show you all the images we have locally:

[Claude runs via Bash tool: `docker images`]

**Here's our image collection:**

```
REPOSITORY                 TAG       IMAGE ID       CREATED        SIZE
nginx                      latest    605c77e624dd   2 weeks ago    141MB
docker/welcome-to-docker   latest    648f93a1ba7d   4 months ago   29.7MB
```

**Notice:**
- `nginx` image we just pulled - 141MB in size
- `docker/welcome-to-docker` from earlier - only 29.7MB (much smaller!)
- Each image has a unique IMAGE ID (like a fingerprint)
- CREATED shows when the image was built, not when we downloaded it

### Step 2.4: Inspect Image Layers

Remember that images are composed of layers? Let me show you the layers in the nginx image:

[Claude runs via Bash tool: `docker image history nginx`]

**Here are the layers:**

```
IMAGE          CREATED       CREATED BY                                      SIZE
605c77e624dd   2 weeks ago   CMD ["nginx" "-g" "daemon off;"]                0B
<missing>      2 weeks ago   STOPSIGNAL SIGQUIT                              0B
<missing>      2 weeks ago   EXPOSE 80                                       0B
<missing>      2 weeks ago   ENTRYPOINT ["/docker-entrypoint.sh"]            0B
<missing>      2 weeks ago   COPY file:9e3b2b63db9f8fc7... /docker-entry…   4.6kB
<missing>      2 weeks ago   RUN /bin/sh -c set -x && addgroup --system ...  112MB
<missing>      2 weeks ago   ENV PKG_RELEASE=1~bookworm                      0B
<missing>      2 weeks ago   ENV NGINX_VERSION=1.25.3                        0B
```

**What we're seeing:**
- Each line is a **layer** stacked on top of each other
- Some layers are 0B - just metadata (ENV, CMD, EXPOSE)
- One layer is 112MB - the `RUN` command that installed nginx and dependencies
- Total image size (141MB) = sum of all layers

**Why layers matter:**
- **Caching**: When building images, unchanged layers are reused (makes builds MUCH faster)
- **Sharing**: Multiple images can share base layers (saves disk space)
- We'll dive deeper into this in Lesson 2 when we build our own images!
- **Efficiency**: Only changed layers need to be downloaded/uploaded

**Using Docker Desktop GUI:**

1. Go to **Images** view
2. Click on an image
3. Select **Layers** tab to visualize the layers

### Checkpoint 1.2: Test Your Understanding of Images

**Before we continue, let me verify you understand images.**

[Use AskUserQuestion tool with these questions:]

**Question 1**: What's the difference between an image and a container?
- A) They're the same thing
- B) Image = template/package, Container = running instance ✅
- C) Image = running, Container = stopped

**Question 2**: When we ran `docker pull nginx`, where did the image come from?
- A) Created locally from scratch
- B) Downloaded from nginx.com
- C) Docker Hub (docker.io registry) ✅

**Question 3**: What do the layers in `docker image history` represent?
- A) Different versions of the image
- B) Each step in building the image ✅
- C) Running containers from the image

**Question 4**: Why do images use layers?
- A) To make images smaller only
- B) Required by Docker specification
- C) Caching (reuse unchanged layers) and sharing (save disk space) ✅

**Question 5**: What does `nginx:latest` mean?
- A) The most recent version of nginx ever released
- B) nginx image with the 'latest' tag ✅
- C) nginx with automatic updates enabled

**Review answers, explain misconceptions, then proceed when user demonstrates understanding.**

**After user answers correctly, pause and ask:**

"Great work! Before we move to Part 3, do you have any questions about:
- The difference between images and containers
- Docker Hub and pulling images
- Image layers and how they work

Or are you ready to continue with container lifecycle management?"

**Wait for user response. Answer any questions. Only proceed when they confirm they're ready.**

---

## Part 3: Container Lifecycle

Now that you understand containers and images, let's explore the complete container lifecycle.

### The Container Lifecycle

```
┌──────────┐
│  Image   │  (Blueprint/Template)
└─────┬────┘
      │ docker run
      ▼
┌──────────┐
│ Created  │  (Container created but not started)
└─────┬────┘
      │ docker start
      ▼
┌──────────┐
│ Running  │  (Container actively running)
└─────┬────┘
      │ docker stop
      ▼
┌──────────┐
│ Stopped  │  (Container stopped but not deleted)
└─────┬────┘
      │ docker rm
      ▼
┌──────────┐
│ Removed  │  (Container deleted)
└──────────┘
```

### Step 3.1: Run a Container in Interactive Mode

So far, we've run containers in the background (`-d`). Now I want you to try **interactive** mode - where you can run commands directly inside a container.

**Your turn! Run this command in your terminal:**

```bash
docker run -it --rm ubuntu bash
```

**Breaking down the command:**
- `-it`: Interactive mode with a terminal
- `--rm`: Remove container automatically when you exit
- `ubuntu`: Ubuntu Linux image
- `bash`: Opens a bash shell

**You're now INSIDE the container!** Try these commands:

```bash
ls          # List files
pwd         # Print working directory
whoami      # Show current user
cat /etc/os-release   # See Ubuntu version
```

Notice you're in a minimal Ubuntu environment. Type `exit` to leave.

**What this shows:** You have a lightweight Linux computer running inside your computer!

**When you're done, let me know what you saw.**

### Step 3.2: View All Containers

Let me show you all containers on the system (running AND stopped):

[Claude runs via Bash tool: `docker ps -a`]

**Here's what we have:**

```
CONTAINER ID   IMAGE                      COMMAND   CREATED          STATUS                       PORTS
f7a3c9d2e1b4   docker/welcome-to-docker   ...       10 minutes ago   Exited (0) 5 minutes ago
a1f7a4bb3a27   nginx                      ...       15 minutes ago   Up 15 minutes                0.0.0.0:8080->80/tcp
```

**Notice the difference:**
- `docker ps` - Shows ONLY running containers
- `docker ps -a` - Shows ALL containers (running + stopped)

**The welcome-to-docker container:**
- STATUS: `Exited (0)` - It stopped (exit code 0 means success)
- Still exists on disk, just not running

**The nginx container:**
- STATUS: `Up 15 minutes` - Still running!

### Step 3.3: Restart a Stopped Container

Let me show you how to restart a stopped container. I'll restart the welcome-to-docker container:

[Claude runs via Bash tool: `docker start <container-id>`]

**Output:**
```
f7a3c9d2e1b4
```

The container ID is returned, confirming it started. Let's verify:

[Claude runs via Bash tool: `docker ps`]

**Now it's running again:**
```
CONTAINER ID   IMAGE                      STATUS         PORTS
f7a3c9d2e1b4   docker/welcome-to-docker   Up 2 seconds   0.0.0.0:8080->80/tcp
```

Notice STATUS changed from `Exited` to `Up 2 seconds`!

### Step 3.4: Viewing Container Logs

Every container's output (stdout/stderr) is captured in logs. Let me show you the nginx container's logs:

[Claude runs via Bash tool: `docker logs <nginx-container-id>`]

**Here's what the web server logged:**
```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
...
2024/02/04 10:45:23 [notice] 1#1: start worker processes
192.168.1.100 - - [04/Feb/2024:10:46:15 +0000] "GET / HTTP/1.1" 200 615
```

**What we're seeing:**
- Container startup messages
- Web server initialization
- HTTP requests logged (when you visited in your browser!)

**Tip:** Use `docker logs -f <container-id>` to follow logs in real-time (like `tail -f`).

### Step 3.5: Executing Commands in Running Containers

We already saw `docker exec` earlier, but let me show you another example - listing files in the nginx container:

[Claude runs via Bash tool: `docker exec <nginx-container-id> ls -la /usr/share/nginx/html`]

**Here are the web server's files:**
```
total 16
drwxr-xr-x 2 root root 4096 Dec 28 00:00 .
drwxr-xr-x 3 root root 4096 Dec 28 00:00 ..
-rw-r--r-- 1 root root  497 Dec 28 00:00 50x.html
-rw-r--r-- 1 root root  615 Dec 28 00:00 index.html
```

**What just happened?**
- We ran `ls -la` **inside** the running container
- The container didn't stop or restart
- We can run any command inside a running container

**Key difference:**
- `docker run`: Creates a **NEW** container from an image
- `docker exec`: Runs a command in an **EXISTING** running container

### Step 3.6: Inspecting Containers

Let me show you how to get detailed information about a container. I'll inspect one of our running containers:

[Claude runs via Bash tool: `docker inspect <container-id>`]

**This outputs detailed JSON. Let me show you some key information:**

[Claude runs via Bash tool: `docker inspect <container-id> --format='Status: {{.State.Status}}'`]
**Output:** `Status: running`

[Claude runs via Bash tool: `docker inspect <container-id> --format='IP Address: {{.NetworkSettings.IPAddress}}'`]
**Output:** `IP Address: 172.17.0.2`

[Claude runs via Bash tool: `docker inspect <container-id> --format='Image: {{.Config.Image}}'`]
**Output:** `Image: nginx`

**What `docker inspect` reveals:**
- Container state (running/stopped/paused)
- Internal IP address
- Port mappings
- Environment variables
- Mounted volumes
- Full configuration

**Basic usage:**
- `docker inspect <container-id>` - Show everything (lots of JSON!)
- `docker inspect <container-id> --format='{{.State.Status}}'` - Show specific field

This is useful for debugging network issues or understanding container configuration!

### Step 3.7: Removing Containers

Let me demonstrate the container cleanup process. First, let's stop a container:

[Claude runs via Bash tool: `docker stop <welcome-container-id>`]

**Output:** `f7a3c9d2e1b4`

Now remove it:

[Claude runs via Bash tool: `docker rm <welcome-container-id>`]

**Output:** `f7a3c9d2e1b4`

The container is now completely removed from disk. Let's verify:

[Claude runs via Bash tool: `docker ps -a | grep welcome`]

**No output** - the container is gone!

**Cleanup commands:**
- `docker stop <id>` - Stop a running container
- `docker rm <id>` - Remove a stopped container
- `docker rm -f <id>` - Force remove (stops and removes in one command)
- `docker container prune` - Remove ALL stopped containers

### Step 3.8: Removing Images

Images can be removed when you no longer need them. Let me demonstrate:

[Claude runs via Bash tool: `docker rmi nginx`]

**Output:**
```
Untagged: nginx:latest
Untagged: nginx@sha256:...
Deleted: sha256:605c77e624dd...
Deleted: sha256:a2abf6c4d29d...
```

**What happened:**
- Removed the tag `nginx:latest`
- Deleted all layers that aren't used by other images
- Freed up ~141MB of disk space

**Cleanup commands:**
- `docker rmi <image>` - Remove a specific image
- `docker image prune` - Remove unused images (dangling)
- `docker image prune -a` - Remove ALL unused images (aggressive cleanup)

### Checkpoint 1.3: Test Your Understanding of Container Lifecycle

**Before we finish this lesson, let me verify you understand the container lifecycle.**

[Use AskUserQuestion tool with these questions:]

**Question 1**: What's the difference between `docker ps` and `docker ps -a`?
- A) They're the same command
- B) `docker ps` is faster
- C) `docker ps` shows only running; `docker ps -a` shows all containers ✅

**Question 2**: What's the difference between `docker run` and `docker exec`?
- A) `run` is for images; `exec` is for containers
- B) `run` creates NEW container; `exec` runs command in EXISTING container ✅
- C) `exec` is faster

**Question 3**: What does `docker logs` show you?
- A) Docker daemon logs
- B) Container filesystem changes
- C) The stdout/stderr output from the container's process ✅

**Question 4**: When you `docker rm` a container, what happens to its data?
- A) Data is automatically backed up
- B) Data inside the container is deleted (unless stored in a volume) ✅
- C) Data persists on disk forever

**Question 5**: What does `docker start` do?
- A) Creates a new container from an image
- B) Starts the Docker daemon
- C) Restarts a STOPPED container ✅

**Review answers, explain misconceptions, then proceed when user demonstrates understanding.**

**After user answers correctly, pause and ask:**

"Excellent! Lesson 1 is complete. Before we finish, do you have any questions about:
- Container lifecycle (ps, logs, exec, start, stop, rm)
- Managing containers and images
- Anything we covered in this lesson

Or are you ready to move on to Lesson 2 where we'll build our own Docker images?"

**Wait for user response. Answer any questions. Only proceed when they confirm they're ready.**
4. Execute `ls /etc/nginx` inside the container
5. Stop the container
6. Verify it's stopped with `docker ps -a`
7. Remove the container

---

## Lesson Completion

Congratulations! You've completed Lesson 1! 🎉

### What You Built

In this lesson, you:
- ✅ Ran your first container (`docker/welcome-to-docker`)
- ✅ Explored containers using Docker Desktop and CLI
- ✅ Understood what images are and how they're composed of layers
- ✅ Pulled images from Docker Hub
- ✅ Managed the complete container lifecycle

### What You Learned

**Concepts:**
- Containers are isolated processes (vs VMs with full OS)
- Containers are lightweight, fast, and portable
- Images are immutable blueprints composed of layers
- Docker Hub is the marketplace for container images
- Container lifecycle: create → run → stop → remove

**Commands:**
```bash
docker run          # Create and start a container
docker ps           # List running containers
docker ps -a        # List all containers
docker stop         # Stop a running container
docker start        # Start a stopped container
docker rm           # Remove a container
docker logs         # View container logs
docker exec         # Execute command in running container
docker pull         # Download an image
docker images       # List images
docker image history # Show image layers
docker rmi          # Remove an image
```

### Key Takeaways

**Reflection Questions:**

1. **What problem does Docker solve?**
   <details>
   <summary>Answer</summary>
   Docker solves the "works on my machine" problem by packaging applications with all their dependencies. This ensures consistent behavior across different environments (your laptop, teammates' machines, production servers).
   </details>

2. **What's the difference between an image and a container?**
   <details>
   <summary>Answer</summary>
   - **Image**: A package/template containing your app and dependencies (like a recipe)
   - **Container**: A running instance of that image (like the actual meal made from the recipe)
   One image can create many containers.
   </details>

3. **What happens to container data when you stop vs remove a container?**
   <details>
   <summary>Answer</summary>
   - **Stop** (`docker stop`): Data preserved, container still exists
   - **Remove** (`docker rm`): Data lost, container deleted
   - **Image remains untouched** in both cases
   We'll learn about persisting important data with volumes in Lesson 3.
   </details>

### 🤓 Want to Go Deeper? (Optional)

**Curious about how containers actually work under the hood?**

If you want to understand the technical details of container isolation, namespaces, and how containers differ from VMs at a kernel level, check out:
- `architecture/container-isolation.md` - Deep dive into Linux namespaces, cgroups, and container vs VM architecture

**But don't feel like you need to read this right now!** The practical knowledge you've gained is enough to continue. You can explore the architecture later if you're curious.

### Next Steps

In **Lesson 2**, you'll learn to build your own custom images:
- Writing Dockerfiles from scratch
- Building images with `docker build`
- Optimizing builds with layer caching
- Creating production-ready images with multi-stage builds

Ready to continue? Let me know when you'd like to start Lesson 2!
