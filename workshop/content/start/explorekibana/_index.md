+++
title = "Explore data in Kibana"
date = 2019-11-26T19:44:40-08:00
weight = 25
+++

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
