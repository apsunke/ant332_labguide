+++
title = "View Metricbeat data in Kibana"
date = 2019-11-26T19:44:40-08:00
weight = 19
+++

Every time you create a new index in Elasticsearch, you have to configure **Index Pattern** in Kibana. This allows Kibana to be aware of the index and and lets you start creating visualizations and dashboards.

Click this deep link to open Kibana in your browser and follow the steps below - http://localhost:9200/_plugin/kibana

Double click on discover → From the drop down, choose metricbeat

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/mb-1.png)

Congratulations! You've configured you're first end-to-end pipeline. You will now be able to see all the metricbeat data in real-time.

***Pro tip:*** On the *top right corner* of Kibana you will find two very useful settings. **Auto-refresh** controls how frequent your dashboard needs to refresh. We recommend setting it to 30 seconds. The next setting labled **last 15 minutes** controls the time-range of the data Kibana will display. When set to 'last 15 minutes', you will only see the most recent 15 minutes of data. You can leave this at the default setting.

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/auto-refresh.png)


Explore the data that Mericbeat is pushing into Elasticsearch. You can type text into the search bar such as **kubernetes** and filter results. After you finish exploring, you can move on to the next step.  
