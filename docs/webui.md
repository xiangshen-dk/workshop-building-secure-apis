# Create the web UI
Time Estimate: 20 - 25 minutes  

In this module, we will download some sample source code for a Vue.js frontend web UI that will use our REST API as its backend. We'll hook this frontend up with our `FeedbackSvc` API and then test it from our Cloud9 development environment. In doing so, you'll see how a real application might interact with our API.

### Download the code 

In your Cloud9 terminal, download the source code for our frontend web UI by running the following commands:

```bash
  aws s3 cp s3://workshop.reinforce.awsdemo.me/feedback-src.tar.gz /tmp/
  cd ~/environment/
  tar xvfz /tmp/feedback-src.tar.gz
```

### Build the code

Now upgrade Node.js and build the code with the following commands:

```bash
cd feedback-ui
npm install
npm run build

```

### Run the app inside Cloud9

To run the app locally within Cloud9, run:

```bash
npm run serve
```

Make sure you don't have any compile errors.

### Preview the application

Select **Preview** > **Preview Running Application**. You will get an error. To fix that, copy the URL from your preview window (without https:// or /), e.g. `xxxxxxx.vfs.cloud9.us-west-2.amazonaws.com`. Use that to update the URL in the file __vue.config.js__ under the project root directory.

![Cloud9 preview](../screenshots/screen9.png)

Refresh the preview window, you should have a web page displayed. If you still have problems, stop npm and run `npm run serve` again.

!!! Warning "Not all features are working yet. Additional configuration is needed from the following steps."

### Configure Cognito for authentication

Now, we need to do some additional configuration in Cognito so that users can sign-in via Cognito from our new frontend. Go back to your [**Cognito**](https://console.aws.amazon.com/cognito/home "AWS Cognito") user pool and in __App client settings__, do the following:

* Enable __Cognito User Pool__ for webapp (mark the checkbox under **Enabled Identity Providers**)
* Enter the entire Cloud9 preview URL with ==__/callback__== appended as the callback URL. For example:

    `https://02b97ba28ee44121bc0e2ed09a4e6e99.vfs.cloud9.us-west-2.amazonaws.com/callback`

* Enable **Implict grant** with __openid__ and __profile__ in __Allowed OAuth scopes__
* Confirm your selections look like the below and save changes  

  ![App client settings](../screenshots/clientsettings.png)

Now, select __Domain name__ on the left panel, type in a name in the `Your domain name` field. You can use any valid name as long as it's unique. (for example: _feedback-users-shenx_). Click **Save changes**.


Make a note of this domain name, for example: `https://feedback-users-shenx.auth.us-west-2.amazoncognito.com`

### Update UI configuration

Now, we need to update our frontend to use your unique resources. Go back to your Cloud9 editor and open the **env.js** file under the **src** directory. Update it with your unique __domain URL__, __client ID__, __API GW endpoint__, and __API Key__. For example:
```javascript
export default {
  'AWS_COGNITO_USER_POOL_DOMAIN': 'feedback-users-shenx.auth.us-west-2.amazoncognito.com',
  'AWS_COGNITO_CLIENT_ID': '4ea782m3eaupr6jrv84m031qo2',
  'API_BASE_URL': 'https://tzr1eokf4k.execute-api.us-west-2.amazonaws.com/Prod',
  'API_KEY': 'your-api-key-here'
}
```

Refresh your preview window and you should see the web page.

### Configure API gateway for [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing "CORS")

Now, to successfuly integrate our app with API Gateway, we need to enable cross-origin resource sharing (CORS) for our API. CORS is a browser security feature that restricts cross-origin HTTP requests that are initiated from scripts running in the browser. 

Go to [**API Gateway**](https://us-west-2.console.aws.amazon.com/apigateway/home?region=us-west-2 "API GW"), select the API we created, and do the following:

* Select the resource __/feedback__ and open the __Actions__ dropdown.

  ![API Gateway - actions](../screenshots/api-gw-0.png)

* Select __Enable CORS__ and click the button __Enable CORS and replace existing CORS headers__.

    - Ignore any errors that pop up. We will fix them in later steps.

* A new method __OPTIONS__ has been created. Select it, then click __Method Response__.

  ![API Gateway - method reponse](../screenshots/api-gw-1.png)

 * Make sure you have the following:

  ![API Gateway - status 200](../screenshots/api-gw-3.png)

* Click the link __<-Method Execution__ to return to the previous page

  ![API Gateway - back](../screenshots/api-gw-4.png)

* Under __Integration Response__, expand the line by clicking the black arrow.

* Make sure you have the folllowing __Header Mappings__ under the '200' response:

|Response header              |Mapping value|
|-----------------------------|-------------|
|Access-Control-Allow-Headers |'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'|	
|Access-Control-Allow-Methods|'POST,GET,OPTIONS'|
|Access-Control-Allow-Origin |'*'|

* It should look like the following:

  ![API Gateway - header](../screenshots/api-gw-5.png)

* Click the __Actions__ button again. Select __Deploy API__, choose the __Prod__ stage and click __Deploy__.

### Test from Cloud9

Back in your Cloud9 preview window, try signing into the app and posting some new feedback. If all is configured properly, it should now work.

!!! note "Use the credentials you created in the previous step, e.g. __firstuser__/__AWS@dmin12345__"
!!! Warning
    If you get an error when you try to sign in from the Cloud9 preview window, you may have to pop out the preview into a new window on your browser by clicking on the button highlighted below:
    ![Preview sign in error](../screenshots/previewpopout.png)
  


