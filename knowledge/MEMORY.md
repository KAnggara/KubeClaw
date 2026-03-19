# MEMORY.md

## OpenClaw Kubernetes ChatOps Memory Model

This document defines how the OpenClaw agent stores, manages, and uses long-term memory during operations.

Memory allows the agent to improve operational efficiency by remembering previously discovered infrastructure context, user preferences, and operational patterns.

The memory system enables the agent to become progressively better at assisting Kubernetes operations without compromising safety.

---

## 1. Memory Philosophy

OpenClaw memory follows these principles:

- Persistent
- Structured
- Context-aware
- Infrastructure-safe
- Non-sensitive

Memory exists to improve operational awareness, not to replace real-time verification.

All operational decisions must still be verified using live Kubernetes data through tools like `kubectl`.

---

## 2. Memory Safety Rules

OpenClaw must not store sensitive information in memory.

Prohibited memory types include:

- API keys
- Access tokens
- Kubeconfig credentials
- Passwords
- Private certificates
- Kubernetes Secret values
- Authentication headers
- Internal credentials

Example:

**Bad:**
```
OPENAI_API_KEY=sk-xxxxxxxx
```

**Good:**
```
AI Provider used: OpenAI
```

Memory must always store **metadata**, not **secrets**.

---

## 3. Memory Categories

OpenClaw memory is divided into several categories.

### 3.1 User Memory

Stores knowledge about the users interacting with the agent.

Examples:
- Preferred namespaces
- Preferred environments
- Frequently requested operations
- Operational habits

Example Entry:
- User: Aji
- Preferred Namespace: fulka-dev
- Common Tasks:
  - check pods
  - restart deployments
  - inspect logs

---

### 3.2 Cluster Memory

Stores knowledge about the Kubernetes cluster.

Examples:
- Cluster name
- Common namespaces
- Important deployments
- Known system components

Example Entry:
- Cluster: production-cluster
- Namespaces:
  - fulka-dev
  - fulka-staging
  - fulka-prod
  - monitoring
- Critical Deployments:
  - krakend
  - chat-service
  - payment-gateway

---

### 3.3 Environment Memory

Stores information about known environments.

Examples:
- **Development**
  - Namespace: fulka-dev
- **Staging**
  - Namespace: fulka-staging
- **Production**
  - Namespace: fulka-prod

---

### 3.4 Operational Memory

Stores recent operational activities.

Examples:
- Restarts
- Deployments
- Debugging sessions
- Failure investigations

Example Entry:
- Date: 2026-03-07
- Action: Restart deployment krakend
- Reason: Pods stuck in CrashLoopBackOff
- Result: Deployment recovered

---

### 3.5 Tool Usage Memory

Stores commonly used operational tools and commands.

Commonly used commands:
- `kubectl get pods`
- `kubectl logs`
- `kubectl describe pod`
- `kubectl rollout restart deployment`
- `kubectl get events`

This helps the agent prioritize familiar operational workflows.

---

## 4. Memory Storage Model

Memory must be stored in a structured format.

Recommended formats:
- JSON
- YAML

Example structure:
```yaml
memory:
  user:
    name: Aji
    preferred_namespace: fulka-dev

  cluster:
    name: production-cluster

  frequent_commands:
    - kubectl get pods
    - kubectl logs
```

Memory storage must remain lightweight and structured.

---

## 5. Memory Lifecycle

Memory evolves over time.

### Creation
Memory can be created when:
- A new cluster environment is discovered
- A user repeatedly performs similar tasks
- A new namespace becomes frequently used
- Operational patterns emerge

### Update
Memory should be updated when:
- Cluster topology changes
- New namespaces appear
- Services are added or removed
- User preferences change

### Expiration
Some types of memory should expire to avoid stale information.

Recommended expiration rules:
- Operational Memory → 7 days
- Incident Memory → 30 days
- Cluster Memory → Persistent but updated
- User Preferences → Persistent

---

## 6. Memory Usage Strategy

When responding to requests, OpenClaw must follow this priority:

1. Live Kubernetes data
2. Current conversation context
3. Stored memory
4. Historical patterns

Memory should assist the agent in making faster decisions.

Example:
- **User request:** `restart krakend`
- **Agent memory:** Namespace: `fulka-dev`
- **Agent executes:** `kubectl rollout restart deployment krakend -n fulka-dev`

---

## 7. Memory Size Control

To prevent uncontrolled growth, memory limits must be applied.

Recommended limits:
- User Memory → up to 50 entries
- Operational Memory → last 100 events
- Incident Memory → last 50 incidents
- Cluster Memory → structured but unlimited

Memory cleanup should be run periodically.

---

## 8. Memory Review Process

The agent must periodically review stored memory.

Tasks include:
- Removing obsolete resources
- Validating cluster topology
- Cleaning up expired operational events
- Verifying stored namespaces still exist

Memory must remain accurate and relevant.

---

## 9. Memory Trust Hierarchy

Not all memory is equally reliable.

Trust priority:
1. Live Kubernetes API
2. Current user input
3. Recent operational memory
4. Historical memory

The agent must **always verify infrastructure status with kubectl before performing critical operations**.

---

## 10. Memory Usage Examples

**User request:**
```
check krakend pods
```

**Stored memory:**
Namespace: `fulka-dev`

**Agent executes:**
```
kubectl get pods -n fulka-dev -l app=krakend
```

Memory reduces the need for repetitive clarification and increases operational speed.

---

## 11. Memory Maintenance Responsibility

The OpenClaw agent is responsible for maintaining memory integrity.

Responsibilities include:
- Validating memory against actual cluster status
- Removing stale operational context
- Avoiding storage of sensitive data
- Keeping memory entries structured

Memory should assist infrastructure operations while maintaining safety and reliability.

---

End of MEMORY.md