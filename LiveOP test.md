---
title: "Service Fabric Customer Profile: LiveOP"
---

Authored by *Kay Slagman* from *LiveOP*, in conjunction with (*individual
names*) from Microsoft.

This is part of a series on customers who’ve worked closely with Microsoft on
Service Fabric over the last year. We look at why they chose Service Fabric, and
we take a closer look at the design of their application. You can read all the
profiles in this series here.

In this installment, we profile *LiveOP*, their *LiveOP X* application running
on Azure, and how they designed the architecture using *Service Fabric*.

*[Insert company or application logo]*

LiveOP is Dutch startup company with an ambitious focus on revolutionizing the
way first responders collaborate and improving their ability to make decisions
in life-or-death situations. As part of the Microsoft BizSpark+ program, LiveOP
has been able to accelerate its growth using software and services provided by
Microsoft.

Using the Azure platform, LiveOP can build services with a level of quality,
scalability, and versatility that was not possible before. The flagship product,
LiveOP X, is a mobile application that runs on Azure and integrates Service
Fabric to provide real-time access to critical information during emergencies.

>   “The Fire Department of the Amsterdam-Amstelland region is dedicated to
>   being a leading force in innovating public safety with smart technologies.
>   We’ve chosen the LiveOP X platform because of its unchallenged performance
>   in innovation and ease-of-use.”

>   —Barry van ‘t Padje, Fire Department of Amsterdam-Amstelland, The
>   Netherlands

Real-time challenge
===================

Managing emergency response processes is an incredibly complex task in which
multiple disciplines of highly trained people collaborate. In an emergency, one
of the great challenge is how to get the right information to the right person
at the right time. Every organization has its own tasks, priorities, and
knowledge base, from the fire and police departments to the health workers and
ambulance services and even local governments.

Given the high stakes, the public safety market is traditionally conservative in
adapting to new technologies on a large scale, making it challenging for
solution innovators. The team at LiveOps wanted to bring the advantages of the
cloud—scale, speed, access—to first-responders, but the solution had to strike
the right balance between security, reliability, and privacy.

The real-time solution
----------------------

The LiveOP X platform offers a solution to this problem. As a multi-tenant
cloud-based platform, LiveOP X is designed to integrate modern technologies with
public safety services. Emergency personnel access these services through a
mobile client that enables them to collaborate from any location, show
personalized content, and connect with external data services such as
IoT-devices and video streams.

For example, LiveOP X supports first responders with an intelligent selection of
relevant data from external data sources. Using their mobile device, emergency
personnel can filter any available informationto retrieve the content they need
most. Whether they want incident updates from the dispatch center, information
on dangerous substances in the surrounding area, floor maps, live camera video
feeds, or the real-time positions of their colleagues, LiveOP X provides the
information immediately on their mobile device.

The move to the cloud
---------------------

In the public safety market, software developers typically prefer to keep their
data in private networks for security reasons. This approach posed a problem for
the team designing the LiveOP X platform, who wanted to take advantage of modern
software development concepts such as continuous delivery, automated testing,
and remote monitoring. If they chose an architecture than ran only on premises,
they would limit the quality and possibilities for their platform. If they
adopted the cloud, they might lead the way technologically but lose their
market.

The decision to adopt the clout felt radical and risky, but how else could their
platform deliver the type of collaboration they wanted to support among
organizations? To enable data sharing across external systems and offer a
multi-tenant solution, the cloud-only solution was the way to go. As a startup
company, LiveOP could use an edge in a market that relies heavily on stability,
safety, and security. Using Azure gave them the assurance of a Microsoft
infrastructure that met high compliance and security standards.

By investing in the Azure ecosystem, LiveOP also benefited from several other
services:

-   Azure Active Directory for secure identity management and integration with
    existing Office 365 environments

-   Service Fabric for the implementation of the LiveOP X platform services

-   Azure Storage for securely storing large amounts of data

-   Azure Notification Hubs for implementing push messaging to tablet clients

-   Azure Service Bus for connecting multiple services through a central message
    provider

-   Azure Key Vault for securely storing certificates and keys

[TODO:DIAGRAM]

From drawing board to… drawing board
====================================

“Moments before we finished the main architecture of the LiveOP X platform,
Microsoft announced the availability of Service Fabric,” Kay Slagman explained.
One of the founders of LiveOP, Kay was also the lead architect of LiveOP X, and
Service Fabric wasn’t part of their original plans. “We were immediately
impressed by the potential and made a radical decision of going back to the
drawing board. It has proven to be the right choice, and Service Fabric has
played a significant role in us raising our ambitions. Without it, the platform
could not have become what it is today.”

Designing and creating the Service Fabric-based platform brought a whole new
challenge for the LiveOP X development team. Not only was Service Fabric a new
platform, it also required a completely different way of thinking about their
solution’s design to an architecture based on microservices. Furthermore, the
Reliable Services and Reliable Collections capabilities in Service Fabric
shifted their programming paradigm completely.

“We must have completely restarted on the architecture at least three times
before we really found that we nailed the design and felt confident,” Kay
Slagman said. “If I could give one tip to anyone starting with Service Fabric,
it Is to forget everything you know, and start with a clear mind. Then build it
up from there, but keep it simple. We went from excited to frustrated but
hopeful to completely thrilled. It was an amazing ride and the end result
couldn’t have made us prouder.”

[TODO:DIAGRAMS]

Advantages of Service Fabric
============================

The LiveOP X development team was enthusiastic about the general concept of
Service Fabric, as its agility, high availability and easy manageability
promised to offer a good foundation for the LiveOP X platform it envisioned to
create.

Their vision of a microservices-based architecture fit well with Service Fabric,
which met many critical requirements of this new platform. The scalability meant
LiveOP could start small and increase the scale as the product grows. With the
support for continuous integration, the efficiency of the deployment process
grows immensely and by offering rolling updates and automatic rollback, LiveOP
can greatly reduce the risk of downtime while upgrading the platform. This means
the LiveOP team can focus more energy on their own expertise, developing
high-quality software and services for the public safety industry, and less on
basic infrastructure maintenance.

The following advantages stood out from the beginning:

-   Maintainability using microservices  
    The team realized from the start that the complexity of the platform,
    combined with the high requirements on quality and stability, could create a
    challenge in the future to keep it up-to- date In the long term. The
    microservices architecture offers a great advantage in preserving the
    manageability in the long term.

-   Scalability  
    LiveOP has received a lot of interest from the market in its services and
    has set ambitious goals on expanding internationally. The scalability of the
    platform creates the opportunity to start relatively small and expand as the
    success of the product grows, without creating exponentially stress on
    infrastructure maintenance.

-   Focus on workload vs infrastructure  
    LiveOP is first and foremost a software company, not an infrastructure
    provider. The entire workflow of developing for Service Fabric integrates
    development and deployment in a way that fits right into the workflow of the
    development team. Being able to trust on a platform like Service Fabric, and
    Azure in general, to deliver the highest quality infrastructure enables
    LiveOP to focus on their specific strengths as a company.

-   Reliability  
    Reliability is a major concern for any SaaS-provider, but in this case the
    risks of unavailability could result in devastating scenarios for LiveOP’s
    end customers. With Service Fabric, LiveOP can minimize this risks with
    out-of-the-box functionality of the platform.

Integrating Azure Services
==========================

[TODO:ARCHITECTURE]

Microservices architecture
==========================

Upon creating the architecture, the team determined a few main goals:

-   Each tenant has to have a separate pipeline for transferring data,
    considering all data is highly sensitive

-   Services should work as independently as possible. In separating concerns,
    the team has to take data-throughput and priority of the information in
    consideration

-   Critical issues must be updated quickly, so a continuous integration
    pipeline is essential

-   Reliable collections should be the main storage solution to support high
    speed and scalability

Typical usage scenarios
-----------------------

To get an idea of typical usage of the services, consider the following
scenarios:

-   When an incident is called in, the user of LiveOP needs to immediately get
    the latest data on the incident from the dispatch center. The service will
    then connect to external data systems to retrieve other relevant information
    that is crucial to this incident, e.g. information on nearby buildings,
    required procedures, dangerous materials and fluids, notes on the location
    that have been stored on previous incidents and warnings about dangerous
    situations. To give an example of the latter, if a fire rescue team needs to
    storm a building that has been flagged as a possible drug house, the fire
    department is in great danger. LiveOP X can immediately alarm the teams on
    their route, so they can take the required measures to prevent a disaster.

-   As soon as teams are dispatched, LiveOP X tracks their location based on
    GPS-data. The application tracks and displays real-time location information
    of all assigned units, requiring the service to process this information as
    fast as possible.

-   Using geographical views, any information that can be linked to a
    geolocation, e.g. buildings, photos, impact circles etc. need to be drawn on
    the map for immediate analysis. Officers in command will draw additional
    instructions on the map and will share these with the other users.

-   Teams on the incident location are able to take pictures and share these
    with all other users immediately.

>   “In my opinion, Service Fabric is the greatest example of what can be
>   achieved by fully embracing cloud technologies. It a game-changer.”

>   Kay Slagman, Director and Lead Architect \@ LiveOP

Using stateful services
-----------------------

The team created a number of stateful services to support these use cases, most
notable:

-   IncidentService, IncidentActorService, IncidentActor  
    The incident service is the most crucial service and is responsible for
    managing incident data.

-   UnitService  
    The unit service is responsible for managing unit (vehicle) data and
    tracking the geo-locations of the unit positions.

-   DocumentService  
    The document service handles document management processes to store
    procedures and other relevant data. For actual storage of the blobs it
    utilizes the Azure Blob Storage.

-   AuthorizationService  
    The Authorization service is responsible for managing authorizion
    configuration and tasks for the platform.

-   NotificationService  
    The NotificationService handles notification tasks, such as managing push
    messages.

-   WebService (stateless)  
    The WebService functions as the gateway to the external application, hosting
    WebAPI controllers to communicate with the tablet application and the
    management web application.

IncidentService
---------------

The IncidentService is responsible for managing the actual incident data and is
therefore the most crucial service of the platform. It receives incident data
and updates from the dispatch center and it provides client applications with
the latest information on all active incidents. The great challenge in the
design was to keep it highly performant, while being able to connect to a large
number of external data sources for real-time additional data.

The development team chose to use the Actor model for this situation. It enabled
the incidents to come alive and become a lot more flexible. As soon as an
incident starts, the IncidentService creates an actor that is responsible for
managing all tasks and data processing related to this incident. It checks the
list of external data sources and starts requesting and processing the
information.

During processing, the actor analyzes the information to leave out anything that
Is not deemed necessary. The system is carefully designed to prevent information
overload in critical situations, as that might offer a threat to the performance
of the first responder in the field. As soon as one of such processes is
finished, it stores the result in the incident data.

When a user requests the incident data, the actor simply returns all the
information it has at that specific moment in time. Having the data available in
memory means the service is not required to wait for external queries to execute
and finish, resulting in an extremely quick response.

UnitService
-----------

The unit service is specifically designed to track the availability and status
information of vehicles and official functions, generally referred to as units.
Each unit can be assigned to an incident. Once a unit has been assigned, the
unit shares the information about its status and position. Information about all
units is shared with all other users connected to the incident.

The UnitService utilizes Reliable Dictionaries to store this data for optimized
access and data throughput. With a large number of small read/write operations,
this is an area where the speed advantage of Reliable Dictionaries really pays
off.

DocumentService
---------------

For the management of documents, the DocumentService combines Reliable
Dictionaries and Azure Storage. Metadata and related information is stored in
the DocumentService for fast access and processing of document data. The
DocumentService offers basic document management functionality and uses Azure
Blob Storage to store the actual byte data.

AuthorizationService
--------------------

Authentication and authorization is supported by the AuthorizationService. For
the actual authentication, the platform uses Azure Active Directory as its main
identity provider. The LiveOP X platform offers an extremely flexible system to
determine which (types of) user can access specific data. The
AuthorizationService handles this management of roles, permissions and
additional user data.

This allows administrators to create their own roles with finely grained access
to data and functionality. Administrators can determine which functionality is
offered in the client/tablet application, which color themes should be applied
and they can configure conditional authorization based on incident properties.

NotificationService
-------------------

Notifications are an important part of the platform as they assist in bringing
important information to users as soon as possible. The development team
designed a service to track important events and to streamline the notification
processes across the platform. The NotificationService connects with Azure
Notification Hub to send push messages to client applications.

WebService
----------

The WebService is a stateless service that hosts the OWIN self-hosted WebAPI.
The individual controllers provide the required functionality to the tablet
application and web application to receive and request data using secure
connections.

Reliable collections vs. external databases
-------------------------------------------

One of the big discussion points during the architectural design was the choice
between the use of databases vs. Reliable Collections. In the first designs the
team took a hybrid approach, initiated by concerns on reliability, flexibility
and backup/restore processes. During development, the team shifted more and more
data sources to Reliable Collections up until It chose to completely remove the
database as the main storage provider. The speed and flexibility of Reliable
Collections proved to offer a great advantage over the traditional database
architecture and it decreased costs of data maintenance as a whole.

Using automated backup and restore processes the team could realize a scenario
where it can now use its own provisioning system to automatically recreate an
entire cluster, including the latest data, in a worst-case scenario.

Summary
=======

Using Azure, and Service Fabric in particular, LiveOP was able to create a
state-of-the-art modern platform that supports emergency services daily in the
most demanding situations. It helped the company to create a platform with a
level of reliability, scalability and performance that used to cost a magnitude
of what is available now. Without Service Fabric and Azure, the company would
not be able to have a platform this advanced running today.

The company has seen exponential growth while decreasing the costs per customer
thanks to choosing Azure technologies as their platform of choice. With Service
Fabric as the core of their new platform, LiveOP is confident to scale up in the
coming future with a platform that is more than ready to keep up with this
growth.
