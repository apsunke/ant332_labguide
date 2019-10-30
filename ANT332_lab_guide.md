# ANT332 Use Amazon ES to visualize and monitor containerized applications

In this lab we will learn to capture logs and monitor metrics from Amazon Elastic Kubernetes Service (Amazon EKS) using a variety of collection agents and analyzing them using Amazon Elasticsearch service (Amazon ES).


![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/Overall+architecture.png "Logo Title Text 1")

# What is Amazon Elasticsearch?
Amazon Elasticsearch Service is a fully managed service that makes it easy for you to deploy, secure, and operate Elasticsearch at scale with zero down time. The service offers open-source Elasticsearch APIs, managed Kibana, and integrations with Logstash and other AWS Services, enabling you to securely ingest data from any source and search, analyze, and visualize it in real time. Amazon Elasticsearch Service lets you pay only for what you use – there are no upfront costs or usage requirements. With Amazon Elasticsearch Service, you get the ELK stack you need, without the operational overhead.

![alt text](
https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/How+AES+works_final.2e3ac88fbb9910d7c401d0748556db0c91c97b33.png)

# What is Amazon EKS?
Amazon Elastic Kubernetes Service (Amazon EKS) makes it easy to deploy, manage, and scale containerized applications using Kubernetes on AWS.

Amazon EKS runs the Kubernetes management infrastructure for you across multiple AWS availability zones to eliminate a single point of failure. Amazon EKS is certified Kubernetes conformant so you can use existing tooling and plugins from partners and the Kubernetes community. Applications running on any standard Kubernetes environment are fully compatible and can be easily migrated to Amazon EKS.

Amazon EKS supports both Windows Containers and Linux Containers to enable all your use cases and workloads.


![alt text](
https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/product-page-diagram-AmazonEKS-v2.dd41321fd3aa0915b93396c13e739351d2160ba8.png)


# Kubernetes 101
@saadf

# Monitoring Kubernetes

To get full visibility into your applications we need two key elements, metrics and logs for the following aspect.

#### Infrastructure level
This level will have data pertaining to the worker nodes. For example metrics such as CPU percentage, memory used and also system logs.

#### Kubernetes level
This level involves everything at the Kubernetes level including pods, containers, error logs and much more.

In this lab we will collect data from both these levels. There's multilple ways to capture and process this data. Lets take a look at two most popular way our customers love to do it.

# Data pipline using Beats and Logstsah
In this lab will discuss two different pipelines involving different technologies to collect the metrics. You can use either in production depending on your needs.

Beats is a family of popular open-source data collection agents. Logstash is a popular open-source,server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then pushes the result to various destinations. Logstash acts at the parsing and buffering layer de-coupling the source and the destination. For this lab, we will deploy multiple Logstash instances in an Auto Scaling Group that scale based on resource needs.

#### Metricbeat -> Logstash -> Amazon ES
 In this case we will use metricbeat to collect system and Kubernetes metrics and send it out to Logstash.

#### Filebeat -> Logstash -> Amazon ES
Filebeat is a light-weight agent that can tail log files and push the data to Logstash.

# Data pipline using Fluentd and Fluentbit

Similarly to the previous pipeline, Fluentd is another popular open-source alternative to collect metrics and logs. Fluentbit is part of the same family of product and acts as a more light-weight forwarder.

#### Fluentbit (metrics) -> Fluentd -> Amazon ES
In this case, we will use Fluentbit, a light-weight agent to collect metrics from Redis and write into Fluentd. Fluentd will parse the data and index into Amazon ES

#### Fluentd -> Amazon ES
Fluentd can act as both the data collector and the data parser. We will define various log inputs to Fluentd.


# Let the lab begin!
@Saad
In order to save time, we have already pre-deployed the environment for you. You account will have the following resources

* AWS Cloud9 IDE environment (this is where you'll spend the most time for this lab)
* Amazon ES domain
* Amazon EKS Cluster, API server, worker nodes
* Bastion host (Amazon EC2 instance) with SSH key

# What we will be monitoring today
We will use a simple 'Guestbook' application, which is a multi-tier web application using Kubernetes and Docker. This example consists of a guestbook application with the following components:

* A single-instance Redis master to store guestbook entries
* Multiple replicated Redis instances to serve reads
* Multiple web frontend instances



![alt text](
https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/guestbook_architecture.png)
# Getting familiar with today's lab environment
For the lab today, you will primarily use two interfaces - AWS Cloud9 and Kibana. We recommend keeping both these open in your broswer as there will significant forth.

**AWS Cloud9** is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. We will run all our commands from here.

**Kibana** is an open source data visualization plugin for Elasticsearch. It provides visualization capabilities on top of the content indexed on an Elasticsearch cluster. We will use Kibana to visualize all the logs and metrics.

# Configure Cloud9 IDE
The owner of a Cloud9 EC2 environment can turn on or off AWS managed temporary credentials for that environment at any time, as follows:

* With the environment open, in the AWS Cloud9 IDE, on the menu bar choose AWS Cloud9, Preferences.

* In the Preferences tab, in the navigation pane, choose AWS Settings, Credentials.

* Use AWS managed temporary credentials to turn AWS managed temporary credentials on or off.

This step is required in order for the workshop to proceed.

Run the following kubectl

**Gain EKS Cluster Access Using kubectl and AWS CLI**

Use following command to enter the folder containing our k8s manifests:
```
cd ant332/final-deploy
```
Once here run the following to allow execution on our loadcreds.sh script and then run it to load EKS credentials into your Cloud9 environment:
```
chmod +x loadcreds.sh
source ./loadcreds.sh <CFN_ACCESS_KEY_ID_OUT> <CFN_SECRET_ACCESS_KEY_OUT>
```

Now we will run the bootstrapping for `kubectl` and `aws-iam-authenticator` using the `startup.sh` script with the following commands:

```
chmod +x startup.sh
source ./startup.sh
```

Now run the following command to ensure you have access to the EKS cluster:
```
kubectl get ns
```

If you have access the output from this command should display the following namespaces:

```
TeamRole:~/environment/ant332/final-deploy (master) $ kubectl get ns
NAME              STATUS   AGE
default           Active   12d
kube-node-lease   Active   12d
kube-public       Active   12d
kube-system       Active   12d
```

From this same working directory, navigate and change permissions for the `guestbook` guestbook deployment under `ant332/final-deploy` using the following commands:

```
cd guestbook
chmod +x deploy.sh
```

You can now deploy the official K8s `guestbook` application using the following commands:
```
./deploy.sh
```

Use the following command to see if all pods within the deployment come up correctly:
```
kubectl -n guestbook get pods
```

*TBD you should see the output below*
@saad

# Configure Kibana

Amazon ES supports **public access** domains, which can receive requests from any internet-connected device, and **VPC access** domains, which are isolated from the public internet and hence highly recommended.

In this lab we have deployed Amazon ES with **VPC access** as its more secure. This endpoint is only visible to the servers inside that same VPC. To access Kibana from your laptop we have to create a SSH tunnel from your laptop to a *Bastion host* running inside the same VPC. Once you setup this up, you can access Kibana directly from your local broswer on your laptop.

To eshtablish an SSH tunnel, you need the following three key details. We will be using these details throughout the workshop, we **highly recommend** you make a note of it and keep it handy.

**Step 1: Public IP address of the Bastion host and Amazon ES endpoint URL**

You can find this under Services -> Cloudformation -> Click on the bottom most stack -> go to Outputs -> Copy IP address next to ```OutputFromNestedBastionStack``` -> Copy Amazon ES endpoint under ```OutputFromNestedNetworkStack```

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion-step1.png)

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion-step2.png)

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion-step3.png)

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion-step4.png)


**Step 2: SSH key file for Bastion host**

**Step 3: Establish SSH tunnel**

@saad windows and mac laptop
*TBD Instructions for SSH tunnel*

**Step 4: Connect to Kibana with your local browser**

Click on this deep link to open Kibana in local browser on your laptop and on the welcome screen click **Explore on my own**

http://localhost:9200/_plugin/kibana

You will will taken to the Kibana home page with many controls and dashboards. Kibana does not have any data yet, so lets switch back to Cloud9 to start emitting data to Amazon ES.

# Deploying Logstash
*TBD @saad steps to deploy logstash and config files*

Logstash is an open source data collection engine with real-time pipelining capabilities. Logstash can dynamically unify data from disparate sources and normalize the data into destinations of your choice. Cleanse and democratize all your data for diverse advanced downstream analytics and visualization use cases.

A Logstash pipeline has three key elements **inputs, filters and outputs**. The input plugins consume data from a source, the filter plugins modify the data as you specify, and the output plugins write the data to a destination.

In this lab we built the following pipeline to parse *TBD log* and output to Amazon ES.
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
    }


    output {
      if "filebeat" in [tags] {
        elasticsearch {
          hosts => ["${ES_DOMAIN}"]
          index => "filebeat.%{+yyyy.MM.dd.HH}"
        }
      }
    }
```

This Logstash pipeline listens for logs from filebeat on the worker nodes, parses the incoming log message using a `grok` filter using a regex if the incoming log contains a tag called `filebeat` and then pushes the log as output to our Elasticsearch cluster.

In a second filter plugin we do a timestamp match using the `date` filter plugin, which reconciles Elasticsearch's default `@timestamp` field against the actual timestamp in the log file.

We can make changes to this config based on the type of logging we're doing.

@Anoop - decide on whether we're doing an exercise on editing the logstash config.

Navigate to the `logstash-manifests` directory under `ant332/final-deploy` (this should be your current working directory) folder by using the following command:
```
cd logstash-manifests
```

First we need to update the `logstash-deployment.yml` with our Elasticsearch endpoint's address, for this run the following command:
```
chmod +x deploy-logstash.sh
./deploy-logstash.sh https://<ELASTICSEARCH_DOMAIN_ENDPOINT_CFN_OUT>:443
```

We're only using a single logstash pod for this workshop and this script will output it's pod IP address, which we need to use with our filebeat deployment later so save it.

You can view the pod's readiness state by using the following command to tail the container's logs:
```
kubectl -n logstash get pods
```

Will give you the pod's name, which you can use with the following command to see if the pipeline started listening on port 5044 for beats input:
```
kubectl -n logstash logs -f <LOGSTASH_POD_NAME>
```
Once the pipeline is ready, we can move to deploying our beats platforms.

# Filebeat 101
* *Send data to logstahs for parsing*
* *Review logs via Kibana*

*@Saad*

Filebeat is a lightweight shipper for forwarding and centralizing log data. Installed as an agent on your servers, Filebeat monitors the log files or locations that you specify, collects log events, and forwards them to either to Elasticsearch or Logstash for indexing.

In this lab we will configure Filebeat to collect *TBD log file* and send the data to logstash.

```
filebeat.inputs:
    - type: log
      symlinks: true
      paths:
        - /var/log/containers/*.log
      processors:
        - add_kubernetes_metadata:
            in_cluster: true
            host: ${NODE_NAME}
            matchers:
            - logs_path:
                logs_path: "/var/log/containers/"s
```

# Metricbeat 101
*TBD review dashboard, deployment steps*

Metricbeat is a lightweight open-source agent to collect system and service statistics.
Metricbeat consists of modules and metricsets. A Metricbeat module defines the basic logic for collecting data from a specific service, such as Redis, MySQL, and so on. The module specifies details about the service, including how to connect, how often to collect metrics, and which metrics to collect.

In this lab we enable **system** and **kubernetes** modules. For more information open the metricbeat.yaml file.

```
- module: system
     period: 10s
     metricsets:
       - cpu
       - load
       - memory
       ....
- module: kubernetes
      metricsets:
        - node
        - system
        - pod
        ....
```


Note: Metricbeat gets its key metrics from a internal kubernetes service called **kube-state-metrics** It is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects. It is focused on the health of the various objects inside, such as deployments, nodes and pods.

#### Install kube-state-metrics
**Run** the following command to install kube-state-metricsets

```
kubectl apply -f kube-state-metrics-cluster-role-binding.yaml &&
kubectl apply -f kube-state-metrics-service-account.yaml &&
kubectl apply -f kube-state-metrics-cluster-role.yaml &&
kubectl apply -f kube-state-metrics-service.yaml &&
kubectl apply -f kube-state-metrics-deployment.yaml
```

If its succesfully created, you should see *createad* messages like the ones below
```clusterrolebinding.rbac.authorization.k8s.io/kube-state-metrics created
serviceaccount/kube-state-metrics created
clusterrole.rbac.authorization.k8s.io/kube-state-metrics created
service/kube-state-metrics created
deployment.apps/kube-state-metrics created
```
**Run** the following command to double-check if its successfully running. You should see ```kube-state-metrics``` listed in the result along with other internal deployments.

```
 kubectl get deployment -n kube-system
 ```

#### Deploy metricbeat
**Run** this command to deploy Metricbeat. Metricbeat is set to automatically load pre-built Kibana dashboards and also starts pushing data into Logstash.
```
kubectl apply -f /home/ec2-user/environment/ant332/metricbeat-deployment.yaml
```

# View Metricbeat data in Kibana
Every time you create a new index in Elasticsearch, you have to configure **Index Pattern** in Kibana. This allows Kibana to be aware of the index and and lets you start creating visualizations and dashboards.

Click this deep link to open Kibana in your browser and follow the steps below - http://localhost:9200/_plugin/kibana

Click Mangement tab -> Index Patterns -> Create Index Pattern -> ***metricbeat-**** -> ***@timestamp*** -> Create Index Pattern

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kibana-mb-step-1.png)

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kibana-mb-step-2.png)

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kibana-mb-step-3.png)


Congratulations! You've configured you're first end-to-end pipeline. You will now be able to see all the metricbeat data in real-time.

***Pro tip:*** On the *top right corner* of Kibana you will find two very useful settings. **Auto-refresh** controls how frequent your dashboard needs to refresh. We recommend setting it to 30 seconds. The next setting labled **last 15 minutes** controls the time-range of the data Kibana will display. When set to 'last 15 minutes', you will only see the most recent 15 minutes of data. You can leave this at the default setting.

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kibana-pro-tip.png)


# Fluentd 101
*Walk through one Fluentd parsing configuration TBD*

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
# What is Fluentbit and how is it different?
*TBD walk-throgh fluentbit deployment + sidecar container*

Fluent Bit is an open source and multi-platform Log Processor and Forwarder which allows you to collect data/logs from different sources, unify and send them to multiple destinations. It's fully compatible with Docker and Kubernetes environments.

Both Fluentd and Fluent Bit share a lot of similarities, Fluent Bit is fully based on the design and experience of Fluentd architecture and general design. Choosing which one to use depends on the final needs, from an architecture perspective we can consider:
* Fluentd is a log collector, processor, and aggregator.
* Fluent Bit is a log collector and processor (it doesn't have strong aggregation features like Fluentd).

Consider Fluentd mainly as an Aggregator and Fluent Bit as a Log Forwarder, we can see both projects complement each other providing a full reliable solution.

# Fluentbit deployment as a 'Sidecar'
In this lab we'll use Fluentbit to capture **metrics** from our Redis deployments. All the agents we discussed so far in this guide were be deployed as a daemonset (a POD that runs on every node of the cluster). With Fluentbit, we're deploying it as a side-car container (for redis contianers) instead. With this approach, whenever redis contaner is deployed there is a dedicated Fluentbit side-car container attached to collect all the relevant data.

Lets take a look at the ```redis-slave.yml``` deployment file for Guestbook application. This ensures that  a Fluentbit container is spun up every time a redis container launches.
```
spec:
      containers:
      - name: slave
        image: gcr.io/google_samples/gb-redisslave:v1
        resources:
        ....
        ports:
        - containerPort: 6379

      - name: redis-statistic
        image: fluent/fluent-bit:0.14.6
        resources:
        ....
```
Once its launched, Fluentbit is configured to run a custom script (loaded into the Fluentbit container path /bin/redis) to capture metrics from redis. It then forwards the data to Fluentd using the forwad protocol that was pre-setup. You can find these configurations are in ```fluent-bit.conf```.

```
<source>
  @type exec
  tag   redis
  command bash /bin/redis
  type json
  run_interval 1m
</source>

<match **>
  @type stdout
</match>

#!/bin/bash

for i in $( (printf 'info\r\n';) | nc -w 1 redis 6379 | grep ':'); do
 key=$(echo "$i" | cut -d ":" -f1 )
 value=$(echo "$i" | cut -d ":" -f2 | sed 's/\r$//g' )
 echo "{\"$key\": \"$value\"}"
done
```




# Deploying Fluentd and Fluentbit

Lets configure Fluentd to output to your Amazon ES domain. Copy your Amazon ES endpoint from the 'Configure Kibana' step. **Run** the following command by **modifying the url with your Amazon ES endpoint**

```
export URL2=vpc-aes-lab-your-amazon-es-endpoint-url
```

A popular method to deploy complex applications in Kubernetes is through **Helm**. Helm is a tool that streamlines installing and managing Kubernetes applications. Think of it like apt/yum/homebrew for Kubernetes. We  built a helm chart that deploys both Fluentbit and Fluentd with a single command.

**Run** this command to deploy.
```
helm upgrade -i --namespace=logging --set elasticsearch.host=$URL2 fluentd-elasticsearch fluentd-elasticsearch
```
**Run** this command and verify if the deployment was successful. You should see ```fluentd-elasticsearch``` is deployed.

```
helm ls
```

Fluentd will instantly start pushing logs into Amazon ES, next step is to configure Kibana.  In this lab, Fluentd is configured to capture all the logs in kubernetes. However to keep the data separate, it is configured to write to 3 different indices in Amazon ES. Understanding what data is in which index will be **important** for the last part of this workshop.

**apache-** This index holds logs from the apache server in 'Guestbook' application

**redis-** This index holds all redis metrics that Fluentbit is collecting

**kubernetes-** This index holds all the other logs


# Configuring Index Patterns in Kibana
Every time you create a new index in Elasticsearch, you have to configure **Index Pattern** in Kibana. This allows Kibana to be aware of the index and and lets you start creating visualizations and dashboards. Follow the next steps to create Index patterns for all three indices - **kubernetes, redis, apache**


# Configure Kibana for kubernetes index
Click this deep link to open Kibana in your browser and follow the steps below - http://localhost:9200/_plugin/kibana

Click Mangement tab -> Index Patterns -> Create Index -> ***kubernetes-**** -> ***@timestamp*** -> Create Index Pattern

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step1.png)

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step2.png)

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step3.png)

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step4.png)

Kibana is now fully setup to start visualizing Kubernetes logs. You can go to the *Discover* tab on the left to start exploring the data.

# Configure Kibana for redis index
Repeat the previous step to create an index pattern for **redis-** index. Click this deep link to open Kibana in your browser and follow the steps below - http://localhost:9200/_plugin/kibana

Click Mangement tab -> Index Patterns -> Create Index -> ***redis-**** -> ***@timestamp*** -> Create Index Pattern
![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/redis-index.png)

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/redis-index-2.png)


# Configure Kibana for apache index
Repeat the previous step again to create an index pattern for **apache-** index. Click this deep link to open Kibana in your browser and follow the steps below - http://localhost:9200/_plugin/kibana
Click Mangement tab -> Index Patterns -> Create Index -> ***apache-**** -> ***@timestamp*** -> Create Index Pattern

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/apache-index.png)

Lets deploy the *Guestbook* application so we can start ingesting logs and metrics.

# Deploy Guestbook application

**Run** this command to deploy the *Guestbook* application. This will deploy frontend, redis master and redis slaves.

```
 kubectl apply -f guestbook/redis-master.yml &&
 kubectl apply -f guestbook/redis-master-service.yaml &&
 kubectl apply -f guestbook/redis-slave.yml &&
 kubectl apply -f guestbook/redis-slave-service.yaml &&
 kubectl apply -f guestbook/frontend-deployment.yaml &&
 kubectl apply -f guestbook/frontend-service.yaml
```

**Run** this command to verify Guestbook deployed successfully

```
kubectl get all -n guestbook
```
You should see all the pods, services and deplyments running similar to the example below

```
pod/frontend-69859f6796-f4nrp      1/1     Running   0          26m
pod/frontend-69859f6796-ktzhv      1/1     Running   0          26m
pod/frontend-69859f6796-l2s9m      1/1     Running   0          26m
pod/redis-master-596696dd4-8g6n9   1/1     Running   0          26m
pod/redis-slave-96685cfdb-9dv57    1/1     Running   0          26m
pod/redis-slave-96685cfdb-xgmbr    1/1     Running   0          26m
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
service/frontend       LoadBalancer   172.20.125.197   <pending>     443:31112/TCP   26m
service/redis-master   ClusterIP      172.20.195.144   <none>        6379/TCP        26m
service/redis-slave    ClusterIP      172.20.248.13    <none>        6379/TCP        26m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/frontend       3/3     3            3           26m
deployment.apps/redis-master   1/1     1            1           26m
deployment.apps/redis-slave    2/2     2            2           26m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/frontend-69859f6796      3         3         3       26m
replicaset.apps/redis-master-596696dd4   1         1         1       26m
replicaset.apps/redis-slave-96685cfdb    2         2         2       26m
```


# Explore in Kibana
Lets explore the log data in Kibana. Fluentd is configured to output all the logs into **kubernetes-*** index. Click this deep link to open Kibana in your browser and follow the steps below - http://localhost:9200/_plugin/kibana

Click Discover tab -> Choose ***kubernetes-**** from dropdown -> use search box to search for guestbook or any other term you're interested in

![Alt Text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/view-kuber-step-1.png)

![Alt Text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/view-kuber-step-2.png)

![Alt Text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/view-kuber-step-3.png)

You will now start seeing all the logs that Fluentd is pushing into Amazon ES. Feel free to search for any other terms your'e interested in such as

Bonus point: If you'd like this data to be more readable, you can create a custom view by choosing specific fields to be displayed. Click on the link below for a quick video on creating your own custom Kibana view.
https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kibana-advanced-view.gif

# Mystery hunt exercise
Great job setting the environment! You now have logs and metrics flowing in Amazon ES and you can visualize all of this data in Kibana.

The last portion of this workshop is called **Mystery Hunt**. We'll ask you to run mystery scripts that make changes to your environment, you'll then need to investigate and find root cause. We'll have hints along the way to guide you.

Kibana is all you need for this exercise, it has all the data including visualizations.
