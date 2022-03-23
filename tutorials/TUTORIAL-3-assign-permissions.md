## Assign permissions

Alice as the tenant owner and admin, is responsible for managing other tenant users.
She is responsibile to create additional roles and assigning these roles to the other tenant users.

Alice does not need to interact with Bill, to complete her day-by-day duties, and Bill do not have to deal with multiple requests coming from multiple tenant owners.
Which is: **the tenant owner is autonomous with respect to the cluster administrator**.

### Identities

Bill has to create the other tenant users identities in the IAM system: this is the sole required responsibility when onboarding new tenant users. It can be done once during the tenant onboarding.

### Roles

Alice can create custom RBAC roles or use the builtin cluster roles:

```
kubectl auth can-i get roles -n oil-development
yes

kubectl auth can-i get rolebindings -n oil-development
yes
```

So, she can assign the admin role on the *oil-development* to Joe, another *oil* tenant user:

```
kubectl --as alice --as-group capsule.clastix.io apply -f - << EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
  name: oil-development:admin
  namespace: oil-development
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: joe
EOF
```

and Joe can operate on his namespace, but not on the other ones of same tenant:

```
kubectl --as joe --as-group capsule.clastix.io auth can-i create pod -n oil-development
yes

kubectl --as joe --as-group capsule.clastix.io auth can-i create pod -n oil-production
no
```
