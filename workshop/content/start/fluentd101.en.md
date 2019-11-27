+++
title = "Fluentd 101"
date = 2019-11-26T19:44:40-08:00
weight = 11
+++

Fluentd is a fully free and fully open-source log collector that instantly enables you to have a 'Log Everything' architecture with 600+ types of systems.Fluentd treats logs as JSON, a popular machine-readable format. It is written primarily in C with a thin-Ruby wrapper that gives users flexibility.

![alt](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/fluentd-architecture.png)

Fluentd has the following key components - Setup, Inputs, Filters, Parsers and Outputs. You can find the these configurations under ```values.yml``` file under ```fluentd-elasticsearch``` folder. We use Fluentd to collect and process multiple sources of logs such as apache logs, kubelet, kube-api-server and more.

#### Setup
The configuration files is the fundamental piece to connect all things together, as it allows to define which Inputs or listeners Fluentd will have and set up common matching rules to route the Event data to a specific Output.

#### Input
Extend Fluentd to retrieve and pull event logs from external sources. An input plugin typically creates a thread socket and a listen socket. It can also be written to periodically pull data from data sources.

In this snippet below the input plugin is configured to tail and collect Apache access logs that are in json format.
```
<source>
  @type tail
  path /var/log/containers/*frontend*.log
  pos_file /var/log/apache-access.log.pos
  format json
  tag apache.access
  ....
</source>
```

#### Parser
Fluentd has a pluggable system that enables the user to create their own parser formats. In the code snippet below, we are using the out-of-the-box ```apache2``` parser to parse the access logs. To parse custom logs, you can leverage ```regexp``` to support regex parsing.

```
  <filter apache.access>
     @type parser
     key_name log
     format apache2
     ....
   </filter>
```

#### Output
This plugin allows you write data to various output destinations.
In the code snippet below the output is configured to Elasticsearch endpoint and port. With this ```logstash_format``` set true, Fluentd uses the conventional index name format logstash-%Y.%m.%d.

Fluentd can also buffer data, in this case we are leverage disk based persistent buffering set by ```@type file``` setting.
```
<match apache.**>
      @type elasticsearch
      host "#{ENV['OUTPUT_HOST']}"
      port "#{ENV['OUTPUT_PORT']}"
      logstash_prefix apache
      logstash_format true
      <buffer>
        @type file
        path /var/log/fluentd-buffers/apache.system.buffer
        flush_mode interval
        ....
      </buffer>
    </match>
```
