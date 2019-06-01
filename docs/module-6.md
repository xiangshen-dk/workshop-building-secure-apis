# Conclusions

### What we have accomplished

#### Created a Restful backend via Amazon API Gateway

Our backend includes a Java Springboot application running in a private subnet, an API hosted on [Amazon API Gateway](https://aws.amazon.com/api-gateway/ "AWS API Gateway"), and a persistent data store using MySQL.
![Restful API](../screenshots/architecture1.png)

#### We tried the following approaches to enhance the secriuty posture of the API

* Resource policy
* Private integration
* Input validation
* Web application firewall
* Usage Plan and API key
* Authentication
* X-Ray for tracing

### Other useful references

* [Serverless whitepapers](https://aws.amazon.com/whitepapers/#serverless)

* AWS Tutorial: [Build a Serverless Web Application](https://aws.amazon.com/serverless/build-a-web-app/)

* Workshops: [Wild Rydes Serverless Workshops](https://github.com/aws-samples/aws-serverless-workshops)

* AWS [Amplify Framework](https://aws-amplify.github.io/) - an opinionated, category-based client framework for building scalable mobile and web apps

* [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)

* [Amazon API Gateway Developer Guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)

* [AWS AppSync Developer Guide](https://docs.aws.amazon.com/appsync/latest/devguide/welcome.html)

* [GraphQL tutorials](https://www.graphql.com/tutorials/)