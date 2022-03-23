## Create tenant's namespaces

Alice can create a new namespace in her tenant:

```
kubectl create ns oil-production
```

> Is a good convention to name namespaces with the name of the tenant as a prefix. Bill can enforce this convention with the `--force-tenant-prefix` flag.

When the enforcement of the naming convention with the `--force-tenant-prefix` option, is enabled, the namespaces are automatically assigned to the right tenant by Capsule because the operator does a lookup on the tenant names.

Instead, when disabled, Alice needs to specify the tenant name as a label `capsule.clastix.io/tenant=<tenant name>` in the namespace manifest:

```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: oil-production
  labels:
    capsule.clastix.io/tenant: gas
```

> Capsule controller which is listening for creation and deletion events,
> assigns to Alice the following roles:
> 
> ```yaml
> ---
> apiVersion: rbac.authorization.k8s.io/v1
> kind: RoleBinding
> metadata:
>   name: namespace:admin
>   namespace: oil-production
> subjects:
> - kind: User
>   name: alice
> roleRef:
>   kind: ClusterRole
>   name: admin
>   apiGroup: rbac.authorization.k8s.io
> ---
> apiVersion: rbac.authorization.k8s.io/v1
> kind: RoleBinding
> metadata:
>   name: namespace-deleter
>   namespace: oil-production
> subjects:
> - kind: User
>   name: alice
> roleRef:
>   kind: ClusterRole
>   name: capsule-namespace-deleter
>   apiGroup: rbac.authorization.k8s.io
> ```

so that Alice can create any resource in the namespace:

```
kubectl -n oil-development run nginx --image=docker.io/nginx 
```

### Namespace quota

Bill can control how many namespace Alice can create in her tenant with `spec.namespaceOptions.quota`.
So the *oil* tenant manifest changes to:

```
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
  namespaceOptions:
    quota: 3
```

While Alice creates namespaces, the Capsule controller updates the status of the tenant.
Bill can check the status:

```
kubectl describe tenant oil

...
status:
  Namespaces:
    oil-development
    oil-production
    oil-test
  size:  3 # current namespace count
...
```

Once the quota as been reached, Alice cannot create further namespaces:

```
kubectl create ns oil-training

Error from server (Cannot exceed Namespace quota: please, reach out to the system administrators):
admission webhook "namespace.capsule.clastix.io" denied the request.
```

Capsule does enforcement through its Dynamic Admission Webhook.
