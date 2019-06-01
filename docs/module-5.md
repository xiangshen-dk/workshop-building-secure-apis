# GraphQL as backend
Time Estimate: 20 - 25 minutes  
In this module, we will use AWS AppSync to create a GraphQL backend API for our application, enabling real-time updates.

### Create the schema

??? info "AWS AppSync"
	AWS AppSync is a serverless back-end for mobile, web, and enterprise applications that securely handles application data management tasks like online and offline data access, data synchroniztion, and data manipulation across multiple data sources. AWS AppSync uses GraphQL, an API query language designed to build client applications by providing a flexible and intuitive system for describing their data requirement. For more information, see https://aws.amazon.com/appsync/.

1. Go to [AWS AppSync](https://us-east-1.console.aws.amazon.com/appsync/home?region=us-east-#/ "AppSync Console"), click the button __Create API__

2. Select __Build from scratch__ and click the button __Start__  
	![Create Schema](../screenshots/CreateAppSync.png)

3. Type in the name __Feature Request__, click __Create__

4. Click __Edit Schema__

5. In the __Schema text box__, delete the existing sample and copy/paste the following to the text box:


```javascript
type Mutation {
	# Create a single Request.
	createRequest(id: ID!, featureRequest: String!, completed: Boolean): Request
	# Delete a single Request by id.
	deleteRequest(id: ID!): Request
	# Update Request
	updateRequest(id: ID!, featureRequest: String, completed: Boolean): Request
}

type Query {
	# Get a single Request by id.
	getRequest(id: ID!): Request
	# Paginate through Requests.
	listRequests(limit: Int, nextToken: String): RequestConnection
}

type Request {
	id: ID!
	featureRequest: String
	completed: Boolean
}

type RequestConnection {
	items: [Request]
	nextToken: String
}

type Subscription {
	subscribeToCreateRequest: Request
		@aws_subscribe(mutations: ["createRequest"])
	subscribeToDeleteRequest: Request
		@aws_subscribe(mutations: ["deleteRequest"])
	subscribeToUpdateRequest: Request
		@aws_subscribe(mutations: ["updateRequest"])
}

schema {
	query: Query
	mutation: Mutation
	subscription: Subscription
}
```

6\. Click __Save Schema__

### Create resources

Click the button __Create Resources__ and do the following:

1. Select __Use existing type__
2. Select __Request__ from the dropdown
3. Disable __Automatically generate GraphQL__ under __Additional Indexes__
	![Create resources 2](../screenshots/AppSyncCreateResources2.png)
4. Click __Create__ and wait until it completes

### Create resolvers

In AppSync, under the __Resolvers__ panel, click __Attach__ for each of the following and select __RequestTable__ as the __Data source name__. When noted below, also be sure to change the request mapping template:

??? note "Screenshots"
	![Attach resolvers](../screenshots/AppSyncAttachResolvers.png)
	![Edit resolvers](../screenshots/AppSyncEditResolvers.png)

* __Mutation__

|Field|Resolver|Template|
|-----|--------|--------|
|createRequest(...): Request|RequestTable|Copy/paste code below|

```json
{
  "version" : "2017-02-28",
  "operation" : "PutItem",
  "key" : {
    ##You can also auto-generate the id using $util.dynamodb.toDynamoDBJson($util.autoId())
    "id": $util.dynamodb.toDynamoDBJson($ctx.args.id)
    },
    "attributeValues" : $util.dynamodb.toMapValuesJson($ctx.args)
}
```

|Field|Resolver|Template|
|-----|--------|--------|
|deleteRequest(...): Request|RequestTable|Copy/paste code below|

```json
{
  "version": "2017-02-28",
  "operation": "DeleteItem",
  "key": {
    "id": { "S": "$context.arguments.id"}
  }
}
```

|Field|Resolver|Template|
|-----|--------|--------|
|updateRequest(...): Request|RequestTable|Copy/paste code below|

```json
{
  "version" : "2017-02-28",
  "operation" : "PutItem",
  "key": {
    "id": { "S": "$context.arguments.id"}
  },
  "attributeValues" : $util.dynamodb.toMapValuesJson($ctx.args)
}
```

* __Query__

|Field|Resolver|Template|
|-----|--------|--------|
|getRequest(...): Request|RequestTable|Default|
|listRequests(...): RequestConnection|RequestTable|Copy/paste code below|

```json
{
  "version": "2017-02-28",
  "operation": "Scan",
  "limit": #if($context.arguments.limit) $context.arguments.limit #else 10 #end,
  "nextToken": #if($context.arguments.nextToken) "$context.arguments.nextToken" #else null #end
}
```

### Test resolvers
Click the __Queries__ link on the left side. In the queries window copy/paste the following code to the editor field:

```javascript
mutation createRequest {
  createRequest(id: "123",
    featureRequest: "Request #1",
    completed: false) {
    id
    featureRequest
    completed
  }
}

query getRequest {
  getRequest(id: "123") {
    id
    featureRequest
  }
}

mutation updateRequest {
  updateRequest(id: "123",
    featureRequest:"Our first request",
    completed: true) {
    id
  }
}
```
You can then click the yellow arrow button to run the queies one by one. You can also click the _Docs_ link in the top right corner to see the API self-documentation. For example:

![AppSync testing](../screenshots/AppSyncTestResolver.png)

??? note "Optional - verify results in DynamoDB"
	Go to [DynamoDB console](https://us-west-2.console.aws.amazon.com/dynamodb/home?region=us-west-2#tables:selected=Feature-Request;tab=items), open the _Feature Request_ table and view the results under the _Items_ tab.

### Configure access

1. Go to __Settings__, and under __Authorization type__ select __Amazon Cognito User Pool__.

2. Under __User Pool configuration__.

	* Select the region (_US-WEST-2_).

	* Select the user pool (_feedback-users_).

	* Select _ALLOW_ for __Default action__.

		![AppSync settings](../screenshots/AppSyncSettings.png)

	* Click the button __Save__ to save the changes.

3. Also make a note for the __API URL__ at the top of the page. You will need it to update the code.

4. Go back to your Cloud9 Editor, open the file __AppSync.js__ under the __src__ directory.

5. Update the __graphqlEndpoint__ value with the __API URL__ you got from step 3. Save the change.

	!!! note "Update the highlighted line using the value you got from the previous step."
	For example:
	``` hl_lines="2"
	export default {
	'graphqlEndpoint': 'https://xxxxx.appsync-api.us-west-2.amazonaws.com/graphql',
	'region': 'us-west-2',
	'authenticationType': 'AMAZON_COGNITO_USER_POOLS'
	}
	```

### Testing the change from Cloud9

Refresh your preview window and click the link __Feature Request__ in the preview application to verify the changes.

!!! note "If the application is not running in your Cloud9 terminal, run the commands below"

	```bash
	cd ~/environment/feedback-ui
	nvm use 8.12.0
	npm run serve
	```

* Click the link __Feature Request__  at the upper right corner
* Type in some words and click __Submit it__
* Go to your [DynamoDB table](https://us-west-2.console.aws.amazon.com/dynamodb/home?region=us-west-2#tables:selected=RequestTable;tab=items "DynamoDB table") and you should be able to see the data you just submitted under __Items__ tab in the feature request table.

### Testing the change from CloudFront

* Go to the [CloudFront console](https://console.aws.amazon.com/cloudfront/home?# "AWS CloudFront") to make sure the deployment status is __Deployed__

* Back in Cloud9, build and deploy our most recent changes to your S3 bucket:

```bash
  cd ~/environment/feedback-ui
  nvm use 8.12.0
  npm run build

  cd dist && \
  aws s3 sync --acl public-read . s3://$BUCKET_NAME/
```

* Wait for a few seconds to allow the cache expire in CloudFront

!!! info "In order to save time for the lab, in the privous step to create the CloudFront distribution we set the TTL only to 2 seconds. The configuration is in the __cloudfront-config.json__ file."

* Open the CloudFront URL (e.g. __`https://d1659etpii3mp.cloudfront.net`__) and test interacting with the application from there. To see the feature requests in real time, open the CloudFront URL in two separate browsers (such as Chrome and Firefox).

* Log into both windows and open the __Feature Request__ page.

* Type something in one of the browser windows and click __Submit it__. You should see the text automatically appear in the other browser window without having to refresh the page.

!!! info "It works because AWS AppSync keeps the app data updated in [real time](https://docs.aws.amazon.com/appsync/latest/devguide/real-time-data.html "AppSync real-time data") for multiple clients."
