# Notes

## Useful commands

`k explain Namespace`   
`k explain pod.spec.containers`

`k logs <pod_name>`  
`k logs deployment/<deploymenvt_name>`

`k exec -it <pod_name> -c <container_name> -- bash`

`k port-forward -n <namespace_name> <pod_name> <local_port>:<pod_port>`  
`k port-forward svc/<deployment_name> <local_port>:<pod_port>`

## Cluster

A cluster is a collection of nodes on which everything runs.
Nodes are the physical and virtual machines that run the Pods.

The advantage of a cluster is that if one node fails, the others can continue running.

- `Control Plane` node: manages the cluster;
- `Worker` node: runs the Pods.

### Commands

`k config get-clusters`
`k config use-context <cluster_name>`

## Namespace

Namespaces help organize resources within the cluster.

### Commands

`k apply -f <namespace_yaml>`  
`kubens <namespace_name>`

`k get namespaces`  
`k get all -n <namespace_name>`

## Pod

A Pod is the basic unit in which containers run.

A Pod can contain multiple containers, though typically these are support containers like Init containers (which run before others) or Sidecar containers (which support the main container).

### Commands

`k get pods`

## ReplicaSet

A ReplicaSet is a wrapper around Pods that allows you to define how many replicas of a Pod should be created.

It automatically manages Pod creation.

### Commands

`k get rs`

## Deployment

A Deployment is a wrapper around a ReplicaSet that allows you to modify resources, perform updates, and manage rollbacks and rollouts.

It automatically manages ReplicaSets.

### Commands

`k get deployments.apps`  

## Rollout e Rollback

Using `k rollout restart deployment <deployment_name>`, a new ReplicaSet is created, and the old one is retained with 0 Pods in case you need to perform a rollback using `k rollout undo deployment <deployment_name>`.

You can also use `k apply -f <file_name>` to update the Pods in the Deployment after making a change.

In both cases, the old Pods are stopped only when the new ones are healthy.

## Service

A Service manages networking for Deployments and acts as a load balancer.

- `ClusterIP`: Internal network only within the cluster;
    - `ClusterIP: None` configures it without an IP, disabling load balancing and allowing clients to directly access individual Pods via DNS inside the cluster.
- `NodePort`: Internal and external network;
    - Don’t forget to configure firewall rules for the NodePort.
- `LoadBalancer`: Uses the cloud provider’s load balancer to route traffic to the cluster.
    - `cloud-provider-kind` allows you to test it locally without a real cloud provider.
          
### Commands

`k get svc`

## CoreDNS

CoreDNS handles routing between different Services based on the Namespace and the Service name within the cluster.

`curl <service_name>:80` works by default if you're in the same Namespace.

`curl <service_name>.<namespace_name>.svc.cluster.local` works even outside your own Namespace thanks to CoreDNS.

### Commands

`k get pods -n kube-system`

## Job

Jobs are used for tasks that need to be executed once or until they are completed.

### Commands

`k get jobs`

## CronJob

CronJobs allow you to schedule Jobs.

Check `https://crontab.guru/` for the correct syntax of the `schedule` field.

### Commands

`k create job --from=cronjob/<cronjob_name> <set_name>`

## DaemonSet

A DaemonSet runs a copy of the Pod on every node (or a specified subset), except for the Control Plane nodes.

This is useful for tasks like collecting logs from all nodes.

## StatefulSet

A StatefulSet is similar to a Deployment, but each Pod has its own unique persistent volume.

It allows modifications to a few fields, but not the volume size.

A StatefulSet requires a headless service.

## ConfigMaps

ConfigMaps allow you to separate configuration data from container images.

- `property-like-keys`: loads environment variables into the Pod;
- `file-like-keys`: loads a configuration file that the Pod expects.

### Commands

`k get cm`

`k exec <pod_name> -c <container_name> -- cat <path/to/config>`  
`k exec <pod_name> -c <container_name> -- printenv`

## Secrets

Secrets are similar to ConfigMaps but allow you to encode data in base64 and manage permissions.

There are different types of Secrets (see [docs](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)).

### Commands

`k get secrets <secret_name> -o yaml | yq '.data.<field_name>'`

## Ingress

Ingress allows routing traffic to services by redirecting LoadBalancer traffic to an Ingress Controller based on configured rules.

By default, it only supports layer 7 (http/https), but with some implementations, you can use layer 4 (tcp/udp) as well.

### How

1. Install your own Ingress Controller;  
`helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace`


2. Get Ingress Controller external IP to add it to `/etc/hosts`;  
`k get svc -n ingress-nginx`

3. Apply all yaml files.


## GatewayAPI

The GatewayAPI is similar to Ingress but adds support for TCP/UDP and allows for more complex routing scenarios.

### Commands

`k get gateway`

## PersistentVolume and PersistentVolumeClaim

These resources allow you to create storage that persists across Pod restarts.

A PersistentVolumeClaim is used to connect Pods to PersistentVolumes, which are created by Kubernetes.

Replica Pods in a Deployment share the same PersistentVolume, while each StatefulSet Pod has its own.

For shared persistent storage between the host and the container, create a PersistentVolume, link it to a PersistentVolumeClaim, and specify the `persistentVolumeClaim` field in the Deployment. Don’t forget to update the cluster configuration paths.

For dynamic shared storage across Deployments, create a PersistentVolume and reference it in the `persistentVolumeClaim` field of the Deployment.

For dynamic storage unique to each StatefulSet, use the `volumeClaimTemplates` field in the StatefulSet.

- `ReadWriteOnce`: Only one Pod can write to it at a time, though others can read. Can only be accessed by one node at a time;
- `ReadWriteMany`: Does not handle concurrency, leaving it to the file system (like NFS). Can be used by multiple nodes;
- `ReadOnlyMany`: Read-only access.

- `ReclaimPolicy: Delete`: The storage is deleted when the PersistentVolumeClaim is deleted;
- `ReclaimPolicy: Retain`: The storage remains even after the PersistentVolumeClaim is deleted.

### Commands

`k get pv`  
`k get pvc`

`k get storageclasses.storage.k8s.io`

## RBAC (ServiceAccount, Role, RoleBinding)

RBAC allows you to define Pod permissions if you need to interact with Kubernetes APIs.

You create a ServiceAccount and a Role, then link them using a RoleBinding. In the Pod, add the `serviceAccountName` field.

A Role grants permissions within a specific Namespace.
A ClusterRole (with ClusterRoleBinding) grants permissions across the entire cluster.

## Some resources
- https://www.chainguard.dev/
    - Security-focused containers;
- https://github.com/BretFisher/podspec
    - Good default settings for Pods;

## TODO

LimitRange, NetworkPolicy, MutatingWebhookConfiguration, ValidatingWebhookConfiguration, HorizontalPodAutoscaler, CustomResourceDefinition


