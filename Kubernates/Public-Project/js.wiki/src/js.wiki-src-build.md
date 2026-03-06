# Reference

- https://docs.requarks.io/dev
- 



# Download and build

You want to know how to download the Wiki.js source code and build it from scratch. I'll provide a step-by-step guide with detailed commands, compatible with Ubuntu (your current environment), and cover all prerequisites, build processes, and verification steps.

### Prerequisites (Must Install First)
Wiki.js is built with **Node.js**, **Git**, **Yarn**, and **Python/build-essential** (for native dependencies). Install all required tools first:
```bash
# 1. Update system packages
sudo apt update && sudo apt upgrade -y

# 2. Install core dependencies
sudo apt install -y git curl build-essential python3

# 3. Install Node.js (LTS version, required by Wiki.js)
# Add NodeSource repository (Node.js 20 LTS)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# 4. Verify Node.js/NPM version (Node ≥18, NPM ≥9)
node -v  # Should output v20.x.x
npm -v   # Should output ≥9.x.x

# 5. Install Yarn (official package manager for Wiki.js)
npm install -g yarn
yarn -v  # Should output ≥1.22.x
```

---

### Step 1: Download (Clone) Wiki.js Source Code
```bash
# 1. Create a working directory (e.g., ~/wiki-js-source)
mkdir -p ~/wiki-js-source
cd ~/wiki-js-source

# 2. Clone the official Wiki.js repository from GitHub
git clone https://github.com/Requarks/wiki.git
cd wiki

# 3. (Optional) Checkout a stable release (recommended instead of main branch)
# List all stable releases: git tag -l | grep -E '^2\.'
git checkout 2.5.300  # Replace with the latest stable version (check GitHub releases)
```

---

### Step 2: Build Wiki.js from Source
The build process has two core parts: **install dependencies** and **compile the code** (frontend + backend).

#### 2.1 Install Dependencies
```bash
# Install all project dependencies (frontend + backend)
yarn install --frozen-lockfile
# --frozen-lockfile: Use exact versions from lockfile (avoids dependency conflicts)
```
- This step may take 5-10 minutes (depends on network/specs)
- If you get `node-gyp` errors: Ensure `python3` and `build-essential` are installed (prerequisites step)

#### 2.2 Build the Project
```bash
# Build both frontend (UI) and backend (API)
yarn build

# Alternative: Build separately (if full build fails)
yarn build:server  # Backend only
yarn build:client  # Frontend only
```
- Successful build output will show:
  ```
  ✨ Built in Xs
  dist/ directory created with compiled files
  ```
- All compiled files are output to the `dist/` folder (this is the production-ready build).

#### 2.3 (Optional) Build Docker Image (If You Want Custom Docker Image)
If you want to build a custom Docker image (instead of using `requarks/wiki:2`), run:
```bash
# Build Docker image (tag as wiki-js:custom)
docker build -t wiki-js:custom .

# Verify the image
docker images | grep wiki-js
```

---

### Step 3: Run the Built Wiki.js (Test)
After building, you can run the compiled version directly (without Docker) to verify:
```bash
# 1. Navigate to the build directory
cd ~/wiki-js-source/wiki/dist

# 2. Copy example config and modify it (match your PostgreSQL setup)
cp config.sample.yml config.yml
nano config.yml  # Edit database settings (same as your Docker config):
# DB_TYPE: postgres
# DB_HOST: 127.0.0.1
# DB_PORT: 5432
# DB_USER: wikiuser
# DB_PASS: Happy-Tony01
# DB_NAME: wikidb

# 3. Start Wiki.js (production mode)
node server.js
```
- If successful, you’ll see the log: `Wiki.js has started on port 3000`
- Access `http://your-server-ip:3000` in your browser to confirm it works.

---

### Step 4: Deploy the Built Version (Optional)
#### Option A: Run as a System Service (Non-Docker)
Create a systemd service to run Wiki.js permanently:
```bash
# 1. Create service file
sudo nano /etc/systemd/system/wiki-js.service

# 2. Paste this content (update paths to match your setup):
[Unit]
Description=Wiki.js
After=network.target postgresql.service

[Service]
Type=simple
User=tony
WorkingDirectory=/home/tony/wiki-js-source/wiki/dist
ExecStart=/usr/bin/node server.js
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

# 3. Reload systemd and start the service
sudo systemctl daemon-reload
sudo systemctl start wiki-js
sudo systemctl enable wiki-js  # Auto-start on boot

# 4. Verify service status
sudo systemctl status wiki-js
```

#### Option B: Use Your Custom Docker Image
If you built a custom Docker image (Step 2.3), run it with your existing config:
```bash
docker run -d \
  --name wiki-custom \
  --network="host" \
  -e "DB_TYPE=postgres" \
  -e "DB_HOST=127.0.0.1" \
  -e "DB_PORT=5432" \
  -e "DB_USER=wikiuser" \
  -e "DB_PASS=Happy-Tony01" \
  -e "DB_NAME=wikidb" \
  -v /home/tony/ws/github/dev-helper-project/wiki-data:/wiki/data \
  wiki-js:custom
```

---

### Common Build Issues & Fixes
1. **Out of memory during build**:
   ```bash
   # Increase Node.js memory limit
   export NODE_OPTIONS=--max_old_space_size=4096
   yarn build
   ```
2. **Yarn lockfile errors**:
   ```bash
   yarn cache clean
   yarn install --force
   ```
3. **PostgreSQL connection errors**:
   - Ensure PostgreSQL is running (`sudo systemctl start postgresql`)
   - Verify `wikiuser` has access to `wikidb` (run `psql -U wikiuser -d wikidb` to test)

---

### Summary
1. **Prerequisites**: Install Node.js (20 LTS), Yarn, Git, and build tools first.
2. **Download Source**: Clone the Wiki.js repo and checkout a stable release (avoid main branch).
3. **Build**: Run `yarn install --frozen-lockfile` → `yarn build` (output to `dist/`).
4. **Run/Deploy**: Test with `node server.js`, then deploy as a system service or custom Docker image.
5. **Key Notes**: Always use stable releases (not main branch) for production; match database config in `config.yml`.

The core takeaways are:  
- Wiki.js requires Node.js LTS (≥18) for building  
- The `dist/` folder contains the production-ready build  
- Custom Docker images let you modify the source code before deployment (e.g., add plugins/custom features).

