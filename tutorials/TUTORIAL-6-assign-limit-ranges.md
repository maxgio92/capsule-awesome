## Assign limit ranges

Bill can also set `LimitRanges` for each namespace in Alice's tenant by defining limits for pods and containers in the tenant spec:

```yaml
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
...
  limitRanges:
    items:
    - type: Pod
      min:
        cpu: "50m"
        memory: "5Mi"
      max:
        cpu: "1"
        memory: "1Gi"
    - type: Container
      defaultRequest:
        cpu: "100m"
        memory: "10Mi"
      default:
        cpu: "200m"
        memory: "100Mi"
      min:
        cpu: "50m"
        memory: "5Mi"
      max:
        cpu: "1"
        memory: "1Gi"
    - type: PersistentVolumeClaim
      min:
        storage: "1Gi"
      max:
        storage: "10Gi"
```

Limits will be inherited by all namespaces created by Alice. In out case when Alice creates the namespace *oil-production*, Capsule creates the following:

> ```yaml
> kind: LimitRange
> apiVersion: v1
> metadata:
>   name: limits
>   namespace: oil-production
>   labels:
>     tenant: oil
> spec:
>   limits:
>   - type: Pod
>     min:
>       cpu: "50m"
>       memory: "5Mi"
>     max:
>       cpu: "1"
>       memory: "1Gi"
>   - type: Container
>     defaultRequest:
>       cpu: "100m"
>       memory: "10Mi"
>     default:
>       cpu: "200m"
>       memory: "100Mi"
>     min:
>       cpu: "50m"
>       memory: "5Mi"
>     max:
>       cpu: "1"
>       memory: "1Gi"
>   - type: PersistentVolumeClaim
>     min:
>       storage: "1Gi"
>     max:
>       storage: "10Gi"
> ```

> Note: being the limit range specific of single resources, there is no aggregate to count.

Also, Alice does not have permissions to modify `ResourceQuota`s and `LimitRange`s:

```
kubectl -n oil-production auth can-i patch resourcequota
no
kubectl -n oil-production auth can-i delete resourcequota
no
kubectl -n oil-production auth can-i patch limitranges
no
kubectl -n oil-production auth can-i delete limitranges
no
```

