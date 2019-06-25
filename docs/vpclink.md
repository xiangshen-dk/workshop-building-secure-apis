# Set up Private Integrations
Time Estimate: 15 - 20 minutes

AWS API gateway supports a variety of integrations including Lambda, public or private HTTP endpoints, and other AWS services such as S3 or DynamoDB.

In this module, you will integrate API Gateway with a web service running inside a private subnet of your VPC.

__Important Note:__ For the rest of the modules, we will need the resources provisioned by CloudFormation in Module 1. To verify these resources are ready, open the [CloudFormation Console](https://console.aws.amazon.com/cloudformation) and ensure that the __ReinforceStack__ has a status of __CREATE_COMPLETE__.

### Connect to Cloud9

2. Go to the [AWS Cloud9](https://console.aws.amazon.com/cloud9) console and click on ***Your environments*** (you may need to expand the left sidebar).

    ??? info "What is AWS Cloud9"
        AWS Cloud9 is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just an internet browser. It includes a code editor, debugger, and terminal. Cloud9 also provides a seamless experience for developing serverless applications enabling you to easily define resources, debug, and switch between local and remote execution of serverless applications. In this project, we are launching Cloud9 on its own EC2 instance, but you can also connect Cloud9 to an existing Linux server. For more information, see https://aws.amazon.com/cloud9/.
    ![Cloud 9](../images/0B-cloud9-environments.png)

3. Find the __reInforceCloud9__ environment and click the __Open IDE__ button as following:
    ![Open IDE](../screenshots/c9-1.png)

    If you have trouble opening Cloud9:
	
	* Ensure you are use either  **Chrome** or **Firefox** browser.
	* Refer to the troubleshooting guide [**here**](https://docs.aws.amazon.com/cloud9/latest/user-guide/troubleshooting.html#troubleshooting-env-loading) to ensure third-party cookies is enabled.

4. You should now see an integrated development environment (IDE)  as shown below. You can view and edit files in the editor and run shell commands in the terminal section just like you would on a local computer.

	![](../images/0B-cloud9-start.png)

	Keep your AWS Cloud9 IDE opened in a tab throughout this workshop as you'll use it to complete several of the remaining activities. 

5. In a separate tab, go to the [EC2 console](https://console.aws.amazon.com/ec2/v2/home) and examine the EC2 instance __lab-backend__. Take note of it's private IP address as shown below:

    ![](../screenshots/lab-backend1.png)

    Notice this instance is running in a private subnet and does not have a public IP address.

6. Back in Cloud9, run the following commands in the terminal:
    
    !!! note "Important!"
        You will run all terminal commands and modify all source code for these labs from within Cloud9 (**not** your local machine). Keep in mind that Cloud9 is a fully fledged IDE running on an Amazon EC2 instance. You can edit code and create new files via the Cloud9 editor in your browser. You can also open multiple terminals if needed.

    !!! tip "Pro tip"
        All of the terminal commands in these modules can be copy/pasted to your terminal for execution, but keep an eye out for any highlighted sections that may need to be changed. To do this, just click the _Copy to clipboard_ icon in the upper right corner of the greyed out section containing the commands. Then paste to your Cloud9 terminal, ensure that all of the commands finish pasting, modify any necessary parameters, and execute. 

    ``` bash
    # Install jq, a lightweight and flexible command-line JSON processor
    sudo yum install jq -y
    ```
    __Note:__ When you execute the following, be sure to replace the IP address on the highlighted line with the privte IP of the lab-backend instance we noted above. 

    ``` bash hl_lines="1"
    curl -s  http://10.0.131.162/feedback|jq
    ```
    
    You should have some output like the following:
    ![](../screenshots/lab-backend2.png)


### Create a network load balancer (NLB)

??? info "Why do we need an NLB?"
    To expose HTTP/HTTPS resources behind an Amazon VPC for access by clients outside of the VPC, we need to set up an API Gateway private integration. To do this, we use a ___VpcLink___ resource to encapsulate connections between API Gateway and the targeted resources. This resource relies on a network load balancer fronting our VPC resources and then targeting that network load balancer as a ___VpcLink___ from our API. For more information, see https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-private-integration.html.

1. Run the following commands in Cloud9 (it might take a couple seconds to paste everything):

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

2. Go to the [EC2 web console](https://console.aws.amazon.com/ec2/v2/home). On the left side under __LOAD BALANCING__, click __Target Groups__ and select __reinforce-targets__. Under the __Targets__ tab, check if the registered target is healthy. If it's in the 'initial' status, wait until it becomes healthy. This may take a minute or two while the load balancer verifies the instance is ready to receive traffic.
    ![](../screenshots/lab-backend3.png)

3. Query the NLB endpoint by running the following commands inside the same terminal:
    ``` bash
    # Get NLB DNS
    NLB_DNS=$(aws elbv2 describe-load-balancers --load-balancer-arns $NLB_ARN|jq '.LoadBalancers[]|.DNSName' -r)
    # Invoke the GET API
    curl -s $NLB_DNS/feedback |jq
    ```

You should see similar output as when you queried the instance directly.

### Create VpcLink

Go back to the [API Gateway console](https://console.aws.amazon.com/apigateway/). On the left side, click __VPC Links__ and select __Create__. Enter ___reinforce-vpcLink___ as the name and select __reinforce-lab-nlb__ as the __Target NLB__. Click __Create__.

![](../screenshots/vpc-link1.png)

__Note__: It may take a mintue or two to provision the VPC link.

### Create a new API to consume the VpcLink

1. Click the __APIs__ link on the left side, select __Create API__ and create a new API named ___FeedbackSvc___.
    ![](../screenshots/lab-backend4.png)

2. From the __Actions__ drop-down list, select __Create Resource__ and create a resource with the name ___feedback___.

3. Select the new ___feedback___ resource and from __Actions__ choose __Create Method__. Select __GET__ from the dropdown directly underneath the resource and click the small checkmark to confirm. Enter parameters for the method as shown in the below screenshot, ==except do **not** tick the checkbox for *Use Proxy Integration*==. For __Endpoint URL__, be sure to enter the DNS name of your NLB starting with ___http://___ and ending with ___/feedback___, for example:
http://reinforce-lab-nlb-XXXXXXXXXXXX.elb.us-east-2.amazonaws.com/feedback

    You can run the following command from your Cloud9 terminal to get this endpoint:
    ```bash
    echo http://$NLB_DNS/feedback
    ```

    __Note__: You have to wait until the VPC link created in the previous step is ready. Otherwise, you will not see it in the __VPC Link__ dropdown list.

    ![](../screenshots/lab-backend5.png)

    Click __Save__ to create the method and you can then use the __TEST__ link to test it. You should once again see the same output as when you queried the instance and the NLB.

4. Create a __POST__ method in the same way as above by switching from GET to POST.
    ![](../screenshots/lab-backend6.png)

5. Finally, under __Actions__, select __Deploy API__ and deploy the API to a __[New Stage]__ named __prod__. Once deployed, you can click the invoke URL to verify the API, for example:
https://21kz57tm75.execute-api.us-east-1.amazonaws.com/prod/feedback.

Note: Don't forget to append __feedback__ to your invoke URL or else you'll get an error.

We now have a simple sample API that allows users to submit feedback comments and view others' comments.