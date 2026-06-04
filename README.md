# Kubernetes & DevOps — Scratch to Pro

> A complete, ordered learning path. Work top to bottom. Don't skip the prerequisites — most people who "fail at Kubernetes" actually failed at Linux, networking, or containers and only noticed when K8s exposed the gap.

---

## How to use this document

This is a **roadmap + reference**, not a tutorial you read once. Treat each phase as a milestone:

1. Read the concept.
2. Do the hands-on commands yourself on a real machine.
3. Break something on purpose, then fix it.
4. Only move on when you can explain the topic out loud without notes.

Estimated time scratch-to-job-ready: **6–9 months** part-time (10–15 hrs/week). Scratch-to-genuinely-senior: **2–3 years** of real work. Anyone selling you "DevOps in 30 days" is selling you something.

**The golden rule of DevOps learning:** you learn by operating things, not by watching things. Every phase below has a "Do this" block. The reading is 20% of the value; the doing is 80%.

---

## The big picture — where Kubernetes fits

DevOps is not a tool. It's a way of working where the people who build software also run it, with heavy automation so that releasing is boring and safe. Kubernetes is one tool in that world — the dominant way to run containers in production — but it sits on top of a whole stack you need first.

Here's the dependency chain. You cannot skip levels:

```
                    ┌─────────────────────────┐
                    │   Cloud / Production     │  ← AWS/GCP/Azure, managed K8s
                    └────────────┬────────────┘
                    ┌────────────┴────────────┐
                    │   Observability          │  ← logs, metrics, traces, alerts
                    └────────────┬────────────┘
                    ┌────────────┴────────────┐
                    │   CI/CD + GitOps         │  ← automate build→test→deploy
                    └────────────┬────────────┘
                    ┌────────────┴────────────┐
                    │   Kubernetes             │  ← orchestrate containers at scale
                    └────────────┬────────────┘
                    ┌────────────┴────────────┐
                    │   Containers (Docker)    │  ← package apps reproducibly
                    └────────────┬────────────┘
                    ┌────────────┴────────────┐
                    │   Linux + Networking +   │  ← the actual foundation
                    │   Git + a scripting lang │
                    └─────────────────────────┘
```

The rest of this document walks **up** this stack, phase by phase.

---

# PHASE 0 — Foundations (do not skip)

If you can already do everything here from memory, skim and move on. If not, this is where you spend your first 4–8 weeks. Everything later assumes this.

## 0.1 Linux

Kubernetes nodes are Linux. Containers are Linux. Your CI runners are Linux. You will live in a terminal. Use a real Linux environment: a cheap cloud VM (Ubuntu 22.04/24.04), WSL2 on Windows, or a local VM.

**What you must be able to do without googling:**

- Navigate and manipulate the filesystem: `cd`, `ls -la`, `pwd`, `cp`, `mv`, `rm`, `mkdir`, `find`, `tree`
- Read and edit files: `cat`, `less`, `tail -f`, `head`, `grep`, `nano`/`vim`
- Permissions: `chmod`, `chown`, what `rwx` and `755`/`644` mean, the difference between a user and root, `sudo`
- Processes: `ps aux`, `top`/`htop`, `kill`, `kill -9`, background jobs (`&`, `jobs`, `fg`)
- Pipes and redirection: `|`, `>`, `>>`, `2>&1`, `xargs`
- Text processing: `grep`, `sed`, `awk`, `cut`, `sort`, `uniq`, `wc`
- Networking from the shell: `curl`, `wget`, `ping`, `ss -tulpn`, `netstat`, `dig`, `nslookup`
- System services: `systemctl status/start/stop/enable`, reading logs with `journalctl -u <service>`
- Package management: `apt`/`dnf` install, update, remove
- Disk and resources: `df -h`, `du -sh`, `free -h`, `lsblk`
- Environment variables: `export`, `env`, `$PATH`, where they're set (`~/.bashrc`, `/etc/environment`)
- SSH: key-based login, `ssh-keygen`, `~/.ssh/config`, `scp`

**Do this:**
- Spin up a fresh Ubuntu VM. Create a non-root user, give it sudo, disable password SSH login and use keys only.
- Write a bash script that backs up a directory to a timestamped `.tar.gz` and deletes backups older than 7 days. Schedule it with `cron`.
- Tail a log file, grep it for errors, count them, and email/print a summary.

## 0.2 Networking

This is the single most under-learned skill and the one that separates juniors from seniors. Kubernetes is, more than anything, a networking system.

**Concepts you must understand cold:**

- The OSI model in practice (you mostly care about L3/L4/L7)
- IP addressing, subnets, CIDR notation (`10.0.0.0/16` — know how to read it)
- Private vs public IP ranges (10.x, 172.16–31.x, 192.168.x)
- TCP vs UDP, the 3-way handshake, ports, well-known ports (80, 443, 22, 53)
- DNS: what a resolver does, A/AAAA/CNAME records, TTL, how a hostname becomes an IP
- HTTP/HTTPS: methods, status codes, headers, what TLS actually does
- NAT, gateways, routing tables (conceptually)
- Firewalls and security groups (allow/deny by port/protocol/source)
- Load balancing (L4 vs L7), reverse proxies (nginx), what a proxy does
- Difference between a forward proxy and reverse proxy

**Do this:**
- On your VM, run a web server (`python3 -m http.server 8080`). From another machine, reach it. Now block it with a firewall rule, confirm it's blocked, then allow it again.
- Use `dig` to resolve a domain step by step. Look at the records. Change a `/etc/hosts` entry and watch resolution change.
- Draw, by hand, what happens when you type a URL and hit enter — DNS lookup, TCP connect, TLS handshake, HTTP request, response. If you can't draw it, you don't know it yet.

## 0.3 Git & version control

Everything in DevOps is in Git, including infrastructure (that's the whole point of GitOps later).

**Must know:** `clone`, `add`, `commit`, `push`, `pull`, `fetch`, branching, merging, rebasing (and the difference), resolving merge conflicts, `.gitignore`, pull requests / merge requests, what a remote is, `git log`, `git diff`, `git stash`, `git revert` vs `git reset` (and why force-push is dangerous on shared branches).

**Do this:** Make a repo, branch, commit, open a PR to yourself on GitHub/GitLab, create a deliberate merge conflict and resolve it. Learn the branching model your team uses (trunk-based or GitFlow).

## 0.4 A scripting language

Bash for glue, **Python** for anything real. You don't need to be a software engineer, but you must be able to write a script that calls an API, parses JSON, loops, handles errors, and reads/writes files.

**Do this:** Write a Python script that calls a public REST API (e.g. GitHub's), parses the JSON response, and writes a filtered summary to a file. Add error handling for when the API is down.

## 0.5 YAML

You will write thousands of lines of YAML. Learn its rules **now** because YAML's whitespace-sensitivity causes more Kubernetes errors than any actual K8s concept.

- Indentation is significant — **spaces only, never tabs**
- Key-value pairs, lists (`-`), nested maps
- Strings, numbers, booleans, null; when to quote
- Multi-document files (`---` separator)
- Anchors and references (`&` and `*`) — useful later

**Do this:** Write a YAML file with nested structure and a list of maps. Validate it with `yamllint` or an online linter. Deliberately use a tab and watch it break.

---

# PHASE 1 — Containers (Docker)

Kubernetes orchestrates containers. If you don't deeply understand containers, Kubernetes will feel like magic — and you can't debug magic. Spend 2–3 weeks here.

## 1.1 What a container actually is

A container is **not** a lightweight VM. It's a normal Linux process that the kernel has *isolated and limited* using two built-in kernel features:

- **Namespaces** — isolate what a process can *see* (its own process tree, network interfaces, mounts, hostname, users). The process thinks it's alone on the machine.
- **cgroups (control groups)** — limit what a process can *use* (CPU, memory, I/O).

A container image is just a tarball of a filesystem plus metadata. There's no hypervisor, no guest OS kernel — containers share the host's kernel. That's why they start in milliseconds and are tiny compared to VMs. It's also why a Linux container can't run on a Windows kernel without a Linux VM underneath (which is what Docker Desktop quietly does).

Internalize this sentence: **a container is an isolated, resource-limited process.** Everything else follows from it.

## 1.2 Core Docker concepts

- **Image** — a read-only template (your app + its dependencies + a minimal OS userland). Built in layers.
- **Container** — a running (or stopped) instance of an image, with a thin writable layer on top.
- **Dockerfile** — the recipe to build an image.
- **Registry** — where images live (Docker Hub, GitHub Container Registry, AWS ECR, etc.).
- **Layer** — each instruction in a Dockerfile creates a cached layer. Order matters for cache efficiency.
- **Volume** — persistent storage that outlives the container's writable layer.

## 1.3 Essential Docker commands

```bash
# Images
docker pull nginx:1.27           # download an image
docker images                    # list local images
docker build -t myapp:1.0 .      # build from Dockerfile in current dir
docker rmi myapp:1.0             # remove an image

# Containers
docker run -d -p 8080:80 --name web nginx   # run detached, map host:container port
docker ps                        # running containers
docker ps -a                     # all containers including stopped
docker logs -f web               # follow logs
docker exec -it web bash         # get a shell INSIDE a running container
docker stop web && docker rm web # stop and remove
docker stats                     # live resource usage

# Cleanup (you'll need this constantly)
docker system prune -a           # remove unused images/containers/networks
```

`docker exec -it <container> bash` (or `sh`) is the single most important debugging command. It puts you inside the container's isolated world so you can see what the process sees.

## 1.4 Writing a good Dockerfile

A bad Dockerfile produces a 1.5 GB image with your secrets baked in. A good one produces a 50 MB image that builds fast. Here's a solid multi-stage example for a Node app:

```dockerfile
# ---- Build stage ----
FROM node:20-slim AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci                      # install deps (cached unless package.json changes)
COPY . .
RUN npm run build

# ---- Runtime stage ----
FROM node:20-slim AS runtime
WORKDIR /app
ENV NODE_ENV=production
# copy ONLY what's needed to run, from the build stage
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
USER node                       # never run as root
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Dockerfile rules that matter:**
- **Multi-stage builds** — build tools stay in the build stage; the final image only ships the artifact. Huge size savings.
- **Order layers by change frequency** — copy `package.json` and install deps *before* copying source, so dependency layers stay cached when only your code changes.
- **Use specific, small base images** — `node:20-slim` or `alpine` or `distroless`, never `:latest`.
- **Don't run as root** — add a `USER` line.
- **Never bake secrets into images** — they're visible to anyone who pulls the image. Use runtime env vars / secret managers.
- **Use `.dockerignore`** — keep `node_modules`, `.git`, secrets out of the build context.
- **One process per container** — a container should do one job.

## 1.5 Docker Compose (multi-container locally)

For running several containers together on one machine (app + database + cache) during development:

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8080:3000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/app
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=app
    volumes:
      - dbdata:/var/lib/postgresql/data
volumes:
  dbdata:
```

```bash
docker compose up -d      # start everything
docker compose logs -f    # watch logs
docker compose down       # tear down
```

Notice `web` reaches `db` by the name `db` — Compose gives containers DNS names. This is your first taste of service discovery, which is central to Kubernetes.

## 1.6 Do this (Phase 1 project)

Take any small app (write a tiny Flask/Express "hello + counter backed by a database" app or grab one). Then:
1. Write a multi-stage Dockerfile. Get the image under 200 MB.
2. Run it with Compose alongside a Postgres or Redis container.
3. Make the app talk to the database over the Compose network.
4. Add a volume so the database data survives `docker compose down`.
5. Push your image to GitHub Container Registry or Docker Hub.
6. `docker exec` into the running container and prove to yourself the database connection works from inside.

If you can do all six from scratch without a tutorial open, you're ready for Kubernetes.

---

# PHASE 2 — Kubernetes Fundamentals

This is the heart of the document. Budget 6–10 weeks. Kubernetes (K8s — there are 8 letters between K and s) is a system for running containers across many machines: scheduling them, restarting them when they die, scaling them, networking them, and exposing them — automatically, declaratively.

## 2.1 The core mental model: declarative + reconciliation

This one idea explains *all* of Kubernetes. Hold onto it:

> You declare the **desired state** ("I want 3 copies of this app running"). Kubernetes continuously compares desired state to **actual state** and takes action to close the gap. This loop never stops.

You don't tell Kubernetes *how* to do things (imperative). You tell it *what you want* (declarative), in YAML, and controllers make reality match. Kill a pod? The controller notices actual (2) ≠ desired (3) and creates a new one. This reconciliation loop is the soul of the system.

## 2.2 Architecture — what's actually running

A Kubernetes **cluster** = a **control plane** + one or more **worker nodes**.

```
                         CONTROL PLANE (the brain)
   ┌──────────────────────────────────────────────────────────┐
   │  kube-apiserver   ← the front door; everything talks to it │
   │  etcd             ← the database; stores ALL cluster state │
   │  kube-scheduler   ← decides which node a new pod runs on   │
   │  controller-mgr   ← runs the reconciliation loops          │
   └──────────────────────────────────────────────────────────┘
                                │  (API calls)
        ┌───────────────────────┼───────────────────────┐
        ▼                       ▼                         ▼
   WORKER NODE 1           WORKER NODE 2            WORKER NODE 3
   ┌────────────┐         ┌────────────┐           ┌────────────┐
   │ kubelet    │         │ kubelet    │           │ kubelet    │  ← agent; runs pods
   │ kube-proxy │         │ kube-proxy │           │ kube-proxy │  ← networking rules
   │ container  │         │ container  │           │ container  │  ← containerd/CRI-O
   │ runtime    │         │ runtime    │           │ runtime    │
   │  [Pods]    │         │  [Pods]    │           │  [Pods]    │  ← your apps live here
   └────────────┘         └────────────┘           └────────────┘
```

**Control plane components:**
- **kube-apiserver** — the only component everything else talks to. You (via `kubectl`), the scheduler, the kubelets — all go through the API server. It validates requests and reads/writes etcd.
- **etcd** — a distributed key-value store; the single source of truth for the whole cluster's state. If you lose etcd without a backup, you lose the cluster. Back it up.
- **kube-scheduler** — watches for pods that have no node assigned and picks the best node for each (based on resource requests, affinity rules, taints, etc.).
- **kube-controller-manager** — runs the controllers (the reconciliation loops): node controller, replication controller, etc.
- **cloud-controller-manager** — integrates with your cloud provider (provisions load balancers, disks).

**Worker node components:**
- **kubelet** — the agent on each node. Talks to the API server, makes sure the containers it's told to run are actually running and healthy.
- **kube-proxy** — maintains network rules on the node so traffic reaches the right pods (implements Services).
- **container runtime** — the thing that actually runs containers (containerd or CRI-O; Docker itself was deprecated as a runtime in K8s 1.24+, though Docker *images* work fine).

You rarely touch these directly — but when something is broken, knowing which component is responsible tells you where to look. "Pod stuck in Pending" → scheduler couldn't place it. "Pod won't start on a node" → kubelet/runtime. "kubectl hangs" → API server or etcd.

## 2.3 Setting up a learning cluster

Don't start on a giant cloud cluster — start local and free.

- **kind** (Kubernetes IN Docker) — fastest, runs a cluster inside Docker containers. Great for CI and learning.
- **minikube** — single-node local cluster, very beginner-friendly, has nice addons.
- **k3s** — lightweight production-capable distro, great for a Raspberry Pi or small VM.

```bash
# Install kind + kubectl, then:
kind create cluster --name learn
kubectl cluster-info
kubectl get nodes          # you should see one node, Ready
```

`kubectl` (pronounced "cube-control" or "cube-cuddle") is your remote control for the cluster. Its config lives in `~/.kube/config` and points at a cluster + credentials + namespace, grouped into a "context." `kubectl config get-contexts` shows them; `kubectl config use-context <name>` switches.

## 2.4 kubectl — the commands you'll use every day

```bash
# Looking at things (you'll do this constantly)
kubectl get pods                       # in current namespace
kubectl get pods -A                    # all namespaces
kubectl get pods -o wide               # more columns (node, IP)
kubectl get pods -w                    # watch live
kubectl get all                        # pods, services, deployments, etc.
kubectl describe pod <name>            # full detail + EVENTS (read the events!)
kubectl logs <pod>                     # container logs
kubectl logs -f <pod>                  # follow
kubectl logs <pod> -c <container>      # specific container in a multi-container pod
kubectl logs <pod> --previous          # logs from the crashed previous instance

# Doing things
kubectl apply -f file.yaml             # create/update from YAML (declarative — USE THIS)
kubectl delete -f file.yaml            # remove what that file created
kubectl exec -it <pod> -- bash         # shell into a pod
kubectl port-forward <pod> 8080:80     # tunnel a pod's port to your laptop
kubectl scale deploy/<name> --replicas=5
kubectl rollout status deploy/<name>
kubectl rollout undo deploy/<name>     # roll back a bad deploy

# Namespaces
kubectl get ns
kubectl apply -f x.yaml -n <namespace>
kubectl config set-context --current --namespace=<ns>   # set default ns

# Explain — your built-in docs (underrated)
kubectl explain pod.spec.containers    # describes every field
```

**Two habits that will save you constantly:**
1. When something's wrong, `kubectl describe` the object and **read the Events section at the bottom**. It almost always tells you exactly what's failing (image pull error, failed scheduling, failed health check).
2. Prefer `kubectl apply -f` (declarative, from version-controlled YAML) over `kubectl create`/`run` (imperative, untracked). Your YAML lives in Git; the cluster is just an expression of it.

## 2.5 The objects — your core vocabulary

Everything in Kubernetes is an **object** you describe in YAML with the same four top-level fields:

```yaml
apiVersion: apps/v1     # which API group/version
kind: Deployment        # what type of object
metadata:
  name: my-app          # its name, labels, namespace
spec:
  ...                   # the desired state — the meat
```

Here are the objects in the order you should learn them.

### Pod — the smallest unit

A Pod is one or more containers that share a network (same IP, can reach each other on `localhost`) and storage. **You almost never create bare Pods** — they don't self-heal. But you must understand them because everything else manages Pods.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
```

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl delete pod nginx        # it does NOT come back — bare pods don't self-heal
```

Why multiple containers in one pod? The **sidecar** pattern: a main container plus a helper (log shipper, proxy). They're tightly coupled and always scheduled together.

### ReplicaSet — keeps N copies alive

Ensures a specified number of identical pods are always running. You rarely create these directly either — Deployments create them for you. But know that this is the controller doing the "actual ≠ desired → fix it" loop for pod count.

### Deployment — what you actually use for stateless apps

A Deployment manages ReplicaSets to give you **rolling updates, rollbacks, and self-healing**. This is your bread-and-butter object for web apps, APIs, workers.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3                    # desired: 3 copies
  selector:
    matchLabels:
      app: web                   # this Deployment manages pods with this label
  template:                      # the pod blueprint
    metadata:
      labels:
        app: web                 # pods get this label (must match selector above)
    spec:
      containers:
        - name: web
          image: myapp:1.0
          ports:
            - containerPort: 3000
          resources:
            requests:            # what the scheduler reserves
              cpu: "100m"        # 0.1 of a CPU core
              memory: "128Mi"
            limits:              # hard ceiling; exceed memory limit → killed (OOMKilled)
              cpu: "500m"
              memory: "256Mi"
```

```bash
kubectl apply -f deploy.yaml
kubectl get pods                 # 3 pods
kubectl delete pod <one-of-them> # watch a replacement appear instantly — self-healing!
kubectl set image deploy/web web=myapp:2.0   # triggers a rolling update
kubectl rollout status deploy/web
kubectl rollout undo deploy/web  # roll back if 2.0 was bad
```

**Labels and selectors** are the glue of Kubernetes. The Deployment finds its pods by label, not by name. Services (next) find pods the same way. Master this loosely-coupled, label-based wiring.

### Service — stable networking for ephemeral pods

Pods are mortal: they die, get rescheduled, get new IPs. You can't hardcode a pod IP. A **Service** gives a stable name and IP that load-balances across all pods matching a label selector. This is Kubernetes' service discovery.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web                     # routes to pods with label app=web
  ports:
    - port: 80                   # the Service's port
      targetPort: 3000           # the pods' port
  type: ClusterIP                # default — internal-only
```

**Service types — know all four:**
- **ClusterIP** (default) — reachable only inside the cluster. Other pods reach it at `web` or `web.<namespace>.svc.cluster.local`. This is how your services talk to each other.
- **NodePort** — opens a port on every node's IP (range 30000–32767). Crude external access; mostly for testing.
- **LoadBalancer** — provisions a real cloud load balancer with an external IP (works on AWS/GCP/Azure/managed K8s). The normal way to expose something externally at L4.
- **ExternalName** — maps the service to an external DNS name (CNAME). For pointing at things outside the cluster.

Inside the cluster, DNS is automatic: a pod can reach the `web` service just by the hostname `web`. That's the in-cluster DNS (CoreDNS) doing its job.

### Ingress — HTTP routing and the front door

A LoadBalancer per service gets expensive and gives you no path/host routing. **Ingress** is an L7 (HTTP/HTTPS) router: one entry point that routes by hostname and path to different services, and terminates TLS.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
```

Important: an Ingress object does nothing on its own. You need an **Ingress Controller** running in the cluster (nginx-ingress, Traefik, etc.) that reads Ingress objects and actually does the routing. (Note: the newer **Gateway API** is gradually replacing Ingress for advanced cases — learn Ingress first, then look at Gateway API.)


### ConfigMap & Secret — configuration and credentials

Never bake config or secrets into images. Inject them at runtime.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  FEATURE_FLAG: "true"
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:                      # stringData lets you write plaintext; K8s base64-encodes it
  DB_PASSWORD: "supersecret"
```

Consume them in a pod as env vars or mounted files:

```yaml
    spec:
      containers:
        - name: web
          image: myapp:1.0
          envFrom:
            - configMapRef:
                name: app-config
          env:
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: DB_PASSWORD
```

**Critical caveat:** Kubernetes Secrets are only **base64-encoded, not encrypted**, by default. Anyone with read access to the namespace can decode them. For real security: enable encryption-at-rest for etcd, use RBAC to lock down access, and use an external secret manager (HashiCorp Vault, AWS Secrets Manager, External Secrets Operator, Sealed Secrets). Never commit raw Secret YAML with real values to Git.

### Namespaces — logical partitions

Namespaces split a cluster into virtual sub-clusters (`dev`, `staging`, `prod`, per-team). They scope names, and you attach resource quotas and RBAC to them.

```bash
kubectl create namespace dev
kubectl apply -f app.yaml -n dev
kubectl get pods -n dev
```

Some objects are namespaced (Pods, Services, Deployments); some are cluster-wide (Nodes, PersistentVolumes, Namespaces themselves).

## 2.6 Health checks — probes

The kubelet uses probes to know whether your container is healthy. Configure these or you'll get silent failures and traffic routed to dead pods.

- **livenessProbe** — "is it alive?" If it fails, kubelet **restarts** the container. Use for detecting deadlocks.
- **readinessProbe** — "is it ready for traffic?" If it fails, the pod is **removed from Service endpoints** (no traffic) but not restarted. Use during startup or when a dependency is down.
- **startupProbe** — for slow-starting apps; disables the other probes until the app has started, so a slow boot isn't mistaken for a crash.

```yaml
          livenessProbe:
            httpGet:
              path: /healthz
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
```

## 2.7 Storage — persisting data

Containers are ephemeral; their filesystem dies with them. For data that must survive (databases, uploads):

- **Volume** — storage attached to a pod (many types: emptyDir, configMap, hostPath...).
- **PersistentVolume (PV)** — a piece of cluster storage (a disk), provisioned by an admin or dynamically.
- **PersistentVolumeClaim (PVC)** — a pod's *request* for storage ("I need 10Gi, ReadWriteOnce"). The claim binds to a PV.
- **StorageClass** — defines *how* to dynamically provision PVs (e.g. "AWS gp3 SSD"). Most clusters use dynamic provisioning: you create a PVC, the StorageClass provisions a real disk automatically.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data
spec:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

Mount it in a pod under `volumeMounts` + `volumes` referencing the PVC.

## 2.8 StatefulSet, DaemonSet, Job, CronJob

Deployments are for stateless apps. The other workload controllers:

- **StatefulSet** — for stateful apps that need stable identity and stable storage (databases, Kafka, anything clustered). Pods get stable, ordered names (`db-0`, `db-1`) and each keeps its own PVC across restarts. Used when pods are *not* interchangeable.
- **DaemonSet** — runs exactly one pod on every node (or a subset). For node-level agents: log collectors, monitoring agents, CNI plugins.
- **Job** — runs a pod to completion once (batch task, migration). Retries on failure until success.
- **CronJob** — runs Jobs on a schedule (cron syntax). For backups, periodic cleanup, reports.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  schedule: "0 2 * * *"          # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: backup-tool:1.0
              command: ["/bin/sh", "-c", "do-backup.sh"]
```

## 2.9 Do this (Phase 2 project)

Deploy a real two-tier app on your local cluster, fully from YAML in a Git repo:
1. A stateless web/API Deployment (3 replicas) with resource requests/limits and liveness + readiness probes.
2. A Postgres **StatefulSet** with a PVC so data survives pod restarts.
3. A ConfigMap for app config and a Secret for the DB password, injected as env vars.
4. ClusterIP Services wiring web → db.
5. An Ingress exposing the web app at a hostname (use an nginx ingress controller; map the host in `/etc/hosts`).
6. Now break things and watch recovery: `kubectl delete pod` a web pod (self-heals), do a rolling update to a bad image (watch it, then `rollout undo`), scale to 6 replicas, delete the db pod and confirm data persisted.
7. `kubectl describe` and `kubectl logs` your way through any failure. Get comfortable reading Events.

When you can do this without copying YAML from a tutorial — building each manifest from `kubectl explain` and memory — you've crossed from beginner to intermediate.

---

# PHASE 3 — Kubernetes, Production-Grade

Now you can run apps. This phase is about running them *safely, securely, and maintainably*. Budget 6–10 weeks.

## 3.1 Helm — package management

Writing raw YAML for every app, every environment, gets unmanageable fast (dev/staging/prod differ only in a few values, but you'd copy hundreds of lines). **Helm** is the package manager for Kubernetes. A **chart** is a templated, parameterized bundle of manifests; a **values.yaml** supplies the per-environment values; a deployed instance is a **release**.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-pg bitnami/postgresql        # install a packaged app
helm list                                     # see releases
helm upgrade my-pg bitnami/postgresql -f values.yaml
helm rollback my-pg 1                          # roll back to revision 1
helm uninstall my-pg
```

Learn to **write your own chart** (`helm create mychart`), templating manifests with `{{ .Values.x }}`, so you have one chart and per-environment values files. This is how most teams package their apps. (Alternatives/companions: **Kustomize**, which does template-free overlays and is built into kubectl via `kubectl apply -k`. Learn both; many teams use Kustomize for env overlays and Helm for third-party apps.)

## 3.2 RBAC — who can do what

**Role-Based Access Control** governs what users and workloads can do against the API server. Four objects:

- **Role** — permissions within a namespace ("can read pods in `dev`").
- **ClusterRole** — permissions cluster-wide.
- **RoleBinding** — grants a Role to a user/group/ServiceAccount in a namespace.
- **ClusterRoleBinding** — grants cluster-wide.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

**ServiceAccounts** are identities for *pods* (not humans). Each pod runs as a ServiceAccount; you bind Roles to it to grant the app only the API permissions it needs. Principle of least privilege — grant the minimum.

## 3.3 Security — the things that get clusters owned

Kubernetes is insecure by default in many ways. The essentials:

- **Don't run containers as root.** Set `securityContext: { runAsNonRoot: true, runAsUser: 1000 }`, drop Linux capabilities, use a read-only root filesystem where possible.
- **Pod Security Standards** (Baseline/Restricted) — enforce safe pod configs at the namespace level (replaced the old PodSecurityPolicies).
- **NetworkPolicies** — by default *all pods can talk to all pods*. NetworkPolicies are firewalls for pod-to-pod traffic; define them to restrict who can reach what. (Requires a CNI that supports them, e.g. Calico, Cilium.)
- **Image security** — scan images for vulnerabilities (Trivy, Grype), use minimal/distroless bases, sign images, pull from trusted registries, pin digests.
- **Secrets** — encrypt etcd at rest, use external secret managers, never commit secrets.
- **Limit the blast radius** — RBAC least privilege, separate namespaces, resource quotas, admission controllers (OPA/Gatekeeper or Kyverno to enforce policy — "no image without a digest," "every pod must have limits," etc.).
- **Keep the cluster patched** — CVEs in Kubernetes and the kubelet are real.

```yaml
# A default-deny NetworkPolicy: nothing reaches these pods unless explicitly allowed
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: prod
spec:
  podSelector: {}
  policyTypes: ["Ingress"]
```

## 3.4 Scheduling control — placing pods where you want

When you need control over which node runs which pod:

- **Resource requests/limits** — the scheduler packs pods onto nodes based on requests; limits cap usage. Set both; pods without requests cause noisy-neighbor and eviction problems.
- **nodeSelector / node affinity** — "run this on GPU nodes" / "spread across zones."
- **Taints & tolerations** — a taint repels pods from a node unless the pod *tolerates* it. Used to reserve nodes (e.g. taint GPU nodes so only GPU workloads land there).
- **Pod affinity / anti-affinity** — "keep replicas on different nodes" (high availability) or "co-locate with the cache."
- **Topology spread constraints** — spread pods evenly across zones/nodes.
- **PriorityClasses & preemption** — critical pods can evict less-important ones under pressure.

## 3.5 Autoscaling

- **HorizontalPodAutoscaler (HPA)** — adds/removes pod replicas based on CPU/memory/custom metrics. Needs metrics-server installed.
- **VerticalPodAutoscaler (VPA)** — adjusts a pod's requests/limits over time.
- **Cluster Autoscaler** — adds/removes *nodes* when pods can't be scheduled / nodes are underused (cloud clusters). **Karpenter** is a popular modern alternative on AWS.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70   # scale up when avg CPU > 70%
```

## 3.6 Cluster networking deeper (CNI)

Pod-to-pod networking is provided by a **CNI plugin** (Container Network Interface): Calico, Cilium, Flannel, Weave. The Kubernetes networking model requires that every pod gets its own IP and all pods can reach each other without NAT. The CNI implements that. **Cilium** (eBPF-based) is increasingly the standard for performance and advanced policy. You don't need to write CNI code, but understand: pod IPs come from the CNI, NetworkPolicies are enforced by the CNI, and CoreDNS provides in-cluster DNS.

## 3.7 Operators & CRDs — extending Kubernetes

Kubernetes is extensible. **CustomResourceDefinitions (CRDs)** let you define your own object types (e.g. `kind: PostgresCluster`). An **Operator** is a custom controller that watches those custom resources and manages complex stateful apps using the same reconciliation pattern ("here's a Postgres cluster I want; the operator handles backups, failover, upgrades"). You'll *use* operators long before you *write* one. Many tools (Prometheus, cert-manager, database operators) ship as operators.

## 3.8 Do this (Phase 3 project)

Take your Phase 2 app and harden it:
1. Package it as a Helm chart with separate `values-dev.yaml` and `values-prod.yaml`.
2. Add a dedicated ServiceAccount + minimal Role/RoleBinding for the app.
3. Set `securityContext` (non-root, read-only FS) on every container.
4. Add a default-deny NetworkPolicy plus explicit allows for web→db only.
5. Add an HPA that scales the web tier on CPU; load-test it and watch it scale.
6. Install **cert-manager** (your first operator) and get a TLS cert onto your Ingress.
7. Add a ResourceQuota and LimitRange to the namespace.

---

# PHASE 4 — CI/CD & GitOps

Kubernetes runs your apps; CI/CD gets your code *into* Kubernetes automatically, safely, repeatably. Budget 4–6 weeks.

## 4.1 The pipeline

- **CI (Continuous Integration)** — on every push: lint, run tests, build the container image, scan it for vulnerabilities, push it to a registry. The goal is to catch problems within minutes of a commit.
- **CD (Continuous Delivery/Deployment)** — take the built image and deploy it to an environment. Delivery = ready to deploy at the click of a button; Deployment = fully automatic to production.

A typical flow:

```
git push → CI runs tests → build image → scan → push to registry
        → update K8s manifests with new image tag → deploy to staging
        → run smoke tests → (manual gate?) → deploy to production
```

## 4.2 Tools

- **GitHub Actions** — easiest to start; YAML workflows in your repo. Learn this first.
- **GitLab CI** — integrated if you use GitLab.
- **Jenkins** — old, ubiquitous in enterprises, plugin-heavy. Know it exists; many jobs still use it.
- **Argo CD / Flux** — GitOps controllers (see below).

A minimal GitHub Actions pipeline:

```yaml
name: ci
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: make test
      - name: Build & push image
        run: |
          echo "${{ secrets.REGISTRY_TOKEN }}" | docker login ghcr.io -u $ --password-stdin
          docker build -t ghcr.io/me/app:${{ github.sha }} .
          docker push ghcr.io/me/app:${{ github.sha }}
      - name: Scan image
        run: trivy image ghcr.io/me/app:${{ github.sha }}
```

## 4.3 GitOps — the modern deploy model

GitOps is the key idea you should aim for: **Git is the single source of truth for the desired state of your cluster.** You don't run `kubectl apply` from a laptop or CI to deploy. Instead, a controller in the cluster (**Argo CD** or **Flux**) continuously watches a Git repo of manifests and makes the cluster match it. To deploy, you merge a PR that changes the image tag; the controller notices and rolls it out. To roll back, you revert the commit.

Why this wins: every change is reviewed, audited, and version-controlled; the cluster self-heals back to the declared state if someone makes a manual change; disaster recovery is "point Argo at the repo." This is the same reconciliation principle as Kubernetes itself, applied to deployment.

**Deployment strategies** to know: rolling update (default), blue-green (two environments, switch traffic), canary (send 5% of traffic to the new version, watch, ramp up). Tools like Argo Rollouts and Flagger automate canary/blue-green.

## 4.4 Do this (Phase 4 project)

1. Put your app in a Git repo with a GitHub Actions pipeline: test → build image → scan with Trivy → push to GHCR, tagged with the commit SHA.
2. Put your Kubernetes manifests (or Helm chart) in a Git repo.
3. Install **Argo CD** in your cluster and point it at that manifests repo.
4. Make the pipeline, on success, open a PR bumping the image tag. Merge it and watch Argo CD deploy automatically.
5. Make a bad change, watch it deploy, then `git revert` and watch it roll back. That's GitOps.

---

# PHASE 5 — Infrastructure as Code (IaC)

Clicking around a cloud console doesn't scale and isn't reproducible. IaC means your infrastructure (clusters, networks, databases, DNS) is defined in code, version-controlled, and applied automatically. Budget 4–6 weeks.

## 5.1 Terraform (provisioning) — learn this

**Terraform** (and its open fork **OpenTofu**) declaratively provisions cloud infrastructure. You describe resources in HCL; Terraform figures out the API calls and tracks what exists in a **state file**.

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "main" }
}

# A managed Kubernetes cluster (EKS) would be a module:
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "prod"
  cluster_version = "1.30"
  # ... subnets, node groups, etc.
}
```

```bash
terraform init      # download providers
terraform plan      # preview changes (ALWAYS read the plan)
terraform apply     # make it real
terraform destroy   # tear it all down
```

**Concepts to master:** providers, resources, variables, outputs, modules (reusable components), state (and **remote state** in S3 + locking — never commit state, it contains secrets), `plan` vs `apply`, drift, `import`. The discipline: nobody touches infra by hand; every change is a code change reviewed in a PR.

## 5.2 Configuration management — Ansible

Where Terraform *provisions* infrastructure, **Ansible** *configures* it (installs packages, edits configs, manages services) over SSH, agentlessly, using YAML playbooks. Less central in a pure-Kubernetes world (containers carry their own config), but still everywhere for VM fleets, bootstrapping nodes, and on-prem. Know the basics: inventory, playbooks, tasks, roles, idempotency.

## 5.3 Do this (Phase 5 project)

1. Use Terraform to provision a small managed Kubernetes cluster on a cloud you have free credits for (EKS/GKE/AKS), with remote state in object storage.
2. Provision a VPC/network, the cluster, and a node group entirely in code.
3. `terraform destroy` it when you're done (watch your bill!), then `apply` to recreate it identically. Reproducibility is the whole point.
4. Bonus: deploy your Phase 3 Helm chart onto this real cloud cluster.

---

# PHASE 6 — Observability

You can't operate what you can't see. When something breaks at 3 AM, observability is the difference between a 5-minute fix and a 5-hour outage. Budget 3–5 weeks. The "three pillars":

## 6.1 Metrics — Prometheus & Grafana

- **Prometheus** scrapes numeric metrics (request rate, error rate, latency, CPU, memory) from your apps and the cluster, stores them as time series, and lets you query with **PromQL**.
- **Grafana** visualizes those metrics in dashboards.
- **Alertmanager** fires alerts (to Slack, PagerDuty) when metrics cross thresholds.

Install the **kube-prometheus-stack** Helm chart and you get Prometheus + Grafana + Alertmanager + cluster dashboards out of the box. Instrument your app to expose `/metrics`. Learn the **RED method** (Rate, Errors, Duration) for services and **USE** (Utilization, Saturation, Errors) for resources.

## 6.2 Logs

Centralize logs so you're not `kubectl logs`-ing across 50 pods. Common stacks: **Loki + Grafana** (lightweight, label-based, pairs with Prometheus) or the **ELK/EFK stack** (Elasticsearch + Fluentd/Fluent Bit + Kibana). A DaemonSet collector (Fluent Bit) ships every pod's stdout to the store. Structure your logs as JSON.

## 6.3 Traces

**Distributed tracing** (OpenTelemetry + Jaeger or Tempo) follows a single request across multiple services, so you can see *which* service in a chain is slow. Essential in microservices. **OpenTelemetry** is the vendor-neutral standard for emitting metrics, logs, and traces — learn it.

## 6.4 SLOs and on-call

The professional layer: define **SLIs** (what you measure, e.g. % of requests under 200ms), **SLOs** (your target, e.g. 99.9%), and **error budgets** (how much you can fail before you stop shipping features and fix reliability). This is the core of **SRE (Site Reliability Engineering)** — read Google's free SRE books. Alert on symptoms users feel (error budget burn), not on every CPU blip, or you'll drown in noise and ignore real alerts.

## 6.5 Do this (Phase 6 project)

1. Install kube-prometheus-stack on your cluster. Open Grafana, explore the cluster dashboards.
2. Instrument your app to expose Prometheus metrics; build a Grafana dashboard showing its request rate, error rate, and p95 latency.
3. Add Loki and view your app's logs in Grafana alongside the metrics.
4. Write an alert ("error rate > 5% for 5 min") and make it fire by breaking the app. Route it to a Slack webhook.

---

# PHASE 7 — Cloud & Production Operations

Almost all production Kubernetes runs on a cloud's **managed** offering, because running the control plane yourself is a job nobody wants. Budget 4–6 weeks; pick one cloud and go deep, the concepts transfer.

## 7.1 Pick a cloud and learn its managed Kubernetes

- **AWS — EKS** (most common in the job market). Learn: IAM, VPC, EC2, S3, ELB/ALB, Route53, ECR, EKS, RDS, Secrets Manager, CloudWatch.
- **GCP — GKE** (best K8s experience, GKE Autopilot is excellent for learning). Learn: IAM, VPC, GCE, GCS, Cloud Load Balancing, GKE, Artifact Registry.
- **Azure — AKS** (common in enterprises). Learn: Entra ID, VNet, VMs, Blob Storage, AKS, ACR.

Managed K8s means the cloud runs the control plane (apiserver, etcd) for you; you manage worker nodes (or not, with Fargate/Autopilot) and your workloads. You still need the underlying cloud primitives: networking (VPC/subnets/security groups), identity (IAM — map cloud identities to K8s ServiceAccounts via IRSA/Workload Identity), storage classes backed by cloud disks, and load balancers wired to your Services/Ingress.

## 7.2 Cost, reliability, and the boring-but-critical stuff

- **Cost (FinOps)** — right-size requests/limits, use spot/preemptible nodes for fault-tolerant work, autoscale down off-hours, watch the bill daily when learning. A misconfigured LoadBalancer or oversized node group burns money fast.
- **High availability** — spread across availability zones, set pod anti-affinity and topology spread, use PodDisruptionBudgets so upgrades don't take all replicas down at once.
- **Backups & disaster recovery** — back up etcd (managed clouds do this), back up PVs (Velero), and rehearse a restore. Untested backups don't exist.
- **Upgrades** — Kubernetes releases ~3 minor versions/year and supports each for ~1 year. You will constantly be upgrading clusters and chasing deprecated APIs. Learn the upgrade process for your managed offering.
- **Capacity & quotas** — ResourceQuotas per namespace, cluster autoscaling, monitoring for resource saturation.

## 7.3 Do this (Phase 7 project)

Stand up a small but *realistic* production-shaped setup on your chosen cloud (mind the cost — tear down when idle):
1. Terraform the whole thing: network, managed cluster across 2–3 AZs, node group.
2. GitOps with Argo CD pulling from a Git repo.
3. Your app behind an Ingress with a real TLS cert (cert-manager + Let's Encrypt) on a real domain.
4. kube-prometheus-stack monitoring it, with alerts to Slack.
5. HPA + cluster autoscaler so it scales under load.
6. A documented runbook: how to deploy, roll back, scale, and recover.

This single project, done well and put on GitHub with a good README, is worth more in interviews than any certificate.

---

# Cross-cutting skills (the human + senior layer)

Tools get you in the door; these keep you employed and get you promoted.

- **Troubleshooting methodology** — form a hypothesis, test it, narrow down; check the layer below before blaming the layer above; read the actual error and events; reproduce before fixing. The best DevOps engineers are relentless, systematic debuggers.
- **Documentation & runbooks** — write down how to operate things so the team (and 3-AM-you) doesn't depend on one person's memory.
- **Communication** — you sit between dev and ops and security; you'll explain trade-offs to non-experts and push back on bad ideas diplomatically. This skill is often what separates senior from staff.
- **Incident response & blameless postmortems** — when prod breaks, restore service first, investigate after, and write up *what the system allowed to happen* without blaming a person.
- **Security mindset (DevSecOps)** — shift security left: scan in CI, least privilege everywhere, secrets management, supply-chain security (SBOMs, signed images).
- **Knowing when NOT to use Kubernetes** — for a simple app, a managed PaaS or a couple of VMs is often the right, cheaper, calmer answer. Senior engineers choose the simplest thing that works. Cargo-culting Kubernetes onto a problem that doesn't need it is a classic junior mistake.

---

# The debugging cheat-sheet (print this)

When (not if) things break, work through this:

```
POD WON'T START / CrashLoopBackOff
  kubectl describe pod <p>        → read Events (image pull? OOM? failed mount?)
  kubectl logs <p> --previous     → why did the last instance die?
  → ImagePullBackOff   = wrong image name/tag or registry auth
  → CrashLoopBackOff   = app exits on startup; check logs + config/secrets
  → OOMKilled          = exceeded memory limit; raise limit or fix leak
  → CreateContainerError = bad command/env/mount

POD STUCK IN Pending
  kubectl describe pod <p>        → Events show scheduling failure
  → insufficient cpu/memory       = nodes full; scale nodes or lower requests
  → no nodes match affinity/taint = fix selectors/tolerations
  → PVC not bound                 = no matching PV / StorageClass issue

CAN'T REACH MY SERVICE
  kubectl get endpoints <svc>     → empty? Service selector doesn't match pod labels
  kubectl get pods --show-labels  → verify labels match the selector
  kubectl exec into a pod, curl the service DNS name
  → check readinessProbe (failing readiness = removed from endpoints)
  → check NetworkPolicy isn't blocking it
  → Ingress not working? check the ingress controller pod + its logs

NODE PROBLEMS
  kubectl get nodes               → NotReady?
  kubectl describe node <n>       → conditions (DiskPressure, MemoryPressure)
  → check kubelet on the node, disk space, the container runtime

GENERAL
  kubectl get events -A --sort-by=.lastTimestamp   → cluster-wide recent events
  kubectl top pods / kubectl top nodes             → resource usage (needs metrics-server)
```

The meta-rule: **`describe` and read the Events, then read the logs.** 90% of answers are right there.

---

# Certifications (optional, but useful signals)

Not required, but they structure your learning and pass HR filters. In rough order:

- **KCNA** (Kubernetes and Cloud Native Associate) — entry-level, theory, good early checkpoint.
- **CKA** (Certified Kubernetes Administrator) — the big one. Hands-on, in a real terminal, time-pressured. Proves you can operate clusters. **This is the certificate that matters most for K8s.**
- **CKAD** (Certified Kubernetes Application Developer) — hands-on, focused on deploying/configuring apps. Good complement to CKA.
- **CKS** (Certified Kubernetes Security Specialist) — advanced, requires CKA first. Strong differentiator.
- **Cloud certs** — AWS (Solutions Architect Associate, then DevOps Engineer Pro), or the GCP/Azure equivalents. A cloud cert + CKA is a strong combo.
- **Terraform Associate** — quick, well-respected for IaC.

Certs prove knowledge; **projects prove you can do the job.** Do both, lead with projects.

---

# How to actually get hired

1. **Build a portfolio on GitHub.** The Phase 7 project, well-documented, is your centerpiece. Also: a repo of your IaC, a Helm chart you wrote, a CI/CD pipeline, a writeup of a problem you debugged. Public, with READMEs that explain the *why*.
2. **Write about what you learn.** A blog or even good READMEs. "Here's how I set up GitOps with Argo CD and what broke" signals real understanding.
3. **Contribute to open source** — even docs fixes to CNCF projects. It's resume gold and teaches you real collaboration.
4. **Target roles honestly.** "DevOps Engineer," "Platform Engineer," "SRE," "Cloud Engineer" overlap heavily. Junior roles exist but often expect 1–2 years of *some* IT/sysadmin/dev background — leverage whatever you have.
5. **Interview prep** — expect: Linux/networking fundamentals, "walk me through what happens when you deploy X," "this pod is in CrashLoopBackOff, debug it" (live), system design ("design a deployment pipeline / a highly-available service"), and behavioral (incident stories). The live debugging and "explain the fundamentals" rounds are where prepared candidates win.

---

# A realistic timeline

| Months | Focus | Milestone |
|---|---|---|
| 1–2 | Phase 0–1: Linux, networking, Git, Docker | Containerize and run a multi-service app with Compose |
| 3–4 | Phase 2: K8s fundamentals | Deploy a two-tier stateful app on a local cluster from YAML, debug failures |
| 5 | Phase 3: production K8s, Helm, security | Hardened, Helm-packaged app with RBAC, NetworkPolicies, HPA |
| 6 | Phase 4–5: CI/CD, GitOps, Terraform | Full pipeline + Argo CD + Terraform-provisioned cluster |
| 7 | Phase 6: observability | Monitored app with dashboards and working alerts |
| 8–9 | Phase 7: cloud production + CKA prep | The capstone cloud project + pass CKA |
| Ongoing | Cross-cutting skills, real incidents, depth | A real job, where the actual learning begins |

Slower is fine. **Depth beats speed.** Someone who deeply understands 70% of this is far more employable than someone who skimmed 100%.

---

# Curated resources

**Official / free:**
- Kubernetes docs (kubernetes.io/docs) — genuinely excellent; the Concepts and Tasks sections are your reference.
- `kubectl explain <resource>` — built-in field docs, always current.
- Google SRE Books (sre.google/books) — free, foundational for the reliability mindset.
- killercoda.com — free interactive K8s scenarios in your browser.
- CNCF landscape (landscape.cncf.io) — the map of the whole ecosystem (overwhelming; use it to look things up, not to learn from).

**Practice environments:**
- kind / minikube / k3s locally (free).
- Cloud free tiers + credits (mind the bill).
- killer.sh — the official CKA/CKAD exam simulator (comes with exam registration).

**Habits:**
- Keep a personal `runbook.md` and a `commands.md` of things you've learned and fixed.
- When you fix a bug, write down the symptom → cause → fix. Future you will thank present you.
- Re-derive YAML from `kubectl explain` instead of copy-pasting. The struggle is the learning.

---

# Final advice

Kubernetes is broad, and the ecosystem around it is genuinely overwhelming — the CNCF landscape has hundreds of tools. **You do not need to learn all of them.** Master the core (the path in this document), and learn specific tools when a real problem demands them. The engineers who succeed aren't the ones who memorized the most tools; they're the ones who understand the *fundamentals* — Linux, networking, the declarative/reconciliation model, how a request flows through the system — deeply enough to reason about anything new.

Build things. Break them. Fix them. Write down what you learned. Repeat. That loop, sustained over months, is the entire secret.

Good luck. The fact that you want the whole path mapped out means you're already approaching this the right way.
