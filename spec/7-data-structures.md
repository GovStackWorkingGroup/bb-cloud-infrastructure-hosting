# 7 Data Structures

{% hint style="success" %}
This section provides information on the core data structures/data models that are used by a Building Block. These data structures describe information that is exchanged between building blocks - they do not dictate internal data structures for a particular implementation. These data structures should also describe the _minimum_ set of information that should be passed in an API call. The data structures can be extended for particular use cases.

For each data model, the following information should be provided:

* Name
* Description
* Fields - the various fields in this data structure. Each field definition should contain the following:
  * Name
  * Type (string, Boolean, number, date, etc)
  * Description
  * Foreign Key (does this field reference another data structure)
  * Constraints (does the field need to be unique, is it the primary key)
  * Required (Y/N)
  * You can also reference any standards that must be adhered to (ie. UTC standard for date/times)

**Resource Model:** This section may also include a resource model diagram which shows the relationship between data objects that are used by this Building Block.
{% endhint %}

This section provides information on the core data structures/data models that are used by this Building Block.

A precise description of all data structures of a full cloud- and container platform would fill hundreds of pages and also imply specific technologies to implement the cloud- and container orchestration layers. Industry attempts to define technology neutral meta APIs and data structures have thus far had limited traction; the most successful one is probably [TOSCA](https://input.scs.community/). Infra-as-Code tools (such as opentofu or ansible) also have their own representation of the resources and their properties – however, they do not abstract away the differences in the object model of different cloud- and container orchestration systems.

Prescribing a meta-API of our own would happen in competition with existing efforts and might suffer from low adoption with poor tooling coverage. This may be a worthwhile initiative on its own as soon as the demand from stakeholders and their willingness to adopt the results does exist.

We shall describe the high-level characteristics of the data structures (this chapter) and APIs (next chapter) to give guidance for the requirements and design of good data structures and APIs. It will be the role of the chosen technology for implementation to define the detailed data structures and APIs. The description shall not be reproduced in a govstack document – instead the upstream’s technology description shall be referenced and only additional requirements, enhancements or restrictions be noted here.

## 7.1. Standards for IaaS and Container Layers <a href="#id-71-standards-for-iaas-and-container-layers" id="id-71-standards-for-iaas-and-container-layers"></a>

Guidelines and protocols for implementing infrastructure as a service (IaaS) and container orchestration layers to ensure interoperability and standardization.

The resources (virtual machines, networks, containers, …) are uniquely identified by UUIDs, assigned by the platform upon creation of the resources. Resources typically also have names that allow human operators to remember which resources serve which purpose. Platforms may or may not enforce the uniqueness of names – referencing resources by UUID is thus preferred.

The data structures are a hierarchical model of the (virtual/containerizedd) resources. They are typically encoded in JSON or YAML.

We shall reproduce one data structure (in JSON) describing the data structure needed to create a virtual machine here as an example:

```
{
  "server": {
    "name": "DemoVM",
    "imageRef": "",
    "flavorRef": "f7da0a83-d1b9-45a4-8b6a-0be086408094",
    "key_name": "SSHkey-Demo",
    "min_count": 1,
    "max_count": 1,
    "security_groups": [
      {
        "name": "c6328f85-1fb0-4c81-9277-5b03d8a04635"
      }
    ],
    "block_device_mapping_v2": [
      {
        "boot_index": 0,
        "uuid": "0b28a60d-6b2e-45d2-b366-df2dd544c63b",
        "source_type": "image",
        "volume_size": "10",
        "destination_type": "volume",
        "delete_on_termination": true
      }
    ],
    "networks": [
      {
        "uuid": "4658d46d-a9a6-4581-b095-779e933c9298"
      }
    ]
  }
}
```

The `block_device_mapping_v2` tells the cloud orchestrator to create a virtual disk (“volume”) with 10 GiB size that should be initialized with the contents of the image identified by uuid `0b28a60d-6b2e-45d2-b366-df2dd544c63b`. The other fields describe the name of the VM, its size (`flavorRef`), the injected ssh key, the number of VMs to be created, the network that it should attach to and the used security group and some more details on the volume, such as size and behavior upon VM deletion.

The data structures to describe the wanted state for a container workload contain the containers, their networking and storage properties etc. An example may look like this:

```
# Example from the book "Kubernetes up and running",
# from Burns, Beda, Hightower (O'Reilly, 2019)
# enhanced with the persistent volume claim.
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-vol-claim
  annotations:
    volume.kubernetes.io/storage-class: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  volumes:
    - name: my-vol
      persistentVolumeClaim:
        claimName: my-vol-claim
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
      volumeMounts:
        - name: my-vol
          mountPath: /data
      resources:
        requests:
          cpu: "50m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
      livenessProbe:
        httpGet:
          path: /healthy
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 30
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
```

This example creates a pod with a single container (to be downloaded from the location specified in `image` setting), with certain CPU and memory allocations and limits, a persistent volume with `ReadWriteOnce` properties and `10GiB` size, mounted to the `/data` path and settings for the orchestrator to probe the container for readiness and healthiness.

Submitting this declaration to the container layer will make it create the resources (a container and a persistent volume) and continuously watch the container’s status.

## 7.2. Standards for IAM and Operational Tooling <a href="#id-72-standards-for-iam-and-operational-tooling" id="id-72-standards-for-iam-and-operational-tooling"></a>

Specifications for identity and access management and operational tools to maintain security and efficiency.

User federation should be supported using OpenID Connect. OpenID Connect build on top of the provien oauth2 mechanisms.\
The [OpenID Connect specifications](https://openid.net/wg/connect/specifications/) are available from the OpenID Connect foundation’s working group.

## 7.3. Resource Allocation and Usage Records <a href="#id-73-resource-allocation-and-usage-records" id="id-73-resource-allocation-and-usage-records"></a>

Data models for tracking the allocation and utilization of cloud resources, such as compute, storage, and network capacities.

Resources are created, changed and deleted upon user requests, within the limits that the authorization (role) and quota for the user allow.

An efficient platform for recording usage does record the creation, change and deletion events. It however needs to monitor for the success of these requests. In addition there may be non-user triggered events that change the life-cycle of resources – these need to be recorded as well.\
The theoretical state of ressources can be constructed from these events. It is required to poll the real state regularly to ensure no event was missed or no error condition caused a state change that would lead to wrong assumptions, wrong usage records and thus wrong bills.

A minimalistic example of a usage record of this kind would be

```
{
    "ResourceID": VMUUID,
    "ProjectID": PRJUUID,
    "ResourceType": "compute",
    "ResourceSize": "SCS-2V-4",
    "StateChange": "created",
    "ExpectedState": "ACTIVE",
    "TimeStamp": "2024-07-02T04:16:51Z",
    "StateChangeReason": "UserRequest"
    "Consumption": []
}
```

This VM with `UUID` was created on Jul 2 in the early morning with the size `SCS-2V-4` based on a user request. Successful `UserRequest`s for `resized`, `stopped`, `started`, `deleted` would result in similar records. `OperatorActions` and `Reconciliation` would be other types of events. The regular polling could also create `StateChange` records of type `None` if the state is unchanged and as expected for audit reasons. The `Consumption` field may be used for resources whose usage is measured, such as e.g. external network traffic (common) or storage data transfer (uncommon).

Note that this is not meant to prescribe a way of creating Usage Records; this may happen in a later revision when the usage records will be exposed to users directly, which is not currently a requirement.

## 7.4. Configuration Settings for Virtual Machines and Containers <a href="#id-74-configuration-settings-for-virtual-machines-and-containers" id="id-74-configuration-settings-for-virtual-machines-and-containers"></a>

Templates and schemas for configuring virtual machines and containers, including hardware specifications and software dependencies.

It is best practice for users not to build highly specialized images for each task a VM might need to perform. This would results in dozens of images for typical workloads and in case a software update (e.g. a security update for the used operating system) is required, all of these would need to be rebuilt and reregistered. Instead, there is a common mechanism to customize images on (first) boot, using so called user-data and cloud-init or alternatives (such as cloudbase-init on Windows or coreos-cloudinit for CoreOS). The cloud-init supported commands are documented by the [cloud-init project](https://cloudinit.readthedocs.io/en/23.3.3)

## 7.5. Logs and Monitoring Data <a href="#id-75-logs-and-monitoring-data" id="id-75-logs-and-monitoring-data"></a>

Structured formats for storing logs and monitoring metrics to facilitate observability, troubleshooting, and performance tuning.

Not specified here, implementation dependant.

Providers should allow customer admins to access billing information and logs that allow to understand when a resource was changed by whom (OPTIONAL).

## 7.6. IAM Roles and Permissions <a href="#id-76-iam-roles-and-permissions" id="id-76-iam-roles-and-permissions"></a>

Data structures defining roles, permissions, and access controls for users and services within the cloud infrastructure.

The cloud infrastructure has a number of predefined roles; these roles apply to a scope. Predefined roles and scopes have a hierarchy; a more powerful roles encompasses all less powerful roles. The combination of a scope with a role is a persona.

The following roles exist, from most powerful to least powerful:

<table><thead><tr><th width="150">Role</th><th>Meaning</th></tr></thead><tbody><tr><td>admin</td><td>Cloud operator, omnipotent</td></tr><tr><td>manager</td><td>Customer role for self-management</td></tr><tr><td>member</td><td>Customer role to create and manage ressources</td></tr><tr><td>reader</td><td>Customer role with read-only access</td></tr></tbody></table>

The following scopes exist (from largest to smallest)

<table><thead><tr><th width="154">Scope</th><th>Meaning</th></tr></thead><tbody><tr><td>system</td><td>The complete cloud region</td></tr><tr><td>domain</td><td>Users, Roles, Project within the domain</td></tr><tr><td>project</td><td>All resources belonging to a particular project</td></tr></tbody></table>

The most commonly used personas are

<table><thead><tr><th width="157">Persona</th><th>Capabilities</th></tr></thead><tbody><tr><td>system-admin</td><td>Omnipotent</td></tr><tr><td>domain-manager</td><td>Manage users, roles, federation, projects</td></tr><tr><td>project-member</td><td>Manage all resources within the project</td></tr><tr><td>project-reader</td><td>Review resource usage within the project</td></tr></tbody></table>

On the container layer, we have the standards roles `cluster-admin` predefined who can manage all aspects of the container cluster. Less powerful roles can be created.
