+++
title = "Mystery hunt 1"
date = 2019-11-26T19:44:40-08:00
weight = 1
pre = "<b>1. </b>"
+++

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
