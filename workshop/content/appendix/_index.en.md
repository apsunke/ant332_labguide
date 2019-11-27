+++
title = "Appendix"
date = 2019-11-26T21:01:43-08:00
weight = 11
chapter = true
pre = "<b>7. </b>"
+++

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
