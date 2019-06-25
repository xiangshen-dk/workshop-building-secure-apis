# [Optional] Host the static site
Time Estimate: 15 - 20 minutes  

In this module, we will deploy the web UI of our application to Amazon S3 and front it with a CloudFront distribution to cache content and serve HTTPS requests. Note that this module is optional, but will show you how to take our sample frontend UI and deploy it to Amazon S3 for public consumption. 

### Create S3 bucket to host the static files

??? info "What is Amazon S3?"
    Amazon S3 is object storage built to store and retrieve any amount of data from anywhere on the Internet. It’s a simple storage service that offers durable, highly available, and infinitely scalable data storage infrastructure at very low costs. Amazon S3 can also be configured to host static websites, serving HTML, JavaScript, images, and other files with no server management. For more information, see https://aws.amazon.com/s3/. 

In Cloud9, open a new terminal by clicking the \+ sign next to the current one and select __New Terminal__. For example:

  ![Cloud9 new terminal](../screenshots/c9-terminal.png)

Choose a unique name for your bucket and run the following command in the Cloud9 terminal:

```bash
export BUCKET_NAME={bucket name}
```
```bash
echo "export BUCKET_NAME=$BUCKET_NAME" >> ~/.bashrc
aws s3 mb s3://$BUCKET_NAME
```
For example:

```bash
export BUCKET_NAME=reinforce-workshop-web-shenx
echo "export BUCKET_NAME=$BUCKET_NAME" >> ~/.bashrc
aws s3 mb s3://$BUCKET_NAME
```

Make a note of the bucket name. We will need it later.

### Rebuild code

Now, let's rebuild our code to be sure we have the most up to date deployment package.

```bash
  cd ~/environment/feedback-ui
  npm run build
```

### Copy the web pages to the bucket

Now, to copy the code from Cloud9 to our newly created S3 bucket, run:

```bash
cd ~/environment/feedback-ui/dist && \
  aws s3 sync --acl public-read . s3://$BUCKET_NAME/
```

!!! info "The files need to be public readable for the users. You could configure a [bucket policy](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteAccessPermissionsReqd.html "S3 bucket polic for public web site") for that. Here we use the '--acl public-read' option for individual files. Therefore, we avoid making the entire bucket public readable."

### Create a static web site using the S3 bucket
To configure our S3 bucket to act as a static website, run the following:
```bash
aws s3 website s3://$BUCKET_NAME/ \
  --index-document index.html --error-document index.html 
```

### Create a CloudFront distribution

??? info "What is Amazon CloudFront?"
    Amazon CloudFront is a content delivery network (CDN) service that gives businesses and web application developers an easy and cost effective way to distribute content to end users with low latency and high data transfer speeds. Here, it will also enable HTTPS for our frontend hosted in Amazon S3. For more information, see https://aws.amazon.com/cloudfront/. 

Create a file __cloudfront-config.json__ with the following content, ==being sure to replace __{bucket name}__ and __{region}__ with your choices==(Note: for us-east-2, you also have to replace - with . before the region):

```javascript hl_lines="11 12 22"
{
  "CallerReference": "web-distribution-2018-11",
  "Aliases": {
    "Quantity": 0
  },
  "DefaultRootObject": "index.html",
  "Origins": {
    "Quantity": 1,
    "Items": [
      {
        "Id": "{bucket name}",
        "DomainName": "{bucket name}.s3-website-{region}.amazonaws.com",
        "CustomOriginConfig": {
          "HTTPPort": 80,
          "HTTPSPort": 443,
          "OriginProtocolPolicy": "http-only"
        }
      }
    ]
  },
  "DefaultCacheBehavior": {
    "TargetOriginId": "{bucket name}",
    "ForwardedValues": {
      "QueryString": true,
      "Cookies": {
        "Forward": "none"
      }
    },
    "TrustedSigners": {
      "Enabled": false,
      "Quantity": 0
    },
    "ViewerProtocolPolicy": "redirect-to-https",
    "MinTTL": 2,
    "DefaultTTL": 2
  },
  "CacheBehaviors": {
    "Quantity": 0
  },
  "CustomErrorResponses": {
    "Quantity": 2,
    "Items": [
      {
        "ErrorCode": 404,
        "ResponsePagePath": "/index.html",
        "ResponseCode": "200",
        "ErrorCachingMinTTL": 0
      },
      {
        "ErrorCode": 403,
        "ResponsePagePath": "/index.html",
        "ResponseCode": "200",
        "ErrorCachingMinTTL": 0
      }
    ]
  },
  "Comment": "",
  "Logging": {
    "Enabled": false,
    "IncludeCookies": true,
    "Bucket": "",
    "Prefix": ""
  },
  "PriceClass": "PriceClass_100",
  "Enabled": true
}
```

Under the same directory as __cloudfront-config.json__, run the command below:

```bash
aws cloudfront create-distribution \
  --distribution-config file://cloudfront-config.json
```
You should get a response simliar to this:

```json hl_lines="3 4"
{
    "Distribution": {
        "Status": "InProgress", 
        "DomainName": "d3hbokjo8idk8r.cloudfront.net", 
        "InProgressInvalidationBatches": 0, 
        "DistributionConfig": {
            "Comment": "", 
            "CacheBehaviors": {
                "Quantity": 0
            }, 
            ......
            ......
            ......
            "Aliases": {
                "Quantity": 0
            }
        }, 
        "ActiveTrustedSigners": {
            "Enabled": false, 
            "Quantity": 0
        }, 
        "LastModifiedTime": "2018-11-04T19:11:30.931Z", 
        "Id": "E2JVRJ2CZIEWMG", 
        "ARN": "arn:aws:cloudfront::313747422589:distribution/E2JVRJ2CZIEWMG"
    }, 
    "ETag": "E12VC97I0FXFA0", 
    "Location": "https://cloudfront.amazonaws.com/2018-06-18/distribution/E2JVRJ2CZIEWMG"
}
```

Notice the __Status__ and __DomainName__. Please make a note of the __DomainName__. You will need it for the following steps.

You can also go to the [**CloudFront**](https://console.aws.amazon.com/cloudfront/home?# "AWS CloudFront") console to view the deployment status of your distribution and find the **DomainName**, which has the format __{random-string}.cloudfront.net__ as showing in the response above.

!!! danger "It takes about 14 ~ 15 minutes to create the CloudFront distribution. While you are waiting, you can work on the step below."

### Add the CloudFront callback URL to Cognito

Now to add the CloudFront callback URL to AWS Cognito using the __DomainName__ from the previous step:

* Open the [Cognito User Pool Console](https://console.aws.amazon.com/cognito/users/ "AWS Cognito")
* Open the user pool you created in the previous step (Default name: _feedback-users_)
* Go to __App client settings__ on the left panel
* Append the CloudFront URL to the __Callback URL(s)__ field. Then save the changes.

    !!! note "You will need to add ==https://== and ==/callback== to the URL."
    For example, your __Calback URL(s)__ field should look like:

    `https://02b97ba28ee44121bc0e2ed09a4e6e99.vfs.cloud9.us-west-2.amazonaws.com/callback`,**`https://d1659etpii3mp.cloudfront.net/callback`**

 * Save the changes

### Testing from CloudFront

!!! note "Use the credentials you created in the previous step, e.g. __firstuser__/__AWS@dmin12345__"
* Once your distribution status is **Deployed**, you can open the CloudFront URL (e.g. `https://d1659etpii3mp.cloudfront.net`) and test interacting with the application from there.