---
stand_alone: true
ipr: trust200902
submissiontype: IETF
keyword: [Internet-Draft, SCIM]
workgroup: SCIM

cat: info
title: 'System for Cross-domain Identity Management: Definitions, Use Cases and Concepts'
author:
- name: Paulo Jorge Correia
  org: Cisco Systems

- name: Pamela Dingle
  org: Microsoft Corporation

--- abstract

This document provides definitions,overview and selected use cases of the System for Cross-domain Identity Management (SCIM).  It lays out the system's concepts, models, and flows, and it includes user scenarios, use cases, and requirements.

--- middle

# Introduction
   This document provides the SCIM definitions, overview, concepts, flows, scenarios, and use cases.  It also provides a list of the requirements derived from the use cases.
   The document's objective is to help with understanding of the design and applicability of the SCIM schema [RFC7643] and SCIM protocol [RFC7644].  
   Unlike the practice of some protocols like Application Bridging for Federated Access Beyond web (ABFAB) and SAML2 WebSSO, SCIM provides provisioning and de-provisioning of resources in a separate context from authentication (aka just-in-time provisioning).  
   This document will describe the different construct that we have in the SCIM protocol and will provide the most typical use case that we will find in the implementation, will also help identify the interactions between the different constructs and guide on the roles that each has in the SCIM protocol.  
   SCIM is a protocol where it relies on one-to-one interaction, in a client-server model. Any interaction is based on a trigger that will start a CRUD event on one or many resources.  

--- back
