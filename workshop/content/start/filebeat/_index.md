+++
title = "Filebeat 101"
date = 2019-11-26T19:44:40-08:00
weight = 18
+++

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

Navigate to the filebeat folder using the following commands and change permissions to executable on our `deploy-filebeat-sh` script as shown:

```
cd /home/ec2-user/environment/ant332/final-deploy/filebeat-manifests
chmod +x deploy-filebeat.sh
```

You can now start the deployment using the following command and substituting the `<logsash_ip_address>` section with the pod IP we saved from Logstash deployment output as shown:

```
./deploy-filebeat.sh <logsash_ip_address>
```

***HELPER:*** Incase you make a mistake during these deployment steps, refer to the [Filebeat Troubleshooting](http://eseksworkshop.com/en/appendix.html#filebeat-troubleshooting) section in this lab's [Appendix](http://eseksworkshop.com/en/appendix.html).

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
