---
title: Deploy your agents
linkTitle: Deploy your agents
draft: false
weight: 30
description: >-
  Learn how to deploy your Discovery Agent and Traceability Agent using Docker
  containers so that you can manage  your AWS API Gateway environment within
  AMPLIFY Central.

  Once agents are correctly deployed, they can collect the data from the AWS API Gateway and send it securely to AMPLIFY Central.
---

## Before you start

* Read [AMPLIFY Central AWS API Gateway connected overview](/docs/central/connect-aws-gateway/)
* [Prepare AMPLIFY Central](/docs/central/connect-aws-gateway/prepare-amplify-central-1/)
* [Prepare AWS API Gateway](/docs/central/connect-aws-gateway/prepare-aws-api-gateway/)
* Docker must be installed and you will need a basic understanding of Docker commands

## Objectives

Learn how to create your Discovery Agent and Traceability Agent configuration files, then install and run your agents.

## Discovery Agent

The Discovery Agent is used to discover new deployments and stage updates to existing deployments. Once they are discovered, the related APIs are published to AMPLIFY Central so that they become available for any consumer.

As soon as an API is published, a new `APIC_ID` tag is added to the stage so that the Discovery Agent will not published it again.

The Discovery Agent only discovers published APIs where the stage has a  tag(s) defined in the agent configuration file. See AWS_DISCOVERYTAGS.

### Create your Discovery Agent configuration

<!-- HTML table removed here, it will need to be added back manually as a Markdown table -->

| Variable name                                                                                                                                                                | Description                                                                                                                                                                                                                                                            |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| AWS variables                                                                                                                                                                |                                                                                                                                                                                                                                                                        |
| AWS_POLLINTERVAL                                                                                                                                                             | The interval at which to poll SQS for messages (ns - default, us, ms, s, m, h). Set to 20s.                                                                                                                                                                            |
| AWS_REGION                                                                                                                                                                   | The region where AWS APIs are stored.                                                                                                                                                                                                                                  |
| AWS_QUEUENAME                                                                                                                                                                | The name of the queue where Discovery Agent will read relevant AWS events. It is the QueueName provided during installation.                                                                                                                                           |
| AWS_LOGGROUPARN                                                                                                                                                              | The ARN for the log group where API Gateway will send Execution events (output of Step 5). See [Prepare AWS Gateway to deploy the Discovery Agent AWS config setup](/docs/central/connect-aws-gateway/prepare-aws-api-gateway/).                                                                                                     |
| AWS_FILTER                                                                                                                                                                   | Filter conditions for discovery based on AWS Stage tags to determine adding the API to Central. [See Filtering APIs to be discovered for conditional expression samples](/docs/central/connect-aws-gateway/filtering-apis-to-be-discovered-1/).                                                                                                |
| AWS_PUSHTAGS                                                                                                                                                                 | Determines whether the AWS Stage tags should be pushed to AMPLIFY Central along with the API definition. Value must be true or false. Default is false.                                                                                                                |
| AWS_AUTH_ACCESSKEY                                                                                                                                                           | The access key of the AWS account where APIs are stored. Generated when the apigw_cloudformation script is run or you can use your own.                                                                                                                                |
| AWS_AUTH_SECRETKEY                                                                                                                                                           | The secret access key of the AWS account where APIs are stored. Generated when the apigw_cloudformation script is run or you can use your own.                                                                                                                         |
| LOG_LEVEL                                                                                                                                                                    | The log level for output messages (debug, info, warn, error).                                                                                                                                                                                                          |
| LOG_FORMAT                                                                                                                                                                   | The format to print log messages (json, line, package).                                                                                                                                                                                                                |
| LOG_OUTPUT                                                                                                                                                                   | The output for the log lines (stdout, file, both).                                                                                                                                                                                                                     |
| LOG_PATH                                                                                                                                                                     | The path (relative or absolute) to save logs files, if output type file or both.                                                                                                                                                                                       |
| AMPLIFY Central variables                                                                                                                                                    |                                                                                                                                                                                                                                                                        |
| CENTRAL_URL                                                                                                                                                                  | The URL to the AMPLIFY Central instance being used for this Discovery Agent.                                                                                                                                                                                           |
| CENTRAL_TENANTID                                                                                                                                                             | The Organization ID from AMPLIFY Central. Locate this at Platform > User > Organization.                                                                                                                                                                               |
| CENTRAL_TEAMID                                                                                                                                                               | The Team ID in AMPLIFY Central that all APIs will be linked. Locate this at AMPLIFY Central > Access > Teams. Open the teams details. The team identifier is the last part of the url (AMPLIFY URL/access/teams/detail/`e4ec6c1a69fd0b8e016a0bb0681e0e8f`).                                                                                                                                                     |
| CENTRAL_MODE                                                                                                                                                                 | Method to send endpoints back to Central. (publishToEnvironment = API Server, publishToCatalog = Catalog, publishToEnvironmentAndCatalog = API Service and as Consumer instance).                                                                                                                                                                            |
| CENTRAL_AUTH_URL                                                                                                                                                             | The AMPLIFY login URL: <https://login.axway.com/auth>                                                                                                                                                                                                                    |
| CENTRAL_AUTH_REALM                                                                                                                                                           | The Realm used to authenticate for AMPLIFY Central.                                                                                                                                                                                                                    |
| CENTRAL_AUTH_CLIENTID                                                                                                                                                        | The name of the Service Account created in AMPLIFY Central. Locate this at AMPLIFY Central > Access > Service Accounts.                                                                                                                                                |
| CENTRAL_AUTH_PRIVATEKEY                                                                                                                                                      | The private key associated with the Service Account.                                                                                                                                                                                                                   |
| CENTRAL_AUTH_PUBLICKEY                                                                                                                                                       | The public key associated to the Service Account. Extract using the following commands: `openssl genpkey -algorithm RSA -out ./private_key.pem -pkeyopt rsa_keygen_bits:2048` `openssl rsa -pubout -in ./private_key.pem -out ./public_key.pem` `openssl rsa -pubout -in ./private_key.pem -out ./public_key.der -outform der` `base64 ./public_key.der > ./public_key` If the keys for APIC Service Account have already been generated, then only the 3rd and 4th bullet points need to be run using the public key that was previously generated. |
| CENTRAL_AUTH_KEYPASSWORD                                                                                                                                                     | The password for the private key, if applicable.                                                                                                                                                                                                                       |
| CENTRAL_AUTH_TIMEOUT                                                                                                                                                         | The timeout to wait for the authentication server to respond (ns - default, us, ms, s, m, h). Set to 10s.                                                                                                                                                              |
| CENTRAL_ADDITIONALTAGS                                                                                                                                                       | Additional tag names to publish separated by a comma.                                                                                                                                                                                                                  |
| CENTRAL_APISERVERENVIRONMENT                                                                                                                                                 | Environment eventually set by download kit in APIC.                                                                                                                                                                                                                    |
| CENTRAL_SSL_MINVERSION                                                                                                                                                       | String value for the minimum SSL/TLS version that is acceptable. If zero, empty TLS 1.0 is taken as the minimum. Allowed values are: TLS1.0, TLS1.1, TLS1.2, TLS1.3.                                                                                                   |
| CENTRAL_SSL_MAXVERSION                                                                                                                                                       | String value for the maximum SSL/TLS version that is acceptable. If empty, then the maximum version supported by this package is used, which is currently TLS 1.3. Allowed values are: TLS1.0, TLS1.1, TLS1.2, TLS1.3.                                                 |
| CENTRAL_SSL_CIPHERSUITES                                                                                                                                                     | An array of strings. It is a list of supported cipher suites for TLS versions up to TLS 1.2. If CipherSuites is nil, a default list of secure cipher suites is used, with a preference order based on hardware performance. [See Supported Cipher Suites](/docs/central/connect-aws-gateway/ssl-tls-advanced-1/).               |
| CENTRAL_SSL_NEXTPROTOS                                                                                                                                                       | An array of strings. It is a list of supported application level protocols, in order of preference, based on the ALPN protocol list. Allowed values are: h2, htp/1.0, http/1.1, h2c.                                                                                   |
| CENTRAL_SSL_INSECURESKIPVERIFY                                                                                                                                               | InsecureSkipVerify controls whether a client verifies the server's certificate chain and host name. If true, TLS accepts any certificate presented by the server and any host name in that certificate. In this mode, TLS is susceptible to man-in-the-middle attacks. |
| CENTRAL_ADDITIONALTAGS                                                                                                                                                       | Additional tag names to publish separated by a comma.                                                                                                                                                                                                                  |

### Create your Discovery Agent environment file

Create a configuration file using the above variables. See the variable descriptions for their values. Below is a sample of what the configuration file will look like.

For example:  

```
# AWS connectivity
AWS_REGION=us-east-2
AWS_QUEUENAME=aws-apigw-discovery-us-east-2
AWS_AUTH_ACCESSKEY=<YOUR AWS ACCESS KEY HERE>
AWS_AUTH_SECRETKEY=<YOUR AWS SECRET KEY HERE>
AWS_LOGGROUPARN=<YOUR LOG GROUP ARN>
AWS_FILTER=tag.PushToAmplify == true
AWS_PUSHTAGS=true

# AMPLIFY connectivity
CENTRAL_URL=https://apicentral.axway.com

#AMPLIFY Central connectivity
# organisation config:
CENTRAL_TENANTID=<YOUR ORGANISATION ID>
CENTRAL_TEAMID=<THE TEAM ID>
CENTRAL_ENVIRONMENT=<NAME OF THE CENTRAL TOPOLOGY ENVIRONMENT>
CENTRAL_AUTH_CLIENTID=<SERVICE ACCOUNT NAME: DOSA_xxxxxxxxx>

CENTRAL_MODE=<publishToCatalog | publishToEnvironment | publishToEnvironmentAndCatalog>
CENTRAL_AUTH_URL=https://login.axway.com/auth
CENTRAL_AUTH_REALM=Broker
CENTRAL_AUTH_KEYPASSWORD=

#CENTRAL_SSL_MINVERSION=
#CENTRAL_SSL_MAXVERSION=
#CENTRAL_SSL_CIPHERSUITES=
#CENTRAL_SSL_NEXTPROTOS=
#CENTRAL_SSL_INSECURESKIPVERIFY=

LOG_LEVEL=info
LOG_OUTPUT=stdout
LOG_PATH=logs
```

### Install and run Discovery Agent

1. Copy the `private_key.pem` and `public_key` files that were originally created when you set up your Service Account to a keys directory. Make sure the directory is located on the machine being used for deployment. Note that the `public_key` comes from Steps 3 and 4 of [Prepare AWS Gateway to deploy the Discovery Agent AWS config setup](/docs/central/connect-aws-gateway/prepare-aws-api-gateway/).
2. Pull the latest image of the Discovery Agent:

    ```
    docker pull axway-docker-public-registry.bintray.io/agent/aws-apigw-discovery-agent:latest
    ```
3. Start the Discovery Agent pointing to the `env_vars` file and the keys directory:

    ```
    docker run --env-file ./env_vars -v <pwd>/keys:/keys  axway-docker-public-registry.bintray.io/agent/aws-apigw-discovery-agent:latest
    ```

    `pwd` relates to the local directory where the docker command is run. For Windows, the absolute path is preferred.
4. Run the following health check command to ensure the agent is up and running:

    ```
    docker inspect --format='{{json .State.Health}}' <container>
    ```

## Traceability Agent

The Traceability Agent is used to filter the AWS CloudWatch logs that are related to discovered APIs and prepare the transaction events that are sent to AMPLIFY platform. Each time an API is called by a consumer, an event (summary + detail) is sent to AMPLIFY Central and is visible in API Observer.

### Create your Traceability Agent configuration

<!-- HTML table removed here, it will need to be added back manually as a Markdown table -->

| Variable name                  | Description                                                                                                                                                                                                                                                            |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| AWS variables                  |                                                                                                                                                                                                                                                                        |
| AWS_AUTH_ACCESSKEY             | The access key of the AWS account where APIs are stored.                                                                                                                                                                                                               |
| AWS_AUTH_SECRETKEY             | The secret access key of the AWS account where APIs are stored.                                                                                                                                                                                                        |
| AWS_REGION                     | The region where AWS APIs are stored.                                                                                                                                                                                                                                  |
| AWS_POLLINTERVAL               | How often SQS queue is polled.                                                                                                                                                                                                                                         |
| AWS_QUEUNAME                   | The name of the queue (TraceabilityQueueName) from Step 5. This is used for logging custom access log entries. See [Prepare AWS Gateway to deploy the Discovery Agent AWS config setup](/docs/central/connect-aws-gateway/prepare-aws-api-gateway/).                                                                                 |
| Amplify Central variables      |                                                                                                                                                                                                                                                                        |
| LOG_LEVEL                      | The log level for the agent.                                                                                                                                                                                                                                           |
| LOG_FORMAT                     | The format to print log messages (json, line, package).                                                                                                                                                                                                                |
| LOG_OUTPUT                     | The output for the log lines (stdout, file, both)                                                                                                                                                                                                                      |
| LOG_PATH                       | LOG_PATH                                                                                                                                                                                                                                                               |
| LOGSTASH_URL                   | The URL of the logstash to forward the transaction log entries. Default value is ingestion-lumberjack.beta.trcblt.com:453                                                                                                                                              |
| CENTRAL_TENANTID               | The Organization ID from AMPLIFY Central. Locate this ID in the Platform > Organization > Org ID.                                                                                                                                                                      |
| CENTRAL_DEPLOYMENT             | The APIC deployment environment.                                                                                                                                                                                                                                       |
| CENTRAL_ENVIRONMENTID          | The unique character string that identifies your monitoring environment. e4e0810XXXXXXXXX                                                                                                                                                                              |
| CENTRAL_AUTH_URL               | The AMPLIFY login URL: <https://login.axway.com/auth>                                                                                                                                                                                                                    |
| CENTRAL_AUTH_REALM             | The Realm used to authenticate for AMPLIFY Central. Locate this in AMPLIFY Central > Access > Service Accounts.                                                                                                                                                        |
| CENTRAL_AUTH_CLIENTID          | The name of the Service Account created in AMPLIFY Central. Locate this in AMPLIFY Central > Access > Service Accounts. DOSA_XXXXXXXXXx                                                                                                                                |
| CENTRAL_AUTH_PRIVATEKEY        | The private key associated with the Service Account.                                                                                                                                                                                                                   |
| CENTRAL_AUTH_PUBLICKEY         | The public key associated to the Service Account.                                                                                                                                                                                                                      |
| CENTRAL_AUTH_KEYPASSWORD       | The password for the private key, if applicable.                                                                                                                                                                                                                       |
| CENTRAL_SSL_MINVERSION         | String value for the minimum SSL/TLS version that is acceptable. If zero, empty TLS 1.0 is taken as the minimum. Allowed values are: TLS1.0, TLS1.1, TLS1.2, TLS1.3.                                                                                                   |
| CENTRAL_SSL_MAXVERSION         | String value for the maximum SSL/TLS version that is acceptable. If empty, then the maximum version supported by this package is used, which is currently TLS 1.3. Allowed values are: TLS1.0, TLS1.1, TLS1.2, TLS1.3.                                                 |
| CENTRAL_SSL_CIPHERSUITES       | An array of strings. It is a list of supported cipher suites for TLS versions up to TLS 1.2. If CipherSuites is nil, a default list of secure cipher suites is used, with a preference order based on hardware performance. See [Supported Cipher Suites] (/docs/central/connect-aws-gateway/ssl-tls-advanced-1/).               |
| CENTRAL_SSL_NEXTPROTOS         | An array of strings. It is a list of supported application level protocols, in order of preference, based on the ALPN protocol list. Allowed values are: h2, htp/1.0, http/1.1, h2c                                                                                    |
| CENTRAL_SSL_INSECURESKIPVERIFY | InsecureSkipVerify controls whether a client verifies the server's certificate chain and host name. If true, TLS accepts any certificate presented by the server and any host name in that certificate. In this mode, TLS is susceptible to man-in-the-middle attacks. |

### Create your Traceability Agent environment file

Create a configuration file using the above variables. See the variable descriptions for their values. Below is a sample of what the configuration file will look like.  

For example:

```
# AWS connectivity
AWS_REGION=us-east-2
AWS_QUEUENAME=aws-apigw-traceability-us-east-2
AWS_AUTH_ACCESSKEY=<YOUR AWS ACCESS KEY HERE>
AWS_AUTH_SECRETKEY=<YOUR AWS SECRET KEY HERE>

#AMPLIFY Central connectivity
# organisation config:
CENTRAL_TENANTID=<YOUR ORGANISATION ID>
CENTRAL_ENVIRONMENT=<NAME OF THE CENTRAL TOPOLOGY ENVIRONMENT>
CENTRAL_AUTH_CLIENTID=<SERVICE ACCOUNT NAME: DOSA_xxxxxxxxx>

CENTRAL_DEPLOYMENT=prod
CENTRAL_AUTH_URL=https://login.axway.com/auth
CENTRAL_AUTH_REALM=Broker

#CENTRAL_SSL_MINVERSION=
#CENTRAL_SSL_MAXVERSION=
#CENTRAL_SSL_CIPHERSUITES=
#CENTRAL_SSL_NEXTPROTOS=
#CENTRAL_SSL_INSECURESKIPVERIFY=


#Condor url
LOGSTASH_URL=ingestion-lumberjack.datasearch.axway.com:453

LOG_LEVEL=debug
LOG_OUTPUT=stdout
LOG_PATH=logs
```

### Install and run Traceability Agent

1. Copy the `private_key.pem` and `public_key` files that were originally created when you set up your Service Account to a keys directory. Make sure the directory is located on the machine being used for deployment.
2. Pull the latest image of the Traceability Agent:

    ```
    docker pull axway-docker-public-registry.bintray.io/agent/aws-apigw-traceability-agent:latest
    ```
3. Start the Traceability Agent pointing to the `env_vars` file and the `keys` directory:

    ```
    docker run --env-file ./env_vars -v <pwd>/keys:/keys  axway-docker-public-registry.bintray.io/agent/aws-apigw-traceability-agent:latest
    ```

    `pwd` relates to the local directory where the docker command is run. For Windows, the absolute path is preferred.
4. Run the following health check command to ensure the agent is up and running:

    ```
    docker inspect --format='{{json .State.Health}}' <container>
    ```