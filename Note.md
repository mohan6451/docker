# Good to have Note about dockerfile and issues i faced in the process:
While containerizing a Python/uWSGI microservice (payment), I ran into a series of real-world Docker issues that took the container from crashing with Exit Code 127 to running smoothly at a fraction of its original size. 

1. mismatch build and runtime image, even both comes from the same base image the provided lib are different.

        The Issue: Compiling uwsgi in the full python:3.9 image and copying the artifacts into python:3.9-alpine or python:3.9-slim caused an immediate crash at runtime (error while loading shared libraries: libxml2.so.2: cannot open shared object file).
        
        The Root Cause: The full python:3.9 image had libxml2-dev pre-installed, so uwsgi dynamically linked against libxml2.so.2 during compilation—a shared library missing in minimal runtime images.
        
        The Fix: Aligned both builder and runtime stages on python:3.9-slim, installing gcc and libc6-dev temporarily in Stage 1 (--no-install-recommends), and verified zero missing dynamic dependencies using ldd /usr/local/bin/uwsgi.

2. Isolating Dependencies with --prefix
        
        The Fix: Used pip3 install --no-cache-dir --prefix=/requiredlib -r requirements.txt in Stage 1 and copied only /requiredlib into /usr/local in Stage 2. Compilers, build headers, and pip caches were completely left behind.

3. Non-Root User & uwsgi Permission Conflicts
        
        The Issue: Running the container securely as a non-root system user (USER roboshop) caused uwsgi to fail because payment.ini hardcoded uid = 1001 and gid = 1001, triggering a setuid permission error.
        
        The Fix: Removed the redundant uid/gid overrides in payment.ini so Docker natively handles process ownership cleanly under USER roboshop. but you can also create user with provided uid and gid. 

# 📉 The Result:
        
        1. Image size shrank from 1.63 GB down to 210 MB (~87% smaller)
        2. Faster rebuilds via proper Docker layer caching
        3. Secure execution as a non-root user
Every Exited (127) error is a great excuse to dig deeper into Linux shared libraries, file permissions, and Docker internals! 💡

### Exited (1) or Exited (127) (Container crashed on startup)
### Build failed (COPY failed: file not found)
### Container deleted or stopped (Exited (0) / removed)

## We can write the dockerfile if we understand the build process.
1. select the suitable minimal OS base image with complier
2. copy the dependent lib list and install
3. add the ENV or secrets (Inject the secrets at runtime instead)
4. copy code and compile        
5. add run time to create the image


raw code,       
lib+dependent lib,  
compiler : by using lib.'s convert raw code into bundles (ready to serve code)

### Language Type	    | Compiler Output	                | What Your Production Container Needs
Go / C++ / Rust 	| Pure CPU Binary (1s and 0s)       | Nothing! Just a bare minimal OS (scratch or alpine).
Java / C#	        | Bytecode (.jar / .dll)	        | A Runtime Engine (JRE / .NET Runtime).
TypeScript / React  | Plain JavaScript & CSS (dist/)	| A Web Server (Nginx) or Node.js runtime.
Python	            | No separate compile step*	        | The Python Interpreter (python3).


### stand process to run the application
1. install programming language on which the application build to compile the code
2. created one directory, user to cut the root access
3. downloaded code and unzip it
4. installed dependencies needed to run the code
5. create service to make the application accessible.
6. then make sure DB connections, schemas, tables exsits
