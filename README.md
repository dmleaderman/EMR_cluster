# EMR_cluster
# Amazon EMR
Amazon EMR (previously called Amazon Elastic MapReduce) is a managed cluster platform that simplifies running big data frameworks, such as Apache Hadoop and Apache Spark, on AWS to process and analyze vast amounts of data.

All example applications and instructions here are based on  **release emr-6.14.0**

## EMRFS
In this tutorial, you use ```EMRFS``` to store data in an S3 bucket. ```EMRFS``` is an implementation of the Hadoop file system that lets you read and write regular files to Amazon S3.

Create the bucket in the same AWS Region where you plan to launch your Amazon EMR cluster.

Buckets and folders that you use with Amazon EMR have the following limitations:

- Names can consist of lowercase letters, numbers, periods (.), and hyphens (-).

- Names cannot end in numbers.

- A bucket name must be unique across all AWS accounts.

- An output folder must be empty.

### Create an S3 bucket
- Bucket name: y**our-emr-bucket-name**, for example: **msba2024-emr-integration**
- AWS Region: **US East(N.Virginia) us-east-1**
- ACLs **disabled** (recommended)
- **Block all public access**
- Bucket Versioning: **Disable**
- Encryption type: **Server-side encryption with Amazon S3 managed keys (SSE-S3)**
- Bucket Key: **Enable**

### Prepare an application with input data for Amazon EMR
The most common way to prepare an application for Amazon EMR is to upload the application and its input data to Amazon S3. Then, when you submit work to your cluster you specify the Amazon S3 locations for your script and data.

Copy [health_violations.py](health_violations.py) and save it to your local named **health_violations.py**

[Download food_establishment_data.csv](https://github.service.emory.edu/GBS/student-technology-tools/raw/main/docs/aws/EMR/food_establishment_data.csv)


Upload PySpark script **health_violations.py** and **food_establishment_data.csv** to the 
bucket your created, for example:  **msba2024-emr-integration**

## Launch an Amazon EMR Cluster

1. Search **EMR** in the search box
2. Under **EMR on EC2** in the left navigation pane, choose **Clusters**, and then choose **Create cluster**.
3. On the Create Cluster page, note the default values for ```Release, Instance type, Number of instances```, and ```Permissions```. These fields automatically populate with values that work for general-purpose clusters.
4. Cluster name: **your-emr-cluster-name**, for example, **annie-emr-cluster**
5. Under **Applications**, choose the **Spark** option to install Spark on your cluster.<br>
    **Note**: Choose the applications you want on your Amazon EMR cluster before you launch the cluster. You can't add or remove applications from a cluster after launch.
6. Under **Cluster logs**, select the **Publish cluster-specific logs to Amazon S3** check box. 
   Replace the **Amazon S3 location** value with the Amazon S3 bucket you created, followed by **/logs**. For example, **s3://msba2024-emr-integration/logs**. 
   Adding `````/logs````` creates a new folder called 'logs' in your bucket, where Amazon EMR can copy the log files of your cluster.

   ![emr1](../../image/emr1.PNG)

7. Under **Security configuration and permissions**, choose your **EC2 key pair**. In the same section, select the **Service role** for Amazon EMR dropdown menu and choose **EMR_DefaultRole**. Then, select the **IAM role for instance profile** dropdown menu and choose **EMR_EC2_DefaultRole**. 


   ![emr2](../../image/emr2.PNG)


8. Click **Create cluster** to launch the cluster and open the cluster details page.


   ![emr3](../../image/emr3.PNG)


9. Find the cluster **Status** next to the cluster name. The status changes from **Starting** to **Running** to **Waiting** as Amazon EMR provisions the cluster. You may need to choose the refresh icon on the right or refresh your browser to see status updates.

Your cluster status changes to **Waiting** when the cluster is up, running, and ready to accept work.


## Manage Your Amazon EMR Cluster

### Submit work to Amazon EMR
After you launch a cluster, you can submit work to the running cluster to process and analyze data. You submit work to an Amazon EMR cluster as a ```step```. A step is a unit of work made up of one or more actions.
1. Under **EMR on EC2** in the left navigation pane, choose **Clusters**, and then select the cluster where you want to submit work. The cluster state must be **Waiting**.
2. Choose the **Steps** tab, and then choose **Add step**.
3. Configure the step according to the following guidelines:
   - For **Type**, choose **Spark application**. You should see additional fields for **Deploy mode**, **Application location**, and **Spark-submit** options.
   - For **Name**, enter a new name. If you have many steps in a cluster, naming each step helps you keep track of them.
   - For **Deploy mode**, leave the default value **Cluster mode**.
   - For **Application location**, enter the location of your **health_violations.py** script in Amazon S3, such as **s3://msba2024-emr-integration/health_violations.py**.
   - Leave the **Spark-submit** options field empty.
   - In the **Arguments** field, enter the following arguments and values:
   ```shell
     --data_source s3://your-EMR-bucket-name/food_establishment_data.csv
     --output_uri s3://your-EMR-bucket-name/emr-output
   
     for example:
     --data_source s3://msba2024-emr-integration/food_establishment_data.csv
     --output_uri s3://msba2024-emr-integration/emr-output
   ```
   - For **Action if step fails**, accept the default option Continue. This way, if the step fails, the cluster continues to run.
     

   ![emr4](../../image/emr4.PNG)


4. Click **Add step** to submit the step. The step should appear in the console with a status of **Pending**.
5. Monitor the step status. It should change from **Pending** to **Running** to **Completed**. To refresh the status in the console, click the **Refresh table** to the right of Filter. The script takes about one minute to run. When the status changes to **Completed**, the step has completed successfully.
   
   
  ![emr5](../../image/emr5.PNG)


6. If the job failed, click **stderr** to see the details.

### View results

After a step runs successfully, you can view its output results in your Amazon S3 output folder.

## Connect to Your Running Amazon EMR Cluster
When you use Amazon EMR, you may want to connect to a running cluster to read log files, debug the cluster, or use CLI tools like the Spark shell. 

### Modify Inbound Rules to Authorize SSH connections to your cluster
Before you connect to your cluster, you need to modify your cluster security groups to authorize inbound SSH connections. 
Amazon EC2 security groups act as virtual firewalls to control inbound and outbound traffic to your cluster. 
<br>[**Modify Inbound Rules**](emrInboundRules.md)

### Connect to your cluster
Regardless of your operating system, you can create an SSH connection to your cluster.

Do one of followings to connect to your cluster: 
#### Using AWS CLI
1. Collect and configure [AWS Academy Lab Credential](../where_to_get_awsacademylab_credentials.pdf)
2. Find out ```Cluster ID```, go to  ```EMR on EC2``` in the left navigation pane, choose ```Clusters```, and then choose the cluster that you want to update, ```Cluster ID``` is under ```Summary``` section, 
3. Use the following command to open an SSH connection to your cluster. Replace <mykeypair.key> with the full path and file name of your key pair file. For example, C:\Users\<username>\.ssh\mykeypair.pem.
   ```shell
   aws emr ssh --cluster-id <j-2AL4XXXXXX5T9> --key-pair-file <~/mykeypair.key>	
   ```
   for example:
   ```
  
   aws emr ssh --cluster-id j-3PSGOP0RLHKL1 --key-pair-file ./integration_key.pem
   ```
  
#### Using OpenSSH Client (recommended)
  - Under **Summary** section, Get **Primary node public DNS**.
  - Windows: use [PuTTY](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html) to connect using user **hadoop**.
  - Linux or MacOS, or Windows ([installing OpenSSH on Windows](../../openssh/README.md#install-openssh-for-windows)) Run the command:
  ```shell
  ssh -i integration_key.pem hadoop@<primaryNodePublicDNS>
  ```
  Replace ```<primaryNodePublicDNS>``` with your cluster's **Primary node public DNS** under **Summary** section
  for example:
  ```
  ssh -i integration_key.pem hadoop@ec2-3-227-12-126.compute-1.amazonaws.com
  ```
  with **sudo**
  ```
  sudo ssh -i integration_key.pem hadoop@ec2-3-227-12-126.compute-1.amazonaws.com
  ```
  Once you connect to the cluster, you can Navigate to **/mnt/var/log/spark** to access the Spark logs on your cluster's master node. 
  Then view the files in that location. 

  ```shell
  cd /mnt/var/log/spark
  ls
  ```
  
## View Web Interfaces Hosted on Amazon EMR Clusters
There are two remaining options for accessing web interfaces on the primary node that provide full browser functionality. Choose one of the following:
- **Option 1 - Local Port Forwarding** (recommended for more technical users): Use an SSH client to connect to the primary node, configure SSH tunneling with local port forwarding, and use an Internet browser to open web interfaces hosted on the primary node. This method allows you to configure web interface access without using a SOCKS proxy.
- **Option 2 - Dynamic Port Forwarding** (recommended for new users): Use an SSH client to connect to the primary node, configure SSH tunneling with dynamic port forwarding, and configure your Internet browser to use an add-on such as FoxyProxy for Firefox or SwitchyOmega for Chrome to manage your SOCKS proxy settings. This method lets you automatically filter URLs based on text patterns and limit the proxy settings to domains that match the form of the primary node's DNS name.

#### [Using Option 1 - Local Port Forwarding](localPortForward.md)
#### [Using Option 2 - Dynamic Port Forwarding](dynamicPortForwarding.md)
#### [Using Python Scripts](../scripts/README.md)

## Troubleshooting
#### [Sqoop could not load db driver class](troubleShooting.md#sqoop-could-not-load-db-driver-class)

### AWS Academy Environment
This Learner Lab provides a sandbox environment for ad-hoc exploration of AWS services with limitations
See [AWS Academy Environment Restrictions](../awsAcademyRestrictions.md)
