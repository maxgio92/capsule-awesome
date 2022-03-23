## Assign resources quota

Bill can set and enforce resources quota and limits for Alice's tenant:

```yaml
piVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
  namespaceOptions:
    quota: 3
  resourceQuotas:
    scope: Tenant
    items:
    - hard:
        limits.cpu: "8"
        limits.memory: 16Gi
        requests.cpu: "8"
        requests.memory: 16Gi
    - hard:
        pods: "10"
  limitRanges:
    items:
    - limits:
      - default:
          cpu: 500m
          memory: 512Mi
        defaultRequest:
          cpu: 100m
          memory: 10Mi
        type: Container
```

> The resource quotas above will be inherited by all the namespaces created by Alice.
> In our case, when Alice creates the namespace `oil-production`, Capsule creates the following resource quotas:
> 
> ```yaml
> kind: ResourceQuota
> apiVersion: v1
> metadata:
>   name: capsule-oil-0
>   namespace: oil-production
>   labels:
>     tenant: oil
> spec:
>   hard:
>     limits.cpu: "8"
>     limits.memory: 16Gi
>     requests.cpu: "8"
>     requests.memory: 16Gi
> ---
> kind: ResourceQuota
> apiVersion: v1
> metadata:
>   name: capsule-oil-1
>   namespace: oil-production
>   labels:
>     tenant: oil
> spec:
>   hard:
>     pods : "10"
> ```

Alice can create any resource according to the assigned quotas:

```
kubectl -n oil-production create deployment nginx --image nginx:latest --replicas 4
```

and inspect the current used resources in the *oil-production* namespace:

```
kubectl -n oil-production get resourcequota capsule-oil-1 -o yaml
...
status:
  hard:
    pods: "10"
    services: "50"
  used:
    pods: "4"
```

### Enforcemenet at tenant level

The enforcement scope can be at:
- Tenant (default)
- Namespace
levels.

By enforcing at tenant level, Capsule aggregates resources usage for all the namespaces in the tenant and adjusts all the `ResourceQuota` usage as aggregate.

In such case, Alice can check the used resources at tenant level, by inspecting the ResourceQuota object's annotations of any tenant namespace:

```
kubectl -n oil-production get resourcequotas capsule-oil-1 -o yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  annotations:
    quota.capsule.clastix.io/used-pods: "4"
    quota.capsule.clastix.io/hard-pods: "10"
```

or:

```
kubectl -n oil-development get resourcequotas capsule-oil-1 -o yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  annotations:
    quota.capsule.clastix.io/used-pods: "4"
    quota.capsule.clastix.io/hard-pods: "10"
...
```

When the aggregated usage for all namespaces reaches the hard quota, the native `ResourceQuota` Kubernetes Admission controller denies Alice's requests to create resources exceeding the quota:

```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-55649fd747-6fzcx   1/1     Running   0          12s
nginx-55649fd747-7q6x6   1/1     Running   0          12s
nginx-55649fd747-86wr5   1/1     Running   0          12s
nginx-55649fd747-h6kbs   1/1     Running   0          12s
nginx-55649fd747-mlhlq   1/1     Running   0          12s
nginx-55649fd747-t48s5   1/1     Running   0          7s
```

and:

```
kubectl -n oil-production get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-55649fd747-52fsq   1/1     Running   0          22m
nginx-55649fd747-9q8n5   1/1     Running   0          22m
nginx-55649fd747-r8vzr   1/1     Running   0          22m
nginx-55649fd747-tkv7m   1/1     Running   0          22m
```

### Enforcement at namespace level

By setting enforcement at the namespace level, i.e. spec.resourceQuotas.scope=Namespace, Capsule does not aggregate the resources usage and all enforcement is done at the namespace level.

For example:

```yaml
piVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
  namespaceOptions:
    quota: 3
  resourceQuotas:
    scope: Namespace
    items:
    - hard:
        limits.cpu: "2"
        limits.memory: 4Gi
        requests.cpu: "2"
        requests.memory: 4Gi
    - hard:
        pods: "3"
  limitRanges:
    items:
    - limits:
      - default:
          cpu: 500m
          memory: 512Mi
        defaultRequest:
          cpu: 100m
          memory: 10Mi
        type: Container
```

