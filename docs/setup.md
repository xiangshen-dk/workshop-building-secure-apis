# Set up the project
Time Estimate: 10 - 15 minutes  

In this module, we will create some of the AWS resources you'll need for your labs via a provided AWS CloudFormation template. Once the CloudFormation template starts you don't have to wait and can move the second module.

??? info "What is AWS CloudFormation?"
    AWS CloudFormation allows you to model your entire infrastructure in a text file. This template can be used to provision your resources in a safe, repeatable manner, and allows you to treat your infrastructure as code. In this module, you will use an existing CloudFormation template (via the links below) to launch a baseline lab environment (diagram below). For more information, see https://aws.amazon.com/cloudformation/. 

## Prerequisites

### AWS Account
In order to complete this workshop, you'll need an AWS account and access to create and manage the AWS resources that are used in this workshop, including Cloud9, Cognito, API Gateway, CloudWatch, WAF, EC2, and IAM policies and roles.

The code and instructions in this workshop assume only one participant is using a given AWS account at a time. If you attempt sharing an account with another participant, you may encounter naming conflicts for certain resources. You can work around this by using distinct Regions, but the instructions do not provide details on the changes required to make this work.

Please make sure __not__ to use a production AWS environment or account for this workshop. It is recommended to instead use a **development account** which provides **full access** to the necessary services so that you do not run into permissions issues.

### Region Selection
Use a single region for the entirety of this workshop. Choose one region from the launch stack links below and continue to use that region for all of the modules.



## Provision the AWS resources required for this workshop

To complete the rest of the modules in this workshop, we will need several resources:

* A Virtual Private Cloud (VPC) to act as our networking environment.
* A Cloud9 environment to use as our integrated development environment (IDE).
* An EC2 instance to host our backend, which uses a MySQL database as the data store of a sample Java Spring Boot application.

Follow the steps below to create the set up these resources via AWS CloudFormation:

1. Select the desired region and click the corresponding **Launch Stack** link to open the CloudFormation console with the template pre-populated.

	&#128161; **When clicking on any link in this instruction, hold the ⌘ (mac) or Ctrl (Windows) so the links open in a new tab**

	Region| Code | Launch
	------|------|-------
    US East (N. Virginia) | <span style="font-family:'Courier';">us-east-1</span> | [![Launch setup resource in us-east-1](../images/cfn-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=ReinforceStack&templateURL=https://s3.amazonaws.com/workshop.reinforce.awsdemo.me/us-east-1/reinforce.yml)
    US East (Ohio) | <span style="font-family:'Courier';">us-east-2</span> | [![Launch setup resource in us-east-2](../images/cfn-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/new?stackName=ReinforceStack&templateURL=https://s3.amazonaws.com/workshop.reinforce.awsdemo.me/us-east-2/reinforce.yml)
	US West (Oregon) | <span style="font-family:'Courier';">us-west-2</span> | [![Launch setup resource in us-west-2](../images/cfn-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=ReinforceStack&templateURL=https://s3.amazonaws.com/workshop.reinforce.awsdemo.me/us-west-2/reinforce.yml)
	

1. Click **Next**.
1. In the **Step 2: Specify stack details** page:
	* Accept the default name ***`ReinforceStack`*** for your stack
	and click **Next**.
	
1. In the **Step 3: Configure stack options** page, accept the default configurations and click **Next**.
1. Review the configuration and on the bottom of the page, check the checkbox **I acknowledge that AWS CloudFormation might create IAM resources with custom names.**
1. Click **Create stack**.

1. The CloudFormation stack will now have a status of *CREATE_IN_PROGRESS*. This will take a few minutes to complete. The **Stack details** page shows the overall stack status. While this completes, you may move on to [**Module 2 - Resource policy**](/resource-policy/). Otherwise, you can wait until the stack is fully launched and shows a status of *CREATE_COMPLETE* (click the refresh icon periodically to see progress updates).

	> Note: When you launch this template, CloudFormation will use a nested stack to deploy the Cloud9 resources. This will appear as a another stack in the CloudFormation console with a name of "aws-cloud9-reInforceCloud9-XXXXXXX", which you can safely ignore.

This CloudFormation stack spins up the below resources:

* A **VPC** with 4 subnets, 2 private and 2 public. 
* A **Cloud9** environment from which you will be developing and launching several of the remaining workshop resources.
* An **EC2 instance** which hosts a sample application and MySQL database.

![initial resources diagram](../screenshots/architecture0.png)
