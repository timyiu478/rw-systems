---
tags:
  - computer-network
  - distributed-systems
  - layered-systems
reference: obsidian://open?vault=systems&file=papers%2Fendtoend.pdf
title: END-TO-END ARGUMENTS IN SYSTEM DESIGN by J.H. Saltzer, D.P. Reed and D.D. Clark
draft: false
description: what network functions should be assigned to the endpoints
---

***Q. Of all of the networking functions you've learned about, what functions belong at what layers?***

* Application Later: Detecting web server crashes
* Transport Layer Security: Secure (encryption & authentication) data tranmission
* Transport Layer: Message FIFO Delivery, Duplicate Message Suppression, At Most Once Message Delivery(Timeout & Retry & Acknowledgement)
* Networking Layer: Routing for end-to-end data transfer
* Link Layer: hop-to-hop direct data transfer

***Q. How might one use the end-to-end argument when designing a system?***

May think about the following questions:

1. What are the needs/assumptions of the system?
2. What are the functions the system needs to provide? (encryption, ordering, deduplication, integrity, ...)
3. Can the function be implemented without the knowledge and the help of the application at the endpoint?
4. Should the lower layer (not) provide the function to optimize performance?

***Q. What does the end-to-end argument state?***

The function in question can completely and correctly be implemented only with the **knowledge** and help of the application standing at the endpoints of the communication system. Therefore, providing that questioned function as a feature of the communication system itself is not possible.

The lower levels need not provide "perfect" reliability.

The end-to-end argument does not tell us where to put the early (packet-error) checks for **performance** enhancement.
* e.g. hop-by-hop packet retry


***Q. What role does trust play in the end-to-end argument?***

End-to-end encryption:
* **Trust Mode**l: only endpoints can trust =>
* Only endpoints know the keys (the knowledge for encryption) =>
* The communication system can't encrypt/decrypt messages

***Q. Where have you seen the impact of the end-to-end argument?***

* Consensus Protocols: application peer level's at-mostp delivery (e.g.retry, acknowledgement)
* Application Level Load Balancer: detects web server crashes using HTTP requests
* QUIC Protocol
* E2E Encryption Messaging Apps
* KV store: 
	* client: retry
		* Why retry? The server can restart after it receives a full request from TCP layer
	* server: deduplication for append operation
		* only the server know the client request ID
