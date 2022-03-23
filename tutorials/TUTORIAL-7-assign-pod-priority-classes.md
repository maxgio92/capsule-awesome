## Assign Pod Priority Classes

Pods can have priority. Priority indicates the importance of a Pod relative to other Pods. If a Pod cannot be scheduled, the scheduler tries to preempt (evict) lower priority Pods to make scheduling of the pending Pod possible.

In a multi-tenant cluster, not all users can be trusted, as a tenant owner could create Pods at the highest possible priorities, causing other Pods to be evicted/not get scheduled.

To prevent misuses of Pod Priority Class, Bill can enforce the allowed Pod Priority Class at tenant level:

```yaml
apiVersion: capsule.clastix.io/v1beta1
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - name: alice
    kind: User
  priorityClasses:
    allowed:
    - default
    allowedRegex: "^tier-.*$"
```

With the said Tenant specification, Alice can create a Pod resource if spec.priorityClassName equals to:

- default
- tier-gold, tier-silver, or tier-bronze, since these compile the allowed regex.

If a Pod is going to use a non-allowed Priority Class, it will be rejected by the Validation Webhook enforcing it.

