# Using AWS Cognito for Authentication & Authorization
Time Estimate: 15 - 20 minutes  

In this module, we will configure Amazon Cognito to authorize access to our API. Amazon Cognito lets you create customizable authentication and authorization solutions for your REST APIs. You can use Amazon Cognito to control who can invoke REST API methods in addition to using the protections we set up previously. We will then test authenticating against Cognito and posting some new data to our API.

Note that you don't have to use Cognito user pools as described below for authorization. You could also use [***IAM permissions***](https://docs.aws.amazon.com/apigateway/latest/developerguide/permissions.html) or a custom [***Lambda authorizer***](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html).

??? info "What is Amazon Cognito?"
    Amazon Cognito lets you easily add user sign-up and authentication to your mobile and web apps. Amazon Cognito also enables you to authenticate users through an external identity provider and provides temporary security credentials to access your app’s backend resources in AWS or any service behind Amazon API Gateway. Cognito will be our mechanism for enabling user authentication for access to our backend API. For more information, see https://aws.amazon.com/cognito/. 

### Create Cognito user pool

??? info "What are Amazon Cognito User Pools?"
    A user pool is a user directory in Amazon Cognito that you can use to let users sign in to your web or mobile app. With user pools, you can do things like set up user sign-up and sign-in services, social sign-in capabilities, user profiles and management, security features like MFA or email verification, and customized workflows. For more information, see here: https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html.  

1. In the AWS console, under **Services**, search for and open [**Amazon Cognito**](https://console.aws.amazon.com/cognito/home "AWS Cognito"){:target="_blank"}.

2. Click on **Manage User Pools** and select **Create a user pool**. Name it
   ```feedback-users```.

3. Select **Review defaults** and click **Create pool**.

### Create app client & user

1. On the left hand side, open **App clients** and select **Add an app
   client**. Do the following:
    - For name, enter ```feedback-app```
    - Deselect **Generate client secret**
    - Select **Enable username-password (non-SRP) flow for app-based authentication (USER_PASSWORD_AUTH)**
    - You screen should look like the below:

    ![Cognito user pool](../screenshots/screen4.png)

2. Click **Create app client** and make a note of the **App client id** value
   (e.g. ```4ea782m3eaupr6jrv84m031qo2```). We will need it later on in the lab.

3. On the left hand side, open **Users and groups** and select **Create user**. Do the following:
    - Enter a **Username** and **Temporary password**, for example ```firstuser``` and ```1234ABcd!```. Make a note of these values for later.
    - Deselect the checkboxes for **Send an invitation to this new user?** and **Mark phone number as verified?**
    - Enter an **Email**.
    - Click on **Create user**.

### Authenticate user

1. Back in Cloud9, create and save a file __auth.json__ with the contents below, being sure to replace the highlighted values with your __ClientId__, __USERNAME__, and __PASSWORD__ from the previous steps:

    ```json hl_lines="3 5 6"
    {
        "AuthFlow": "USER_PASSWORD_AUTH",
        "ClientId": "4ea782m3eaupr6jrv84m031qo2",
        "AuthParameters": {
            "USERNAME": "firstuser",
            "PASSWORD": "1234ABcd!"
        }
    }
    ```

2. In your Cloud9 terminal, run the below to authenticate against Cognito:

    ```bash
    aws cognito-idp initiate-auth --cli-input-json file://auth.json
    ```

3. You should get a challenge asking for a new password (since this is our first time authenticating). Make a note of the ==*Session*== string from the response in a new file __auth2.json__ with the below contents. Be sure to change it to your unique values for __ClientId__, __Session__, and __USERNAME__ and notice that we are changing this user's password to **NEW_PASSWORD**'s value (which you can change if you'd like).

    ```json hl_lines="2 4 6"
    {
        "ClientId": "22fs8b9vcq5kvkqdsom82lb0c2",
        "ChallengeName": "NEW_PASSWORD_REQUIRED",
        "Session": "g3P-wtle3yPDqoS8AvY5Z85Hg_WA1dG......oGBdIyrqCp0YlQz_p1Iw",
        "ChallengeResponses": {
            "USERNAME": "firstuser",
            "NEW_PASSWORD": "AWS@dmin12345"
        }
    }
    ```
Make a note of the new password. You will need it to login to the application.

4. Now, run the command:

    ```bash
    aws cognito-idp respond-to-auth-challenge --cli-input-json file://auth2.json
    ```

    Alternatively, you can just run the command with arguments in the command line, like the below:

    !!! note "If you already ran the command above, you don't need to run this one again."
        ```bash
        aws cognito-idp respond-to-auth-challenge --client-id 4ea782m3eaupr6jrv84m031qo2 \
        --challenge-name NEW_PASSWORD_REQUIRED \
        --challenge-responses '{"USERNAME": "firstuser", "NEW_PASSWORD": "AWS@dmin12345"}' \
        --session {value from the previous command at step #2}
        ```

    Make a note of the ==*IdToken*== in the response. We'll need to use this token to authorize access with our API.

### Add authorizer to API Gateway

1. In another browser tab, open [**API Gateway**](https://console.aws.amazon.com/apigateway/home "API Gateway"){:target="_blank"} in the AWS console. Select the __FeedbackSvc__ API and click on **Authorizers** in the menu. 

2. Click on **Create New Authorizer** and enter the following values:
    - For **Name**, enter ```feedback-authorizer```.
    - For **Type**, select **Cognito**. 
    - In the field for **Cognito User Pool**, select the `feedback-users` user pool we created earlier. 
    - In **Token Source**, enter ```Authorization```. Your screen should look like the below:

    ![](../screenshots/screen7.PNG)

3. Click on **Create** and then select __Test__ for our newly created authorizer. Enter the IdToken we got in the previous command for the **Authorization (header)** value. Verify you get a 200 response with claims about your user.

4. Now, we need to add this authorizer to our POST method to ensure that only authorized users can post data to our API.
    - Open **Resources** in the menu for the **FeedbackSvc** API. 
    - Select **/feedback** and then open the **POST** method for that resource. 
    - Open **Method Request**. Click on the small pencil icon next to **Authorization** and select the Cognito user pool authorizer that we just created (you may have to refresh the page for this to show up). 
    - Click on the checkmark next to the dropdown to confirm your selection. The Method Request should now look like the below:
    
    ![](../screenshots/screen8.PNG)

5. Now, under **Actions**, select **Deploy API** and choose `prod` as the **Deployment Stage**. Click **Deploy**.

### Test adding feedback

1. Back in Cloud9, test adding some new feedback with the below command, being sure to replace ```eyJraW......rCtYSnaA``` in the Authorization header with your own ==*IdToken*== value.

    ```bash hl_lines="1"
    curl -H "Authorization: eyJraW......rCtYSnaA" \
    -X POST $API_ENDPOINT \
    -H "Content-Type: application/json" \
    --data @- <<REQUEST_BODY
        {  
        "name":"auth-user-test",
        "comment": "It is a test for authorization.",
        "imageUrl":"https://en.wikipedia.org/wiki/Unicorn#/media/File:Oftheunicorn.jpg",
        "star": 4
        }
    REQUEST_BODY
    ```
    If successful, you should see an output of _{"result": "ok"}_.

2. Now to test reading the new feedback:

    ```bash
    curl -s -H"x-api-key: $API_KEY" $API_ENDPOINT | jq
    ```

    If successful, you should see a response containing the feedback you just submitted.
