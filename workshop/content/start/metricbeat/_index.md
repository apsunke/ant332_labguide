+++
title = "Metricbeat 101"
date = 2019-11-26T19:44:40-08:00
weight = 19
+++

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

**Run** this command to deploy Metricbeat. Metricbeat is set to automatically load pre-built Kibana dashboards and also starts pushing data into Logstash. Replace `logsash_ip_address` with the Logstash IP we noted in the Deploying Logstash step earlier. Next use `OutputFromNestedESStack` from your Cloudformation stack output along with port `443` as shown in the script below.

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
