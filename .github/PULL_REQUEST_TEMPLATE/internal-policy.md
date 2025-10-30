---
name: Internal Network Policy
about: Add or modify policies for communication within the namespace
title: '[INTERNAL] '
labels: 'policy:internal'
assignees: ''
---

## 🏠 Internal Network Policy Change

**✅ This PR can be auto-approved** (no security team review needed)

---

### What's Changing?

<!-- Describe the policy changes -->

---

### Services Affected

**Source Service:**
<!-- Which service is making the connection? -->

**Destination Service:**
<!-- Which service is receiving the connection? -->

**Port & Protocol:**
<!-- Example: port 8080/TCP, port 5432/TCP (PostgreSQL) -->

---

### Reason for Change

<!-- Why are you making this change? -->
- [ ] New feature requiring service-to-service communication
- [ ] Bug fix for connectivity issue
- [ ] Performance optimization
- [ ] Refactoring/cleanup
- [ ] Other: ___________

---

### Testing

**How did you test this?**
<!-- Describe your testing approach -->

---

### Additional Notes

<!-- Any other context or information -->

---

## Checklist

- [ ] I have tested this change
- [ ] Only affects communication within my namespace
- [ ] Does not connect to external services
- [ ] Documentation updated (if needed)

---

**Note:** This PR will be auto-approved since it only affects internal namespace policies. You can merge once all CI checks pass! 🚀
