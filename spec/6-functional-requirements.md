# 6 Functional Requirements

{% hint style="success" %}
The functional requirements section lists the technical capabilities that this building block should have. These requirements should be sufficient to deliver all functionality that is listed in the Key Digital Functionalities section.

These functional requirements do not define specific APIs - they provide a list of information about functionality that must be implemented within the building block.

These requirements should be defined by subject-matter experts and don’t have to be highly technical in this section.

If there are multiple functional components of a building block, the functional requirements for each component may have its own section
{% endhint %}

This section lists the technical capabilities of this Building Block:

## 6.1 Management of physical hardware <a href="#id-61-management-of-physical-hardware" id="id-61-management-of-physical-hardware"></a>

* Asset management for the physical infrastructure and automation to deploy the basic infrastructure tooling for rolling out the virtualization management platform and operational tooling (REQUIRED).

## 6.2 Virtualization platform <a href="#id-62-virtualization-platform" id="id-62-virtualization-platform"></a>

* Virtualization management platform that supports Virtualization of compute, storage, and network resources on top of commodity server hardware (REQUIRED).&#x20;
* Compute virtualization offers the creation of virtual machines of various standardized sizes/properties (with respect to virtual CPUs and memory) (REQUIRED).&#x20;
  * The available sizes/properties may be limited to a list of predefined templates (flavors) or be chosen freely (OPTIONAL).
* Optionally, virtualized or pass-through GPU compute capacity is exposed to the VMs (OPTIONAL).
* There is a scheduling mechanism that finds the most suitable host that fulfills all the constraints from the user (REQUIRED).&#x20;
  * Situations where common VM flavors can no longer be scheduled due to capacity shortage should be avoided (RECOMMENDED).&#x20;
  * Users must have ways to prevent or force scheduling of VMs on the same host (REQUIRED)
  * Users may express softer preferences for the placement (RECOMMENDED).
* The virtual machines are booted using operating system images. Providers provide a set of regularly updated standard images (REQUIRED).&#x20;
  * The origin (or the custom build process) must be documented and metadata be available that allows users to see what patch status the image has and what updates and lifetime to expect (REQUIRED).
* The provided standard OS images must support the processing of injected user-data for customization using mechanisms like cloud-init or the established alternatives (REQUIRED), see chapter 7.4.
* The virtualization platform must allow users to upload custom images (REQUIRED). This may be subject to quota limits or billing to compensate for the associated storage cost.
* There must be a way for users to reliably inject per-VM instance data that is being consumed by the VM on (first) boot to customize the instance’s configuration and role (REQUIRED).
* Securely isolated private networks can be created (SDN = Software Defined Networking) and VMs attached to them (REQUIRED).&#x20;
  * It must be possible to connect networks via routers (REQUIRED).
* User-defined rule sets determine which communication is allowed between virtual networks and VMs (REQUIRED).
* The virtual networks can be connected to external networks, providing outbound and inbound connectivity. It must be possible to do this by attaching additional IP addresses from the external network to existing internal network ports (REQUIRED).&#x20;
  * In public clouds, it must be possible to connect to the internet this way (CONDITIONALLY REQUIRED) – in private clouds, the connectivity to the internet may be heavily restricted on non-existent. In any case, the user-defined rule sets for connections apply also for external networks connected this way.
* The network provides user-controlled load-balancers which redirect traffic to a set of backend services depending on their load and availability (REQUIRED).
* The storage subsystem exposes block storage, that can be attached as virtual hard disks to the VMs (REQUIRED).
* The storage subsystem supports user controlled snapshots and backups (REQUIRED).
* The storage also exposes a standardized object storage interface (REQUIRED).
* The storage offers several storage classes with different performance and encryption attributes (RECOMMENDED) whose properties are documented and discoverable.
  * Optionally supporting dedicated storage solutions (OPTIONAL).
* There is a key management function that can be used to securely handle secrets for storage and network encryption (RECOMMENDED).
* Optionally, the platform provides a Domain Name Service capability (RECOMMENDED).
* The provider must ensure security updates for the cloud management software are deployed, notifying users when this leads to workload or control plane disruption (REQUIRED).&#x20;
  * Providers should minimize disruption e.g. by using live migration of VMs for host maintenance (RECOMMENDED).
* All of the virtualized resources are controlled by REST APIs that allow on-demand self-service for authenticated platform users (REQUIRED).
* The virtualization platform offers a service catalog that allows to discover the available services, how to access them and what optional features the services support (REQUIRED).
* The REST APIs must be documented publicly (REQUIRED)&#x20;
* The REST APIs must be supported by common Infrastructure as Code Tools (REQUIRED).&#x20;
* Ideally they are covered by an OpenAPI 3.1+ specification (RECOMMENDED).
* The capabilities for users are limited by their assigned roles (REQUIRED) and by quotas that cap resource usage (REQUIRED).&#x20;
  * Role assignment via groups and user assignment to groups must be supported (REQUIRED).
  * There must be predefined roles for read-only access (REQUIRED). (See also chapter 7.6)
* API calls use tokens or client certs that have limited life time of maximum one day (REQUIRED), shorter is recommended.&#x20;
  * It must be possible to revoke tokens/certs (REQUIRED).
* Compute resources are split into several (ideally three or more) availability zones. These zones have a meaningful independence from each other on the hardware level in terms of power supply, backup power, cooling, fire protection, core routers, network connectivity, etc. in order to have a significantly lower chance of more than one availability zone failure compared to a single availability zone failure (RECOMMENDED).&#x20;
  * The zones need to be connected with high-bandwidth low-latency networks (REQUIRED).&#x20;
  * The details of connectivity and independence may vary but need to be made transparent (REQUIRED).&#x20;
  * This allows for users to create services that survive common hardware outage scenarios. The storage and especially the network resources are best designed to work across the availability zones and survive the failure of one (by using redundancy) (RECOMMENDED);&#x20;
  * _if_ they are not resilient to availability zone failure, they must be also be provided separately per availability zone (CONDITIONALLY REQUIRED).
* Providers may offer multiple regions (OPTIONAL).&#x20;
  * Regions may just support federation like with other compatible providers; they may offer additional synchronisation for user convenience (OPTIONAL).&#x20;
  * Regions must not depend on anything from another region to be fully functional (REQUIRED).
* The web interface should be configured to require two-factor authentication (RECOMMENDED).

## 6.3 Container orchestration platform <a href="#id-63-container-orchestration-platform" id="id-63-container-orchestration-platform"></a>

* There is a Self-service function to create Container Orchestration Clusters within customer projects on-demand (REQUIRED).
* The offered APIs are discoverable (REQUIRED).
* The size of these clusters (how many machines with what size) can be chosen by users (REQUIRED).
* Unless explicitly chosen differently by the user (e.g. by creating single-node clusters), the clusters are distributed over several physical machines to ensure resiliency against outage of single physical machines (REQUIRED).
* The clusters can be enlarged or reduced in size on the fly (REQUIRED) and without disruption (as long as users don’t request to go below a minimal size).
* The container orchestration software can be updated without disruption to the user workload nor the control place (REQUIRED). (This may be subject to minimal size requirements again, a single node cluster can not do rolling upgrades, obviously.)
* There must be a documented way to get a metrics service deployed that allows to observe the load on the system (REQUIRED).&#x20;
  * It is recommended that this is enabled by default, i.e. implemented as opt-out (RECOMMENDED).
* There must be clear responsibilities to provide (REQUIRED)&#x20;
  * Optionally, install security upgrades to the orchestration software by the provider (OPTIONAL).
* The container clusters must provide access to persistent storage (REQUIRED).
* The container clusters must provide ways to control securely isolated networking between containers (REQUIRED)&#x20;
* The container clusters must provide ways to expose services to outside users (REQUIRED).
* The container orchestration layer must allow expressing scheduling preferences to mandate or avoid avoid containers/pods being scheduled on the same node and observe these. (REQUIRED)
* Container cluster creators can create roles with limited capabilities inside their container clusters to implement a least-privilege approach (REQUIRED).
* The management of the container clusters is done via REST APIs (REQUIRED).
* The management of the workload containers in the cluster is done via REST APIs (REQUIRED).
* The APIs follow the reconciliation loop paradigm where the user submits the wanted infrastructure state in a hierachical data structure (typically JSON or YAML) and the container orchestrator then repeatedly attempts to ensure that the reality matches the wanted state (REQUIRED).
* The API is extensible, providing a mechanism by which schema for custom resources can be supplied and then used to validate customer resource creation requests (REQUIRED).&#x20;
  * The customer can provide the needed components (containers, hooks, etc.) to provide the same automation for custom resources that the container platform provides for its native resources (REQUIRED). Sidenote: The reconciliation loop paradigm is implemented by so-called operators.

## 6.4 Identity and Access Management <a href="#id-64-identity-and-access-management" id="id-64-identity-and-access-management"></a>

* The platform should offer federatable identity management that can be used by customers for controlling access to the Virtualization Platform and to the Container Platform.
* The platform should allow for customer-controlled user federation from external Identity Providers via industry standard protocols such as Open ID Connect (RECOMMENDED).&#x20;
  * The Identity Provider may be another compatible cloud (RECOMMENDED).
* The multi-tenancy supports two layers (RECOMMENDED).
  * Virtual resources belong to a project and only users that have roles with access to this project can see and manage them (REQUIRED).
  * By default, resources from different projects are isolated from each other (REQUIRED).
  * User and role management is possible in a self-service manner with isolated domains/realms (RECOMMENDED).
* The container management platform allows to federate users from external identity providers using OpenID connect (and possibly other established standards), also allowing to leverage the same identities that are used on the virtualization layer (if so wanted by the user). (REQUIRED)

## 6.5 Operational and Security Tooling <a href="#id-65-operational-and-security-tooling" id="id-65-operational-and-security-tooling"></a>

The platform comes with a standard set of operational tooling that eases the operation of the platform:

* Lifecycle management: Tooling to deploy, remove, change and upgrade the software components at all layers (REQUIRED).&#x20;
  * It is crucial that the rollout of an updated software component is a normal and regular activity with minimal customer impact, ensuring that security updates can be deployed short-term using regular processes (REQUIRED).
* Observability: Tooling to permanently collect internal information from the infrastructure. Ability to configure alerts for irregularities (errors) and thresholds (e.g. capacity limits) (REQUIRED).
* Health monitoring does complement this by running scenario tests from a user perspective (REQUIRED). This allows to observe the user-visible platform performance and monitoring errors, response times and other benchmarks as observed by users.
* Logging and auditing: Important events, especially security-relevant ones are recorded and collected on protected log aggregation infrastructure for inspection and analysis (REQUIRED).&#x20;
  * The logs are being automatically analyzed for anomalies (RECOMMENDED)
  * If this is not the case, they must be inspected manually regularly (CONDITIONALLY REQUIRED)
* Metering: Usage information is collected in a traceable way in order to be able to create billing information. (REQUIRED for public clouds, obviously, but still RECOMMENDED for others)
* Resource Management: In case of hardware or software errors, the cloud-/container platform may end up with resources that are still allocated but no longer in use. These need to be detected and be handled by the Operations team (REQUIRED).&#x20;
  * It is recommended to automate the handling for common cases (RECOMMENDED).
  * For public clouds, it must be ensured that broken resources are not billed to the customer (REQUIRED for public clouds).
* The platform offers a container registry where users can manage and analyze container (and optionally also VM) images (RECOMMENDED).
* The platform offers a status page that the provider uses to communicate that availability and any deviations from it to users (REQUIRED).
