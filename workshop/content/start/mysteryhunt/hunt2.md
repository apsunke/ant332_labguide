+++
title = "Mystery hunt 2"
date = 2019-11-26T19:44:40-08:00
weight = 2
pre = "<b>2. </b>"
+++

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
```
