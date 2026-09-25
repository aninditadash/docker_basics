# docker_basics

Alpine vs Debian slim vs full Debian vs Ubuntu: Alpine, Debian Slim, full Debian and Ubuntu are popular Docker base image choices that trade off image size, package management, and C-library compatibility.
Alpine Linux: Extremely small, around 5 MB. Uses musl libc instead of glibc. Has fast builds, minimal attack surface, and great for simple runtime environments. Uses apk package manager.
Debian Slim: Small, usually around 50–80 MB (larger than Alpine). Uses standard glibc. Offers great balance between a small footprint and high compatibility. Uses apt package manager and works with almost all standard Linux software.
Debian (Full): Larger, around 100–150 MB. Uses glibc. Rock-solid stability, comprehensive software repositories, and predictable, conservative updates. Unnecessarily large for standard production container deployments where only the runtime is needed. Both Debian and Ubuntu versions have slim tagged images. Image size is significantly reduced from the standard image by a process called slimification.
Ubuntu: Largest of the group, typically 70–180 MB+. Uses glibc. Debian based OS but it maintains its own versioning separately.

Create a calculator using a Python file and execute that file in the Alpine Linux environment present inside the container
------------------------------------------------------------------------------------------------------------------------
```
sh
docker build -t my-python-app:v1 -f Dockerfile.calculator .
docker run --rm -it --name my-calc-app my-python-app:v1 sh
docker run --rm -it --name my-calc-app my-python-app:v1
```

How To Optimize Docker Image

Crucial for building efficient and secure applications.
- Faster deployment: smaller images take less time to pull from registry and deploy, significantly speeding up CI/CD pipelines and scaling operations.
- Reduced Resource consumption: optimized images consume less disk space on host machines and in container registries, lowering storage costs and reducing network bandwidth usage.
- Enhanced Security: by removing unnecessary files, tools, and dependencies, we reduce the image's attack surface, making it less vulnerable to security exploits.
- Improved Scalability: Lighter images allow container orchestration platforms like Kubernetes to launch new container instances more quickly, enabling applications to scale efficiently in response to demand.

Dockerfile Best Practices for Efficient Image Building

Minimize the Number of Layers: Minimize number of levels in the Dockerfile by combining instructions into a single RUN directive for related instructions. Each instruction in a Dockerfile (like RUN, COPY, ADD) creates a new layer in the image. To reduce image size and improve build performance, consolidate related commands into a single RUN instruction using the && operator.
```bash
FROM base_image
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean
```
Use Minimal Base Images: Always start with smallest, most appropriate base image for the application. Using a full-featured OS image like ubuntu can add hundreds of megabytes of unnecessary files. Instead, opt for minimal images like alpine, distroless, or slim variants of official images.
```bash
FROM alpine:latest
```
Use Docker Multistage Builds: For speeding up image building process, use Docker's multi-stage builds, that separate build needs from the final running environment. Multistage builds are a powerful feature for creating lean production images. They allow us to use one container image with a full build environment (the "builder" stage) to compile code or build assets, and then copy only the necessary artifacts into a separate, minimal production image. This separates build-time dependencies (like compilers, and development libraries) from runtime dependencies, drastically reducing the final image size.
```bash
FROM build_image AS builder
# Build your application
FROM base_image
COPY --from=builder /app /app
```
Remove unnecessary files & Clean up Layers: After installing packages, always clean up caches and temporary files within the same RUN instruction. If we create a cleanup instruction in a new layer, the previous layer containing the unnecessary files will still be part of the image, and the size won't be reduced.
```sh
RUN apt-get install -y package \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
```
Use .dockerignore: prevents unnecessary files and directories from being sent to the Docker daemon during the build process. Crucial for avoiding the inclusion of local development files, logs, and build artifacts (node_modules, .git directory, etc.), which can unintentionally increase image size and pose security risks.

Docker Build Arguments: Use ARG to pass variables to the Docker build process. This allows for more flexible and reusable Dockerfiles, enabling you to customize builds for different environments (e.g., development vs. production) without altering the Dockerfile itself.
```sh
ARG BUILD_ENV
RUN if [ "$BUILD_ENV" = "production" ]; then \
        npm install --only=production; \
    else \
        npm install; \
    fi
```
Update Base Images: Regularly update base images to ensure we have the latest security patches and performance improvements. Specify a version tag (e.g., node:18-alpine) instead of latest to ensure the builds are predictable and reproducible.

Understanding Efficient Docker Caching Strategies

Caching helps in improving the effectiveness and speed of image development in Docker builds. Every command in a Dockerfile creates a new layer. Docker caches the layers from previous builds. If a Dockerfile instruction and its context have not changed, Docker will reuse the cached layer, making builds much faster. To leverage this effectively, structure the Dockerfile from the least frequently changing instructions to the most frequently changing ones.
* Install OS dependencies first.
* Copy package manager files (e.g., package.json, requirements.txt) and install dependencies.
* Copy your application source code last, as it changes most often.

```sh
# Set base image
FROM ubuntu:20.04 AS builder
# Install build dependencies (least likely to change)
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*
# Copy only necessary build files (potentially changing)
WORKDIR /app
COPY . .
# Build the application (most likely to change)
RUN make
# Final stage for production image
FROM alpine:latest
# Copy built application from the builder stage
COPY --from=builder /app/app /app
# Set entry point
ENTRYPOINT ["/app"]
```

Multistage Dockerfile: It is a feature introduced in Docker to address the challenge of creating lean and efficient container images Traditionally, Docker images used to contain all the dependencies, libraries, and tools required to run an application, leading to bloated images that consume unnecessary disk space and hence increase the deployment times. Now multistage builds allow developers to build multiple intermediate images within a single Dockerfile, and each intermediate image serves a specific purpose in the build process. 
