## Assign Tenant ownership

### User as tenant owner

Bill onboards Alice tenant. 

#### Client certificate

Since Alice is a tenant owner, Bill needs to assign alice the Capsule group defined by `--capsule-user-group` option, which defaults to `capsule.clastix.io`.
He creates a client X.509 certificate with `"/CN=alice/O=capsule.clastix.io"`.

#### Tenant resource

Bill also needs to create a Tenant resource, for which the manifest:

```yaml
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
```

Namespaces are not already assigned to Oil Alice's tenant.
Alice with her credentials can create and delete namespaces:
```
kubectl auth can-i create namespaces
yes

kubectl auth can-i delete ns -n oil-production
yes
```

However, cluster resources are not accessible to Alice:
```
kubectl auth can-i get namespaces
no

kubecl auth can-i get nodes
no

kubecl auth can-i get tenants
no
```

### Group as tenant owner

The same is applicable to a group of users, like Oil users, composed by Bill and Alice.
Bill can do it by creating a group and declaring the oil tenant as so:

```yaml
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: oil-users
    kind: Group
```

### Robot account as tenant owner

An application is likely to be a Tenant Owner.
Bill can create a service account in the default namespace, named *robot*.
Since service accounts by default are members of the group `system:serviceaccounts:<namespace>`, Bill has to configure Capsule to consider also these groups as tenant groups, by declaring this `CapsuleConfiguration` resource:

```yaml
apiVersion: capsule.clastix.io/v1alpha1
kind: CapsuleConfiguration
metadata:
  name: default
spec:
  userGroups:
  - system:serviceaccounts:default
```

And create a tenant defined as below:

```yaml
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: system:serviceaccount:default:robot
    kind: ServiceAccount
```

Finally, *robot* service account can manage his tenant:

```yaml
kubectl --as system:serviceaccount:default:robot --as-group capsule.clastix.io auth can-i create namespaces
yes
```
