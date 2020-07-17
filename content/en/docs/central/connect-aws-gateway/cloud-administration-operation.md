---
title: Cloud administration/operation
linkTitle: Cloud administration/operation
draft: false
weight: 90
description: As a Cloud Administrator / Operator, you are responsible for configuring and managing your organization’s AWS infrastructure. This topic contains setup and test details for the additional AWS services that are required for Axway’s agents to govern your AWS API Gateway service. The additional services which will be configured are AWS CloudWatch, AWS SQS, AWS Config, and AWS Lambda.
---

## Overview

Connecting AWS API Gateway to AMPLIFY Central will provide you with a connected/managed environment, and a global centralized view of your APIs and their related traffic, allowing users to have a centralized governance (creation/deployment/publish/subscription) and monitoring of the traffic for AWS API Gateway hosted APIs.

Each AWS Gateway is represented by an AMPLIFY Central environment allowing you to better filter APIs and their traffic. Supplied with the environment, two agents, Discovery and Traceability, interact with AWS API Gateway and AMPLIFY Central.

### Discovery Agent

To deploy an API In the AWS API Gateway, you create an API deployment and associate it with a stage. The Axway Discovery Agent listens for new deployments and for stage updates to existing deployments. When the agent receives an event it will publish, or update AMPLIFY Central with the API details. It is possible for the agent to publish the API information directly into the Unified Catalog or to be added to the environment associated with the agent in AMPLIFY Central.

In order for the Discovery Agent to receive the API details, the following AWS services are used:

| AWS Service    | Purpose                                                                                          |
|----------------|--------------------------------------------------------------------------------------------------|
| AWS Config     | Set up to monitor any configuration changes on API Gateway resources, specifically REST API’s and stages. When those changes are detected, they are sent to CloudWatch logs, and then they are sent to SQS.|
| AWS SQS        | The queue receives messages available to the Discovery Agent to find and determine what kind of resource that message is, what type of changes were made (update, delete, create), and it will query against API Gateway to get additional information about those changes, if needed, finally that info is sent to the AMPLIFY Platform.|
| AWS CloudWatch | Monitors resources and changes that the Discovery Agent made to the logging.                     |

![Service Discovery](/Images/central/ServiceDiscovery.png)

The AWS Discovery Agent discovers newly created, previously undiscovered REST APIs, as well as changes to the API’s stage(s), which then updates the logging that enables the Traceability Agent (see below).

The agent only publishes APIs that pass the tagging criteria that is configured in the agent configuration file, see [Filtering APIs to be discovered](/docs/central/connect-aws-gateway/filtering-apis-to-be-discovered-1/). The agent will use the tags which are associated with the stage that is associated with the API.

As soon as an API is published to AMPLIFY Central, a new tag (APIC_ID) is added to the stage so that the Discovery Agent will not publish it again. The value of the APIC_ID tag is the ID of the resource representing the API in Central. It only discovers published APIs where the stage has one or more tags as defined in the agent configuration file.

### Traceability Agent

The Traceability Agent sends summaries to AMPLIFY Central of the API traffic that has passed through the AWS API Gateway. The agent only sends a traffic summary for APIs that have been discovered (i.e. tagged with APIC_ID).

The Traceability Agent is used to filter the AWS CloudWatch logs that are related to discovered APIs and prepare the transaction events that are sent to AMPLIFY platform. Each time an API is called by a consumer it will result in an event (summary + detail) being sent to AMPLIFY Central. API Observer provides a view of the traffic and API usage of APIs deployed to the Gateway.

In order for the Traceability Agent to monitor API traffic, the following AWS services are used:

| AWS Service       | Purpose                                                                                          |
|-------------------|--------------------------------------------------------------------------------------------------|
| AWS Lambda        | Runs code in response to events and automatically manages the computing resources required by that code. CloudWatch will write whenever a usage of an API invoked, and that is sent to the Lambda function, which parses out some pertinent information in order to track that usage and send it to SQS.|
| AWS SQS           | Those SQS messages are read by the Traceability Agent. The REST API ID and the stage ID are then queried back to CloudWatch for the additional transaction details (i.e. headers), in order to fully create a transaction object, which is then sent to the Amplify platform.|
| AWS CloudWatch    | Monitors when an API is consumed, and if the Discovery Agent made changes to the logging. Those events are logged to CloudWatch. |

The Traceability Agent requires read write access to SQS and read only access to CloudWatch.

![Service Discovery](/Images/central/Traceabilityagent.png)

### Minimum requirements

* An AMPLIFY Central Service Account. See [Prepare AMPLIFY Central](https://axway.bintray.com/generic-repo/aws-agents/aws_apigw_agent_config/latest/)

* [API Key credentials on AWS](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html). Allowing for CLI access.

* [Amazon CloudWatch Service](https://aws.amazon.com/cloudwatch/)

* [Amazon Simple Queue Service](https://aws.amazon.com/sqs/)

* [AWS Lambda](https://aws.amazon.com/lambda/)

* Agent Config Package [downloaded](https://axway.bintray.com/generic-repo/aws-agents/aws_apigw_agent_config/latest/)

### CloudFormation templates

The agent config package contains a Lambda function (`traceability_lambda.zip`), and two CloudFormation templates (`apigw_iam_setup.yaml apigw_cloudformation.yaml`) which configure the additional AWS services (`AWS CloudWatch, AWS SQS, and AWS Lambda`) that the agents require to function normally. Those resources are uploaded to an S3 Bucket (see`<ConfigBucketName>` below).

The templates are broken into two parts:

* IAM Setup (`apigw_iam_setup.yaml`)

* Resource CloudFormation (`apigw_cloudformation.yaml`)

As a Cloud Administrator, you may choose whether to create IAM resources or not. The resources created by the IAM Setup are a User, three Roles, and a Key and Group number. {{< alert title="Note" color="primary" >}}The Role ARN Outputs are consumed as inputs for the Resource CloudFormation Template.{{< /alert >}}

#### Parameters (IAM Setup)

The inputs to the IAM Setup CloudFormation Template (`apigw_iam_setup.yaml`):

| Parameter name       | Description                                                              | Default Value       |
|----------------------|--------------------------------------------------------------------------|---------------------|
| SetupConfigService   | If set to true the IAM Role for the Config Service will be created.       | true                |
| SetupAPIGWCWRole     | If set to true the IAM Role for the API Gateway Service to write logs to CloudWatch will be created. | true |
| ConfigBucketName     | The name of the bucket the Config Service, if enabled, will store AWS Config Logs. The account number and region will be appended to this. | apigw-config-discovery |
| ConfigBucketExists   | If set to true the Config Bucket will not be created.                     | false               |
| DiscoveryQueueName   | The name of the Queue that the Discovery Agent will read from.            | aws-apigw-discovery |
| TraceabilityQueueName | The name of the Queue that the Traceability Lambda will be writing to and the Traceability Agent will read from. | aws-apigw-traceability |

#### Resources (IAM Setup CloudFormation template)

The resources created by the IAM Setup CloudFormation template:

| Resource Type      | Resource Name        | Condition         | Description                                           |
|--------------------|----------------------|-------------------|-------------------------------------------------------|
| AWS::IAM::Role     | ConfigServiceIAMRole | ShouldSetupConfigService | Creates the IAM Role for the Config Service. |
| AWS::IAM::Role     | TraceabilityLambdaIAMRole |           | Creates the IAM Role for the Traceability Lambda Function. |
| AWS::IAM::Role     | TraceabilityAPIGWCWIAMRole | ShouldSetupAPIGWCWRole | Creates the IAM Role for API Gateway to write logs to CloudWatch. |
| AWS::IAM::User     | APICAgentsUser       |                   | Creates the user needed for the agents.                |
| AWS::IAM::AccessKey | APICAgentsKey      |                   | Creates the Access and Secret Keys for the agent user. |

#### Outputs (IAM Seup CloudFormation template)

The outputs from the IAM Setup CloudFormation template; the Role ARN Outputs are consumed as inputs for the Resource CloudFormation Template (below):

| Output Name                         | Description                                                                    |
|-------------------------------------|--------------------------------------------------------------------------------|
| APICAgentAccessKey                  | The Access Key for the APIC Agents.                                            |
| APICAgentSecretKey                  | The Secret Key for the APIC Agents.                                            |
| ConfigServiceRoleArn                | The ARN for the Config Service IAM Role.                                       |
| TraceabilityLambdaRoleArn           | The ARN for the Traceability Lambda IAM Role.                                  |
| TraceabilityAPIGWCWRoleArn          | The ARN for the API Gateway push to CloudWatch IAM Role.                       |

#### Parameters (Resource CloudFormation template)

The inputs to the Resource CloudFormation template (`apigw_cloudformation.yaml`):

| Parameter Name              | Description                                             | Default Value                |
|-----------------------------|---------------------------------------------------------|------------------------------|
| SetupConfigService          | This parameter is used to disable the configuration of AWS Config Service, and all of its dependencies, while building the stack. | true |
| ConfigBucketName            | The name of the bucket the Config Service, if enabled, will store AWS Config Logs. The account number and region will be appended to this. | apigw-config-discovery |
| ConfigBucketExists          | If set to true the Config Bucket will not be created.   | false                        |
| ConfigServiceRoleArn        | The ARN for the Config Service IAM Role.                |                              |
| DiscoveryQueueName          | The name of the queue that will hold only changes made to API Gateway resources. The region will be appended to this. | aws-apigw-discovery |
| TraceabilityLambdaRoleArn   | The Log Group created to track access of APIC tracked API Gateway endpoints. | APIGW_Traceability_Logs |
| TraceabilityAPIGWCWRoleArn  | The ARN for the IAM role that allows API Gateway the permission to write CloudWatch logs. Leave blank if this does not need configured. |  |
| TraceabilityQueueName       | The name of the queue that will hold traceability logs of API Gateway resources. The region will be appended to this. | aws-apigw-traceability |
| TraceabilityFunctionBucket  | The S3 bucket that has the executable for the traceability lambda function. |  |
| TraceabilityFunctionKey     | The key of the traceability lambda function in the bucket. |  |

#### Resources (CloudFormation template)

The services that are configured with this CloudFormation template:

| Resource Type           | Resource Name           | Condition             | Description                              |
|-------------------------|-------------------------|-----------------------|------------------------------------------|
| AWS::Config::ConfigurationRecorder | DiscoveryConfigRecorder | ShouldSetupConfigService | The setup needed to have Config start recording changes. |
| AWS::Config::DeliveryChannel | DiscoveryConfigDeliveryChannel | ShouldSetupConfigService | The delivery channel used by Config to send the configurations to ConfigBucket. |
| AWS::S3::Bucket         | DiscoveryConfigBucket   | ShouldCreateConfigBucket | The S3 bucket used to store all current configurations, required for Config. |
| AWS::Events::Rule       | DiscoveryConfigCloudWatchRule |  | DiscoveryConfigCloudWatchRule. |
| AWS::SQS::Queue         | DiscoveryConfigSqsQueue  |  | The Queue that all API Gateway configuration changes are pushed to. |
| AWS::SQS::QueuePolicy   | DiscoveryConfigSqsQueuePolicy |  | The policy that grants permission to push to the SqsQueue. |
| AWS::Logs::LogGroup     | TraceabilityAccessLogGroup |  | Creates the Log Group to track access of APIC tracked API Gateway endpoints. |
| AWS::ApiGateway::Account | TraceabilityAPIGWCWRole  | ShouldSetupAPIGWCWRoleArn | Sets the CloudWatch Role ARN in the API Gateway Settings. |
| AWS::Lambda::Function   | TraceabilityLambda       |  | The Lambda function that takes Cloud Watch events in the Log Group and sends them to the SQS Queue. |
| AWS::Lambda::Permission | TraceabilityLambdaCWInvoke |  | Allows Cloud Watch events to trigger the Lambda Function. |
| AWS::SQS::Queue         | TraceabilitySqsQueue       |  | The Queue that all API Gateway access logs are pushed to. |
| AWS::Logs::SubscriptionFilter | TraceabilityLogToLambdaFilter |  | Filter events from the Tracebaility Logs to the Lambda Function. |

#### Outputs (Resource CloudFormation template)

The outputs from the Resource CloudFormation template:

| Output Name              | Description                                                                              |
|--------------------------|------------------------------------------------------------------------------------------|
| DiscoveryQueueName       | Amazon SQS Queue name containing the changes to API Gateway.                             |
| TraceabilityLogGroupArn  | The ARN of the Log Group for tracking API Gateway endpoints.                             |
| TraceabilityQueueName    | Amazon SQS Queue name containing the API Gateway Access Logs.                            |

The results of running the terraform script need to be configured inside the agent configuration files so that the agent can utilize the newly provisioned AWS services.

Resource key data that needs to be inputted into agent config file(s):

| Key                        | Value                            | Agent(s) Config                                     |
|----------------------------|----------------------------------|-----------------------------------------------------|
| APICAgentsGroup            | aws:auth:accessKey               | Discovery & Traceability Agents                     |
| APICAgentsKey              | aws:auth:secretKey               | Discovery & Traceability Agents                     |
| TraceabilityLogGroupArn    | aws:logGroupArn                  | Discovery Agent only                                |
| DiscoveryQueueName         | aws:queueName                    | Discovery Agent only                                |
| TraceabilityQueueName      | aws:queueName                    | Traceability Agent only                             |

### Testing the setup for the AWS Discovery Agent

* Make a change to an existing REST API/Stage OR create a new REST API/Stage

* Validate that the `<DiscoveryQueueName>` has messages waiting

### Testing the setup for the AWS Traceability Agent

{{< alert title="Note" color="primary" >}}To test the AWS Traceability Agent setup, the AWS Discovery Agent should be running and have discoverd an API, this is required as the Stage config is updated to log transactional data.{{< /alert >}}

* Send traffic through an API that the DA has discovered

* Validate messages received in AWS SQS

* Validate logging in CloudWatch under the Log group named after the REST API ID/Stage

### Connecting AWS API Gateway to AMPLIFY Central QuickStart

* [Prepare AWS API Gateway](/docs/central/connect-aws-gateway/prepare-aws-api-gateway/)

* [Prepare AMPLIFY Central](/docs/central/connect-aws-gateway/prepare-amplify-central-1/)

* [Deploy your agents](/docs/central/connect-aws-gateway/deploy-your-agents-1/)

### Troubleshooting

| Question                                            | Answer                                                        |
|-----------------------------------------------------|---------------------------------------------------------------|
| Why isn’t my API discovered?                        | Check that the tag set on the stage has a correct name and value based on the AWS_FILTER variable. See [Filtering APIs to be discovered](/docs/central/connect-aws-gateway/filtering-apis-to-be-discovered-1/). |
| Why can’t my agents connect to AWS API Gateway?     | Go to AWS console / IAM service and make sure that `AWS_REGION`, `AWS_AUTH_ACCESSKEY` and `AWS_AUTH_SECRETKEY` are valid and not inactivated. |
| Why can’t my agents connect to AMPLIFY Central?     | Go to **AMPLIFY Central UI > Access > Service Accounts** and make sure that the Service Account is correctly named and valid. Make sure that the tenantID and teamID are correct. |
| Why don’t I see traffic in AMPLIFY Central?         | Make sure that the Condor URL is accessible from the machine where Traceability Agent is installed. |
| How to verify that the Agent is running?            | `docker inspect --format='{{json .State.Health}}' <container>`
