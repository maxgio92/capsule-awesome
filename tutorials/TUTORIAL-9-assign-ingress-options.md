## Assign Ingress options

### Restrict Ingress Classes

Bill can assign a set of dedicated Ingress Classes to the *oil* tenant to force the applications in the *oil* tenant to be publishedf only by the assigned Ingress Controller:

```yaml
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
  ingressOptions:
    allowedClasses:
      allowed:
      - default
      allowedRegex: ^\w+-lb$
```

Alice can create an `Ingress` using only an allowed Ingress Class:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  namespace: oil-production
  annotations:
    kubernetes.io/ingress.class: default
spec:
  rules:
  - host: oil.acmecorp.com
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
        path: /
```

Any attempt of Alice to use a non-valid Ingress Class, or missing it, is denied by the Validation Webhook enforcing it.

### Restrict Ingress Hostnames

Bill can control ingress hostnames in the oil tenant to force the applications to be published only using the given hostname or set of hostnames:

```yaml
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
  ingressOptions:
    allowedHostnames:
      allowed:
        - oil.acmecorp.com
      allowedRegex: ^.*acmecorp.com$
```

The Capsule controller assures that all Ingresses created in the tenant can use only one of the valid hostnames.
Alice can create an `Ingress` using any allowed hostname:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  namespace: oil-production
  annotations:
    kubernetes.io/ingress.class: oil
spec:
  rules:
  - host: web.oil.acmecorp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

Any attempt of Alice to use a non-valid hostname is denied by the Validation Webhook enforcing it.

### Control Hostname collision

In a multi-tenant environment, as more and more ingresses are defined, there is a chance of collision on the hostname leading to unpredictable behavior of the Ingress Controller. Bill, the cluster admin, can enforce hostname collision detection at different scope levels:
1. Cluster
1. Tenant
1. Namespace
1. Disabled (default)

```
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
  - name: joe
    kind: User
  ingressOptions:
    hostnameCollisionScope: Tenant
```

When a collision is detected at scope defined by `spec.ingressOptions.hostnameCollisionScope`, the creation of the Ingress resource will be rejected by the Validation Webhook enforcing it.
When `hostnameCollisionScope=Disabled`, no collision detection is made at all.

