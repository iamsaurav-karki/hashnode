---
title: "Deploying Serverless Applications with AWS Lambda and API Gateway"
datePublished: Thu Aug 07 2025 09:09:45 GMT+0000 (Coordinated Universal Time)
cuid: cmejkqkwe000802l24yk94r3h
slug: deploying-serverless-applications-with-aws-lambda-and-api-gateway-f513ee7183af
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1755670250513/02930eec-2f07-4ddd-851a-e52d99536368.png

---

### Introduction

Serverless Applications are those applications which allows you to run code without provisioning or managing servers. You just have to run the code and Server manaement will be handled by the cloud providers , in this case AWS. AWS Lambda, combined with API Gateway, lets you build scalable APIs and backend logic that automatically scales with usage and you pay only for what you use.

In this blog, I’ll demonstrate how to build and deploy a simple REST API using AWS Lambda and API Gateway, complete with practical code snippets and deployment steps.

### What You’ll Need

* AWS Account (Free tier available)
    
* AWS CLI installed and configured (aws configure)
    
* Node.js (for Lambda function runtime example)
    
* Basic knowledge of HTTP APIs
    

### Step 1: Writing a Simple Lambda Function

Create a new directory, e.g., lambda, and add index.js:

```markdown
exports.handler = async (event) => {
const name = event.queryStringParameters?.name || "World";
const response = {
statusCode: 200,
body: JSON.stringify({ message: `Hello, ${name}!` }),
};
return response;
};
```

This Lambda function reads a name query parameter and returns a greeting.

### Step 2: Packaging and Deploying Lambda

#### Zip the function code

zip function.zip index.js

First Install aws CLI and run :

```markdown
aws configure
```

( Enter the access key and secret access key to authenticate yourself to aws account).

You can create access keys and secret as:

* Login to aws dashboard.
    
* search IAM.
    
* Click and navigate to users section and create user.
    
* Create new user with permissions of your choice.
    
* Click on the created user and navigate to security credentials and create access keys.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755670244427/818fc584-efc9-45cc-a787-8c64f2f58444.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755670245876/0f1b6e51-ea77-4a95-9328-3d79654ef7cd.png align="left")

Create IAM Role in AWS for Lambda

aws iam create-role --role-name lambda-exec-role --assume-role-policy-document file://trust-policy.json

Before that create a trust-policy as : trust-policy.json

```markdown
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Principal": { "Service": "lambda.amazonaws.com" },
"Action": "sts:AssumeRole"
}
]
}
```

Note: You can also use AWS Dashboard GUI to create the roles and other setup.

Attach policy:

aws iam attach-role-policy --role-name lambda-exec-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

**Deploy Lambda function**

```markdown
aws lambda create-function \
--function-name HelloWorldFunction \
--runtime nodejs18.x \
--role arn:aws:iam:::role/lambda-exec-role \
--handler index.handler \
--zip-file fileb://function.zip
```

Replace with your AWS account ID.

### Step 3: Create an API Gateway HTTP API

Create API Gateway HTTP API linked to Lambda function:

```markdown
aws apigatewayv2 create-api \
--name HelloWorldAPI \
--protocol-type HTTP

Note the returned API ID (apiId).
```

### Step 4: Integrate Lambda with API Gateway

Create Lambda integration:

```markdown
aws apigatewayv2 create-integration \
--api-id \
--integration-type AWS_PROXY \
--integration-uri arn:aws:lambda:::function:HelloWorldFunction \
--payload-format-version 2.0

Note the integration ID (integrationId).
```

### Step 5: Create Route and Attach Integration

Create a route for path /hello with GET method:

```markdown
aws apigatewayv2 create-route \
--api-id \
--route-key "GET /hello" \
--target integrations/
```

### Step 6: Deploy the API

Create a stage named prod and deploy:

```markdown
aws apigatewayv2 create-stage \
--api-id \
--stage-name prod \
--auto-deploy
```

### Step 7: Grant API Gateway permission to invoke Lambda

```markdown
aws lambda add-permission \
--function-name HelloWorldFunction \
--statement-id apigateway-invoke \
--action lambda:InvokeFunction \
--principal apigateway.amazonaws.com \
--source-arn 'arn:aws:execute-api:::/*/*/hello'
```

Replace with your details.

### **Step 8: Test your API**

```markdown
curl "https://<api-id>.execute-api.<region>.amazonaws.com/prod/hello?name=Saurav"
```

eg:

```markdown
curl "https://rlfnawmjda.execute-api.us-east-1.amazonaws.com/prod/hello?name=Saurav"
```

Output:

```markdown
{"message":"Hello, Saurav!"}%
```

Check Lambda logs in CloudWatch to debug:

```markdown
aws logs filter-log-events --log-group-name /aws/lambda/HelloWorldFunction --limit 10
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755670247343/02379cc1-cd5d-4c99-8974-0173c64c6852.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755670248937/b24bb432-af53-49b7-b365-0f036c96aa20.png align="left")

### Step 9: Clean Up resources

```markdown
aws lambda delete-function --function-name HelloWorldFunction
aws apigatewayv2 delete-api --api-id 
aws iam delete-role --role-name
```

### Conclusion

In just a few steps, you built a serverless API using AWS Lambda and API Gateway that scales automatically and charges you only for what you use.