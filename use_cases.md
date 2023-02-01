#               System for Cross-domain Identity Management:
#           Definitions, Overview, Concepts, and Requirements

## Abstract
   This document provides definitions and an overview of the System for
   Cross-domain Identity Management (SCIM).  It lays out the system's
   concepts, models, and flows, and it includes user scenarios, use
   cases, and requirements.

## 1.  Introduction
   This document provides the SCIM definitions, overview, concepts, flows, scenarios, and use cases.  It also provides a list of the requirements derived from the use cases.
   The document's objective is to help with understanding of the design and applicability of the SCIM schema [RFC7643] and SCIM protocol [RFC7644].
   Unlike the practice of some protocols like Application Bridging for Federated Access Beyond web (ABFAB) and SAML2 WebSSO, SCIM provides provisioning and de-provisioning of resources in a separate context from authentication (aka just-in-time provisioning).
   This document will describe the different construct that we have in the SCIM protocol and will provide the most typical use case that we will find in the implementation, will also help identify the interactions between the different constructs and guide on the roles that each has in the SCIM protocol.
   SCIM is a protocol where it relies on one-to-one interaction, in a client-server model. Any interaction is based on a trigger that will start a CRUD event on one or many resources.

###    1.1.  Terminology
      The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119] when they appear in ALL CAPS.  These words may also appear in this document in lowercase as plain English words, absent their normative meanings.
      Here is a list of acronyms and abbreviations used in this document:
         - **COI:** Community of Interest
         - **CRM:** Customer Relationship Management
         - **CRUD:** Create, Read, Update, Delete
         - **RC:** Resource Creator
         - **RU:** Resource Updater
         - **RM:** Resource Manager 
         - **RS:** Resource Subscriber 
         - **RO:** Resource Object 
         - **RA:** Resource Attribute 
         - **ERC:** External Resource Creator 
         - **IaaS:** Infrastructure as a Service
         - **JIT:** Just In Time
         - **PaaS:** Platform as a Service
         - **SaaS:** Software as a Service
         - **IDaaS:** ID as a Service
         - **IdM:** Identity Manager
         - **SAML:** Security Assertion Markup Language
         - **SCIM:** System for Cross-domain Identity Management
         - **SSO:** Single Sign-On

## 2.  SCIM User Scenarios
###   2.1.  Background and Context
      The System for Cross-domain Identity Management (SCIM) specification is designed to manage resources and services in cloud-based applications in a standardized way to enable interoperability, security, and scalability.
      The specification suite seeks to build upon experience with existing schemas and deployments, placing specific emphasis on simplicity of development and integration, while applying existing authentication, authorization, and privacy models.
      The intent of the SCIM specification is to reduce the cost and complexity of user management operations by providing a common user schema and extension model, as well as binding documents to provide patterns for exchanging this schema using standard protocols. In essence, make it fast, cheap, and easy to move users in to, out of, and around the cloud.
      The SCIM scenarios are overviews of user stories designed to help clarify the intended scope of the SCIM effort.

###   2.2.  Model Concepts
####      2.2.1.  Triggers
         Quite simply, triggers are actions or activities that start SCIM flows.
         Triggers may not be relevant at the protocol level or the schema level; they really serve to help identify the type or activity that resulted in a SCIM protocol exchange. 
         Triggers used to allow CRUD (Create, Read, Update, Delete) operations as it is designed to capture a class of use case that makes sense to the actor requesting it rather than to describe a protocol operation.
            - **Instruction to Create SCIM Resource -** Service On-boarding Trigger: This is a service for the on-boarding activity in which a business action such as a new hire or new service subscription is initiated.
            An example of this could be the RC (Resource Creator) pushes the RO (Resource Object) to the RM (Resource Manager).
            - **Notification of Creation of a SCIM Resource –** Service Notification of creation Trigger: This is a service for the on-boarding activity in which a business action such as a new hire or new service subscription is initiated.
            An example of this could be the RC to create send an event to RM notifying him that an resource has been created. This trigger can send the information of the RO was created and provide its RA or can just provide the information on the it was created and expect that the RM pull the RO/RA from the RC. 
            - **Instruction to Update SCIM Resource -** Service Change Trigger: An "update SCIM resource" trigger is a service change activity as a result of a resource moving or changing its service level.
            An example of this could be the RC (Resource Creator) or RU (Resource Updater) pushes the update of RO (Resource Object) or its RA (Resource Attributes) to the RM (Resource Manager).
            - **Notification of Update SCIM Resource -** Service Notification of Change Trigger: An "update SCIM resource" trigger is a service change activity as a result of a resource moving or changing its service level.
            An example of this could be the RC or RU sends an event to RM notifying him that RO or RA has been updated. This trigger can send the information of the RO updated and provide its RA or can just provide the information on the it was updated and expect that the RM pull the RO/RA from the RC or RU. 
            - **Instruct to Delete SCIM Resource -** Service Termination Trigger: A "delete SCIM resource" trigger represents a specific and deliberate action to remove a resource from a given SCIM service point.
            An Example of this could be the RC (Resource Creator) or RU (Resource Updater) pushes the delete operation to the RM (Resource Manager).
            - **Notification of Deletion of a SCIM Resource –** Service Notification of termination Trigger: A "delete SCIM resource" trigger represents a specific and deliberate action to remove a resource from a given SCIM service point.
            An example of this could be the RC or RU to send an event to the RM notifying him that a resource has been deleted. This trigger can send the information of the RO was deleted. 

####      2.2.2.  Roles/Constructs
         Constructs are the operating parties that take part in both sides of a SCIM protocol exchange and help identify the source of a given Trigger. 
         A specific element can have one or more constructs roles, depending on the type of services that is delivering in the SCIM architecture.
         So far, we have identified the following SCIM constructs:
            - **Resource Object (RO):** Is and object that is going to be manipulated (CRUD) by the different SCIM players, and in the end the ultimate goal to be pass across different systems and to make sure that consistent information is exchange. The Resource Object have attributes that are define by Schemas, an example of that is the SCIM Core Schema defines in RFC 7643.
            - **Resource Attributes (RA):** Is one element of the Resource Object (RO), it can have a single value or continue multiple values to describe a specific resource and all its characteristics, an example of this can be the different attributes for user and/or groups under the SCIM Core Schema defined in RFC 7643.
            - **Resource Creator (RC):** Is an entity operating in a given service, is responsible of creating the Resource Object (RO) with is attributes (RA), typically we can see this role in HR or resource management applications that are responsible to create resources and be authorities for some or all its attributes. 
            - **Resource Updater (RU):** Is an entity that is responsible for update specific attributes (RA) of a Resource Object (RO). Typically, this role is use in conjunction with other SCIM roles that allow this SCIM entity to be authority for a specific Resource Attribute (RA)
            - **Resource Manager (RM):** Is an entity that consolidated the resource Objects (RO) from the Resource Creators/Updaters (RC/RU) and make it available for the Resource Subscribers (RS), typically this entity/role is handle by the IDaaS.
            - **Resource Subscriber (RS):** Is an entity that consumes Resource Objects (RO) but that is not authoritative to create them or any of its Resource Attribute (RA), normally this entity is only interested in part of the Resource Objects available in the Resource Manager (RM), typically it is application that requires information on resources to operate.
            - **External Resource Creator (ERC):** IS an entity that has information about resources and its attributes, but that doesn’t understand SCIM, typically it is going to provide the information on the resources to the Resources Manager, using non SCIM protocols/mechanisms, an example of this would be an services that gets information about users from an LDAP server and provide it to an IDaaS using some kind of proprietary REST APIs.

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
                                    Figure 1: SCIM Roles Constructs


## 3.  SCIM Use Cases
   This section we will describe the most common SCIM use cases, and will explain when, where, why and how we find them in the cross domain environment for managing resources. This list by no way tries to be exhaustive and complete and tried to guide developers for the possibility of such models and will try to explain the challenges and the components.
   As mention before SCIM is a protocol for cross domains where two entities exchange information about a resource, with the use cases we try to go further and explain on how the different components can interact to allow from simple to complex architectures for cross domain resource management.
   Typically each bellow  use case add something on top of the previous one, starting in the most simple one, and finishing in the most complex ones, to make it easier the explanation, assume that what was describe in the previous use case applies to the use cases that come after.

###   3.1.  Single RM/RC/RU and multiple RS
      This is very common SCIM use case and basic use case, allows that the IdM do all CRUD operation with the resources, then using the trigger mechanisms described before we achieve that the resource information reach the Resource Subscribers. The RS will take the decision on which resource attributes to take and how the Resource Object will show in their resource database.
      Typically we can find this kind of use case in small to mid size organization, where no structure method to handle the resources and the Organization start fresh or it is a greenfield Organization.

###   3.2.  One or more ERC with single RM/RC/RU and multiple RS
      This is the most common use case, because it allow the organization to adopt SCIM protocol for CRUD operations of their resources, but in this use case the organization already have an existent database of resources that is going to be the source of truth for the Resource Manager or is going to be the starting point. At no point in time the SCIM RM will provide SCIM operation with that External Resource Creator.
      Normally this ERC, specially if we are talking about user Identity, will have a User database that can be accessible using LDAP or can provide information of their user attributes by doing an SAML Single SignOn using Just in time Provision. Most of the IDaaS also provide softwares that allow them to get resource information by using proprietary protocols, generally using HTTP REST to get the information from the ERC to the RM.
      Typically in this use case the RM will become the new source of truth for the resources of our Organization, and will add extra Resource Attributes and ignore other that existed in the ERC.
      Some organization that already realize that going forward the RM will be the authority answer for the Resources Object and Attributes, will start create new Resource Objects in this service.
      The Resource Subscribers will consume all the resource information from the RM.
      Typically we will see this use case in small to mid size organization where resources were organized in a non standard and non open platform for Resources Management and it isn't possible to cut/replace everything with a new system.
    
###   3.3.  One or more RC/RU, with single RM and multiple RS
      In this use case, the authority for the CRUD operation to the Resource Object and its Resource Attributes does not belong to the Resource Manager, thi sis done in a separate entity that has this responsibilities. 
      A good example of this is use case is those Organization that have their HR application, and the lifecycle of the resource (typically groups and Users) is done by that application.
      We could also have this use case where the RM is extended with the Roles of RC/RU for extra resources that are not authoritative by the "HR System", but normally that bring more complexity to the authority models for the CRUD operation of the resources.  
      The Resource Subscribers will consume all the resource information from the RM.
      Typically we will see this use case in mid to large organization where no structure method to handle the resources and the Organization start fresh or it is a greenfield Organization.


###   3.4.  One or more ERC, one RC/RU, with single RM and multiple RS
      In this use case the Resource information is in a External Resource Creator, and the entity that has the role of RC/RU (example given before the HR System) consumes information from the ERC, but it add extra Resource Attributes, so from a model perspective, the RM get its authoritative Information from both systems the RC/RU and ERC.
      In this model there need to be careful thoughts so that we avoid loops where specific Resource Attributes write over and over again by the ERC and RC/RU.
      The Resource Subscribers will consume all the resource information from the RM.
      Typically we will see this use case in mid to large organization where the where there were no structure method to handle the resources and the Organization start fresh or it is a greenfield Organization.
      The Resource Subscribers will consume all the resource information from the RM.
      Typically we will see this use case in mid to large organization where resources were organized in a non standard and non open platform for Resources Management and it isn't possible to cut/replace everything with a new system.

###   3.5.  One or more ERC, one or more RC/RU, with single RM/RU/RS and multiple RS/RU

###   3.6.  One or more ERC, one or more RC/RU/RS, with single RM/RU/RS and multiple RS/RU

###   3.7.  One or more ERC, one or more RC/RU/RS, with one or more RM/RU/RS and multiple RS/RU





## 4.  Security Considerations
   Authentication and authorization must be guaranteed for the SCIM
   operations to ensure that only authenticated entities can perform the
   SCIM requests and the requested SCIM operations are authorized.
   SCIM resources (e.g., Users and Groups) can contain sensitive
   information.  Thus, data confidentiality MUST be guaranteed at the
   transport layer.
   There can be privacy issues that go beyond transport security, e.g.,
   moving personally identifying information (PII) offshore between
   CSPs.  Regulatory requirements shall be met when migrating identity
   information between jurisdictional regions (e.g., countries and
   states may have differing regulations on privacy).
   Additionally, privacy-sensitive data elements may be omitted or
   obscured in SCIM transactions or stored records to protect these data
   elements for a user.  For instance, a role-based identifier might be
   used in place of an individual's name.
   Detailed security considerations are specified in Section 7 of the
   SCIM protocol [RFC7644] and Section 9 of the SCIM schema [RFC7643].



## 5.  References
###   5.1.  Normative References
      [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
               Requirement Levels", BCP 14, RFC 2119,
               DOI 10.17487/RFC2119, March 1997,
               <http://www.rfc-editor.org/info/rfc2119>.
###   5.2.  Informative References
      [RFC7643]  Hunt, P., Ed., Grizzle, K., Wahlstroem, E., and
               C. Mortimore, "System for Cross-domain Identity
               Management: Core Schema", RFC 7643, DOI 10.17487/RFC7643,
               September 2015, <http://www.rfc-editor.org/info/rfc7643>.
      [RFC7644]  Hunt, P., Ed., Grizzle, K., Ansari, M., Wahlstroem, E.,
               and C. Mortimore, "System for Cross-domain Identity
               Management: Protocol", RFC 7644, DOI 10.17487/RFC7644,
               September 2015, <http://www.rfc-editor.org/info/rfc7644>.

## Acknowledgments

## Authors' Addresses

