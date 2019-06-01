# Cleanup

### Delete the AWS WAF if you created one

* Go to the [WAF Console](https://console.aws.amazon.com/waf/home)
* In the navigation pane, choose **Web ACLs**.
* Choose the `ProtectInput` web ACL you created in the WAF odule
* On the **Rules** tab in the right pane, choose Edit web ACL.
* Remove all rules from the web ACL by choosing the **x** at the right of the row for each rule. This doesn't delete the rules from AWS WAF, it just removes the rules from this web ACL.
* Choose **Update**
* Dissasociate the API gateway from the WAF by going to the section **AWS resources using this web ACL** in the **Rules** tab and clicking the  **x** at the right of the API gateway stage
* On the **Web ACLs** page, confirm that the web ACL that you want to delete is selected, and then choose **Delete**.
* In the navigation pane, choose **Rules**. 
* Go to each of the 3 rules we created, edit the rule to disassociate all the conditions for each rule
* Delete the rules
* Delete the 3 conditions we created in the workshop

### Delete the Cognito user pool

  * Go to [Amazon Cognito console](https://us-west-2.console.aws.amazon.com/cognito/users/?region=us-west-2#/ "Amazon Cognito concole")
  * Select the user pool you created for the workshop
  * Select __Domain name__ on the left panel
  * Click the button __Delete domain__
  * Check the check box and click __Delete domain__
  
  * Click the link __General settings__ on the top of the left panel
  * Click the button __Delete pool__ at the upper right corner
  * Type __delete__ in the text box and click __Delete pool__
  * Refresh your browser to make sure the user pool is deleted

### Delete the CloudFront distribution

  * Go to [Amazon CloudFront console](https://console.aws.amazon.com/cloudfront/home?region=us-west-2 "Amazon CloudFront concole")
  * Select the distribution you created for the workshop
  * Click the button __Disable__ at the top
  * Click the button __Yes, Disable__ and __Close__
  * The value of the State column immediately changes to Disabled. Wait until the value of the Status column changes to __Deployed__.
  
!!! note "It probably will take a few minutes for the distribution to be disabled. You can come back late to delete it if you are in a hurry"
  
  * Check the check box for the distribution that you want to delete.
  * Click __Delete__, and click __Yes, Delete__ to confirm. Then click __Close__.
  
### Delete `ReinforStack` CloudFormation stack
	
* Go to the [CloudFormation Console](https://console.aws.amazon.com/cloudformation/home)
* Select the `ReinforStack` Stack
* Under **Actions**, choose **Delete Stack**
