+++
title = "Lab Setup"
date = 2019-11-26T19:44:40-08:00
weight = 2
+++

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
