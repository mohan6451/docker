# Optimized Docker Image: Python/uWSGI Payment Microservice

A sample payment microservice (from a learning project) containerized with a multi-stage Dockerfile.
**Image size: 1.63 GB → 210 MB (~87% smaller)**, running as a non-root user.

## What's inside
- `Dockerfile`: multi-stage build on `python:3.9-slim`
- `payment.ini`, `payment.py`, `rabbitmq.py`: application files
- `Note.md`: issues I hit and how I fixed them
- Docker learning notes (below)

## Key techniques
- **Multi-stage build:** compile in stage 1, ship only the installed libraries in stage 2
- **Same base image for both stages:** avoids missing shared libraries (`libxml2.so.2`, Exit Code 127)
- **`pip --prefix=/requiredlib`:** copies only the installed packages, leaving compilers and caches behind
- **Non-root user:** runs as `roboshop`, with no hardcoded uid/gid in `payment.ini`
- **Layer caching:** `requirements.txt` is copied before the app code
- **No secrets in the image:** `AMQP_PASS` is passed at runtime

## Build and run
```bash
docker build -t payment:v1 .
docker images payment:v1

docker run -d --name payment \
  -e AMQP_PASS=<your-password> \
  -p 8080:8080 payment:v1
```

## Verify
```bash
docker run --rm payment:v1 id                  # shows roboshop, not root
docker inspect payment:v1 | grep -i AMQP_PASS  # prints nothing
```

## Before / after
<!-- paste your `docker images` screenshot here -->

---

# My Docker Learning Notes

## 1. Docker and Docker Engine

**Docker** is a containerization platform used to build, ship and run applications. Containers share the host machine's OS kernel instead of booting a full virtual machine, so they are lightweight and start fast.

**Docker Engine** is the client-server application that does the work. It has three parts:

- **Docker CLI (`docker`):** the command-line tool where you type commands such as `docker build` and `docker run`.
- **REST API:** the layer that carries CLI commands (and requests from other tools) to the daemon.
- **Docker Daemon (`dockerd`):** the background service that manages images, containers, networks and volumes.

### What happens on `docker run`
1. **Translate:** the CLI converts the command into a REST API request.
2. **Receive:** the request reaches `dockerd` over a local socket or the network.
3. **Pull:** the daemon checks for the image locally, or downloads it from a registry such as Docker Hub.
4. **Run:** `dockerd` hands execution to `containerd` and `runc`, which start the isolated process.

### Under the hood
- **Namespaces:** isolate each container's processes, network and file system.
- **Control groups (cgroups):** limit and monitor CPU, memory and disk I/O per container.
- **Union file systems (OverlayFS):** stack read-only image layers under a thin writable container layer.

### Role with the kernel
- **At start and stop:** the engine (via `containerd` and `runc`) asks the kernel to create namespaces and apply cgroup limits.
- **While running:** the container is a normal isolated process that talks to the host kernel directly through system calls, so there is little overhead.

## 2. Dockerfile, image, container

- **Dockerfile:** a text file with step-by-step instructions for building an environment: base OS, system packages, app code, runtime, libraries and metadata. The default name is `Dockerfile`, but a custom name can be used with `-f`.
- **Image:** a static, read-only template built from a Dockerfile.
- **Container:** a live, running instance of an image.

## 3. Dockerfile instructions

| Instruction | Purpose |
|---|---|
| `FROM <image>:<tag>` | Sets the base image |
| `WORKDIR <path>` | Sets the default working directory |
| `RUN <command>` | Runs a shell command at build time |
| `COPY <src> <dest>` | Copies files from the build context into the image |
| `ADD <src> <dest>` | Like `COPY`, but can also download from a URL and auto-extract local `.tar` files |
| `ARG KEY=value` | Build-time variable; not available inside the running container |
| `ENV KEY=value` | Variable available while the container runs |
| `LABEL key=value` | Adds metadata |
| `EXPOSE <port>` | Documents the port the app listens on; it does not publish it (use `-p` for that) |
| `ONBUILD <instruction>` | Adds a trigger that runs when another Dockerfile uses this image as its base |
| `USER <username>` | Switches the user for all later instructions and for the running container |
| `ENTRYPOINT ["executable", "param"]` | Sets the main executable; can be overridden only with `--entrypoint` |
| `CMD ["executable", "param"]` | Default command or default arguments; easily overridden at `docker run` |

> If both `ENTRYPOINT` and `CMD` are defined, `CMD` provides the default arguments to `ENTRYPOINT`.

### How to write a Dockerfile
1. Pick a suitable minimal base image (with the compiler if you need to build).
2. Copy the dependency list and install it.
3. Set non-secret `ENV` values. Inject secrets at runtime instead.
4. Copy the code and compile or bundle it.
5. Add the runtime command (`CMD` / `ENTRYPOINT`).

### What a production container needs, by language

| Language | Build output | Production container needs |
|---|---|---|
| Go / C++ / Rust | Native binary | Almost nothing: `scratch`, `alpine` or distroless |
| Java / C# | Bytecode (`.jar` / `.dll`) | A runtime (JRE / .NET runtime) |
| TypeScript / React | Static JS and CSS (`dist/`) | A web server (Nginx) or Node.js |
| Python | No separate compile step* | The Python interpreter |

\*Python packages with C extensions (like `uwsgi`) do compile during install, so they need `gcc` in the build stage.

## 4. Layering

An image is a stack of read-only layers. `RUN`, `COPY` and `ADD` create filesystem layers; other instructions only add metadata.

- **Build caching:** Docker caches each layer. If you change app code but not `requirements.txt`, the dependency layer is reused instead of reinstalled.
- **Shared storage:** ten containers from the same base image share one copy of its layers on disk.
- **Faster transfers:** push and pull only move the layers that changed.

Tips:
1. Combine related `RUN` commands to avoid extra layers.
2. Order instructions from least to most frequently changed (dependencies first, app code last).

## 5. Multi-stage builds

Use a separate build environment and runtime environment in one Dockerfile. Compile with heavy tools in the first stage, then copy only the final output into a small, clean image. The result is a smaller image, faster deployments and a smaller attack surface, because fewer packages means fewer things to patch.

See the `Dockerfile` in this repo for a working example.

## 6. `.dockerignore`

A plain-text file listing files and folders Docker should leave out of the build context. When you run `docker build`, the CLI collects the project folder (the build context), filters out anything matching `.dockerignore`, then sends the rest to the daemon.

- Syntax: file names (`config.json`), nested paths (`backend/config/local.json`) or folders (`node_modules/`), relative to the project root.
- **Faster builds:** skips heavy folders like `node_modules` or `venv`.
- **Smaller images:** keeps logs, tests and temp files out.
- **Better security:** keeps `.env` files, SSH keys and `.git` history out of image layers.

## 7. Docker Hub

A cloud-based container registry for storing, sharing and distributing images.

- **Pull:** download official base images (`ubuntu`, `node`, `postgres`, ...).
- **Push:** upload your own images for backup, team use or deployment.
- **Access control:** repositories can be public or private.

To push, the image name must include your Docker Hub username:

```bash
docker tag payment:v1 <dockerhub-username>/payment:v1
docker push <dockerhub-username>/payment:v1
```

## 8. Volumes

Containers are ephemeral: data written inside is lost when the container is removed. Volumes keep data outside the container lifecycle.

1. **Named volume:** managed by Docker in its own storage area on the host.
   `docker run -v mydata:/app/data ...` (inspect with `docker volume inspect mydata`)
2. **Anonymous volume:** same as above, but Docker generates a random name.
   `docker run -v /app/data ...`
3. **Bind mount:** maps an exact host file or folder into the container. Use an absolute host path.
   `docker run -v "$(pwd)/src:/app/src" ...`
4. **tmpfs mount:** stored only in the host's RAM and never written to disk. Good for sensitive session tokens or fast temporary caches.
   `docker run --tmpfs /app/cache ...`

## 9. Networks

An isolated virtual network that lets containers talk to each other, the host and the internet.

- **bridge (default):** a private network on the host. On a **custom** bridge network, containers can reach each other by container name through Docker's built-in DNS. (The default bridge does not provide name-based DNS.)
- **host:** removes network isolation; the container uses the host's IP and ports directly.
- **none:** disables networking completely.

```bash
docker network create app-net     # create a custom bridge network
```

## 10. Docker Compose

A tool for defining and running multi-container applications from one YAML file (`compose.yaml`).

**Problem it solves:** a real app has a frontend, several backend services and databases. Running `docker build` and `docker run` for each one by hand takes effort and invites mistakes. With Compose, you write the setup once and start everything with one command.

- **Automation:** replaces repeated manual commands with one repeatable workflow.
- **Consistency:** ports, environment variables and volumes are defined in one place.
- **Built-in networking:** services reach each other by service name.

```bash
docker compose up -d --build   # build images and start all services
docker compose ps              # list services
docker compose logs            # logs for all services
docker compose logs <service>  # logs for one service
docker compose down            # stop and remove containers and networks
```

## 11. Common commands

`docker run` = `docker pull` + `docker create` + `docker start`

```bash
docker pull <image>:<tag>            # download an image
docker create <image>                # create a container from an image
docker start <container>             # start a container

docker build -t <image>:<version> <path>   # build an image
docker run -d <image>:<version>            # run a container in the background
docker run -d --name my-nginx -p 8080:80 --network app-net \
  -v "$(pwd)/html:/usr/share/nginx/html" nginx:1.27

docker exec -it <container> bash     # open a shell (use sh on alpine images)
docker ps                            # running containers (add -a for all)
docker images                        # list images
docker inspect <container|volume|network|image>   # show metadata
docker volume ls                     # list volumes
docker network --help                # network commands
docker stop <container>              # stop a container
docker rm <container>                # remove a container
docker rmi <image>                   # remove an image
docker login                         # log in to Docker Hub
docker tag <image>:<version> <username>/<image>:<version>   # tag for pushing
```

## 12. Debugging

```bash
docker run --rm -it <image> bash       # temporary container, deleted on exit
docker logs <container>                # container logs
docker logs --tail 50 <container>      # last 50 log lines
docker compose logs --tail=50 <service>   # last 50 lines for a Compose service
```

Common failures:
- **Exited (127):** command or shared library not found. Check `ldd` on the binary and make sure build and runtime images match.
- **Exited (1):** the app crashed on startup. Check `docker logs`.
- **`COPY failed: file not found`:** the file is outside the build context or excluded by `.dockerignore`.
- **Exited (0) or container removed:** the main process finished, or the container was started with `--rm`.

See `Note.md` for the full debugging story behind this project.

## 13. Best practices

1. Use official small base images (`slim`, `alpine`) and pin the tag.
2. Use multi-stage builds.
3. Use volumes for persistent data and custom bridge networks for service communication.
4. Use `LABEL` and `EXPOSE` for documentation.
5. Optimize layers: combine related `RUN` steps and order from least to most frequently changed.
6. Never hardcode secrets in `ENV` or the Dockerfile; inject them at runtime.
7. Use `.dockerignore` to keep unnecessary and sensitive files out of the image.
8. Run as a non-root user.
9. Use `ENTRYPOINT` and `CMD` deliberately.