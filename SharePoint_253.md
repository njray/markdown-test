---
title: Run a high availability SharePoint Server 2016 farm in Azure
description:  This reference architecture shows a set of proven practices for setting up a high availability SharePoint Server 2016 farm using MinRole topology and SQL Server Always On availability groups on Azure.
author: njray
ms.date: 05/30/17
---

# Run a high availability SharePoint Server 2016 farm in Azure
[!INCLUDE [header](../../_includes/header.md)]
Contributors: Neil Hodgkinson, Bob Fox, Joe Davies, Paul Stork, Larry Brader

This reference architecture shows a set of proven practices for setting up a
high availability SharePoint Server 2016 farm using MinRole topology and SQL
Server Always On availability groups on Azure. The SharePoint farm is deployed
in a secured intranet with no Internet-facing endpoint or presence.

![[0]][0]
Figure 1. Highly available SharePoint Server 2016 farm

> [!NOTE]
>  This template deploys the virtual network and the components within it (shown in blue). This architecture doesn’t implement the Active Directory trust relationship (shown in red) with an existing on-premises environment (shown in purple). You must also configure the gateway yourself.
>
>


## Architecture

This architecture builds on the one shown one shown in [Run Windows VMs for an n-tier application][n-tier]. In the Azure IaaS environment, this architecture deploys a SharePoint Server
2016 farm with high availability. It is suitable for a test or production
environment, a SharePoint hybrid infrastructure with Office 365, or as the basis
for a disaster recovery scenario.

The architecture consists of the following components:

-   **Resource groups.** For different roles or tiers, separate [resource groups][resource-group] are 
    typically created so assets can be deployed, managed, and monitored as a
    group. Here, a single resource group is used for the SharePoint servers. A
    separate resource group is used for infrastructure components that are
    independent of VMs, such as the virtual network and load balancers.

-   **Availability sets.** For high availability, an [availability set][availability-set]
    is created for each SharePoint role, and at least two virtual machines are
    provisioned for each role. This makes the virtual machines eligible for the
    availability [service level agreement][sla] (SLA) for Azure VMs.

-   **Internal load balancer.** Client-based SharePoint request traffic from the
    intranet is distributed to the front-end web servers of the SharePoint farm
    through an [internal load balancer][ilb].

-   **Virtual network**. Virtual machines are deployed in a virtual network with
    a unique intranet address space. A separate subnet is created for each group
    of virtual machines for a specific role and for the gateway subnet. To
    specify the address range and subnet mask, CIDR notation is used. Assign the
    [gateway subnet address space](https://blogs.technet.microsoft.com/solutions_advisory_board/2016/12/01/calculating-the-gateway-subnet-address-space-for-azure-virtual-networks/) from the last part of the virtual network address space.

-   **Virtual machines**. Private static IP addresses are assigned to all the
    virtual machines. Dynamic addresses are not recommended for server virtual machines 
    to avoid issues with IP address caching and changes of addresses across a restart.

-   **Network security groups**. For each subnet that contains virtual machines,
    a [network security group][nsg] is created. These groups enable you to perform subnet isolation.

-   **Gateway.** This
    [gateway][gateway] provides a connection between your on-premises network and the Azure virtual
    network. Your connection can use ExpressRoute or site-to-site [VPN][vpn].

-   **Windows Server Active Directory (AD) domain controllers.** Because
    SharePoint Server 2016 does not support using Azure Active Directory Domain
    Services, you must deploy Windows Server AD domain controllers. These domain
    controllers run in the Azure virtual network and have a trust relationship
    with the on-premises Windows Server AD forest. Client web requests for
    SharePoint farm resources are authenticated in the virtual network rather
    than sending that authentication traffic across the cross-premises
    connection to your on-premises network. In DNS, intranet A or CNAME records
    are used so intranet users can resolve the name of the SharePoint farm to
    the private IP address of the internal load balancer.

-   **SQL Server Always On availability group.** For high availability, two
    virtual machines are used for SQL Server—one that contains the primary
    database replica of an [availability group][availability-group] and the other for 
    the secondary backup replica.

-   **Majority node.** For the failover cluster, a [majority node][majority-node] virtual machine is created.

-   **SharePoint servers**. The SharePoint servers perform the web front-end,
    caching, application, and search [roles][roles].


##Recommendations

Your requirements might differ from the architecture described here. Use these
recommendations as a starting point.

### Virtual network and subnet recommendations

The Azure best practice is to use one
[subnet][subnet] per role or tier, plus a subnet for the gateway and one for the management
subnet.

### Availability set recommendations

Make sure you have a sufficient number of cores in your subscription or the
deployment will fail. You can calculate the [sizes for Windows virtual machines in Azure][vm-sizes].
Based on Standard DSv2 virtual machine sizes, this architecture requires at
least 38 cores:

```
8 SharePoint servers × 4 cores each = 32 Standard\_DS3\_v2  
  + 2 Active Directory domain controllers × 1 core each = 2 Standard\_DS1\_v2  
  + 1 majority node × 1 core = 1 Standard\_DS1\_v2  
  + 1 management server × 1 core = 1 Standard\_DS1\_v2  
  + 2 SQL servers × 1 core each = 2 Standard\_DS1\_v2  
= 38 total
```

For more information, see [Manage the availability of Windows virtual machines in Azure][manage-availability].

### Resource group recommendations

We recommend separating resource groups according to the server role, and having
a separate resource group for infrastructure components that are global
resources. In this architecture, the SharePoint resources form one group, while
the SQL Server and other utility assets form another.

> [!NOTE]
>  After you create a resource group, you can’t rename it.
>
>

For more information about resource groups, see the [Azure Resource Manager overview][arm].

### Network security group recommendations

In addition to the default [network security group][nsg]
rules, this architecture implements a rule in the SQL Server network security
group that allows TCP port 1433 or port 3389 traffic from any intranet location.

The Azure best practice is to use one network security group per subnet
containing virtual machines. In this reference architecture, the SharePoint
servers are assigned to subnets based on their role, and each subnet is
associated with an availability group for high availability and a network
security group for subnet isolation. To support subnet isolation, the network security group for each subnet
can be initially configured with a customized, explicit "all traffic" rule that allows open, inter-subnet traffic. 
Then, for isolation, create the set of rules specifying the allowed inter-subnet traffic, reassociate 
the new rules with the subnet you want to isolate, and don't include the “all traffic” rule.

Do not assign a network security group to the [gateway subnet][gateway-subnet-ra],
because the gateway will stop functioning.

These Azure best practices primarily follow the reference architecture for
[n-tier applications][n-tier], 
but deployment models can vary. For more information, 
see [Filter network traffic with network security groups][filter-network].

### Storage recommendations

The storage configuration of the virtual machines in the farm should match the
appropriate best practices used for on-premises deployments. SharePoint servers
should have a separate disk for logs. SharePoint servers hosting search index
roles require additional disk space for the search index to be stored. For SQL
Server, the standard practice is separation of data and logs. Add more disks for
database backup storage, and use a separate disk for [tempdb][tempdb].

For simplicity, we recommend using Azure Managed Disks to manage the storage
accounts associated with the VM disks. For the entire deployment, use Premium
storage for all roles except the majority node server, domain controllers, and
the management server.

The Managed Disks service makes it much simpler to deploy your VMs. For example,
you no longer have to set up additional storage accounts per VM to handle limits
(such as 20,000 IOPS per account), because Managed Disks handles storage behind
the scenes. Managed Disks also makes sure the disks of VMs in an availability
set are sufficiently isolated from each other to avoid single points of failure.

To learn more, see [Azure Managed Disks Overview][managed-disks].

### SharePoint Server recommendations

Before configuring the SharePoint farm, make sure you have one Windows Server AD
service account per service. For this architecture, you need at a minimum the
following domain-level accounts to isolate privilege per role:

-   SQL Server Service account
-   Setup User account
-   Server Farm account
-   Search Service account
-   Content Access account
-   Web App Pool accounts
-   Service App Pool accounts
-   Cache Super User account
-   Cache Super Reader account

For cost efficiency, this deployment uses DS3-size VMs for all roles as noted in the "Availability set recommendations" section above. 
However, in production workloads, we recommend that organizations use at least the DS13 size for the Search Indexer. 
See [Hardware and software requirements for SharePoint Server 2016][hardware-reqs]. 
To meet the support requirement for disk throughput of 200 MB per second minimum, make sure to [plan
the Search architecture][plan-search]. To manage the process, use the best practices for 
[crawling content][crawling-content] to build a search index.

In addition, store the search component data on a separate storage volume or
partition with high performance. To reduce load and improve throughput,
configure the [object cache user accounts][object-cache], 
which are required in this architecture. Split the Windows Server operating
system files, the SharePoint Server 2016 program files, and diagnostics logs
across three separate [storage volumes][storage-volumes] or partitions with normal performance.

For more information about these recommendations, see 
[Initial deployment administrative and service accounts in SharePoint Server 2016][initial-accounts].

### SQL Server Always On Availability Groups

This architecture uses SQL Server virtual machines because SharePoint Server
2016 can’t use Azure SQL Database. To support high availability in SQL Server,
we recommend [creating Always On Availability Groups][creating-always-on],
which specify a set of databases that fail over together, making them
highly-available and recoverable. In comparison, an Azure availability set
allocates virtual machines to different fault domains. In this reference
architecture, the databases are created during deployment, but you must manually
[enable Always On][enable-always-on] Availability Groups and 
[add the SharePoint databases][add-databases] to an availability group.

We also recommend adding a listener IP address to the cluster, which is the
private IP address of the internal load balancer for the SQL Server virtual
machines.

To promote performance and security, see the [best practices][sql-best-practices] 
for SQL Server in SharePoint Server 2016 farm.

### Majority node server recommendations

We strongly recommend that the majority node server reside on a separate
computer from the replication partners. The server enables the secondary
replication partner server in a high-safety mode session to recognize whether to
initiate an automatic failover. Unlike the two partners, the majority node
server doesn’t serve the database but rather supports automatic failover.

##Scalability considerations

To scale up the existing servers, simply change the VM size. Note that
SharePoint Server 2016 doesn’t support the use of virtual machine scale sets for
auto-scaling.

With the new [MinRoles][minroles] capability in SharePoint Server 2016, you can scale out servers based on the
server's role and also remove servers from a role. When you add servers to a
role, you can specify any of the [single roles or one of the combined roles][sharepoint-roles]. 
But if adding servers to the [Search][search] role, you must also reconfigure the search topology using PowerShell. 
You can also [convert roles][convert-roles] using MinRoles.

Learn more about how to [manage your MinRole farm deployment][manage-minroles].

##Availability considerations

This reference architecture supports high availability, but for even higher
availability, create a separate disaster recovery farm in an Azure paired region[paired-region]. 
Your recovery time objectives (RTOs) and recovery point objectives
(RPOs) will determine the setup requirements. For details, see 
[Choose a disaster recovery strategy for SharePoint 2016][choose-dr].

##Manageability considerations

To operate and maintain servers, server farms, and sites, follow the recommended
practices for [SharePoint operations][sharepoint-operations].
For example, see the best practices for [backup and restore][backup-restore].

The tasks to consider when managing SQL Server in a SharePoint environment may
differ from the ones typically considered for a database application. A best
practice is to fully back up all SQL databases weekly with incremental nightly
backups. Back up transaction logs every 15 minutes. Another practice is to
implement [SQL Server maintenance tasks][sql-maintenance] 
on the databases while disabling the built-in SharePoint ones. For more
information, see [Storage and SQL Server capacity planning and configuration][storage-planning].

##SharePoint hybrid considerations

After the farm has been deployed and configured in Azure IaaS, you’ll have a
fully functioning SharePoint Server 2016 farm that is ready for the supported
[hybrid scenarios][hybrid]. In this case, “hybrid” refers to
deployments that extend SharePoint Server 2016 to Office 365 SharePoint Online.
(By comparison, in an IaaS context, “hybrid” refers to cross-premises virtual
network.) If you have Office Online Server, see [Office Web Apps and Office Online Server supportability in Azure][owa].

The default service applications in this deployment are designed to support
hybrid workloads. All SharePoint Server 2016 and Office 365 hybrid workloads can
be deployed to this farm without changes to the SharePoint infrastructure—with
one exception. The Cloud Hybrid Search Service Application must not be deployed
onto servers hosting an existing search topology. Therefore, one or more
search-role-based virtual machines must be added to the farm to support this
hybrid scenario.

##Security considerations

The domain-level service accounts used to run SharePoint Server 2016 require
Windows Server AD domain controllers for domain-join and authentication
processes. Azure Active Directory Domain Services can’t be used instead. 
To extend the Windows Server AD identity infrastructure already in place in the intranet, 
this architecture uses two Windows Server AD replica domain controllers of an existing on-premises Windows Server AD forest. 
See also [Create an AD DS resource forest in Azure][ad-ds].

In addition, it’s always wise to plan for [security hardening][security-hardening].
Other recommendations include:

-   Add rules to network security groups to isolate subnets and roles.

-   Don’t assign public IP addresses to VMs.

-   For intrusion detection and analysis of payloads, consider using a network
    virtual appliance in front of the front-end web servers instead of an
    internal Azure load balancer.

-   As an option, use IPsec policies for encryption of cleartext traffic between
    servers. If you are also doing subnet isolation, update your network
    security group rules to allow IPsec traffic.

-   Install anti-malware agents for the VMs.

##Community

Communities can answer questions and help you set up a successful deployment.
Consider the following:

-   [Azure Forum][azure-forum]

-   [TechNet Wiki: SharePoint: Community Best Practices][technet-wiki]

-   [SharePoint Microsoft Tech Community][sharepoint-community]

##Deploy infrastructure on Azure

An [Azure Resource Manager][arm] deployment template for this architecture is available on [GitHub][github]. The architecture is based on the [Windows VM workloads][windows-vm-workload] pattern and is deployed in three stages. To deploy the architecture, follow these steps:

1. Click the button below to begin the first stage of deployment.
   [![Deploy to Azure](/azure/guidance/_images/blueprints/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fguidance-compute-single-vm%2Fazuredeploy.json)
2. When the link has opened in Azure portal, enter the following values: 
   
   1. For **Resource group**, select **Create New** and enter `ra-sp2016-network-rg` in the text box. The name is already defined in the parameter file.
   2. In the **Location** box, select a region.
   3. Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.
   4. Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.
   5. Click the **Purchase** button.

3. Check Azure portal** Notifications** for a message that the first stage of the deployment is complete.
4. Click the button below to begin the second stage of deployment.
   [![Deploy to Azure](/azure/guidance/_images/blueprints/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fguidance-compute-single-vm%2Fazuredeploy.json)
5. When the link has opened in the Azure portal, enter the following values:

   1. For **Resource group**, select **Create New** and enter `ra-sp2016-workload-rg` in the text box. The name is already defined in the parameter file.
   2. In the **Location** box, select a region.
   3. Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.
   4. Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.
   5. Click the **Purchase** button.

6. Check Azure portal** Notifications** for a message that the second stage of the deployment is complete.
7. Click the button below to begin the third stage of deployment.
   [![Deploy to Azure](/azure/guidance/_images/blueprints/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fguidance-compute-single-vm%2Fazuredeploy.json)
8. When the link has opened in the Azure portal, enter the following values:

   1. For **Resource group**, select **Create New** and enter `ra-sp2016-network-rg` in the text box. The name is already defined in the parameter file.
   2. In the **Location** box, select a region.
   3. Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.
   4. Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.
   5. Click the **Purchase** button.

9. Check Azure portal** Notifications** for a message that the third stage of the deployment is complete.
10. In the parameter files, we strongly recommend changing the hard-coded administrator username and password on all VMs.To do this:

  1. For each VM in the Azure portal, click **Reset password** in the **Support + troubleshooting** blade.
  2. In the **Mode** box, select **Reset password**, then select a new **User name** and **Password**. 
  3. Click the **Update** button to save the new user name and password.




[add-databases]: https://technet.microsoft.com/library/mt793548(v=office.16).aspx
[ad-ds]: /azure/architecture/reference-architectures/identity/adds-forest
[always-on]: https://technet.microsoft.com/en-us/library/mt793548(v=office.16).aspx
[arm]: /azure/azure-resource-manager/resource-group-overview
[availability-set]: /azure/virtual-machines/windows/manage-availability#configure-each-application-tier-into-separate-availability-sets
[availability-group]: https://msdn.microsoft.com/library/hh510230.aspx
[azure-forum]: https://azure.microsoft.com/en-us/support/forums/
[backup-restore]: https://technet.microsoft.com/en-us/library/gg266384(v=office.16).aspx
[choose-dr]: https://technet.microsoft.com/en-us/library/ff628971(v=office.16).aspx
[crawling-content]: https://technet.microsoft.com/en-us/library/dn535606(v=office.16).aspx
[creating-always-on]: https://technet.microsoft.com/en-us/library/mt793548(v=office.16).aspx
[convert-roles]: https://technet.microsoft.com/en-us/library/mt790700(v=office.16).aspx
[dhcp]: https://technet.microsoft.com/en-us/library/cc755277(v=ws.11).aspx
[enable-always-on]: https://technet.microsoft.com/en-us/library/mt793550(v=office.16).aspx
[filter-network]: /azure/virtual-network/virtual-networks-ns
[gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[gateway-subnet-ra]: /azure/architecture/reference-architectures/hybrid-networking/vpn
[github]: http://www.github.com
[hardware-reqs]: https://technet.microsoft.com/en-us/library/cc262485(v=office.16).aspx
[hybrid]: https://aka.ms/sphybrid
[hybrid-net-vpn]: /azure/architecture/reference-architectures/hybrid-networking/vpn
[ilb]: /azure/load-balancer/load-balancer-internal-overview
[initial-accounts]: https://technet.microsoft.com/EN-US/library/ee662513(v=office.16).aspx
[legal-sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[majority-node]: https://technet.microsoft.com/en-us/library/cc731739(v=ws.11).aspx
[manage-availability]: /azure/virtual-machines/windows/manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[manage-minroles]: https://technet.microsoft.com/en-us/library/mt743705(v=office.16).aspx
[minroles]: https://technet.microsoft.com/en-us/library/mt346114(v=office.16).aspx
[nsg]: /azure/virtual-network/virtual-networks-nsg
[n-tier]: /azure/architecture/reference-architectures/virtual-machines-windows/n-tier
[object-cache]: https://technet.microsoft.com/en-us/library/ff758656(v=office.16).aspx
[owa]: https://support.microsoft.com/en-us/help/3199955/office-web-apps-and-office-online-server-supportability-in-azure
[paired-region]: /azure/best-practices-availability-paired-regions
[plan-accounts]: https://technet.microsoft.com/en-us/library/cc263445.aspx
[plan-search]: https://technet.microsoft.com/en-us/library/dn342836.aspx#BKMK_ChooseHWResources
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[roles]: https://technet.microsoft.com/en-us/library/mt743704(v=office.16).aspx
[technet-community]: https://social.technet.microsoft.com/wiki/contents/articles/12438.sharepoint-community-best-practices.aspx
[search]: https://technet.microsoft.com/library/jj862354(v=office.16).aspx
[security-hardening]: https://technet.microsoft.com/en-us/library/cc262849(v=office.16).aspx
[service-accounts]: https://technet.microsoft.com/en-us/library/cc263445.aspx 
[sharepoint-community]: https://techcommunity.microsoft.com/t5/SharePoint/bd-p/SharePoint_General
[sharepoint-operations]: https://technet.microsoft.com/en-us/library/cc262289(v=office.16).aspx
[sharepoint-roles]: https://technet.microsoft.com/en-us/library/mt667910(v=office.16).aspx
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[sql-best-practices]: https://technet.microsoft.com/en-us/library/hh292622(v=office.16).aspx
[sql-maintenance]: https://technet.microsoft.com/en-us/library/hh292622(v=office.16).aspx
[storage-planning]: https://technet.microsoft.com/en-us/library/cc298801(v=office.16).aspx
[storage-volumes]: https://technet.microsoft.com/en-us/library/cc263445.aspx
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[technet-wiki]: https://social.technet.microsoft.com/wiki/contents/articles/12438.sharepoint-community-best-practices.aspx
[tempdb]: https://technet.microsoft.com/en-us/library/ms175527(v=sql.105).aspx
[vm-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[vpn]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[windows-vm-workload]: /azure/architecture/reference-architectures/virtual-machines-windows/
[0]: ./images/Sharepoint-2016-RA-diagram.png "Highly available SharePoint Server 2016 farm in Azure."