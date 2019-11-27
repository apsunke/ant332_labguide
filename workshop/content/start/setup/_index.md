+++
title = "Lab Setup and Login to AWS"
date = 2019-11-26T19:44:40-08:00
weight = 11
+++

## Instructions to log into your AWS account

### Step 1:
On your table, you will have a piece of paper that holds information needed to log into your AWS account

You will find a URL under the column named **team-hash-login**. The URL will look like this ```https://dashboard.eventengine.run/login?hash=yourHash123```

Copy paste this URL into a local browser on your laptop.

### Step 2: Login with team hash
Enter the **team-hash** value into the box and then click accept.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/teams-setup.png)

Now click on **AWS Console** to open the relevant links.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/teams-setup-2.png)

Now Click on **Open AWS Console** to log into your console.

![](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/teams-setup-3.png)

## Lab Setup Overview

* AWS Cloud9 IDE environment (this is where you'll spend the most time for this lab)
* Amazon ES domain
* Amazon EKS Cluster, API server, worker nodes
* Bastion host (Amazon EC2 instance) with SSH key

Follow the instructions below to note down information such as access key, IP address and Amazon Elasticsearch endpoints. We'll refer to these throught the workshop by their key name, for example ```OutputFromNestedESStack``` will refer to your Amazon ES endpoint.

Click on 'Services' tab and type 'Cloudformation' in the search box.

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cfn-1.png)

Now click on the bottom most stack on your screen. The name of your stack can be different from the one in the screenshot.

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cfn-2.png)

Click on the 'Outputs' tab and copy all the values highlighted in the screenshot. We recommend copying these into a notepad as we refer to these multiple times throught the lab.

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/cfn-3.png)
