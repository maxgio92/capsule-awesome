## Assign Nodes Pool

Bill can dedicate a pool of worker nodes to the *oil* tenant, to isolate the tenant applications from other noisy neighbors.

Those nodes are labeled by Bill as `pool=oil`:

```
kubectl get nodes --show-labels

NAME                      STATUS   ROLES             AGE   VERSION   LABELS
...
worker06.acme.com         Ready    worker            8d    v1.18.2   pool=oil
worker07.acme.com         Ready    worker            8d    v1.18.2   pool=oil
worker08.acme.com         Ready    worker            8d    v1.18.2   pool=oil
```

To assign them to *oil* tenant, he can configure the tenant as below:

```yaml
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
  nodeSelector:
    pool: oil
    kubernetes.io/os: linux
```

The Capsule controller makes sure that any namespace created in the tenant has the annotation `scheduler.alpha.kubernetes.io/node-selector: pool=oil`. 

This annotation tells the scheduler of Kubernetes to assign the node selector `pool=oil` to all the pods deployed in the tenant. The effect is that all the pods deployed by Alice are placed only on the designated pool of nodes.

### Multiple node selectors

Bill can assign multiple node pools to the *oil* tenant:

```yaml
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
  nodeSelector:
    pool: oil
    kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    hardware: gpu
```

Any attempt of Alice to change the selector on the pods will result in an error from the `PodNodeSelector` Admission Controller plugin.

Also, RBAC prevents Alice to change the annotation on the namespace:

```
kubectl auth can-i edit ns -n oil-production
no
```
