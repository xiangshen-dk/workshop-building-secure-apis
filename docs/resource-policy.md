# Control Access to an API with Resource Policy
Time Estimate: 15 - 20 minutes  
In this module, we will use a resource policy to control the access of an example API hosted on Amazon API Gateway.


??? info "What is Amazon API Gateway"
    Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. Amazon API Gateway handles all of the tasks involved in accepting and processing up to hundreds of thousands of concurrent API calls, including traffic management, authorization and access control, monitoring, and API version management. For more information, see https://aws.amazon.com/api-gateway/. 

### Create the example API

??? info "Amazon API Gateway resource policies"
	Amazon API Gateway resource policies are JSON policy documents that you attach to an API to control whether a specified principal (typically an IAM user or role) can invoke the API. You can use API Gateway resource policies to allow your API to be securely invoked by: 1) users from a specified AWS account; 2) specified source IP address ranges or CIDR blocks; 3) specified virtual private clouds (VPCs) or VPC endpoints (in any account), see https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-resource-policies.html.

1. Go to [Amazon API Gateway](https://us-west-2.console.aws.amazon.com/apigateway/home?region=us-east-1#/ "API Gateway Console"), click the button __Create API__(or __Get Started__ if it's your first time to use API Gateway)

2. Select __Example API__ and click the button __Import__ at the bottom of the page:
  ![Create new API](../screenshots/example-api-1.png)
3. After the __Pet Store__ example API is created, select the __GET__ methods and you can see the integration endpoints on the right side. It could be a __Mock Endpoint__ or an __http endpoint__. For instance, in the following screenshot, we have http://petstore-demo-endpoint.execute-api.com/petstore/pets as the endpoint. You can also click the __TEST__ link in the middle to invoke the endpoint. It will provide the request and response for the API call.

  ![Integration endpoint](../screenshots/example-api-1a.png)
4. The API is not public visible until you deploy it. Click the button __Actions__ and select __Deploy API__:

  ![Deploy API](../screenshots/example-api-2.png)

5\. Select __New Stage__ in the 'Deployment stage' drop-down list and type ___test___ for the __Stage name__

6\. Click the button __Deploy__ and your API will be deployed.

7\. Once the API is deployed, you should have a page similar to the following:
  ![Deployed API](../screenshots/example-api-3.png)

8\. Open the invoke URL link and you will have the following page. Feel free to click the hyperlinks in the page to exlore.

  ![Pet Store API](../screenshots/example-api-4.png)

### Configure access

1. Get the IP address of your computer by click this link: [checkip](https://checkip.amazonaws.com){:target="_blank"}. We will use the IP in the next step as the 'SourceIp'. __Note:__ the 'SourceIp' field needs to be in the standard CIDR format in the resource policy. Notice the __/32__ in the screenshot below.

2. Click the link __Resource Policy__ at the left side of the Pet Store API. Click the button __IP Range Blacklist__ at the bottom of the page and replace the placeholders as following:

  ![Pet Store Resource Policy](../screenshots/resource-policy-1.png)
3. Click __Save__ and go back to __Resources__.

4\. Click the __Actions__ button and deploy the API again to the _test_ stage.

5\. Click the 'Invoke URL' and verify that you don't have access anymore. Note: you may have to wait for a few seconds and refresh a couple of times.

6\. The current error message gives away a bit more info. You can update the error message by chaning the __Mapping Templates__ under 'Access Denied[403]' in 'Gateway Responses':

  ![Pet Store API](../screenshots/example-api-5.png)

Save and redeploy the API to your test stage. Refresh a couple of times to verify the change.

7\. Go back to the resource policy and delete the policy. Redeploy the API to the test stage and verify you have the access again.