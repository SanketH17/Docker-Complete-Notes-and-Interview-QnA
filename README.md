# 🐳 Docker — Complete Notes

---

## 📋 Table of Contents

1. [Why Do We Need Docker?](#1-why-do-we-need-docker)
2. [What is Docker?](#2-what-is-docker)
3. [Docker Architecture — How Docker Works Internally](#3-docker-architecture--how-docker-works-internally)
4. [Docker Containers](#4-docker-containers)
5. [Docker Images](#5-docker-images)
6. [Docker Image vs Docker Container — Key Differences](#6-docker-image-vs-docker-container--key-differences)
7. [Setting Up Docker](#7-setting-up-docker)
8. [Docker Hub and Registries](#8-docker-hub-and-registries)
9. [Essential Docker Commands](#9-essential-docker-commands)
10. [Image Versions and Tags](#10-image-versions-and-tags)
11. [Image Layers — How Layering Works](#11-image-layers--how-layering-works)
12. [Port Binding](#12-port-binding)
13. [Troubleshooting Docker Containers](#13-troubleshooting-docker-containers)
14. [Docker vs Virtual Machine](#14-docker-vs-virtual-machine)
15. [Docker Networking](#15-docker-networking)
16. [Practical Project: Node.js App + MongoDB + MongoExpress](#16-practical-project-nodejs-app--mongodb--mongoexpress)
17. [Docker Compose](#17-docker-compose)
18. [Dockerizing Your Own Application (Dockerfile)](#18-dockerizing-your-own-application-dockerfile)
19. [Building and Running Your Custom Docker Image](#19-building-and-running-your-custom-docker-image)
20. [Publishing Your Image to Docker Hub](#20-publishing-your-image-to-docker-hub)
21. [Docker Volumes — Data Persistence](#21-docker-volumes--data-persistence)
22. [Advanced Docker Networking](#22-advanced-docker-networking)
23. [Docker Best Practices](#23-docker-best-practices)
24. [Common Mistakes and How to Avoid Them](#24-common-mistakes-and-how-to-avoid-them)
25. [Quick Reference: All Docker Commands](#25-quick-reference-all-docker-commands)

---

## 1. Why Do We Need Docker?

### The Classic Problem: "It Works on My Machine"

Imagine you are working on a Node.js application with your team. On your laptop you have:
- **Node.js v16**
- **MongoDB v4.2**

A new team member joins and sets up the same project on their Mac. But when they install dependencies, they accidentally install:
- **Node.js v20** (newer version)
- **MongoDB v6** (newer version)

Now the application behaves differently on their machine. Bugs appear that did not exist before. This is the famous **"It works on my machine"** problem.

```
  Developer A (Windows)          Developer B (Mac)
  ┌─────────────────┐           ┌─────────────────┐
  │  Node.js v16    │           │  Node.js v20    │
  │  MongoDB v4.2   │           │  MongoDB v6     │
  │  npm v8         │           │  npm v10        │
  │                 │           │                 │
  │  App works ✅   │           │  App broken ❌  │
  └─────────────────┘           └─────────────────┘
          │                              │
          └──── Same source code! ───────┘
```

### Why Does This Happen?

When we manually set up an environment on a new machine, several things can go wrong:

| Problem | Explanation |
|---|---|
| **Version mismatch** | A team member installs a newer or older version of a dependency by accident |
| **Missing dependencies** | Real-world apps have dozens of dependencies — manually installing all of them is error-prone |
| **OS-specific differences** | Some commands or file paths work on Mac/Linux but fail on Windows |
| **Inconsistent setup** | Each developer ends up with a slightly different environment |
| **Environment drift** | Over time, developers update their tools at different times, making environments diverge |

This problem gets **worse as teams grow**. With 2-4 people it is manageable, but with 20+ developers all working on the same project, it becomes a serious bottleneck.

### How Docker Solves This

With Docker, every developer runs the **exact same environment** — regardless of their operating system or local setup.

```
  Developer A (Windows)          Developer B (Mac)           Developer C (Linux)
  ┌─────────────────┐           ┌─────────────────┐         ┌─────────────────┐
  │                 │           │                 │         │                 │
  │  ┌───────────┐  │           │  ┌───────────┐  │         │  ┌───────────┐  │
  │  │  Docker   │  │           │  │  Docker   │  │         │  │  Docker   │  │
  │  │ Container │  │           │  │ Container │  │         │  │ Container │  │
  │  │           │  │           │  │           │  │         │  │           │  │
  │  │ Node v16  │  │           │  │ Node v16  │  │         │  │ Node v16  │  │
  │  │ Mongo 4.2 │  │           │  │ Mongo 4.2 │  │         │  │ Mongo 4.2 │  │
  │  │ App ✅    │  │           │  │ App ✅    │  │         │  │ App ✅    │  │
  │  └───────────┘  │           │  └───────────┘  │         │  └───────────┘  │
  └─────────────────┘           └─────────────────┘         └─────────────────┘
          │                              │                           │
          └──────── Same Docker Image ── ┴ ──────────────────────────┘
```

> 💡 **Docker was built specifically to solve this problem** — by packaging your application together with ALL its dependencies into one portable unit that runs the same everywhere.

---

## 2. What is Docker?

In the simplest terms:

> **Docker is a platform that helps you build, ship, and run containers.**

Think of Docker like a **shipping container** on a cargo ship. No matter what is inside (furniture, electronics, food), the container is always the same standard size and shape. Ships, cranes, and trucks can handle any container — they do not care what is inside.

Similarly, Docker wraps your application and all its dependencies into a **standardized container** that can run anywhere — on your laptop, a colleague's Mac, a Linux server, or a cloud machine — without any manual setup.

### The Three Key Actions of Docker

Docker does three things really well:

```
  ┌─────────┐        ┌─────────┐        ┌─────────┐
  │  BUILD  │  ───►  │  SHIP   │  ───►  │   RUN   │
  │         │        │         │        │         │
  │ Create  │        │ Share   │        │ Start   │
  │ image   │        │ image   │        │ container│
  │ from    │        │ via     │        │ from    │
  │ Docker- │        │ Docker  │        │ image   │
  │ file    │        │ Hub     │        │ anywhere│
  └─────────┘        └─────────┘        └─────────┘
```

1. **Build** — Package your application into a Docker image using a Dockerfile
2. **Ship** — Share the image through Docker Hub (or a private registry)
3. **Run** — Create containers from the image on any machine

To understand Docker fully, you need to understand two key terms:
1. **Docker Container** — The running application
2. **Docker Image** — The blueprint for creating containers

---

## 3. Docker Architecture — How Docker Works Internally

Before diving deeper, it helps to understand how Docker is structured behind the scenes.

Docker follows a **client-server architecture**. There are three main parts:

```
┌─────────────────────────────────────────────────────────────────┐
│                        YOUR MACHINE                              │
│                                                                  │
│  ┌──────────────┐         ┌──────────────────────────────────┐  │
│  │              │         │         Docker Daemon             │  │
│  │   Docker     │  REST   │        (dockerd)                 │  │
│  │   Client     │  API    │                                  │  │
│  │   (CLI)      │ ──────► │  ┌──────────┐  ┌──────────┐     │  │
│  │              │         │  │Container │  │Container │     │  │
│  │ docker run   │         │  │    1     │  │    2     │     │  │
│  │ docker build │         │  └──────────┘  └──────────┘     │  │
│  │ docker pull  │         │                                  │  │
│  │ docker ps    │         │  ┌──────────┐  ┌──────────┐     │  │
│  │              │         │  │ Image A  │  │ Image B  │     │  │
│  └──────────────┘         │  └──────────┘  └──────────┘     │  │
│                           └──────────────────────────────────┘  │
│                                        │                         │
│                                        │ pull / push             │
│                                        ▼                         │
│                           ┌──────────────────────┐              │
│                           │    Docker Registry    │              │
│                           │    (Docker Hub)       │              │
│                           └──────────────────────┘              │
└─────────────────────────────────────────────────────────────────┘
```

### The Three Parts Explained

**1. Docker Client (CLI)**
This is what you use when you type commands like `docker run` or `docker build` in your terminal. The client sends your commands to the Docker Daemon.

> Think of the client as a **remote control** — you press buttons, but the actual work happens in the TV (daemon).

**2. Docker Daemon (dockerd)**
This is the "brain" of Docker. It runs in the background on your machine and does the real work:
- Builds images
- Creates containers
- Manages networks and volumes
- Pulls/pushes images to registries

> Think of the daemon as a **factory manager** — it receives orders (from the client) and manages all the workers (containers).

**3. Docker Registry (Docker Hub)**
This is where Docker images are stored and shared. Docker Hub is the default public registry, but companies can also set up private registries.

> Think of a registry as a **warehouse** — it stores all the blueprints (images) that the factory (daemon) can use.

### How a `docker run` Command Flows

When you type `docker run nginx`, here is what happens step by step:

```
You type: docker run nginx
        │
        ▼
┌─────────────┐    "Run nginx please"     ┌───────────────┐
│   Docker    │ ────────────────────────► │  Docker       │
│   Client    │                           │  Daemon       │
└─────────────┘                           └───────┬───────┘
                                                  │
                                          Does image exist
                                          locally?
                                                  │
                                         ┌────────┴────────┐
                                         │                 │
                                        YES               NO
                                         │                 │
                                         │         ┌───────▼───────┐
                                         │         │  Pull image   │
                                         │         │  from Docker  │
                                         │         │  Hub          │
                                         │         └───────┬───────┘
                                         │                 │
                                         ▼                 ▼
                                  ┌─────────────────────────────┐
                                  │   Create and start a new    │
                                  │   container from the image  │
                                  └─────────────────────────────┘
```

---

## 4. Docker Containers

### What is a Container?

A **Docker Container** is a single, self-contained unit (a bundle) that packages:
- Your **application code**
- All its **dependencies** (libraries, frameworks, runtime)
- The **configuration** needed to run it

> 🎒 **Analogy:** Think of a container like a packed lunchbox. Instead of carrying your rice, curry, spoon, and napkin separately every day, you pack them all in one box. Now you can take that same lunchbox anywhere — office, picnic, trip — and everything you need is right there.

### Properties of Docker Containers

**1. Portable**
A container can be shared across different machines and operating systems. You do not need to worry about whether the receiving machine is Windows, Mac, or Linux — the container works the same everywhere.

**2. Lightweight**
Containers are very small in size compared to traditional virtual machines. Creating, updating, or deleting them is fast and easy. You can run dozens of containers on a single machine without performance issues.

**3. Isolated Environment**
Each container runs in its own isolated environment, completely separate from:
- The host machine (your laptop or server)
- Other containers running on the same machine

> 🏠 **Analogy:** Think of containers like separate apartments in a building. Each apartment has its own kitchen, bathroom, and electricity — fully independent. What happens in one apartment does not affect the others, and none of them affect the building's foundation (the host machine).

**4. Multiple Versions on the Same Machine**
Without Docker, if your machine has Node.js v16 installed, *all* of your apps must use v16.

With Docker, you can run:
- **App 1** in Container 1 → Uses Node.js **v16**
- **App 2** in Container 2 → Uses Node.js **v20**

Both on the *same machine*, simultaneously. No conflicts.

```
  ┌──── Your Computer ────────────────────────────┐
  │                                                │
  │  ┌─── Container 1 ───┐  ┌─── Container 2 ───┐ │
  │  │  Node.js v16      │  │  Node.js v20      │ │
  │  │  MongoDB v4.2     │  │  PostgreSQL v15   │ │
  │  │  App A            │  │  App B            │ │
  │  └───────────────────┘  └───────────────────┘ │
  │                                                │
  │  No conflicts! Each container is independent.  │
  └────────────────────────────────────────────────┘
```

### Container Lifecycle

A container goes through different states during its life:

```
                     docker run
                         │
                         ▼
  ┌─────────┐       ┌─────────┐       ┌─────────┐
  │         │       │         │       │         │
  │ CREATED │──────►│ RUNNING │──────►│ STOPPED │
  │         │ start │         │ stop  │(EXITED) │
  └─────────┘       └────┬────┘       └────┬────┘
                         │                  │
                         │                  │  docker start
                         │                  │────────┘
                         │                       │
                         │               docker rm│
                         │                       ▼
                         │               ┌─────────┐
                         │               │         │
                         └──────────────►│ REMOVED │
                           docker rm     │(DELETED)│
                                         └─────────┘
```

- **Created** — Container exists but has not started yet
- **Running** — Container is active and doing its job
- **Stopped (Exited)** — Container has been stopped but still exists on disk (can be restarted)
- **Removed (Deleted)** — Container is permanently gone

---

## 5. Docker Images

### What is a Docker Image?

A **Docker Image** is NOT a picture or photo. It is:

> **A read-only template (a blueprint) that contains all the instructions and files needed to create a Docker Container.**

An image includes:
- A base operating system (like a minimal Linux)
- Your application code
- All libraries and dependencies
- Environment variables and configuration
- A startup command (what to run when the container starts)

### The Class and Object Analogy

If you know programming, the relationship between a Docker Image and a Docker Container is exactly like the relationship between a **Class** and an **Object** in Object-Oriented Programming:

| OOP Concept | Docker Equivalent |
|---|---|
| **Class** | **Docker Image** |
| **Object (instance)** | **Docker Container** |
| A class is a blueprint | An image is a blueprint |
| You create multiple objects from one class | You create multiple containers from one image |
| Objects use system memory and resources | Containers use CPU, RAM, and disk resources |
| The class file itself does not consume RAM | The image file itself is just stored on disk |

```
                     ┌─────────────┐
               ┌────►│ Container 1 │  (Running instance)
               │     └─────────────┘
┌──────────┐   │     ┌─────────────┐
│  Docker  │───┼────►│ Container 2 │  (Running instance)
│  Image   │   │     └─────────────┘
│(Blueprint)│   │     ┌─────────────┐
└──────────┘   └────►│ Container 3 │  (Running instance)
                     └─────────────┘

One image can create many containers.
Each container is independent of the others.
```

### How Image Sharing Works

When sharing your application with teammates, you **do not share the container** — you **share the Docker Image**.

```
  Developer A                   Docker Hub                  Developer B
  ┌──────────┐   docker push   ┌──────────┐  docker pull  ┌──────────┐
  │  Builds  │ ──────────────► │  Stores  │ ◄──────────── │  Pulls   │
  │  image   │                 │  image   │               │  image   │
  └──────────┘                 └──────────┘               └────┬─────┘
                                                               │
                                                          docker run
                                                               │
                                                               ▼
                                                         ┌──────────┐
                                                         │Container │
                                                         │(Running) │
                                                         └──────────┘
```

Each team member:
1. Pulls the Docker Image from a registry (like Docker Hub)
2. Runs it on their local machine
3. A new container is automatically created from that image

This ensures everyone gets the **exact same environment**, no matter what OS they are on.

---

## 6. Docker Image vs Docker Container — Key Differences

| Feature | Docker Image | Docker Container |
|---|---|---|
| **What it is** | A static blueprint or snapshot | A running instance of an image |
| **State** | Immutable (read-only, cannot be changed) | Active and running (read-write layer on top) |
| **Resource usage** | Minimal — just disk space | Uses CPU, RAM, network |
| **Created from** | A Dockerfile | A Docker Image |
| **Analogy** | A recipe | The cooked dish |
| **Another analogy** | A cookie cutter | The actual cookie |
| **Size** | Small (can be in KBs or MBs) | Larger (includes the running environment) |
| **Shared with team?** | ✅ Yes — you share images | ❌ No — containers run locally |
| **Can be stored in registry?** | ✅ Yes (Docker Hub, etc.) | ❌ No |

```
  ┌──────────────────┐                 ┌──────────────────┐
  │   Docker Image   │   docker run    │  Docker Container │
  │                  │ ──────────────► │                  │
  │  - Read-only     │                 │  - Read-write    │
  │  - Blueprint     │                 │  - Running app   │
  │  - Stored on disk│                 │  - Uses CPU/RAM  │
  │  - Shareable     │                 │  - Isolated      │
  └──────────────────┘                 └──────────────────┘
```

---

## 7. Setting Up Docker

### Step 1: Download Docker Desktop

Visit **[docker.com](https://docker.com)** and download **Docker Desktop** for your operating system:
- **Mac** → Choose Intel chip or Apple Silicon (M1/M2/M3/M4)
- **Windows** → Choose Windows AMD64
- **Linux** → Use the appropriate package for your distribution

### Step 2: Install Docker Desktop

**On Windows:**
1. Run the installer
2. Click **OK** when asked for admin permissions
3. Make sure "Use WSL 2 instead of Hyper-V" is checked (recommended)
4. Click **Close and Restart** when installation finishes
5. After restart, accept the license agreement
6. Choose **Recommended Settings** → Click **Finish**
7. You can skip the sign-in step

**On Mac:**
1. Open the `.dmg` file
2. Drag Docker to Applications
3. Open Docker from Applications
4. Accept the license agreement
5. Choose Recommended Settings → Finish

**On Linux (Ubuntu/Debian):**
```bash
# Update packages
sudo apt-get update

# Install Docker using the official convenience script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to the docker group (so you don't need sudo every time)
sudo usermod -aG docker $USER

# Log out and log back in for the group change to take effect
```

### Step 3: Verify Installation

Open your **Terminal** (Mac/Linux) or **PowerShell** (Windows) and run:

```bash
docker --version
```

**Expected output:**
```
Docker version 27.5.1, build ...
```

To check more detailed information about your Docker installation:
```bash
docker info
```

This shows the number of containers, images, server version, storage driver, and more.

To see all available Docker commands:
```bash
docker
```

If you see a list of commands, Docker is successfully installed! ✅

### Docker Desktop Interface

After opening Docker Desktop, you will see:
- **Containers** tab — shows all your running and stopped containers
- **Images** tab — shows all Docker images downloaded on your machine
- **Volumes** tab — shows all your Docker volumes
- **Builds** tab — shows build history

---

## 8. Docker Hub and Registries

### What is a Docker Registry?

A **Docker Registry** is a storage and distribution service for Docker images. Think of it as a library where Docker images are kept.

### What is Docker Hub?

**Docker Hub** ([hub.docker.com](https://hub.docker.com)) is the **default public registry** for Docker images. It is to Docker what **GitHub** is to code.

| GitHub | Docker Hub |
|---|---|
| Stores code repositories | Stores Docker image repositories |
| You push/pull code | You push/pull Docker images |
| Public and private repos | Public and private repos |
| Anyone can share code | Anyone can share Docker images |

```
  ┌─────────────────────────────────────────────────────┐
  │                    Docker Hub                        │
  │                                                     │
  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │
  │  │  nginx   │ │  mysql   │ │  node    │ │ redis  │ │
  │  │  :latest │ │  :8.0    │ │  :20     │ │ :7     │ │
  │  │  :alpine │ │  :8.4    │ │  :18     │ │ :latest│ │
  │  └──────────┘ └──────────┘ └──────────┘ └────────┘ │
  │                                                     │
  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │
  │  │ postgres │ │  ubuntu  │ │  python  │ │ mongo  │ │
  │  │  :16     │ │  :22.04  │ │  :3.12   │ │ :7     │ │
  │  │  :15     │ │  :24.04  │ │  :3.11   │ │ :latest│ │
  │  └──────────┘ └──────────┘ └──────────┘ └────────┘ │
  │                                                     │
  │         Thousands of ready-to-use images!           │
  └─────────────────────────────────────────────────────┘
```

### What You Will Find on Docker Hub

- **Official images** for popular technologies: MySQL, MongoDB, Node.js, Redis, Nginx, Ubuntu, Python, PostgreSQL, etc.
- Each image comes with:
  - Different **tags** (versions)
  - **Documentation** on how to use it
  - List of required **environment variables**
  - Example commands

### Other Registries

Docker Hub is not the only registry. Companies often use private registries:

| Registry | Description |
|---|---|
| **Docker Hub** | Default public registry (free and paid) |
| **GitHub Container Registry (ghcr.io)** | GitHub's own registry |
| **Amazon ECR** | AWS private registry |
| **Google Artifact Registry** | Google Cloud registry |
| **Azure Container Registry** | Microsoft Azure registry |
| **Self-hosted** | Companies can run their own private registries |

> 💡 For learning and personal projects, Docker Hub is all you need.

---

## 9. Essential Docker Commands

Let us learn all the important Docker commands. We will start simple and build up.

---

### 9.1 `docker pull` — Download an Image

Downloads (pulls) a Docker image from Docker Hub to your local machine.

- **What it does:** Fetches a copy of the image and saves it locally
- **Why use it:** To get pre-built images before running them
- **When to use:** When you want to download an image without starting a container

```bash
docker pull <image-name>
```

**Example:**
```bash
docker pull nginx
```

**What happens step by step:**
1. Docker checks if the image already exists on your machine
2. If not, it contacts Docker Hub
3. Downloads the image layer by layer
4. Saves it locally

**Output:**
```
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete
a9edb18cadd1: Pull complete
589b7251471a: Pull complete
...
Status: Downloaded newer image for nginx:latest
```

> 💡 If you do not specify a tag, Docker pulls the `latest` tag by default.

---

### 9.2 `docker images` — List Downloaded Images

Shows all Docker images currently available on your local machine.

- **What it does:** Lists all images with their name, tag, ID, creation date, and size
- **Why use it:** To see what images you have downloaded
- **When to use:** When you want to check which images are available locally

```bash
docker images
```

**Output:**
```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    a6bd71f48f68   2 days ago     187MB
hello-world   latest    d2c94e258dcb   8 months ago   13.3kB
ubuntu        latest    3b418d7b466a   3 weeks ago    77.9MB
```

> 💡 Notice how tiny images can be — `hello-world` is only 13KB! This is why Docker is called **lightweight**.

---

### 9.3 `docker run` — Create and Start a Container

Creates a **new container** from an image and starts it. This is the most commonly used Docker command.

- **What it does:** Creates a brand new container from an image and starts it immediately
- **Why use it:** To run an application inside a container
- **When to use:** Every time you want to start a fresh container

```bash
docker run <image-name>
```

**Example — Run a simple Nginx web server:**
```bash
docker run nginx
```

> ⚠️ **Key difference between `docker pull` and `docker run`:**
> - `docker pull` → ONLY downloads the image (does not start anything)
> - `docker run` → Downloads the image (if not present) AND creates AND starts a new container

```
  docker pull                    docker run
  ┌──────────┐                  ┌──────────┐
  │ Download │                  │ Download │ (if needed)
  │ image    │                  │ image    │
  └──────────┘                  ├──────────┤
      Done!                     │ Create   │
                                │ container│
                                ├──────────┤
                                │ Start    │
                                │ container│
                                └──────────┘
```

**Important flags for `docker run`:**

#### `-d` — Detach Mode (Run in Background)

Runs the container in the background, so your terminal is free to use.

```bash
docker run -d nginx
```

Without `-d`, the container runs in the foreground — your terminal is "attached" to it and you cannot type other commands. With `-d`, you get your terminal back immediately.

```
  Without -d (foreground)        With -d (background)
  ┌─────────────────────┐       ┌─────────────────────┐
  │ $ docker run nginx  │       │ $ docker run -d nginx│
  │ ... nginx logs ...  │       │ a1b2c3d4e5f6...     │
  │ ... logs continue...│       │ $                    │
  │ (terminal is stuck) │       │ (terminal is free!)  │
  └─────────────────────┘       └─────────────────────┘
```

#### `-it` — Interactive Mode

Opens the container's terminal so you can type commands inside it. The `-i` flag keeps input open, and the `-t` flag gives you a nice terminal display.

```bash
docker run -it ubuntu
```

**What happens:** You enter the Ubuntu container's terminal. Your prompt changes to something like:
```
root@abc123def456:/#
```

That alphanumeric code (`abc123def456`) is your **container's ID**. You are now "inside" the container! You can run:
```bash
ls           # list files inside the container
mkdir test   # create a folder inside the container
env          # see environment variables
cat /etc/os-release  # check which OS is running
exit         # exit the container
```

When you type `exit`, the container **stops automatically** (because the main process was the shell, and you ended it).

#### `--name` — Give a Custom Name

By default, Docker assigns random funny names like `happy_einstein` or `silly_morse`. You can set your own meaningful name:

```bash
docker run --name my-web-server nginx
```

> 💡 Custom names are easier to remember and use in other commands like `docker stop my-web-server`.

#### `-e` — Set Environment Variables

Many containers need configuration through environment variables (like database passwords, usernames, or connection strings):

```bash
docker run -e MY_VARIABLE=value <image-name>
```

Multiple environment variables:
```bash
docker run -e DB_USER=admin -e DB_PASS=secret123 mysql
```

#### `--rm` — Automatically Remove Container When It Stops

Normally, stopped containers stay on disk (visible with `docker ps -a`). The `--rm` flag cleans up automatically:

```bash
docker run --rm nginx
```

> 💡 This is useful for quick tests where you do not want leftover containers cluttering up your system.

#### `-p` — Port Binding (covered in detail in [Section 12](#12-port-binding))

```bash
docker run -p <host-port>:<container-port> <image-name>
```

#### Combining Multiple Flags

You can combine flags together. Here is a realistic example:

```bash
docker run -d --name my-nginx -p 8080:80 --rm nginx
```

This runs an Nginx container:
- In the **background** (`-d`)
- With the name **my-nginx** (`--name`)
- Accessible at **localhost:8080** (`-p 8080:80`)
- **Auto-removes** when stopped (`--rm`)

---

### 9.4 `docker ps` — List Running Containers

Shows all **currently running** containers.

- **What it does:** Displays a table of active containers with their ID, image, status, ports, and name
- **Why use it:** To see what is running right now
- **When to use:** Frequently — to monitor your containers

```bash
docker ps
```

**Output:**
```
CONTAINER ID   IMAGE   COMMAND                  CREATED         STATUS         PORTS                  NAMES
a1b2c3d4e5f6   nginx   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp   my-nginx
```

**Understanding the columns:**
| Column | Meaning |
|---|---|
| CONTAINER ID | Unique short ID for the container |
| IMAGE | Which image the container was created from |
| COMMAND | The startup command running inside |
| CREATED | When the container was created |
| STATUS | Current state (Up, Exited, etc.) |
| PORTS | Port mappings (host → container) |
| NAMES | The container name |

### `docker ps -a` — List ALL Containers (Including Stopped)

```bash
docker ps -a
```

Shows running AND stopped containers. Every container you have ever created (and not removed) appears here.

---

### 9.5 `docker start` — Start an Existing Container

`docker run` creates a **brand new** container every time. If you want to **restart an existing stopped container** (keeping its data and configuration), use `docker start`:

- **What it does:** Restarts an existing stopped container
- **Why use it:** To resume a container without creating a new one
- **When to use:** When you have stopped a container and want it running again

```bash
docker start <container-name-or-id>
```

**Example:**
```bash
docker start my-nginx
```

> 🔁 Think of it like this:
> - `docker run` = Buying a new car and driving it off the lot
> - `docker start` = Starting your existing parked car that you turned off earlier

---

### 9.6 `docker stop` — Stop a Running Container

Gracefully stops a running container. The container sends a termination signal to the main process, giving it time to clean up.

- **What it does:** Stops the container (it still exists, just not running)
- **Why use it:** To pause a container without deleting it
- **When to use:** When you want to free up resources but keep the container for later

```bash
docker stop <container-name-or-id>
```

**Example:**
```bash
docker stop my-nginx
```

The container moves to "Exited" status — it still exists on disk, just not running. You can restart it later with `docker start`.

> 💡 `docker stop` waits up to 10 seconds for the container to shut down gracefully. If it does not stop, Docker forcefully kills it. You can use `docker kill` to immediately force-stop a container.

---

### 9.7 `docker rm` — Remove/Delete a Container

Permanently deletes a stopped container from disk.

- **What it does:** Removes the container completely
- **Why use it:** To clean up containers you no longer need
- **When to use:** When a container is stopped and you will not need it again

```bash
docker rm <container-name-or-id>
```

> ⚠️ You cannot remove a running container. Stop it first with `docker stop`, then remove it.

**Force remove a running container (stop + remove in one step):**
```bash
docker rm -f <container-name-or-id>
```

**Remove all stopped containers at once:**
```bash
docker container prune
```

---

### 9.8 `docker rmi` — Remove/Delete an Image

Permanently deletes a Docker image from your local machine.

- **What it does:** Removes the image and frees up disk space
- **Why use it:** To clean up images you no longer need
- **When to use:** When you have old or unused images taking up space

```bash
docker rmi <image-name-or-id>
```

**Example:**
```bash
docker rmi nginx
```

> ⚠️ If a container (even a stopped one) was created from this image, you must **delete the container first** before deleting the image. Otherwise you will get an error:
>
> ```
> Error: image is being used by stopped container abc123
> ```

**Correct order to clean up:**
```bash
# Step 1: Remove the container
docker rm my-nginx

# Step 2: Now remove the image
docker rmi nginx
```

**Remove all unused images at once:**
```bash
docker image prune -a
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
| `docker run --rm <image>` | Run and auto-remove when stopped |
| `docker run -e KEY=VALUE <image>` | Set an environment variable |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (running + stopped) |
| `docker start <name/id>` | Restart an existing stopped container |
| `docker stop <name/id>` | Stop a running container |
| `docker rm <name/id>` | Delete a stopped container |
| `docker rm -f <name/id>` | Force delete a running container |
| `docker rmi <name/id>` | Delete an image |
| `docker container prune` | Delete all stopped containers |
| `docker image prune -a` | Delete all unused images |

---

## 10. Image Versions and Tags

### What Are Tags?

Every Docker image can have multiple **tags**. A tag is a **version label** for an image.

> 🏷️ **Analogy:** Think of tags like labels on a juice bottle. The brand is the same ("Orange Juice"), but the labels tell you what variant you are getting — "No Pulp", "Extra Pulp", "Diet", etc. Similarly, `node` is the image, and `:20`, `:18`, `:20-alpine` are the tags (variants).

Common tag examples:
- `mysql:latest` → The latest MySQL version
- `mysql:8.0` → MySQL version 8.0 specifically
- `node:20` → Node.js version 20
- `node:20-alpine` → Node.js v20 on a minimal Alpine Linux base
- `nginx:1.25` → Nginx version 1.25
- `python:3.12-slim` → Python 3.12 on a slim (smaller) base image

### Pulling a Specific Tag/Version

```bash
# Pull the latest version (default)
docker pull mysql

# Pull a specific version
docker pull mysql:8.0

# Pull a specific variant
docker pull node:20-alpine
```

### What is Alpine?

You will often see images with the `-alpine` suffix. **Alpine Linux** is a very small, lightweight Linux distribution (about 5MB). Alpine-based images are much smaller than regular ones.

```
  Regular node image:    ~1000 MB
  node:20-slim:          ~200 MB
  node:20-alpine:        ~130 MB
```

> 💡 Use Alpine images when you want smaller, faster containers. They are great for production.

### Running Multiple Versions Side by Side

You can now have two different versions of MySQL running as separate containers on the same machine — something impossible without Docker!

```bash
# Container 1 with latest MySQL
docker run -d --name mysql-new -e MYSQL_ROOT_PASSWORD=secret mysql:latest

# Container 2 with MySQL 8.0
docker run -d --name mysql-old -e MYSQL_ROOT_PASSWORD=secret mysql:8.0
```

Both run simultaneously without any conflict.

```
  ┌──── Your Computer ──────────────────────────┐
  │                                              │
  │  ┌─── Container 1 ───┐  ┌─── Container 2 ──┐│
  │  │  mysql:latest      │  │  mysql:8.0       ││
  │  │  (version 9.x)    │  │  (version 8.0)   ││
  │  │  Port: 3306       │  │  Port: 3306      ││
  │  │  Host: 3306       │  │  Host: 3307      ││
  │  └───────────────────┘  └──────────────────┘│
  │                                              │
  │  Both running at the same time. No conflict! │
  └──────────────────────────────────────────────┘
```

---

## 11. Image Layers — How Layering Works

### The Concept

Every Docker image is made up of **multiple stacked layers**, like a multi-layered cake. Each instruction in a Dockerfile creates a new layer.

```
  ┌─────────────────────────────────────┐
  │  Layer 5: CMD ["node", "server.js"] │  ← Startup command
  ├─────────────────────────────────────┤
  │  Layer 4: COPY . /app               │  ← Your application code
  ├─────────────────────────────────────┤
  │  Layer 3: RUN npm install           │  ← Installed dependencies
  ├─────────────────────────────────────┤
  │  Layer 2: Node.js runtime           │  ← Node.js environment
  ├─────────────────────────────────────┤
  │  Layer 1: Debian Linux base         │  ← Operating system
  └─────────────────────────────────────┘
```

### Key Rules About Layers

- Each layer is **read-only** (immutable — cannot be changed)
- Layers are stacked on top of each other
- When a container runs, Docker adds a thin **writable layer** on top — this is where any file changes happen
- The underlying image layers are never modified

```
  ┌─────────────────────────────────────┐
  │  Writable Container Layer           │  ← Changes go here (temporary)
  ╞═════════════════════════════════════╡
  │  Layer 4: Application code          │  ← Read-only
  ├─────────────────────────────────────┤
  │  Layer 3: Dependencies              │  ← Read-only
  ├─────────────────────────────────────┤
  │  Layer 2: Runtime (Node.js)         │  ← Read-only
  ├─────────────────────────────────────┤
  │  Layer 1: Base OS (Debian)          │  ← Read-only
  └─────────────────────────────────────┘
        Image Layers (permanent)
```

### Why Layers Are Smart — Caching

When you pull `mysql:latest` and then `mysql:8.0`, Docker is smart enough to **not re-download layers that are shared** between the two versions.

**Example output when pulling the second version:**
```
Layer 1: Already exists  ← Shared with mysql:latest, skip download
Layer 2: Already exists  ← Shared with mysql:latest, skip download
Layer 3: Pull complete   ← Different from mysql:latest, download it
Layer 4: Pull complete   ← Different from mysql:latest, download it
```

```
  mysql:latest              mysql:8.0
  ┌──────────────┐         ┌──────────────┐
  │  Layer 4     │         │  Layer 4*    │  ← Different
  ├──────────────┤         ├──────────────┤
  │  Layer 3     │         │  Layer 3*    │  ← Different
  ├──────────────┤         ├──────────────┤
  │  Layer 2     │◄───────►│  Layer 2     │  ← SHARED (same layer!)
  ├──────────────┤         ├──────────────┤
  │  Layer 1     │◄───────►│  Layer 1     │  ← SHARED (same layer!)
  └──────────────┘         └──────────────┘

  Shared layers are downloaded only ONCE — saves disk space and time!
```

### Why This Matters for Dockerfile Design

When you rebuild a Docker image, Docker uses **cached layers** for any instruction that has not changed. This makes builds faster.

**Example:** If you only changed your application code (Layer 4), Docker reuses Layers 1-3 from cache and only rebuilds Layer 4.

> 💡 **Tip:** Put instructions that change frequently (like `COPY . /app`) at the bottom of your Dockerfile, and instructions that rarely change (like `RUN npm install`) higher up. This maximizes cache usage.

---

## 12. Port Binding

### The Problem

Every container has its **own isolated network with its own ports**. These are completely separate from your host machine's ports.

> 🏢 **Analogy:** Imagine a big office building (your computer). Each container is a separate office room on a different floor. Each room has its own internal phone extension (container port). But to call from outside the building (your browser), you need to connect through the building's main switchboard (host port). Port binding is the act of connecting the switchboard to a specific room.

When Nginx runs inside a container, it listens on **port 80** — but that is the *container's* port 80, not your laptop's port 80. If you open `localhost:80` in your browser, nothing happens — because there is no connection between your machine's port 80 and the container's port 80.

### How Port Binding Works

To access a container's service from your browser or from another application on your machine, you need to **map (bind) a host machine port to a container port**.

```
  ┌────────┐          ┌───────────────────┐          ┌──────────────────┐
  │        │          │                   │          │                  │
  │Browser │ ──────►  │  HOST MACHINE     │ ──────►  │    CONTAINER     │
  │        │          │  Port 8080        │          │    Port 80       │
  │        │          │  (your computer)  │          │    (Nginx)       │
  └────────┘          └───────────────────┘          └──────────────────┘
      │                        │                              │
      └── localhost:8080 ──────┘──── forwarded to ────────────┘
```

**The `-p` flag syntax:**
```bash
docker run -p <host-port>:<container-port> <image-name>
```

> 📌 **Memory trick:** Think of it as `outside:inside` — the host port (outside the container) comes first, then the container port (inside the container).

**Example — Run Nginx and access it from your browser:**
```bash
docker run -d -p 8080:80 --name my-web nginx
```

Now open your browser and go to `http://localhost:8080` — you will see the Nginx welcome page!

**Example — Run MySQL and bind it to a custom port:**
```bash
docker run -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=secret mysql
```

Now MySQL inside the container is accessible at `localhost:3307` on your machine.

### Important Rule: Each Host Port Can Only Be Used Once

If port 8080 on your machine is already bound to Container 1, you **cannot** bind it to Container 2 as well.

**This will fail:**
```bash
docker run -d -p 8080:80 --name web-1 nginx        # Works ✅
docker run -d -p 8080:80 --name web-2 nginx         # ERROR ❌
# Error: bind: address already in use
```

**Solution:** Use a different host port for the second container:
```bash
docker run -d -p 9090:80 --name web-2 nginx         # Works ✅
```

The container ports can be the same (both using 80 internally), but the **host machine ports must be unique**.

```
  ┌──── Your Computer ───────────────────────────────────┐
  │                                                       │
  │  Host Port 8080 ───► Container 1 (nginx) → Port 80  │
  │  Host Port 9090 ───► Container 2 (nginx) → Port 80  │
  │                                                       │
  │  Both containers use port 80 internally — no problem! │
  │  But each needs a UNIQUE host port.                   │
  └───────────────────────────────────────────────────────┘
```

### Common Default Ports for Popular Services

| Service | Default Container Port |
|---|---|
| Nginx | 80 |
| MySQL | 3306 |
| PostgreSQL | 5432 |
| MongoDB | 27017 |
| Redis | 6379 |
| Node.js (typical) | 3000 or 5000 |

---

## 13. Troubleshooting Docker Containers

When something goes wrong, Docker gives you tools to investigate.

### 13.1 `docker logs` — View Container Logs

When something goes wrong inside a container, the first step is to check its logs.

```bash
docker logs <container-name-or-id>
```

**Example:**
```bash
docker logs my-web
```

**Useful flags:**

```bash
# Follow logs in real-time (like tail -f)
docker logs -f my-web

# Show only the last 50 lines
docker logs --tail 50 my-web

# Show logs with timestamps
docker logs -t my-web

# Combine flags: last 20 lines with timestamps, followed in real-time
docker logs -f --tail 20 -t my-web
```

> 💡 **When to check logs:** Whenever a container exits unexpectedly, or your application is not working as expected.

---

### 13.2 `docker exec -it` — Enter a Running Container's Terminal

If a container is **already running** and you want to go inside it and run commands, use `exec`:

```bash
docker exec -it <container-name> /bin/bash
```

Or for containers that do not have `bash` (like Alpine-based images):
```bash
docker exec -it <container-name> /bin/sh
```

**Example:**
```bash
docker exec -it my-web /bin/bash
```

**Inside the container you can:**
```bash
ls                    # list files
env                   # check environment variables
cat /etc/os-release   # check the OS
whoami                # check which user you are
curl localhost        # test if the service is responding internally
exit                  # leave the container (container keeps running!)
```

> 🔑 **Key difference from `-it` with `docker run`:**
> - `docker run -it` → Creates a **new** container AND enters it. Exiting **STOPS** the container.
> - `docker exec -it` → Enters an **ALREADY RUNNING** container. Exiting does **NOT** stop the container.

---

### 13.3 `docker inspect` — View Detailed Container Information

Shows everything about a container in JSON format: network settings, volumes, environment variables, and more.

```bash
docker inspect <container-name-or-id>
```

**Get a specific piece of information (like the IP address):**
```bash
docker inspect --format '{{.NetworkSettings.IPAddress}}' my-web
```

---

### 13.4 `docker stats` — Monitor Resource Usage

Shows a live stream of CPU, memory, and network usage for all running containers.

```bash
docker stats
```

**Output:**
```
CONTAINER ID   NAME      CPU %   MEM USAGE / LIMIT     MEM %   NET I/O
a1b2c3d4e5f6   my-web    0.00%   3.5MiB / 7.77GiB     0.04%   1.5kB / 0B
```

> 💡 This is useful to check if a container is using too much memory or CPU.

---

### 13.5 Troubleshooting Flowchart

When a container is not working, follow this process:

```
  Container not working?
         │
         ▼
  ┌──────────────────┐     Running?     ┌──────────────────┐
  │ docker ps -a     │────────────────►│ docker logs <name>│
  │ (check status)   │     YES          │ (check for errors)│
  └──────────────────┘                  └──────────────────┘
         │
         │ NOT running (Exited)
         ▼
  ┌──────────────────┐
  │ docker logs <name>│ ← Check why it crashed
  └──────────────────┘
         │
         ▼
  Common causes:
  - Missing environment variables (-e flag)
  - Port already in use (-p flag)
  - Wrong image name or tag
  - Application error (check logs for stack trace)
```

---

## 14. Docker vs Virtual Machine

Both Docker containers and Virtual Machines (VMs) let you run isolated environments. But they work very differently under the hood.

### How an Operating System Works

Every computer has two main layers:
1. **Kernel** — The core of the OS. It talks directly to the hardware (CPU, RAM, disk, network). You never interact with it directly.
2. **Application Layer** — Where all your apps run (browser, terminal, editor, etc.). This is what you see and use.

```
  ┌────────────────────────────┐
  │      Application Layer     │  ← Your apps, tools, libraries
  ├────────────────────────────┤
  │         Kernel             │  ← Talks to hardware
  ├────────────────────────────┤
  │     Hardware (CPU/RAM)     │  ← Physical machine
  └────────────────────────────┘
```

### Virtual Machine Approach

A VM **virtualizes the entire machine** — it has its own Kernel AND Application Layer. A special program called a **Hypervisor** sits between the host OS and the VMs.

```
  ┌───────────────────────────────────────────────────────┐
  │                    HOST MACHINE                        │
  │  ┌─────────────────────────────────────────────────┐  │
  │  │              Hypervisor (VMware, VirtualBox)     │  │
  │  │  ┌────────────────┐    ┌────────────────┐       │  │
  │  │  │     VM 1       │    │     VM 2       │       │  │
  │  │  │  ┌──────────┐  │    │  ┌──────────┐  │       │  │
  │  │  │  │  App     │  │    │  │  App     │  │       │  │
  │  │  │  ├──────────┤  │    │  ├──────────┤  │       │  │
  │  │  │  │ Libraries│  │    │  │ Libraries│  │       │  │
  │  │  │  ├──────────┤  │    │  ├──────────┤  │       │  │
  │  │  │  │  Guest   │  │    │  │  Guest   │  │       │  │
  │  │  │  │  Kernel  │  │    │  │  Kernel  │  │       │  │
  │  │  │  └──────────┘  │    │  └──────────┘  │       │  │
  │  │  │  (Full OS)     │    │  (Full OS)     │       │  │
  │  │  └────────────────┘    └────────────────┘       │  │
  │  └─────────────────────────────────────────────────┘  │
  │  ┌─────────────────────────────────────────────────┐  │
  │  │                Host OS Kernel                    │  │
  │  └─────────────────────────────────────────────────┘  │
  │  ┌─────────────────────────────────────────────────┐  │
  │  │                   Hardware                       │  │
  │  └─────────────────────────────────────────────────┘  │
  └───────────────────────────────────────────────────────┘

  Each VM carries its OWN Kernel → Heavy, slow, but fully isolated
```

### Docker's Approach

Docker only **virtualizes the Application Layer**, and shares the host machine's Kernel. No hypervisor needed — Docker uses the host Kernel directly.

```
  ┌───────────────────────────────────────────────────────┐
  │                    HOST MACHINE                        │
  │  ┌─────────────────────────────────────────────────┐  │
  │  │                Docker Engine                     │  │
  │  │  ┌──────────────┐    ┌──────────────┐           │  │
  │  │  │ Container 1  │    │ Container 2  │           │  │
  │  │  │  ┌────────┐  │    │  ┌────────┐  │           │  │
  │  │  │  │  App   │  │    │  │  App   │  │           │  │
  │  │  │  ├────────┤  │    │  ├────────┤  │           │  │
  │  │  │  │  Libs  │  │    │  │  Libs  │  │           │  │
  │  │  │  └────────┘  │    │  └────────┘  │           │  │
  │  │  │ (App layer   │    │ (App layer   │           │  │
  │  │  │  only!)      │    │  only!)      │           │  │
  │  │  └──────────────┘    └──────────────┘           │  │
  │  └─────────────────────────────────────────────────┘  │
  │  ┌─────────────────────────────────────────────────┐  │
  │  │           SHARED Host OS Kernel                  │  │
  │  └─────────────────────────────────────────────────┘  │
  │  ┌─────────────────────────────────────────────────┐  │
  │  │                   Hardware                       │  │
  │  └─────────────────────────────────────────────────┘  │
  └───────────────────────────────────────────────────────┘

  Containers share the host Kernel → Small, fast, efficient
```

### Side-by-Side Comparison

| Feature | Docker Container | Virtual Machine |
|---|---|---|
| **Size** | MBs (lightweight) | GBs (heavy) |
| **Startup time** | Seconds | Minutes |
| **Performance** | Near-native speed | Slower due to virtualization overhead |
| **Isolation** | Application layer only | Full OS isolation |
| **OS Compatibility** | Shares host kernel | Can run any OS (Windows VM on Linux, etc.) |
| **Resource usage** | Low | High |
| **What it virtualizes** | Application layer only | Entire OS (kernel + application) |
| **How many can run?** | Dozens or hundreds | A few (limited by resources) |

### Docker's Limitation — and How It Is Solved

Since Docker shares the host Kernel, a Linux-based Docker container needs a Linux kernel to run. So how does Docker work on Windows and Mac?

**Docker Desktop** on Mac and Windows runs a **lightweight Linux virtual machine** behind the scenes. Your containers run inside this hidden Linux VM. This is completely transparent — you never see or manage this VM directly.

```
  Windows / Mac
  ┌─────────────────────────────────┐
  │  Docker Desktop                 │
  │  ┌───────────────────────────┐  │
  │  │  Lightweight Linux VM     │  │
  │  │  ┌─────────────────────┐  │  │
  │  │  │   Docker Engine     │  │  │
  │  │  │   ┌───┐ ┌───┐ ┌───┐│  │  │
  │  │  │   │ C1│ │ C2│ │ C3││  │  │
  │  │  │   └───┘ └───┘ └───┘│  │  │
  │  │  └─────────────────────┘  │  │
  │  └───────────────────────────┘  │
  └─────────────────────────────────┘

  On Linux, Docker runs natively (no VM needed)
```

---

## 15. Docker Networking

### The Problem

When you have multiple containers running (for example, a Node.js app container + a MongoDB container), they need to talk to each other. But by default, containers are isolated — they cannot communicate freely.

### Docker Networks

A **Docker Network** is an isolated virtual network where you can place multiple containers so they can **communicate with each other directly** — without needing to expose ports to the host machine.

> 🛣️ **Analogy:** Think of a Docker Network like a private road connecting houses in a gated community. All houses on the same road can visit each other freely without going through the main gate. But people from outside can only come in through the gate (port binding).

```
  ┌──── Docker Network (mongo-network) ──────────────────┐
  │                                                       │
  │  ┌──────────────┐        ┌──────────────┐            │
  │  │   MongoDB    │◄──────►│ MongoExpress │            │
  │  │   Container  │  Talk  │   Container  │            │
  │  │   "mongo"    │ freely │  "mongo-     │            │
  │  │              │        │  express"    │            │
  │  └──────┬───────┘        └──────┬───────┘            │
  │         │                       │                     │
  └─────────┼───────────────────────┼─────────────────────┘
            │                       │
    Port 27017                Port 8081
    (exposed to host)         (exposed to host)
            │                       │
            ▼                       ▼
    localhost:27017          localhost:8081
    (your machine)           (your browser)
```

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

Docker creates three default networks automatically:
- **bridge** — Default network for containers (most used)
- **host** — Container uses the host's network directly (no isolation)
- **none** — No networking at all (fully isolated)

### Creating a Custom Network

```bash
docker network create <network-name>
```

**Example:**
```bash
docker network create my-app-network
```

### Connecting a Container to a Network

Use the `--network` flag when running a container:

```bash
docker run -d --name my-app --network my-app-network nginx
```

### Why Custom Networks Matter

All containers placed in the **same custom network** can communicate with each other by using their **container names** as hostnames — no IP addresses needed.

**Example:** If you have a MongoDB container named `mongo` and a Node.js container named `app`, both in the same network, then from the `app` container you can connect to MongoDB using:
```
mongodb://mongo:27017
```

You use the container name `mongo` as the hostname. Docker's built-in DNS resolves it automatically.

> ⚠️ This name-based communication (DNS) **only works on custom networks**, not on the default `bridge` network!

---

## 16. Practical Project: Node.js App + MongoDB + MongoExpress

This real-world example brings everything together. We will set up a multi-container application step by step.

### Project Overview

We have a simple **Node.js web application** (a university registration page) that:
- Lets users create accounts (stored in MongoDB)
- Fetches user data from MongoDB
- Runs on **port 5050**

Instead of installing MongoDB locally, we will run it inside a Docker container. We will also run **MongoExpress** — a visual browser-based GUI for MongoDB.

### Architecture

```
  ┌────────────────────────────────────────────────────────────┐
  │                        YOUR COMPUTER                        │
  │                                                            │
  │  ┌────────────┐                                            │
  │  │   Browser  │ ◄──── http://localhost:8081                │
  │  │            │ ◄──── http://localhost:5050                │
  │  └──────┬─────┘                                            │
  │         │                                                  │
  │         ▼                                                  │
  │  ┌────────────────┐                                        │
  │  │  Node.js App   │ (running locally or in container)     │
  │  │  Port 5050     │                                        │
  │  └───────┬────────┘                                        │
  │          │ connects to                                     │
  │          ▼                                                  │
  │  ┌──── Docker Network (mongo-network) ──────────────┐     │
  │  │                                                   │     │
  │  │  ┌──────────────┐      ┌──────────────────┐     │     │
  │  │  │   MongoDB    │◄────►│  MongoExpress    │     │     │
  │  │  │   :27017     │      │  :8081           │     │     │
  │  │  │              │      │  (Web GUI for    │     │     │
  │  │  │  (Database)  │      │   MongoDB)       │     │     │
  │  │  └──────────────┘      └──────────────────┘     │     │
  │  │                                                   │     │
  │  └───────────────────────────────────────────────────┘     │
  └────────────────────────────────────────────────────────────┘
```

### Step 1: Create a Custom Network

```bash
docker network create mongo-network
```

Both MongoDB and MongoExpress will be placed in this network so they can talk to each other by name.

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

**What each flag does:**

| Flag | Purpose |
|---|---|
| `-d` | Run in background (detach mode) |
| `-p 27017:27017` | Bind host port 27017 to container port 27017 |
| `--name mongo` | Name the container "mongo" (used as hostname in the network) |
| `--network mongo-network` | Place it in our custom network |
| `-e MONGO_INITDB_ROOT_USERNAME=admin` | Set the admin username |
| `-e MONGO_INITDB_ROOT_PASSWORD=qwerty` | Set the admin password |
| `mongo` | The image to use |

### Step 3: Run the MongoExpress Container

MongoExpress is a web-based UI to visually manage your MongoDB database (like phpMyAdmin for MySQL).

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

**Key point:** In `ME_CONFIG_MONGODB_URL`, we use `mongo` (the container name) as the hostname instead of `localhost` or an IP address. This works because both containers are in the same `mongo-network`.

### Step 4: Access MongoExpress

Open your browser and go to: **http://localhost:8081**

- Default login: `admin` / `pass`

You will see a UI showing your MongoDB databases. From here you can:
- Create databases
- Create collections (similar to SQL tables)
- Add, edit, delete documents (similar to SQL rows)

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

You will see the document you added in MongoExpress — proof that your Node.js app is successfully communicating with the MongoDB Docker container! ✅

### Cleanup

When you are done, clean up everything:
```bash
docker stop mongo mongo-express
docker rm mongo mongo-express
docker network rm mongo-network
```

---

## 17. Docker Compose

### The Problem Docker Compose Solves

Look at the MongoDB and MongoExpress setup from the previous section. It required:
- Creating a network manually
- Running two long `docker run` commands with many flags
- Remembering the exact environment variables, ports, and network names

This is just TWO containers. In real-world applications, you might have 5-10 containers (app, database, cache, message queue, monitoring, etc.). Managing all of them through the terminal is messy, error-prone, and hard to reproduce.

**Docker Compose solves this** by letting you define all your containers in a single structured **YAML file**, then start or stop all of them with one command.

> 📋 **Analogy:** Think of Docker Compose like a restaurant order sheet. Instead of verbally telling the kitchen each dish one by one ("I want a burger with no onions, then a salad with extra dressing, then a coke with ice..."), you write it all on one sheet and hand it in. The kitchen (Docker) reads the sheet and prepares everything together.

### Benefits of Docker Compose

1. **Structured configuration** — All container settings in one readable file
2. **Easy to edit** — Changing a port or environment variable is simple text editing
3. **One command** — Start all containers with `docker compose up`, stop with `docker compose down`
4. **Automatic networking** — Docker Compose automatically creates a shared network for all defined containers (no manual `docker network create` needed!)
5. **Version controlled** — The YAML file can be committed to Git, so your whole team uses the same setup
6. **Reproducible** — Anyone can run the exact same environment with one command

### Docker Compose File Format

The file is typically named `docker-compose.yml` (or `compose.yml`). Here is how our MongoDB + MongoExpress setup looks:

```yaml
services:
  mongo:
    image: mongo
    container_name: mongo
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: qwerty

  mongo-express:
    image: mongo-express
    container_name: mongo-express
    ports:
      - "8081:8081"
    depends_on:
      - mongo
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: qwerty
      ME_CONFIG_MONGODB_URL: mongodb://admin:qwerty@mongo:27017/
    restart: unless-stopped
```

**Breaking it down:**

| Section | Meaning |
|---|---|
| `services` | List of all containers you want to create |
| `image` | Which Docker image to use |
| `container_name` | Custom name for the container (optional) |
| `ports` | Port binding in format `"host:container"` |
| `environment` | Environment variables as key-value pairs |
| `depends_on` | Start this container AFTER the listed containers start |
| `restart` | Restart policy (automatically restart if the container crashes) |

> 📝 **Indentation matters in YAML!** Use consistent 2-space indentation. Wrong indentation will cause errors.

### Comparing Terminal Commands vs Docker Compose

```
  BEFORE (Terminal — messy, error-prone)
  ─────────────────────────────────────
  docker network create mongo-network
  docker run -d -p 27017:27017 --name mongo --network mongo-network \
    -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=qwerty mongo
  docker run -d -p 8081:8081 --name mongo-express --network mongo-network \
    -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin ...long command... mongo-express

  AFTER (Docker Compose — clean, one file, one command)
  ─────────────────────────────────────
  docker compose up -d
```

### Running Docker Compose

> 📝 **Note:** In older Docker versions, the command was `docker-compose` (with a hyphen). In modern Docker (V2+), it is `docker compose` (with a space). Both work, but `docker compose` is recommended.

**Start all containers (in background):**
```bash
docker compose up -d
```

If your file is not named `docker-compose.yml`, specify the filename:
```bash
docker compose -f mongodb.yml up -d
```

**Stop and remove all containers:**
```bash
docker compose down
```

**What `up` does:**
1. Creates a default shared network automatically
2. Pulls any images not already available locally
3. Creates and starts all containers defined in the file
4. Respects `depends_on` order

**What `down` does:**
1. Stops all containers
2. Removes all containers
3. Removes the auto-created network
4. Does NOT delete volumes (your data is safe)

```
  docker compose up                    docker compose down
  ┌─────────────────┐                 ┌─────────────────┐
  │ 1. Create       │                 │ 1. Stop all     │
  │    network      │                 │    containers   │
  │ 2. Pull images  │                 │ 2. Remove all   │
  │    (if needed)  │                 │    containers   │
  │ 3. Create       │                 │ 3. Remove       │
  │    containers   │                 │    network      │
  │ 4. Start all    │                 │                 │
  └─────────────────┘                 └─────────────────┘
```

### Other Useful Docker Compose Commands

```bash
# View running containers managed by compose
docker compose ps

# View logs of all containers
docker compose logs

# Follow logs in real-time
docker compose logs -f

# Restart all services
docker compose restart

# Stop containers without removing them
docker compose stop

# Start stopped containers
docker compose start

# Rebuild images (if using Dockerfile)
docker compose up --build -d
```

### A More Complete Example: Full-Stack App

Here is a more realistic Docker Compose file for a full-stack application with a Node.js app, PostgreSQL database, and Redis cache:

```yaml
services:
  app:
    build: .
    container_name: my-app
    ports:
      - "3000:3000"
    depends_on:
      - db
      - cache
    environment:
      DATABASE_URL: postgresql://admin:secret@db:5432/mydb
      REDIS_URL: redis://cache:6379

  db:
    image: postgres:16
    container_name: my-db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - db-data:/var/lib/postgresql/data

  cache:
    image: redis:7-alpine
    container_name: my-cache
    ports:
      - "6379:6379"

volumes:
  db-data:
```

> 💡 Notice the `volumes` section at the bottom — this creates a **named volume** so your database data persists even after `docker compose down`.

---

## 18. Dockerizing Your Own Application (Dockerfile)

### What is "Dockerizing"?

**Dockerizing** means taking your own application (Node.js, Python, Java, etc.) and packaging it into a Docker image, so it can be run anywhere as a container.

> 📦 **Analogy:** Imagine you cooked an amazing dish. Dockerizing is like vacuum-sealing that dish with all its ingredients, spices, and cooking instructions into one sealed package. Anyone who receives the package can just heat it up and eat — they do not need to go shopping or cook from scratch.

### What is a Dockerfile?

A **Dockerfile** is a plain text file (no file extension) named exactly `Dockerfile` that contains step-by-step instructions telling Docker how to build your image.

> 📐 **Analogy:** A Dockerfile is like a cooking recipe. It tells Docker: "First get these ingredients (base image), then do these preparation steps (install dependencies, copy code), and finally serve it this way (run this command)."

### How the Build Process Works

```
  Dockerfile           docker build          Docker Image        docker run        Container
  ┌──────────┐        ┌──────────┐          ┌──────────┐       ┌──────────┐      ┌──────────┐
  │ FROM     │        │          │          │          │       │          │      │  Running │
  │ WORKDIR  │ ─────► │  Docker  │ ───────► │  Image   │ ────► │  Docker  │ ───► │  App     │
  │ COPY     │        │  Engine  │          │ (saved   │       │  Engine  │      │          │
  │ RUN      │        │ (builds) │          │  on disk)│       │ (creates)│      │          │
  │ CMD      │        │          │          │          │       │          │      │          │
  └──────────┘        └──────────┘          └──────────┘       └──────────┘      └──────────┘
   (text file)         (reads &              (blueprint)        (reads image       (running
                        executes                                & starts           instance)
                        instructions)                           container)
```

### Key Dockerfile Instructions

#### `FROM` — Set the Base Image (Required, must be first)

Every image is built on top of another image (called the **base image**). Your base image provides the runtime environment your app needs.

```dockerfile
FROM node:20
```

**Why do we need a base image?**
Your Node.js app will not run unless Node.js is already installed. The `node` image provides a pre-installed Node.js environment. You build your app on top of it.

```
  ┌─────────────────────────┐
  │  Your Application       │  ← Added by your Dockerfile
  ├─────────────────────────┤
  │  Node.js v20 runtime    │  ← Provided by `node:20`
  ├─────────────────────────┤
  │  Debian Linux           │  ← Base of the node image
  └─────────────────────────┘
```

Common base images:
| Application Type | Base Image |
|---|---|
| Node.js app | `node:20` or `node:20-alpine` |
| Python app | `python:3.12` or `python:3.12-slim` |
| Java app | `eclipse-temurin:21` |
| Go app | `golang:1.22` |
| Static website | `nginx:alpine` |
| General purpose | `ubuntu:24.04` or `alpine:3.19` |

#### `WORKDIR` — Set the Working Directory

Sets the directory inside the container where all following commands will run. If the directory does not exist, Docker creates it.

```dockerfile
WORKDIR /app
```

> 💡 Think of `WORKDIR` as running `cd /app` inside the container. All `RUN`, `COPY`, and `CMD` commands after this will use `/app` as the current directory.

This is **much cleaner** than using `RUN mkdir -p /app` followed by absolute paths everywhere.

#### `COPY` — Copy Files from Host to Image

Copies files from your host machine (where you run `docker build`) into the image.

```dockerfile
COPY . .
```

- First argument (`.`): source path on your host machine (`.` means current directory)
- Second argument (`.`): destination path inside the container (relative to `WORKDIR`)

**More specific examples:**
```dockerfile
# Copy only package.json first (for caching — see tip below)
COPY package.json .

# Copy everything
COPY . .
```

#### `RUN` — Execute Commands During Build

Used to run shell commands while the image is being **built** (not when the container starts). Common uses: install packages, create directories, download files.

```dockerfile
RUN npm install
```

You can have **multiple** `RUN` instructions in a Dockerfile. Each `RUN` creates a new layer.

**More examples:**
```dockerfile
RUN apt-get update && apt-get install -y curl
RUN pip install -r requirements.txt
RUN mkdir -p /app/logs
```

#### `CMD` — Default Command to Run When Container Starts

Defines the command that runs when a container is created from this image. There can be **only ONE** `CMD` in a Dockerfile (if you write multiple, only the last one takes effect).

```dockerfile
CMD ["node", "server.js"]
```

Written as an array of strings (called the "exec form"). This is the recommended format.

#### `ENTRYPOINT` — Similar to CMD but Harder to Override

`ENTRYPOINT` is like `CMD`, but the command cannot be easily overridden when running the container.

```dockerfile
ENTRYPOINT ["node"]
CMD ["server.js"]
```

With this setup:
- `docker run my-app` → runs `node server.js`
- `docker run my-app worker.js` → runs `node worker.js` (CMD is overridden, but ENTRYPOINT stays)

> 📌 **`RUN` vs `CMD` — When does each execute?**
> - `RUN` → Executes during **image build** time (to set up the environment)
> - `CMD` → Executes when a **container starts** (to run your app)
>
> ```
>   Dockerfile is read   ───►   RUN commands execute (build time)
>   Image is saved       ───►   (sitting on disk, doing nothing)
>   docker run           ───►   CMD command executes (run time)
> ```

#### `ENV` — Set Environment Variables

```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
```

These variables are available inside the container when it runs.

> ⚠️ **Security tip:** Do NOT put sensitive passwords in `ENV` inside your Dockerfile. Dockerfiles are usually committed to Git and can be seen by anyone. Instead, pass secrets using the `-e` flag at runtime or use Docker secrets.

#### `EXPOSE` — Document the Port

```dockerfile
EXPOSE 3000
```

This is **informational only** — it tells anyone reading the Dockerfile which port the app listens on. It does NOT actually open or publish the port. You still need the `-p` flag when running the container.

#### `ARG` — Build-time Variables

Variables that are only available during the build process (not at runtime):

```dockerfile
ARG NODE_VERSION=20
FROM node:${NODE_VERSION}
```

You can pass values at build time:
```bash
docker build --build-arg NODE_VERSION=18 -t my-app .
```

### Complete Dockerfile Example — Node.js Application

Here is a well-structured Dockerfile for a Node.js application:

```dockerfile
# Use Node.js 20 on Alpine Linux (smaller image)
FROM node:20-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package files first (for better caching)
COPY package.json package-lock.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Document which port the app uses
EXPOSE 5050

# Start the application when the container runs
CMD ["node", "server.js"]
```

**Why we copy `package.json` before the rest of the code:**
When you rebuild the image after changing your code, Docker can **reuse the cached `npm install` layer** (because `package.json` did not change). This makes rebuilds much faster.

```
  Slow approach (no caching):     Fast approach (with caching):
  ┌─────────────────────────┐     ┌─────────────────────────┐
  │ COPY . .                │     │ COPY package.json .     │ ← cached ✅
  │ RUN npm install         │     │ RUN npm install         │ ← cached ✅
  │                         │     │ COPY . .                │ ← rebuilt
  │ (npm install runs EVERY │     │                         │
  │  time you change code!) │     │ (npm install is skipped │
  └─────────────────────────┘     │  if package.json hasn't │
                                  │  changed!)              │
                                  └─────────────────────────┘
```

### The .dockerignore File

Just like `.gitignore` tells Git which files to skip, `.dockerignore` tells Docker which files to NOT copy into the image.

Create a file named `.dockerignore` in your project root:

```
node_modules
.git
.env
npm-debug.log
Dockerfile
docker-compose.yml
README.md
```

> 💡 Always add `node_modules` to `.dockerignore`. The `RUN npm install` inside the Dockerfile will install fresh dependencies — you do not want to copy your local `node_modules` into the image.

### Another Dockerfile Example — Python Application

```dockerfile
# Use Python 3.12 on slim Debian
FROM python:3.12-slim

# Set working directory
WORKDIR /app

# Copy requirements first (for caching)
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application
COPY . .

# Document the port
EXPOSE 8000

# Start the application
CMD ["python", "app.py"]
```

**Save the Dockerfile** as `Dockerfile` (capital D, no file extension) in your project root.

---

## 19. Building and Running Your Custom Docker Image

### Build the Image

```bash
docker build -t <image-name>:<tag> .
```

- `-t` → Tag (name) for the image
- `.` → Build context — tells Docker to look for the `Dockerfile` in the current directory

**Example:**
```bash
docker build -t my-node-app:1.0 .
```

**What happens during the build:**

```
  $ docker build -t my-node-app:1.0 .
  
  Step 1/6 : FROM node:20-alpine        ← Pull base image
  Step 2/6 : WORKDIR /app               ← Set working directory
  Step 3/6 : COPY package.json .        ← Copy package file
  Step 4/6 : RUN npm install            ← Install dependencies
  Step 5/6 : COPY . .                   ← Copy application code
  Step 6/6 : CMD ["node", "server.js"]  ← Set startup command
  
  Successfully built a1b2c3d4e5f6
  Successfully tagged my-node-app:1.0
```

Each step creates a layer. Docker caches each layer for future builds.

**Verify the image was created:**
```bash
docker images
```

You should see `my-node-app` with tag `1.0` in the list.

### Run a Container from Your Custom Image

```bash
docker run -d -p 5050:5050 --name my-app my-node-app:1.0
```

**Output:**
```
Server running on port 5050
```

Open your browser at `http://localhost:5050` — your custom application is running inside a Docker container!

### Rebuild After Code Changes

When you change your application code and want to update the image:

```bash
# Rebuild the image with a new tag
docker build -t my-node-app:1.1 .

# Stop the old container
docker stop my-app

# Remove the old container
docker rm my-app

# Run a new container with the updated image
docker run -d -p 5050:5050 --name my-app my-node-app:1.1
```

### Run in Interactive Mode to Inspect

If you want to look inside the container to verify everything was set up correctly:

```bash
docker run -it my-node-app:1.0 /bin/sh
```

Inside the container:
```bash
pwd                   # Should show /app
ls                    # See your copied files
ls node_modules       # See installed dependencies
env                   # Verify environment variables
exit
```

### Tagging an Existing Image

You can add additional tags to an image without rebuilding:

```bash
docker tag my-node-app:1.0 my-node-app:latest
```

Now the same image has two tags: `1.0` and `latest`.

---

## 20. Publishing Your Image to Docker Hub

### Why Publish?

Once your image is on Docker Hub, anyone on your team can pull it and run your application with a single command — no source code, no manual setup, no dependency installation.

```
  ┌────────────────┐   docker push    ┌──────────┐   docker pull   ┌────────────────┐
  │ Your Machine   │ ───────────────► │  Docker  │ ◄───────────── │ Team Member's  │
  │                │                  │   Hub    │                │   Machine      │
  │ my-app:1.0     │                  │          │                │                │
  └────────────────┘                  └──────────┘                └────────────────┘
```

### Step 1: Create a Docker Hub Account

Go to **[hub.docker.com](https://hub.docker.com)** → Sign up for a free account

### Step 2: Create a Repository

1. Click **Repositories** → **Create Repository**
2. Give it a name: `test-application`
3. Choose **Public** or **Private**
4. Click **Create**

### Step 3: Login from Terminal

```bash
docker login
```

Docker will ask for your Docker Hub username and password. Alternatively, it may open a browser for you to authenticate (more secure).

### Step 4: Tag Your Image for Docker Hub

The image name must match your Docker Hub repository path: `<username>/<repo-name>:<tag>`

If you already have a local image, tag it for Docker Hub:
```bash
docker tag my-node-app:1.0 yourusername/test-application:1.0
```

Or build with the Docker Hub name directly:
```bash
docker build -t yourusername/test-application:1.0 .
```

### Step 5: Push the Image

```bash
docker push yourusername/test-application:1.0
```

Docker pushes all image layers to Docker Hub. When done, visit your Docker Hub repository — you will see your image there!

### Step 6: Sharing with Your Team

Anyone can now pull and run your application with just two commands:
```bash
docker pull yourusername/test-application:1.0
docker run -d -p 5050:5050 yourusername/test-application:1.0
```

No source code needed, no manual dependency installation. The whole environment is packaged in the image. ✅

### Complete Push/Pull Flow

```
  Developer                 Docker Hub                  Server / Teammate
  ┌──────────────┐         ┌──────────────┐            ┌──────────────┐
  │ 1. Write code│         │              │            │              │
  │ 2. Write     │         │              │            │              │
  │    Dockerfile│         │              │            │              │
  │ 3. Build     │         │              │            │              │
  │    image     │         │              │            │ 5. Pull image│
  │ 4. Push image│ ──────► │  Stores      │ ◄───────── │    from Hub  │
  │              │         │  image       │            │ 6. Run       │
  │              │         │              │            │    container │
  └──────────────┘         └──────────────┘            └──────────────┘
```

---

## 21. Docker Volumes — Data Persistence

### The Data Loss Problem

By default, all data stored inside a container lives in the container's **writable layer** (virtual filesystem). This data is **temporary**:
- When the container is **stopped and removed**, all data is **lost**
- When you create a new container from the same image, it starts **fresh with no data**

**Example:** You add 10 users to a MongoDB database running in a container. You run `docker rm` to remove the container. You start a new MongoDB container. The 10 users are **gone** — you are back to a blank database.

This is a big problem for databases and any application that stores important data!

```
  WITHOUT Volumes (data is LOST):

  ┌──────────────┐    docker rm    ┌──────────────┐
  │  MongoDB     │  ─────────────► │  MongoDB     │
  │  Container   │                 │  Container   │
  │              │                 │  (new)       │
  │  10 users ✅ │                 │  0 users ❌  │
  │  (in memory) │                 │  (fresh DB)  │
  └──────────────┘                 └──────────────┘
      Data lives inside              Data is gone!
      the container
```

### What Are Volumes?

**Docker Volumes** are a way to **store data outside of a container**, on the host machine's filesystem. This data persists even after the container is stopped or deleted.

> 🗄️ **Analogy:** Think of a volume like an external hard drive plugged into your container. The container stores its data on the external drive. Even if you throw away the container (like replacing a laptop), the external drive and its data remain safe. When you plug the drive into a new container, all the data is still there.

### How Volumes Work

```
  WITH Volumes (data is SAFE):

  ┌──────────────┐      ┌──────────────┐
  │  MongoDB     │      │    Volume    │
  │  Container   │◄────►│  (on host    │
  │              │ sync │   machine)   │
  │  10 users    │      │  10 users ✅ │
  └──────────────┘      └──────────────┘
        │                      │
   docker rm                   │ (data persists!)
        │                      │
        ▼                      │
  ┌──────────────┐             │
  │  MongoDB     │             │
  │  Container   │◄────────────┘
  │  (new)       │  reconnect
  │  10 users ✅ │  (data restored!)
  └──────────────┘
```

- Data written inside the container is **automatically synced** to the volume on the host
- If the container is stopped or deleted, data stays safely in the volume
- When a new container connects to the same volume, it **picks up the old data**

### Three Types of Volumes

#### Type 1: Named Volumes (Recommended for Production)

Docker manages the storage location. You just give it a name. Docker decides where to physically store the files on your host machine.

```bash
docker run -v my-data:/data/db mongo
```

- `my-data` → Name of the volume (you choose the name)
- `/data/db` → Path inside the container where data is stored

```
  -v my-data:/data/db
     ──┬──   ───┬────
       │        │
       │        └── Path INSIDE the container
       └── Volume name (Docker manages storage location)
```

Create a named volume explicitly:
```bash
docker volume create my-data
```

> 💡 Named volumes are the **best choice for most use cases**. Docker manages everything for you.

#### Type 2: Anonymous Volumes

Docker manages storage, but no name is given. Docker auto-generates a random ID.

```bash
docker run -v /data/db mongo
```

Only the container path is specified. These volumes are hard to reuse because you need to know the random ID.

> ⚠️ Anonymous volumes are usually cleaned up with `docker volume prune`. Not recommended for important data.

#### Type 3: Bind Mounts

You specify **exactly which folder on your host machine** maps to a path inside the container. You have full control over the location.

```bash
docker run -v /Users/yourname/Desktop/data:/data/db mongo
```

- `/Users/yourname/Desktop/data` → Folder on YOUR host machine
- `/data/db` → Path inside the container

```
  Bind Mount:

  HOST MACHINE                    CONTAINER
  ┌──────────────────┐           ┌──────────────────┐
  │ /Desktop/data/   │◄────────►│ /data/db/         │
  │  ├── file1.json  │  synced  │  ├── file1.json   │
  │  ├── file2.json  │  both    │  ├── file2.json   │
  │  └── file3.json  │  ways    │  └── file3.json   │
  └──────────────────┘           └──────────────────┘
  You can see and edit             Container reads
  these files directly!            and writes here
```

Any files created inside the container at `/data/db` will appear on your desktop in real time — and vice versa!

> 💡 **Bind mounts are great for development** — you can edit code on your host machine and see the changes reflected inside the container instantly, without rebuilding the image.

### Volume in Docker Compose

To add volumes to your `docker-compose.yml`:

```yaml
services:
  mongo:
    image: mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: qwerty

# Declare named volumes at the bottom
volumes:
  mongo-data:
```

Now even if you run `docker compose down` (which removes containers), your MongoDB data is safe in the `mongo-data` volume. Next time you run `docker compose up`, the new MongoDB container picks up exactly where you left off.

> 💡 `docker compose down` does NOT delete volumes. To also delete volumes, use `docker compose down -v` (be careful — this deletes your data!).

### Volume Commands

| Command | Purpose |
|---|---|
| `docker volume ls` | List all volumes |
| `docker volume create <name>` | Create a named volume |
| `docker volume inspect <name>` | View details of a volume (including where data is stored) |
| `docker volume rm <name>` | Delete a specific volume |
| `docker volume prune` | Delete all unused anonymous volumes |

### Named vs Anonymous vs Bind Mount — Summary

| Type | Syntax Example | Who manages storage? | Survives `docker rm`? | Best for |
|---|---|---|---|---|
| **Named Volume** | `-v my-data:/data/db` | Docker | ✅ Yes | Production databases |
| **Anonymous Volume** | `-v /data/db` | Docker (random ID) | ⚠️ Hard to track | Temporary data |
| **Bind Mount** | `-v /host/path:/container/path` | You (host path) | ✅ Yes | Development, sharing files |

---

## 22. Advanced Docker Networking

### The Three Built-in Network Drivers

When Docker creates a network, it uses a **driver** — a type of network that defines how containers communicate.

> 🛣️ **Analogy:** Think of drivers like types of roads:
> - **Bridge** = A regular road connecting houses in a neighborhood (containers can visit each other and go to the highway)
> - **Host** = Living right on the main highway (no separation from traffic)
> - **None** = A house with no road at all (completely isolated)

#### Driver 1: Bridge (Default and Most Common)

The default network driver. When you create a container without specifying a network, it joins the default `bridge` network.

**Use case:** Containers on the same host machine that need to communicate with each other AND with the outside world.

**Two types of bridge networks:**

| Feature | Default Bridge | Custom Bridge |
|---|---|---|
| **How created** | Automatically by Docker | You create it with `docker network create` |
| **Container communication** | By IP address only (fragile — IPs can change) | By container name (reliable — Docker handles DNS) |
| **Isolation** | Less isolated (all containers share it) | Better isolation (only your containers) |
| **Recommendation** | ❌ Avoid for multi-container apps | ✅ Always use custom bridges |

```
  Default Bridge:                Custom Bridge:
  ┌─────────────────────┐       ┌─────────────────────┐
  │  All containers     │       │  Only YOUR          │
  │  share this network │       │  containers are here │
  │                     │       │                     │
  │  Container A        │       │  Container A        │
  │  IP: 172.17.0.2     │       │  Name: "app"        │
  │       │             │       │       │             │
  │       │ IP address  │       │       │ name-based  │
  │       ▼ (fragile)   │       │       ▼ (reliable)  │
  │  Container B        │       │  Container B        │
  │  IP: 172.17.0.3     │       │  Name: "db"         │
  └─────────────────────┘       └─────────────────────┘
```

> 💡 **Always use custom bridge networks** when you need containers to communicate. Container name resolution (using names instead of IPs) **only works in custom bridge networks**.

#### Driver 2: Host

The container shares the host machine's network directly. It has **no isolated network** and **no separate IP address**. The container's ports ARE the host's ports — no port mapping needed.

**Use case:** When you need maximum network performance and do not need network isolation (rare in practice).

```bash
docker run --network host nginx
```

With host networking, Nginx on port 80 inside the container IS port 80 on your machine. No `-p` flag needed.

#### Driver 3: None (Null)

The container has **no network access at all**. Completely isolated from everything — other containers, the host, and the internet.

**Use case:** Running security-sensitive workloads that must not have any network access (for example, processing sensitive data offline).

```bash
docker run --network none alpine
```

### Driver Comparison

| Driver | Container-to-Container | Container-to-Host | Internet Access | Isolation Level |
|---|---|---|---|---|
| **Bridge** (custom) | ✅ Yes (by name) | ✅ Via port mapping | ✅ Yes | High |
| **Bridge** (default) | ✅ Yes (by IP only) | ✅ Via port mapping | ✅ Yes | Medium |
| **Host** | ✅ Yes (shared network) | ✅ Direct (no port mapping) | ✅ Yes | None |
| **None** | ❌ No | ❌ No | ❌ No | Total |

### Creating a Network with a Specific Driver

```bash
# Bridge (default driver — no need to specify)
docker network create my-network

# Explicit bridge
docker network create --driver bridge my-network
```

### Useful Network Commands

```bash
# List all networks
docker network ls

# Create a new network
docker network create my-network

# View details of a network (shows which containers are connected)
docker network inspect my-network

# Connect a running container to a network
docker network connect my-network my-container

# Disconnect a container from a network
docker network disconnect my-network my-container

# Delete a network
docker network rm my-network

# Delete all unused networks
docker network prune
```

---

## 23. Docker Best Practices

These tips will help you write better Dockerfiles and manage containers more effectively.

### Dockerfile Best Practices

**1. Use specific image tags — avoid `latest`**
```dockerfile
# ❌ Bad — you don't know which version you'll get tomorrow
FROM node:latest

# ✅ Good — always the same version
FROM node:20-alpine
```

**2. Use `.dockerignore` to keep images small**
```
node_modules
.git
.env
*.log
```

**3. Use `WORKDIR` instead of `RUN cd`**
```dockerfile
# ❌ Bad
RUN cd /app && npm install

# ✅ Good
WORKDIR /app
RUN npm install
```

**4. Copy dependency files before code (for caching)**
```dockerfile
# ✅ Good — npm install is cached when only code changes
COPY package.json package-lock.json ./
RUN npm install
COPY . .
```

**5. Use Alpine or slim base images for smaller size**
```dockerfile
# ❌ Full image (~1GB)
FROM node:20

# ✅ Alpine image (~130MB)
FROM node:20-alpine
```

**6. Do not run containers as root (for security)**
```dockerfile
# Create a non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

**7. Combine related RUN commands to reduce layers**
```dockerfile
# ❌ Bad — 3 layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean

# ✅ Good — 1 layer
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean
```

### Container Best Practices

**1. Always name your containers**
```bash
docker run --name my-app nginx    # ✅ Easy to manage
docker run nginx                  # ❌ Random name assigned
```

**2. Use `--rm` for temporary containers**
```bash
docker run --rm nginx             # Auto-removed when stopped
```

**3. Use Docker Compose for multi-container apps**
Instead of multiple `docker run` commands, use a `docker-compose.yml` file.

**4. Use volumes for persistent data**
Always attach volumes to database containers to avoid data loss.

**5. Use custom networks for container communication**
Never rely on the default bridge network for multi-container setups.

---

## 24. Common Mistakes and How to Avoid Them

### Mistake 1: Forgetting to Expose Ports
**Symptom:** Container is running but you cannot access it in the browser.
**Fix:** Use the `-p` flag:
```bash
docker run -p 8080:80 nginx
```

### Mistake 2: Using `docker run` When You Meant `docker start`
**Symptom:** You have multiple duplicate containers.
**Remember:**
- `docker run` = Creates a NEW container every time
- `docker start` = Restarts an EXISTING stopped container

### Mistake 3: Forgetting Environment Variables
**Symptom:** Container starts but immediately crashes.
**Fix:** Check the image documentation on Docker Hub for required `-e` variables.
```bash
# MySQL REQUIRES a root password
docker run -e MYSQL_ROOT_PASSWORD=secret mysql    # ✅
docker run mysql                                   # ❌ Crashes!
```

### Mistake 4: Port Already in Use
**Symptom:** Error `bind: address already in use`
**Fix:** Use a different host port or stop the container using that port:
```bash
docker ps                              # Find what's using the port
docker stop <container-using-port>     # Stop it
```

### Mistake 5: Trying to Remove an Image with Existing Containers
**Symptom:** Error `image is being used by stopped container`
**Fix:** Remove the container first, then the image:
```bash
docker rm <container-id>
docker rmi <image-name>
```

### Mistake 6: Data Loss After Removing a Container
**Symptom:** Database is empty after recreating a container
**Fix:** Always use volumes for databases:
```bash
docker run -v my-data:/data/db mongo
```

### Mistake 7: Copying `node_modules` into the Image
**Symptom:** Image is very large, or wrong platform binaries
**Fix:** Add `node_modules` to `.dockerignore` and let `RUN npm install` handle it inside the container.

---

## 25. Quick Reference: All Docker Commands

### Image Commands

```bash
docker pull <image>                  # Download an image from Docker Hub
docker pull <image>:<tag>            # Download a specific version
docker images                        # List all local images
docker rmi <image>                   # Remove an image
docker rmi <image>:<tag>             # Remove a specific version
docker image prune -a                # Remove all unused images
docker build -t <name>:<tag> .       # Build image from Dockerfile
docker tag <image> <new-name>:<tag>  # Add a new tag to an existing image
docker push <username>/<image>:<tag> # Push image to Docker Hub
docker history <image>               # Show the layers of an image
```

### Container Commands

```bash
docker run <image>                      # Create + start a new container
docker run -d <image>                   # Run in background (detach mode)
docker run -it <image>                  # Run in interactive/terminal mode
docker run -it <image> /bin/bash        # Run and open bash shell
docker run -it <image> /bin/sh          # Run and open sh shell (Alpine)
docker run --name <n> <image>           # Set a custom name
docker run --rm <image>                 # Auto-remove when stopped
docker run -p <host>:<container> <img>  # Bind ports
docker run -e KEY=VALUE <image>         # Set environment variable
docker run -v <host>:<container> <img>  # Mount a bind mount
docker run -v <name>:<container> <img>  # Mount a named volume
docker run --network <net> <img>        # Connect to a network

docker ps                               # List running containers
docker ps -a                            # List ALL containers
docker start <name/id>                  # Start a stopped container
docker stop <name/id>                   # Stop a running container
docker restart <name/id>                # Restart a container
docker rm <name/id>                     # Delete a stopped container
docker rm -f <name/id>                  # Force delete (even if running)
docker container prune                  # Delete all stopped containers

docker logs <name/id>                   # View container logs
docker logs -f <name/id>               # Follow logs in real-time
docker logs --tail 50 <name/id>        # View last 50 lines of logs
docker exec -it <name/id> /bin/bash     # Enter a running container
docker exec -it <name/id> /bin/sh      # Enter (for Alpine containers)
docker inspect <name/id>                # View detailed container info
docker stats                            # Monitor resource usage (live)
docker top <name/id>                    # Show running processes in container
docker cp <name>:/path /local/path     # Copy file from container to host
```

### Network Commands

```bash
docker network ls                       # List all networks
docker network create <name>            # Create a new network
docker network inspect <name>           # View network details
docker network connect <net> <cont>     # Connect container to network
docker network disconnect <net> <cont>  # Disconnect container from network
docker network rm <name>                # Delete a network
docker network prune                    # Delete all unused networks
```

### Volume Commands

```bash
docker volume ls                        # List all volumes
docker volume create <name>             # Create a named volume
docker volume inspect <name>            # View volume details
docker volume rm <name>                 # Delete a specific volume
docker volume prune                     # Delete all unused volumes
```

### Docker Compose Commands

```bash
docker compose up -d                    # Start all services (background)
docker compose up --build -d            # Rebuild images and start
docker compose down                     # Stop + remove containers + network
docker compose down -v                  # Also remove volumes (deletes data!)
docker compose ps                       # List running compose services
docker compose logs                     # View logs of all services
docker compose logs -f                  # Follow logs in real-time
docker compose stop                     # Stop services (keep containers)
docker compose start                    # Start stopped services
docker compose restart                  # Restart all services
docker compose exec <service> /bin/sh   # Enter a running service container
```

### System / Cleanup Commands

```bash
docker system df                        # Show Docker disk usage
docker system prune                     # Remove all unused data (careful!)
docker system prune -a                  # Remove everything unused (very aggressive)
docker info                             # Show Docker system information
docker version                          # Show Docker version details
```

---

## 🎓 Key Concepts Summary

| Concept | One-Line Explanation |
|---|---|
| **Docker** | Platform to build, ship, and run containers |
| **Container** | Packaged app + dependencies = portable, isolated running unit |
| **Image** | Read-only blueprint for creating containers (like a class) |
| **Dockerfile** | Text file with instructions to build a custom Docker image |
| **Docker Hub** | Public registry to store and share Docker images |
| **Tag** | Version label for an image (e.g., `mysql:8.0`, `node:20-alpine`) |
| **Layers** | Images are built in stacked, cached, read-only layers |
| **Port Binding** | Mapping a host port to a container port (`-p 8080:80`) |
| **Docker Network** | Virtual network for containers to communicate by name |
| **Docker Compose** | Tool to define and run multi-container apps via a YAML file |
| **Volume** | Persistent storage that survives container restarts and deletions |
| **Bind Mount** | Direct mapping of a host folder to a container folder |
| **Bridge Driver** | Default network driver for container-to-container communication |
| **Docker Daemon** | Background service that manages containers, images, and networks |
| **Docker Client** | CLI tool you use to interact with the Docker Daemon |
| **Alpine** | A tiny Linux distribution used to make smaller Docker images |
| `.dockerignore` | File that tells Docker which files to exclude when building |
| **WORKDIR** | Sets the working directory inside the container |
| **ENTRYPOINT** | Like CMD but harder to override — defines the main executable |

---

## 📝 Docker Workflow Cheat Sheet

Here is the typical Docker workflow from start to finish:

```
  1. Write your application code
                │
                ▼
  2. Create a Dockerfile
                │
                ▼
  3. Create a .dockerignore file
                │
                ▼
  4. Build your image
     $ docker build -t my-app:1.0 .
                │
                ▼
  5. Test locally
     $ docker run -d -p 3000:3000 my-app:1.0
                │
                ▼
  6. Push to Docker Hub
     $ docker push username/my-app:1.0
                │
                ▼
  7. Team members pull and run
     $ docker pull username/my-app:1.0
     $ docker run -d -p 3000:3000 username/my-app:1.0
                │
                ▼
  8. For multi-container apps, use Docker Compose
     $ docker compose up -d
```

---

*Happy Dockerizing! 🐳*
