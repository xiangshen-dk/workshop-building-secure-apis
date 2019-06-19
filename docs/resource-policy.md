# Control Access to an API with Resource Policy
Time Estimate: 15 - 20 minutes  

In this module, we will first create a sample API in Amazon API Gateway while our CloudFormation stack launches. We'll then explore how to use a resource policy to control whether a specified principal can access that API.

??? info "What is Amazon API Gateway?"
    Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. Amazon API Gateway handles all of the tasks involved in accepting and processing up to hundreds of thousands of concurrent API calls, including traffic management, authorization and access control, monitoring, and API version management. For more information, see https://aws.amazon.com/api-gateway/. 

### Create the example API

1. Go to [Amazon API Gateway](https://console.aws.amazon.com/apigateway/ "API Gateway Console") in your AWS console and click the button __Create API__ (or __Get Started__ if it's your first time using API Gateway).

2. Select __Example API__ and click the  __Import__ button at the bottom of the page. This will create a sample pet store API.
  ![Create new API](../screenshots/example-api-1.png)

3. After the __Pet Store__ API is created, select the __GET__ method directly under __/pets__ and you will be able see the integration endpoint on the right side. In this case, our method is calling an existing HTTP endpoint (http://petstore-demo-endpoint.execute-api.com/petstore/pets), but this could similarly be a mock endpoint, a Lambda function, or even another AWS service. You can also click the __TEST__ link in the middle to invoke the endpoint and test out a sample request and response.
  ![Integration endpoint](../screenshots/example-api-1a.png)

4. The API is not publicly accessible until you deploy it. Click the __Actions__ dropdown and select __Deploy API__:
  ![Deploy API](../screenshots/example-api-2.png)

5. Select __[New Stage]__ in the __Deployment stage__ dropdown and enter ___test___ for  __Stage name__.

6. Click __Deploy__ to deploy your API to this new stage.

7. Once the API is deployed, you should have a page similar to the following:
  ![Deployed API](../screenshots/example-api-3.png)

8. Open the invoke URL and you will see the following page. Feel free to explore the various hyperlinks.

  ![Pet Store API](../screenshots/example-api-4.png)

### Configure access with resource policy

??? info "Amazon API Gateway resource policies"
	Amazon API Gateway resource policies are JSON policy documents that you attach to an API to control whether a specified principal (typically an IAM user or role) can invoke the API. You can use API Gateway resource policies to allow your API to be securely invoked by: 1) users from a specified AWS account; 2) specified source IP address ranges or CIDR blocks; 3) specified virtual private clouds (VPCs) or VPC endpoints (in any account), see https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-resource-policies.html.

1. Get the IP address of your computer by clicking this link: [checkip](https://checkip.amazonaws.com){:target="_blank"}. We will use this IP address in the next step as the 'SourceIp'. __Note:__ the 'SourceIp' field needs to be in standard CIDR format in the resource policy. Notice the appended __/32__ in the screenshot below.

2. Back in the API Gateway console, open __Resource Policy__ on the left side of the Pet Store API. Click the button __IP Range Blacklist__ at the bottom of the page and replace the placeholders as following:
  ![Pet Store Resource Policy](../screenshots/resource-policy-1.png)

3. Click __Save__ and go back to __Resources__.

4. Click the __Actions__ button and deploy the API to the ___test___ stage.

5. Click the 'Invoke URL' and verify that you can no longer access the API. This happens because in the above resource policy, we were actually telling API Gateway to explicitly "Deny" access when the source IP address is equal to your computer's. You could similarly blacklist a certain range of IP addresses, or whitelist certain AWS accounts & VPCs. 

    > Note: you may have to wait a few seconds and refresh the page a few times to see your access denied.

6. The current error message gives away a lot of info. We can modify this error message by opening __Gateway Responses__, expanding __Access Denied [403]__, and changing the __Mapping Template__:
  ![Pet Store API](../screenshots/example-api-5.png)
  Modify the above, click __Save__ and redeploy the API to the ___test___ stage. Once again, go to the invoke URL and refresh a few times to verify the change.

7. To regain access to the API, go back to the resource policy and delete everything in the text box. Click __Save__ and redeploy the API to the ___test___ stage.