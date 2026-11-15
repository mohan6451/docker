# Dockerfile Notes: Issues I Faced and How I Fixed Them

While containerizing a Python/uWSGI microservice (payment), I ran into a series of real-world Docker issues. They took the container from crashing with Exit Code 127 to running smoothly at a fraction of its original size.

## 1. Mismatched build and runtime images

Even when two images come from the same base image family, the libraries they ship can be different.

**The issue:** I compiled `uwsgi` in the full `python:3.9` image and copied the artifacts into `python:3.9-alpine` or `python:3.9-slim`. The container crashed immediately at runtime:

```
error while loading shared libraries: libxml2.so.2: cannot open shared object file
```

**The root cause:** The full `python:3.9` image has `libxml2-dev` pre-installed, so `uwsgi` dynamically linked against `libxml2.so.2` during compilation. That shared library is missing in the minimal runtime images.

**The fix:** I used `python:3.9-slim` for both the builder and runtime stages. In stage 1, I installed `gcc` and `libc6-dev` temporarily with `--no-install-recommends`. Then I checked that no dynamic dependencies were missing:

```bash
ldd /usr/local/bin/uwsgi
```

## 2. Isolating dependencies with `--prefix`

**The fix:** In stage 1, install the dependencies into a separate folder:

```bash
pip3 install --no-cache-dir --prefix=/requiredlib -r requirements.txt
```

In stage 2, copy only `/requiredlib` into `/usr/local`. Compilers, build headers and pip caches are left behind in the builder stage.

## 3. Non-root user and uwsgi permission conflicts

**The issue:** Running the container as a non-root system user (`USER roboshop`) made `uwsgi` fail. `payment.ini` hardcoded `uid = 1001` and `gid = 1001`, which triggered a setuid permission error.

**The fix:** I removed the redundant `uid` / `gid` lines from `payment.ini`, so Docker's `USER` instruction handles process ownership cleanly. Alternatively, you can create the user with the same uid and gid that `payment.ini` specifies.

## 📉 The result

1. Image size shrank from 1.63 GB to 210 MB (~87% smaller).
2. Faster rebuilds thanks to Docker layer caching (`requirements.txt` is copied before the app code).
3. The app runs securely as a non-root user.

Every Exited (127) error is a great excuse to dig deeper into Linux shared libraries, file permissions and Docker internals! 💡

---

## Common startup failures

- **Exited (1) or Exited (127):** the container crashed on startup.
- **Build failed (`COPY failed: file not found`):** the file is missing from the build context.
- **Container deleted or stopped (Exited (0) / removed):** the main process finished, or the container was started with `--rm`.

---

## Writing a Dockerfile: understand the build process first

1. Select a suitable minimal base image (with a compiler if you need to build).
2. Copy the dependency list and install it.
3. Set non-secret `ENV` values. Inject secrets at runtime instead.
4. Copy the code and compile it.
5. Add the runtime command to finish the image.

The build process turns three things into one deliverable:

- **Raw code**
- **Libraries and their dependencies**
- **Compiler:** uses the libraries to convert the raw code into a bundle that is ready to serve

### What a production container needs, by language

| Language | Compiler output | What your production container needs |
|---|---|---|
| Go / C++ / Rust | Pure CPU binary | Almost nothing: a bare minimal OS (`scratch` or `alpine`) |
| Java / C# | Bytecode (`.jar` / `.dll`) | A runtime engine (JRE / .NET runtime) |
| TypeScript / React | Plain JavaScript and CSS (`dist/`) | A web server (Nginx) or Node.js runtime |
| Python | No separate compile step* | The Python interpreter (`python3`) |

\*Python packages with C extensions (like `uwsgi`) do compile during install, so they need `gcc` in the build stage.

---

## Standard process to run an application (without Docker)

1. Install the programming language the application is built with.
2. Create a directory and a dedicated user to avoid running as root.
3. Download the code and unzip it.
4. Install the dependencies needed to run the code.
5. Create a service to make the application accessible.
6. Make sure the database connection, schemas and tables exist.