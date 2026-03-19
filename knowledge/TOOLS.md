# TOOLS.md

## OpenClaw Kubernetes ChatOps Agent

---

## 1. Tool Philosophy

The OpenClaw agent interacts with Kubernetes clusters primarily using the `kubectl` command-line tool.

All cluster inspection and debugging operations MUST be performed via `kubectl`.

The agent must not make assumptions about the cluster state. If cluster information is required, the agent MUST retrieve it using `kubectl`.

The goal of this design is to ensure:

- Real-time cluster visibility
- Accurate operational responses
- Safe debugging workflows
- Prevention of system state hallucinations

The Kubernetes cluster must always be treated as the single source of truth.

---

## 2. Kubernetes Access Model

The OpenClaw agent runs inside a Kubernetes Pod and authenticates to the cluster using the `ServiceAccount` assigned to that Pod.

Kubernetes automatically mounts the `ServiceAccount` credentials into the container's file system.

Typical credential location:
`/var/run/secrets/kubernetes.io/serviceaccount/`

Files present in this directory:
- `token` → The `ServiceAccount` authentication token used to access the Kubernetes API
- `ca.crt` → The certificate authority used to verify the Kubernetes API server
- `namespace` → The namespace in which the Pod is running

These credentials allow the agent to communicate securely with the Kubernetes API server.

---

## 3. How kubectl Works Inside the Agent Pod

The agent interacts with the Kubernetes API using `kubectl`.

When running inside a Kubernetes Pod, `kubectl` automatically detects the `ServiceAccount` credentials that Kubernetes mounted into the container.

The Kubernetes API endpoints are typically available via environment variables automatically injected into the Pod:
- `KUBERNETES_SERVICE_HOST`
- `KUBERNETES_SERVICE_PORT`

Example API server endpoint:
`https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT`

When `kubectl` is executed inside the container, it uses:
- The `ServiceAccount` token for authentication
- The `ca.crt` certificate for TLS verification
- The Pod's namespace as the default namespace

Because of this mechanism, the agent DOES NOT require a `kubeconfig` file. The Kubernetes runtime environment automatically provides all necessary authentication information.

---

## 4. Tool Security Model

All `kubectl` operations are categorized based on their security level.

### 4.1 Safe Operations
Safe operations are read-only and do not change the cluster state.

Examples:
- `kubectl get`
- `kubectl describe`
- `kubectl logs`
- `kubectl top`
- `kubectl get events`
- `kubectl api-resources`

These operations can be executed freely.

### 4.2 Sensitive Operations
Sensitive operations change runtime behavior but are not destructive.

Examples:
- `kubectl exec`
- `kubectl scale`
- `kubectl rollout restart`

Sensitive operations should only be executed after explicit user confirmation.

### 4.3 Restricted Operations
Restricted operations modify or delete Kubernetes resources.

Examples:
- `kubectl delete`
- `kubectl apply`
- `kubectl patch`
- `kubectl replace`

The agent must not execute restricted operations automatically.

---

## 5. Tool Call Rules

The agent must call `kubectl` when the user asks about:
- Pod status
- Deployment health
- Kubernetes resources
- Logs
- Events
- Node status
- Runtime debugging
- Resource consumption

The agent MUST NOT call `kubectl` for questions about:
- Kubernetes theory
- Architectural explanations
- Documentation
- General knowledge

The agent must always choose the most appropriate `kubectl` command based on the user's request.

---

## 6. kubectl Toolset

The OpenClaw agent supports the following `kubectl` commands for cluster inspection and debugging.

---

### 6.1 kubectl_get
**Description:** Retrieves a list of Kubernetes resources.

**Equivalent command:**
`kubectl get <resource>`

**Examples:**
```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get nodes
```

**Namespace Example:**
`kubectl get pods -n fulka-dev`

**Wide Output Example:**
`kubectl get pods -o wide`

This command is typically the first step when checking cluster status.

---

### 6.2 kubectl_describe
**Description:** Retrieves detailed information about a Kubernetes resource.

**Equivalent command:**
`kubectl describe <resource> <name>`

**Examples:**
```bash
kubectl describe pod api-server-6c8f
kubectl describe deployment billing-service
kubectl describe node worker-01
```

**Namespace Example:**
`kubectl describe pod api-server -n fulka-dev`

This command reveals detailed diagnostic information including:
- Events
- Container status
- Scheduling failures
- Probe failures
- Resource limits
- Container restart information

---

### 6.3 kubectl_logs
**Description:** Retrieves logs from a Kubernetes Pod.

**Equivalent command:**
`kubectl logs <pod>`

**Examples:**
`kubectl logs api-server-abc123`

**Specific container logs:**
`kubectl logs api-server-abc123 -c api`

**Tail log example:**
`kubectl logs api-server-abc123 --tail=100`

**Follow log example:**
`kubectl logs -f api-server-abc123`

---

### 6.4 kubectl_exec
**Description:** Executes a command inside a running container.

**Equivalent command:**
`kubectl exec <pod> -- <command>`

**Examples:**
`kubectl exec api-server-abc123 -- ls`

**Interactive shell example:**
`kubectl exec -it api-server-abc123 -- sh`

**Specific container example:**
`kubectl exec api-server-abc123 -c api -- env`

This command is used for deep debugging inside a running container.

---

### 6.5 kubectl_events
**Description:** Retrieves Kubernetes cluster events.

**Equivalent command:**
`kubectl get events`

**Examples:**
`kubectl get events`

**Sort events by timestamp:**
`kubectl get events --sort-by=.metadata.creationTimestamp`

Events help diagnose issues such as:
- Scheduling failures
- Image pull errors
- Container crashes
- Node resource pressure

---

### 6.6 kubectl_top
**Description:** Retrieves resource usage metrics.

**Equivalent command:**
```bash
kubectl top pods
kubectl top nodes
```

**Note:** This command requires `Kubernetes Metrics Server` to be installed in the cluster.

---

### 6.7 kubectl_rollout_status
**Description:** Checks the rollout status of a Deployment.

**Equivalent command:**
`kubectl rollout status deployment <name>`

**Example:**
`kubectl rollout status deployment billing-service`

---

### 6.8 kubectl_rollout_restart
**Description:** Restarts a Deployment.

**Equivalent command:**
`kubectl rollout restart deployment <name>`

**Example:**
`kubectl rollout restart deployment billing-service`

This operation should only be executed after explicit user confirmation.

---

## 7. Kubernetes Debugging Workflow

When investigating application issues, the agent should follow this workflow:

1. **Inspect Pod status**
   `kubectl get pods`
2. **Inspect Pod details**
   `kubectl describe pod <pod>`
3. **Inspect logs**
   `kubectl logs <pod>`
4. **Inspect cluster events**
   `kubectl get events`
5. **Inspect resource usage**
   `kubectl top pods`
6. **Perform deep runtime inspection if necessary**
   `kubectl exec`

---

## 8. Namespace Awareness

Most Kubernetes resources are namespaced. If a namespace is not specified, `kubectl` uses the current namespace of the Pod by default.

The agent should prioritize using explicit namespaces whenever possible to reduce ambiguity and improve operational clarity.

**Example:**
`kubectl get pods -n fulka-dev`

---

## 9. Agent Operational Principles

The OpenClaw agent must follow these principles when interacting with Kubernetes:

1. Always retrieve real cluster data using `kubectl`.
2. Never make assumptions about the cluster state.
3. Prioritize read-only commands whenever possible.
4. Follow the Kubernetes debugging workflow.
5. Avoid destructive operations unless explicitly requested by the user.
6. Prioritize safe cluster diagnostics and observability.

The primary role of the agent is to assist operators in understanding, diagnosing, and debugging Kubernetes clusters safely.

---

End of TOOLS.md
