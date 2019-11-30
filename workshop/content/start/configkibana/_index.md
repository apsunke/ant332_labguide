+++
title = "Configure Kibana"
date = 2019-11-26T19:44:40-08:00
weight = 16
+++

Amazon ES supports **public access** domains, which can receive requests from any internet-connected device, and **VPC access** domains, which are isolated from the public internet and hence highly recommended.

In this lab we have deployed Amazon ES with **VPC access** as its more secure. This endpoint is only visible to the servers inside that same VPC. To access Kibana from your laptop we have to create a SSH tunnel from your laptop to a *Bastion host* running inside the same VPC. Once you setup this up, you can access Kibana directly from your local broswer on your laptop.

To eshtablish an SSH tunnel, you need the following three key details. We will be using these details throughout the workshop, we **highly recommend** you make a note of it and keep it handy.


## Step 1: Public IP address of the Bastion host and Amazon ES endpoint URL
You've already noted this down in step 1 of the workshop. ```OutputFromNestedBastionStack``` refers to the public IP address for bastion host and ```OutputFromNestedESStack``` refers to the Amazon ES domain endpoint

## Step 2: SSH key file for Bastion host
You can find the name of your private key's S3 bucket by -> Cloudformation -> Click on the bottom most stack -> go to Outputs -> Copy S3 bucket name next to ```BastionHostPrivateKeyBucket```

Now we need to download the private key for our bastion host from this S3 bucket. Use the following steps by going to the S3 console:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion_300.png)


Then search for our previously copied bucket name:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion_400.png)

Click on the bucket name in order to open it and you will see our private key file:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion+key+file.png)

Click on the `bastion-rsav2` file and you can then download the key to your local machine by clicking the resulting `Download` button as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion_700.png)

## Step 3: Establish SSH tunnel

In order to load Kibana on your laptop’s browser, you need to send the traffic to your Amazon ES domain. Since the domain is in your VPC, and your Amazon ES cluster is in a private subnet, you must port forward to our bastion box and then access Kibana.

You need the public IP address of the bastion host and endpoint address of the Elasticsearch deployment which you copied as part of step 1.

**MacOS and Linux:**

In your **local MacOS or Linux terminal (not Cloud9)** navigate to the folder where you downloaded the .pem file and run the following command. Replace `<OutputFromNestedESStack>` and `<OutputFromNestedBastionStack>` with the respective values copied from the previous step.

```
cd /path/to/your_bastion_pem_folder
chmod 400 bastion-rsav2
ssh -i bastion-rsav2 -N -L 9200:<OutputFromNestedESStack>:80 ec2-user@<OutputFromNestedBastionStack>
```

**Windows:**

To get started on a Windows machine, download Putty's MSI installer and use it to install Putty from the following download links:

[Windows 10 64-bit](https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.73-installer.msi)

[Windows 10 32-bit](https://the.earth.li/~sgtatham/putty/latest/w32/putty-0.73-installer.msi)

Once Putty has installed successfully, navigate to the Start Menu and open PuttyGen by searching for it as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/puttygen-open.png)

From the menu bar in PuttyGen, click **Conversions** and then **Import key** as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/puttygen-importkey-1.png)

From the resulting file explorer window, navigate to the location of your **bastion-rsav2** file that was downloaded previously from the S3 console as shown and click open:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/puttygen-importkey-2.png)

If the key is loaded into PuttyGen Successfully, we'll see the following screen:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/puttygen-saveppk-1.png)

On this screen, click **Save private key** and from the resulting file explorer window, save the resulting ppk file by giving it a name and clicking **Save** as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/puttygen-saveppk-2.png)

Now you can close PuttyGen and navigate back to the Start Menu and open Putty by searching for it as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-open.png)

Launch PuTTY and create a new session. Use the `OutputFromNestedBastionStack` parameter from the CloudFormation template outputs as the IP address under **Host Name**.

Give the session a name since we will want to save this in case we lose the session (low laptop battery, etc).

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/bastion_800.png)

Expand the SSH section and navigate to the Tunnel.

As shown in the screenshot below, enter `<OutputFromNestedESStack>:80` in the Destination input to connect to `port 80`. This will create a tunnel with a local port
of `9200` and this destination.

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-tunnel-1.png)

Navigate to the SSH section and add your key or if using pageant, allow agent forwarding:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-load-ppk.png)

Now set the connection keep-alives so our session does not die. This is in the connection section as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-set-keepalive.png)

Finally, save the config by scrolling back up to session and clicking save as shown:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-save-cfg.png)

Go ahead and open the session:

![alt text](https://ant332.s3-us-west-2.amazonaws.com/ant332-lab-guide-artifacts/putty-open-session.png)

## Step 4: Connect to Kibana with your local browser

Open this link in deep link to open Kibana in local browser on your laptop and on the welcome screen click **Explore on my own**

(Open new tab)

http://localhost:9200/_plugin/kibana

You will will taken to the Kibana home page with many controls and dashboards. Kibana does not have any data yet, so lets switch back to Cloud9 to start emitting data to Amazon ES.
