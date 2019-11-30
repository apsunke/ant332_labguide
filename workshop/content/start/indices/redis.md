+++
title = "Configure Kibana for redis index"
date = 2019-11-26T19:44:40-08:00
weight = 2
pre = "<b>2. </b>"
+++

Repeat the previous step to create an index pattern for **redis-** index. Click this deep link to open Kibana in your browser and follow the steps below -

[http://localhost:9200/_plugin/kibana](http://localhost:9200/_plugin/kibana)


Click **Mangement** tab

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step1.png)

Now click on **Index Patterns**

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step2.png)

Click **Create Index Pattern**

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/kubernetes-*-step3.png)

Type ***redis-**** in the input box

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/redis-index.png)

Select ***@timestamp*** field from the drop down and Click  **Create Index Pattern**

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/choose-timestamp.png)

Kibana is now fully setup to start visualizing Redis logs. Proceed to next step.
