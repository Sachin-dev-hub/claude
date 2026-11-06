# Docker — Interview Q&A

---

## Q1. How do you write a good Dockerfile? (best practices)

- **Multi-stage builds:** use a full image to compile, copy only the result into a small runtime image. Result: much smaller image.
- Use **specific base image tags** (`python:3.11-slim`), not `latest` (reproducibility).
- **Order matters:** put rarely-changing layers first (install dependencies) so caching works — rebuilds are fast.
- **Layers:** every RUN/COPY creates a layer. Combine commands with `&&` to reduce layers.
- **Run as non-root user** (security), use `.dockerignore`.

```dockerfile
# Stage 1: build
FROM maven:3.8-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package

# Stage 2: runtime (small)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/app.jar app.jar
USER 10001
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## Q2. CMD vs ENTRYPOINT?

- **ENTRYPOINT:** the main command that always runs. Hard to override (`docker run image --flag` appends to it).
- **CMD:** default arguments or the default command. Easily overridden.
- **Together:** `ENTRYPOINT ["java", "-jar", "app.jar"]` + `CMD ["--port=8080"]` — user can pass different flags.
- **Rule:** ENTRYPOINT = what runs, CMD = default args.

---

## Q3. How do you secure container images?

1. **Scan images** for vulnerabilities — Trivy, Anchore, or AWS ECR scanning (JD mentions security). Fail the CI build if critical CVEs found.
2. Run as **non-root** user, minimal base image (distroless/slim).
3. Don't put secrets in the image (use K8s Secrets / Secrets Manager).
4. Use signed/trusted base images, pin versions.

---

## Q4. Docker vs Kubernetes — difference?

- **Docker** builds and runs containers (on one host).
- **Kubernetes** orchestrates containers across many machines — scaling, self-healing, load balancing, rolling updates.
- **Docker Compose** is for local multi-container apps on one machine; Kubernetes is for production at scale.

---

## Q5. Docker image layers — what are they?

- Each instruction in a Dockerfile (`FROM`, `RUN`, `COPY`) creates a layer.
- Layers are **cached and reused** — if a layer doesn't change, Docker reuses it.
- This is why you put `COPY pom.xml` before `COPY src` — dependency downloads get cached.
- Images are just a stack of read-only layers + a writable container layer at runtime.

---

## Quick commands to remember

| Command | What it does |
|---|---|
| `docker build -t myapp:1.0 .` | Build an image |
| `docker run -p 8080:8080 myapp` | Run a container |
| `docker push myapp:1.0` | Push image to registry |
| `docker images` | List images |
| `docker ps -a` | List containers (incl. stopped) |
| `docker logs <container>` | Container logs |
| `docker exec -it <container> sh` | Shell into a container |
| `docker system prune` | Clean unused images/containers |
| `docker scan <image>` | Scan for vulnerabilities |