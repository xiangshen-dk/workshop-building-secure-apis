# Conclusions

### Congratulations! You've made it through the workshop. Let's look back through what you accomplished...

#### You created a RESTful backend with Amazon API Gateway

Our backend includes a Java Springboot application with a MySQL database running on an EC2 instance in a private subnet. This is exposed via a private integration with an API hosted on [Amazon API Gateway](https://aws.amazon.com/api-gateway/ "AWS API Gateway").
![Restful API](../screenshots/architecture1.png)

#### You used the following to enhance the security posture of your API:

* A [***resource policy***](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-resource-policies.html) to control which principals can invoke your API.
* A [***private integration***](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-private-integration.html) to expose a private resource via your API.
* [***Input validation***](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-method-request-validation.html) to ensure only valid inputs get passed along to our backend through your API.
* A [***web application firewall***](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-aws-waf.html) to help protect against common web exploits like SQL injection and cross-site scripting.
* A [***usage plan*** and ***API key***](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-usage-plans.html) to set limits on the request rate for consumers of your API.
* [***Authentication***](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html) via Amazon Cognito to ensure that only authenticated and authorized users may invoke your API.
* [***X-Ray***](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html) to trace requests through your API.

### Want to explore more? Check out these other useful resources:

* [Serverless whitepapers](https://aws.amazon.com/whitepapers/#serverless)

* AWS Tutorial: [Build a Serverless Web Application](https://aws.amazon.com/serverless/build-a-web-app/)

* Workshops: [Wild Rydes Serverless Workshops](https://github.com/aws-samples/aws-serverless-workshops)

* AWS [Amplify Framework](https://aws-amplify.github.io/) - an opinionated, category-based client framework for building scalable mobile and web apps

* [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)

* [Amazon API Gateway Developer Guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)

* [AWS AppSync Developer Guide](https://docs.aws.amazon.com/appsync/latest/devguide/welcome.html)

* [GraphQL tutorials](https://www.graphql.com/tutorials/)