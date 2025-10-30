# GitOps Network Policy Management

**Developer self-service network policies with automated security controls**

## 🎯 Overview

This repository demonstrates a GitOps approach to managing Cilium Network Policies with:
- ✅ **Developer self-service** for internal policies
- 🔒 **Automated security controls** for external connections
- 🤖 **GitHub Actions** for validation and approval
- 🚀 **ArgoCD** for continuous deployment

## 📁 Repository Structure

```
demo-gitops-network-policy/
├── .github/
│   ├── CODEOWNERS                      # Defines approval requirements
│   ├── workflows/                      # Automated validation & approval
│   │   ├── 01-validate-app.yml        # Validate application manifests
│   │   ├── 02-validate-policies.yml   # Validate network policies
│   │   ├── 03-security-check.yml      # Require security approval
│   │   └── 04-auto-approve-internal.yml # Auto-approve safe changes
│   └── PULL_REQUEST_TEMPLATE/         # PR templates
│       ├── external-policy.md
│       └── internal-policy.md
├── argocd/                             # ArgoCD App-of-Apps configuration
│   ├── bootstrap/
│   │   └── root-app-netpol-demo.yaml  # Root application (deploy manually once)
│   └── applications/
│       ├── tenant-a.yaml              # Tenant A application
│       └── tenant-b.yaml              # Tenant B application
├── tenant-a/                           # Tenant A namespace (isolated)
│   ├── apps/                          # Tenant A applications
│   └── policies/                      # Tenant A network policies
│       ├── 00-base/                   # 🔒 Security team only
│       ├── 10-internal/               # ✅ Dev team can modify
│       └── 20-external/               # ⚠️ Requires security approval
└── tenant-b/                           # Tenant B namespace (isolated)
    ├── apps/                          # Tenant B applications
    └── policies/                      # Tenant B network policies
        ├── 00-base/                   # 🔒 Security team only
        ├── 10-internal/               # ✅ Dev team can modify
        └── 20-external/               # ⚠️ Requires security approval
```

## 🔐 Security Model

### Policy Categories

| Directory | Owner | Approval Required | Auto-Merge |
|-----------|-------|-------------------|------------|
| **`tenant-*/policies/00-base/`** | Security Team | Security Team | ❌ Never |
| **`tenant-*/policies/10-internal/`** | Dev Team | None | ✅ Auto-approved |
| **`tenant-*/policies/20-external/`** | Dev Team proposes | Security Team | ❌ Manual |
| **`tenant-*/apps/`** | Dev Team | None | ✅ Auto-approved |

### What Developers Can Do

#### ✅ **Can Modify Freely** (Auto-approved)
- Application deployments, services, configmaps (`tenant-*/apps/`)
- Internal namespace policies (`tenant-*/policies/10-internal/`)
- Service-to-service communication within tenant namespace
- Any changes that don't affect external connectivity

#### ⚠️ **Can Propose** (Requires approval)
- External API connections (`tenant-*/policies/20-external/`)
- Cross-namespace communication
- Internet egress policies
- Security team reviews and approves

#### ❌ **Cannot Modify** (Protected)
- Base security policies (`tenant-*/policies/00-base/`)
- Default deny-all rules
- DNS policies
- Monitoring policies

## 🚀 Developer Workflow

### Scenario 1: Add Your Application (Tenant A)

```bash
# 1. Create your application manifest for Tenant A
cat > tenant-a/apps/frontend-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: tenant-a
spec:
  # ... your deployment spec
EOF

# 2. Create PR
git checkout -b tenant-a/add-frontend
git add tenant-a/apps/
git commit -m "tenant-a: Add frontend deployment"
git push origin tenant-a/add-frontend

# 3. GitHub Actions automatically:
#    ✅ Validates manifests
#    ✅ Auto-approves (no security review needed)
#    → Merge immediately!
#    → ArgoCD deploys to tenant-a namespace
```

### Scenario 2: Add Internal Service Communication (Tenant B)

```bash
# 1. Create internal policy for Tenant B
cat > tenant-b/policies/10-internal/api-to-cache.yaml <<EOF
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: api-to-cache
  namespace: tenant-b
spec:
  description: "Allow API to access Redis cache"
  # ... your policy spec
EOF

# 2. Create PR with internal policy template
git checkout -b tenant-b/policy/api-cache-communication
git add tenant-b/policies/10-internal/
git commit -m "tenant-b: Allow API to access Redis cache"
git push

# 3. GitHub Actions:
#    ✅ Validates policy syntax
#    ✅ Auto-approves (internal only)
#    → Merge after checks pass!
#    → ArgoCD deploys to tenant-b namespace
```

### Scenario 3: Request External Connection (Tenant A)

```bash
# 1. Create external policy with justification for Tenant A
cat > tenant-a/policies/20-external/api-to-stripe.yaml <<EOF
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: api-to-stripe
  namespace: tenant-a
  annotations:
    justification: "Payment processing integration"
spec:
  description: "Allow API to access Stripe"
  # ... your policy spec
EOF

# 2. Create PR using external policy template
git checkout -b tenant-a/policy/stripe-integration
git add tenant-a/policies/20-external/
git commit -m "tenant-a: Add Stripe API connection for payments

Destination: api.stripe.com:443
Protocol: HTTPS
Justification: Payment processing integration
Security: TLS 1.2+, API key authentication"
git push

# 3. GitHub Actions:
#    ✅ Validates policy
#    ⏳ Notifies security team
#    🔒 Waits for approval
#    → Merge after security team approves
#    → ArgoCD deploys to tenant-a namespace
```

## 🤖 Automated Workflows

### 1. App Validation (`01-validate-app.yml`)
**Triggers:** Changes to `tenant-*/apps/`
- ✅ YAML syntax validation
- ✅ Kubernetes manifest validation
- ✅ Best practices check
- ✅ Auto-comments on PR with results

### 2. Policy Validation (`02-validate-policies.yml`)
**Triggers:** Changes to `tenant-*/policies/`
- ✅ YAML syntax validation
- ✅ Cilium policy structure check
- 🛡️ Base policy protection (blocks modifications)
- 🏷️ Categorizes changes (internal/external/base)
- 🏷️ Auto-labels PRs
- ⚠️ Security anti-pattern detection

### 3. Security Check (`03-security-check.yml`)
**Triggers:** Changes to `tenant-*/policies/00-base/` or `tenant-*/policies/20-external/`
- 🔒 Requires security team approval
- 💬 Auto-comments with requirements
- 🔔 Notifies security team
- ❌ Blocks merge until approved

### 4. Auto-Approve Internal (`04-auto-approve-internal.yml`)
**Triggers:** Changes to `tenant-*/policies/10-internal/` or `tenant-*/apps/`
- ✅ Auto-approves if no base/external changes
- 🏷️ Adds "auto-approved" label
- 💬 Comments with merge instructions
- 🚀 Ready to merge immediately

## 📋 PR Templates

### External Policy Template
Use when requesting external connections. Includes:
- Destination information
- Justification
- Security considerations
- Testing plan
- Automatically tags security team

### Internal Policy Template
Use when modifying internal namespace policies. Includes:
- Services affected
- Reason for change
- Testing approach
- Notes that it will be auto-approved

## 🎓 Examples

### Example 1: Frontend → Backend Communication (Tenant A)

Create a policy file `tenant-a/policies/10-internal/frontend-to-backend.yaml`:

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: frontend-to-backend
  namespace: tenant-a
spec:
  description: "Allow frontend to connect to backend API"
  endpointSelector:
    matchLabels:
      app: frontend
  egress:
    - toEndpoints:
        - matchLabels:
            app: backend
      toPorts:
        - ports:
            - port: "8080"
              protocol: TCP
```

**Action:** Developers can add/modify this freely ✅

### Example 2: Backend → External API (Tenant B)

Create a policy file `tenant-b/policies/20-external/backend-to-stripe.yaml`:

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: backend-to-stripe
  namespace: tenant-b
  annotations:
    justification: "Payment processing"
    approved-by: "security-team"
spec:
  description: "Allow backend to access Stripe API for payments"
  endpointSelector:
    matchLabels:
      app: backend
  egress:
    - toFQDNs:
        - matchName: "api.stripe.com"
      toPorts:
        - ports:
            - port: "443"
              protocol: TCP
```

**Action:** Requires security team approval ⚠️

## 🔧 Setup Instructions

### 1. Configure Teams in GitHub

Create these teams in your organization:
- `@ciscocpa/dev-team` - Developers
- `@ciscocpa/security-team` - Security reviewers

### 2. Add Team Members

```bash
# Add developers
gh api orgs/ciscocpa/teams/dev-team/memberships/USERNAME -X PUT

# Add security team members
gh api orgs/ciscocpa/teams/security-team/memberships/USERNAME -X PUT
```

### 3. Configure Branch Protection

In GitHub repo settings → Branches → Add rule for `main`:
- ✅ Require pull request before merging
- ✅ Require status checks:
  - `Validate App Manifests`
  - `Protect Base Policies`
  - `Validate Policy Syntax`
  - `Require Security Team Approval` (for external/base changes)
- ✅ Require review from code owners
- ✅ Dismiss stale reviews

### 4. Deploy with ArgoCD (App-of-Apps Pattern)

This repository uses the **App-of-Apps pattern** for automated deployment:

#### Architecture

```
Root Application (argocd/bootstrap/root-app-netpol-demo.yaml)
    │
    ├── Monitors: argocd/applications/
    │
    ├── Child App 1: tenant-a.yaml
    │   └── Deploys: tenant-a/ directory → tenant-a namespace
    │       ├── apps/ → Applications for Tenant A
    │       └── policies/ → Network policies for Tenant A
    │
    └── Child App 2: tenant-b.yaml
        └── Deploys: tenant-b/ directory → tenant-b namespace
            ├── apps/ → Applications for Tenant B
            └── policies/ → Network policies for Tenant B
```

#### Initial Setup (One-time)

Deploy the root application to your ArgoCD instance:

```bash
# Method 1: Using kubectl
kubectl apply -f argocd/bootstrap/root-app-netpol-demo.yaml

# Method 2: Using ArgoCD CLI
argocd app create root-app-netpol-demo \
  --repo https://github.com/ciscocpa/demo-gitops-network-policy.git \
  --path argocd/applications \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace argocd \
  --sync-policy automated \
  --auto-prune \
  --self-heal

# Wait for root app to sync
argocd app wait root-app-netpol-demo
```

#### Verify Deployment

```bash
# Check root application status
argocd app get root-app-netpol-demo

# Check child applications
argocd app list | grep tenant-

# Expected output:
# tenant-a         Synced    Healthy   https://github.com/...
# tenant-b         Synced    Healthy   https://github.com/...

# Check deployed resources in tenant namespaces
kubectl get all,ciliumnetworkpolicies -n tenant-a
kubectl get all,ciliumnetworkpolicies -n tenant-b
```

#### How It Works

1. **Root Application** (`argocd/bootstrap/root-app-netpol-demo.yaml`)
   - Deployed manually once after cluster setup
   - Watches `argocd/applications/` directory
   - Automatically creates/updates child applications

2. **Child Applications** (`argocd/applications/*.yaml`)
   - Automatically created by root app
   - `tenant-a.yaml` - Deploys all YAML in `tenant-a/` → `tenant-a` namespace
   - `tenant-b.yaml` - Deploys all YAML in `tenant-b/` → `tenant-b` namespace
   - Auto-sync enabled with prune and self-heal
   - Each tenant is isolated in its own namespace

3. **GitOps Workflow**
   - Merge PR → ArgoCD detects change → Auto-deploys to respective tenant namespace
   - Changes in `tenant-a/` trigger sync to `tenant-a` namespace
   - Changes in `tenant-b/` trigger sync to `tenant-b` namespace
   - Changes in `argocd/applications/` automatically update child apps

#### Benefits

✅ **Zero-touch deployment** - Root app manages everything
✅ **Automatic updates** - Child apps sync on git push
✅ **Self-healing** - Resources recreated if manually deleted
✅ **Prune old resources** - Deleted files removed from cluster
✅ **Multi-tenant isolation** - Each tenant in separate namespace
✅ **Independent scaling** - Tenants can be managed independently
✅ **Clear separation** - Applications and policies organized by tenant

## 🐛 Troubleshooting

### PR Not Auto-Approving

**Check:**
1. Are you only modifying `tenant-*/apps/` or `tenant-*/policies/10-internal/`?
2. Did you accidentally change base or external policies?
3. Are all validation checks passing?
4. Are changes scoped to a single tenant?

### Security Check Failing

**Check:**
1. Are you modifying `tenant-*/policies/00-base/`? (Not allowed)
2. Are you modifying `tenant-*/policies/20-external/`? (Needs approval)
3. Has security team approved the PR?
4. Verify the correct tenant namespace in policy metadata

### Policies Not Applying

**Check:**
1. Is ArgoCD syncing? (`argocd app get tenant-a` or `argocd app get tenant-b`)
2. Are policies in correct tenant namespace?
3. Check Cilium status: `cilium status`
4. Check for ArgoCD sync errors: `argocd app sync tenant-a`
5. Verify namespace matches tenant name in policy metadata

### ArgoCD Not Syncing

**Check:**
1. Is root app healthy? `argocd app get root-app-netpol-demo`
2. Are child apps created? `argocd app list | grep tenant-`
3. Check ArgoCD logs: `kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller`
4. Verify repository access: `argocd repo list`

**Manual sync if needed:**
```bash
# Sync root app
argocd app sync root-app-netpol-demo

# Sync tenant apps
argocd app sync tenant-a
argocd app sync tenant-b
```

### ArgoCD Shows "OutOfSync"

**Common causes:**
1. Manual changes to cluster (use git, not kubectl!)
2. Resource modified by another controller
3. Check app diff: `argocd app diff tenant-a`
4. Force sync if needed: `argocd app sync tenant-a --force`
5. Verify correct tenant namespace in resource metadata

## 📚 Additional Resources

- [Cilium Network Policy Documentation](https://docs.cilium.io/en/stable/policy/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Open a pull request
5. Follow the PR template
6. Wait for automated checks
7. Merge after approval

## 📝 License

This project is provided as-is for demonstration purposes.

---

**Questions?** Contact @ciscocpa/dev-team or @ciscocpa/security-team
