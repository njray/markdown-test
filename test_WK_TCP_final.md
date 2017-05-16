---
title: "Service Fabric Customer Profile: Wolters Kluwer CCH"
---

Authored by *Jason Hamilton* and *Bill Hilke* from Wolters Kluwer, in
conjunction with *Christoph Schittko, Bart Robertson,* and *Ed Price* from
Microsoft.

This article is part of a series on customers who’ve worked closely with
Microsoft on Service Fabric over the last year. We look at why they chose
Service Fabric, and we take a closer look at the design of their application.
For more profiles, see our [Service Fabric series](http://aka.ms/TCP).

In this installment, we profile Wolters Kluwer CCH, their CCH Axcess tax
preparation application running on Azure, and how they designed the architecture
using Service Fabric and Azure Application Gateway.

![http://wolterskluwer.com/images/logo/wolters-kluwer-logo-dark-simple-full.png](media/7ecd38a849364e3f2a2398fe79d8b6a7.png)

Wolters Kluwer is a leading global provider of tax, accounting, and audit
information, solutions, and services. The Tax and Accounting division delivers
solutions that help professionals worldwide navigate and comply with complex
regulations and requirements, effectively manage their practices, and strengthen
relationships with their clients.

This all-encompassing approach is reflected in their CCH Axcess Suite, a hybrid
cloud-based tax preparation, compliance, and workflow solution. Tax and
accounting firms run their business on this suite, using it for everything from
interacting with their customers to filing a return. As a mature platform, CCH
Axcess was architected before newer design trends such as microservices were
available. The Wolters Kluwer software division started a project to modernize
their legacy platform, hoping to find a cloud solution that would deliver high
availability and scalability without requiring a complete rewrite of their
extensive code base.

>   We needed a distributed clustering technology that would deliver a
>   microservice transformation at massive scale, as in multimillions of
>   concurrent jobs. The high-availability guarantees from Service Fabric gave
>   us our answer.

>   —Jason Hamilton, Chief Architect, Software Development

Wanted: one highly available, highly scalable, and flexible tax preparation engine
==================================================================================

The company’s motto, “When you have to be right,” implies some of the challenges
they face. Wolters Kluwer’s tax solution is used by large agencies to prepare
the world’s most complex United States tax returns. CCH Axcess supports tax
rules for United States federal and state returns—rules that must persist for
seven years and can change within a year and across years. The CCH Axcess
platform needs to handle these dynamic and sometimes late-breaking changes
seamlessly so customers aren’t impacted. As a result, the Tax and Accounting
software team uses approximately 7,000 different .NET libraries to define the
tax rules for each tax year.

For Wolters Kluwer’s customers, inaccuracies or delays in filing tax returns can
lead to severe monetary penalties. The filing deadlines in April and October
mean huge spikes in the tax preparation workload with peak data transfer rates
of 7 TB per day and 1,500 CPU hours per day. The tax preparation system must
remain continuously available during filing deadlines, and its calculation times
can’t be affected by an increase of load.

The earlier generation of the CCH Axcess platform used a monolithic calculation
service that was built on a standard *n*-tier architecture. It ran on a
traditional infrastructure that required Wolters Kluwer to pay for peak loads 12
months of the year. The software development team wanted a more cost-effective
approach but couldn’t simply retire their legacy platform. They also wanted a
solution that would make it easy for their internal developers and partners to
support both the legacy and new systems at the compute level.

A hackathon is born
===================

The Tax and Accounting software team started their search for a new architecture
by defining their guiding principles:

-   Use Service Fabric capabilities where possible.

-   Keep services in the cluster and limit external dependencies.

-   Make it as simple as possible.

-   Protect the data.

-   Support automated testing.

-   Support high availability, monitoring, and billing.

-   Prevent noisy neighbors by isolating tenants.

The next step was a hackathon, an event the team sponsors regularly to generate
innovation and best practices. During the hackathon, the developers coined the
term “tax calculation unit” as a key metric that could completely transform
their business model. In their legacy architecture, one customer might submit a
few very large calculations that would affect the other calculations running
simultaneously. This issue, known as the noisy neighbor problem, could be solved
if they had customer-level control at the level of tax calculations, because
they could then scale the system to deliver customer-specific service levels.

>   Microsoft’s deep commitment to their customers enabled a hands-on design and
>   validation cycle with embedded Microsoft architects and Azure service
>   product team support. This enabled us to associate tangible business value
>   to our technology choices as we made them.

>   —Jason Hamilton, Chief Architect, Software Development

The tax calculation unit, or TCU, reminded the Wolters Kluwer team of nearby
Texas Christian University, whose mascot is the horned frog—and so “Frog” became
the code-name for their new tax calculation service. Given the transaction rates
during peak times, they began investigating microservices architectures and
distributed clustering technologies. Frogs became their vision for a
self-contained, generic job system that could support multimillions of
concurrent jobs and handle multiple-terabyte bursts of data per day.

Wolters Kluwer’s microservices approach to tax preparation
==========================================================

The new system is built on a microservice architecture running in Azure with
services managed by Service Fabric. The solution is cost-effective—Wolters
Kluwer can manage compute capacity on demand by quickly provisioning additional
capacity during the April and October spikes, and scale back during non-peak
months. For customers, CCH Axcess now provides better overall performance,
especially during the prime tax seasons.

For high availability, the workload is replicated across Azure datacenters using
Azure Paired Regions. ExpressRoute extends their on-premises networks into the
Azure cloud over a dedicated, private connection. Azure Application Gateway is
used to securely expose REST APIs to Wolters Kluwer development teams,
customers, and system integrators. The solution uses Azure Event Hubs, a
messaging service that can ingest millions of events per second and make them
available for storage and analysis.

![](media/dda16768255c80c19b4c64b5378f2a9d.png)

Figure 1. Architecture of the CCH Axcess job system in Azure.

Because CCH Axcess is a large, mature software service, Wolters Kluwer adopted a
hybrid cloud strategy for migration. With this architecture, they can migrate
critical services to Azure while continuing to host parts of CCH Axcess in a
datacenter on premises. With each software release, they can move more services
to Azure, using the support in Express Route to create a secure and highly
available hybrid cloud. The application delivery controller is Application
Gateway, an Azure managed service with sophisticated service routing
capabilities. CCH Axcess uses Application Gateway in an active-active, fan-out
implementation that further reduces costs and increases scale while providing a
geo-redundant, highly available system.

Service Fabric benefits
=======================

To move to a microservices-based architecture, Wolters Kluwer needed to break up
their sizeable software system into smaller, independent services. They knew
that architecting fine-grained microservice applications would enable continuous
integration and continuous development (CI/CD) practices and accelerate delivery
of new functions into the application. While flexible, distributed deployments
are also complex to manage manually. That’s where Service Fabric fits in.

By using Service Fabric, the Tax and Accounting software team could focus on
implementing business logic, knowing that the services would be scalable,
reliable, and manageable. Service Fabric resolved a number of the team’s
computational, deployment, and management concerns as well by providing the
following:

-   **Stateful services**. Complex tax calculations can run for hours or even
    days. Stateful services preserve their state, so if something goes wrong,
    complete restarts aren’t needed. Even when long-running calculations are in
    flight, stateful services enable the Service Fabric cluster to manage
    capacity by moving replicas.

-   **Resource optimization**. Services are created dynamically to ensure
    optimal allocation of compute resources and isolation of tenants. The team
    also created custom metrics in Service Fabric that help them optimize
    cluster use. To reduce the effect of noisy neighbors, they use prioritized
    queues in Service Fabric, which enable them to throttle services at the
    customer level.

-   **Parallel deployments**. The team needed strong versioning and side-by-side
    support so they could respond to late-breaking changes in the tax code by
    deploying different versions of the same service simultaneously. Service
    Fabric can execute multiple versions of the same application in a single
    environment, enabling the software team to update code without downtime or
    interrupting long-running calculations.

-   **Application availability**. Applications remain available throughout an
    update, because Service Fabric performs rolling upgrades in stages. Even
    during peak use, pushing bug fixes requires no downtime. The team also uses
    a Blue-Green deployment strategy in the same cluster, and placement
    constraints in Service Fabric let them specify where deployments take place.
    If Blue is live, then pre-deployment testing takes place on Green. Placement
    constraints also help them optimize service placement.

-   **Reliable data**. For highly available, durable data storage that is
    tenant-isolated, the software team uses the Reliable Collections API in
    Service Fabric, a set of classes that automatically manage the replicated
    and local state of applications. The team’s custom collections provide
    encryption at rest and in motion for stateful service data. Tenant isolation
    provides an extra security layer and also helps prevent the noisy neighbor
    issues in traditional single-point-of-storage solutions.

    >   Having Service Fabric enable so many of the essential requirements of
    >   functionality within the platform enabled us to think about our own
    >   intellectual property and value-add in terms of making the customer
    >   experience as scalable and resilient as possible. It enabled us to go
    >   further quicker by using their building blocks.

    >   —Bill Hilke, Lead Architect, Software Development

Service Fabric architecture
===========================

Service Fabric handles the heavy lifting of scheduling and running thousands of
service instances for CCH Axcess. Application Gateway handles security and
tenant isolation. Together, they provide an easy-to-use, developer-friendly
solution.

Because the data and compute tiers are closer together, latency is reduced and
throughput is independent of other tenants and services. Now Wolters Kluwer can
deliver more consistent performance at scale for a lower cost than their
previous systems design.

![](media/d887c4676d7d78adffdf65b18ad23bdf.png)

Figure . Service discovery and routing in the Service Fabric cluster.

Service Fabric reverse proxy
----------------------------

CCH Axcess relies heavily on the reverse proxy in Service Fabric for
communication to microservices from inside and outside the cluster.
Microservices, especially stateful services, typically run on a subset of
virtual machines in the cluster. They can move from one virtual machine to
another for various reasons, so the endpoints for microservices can change
dynamically. The reverse proxy in Service Fabric runs on all the nodes in the
cluster and performs the entire service resolution process, then forwards
requests to the service endpoint. Wolters Kluwer can use standard HTTPS
communication libraries to talk to a target service and rely on Service Fabric
for service resolution.

Container orchestration
-----------------------

One of the goals of the team was to create a strategy for a multi-variable
digital transformation, focusing on the journey from a monolithic datacenter to
a hybrid cloud model. They hoped a public cloud microservice architecture could
support large distributed systems. Given the size of Axcess, an incremental
approach was necessary.

As part of their technical design, the team realized that they needed to focus
on tenant isolation at the service layer. So CCH Axcess takes advantage of
Service Fabric as a container orchestration engine. The development team can
choose the best microservice implementation based on their business
requirements. Many services will be migrated to a container while key services
will be native to Service Fabric and take full advantage of stateful services
and other platform features.

>   Another reason why Service Fabric was compelling to us was it enabled
>   multiple scenarios, from developing self-contained systems to enabling a
>   Graph API development standard with reverse proxy, to container
>   orchestration. This standardization lowered total cost of ownership in terms
>   of support and maintenance.

>   —Jason Hamilton, Chief Architect, Software Development

Hybrid storage
--------------

For the storage tier in Azure, Wolters Kluwer created an Azure availability set
for pairs of virtual machines running Panzura, an Azure partner cloud storage
solution. This makes the virtual machines eligible for a higher service-level
agreement (SLA). Panzura uses Express Route to route changes to the NetApp
storage appliances in the on-premises datacenter.

This hybrid approach makes it simpler for Wolters Kluwer to move parts of CCH
Axcess to Azure incrementally and avoid a complex lift-and-shift operation in a
very tight timeframe. As more CCH Axcess services move to Azure, the team will
be able to easily change the storage subsystem and take advantage of Azure
Storage, further reducing complexity and operating costs.

Conclusion
==========

Filing tax returns is a seasonal task, and the elasticity of Azure gave Wolters
Kluwer the capacity they needed to accommodate peak customer usage while
reducing their overall cost of ownership. Service Fabric gave them a path
forward into the public cloud through a flexible, highly available
microservices-based architecture. As they move more services to the cloud,
Service Fabric is changing how the team thinks about development and opening the
door to new customer services and business models.
