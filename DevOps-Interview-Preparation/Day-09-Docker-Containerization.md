# Day 9: Docker and Containerization — Interview Questions and Answers

## Interview questions

### Q1. Explain Docker architecture.
**Answer:**

The Docker client sends requests to the Docker daemon, which manages images,  
containers, networks, and volumes. Registries store images for distribution.  
Containers run isolated processes using host-kernel capabilities; they do  
not each include a separate guest kernel like conventional virtual machines.

For example, `docker run` asks the daemon to create and start a container from an  
image, pulling it first if needed. The host kernel provides isolation and resource  
control. I distinguish client connectivity problems, daemon problems, registry access, and  
application startup failures because they occur at different architectural layers.

### Q2. What is the difference between an image and a container?
**Answer:**

An image is a packaged filesystem and execution configuration built in  
layers. A container is a running or stopped instance with its own writable  
layer and runtime settings. Removing the container normally removes that  
layer; durable application data should use appropriate persistent storage.

An image can create many containers, each with its own process state and  
writable layer. Changing a running container does not automatically update the  
source Dockerfile or registry image. I make durable application changes in the  
build source and rebuild so future containers receive a reproducible version.

### Q3. What are the main Dockerfile instructions?
**Answer:**

`FROM` selects a base image, `WORKDIR` sets the working directory, `COPY`  
adds files, and `RUN` executes build-time commands. `ENV` supplies environment  
defaults, `USER` selects the runtime user, and `ENTRYPOINT`/`CMD`  
define startup behavior. `EXPOSE` documents a port but does not publish it.

I explain build time and runtime separately: `RUN` executes while creating the image,  
whereas the startup command executes when a container starts. I copy only necessary files  
and choose an explicit working directory. Environment-specific secrets should be supplied  
through approved runtime mechanisms instead of embedded in Dockerfile instructions.

### Q4. How do CMD and ENTRYPOINT differ?
**Answer:**

`ENTRYPOINT` commonly defines the main executable and `CMD` its default  
arguments, or a replaceable default command. Runtime arguments replace `CMD`  
and can be appended to an exec-form entrypoint. Prefer exec-form startup  
for predictable argument handling and signal delivery to the main process.

For example, an image can use an entrypoint for the application executable and a  
default command for its configuration arguments. Shell-form startup can introduce an  
extra shell that complicates signal handling. I verify shutdown behavior so the  
application receives termination signals and can finish work within its grace period.

### Q5. Why use multi-stage builds?
**Answer:**

Build and test with the required SDK/toolchain in one stage, then copy only runtime outputs  
into a smaller final stage. This reduces image size and unnecessary tooling in production.  
It does not automatically remove vulnerabilities or secrets copied into the final  
stage. See [Docker build practices](https://docs.docker.com/build/building/best-practices/).

For a compiled service, the build stage contains the compiler and dependency tools,  
while the final stage contains the compatible runtime and application output. I test  
that runtime image directly. A successful build stage does not prove native libraries,  
certificates, users, or filesystem permissions are correct in the final image.

### Q6. How do Docker layers affect build caching?
**Answer:**

A changed instruction or input can invalidate its cache and subsequent build  
steps. Copy dependency manifests before frequently changing source when the build  
allows it. Use `.dockerignore` to reduce unnecessary context, and preserve  
lockfile-driven dependency installation so cache speed does not undermine correctness.

For example, copying the entire source tree before installing dependencies can invalidate  
expensive installation steps for every small source change. I arrange layers around  
actual change frequency. At the same time, I rebuild intentionally for updated base images  
and security fixes instead of treating old cached layers as permanently acceptable.

### Q7. Compare volumes, bind mounts, and tmpfs mounts.
**Answer:**

Volumes are managed persistent storage objects. Bind mounts expose a  
specific host path and couple behavior to that host. A tmpfs mount stores  
data in memory on supported hosts and is not durable. Choose based on  
persistence, portability, security, backup, and performance requirements.

For a database, I choose persistent storage with a clear backup and recovery  
design rather than relying on the container layer. For local development, a bind  
mount can make source edits immediately available. I check ownership and host-path  
exposure because mounts can grant the container access to sensitive host files.

### Q8. How does Docker networking work?
**Answer:**

Networks connect containers and control communication patterns. Containers on  
a user-defined bridge network can resolve one another by appropriate container  
names or aliases. Publishing `-p 8080:80` maps a host port to a container port.  
`localhost` inside a container refers to that container's network namespace.

For two application containers, I create an intentional shared network and use the  
expected service name and port. Host publishing is needed for access through the  
host, not for every container-to-container flow. I also check listener binding because  
an application bound only to loopback may reject otherwise valid network traffic.

### Q9. What is Docker Compose used for?
**Answer:**

Compose defines a multi-container application, networks, volumes, and  
configuration in a YAML file. `docker compose up -d` creates and starts the  
defined services. Startup ordering alone does not prove dependency readiness;  
use health checks where supported and application-level retry logic.

A Compose file helps reproduce the same service relationships across developer machines  
and test environments. I keep credentials outside committed values and document  
required environment inputs. For dependency failures, the application should retry appropriately  
because the database can restart after the initial startup order has completed.

### Q10. What is Azure Container Registry?
**Answer:**

ACR is Azure's managed registry for container images and related artifacts. Build  
an image, tag it with the registry/repository reference, authenticate with an  
authorized identity, and push it. For deployments, use immutable versions or digests,  
restrict push/pull permissions, and consider private connectivity requirements.

I separate image push permission used by CI from pull permission used by runtime  
infrastructure. A successful login on a developer laptop does not validate the node's  
registry access. I verify image existence, digest, architecture, and network  
reachability from the actual deployment path before investigating application startup.

### Q11. How do you reduce container attack surface?
**Answer:**

Use maintained minimal bases, run as a nonroot user when possible, remove unnecessary  
packages, scan dependencies, and drop unneeded capabilities. Keep secrets  
out of image layers and build arguments; use supported build secret mounts. Apply  
resource limits and restrictive runtime settings according to the application.

I also avoid mounting the Docker socket into ordinary application containers because it  
can expose powerful host control. Runtime restrictions should be tested against real behavior  
so necessary writes or capabilities are understood. Security scanning, patching, and  
deployment identity controls complement a small image rather than replacing one another.

### Q12. Which commands help troubleshoot containers?
**Answer:**

`docker ps -a` shows state, `docker logs` output, `docker inspect`  
configuration, and `docker stats` resource usage. `docker exec` starts a  
diagnostic command in a running container when the image supports it.  
Inspect exit codes and health results before changing restart policies.

I start with state and logs, then inspect the relevant configuration instead of immediately  
opening an interactive shell. Minimal images may not include diagnostic utilities.  
If troubleshooting requires an alternate diagnostic container, I keep it controlled and  
avoid modifying the production container in ways that cannot be reproduced from source.

## Scenario-based questions

### S1. A container exits immediately. How would you investigate?
**Answer:**

Inspect `docker ps -a`, its exit code, and logs. Confirm the entrypoint, arguments, required files,  
configuration, permissions, and architecture. The primary process may have completed normally or  
failed; a container is not a background VM. Run the intended foreground process, correct the startup  
issue, and verify sustained health rather than keeping it alive with an unrelated sleep command.

I check whether the exit is a normal completion, a configuration error, an OOM  
event, or an external stop. Increasing the restart count cannot repair a consistently  
invalid command. After correction, I verify readiness and graceful shutdown  
as well as startup so the container behaves correctly throughout its lifecycle.

### S2. A containerized application cannot reach its database. What do you check?
**Answer:**

Verify the configured hostname and port from the application's network context. In Compose, use  
the database service name on the shared network rather than `localhost`. Check database readiness,  
listener address, firewall/network rules, credentials, and TLS. Separate name-resolution and TCP  
failures from authentication failures, then test an actual query after fixing the identified cause.

I use errors to separate name resolution, connection refusal, timeout, TLS, and  
authentication failures. The database may be running but not ready to accept queries.  
After correcting the connection path, I test reconnect behavior during a database  
restart so the application can recover without manual container replacement.

### S3. An image is 2 GB and slow to deploy. How do you improve it?
**Answer:**

Inspect image layers and determine whether build tools, caches, tests, or unnecessary files  
entered the runtime image. Add a multi-stage build and `.dockerignore`, choose a compatible smaller  
base, and avoid duplicating large files across layers. Scan and test the resulting runtime  
image, including required native libraries, and compare size, pull time, and startup behavior.

I compare build and runtime layers because deleting a large file in a later  
layer may not remove its bytes from earlier image history. A new multi-stage  
layout can prevent those bytes from entering the final image. I verify actual  
deployment pull time rather than judging success only by local build speed.

### S4. An image works locally but fails on AKS with an execution-format error. Why?
**Answer:**

Compare image architecture with the node architecture, especially ARM development machines  
versus x86 nodes. Inspect the image manifest and executable format, and check interpreter  
paths and line endings. Build for the target platform or publish a correct multi-platform  
image. Deploy by the new digest and verify it on a representative node before wider rollout.

I also ensure that every referenced base image and downloaded binary supports the  
selected platform. A multi-platform manifest is useful only when each image  
actually contains matching executables. After publishing, I inspect the manifest and  
test the startup command on the same architecture used by the target node pool.
