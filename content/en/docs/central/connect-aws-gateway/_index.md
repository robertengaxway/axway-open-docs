---
title: Connect AWS Gateway
linkTitle: Connect AWS Gateway
draft: false
no_list: true
weight: 140
description: Understand why you would want a connected / managed environment for
  AWS API Gateway within AMPLIFY Central. Learn how you can govern and monitor
  the creation / deployment / publishing and subscriptions of the AWS API
  Gateway hosted APIs in one central location.
---

## Why do you want to connect AWS API Gateway and AMPLIFY Central?

Connecting AWS API Gateway to AMPLIFY Central will provide you with a global centralized view of your APIs and their related traffic.

Each AWS Gateway can be represented by an AMPLIFY Central environment allowing you to better filter APIs and their traffic. Supplied with the environment, two agents (Discovery and Traceability) interact with AWS API Gateway and AMPLIFY Central to:

* Detect changes to AWS API Gateway stages and deployments using the Discovery Agent. The Discovery Agent pushes the service configuration as an API service for the environment, which can then be published as a catalog item to be used by consumers to subscribe to the service.
* Filter the AWS Cloudwatch logs that are related to discovered APIs and prepare the transaction events that are sent to AMPLIFY platform.

### Discovery Agent

The Discovery Agent is used to discover  new deployments and stage updates to existing deployments for publishing related APIs in AMPLIFY Central (either as  a catalog item or as an API service). As part of the deployment package, use the provided cloud formation scripts  to set up the following agent-dependent AWS Services.

* AWS Config - Administers, audits and monitors resource configurations. Records and validates configuration changes.
* AWS CloudWatch - Monitors resources and AWS applications in real time. Receives and routes supported AWS Service events.
* AWS SQS - Decouples and scales microservices, distributed systems and serverless applications.

![Service Discovery](/Images/central/ServiceDiscovery.png)

### Traceability Agent

The Traceability Agent is used to filter the AWS CloudWatch logs and prepare the transaction events that are sent to AMPLIFY Central and visible in the API observer. Viewing your traffic helps you to identify the bottleneck and errors. The traffic can be filtered by environment in case multiples are involved in your topography. As part of the deployment package, you can use cloud formation scripts to set up the following agent-dependent AWS Services.

* AWS CloudWatch - Monitors resources and AWS applications in real time. Receives and routes supported AWS Service events.
* AWS Lamda - Runs code in response to events and automatically manages the computing resources required by that code.
* AWS SQS - Decouples and scales microservices, distributed systems and serverless applications.

The types of logging you can do with API Gateway to CloudWatch.

* Execution logging - API Gateway manages the CloudWatch logs.
* Access logging - Developer managed custom logging.

For additional logging information, see <https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-logging.html>.

![Service Discovery](/Images/central/Traceabilityagent.png)

## Minimum requirements

An AMPLIFY Central Service Account. See [Manage an API proxy using AMPLIFY CLI](/docs/central/cli_proxy_flow/).

* API Key credentials on AWS. Allow for CLI access
* Amazon CloudWatch Service
* Amazon Simple Queue Service (AWS SQS)
* AWS Lambda
* CloudFormation template. Download from [https://axway.bintray.com/generic-repo/aws-agents/aws_apigw_agent_config/](<https://axway.bintray.com/generic-repo/aws-agents/aws_apigw_agent_config/>)

## Connect AWS API Gateway to AMPLIFY Central quickstart

### 1. Set up AMPLIFY Central

* Set up a [Service Account](/docs/central/connect-aws-gateway/prepare-amplify-central-1/#create-a-service-account).
* Create an [environment](/docs/central/connect-aws-gateway/prepare-amplify-central-1/#create-an-environment) object in AMPLIFY Central using either the UI, API or CLI.  

### 2. Set up using AWS CloudFormation

* Set up using the resources using [AWS CloudFormation](/docs/central/connect-aws-gateway/prepare-aws-api-gateway/#set-up-the-cloudformation)
* Get the required templates at [https://axway.bintray.com/generic-repo/aws-agents/aws_apigw_agent_config/](<https://axway.bintray.com/generic-repo/aws-agents/aws_apigw_agent_config/>)
    * IAM configuration (apigw_iam_setup.yaml)
    * Discovery and Traceability resource configuration (apigw_cloudformation.yaml)  

### 3. Create a Discovery Agent environment file

* Learn more about the [Discovery Agent](/docs/central/connect-aws-gateway/deploy-your-agents-1/#discovery-agent)
    * Create the Discovery Agent configuration file
    * Pull the latest Docker image of the Discovery Agent
    * Start the Discovery Agent  

### 4. Create a Traceability Agent environment file

* Learn more about the [Traceability Agent](/docs/central/connect-aws-gateway/deploy-your-agents-1/#traceability-agent)
    * Create the Traceability Agent configuration file
    * Log into the Artifactory Repository and pull the latest image of the Traceability Agent.
    * Start the Traceability Agent.
