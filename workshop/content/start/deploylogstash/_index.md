+++
title = "Deploying Logstash"
date = 2019-11-26T19:44:40-08:00
weight = 17
+++

Logstash is an open source data collection engine with real-time pipelining capabilities. Logstash can dynamically unify data from disparate sources and normalize the data into destinations of your choice. Cleanse and democratize all your data for diverse advanced downstream analytics and visualization use cases.

A Logstash pipeline has three key elements **inputs, filters and outputs**. The input plugins consume data from a source, the filter plugins modify the data as you specify, and the output plugins write the data to a destination.

In this lab we built the following pipeline to parse filebeat, metricbeat data and output to Amazon ES. This Logstash pipeline (snippet below) listens for logs from filebeat on the worker nodes and metricbeat on both workers nodes and pods, parses the incoming filebeat log message using a `grok` filter as a regex. Metricbeat logs don't contain log messages so we are not using any field parsing for them. If the incoming log contains tags called either `filebeat` or `metricbeat` it pushes the log to the appropriate index in Elasticsearch as output.

In a second filter plugin we do a timestamp match using the `date` filter plugin, which reconciles Elasticsearch's default `@timestamp` field against the actual timestamp in the log file. This is done for both filebeat and metricbeat logs.

We can make changes to this config based on the type of logging we're doing. Below is a snippet from the Logstash config file for your reference. Review the config and move on to the next step.
```
    input {
      beats {
        port => 5044
      }
    }

    filter {
      if "filebeat" in [tags] {
        grok {
            match => {
                "message" => '(?<timestamp>\S+\s\d+\s\S+)\s(?<hostname>\S+)\s(?<log_message>.*)'
            }
            add_tag => [ "parsed_msg" ]
        }
        date {
          match => ["timestamp", "MMM dd HH:mm:ss"]
          target => "@timestamp"
          add_tag => [ "parsed_time" ]
        }
      }
      else if "metricbeat" in [tags] {
        date {
          match => ["timestamp", "MMM dd HH:mm:ss"]
          target => "@timestamp"
          add_tag => [ "parsed_time" ]
        }
      }
    }

    output {
      if "filebeat" in [tags] {
        elasticsearch {
          hosts => ["${ES_DOMAIN}"]
          index => "filebeat-%{+yyyy.MM.dd.HH}"
        }
      }
      else if "metricbeat" in [tags] {
        elasticsearch {
          hosts => ["${ES_DOMAIN}"]
          index => "metricbeat-%{+yyyy.MM.dd.HH}"
        }
      }
    }
```

Navigate to the `logstash-manifests` directory under `ant332/final-deploy` (this should be your current working directory) folder by using the following command:
```
cd /home/ec2-user/environment/ant332/final-deploy/logstash-manifests
chmod +x deploy-logstash.sh
```

First we need to update the `logstash-deployment.yml` with our Elasticsearch endpoint's address, for this run the following command.

Replace the `<OutputFromNestedESStack>` with the Elasticsearch endpoint the Cloud9 setup step. Please ensure that the URL has `https` followed by `443` port number.

```
./deploy-logstash.sh https://<OutputFromNestedESStack>:443
```

***HELPER:*** Incase you make a mistake during these deployment steps, refer to the [Logstash Troubleshooting](http://eseksworkshop.com/en/appendix.html#logstash-troubleshooting) section in this lab's [Appendix](http://eseksworkshop.com/en/appendix.html).

Move on to the next step. Note down the Logstash IP address, going forward we will refer to this as `logsash_ip_address`.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/logstash-ip.png)

Incase the aforementioned command did not provide the logstash IP, usually due to a small delay in logstash becoming available, run the following command:

```
kubectl -n logstash get pod -oyaml | yq r - 'items[0].status.podIP'
```

You can view the pod's readiness state by running the following command. You will notice logstash pods in running state.

```
kubectl -n logstash get pods
```
