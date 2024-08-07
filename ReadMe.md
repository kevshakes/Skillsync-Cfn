### Deployment of the CloudFormation Template Using AWS CLI

Below is the detailed guide on how to deploy the `SkillSync-ASG.yml` CloudFormation template using AWS CLI.

#### **Step 1: Prerequisites**

1. **AWS CLI Installed**: Ensure you have the AWS CLI installed on your machine. You can download and install it from [here](https://aws.amazon.com/cli/).
2. **Configured AWS CLI**: Configure your AWS CLI with your credentials. Run:
   ```bash
   aws configure
   ```
   and provide your `AWS Access Key ID`, `AWS Secret Access Key`, `Default region name`, and `Default output format`.

3. **IAM Permissions**: Ensure that the IAM user or role you are using has sufficient permissions to create and manage CloudFormation stacks, VPCs, EC2 instances, and Auto Scaling Groups.

#### **Step 2: Create the CloudFormation Template File**

Create a file named `SkillSync-ASG.yml` and paste the CloudFormation template into this file.

#### **Step 3: Create the CloudFormation Stack**

Use the following AWS CLI command to create the CloudFormation stack. This command will deploy the resources defined in your `SkillSync-ASG.yml` template.

```bash
aws cloudformation create-stack \
    --stack-name SkillSync-ASG-Stack \
    --template-body file://SkillSync-ASG.yml \
    --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
    --region <your-region>
```

Replace `<your-region>` with your desired AWS region, such as `us-east-1`.

#### **Step 4: Monitor the Stack Creation**

You can monitor the progress of your stack creation using the AWS Management Console or the following AWS CLI command:

```bash
aws cloudformation describe-stacks \
    --stack-name SkillSync-ASG-Stack
```

This command will provide the status of your stack and any resources being created.

#### **Step 5: Update the CloudFormation Stack**

If you need to make changes to your template and update the stack, use the `update-stack` command:

```bash
aws cloudformation update-stack \
    --stack-name SkillSync-ASG-Stack \
    --template-body file://SkillSync-ASG.yml \
    --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
    --region <your-region>
```


#### **Verifying Auto Scaling Activities using Stress Tool**

To test the scaling in and scaling out of your Auto Scaling Group (ASG), you can simulate load on your EC2 instances using a tool like `stress`. Here’s a step-by-step guide on how to do this:

### Step 1: Connect to an EC2 Instance

First, you need to connect to one of the EC2 instances launched by your ASG.

1. **Find the Instance ID**:
   ```bash
   aws ec2 describe-instances \
       --filters "Name=tag:Name,Values=Skillsync-ASG" \
       --query "Reservations[*].Instances[*].InstanceId" \
       --output text
   ```

2. **Connect to the Instance**:
   ```bash
   ssh -i /path/to/your/Skillsync-Key.pem ec2-user@<Public-IP-Address>
   ```

### Step 2: Install `stress` Tool

Once connected to the instance, install the `stress` tool.

```bash
sudo yum install -y epel-release
sudo yum install -y stress
```

### Step 3: Trigger Scale Out

Run the `stress` command to simulate high CPU load, which should trigger the scale-out policy.

```bash
stress --cpu 4 --timeout 300
```

This command stresses 4 CPU cores for 300 seconds (5 minutes). Adjust the parameters based on your scaling policies and instance specifications.

### Step 4: Monitor Scaling Activity

Monitor the scaling activity to see if the ASG launches additional instances.

```bash
aws autoscaling describe-scaling-activities \
    --auto-scaling-group-name Skillsync-ASG
```

You can also use the AWS Management Console to check the status of your ASG and see new instances being added.

### Step 5: Trigger Scale In

To test scaling in, you can stop the `stress` tool (if it’s running in the background) or simply wait for the cooldown period to end and the CPU usage to go back to normal levels.

### Step 6: Verify Scaling In

After the load decreases, the ASG should terminate some of the instances according to the scale-in policy. Again, you can monitor this using the AWS CLI or the AWS Management Console.

#### **Testing Auto Scaling Policies using Execute-Policy**

To test the scaling in and out policies, you can manually trigger them or simulate a load condition.

1. **List Scaling Policies**

   ```bash
   aws autoscaling describe-policies \
       --auto-scaling-group-name <Your-ASG-Name>
   ```

2. **Execute Scale-Out Policy**

   ```bash
   aws autoscaling execute-policy \
       --auto-scaling-group-name <Your-ASG-Name> \
       --policy-name ScaleOutPolicy
   ```

3. **Execute Scale-In Policy**

   ```bash
   aws autoscaling execute-policy \
       --auto-scaling-group-name <Your-ASG-Name> \
       --policy-name ScaleInPolicy
   ```

4. **Simulate Load with Custom Metrics**

   ```bash
   aws cloudwatch put-metric-data \
       --namespace "AWS/EC2" \
       --metric-name "CPUUtilization" \
       --dimensions "AutoScalingGroupName=<Your-ASG-Name>" \
       --value 90 \
       --unit Percent
   ```

#### **Verifying Auto Scaling Activities**

1. **Check Scaling Activities**

   ```bash
   aws autoscaling describe-scaling-activities \
       --auto-scaling-group-name <Your-ASG-Name>
   ```

2. **Check Instance Status**

   ```bash
   aws ec2 describe-instances \
       --filters "Name=tag:Name,Values=Skillsync-ASG"
   ```

### Example Commands

1. **Create Stack**

   ```bash
   aws cloudformation create-stack \
       --stack-name SkillSync-ASG-Stack \
       --template-body file://SkillSync-ASG.yml \
       --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
       --region us-east-1
   ```

2. **Update Stack**

   ```bash
   aws cloudformation update-stack \
       --stack-name SkillSync-ASG-Stack \
       --template-body file://SkillSync-ASG.yml \
       --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
       --region us-east-1
   ```

3. **Delete Stack**

   ```bash
   aws cloudformation delete-stack \
       --stack-name SkillSync-ASG-Stack \
       --region us-east-1
   ```

4. **Trigger Scale-Out Policy**

   ```bash
   aws autoscaling execute-policy \
       --auto-scaling-group-name Skillsync-ASG \
       --policy-name ScaleOutPolicy
   ```

5. **Trigger Scale-In Policy**

   ```bash
   aws autoscaling execute-policy \
       --auto-scaling-group-name Skillsync-ASG \
       --policy-name ScaleInPolicy
   ```

### Cleaning up the Resources
To clean up the resources created using Cloudformation you need to delete the stack and all associated resources, use the `delete-stack` command:

```bash
aws cloudformation delete-stack \
    --stack-name SkillSync-ASG-Stack \
    --region <your-region>
```