---
title: Patents
description: |
  A page listing my patents.
layout: :theme/page
---

# Patents

Below is a list of patents I've published with the US Patent Office while working at Red Hat, Inc.

## Granted Patents

### US11893369B2 - Determining a Distributed System Topology from a Single Application Deployment
**Filed:** January 21, 2022
**Granted:** February 6, 2024
**Inventors:** Kenneth Finnigan, Paul Robinson

A system for analyzing microservice-based applications to recommend optimal deployment configurations. By examining request traces through application components and calculating interaction frequencies between components, the technology enables automated topology recommendations. Components with high interaction rates are suggested for co-location, while those with minimal communication can be deployed separately, improving operational efficiency and reducing costs.

[View on Google Patents](https://patents.google.com/patent/US11893369B2/en)

---

### US11150938B2 - Non-repudiable Transaction Protocol
**Filed:** February 25, 2019
**Granted:** October 19, 2021
**Inventors:** Paul Robinson, Thomas Jenkinson

A system and method for executing transactions where neither party can deny their involvement. A transaction manager acts as a neutral intermediary, exchanging digitally signed tokens between clients and resource managers at four key stages: work request origin, work receipt, completion request, and completion receipt. This balanced exchange of cryptographic evidence prevents either party from gaining unfair advantage, enabling secure distributed transactions across multiple systems.

[View on Google Patents](https://patents.google.com/patent/US11150938B2/en)

---

### US10901776B2 - Efficient and Scalable Transaction Processing Using a Consensus-based Transaction Model
**Filed:** December 4, 2017
**Granted:** January 26, 2021
**Inventors:** Thomas J. Jenkinson, Paul F. Robinson

A transaction processing system enabling atomic updates across multiple heterogeneous resources like databases. The system employs a consensus protocol where a transaction processor checks each resource's status and performs necessary actions to move all subtransactions to committed status. The innovation maintains state within resources rather than the transaction manager, eliminating the need for state persistence and enabling horizontal scalability, fault tolerance, and clustering without requiring leader election after failures.

[View on Google Patents](https://patents.google.com/patent/US10901776B2/en)

---

### US10663835B2 - Dynamic Privacy Glasses
**Filed:** January 26, 2018
**Granted:** May 26, 2020
**Inventors:** Paul Fletcher Robinson, Thomas John Jenkinson

A system enabling secure display viewing through synchronized eyewear and screen technology. The display emits light at multiple polarization angles—some carrying protected content and others emitting noise across various angles. Paired glasses contain filters that selectively pass light at the content's polarization while blocking noise wavelengths. The system dynamically changes polarization factors throughout a viewing session, preventing unauthorized viewers from accessing displayed information.

[View on Google Patents](https://patents.google.com/patent/US10663835B2/en)

---

### US10181025B2 - Non-repudiation of Broadcast Messaging
**Filed:** January 19, 2018
**Granted:** January 15, 2019
**Inventors:** Tom Jenkinson, Paul Robinson

A system enabling secure message exchange in broadcast scenarios. A computing system acts as a trusted intermediary, receiving messages from publishers along with authentication evidence, then distributing those messages to multiple subscribers. The system time-stamps and stores proof from both senders and receivers, preventing either party from later denying they sent or received communications.

[View on Google Patents](https://patents.google.com/patent/US10181025B2/en)

---

### US10048983B2 - Systems and Methods for Enlisting Single Phase Commit Resources in a Two Phase Commit Transaction
**Filed:** April 2, 2014
**Granted:** August 14, 2018
**Inventors:** Tom Jenkinson, Michael Musgrove, Paul Robinson, Jonathan Halliday, Jesper Pedersen, Mark Little

Addresses a technical challenge in distributed transaction processing where single-phase commit (1PC) and two-phase commit (2PC) resources typically cannot coexist in the same transaction. The innovation assigns unique identifiers to 1PC resource managers within 2PC transactions, enabling proper recovery procedures: if system failures occur, a recovery manager can verify whether 1PC operations completed by checking for the identifier, then appropriately commit or rollback the corresponding 2PC operations.

[View on Google Patents](https://patents.google.com/patent/US10048983B2/en)

---

### US9886573B2 - Non-repudiation of Broadcast Messaging
**Filed:** August 6, 2015
**Granted:** February 6, 2018
**Inventors:** Tom Jenkinson, Paul Robinson

A computing system method that receives messages and authentication evidence from publishing entities, time-stamps and stores this evidence, then broadcasts messages to subscribing entities. The system uses a trusted third party (message broker) to manage non-repudiation evidence in publish-subscribe messaging, specifically implementing this within Java Message Service (JMS) architecture.

[View on Google Patents](https://patents.google.com/patent/US9886573B2/en)

---

### US9858312B2 - Transaction Compensation for Single Phase Resources
**Filed:** October 14, 2014
**Granted:** January 2, 2018
**Inventors:** Paul Fletcher Robinson, Thomas John Jenkinson

A method for managing distributed transactions across multiple data stores using a one-phase commit protocol with compensation data. Rather than employing traditional two-phase commit protocols that lock resources, the innovation writes additional data into data items with data item updates that enable undoing changes. The system operates at the middleware layer, allowing updates to multiple data items across heterogeneous databases—including NoSQL and relational systems—while maintaining atomicity guarantees.

[View on Google Patents](https://patents.google.com/patent/US9858312B2/en)

---

### US9654294B2 - Non-repudiable Atomic Commit
**Filed:** February 26, 2015
**Granted:** May 16, 2017
**Inventors:** Thomas John Jenkinson, Paul Fletcher Robinson

A system enabling distributed transactions where parties exchange digitally signed tokens to prevent denial of actions. A client initiates a transaction request containing a cryptographic token to a transaction manager, who coordinates work across multiple resource managers. Each participant generates and exchanges tokens at different transaction stages, with the manager acting as a trusted intermediary logging all tokens. This approach allows parties to prove their involvement and instructions throughout the transaction lifecycle.

[View on Google Patents](https://patents.google.com/patent/US9654294B2/en)

---

*All patents listed above were assigned to Red Hat, Inc. and relate to computing systems, distributed transactions, messaging technologies, and microservices.*