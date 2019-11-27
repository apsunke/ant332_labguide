+++
title = "Data pipeline 2: Fluentd and Fluentbit"
date = 2019-11-26T20:10:38-08:00
weight = 9
chapter = true
pre = "<b>5. </b>"
+++

Similarly to the previous pipeline, Fluentd is another popular open-source alternative to collect metrics and logs. Fluentbit is part of the same family of product and acts as a more light-weight forwarder.

#### Fluentbit (metrics) -> Fluentd -> Amazon ES
In this case, we will use Fluentbit, a light-weight agent to collect metrics from Redis and write into Fluentd. Fluentd will parse the data and index into Amazon ES

#### Fluentd -> Amazon ES
Fluentd can act as both the data collector and the data parser. We will define various log inputs to Fluentd.
