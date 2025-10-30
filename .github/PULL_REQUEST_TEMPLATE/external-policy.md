---
name: External Network Policy Request
about: Request a new external network connection (to internet or other namespaces)
title: '[EXTERNAL] '
labels: 'policy:external, needs-security-review'
assignees: ''
---

## 🌐 External Network Policy Request

**⚠️ This PR requires security team approval before merging.**

---

### Destination Information

**Service/Domain:**
<!-- Example: api.stripe.com, api.github.com, external-namespace.svc.cluster.local -->

**IP Address/CIDR:** (if known)
<!-- Example: 8.8.8.8/32, 10.0.0.0/16 -->

**Port:**
<!-- Example: 443, 8080 -->

**Protocol:**
<!-- Example: TCP, UDP, HTTP, HTTPS -->

---

### Justification

**Why is this connection needed?**
<!-- Explain the business/technical reason for this external connection -->

**What service/feature depends on this?**
<!-- Example: Payment processing, OAuth authentication, External API integration -->

**What happens if this connection is blocked?**
<!-- Describe the impact -->

---

### Security Considerations

**Data Sensitivity:**
<!-- What kind of data will be transmitted? (e.g., PII, financial, public) -->

**Authentication Method:**
<!-- How will your app authenticate to the external service? (e.g., API key, OAuth, mTLS) -->

**Encryption:**
<!-- Is the connection encrypted? (e.g., TLS 1.2+, HTTPS only) -->

**Known Risks:**
<!-- Any security concerns or risks you're aware of? -->

---

### Testing Plan

**How will you verify this works?**
<!-- Describe your testing approach -->

**Rollback Plan:**
<!-- What if something goes wrong? -->

---

### Additional Context

<!-- Add any other context, screenshots, or documentation links -->

---

## Checklist

- [ ] I have provided all required information above
- [ ] I have considered security implications
- [ ] I have a testing plan
- [ ] I understand this requires security team approval
- [ ] I have updated relevant documentation

---

/cc @ciscocpa/security-team - Please review this external policy request
