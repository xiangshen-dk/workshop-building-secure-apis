# Input validation on API Gateway
Time Estimate: 10 - 15 minutes

A quote from the OWASP website: 

> "*The most common web application security weakness is the failure to properly validate input from the client or environment*."
> 
>  --- [**OWASP** (The Open Web Application Security Project)](https://www.owasp.org/index.php/Data_Validation)

A great [XKCD comic](https://xkcd.com/327/) demonstrate this point well: 

![xkcd exploits_of_a_mom ](https://imgs.xkcd.com/comics/exploits_of_a_mom.png)

You can configure API Gateway to perform basic validation of an API request before proceeding with the integration request. When the validation fails, API Gateway immediately fails the request, returns a 400 error response to the caller, and publishes the validation results in CloudWatch Logs. This reduces unnecessary calls to the backend. More importantly, it lets you focus on the validation efforts specific to your application.

For example, let's assume that in our application, we want to be sure that all posted feedback contains the following:

 - A **name** for the user.
 - A URL for an **image**.
 - A string for the **comment**.
 - An integer between 1 and 10 for **stars**.

This information should be in the request body to submit new feedback and follow certain patterns, e.g. the imageUrl should be a valid URL and the star should be a numeric value.

By leveraging input validation on API Gateway, you can enforce such required parameters as well as the regex patterns each parameter must adhere to. This allows you to remove boilerplate validation logic from your backend implementations and focus on actual business logic and deep validation.

## Create a model for your Customizations

In API Gateway, a [**model**](https://docs.aws.amazon.com/apigateway/latest/developerguide/models-mappings.html#models-mappings-models) defines the data structure of a payload, using the [JSON schema draft 4](https://tools.ietf.org/html/draft-zyp-json-schema-04).

When we define our model, we can ensure that the parameters we are receiving are in the format we are expecting. Furthermore, you can check them against regex expressions. A good tool to test if your regex is correct is [regexr.com](https://regexr.com/). 

For our **POST /feedback** API, we are going to use the following model:

```json
{
  "title": "Customizations",
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "required": [
    "imageUrl",
    "name",
    "comment",
    "star"
  ],
  "properties": {
    "imageUrl": {
      "type": "string",
      "title": "The Imageurl Schema",
      "pattern": "^https?:\/\/[-a-zA-Z0-9@:%_+.~#?&//=]+$"
    },
    "name": {
      "type": "string",
      "title": "The name Schema",
      "pattern": "^[a-zA-Z0-9- ]+$"
    },
    "comment": {
      "type": "string",
      "title": "The comment Schema"
    },
    "star": {
      "type": "number",
      "minimum": 1,
      "maximum": 10
    }
  }
}
```

Now, follow these steps:

1. Go back to the [API Gateway console](https://console.aws.amazon.com/apigateway).
2. Click on the API **FeedbackSvc**.
3. Click on **Models**.
4. Click **Create** and create a model with the following values:
	- Model name: `FeedbackPost`
	- Content type: `application/json`
1. In the model schema, use the one provided above (the *json* before this section).
1. Once everything is entered like below, click on **Create model**.
	
	![Create model](../images/06_api_model.png)

Once we have created our model, we need to apply it to our POST /feedback method.

1. Within the API Gateway Console, expand **FeedbackSvc** and select **Resources**.
1. Under **/feedback** open the **POST** method.
1. Click on **Method Request**.

	![Customizations ](../images/06_customizations.png)

1. Next to **Request Validator**, click on the pencil to edit. Select **Validate Body**. Then, click on the tick to confirm the change.
1. Under **Request Body**, click on **Add model** with the following values:
	- Content type: `application/json`
	- Model name: `FeedbackPost`
1. Click to the tick to confirm.

	![Method Execution](../images/06_method_execution.png)
	
	> On step number 2 you might have noticed that we can also validate query parameters and request headers in addition to the request body. This is useful when our application uses both at the same time and we want to have complex validations. If you want to find more information, see our documentation [here](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-method-request-validation.html).

1. Now it's time to deploy and test! Go to the **Actions** menu and click on **Deploy API**. Select `prod` as the **Deployment stage** and confirm by clicking **Deploy**. Take note of the invoke URL for testing.

## Test your Validation

Using curl, you can try making requests to the **POST /feedback** API using invalid parameters and see the input validation kick in. 

### Wrong parameters = Invalid request

Here are some example request bodies that fail:

!!! danger "Please replace the BASE_URL value with your invoke URL from the deployed stage."

* Test case #1: missing fields

    ```bash hl_lines="1"
    export BASE_URL=https://8ymfcyzexk.execute-api.us-east-2.amazonaws.com/prod
    ```

    ```bash
    curl -X POST $BASE_URL/feedback \
    -H "Content-Type: application/json" \
    --data @- <<REQUEST_BODY
      {  
        "name":"test user",
        "imageUrl":"https://en.wikipedia.org/wiki/Cherry#/media/File:Cherry_Stella444.jpg"
      }
    REQUEST_BODY
    ```

* Test case #2: the `imageUrl` is not a valid URL

    ```bash
    curl -X POST $BASE_URL/feedback \
    -H "Content-Type: application/json" \
    --data @- <<REQUEST_BODY
      {  
        "name":"test user",
        "imageUrl":"htt://en.wikipedia.org/wiki/Cherry#/media/File:Cherry_Stella444.jpg",
        "comment": "It is a test" ,
        "star": 4
      }
    REQUEST_BODY
    ```

* Test case #3: the `star` parameter is not a number

    ```bash
    curl -X POST $BASE_URL/feedback \
    -H "Content-Type: application/json" \
    --data @- <<REQUEST_BODY
      {  
        "name":"test user",
        "imageUrl":"https://en.wikipedia.org/wiki/Unicorn#/media/File:Oftheunicorn.jpg",
        "comment": "It is a test" ,
        "star": "4a"
      }
    REQUEST_BODY
    ```


For all of the above, you should get a 400 Bad Request response: 

  ```javascript
  {"message": "Invalid request body"}
  ```


### Correct parameters

Test the **POST /feedback** API with correct parameters:

```bash
curl -X POST $BASE_URL/feedback \
-H "Content-Type: application/json" \
--data @- <<REQUEST_BODY
	{  
	  "name":"test user",
    "imageUrl":"https://en.wikipedia.org/wiki/Unicorn#/media/File:Oftheunicorn.jpg",
    "comment": "It is a test" ,
    "star": 4
	}
REQUEST_BODY
```

The result should be:

```javascript
{"result": "ok"}
```

You can run the following command to verify if the POST call is successful.
```bash
curl -s $NLB_DNS/feedback |jq
```
You should be able to see that the last record is now returned from the API.

## Additional input validation options

As you have now seen, API Gateway input validation gives you basic features such as type checking and regex matching. In a production application, this is often not enough and you may have additional constraints on the input to your API. 

To further protect your API, you should consider the below in addition to the input validation features discussed here:

* Add an AWS WAF ACL to your API Gateway - check out [**Module 5**](../waf/).
* Add further input validation logic to your application code.