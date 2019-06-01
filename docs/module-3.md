# Create the web UI
Time Estimate: 20 - 25 minutes  
In this module, we will download the source code for the frontend web UI of our application, hook it into our API, and test using it from our development environment.

### Download the code 

In your Cloud9 terminal, download the source code for our frontend web UI by running the following commands:

```bash
  aws s3 cp s3://shenx-demo-public/workshop/feedback-ui.tar.gz /tmp/
  cd ~/environment/
  tar xvfz /tmp/feedback-ui.tar.gz
```

### Build the code

Now upgrade Node.js and build the code with the following commands:

```bash
cd feedback-ui
nvm install 8.12.0
nvm use 8.12.0
npm install
npm run build

```

### Run the app inside Cloud9

To run the app locally within Cloud9, run:

```bash
npm run serve
```

Make sure you don't have compile errors.

### Preview the application

Select **Preview** > **Preview Running Application**. You will get an error. To fix that, copy the URL from your preview window, e.g. `xxxxxxx.vfs.cloud9.us-west-2.amazonaws.com`. Use that to update the URL in the file __vue.config.js__ under the project root directory.

![Cloud9 preview](../screenshots/screen9.png)

Refresh the preview window, you should have a web page displayed. If you still have problems, stop npm and run `npm run serve` again.

!!! Warning "Not all features are working yet. Additional configuration is needed from the following steps."

### Configure Cognito for authentication

Go back to your [**Cognito**](https://us-west-2.console.aws.amazon.com/cognito/home?region=us-west-2 "AWS Cognito") user pool and in __App client settings__, do the following:

* Enable __Cognito User Pool__ for webapp (mark the checkbox under **Enabled Identity Providers**)
* Add Cloud9 preview URL with ==__/callback__== to the callback URL. For example:

    `https://02b97ba28ee44121bc0e2ed09a4e6e99.vfs.cloud9.us-west-2.amazonaws.com/callback`

* Enable Implict grant with __openid__ and __profile__ in __Allowed OAuth scopes__
* Confirm your selections look like the below and save changes  

  ![App client settings](../screenshots/clientsettings.png)

Now, select __Domain name__ on the left panel, type in a name in the field _your domain name_. You can use any valid name as long as it's unique. (for example: _feedback-users-shenx_). Save changes.


Make a note for the domain name, for example: `https://feedback-users-shenx.auth.us-west-2.amazoncognito.com`

### Update UI configuration

Go back to your Cloud9 editor and open the **env.js** file under the **src** directory. Update it with the __domain URL__, __client ID__, and __API GW endpoint__. For example:
```javascript
export default {
  'AWS_COGNITO_USER_POOL_DOMAIN': 'feedback-users-shenx.auth.us-west-2.amazoncognito.com',
  'AWS_COGNITO_CLIENT_ID': '4ea782m3eaupr6jrv84m031qo2',
  'API_BASE_URL': 'https://tzr1eokf4k.execute-api.us-west-2.amazonaws.com/Prod'
}
```

Refresh your preview window and you should see the web page.

### Configure API gateway for [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing "CORS")

Go to [**API Gateway**](https://us-west-2.console.aws.amazon.com/apigateway/home?region=us-west-2 "API GW"), select the API we created, and do the following:

* Select the resource __/feedback__ and open the __Actions__ dropdown.

  ![API Gateway - actions](../screenshots/api-gw-0.png)

* Select __Enable CORS__ and click the button __Enable CORS and replace existing CORS headers__.

    - Ignore the errors in the reponse. We will fix them in later steps.

* A new method __OPTIONS__ has been created. Select it, then click __Method Response__.

  ![API Gateway - method reponse](../screenshots/api-gw-1.png)

* Click the link __+ Add Response__ and add HTTP status code __200__. Click the {++Create++} checkmark to save.

  ![API Gateway - status 200](../screenshots/api-gw-2.png)

* Expand the code 200. Add below names to __Response Headers for 200__ one by one

    |Name             |
    |-----------------------------|
    |Access-Control-Allow-Headers|
    |Access-Control-Allow-Methods|
    |Access-Control-Allow-Origin|


 * The final result should look like this:

  ![API Gateway - status 200](../screenshots/api-gw-3.png)

* Click the link __<-Method Execution__ to return to the previous page

  ![API Gateway - back](../screenshots/api-gw-4.png)

* Under __Integration Response__, expand the line by clicking the black arrow.

* Add the folllowing to __Header Mappings__:

|Response header              |Mapping value|
|-----------------------------|-------------|
|Access-Control-Allow-Headers |'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'|	
|Access-Control-Allow-Methods|'POST,GET,OPTIONS'|
|Access-Control-Allow-Origin |'*'|

* The result looks like the following:

  ![API Gateway - header](../screenshots/api-gw-5.png)

* Click the __Actions__ button again. Select __Deploy API__; select the __Prod__ stage and click __Deploy__.

### Test from Cloud9

Back in your Cloud9 preview window, login to the app and post some feedback. If all is configured properly, it should now work.
!!! note "Use the credential you created in the previous step, e.g. __firstuser__/__AWS@dmin12345__"
!!! Warning
    If you get an error when you try to sign in from the Cloud9 preview window, you may have to pop out the preview into a new window on your browser by clicking on the button highlighted below:
    ![Preview sign in error](../screenshots/previewpopout.png)
  


