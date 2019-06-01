# Using AWS Cognito to protect the APIs
Time Estimate: 15 - 20 minutes  
In this module, we will configure our API to use Amazon Cognito for authorization and test authenticating and posting some data to our API.

### Create Cognito user pool

??? info "What is Amazon Cognito"
    Amazon Cognito lets you easily add user sign-up and authentication to your mobile and web apps. Amazon Cognito also enables you to authenticate users through an external identity provider and provides temporary security credentials to access your app’s backend resources in AWS or any service behind Amazon API Gateway. Cognito will be our mechanism for enabling user authentication for access to our backend API. For more information, see https://aws.amazon.com/cognito/. 

1. Now, under **Services**, search for and open [**Cognito**](https://us-west-2.console.aws.amazon.com/cognito/home?region=us-west-2 "AWS Cognito"){:target="_blank"}.

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

1. Back in Cloud9, create and save a file __auth.json__ like the below, being sure to replace the highlighted values with your __ClientId__, __USERNAME__, and __PASSWORD__ from the previous steps:

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

2. In your Cloud9 terminal, run:

    ```bash
    aws cognito-idp initiate-auth --cli-input-json file://auth.json
    ```

3. Make a note of the ==*Session*== string from the response in a new file like the below named __auth2.json__. Be sure to also fill in your values for __ClientId__ and __USERNAME__ and notice we are changing this user's password to **NEW_PASSWORD**'s value (which you can change if you'd like).

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

    Make a note of the ==*IdToken*== in the response.

### Add authorizer to API Gateway

1. In another browser tab, open [**API Gateway**](https://us-west-2.console.aws.amazon.com/apigateway/home?region=us-west-2 "API Gateway"){:target="_blank"} in the AWS console. Select the API __awscodestar-feedback-svc-xxxxxx__ and click on **Authorizers** in the menu. This API was automatically deployed by our CodePipeline based on the provided source code.

2. Click on **Create New Authorizer** and enter the following values:
    - For **Name**, enter ```feedback-authorizer```.
    - For **Type**, select **Cognito**. 
    - In the field for **Cognito User Pool**, select the user pool we created earlier. - In **Token Source**, enter ```Authorization```. Your screen should look like the below:

    ![](../screenshots/screen7.PNG)

3. Click on **Create** and then select __Test__ for our newly created authorizer. Enter the IdToken we got in the previous command for the **Authorization (header)** value and verify you get a 200 response with claims about your user.

4. Now, we need to add this authorizer to our POST method.
    - Open **Resources** in the menu for your API. 
    - Select **/feedback** and then open the **POST** method for that resource. 
    - Open **Method Request**. Click on the small pencil icon next to **Authorization** and select the Cognito user pool authorizer that we just created (you may have to refresh the page for this to show up). 
    - Click on the checkmark next to the dropdown to confirm your selection. The Method Request should now look like the below:
    
    ![](../screenshots/screen8.PNG)

5. Now, under **Actions**, select **Deploy API** and choose ***Prod*** as the **Deployment Stage**. Click **Deploy**. Make a note of the **Invoke URL** for your API's Prod Stage, e.g. `https://oq0x443wv7.execute-api.us-west-2.amazonaws.com/Prod`

### Test adding feedback

1. Back in Cloud9, test adding some new feedback with the below command, being sure to replace both ```eyJraW......rCtYSnaA``` in the Authorization header with your own ==*IdToken*== value and the URL with your own API's prod stage invoke URL with ==/feedback== appended.

    ```bash
    curl -H "Authorization: eyJraW......rCtYSnaA" \
      -d '{"name":"test_user", "feedback": "test feedback"}' \
      -X POST https://xxxxxx.execute-api.us-west-2.amazonaws.com/Prod/feedback
    ```

    - If successful, you should see an output of *{"Output": "Data Saved"}*

2. Now to test reading the new feedback:

    ```bash
    curl  https://xxxxxxxxxx.execute-api.us-west-2.amazonaws.com/Prod/feedback
    ```

    - If successful, you should see a response containing the feedback you just submitted.
