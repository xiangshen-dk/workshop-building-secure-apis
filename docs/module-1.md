# Set up the project
Time Estimate: 10 - 15 minutes  
In this module, we will create the AWS resources you need for your labs via AWS CloudFormation. Once the CloudFormation template starts you don't have to wait and can move the second module.

## Prerequisites

### AWS Account
In order to complete this workshop, you'll need an AWS account and access to create and manage the AWS resources that are used in this workshop, including Cloud9, Cognito, API Gateway, CloudWatch, WAF, EC2, and IAM policies and roles.

The code and instructions in this workshop assume only one participant is using a given AWS account at a time. If you attempt sharing an account with another participant, you may encounter naming conflicts for certain resources. You can work around this by using distinct Regions, but the instructions do not provide details on the changes required to make this work.

Please make sure __not__ to use a production AWS environment or account for this workshop. It is recommended to instead use a **development account** which provides **full access** to the necessary services so that you do not run into permissions issues.

### Region Selection
Use a single region for the entirety of this workshop. Choose one region from the launch stack links below and continue to use that region for all of the Auth workshop activities.



## Provision the AWS resources required for this workshop

A VPC is required for our workshop so we can:

* Leverage a Cloud9 environment as our IDE (integrated development environment)
* Use an EC2 instance as our backend server, which also hosts the MySQL database as the data store of the workshop Java Spring Boot application.

Follow the steps below to create the set up resources (VPC, Cloud9 environment, etc.)

1. Select the desired region. Since we are going to use services like EC2 and Cloud9, please choose one of these following and click the corresponding **Launch Stack** link

	&#128161; **When clicking on any link in this instruction, hold the ⌘ (mac) or Ctrl (Windows) so the links open in a new tab**

	Region| Code | Launch
	------|------|-------
    US East (N. Virginia) | <span style="font-family:'Courier';">us-east-1</span> | [![Launch setup resource in us-east-1](../images/cfn-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=ReinforceStack&templateURL=https://s3.amazonaws.com/workshop.reinforce.awsdemo.me/us-east-1/reinforce.yml)
    US East (Ohio) | <span style="font-family:'Courier';">us-east-2</span> | [![Launch setup resource in us-east-2](../images/cfn-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/new?stackName=ReinforceStack&templateURL=https://s3.amazonaws.com/workshop.reinforce.awsdemo.me/us-east-2/reinforce.yml)
	US West (Oregon) | <span style="font-family:'Courier';">us-west-2</span> | [![Launch setup resource in us-west-2](../images/cfn-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=ReinforceStack&templateURL=https://s3.amazonaws.com/workshop.reinforce.awsdemo.me/us-west-2/reinforce.yml)
	

1. 	Click **Next**
1. In the **Step 2: Specify stack details** page:
	* name you stack ***`ReinforceStack`***
	and click **Next**
	
1. In the **Step 3:Configure stack options** page, accept the default configurations and click **Next**
1. Review the configuration and click **Create stack**

1. It will take a few minutes for the Stack to create. Choose the **Stack Info** tab to go to the overall stack status page and wait until the stack is fully launched and shows a status of *CREATE_COMPLETE*. Click the refresh icon periodically to see progress update.

	> Note: When you launch the stack, CloudFormation deploys a nested CloudFormation stack to launch the Cloud9 resources. You can safely ignore that template which is prefixed with "aws-cloud9-reInforce".

This CloudFormation stack spins up the below resources:

* A **VPC** with 4 subnets, 2 private and 2 public. 
* A **Cloud9** environment where you will be developing and launching the rest of the workshop resources from.
* An **EC2 instance** which hosts the lab application and MySQL database

![initial resources diagram](../screenshots/architecture0.png)
