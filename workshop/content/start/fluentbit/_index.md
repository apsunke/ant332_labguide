+++
title = "What is Fluentbit and how is it different?"
date = 2019-11-26T19:44:40-08:00
weight = 21
+++

Fluent Bit is an open source and multi-platform Log Processor and Forwarder which allows you to collect data/logs from different sources, unify and send them to multiple destinations. It's fully compatible with Docker and Kubernetes environments.

Both Fluentd and Fluent Bit share a lot of similarities, Fluent Bit is fully based on the design and experience of Fluentd architecture and general design. Choosing which one to use depends on the final needs, from an architecture perspective we can consider:

* Fluentd is a log collector, processor, and aggregator.
* Fluent Bit is a log collector and processor (it doesn't have strong aggregation features like Fluentd).

Consider Fluentd mainly as an Aggregator and Fluent Bit as a Log Forwarder, we can see both projects complement each other providing a full reliable solution.
