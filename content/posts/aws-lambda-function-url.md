---
date: "2023-03-12"
title: "AWS Lambda Function URLs"
tags: ["aws", "lambda", "terraform"]
author: "masoudkf"
description: "Function URLs are an easy way to expose your Lambda functions to the public world."
---

So, you created a Lambda function and now want to use it in your application which will be publicly available? You need to find a way to expose it to the world.

Although the go-to service for exposing Lambda functions to the public Internet is AWS API Gateway, the service does have a learning curve, and it takes some time to properly set it up.

If you want to quickly test your Lambda function and don't want to go through the whole APIGW setup, you can use Function URLs. Function URLs are a rather new feature with Lambda that let you assign an HTTPS endpoint to your Lambda function for invokation. It's also free and you only pay for the Lambda usage.

The advantages of using Function URLs over API Gateway are:
- Ease of use 
- Cost
- Speed of development

However, API Gateway comes with great features such as Usage Plans, API Keys, Cognito Authentication, Request Throttling, and Caching.

## Lambda Function URL with Terraform
Here, we create a simple Lambda function with Python that returns the HTTP method it was invoked with, plus the name of the invoker. Let's start with our function code:

`main.py`
```python
import json

def lambda_handler(event, context):
    print(event)
    http_method = event["requestContext"]["http"]["method"].lower()
    invoker = None

    if http_method == "post" or http_method == "put":
        # POST and PUT use the request body to get info about the request
        body = json.loads(event["body"])
        invoker = body["invoker"]
    elif http_method == "get" or http_method == "delete":
        # GET and DELETE use query string parameters to get info about the request
        invoker = event["queryStringParameters"]["invoker"]
    
    return {
        "statusCode": 200,
        "body": json.dumps({
            "method": http_method,
            "invoker": invoker
        })
    }
```

How do I know about the request structure? There answer's [here](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html). Function URL uses API Gateway version `2.0` payload type, which is like this:

```json
{
  "version": "2.0",
  "routeKey": "$default",
  "rawPath": "/my/path",
  "rawQueryString": "parameter1=value1&parameter1=value2&parameter2=value",
  "cookies": [
    "cookie1",
    "cookie2"
  ],
  "headers": {
    "header1": "value1",
    "header2": "value1,value2"
  },
  "queryStringParameters": {
    "parameter1": "value1,value2",
    "parameter2": "value"
  },
  "requestContext": {
    "accountId": "123456789012",
    "apiId": "api-id",
    "authentication": {
      "clientCert": {
        "clientCertPem": "CERT_CONTENT",
        "subjectDN": "www.example.com",
        "issuerDN": "Example issuer",
        "serialNumber": "a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1",
        "validity": {
          "notBefore": "May 28 12:30:02 2019 GMT",
          "notAfter": "Aug  5 09:36:04 2021 GMT"
        }
      }
    },
    "authorizer": {
      "jwt": {
        "claims": {
          "claim1": "value1",
          "claim2": "value2"
        },
        "scopes": [
          "scope1",
          "scope2"
        ]
      }
    },
    "domainName": "id.execute-api.us-east-1.amazonaws.com",
    "domainPrefix": "id",
    "http": {
      "method": "POST",
      "path": "/my/path",
      "protocol": "HTTP/1.1",
      "sourceIp": "192.0.2.1",
      "userAgent": "agent"
    },
    "requestId": "id",
    "routeKey": "$default",
    "stage": "$default",
    "time": "12/Mar/2020:19:03:58 +0000",
    "timeEpoch": 1583348638390
  },
  "body": "Hello from Lambda",
  "pathParameters": {
    "parameter1": "value1"
  },
  "isBase64Encoded": false,
  "stageVariables": {
    "stageVariable1": "value1",
    "stageVariable2": "value2"
  }
}
```

Now, let's create a Lambda function off of the `main.py` file we wrote earlier:

`main.tf`
{{< gist masoudkarimif 00fb0872190b7ddb04e32b281d784c9e>}}

Apply the Terraform configuration using `aws-vault`:

```bash
aws-vault exec <profile-name> --no-session -- terrform apply
```

If successful, you should get the function URL as an output. You can now use the endpoint--which is in the format of `https://<url-id>.lambda-url.<region>.on.aws`--to send requests.

I use `curl` to send requests with different HTTP methods to Lambda:

### Get
Request:
```bash
curl <endpoint>?invoker=batman
```

Response:
```json
{"method": "get", "invoker": "batman"}
```

### POST
Request:
```bash
curl -X POST <endpoint> --data '{"invoker": "batman"}' --header "Content-Type: application/json"
```

Response:
```json
{"method": "post", "invoker": "batman"}
```

### PUT
Request:
```bash
curl -X PUT <endpoint> --data '{"invoker": "batman"}' --header "Content-Type: application/json"
```

Response:
```json
{"method": "put", "invoker": "batman"}
```

### DELETE
Request:
```bash
curl -X DELETE <endpoint>?invoker=batman
```

Response:
```json
{"method": "delete", "invoker": "batman"}
```