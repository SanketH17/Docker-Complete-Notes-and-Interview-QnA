# 🐳 Docker — Complete Beginner's Notes
### *A Beginner-Friendly, End-to-End Docker Guide*

---

## 📋 Table of Contents

1. [Why Do We Need Docker?](#1-why-do-we-need-docker)
2. [What is Docker?](#2-what-is-docker)
3. [Docker Containers](#3-docker-containers)
4. [Docker Images](#4-docker-images)
5. [Docker Image vs Docker Container — Key Differences](#5-docker-image-vs-docker-container--key-differences)
6. [Setting Up Docker](#6-setting-up-docker)
7. [Docker Hub](#7-docker-hub)
8. [Essential Docker Commands](#8-essential-docker-commands)
9. [Image Versions and Tags](#9-image-versions-and-tags)
10. [Image Layers — How Layering Works](#10-image-layers--how-layering-works)
11. [Port Binding](#11-port-binding)
12. [Troubleshooting Docker Containers](#12-troubleshooting-docker-containers)
13. [Docker vs Virtual Machine](#13-docker-vs-virtual-machine)
14. [Docker Networking](#14-docker-networking)
15. [Practical Project: Node.js App + MongoDB + MongoExpress](#15-practical-project-nodejs-app--mongodb--mongoexpress)
16. [Docker Compose](#16-docker-compose)
17. [Dockerizing Your Own Application (Dockerfile)](#17-dockerizing-your-own-application-dockerfile)
18. [Building and Running Your Custom Docker Image](#18-building-and-running-your-custom-docker-image)
19. [Publishing Your Image to Docker Hub](#19-publishing-your-image-to-docker-hub)
20. [Docker Volumes — Data Persistence](#20-docker-volumes--data-persistence)
21. [Advanced Docker Networking](#21-advanced-docker-networking)
22. [Quick Reference: All Docker Commands](#22-quick-reference-all-docker-commands)

---

## 1. Why Do We Need Docker?

### The Classic Problem: "It Works on My Machine"

Imagine you're working on a Node.js application with your team. On your laptop you have:
- **Node.js v16**
- **MongoDB v4.2**

A new team member joins and sets up the same project on their Mac. But when they install dependencies, they accidentally install:
- **Node.js v20** (newer version)
- **MongoDB v6** (newer version)

Now the application behaves differently on their machine. Bugs appear that didn't exist before. This is the famous **"It works on my machine"** problem.

### Why does this happen?

When we manually replicate an environment on a new machine, several things can go wrong:

| Problem | Explanation |
|---|---|
| **Version mismatch** | A new team member installs a newer version of Node/MongoDB by accident |
| **Missing dependencies** | Real-world apps have dozens of dependencies — manually installing all of them is error-prone |
| **OS-specific commands** | Some CLI commands work on Mac/Linux but fail on Windows |
| **Inconsistent setup** | Each developer ends up with a slightly different environment |

This problem gets **worse as teams grow**. With 2-4 people it's manageable, but with 20+ developers all working on the same project, it becomes a serious bottleneck.

> 💡 **Docker was built specifically to solve this problem** — by packaging your application together with ALL its dependencies into one portable unit.

---

## 2. What is Docker?

In the simplest terms:

> **Docker is a platform that helps you build, ship, and run containers.**

Think of Docker like a **shipping container** on a cargo ship. No matter what's inside (furniture, electronics, food), the container is always the same standard size and shape. Ships, cranes, and trucks can handle any container — they don't care what's inside.

Similarly, Docker wraps your application and all its dependencies into a **standardized container** that can run anywhere — on your laptop, a colleague's Mac, a Linux server, or a cloud machine — without any manual setup.

To understand Docker fully, you need to understand two key terms:
1. **Docker Container**
2. **Docker Image**

---

## 3. Docker Containers

### What is a Container?

A **Docker Container** is a single, self-contained unit (a bundle) that packages:
- Your **application code**
- All its **dependencies** (libraries, frameworks, runtime)
- The **configuration** needed to run it

> 🎒 **Analogy:** Think of a container like a packed travel bag. Instead of separately carrying your clothes, charger, toiletries and medicine every time you travel, you pack them all into one bag. Now you can take that same bag anywhere — hotel, hostel, friend's house — and everything you need is right there.

### Properties of Docker Containers

**1. Portable**
A container can be shared across different machines and operating systems. You don't need to worry about whether the receiving machine is Windows, Mac, or Linux — the container works the same everywhere.

**2. Lightweight**
Containers are very small in size compared to traditional virtual machines. Creating, updating, or deleting them is fast and easy. You can run multiple containers on a single machine without performance issues.

**3. Isolated Environment**
Each container runs in its own isolated environment, completely separate from:
- The host machine (your laptop/server)
- Other containers running on the same machine

> 🏠 **Analogy:** Think of containers like separate apartments in a building. Each apartment has its own kitchen, bathroom, electricity — fully independent. What happens in one apartment doesn't affect the others, and none of them affect the building's foundation (the host machine).

**4. Multiple Versions on the Same Machine**
Without Docker, if your machine has Node.js v16 installed, *both* of your apps will use v16.

With Docker, you can run:
- **App 1** in Container 1 → Uses Node.js **v16**
- **App 2** in Container 2 → Uses Node.js **v20**

Both on the *same machine*, simultaneously. No conflicts.

---

## 4. Docker Images

### What is a Docker Image?

A **Docker Image** is NOT a picture or photo. It is:

> **An executable file (a blueprint) that contains all the instructions needed to build a Docker Container.**

### The Class and Object Analogy

The relationship between a Docker Image and a Docker Container is exactly like the relationship between a **Class** and an **Object** in Object-Oriented Programming:

| OOP Concept | Docker Equivalent |
|---|---|
| **Class** | **Docker Image** |
| **Object (instance)** | **Docker Container** |
| A class is a blueprint | An image is a blueprint |
| You create multiple objects from one class | You create multiple containers from one image |
| Objects use system memory/resources | Containers use CPU/RAM/disk resources |
| The class file itself doesn't consume RAM | The image file itself is just stored on disk (small size) |

```
One Docker Image  ──►  Container 1
                  ──►  Container 2
                  ──►  Container 3
```

### How Image Sharing Works

When sharing your application with teammates, you **don't share the container** — you **share the Docker Image**.

Each team member:
1. Receives the Docker Image
2. Runs it on their local machine
3. A new container is automatically created from that image

This ensures everyone gets the **exact same environment**, no matter what OS they're on.

---

## 5. Docker Image vs Docker Container — Key Differences

| Feature | Docker Image | Docker Container |
|---|---|---|
| **What it is** | A static blueprint / snapshot | A running instance |
| **State** | Immutable (read-only) | Active and running |
| **Resource usage** | Minimal — just disk space | Uses CPU, RAM, network |
| **Analogy** | A cookie cutter | The actual cookie |
| **Size** | Small (can be in KBs or MBs) | Larger (running environment) |
| **Shared with team?** | ✅ Yes — you share images | ❌ No — containers run locally |

---

## 6. Setting Up Docker

### Step 1: Download Docker Desktop

Visit **[docker.com](https://docker.com)** and download **Docker Desktop** for your operating system:
- **Mac** → Choose Intel chip or Apple Silicon (M1/M2/M3)
- **Windows** → Choose Windows AMD64
- **Linux** → Use the appropriate package

### Step 2: Install Docker Desktop

**On Windows:**
1. Run the installer
2. Click **OK** when asked for admin permissions
3. Click **Close and Restart** when installation finishes
4. After restart, accept the license agreement
5. Choose **Recommended Settings** → Click **Finish**
6. You can skip the sign-in step

**On Mac:**
1. Open the `.dmg` file
2. Drag Docker to Applications
3. Open Docker from Applications
4. Accept the license agreement
5. Choose Recommended Settings → Finish

### Step 3: Verify Installation

Open your **Terminal** (Mac/Linux) or **Command Prompt / PowerShell** (Windows) and run:

```bash
docker --version
```

**Expected output:**
```
Docker version 27.5.1, build ...
```

To see all available Docker commands:
```bash
docker
```

If you see a list of commands, Docker is successfully installed! ✅

### Docker Desktop Interface

After opening Docker Desktop, you'll see:
- **Containers** tab — shows all your running and stopped containers
- **Images** tab — shows all Docker images downloaded on your machine
- **Volumes** tab — shows all your Docker volumes
- **Networks** (under settings) — shows Docker networks

---

## 7. Docker Hub

### What is Docker Hub?

**Docker Hub** ([hub.docker.com](https://hub.docker.com)) is to Docker images what **GitHub** is to code.

| GitHub | Docker Hub |
|---|---|
| Stores code repositories | Stores Docker image repositories |
| You push/pull code | You push/pull Docker images |
| Public and private repos | Public and private repos |
| Anyone can share code | Anyone can share Docker images |

### What You'll Find on Docker Hub

- **Official images** for popular technologies: MySQL, MongoDB, Node.js, Redis, Nginx, Ubuntu, Python, etc.
- Each image comes with:
  - Different **tags** (versions)
  - **Documentation** on how to use it
  - List of required **environment variables**
  - Example commands

---

## 8. Essential Docker Commands

Let's learn all the important Docker commands. We'll start simple and build up.

---

### 8.1 `docker pull` — Download an Image

Downloads (pulls) a Docker image from Docker Hub to your local machine.

```bash
docker pull <image-name>
```

**Example:**
```bash
docker pull hello-world
```

**What happens:**
1. Docker checks if the image exists locally
2. If not, it downloads it from Docker Hub
3. The latest version (tag) is pulled by default

**Output:**
```
Using default tag: latest
latest: Pulling from library/hello-world
...
Status: Downloaded newer image for hello-world:latest
```

---

### 8.2 `docker images` — List Downloaded Images

Shows all Docker images currently available on your local machine.

```bash
docker images
```

**Output:**
```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d2c94e258dcb   ...            13.3kB
ubuntu        latest    3b418d7b466a   ...            77.9MB
```

> 💡 Notice how tiny images are — `hello-world` is only 13KB! This is why Docker is called **lightweight**.

---

### 8.3 `docker run` — Create and Start a Container

Creates a **new container** from an image and starts it. This is the most commonly used Docker command.

```bash
docker run <image-name>
```

**Example:**
```bash
docker run hello-world
```

> ⚠️ **Key difference between `docker pull` and `docker run`:**
> - `docker pull` → Only downloads the image
> - `docker run` → Downloads the image (if not already present) AND creates + starts a new container from it

**Important flags for `docker run`:**

#### `-d` — Detach Mode (Background)

Run the container in the background, so your terminal isn't occupied.

```bash
docker run -d <image-name>
```

Without `-d`, the container runs in the foreground (your terminal is "attached" to it).

#### `-it` — Interactive Mode

Opens the container's terminal so you can type commands inside it.

```bash
docker run -it ubuntu
```

**What happens:** You enter the Ubuntu container's terminal. Your prompt changes to:
```
root@abc123def456:/#
```

That alphanumeric code (`abc123def456`) is your **container's ID**. You're now "inside" the container! You can run:
```bash
ls           # list files inside container
mkdir test   # create a folder inside container
env          # see environment variables
exit         # exit the container
```

When you type `exit`, the container **stops automatically**.

#### `--name` — Give a Custom Name

By default, Docker assigns random funny names like `happy_einstein`. You can set your own name:

```bash
docker run --name my-container-name <image-name>
```

#### `-e` — Set Environment Variables

Some containers require environment variables to be set (like database passwords):

```bash
docker run -e MY_VARIABLE=value <image-name>
```

Multiple environment variables:
```bash
docker run -e VAR1=value1 -e VAR2=value2 <image-name>
```

#### `-p` — Port Binding (covered in detail in Section 11)

```bash
docker run -p <host-port>:<container-port> <image-name>
```

---

### 8.4 `docker ps` — List Running Containers

Shows all **currently running** containers:

```bash
docker ps
```

**Output:**
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS   PORTS   NAMES
```
(Empty if no containers are running)

### `docker ps -a` — List ALL Containers (including stopped)

```bash
docker ps -a
```

Shows running + stopped containers. Every container you've ever created appears here.

---

### 8.5 `docker start` — Start an Existing Container

`docker run` creates a **brand new** container every time. If you want to **restart an existing stopped container**, use `docker start`:

```bash
docker start <container-name-or-id>
```

**Example:**
```bash
docker start my-ubuntu
```

> 🔁 Think of it like this:
> - `docker run` = Buying a new car and driving it
> - `docker start` = Starting your existing parked car

---

### 8.6 `docker stop` — Stop a Running Container

Gracefully stops a running container:

```bash
docker stop <container-name-or-id>
```

**Example:**
```bash
docker stop my-ubuntu
```

The container moves to "Exited" status — it still exists, just not running.

---

### 8.7 `docker rm` — Remove/Delete a Container

Permanently deletes a stopped container:

```bash
docker rm <container-name-or-id>
```

> ⚠️ You cannot remove a running container. Stop it first with `docker stop`, then remove it.

---

### 8.8 `docker rmi` — Remove/Delete an Image

Permanently deletes a Docker image:

```bash
docker rmi <image-name-or-id>
```

**Example:**
```bash
docker rmi hello-world
```

> ⚠️ If a container (even a stopped one) was created from this image, you must **delete the container first** before deleting the image. Otherwise you'll get:
>
> ```
> Error: image is being used by stopped container abc123
> ```

**Correct order to clean up:**
```bash
# Step 1: Remove the container
docker rm <container-id>

# Step 2: Now remove the image
docker rmi <image-name>
```

---

### Summary of Basic Commands

| Command | Purpose |
|---|---|
| `docker pull <image>` | Download an image from Docker Hub |
| `docker images` | List all local images |
| `docker run <image>` | Create + start a new container |
| `docker run -d <image>` | Run container in background |
| `docker run -it <image>` | Run container in interactive/terminal mode |
| `docker run --name <n> <image>` | Run with a custom name |
| `docker run -e KEY=VALUE <image>` | Set an environment variable |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (running + stopped) |
| `docker start <name/id>` | Start an existing stopped container |
| `docker stop <name/id>` | Stop a running container |
| `docker rm <name/id>` | Delete a container |
| `docker rmi <name/id>` | Delete an image |

---

## 9. Image Versions and Tags

### What Are Tags?

Every Docker image can have multiple **tags**. A tag is basically a **version label** for an image.

Think of tags like software versions:
- `mysql:latest` → The latest MySQL version
- `mysql:8.0` → MySQL version 8.0 specifically
- `node:20` → Node.js version 20
- `node:16-alpine` → Node.js v16 on a minimal Alpine Linux base

### Pulling a Specific Tag/Version

```bash
# Pull the latest version (default)
docker pull mysql

# Pull a specific version
docker pull mysql:8.0
```

**Why this matters:** You can now have two different versions of MySQL running as separate containers on the same machine — something impossible before Docker!

```bash
# Container 1 with latest MySQL
docker run -d --name mysql-latest mysql:latest

# Container 2 with MySQL 8.0
docker run -d --name mysql-old mysql:8.0
```

Both run simultaneously without any conflict.

---

## 10. Image Layers — How Layering Works

### The Concept

Every Docker image is made up of **multiple stacked layers**, like a multi-layered cake.

```
┌─────────────────────────────────┐
│    Your Application Layer        │  ← Your code
├─────────────────────────────────┤
│    Node.js Layer                 │  ← Node runtime
├─────────────────────────────────┤
│    Debian Linux Layer            │  ← OS base
└─────────────────────────────────┘
```

- Each layer is **read-only** (immutable)
- The topmost layer (container layer) is the only **writable** one
- When you modify files inside a container, only the writable container layer changes

### Why Layers Are Smart — Caching

When you pull `mysql:latest` and then `mysql:8.0`, Docker is smart enough to **not re-download layers that are common** between the two versions.

**Example output when pulling the second version:**
```
Layer 1: Already exists  ← Shared, already downloaded
Layer 2: Already exists  ← Shared, already downloaded
Layer 3: Pull complete   ← Different, needs downloading
Layer 4: Pull complete   ← Different, needs downloading
```

This makes Docker very efficient with storage and download bandwidth.

---

## 11. Port Binding

### The Problem

Every container has its **own isolated network with its own ports**. These are completely separate from your host machine's ports.

> 🏢 **Analogy:** Imagine an office building (your computer). Each container is a separate office on a different floor. Each office has its own internal phone extension system (container ports). But to call from outside the building (your browser), you use the main building's phone lines (host machine ports).

When MySQL runs inside a container, it listens on **port 3306** — but that's the *container's* port 3306, not your laptop's port 3306.

### How Port Binding Works

To access a container's service from your browser or from another application, you need to **map (bind) a host machine port to a container port**.

```
Browser  →  localhost:8080  →  [HOST MACHINE]  →  Container:3306
```

**The `-p` flag syntax:**
```bash
docker run -p <host-port>:<container-port> <image-name>
```

**Example — Binding host port 8080 to MySQL's container port 3306:**
```bash
docker run -d -p 8080:3306 -e MYSQL_ROOT_PASSWORD=secret mysql
```

Now, when you access `localhost:8080` on your machine, it forwards to port 3306 inside the MySQL container.

### Important Rule: Each Host Port Can Only Be Used Once

If port 8080 on your machine is already bound to Container 1, you **cannot** bind it to Container 2 as well.

**This will fail:**
```bash
docker run -p 8080:3306 --name container-1 mysql:latest   # Works ✅
docker run -p 8080:3306 --name container-2 mysql:8.0       # ERROR ❌
# Error: bind: address already in use
```

**Solution:** Use a different host port for the second container:
```bash
docker run -p 5000:3306 --name container-2 mysql:8.0      # Works ✅
```

The container ports can be the same (both using 3306 internally), but the **host machine ports must be unique**.

---

## 12. Troubleshooting Docker Containers

### 12.1 `docker logs` — View Container Logs

When something goes wrong inside a container, check its logs:

```bash
docker logs <container-name-or-id>
```

**Example:**
```bash
docker logs mysql-latest
```

**Output shows** things like:
```
[System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections.
```

You can also view logs in Docker Desktop by clicking on the container name.

---

### 12.2 `docker exec -it` — Enter a Running Container's Terminal

If a container is **already running** and you want to go inside it and run commands, use `exec`:

```bash
docker exec -it <container-name> /bin/bash
```

Or for containers without `bash`:
```bash
docker exec -it <container-name> /bin/sh
```

**Example:**
```bash
docker exec -it mysql-latest /bin/bash
```

**Inside the container you can:**
```bash
ls                    # list files
env                   # check environment variables
cat /etc/os-release   # check the OS
exit                  # leave the container (container keeps running!)
```

> 🔑 **Key difference from `-it` with `docker run`:**
> - `docker run -it` → Creates a new container AND enters it. Exiting STOPS the container.
> - `docker exec -it` → Enters an ALREADY RUNNING container. Exiting does NOT stop the container.

---

## 13. Docker vs Virtual Machine

Both Docker containers and Virtual Machines (VMs) let you run isolated environments. But they work very differently:

### How an OS Works

Every computer has:
1. **Kernel** — The core of the OS that talks directly to hardware
2. **Application Layer** — Where all your apps run (browser, terminal, etc.)

### Virtual Machine Approach

A VM **virtualizes both the Kernel AND the Application Layer**.

```
HOST OS (Kernel + App Layer)
    └── VM (Has its OWN Kernel + App Layer)
            └── Application inside VM
```

**Result:** VMs are large (can be several GBs), slow to start (minutes), but are compatible with any host OS.

### Docker's Approach

Docker only **virtualizes the Application Layer**, and shares the host machine's Kernel.

```
HOST OS (Kernel + App Layer)
    └── Docker (Virtualizes App Layer only, shares Host Kernel)
            └── Application inside Container
```

**Result:** Containers are tiny (MBs), start in seconds, but depend on the host OS's Kernel.

### Comparison Table

| Feature | Docker | Virtual Machine |
|---|---|---|
| **Size** | MBs (lightweight) | GBs (heavy) |
| **Startup time** | Seconds | Minutes |
| **Performance** | Near-native | Overhead from virtualization |
| **Isolation** | Application layer only | Full OS isolation |
| **OS Compatibility** | Depends on host kernel | Can run any OS |
| **Resource usage** | Low | High |

### Docker's Limitation

Since Docker relies on the host Kernel, historically it was built for Linux. However, Docker Desktop on Mac and Windows uses a lightweight Linux virtual machine internally, so this limitation is transparent to users in modern setups.

---

## 14. Docker Networking

### The Problem

When you have multiple containers running (e.g., a Node.js app container + a MongoDB container), they need to talk to each other. But by default, containers are isolated — they can't communicate freely.

### Docker Networks

A **Docker Network** is an isolated network environment where you can place multiple containers so they can **communicate with each other directly** — without needing to expose ports.

> 🏘️ **Analogy:** Think of a Docker Network like a private gated community. All houses inside the community can visit each other freely without going through the main gate. But people outside can only come in through the gate (port binding).

### Viewing All Networks

```bash
docker network ls
```

**Output:**
```
NETWORK ID     NAME      DRIVER    SCOPE
a1b2c3d4e5f6   bridge    bridge    local
f6e5d4c3b2a1   host      host      local
e5d4c3b2a1f6   none      null      local
```

Docker creates three default networks:
- `bridge` — Default network for containers
- `host` — Container uses the host's network directly
- `none` — No networking (fully isolated)

### Creating a Custom Network

```bash
docker network create <network-name>
```

**Example:**
```bash
docker network create mongo-network
```

### Connecting a Container to a Network

Use the `--network` flag when running a container:

```bash
docker run -d --name my-container --network mongo-network <image-name>
```

All containers placed in the **same custom network** can communicate with each other by using their **container names** as hostnames — no IP addresses needed.

---

## 15. Practical Project: Node.js App + MongoDB + MongoExpress

This real-world example brings everything together.

### Project Overview

We have a simple **Node.js web application** (a university registration page) that:
- Lets users create accounts (stored in MongoDB)
- Fetches user data from MongoDB
- Runs on **port 5050**

Instead of installing MongoDB locally, we'll run it inside a Docker container. We'll also run **MongoExpress** — a visual browser-based GUI for MongoDB.

**Architecture:**
```
Browser
  │
  ▼
Node.js App (running locally on port 5050)
  │
  ▼
Docker Environment
  ├── MongoDB Container (port 27017) ── mongo-network
  └── MongoExpress Container (port 8081) ── mongo-network
```

### Step 1: Create a Custom Network

```bash
docker network create mongo-network
```

Both MongoDB and MongoExpress will be placed in this network so they can talk to each other.

### Step 2: Run the MongoDB Container

```bash
docker run -d \
  -p 27017:27017 \
  --name mongo \
  --network mongo-network \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=qwerty \
  mongo
```

**What each part does:**
- `-d` → Run in background (detach mode)
- `-p 27017:27017` → Bind host port 27017 to container port 27017
- `--name mongo` → Give the container the name "mongo"
- `--network mongo-network` → Place it in our custom network
- `-e MONGO_INITDB_ROOT_USERNAME=admin` → Set admin username
- `-e MONGO_INITDB_ROOT_PASSWORD=qwerty` → Set admin password
- `mongo` → The image name to use

### Step 3: Run the MongoExpress Container

MongoExpress is a web-based UI to visually manage your MongoDB database.

```bash
docker run -d \
  -p 8081:8081 \
  --name mongo-express \
  --network mongo-network \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=qwerty \
  -e ME_CONFIG_MONGODB_URL=mongodb://admin:qwerty@mongo:27017/ \
  mongo-express
```

**Key environment variable:** `ME_CONFIG_MONGODB_URL` tells MongoExpress where to find MongoDB. Notice we use `mongo` (the container name) as the hostname — this works because both containers are in the same `mongo-network`.

### Step 4: Access MongoExpress

Open your browser and go to: **http://localhost:8081**

- Default login: `admin` / `pass`

You'll see a beautiful UI showing your MongoDB databases. From here you can:
- Create databases
- Create collections (like SQL tables)
- Add, edit, delete documents

### Step 5: Create the Database and Collection

In MongoExpress:
1. Create a new database: `college-db`
2. Inside it, create a collection: `users`
3. Add a sample document:
   ```json
   {
     "email": "john@example.com",
     "username": "johndoe",
     "password": "secret"
   }
   ```

### Step 6: Test the Node.js App

With the Node.js server running locally:
```bash
node server.js
```

Test the GET endpoint:
```
GET http://localhost:5050/get-users
```

You'll see the document you added in MongoExpress — proof that your Node.js app is successfully communicating with the MongoDB Docker container! ✅

---

## 16. Docker Compose

### The Problem Docker Compose Solves

Look at the MongoDB setup command from Section 15:

```bash
docker run -d -p 27017:27017 --name mongo --network mongo-network \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=qwerty \
  mongo
```

This is just ONE container. In real apps, you might have 5-10 containers. Managing all of them through the terminal is messy and error-prone.

**Docker Compose solves this** by letting you define all your containers in a single structured **YAML file**, then start/stop all of them with one command.

### Benefits of Docker Compose

1. **Structured configuration** — All container settings in one readable file
2. **Easy to edit** — Changing a port or environment variable is simple
3. **One command to rule them all** — Start all containers with `docker-compose up`, stop with `docker-compose down`
4. **Automatic networking** — Docker Compose automatically creates a shared network for all defined containers (no manual `docker network create` needed!)

### YAML File Format

The file is named with a `.yml` extension (e.g., `mongodb.yml`). Here's how our MongoDB + MongoExpress setup looks:

```yaml
version: '3.8'
services:
  mongo:
    image: mongo
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: qwerty

  mongo-express:
    image: mongo-express
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: qwerty
      ME_CONFIG_MONGODB_URL: mongodb://admin:qwerty@mongo:27017/
```

**Breaking it down:**

| Section | Meaning |
|---|---|
| `version` | Docker Compose version (can be omitted in latest Docker versions) |
| `services` | List of all containers you want to create |
| `image` | Which Docker image to use |
| `ports` | Port binding in format `"host:container"` |
| `environment` | Environment variables as key-value pairs |

> 📝 **Indentation matters in YAML!** Use consistent 2-space indentation. Wrong indentation = errors.

### Running Docker Compose

**Start all containers (detach mode):**
```bash
docker-compose -f mongodb.yml up -d
```

**Stop and remove all containers:**
```bash
docker-compose -f mongodb.yml down
```

**What `up` does:**
1. Creates a default shared network automatically
2. Pulls any images not already available locally
3. Creates and starts all containers defined in the file

**What `down` does:**
1. Stops all containers
2. Removes all containers
3. Removes the auto-created network

> 💡 You do **not** need to manually create a network when using Docker Compose. It creates one automatically!

---

## 17. Dockerizing Your Own Application (Dockerfile)

### What is "Dockerizing"?

**Dockerizing** means taking your own application (Node.js, Python, Java, etc.) and packaging it into a Docker image, so it can be run anywhere as a container.

### What is a Dockerfile?

A **Dockerfile** is a plain text file (no extension) named exactly `Dockerfile` that contains step-by-step instructions telling Docker how to build your image.

> 📐 **Analogy:** A Dockerfile is like a recipe card. It tells Docker: "First get these ingredients (base image), then do these steps (install dependencies, copy code), and finally serve it this way (run this command)."

### Key Dockerfile Instructions

#### `FROM` — Set the Base Image

Every image is built on top of another image (called the base image). For a Node.js app, the base image is `node`:

```dockerfile
FROM node:20
```

**Why do we need a base image?**
Your Node.js app won't run unless Node.js is already installed inside the container. The `node` image provides that pre-installed Node.js environment.

This is the **layer concept in practice** — your image layers on top of the Node image, which itself layers on top of a Debian Linux image.

```
Your App
   ↑
Node.js image
   ↑
Debian Linux image
```

#### `ENV` — Set Environment Variables

```dockerfile
ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PASSWORD=qwerty
```

#### `RUN` — Execute Commands During Build

Used to run shell commands while building the image (e.g., install packages, create directories):

```dockerfile
RUN mkdir -p /home/app
```

You can have **multiple** `RUN` instructions in a Dockerfile.

#### `COPY` — Copy Files from Host to Image

```dockerfile
COPY . /home/app
```

- First argument: source path on your host machine (`.` means current directory)
- Second argument: destination path inside the container

#### `CMD` — Default Command to Run the Container

Defines the command that runs when a container starts. There can be **only ONE** `CMD` in a Dockerfile:

```dockerfile
CMD ["node", "/home/app/server.js"]
```

Written as an array of strings.

> 📌 **`RUN` vs `CMD`:**
> - `RUN` → Executes during **image build** time (to set up the environment)
> - `CMD` → Executes when a **container starts** (to run your app)

#### `EXPOSE` — Document the Port

```dockerfile
EXPOSE 5050
```

This documents which port your application listens on (informational — you still need `-p` for actual port binding).

### Complete Dockerfile Example

Here's a full Dockerfile for our Node.js application:

```dockerfile
FROM node:20

# Set environment variables
ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PASSWORD=qwerty

# Create app directory inside the container
RUN mkdir -p /home/app

# Copy all files from current directory to /home/app in the image
COPY . /home/app

# Start the application
CMD ["node", "/home/app/server.js"]
```

**Save this file** as `Dockerfile` (capital D, no extension) in your project root.

---

## 18. Building and Running Your Custom Docker Image

### Build the Image

```bash
docker build -t <image-name>:<tag> .
```

- `-t` → Tag (name) for the image
- `.` → Look for `Dockerfile` in the current directory

**Example:**
```bash
docker build -t my-node-app:1.0 .
```

**What happens during build:**
1. Docker reads the `Dockerfile`
2. Executes each instruction line by line
3. Creates a new layer for each instruction
4. The final image is saved locally

**Verify the image was created:**
```bash
docker images
```

You should see `my-node-app` with tag `1.0` in the list.

### Run a Container from Your Custom Image

```bash
docker run my-node-app:1.0
```

**Output:**
```
Server running on port 5050
```

Your custom application is now running inside a Docker container!

### Run in Interactive Mode to Inspect

```bash
docker run -it my-node-app:1.0 /bin/bash
```

Inside the container:
```bash
ls                    # See your copied files
ls /home/app          # See your app files
env                   # Verify environment variables
```

---

## 19. Publishing Your Image to Docker Hub

### Why Publish?

Once your image is on Docker Hub, anyone on your team can pull it and run your application without needing your source code or manual setup.

### Step 1: Create a Docker Hub Account

Go to **[hub.docker.com](https://hub.docker.com)** → Sign up

### Step 2: Create a Repository

1. Click **Repositories** → **Create Repository**
2. Give it a name: `test-application`
3. Choose **Public** or **Private**
4. Click **Create**

### Step 3: Login from Terminal

**Option A — Username/password:**
```bash
docker login
# Enter username and password when prompted
```

**Option B — Via browser (more secure):**
```bash
docker login
# Docker provides a URL and a one-time code
# Open the URL in your browser, enter the code, confirm
```

### Step 4: Build Image with Your Docker Hub Username

The image name must match your Docker Hub repository path: `<username>/<repo-name>:<tag>`

```bash
docker build -t yourusername/test-application:1.0 .
```

### Step 5: Push the Image

```bash
docker push yourusername/test-application:1.0
```

Docker pushes all image layers to Docker Hub. When done, visit your Docker Hub repository — you'll see your image there!

### Sharing with Your Team

Anyone can now pull and run your application with just:
```bash
docker pull yourusername/test-application:1.0
docker run yourusername/test-application:1.0
```

No source code needed, no manual dependency installation. The whole environment is packaged in the image.

---

## 20. Docker Volumes — Data Persistence

### The Data Loss Problem

By default, all data stored inside a container lives in the container's **virtual filesystem**. When the container stops or is deleted, **all data is lost**.

**Example:** You add 10 users to MongoDB running in a container. You stop the container. You restart it. The 10 users are gone — you're back to a blank database.

This is a big problem for databases!

### What Are Volumes?

**Docker Volumes** are a way to **persist data outside of a container**, so that data survives container restarts and deletions.

> 🗄️ **Analogy:** Think of a volume like an external hard drive that you plug into your container. The container stores its data ON the external drive. Even if you throw away the container, the external drive (volume) and its data remain safe.

### How Volumes Work

```
┌─────────────────────────────────────────┐
│           HOST MACHINE                   │
│                                          │
│  ┌─────────────┐    ┌─────────────────┐ │
│  │  Container   │◄──►│  Volume (Storage)│ │
│  │  (MongoDB)   │    │  /desktop/data  │ │
│  └─────────────┘    └─────────────────┘ │
│                                          │
└─────────────────────────────────────────┘
```

- Data written inside the container is **automatically replicated** to the volume
- If the container is stopped or deleted, data stays in the volume
- When a new container starts, it connects to the same volume and **restores the old data**

### Three Types of Volumes

#### Type 1: Named Volumes (Recommended for Production)

Docker manages the storage location. You just give it a name.

```bash
docker run -v my-volume:/data/db mongo
```

- `my-volume` → Name of the volume (Docker manages where it's stored on the host)
- `/data/db` → Path inside the container where data is stored

Create a named volume explicitly:
```bash
docker volume create my-volume
```

#### Type 2: Anonymous Volumes

Docker manages storage, but no name is given. Used for temporary storage.

```bash
docker run -v /data/db mongo
```

Only the container path is specified. Docker auto-generates a random ID for the volume.

#### Type 3: Bind Mounts

You specify **exactly which folder on your host machine** maps to a path inside the container.

```bash
docker run -v /Users/yourname/Desktop/data:/test/data ubuntu
```

- `/Users/yourname/Desktop/data` → Folder on YOUR host machine
- `/test/data` → Path inside the container

Any files created in `/test/data` inside the container will appear in `/Users/yourname/Desktop/data` on your desktop — in real time!

### Volume in Docker Compose

To add a volume to your `mongodb.yml`:

```yaml
services:
  mongo:
    image: mongo
    ports:
      - "27017:27017"
    volumes:
      - /Users/yourname/Desktop/data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: qwerty
```

Now even if you run `docker-compose down` (which deletes containers), your MongoDB data is safe on your desktop. Next time you run `docker-compose up`, the new MongoDB container picks up exactly where you left off.

### Volume Commands

| Command | Purpose |
|---|---|
| `docker volume ls` | List all volumes |
| `docker volume create <name>` | Create a named volume |
| `docker volume prune` | Delete all unused anonymous volumes |

**View volumes in Docker Desktop:** Click the **Volumes** tab.

### Named vs Anonymous vs Bind Mount — Summary

| Type | Who manages storage? | Survives `docker rm`? | Best for |
|---|---|---|---|
| **Named Volume** | Docker | ✅ Yes | Production databases |
| **Anonymous Volume** | Docker | ❌ Usually not | Temporary data |
| **Bind Mount** | You (host path) | ✅ Yes | Development, local files |

---

## 21. Advanced Docker Networking

### The Three Built-in Network Drivers

When Docker creates a network, it uses a **driver** — a program that defines how containers communicate over that network.

#### Driver 1: Bridge (Default & Most Common)

The default network driver. When you create a container without specifying a network, it joins the default `bridge` network.

**Use case:** Containers on the same host that need to communicate with each other AND with the outside world.

**Two types of bridge networks:**

| | Default Bridge | Custom Bridge |
|---|---|---|
| **How created** | Automatically by Docker | You create it with `docker network create` |
| **Container communication** | By IP address (fragile) | By container name (reliable) |
| **Isolation** | Less isolated | Better isolation |
| **Our MongoDB example** | ❌ Not used | ✅ Used — enables `@mongo:27017` in URL |

> 💡 **Always use custom bridge networks** when you need containers to communicate. Container name resolution (using names instead of IPs) only works in custom bridge networks.

#### Driver 2: Host

The container shares the host machine's network directly. It has **no isolated network** and **no separate IP**.

**Use case:** When you need maximum network performance and don't need isolation (e.g., high-performance applications).

#### Driver 3: None (Null)

The container has **no network access at all**. Completely isolated from everything — other containers, the host, and the internet.

**Use case:** Running highly sensitive workloads that must not have any network access.

### Driver Comparison

| Driver | Container-to-Container | Container-to-Host | Internet Access | Isolation |
|---|---|---|---|---|
| **Bridge** | ✅ Yes | ✅ Via ports | ✅ Yes | High |
| **Host** | ✅ Yes | ✅ Direct | ✅ Yes | None |
| **None** | ❌ No | ❌ No | ❌ No | Total |

### Creating a Network with a Specific Driver

```bash
# Bridge (default - no need to specify)
docker network create my-network

# Explicit bridge
docker network create --driver bridge my-network

# Host
docker network create --driver host my-network

# None
docker network create --driver null my-network
```

### Deleting a Network

```bash
docker network rm <network-name>
```

**Example:**
```bash
docker network rm mongo-network
```

---

## 22. Quick Reference: All Docker Commands

### Image Commands

```bash
docker pull <image>              # Download an image from Docker Hub
docker pull <image>:<tag>        # Download a specific version
docker images                    # List all local images
docker rmi <image>               # Remove an image
docker rmi <image>:<tag>         # Remove a specific version
docker build -t <name>:<tag> .   # Build image from Dockerfile
docker push <username>/<image>   # Push image to Docker Hub
```

### Container Commands

```bash
docker run <image>                      # Create + start a new container
docker run -d <image>                   # Run in background (detach)
docker run -it <image>                  # Run in interactive/terminal mode
docker run -it <image> /bin/bash        # Run and open bash shell
docker run --name <n> <image>           # Set a custom name
docker run -p <host>:<container> <img>  # Bind ports
docker run -e KEY=VALUE <image>         # Set environment variable
docker run -v <host-path>:<cont-path>   # Mount a volume (bind mount)
docker run -v <vol-name>:<cont-path>    # Mount a named volume
docker run --network <net-name> <img>   # Connect to a network

docker ps                               # List running containers
docker ps -a                            # List ALL containers
docker start <name/id>                  # Start a stopped container
docker stop <name/id>                   # Stop a running container
docker rm <name/id>                     # Delete a container
docker logs <name/id>                   # View container logs
docker exec -it <name/id> /bin/bash     # Enter a running container
```

### Network Commands

```bash
docker network ls                       # List all networks
docker network create <name>            # Create a new network
docker network rm <name>                # Delete a network
```

### Volume Commands

```bash
docker volume ls                        # List all volumes
docker volume create <name>             # Create a named volume
docker volume prune                     # Delete unused volumes
```

### Docker Compose Commands

```bash
docker-compose -f <file.yml> up -d      # Start all containers (background)
docker-compose -f <file.yml> down       # Stop + remove all containers
```

---

## 🎓 Key Concepts Summary

| Concept | One-Line Explanation |
|---|---|
| **Docker** | Platform to build and run containers |
| **Container** | Packaged app + dependencies = portable, isolated unit |
| **Image** | Blueprint for creating containers (like a class) |
| **Docker Hub** | Public registry of Docker images (like GitHub for images) |
| **Tag** | Version label for an image (e.g., `mysql:8.0`) |
| **Layers** | Images are built in stacked, cached layers |
| **Port Binding** | Mapping host port to container port |
| **Docker Network** | Isolated network for containers to communicate |
| **Docker Compose** | Tool to define and run multi-container apps via YAML |
| **Dockerfile** | Recipe file with instructions to build a custom image |
| **Dockerizing** | Converting your app into a Docker image |
| **Volume** | Persistent storage that survives container restarts |
| **Bridge Driver** | Default network driver for container communication |

---

*Happy Dockerizing! 🐳*