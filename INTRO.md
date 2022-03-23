# Multi-tenancy current status in Kubernetes: the cluster sprawl

Kubernetes introduces the Namespace object type to create logical partitions of the cluster as isolated slices. However, implementing advanced multi-tenancy scenarios, it soon becomes complicated because of the flat structure of Kubernetes namespaces and the impossibility to share resources among namespaces belonging to the same tenant. To overcome this, cluster admins tend to provision a dedicated cluster for each groups of users, teams, or departments. As an organization grows, the number of clusters to manage and keep aligned becomes an operational nightmare, described as the well know phenomena of the clusters sprawl.

# Multi-tenancy made easy: Capsule

## Tenant-side: Capsule Controller

Capsule takes a different approach. In a single cluster, the Capsule Controller aggregates multiple namespaces in a lightweight abstraction called Tenant, basically a grouping of Kubernetes Namespaces. Within each tenant, users are free to create their namespaces and share all the assigned resources.

## Cluster-side: Capsule Policy Engine

On the other side, the Capsule Policy Engine keeps the different tenants isolated from each other. Network and Security Policies, Resource Quota, Limit Ranges, RBAC, and other policies defined at the tenant level are automatically inherited by all the namespaces in the tenant. Then users are free to operate their tenants in autonomy, without the intervention of the cluster administrator.

### Network policies

Filter ingress/egress traffic between pods in different namespaces, from a namespace to external, from external to a namespace.

### Pod Security policies (deprecated)

Security policies on containers.
A Pod Security Policy is a cluster-level resource that controls security sensitive aspects of the pod specification. The PodSecurityPolicy objects define a set of conditions that a pod must run with in order to be accepted into the system, as well as defaults for the related fields. They allow an administrator to control the following:
- Running of privileged containersprivileged
- Usage of host namespaceshostPID, hostIPC
- Usage of host networking and portshostNetwork, hostPorts
- Usage of volume typesvolumes
- Usage of the host filesystemallowedHostPaths
- Allow specific FlexVolume driversallowedFlexVolumes
- Allocating an FSGroup that owns the pod's volumesfsGroup
- Requiring the use of a read only root file systemreadOnlyRootFilesystem
- The user and group IDs of the containerrunAsUser, runAsGroup, supplementalGroups
- Restricting escalation to root privilegesallowPrivilegeEscalation, defaultAllowPrivilegeEscalation
- Linux capabilitiesdefaultAddCapabilities, requiredDropCapabilities, allowedCapabilities
- The SELinux context of the containerseLinux
- The Allowed Proc Mount types for the containerallowedProcMountTypes
- The AppArmor profile used by containersannotations
- The seccomp profile used by containersannotations
- The sysctl profile used by containersforbiddenSysctls,allowedUnsafeSysctls

To make policies take effect:
- Enable the admission controller
- Authorize policies: the requesting user or target pod's service account must be authorized to use the policy, by allowing the use verb on the policy. Either to resource-specific controller (e.g. deployment) or to the pod's service account.

### Resource Quotas

Quota of resources per namespace.
Resource quotas work like this:

- Different teams work in different namespaces. Currently this is voluntary, but support for making this mandatory via ACLs is planned.
- The administrator creates one ResourceQuota for each namespace.
- Users create resources (pods, services, etc.) in the namespace, and the quota system tracks usage to ensure it does not exceed hard resource limits defined in a ResourceQuota.
- If creating or updating a resource violates a quota constraint, the request will fail with HTTP status code 403 FORBIDDEN with a message explaining the constraint that would have been violated.
- If quota is enabled in a namespace for compute resources like cpu and memory, users must specify requests or limits for those values; otherwise, the quota system may reject pod creation. Hint: Use the LimitRanger admission controller to force defaults for pods that make no compute resource requirements.

Examples of policies that could be created using namespaces and quotas are:

- In a cluster with a capacity of 32 GiB RAM, and 16 cores, let team A use 20 GiB and 10 cores, let B use 10GiB and 4 cores, and hold 2GiB and 2 cores in reserve for future allocation.
- Limit the "testing" namespace to using 1 core and 1GiB RAM. Let the "production" namespace use any amount.

In the case where the total capacity of the cluster is less than the sum of the quotas of the namespaces, there may be contention for resources. This is handled on a first-come-first-served basis.
Neither contention nor changes to quota will affect already created resources.

### Limit ranges

Quota of resources per pod, per namespace.
A LimitRange provides constraints that can:

- Enforce minimum and maximum compute resources usage per Pod or Container in a namespace.
- Enforce minimum and maximum storage request per PersistentVolumeClaim in a namespace.
- Enforce a ratio between request and limit for a resource in a namespace.
- Set default request/limit for compute resources in a namespace and automatically inject them to Containers at runtime.


### RBAC

Role-based access control, per identities/group of identities, per namespace.
