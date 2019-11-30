+++
title = "Deploying Fluentd and Fluentbit"
date = 2019-11-26T19:44:40-08:00
weight = 23
+++

Lets configure Fluentd to output to your Amazon ES domain. Copy your Amazon ES endpoint from the 'Configure Kibana' step. **Run** the following command by **modifying the url with your Amazon ES endpoint**

```
export URL2=OutputFromNestedESStack
```

A popular method to deploy complex applications in Kubernetes is through **Helm**. Helm is a tool that streamlines installing and managing Kubernetes applications. Think of it like apt/yum/homebrew for Kubernetes. We  built a helm chart that deploys both Fluentbit and Fluentd with a single command.

**Run** this command to setup the deployment files for Fluentd

```
cd /home/ec2-user/environment/ant332 && unzip deploy.zip && cd deploy
```

**Run** this command to deploy Fluentd and Fluentbit
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
