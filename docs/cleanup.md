# Cleanup

To delete the resources you created during the workshop, follow the below steps. Feel free to leave the resources running and continue playing around with them, but **be aware that this may incur additional charges against your account**.

### Delete AWS WAF resources

* Go to the [WAF Console](https://console.aws.amazon.com/waf/home)
* In the navigation pane, choose **Web ACLs**.
* Choose the `ProtectInput` web ACL you created in the WAF module.
* On the **Rules** tab in the right pane, choose **Edit web ACL**.
* Remove all rules from the web ACL by choosing the **x** at the right of the row for each rule. This doesn't delete the rules from AWS WAF, it just removes the rules from this web ACL.
* Click **Update**.
* Dissasociate the API gateway from the WAF by going to the section **AWS resources using this web ACL** in the **Rules** tab and clicking the  **x** at the right of the API gateway stage
* On the **Web ACLs** page, confirm that the web ACL that you want to delete is selected, and then choose **Delete**.
* In the navigation pane, choose **Rules**. 
* Go to each of the 3 rules we created (you may have to filter it to your specific region) and edit the rule to disassociate all of the conditions for each rule.
* Delete the rules.
* Under **Conditions**, select **Size constraints**, delete our condition's filters, and delete the condition. Repeat this for **SQL injection**.

### Delete the Cognito user pool

  * Go to the [Amazon Cognito console](https://us-west-2.console.aws.amazon.com/cognito/users/?region=us-west-2#/ "Amazon Cognito console").
  * Select the user pool you created for the workshop.
  * Select __Domain name__ on the left panel.
  * Click the button __Delete domain__.
  * Check the check box and click __Delete domain__.
  
  * Click the link __General settings__ on the top of the left panel.
  * Click the button __Delete pool__ at the upper right corner.
  * Type `delete` in the text box and click __Delete pool__.
  * Refresh your browser to make sure the user pool is deleted.

### Delete FeedbackSvc API
  * Go to the [Amazon API Gateway console](https://us-east-2.console.aws.amazon.com/apigateway/home?region=us-east-2#/vpc-links).
  * Select the **FeedbackSvc** API created for the workshop and select **Actions** --> **Delete API**.
  * Enter the name of the API to confirm and click **Delete API**.

### Delete VPC Link & NLB
  * Go to the [Amazon API Gateway console](https://us-east-2.console.aws.amazon.com/apigateway/home?region=us-east-2#/vpc-links) --> **VPC Links**.
  * Click on the small **x** icon in the upper right corner of the box for the VPC Link you created for the workshop. Click **Delete** to confirm.
  * Go to the [Amazon EC2 console](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#LoadBalancers:sort=loadBalancerName) --> **Load Balancers**.
  * Select the NLB you created for the workshop and select **Actions** --> **Delete**.
  * Click **Yes, Delete**.

### Delete the CloudFront distribution (if you created one)

  * Go to the [Amazon CloudFront console](https://console.aws.amazon.com/cloudfront/home?region=us-west-2 "Amazon CloudFront console").
  * Select the distribution you created for the workshop.
  * Click the button __Disable__ at the top.
  * Click the button __Yes, Disable__ and __Close__.
  * The value of the State column immediately changes to Disabled. Wait until the value of the Status column changes to __Deployed__.
  
!!! note "It probably will take a few minutes for the distribution to be disabled. You can come back later to delete it if necessary."
  
  * Tick the check box for the distribution that you want to delete.
  * Click __Delete__, and click __Yes, Delete__ to confirm. Then click __Close__.

### Delete S3 Bucket (if you created one)
  * Go to the [Amazon S3 console](https://s3.console.aws.amazon.com/s3/home?region=us-east-2) and tick the checkbox next to the bucket you created for the workshop.
  * Click **Delete**, type the name of the bucket to confirm deletion, and click **Confirm**.

### Delete API Key & Usage Plan
  * Go to the [Amazon API Gateway console](https://us-east-2.console.aws.amazon.com/apigateway/home?region=us-east-2) --> **API Keys**.
  * Select the API key you created for the workshop and click **Delete API Key**. Confirm **Delete**.
  * Now go to **Usage Plans** and select the usage plan you created for the workshop. Click on **Actions** --> **Delete Usage Plan**. Confirm **Delete**.

### Delete PetStore API
  * Go to the [Amazon API Gateway console](https://us-east-2.console.aws.amazon.com/apigateway/home?region=us-east-2).
  * Select the **PetStore** API you created in Module 2 and select **Actions** --> **Delete API**.
  * Enter the name of the API to confirm and click **Delete API**.

### Delete CloudFormation stack
	
* Go to the [CloudFormation Console](https://console.aws.amazon.com/cloudformation/home).
* Select the `ReinforceStack` Stack.
* Under **Actions**, choose **Delete Stack** (if you are in the newest version of the console there may be a specific **Delete** button).
* Ensure that there are no errors and that the process completes successfully.
