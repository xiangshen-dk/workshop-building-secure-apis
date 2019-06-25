# Cleanup

To delete the resources you created during the workshop, follow the below steps. Feel free to leave the resources running and continue playing around with them, but **be aware that this may incur additional charges against your account**.

### Delete the PetStore API
#### CLI steps (assuming you follow the exact lab steps in the previous modules):
```bash

PetStoreAPI=$(aws apigateway get-rest-apis|jq -r '.items[]|select(.name=="PetStore")|.id')

aws apigateway delete-rest-api --rest-api-id $PetStoreAPI
```
#### Alternatively, you can use the following steps from the web console:
  * Go to the [Amazon API Gateway console](https://console.aws.amazon.com/apigateway/home).
  * Select the **PetStore** API you created in Module 2 and select **Actions** --> **Delete API**.
  * Enter the name of the API to confirm and click **Delete API**.

### Delete AWS WAF resources

#### CLI steps (assuming you follow the exact lab steps in the previous modules):
```bash

WebACLId=$(aws waf-regional list-web-acls | jq -r '.WebACLs[] | select(.Name=="ProtectInput") | .WebACLId')

# Cleanup SQL injection rule
SQLinjectionRule=$(aws waf-regional list-rules|jq -r '.Rules[]|select(.Name=="SQLinjectionRule")|.RuleId')

SqlInjectionMatchSetId=$(aws waf-regional list-sql-injection-match-sets| \
  jq -r '.SqlInjectionMatchSets[]|select(.Name=="SQLinjectionMatch")|.SqlInjectionMatchSetId')

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional update-sql-injection-match-set \
--sql-injection-match-set-id $SqlInjectionMatchSetId \
--change-token $CHANGE_TOKEN \
--updates 'Action="DELETE",SqlInjectionMatchTuple={FieldToMatch={Type="QUERY_STRING"},TextTransformation="URL_DECODE"}'

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional update-sql-injection-match-set \
--sql-injection-match-set-id $SqlInjectionMatchSetId \
--change-token $CHANGE_TOKEN \
--updates 'Action="DELETE",SqlInjectionMatchTuple={FieldToMatch={Type="BODY"},TextTransformation="URL_DECODE"}'

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional update-sql-injection-match-set \
--sql-injection-match-set-id $SqlInjectionMatchSetId \
--change-token $CHANGE_TOKEN \
--updates 'Action="DELETE",SqlInjectionMatchTuple={FieldToMatch={Type="BODY"},TextTransformation="NONE"}'

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional update-sql-injection-match-set \
--sql-injection-match-set-id $SqlInjectionMatchSetId \
--change-token $CHANGE_TOKEN \
--updates 'Action="DELETE",SqlInjectionMatchTuple={FieldToMatch={Type="URI"},TextTransformation="URL_DECODE"}'

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional update-rule \
    --rule-id $SQLinjectionRule \
    --change-token $CHANGE_TOKEN \
    --updates "Action='DELETE',Predicate={Negated=false,Type='SqlInjectionMatch',DataId='$SqlInjectionMatchSetId'}"

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional delete-sql-injection-match-set \
--sql-injection-match-set-id $SqlInjectionMatchSetId \
--change-token $CHANGE_TOKEN

# Cleanup large size body rule
LargeBodyMatchRule=$(aws waf-regional list-rules|jq -r '.Rules[]|select(.Name=="LargeBodyMatchRule")|.RuleId')

SizeConstraintSetId=$(aws waf-regional list-size-constraint-sets| \
  jq -r '.SizeConstraintSets[]|select(.Name=="LargeBodyMatch")|.SizeConstraintSetId')

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional update-rule \
    --rule-id $LargeBodyMatchRule \
    --change-token $CHANGE_TOKEN \
    --updates "Action='DELETE',Predicate={Negated=false,Type='SizeConstraint',DataId='$SizeConstraintSetId'}"

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional update-size-constraint-set \
--size-constraint-set-id $SizeConstraintSetId \
--change-token $CHANGE_TOKEN \
--updates 'Action="DELETE",SizeConstraint={FieldToMatch={Type="BODY"},TextTransformation="NONE",ComparisonOperator="GE",Size=3000}'

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional delete-size-constraint-set \
--size-constraint-set-id $SizeConstraintSetId \
--change-token $CHANGE_TOKEN

# Get rate limiting rule id
RequestFloodRule=$(aws waf-regional list-rate-based-rules|jq -r '.Rules[]|select(.Name=="RequestFloodRule")|.RuleId')

# Delete rules in web acl
CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional update-web-acl --web-acl-id $WebACLId --change-token $CHANGE_TOKEN  \
  --updates "Action='DELETE',ActivatedRule={Priority=1,RuleId='${LargeBodyMatchRule}',Action={Type='BLOCK'},Type='REGULAR'}"

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional update-web-acl --web-acl-id $WebACLId --change-token $CHANGE_TOKEN  \
  --updates "Action='DELETE',ActivatedRule={Priority=2,RuleId='${SQLinjectionRule}',Action={Type='BLOCK'},Type='REGULAR'}"

CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional update-web-acl --web-acl-id $WebACLId --change-token $CHANGE_TOKEN  \
  --updates "Action='DELETE',ActivatedRule={Priority=3,RuleId='${RequestFloodRule}',Action={Type='BLOCK'},Type='RATE_BASED'}"

# Delete the web acl
CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional delete-web-acl --web-acl-id $WebACLId --change-token $CHANGE_TOKEN

# Delete the rules
CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional delete-rule --rule-id $SQLinjectionRule --change-token $CHANGE_TOKEN
CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional delete-rule --rule-id $LargeBodyMatchRule --change-token $CHANGE_TOKEN
CHANGE_TOKEN=$(aws waf-regional get-change-token | jq -r '.ChangeToken')
aws waf-regional delete-rate-based-rule --rule-id $RequestFloodRule --change-token $CHANGE_TOKEN
```

#### Alternatively, you can use the following steps from the web console:

* Go to the [WAF Console](https://console.aws.amazon.com/waf/home)
* In the navigation pane, choose **Web ACLs**.
* Choose the `ProtectInput` web ACL you created in the WAF module (you may have to filter it to your specific region).
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
#### CLI steps (assuming you follow the exact lab steps in the previous modules):
```bash
UserPoolId=$(aws cognito-idp list-user-pools --max-results 10|jq -r '.UserPools[]|select(.Name="test")|.Id')

# Try to get the domain we created in the user pool previously
CP_DOMAIN=$(grep AWS_COGNITO_USER_POOL_DOMAIN ~/environment/feedback-ui/src/env.js | cut -f1 -d.| cut -f4 -d"'")
# Delete user pool domain
aws cognito-idp delete-user-pool-domain --domain $CP_DOMAIN --user-pool-id $UserPoolId

aws cognito-idp delete-user-pool --user-pool-id $UserPoolId
```

#### Alternatively, you can use the following steps from the web console:
  * Go to the [Amazon Cognito console](https://console.aws.amazon.com/cognito/users/ "Amazon Cognito console").
  * Select the user pool you created for the workshop.
  * Select __Domain name__ on the left panel.
  * Click the button __Delete domain__.
  * Check the check box and click __Delete domain__.
  
  * Click the link __General settings__ on the top of the left panel.
  * Click the button __Delete pool__ at the upper right corner.
  * Type `delete` in the text box and click __Delete pool__.
  * Refresh your browser to make sure the user pool is deleted.

### Delete FeedbackSvc API
#### CLI steps (assuming you follow the exact lab steps in the previous modules):
```bash

FeedbackSvc=$(aws apigateway get-rest-apis|jq -r '.items[]|select(.name=="FeedbackSvc")|.id')

aws apigateway delete-rest-api --rest-api-id $FeedbackSvc
```

#### Alternatively, you can use the following steps from the web console:
  * Go to the [Amazon API Gateway console](https://console.aws.amazon.com/apigateway/home "vpc-links").
  * Select the **FeedbackSvc** API created for the workshop and select **Actions** --> **Delete API**.
  * Enter the name of the API to confirm and click **Delete API**.

### Delete VPC Link & NLB
#### CLI steps (assuming you follow the exact lab steps in the previous modules):
```bash
# Get VpcLinkId
VpcLinkId=$(aws apigateway get-vpc-links|jq -r '.items[]|select(.name=="reinforce-vpclink")|.id')

# Delete VpcLink
aws apigateway delete-vpc-link --vpc-link-id $VpcLinkId
```
__Note__: Please wait a bit until the VpcLink is completely deleted. Otherwise, you mighe have errors when remove the NLB.
```bash
# Delete the NLB and its target group
LoadBalancerArn=$(aws elbv2 describe-load-balancers|\
  jq -r '.LoadBalancers[]|select(.LoadBalancerName=="reinforce-lab-nlb")|.LoadBalancerArn')

ListenerArn=$(aws elbv2 describe-listeners --load-balancer-arn $LoadBalancerArn|\
  jq -r '.Listeners[]|.ListenerArn')

TargetGroupArn=$(aws elbv2 describe-target-groups --load-balancer-arn $LoadBalancerArn|\
 jq -r '.TargetGroups[]|.TargetGroupArn')

RuleArn=$(aws elbv2 describe-rules --listener-arn $ListenerArn|jq -r '.Rules[]|.RuleArn')

aws elbv2 delete-listener --listener-arn $ListenerArn

aws elbv2 delete-load-balancer --load-balancer-arn $LoadBalancerArn

aws elbv2 delete-rule --rule-arn $RuleArn

aws elbv2 delete-target-group --target-group-arn $TargetGroupArn
```

#### Alternatively, you can use the following steps from the web console:
  * Go to the [Amazon API Gateway console](https://console.aws.amazon.com/apigateway/home "vpc-links") --> **VPC Links**.
  * Click on the small **x** icon in the upper right corner of the box for the VPC Link you created for the workshop. Click **Delete** to confirm.
  * Go to the [Amazon EC2 console](https://us-east-1.console.aws.amazon.com/ec2/v2/home?region=us-east-1#LoadBalancers:sort=loadBalancerName) --> **Load Balancers**.
  * Select the NLB you created for the workshop, delete the listen first then select **Actions** --> **Delete**.
  * Click **Yes, Delete**.
  * Select the target group (reinforce-targets) and select **Actions** --> **Delete**.
  * Click **Yes, Delete**.

### Delete API Key & Usage Plan
#### CLI steps (assuming you follow the exact lab steps in the previous modules):
Change the name __test company__ to your API key name if it's a different one.

```bash hl_lines="1"

ApiKeyId=$(aws apigateway get-api-keys|jq -r '.items[]|select(.name="test company")|.id')

UsagePlanId=$(aws apigateway get-usage-plans --key-id $ApiKeyId|jq -r '.items[]|.id')

aws apigateway delete-usage-plan --usage-plan-id $UsagePlanId

aws apigateway delete-api-key --api-key $ApiKeyId
```

#### Alternatively, you can use the following steps from the web console:
  * Go to the [Amazon API Gateway console](https://console.aws.amazon.com/apigateway/home) --> **API Keys**.
  * Select the API key you created for the workshop and click **Delete API Key**. Confirm **Delete**.
  * Now go to **Usage Plans** and select the usage plan you created for the workshop. Click on **Actions** --> **Delete Usage Plan**. Confirm **Delete**.

### Delete CloudFormation stack

!!! Warning
    After this step, you will __NOT__ be able to use the Cloud9 instance!
  
#### CLI steps (assuming you follow the exact lab steps in the previous modules):
```bash
aws cloudformation delete-stack --stack-name ReinforceStack
```
#### Alternatively, you can use the following steps from the web console:
* Go to the [CloudFormation Console](https://console.aws.amazon.com/cloudformation/home).
* Select the `ReinforceStack` Stack.
* Under **Actions**, choose **Delete Stack** (if you are in the newest version of the console there may be a specific **Delete** button).
* Ensure that there are no errors and that the process completes successfully.

### Delete the CloudFront distribution (if you created one)

  * Go to the [Amazon CloudFront console](https://console.aws.amazon.com/cloudfront/home "Amazon CloudFront console").
  * Select the distribution you created for the workshop.
  * Click the button __Disable__ at the top.
  * Click the button __Yes, Disable__ and __Close__.
  * The value of the State column immediately changes to Disabled. Wait until the value of the Status column changes to __Deployed__.
  
!!! note "It probably will take a few minutes for the distribution to be disabled. You can come back later to delete it if necessary."
  
  * Tick the check box for the distribution that you want to delete.
  * Click __Delete__, and click __Yes, Delete__ to confirm. Then click __Close__.

### Delete S3 Bucket (if you created one)
  * Go to the [Amazon S3 console](https://s3.console.aws.amazon.com/s3/home) and tick the checkbox next to the bucket you created for the workshop.
  * Click **Delete**, type the name of the bucket to confirm deletion, and click **Confirm**.
