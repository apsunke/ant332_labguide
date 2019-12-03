+++
title = "Appendix"
date = 2019-11-26T21:01:43-08:00
weight = 11
chapter = true
pre = "<b>7. </b>"
+++

## Logstash troubleshooting
```
kubectl -n logstash get pods
```

Will give you the pod's name, which you can use with the following command to see if the pipeline started listening on port 5044 for beats input. Once the pipeline is ready, we can move to deploying our beats platforms.
```
kubectl -n logstash logs -f <LOGSTASH_POD_NAME>
```
Use shortcut **ctrl+c** to get back to terminal.

***HELPER:*** Incase you have made a mistake in your Logstash deployment run the following command to navigate to the `logstash-manifests` folder:

```
cd /home/ec2-user/environment/ant332/final-deploy/logstash-manifests
```

Change permissions on the `delete-logstash.sh` file to executable using the following command:

```
chmod +x delete-logstash.sh
```

Now run the `delete-logstash.sh` script using the following command:
```
./delete-logstash.sh
```
In a few seconds you should see output for deleted resources under the `logstash` namespace.

Go ahead and repeat the Logstash deployment steps again.

## Metricbeat troubleshooting

The following command will give you the running metricbeat pods' names:

```
kubectl -n kube-system get pods | grep metricbeat
```

You can get container logs as well if you want to confirm events are being published to Logstash by using the following command:
```
kubectl -n kube-system logs -f <METRICBEAT_POD_NAME>
```

Replace `<METRICBEAT_POD_NAME>` with the any of the pods shown in the previous command.

Use shortcut **ctrl+c** to get back to terminal.

***HELPER:*** Incase you have made a mistake in your Metricbeat deployment run the following command to navigate to the `metricbeat` folder:

```
cd /home/ec2-user/environment/ant332/final-deploy/metricbeat
```

Change permissions on the `delete-metricbeat.sh` file to executable using the following command:

```
chmod +x delete-metricbeat.sh
```

Now run the `delete-metricbeat.sh` script using the following command:
```
./delete-metricbeat.sh
```

In a few seconds you should see output for deleted metricbeat resources.

Go ahead and repeat the metricbeat deployment steps again.

## Filebeat troubleshooting

The following command will give you the running filebeat pods' names:

```
kubectl get pods | grep filebeat
```

You can get container logs as well if you want to confirm events are being published to Logstash by using the following command:
```
kubectl logs -f <FILEBEAT_POD_NAME>
```

Replace `<FILEBEAT_POD_NAME>` with the any of the pods shown in the previous command.

Use shortcut **ctrl+c** to get back to terminal.

***HELPER:*** Incase you have made a mistake in your Filebeat deployment run the following command to navigate to the `filebeat-manifests` folder:

```
cd /home/ec2-user/environment/ant332/final-deploy/filebeat-manifests
```

Change permissions on the `delete-filebeat.sh` file to executable using the following command:

```
chmod +x delete-filebeat.sh
```

Now run the `delete-filebeat.sh` script using the following command:
```
./delete-filebeat.sh
```

In a few seconds you should see output for deleted filebeat resources.

Go ahead and repeat the metricbeat deployment steps again.
