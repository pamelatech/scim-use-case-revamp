---
stand_alone: true
docname: draft-correia-scim-use-cases-00
ipr: trust200902
submissiontype: IETF
keyword: [Internet-Draft, SCIM]
workgroup: SCIM

cat: info
title: 'System for Cross-domain Identity Management: Definitions, Overview, Concepts, and Requirements'
abbrev: 'SCIM Use Cases'
Lang: en
author:
- name: Paulo Jorge Correia
  org: Cisco Systems
  email: paucorre@cisco.com

- name: Pamela Dingle
  org: Microsoft Corporation
  email: 'pamela.dingle@microsoft.com'

normative:
  RFC2119:

informative:
  RFC7643:
  RFC7644:
  RFC9110:
  RFC9112:
  RFC8417:
  Device Schema Extensions to the SCIM model:
     target: https://datatracker.ietf.org/doc/draft-shahzad-scim-device-model
     title: Device Schema Extensions to the SCIM model
     author:
     - name: M. Shahzad
     - name: H. Iqbal
     - name: E. Lear
     date: 2023-07 
  SCIM Profile for Security Event Tokens:
     target: https://datatracker.ietf.org/doc/draft-ietf-scim-events
     title: SCIM Profile for Security Event Tokens
     author:
     - name: P. Hunt
     - name: N. Cam-Winget
     - name: M. Kiser
     date: 2023-07 

--- abstract

This document provides definitions, overview and selected use cases of the System for Cross-domain Identity Management (SCIM).  It lays out the system's concepts, models, and flows, and it includes use cases, and implementation considerations.

--- middle

# Introduction
The System for Cross-domain Identity Management (SCIM) family of specifications [RFC7643] and [RFC7644] is designed to manage resources used in the practice of identity management that need to be communicated across internet domains and services, with users and groups as the default resources supported (and an extensibility model additional resource definition). 
The specifications have two primary goals: 
 1. A common representation of a resource object and its attributes, and 
 1. Standardized patterns for how those resources can be operated on, including "CRUD" operations that create, read, update or delete resource objects and more advanced goals such as search filters, synchronization of large resource populations, etc. These goals are codified as a data model in [RFC7643] defining resources, attributes and default schema, as well as a protocol definition built on HTTP in [RFC7644]. By standardizing the data model and protocol for resource management, entire ecosystems can achieve better interoperability, security, and scalability.

This document provides definitions, overview, concepts, flows, and use cases implementers may need to understand the design and applicability of the SCIM schema [RFC7643] and SCIM protocol [RFC7644]. Unlike the practice of some protocols like Application Bridging for Federated Access Beyond web (ABFAB) and SAML2 WebSSO, SCIM provides provisioning and de-provisioning of resources in a separate context from authentication. While SCIM is a protocol that standardizes movement of data only between two parties in a HTTP client-server model, implementation patterns are discussed in this document that use concepts beyond the core schema and protocol, but that are needed to understand how SCIM actions can fit into greater architectures.


# Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119] when they appear in ALL CAPS. These words may also appear in this document in lowercase as plain English words, absent their normative meanings. Here is a list of acronyms and abbreviations used in this document:

* CRUD: Create, Read, Update, Delete
* ERC: External Resource Creator 
* IaaS: Infrastructure as a Service
* IDaaS: Identity as a Service
* IdM: Identity Manager
* JIT: Just In Time
* RC: Resource Creator
* RU: Resource Updater
* RM: Resource Manager 
* RS: Resource Subscriber 
* RO: Resource Object 
* RA: Resource Attribute 
* SaaS: Software as a Service
* SAML: Security Assertion Markup Language
* SCIM: System for Cross-domain Identity Management
* SET: Security Event Token
* SSO: Single Sign-On


# SCIM Components and Architecture
SCIM architecture is a client-server model centered on a normative concept of a "resource". Resources have types (such as a user or a group) and each unique instance of a resource type is represented by a JSON object, actively accessed via a standardized REST API.  Each resource object can be managed individually or managed in bulk using actions that by default are specified in [RFC9110] (HTTP GET, PUT, POST etc), but that may expand to concepts in extension documents, for example security event tokens (SETs). This model therefore enables organizations to represent information about user populations and the groups that these user populations are part of using the core specifications and to extend to other important resources using extension drafts in the same family with the high level concepts of performing SCIM actions on resource objects. SCIM actions result in resource object and associated data "moving" between the client and server, as clients actively push and pull information that reflects change over time. This communication of data enables systems within domains and across domains to operate on the freshest possible version of object state.

~~~
+---------+                       +--------+
|  SCIM   |                       |        | 
| Server  |                       |  SCIM  | 
|         | <--- SCIM Action ---  | Client |
| /Users  |                       |        |
| /Groups |                       |        |
+---------+                       +--------+
~~~

 
The intent of the SCIM specification is to reduce the cost and complexity of resource management operations by providing a common schemas and extension model, as well as binding documents to provide patterns for exchanging this schema using standard protocols. In essence, make it fast, cheap, and easy to move resources in to, out of, and around the applications.  
The SCIM scenarios are overviews of user stories designed to help clarify the intended scope of the SCIM effort.  

## Implementation Concepts
To understand the use cases we need to understand 5 different concepts of the SCIM protocol (Data Models, Protocols Roles, Orchestrator Roles, Triggers, Actions).

### Data Models
SCIM defines two types of data entities: Resources and Attributes.

#### Resource Object (RO)
A JSON object representing a user, group (or extension object like devices) to used by the CRUD operations through the SCIM protocol. The Resource Object contains attributes defined by schemas such as those defined in [RFC7643] and can be implemented via the endpoints and parameters defined in [RFC7644].  

#### Resource Attribute (RA)
A named element of a Resource Object (RO). Attributes are defined in section 2 of [RFC7643] and include characteristics like cardinality (single or multiple values), data types (string, boolean, binary etc) and characteristics (required, unique etc). 

### Protocol Roles
SCIM is based in the HTTP protocol, HTTP client and server roles are defined in [RFC9110] and [RFC9112]. Any SCIM interaction requires one participant to be a SCIM server and the other to be a SCIM client. 

#### SCIM Server (also known as a SCIM Service Provider)
An HTTP web application that provides identity information via the SCIM protocol.  
A SCIM Server is a RESTful API endpoint offering access to a data model that can be used to push or pull data between two parties. SCIM servers have additional responsibilities such as API Security, managing client identifiers & keys as well as performance management such as API throttling.  

#### SCIM Client
A website or application that uses the SCIM protocol to manage identity data maintained by the service provider.  The client can initiates SCIM HTTP requests to a target SCIM Server. A SCIM Client is active software that can push or pull data between two parties.   

### Orchestrator Roles
Orchestrators are the operating parties that take part in a SCIM protocol exchange and ensure data is moving in the correct flows.
An entity can have one or more orchestrators roles, depending on the overall architecture.  

#### Resource Creator (RC)
An entity responsible for creating the Resource Object (RO). Typically we can see this role in HR or Resource Management (RM) applications that are responsible for creating resources and its attributes.  

#### Resource Updater (RU)
An entity responsible for updating specific Resource Attributes (RA) of a Resource Object (RO) or the RO itself. Typically, this role is used in conjunction with other SCIM roles that allow this SCIM entity to manage specific Resource Attribute (RA) and/or Resource Objects (RO).  

#### Resource Manager (RM)
An entity that aggregates or transforms Resource Objects (RO) from resource creators/updaters (RC/RU) and make them available for Resource Subscribers (RS) using multiple SCIM interactions, an example of this role could be an Identity-as-a-Service (IDaaS) cloud service.  

#### Resource Subscriber (RS)
An entity that consumes Resource Objects (RO) and typically don't create new Objects or Attributes. An example would be a SaaS application that deliver a service and needs to create a database of Objects and would get those from a RM/RC/RU. 

#### External Resource Creator (ERC)
An entity that has information about Resource Objects (RO)  and its Resource Attributes (RA), but does not participate in SCIM flows, examples include databases or internally-facing applications.

~~~~~~~~
   +-------------+ +-------------+   +-------------+ +-------------+
   |(RO) Resource| |(RA) Resource|   |(RO) Resource| |(RA) Resource|
   |   Object1   | |  Attribute1 |   |   Object2   | |  Attribute2 |
   +-------------+ +-------------+   +-------------+ +-------------+
          |               |                 |               |
   +-------------+ +-------------+   +-------------+ +-------------+
   |(RC) Resource| |(RU) Resource|   |(RC) Resource| |(RU) Resource|
   |  Creators   | |  Updaters   |   |  Creators   | |  Updaters   |
   +-------------+ +-------------+   +-------------+ +-------------+
       |               |                 |                |
       +--------+------+-----------------+-------+--------+
                |                                |
                v                                v
       +----------------+              +----------------+
       | (RM) Resource  |              | (RM) Resource  |
       |     Manager    |              |     Manager    |
       +----------------+              +----------------+
                |                                |
       +----------------+              +----------------+
       |                |              |                |
       v                v              v                v
  +-------------+ +-------------+   +-------------+ +-------------+
  |(RS) Resource| |(RS) Resource|   |(RS) Resource| |(RS) Resource|
  |  Subscriber | |  Subscriber |   |  Subscriber | |  Subscriber |
  +-------------+ +-------------+   +-------------+ +-------------+
          |                                  |
    +----------------+                  +----------------+
    |                |                  |                |
    v                v                  v                v
 +-------------+ +-------------+   +-------------+ +-------------+
 |(RO) Resource| |(RO) Resource|   |(RO) Resource| |(RO) Resource|
 |   Object1   | |   Object2   |   |   Object1   | |   Object2   |
 +-------------+ +-------------+   +-------------+ +-------------+
                      Figure 1: SCIM Orchestrators Roles
~~~~~~~~

### Triggers
Triggers are activities that may cause a SCIM action to occur.  Triggers can occur as a result of business processes like a corporate hiring event, but can also be scheduled events such as a unix bash script running as a chron job, they can also be a SSO just-in-time events arriving at a federated relying party that identifies a not-seen-before user. Triggers can also be standardized events, such as those in the OpenID Shared Signals Framework. 
Triggers are used to start a CRUD (Create, Read, Update, Delete) using SCIM Actions, the use cases described in this document can use or or multiple triggers mechanisms to achieve the goal of the SCIM element.

#### Periodic Intervals
A periodic interval trigger is a configured-in-advance agreement where a SCIM client or server performs an action at a specific time. This trigger is often recurring, and will start an action typically from the SCIM Client, but can also in some use cases be done by the SCIM Server. An example of a periodic interval trigger could be a UNIX chron job calling a script.

#### Events
Event triggers are activities, contexts or notifications that could happen at any time. A SCIM client may be configured to perform a given SCIM action in response to a specific event occurring such as a specific entry written into an audit log, a signal of a corporate workflow completion, or a device management platform notification. A SCIM actions could also be triggered by a Security Event Token (SET) as described in [RFC8417] or a SCIM event corresponding to [SCIM Profile for Security Event Tokens].

#### Application Triggers
Application triggers occur when administrative or end-user interfaces are manipulated. An example of an application trigger might be a user modifying their profile information, resulting in a SCIM client performing an HTTP POST to update the user's resource object at the SCIM server. Another example might be an Identity Administrator create a new User in the IdM and the IdM and immediately wants to update one of more resource Subscriber (typically a SaaS application that is a SCIM Server)

#### SSO (Single Sign-on)
Single Sign-on triggers occur when a user authenticates via federated protocols such as SAML 2.0 or OpenID Connect. If a federated assertion arrives for a user who has not yet been provisioned into the destination application, the application may be triggered to perform just-in-time (JIT) provisioning. This trigger occurs in scenarios where a Single Sign-On flow happens, but not all the resource attributes for the user object are passed in the federated assertion, resulting in a SCIM action to push or pull remaining needed attributes.  

~~~~~~~~
+---------------+                                   +---------------+
|               |                                   |               |
|               |                                   |               |
|               |                                   |     SCIM      |
|    Client     |                (1)                |    Server     | 
|               | <-------------------------------> |               |
|  (typically   |                                   | (typically an |
|   an IdM)     |                (2)                |      SaaS     |
|               | <-------------------------------> | Application)  |   
|               |                                   |               |
|    RC/RU/RM   |                                   |      RS       |
|               |                                   |               |
+---------------+                                   +---------------+
          Figure 2:  SCIM  Flow and Entities map
~~~~~~~~

 1. SSO trigger that creates the user and might create some RA (Resource Attributes) of a RO (Resource Object)       
 1. SCIM actions that will complement the attributes created before with an SSO JIT with additional RA (Resource Attributes) of the RO (Resource Objects) created before.   
This use case combines the SCIM protocol with other protocols used for Single Sign-On, specially in the use case of JIT (Just in time Provision), specially useful with protocols like SAML that is limit by the number of characters in the URL.

### SCIM Actions
The SCIM protocol defines interactions between two standardized parties that conform to HTTP RESTful conventions. The protocol enables CRUD operations by corresponding those activities to HTTP verbs such as POST, PUT, GET, DELETE etc.  The protocol itself doesn't assume a direction of data flow, and use cases discussed in section 4 are created using the orchestrator roles and an SCIM entity can have multiple roles, depending on the objective of the use case that we are describing.

#### Client active Push
A SCIM client uses HTTP verbs POST, PUT or PATCH to create or update objects and/or attributes at a SCIM server. The SCIM client is actively "pushing" the data to the endpoint. This SCIM action can occur when the SCIM client is the primary Resource Creator/Updater (RC/RU).
   
The most common example, and most widely deployed example is a SCIM Client that is going to provide information about a RO and its RA to a Server, that can also be called a SCIM Server in [RFC7643] and [RFC7644].

~~~~~~~~
+----------------+                                   +----------------+
|                |                                   |                |
|                |                                   |                |
|                |                                   |                |
|      SCIM      |                (1)                |      SCIM      |
|     Client     |  -------------------------------> |     Server     |
|                |                                   |                |
|                |                (2)                |                |
|                | <-------------------------------- |                |   
|     RM/RC/RU   |                                   |        RS      |
|                |                                   |                |
|                |                                   |                |
+----------------+                                   +----------------+
              Figure 3: SCIM  Flow and Orchestrator roles maps
~~~~~~~~

1. There will be push using a HTTP POST, PUT, PATCH, DELETE depending on the operation that the Client want to achieve at the Server. 
2. The Service Provider will return the RO/RA with additional metadata information to allow for audit.   

#### Client Active Pull
A SCIM client uses the HTTP GET verb to ask for data from a SCIM Server, the client with the action of active pull will fetch one object or multiple objects from a SCIM server. 
Client active pulls can be used in situations where a client needs to maintain a synchronized large body of objects, such as a device list or user address book, but where there isn't any need to track individual RO/RA. 
There are case where the client does an one time pull of specific RO and its RA from a server that manages many RO, for example a mobile app (SCIM Client) that gets the current license entitlement from a Device manager (SCIM Server) 

~~~~~~~~
+----------+                                   +----------+
|          |                                   |          |
|          |                                   |          |
|          |                                   |          |
|   SCIM   |                (1)                |   SCIM   |
|  Server  | <-------------------------------- |  Client  |
|          |                                   |          |
|          |                (2)                |          |
|          | --------------------------------> |          |   
| RC/RU/RM |                                   |    RS    |
|          |                                   |          |
|          |                                   |          |
+----------+                                   +----------+
         Figure 4:  SCIM  Flow and Orchestrator roles maps
~~~~~~~~
   
1. The SCIM client will do an HTTP GET to obtain the selected list of RO (Resource Object) and its RA (Resource Attributes).  
2. The SCIM Service Provider will return the RO and its RA with additional metadata information to allow for audit.  


********************************************************************************

#### Active Dynamic Query
A SCIM client uses the HTTP GET verb to ask for data from a SCIM Server, the client with the action of active pull will fetch one object or multiple objects from a SCIM server. At this point the SCIM server will provide an DQ token (Dynamic Query token) that will allow to locate in the Resource Object (RO) Database from which point in time the next update needs to start from, this way instead of providing a full sync every time that SCIM actions run, we achieve a delta update, just with the CRUD operation since the DQ token.
With this kind of actions we would allow for SCIM reconciliations, where the SCIM client could fix inconsistences creates by changes in the SCIM Server.

~~~~~~~~
+----------+                                   +----------+
|          |                                   |          |
|          |                                   |          |
|          |                                   |          |
|   SCIM   |                (1)                |          |
|  Server  | <-------------------------------- |  Client  |
|          |                                   |          |
|          |                (2)                |          |
|          | --------------------------------> |          |   
| RC/RU/RM |                                   | RS/RU/RS |
|          |                                   |          |
|          |                                   |          |
+----------+                                   +----------+
         Figure 5:  SCIM  Flow and Orchestrator roles maps
~~~~~~~~
   
1. The SCIM Client will do an HTTP GET to obtain a delta list of RO (Resource Object) and its RA (Resource Attributes), since the previous SCIM action.
2. The SCIM Service Provider will return the delta list of RO and its RA with additional metadata information to allow for audit.  

#### Domain Replication Mode
This is a action just for triggers that are events, for this mode there is an administrative relationship spanning multiple operational domains, data shared in Events typically uses the full mode variation of change events including the data payload attribute.  This eliminates the need for a call back to retrieve additional data.
"Domain Based Replication" events (DBR) is to synchronize resource changes between SCIM service providers in a common administrative domain. 

~~~~~~~~
+--------+                +---------------+                 +---------+
|        |                |               |                 |         |
|  SCIM  |                |               |                 |         |
| Client |                |  SCIM Server  |                 |         |
|        |     (1)        |               |      (3)        |  SCIM   |
|        | <------------- |               | --------------> | Server  |
|        |                |               |                 |         | 
| RM/RC  |     (2)        |               |                 |         |
|  /RU   | -------------> |               |                 |         |
|        |                |     RS/RC/RU  |                 |         |
|        |                |               |                 | RS/RC   |
|        |                |               |                 |  /RU    |
+--------+                +---------------+                 +---------+
                     Figure 6:  SCIM  Flow and Orchestrator roles maps
~~~~~~~~

1. SCIM Operation.   
2. SCIM Response.   
3. Event SCIM:prov:<op> id:xyz

#### Co-Ordinated Provision 
In these relationships an Event Publisher and Receiver [SCIM Profile for Security Event Tokens] typically exchange resource change events without exchanging data.  For a receiver to know the value of the data, the Event Receiver usually has calls back to the SCIM Event Publisher domain to receive a new copy of data (e.g. Uses a SCIM GET request).
In any Event Publisher and Receiver relationship, the set of SCIM resources (e.g.  Users) that are linked or co-ordinated is managed within the context of an event feed and which MAY be a subset of the total set of resources on either side.  For example, an event feed could be limited to users who have consented to the sharing of information between domains.  To support capability, "feed" specific events are defined to indicate the addition and removal of SCIM resources from a feed.  


************************************************


# SCIM Use Cases
This section describes some common SCIM use cases, explaining when, where, why and how we find them in the cross-domain environment. The ultimate goal is guidance for developers working on common models, explaining challenges and components.
Because SCIM is a protocol where two entities exchange information about resources across domains, the use cases explain how the different components can interact to allow from simple to complex architectures for cross domain resource management. Orchestrators roles are mapped to the use cases to simplify the task of explaining the multiple functions of the SCIM elements. Use cases build on each other, starting with simple cases, and ending with the most complex ones. 

## Self-Referential Resource Updates via /me
Get information about persona /me endpoint.  
A use case cover in [RFC7644] where a SCIM client can do CRUD operation on the entity of the user, in this use case the SCIM client that is the RM (Resource Manager), RC (Resource Creator) and RU (Resource Updater), will be able to read, create, update the RO (Resource Object) and its RA (Resource Attributes) in the RS (Resource Subscriber). the RS will provide an /me URI to achieve this.  
Special considerations exist from authorization perspective; unlike other listed CRUD use cases, the authorization for this use case only allows access to the RO (Resource Object) of the resource owner.  

## Simple Resource Update
Single RM/RC/RU and multiple RS.  
This is a very common and simple SCIM use case, we have the IdM/Device Managers/etc. do all CRUD operation with the resources, then after the trigger mechanisms the resources information RO/RA reach the RS (Resource Subscribers), also know as the SaaS Application.  
The RS (Resource Subscriber) will take the decision on which RA (Resource Attributes) to consider and how the RO (Resource Object) will show in its resource database.  
Typically we will find this kind of use case in small to mid size organization, where there is no structure method to handle the resources and typically in Organization that start with a blank sheet of paper in a greenfield deployment.

## Resource Updates Originating at a Non-SCIM Source
One or more ERC with single RM/RC/RU and multiple RS.  
This is another common use case, because it allow the organization to adopt SCIM protocol for CRUD operations of their resources. In this use case the organization already have an existent database of resources that is going to be the source of truth for the Resource Manager.   
Normally this ERC, specially if we are talking about user Identity, will have a User database that can be accessible using LDAP, some times the ERC can provide RO/RA using SAML Single Sign-On using Just in time Provision. We also see some IDaaS providing softwares that allow them to exchange resource information by using proprietary protocols, very common using HTTP REST to get the information from the ERC to the RM.  
Typically in this use case the RM will become the new source of truth for the resources of our Organization, will add extra RA (Resource Attributes) and ignore other RA that existed in the ERC.  
Some organization that already realize that going forward in the SCIM path, the RM will manage the RO/RA, will also start create new RO in the RM.  
The Resource Subscribers will consume all or a subset of the RO/RA from the RM.  
Typically we will see this use case in small to mid size organization where resources were organized in a non standardize platform for Resources Management, where it isn't possible to cut/replace everything with a new system.    
    
## Resources from Multiple SCIM Sources Coordinated by a Resource Manager
One or more RC/RU, with single RM/RC/RU/RS and multiple RS.  
In this use case, the the CRUD operation for the RO (Resource Object) and its RA (Resource Attributes) does not belong to the RM (Resource Manager), this is done in a separate SCIM entity, the Resource Creator/Resource Updater.   
A good example of this is use case are Organization that have their HR application, and the lifecycle of the resource (typically groups and Users) is done by that application.  
We could also have devices where the creation and update operations are always done by the device  itself or by a mobile application/web server on their behalf, in this use case the roles of RC/RU moves away from the RM.
We could also have this use case where the RM is extended with the Roles of RC/RU for extra RA (Resources Attributes), but the RO (Resource Object) is typically created by the "HR System"/device.   
Typically we will see this use case in mid to large organization where no structure method to handle the resources start with a blank sheet of paper in a  greenfield deployment. 

## Resources from SCIM and Non-SCIM sources, Coordinated by a Resource Manager
One or more ERC, one or more RC/RU, with single RM/RC/RU/RS and multiple RS.  
In this use case, one source of the Resource information is an ERC (External Resource Creator), or the entity that has the role of RC/RU (such as an HR System). In some cases the HR system can also consume information from the ERC and complement it. 
This doesn't mean that the RM will not need to consolidate RO/RA from the SCIM and non SCIM entities and consolidate and aggregate RO/RAs for those multiple sources. The RM gets its RO (Resource Object) from both systems the RC/RU and from the ERC, and need to define rules which ones to take and to ignore.

## Complex sources including Multiple Resource Updaters
One or more ERC, one or more RC/RU, with single RM/RC/RU/RS and multiple RS/RU.  
In this use case we add the capability of the Resource Subscriber to be also an Resource Update, it is very common that an SaaS application can be the source of truth for specifics RA and add extra details to the RO.  
Typically we will see this use case in large organization where resources were organized in a non standardize platform for Resources Management and it isn't possible to cut/replace everything with a new system. Those organization start to adopt many application that brings new attributes to the different resources that already exist in the system.  

## Complex Multi-directional Object and Resource Management with simple Resource Subscribers
One or more ERC, one or more RC/RU/RS, with single RM/RC/RU/RS and multiple RS/RU.  
In this use case we introduce the possibility of the RC/RU (example given before the HR System) be interested in the attribute that was created updated by the RS/RU (also known as the SaaS application), an example could be adding the business email that was created by the mail service (that came from RS/RU) to the HR information service (the RC/RU/RS element).  
Typically we will see this use case in large organization where resources were organized in a non standardize platform for Resources Management and it isn't possible to cut/replace everything with a new system. Those organization start to adopt many application that brings attributes to the different resources that already exist in the system, but they need to have all the important attributes of Resources in a application in our examples "HR application".  

## Complex Multi-direction Object/Resource Management with bi-directional Resource Subscriber/Updaters 
One or more ERC, one or more RC/RU/RS, with one or more RM/RC/RU/RS and multiple RS/RU.  
In this use case we introduce the possibility of having multiple Resource Managers, where the information from the RO/RA is consolidated across different domains/services.  
Typically we will see this use case in large organizations, or between organizations that have their own business to business communication and have the need to exchange information about Resources. This example also happens during mergers or acquisitions, where multiple RMs exist and IT departments have to manage each RM in parallel.

# Security Considerations
Authentication and authorization must be guaranteed for the SCIM operations to ensure that only authenticated entities can perform the SCIM requests and the requested SCIM operations are authorized. 
SCIM resources (e.g., Users and Groups) can contain sensitive information.  Thus, data confidentiality MUST be guaranteed at the transport layer.
There can be privacy issues that go beyond transport security, e.g., moving personally identifying information (PII) offshore between different SCIM elements.
Regulatory requirements shall be met when migrating identity information between jurisdictional regions (e.g., countries and states may have differing regulations on privacy).
Additionally, privacy-sensitive data elements may be omitted or obscured in SCIM transactions or stored records to protect these data elements for a user. For instance, a role-based identifier might be used in place of an individual's name.
Detailed security considerations are specified in Section 7 of the SCIM protocol [RFC7644] and Section 9 of the SCIM schema [RFC7643].

# IANA Considerations
There are no additional IANA considerations to those specified [RFC7643] and [RFC7644].

# Acknowledgements


--- back
