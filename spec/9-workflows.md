# 9 Internal Workflows

This section provides a detailed view of how this Building Block will interact with other Building Blocks to support common use cases.

Main interactions with other building blocks (BBs) is that other BBs run on the virtualized or containerized infrastructure as provided by this BB.

The interaction is typically as follows:

* A virtualized application will come with opentofu configuration or ansible playbooks or pulumi recipes to create the needed infrastructure.
  * Some parameters may need to be adjusted to fit the specific infrastructure (standardization helps here)
  * The Infra-as-Code tool applies the steps as described in the recipe, observing the dependencies by using the API calls to create the wanted resources.
  * Configuration changes can be applied on top, leading to incremental changes (but depending on the tool sometimes in surprising recreations caused by the way the tool works)
  * The task for the infrastructure in all this is really to carefully act as instructed and report status back
* For containerized applications, the workflow typically is as follows:
  * A container cluster gets created via the appropriate API calls (if there is none already)
  * The application will come with a deployment file (yaml) or will use some templating engine that processes the files before submitting them to the container orchestration API
  * The services will create the needed resources – typically downloading the required container images (from the internet or the provided registry), starting them. Typically, no dependency handling is done, as cloud-native applications should be programmed to just retry again and again until all prerequisites are available, making them resilient against sequencing variance in deployment but also in recovery scenarios.
  * Here again, the role of the infra is to accept the requests, validate them and – if done so successfully – execute them.
  * Due to the asynchronous nature of reconciliation loops and retried connections, the final result often needs to be retrieved by looking at the pod and service statuses or the log files.

## 9.1. Workflow for Provisioning a New Virtual Machine <a href="#id-91-workflow-for-provisioning-a-new-virtual-machine" id="id-91-workflow-for-provisioning-a-new-virtual-machine"></a>

Steps and processes involved in creating and configuring a new virtual machine, from resource allocation to OS installation and network setup.

A VM typically needs some prerequisites: A flavor (VM size) needs to be chosen, an image to boot from, a network to connect to at boot time (later connections are possible as well), possibly a security group setting to control the network access to the machine (can be done later as well), an ssh key to inject (on Linux VMs) and possibly storage to attach at boot time (can also be done later, except if there is no disk to boot from).

A typical workflow for creating a single VM is:

* Choose name
* Choose VM size/properties (flavor)
* Choose image to boot
* Choose volume size for the boot disk (unless a flavor is used that already has a fixed size local disk attached)
* Choose one (or multiple) network(s) to connect to or create them via API calls or GUI, optionally connect the network to a router (API or GUI)
* Choose an existing security group or create one (via API calls or GUI) to control network access to the machines network ports
* Choose or create an ssh keypair
* Use API call (or GUI) to create the VM

See chapter 7.1 for an example.

The API calls can be done via a command line interface tool or a programming/scripting language. Most people prefer IaC tools at least once there is more than one VM involved.

## 9.2. Workflow for Deploying a Containerized Application <a href="#id-92-workflow-for-deploying-a-containerized-application" id="id-92-workflow-for-deploying-a-containerized-application"></a>

Detailed sequence of actions for deploying an application using container technologies, including image fetching, container orchestration, and network setup.

A typical workflow would look like this:

* Ensure there is a container cluster
* Adjust parameters in template parameter file
* Render the template into the customized deployment file and submit it to the container orchestration engine’s API
* Watch the creation of pods and services and look at the logs in case things go wrong

Containers orchestration tends to be built in a way that resources just try to connect to needed resources on the fly and retry if they fail. This avoids the need for dependency handling in many cases and makes solutions also robust against services that need to be restarted. On the flipside, many tools don’t even have a notion of dependency handling.

## 9.3. Workflow for Scaling Infrastructure Based on Demand <a href="#id-93-workflow-for-scaling-infrastructure-based-on-demand" id="id-93-workflow-for-scaling-infrastructure-based-on-demand"></a>

Procedures for dynamically scaling up or down the cloud resources based on current load and performance metrics, ensuring efficient resource utilization.

The virtualization layer may offer an orchestration service that allows dynamic creation of VMs behind the loadbalancer based on time or load criteria. (OPTIONAL)

The container orchestration layer supports autoscaling – additional pods (replicas) get deployed based on the load. Alternatively additional CPU or RAM resources can be automatically assigned to pods under high load.(REQUIRED)

The container cluster itself may not have enough capacity, so the cluster size needs to be enhanced; it is recommended to provide the mechanisms to enable cluster autoscaling (RECOMMENDED).

## 9.4. Workflow for Integrating with IAM Systems <a href="#id-94-workflow-for-integrating-with-iam-systems" id="id-94-workflow-for-integrating-with-iam-systems"></a>

Processes for integrating the cloud infrastructure with external identity and access management systems to enable seamless user authentication and authorization across services.

The IAM component in the infrastructure needs to provide APIs (and a GUI) that allow users with the appropriate privileges to create users, groups, roles and assign them to authorize them to access virtualization and/or containerization infrastructure. This user management may instead be performed on a separate Identity Provider that is then consumed by the cloud infrastructure.

Typical workflow:

* Create user
* Assign user to one or several existing groups
  * Review roles that are assigned to the groups to understand the authorizations that come with the groups
  * Optionally create new roles and assign them to new groups
  * Optionally create new projects and add authorizations for the project to existing or new roles
* Ask the user to authenticate and test whether she can access what’s required
