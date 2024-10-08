---
description: >-
  Key Digital Functionalities describe the core (required) functions that this
  Building Block must be able to perform.
---

# 4 Key Digital Functionalities

## 4.1. Virtualization Layer (Storage, Network, Compute) <a href="#id-41-software-defined-infrastructure-storage-network-compute" id="id-41-software-defined-infrastructure-storage-network-compute"></a>

After physical hardware has been deployed in the data center and added to an asset management system, it needs to be virtualized, so it can be sliced up in small pieces and on-demand composed into securely isolated small, medium and big chunks of capacity in a way consumable by customers. Compute capacity (the CPUs and increasingly also the GPUs and main memory) are virtualized by using hypervisor software (supported by commodity hardware technologies), which allows users to use fractions or large parts of a system as needed, securely isolated from other fractions.

Storage is virtualized by having multi-tenant storage systems which emulate virtual disks (block storage) and provide object storage capabilities. Using replication and error correction codes makes large storage systems resilient against failure of single storage media or single storage systems. Virtualized storage opens the possibility of distributed storage solutions.

Network is virtualized giving users that ability to create networks on the fly which are securely segregated from other users’ networks without the need to recable the data center. Good virtualized networks are also resilient against the failure of single network interface cards, cables or physical switches.

Combining these three basic virtualized capabilities – compute, network, storage – allows users to design their virtual data centers and roll out software and run workloads just like on purpose-built hardware setups. With standardized APIs, a virtual setup can be built in minutes that would take months to procure and build on physical hardware. The full automation and the quick creation of virtual resources allow to consider the virtual hardware setup as a fluid resource; scalable workloads can adjust the allocation of virtual resources to the capacity needs by the minute. This is what is meant by rapid elasticity. Virtual hardware has a few extra tricks available – memory, virtual disks, network interfaces may be hot-plugged (and hot-unplugged when not being used any longer) while the workload is running in a virtual machine. A virtual machine running on hardware that needs hardware maintenance (e.g. for exchanging a faulty RAM module or for installing a security update to the firmware or hypervisor) may be live-migrated to another hardware system without notable interruption. This way, hardware maintenance without disruption to user workloads is possible in a cloud environment. The configuration for these workloads can be stored in version control systems and subjected to the same reviews and testing rigor as code – the virtual infrastructure is managed like code – Infrastructure as Code.

There are mature open source technologies for the virtualization of compute, network and storage as well as for the virtualization management layer that abstracts these further by offering standardized APIs for consuming these. The virtualization management layer, the cloud management system, allows for many users to collaborate in common projects within a domain, while being shielded from other projects or users in other domains. It thus creates a secure multi-tenant platform, tracking all these virtual resources (and their assignment to physical resources) using hardware capabilities and virtualization technology to securely isolate the virtual resources from each other while offering the self-service APIs that allows users to e.g. plug virtual network cables into virtual switches to connect a virtual machine to a virtual network or moving a public internet-exposed IP address from one VM to another.

Using these APIs and readily available open source software, higher level abstractions can be built, e.g. for backup services, virtual load balancers, DNS service or secret stores and exposed via standardized APIs again. The mentioned examples should be considered standard features of a capable cloud management platform.

Cloud platforms that are typically used for building private clouds often allow for a huge amount of configurability. While this allows for highly specialized clouds, it comes with the downside of destroying the network effects of highly standardized platforms that has made the public cloud platforms so successful, facilitating the emergence of large ecosystems of applications. Standardizing the APIs and system behavior of cloud platforms (without necessarily prescribing many of the implementation details) and codifying best practices for these is thus a prerequisite for creating platforms that support self-reinforcing ecosystems. Where differences do provide meaningful differentiation for providers, standardized ways to make these discoverable are required.

## 4.2. Container Cluster (Modern platform for Application Management) <a href="#id-42-modern-platform-for-application-management-container-technology" id="id-42-modern-platform-for-application-management-container-technology"></a>

Virtualization technology emulates real hardware with the advantage that you can run your own operating system (that was developed to run on physical hardware) inside a virtual machine. While this gives a lot of flexibility, it also creates some overhead. The operating system needs to be booted to initialize itself and the emulated hardware may not be the best level of abstraction for the individual services and components that comprise a workload. Taking the speed of elasticity to the next level, containers are being used. These are launched like normal processes in an already running operating system, just with their individual view of the file system (and thus e.g. the set of libraries being available), network connections and CPU and memory allocation. Elasticity is possible a the scale of seconds and below using container technology.

While the level of isolation between containers is not typically considered high enough to support scenarios where containers from potentially adversary parties should run on the same node, the isolation is certainly good enough to cleanly separate the various components of workloads run by one entity. The lack of interference has made container building a standard way of distributing complex software, allowing to overcome difficult to manage dependencies on system libraries and requirements for specific versions of supporting software.

To take advantage of the high flexibility of creating, changing and deleting containers on the fly, new abstractions to manage a fleet of containers that comprise a workload have been developed. Containers that belong together can be grouped and scaled as groups. Policies with respect to the network connections may be enforced centrally. Memory or CPU limits may be imposed. Containers may be replicated over several nodes to ensure resilience against failure.

Modern container orchestration technology is characterized by using a declarative way to describe the configuration of containers and their network and storage setups. Using sufficiently powerful abstractions in the declarative description of the desired state allows the container orchestrator’s reconciliation loop to continuously take actions as needed to adjust the reality to the desired state. This takes the burden off the teams that operate workloads to react to unexpected events (e.g. an application crash due to overload), as the container management system can be instructed to restart automatically (and to avoid the problem in the first place by using autoscaling on the service container before it becomes overloaded).

The container orchestration solutions knows what actions it needs to take to reconcile reality with the desired state (e.g. start a container if there should be one running but there is none). Building upon this powerful concept, so-called operators have emerged as a technology to reconcile reality with the desired state for more complex, custom-defined resources, such as e.g. a database service.

Containers are managed inside a set of nodes that belong together – a container cluster. In a virtualized environment, these nodes are typically virtual machines. To work well, these clusters should take advantage of the underlaying cloud infrastructure, by e.g. using the network management capabilities, accessing and managing persistent storage, ensuring that nodes use optimized node images and run on different physical hardware etc. This is the job of the container cluster management solution. With it, users can not just automate the deployment of workloads to an existing container cluster, but they can create, scale, upgrade, change and remove clusters on the fly (in the scale of minutes). This way, container clusters can become an elastic resource, to be created on demand for development, test, reference or production usage. All of this of course can be done in automated, API-driven approaches. DevOps teams often love to keeping all of their work inside the git version control system, using the same review and test processes for all of their work. While some git commits trigger code to be compiled and tested, others may cause test infrastructure to spin up and perform testing of an infrastructure change. This modern approach to managing virtualized and/or containerized workloads is called gitops. The declarative approach to container management makes it particularly suitable for this.

All the fluidity of code and infrastructure may seem to create instability. And while it’s true that changes can happen a lot faster across many more layers of a production environment in a container orchestration environment, good engineering teams have built practices that leverage the automation and flexibility to impose rigorous testing practises. If an infrastructure change can be tested just like a software code change can, it will be. Test environments can be created and deleted on the fly; it is much more practical in such environments to have test environments that closely resemble the real production environment. Except for using mock data and some artificial automated tests, it is best practice to use exactly the same code with very similar configuration to do the integration testing as is used for production rollout. This can happen on every proposed code change – an approach called continuous integration (testing). It is very common on containerized application development.

All these advantages have made containerized workloads the standard choice for most freshly developed modern application development. This is why many people equate the term “cloud native” with containerized micro-service architectures.

## 4.3. Security by Design <a href="#id-43-security-by-design" id="id-43-security-by-design"></a>

In this section security approaches of cloud computing environments are described in addition to: [https://govstack.gitbook.io/specification/security-requirements](https://govstack.gitbook.io/specification/security-requirements).

Complex technology comes with weaknesses. Whether those are conceptual weaknesses, software bugs, hardware bugs or just limitations that are not well documented, these put the usefulness of IT solutions at risk. Worse, a signifcant fraction of these issues may be abused by actors to gain unauthorized access to data or even control over other parties’ IT systems. These actors may range from curious (and otherwise harmless) engineers or researchers via cybercriminals that want to extort money to intelligence services or military units from nation states. Having several layers of protection against such actors and automated or trained reactions in case of breaches is a baseline security requirement for every IT solution outside of highly segregated environments.

Some best practices need to be built into the operational processes and the architecture of cloud and container solutions to be resilient:

(1) A clear distinction between different roles and the coresponding authorizations; a well-documented way to manage these authorizations, and strong authentication mechanisms.

(2) Well-trained staff that understands the risks of falling prey to social engineering attacks and with sufficient staffing that allows people to think twice and to follow four-eyes principles for administrative operations with elevated privileges.

(3) A set of rules, permissions and processes that allows people to do their work without violating the rules and that is thus followed and enforced in real-life.

(4) Using well-defined and limited interfaces with good abstractions that reduce the attack surface (for example the virtualization abstraction where a well-understand hardware interface is being used for isolation).

(5) A learning culture that does avoid shaming people for making mistakes but focuses on analyzing errors, making issues transparent and providing a learning experience beyond just the involved indidividuals, creating processes and automation that ensures similar mistakes can not happen again.

(6) A collaboration culture that encourages reviews of each other’s work, where questioning the design and implementation of a solution is considered valuable feedback.

(7) An approach of least privileges; each actor (human or machine) only has the privileges it needs to do its job. This also means that actors that needs lots of different privileges probably should be split into several actors. This may mean that human beings (often operators) may be assigned several roles, but can only perform actions with one of them at a time.

(8) A system of defense in depth, where the elevation of privileges on one system tends to be contained to affect this one system only.

(9) Using encryption (with published, well-understood, state-of-the-art algorithms) for data at rest and in transit to ensure that eavesdropping does not compromise the confidentiality of data.

(10) A clear understanding that customer input may never be trusted and always needs to be validated / sanitized when being processed.

(11) The usage of secure programming languages and/or tools that scan for typical programming mistakes.

(12) Using penetration testers to find weaknesses in a system.

(13) Offering a way for security researchers to report findings and reward them.

(14) Creating a security team and providing contact information for outsiders to contact it.

(15) Evaluating published security reports and be connected to relevant pre-disclosure security channels to receive advance warnings.

(16) Publishing security advisories and providing and/or deploying security fixes short-term. This requires running reference environments such that security issues can be reproduced, fixes be validated and then deployed to production with confidence without long lead times.

(17) Adhering to relevant security standards and certifying compliance against them.

## 4.4. IAM (Identity and Access Management) <a href="#id-45-iam-identity-and-access-management" id="id-45-iam-identity-and-access-management"></a>

In this section Identity and Access Management approaches of cloud computing environments are described in addition to the creation, verification and authentication of identities: [https://govstack.gitbook.io/bb-identity](https://govstack.gitbook.io/bb-identity).

State of the art identity and access management systems allow user administrators to assign authorizations to users by assigning them to a group that reflects their job assignment. Experts inside the organization that understand how cloud infrastructure is being employed inside a project can then do a fine-grained setting for that group, deciding to what virtual environments and resources full access and read-only access is required.

In a federated scenario, authentication and group assignment are typically done within a customer Identity Management system; the cloud’s Identity and Access management system then needs to support the federation with the customer’s system. Ideally this can be configured by the customer itself. The IAM system in the specific cloud then needs to be configured to map the groups to specific authorizations. This is also under the customer’s control.

It is recommended to keep the authorization primitives simple as in “group G has the right to fully control resources of type T in project P” or “group H has the right to have read-only acccess to resources of type U in project Q”. This makes audits possible and allows for reasoning about security properties. More complex systems may allow for more flexibility, but easily end up being not fully understood by users and operators, provoking misuse that can easily result in security vulnerabilities.

Identity and Access Management (IAM) needs to support federation; allowing to keep authentication and group assignment in a corporate directory or use the system from another cloud that uses the same standards. It needs to allow for a high degree of self-service. It needs to cover access decisions to both the virtualization and the container layer (if both are offered by an infrastructure offering). It needs to provide a high level of availability.

## 4.5. Observability <a href="#id-46-observability" id="id-46-observability"></a>

Highly distributed dynamic systems such as cloud and container platforms have a high tendency for complex patterns of unwanted behavior. With increasing maturity of the management software, this may become less frequent, but it is absolutely best practice to closely monitor the availability of all services on a platform. Beyond just the monitoring of service availability (“does the service respond when I connect to it?”), it is recommended to do scenario testing, where a model workload is being automatically deployed into the cloud and/or container infrastructure and tests are done to measure the workload’s functioning. Recording success rates and also performance allows for seeing trends and detecting services that may respond to requests but don’t successfully fulfill them. This is often called health-monitoring.

Alerts can be created and simple, recurring issues can be acted on automatically, whereas other alerts need to reach human beings for further rectification and resolution. Alerts should also be triggered based on trends, where extrapolating e.g. a usage trend can engage capacity management actions such as procuring and deploying additional hardware before capacity runs out.

Health monitoring can best be done from a user’s perspective with normal user’s rights (i.e. without any elevanted privileges). Good providers share the results with their customers, providing real-time transparency over the health of the platform.

Health-monitoring from a user perspective (top-down) should be complemented by infrastructure monitoring (bottom-up), where hardware and software failures are reported. While a resilient setup hides most of these failures from becoming user-visible, they do tend to impact the level of redundancy or performance, so they do need to be acted upon, just with significantly lower urgency.

For debugging difficult issues, logs will be collected and need to be aggregated, correlated and indexed for searchability. Logs may also be evaluated in case of security incidents or to audit that system’s compliance status.

## 4.6. Life cycle management <a href="#id-47-life-cycle-management" id="id-47-life-cycle-management"></a>

The installation of cloud infrastucture can be automated – once all the hardware setup is completed and properly recorded in the asset management system. The automation of course is especially useful when setting up test or reference clouds. The freedom to do this regularly offers the ability to test various software states in an automated way, e.g. performing dozens of upgrades (and rollbacks) before touching the production environment. It is recommended to keep the system configuration in a versioned software revision control system such as git.

The process how to enlarge an environment with additional hardware (or virtual hardware when talking about the container layer) needs to be automated and documented. The same holds true for retiring old (or broken) hardware.

During operation, updates and especially bug and security patches may need to be installed rather often; the lifecycle management tools need to facilitate the collection of the current software state and the rollout of a defined newer state. Rollback needs to be possible as well to be able to go back to a known working (though potentially vulnerable) state.

Software occasionally makes larger jumps, adding lots of new functionality, but then also including breaking changes. Good software management minimizes breaking changes and announces these ahead of time via e.g. deprecation notices. Good release notes for major updates help users to assess the impact.

The life cycle management tools need to support with installing major updates. The process documentation should contain information on the expected impact, e.g. downtimes for the control plane, reduced performance or so. Providers should use these to announce the impact to their customer ahead of time.

## 4.7. Metering <a href="#id-48-metering" id="id-48-metering"></a>

The cost of running the platform needs to be allocated according to the usage; what is a requirement for a public cloud still makes sense for internal platforms to track the needs and usage of various departments (and encourage avoiding unecessarily large consumption of resources and energy).

The cloud and container platforms need to support recording the usage; this requires at least the recording of creation and deletion events for the metered resources. As events can get lost, the metering solution should compare the state derived from the state machine fed from the events with the status that the platform reports – if an event that a VM has been created was received, but no deletion event, then the state machine would predict that the VM must exist, which can be validated. Errors must be flagged, so operators can investigate and erroneous invoices can be avoided. With such a validation logic, an event based system is preferable over a pure polling system due to much higher efficiency and better accuracy.
