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
├── apps/                               # Application manifests (add your app YAMLs here)
└── policies/
    ├── 00-base/                        # 🔒 PROTECTED - Security team only
    │                                   #    - Default deny-all
    │                                   #    - DNS resolution
    │                                   #    - Prometheus metrics
    ├── 10-internal/                    # ✅ Dev team can modify
    │                                   #    - Internal service communication
    │                                   #    - Service-to-service policies
    └── 20-external/                    # ⚠️ Requires security approval
                                        #    - External API connections
```

## 🔐 Security Model

### Policy Categories

| Directory | Owner | Approval Required | Auto-Merge |
|-----------|-------|-------------------|------------|
| **`00-base/`** | Security Team | Security Team | ❌ Never |
| **`10-internal/`** | Dev Team | None | ✅ Auto-approved |
| **`20-external/`** | Dev Team proposes | Security Team | ❌ Manual |
| **`apps/`** | Dev Team | None | ✅ Auto-approved |

### What Developers Can Do

#### ✅ **Can Modify Freely** (Auto-approved)
- Application deployments, services, configmaps (`apps/`)
- Internal namespace policies (`policies/10-internal/`)
- Service-to-service communication within namespace
- Any changes that don't affect external connectivity

#### ⚠️ **Can Propose** (Requires approval)
- External API connections (`policies/20-external/`)
- Cross-namespace communication
- Internet egress policies
- Security team reviews and approves

#### ❌ **Cannot Modify** (Protected)
- Base security policies (`policies/00-base/`)
- Default deny-all rules
- DNS policies
- Monitoring policies

## 🚀 Developer Workflow

### Scenario 1: Add Your Application

```bash
# 1. Create your application manifest
cat > apps/frontend-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: demo-app
spec:
  # ... your deployment spec
EOF

# 2. Create PR
git checkout -b feature/add-frontend
git add apps/
git commit -m "Add frontend deployment"
git push origin feature/add-frontend

# 3. GitHub Actions automatically:
#    ✅ Validates manifests
#    ✅ Auto-approves (no security review needed)
#    → Merge immediately!
```

### Scenario 2: Add Internal Service Communication

```bash
# 1. Create internal policy
cat > policies/10-internal/api-to-cache.yaml <<EOF
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: api-to-cache
  namespace: demo-app
spec:
  description: "Allow API to access Redis cache"
  # ... your policy spec
EOF

# 2. Create PR with internal policy template
git checkout -b policy/api-cache-communication
git add policies/10-internal/
git commit -m "Allow API to access Redis cache"
git push

# 3. GitHub Actions:
#    ✅ Validates policy syntax
#    ✅ Auto-approves (internal only)
#    → Merge after checks pass!
```

### Scenario 3: Request External Connection

```bash
# 1. Create external policy with justification
cat > policies/20-external/api-to-stripe.yaml <<EOF
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: api-to-stripe
  namespace: demo-app
  annotations:
    justification: "Payment processing integration"
spec:
  description: "Allow API to access Stripe"
  # ... your policy spec
EOF

# 2. Create PR using external policy template
git checkout -b policy/stripe-integration
git add policies/20-external/
git commit -m "Add Stripe API connection for payments

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
```

## 🤖 Automated Workflows

### 1. App Validation (`01-validate-app.yml`)
**Triggers:** Changes to `apps/`
- ✅ YAML syntax validation
- ✅ Kubernetes manifest validation
- ✅ Best practices check
- ✅ Auto-comments on PR with results

### 2. Policy Validation (`02-validate-policies.yml`)
**Triggers:** Changes to `policies/`
- ✅ YAML syntax validation
- ✅ Cilium policy structure check
- 🛡️ Base policy protection (blocks modifications)
- 🏷️ Categorizes changes (internal/external/base)
- 🏷️ Auto-labels PRs
- ⚠️ Security anti-pattern detection

### 3. Security Check (`03-security-check.yml`)
**Triggers:** Changes to `policies/00-base/` or `policies/20-external/`
- 🔒 Requires security team approval
- 💬 Auto-comments with requirements
- 🔔 Notifies security team
- ❌ Blocks merge until approved

### 4. Auto-Approve Internal (`04-auto-approve-internal.yml`)
**Triggers:** Changes to `policies/10-internal/` or `apps/`
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

### Example 1: Frontend → Backend Communication

Create a policy file `policies/10-internal/frontend-to-backend.yaml`:

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: frontend-to-backend
  namespace: demo-app
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

### Example 2: Backend → External API

Create a policy file `policies/20-external/backend-to-stripe.yaml`:

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: backend-to-stripe
  namespace: demo-app
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

### 4. Deploy with ArgoCD

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demo-gitops-network-policy
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/ciscocpa/demo-gitops-network-policy.git
    targetRevision: main
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: demo-gitops-network-policy
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## 🐛 Troubleshooting

### PR Not Auto-Approving

**Check:**
1. Are you only modifying `apps/` or `policies/10-internal/`?
2. Did you accidentally change base or external policies?
3. Are all validation checks passing?

### Security Check Failing

**Check:**
1. Are you modifying `policies/00-base/`? (Not allowed)
2. Are you modifying `policies/20-external/`? (Needs approval)
3. Has security team approved the PR?

### Policies Not Applying

**Check:**
1. Is ArgoCD syncing? (`argocd app get demo-gitops-network-policy`)
2. Are policies in correct namespace?
3. Check Cilium status: `cilium status`

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
