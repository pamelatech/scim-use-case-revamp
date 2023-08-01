---
stand_alone: true
docname: draft-correia-scimusecases-01
ipr: trust200902
submissiontype: IETF
keyword: [Internet-Draft, SCIM]
workgroup: SCIM

cat: info
title: 'SCIM Use Cases'
author:
- name: Paulo Jorge Correia
  org: Cisco Systems

- name: Pamela Dingle
  org: Microsoft Corporation

--- abstract

This document provides definitions, overview and selected use cases of the System for Cross-domain Identity Management (SCIM).  It lays out the system's concepts, models, and flows, and it includes use cases, and implementation considerations.

--- middle

# Introduction
The System for Cross-domain Identity Management (SCIM) family of specifications are designed to manage resources used in the practice of identity management that are often communicated across internet domains and services, with users and groups as the default resources supported and an extensibility model compatible with adding additional resources. The specifications have two primary goals: 1) A common representation of a resource object and its attributes, and 2) Standardized patterns for how those resources can be operated on, including "CRUD" operations that create, read, update or delete resource objects and more advanced goals such as search filters, synchronization of large resource populations, etc. These goals are codified as a data model in [RFC7643] defining resources, attributes and default schema, as well as a protocol definition built on HTTP in [RFC7644]. By standardizing the data model and protocol for resource management, entire ecosystems can achieve better interoperability, security, and scalability.

This document provides definitions, overview, concepts, flows, and use cases implementers may need to understand the design and applicability of the SCIM schema [RFC7643] and SCIM protocol [RFC7644]. Unlike the practice of some protocols like Application Bridging for Federated Access Beyond web (ABFAB) and SAML2 WebSSO, SCIM provides provisioning and de-provisioning of resources in a separate context from authentication. While SCIM is a protocol that standardizes movement of data only between two parties in a HTTP client-server model, implementation patterns are discussed in this document that surround the SCIM actions and are mport



# Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119] when they appear in ALL CAPS. These words may also appear in this document in lowercase as plain English words, absent their normative meanings. Here is a list of acronyms and abbreviations used in this document:

* CRUD: Create, Read, Update, Delete
* RC: Resource Creator
* RU: Resource Updater
* RM: Resource Manager 
* RS: Resource Subscriber 
* RO: Resource Object 
* RA: Resource Attribute 
* ERC: External Resource Creator 
* IaaS: Infrastructure as a Service
* JIT: Just In Time
* PaaS: Platform as a Service
* SaaS: Software as a Service
* IDaaS: Identity as a Service
* IdM: Identity Manager
* SAML: Security Assertion Markup Language
* SCIM: System for Cross-domain Identity Management
* SSO: Single Sign-On


# SCIM Components and Architecture
The System for Cross-domain Identity Management (SCIM) family of specifications are designed to manage resources and services using standards to enable better interoperability, security, and scalability. When every separate domain manages profiles of users, groups, devices and other

The specification suite seeks to build upon experience with existing schemas and deployments, placing specific emphasis on simplicity of development and integration, while applying existing authentication, authorization, and privacy models.  
The intent of the SCIM specification is to reduce the cost and complexity of resource management operations by providing a common schemas and extension model, as well as binding documents to provide patterns for exchanging this schema using standard protocols. In essence, make it fast, cheap, and easy to move resources in to, out of, and around the applications.  
The SCIM scenarios are overviews of user stories designed to help clarify the intended scope of the SCIM effort.  

## Implementation Concepts
To understand the use cases we need to understand 4 different concepts of the protocol, that will describe underlying protocol, the different orchestrators roles, how we start the SCIM interaction and what methods we have to execute the actions.

### Data Model
SCIM defines two types of data entities: resources and attributes.

#### Resource Object (RO)
A JSON object representing a user, group (or extension object) to be manipulated through the SCIM protocol. The Resource Object contains attributes defined by schemas such as those defined in [RFC 7643] and can be acted on via the endpoints and parameters defined in [RFC7644].  

#### Resource Attribute (RA)
A named element of a Resource Object. Attributes are defined in section 2 of [RFC7643] and include characteristics like cardinality (single or multiple values), data types (string, boolean, binary etc) and characteristics (required, unique etc). 

### HTTP Client-Server Roles
HTTP client and server roles are defined in [RFC 9110] and [RFC 9112]. Any SCIM interaction requires one participant to be a SCIM server and the other to be a SCIM client. 

#### SCIM Server (also known as a SCIM Service Provider)
An HTTP web application that provides identity information via the SCIM protocol.  
A SCIM Server is a RESTful API endpoint offering access to a data model that can be used to push or pull data between two parties. SCIM servers have additional responsibilities such as API Security, managing client identifiers & keys as well as performance management such as API throttling.  

#### SCIM Client
A website or application that uses the SCIM protocol to manage identity data maintained by the service provider.  The client initiates SCIM HTTP requests to a target SCIM Server. A SCIM Client is active software that can push or pull data between two parties.   

### Orchestrator Roles
Orchestrators are the operating parties that take part in a SCIM protocol exchange and ensure data is moving in the correct flows.
An entity can have one or more orchestrators roles, depending on the overall provisioning architecture.  

#### Resource Creator (RC)
An entity responsible for creating the Resource Object (RO). Typically we can see this role in HR or resource management applications that are responsible for creating resources and authorities for some or all of its attributes.  

#### Resource Updater (RU)
An entity responsible for updating specific attributes of a Resource Object (RO) or the RO itself. Typically, this role is used in conjunction with other SCIM roles that allow this SCIM entity to manage a specific Resource Attribute (RA).  

#### Resource Manager (RM)
An entity that aggregates or transforms resource objects from resource creators/updaters (RC/RU) and make them available for resource subscribers (RS) using multiple SCIM interactions, an example of this role could be an Identity-as-a-Service (IDaaS) cloud platform.  

#### Resource Subscriber (RS)
An entity that consumes information in resource objects (RO) but is not authoritative to create new objects or attributes. An example of entities that play this role include SaaS applications relying on an IDaaS cloud platform. 

#### External Resource Creator (ERC)
An entity that has information about resources and its attributes, but does not participate in SCIM flows, examples include databases or internally-facing applications.

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
Triggers are actions or activities that may cause a SCIM action to occur.  Triggers can occur as a result of business processes like a corporate hiring event, but can also be scheduled events such as a unix bash script running as a chron job, or can be just-in-time events such as SAML assertion arriving at a federated relying party that identifies a not-seen-before user. Triggers can also be standardized events, such as those in the OpenID Shared Signals Framework. Triggers used to allow CRUD (Create, Read, Update, Delete) using SCIM Actions or Operations as it is designed to capture a class of use case that makes sense to the actor requesting it rather than to describe a protocol operation.  

#### Periodic Intervals
A periodic interval trigger is a configured-in-advance agreement where a SCIM client performs an action at a specific time. This trigger is often recurring, and in that case the combination of trigger and action together may be referred to as "polling" the SCIM server. An example of a periodic interval trigger could be a UNIX chron job calling a script.

#### Events
Event triggers are activities, contexts or notifications that could happen at any time. A SCIM client may be configured to perform a given SCIM action in response to a specific event occuring such as a specific entry written into an audit log, a signal of a corporate workflow completion, or a device management platform notification. A SCIM action could also be triggered by a Security Event Token (SET) as described in [RFC8417] or a SCIM event corresponding to [draft-ietf-scim-events]; for example an application acting as a resource subscriber and SCIM client could receive a SCIM event denoting creation of a new user object, triggering a SCIM action to fetch all the attributes for that user.

#### Application Triggers
Application triggers occur when administrative or end-user interfaces are manipulated. An example of an application trigger might be a user modifying their profile information, resulting in a SCIM client performing an HTTP POST to update the user's resource object at the SCIM server. 

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
A SCIM client uses HTTP verbs POST, PUT or PATCH to create or update objects and/or attributes at a SCIM server. The SCIM client is actively "pushing" the data to the endpoint. This SCIM action can occur when the SCIM client is the primary resource creator/updater, or when the SCIM client is primarly a resource subscriber but is authoritative over only a subset of attributes.
   
##### Resource Object creation/update from Client to Server
In this model we will have a Client that is going to provide information about a RO and its RA to a Server, that can also be called as SCIM Server in [RFC 7643] and [RFC 7644].

~~~~~~~~
+----------------+                                   +----------------+
|                |                (1)                |                |
|                | --------------------------------> |                |
|                |                                   |                |
|                |                (2)                |      SCIM      |
|     Client     | <-------------------------------- |     Server     |
|   (typically   |                                   |  (typically a  |
|    an IdM)     |                (3)                |   Application) |
|                | --------------------------------> |                |   
|     RM/RC/RU   |                                   |        RS      |
|                |                (4)                |                |
|                | <-------------------------------- |                |
+----------------+                                   +----------------+
              Figure 3: SCIM  Flow and Orchestrator roles maps
~~~~~~~~

1. Before creating/updating a RO/RA the SCIM client will always do a HTTP GET to get current information from the SCIM Server.   
2. SCIM Server will provide the current information on the resources asked by the SCIM Client.   
3. Based on the RO and RA returned by the Server, there will be a HTTP POST, PUT, PATCH depending on the operation that the Client want to achieve.   
4. The Service Provider will return the RO/RA with additional metadata information to allow for audit.   

The SCIM client will map to the RM/RC/RU and the Server will map into RS.   

##### Resource Object creation from a Creation Entity 
In this model we will have a Client that is going to provide information about a RO and its RA to a Server, can also be called as Service Provider in [RFC 7643] and [RFC 7644], in this model the Client is just responsible for a limit set of attributes and do not do any management overall, and the Resource management function resides on the Server.

~~~~~~~~
+--------------+                                   +---------------+
|              |                (1)                |               |
|              | --------------------------------> |               |
|              |                                   |               |
|              |                (2)                |     SCIM      |
|    Client    | <-------------------------------- |    Server     |
| (typically   |                                   | (typically an |
|  an HR       |                (3)                |      IdM)     |
| Application) | --------------------------------> |               |   
|              |                                   |     RM/RS     |
|   RC/RU      |                (4)                |               |
|              | <-------------------------------- |               |
+--------------+                                   +---------------+
             Figure 4:  SCIM  Flow and Orchestrator roles maps
   
~~~~~~~~
1. Before creating/updating a RO/RA the SCIM client will always do a HTTP GET to get current information from the SCIM Server.   
2. SCIM Server will provide the current information on the resources asked by the SCIM Client.   
3. Based on the RO and RA returned by the Server, there will be a HTTP POST, PUT, PATCH depending on the operation that the Client want to achieve.  
4. The Service Provider will return the RO/RA with additional metadata information to allow for audit. 

The SCIM client will map to the RC/RU and the Server will map into RM/RS. The SCIM client is sometimes called as the "HR Application", because it responsibilities are only on be the creator and updater of the RO and specific number of its RA, the client in this case has no responsibilities on the management of Resources, typically done by an IdM.  

##### Resource Object creation from a Creation Entity and consumption from an Application
In this model we will have a Client that is going to provide information about a RO and its RA to a Server, this Client is just responsible for a limit set of attributes and do not do any management overall the RO. This SCIM element that is going to manage the RO will then be the Client for other SCIM services that will consume the RO/RA, that might have more RA than the original RO provided by the originator of the RO. 

~~~~~~~~
+--------+                +---------------+                 +---------+
|        |     (1)        |               |      (1)        |         |
|        | -------------> |               | --------------> |         |
| Client |                |SCIM Server    |                 |         |
|        |     (2)        |               |      (2)        |  SCIM   |
|        | <------------- |               | <-------------- | Server  |
|        |                |         Client|                 |         | 
|        |     (3)        |               |      (3)        |         |
|        | -------------> |               | --------------> |         |
|        |                |  RM/RS/RC/RU  |                 |         |
| RC/RU  |     (4)        |               |      (4)        |   RS    |
|        | <------------- |               | <-------------- |         |
+--------+                +---------------+                 +---------+
                     Figure 5:  SCIM  Flow and Orchestrator roles maps
~~~~~~~~
1. Before creating/updating a RO/RA the SCIM client will always do a HTTP GET to get current information from the SCIM Server.   
2. SCIM Server will provide the current information on the resources asked by the SCIM Client.   
3. Based on the RO and RA returned by the Server, there will be a HTTP POST, PUT, PATCH depending on the operation that the Client want to achieve.  
4. The Service Provider will return the RO/RA with additional metadata information to allow for audit. 

The SCIM client on the left will map to the RC/RU and the Server in the middle will map into RM/RS, the SCIM client on the left is also sometimes called as the "HR Application", because it responsibilities are only on be the creator and updater of the RO and specific number of its RA, the client in this case has no responsibilities in doing any management of the Resources, typically done by an IdM.   
The center component as describe is the Server for the client on the left, will act as the Client for the server on the right, which typically is an SaaS application that want to consume RO and its RA from an RM.

##### Resource Object creation from a Creation Entity and consumption from an Application when different Resource Attributes are generated in different entities                
In this model we will have a Client that is going to provide information about a RO and its RA to a Server, this Client is just responsible for a limit set of attributes and do not do any management overall the RO. This SCIM element that is going to manage the RO will then be the Client for other SCIM services that will consume the RO/RA, that might have more RA than the original RO provided by the originator of the RO. 
Now the right SCIM element will have it own RA that needs to be updated in the RM (Resource Manager), that will also update the SCIM element on the left.

~~~~~~~~
 +----------+               +---------------+                +--------+
 |          | -----(1)----> |               | -----(1)-----> |        |
 |  Client  | <----(2)----- |SCIM           | <----(2)------ |  SCIM  |
 |          | -----(3)----> |Server         | -----(3)-----> | Server |
 |          | <----(4)----- |         Client| <----(4)------ |        |
 |          |               |               |                |        |
 |          |               |               |                |        |
 | RC/RU/RS | <----(1)----- |  RM/RS/RC/RU  | <----(1)------ |   RS   |
 |          | -----(2)----> |Client         | -----(2)-----> |        |
 |   SCIM   | <----(3)----- |           SCIM| <----(3)------ | Client |
 |  Server  | -----(4)----> |         Server| -----(4)-----> |        |
 +----------+               +---------------+                +--------+
                 Figure 6:  SCIM  Flow and Orchestrator roles maps
   
~~~~~~~~
1. Before creating/updating a RO/RA the SCIM client will always do a HTTP GET to get current information from the SCIM Server.   
2. SCIM Server will provide the current information on the resources asked by the SCIM Client.   
3. Based on the RO and RA returned by the Server, there will be a HTTP POST, PUT, PATCH depending on the operation that the Client want to achieve.  
4. The Service Provider will return the RO/RA with additional metadata information to allow for audit. 

The SCIM client on the left will map to the RC/RU and the Server in the middle will map into RM/RS, the SCIM client on the left is also sometimes called as the "HR Application", because it responsibilities are only on be the creator and updater of the RO and specific number of its RA, the client in this case has no responsibilities in doing any management of the Resources, typically done by an IdM.   
The center component as describe is the Server for the client on the left, will act as the Client for the server on the right, which typically is an SaaS application that want to consume RO and its RA from an RM.
In addition to the models seen before now the "HR Application" also subscribe to RA that are created by the RS and reported by the RM, the Application will be the creator of specific attributes.  
So we will see that the 3 SCIM elements will be RC/RU/RS for each RO/RA.   

#### Client Active Pull
A SCIM client uses the HTTP GET verb to ask for data from a SCIM Server. A client active pull can be used to fetch one object, a subset of objects, or all objects from a SCIM server. In cases where the client is a resource updater, it may perform an active pull of an object or objects in order to determine whether an active push of new data is necessary. Client active pulls can be used in situations where a client needs to maintain a synchronized large body of objects, such as a device list or user address book, but where there isn't any need to track individual RO/RA. 
Another example of a client active pull would be a client that needs to have details of a specific device that was onboarded by a mobile application and that need to provide the RO/RA information on the behalf of the device.

##### Resource Object Creation or Update
In this model we will have a Client that is going to pull information about a RO/RA from a Server. In this model the Client is going to management all the RO (Resource Objects) and its RA (Resource Attributes), that are provided by the Server, and the RM (Resource Management) function resides on the Client.

~~~~~~~~
+----------+                                   +----------+
|          |                                   |          |
|          |                                   |          |
|          |                                   |          |
|          |                (1)                |   SCIM   |
|  Client  | --------------------------------> |  Server  |
|          |                                   |          |
|          |                (2)                |          |
|          | <-------------------------------- |          |   
|  RS/RM   |                                   |   RC/RU  |
|          |                                   |          |
|          |                                   |          |
+----------+                                   +----------+
        Figure 7:  SCIM  Flow and Orchestrator roles maps
~~~~~~~~
   
1. The SCIM client will do an HTTP GET to obtain the RO/RA that will be available in the Server.   
2. The SCIM Server will return the RO/RA with additional metadata information to allow for audit.  

A typical example of this use case is a device that is going to use a mobile application or browser base to enroll devices and gathers its attributes, that mobile application or browser after enrollment process is finish will do a trigger to notify the client that is ready to provide the RO/RA of the device. It is the SCIM client that will do al the Resource management for all the devices.   

##### Resources Subscription
In this model we will have the Client that is going to pull information about a RO/RA from the Server. In this model in the Client there is no status/change database, and it gets a list of all the RO/RA based on filters provided by the client, so there will be a full update every synchronization cycle.  

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
| RC/RU/RM |                                   |    RS    |
|          |                                   |          |
|          |                                   |          |
+----------+                                   +----------+
         Figure 8:  SCIM  Flow and Orchestrator roles maps
~~~~~~~~
   
1. The SCIM client will do an HTTP GET to obtain the selected list of RO (Resource Object) and its RA (Resource Attributes).  
2. The SCIM Service Provider will return the RO and its RA with additional metadata information to allow for audit.  

A good example would be SaaS service that needs to consume a list of contacts or devices, this SaaS service will need to know the relevant RO/RA, this operation will happen periodically and every time will get a full list of all the RO (Resource Objects).   
   
##### Resource Object Creation or Update and Subscription
In this model we will bring together both of the two previous SCIM actions for pull information, where a typically a device can be the creator or their own attributes and will allow an SaaS service to subscribe to all the different RO/RA and deliver additional services for itself and other devices. It isn't expected from any of the SCIM clients in the Active pull model to create any status database of attributes changes, so the clients will always do a pull on one or many RO (Resource Objects) based on triggers.

~~~~~~~~
+----------+                 +---------------+               +--------+
|          |                 |               |               |        |
|          |                 |               |               |        |
|          |                 |               |               |        |
|  SCIM    |       (1)       |Client         |      (3)      |        |
| Server   | <-------------- |           SCIM| <------------ | Client |
|          |                 |         Server|               |        | 
|          |       (2)       |               |      (4)      |        |
|          | --------------> |               | ------------> |        |
|          |                 |  RM/RS/RC/RU  |               |        |
|   RC/RU  |                 |               |               |   RS   |
|          |                 |               |               |        |
+----------+                 +---------------+               +--------+
                    Figure 9:  SCIM  Flow and Orchestrator roles maps
~~~~~~~~
1. The SCIM client will do an HTTP GET to obtain the RO/RA that will be available in the Server.   
2. The SCIM Server will return the RO/RA with additional metadata information to allow for audit.  
3. The SCIM client will do an HTTP GET to obtain the selected list of RO (Resource Object) and its RA (Resource Attributes).  
4. The SCIM Service Provider will return the RO and its RA with additional metadata information to allow for audit.  

A typical example of this use case is a device that is going to use a mobile application or browser base to enroll devices and gathers its attributes, that mobile application or browser after enrollment process is finish will do a trigger to notify the client that is ready to provide the RO/RA of the device. It is the SCIM client that will do all the Resource management for all the devices.  
This SCIM element in the center will also provide list list of contacts or devices, that can be consume by different SCIM entities, this operation will happen when a specific trigger will be execute by the client on the right, to get a list RO (Resource Objects) and RA (Resource Attributes) that will be defined by the filter on the client in the right.

# SCIM Use Cases
This section describes some common SCIM use cases, explaining when, where, why and how we find them in the cross-domain environment. The ultimate goal is guidance for developers working on common models, explaining challenges and components.
Because SCIM is a protocol where two entities exchange information about resources across domains, the use cases explain how the different components can interact to allow from simple to complex architectures for cross domain resource management. Orchestrators roles are mapped to the use cases to simplify the task of explaining the multiple functions of the SCIM elements. Use cases build on each other, starting with simple cases, and ending with the most complex ones. 

## Self-Referential SCIM Actions (/me)
Get information about persona /me endpoint.  
A use case cover in [RFC7644] where a SCIM client can do CRUD operation on the entity of the user, in this use case the SCIM client that is the RM (Resource Manager), RC (Resource Creator) and RU (Resource Updater), will be able to read, create, update the RO (Resource Object) and its RA (Resource Attributes) in the RS (Resource Subscriber). the RS will provide an /me URI to achieve this.  
Special considerations exist from authorization perspective; unlike other listed CRUD use cases, the authorization for this use case only allows access to the RO (Resource Object) of the user that authenticates.  

## IdM doing CRUD operations on SaaS applications
Single RM/RC/RU and multiple RS.  
This is a very common and simple SCIM use case, we have the IdM/Device Managers/etc. do all CRUD operation with the resources, then after the trigger mechanisms the resources information RO/RA reach the RS (Resource Subscribers), also know as the SaaS Application.  
The RS (Resource Subscriber) will take the decision on which RA (Resource Attributes) to consider and how the RO (Resource Object) will show in its resource database.  
Typically we will find this kind of use case in small to mid size organization, where there is no structure method to handle the resources and typically in Organization that start with a blank sheet of paper in a greenfield deployment.

## IdM doing CRUD operations on SaaS applications, and Objects coming from external non SCIM source.
One or more ERC with single RM/RC/RU and multiple RS.  
This is another common use case, because it allow the organization to adopt SCIM protocol for CRUD operations of their resources. In this use case the organization already have an existent database of resources that is going to be the source of truth for the Resource Manager.   
Normally this ERC, specially if we are talking about user Identity, will have a User database that can be accessible using LDAP, some times the ERC can provide RO/RA using SAML Single Sign-On using Just in time Provision. We also see some IDaaS providing softwares that allow them to exchange resource information by using proprietary protocols, very common using HTTP REST to get the information from the ERC to the RM.  
Typically in this use case the RM will become the new source of truth for the resources of our Organization, will add extra RA (Resource Attributes) and ignore other RA that existed in the ERC.  
Some organization that already realize that going forward in the SCIM path, the RM will be the authority answer for the RO/RA, will start create new RO in the RM.  
The Resource Subscribers will consume all or a subset of the RO/RA from the RM.  
Typically we will see this use case in small to mid size organization where resources were organized in a non standardize platform for Resources Management, where it isn't possible to cut/replace everything with a new system.    
    
## IdM doing CRUD operations on SaaS applications, and Objects coming from external SCIM source.
One or more RC/RU, with single RM/RC/RU/RS and multiple RS.  
In this use case, the the CRUD operation for the RO (Resource Object) and its RA (Resource Attributes) does not belong to the RM (Resource Manager), this is done in a separate SCIM entity, the Resource Creator/Resource Updater.   
A good example of this is use case are Organization that have their HR application, and the lifecycle of the resource (typically groups and Users) is done by that application.  
We could also have devices where the creation and update operations are always done by the device  itself or by a mobile application/web server on their behalf, in this use case the roles of RC/RU moves away from the RM.
We could also have this use case where the RM is extended with the Roles of RC/RU for extra RA (Resources Attributes) that are not authoritative by the "HR System"/device, but normally that bring more complexity to the authority models for the CRUD operation of the resources.    
Typically we will see this use case in mid to large organization where no structure method to handle the resources start with a blank sheet of paper in a  greenfield deployment. 

## IdM doing CRUD operations on SaaS applications, and Objects coming from external SCIM and non SCIM source including the IDM itself.
One or more ERC, one or more RC/RU, with single RM/RC/RU/RS and multiple RS.  
In this use case, one source of the Resource information is an ERC (External Resource Creator), or the entity that has the role of RC/RU (such as an HR System). In some cases the HR system can also consume information from the ERC and complement it. 
This doesn't mean that the RM will not need to consolidate RO/RA from the SCIM and non SCIM entities and consolidate and aggregate RO/RAs for those multiple sources. The RM gets its authoritative Information from both systems the RC/RU and from the ERC, and need to define rules which ones to take and to ignore.

## IdM doing CRUD operations on SaaS applications, and Objects coming from external SCIM and non SCIM source including the IDM itself, where some Resource Attributes come from SaaS application.
One or more ERC, one or more RC/RU, with single RM/RC/RU/RS and multiple RS/RU.  
In this use case we add the capability of the Resource Subscriber to be also an Resource Update, it is very common that an SaaS application can be the source of truth for specifics RA and add extra details to the RO.  
Typically we will see this use case in large organization where resources were organized in a non standardize platform for Resources Management and it isn't possible to cut/replace everything with a new system. Those organization start to adopt many application that brings new attributes to the different resources that already exist in the system.  

## IdM doing CRUD operations on SaaS applications, and Objects coming from external SCIM and non SCIM sources including the IdM itself, where some Resource Attributes come from SaaS application, and are updated in the SCIM object creator.
One or more ERC, one or more RC/RU/RS, with single RM/RC/RU/RS and multiple RS/RU.  
In this use case we introduce the possibility of the RC/RU (example given before the HR System) be interested in the attribute that was created updated by the RS/RU (also known as the SaaS application), an example could be adding the business email that was created by the mail service (that came from RS/RU) to the HR information service (the RC/RU/RS element).  
Typically we will see this use case in large organization where resources were organized in a non standardize platform for Resources Management and it isn't possible to cut/replace everything with a new system. Those organization start to adopt many application that brings attributes to the different resources that already exist in the system, but they need to have all the important attributes of Resources in a application in our examples "HR application".  

## Multiple IdM doing CRUD operations on SaaS applications, and Objects coming from external SCIM and non SCIM sources including the IdM itself, where some Resource Attributes come from SaaS application, and are updated in the SCIM object creator.
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


# References

## Normative References
   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
   Requirement Levels", BCP 14, RFC 2119,
   DOI 10.17487/RFC2119, March 1997,
   <http://www.rfc-editor.org/info/rfc2119>.

## Informative References
   [RFC7643]  Hunt, P., Ed., Grizzle, K., Wahlstroem, E., and
   C. Mortimore, "System for Cross-domain Identity
   Management: Core Schema", RFC 7643, DOI 10.17487/RFC7643,
   September 2015, <http://www.rfc-editor.org/info/rfc7643>.
   
   [RFC7644]  Hunt, P., Ed., Grizzle, K., Ansari, M., Wahlstroem, E.,
   and C. Mortimore, "System for Cross-domain Identity
   Management: Protocol", RFC 7644, DOI 10.17487/RFC7644,
   September 2015, <http://www.rfc-editor.org/info/rfc7644>.

   [RFC7642] K. LI, P. Hunt, B. Khasnabish, A. Nadalin and Z. Zeltsan,
   "System for Cross-domain Identity Management: Definitions, Overview, Concepts, and Requirements", 
   RFC 7642, September 2015, <http://www.rfc-editor.org/info/rfc7642>.

   [Device Schema Extensions to the SCIM model] M. Shahzad, H. Iqbal and E. Lear
   July 2023, <https://datatracker.ietf.org/doc/draft-shahzad-scim-device-model>.

   [SCIM Profile for Security Event Tokens] P. Hunt, N. Cam-Winget and M. Kiser
   July 2023, <https://datatracker.ietf.org/doc/draft-ietf-scim-events>.

## Acknowledgements


--- back
