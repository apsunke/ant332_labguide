+++
title = "Configure Kibana for Kubernetes Index"
date = 2019-11-26T19:44:40-08:00
weight = 1
pre = "<b>1. </b>"
+++

Click this deep link to open Kibana in your browser and follow the steps below -

[http://localhost:9200/_plugin/kibana](http://localhost:9200/_plugin/kibana)



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
