## Assign multiple tenants

A single team can be responsible for mulitple lines of business. I.e. Alice is responsible for both the Oil and Gas lines of business, part of Acme Corp.

So, Alice needs access to two different and isolated tenants.
As Capsule tenants are a further flat levels after namespaces, there is no support for hierarchy of tenants.
However, Bill can assign ownership of multiple tenants to Alice.

```
---
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
---
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: gas
spec:
  owners:
  - name: alice
    kind: User
```

Also, Bill can instead assign the ownership to a group:

```
---
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: oil-and-gas
    kind: Group
---
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: gas
spec:
  owners:
  - name: oil-and-gas
    kind: Group
```

The resources (e.g. Resource Quota, Nodes Pool, Storage Class, Ingress Class) and policies (e.g. Network Policy, PodSecurityPolicy, Trusted Registry) remain isolated for each tenant.

