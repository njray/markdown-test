---
title: Run a Jenkins server on Azure
description:  This reference architecture shows shows how to deploy and operate a scalable,
enterprise-grade Jenkins server on Azure secured with single sign-on (SSO).
author: njray
ms.date: 01/21/18
---

# Run a Jenkins server on Azure

This reference architecture shows how to deploy and operate a scalable, enterprise-grade Jenkins server on Azure secured with single sign-on (SSO). The focus of this document is on the core Azure operations needed to support Jenkins, including the use of Azure Storage to maintain build artifacts, the security items needed for SSO, other services that can be integrated, and scalability for the pipeline. The deployed architecture also uses Azure monitoring services to monitor the state of the Jenkins server.

This architecture supports disaster recovery with Azure services but does not cover more advanced scale-out scenarios involving multiple masters or high availability (HA) with no downtime. For general insights about the various Azure components, including a step-by-step tutorial about building out a CI/CD pipeline on Azure, see [Jenkins on  Azure][jenkins-on-azure].


![[0]][0]
Figure 1. Jenkins architecture on Azure.


> [!NOTE]
>  This reference architecture requires an Azure subscription. These instructions assume that the Jenkins administrator is also an Azure user with at least Contributor privileges.
>
>



## Architecture

This reference architecture creates a Jenkins virtual machine and installs and configures Jenkins and Azure security and monitoring services. The architecture is designed to work with an existing source control repository. For example, a common scenario is to start Jenkins jobs based on GitHub commits.

The architecture consists of the following components:

-   **Resource group.** A [resource group][rg] is used to group Azure assets so they can be managed by lifetime, owner, and other criteria. Use resource groups to deploy and monitor Azure assets as a group and track billing costs by resource group. You can also delete resources as a set, which is very useful for test deployments.

-   **Jenkins server**. A virtual machine is deployed to run [Jenkins][azure-market] as an automation server and serve as Jenkins Master. This reference architecture uses the [solution template for Jenkins on Azure][solution], installed on a Linux (Ubuntu 14.04 LTS) virtual machine on Azure. Other Jenkins offerings are available in the Azure Marketplace.

> [!NOTE]
>  Nginx is installed on the Microsoft Jenkin server as a reverse proxy and can be configured to run SSL on your server.
>
>


-   **Virtual network**. A [virtual network][vnet] connects Azure resources to each other and provides logical isolation. In this architecture, the Jenkins server runs in a virtual network.

-   **Subnets**. The Jenkins server is isolated in a [subnet][subnet] to make it easier to manage and segregate network traffic without impacting performance.

-   **NSGs**. Use [network security groups][nsg] (NSGs) to restrict network traffic from the Internet to the subnet of a virtual network.

-   **Managed disks**. A [managed disk][managed-disk] is a persistent virtual hard disk (VHD) used for application storage and also to maintain the state of the Jenkins server and provide disaster recovery. Data disks are stored in Azure Storage. For high performance, [premium storage][premium] is recommended.

-   **Azure Blob Storage**. The [Windows Azure Storage plugin][configure-storage] uses Azure Blob  Storage to store the build artifacts that are created and shared with other Jenkins builds.

-   **Azure Active Directory (Azure AD)**. [Azure AD][ad] supports user authentication, allowing you to set up SSO. Azure AD [service principals][service-principal] define the policy and permissions for each role authorization in the workflow via [role-based access control][rbac] (RBAC). Each service principal is associated with a Jenkins job.

-   **Azure Key Vault.** To manage secrets and cryptographic keys used to provision Azure resources when secrets are required, this architecture uses [Key Vault][key-vault]. For added help storing secrets associated with the application in the pipeline, see also the [Azure Credentials][configure-credential] plugin for Jenkins.

-   **Azure monitoring services**. This service [monitors][monitor] the Azure virtual machine hosting Jenkins. This deployment monitors the virtual machine status and CPU utilization and sends alerts.

## Recommendations

The following recommendations provide best practices for building and deploying a Jenkins server on Azure.

### Azure AD

The [Azure AD][azure-ad] tenant for your Azure subscription is used to enable SSO for Jenkins users and
set up [service principals][service-principal] that enable Jenkins jobs to access Azure resources.

SSO authentication and authorization are implemented by the Azure AD plugin  installed on the Jenkins server. SSO allows you to authenticate using your organization credentials from Azure AD when logging on to the Jenkins server. When configuring the Azure AD plugin, you can specify the level of a user’s authorized access to the Jenkin server.

To provide Jenkins jobs with access to Azure resources, an Azure AD administrator creates service principals. These grant applications—in this case, the Jenkins jobs—[authenticated, authorized access][ad-sp] to Azure resources.

[RBAC][rbac] further defines and controls access to Azure resources for users or service principals through their assigned role. Both built-in and custom roles are supported. Roles also help secure the pipeline and ensure that a user’s or agent’s responsibilities are assigned and authorized correctly.

In addition, RBAC can be set up to limit access to Azure assets. For example, a user can be limited to working with only the assets in a particular resource group.

### Key Vault

[Key Vault][key-vault] provides secrets management and is added to the Jenkins server resource group. The Key Vault service is made available by the [Azure Credential plugin][configure-credential].  Key Vault stores the service principals, passwords, tokens, and other secrets required by the Jenkins jobs to use applications and access Azure services that require authorization, such as Azure Container Service.

### Storage

Use the Jenkins [Windows Azure Storage plugin][storage-plugin], which is installed from the Azure Marketplace, to store build artifacts that can be shared with other builds and tests. An Azure Storage account must be configured before this plugin can be used by the Jenkins jobs.

### Jenkins Azure plugins

The solution template for Jenkins on Azure installs several Azure plugins. The Azure DevOps Team builds and maintains the solution template and the following plugins, which work with other Jenkins offerings in Azure Marketplace as well as any Jenkins master set up on premises:

-   [Azure AD plugin]configure-azure-ad] allows the Jenkins server to support SSO for users based on Azure AD.

-   [Azure VM Agents][configure-agent] plugin uses an Azure Resource Manager (ARM) template to create Jenkins agents in Azure virtual machines.

-   [Azure Credentials][configure-credential] plugin allows you to store Azure service principal credentials in Jenkins.

-   [Windows Azure Storage plugin][configure-storage] uploads build artifacts to, or downloads build dependencies from, [Azure Blob storage][blob].

We also recommend reviewing the growing list of all available Azure plugins that work with Azure resources. To see all the latest list, visit [Jenkins Plugin Index][index] and search for Azure. For example, the following plugins are available for deployment:

-   [Azure Container Agents][container-agents] helps you to run a container as an agent in Jenkins.

-   [Kubernetes Continuous Deploy](https://aka.ms/azjenkinsk8s) deploys resource configurations to a Kubernetes cluster.

-   [Azure Container Service][acs] deploys configurations to Azure Container Service with Kubernetes, DC/OS with Marathon, or Docker Swarm.

-   [Azure Functions][functions] deploys your project to Azure Function.

-   [Azure App Service][app-service] deploys to Azure App Service.

### Azure Monitor

Use [Azure monitoring services][monitoring] to monitor the Jenkins server or other Azure resources used by Jenkins. In this architecture, [Azure portal][portal] is used to provision the monitoring services (step 6 in the deployment).

Azure monitoring services monitor CPU utilization and the status of the virtual machine used for the Jenkins server. A notification is sent if CPU usage exceeds 80 percent, indicating that you might want to scale up the Jenkins server virtual machine. If the virtual machine fails, a designated user is notified that the server is down.

## Scalability considerations

Jenkins can scale to support very large workloads. For elastic builds, do not run builds on the Jenkins master server. One option for scaling out using the virtual machines is to use the [Azure VM Agents plugin][vm-agent]. Another option is to scale out the build to the cloud using the container technology in the Azure Container plugin. Virtual machines cost more to scale than containers. To use containers for scaling, your build process must run with Containers.

Also consider using Azure Storage to share build artifacts that may be used in the next stage of the pipeline by other build agents.

Take advantage of the [Azure VM Agents][configure-agent] plugin to offload build tasks, allow scale-up of the Jenkins server, and support elastic scale-out of the pipeline. This plugin enables elastic scale-out for agents and can use distinct types of virtual machines.

### Scaling the Jenkins server 

Scale the Jenkins server virtual machine up or down by changing the virtual machine size. The [solution template for Jenkins on Azure][azure-market] specifies the DS2 v2 size (with two CPUs, 7 GB) by default. This size handles a small to medium team workload. Change the virtual machine size by choosing a different option when building out the server.

Selecting the correct server size depends on the size of the expected workload. The Jenkins community maintains a [selection guide][selection-guide] to help identify the configuration that best meets your requirements. Azure offers many [sizes for Linux VMs][sizes-linux]to meet any requirements.

For more information about scaling the Jenkins master, refer to the Jenkins community of [best practices][best-practices], which also
includes details about scaling Jenkins master.

### Scaling the Jenkins pipeline with Azure VM Agents plugin

The [Azure VM Agents plugin][vm-agent] supports elasticity by building out or reducing the number of build agents. You can select a different base image from Azure Marketplace or use a custom image. For details about how the Jenkins agents scale, see [Architecting for Scale][scale] in the Jenkins documentation.

## Availability considerations

Assess the availability requirements for your workflow and how to recover the Jenkin state should the Jenkin server goes down. To assess your availability
requirements, consider two common metrics:

-   Recovery Time Objective (RTO) specifies how long you can go without Jenkins.

-   Recovery Point Objective (RPO) indicates how much data you can afford to lose if a disruption in service affects Jenkins.

In practice, RTO and RPO imply redundancy and backup. Availability is not a question of hardware recovery—that is part of Azure—but rather ensuring you maintain the state of your Jenkins server. This reference architecture uses the [Azure service level agreement][sla] (SLA), which guarantees 99.9-percent uptime for a single virtual machine. If this SLA doesn't meet your uptime requirements, make sure you have a plan for disaster recovery, or consider using a [multi-master Jenkins server][multi-master] deployment (not covered in this document).

Consider using the disaster recovery scripts in step 7 of the deployment to create an Azure Storage account with managed disks to store the Jenkins server state. If Jenkins goes down, it can be restored to the state stored in this separate storage account.

## Security considerations

Use the following approaches to help lock down security on a basic Jenkins server, since in its basic state, it is not secure.

-   Set up a way to secure logon to the Jenkin server. HTTP is not secure By default, this architecture uses HTTP and has a public IP. Consider setting up [HTTPS on the nginx server][nginx] being used for a secure logon.

> [!NOTE]
>  When adding SSL to your server, create an NSG rule for the Jenkins subnet to [open port 443][port443].
>
>



-   Ensure that the Jenkins configuration prevents cross site request forgery (Manage Jenkins \> Configure Global Security). This is the default for Microsoft Jenkins Server.

-   Configure read-only access to the Jenkins dashboard by using the [Matrix Authorization Strategy Plugin][matrix].

-   Install the [Azure Credentials][configure-credential] plugin to use Key Vault to handle secrets for the Azure assets, the agents in the pipeline, and third-party components.

-   Create a security profile that defines the resources required by users, services, and pipeline agents to do their jobs—but no more. This step becomes critical when considering your security settings.

To get a central view of the security state of your Azure resources, use [Azure Security Center][security-center]. Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment. Security Center is configured per Azure subscription. Enable security data collection as described in the [Azure Security Center quick start guide][quick-start]. When data collection is enabled, Security Center automatically scans any virtual machines created under that subscription.

The Jenkins server has its own user management system, and the Jenkins community provides best practices for [securing a Jenkins instance on Azure][secure-jenkins]. The solution template for Jenkins on Azure implements these best practices.

## Manageability considerations

This reference architecture uses a resource group to make it easier to manage Azure resources. You can monitor each environment’s resources and roll up billing costs by resource group. You can also delete resources as a set, which is very useful for test deployments.

Azure also provides several functions for [monitoring and diagnostics][monitoring-diag] of the overall infrastructure. To monitor CPU usage, this architecture deploys Azure Monitor.

## Communities

Communities can answer questions and help you set up a successful deployment. Consider the following:

-   [Jenkins Community Blog](https://jenkins.io/node/)

-   [Azure Forum](https://azure.microsoft.com/en-us/support/forums/)

-   [Stack Overflow Jenkins](https://stackoverflow.com/tags/jenkins/info)

For more best practices from the Jenkins community, visit [Jenkins best practices][jenkins-best].

## Solution deployment

To deploy this architecture, follow the steps below to install the [solution template for Jenkins on Azure][azure-market], then install the scripts that set up monitoring and disaster recovery. These scripts are available on [Github][github].

**Prerequisite:**

-   To create an Azure service principal, you must have admin rights to the Azure AD tenant that is associated with the deployed Jenkins server.

### Step 1: Deploy the Jenkins server

1.  Open the [Azure marketplace image for Jenkins][azure-market] in your web browser and select **GET IT NOW** from the left side of the page.

2.  Review the pricing details and select **Continue**, then select **Create** to configure the Jenkins server in the Azure portal.

For detailed instructions, see [Create a Jenkins server on an Azure Linux VM from the Azure portal][create-jenkins]. For this reference architecture, it is sufficient to get the server up and running with the admin logon. Then you can provision it to use various other services.

### Step 2: Set up SSO

The step is run by the Jenkins administrator, who must also have a user account in the subscription’s Azure AD directory and must be assigned the Contributor role.

Use the [Azure AD Plugin][configure-azure-ad] from the Jenkins Update Center in the Jenkin server and follow the instructions to set up SSO.

### Step 3: Provision Jenkins server with Azure VM Agent plugin

The step is run by the Jenkins administrator to set up the Azure VM Agent plugin, which is already installed.

[Follow these steps to configure the plugin][configure-agent]. For a tutorial about setting up service principals for the plugin, see [Scale your  Jenkins deployments to meet demand with Azure VM agents][scale-agent].

### Step 4: Provision Jenkins server with Azure Storage

The step is run by the Jenkins administrator, who sets up the Windows Azure Storage Plugin, which is already installed.

[Follow these steps to configure the plugin][configure-storage].

### Step 5: Provision Jenkins server with Azure Credential plugin

The step is run by the Jenkins administrator to set up the Azure Credential plugin, which is already installed.

[Follow these steps to configure the plugin][configure-credential].

### Step 6: Provision Jenkins server for monitoring by the Azure Monitor Service

To set up monitoring for your Jenkins server, follow the instructions to [create metric alerts in Azure Monitor for Azure services—Azure portal.][create-metric].

### Step 7: Provision Jenkins server with Managed Disks for disaster recovery

The Microsoft Jenkins product group has created disaster recovery scripts that build a managed disk used to save the Jenkins state. If the server goes down, it can be restored to its latest state.

Download and run the disaster recovery scripts from [GitHub][disaster].

[acs]: https://aka.ms/azjenkinsacs
[ad]: https://azure.microsoft.com/en-us/services/active-directory/
[ad-sp]: /azure/active-directory/develop/active-directory-integrating-applications
[app-service]: https://plugins.jenkins.io/azure-app-service
[azure-ad]: https://azure.microsoft.com/en-us/services/active-directory/
[azure-market]: https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview
[best-practices]: https://jenkins.io/doc/book/architecting-for-scale/
[blob]: /azure/storage/common/storage-java-jenkins-continuous-integration-solution
[configure-azure-ad]: https://plugins.jenkins.io/azure-ad
[configure-agent]: https://plugins.jenkins.io/azure-vm-agents
[configure-credential]: https://plugins.jenkins.io/azure-credentials
[configure-storage]: https://plugins.jenkins.io/windows-azure-storage
[container-agents]: https://aka.ms/azcontaineragent
[create-jenkins]: /azure/jenkins/install-jenkins-solution-template
[create-metric]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[disaster]: https://github.com/zackliu/haforjenkins/blob/master/HA-ManagedDisk.md
[functions]: https://aka.ms/azjenkinsfunctions
[github]: http://www.github.com
[index]: https://plugins.jenkins.io
[jenkins-best]: https://wiki.jenkins.io/display/JENKINS/Jenkins+Best+Practices
[jenkins-on-azure]: /azure/jenkins/
[key-vault]: https://azure.microsoft.com/en-us/services/key-vault/
[managed-disk]: (https://azure.microsoft.com/en-us/services/managed-disks/
[matrix]: https://plugins.jenkins.io/matrix-auth
[monitor]: https://azure.microsoft.com/en-us/services/monitor/
[monitoring]: /azure/monitoring-and-diagnostics/monitoring-overview
[monitoring-diag]: /azure/architecture/best-practices/monitoring
[multi-master]: https://jenkins.io/doc/book/architecting-for-scale/
[nginx]: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
[nsg]: /azure/virtual-network/virtual-networks-nsg
[quick-start]: /azure/security-center/security-center-get-started
[portal]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[port443]: /azure/virtual-machines/windows/nsg-quickstart-portal
[premium]: /azure/virtual-machines/windows/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rg]: /azure/azure-resource-manager/resource-group-overview
[scale]: https://jenkins.io/doc/book/architecting-for-scale/
[scale-agent]: /azure/jenkins/jenkins-azure-vm-agents
[selection-guide]: https://jenkins.io/doc/book/hardware-recommendations/
[secure-jenkins]: https://jenkins.io/blog/2017/04/20/secure-jenkins-on-azure/
[security-center]: /azure/security-center/security-center-intro
[service-principal]: /azure/active-directory/develop/active-directory-application-objects)
[sizes-linux]: /azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[solution]: https://azure.microsoft.com/en-us/blog/announcing-the-solution-template-for-jenkins-on-azure/
[sla]: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_6/
[storage-plugin]: https://wiki.jenkins.io/display/JENKINS/Windows+Azure+Storage+Plugin
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[vm-agent]: https://wiki.jenkins.io/display/JENKINS/Azure+VM+Agents+plugin
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/Jenkins-diagram.png "Jenkins architecture on Azure."