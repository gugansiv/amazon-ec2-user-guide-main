# Use Amazon EFS with Amazon EC2<a name="AmazonEFS"></a>

Amazon EFS provides scalable file storage for use with Amazon EC2\. You can use an EFS file system as a common data source for workloads and applications running on multiple instances\. For more information, see the [Amazon Elastic File System product page](https://aws.amazon.com/efs)\.

**Important**  
Amazon EFS is not supported on Windows instances\.

You can mount an EFS file system to your instance in the following ways:

**Topics**
+ [Create an EFS file system using Amazon EFS Quick Create](#quick-create)
+ [Create an EFS file system and mount it to your instance](#create-mount)

## Create an EFS file system using Amazon EFS Quick Create<a name="quick-create"></a>

You can create an EFS file system and mount it to your instance at the time of launch using the Amazon EFS Quick Create feature of the Instance Launch Wizard\.

When you create an EFS file system using EFS Quick Create, the file system is created with the following service recommended settings:
+ Automatic backups turned on\. For more information, see [Using AWS Backup with Amazon EFS](https://docs.aws.amazon.com/efs/latest/ug/awsbackup.html) in the *Amazon Elastic File System User Guide*\.
+ Mount targets in each default subnet in the selected VPC, using the VPC's default security group\. For more information, see [Managing file system network accessibility](https://docs.aws.amazon.com/efs/latest/ug/manage-fs-access.html) in the *Amazon Elastic File System User Guide*\.
+ General Purpose performance mode\. For more information, see [ Performance Modes](https://docs.aws.amazon.com/efs/latest/ug/performance.html#performancemodes) in the *Amazon Elastic File System User Guide*\.
+ Bursting throughput mode\. For more information, see [ Throughput Modes](https://docs.aws.amazon.com/efs/latest/ug/performance.html#throughput-modes) in the *Amazon Elastic File System User Guide*\.
+ Encryption of data at rest enabled using your default key for Amazon EFS \(`aws/elasticfilesystem`\)\. For more information, see [Encrypting Data at Rest](https://docs.aws.amazon.com/efs/latest/ug/encryption-at-rest.html) in the *Amazon Elastic File System User Guide*\.
+ Amazon EFS lifecycle management enabled with a 30\-day policy\. For more information, see [EFS lifecycle management](https://docs.aws.amazon.com/efs/latest/ug/lifecycle-management-efs.html) in the *Amazon Elastic File System User Guide*\.

**To create an EFS file system using Amazon EFS Quick Create**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Launch Instance**\.

1. On the **Choose an AMI** page, choose a Linux AMI\.

1. On the **Choose an Instance Type** page, select an instance type and then choose **Next: Configure Instance Details**\. 

1. On the **Configure Instance Details** page, for **File systems**, choose **Create new file system**, enter a name for the new file system, and then choose **Create**\.

   To enable access to the file system, the following security groups are automatically created and attached to the instance and the mount targets of the file system\. 
   + **Instance security group**—Includes no inbound rules and an outbound rule that allows traffic over the NFS 2049 port\. 
   + **File system mount targets security group**—Includes an inbound rule that allows traffic over the NFS 2049 port from the instance security group \(described above\), and an outbound rule that allows traffic over the NFS 2049 port\.

   You can also choose to manually create and attach the security groups\. To do this, clear **Automatically create and attach the required security groups**\.

   Configure the remaining settings as needed and choose **Next: Add Storage**\.

1. On the **Add Storage** page, specify the volumes to attach to the instances, in addition to the volumes specified by the AMI \(such as the root device volume\)\. Ensure that you provision enough storage for the Nvidia CUDA Toolkit\. Then choose **Next: Add Tags**\.

1. On the **Add Tags** page, specify a tag that you can use to identify the temporary instance, and then choose **Next: Configure Security Group**\.

1. On the **Configure Security Group** page, review the security groups and then choose **Review and Launch**\.

1. On the **Review Instance Launch** page, review the settings, and then choose **Launch** to choose a key pair and to launch your instance\.

## Create an EFS file system and mount it to your instance<a name="create-mount"></a>

In this tutorial, you create an EFS file system and two Linux instances that can share data using the file system\.

**Topics**
+ [Prerequisites](#efs-prerequisites)
+ [Step 1: Create an EFS file system](#efs-create-file-system)
+ [Step 2: Mount the file system](#efs-mount-file-system)
+ [Step 3: Test the file system](#efs-test-file-system)
+ [Step 4: Clean up](#efs-clean-up)

### Prerequisites<a name="efs-prerequisites"></a>
+ Create a security group \(for example, efs\-sg\) to associate with the EC2 instances and EFS mount target, and add the following rules:
  + Allow inbound SSH connections to the EC2 instances from your computer \(the source is the CIDR block for your network\)\.
  + Allow inbound NFS connections to the file system via the EFS mount target from the EC2 instances that are associated with this security group \(the source is the security group itself\)\. For more information, see [Amazon EFS rules](security-group-rules-reference.md#sg-rules-efs), and [Creating security Groups](https://docs.aws.amazon.com/efs/latest/ug/accessing-fs-create-security-groups.html) in the *Amazon Elastic File System User Guide*\.
+ Create a key pair\. You must specify a key pair when you configure your instances or you can't connect to them\. For more information, see [Create a key pair](get-set-up-for-amazon-ec2.md#create-a-key-pair)\.

### Step 1: Create an EFS file system<a name="efs-create-file-system"></a>

Amazon EFS enables you to create a file system that multiple instances can mount and access at the same time\. For more information, see [Creating Resources for Amazon EFS](https://docs.aws.amazon.com/efs/latest/ug/creating-using.html) in the *Amazon Elastic File System User Guide*\.

**To create a file system**

1. Open the Amazon Elastic File System console at [https://console\.aws\.amazon\.com/efs/](https://console.aws.amazon.com/efs/)\.

1. Choose **Create file system**\.

1. \(Optional\) For **Name**, enter a name for the file system\. This creates a tag with **Name** as the key and the name of the file system as the value\.

1. For **Virtual Private Cloud \(VPC\)**, select the VPC to use for your instances\.

1. Choose **Create**\.

1. After the file system is created, note the file system ID\. It is used later in this tutorial\.

1. Choose the file system ID\.

1. On the file systems page, choose **Network**, **Manage**\. View the mount targets that Amazon EFS creates in each Availability Zone in the Region in which your VPC resides\. For each Availability Zone for your instances, ensure that the value for **Security groups** is the security group that you created in [Prerequisites](#efs-prerequisites)\.

1. Choose **Save**\.

### Step 2: Mount the file system<a name="efs-mount-file-system"></a>

Use the following procedure to launch two `t2.micro` instances\. Note that T2 instances must be launched in a subnet\. You can use a default VPC or a nondefault VPC\.

**Note**  
There are other ways that you can mount the volume \(for example, on an already running instance\)\. For more information, see [Mounting File Systems](https://docs.aws.amazon.com/efs/latest/ug/mounting-fs.html) in the *Amazon Elastic File System User Guide*\.

**To launch two instances and mount an EFS file system**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Launch Instance**\.

1. For **Step 1: Choose an Amazon Machine Image \(AMI\)**, select an Amazon Linux AMI\.

1. For **Step 2: Choose an Instance Type**, keep the default instance type, `t2.micro`, and choose **Next: Configure Instance Details**\.

1. For **Step 3: Configure Instance Details**, do the following:

   1. For **Number of instances**, enter **2**\.

   1. \[Default VPC\] If you have a default VPC, it is the default value for **Network**\. Keep the default VPC and the default value for **Subnet** to use the default subnet in the Availability Zone that Amazon EC2 chooses for your instances\.

      \[Nondefault VPC\] Select your VPC for **Network**, and a public subnet from **Subnet**\.

   1. \[Nondefault VPC\] For **Auto\-assign Public IP**, choose **Enable**\. Otherwise, your instances do not get public IP addresses or public DNS names\.

   1. For **File systems**, choose **Add file system**\. Ensure that the value matches the file system ID that you created in [Step 1: Create an EFS file system](#efs-create-file-system)\. The path shown next to the file system ID is the mount point that the instance will use, which you can change\. Under **Advanced Details**, the **User data** is automatically generated, and includes the commands needed to mount the file system\.

   1. Advance to Step 6 of the wizard\.

1. On the **Configure Security Group** page, choose **Select an existing security group** and select the security group that you created in [Prerequisites](#efs-prerequisites)\. Then choose **Review and Launch**\.

1. On the **Review Instance Launch** page, choose **Launch**\.

1. In the **Select an existing key pair or create a new key pair** dialog box, select **Choose an existing key pair** and choose your key pair\. Select the acknowledgment check box, and choose **Launch Instances**\.

1. In the navigation pane, choose **Instances** to see the status of your instances\. Initially, their status is `pending`\. After the status changes to `running`, your instances are ready for use\.

Your instance is now configured to mount the Amazon EFS file system at launch and whenever it's rebooted\. 

### Step 3: Test the file system<a name="efs-test-file-system"></a>

You can connect to your instances and verify that the file system is mounted to the directory that you specified \(for example, /mnt/efs\)\.

**To verify that the file system is mounted**

1. Connect to your instances\. For more information, see [Connect to your Linux instance](AccessingInstances.md)\.

1. From the terminal window for each instance, run the df \-T command to verify that the EFS file system is mounted\.

   ```
   $ df -T
   Filesystem     Type              1K-blocks    Used          Available Use% Mounted on
   /dev/xvda1     ext4                8123812 1949800            6073764  25% /
   devtmpfs       devtmpfs            4078468      56            4078412   1% /dev
   tmpfs          tmpfs               4089312       0            4089312   0% /dev/shm
   efs-dns        nfs4       9007199254740992       0   9007199254740992   0% /mnt/efs
   ```

   Note that the name of the file system, shown in the example output as *efs\-dns*, has the following form\.

   ```
   file-system-id.efs.aws-region.amazonaws.com:/
   ```

1. \(Optional\) Create a file in the file system from one instance, and then verify that you can view the file from the other instance\.

   1. From the first instance, run the following command to create the file\.

      ```
      $ sudo touch /mnt/efs/test-file.txt
      ```

   1. From the second instance, run the following command to view the file\.

      ```
      $ ls /mnt/efs
      test-file.txt
      ```

### Step 4: Clean up<a name="efs-clean-up"></a>

When you are finished with this tutorial, you can terminate the instances and delete the file system\.

**To terminate the instances**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Select the instances to terminate\.

1. Choose **Instance state**, **Terminate instance**\.

1. Choose **Terminate** when prompted for confirmation\.

**To delete the file system**

1. Open the Amazon Elastic File System console at [https://console\.aws\.amazon\.com/efs/](https://console.aws.amazon.com/efs/)\.

1. Select the file system to delete\.

1. Choose **Actions**, **Delete file system**\.

1. When prompted for confirmation, enter the file system ID and choose **Delete file system**\.