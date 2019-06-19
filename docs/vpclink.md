# Set up Private Integrations
Time Estimate: 15 - 20 minutes

AWS API gateway supports a variety of integrations including Lambda, public/private http endpoints, and other AWS services such as S3, DynamoDB.

In this module, you will integrate API Gateway to a web service running inside a private subnet of your VPC.

### Verify project setup

1. Go to the [CloudFormation Console](https://console.aws.amazon.com/cloudformation) and make sure the workshop CloudFormation stacks in the __CREATE_COMPLETE__ status.

2. Go to the [AWS Cloud9](https://console.aws.amazon.com/cloud9) console, click on ***Your environments*** (you may need to expand the left sidebar)

    ??? info "What is AWS Cloud9"
        AWS Cloud9 is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just an internet browser. It includes a code editor, debugger, and terminal. Cloud9 also provides a seamless experience for developing serverless applications enabling you to easily define resources, debug, and switch between local and remote execution of serverless applications. In this project, we are launching Cloud9 on its own EC2 instance, but you can also connect Cloud9 to an existing Linux server. For more information, see https://aws.amazon.com/cloud9/.
    ![Cloud 9](../images/0B-cloud9-environments.png)

3. Find the __reInforceCloud9__ environment and click the button __Open IDE__ as following:
    ![Open IDE](../screenshots/c9-1.png)

    If you have trouble opening cloud9, ensure you are using:
	
	* Either  **Chrome** or **Firefox** browser 
	* Refer to the troubleshooting guide [**here**](https://docs.aws.amazon.com/cloud9/latest/user-guide/troubleshooting.html#troubleshooting-env-loading) to ensure third-party cookies is enabled.

4. You should now see an integrated development environment (IDE)  as shown below. You can view file in the editor and run shell commands in the terminal section just like you would on your local computers.

	![](../images/0B-cloud9-start.png)

	Keep your AWS Cloud9 IDE opened in a tab throughout this workshop as you'll be using it for most all activities.

5. Go to the [EC2 console](https://console.aws.amazon.com/ec2/v2/home), find the EC2 instance __lab-backend__:

    ![](../screenshots/lab-backend1.png)

    Notice this instance is running in a private subnet and it doesn't have a public IP.

6. Add the security group rule Protocol:TCP, Port Range:80, Source:0.0.0.0/0 to the __default__ security group.

    A. From EC2 console, click on default security group
    
    ![](../screenshots/securitygroup-1.png)
    
    B. Click on Edit and add security group rule
    
    ![](../screenshots/securitygroup-2.png)

7. Once in Cloud9, run the following commands in a terminal:
    
    !!! note "Important!"
        You will run all terminal commands and modify all source code for these labs from within Cloud9 (**not** your local machine). Keep in mind that Cloud9 is a full fledged IDE running on an Amazon EC2 instance. You can edit code and create new files via the Cloud9 editor in your browser. You can also open multiple terminals if needed.

    ``` bash hl_lines="2"
    sudo yum install jq -y
    curl -s  http://10.0.131.162/feedback|jq
    ```
You need replace the IP highlighted above to the privte IP of the lab-backend instance. You should have some output like the following:
    ![](../screenshots/lab-backend2.png)


### Create a network load balancer (NLB)

??? info "Why we need a NLB"
    The private integration uses an API Gateway resource ___VpcLink___ to encapsulate connections between API Gateway and targeted VPC resources. A NLB is required as the target of the VpcLink.

1. Run the following commands:

    ``` bash
    # Get VPC Id. If your CloudFormaton stack or VPC name
    # is different, update it in the command
    VPC_ID=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=ReinforceStack/reInforceWorkshopVpc|jq '.Vpcs[] |  .VpcId' -r)
    
    # Get private subnets
    SUBNETS=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC_ID Name=tag:aws-cdk:subnet-type,Values=Private|jq '.Subnets[]|.SubnetId' -r|paste -sd' ' -)
    
    # Create the NLB
    NLB_ARN=$(aws elbv2 create-load-balancer --name reinforce-lab-nlb --type network --scheme internal --subnets $SUBNETS |jq '.LoadBalancers[]|.LoadBalancerArn' -r)
    
    # Create the target group
    TG_ARN=$(aws elbv2 create-target-group --name reinforce-targets --protocol TCP --port 80 --vpc-id $VPC_ID|jq '.TargetGroups[]|.TargetGroupArn' -r)
    
    # Get the instance id
    INSTANCE_ID=$(aws ec2 describe-instances --filters 'Name=tag:Name,Values=lab-backend*' --output text --query 'Reservations[*].Instances[*].InstanceId')
    
    # Register targets
    aws elbv2 register-targets --target-group-arn $TG_ARN --targets Id=$INSTANCE_ID

    # Create listener
    aws elbv2 create-listener --load-balancer-arn \
    $NLB_ARN --protocol TCP --port 80 \
    --default-actions Type=forward,TargetGroupArn=$TG_ARN

    ```
2. Go to the [Target Groups](https://console.aws.amazon.com/ec2/v2/home) under __LOAD BALANCING__ to check if the rgistered targets are healthy. If it's in the 'initial' status, wait until it becomes healthy.
    ![](../screenshots/lab-backend3.png)

3. Check the NLB endpoint by running the following commands inside the same terminal:
    ``` bash
    # Get NLB DNS
    NLB_DNS=$(aws elbv2 describe-load-balancers --load-balancer-arns $NLB_ARN|jq '.LoadBalancers[]|.DNSName' -r)
    # Invoke the GET API
    curl -s $NLB_DNS/feedback |jq
    ```

You should see similar json output as previously.

### Create VpcLink

Go back to API Gateway, under __VPC Links__ create a new one and name it ___reinforce-vpclink___ as following:

![](../screenshots/vpc-link1.png)

Note: It might take a mintue or two to provision the VPC link.

### Create a new API to consume the VpcLink

1. Create a new API ___FeedbackSvc___
    ![](../screenshots/lab-backend4.png)

2. From the __Actions__ drop-down list, create a new resource ___feedback___

3. Select the new ___feedback___ resource, from __Actions__ choose __Create Method__. Add a __GET__ method for the resource as following. Make sure the __Endpoint URL__ is the DNS name of the NLB starting with ___http___ and ends with ___feedback___, for example:
http://reinforce-lab-nlb-1f8e6550f0e7ac3f.elb.us-east-2.amazonaws.com/feedback

    Note: You have to wait until the VPC link created in the previous step is ready. Otherwise, you will not see it in the __VPC Link__ dropdown list.

    ![](../screenshots/lab-backend5.png)

    Click __Save__ to create the method and you can use the __TEST__ link to test it.

4. Create a __POST__ method in the same way by switching from GET to POST.
    ![](../screenshots/lab-backend6.png)

5. Finally, deploy the API to a __prod__ stage. You can click the link to verify the API, for example:
https://21kz57tm75.execute-api.us-east-1.amazonaws.com/prod/feedback

Note: Don't forget the append __feedback__ at the end of the endpoint.