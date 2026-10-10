# Architecture Deep-Dive: Image Layers

## Overview

This document explains **how** Docker images are structured using layers, **why** this design exists, and **what** it means for building efficient images.

You've learned the practical benefits of layers (caching!). Now let's see how it works under the hood.

---

## The Core Concept: Layers

**Every instruction in your Dockerfile creates a layer.**

**Example Dockerfile:**
```dockerfile
FROM node:18-alpine    # Layer 1: Base image
WORKDIR /app          # Layer 2: Create /app directory
COPY package.json .   # Layer 3: Add package.json
RUN npm install       # Layer 4: Add node_modules
COPY . .             # Layer 5: Add application code
CMD ["node", "app.js"] # Layer 6: Metadata (no filesystem changes)
```

**Think of layers like:**
- Transparent sheets stacked on top of each other
- Each sheet adds, modifies, or removes files
- Final image = All sheets combined

---

## How Layers Work: Union Filesystem

Docker uses a **union filesystem** (specifically OverlayFS on Linux) to combine layers.

### Simple Analogy: Transparency Layers

Imagine you're making a comic book:

**Layer 1 (base):** White paper with border
**Layer 2:** Background drawing (sky, ground)
**Layer 3:** Character drawing
**Layer 4:** Speech bubbles

Stack them up → You see the complete scene!

### In Docker

**Layer 1 (FROM node:18-alpine):**
```
/bin/sh
/usr/bin/node
/usr/bin/npm
... (base OS files)
```

**Layer 2 (WORKDIR /app):**
```
/app/         # New directory created
```

**Layer 3 (COPY package.json):**
```
/app/package.json    # File added
```

**Layer 4 (RUN npm install):**
```
/app/node_modules/   # Directory with dependencies
```

**Layer 5 (COPY . .):**
```
/app/src/
/app/index.js
... (your code)
```

**Final image** = All layers stacked:
```
/
├── bin/
│   └── sh
├── usr/
│   └── bin/
│       ├── node
│       └── npm
└── app/
    ├── node_modules/
    ├── package.json
    ├── src/
    └── index.js
```

---

## Why Layers Exist

### 1. **Efficient Storage**

**Problem without layers:**
- Image A: Ubuntu + Python + App A = 500MB
- Image B: Ubuntu + Python + App B = 500MB
- Total: 1000MB

**With layers:**
- Ubuntu base: 100MB (shared!)
- Python layer: 150MB (shared!)
- App A layer: 10MB
- App B layer: 15MB
- **Total: 275MB** (Ubuntu and Python stored once!)

Multiple images share the same base layers = huge space savings!

### 2. **Fast Builds with Caching**

**Without layers:**
Every change → Rebuild everything from scratch

**With layers:**
- Unchanged layer → Reused from cache (instant!)
- Changed layer → Rebuild only that layer and layers after it

**Example:**
```dockerfile
FROM node:18-alpine    # Layer 1: Cached (never changes)
WORKDIR /app          # Layer 2: Cached (never changes)
COPY package.json .   # Layer 3: Cached (rarely changes)
RUN npm install       # Layer 4: Cached (rarely changes)
COPY . .             # Layer 5: Changed (you edit code)
CMD ["node", "app.js"] # Layer 6: Cached (metadata only)
```

Only Layer 5 rebuilds! Layers 1-4 reused from cache.

**Real impact:**
- First build: 60 seconds
- Rebuild after code change: 2 seconds

### 3. **Efficient Distribution**

**Pulling an image:**
- Docker checks which layers you already have
- Only downloads missing/changed layers
- Reuses layers from other images

**Example:**
You already have `node:18-alpine` image (from another project).
When you pull `my-app` (based on `node:18-alpine`):
- Base layers: Already have them! (skipped)
- App layers: Download only these (small)

**Result:** Faster pulls and less bandwidth!

---

## Layer Immutability

**Key principle: Layers are read-only and immutable.**

Once a layer is created, it never changes. This is crucial for caching and sharing.

### What "Immutable" Means

**You can't modify a layer after it's created.**

**Example:**
```dockerfile
RUN apt-get install -y nginx       # Layer 1: Adds nginx
RUN apt-get remove -y nginx        # Layer 2: "Removes" nginx
```

**What actually happens:**
- Layer 1: nginx installed (100MB added)
- Layer 2: nginx marked as deleted (0MB removed, but Layer 1 still exists!)
- **Final image size: Still includes 100MB from Layer 1!**

The file is hidden, not deleted!

### Why This Matters

**To truly reduce image size:**
```dockerfile
# Bad: Two layers
RUN apt-get install -y nginx build-essential
RUN apt-get remove -y build-essential    # Doesn't reduce size!

# Good: One layer
RUN apt-get install -y nginx build-essential && \
    apt-get remove -y build-essential     # Now it's smaller!
```

Combining commands in one RUN means everything happens in one layer.

---

## Copy-on-Write

**What happens when a container modifies a file?**

**Layers in image (read-only):**
```
Layer 1: /app/config.json (default config)
```

**Container starts and modifies it:**
```bash
# Inside container
echo '{"theme": "dark"}' > /app/config.json
```

**What happens:**
1. Container can't modify the read-only layer
2. Docker **copies** the file to the container's writable layer
3. Container modifies the copy
4. When reading, Docker shows the modified version

This is called **copy-on-write**.

**Visualized:**
```
┌─────────────────────────────────┐
│ Container Writable Layer        │
│ /app/config.json (modified) ←   │ This is what you see
└─────────────────────────────────┘
┌─────────────────────────────────┐
│ Image Layer (read-only)         │
│ /app/config.json (original)     │ Still here, but hidden
└─────────────────────────────────┘
```

**Why this matters:**
- Multiple containers from same image don't interfere with each other
- Each container has its own writable layer
- Original image remains unchanged

**But also:**
- When container is deleted, modified files are lost (unless you use volumes!)

---

## Practical Implications

### 1. Layer Ordering Strategy

**Put things that change rarely at the top:**

```dockerfile
# Changes never: Base image
FROM node:18-alpine

# Changes never: Working directory
WORKDIR /app

# Changes rarely: Dependencies
COPY package.json package-lock.json ./
RUN npm install

# Changes often: Your code
COPY . .

# Changes never: Start command
CMD ["node", "app.js"]
```

**Result:** Most layers cached, fast rebuilds!

### 2. Minimize Layers in Production

**Development Dockerfile (many layers for debugging):**
```dockerfile
FROM node:18
COPY package.json .
RUN npm install
COPY src/ ./src/
COPY config/ ./config/
CMD ["node", "src/app.js"]
```

**Production Dockerfile (fewer, optimized layers):**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force
COPY . .
CMD ["node", "src/app.js"]
```

### 3. Use .dockerignore

**Prevent unnecessary layer invalidation:**

Without `.dockerignore`:
```
COPY . .     # Copies EVERYTHING including .git, node_modules, logs
             # Any change in any file → Layer invalidated
```

With `.dockerignore`:
```
node_modules
.git
*.log
```
```
COPY . .     # Only copies relevant files
             # Changes to .git or logs don't invalidate cache
```

---

## How to Inspect Layers

**See layers of an image:**
```bash
docker image history my-app
```

**Output:**
```
IMAGE          CREATED BY                                      SIZE
a1b2c3d4e5f6   CMD ["node" "app.js"]                          0B
b2c3d4e5f6a7   COPY . .                                        1.2MB
c3d4e5f6a7b8   RUN npm install                                 45MB
d4e5f6a7b8c9   COPY package.json .                             2kB
e5f6a7b8c9d0   WORKDIR /app                                    0B
f6a7b8c9d0e1   FROM node:18-alpine                             150MB
```

**Each line is a layer!**

**Useful flags:**
```bash
# Show full commands (not truncated)
docker image history --no-trunc my-app

# Show only layers that add size
docker image history --format "{{.Size}}" my-app
```

---

## Advanced: Multi-Stage Builds and Layers

**Multi-stage builds create separate layer sets:**

```dockerfile
# Build stage layers
FROM node:18 AS builder        # Layer B1
WORKDIR /app                   # Layer B2
COPY . .                       # Layer B3
RUN npm install                # Layer B4: Includes dev dependencies
RUN npm run build              # Layer B5: Build artifacts

# Production stage layers (completely separate!)
FROM node:18-alpine            # Layer P1
WORKDIR /app                   # Layer P2
COPY --from=builder /app/dist ./dist    # Layer P3: Only built code
CMD ["node", "dist/app.js"]    # Layer P4
```

**Final image only includes Production stage layers (P1-P4)!**

Build stage layers (B1-B5) are discarded = smaller image!

---

## Summary

**Key Takeaways:**

1. **Every Dockerfile instruction creates a layer**
   - Layers stack to form the final image
   - Like transparent sheets overlaid

2. **Layers are immutable and cached**
   - Once created, never changes
   - Reused across builds and images
   - Enables fast builds and efficient storage

3. **Layer ordering matters**
   - Rarely-changing things first (dependencies)
   - Frequently-changing things last (code)
   - Maximizes cache hits

4. **Copy-on-write for containers**
   - Containers get a writable layer on top
   - Original image layers remain read-only
   - Changes persist in container layer only

5. **Combine commands to reduce layers**
   - `RUN install && remove` in one layer
   - Smaller final images

**Understanding layers helps you:**
- ✅ Write faster-building Dockerfiles
- ✅ Create smaller images
- ✅ Debug build issues
- ✅ Optimize for production

---

**Back to Tutorial:** Return to Lesson 2 to continue learning!
