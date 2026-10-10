# Lesson 3: Running Applications - Networks, Data & Multi-Container Apps

## Learning Objectives

By the end of this lesson, you will:

- **Publish** container ports so you can access applications
- **Understand** why containers can't be accessed by default
- **Persist** data using volumes (so it doesn't disappear!)
- **Connect** multiple containers together
- **Use** Docker Compose to manage multi-container applications

**Time**: 45-60 minutes

## The Problems We're Solving

You've learned to run containers and build images. But real applications need more:

### Problem 1: Can't Access Containers

```bash
docker run -d nginx
# Container is running... but how do I see the website?
# Can't access it in my browser! 🤔
```

**Why?** Containers are isolated. They have their own network.

### Problem 2: Data Disappears

```bash
# Add a todo item
docker run my-todo-app
# Add: "Buy groceries"

# Stop and remove container
docker stop my-todo && docker rm my-todo

# Start a new one
docker run my-todo-app
# Todo is gone! 😱
```

**Why?** Container filesystem is temporary.

### Problem 3: Multi-Container Complexity

**Real apps have multiple services:**
- Web app (frontend)
- API (backend)
- Database

```bash
# Managing manually is painful:
docker run -d --name db postgres
docker run -d --name api --link db my-api
docker run -d --name web -p 3000:3000 --link api my-web
# Too many commands! 😫
```

**This lesson solves all three problems!**

---

## Part 1: Networking Basics - Publishing Ports

### Why Can't I Access My Container?

**Containers are isolated environments.** They have their own network stack.

**Think of it like this:**
- Your computer = An apartment building
- Each container = An apartment with its own door
- By default, apartment doors are locked
- Publishing ports = Giving visitors a key

### What is Port Publishing?

**Port publishing** creates a forwarding rule:
- Traffic to `YOUR_COMPUTER:8080` → Forwarded to `CONTAINER:80`

**The `-p` flag syntax:**
```bash
docker run -p HOST_PORT:CONTAINER_PORT image
```

- `HOST_PORT` - Port on your computer
- `CONTAINER_PORT` - Port inside container

**Example:**
```bash
docker run -d -p 8080:80 nginx
```
- Container's nginx listens on port 80 (inside container)
- Your computer forwards port 8080 → container's port 80
- Access at: http://localhost:8080

### Step 1.1: Publish a Port

Let me run an nginx web server and publish its port:

[Claude runs via Bash tool: `docker run -d -p 8080:80 --name my-nginx nginx`]

**Output:** `abc123def456...` (container ID)

**Command breakdown:**
- `-d` - Run in background
- `-p 8080:80` - Forward host port 8080 to container port 80
- `--name my-nginx` - Name it "my-nginx"
- `nginx` - The image to use

### Step 1.2: Understanding Port Mapping

Let me check the port mapping:

[Claude runs via Bash tool: `docker ps`]

**Here's what we see:**

```
CONTAINER ID   IMAGE   COMMAND   PORTS                   NAMES
abc123def456   nginx   ...       0.0.0.0:8080->80/tcp    my-nginx
```

**Looking at the PORTS column: `0.0.0.0:8080->80/tcp`**

This means:
- `0.0.0.0:8080` - Listen on all network interfaces, port 8080
- `->` - Forward to
- `80/tcp` - Container's port 80, TCP protocol

**Now open http://localhost:8080 in your browser.**

You should see "Welcome to nginx!" 🎉

### Step 1.3: Using Different Ports

Let me show you something cool - running multiple nginx containers simultaneously on different ports:

[Claude runs via Bash tool: Three docker run commands with different names and ports]

**Now let me check all running containers:**

[Claude runs via Bash tool: `docker ps`]

**Here's what we have:**

```
CONTAINER ID   IMAGE   PORTS                   NAMES
abc...         nginx   0.0.0.0:8080->80/tcp    nginx1
def...         nginx   0.0.0.0:8081->80/tcp    nginx2
ghi...         nginx   0.0.0.0:8082->80/tcp    nginx3
```

**All three run simultaneously:**
- http://localhost:8080 → nginx1
- http://localhost:8081 → nginx2
- http://localhost:8082 → nginx3

**Why this works:** Each container has its own isolated network namespace!

### Step 1.4: Let Docker Choose the Port

Don't care which port? Let me show you how Docker can pick one automatically:

[Claude runs via Bash tool: `docker run -d -p 80 --name auto-port nginx`]

When you omit the host port, Docker picks a random available port. Let's see what it chose:

[Claude runs via Bash tool: `docker ps | grep auto-port`]

**Output:**

```
CONTAINER ID   IMAGE   PORTS                    NAMES
jkl...         nginx   0.0.0.0:54321->80/tcp    auto-port
```

Docker picked port 54321! You can access it at http://localhost:54321

This is useful for automated setups where you don't care about the specific port number.

### Checkpoint 3.1: Test Your Understanding of Port Publishing

**Let me verify you understand port publishing.**

[Use AskUserQuestion tool with these questions:]

**Question 1**: What does `-p 3000:80` do?
- A) Forwards container port 3000 to host port 80
- B) Opens port 3000 on both
- C) Forwards host port 3000 to container port 80 ✅

**Question 2**: In the PORTS column showing `0.0.0.0:8080->80/tcp`, which is the host port?
- A) 80
- B) 8080 ✅
- C) 0.0.0.0

**Question 3**: Why can two containers both listen on port 80 internally?
- A) Docker automatically changes one to port 81
- B) Each container has its own isolated network namespace ✅
- C) They can't - it will conflict

**Question 4**: What happens if port 8080 is already in use on your computer?
- A) Docker automatically picks another port
- B) The container starts but is inaccessible
- C) Docker errors: "port is already allocated" ✅

**Question 5**: When you omit the host port (like `-p 80`), what happens?
- A) It uses port 80 on the host
- B) Docker picks a random available host port ✅
- C) The container won't start

**Review answers, explain misconceptions, then continue to Part 2!**

**After user answers correctly, pause and ask:**

"Great work! Before we move to Part 2, do you have any questions about:
- Port publishing and the -p flag
- Why containers are network-isolated
- How to access containers from your browser

Or are you ready to learn about persisting data with volumes?"

**Wait for user response. Answer any questions. Only proceed when they confirm they're ready.**

---

## Part 2: Data Persistence - Volumes

### The Problem: Disappearing Data

**Containers are ephemeral (temporary):**

```bash
# Run a database
docker run -d --name db postgres

# Add data to database
# ... (data stored in container)

# Remove container
docker rm -f db

# Data is GONE! 😱
```

**Why?** Container filesystem is deleted when container is removed.

### The Solution: Volumes

**Volumes** persist data outside the container.

**Think of volumes like:**
- An external hard drive
- Container is deleted → External drive keeps the data
- New container → Plug in same external drive → Data still there!

### How Volumes Work

**Volume = A directory managed by Docker, outside containers**

```
Your Computer
├── /var/lib/docker/volumes/my-data/     # Volume storage
│
Containers
├── Container A (removed) ❌
│   └── /data → linked to volume          # Data preserved
│
└── Container B (new) ✅
    └── /data → linked to same volume     # Data still accessible!
```

### Step 2.1: Create and Use a Volume

Let me create a named volume:

[Claude runs via Bash tool: `docker volume create my-data`]

**Output:** `my-data`

Now let me run a container with this volume mounted:

[Claude runs via Bash tool: `docker run -d --name my-app -v my-data:/app/data alpine sleep 3600`]

**Command breakdown:**
- `-v my-data:/app/data` - Mount volume "my-data" to "/app/data" inside container
- `alpine sleep 3600` - Simple container that stays running for testing
```

**Breaking it down:**
- `-v my-data:/app/data` - Mount volume
  - `my-data` - Volume name
  - `:` - Separator
  - `/app/data` - Where to mount inside container

**Now `/app/data` in the container** stores files in the volume!

### Step 2.2: Test Data Persistence

Let me prove data persists across container deletions. First, write a file to the volume:

[Claude runs via Bash tool: `docker exec my-app sh -c "echo 'Hello from container 1!' > /app/data/message.txt"`]

Now read it back:

[Claude runs via Bash tool: `docker exec my-app cat /app/data/message.txt`]

**Output:** `Hello from container 1!`

Now let me remove the container:

[Claude runs via Bash tool: `docker rm -f my-app`]

The container is gone. Let me start a NEW container with the SAME volume:

[Claude runs via Bash tool: `docker run -d --name my-app-2 -v my-data:/app/data alpine sleep 3600`]

Now read the file from the new container:

[Claude runs via Bash tool: `docker exec my-app-2 cat /app/data/message.txt`]

**Output:** `Hello from container 1!`

**The file survived container deletion!** ✅ This is the power of volumes.

### Step 2.3: Real Example - Database Persistence

This same principle applies to databases. When you run a database (MySQL, PostgreSQL, MongoDB) with a volume, the data survives container restarts.

**Pattern:**
```bash
docker run -d \
  --name my-db \
  -e POSTGRES_PASSWORD=secret \
  -v postgres-data:/var/lib/postgresql/data \  # Database writes here
  postgres
```

The database writes to `/var/lib/postgresql/data`, which is mounted to the `postgres-data` volume. Even if you remove and recreate the container, your data persists!

### Step 2.4: Managing Volumes

Let me show you how to manage volumes:

[Claude runs via Bash tool: `docker volume ls`]

**Output:**
```
DRIVER    VOLUME NAME
local     my-data
local     postgres-data
```

Let me inspect our volume:

[Claude runs via Bash tool: `docker volume inspect my-data`]

**Output:**
```json
[
    {
        "CreatedAt": "2024-02-04T12:00:00Z",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/my-data/_data",
        "Name": "my-data"
    }
]
```

**Key field:** `Mountpoint` shows where Docker stores the volume data on your computer.

**Volume commands:**
- `docker volume ls` - List all volumes
- `docker volume inspect <name>` - Show volume details
- `docker volume rm <name>` - Remove volume (must not be in use)
- `docker volume prune` - Remove all unused volumes

### Checkpoint 3.2: Test Your Understanding of Volumes

**Let me verify you understand data persistence.**

[Use AskUserQuestion tool with these questions:]

**Question 1**: What happens to data inside a container when you remove it?
- A) Data is automatically backed up
- B) Data is deleted unless stored in a volume ✅
- C) Data persists in the container forever

**Question 2**: Why use volumes instead of storing data in containers?
- A) Volumes are faster to create
- B) Data persists across container restarts/recreations ✅
- C) Containers can't store data

**Question 3**: What does `-v my-data:/app/data` do?
- A) Copies data from my-data to /app/data
- B) Mounts volume "my-data" to "/app/data" inside container ✅
- C) Creates a new volume called /app/data

**Question 4**: In our test, the message.txt file survived because:
- A) Docker automatically backs up files
- B) It was stored in a volume, which exists outside the container ✅
- C) We didn't actually delete the container

**Question 5**: When can you remove a volume?
- A) Anytime
- B) Only when no containers are using it ✅
- C) Never - volumes can't be removed

**Review answers, explain misconceptions, then continue to Part 3!**

**After user answers correctly, pause and ask:**

"Excellent! Before we move to Part 3, do you have any questions about:
- Volumes and how they persist data
- The -v flag and mounting volumes
- Why data in containers is temporary without volumes

Or are you ready to learn about Docker Compose for multi-container applications?"

**Wait for user response. Answer any questions. Only proceed when they confirm they're ready.**

---

## Part 3: Multi-Container Apps with Docker Compose

### The Problem: Manual Container Management

**A typical application:**
- Frontend (React)
- Backend API (Node.js)
- Database (PostgreSQL)

**Managing manually:**
```bash
# Start database
docker run -d --name db -e POSTGRES_PASSWORD=secret postgres

# Start API (needs to connect to db)
docker run -d --name api --link db -e DB_HOST=db my-api

# Start frontend (needs to connect to api)
docker run -d --name web -p 3000:3000 --link api my-frontend
```

**Problems:**
- Many commands to remember
- Easy to forget flags or make mistakes
- Hard to share with team
- Order matters (db before api, api before web)
- No easy way to start/stop everything together

### The Solution: Docker Compose

**Docker Compose** = Define multi-container apps in one file

**Instead of multiple commands**, you write one config file:

**compose.yaml:**
```yaml
services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: secret

  api:
    build: ./api
    depends_on:
      - db

  web:
    build: ./web
    ports:
      - "3000:3000"
    depends_on:
      - api
```

**Then run ONE command:**
```bash
docker compose up
```

**Everything starts in the right order!** 🎉

### Step 3.1: Your First Compose File

Let me create a simple multi-container app: WordPress + MySQL.

First, create a directory:

[Claude runs via Bash tool: `mkdir my-wordpress && cd my-wordpress`]

Now let me create a `compose.yaml` file:

[Claude runs via Write tool to create compose.yaml]

**Here's the compose.yaml we created:**

```yaml
services:
  # Database service
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - db-data:/var/lib/mysql

  # WordPress service
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - db

# Define named volumes
volumes:
  db-data:
```

**Understanding the file:**
- `services:` - Defines containers (like separate `docker run` commands)
- `db:` - Database service (MySQL container)
- `wordpress:` - Web service (WordPress container)
- `depends_on: [db]` - Start db before wordpress
- `volumes:` - Named volume for persistent database data
- `WORDPRESS_DB_HOST: db` - WordPress connects to database using service name "db"

### Step 3.2: Start Everything

Now let me start the entire application with ONE command:

[Claude runs via Bash tool: `docker compose up -d`]

**Here's what happened:**

```
[+] Running 3/3
 ✔ Network my-wordpress_default    Created
 ✔ Container my-wordpress-db-1        Started
 ✔ Container my-wordpress-wordpress-1 Started
```

**Notice:**
1. Docker created a network (`my-wordpress_default`)
2. Started `db` container first
3. Started `wordpress` container after (because of `depends_on`)

Let me check the status:

[Claude runs via Bash tool: `docker compose ps`]

**Output:**

```
NAME                        IMAGE              STATUS         PORTS
my-wordpress-db-1           mysql:8            Up 10 seconds  3306/tcp
my-wordpress-wordpress-1    wordpress:latest   Up 8 seconds   0.0.0.0:8080->80/tcp
```

Both services are running! Let me view the logs:

[Claude runs via Bash tool: `docker compose logs`]

**Sample logs:**
```
db-1         | MySQL ready for connections
wordpress-1  | WordPress installation started...
```

**Now open http://localhost:8080 in your browser.**

You should see WordPress setup! 🎉

### Step 3.3: Service Communication

Let me demonstrate how containers communicate. I'll execute a command in the wordpress container to ping the database:

[Claude runs via Bash tool: `docker compose exec wordpress ping -c 2 db`]

**Output:**

```
PING db (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: icmp_seq=0 ttl=64 time=0.123 ms
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.098 ms
```

**See that?** The service name `db` resolved to the database container's IP (172.18.0.2)!

**Docker Compose automatically:**
- Creates a network for all services
- Adds DNS resolution (service name = hostname)
- Isolates services from other projects

This is why `WORDPRESS_DB_HOST: db` works - "db" is the service name!

### Step 3.4: Managing the Application

Let me show you the lifecycle commands.

**Stop everything (containers remain):**

[Claude runs via Bash tool: `docker compose stop`]

**Start everything again:**

[Claude runs via Bash tool: `docker compose start`]

**Stop and remove everything (but keep volumes):**

[Claude runs via Bash tool: `docker compose down`]

**Output:**
```
[+] Running 3/3
 ✔ Container my-wordpress-wordpress-1  Removed
 ✔ Container my-wordpress-db-1         Removed
 ✔ Network my-wordpress_default        Removed
```

Containers and network are removed, but `db-data` volume persists!

**Stop, remove, AND delete volumes:**

[Would run: `docker compose down -v` - but we'll skip to preserve data]

**Key commands:**
- `docker compose up -d` - Start all services
- `docker compose stop` - Stop (but don't remove)
- `docker compose start` - Restart stopped services
- `docker compose down` - Stop and remove containers & networks
- `docker compose down -v` - Also remove volumes
- `docker compose ps` - Show status
- `docker compose logs` - View all logs
- `docker compose logs <service>` - View logs for one service
- `docker compose exec <service> <cmd>` - Execute command in service

### Checkpoint 3.3: Test Your Understanding of Docker Compose

**Let me verify you understand Docker Compose.**

[Use AskUserQuestion tool with these questions:]

**Question 1**: What does Docker Compose do?
- A) Composes images from layers
- B) Manages multi-container apps with a single YAML file ✅
- C) Compresses Docker images

**Question 2**: How do containers communicate in Compose?
- A) Using IP addresses only
- B) They can't communicate
- C) Using service names as hostnames (automatic DNS) ✅

**Question 3**: What does `depends_on: [db]` do?
- A) Links the containers
- B) Starts db before starting this service ✅
- C) Shares volumes between services

**Question 4**: When we pinged "db" from wordpress, why did it work?
- A) They're on the same computer
- B) Docker Compose created DNS resolution for service names ✅
- C) MySQL has a built-in DNS server

**Question 5**: What does `docker compose down` do?
- A) Only stops containers
- B) Stops and removes containers and networks (but keeps volumes) ✅
- C) Deletes everything including volumes

**Review answers, explain misconceptions. Congratulations - you've completed all 3 lessons!**

**After user answers correctly, pause and ask:**

"Congratulations! You've completed all 3 lessons! 🎉 Before we finish, do you have any questions about:
- Docker Compose and managing multi-container apps
- Service communication and networking
- Anything from the entire tutorial

Or are you ready to start using Docker in your own projects?"

**Wait for user response. Answer any questions. Celebrate their completion!**

---

## Lesson Completion

Congratulations! You've completed Lesson 3 and the entire Docker tutorial! 🎉🎉🎉

### What You Built

In this lesson, you:
- ✅ Published container ports to access applications
- ✅ Persisted data with volumes (database survives restarts!)
- ✅ Connected multiple containers with networking
- ✅ Managed multi-container apps with Docker Compose
- ✅ Built a complete full-stack application (app + database)

### What You Learned

**Concepts:**
- Containers are network-isolated by default
- Port publishing creates forwarding rules
- Volumes persist data outside containers
- Containers can share volumes
- Docker Compose simplifies multi-container management
- Service names become hostnames in Compose networks

**Commands:**
```bash
# Ports
docker run -p HOST:CONTAINER image    # Publish port

# Volumes
docker volume create name              # Create volume
docker volume ls                       # List volumes
docker volume rm name                  # Remove volume
docker run -v name:/path image        # Mount volume

# Docker Compose
docker compose up                      # Start services
docker compose up -d                   # Start in background
docker compose down                    # Stop and remove
docker compose ps                      # List services
docker compose logs                    # View logs
docker compose exec service cmd       # Run command in service
```

**compose.yaml structure:**
```yaml
services:
  service-name:
    image: image-name          # Or 'build: .'
    ports:
      - "HOST:CONTAINER"
    environment:
      KEY: value
    volumes:
      - volume:/path
    depends_on:
      - other-service

volumes:
  volume-name:
```

### Key Takeaways

**You now have the skills to:**
1. ✅ **Package applications** - Build images with Dockerfiles
2. ✅ **Run applications** - Start containers with correct ports
3. ✅ **Persist data** - Use volumes for databases and important files
4. ✅ **Connect services** - Multi-container apps with Compose
5. ✅ **Share applications** - Team members run identical environments

**Real-world impact:**
- No more "works on my machine" problems
- Setup time: Hours → Seconds
- Consistent environments: dev, staging, production
- Easy collaboration: Share compose.yaml with team

### Complete Tutorial Summary

**Lesson 1: Fundamentals**
- Learned what Docker is and why it exists
- Ran first containers
- Understood images and containers
- Managed container lifecycle

**Lesson 2: Building Images**
- Created Dockerfiles
- Built custom images
- Optimized builds with layer caching
- Reduced image sizes

**Lesson 3: Running Apps**
- Published ports for networking
- Persisted data with volumes
- Orchestrated multi-container apps with Compose

**You're now equipped to containerize and deploy real applications!** 🚀

### What's Next?

**Continue learning:**
- Docker security best practices
- Container orchestration (Kubernetes basics)
- CI/CD with Docker
- Docker in production
- Advanced networking
- Docker Swarm

**Practice projects:**
- Containerize your own application
- Set up a development environment with Compose
- Deploy a multi-service app
- Create a CI/CD pipeline

**Resources:**
- [Official Docker docs](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/) - Explore images
- Practice with `docker ai` for questions

### 🤓 Want to Go Deeper? (Optional)

**Curious about the technical details?**

**Networking:**
- `architecture/networking.md` - How bridge networks, port forwarding, and DNS work

**Compose:**
- `architecture/compose-orchestration.md` - How Compose manages services, networks, and volumes

**But you don't need these to use Docker effectively!** The practical knowledge you have is what matters most.

---

## Congratulations! 🎉

You've completed the entire Docker learning tutorial!

**You started with:**
- ❓ "What is Docker?"
- ❓ "How do I run containers?"

**You now know:**
- ✅ How to package applications as images
- ✅ How to run containers reliably
- ✅ How to persist data
- ✅ How to deploy multi-container apps
- ✅ How to solve "works on my machine" problems

**You're ready to use Docker in real projects!**

Keep practicing, keep building, and enjoy your Docker journey! 🐳
