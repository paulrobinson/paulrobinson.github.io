---
layout: :theme/post
title: "Compensating Transactions: When ACID is too much (Part 1: Introduction)"
description: This is my first article ever made with Quarkus Roq
tags: transactions
author: paul
---

ACID transactions are a useful tool for application developers and can provide very strong guarantees, even in the presence of failures. 
However, ACID transactions are not always appropriate for every situation.
In this series of blog posts.
I'll present several such scenarios and show how an alternative non-ACID transaction model can be used.

The isolation property of an ACID transaction is typically achieved through [optimistic or pessimistic concurrency control](http://en.wikipedia.org/wiki/Lock_(computer_science)).
Both approaches can impose negative impacts on certain classes of applications, if the duration of the transaction exceeds a few seconds (see [here](http://www.theserverside.com/news/1365143/ACID-is-Good-Take-it-in-Short-Doses) for a good explanation).
This can frequently be the case for transactions involving slow participants (humans, for example) or those distributed over high latency networks (such as the Internet).
Also, some actions cannot simply be rolled back; such as, the sending of an email or the invocation of some third-party service.