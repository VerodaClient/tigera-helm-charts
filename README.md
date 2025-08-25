# Tigera Helm Charts

This repository provides Helm charts for managing **Tigera RBAC** (Calico/Tigera) permissions in Kubernetes.  
It includes two key charts:

- **`tigera-clusterroles`** → Deploys shared `ClusterRole` resources (`tigera-basic-ui`, `namespace-views`, `namespace-edits`) that remain persistent across installations.
- **`tigera-bindings`** → Manages per-user or per-group `RoleBinding` and `ClusterRoleBinding` resources, allowing scoped read-only or read-write access to specified namespaces.

---

## 📦 Charts Overview

### 1. `tigera-clusterroles`

This chart contains only the **ClusterRole** definitions:

- `tigera-basic-ui` → Required for the Tigera UI.
- `namespace-views` → Grants **read-only** access at namespace scope.
- `namespace-edits` → Grants **read-write** access at namespace scope.

**Use case:**  
Install this once to persist these roles independently of user-specific bindings.

---

### 2. `tigera-bindings`

This chart allows you to bind an existing **user** or **group** to the necessary roles:

- Creates a **ClusterRoleBinding** to `tigera-basic-ui`
- Creates **RoleBindings** in specified namespaces to either:
  - `namespace-views` (read-only)
  - `namespace-edits` (read-write)

---

## 🚀 Usage

### Step 1 - Install `tigera-clusterroles`

    helm install tigera-clusterroles ./tigera-clusterroles

This ensures the shared roles are installed once and remain available for any bindings.

---

### Step 2 - Install `tigera-bindings` for a User or Group

    helm install tigera-bindings ./tigera-bindings \
      --set subject.kind=Group \         # "User" or "Group"
      --set subject.name=dev-team \      # The user or group name
      --set accessMode=ro \              # "ro" (read-only) or "rw" (read-write)
      --set namespaces='{hipstershop,frontend}'

This generates:

- A **ClusterRoleBinding** named:  
  `{subject.name}-basic-ui-{accessMode}`
- One **RoleBinding per namespace** named:  
  `{subject.name}-{namespace}-{accessMode}`

---

## ⚙️ Configuration - `values.yaml`

    subject:
      kind: Group      # "User" or "Group"
      name: dev-team   # username or group name

    accessMode: ro     # "ro" for read-only or "rw" for read-write

    namespaces:
      - hipstershop
      - frontend

| Key            | Description                                       | Example           |
|---------------|---------------------------------------------------|-------------------|
| `subject.kind` | Type of subject to bind (`User` or `Group`)       | `Group`           |
| `subject.name` | Name of the user or group                        | `dev-team`        |
| `accessMode`   | Access level (`ro` or `rw`)                      | `rw`              |
| `namespaces`   | List of namespaces where RoleBindings are applied | `["frontend"]`    |

---

## 🗑️ Uninstallation Behavior

| Chart                  | Removes ClusterRoles? | Removes Bindings? |
|-----------------------|------------------------|--------------------|
| **`tigera-clusterroles`** | ✅ Yes - uninstalling this chart **removes** `ClusterRole`s | ❌ No |
| **`tigera-bindings`**    | ❌ No - shared roles remain untouched      | ✅ Yes - bindings only |

---

## 🧩 Example Workflows

### Example 1 - Deploy shared roles once

    helm install tigera-clusterroles ./tigera-clusterroles

### Example 2 - Create bindings for a **group** with **read-only** access

    helm install tigera-bindings-ro ./tigera-bindings \
      --set subject.kind=Group \
      --set subject.name=frontend-team \
      --set accessMode=ro \
      --set namespaces='{frontend,backend}'

### Example 3 - Create bindings for a **user** with **read-write** access

    helm install tigera-bindings-rw ./tigera-bindings \
      --set subject.kind=User \
      --set subject.name=jane.doe \
      --set accessMode=rw \
      --set namespaces='{dev,staging}'

---

## ❓ Why Separate Charts?

By splitting **cluster-scoped roles** and **subject-specific bindings**:

- ✅ Shared `ClusterRole`s are installed once and reused across teams
- ✅ Bindings can be added or removed without affecting other users
- ✅ Uninstalling bindings won't remove the shared roles

---

## 🤝 Contributing

Pull requests and issues are welcome!  
If you have feature requests, examples, or bug reports, please open an issue.

---

## 📜 License

*(Add your license details here if applicable)*

---

## 🔗 Useful Helm Commands

| Command                                           | Description            |
|--------------------------------------------------|------------------------|
| `helm install tigera-bindings ./tigera-bindings` | Install bindings      |
| `helm upgrade tigera-bindings ./tigera-bindings` | Upgrade bindings     |
| `helm uninstall tigera-bindings`                 | Remove bindings only |
| `helm uninstall tigera-clusterroles`             | Remove shared roles |
| `helm template tigera-bindings ./tigera-bindings`| Preview manifests before applying |

