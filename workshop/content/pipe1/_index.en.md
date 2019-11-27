+++
title = "Data pipeline 1:  Beats and Logstash"
date = 2019-11-26T20:10:33-08:00
weight = 8
chapter = true
pre = "<b>4. </b>"
+++

In this lab will discuss two different pipelines involving different technologies to collect the metrics. You can use either in production depending on your needs.

Beats is a family of popular open-source data collection agents. Logstash is a popular open-source,server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then pushes the result to various destinations. Logstash acts at the parsing and buffering layer de-coupling the source and the destination. For this lab, we will deploy multiple Logstash instances in an Auto Scaling Group that scale based on resource needs.

#### Metricbeat -> Logstash -> Amazon ES
 In this case we will use metricbeat to collect system and Kubernetes metrics and send it out to Logstash.

#### Filebeat -> Logstash -> Amazon ES
Filebeat is a light-weight agent that can tail log files and push the data to Logstash.
