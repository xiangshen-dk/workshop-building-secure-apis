# Usage Plan
Time Estimate: 10 - 15 minutes

You can leverage Usage Plans with Amazon API Gateway to set limits on request rate for consumers of your API to protect it from being abused by a particular misbehaving client.

To tally the number of requests based on the caller, API Gateway uses API Keys to keep track of different consumers for your API. In our use case, requests coming from different companies can be calculated separately. 

## Create an API Gateway usage plan 
1. In the API Gateway console, go to **Usage Plans** tab, and click **Create** 
1. Fill in the details for the usage plan 
	
	* **Name**: ```Basic```
	* **Description** : ```Basic usage plan for partners```
	* **Enable throttling**: check yes
	* **Throttling Rate** : ```1``` request per second
	* **Throttling Burst** : 1 
	* **Enable Quota**: check yes and use ```100``` requests per ```month```

	![Create Usage Plan screenshot](images/create-usage-plan.png)
	
	Click **Next**
	
1. Associate the API we created previously with the usage plan. Pick `prod` stage. And click the checkmark to confirm. Then click **Next**

	![add stage to Usage plan](images/5A-add-stage-to-plan.png)


1. We currently don't have any API keys set up. In this step, click **Create API Key and add to Usage Plan** to create an API key for users.

	* For Name, pick any name e.g.  `test company`.
	* For API Key, select **Auto Generate**
	* Click **Save**

  ![](../images/5A-auto-generate-API-key.png)

	
1. After the API key has been created, click **Done**.

	![](images/5A-API-key-created.png)
	
## Update API Gateway to enforce API keys

Now, we need to modify our API gateway so requests must have an API key present.

1. Go the [API Gateway console](https://console.aws.amazon.com/apigateway/home), navigate to the **FeedbackSvc API** -> **Resources** --> Pick the first GET method --> click on **Method Request**. 

  Set **API Key Required** field to `true`
  ![](../images/5B-confirm-usage-plan-requirement.png)

2\. Deploy the API again to the __prod__ stage

## Test request with API keys

1\. Go back to your terminal and use curl to invoke the API. Now the API is enforcing API keys, the request will fail if you don't include the API key header.

Try sending the following request using Postman like you did before. You should see the request fail with a `{"message": "Forbidden"}` response.

```bash
curl https://8ymfcyzexk.execute-api.us-east-2.amazonaws.com/prod/feedback
```

2\. You can add the API key request header when using curl, for example:
  
```bash
curl -H"x-api-key: o1JHyy8AKZXOT5HZiNn06Yse2L8r6yz8HShW0zM1" https://8ymfcyzexk.execute-api.us-east-2.amazonaws.com/prod/feedback
```
  
You can find the auto-generated API key value by going to the **API Keys** tab in the API gateway console --> click on the API key you created in previous steps --> click **Show** next to **API Key**

  ![](../images/5C-find-api-key.png)


You should now see the request go through

## Extra credit

If you want extra credit (karma points), here are some ideas:

* Try viewing/downloading the usage data for a given client. 
	
	> **Hint**: See [here](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-create-usage-plans-with-console.html#api-gateway-usage-plan-manage-usage) on some documentation 

* Try configure different throttling thresholds for different API methods

	> **Hint**: See [here](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html#apig-request-throttling-stage-and-method-level-limits) on some documentation 
