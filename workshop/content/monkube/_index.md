+++
title = "Monitoring Kubernetes"
date = 2019-11-26T20:07:58-08:00
weight = 7
chapter = true
pre = "<b>3. </b>"
+++

To get full visibility into your applications we need two key elements, metrics and logs for the following aspect.

#### Infrastructure level
This level will have data pertaining to the worker nodes. For example metrics such as CPU percentage, memory used and also system logs.

#### Kubernetes level
This level involves everything at the Kubernetes level including pods, containers, error logs and much more.

In this lab we will collect data from both these levels. There's multilple ways to capture and process this data. Lets take a look at two most popular way our customers love to do it.
