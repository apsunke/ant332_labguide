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
Kubernetes is an open-source container orchestration and scheduling platform. It's become very popular for web-scale deployments in recent years, as it reduces the operational complexity and time involved in deploying highly scalable infrastructure for service-oriented architectures.

We will be using the Amazon EKS service for deploying our Kubernetes clusters for the course of this workshop.

We will be using the `kubectl` CLI tool for interacting with our EKS cluster's API server. The following cheatsheet describes some common commands to use with `kubectl`:
<Insert Cheatsheet Images here>

# Monitoring Kubernetes

To get full visibility into your applications we need two key elements, metrics and logs for the following aspect.

#### Infrastructure level
This level will have data pertaining to the worker nodes. For example metrics such as CPU percentage, memory used and also system logs.

#### Kubernetes level
This level involves everything at the Kubernetes level including pods, containers, error logs and much more.

In this lab we will collect data from both these levels. There's multilple ways to capture and process this data. Lets take a look at two most popular way our customers love to do it.

# Data pipeline 1:  Beats and Logstash
In this lab will discuss two different pipelines involving different technologies to collect the metrics. You can use either in production depending on your needs.

Beats is a family of popular open-source data collection agents. Logstash is a popular open-source,server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then pushes the result to various destinations. Logstash acts at the parsing and buffering layer de-coupling the source and the destination. For this lab, we will deploy multiple Logstash instances in an Auto Scaling Group that scale based on resource needs.

#### Metricbeat -> Logstash -> Amazon ES
 In this case we will use metricbeat to collect system and Kubernetes metrics and send it out to Logstash.

#### Filebeat -> Logstash -> Amazon ES
Filebeat is a light-weight agent that can tail log files and push the data to Logstash.

# Data pipeline 2: Fluentd and Fluentbit

Similarly to the previous pipeline, Fluentd is another popular open-source alternative to collect metrics and logs. Fluentbit is part of the same family of product and acts as a more light-weight forwarder.

#### Fluentbit (metrics) -> Fluentd -> Amazon ES
In this case, we will use Fluentbit, a light-weight agent to collect metrics from Redis and write into Fluentd. Fluentd will parse the data and index into Amazon ES

#### Fluentd -> Amazon ES
Fluentd can act as both the data collector and the data parser. We will define various log inputs to Fluentd.


# Let the lab begin!
In order to save time, we have already pre-deployed the environment for you. You account will have the following resources

* AWS Cloud9 IDE environment (this is where you'll spend the most time for this lab)
* Amazon ES domain
* Amazon EKS Cluster, API server, worker nodes
* Bastion host (Amazon EC2 instance) with SSH key

Follow the instructions below to note down information such as access key, IP address and Amazon Elasticsearch endpoints. We'll refer to these throught the workshop by their key name, for example ```OutputFromNestedESStack``` will refer to your Amazon ES endpoint.

Click on 'Services' tab and type 'Cloudformation' in the search box.

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cfn-1.png)

Now click on the bottom most stack on your screen. The name of your stack can be different from the one in the screenshot.

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cfn-2.png)

Click on the 'Outputs' tab and copy all the values highlighted in the screenshot. We recommend copying these into a notepad as we refer to these multiple times throught the lab.

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cfn-3.png)


# What we will be monitoring today
We will use a simple 'Guestbook' application, which is a multi-tier web application using Kubernetes and Docker. This example consists of a guestbook application with the following components:

* A single-instance Redis master to store guestbook entries
* Multiple replicated Redis instances to serve reads
* Multiple web frontend instances

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/guestbook_architecture.png)

# Get familiar with lab environment
For the lab today, you will primarily use two interfaces - AWS Cloud9 and Kibana. We recommend keeping both these open in your broswer as there will significant forth.

**AWS Cloud9** is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. We will run all our commands from here.

**Kibana** is an open source data visualization plugin for Elasticsearch. It provides visualization capabilities on top of the content indexed on an Elasticsearch cluster. We will use Kibana to visualize all the logs and metrics.


# Configure Cloud9 IDE

**AWS Cloud9** is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. It provides an interface to the deployed assets for this workshop, that include a Kubernetes cluster and an AWS Elasticsearch domain. We will run all our commands for this workshop from here.

Lets setup Cloud9 and get started. Cloud9 normally manages IAM credentials dynamically. This isn’t currently compatible with the EKS IAM authentication, so we will disable it and rely on the IAM role instead.**This step is a required step for the rest of the workshop.**

* With the environment open, in the AWS Cloud9 IDE, on the menu bar choose Cloud9 as shown:
![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9-1.png)

* Open the deployed IDE console by clicking the "Open IDE" button as shown:
![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9-2.png)

* When it comes up, customize the environment by closing the **welcome tab** and **lower work** area


![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9+close+tab.png)

Open a new **terminal** tab in the main work area.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9+open+terminal.png)

* Your workspace should now look like this. You will execute all commands from this terminal.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9+end+result.png)

* Click the the Preferences tab from the "AWS Cloud9" dropdown button, in the navigation pane as shown:
![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9-3.png)

* From the resulting menu choose AWS Settings --> Credentials and set the toggle button for "AWS managed temporary credentials" to disabled as shown:
![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9-4.png)

* Now clean up your Cloud9 IDE Console windows to bring up just a terminal window in the middle and file explorer to the left as shown:
<Insert gif resizing terminal window and closing extra cloud9 windows>

**Gain EKS Cluster Access Using kubectl and AWS CLI**

With a bash terminal open in the lower window of your Cloud9 IDE, use the following command to enter the folder containing our k8s manifests:
```
cd /home/ec2-user/environment/ant332/final-deploy
```
Once here run the following to allow execution on our loadcreds.sh script and then run it to load EKS credentials into your Cloud9 environment. Replace ```<OutputCloud9AdminAccessKeyID>``` and ```<OutputCloud9AdminSecretAccessKey>``` with corresponding values from your environment.
```
chmod +x loadcreds.sh
source ./loadcreds.sh <OutputCloud9AdminAccessKeyID> <OutputCloud9AdminSecretAccessKey>
```

Now we will run the bootstrapping for `kubectl` and `aws-iam-authenticator` using the `startup.sh` script with the following commands.

**Note:** This command will output several lines indicating current status. Thats part of the workflow and there's no need to worry. Move on to the next step after when the prompt is ready.

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

# Deploy guestbook application

**Run** the following command to setup and delpoy guestbook application deployment.

```
cd guestbook
chmod +x deploy.sh
./deploy.sh
```

Use the following command to see if all pods within the deployment come up correctly:
```
kubectl -n guestbook get pods
```

Lets access the guestbook application and submit a few entries. Guestbook in configured to deploy an Elastic Load Balancer (ELB) to handle incoming traffic.  **Run** the following command to get the ```<Loadbalancer Ingress>``` DNS address for the guestbook app.

```
kubectl get svc frontend -n guestbook -oyaml | yq r - "status.loadBalancer.ingress[0].hostname"
```

Output should look like this:

```
a3adf8asjkdhaj1231f6e67d0c-30d08edda21d396f.elb.us-east-1.amazonaws.com
```

Lets load this ELB address ```<Loadbalancer Ingress>``` into an environment variable for future use.

```
export URL3=<Loadbalancer Ingress>
```

Now open a web broswer like Safari on your laptop and paste ```Loadbalancer Ingress```. It will take you to the guestbook homepage.

**It may take a few minutes for the application to be ready. In case you don't get a response please wait 5 minutes and try again**

Once you see the Guestbook page appear, enter your 2 of your favorite guest names and click **submit**.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/broswer-guestbook.png)


# Configure Kibana

Amazon ES supports **public access** domains, which can receive requests from any internet-connected device, and **VPC access** domains, which are isolated from the public internet and hence highly recommended.

In this lab we have deployed Amazon ES with **VPC access** as its more secure. This endpoint is only visible to the servers inside that same VPC. To access Kibana from your laptop we have to create a SSH tunnel from your laptop to a *Bastion host* running inside the same VPC. Once you setup this up, you can access Kibana directly from your local broswer on your laptop.

To eshtablish an SSH tunnel, you need the following three key details. We will be using these details throughout the workshop, we **highly recommend** you make a note of it and keep it handy.


## Step 1: Public IP address of the Bastion host and Amazon ES endpoint URL
You've already noted this down in step 1 of the workshop. ```OutputFromNestedBastionStack``` refers to the public IP address for bastion host and ```OutputFromNestedESStack``` refers to the Amazon ES domain endpoint

## Step 2: SSH key file for Bastion host
You can find the name of your private key's S3 bucket by -> Cloudformation -> Click on the bottom most stack -> go to Outputs -> Copy S3 bucket name next to ```BastionHostPrivateKeyBucket```

Now we need to download the private key for our bastion host from this S3 bucket. Use the following steps by going to the S3 console:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion_300.png)

Then search for our previously copied bucket name:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion_400.png)

Click on the bucket name in order to open it and you will see our private key file:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion_600.png)

Click on the `bastion.pem` file and you can then download the key to your local machine by clicking the resulting `Download` button as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion_700.png)

## Step 3: Establish SSH tunnel

In order to load Kibana on your laptop’s browser, you need to send the traffic to your Amazon ES domain. Since the domain is in your VPC, and your Amazon ES cluster is in a private subnet, you must port forward to our bastion box and then access Kibana.

You need the public IP address of the bastion host and endpoint address of the Elasticsearch deployment which you copied as part of step 1.

**MacOS and Linux:**

In your **local MacOS or Linux terminal (not Cloud9)** navigate to the folder where you downloaded the .pem file and run the following command. Replace `<OutputFromNestedESStack>` and `<OutputFromNestedBastionStack>` with the respective values copied from the previous step.

```
cd /path/to/your_bastion_pem_folder
chmod 400 bastion.pem
ssh -i bastion.pem -N -L 9200:<OutputFromNestedESStack>:80 ec2-user@<OutputFromNestedBastionStack>
```

**Windows:**

To get started on a Windows machine, download Putty's MSI installer and use it to install Putty from the following download links:

[Windows 10 64-bit](https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.73-installer.msi)

[Windows 10 32-bit](https://the.earth.li/~sgtatham/putty/latest/w32/putty-0.73-installer.msi)

Once Putty has installed successfully, navigate to the Start Menu and open PuttyGen by searching for it as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/puttygen-open.png)

From the menu bar in PuttyGen, click **Conversions** and then **Import key** as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/puttygen-importkey-1.png)

From the resulting file explorer window, navigate to the location of your **bastion-rsav2** file that was downloaded previously from the S3 console as shown and click open:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/puttygen-importkey-2.png)

If the key is loaded into PuttyGen Successfully, we'll see the following screen:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/puttygen-saveppk-1.png)

On this screen, click **Save private key** and from the resulting file explorer window, save the resulting ppk file by giving it a name and clicking **Save** as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/puttygen-saveppk-2.png)

Now you can close PuttyGen and navigate back to the Start Menu and open Putty by searching for it as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-open.png)

Launch PuTTY and create a new session. Use the `OutputFromNestedBastionStack` parameter from the CloudFormation template outputs as the IP address under **Host Name**.

Give the session a name since we will want to save this in case we lose the session (low laptop battery, etc).

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion_800.png)

Expand the SSH section and navigate to the Tunnel.

Using the `<OutputFromNestedESStack>` output parameter, create a tunnel with a local port
of 9200 and a destination found in the value of this param as seen below:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-tunnel-1.png)

Navigate to the SSH section and add your key or if using pageant, allow agent forwarding:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-load-ppk.png)

Now set the connection keep-alives so our session does not die. This is in the connection section as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-set-keepalive.png)

Finally, save the config by scrolling back up to session and clicking save as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-save-cfg.png)

Go ahead and open the session:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-open-session.png)

## Step 4: Connect to Kibana with your local browser

Open this link in deep link to open Kibana in local browser on your laptop and on the welcome screen click **Explore on my own**

(Open new tab)

http://localhost:9200/_plugin/kibana

You will will taken to the Kibana home page with many controls and dashboards. Kibana does not have any data yet, so lets switch back to Cloud9 to start emitting data to Amazon ES.

# Deploying Logstash

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
```

First we need to update the `logstash-deployment.yml` with our Elasticsearch endpoint's address, for this run the following command. Replace the `<OutputFromNestedESStack>` with the Elasticsearch endpoint the Cloud9 setup step. Replace the `<OutputFromNestedESStack>` with the Elasticsearch endpoint the Cloud9 setup step.

```
chmod +x deploy-logstash.sh
./deploy-logstash.sh https://<OutputFromNestedESStack>:443
```

Note down the Logstash IP address, going forward we will refer to this as `logsash_ip_address`.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/logstash-ip.png)

Incase the aforementioned command did not provide the logstash IP, usually due to a small delay in logstash becoming available, run the following command:

```
kubectl -n logstash get pod -oyaml | yq r - 'items[0].status.podIP'
```

You can view the pod's readiness state by running the following command. You will notice logstash pods in running state.

```
kubectl -n logstash get pods
```


Move on to the next step.


# Filebeat 101

Filebeat is a lightweight shipper for forwarding and centralizing log data. Installed as an agent on your servers, Filebeat monitors the log files or locations that you specify, collects log events, and forwards them to either to Elasticsearch or Logstash for indexing.

In this lab we will configure Filebeat to collect Kubelet logs and send the data to logstash.

Kubelet is a process running on every worker nodes inside of a Kubernetes cluster. It is responsible for coordinating the scheduling/termination of containers running on the worker node.

```
filebeat.inputs:
    - type: log
      paths:
        - /var/log/messages
      tags: ["filebeat"]
      processors:
        - add_kubernetes_metadata:
            in_cluster: true
            host: ${NODE_NAME}
            matchers:
            - logs_path:
                logs_path: "/var/log/containers/"
    logging.level: debug
```

Our filebeat deployment for this workshop runs a DaemonSet resource in Kubernetes, which means a logging container deployed per EKS worker node.

You can start the deployment using the following command and substituting the `<logsash_ip_address>` section with the pod IP we saved from Logstash deployment output:

```
cd /home/ec2-user/environment/ant332/final-deploy/filebeat-manifests
chmod +x deploy-filebeat.sh
./deploy-filebeat.sh <logsash_ip_address>
```

You can confirm deployment using the following command:

```
kubectl get pods
```

The output should look like the following if filebeat has deployed successfully:

```
TeamRole:~/environment/ant332/final-deploy/filebeat-manifests (master) $ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
filebeat-977rj   1/1     Running   0          12s
filebeat-gpd2p   1/1     Running   0          12s
filebeat-rst4k   1/1     Running   0          12s
filebeat-tgblm   1/1     Running   0          12s
```

# Metricbeat 101

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
cd /home/ec2-user/environment/ant332/final-deploy/kube-state-metrics &&
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

**Run** this command to deploy Metricbeat. Metricbeat is set to automatically load pre-built Kibana dashboards and also starts pushing data into Logstash. Replace `logsash_ip_address` with the Logstash IP we noted in the Deploying Logstash step earlier and replace `OutputFromNestedESStack` with corresponding values from your Cloudformation stack output.

```
cd /home/ec2-user/environment/ant332/final-deploy/metricbeat
chmod +x deploy-metricbeat.sh
./deploy-metricbeat.sh <logsash_ip_address> <OutputFromNestedESStack>:443
```

You can run the following command to see if the metricbeat pods started correctly:

```
kubectl -n kube-system get pods | grep metricbeat
```
Output should look like the following if the pods started successfully:
```
TeamRole:~/environment/ant332/deploy (master) $ kubectl -n kube-system get pods | grep metricbeat
metricbeat-578b596685-brp9s           1/1     Running   0          30h
metricbeat-fnzrw                      1/1     Running   0          30h
metricbeat-l68nz                      1/1     Running   0          30h
metricbeat-vfqsn                      1/1     Running   0          30h
metricbeat-xwr7g                      1/1     Running   0          30h
```

Replace `<METRICBEAT_POD_NAME>` with the pod pod's name you got in the previous command. You can pick **any** of the four available metricbeat pods' name, they all work in the same way.

# View Metricbeat data in Kibana
Every time you create a new index in Elasticsearch, you have to configure **Index Pattern** in Kibana. This allows Kibana to be aware of the index and and lets you start creating visualizations and dashboards.

Click this deep link to open Kibana in your browser and follow the steps below - http://localhost:9200/_plugin/kibana

Double click on discover → From the drop down, choose metricbeat

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/mb-1.png)

Congratulations! You've configured you're first end-to-end pipeline. You will now be able to see all the metricbeat data in real-time.

***Pro tip:*** On the *top right corner* of Kibana you will find two very useful settings. **Auto-refresh** controls how frequent your dashboard needs to refresh. We recommend setting it to 30 seconds. The next setting labled **last 15 minutes** controls the time-range of the data Kibana will display. When set to 'last 15 minutes', you will only see the most recent 15 minutes of data. You can leave this at the default setting.

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/auto-refresh.png)


Explore the data that Mericbeat is pushing into Elasticsearch. You can type text into the search bar such as **kubernetes** and filter results. After you finish exploring, you can move on to the next step.  

# Fluentd 101

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
export URL2=OutputFromNestedESStack
```

A popular method to deploy complex applications in Kubernetes is through **Helm**. Helm is a tool that streamlines installing and managing Kubernetes applications. Think of it like apt/yum/homebrew for Kubernetes. We  built a helm chart that deploys both Fluentbit and Fluentd with a single command.

**Run** this command to setup the deployment files for Fluentd

```
cd /home/ec2-user/environment/ant332 && unzip deploy.zip && cd deploy
```

**Run** this command to deploy Fluetnd and Fluentbit
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


# Configure Kibana for Kubernetes Index
Click this deep link to open Kibana in your browser and follow the steps below - http://localhost:9200/_plugin/kibana

Click **Mangement** tab

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step1.png)

Now click on **Index Patterns**

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step2.png)

Click **Create Index Pattern**

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step3.png)

Type ***kubernetes-**** in the input box

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step4.png)

Select ***@timestamp*** field from the drop down and Click  **Create Index Pattern**

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/choose-timestamp.png)

Kibana is now fully setup to start visualizing Kubernetes logs. Lets setup the rest of the index patterns so we can visualize all data. Proceed to next step.

# Configure Kibana for redis index
Repeat the previous step to create an index pattern for **redis-** index. Click this deep link to open Kibana in your browser and follow the steps below - http://localhost:9200/_plugin/kibana

Click **Management tab** -> Click **Index Patterns** -> Click **Create Index Pattern** -> Type ***redis-**** in input box -> Choose ***@timestamp*** from drop down-> Click **Create Index Pattern**

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/redis-index.png)

# Configure Kibana for apache index
Repeat the previous step again to create an index pattern for **apache-** index. Click this deep link to open Kibana in your browser and follow the steps below -
http://localhost:9200/_plugin/kibana

Click **Management tab** -> Click **Index Patterns** -> Click **Create Index Pattern** -> Type ***apache-**** in input box -> Choose ***@timestamp*** from drop down-> Click **Create Index Pattern**

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/apache-index.png)

# Explore in Kibana
Lets explore the log data in Kibana. Fluentd is configured to output all the logs into **kubernetes-*** index. Click this deep link to open Kibana in your browser and follow the steps below - http://localhost:9200/_plugin/kibana

Click Discover tab -> Choose ***kubernetes-**** from dropdown -> use search box to search for guestbook or any other term you're interested in

Double-click **Discover** tab

![Alt Text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/view-kuber-step-1.png)

From the dropdown, choose ***kubernetes-****

![Alt Text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/view-kuber-step-2.png)


Type **guestbook** in the search bar and review results.

![Alt Text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/view-kuber-step-3.png)

You will now start seeing all the logs that Fluentd is pushing into Amazon ES. You can also switch the index from the dropdown to view other index data such as redis, apache etc.

Once you're done exploring, move on to the next step.



**(Optional)**
Bonus point: If you'd like this data to be more readable, you can create a custom view by choosing specific fields to be displayed. Click on the link below for a quick video on creating your own custom Kibana view.
https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kibana-advanced-view.gif

# Mystery hunt exercise
Great job setting the environment! You now have logs and metrics flowing in Amazon ES and you can visualize all of this data in Kibana.

The last portion of this workshop is called **Mystery Hunt**. We'll ask you to run mystery scripts that make changes to your environment, you'll then need to investigate and find root cause. We'll have hints along the way to guide you.

Kibana is all you need for this exercise, it has all the data including visualizations.


# Mystery hunt 1

Your application developers have raised a ticket that you have to investigate. They are unable to reach their throughput goals despite scaling their application pods. What could be the issue?

#### Simulate the scenario
**Run** the command below to simulate the scenario

```
 curl -O https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/mystery_script_1.sh && chmod +x mystery_script_1.sh && ./mystery_script_1.sh
```
#### Hint

**[Metricbeat Kubernetes] Overview ECS** Kibana dashboard is a great place to start to get a quick high level overview of your Kubernetes environment.

![Alt Text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/mb-kibana-dashboard.png)


#### Mystery Solved
The mystery_script_1.sh scaled up the frontend deployment to 100 replicas. However due to resource constraints the current EKS cluster can only deploy 39 pods across all deployments. So, the rest of the scale up requests are queued for future deployment when resources become available. You will notice a high number of unavailable pods frontend deployment has the highest number of unavailable pods. One possible resolution is to provision more worker nodes to add more capacity.

![Alt Text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/mystery-1-solved.png)

**(Optional)**

If you'd like to experiment in real-time, execute the following command to scale down frontend to 2 replicas, you should see this reflect in you Kibana dashboard instantly.

```
kubectl scale --replicas=2 deployment/frontend -n guestbook
```

# Mystery hunt 2
Your customers have complained that Guestbook website is throwing 404 not found errors. Investigate and identify the root cause.

#### Simulate the scenario
**Run** the command below to simulate the scenario
```
curl -O https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/mystery_script_2.sh && chmod +x mystery_script_2.sh && ./mystery_script_2.sh $URL3

```


#### Hint

Remeber we created ```apache-``` index? It will have useful information to troubleshoot.


#### Mystery Solved

Open Kibana, Double-click Discover tab. Then choose apache-* index pattern and search for 404 errors. You will see all the logs records that contain 404 errors. **Make sure the search bar on top is empty**

If you expand the the logs lines you will notice that 404 errors are being generated by paths that do not exist such as ```/index/menu.html``` and ```/index/contactus.html```

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/mystery-hunt-2-hint-1.png)

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/mystery-hunt-2-hint-2.png)


**(Optional)**

We’ve built a custom visualization that will make this data more readable. Click on the deep link below to download the visualization and unzip the file.

**Download Visualization**

https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/apache-http-request.json.zip

**Watch this video on how to import visualization in Kibana**

https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kibana-import-video.mp4

# Mystery Hunt 3

Your development team has reported that redis performance has degraded over time. They suspect memory fragmentation may be the root cause. They've asked for an easy way to get visibility into trend of memory fragmentation over a period of time.

#### Hint
Remember we created the ***redis-*** index? Examine the contents of this index and build a visualization in Kibana.



#### Mystery Solved
The ***redis-*** index contains all the key redis metrics  collected by flunetbit. The ```mem_fragmentation_ratio``` field gives the ratio of memory used as seen by the operating system to memory allocated by Redis.

Tracking fragmentation ratio is important for understanding your Redis instance’s performance. A fragmentation ratio greater than 1 indicates fragmentation is occurring. A ratio in excess of 1.5 indicates excessive fragmentation, with your Redis instance consuming 150% of the physical memory it requested. A fragmentation ratio below 1 tells you that Redis needs more memory than is available on your system, which leads to swapping. Swapping to disk will cause significant increases in latency (see used memory). Ideally, the operating system would allocate a contiguous segment in physical memory, with a fragmentation ratio equal to 1 or slightly greater.

**Watch this video to create memory fragmentation visualization**

https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/redis-mem-fragmentation-video.mp4

**(Optional)**
We've built a custom dashboard to monitor redis metrics. Follow the steps below to import this dashboard and explore.

**Download and unzip dashboard and visualization**

https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/redis-dashboard-visualization.zip

**Watch this video to import visualizations into Kibana**

https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/redis-vis-import.mp4


### Appendix

**Logstash troubleshooting**
```
kubectl -n logstash get pods
```

Will give you the pod's name, which you can use with the following command to see if the pipeline started listening on port 5044 for beats input. Once the pipeline is ready, we can move to deploying our beats platforms.
```
kubectl -n logstash logs -f <LOGSTASH_POD_NAME>
```
Use shortcut **ctrl+c** to get back to terminal.

**Metricbeat troubleshooting**

You can get container logs as well if you want to confirm events are being published to Logstash by using the following command:
```
kubectl -n kube-system logs -f <METRICBEAT_POD_NAME>
```

Replace `<METRICBEAT_POD_NAME>` with the pod pod's name you got in the previous command.

Use shortcut **ctrl+c** to get back to terminal.
