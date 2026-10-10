# Checkpoint 2.2: Build Optimization

## Purpose

This checkpoint verifies that you understand layer caching, can optimize Dockerfiles for faster rebuilds, and know how to use .dockerignore effectively. These optimizations are crucial for real-world development!

## Verification Questions

<details>
<summary>Question 1: Why is layer ordering important for build speed?</summary>

**Answer:** **Because Docker caches layers!**

**How caching works:**
- Docker checks each instruction
- If nothing changed → Reuse cached layer (instant!)
- If something changed → Rebuild that layer AND all layers after it

**Example:**

**Bad order (slow):**
```dockerfile
COPY . .              # Copies ALL code
RUN npm install       # Installs dependencies
```
Every code change invalidates `npm install` cache → Slow rebuilds!

**Good order (fast):**
```dockerfile
COPY package.json .   # Copy only dependencies list
RUN npm install       # Cached unless package.json changes
COPY . .             # Code changes don't affect npm install!
```
Code changes don't force reinstalling dependencies → Fast rebuilds!

**Real impact:**
- Bad order: 3 minutes per rebuild (reinstalling dependencies)
- Good order: 5 seconds per rebuild (code copy only)

**Rule of thumb:** Put things that change rarely (dependencies) BEFORE things that change frequently (your code).
</details>

<details>
<summary>Question 2: What does .dockerignore do and why use it?</summary>

**Answer:** **Excludes files from the build context**, similar to `.gitignore`.

**What to exclude:**
```
node_modules/      # We install these with npm install
.git/             # Not needed in image
*.log             # Development/debug files
.env              # Secrets shouldn't be in images!
README.md         # Documentation
.dockerignore     # Meta file
Dockerfile        # Not needed inside image
```

**Benefits:**
1. **Faster builds** - Less to copy into build context
2. **Smaller images** - Unnecessary files don't end up in final image
3. **Better caching** - Fewer file changes = more cache hits
4. **Security** - Prevents accidentally including secrets

**Why this matters:** Without .dockerignore, `COPY . .` copies EVERYTHING, including huge `node_modules/` directories. This slows builds and wastes space since we're reinstalling dependencies anyway!
</details>

<details>
<summary>Question 3: When does Docker invalidate a layer's cache?</summary>

**Answer:** Docker invalidates cache when:

**1. Instruction text changes:**
```dockerfile
# Cache invalidated if you change the command
RUN npm install         # Cached
RUN npm install --production  # Cache MISS - command changed
```

**2. Files being copied change:**
```dockerfile
COPY package.json .    # Cache invalidated if package.json content changes
```

**3. Base image changes:**
```dockerfile
FROM node:18-alpine    # Cache invalidated if this image updated
```

**4. Any previous layer is invalidated:**
```dockerfile
FROM node:18-alpine    # Layer 1
WORKDIR /app          # Layer 2
COPY package.json .   # Layer 3 - IF THIS INVALIDATES...
RUN npm install       # Layer 4 - ...THIS REBUILDS
COPY . .             # Layer 5 - ...AND THIS REBUILDS
```

**Rule:** When one layer is invalidated, all subsequent layers are invalidated too!

**Why this matters:** Understanding cache invalidation helps you write Dockerfiles that rebuild as little as possible.
</details>

<details>
<summary>Question 4: What are multi-stage builds good for?</summary>

**Answer:** **Separating build dependencies from runtime dependencies** to create smaller final images.

**The problem:**
```dockerfile
FROM node:18          # Includes npm, build tools, dev dependencies
RUN npm install       # Installs EVERYTHING
COPY . .
RUN npm run build     # Build step
CMD ["npm", "start"]
```
Final image includes npm, dev dependencies, build tools - not needed in production!

**Multi-stage solution:**
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
RUN npm install       # All dependencies
RUN npm run build     # Build the app

# Stage 2: Production
FROM node:18-alpine
RUN npm install --production  # Only production dependencies
COPY --from=builder /app/dist ./dist  # Copy built code only
CMD ["node", "dist/index.js"]
```
Final image only has production dependencies and built code!

**Best use cases:**
- **Compiled languages** (Go, Rust, Java): Build with compiler, run with minimal runtime
  - Example: 1GB build image → 20MB runtime image
- **Frontend builds** (React, Vue): Build with Node/webpack, serve with nginx
  - Example: 500MB build → 30MB nginx + static files
- **Any app with build steps**: Separate build tools from runtime

**Why this matters:** Smaller images = faster deployments, less attack surface, lower costs.
</details>

## Implementation Check

Let's verify your optimizations work!

**Test 1: Verify layer caching works**

```bash
# First build (fresh)
time docker build -t my-todo-app .
# Note the time

# Make a small code change (doesn't affect dependencies)
echo "// optimization test" >> src/index.js

# Rebuild
time docker build -t my-todo-app .
# Should be MUCH faster!
```

**Expected output on second build:**
```
=> CACHED [3/6] COPY package.json package-lock.json ./
=> CACHED [4/6] RUN npm install
=> [5/6] COPY . .        # Only this layer rebuilds!
```

Look for `CACHED` on the `npm install` step!

**Test 2: Verify .dockerignore works**

```bash
# Create .dockerignore if you haven't
cat > .dockerignore << EOF
node_modules
.git
*.log
EOF

# Check build context size
docker build -t my-todo-app . 2>&1 | grep "transferring context"
```

**Expected:** Should show smaller context size (a few KB or MB, not hundreds of MB).

**Test 3: Compare build times**

```bash
# Without cache (clean build)
docker build --no-cache -t my-todo-app .
# Note: Takes 30-60 seconds

# With cache (code change only)
echo "// test" >> src/index.js
docker build -t my-todo-app .
# Note: Takes 2-5 seconds!
```

**Cache should make rebuilds 10-20x faster!**

## Common Issues

### Issue 1: Cache not being used (always rebuilding)

**Symptom:**
Every build shows no `CACHED` layers, even when nothing changed.

**Causes and solutions:**

**Cause 1: Using --no-cache flag**
```bash
# Wrong: Forces rebuild
docker build --no-cache -t my-app .

# Right: Allow caching
docker build -t my-app .
```

**Cause 2: Files in COPY are always changing**
```bash
# Check what files changed
git status
```
Files like `.env`, logs, or temp files might be changing. Add them to `.dockerignore`.

**Cause 3: Timestamps changing**
```dockerfile
# Wrong: Timestamp always changes
RUN echo "Built at $(date)" > build-time.txt

# Right: Don't include timestamps
```

**Cause 4: Different build context**
```bash
# If you run build from different directories, context differs
cd /path/to/app && docker build -t my-app .  # Always use same path
```

---

### Issue 2: .dockerignore not working

**Symptom:**
Files listed in `.dockerignore` are still being copied into the image.

**Solutions:**

**Step 1: Check filename**
```bash
# Must be exactly this name (with dot prefix)
ls -la .dockerignore
```

**Step 2: Check location**
The `.dockerignore` file must be in the **build context root** (same directory where you run `docker build`).

**Step 3: Check syntax**
```
# Correct .dockerignore:
node_modules
*.log
.git

# Wrong (with comments on same line):
node_modules  # dependencies
```

**Step 4: Verify it's working**
```bash
# Build and check what was copied
docker build -t my-app .

# Inspect the image
docker run --rm my-app ls -la
# node_modules shouldn't appear here if dockerignored
```

---

### Issue 3: Slow rebuilds despite optimization

**Symptom:**
Even with optimized Dockerfile, rebuilds are slow.

**Diagnostic steps:**

**Check 1: Is npm install running every time?**
```bash
docker build -t my-app .
# Look for "CACHED" on RUN npm install line
```

If not cached, dependencies are changing. Check:
```bash
git diff package.json package-lock.json
```

**Check 2: Large files being copied?**
```bash
# Check build context size
docker build -t my-app . 2>&1 | head -20
# Should see "transferring context: XX"
```

If context is huge (100s of MB), update `.dockerignore`.

**Check 3: Network slow?**
Base image downloads can be slow. First build is always slower.

**Optimization tips:**
```dockerfile
# Use smaller base images
FROM node:18-alpine  # ~170MB
# instead of
FROM node:18         # ~900MB

# Combine RUN commands to reduce layers
RUN npm install && \
    npm cache clean --force
```

---

### Issue 4: Image size too large

**Symptom:**
`docker images` shows your image is very large (500MB+ for a simple app).

**Solutions:**

**1. Use alpine base images:**
```dockerfile
# Large:
FROM node:18           # ~900MB

# Small:
FROM node:18-alpine    # ~170MB
```

**2. Don't include dev dependencies:**
```dockerfile
# Install only production dependencies
RUN npm install --production

# Or use npm ci for clean install
RUN npm ci --only=production
```

**3. Clean up in same layer:**
```dockerfile
# Bad: Two layers
RUN npm install
RUN npm cache clean --force

# Good: One layer
RUN npm install && npm cache clean --force
```

**4. Use multi-stage builds:**
```dockerfile
# Build stage
FROM node:18 AS builder
RUN npm install
RUN npm run build

# Production stage
FROM node:18-alpine
COPY --from=builder /app/dist ./dist
# Only the built code, no build tools!
```

**5. Check for large files:**
```bash
# See what's in your image
docker run --rm my-app du -sh /*
```

---

## Testing Checklist

Verify your optimizations:

**Layer caching:**
- [ ] First build completes successfully
- [ ] Making code change and rebuilding shows `CACHED` on `npm install`
- [ ] Second build is significantly faster (10x or more)

**.dockerignore:**
- [ ] `.dockerignore` file exists in project root
- [ ] Contains `node_modules`, `.git`, `*.log` at minimum
- [ ] Build context size is reasonable (shown during build)

**Build speed:**
- [ ] Clean build: 30-60 seconds
- [ ] Cached rebuild (code change): 2-5 seconds
- [ ] Understand why it's faster

**Understanding:**
- [ ] Can explain how layer caching works
- [ ] Know when cache is invalidated
- [ ] Understand benefit of proper layer ordering

## Real-World Impact

**Example from a real project:**

**Before optimization:**
- 30 dependencies
- Every code change → rebuild everything
- 3 minutes per build
- Developers rebuild 20 times/day
- **1 hour wasted per developer per day!**

**After optimization:**
- Proper layer ordering
- .dockerignore configured
- Code changes → 5 second rebuilds
- Dependencies changes → 3 minute rebuilds (rare)
- **Saved 55 minutes per developer per day!**

**On a team of 10 developers: 550 minutes (9+ hours) saved DAILY!**

This is why optimization matters!

## Next Steps

Excellent! You now understand:
- ✅ How layer caching works
- ✅ Why layer ordering matters
- ✅ How to use .dockerignore
- ✅ When to use multi-stage builds
- ✅ How to optimize for faster development

**Ready for Lesson 3?** We'll cover:
- Publishing ports for network access
- Persisting data with volumes
- Running multi-container applications with Docker Compose

Continue when ready!
