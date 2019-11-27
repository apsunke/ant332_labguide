+++
title = "Fluentbit deployment as a 'Sidecar'"
date = 2019-11-26T19:44:40-08:00
weight = 22
+++

In this lab we'll use Fluentbit to capture **metrics** from our Redis deployments. All the agents we discussed so far in this guide were be deployed as a daemonset (a POD that runs on every node of the cluster). With Fluentbit, we're deploying it as a side-car container (for redis contianers) instead. With this approach, whenever redis contaner is deployed there is a dedicated Fluentbit side-car container attached to collect all the relevant data.

Lets take a look at the ```redis-slave.yml``` deployment file for Guestbook application. This ensures that  a Fluentbit container is spun up every time a redis container launches.
```
spec:
      containers:
      - name: slave
        image: gcr.io/google_samples/gb-redisslave:v1
        resources:
        ....
        ports:
        - containerPort: 6379

      - name: redis-statistic
        image: fluent/fluent-bit:0.14.6
        resources:
        ....
```
Once its launched, Fluentbit is configured to run a custom script (loaded into the Fluentbit container path /bin/redis) to capture metrics from redis. It then forwards the data to Fluentd using the forwad protocol that was pre-setup. You can find these configurations are in ```fluent-bit.conf```.

```
<source>
  @type exec
  tag   redis
  command bash /bin/redis
  type json
  run_interval 1m
</source>

<match **>
  @type stdout
</match>

#!/bin/bash

for i in $( (printf 'info\r\n';) | nc -w 1 redis 6379 | grep ':'); do
 key=$(echo "$i" | cut -d ":" -f1 )
 value=$(echo "$i" | cut -d ":" -f2 | sed 's/\r$//g' )
 echo "{\"$key\": \"$value\"}"
done
```
