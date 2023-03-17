---
title: Cloud Resume Challenge – Part 2
series: ["Cloud Resume Challenge"]
series_order: 2
date: 2022-07-16
tags:
- cloudresume
- aws

showtableofcontents: true
showtaxonomies: true
showsummary: true
summary: "Part 2 of my Cloud Resume Challenge attempt" 
 
---


I&#8217;ve tackled some steps of the challenge out of turn here for a bit as it made sense to me to work on the Database/Lambda/API before working on the Javascript that requires them.

## Step 8: Database

For this step I really leaned into trying to deploy via CloudFormation first rather than deploying via the GUI.  
Our Database will consist of a DynamoDB as per the recommendation on the Cloud Resume Challenge website. Our table will consist of two attributes, `record_id` and `record_count.`  
I ran into an issue not quite understanding how CloudFormation wanted to have the attributes presented, in the end I created just the key attribute of `record_id` and had to manually add the `record_count` attribute from the gui.  


I followed the Cloud Foundation documentation on DynamoDB [here][1]  
Here is my the relevant CloudFoundation YAML code:
```yaml
VisitorCountTable:
  Type: AWS::DynamoDB::Table
  Properties:
  AttributeDefinitions: 
    -
      AttributeName: "record_id"
      AttributeType: "S"
  TableName: "Visitor_Count"
  ProvisionedThroughput: 
    ReadCapacityUnits: 5
    WriteCapacityUnits: 5
  KeySchema: 
    - 
      AttributeName: "record_id"
      KeyType: "HASH"
```

Once created, log into the AWS GUI and find your DynamoDB table. Click add item, add the attribute `record_count` with an value of 1

![](/Images/CloudResumeChallenge/image2.png)

## Step 8.5 and 10: Lambda and Python

We&#8217;ll need to create two Lambda functions, one to get the visitor count and one to increase it.  
I again went with the CloudFormation first approach and found I was starting to get the hang of it.  
I followed the documentation on Lambda functions [here][2]

First I manually created a IAM role for my Lambda functions, I included the following policies

![](/Images/CloudResumeChallenge/image3.png)



For this to work, we will need to create a couple of small Python scripts that the function will invoke

getvisitor.py simply connects to our DynamoDB database and returns the current value of the `record_count` attribute
```python
import json
import boto3
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Visitor_Count')
def lambda_handler(event, context):
    response = table.get_item(Key={
       'record_id':'0'
    })
    return response&#91;'Item']&#91;'record_count']
```

addvisitor.py is similar, this time increasing the current value of the `record_count` attribute.

```python
import json
import boto3
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Visitor_Count')
def lambda_handler(event, context):
    response = table.get_item(Key={
            'record_id':'0'
    })
    record_count = response&#91;'Item']&#91;'record_count']
    record_count = record_count + 1
    print(record_count)
    response = table.put_item(Item={
            'record_id':'0',
            'record_count': record_count
    })
    
    return "Records added successfully!"
```

We need to save and zip these two files up and upload them to a S3 bucket so the Lambda function can access

![](/Images/CloudResumeChallenge/image4.png)


Now deploy it with the CloudFoundation YAML Code

```yaml
    LambdaGetVisitor:
        Type: "AWS::Lambda::Function"
        Properties:
            FunctionName: "CloudResumeGetVisitor"
            Handler: "getvisitor.lambda_handler"
            Architectures: 
              - "x86_64"
            Code:
              S3Bucket: "cloudresumelambdafunctions"
              S3Key: "getvisitor.zip"

            Role: "arn:aws:iam::337461354076:role/CloudResume-LambdaFunctions"
            Runtime: "python3.9"

    LambdaAddVisitor:
        Type: "AWS::Lambda::Function"
        Properties:
            FunctionName: "CloudResumeAddVisitor"
            Handler: "addvisitor.lambda_handler"
            Architectures: 
              - "x86_64"
            Code:
              S3Bucket: "cloudresumelambdafunctions"
              S3Key: "addvisitor.zip"

            Role: "arn:aws:iam::337461354076:role/CloudResume-LambdaFunctions"
            Runtime: "python3.9"
```

All going well, you should have two shiny functions in your gui

![](/Images/CloudResumeChallenge/image5.png)


## Step 9: API

We&#8217;ll need to create ourselves a REST API with two methods being GET to reference our LambdaGetVisitor function and POST to reference our LambdaAddVisitor function.

First we create the API Gateway object, as before I have preferred to go the CloudFormation root for this deployment



```yaml
    ApiGateway:
        Type: "AWS::ApiGateway::RestApi"
        Properties:
            Name: "visitors"
            ApiKeySourceType: "HEADER"
            EndpointConfiguration: 
                Types: 
                  - "REGIONAL"
```

Pretty straightforward that one.  
Next we move onto the GET method, I had a pretty rough time trying to figure this part out as pure CloudFormation code. I ended up completing this in the GUI, exported the code via [Former2][3], deleting from GUI and redeploying with the newly written code.  

```
    ApiGatewayGet:
        Type: "AWS::ApiGateway::Method"
        Properties:
            RestApiId: !Ref ApiGateway
            ResourceId: !GetAtt ApiGateway.RootResourceId
            HttpMethod: "GET"
            AuthorizationType: "NONE"
            ApiKeyRequired: false
            RequestParameters: {}
            MethodResponses: 
              - 
                ResponseModels: 
                    "application/json": "Empty"
                StatusCode: "200"
            Integration: 
                CacheNamespace: !GetAtt ApiGateway.RootResourceId
                ContentHandling: "CONVERT_TO_TEXT"
                IntegrationHttpMethod: "POST"
                IntegrationResponses: 
                  - 
                    ResponseTemplates: {}
                    StatusCode: "200"
                PassthroughBehavior: "WHEN_NO_MATCH"
                TimeoutInMillis: 29000
                Type: "AWS"
                Uri: !Sub "arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:${LambdaGetVisitor}/invocations"
```

Same as above for the POST method, deployed via gui, exported code, deleted and redeployed via CloudFormation

```
    ApiGatewayPost:
        Type: "AWS::ApiGateway::Method"
        Properties:
            RestApiId: !Ref ApiGateway
            ResourceId: !GetAtt ApiGateway.RootResourceId
            HttpMethod: "POST"
            AuthorizationType: "NONE"
            ApiKeyRequired: false
            RequestParameters: {}
            MethodResponses: 
              - 
                ResponseModels: 
                    "application/json": "Empty"
                StatusCode: "200"
            Integration: 
                CacheNamespace: !GetAtt ApiGateway.RootResourceId
                ContentHandling: "CONVERT_TO_TEXT"
                IntegrationHttpMethod: "POST"
                IntegrationResponses: 
                  - 
                    ResponseTemplates: {}
                    StatusCode: "200"
                PassthroughBehavior: "WHEN_NO_MATCH"
                TimeoutInMillis: 29000
                Type: "AWS"
                Uri: !Sub "arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:${LambdaAddVisitor}/invocations"
```

Once I deployed I went to the GUI to test my methods, but found both reported a permission denied error when I tried to run them.  
When deploying from the GUI you get a prompt asking if you want to give your API method access to the Lambda function, obviously that didnt happen when deploying via CloudFormation. I found [this][4] help page that exactly explained my issue. 

![](/Images/CloudResumeChallenge/image6.png)
Here are my code snippets to address that problem

```
    APIGetPermissions:
        Type: AWS::Lambda::Permission
        Properties:
          Action: "lambda:InvokeFunction"
          FunctionName: !Ref LambdaGetVisitor
          Principal: "apigateway.amazonaws.com"
          SourceArn: arn:aws:execute-api:ap-southeast-2:337461354076:&lt;api-id&gt;/*/GET/

    APIPostPermissions:
        Type: AWS::Lambda::Permission
        Properties:
          Action: "lambda:InvokeFunction"
          FunctionName: !Ref LambdaAddVisitor
          Principal: "apigateway.amazonaws.com"
          SourceArn: arn:aws:execute-api:ap-southeast-2:337461354076:&lt;api-id&gt;/*/POST/
```

Thats it for now, part 3 will try and merge this all together and get a nice counter on our Resume

 [1]: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html
 [2]: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html
 [3]: https://former2.com/
 [4]: https://aws.amazon.com/premiumsupport/knowledge-center/api-gateway-rest-api-lambda-integrations/