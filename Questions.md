# WIP - 2x lectures
# Introduction

# Content and reflection

## k8s storage


### Reading material

- Kubernetes for Developers:
  - Chapter 9.1 Volumes, Persistent Volumes, Claims and Storage Classes
  - Chapter 9.3 Migrating/Recovering Disks (Read Chapter 9.2 prior - see below)
- Static vs. Dynamic Storage Provisioning: A Look Under the Hood
  - [Static vs. Dynamic Storage Provisioning: A Look Under the Hood](https://bluexp.netapp.com/blog/cvo-blg-static-vs.-dynamic-storage-provisioning-a-look-under-the-hood)

### Questions

- Describe the concept of a _Volume_?
  - A volume in Kubernetes is an abstraction of a storage medium that pods can access. It provides a way to persist data beyond the lifetime of a pod and allows sharing data between containers within the same pod.

- Which different types do you recon there are?
  - Types of volumes in Kubernetes include **emptyDir**, **hostPath**, **persistentVolumeClaim**, and more. Each type has its characteristics and use cases, such as **emptyDir** for ephemeral storage and **persistentVolumeClaim** for dynamic provisioning of storage.

- What is a _Storage Class_ and how does this differ from a _Volume_?
  - A StorageClass in Kubernetes is an object that defines the type and properties of the storage that can be dynamically provisioned. It differs from a Volume in that a Volume is an instance of storage attached to a pod, while a StorageClass defines how volumes are provisioned.

- How are _Persistent Volumes_ and _Persistent Volume Claims_ related?
  - Persistent Volumes are storage resources provisioned in a Kubernetes cluster, while Persistent Volume Claims are requests for storage by pods. They are related in that a Persistent Volume Claim binds to a Persistent Volume, allowing the pod to use the requested storage.

- _Bound_ - what does this mean?
  - In Kubernetes, when a Persistent Volume Claim is bound, it means that the claim has been successfully matched to a Persistent Volume and can be used by a pod.

- Which objects mentioned are actually _cluster-wide_?
  - Persistent Volumes and StorageClasses are cluster-wide objects in Kubernetes. They can be accessed and used by any pod within the cluster.

- What is the difference between _Static_ and _Dynamic_ provisioning?
  - Static provisioning involves manually provisioning storage in advance and associating it with a Persistent Volume, while dynamic provisioning automatically creates Persistent Volumes as needed based on the Persistent Volume Claim specifications.

- When is an application _stateful_ and when is it _stateless_?
  - An application is stateful if it maintains data or state between sessions, such as databases. It is stateless if it does not retain data between sessions, like web servers.

  - In which circumstances do you use each of these?
    - Stateful applications are used when data persistence is necessary, ensuring that data is retained even if the pod is restarted or rescheduled. Stateless applications are used when the application's behavior is solely determined by the input it receives, and there's no need to maintain state between sessions.

## k8s StateFulSets

### Reading material

- Kubernetes for Developers:
  - Chapter 9.2 StatefulSet
  
### Questions

- What does a _StatefulSet_ entail?
  - A StatefulSet in Kubernetes manages the deployment and scaling of stateful applications. It provides guarantees about the ordering and uniqueness of pods, ensuring stable network identities and persistent storage.

- How is it different from a _Deployment_?
  - Unlike a Deployment, which manages stateless applications and can scale pods independently, a StatefulSet maintains a unique identity for each pod it manages and provides guarantees about pod ordering and naming.

- When would you use a _StatefulSet_ over a _Deployment_?
  - StatefulSets are used for stateful applications that require stable, persistent storage and network identities. They are suitable for applications like databases, caches, or messaging queues where each instance needs to maintain its identity and state.

## k8s Environment variables, Secrets and ConfigMaps

### Reading material

- Kubernetes for Developers:
  - Chapter 9.1.1 Volumes (ConfigMap section - recap)
  - Chapter 11.4 Secrets

- Environment Variables online:
  - [Define Environment Variables in Kubernetes Containers](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)
- ConfigMaps online:
  - [Kubernetes ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- Secrets online:
  - [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

### Questions

- Basic example of what you did put in environment variables:
  - Environment variables can store configuration settings like database credentials, API keys, or connection strings. For example, you can set the environment variable `DATABASE_URL` to `mysql://username:password@hostname:port/database`.

- ConfigMaps are for configuration - Find examples of where:
  - Data from here is extracted and placed in environment variables:
    - ConfigMaps can be used to store configuration data like application settings, which can then be mounted as environment variables in pods. For instance, a ConfigMap named `app-config` containing key-value pairs can be mounted as environment variables in a pod.
  - Data is used to generate a configuration file on disk:
    - Similarly, ConfigMaps can be used to generate configuration files on disk. For example, a ConfigMap containing a YAML configuration file can be mounted as a volume in a pod, allowing the application to read the configuration file from the mounted path.

- What is a _Secret_ and how do you denote it?
  - A Secret in Kubernetes is an object used to store sensitive information like passwords, tokens, or keys. Secrets are denoted using the Secret resource type in Kubernetes. They are base64 encoded for storage and can be referenced by pods securely.

## k8s DaemonSets

### Reading material

- Kubernetes for Developers:
  - Chapter 12.2 Deploying Node Agents with DaemonSet (Read 12.3 for other uses of DaemonSet)

### Questions

Finally, we have _DaemonSets_:
- What is the concept behind?
  - DaemonSets ensure that a copy of a pod runs on each node in the cluster, ensuring that specific tasks or services are available on every node.

- What do you use them for?
  - DaemonSets are used for deploying system daemons or monitoring agents that need to run on every node. For example, logging agents, monitoring agents, or networking agents.

- Would you control where they are deployed?
  - Yes, DaemonSets can be controlled to deploy only on specific nodes using node selectors or affinity rules.

  - How can this be controlled if so desired?
    - DaemonSets can be controlled using node selectors to specify on which nodes they should run or using affinity rules to define preferences for certain nodes based on labels.
