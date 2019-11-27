+++
title = "What we are monitoring"
date = 2019-11-26T19:44:40-08:00
weight = 3
+++

We will use a simple 'Guestbook' application, which is a multi-tier web application using Kubernetes and Docker. This example consists of a guestbook application with the following components:

* A single-instance Redis master to store guestbook entries
* Multiple replicated Redis instances to serve reads
* Multiple web frontend instances

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/guestbook_architecture.png)
