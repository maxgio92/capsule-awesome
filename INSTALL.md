# Getting started

## Requirements

- Kubernetes 1.16+ cluster
- Admission controllers enabled:
  - PodNodeSelector
  - LimitRanger
  - ResourceQuota
  - MutatingAdmissionWebhook
  - ValidatingAdmissionWebhook
- cluster-admin permissions on the Kubernetes cluster

## Quickstart

```
helm repo add clastix https://clastix.github.io/charts
helm install capsule clastix/capsule -n capsule-system --create-namespace
```

### Created resources

- Namespace
- Capsule service account
- RBAC cluster roles
- Controller
- CA and Certificate
- `Tenant` and `CapsuleConfiguration` CRDs
- Validating and mutating webhook configurations
- Metrics service
