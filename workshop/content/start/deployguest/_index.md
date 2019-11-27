+++
title = "Deploy guestbook application"
date = 2019-11-26T19:44:40-08:00
weight = 15
+++

**Run** the following command to setup and delpoy guestbook application deployment.

```
cd guestbook
chmod +x deploy.sh
./deploy.sh
```

Use the following command to see if all pods within the deployment come up correctly:
```
kubectl -n guestbook get pods
```

Lets access the guestbook application and submit a few entries. Guestbook in configured to deploy an Elastic Load Balancer (ELB) to handle incoming traffic.  **Run** the following command to get the ```<Loadbalancer Ingress>``` DNS address for the guestbook app.

```
kubectl get svc frontend -n guestbook -oyaml | yq r - "status.loadBalancer.ingress[0].hostname"
```

Output should look like this:

```
a3adf8asjkdhaj1231f6e67d0c-30d08edda21d396f.elb.us-east-1.amazonaws.com
```

Lets load this ELB address ```<Loadbalancer Ingress>``` into an environment variable for future use.

```
export URL3=<Loadbalancer Ingress>
```

Now open a web broswer like Safari on your laptop and paste ```Loadbalancer Ingress```. It will take you to the guestbook homepage.

**It may take a few minutes for the application to be ready. In case you don't get a response please wait 5 minutes and try again**

Once you see the Guestbook page appear, enter your 2 of your favorite guest names and click **submit**.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/broswer-guestbook.png)
