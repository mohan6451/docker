# How Docker Engine Works

**Docker Engine** is a client-server application that builds, runs, and manages lightweight containers by sharing the host machine's OS kernel rather than booting full virtual machines.

## Core Architecture
* **Docker CLI (`docker`):** The command-line interface used to type commands (e.g., `docker run`).
* **REST API:** The communication layer that connects the CLI and external tools to the background daemon.
* **Docker Daemon (`dockerd`):** The background service that manages images, containers, networks, and storage volumes.

## Execution Flow (`docker run`)
1. **Translate:** The CLI converts your terminal command into a REST API request.
2. **Receive:** The request travels over a local socket or network to `dockerd`.
3. **Pull:** The daemon checks for the requested image locally or downloads it from a registry (like Docker Hub).
4. **Run:** `dockerd` delegates container execution to lower-level runtimes (`containerd` and `runc`) to start the isolated process.

## Under the Hood
* **Namespaces:** Isolate each container's processes, network, and file system from the host and other containers.
* **Control Groups (cgroups):** Limit and monitor CPU, memory, and disk I/O usage per container.
* **Union File Systems:** Stack read-only image layers beneath a thin, writable container layer to save disk space.


#layering:
reduce the build time. 

Why Layering Is So Efficient

• Build Caching. Docker caches every layer during docker build. If you edit your application code (COPY . .) without changing package.json, Docker reuses the cached dependency layer (RUN npm install) instead of downloading everything again.
• Shared Disk Storage. If ten different containers run from the same ubuntu or node base image, they all share that exact same read-only base layer on disk rather than duplicating it ten times.
• Faster Transfers. When pushing or pulling updates to Docker Hub, Docker only transfers the specific layers that changed rather than re-uploading the entire image.

