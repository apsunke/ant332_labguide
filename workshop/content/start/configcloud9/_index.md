+++
title = "Configure Cloud9 IDE"
date = 2019-11-26T19:44:40-08:00
weight = 14
+++

**AWS Cloud9** is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. It provides an interface to the deployed assets for this workshop, that include a Kubernetes cluster and an AWS Elasticsearch domain. We will run all our commands for this workshop from here.

Lets setup Cloud9 and get started. Cloud9 normally manages IAM credentials dynamically. This isn’t currently compatible with the EKS IAM authentication, so we will disable it and rely on the IAM role instead.**This step is a required step for the rest of the workshop.**

* With the environment open, in the AWS Cloud9 IDE, on the menu bar choose Cloud9 as shown:
![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9-1.png)

* Open the deployed IDE console by clicking the "Open IDE" button as shown:
![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9-2.png)

* When it comes up, customize the environment by closing the **welcome tab** and **lower work** area


![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9+close+tab.png)

Open a new **terminal** tab in the main work area.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9+open+terminal.png)

* Your workspace should now look like this. You will execute all commands from this terminal.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9+end+result.png)

* Click the the Preferences tab from the "AWS Cloud9" dropdown button, in the navigation pane as shown:
![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9-3.png)

* From the resulting menu choose AWS Settings --> Credentials and set the toggle button for "AWS managed temporary credentials" to disabled as shown:
![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cloud9-4.png)

**Gain EKS Cluster Access Using kubectl and AWS CLI**

With a bash terminal open in the lower window of your Cloud9 IDE, use the following command to enter the folder containing our k8s manifests:
```
cd /home/ec2-user/environment/ant332/final-deploy
chmod +x loadcreds.sh
```
Once here run the following to allow execution on our loadcreds.sh script and then run it to load EKS credentials into your Cloud9 environment. Replace ```<OutputCloud9AdminAccessKeyID>``` and ```<OutputCloud9AdminSecretAccessKey>``` with corresponding values from your environment.
```
source ./loadcreds.sh <OutputCloud9AdminAccessKeyID> <OutputCloud9AdminSecretAccessKey>
```

Now we will run the bootstrapping for `kubectl` and `aws-iam-authenticator` using the `startup.sh` script with the following commands.

**Note:** This command will output several lines indicating current status. Thats part of the workflow and there's no need to worry. Move on to the next step after when the prompt is ready.

```
chmod +x startup.sh
source ./startup.sh
```

Now run the following command to ensure you have access to the EKS cluster:
```
kubectl get ns
```

If you have access the output from this command should display the following namespaces:

```
TeamRole:~/environment/ant332/final-deploy (master) $ kubectl get ns
NAME              STATUS   AGE
default           Active   12d
kube-node-lease   Active   12d
kube-public       Active   12d
kube-system       Active   12d
```
