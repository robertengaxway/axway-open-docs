---
title: Deploy your agents
linkTitle: Deploy your agents
draft: false
weight: 30
description: Learn how to deploy your Discovery Agent and Traceability Agent so
  that you can manage  your Axway API Gateway environment within AMPLIFY
  Central.
---

## Before you start

* Read [AMPLIFY Central and Axway API Manager connected overview](/docs/central/connect-api-manager/)
* Be sure you have [Prepared AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/index.html)
* You will need a basic knowledge of Axway API Management solution:

    * Where the solution is running (host / port / path to the logs / users)
    * How to create / publish and API
    * How to call an API

## Objectives

Learn how to install, customize and run your Discovery and Traceability agents.

## Discovery Agent

The Discovery Agent is used to discover new published APIs or any updated APIs. Once they are discovered, the related APIs are published to AMPLIFY Central, in one of the following publication modes, so that they become available for any consumer:

* Catalog item publication (disconnected mode): Customers expose their APIs globally for their consumers but keep the API management at the Gateway level.
* Environment / API Service publication (connected mode): Customers manage their APIs from the AMPLIFY platform.

The Discovery Agent only discovers APIs that have the tag(s) defined in the agent configuration file. See [filtering apis to be discovered](/docs/central/connect-api-manager/filtering-apis-to-be-discovered/). By default, the filter is empty and thus the agent will discover all published API.

As soon as an API is published, the identifier of the asset in AMPLIFY Central is kept in a custom field at the API level in API Manager to help the agent remember what is already published.

The agent can run in the following modes:

* With a yaml configuration file having the same name as the agent binary - `discovery_agent.yml`:

    * Default: located in the same directory as the agent binary.
    * Optional: use a dedicated folder where the configuration file is located (use the --pathConfig flag in the agent command line to access the file path).
    * Advanced configuration: properties inside the configuration file can reference environment variables. This enables you to set up only one configuration file that addresses different behaviors (depending on the environment variables). See [Discovery Agent variables](/docs/central/connect-api-manager/discovery-agent-variables/).

* With command line arguments. See [Discovery Agent flags](/docs/central/connect-api-manager/discovery-agent-flags/).

### Installing the Discovery Agent

**Step 1**: Download the latest version of the zip file from the Axway public repository using the following command:

```
curl -L "https://axway.bintray.com/generic-repo/v7-agents/v7_discovery_agent/latest/discovery_agent-latest.zip" -o discovery_agent-latest.zip
```

**Step 2**: Unzip the file discovery_agent-latest.zip to get the agent binary (discovery_agent) and a template configuration file (discovery_agent.yml).

```
unzip discovery_agent-latest.zip
```

**Step 3**: Copy those 2 files into a folder (/home/APIC-agents for instance) on the machine where the API Manager environment is located.

**Step 4**: Move the `private_key.pem` and `public_key.pem` files that were originally created when you set up your Service Account to the agent directory (APIC-agents). Note that the `public_key` comes from Steps 3 or 4 of [Create a Service Account](/docs/central/connect-api-manager/prepare-amplify-central/#create-a-service-account) depending if you choose to use the `der` format or not.

### Customizing the Discovery Agent configuration file

The `discovery_agent.yml` configuration file contain 3 sections to personalize: apimanager, central and log.

#### Customizing apimanager section

This section connects the agent to API Manager and determines which APIs should be discovered and published to AMPLIFY Central.

`host`: Machine name where API Manager is running. Use a hostname according to the certificate returned by the API-Manager.

`port`: API Manager port number (8075 by default).

`discoveryIgnoreTags` (optional): Comma-separated blacklist of tags. If an API has one or several of these blacklist tags, the agent ignores this API and will not publish it to AMPLIFY Central. This property takes precedence over the filter property below. The default value is empty, which means no API is ignored.

`filter` (optional): Expression to filter the API you want the agent to discover. See [Filtering APIs to be discovered](/docs/central/connect-api-manager/filtering-apis-to-be-discovered/). Leaving this field empty tells the agent to discover all published APIs (REST / SOAP).

`proxyApicIDField` (optional): The field name used to store AMPLIFY Central identifier for the front-end proxy in API Manager. Default value is **apicId**. If you do not intend to change it, comment this property. Be aware that the field will not be visible in the API Manager front-end proxy, as it is a specific configuration. If you want to see that field or customize it, refer to Add a custom property to APIs in [Customize API Manager](/docs/apim_administration/apimgr_admin/api_mgmt_custom/index.html#customize-api-manager-data) documentation.

`subscriptionApplicationField` (optional): The field name used to store AMPLIFY Central subscription identifier inside the API Manager application securing the front end proxy. Default value is **subscriptions**. If you do not intend to change it, comment this property. Be aware that the field will not be visible in the API Manager application, as it is a specific configuration. If you want to see that field or customize it, refer to Add a custom property to applications in [Customize API Manager](/docs/apim_administration/apimgr_admin/api_mgmt_custom/index.html#customize-api-manager-data) documentation.

`pollInterval`: The frequency in which API Manager is polled for new endpoints. Default value is 30s.

`auth.username`: An API Manager user the agent will use to connect to the API Manager. This user must have either the “API Manager Administrator” or “Organization administrator” role. Based on the role of this user, the agent is able to:

* discover any API from any organization (“API Manager Administrator”)  
* discovery any API from a specific organization (“Organization administrator”)

`auth.password`: The password of the API Manager user in clear text.

Once all data is gathered, this section should looks like:

```
apimanager:
  host: localhost
  port: 8075
  discoveryIgnoreTags: tag1, tag2
  filter:
#  proxyApicIDField: apicId
#  subscriptionApplicationField: subscriptions
  pollInterval: 30s
  auth:
    username: apiManagerUser
    password: apiManagerUserPassword
```

#### Customizing central section

This section connects the agent to AMPLIFY Central and determines how to published the discovered APIs.

`url`: The AMPLIFY Central url. Default value is **<https://apicentral.axway.com>**

`platformURL:` The AMPLIFY platform url. Needed for finding the user email during the subscription process with email notification. Default value is **<https://platform.axway.com>**

`teamID`: The Team identifier in AMPLIFY Central that all APIs will be linked. Locate this at AMPLIFY Central > Access > Team Assets. Open the teams details. The team identifier is the last part of the url (AMPLIFY URL/access/teams/detail/**e4ec6c1a69fd0b8e016a0bb0681e0e8f**).

`tenantID`: The Organization ID from AMPLIFY Central. Locate this at Platform > User > Organization > Org ID field.

`environment`: The environment name you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/).

`apiServerVersion`: The version of AMPLIFY Central API the agent is using. Default value is **v1alpha1**

`mode`: The method to send endpoints back to Central. (publishToEnvironment = API Service, publishToCatalog = Catalog, publishToEnvironmentAndCatalog = API Service and as Consumer instance).  

`pollInterval`: The frequency the agent is polling AMPLIFY Central to get some event. Default value is **30s**

`auth.url`: The AMPLIFY login URL. Default value is **<https://login.axway.com/auth>**

`auth.realm`: The Realm used to authenticate for AMPLIFY Central. Default value is **Broker**

`auth.clientId`: The Client ID of the Service Account (DOSA_....) you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Locate this at AMPLIFY Central > Access > Service Accounts.

`auth.privateKey`: The location of the private key file you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Absolute file path is recommended to avoid confusion.

`auth.publicKey`:  The location of the public key file you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Absolute file path is recommended to avoid confusion.  

`auth.keyPassword`: The key password to open the key. None set up by default.

`auth.timeout`: Timeout for the authentication. Default value is **10s**

Once all data is gathered, this section should look like:

```
central:
  url: https://apicentral.axway.com
  platformURL: https://platform.axway.com
  teamID: e4ec6c1a69fd0b8e016a0bb0681e0e8f
  tenantID: 68794y2
  environment: my-v7-env
  apiServerVersion: v1alpha1
  mode: publishToEnvironmmentAndCatalog
  pollInterval:
  auth:
    url: https://login.axway.com/auth
    realm: Broker
    clientId: DOSA_66743...
    privateKey: /home/APIC-agents/private_key.pem
    publicKey: /home/APIC-agents/public_key
    keyPassword:
    timeout: 10s
```

#### Customizing SMTP Notification (subscription)

The SMTP Notification section defines how the agent manages email settings for subscriptions.

`host`: SMTP server where the email notifications will originate from.

`port`: Port of the SMTP server.

`fromAddress`: Email address which will represent the sender.

`username`: Login user for the SMTP server.

`password`: Login password for the SMTP server.

`subscribe.subject`: Subject of the email notification for action subscribe. Default is **Subscription Notification**.

`subscribe.body`: Body of the email notification for action subscribe. Default is **Subscription created for Catalog Item:  {catalogItem} Subscription key: {subscriptionKey}**.

`unsubscribe.subject`: Subject of the email notification for action unsubscribe. Default is **Subscription Removal Notification**.

`unsubscribe.body`: Body of the email notification for action unsubscribe. Default is **Subscription for Catalog Item: {catalogItem} has been unsubscribed**.

`subscribeFailed.subject`: Subject of the email notification for action subscribe failed. Default is **Subscription Failed Notification**.

`subscribeFailed.body`: Body of the email notification for action subscribe failed. Default is **Could not subscribe to CatalogItem: {catalogItem}**.

`unsubscribeFailed.subject`: Subject of the email notification for action unsubscribe failed. Default is **Subscription Removal Failed Notification**.

`unsubscribeFailed.body` : Body of the email notification for action unsubscribe failed. Default is **Could not unsubscribe to Catalog Item: {catalogItemURL} {catalogItemName}**.

### Customizing Webhook Notification (subscription)

The webhook Notification section defines how the agent manages to send notifications to a webhook URL.

`url`: url where the webhook server is defined

`headers`: information used to verify the webhook. Provided by the customer, and may include such information as contentType and Authorization

Both webhook and smtp  sections can be configured at the same time.  The agent will attempt the subscription notifications that are set in the agent config.

Once all data is gathered, this section should look like this for subscription Notification:

```
subscriptions:
  webhook:
    url: https://prod.cloud-elements.com/elements/webhook-v1/mywebhook
    headers: Header=contentType,Value=application/json, Header=Elements-Formala-Instance-Id,Value=5551212, Header=Authorization,Value=UserF+rYQSfu0w5yIa5q7uNs2MKYcifk8pYpgAUwJtXFnzc=, Organization a171385jbbde8f54f4f55ff8c3bd8bfe
  smtp:
    host: mail.outlook.com
    port: 25
    fromAddress: fromAddress@outlook.com
    username: outlookuser
    password:
    subscribe:
      subject: Subscription Notification
      body: |
        Subscription created for Catalog Item:  <a href= ${catalogItemUrl}> ${catalogItemName} </a> <br/>
        Subscription key: <b>${key}</b>
    unsubscribe:
      subject: Subscription Removal Notification
      body: |
        Subscription for Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName} </a> has been unsubscribed
    subscribeFailed:
      subject: Subscription Failed Notification
      body: |
        Could not subscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName} </a>
    unsubscribeFailed:
      subject: Subscription Removal Failed Notification
      body: |
        Could not unsubscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName} </a>
```

#### Customizing log section (log)

The log section defines how the agent is managing its logs.

`level`: The log level for output messages (debug, info, warn, error). Default value is **info**

`format`: The format to print log messages (json, line, package). Default value is **json**

`output`: The output for the log lines (stdout, file, both). Default value is **stdout**

`path`: The path (relative to the agent binary or absolute) to save logs files, if output type file or both. Default value is relative path **logs**

Once all data is gathered, this section should look like:

```
log:
  level: info
  format: json
  output: stdout
  path: logs
```

#### Validating your custom Discovery Agent configuration file

After customizing all the sections, your discovery_agent.yml file should look like:

```yaml
apimanager:
  host: localhost
  port: 8075
  discoveryIgnoreTags: tag1, tag2
  filter:
#  proxyApicIDField: apicId
#  subscriptionApplicationField: subscriptions
  pollInterval: 30s
  auth:
    username: apiManagerUser
    password: apiManagerUserPassword

central:
  url: https://apicentral.axway.com
  platformURL: https://platform.axway.com
  teamID: e4ec6c1a69fd0b8e016a0bb0681e0e8f
  tenantID: 68794y2
  environment: my-v7-env
  apiServerVersion: v1alpha1
  mode: publishToEnvironmmentAndCatalog
  pollInterval:
  auth:
    url: https://login.axway.com/auth
    realm: Broker
    clientId: DOSA_66743...
    privateKey: /home/APIC-agents/private_key.pem
    publicKey: /home/APIC-agents/public_key
    keyPassword:
    timeout: 10s

subscriptions:
  webhook:
    url: https://prod.cloud-elements.com/elements/webhook-v1/mywebhook
    headers: Header=contentType,Value=application/json, Header=Elements-Formala-Instance-Id,Value=5551212,  Header=Authorization,Value=UserF+rYQSfu0w5yIa5q7uNs2MKYcifk8pYpgAUwJtXFnzc=, Organization a171385jbbde8f54f4f55ff8c3bd8bfe
  smtp:
    host: mail.outlook.com
    port: 25
    fromAddress: fromAddress@outlook.com
    username: outlookuser
    password:
    subscribe:
      subject: Subscription Notification
      body: |
        Subscription created for Catalog Item:  <a href= ${catalogItemUrl}> ${catalogItemName} </a> <br/>
        Subscription key: <b>${key}</b>
    unsubscribe:
      subject: Subscription Removal Notification
      body: |
        Subscription for Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName} </a> has been unsubscribed
    subscribeFailed:
      subject: Subscription Failed Notification
      body: |
        Could not subscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName} </a>
    unsubscribeFailed:
      subject: Subscription Removal Failed Notification
      body: |
        Could not unsubscribe to Catalog Item: <a href= ${catalogItemUrl}> ${catalogItemName} </a>

log:
  level: info
  format: json
  output: stdout
  path: logs
```

### Running the Discovery Agent

Open a shell and run the following command to start up your agent:

```shell
cd /home/APIC-agents
./discovery_agent &

{"level":"info","msg":"Starting Discovery agent for V7 APIGateway (-)","time":"2020-07-06T02:56:20-07:00"}
{"level":"info","msg":"Services are Ready","time":"2020-07-06T02:56:22-07:00"}
{"level":"info","msg":"Found new frontend proxy: EMR-System-Surgery","time":"2020-07-06T02:56:22-07:00"}
{"level":"info","msg":"Found new frontend proxy: Security-HIPAA-Control","time":"2020-07-06T02:56:22-07:00"}
.....
...
```

To verify if the agent is up and running, open another shell command and run:

```shell
cd /home/APIC-agents
./discovery_agent --status

{
  "name": "discovery_agent",
  "version": "-",
  "status": "OK",
  "statusChecks": {
    "apimanager": {
      "name": "API Manager",
      "endpoint": "apimanager",
      "status": {
        "result": "OK"
      }
    },
    "central": {
      "name": "AMPLIFY Central",
      "endpoint": "central",
      "status": {
        "result": "OK"
      }
    }
  }
}
```

## Traceability Agent

The traceability agent is used to filter the Axway API Gateway logs that are related to discovered APIs and prepare the transaction events that are sent to AMPLIFY platform. Each time an already discovered API is called by a consumer, an event (summary + detail) is sent to AMPLIFY Central and is visible in API Observer.

The agent can run in the following modes:

* With a yaml configuration file having the same name as the agent binary - `traceability_agent.yaml`:

    * Default: located in the same directory as the agent binary.
    * Optional: use a dedicated folder where the configuration file is located (use the --path.config flag in the agent command line to access the file path).
    * Advanced configuration: properties inside the configuration file can reference environment variables. This enables you to set up only one configuration file that addresses different behaviors (depending on the environment variables). See [Discovery Agent variables](/docs/central/connect-api-manager/discovery-agent-variables/).
* With command line argument. See [Traceability Agent flags](/docs/central/connect-api-manager/traceability-agent-flags/).

### Installing the Traceability Agent

**step 1**: Download the latest version of the zip file from the Axway public repository using the following command:

```
curl -L "https://axway.bintray.com/generic-repo/v7-agents/v7_traceability_agent/latest/traceability_agent-latest.zip" -o traceability_agent-latest.zip
```

**step 2**: Unzip the file traceability_agent-latest.zip to get the agent binary (traceability_agent) and a template configuration file (traceability_agent.yml).

```
unzip traceability_agent-latest.zip
```

**step 3**: Copy those 2 files into a folder (/home/APIC-agents for instance) on the machine where the API Manager environment is located.

**step 4**: If not done yet, move the `private_key.pem` and `public_key.pem` files that were originally created when you set up your Service Account to the agent directory (APIC-agents). Note that the `public_key` comes from Steps 3 or 4 of [Create a Service Account](/docs/central/connect-api-manager/prepare-amplify-central/#create-a-service-account) depending if you choose to use the `der` format or not.

### Customizing the Traceability Agent configuration file

The `traceability_agent.yml` configuration file contain 6 sections to customize: the beat, logstash, central, apigateway, apimanager and log.

By default the `traceability_agent.yml` file contains references to environment variables: ${VARIABLE_NAME: value}. You can remove the ${VARIABLE_NAME} and keep only the value for simplifying the configuration.

#### Customizing beat section (traceability_agent)

This section describes where the API Gateway logs are located on the machine so the beat can read them.

`paths`: List all API Gateway event log files (absolute path) the beat will listen to. It could be a single file if there is only one gateway installed on the machine, or multiple files if several gateways are installed on the same machine.

Single Gateway - explicit file name

```
traceability_agent:
  inputs:
    - type: log
      paths:
        - <API GATEWAY INSTALL DIRECTORY>/apigateway/events/group-2_instance-1.log
```

Multiple Gateways on the same machine - explicit file names

```
traceability_agent:
  inputs:
    - type: log
      paths:
        - <API GATEWAY INSTALL DIRECTORY>/apigateway/events/group-2_instance-1.log
        - <API GATEWAY INSTALL DIRECTORY>/apigateway/events/group-2_instance-3.log
        - <API GATEWAY INSTALL DIRECTORY>/apigateway/events/group-2_instance-7.log
```

Multiple Gateways on the same machine - file path with wildcard

```
traceability_agent:
  inputs:
    - type: log
      paths:
        - <API GATEWAY INSTALL DIRECTORY>/apigateway/events/group-2_instance-?.log
```

#### Customizing logstash section (output.traceability)

This section describes where the logs should be sent on AMPLIFY Central.

`hosts`: The URL of the logstash to forward the transaction log entries. Default value is **ingestion-lumberjack.datasearch.axway.com:453**

`cipher_suites`: List the cipher suites for the TLS connectivity. See the [SSL / TLS advanced](/docs/central/connect-api-manager/ssl-tls-advanced/) topic for more information

`proxy_url`: The URL for the proxy for logstash (**socks5://username:password@hostname:port**) to use when the API Management eco-system is not allowed to access the internet world where AMPLIFY Central is installed. **username** and **password** are optional and can be omitted if not required by the proxy configuration. Leaving this value empty means that no proxy will be used to connect to AMPLIFY Central logstash.

Once all data is gathered, the section should look like this if you do not use a proxy:

```
# Send output to Central Database
output.traceability:
  enabled: true
  hosts: ingestion-lumberjack.datasearch.axway.com:453
  ssl:
    enabled: true
    verification_mode: none
    cipher_suites:
      - "ECDHE-ECDSA-AES-128-GCM-SHA256"
      - "ECDHE-ECDSA-AES-256-GCM-SHA384"
      - "ECDHE-ECDSA-AES-128-CBC-SHA256"
      - "ECDHE-ECDSA-CHACHA20-POLY1305"
      - "ECDHE-RSA-AES-128-CBC-SHA256"
      - "ECDHE-RSA-AES-128-GCM-SHA256"
      - "ECDHE-RSA-AES-256-GCM-SHA384"
#  proxy_url: socks5://username:password@hostname:port
```

#### Customizing central section (output.traceability.agent.central)

This section connects the agent to AMPLIFY Central and determine how to published the discovered APIs.

`url`: The amplify central url. Default value is **<https://apicentral.axway.com>**

`tenantID`: The Organization ID from AMPLIFY Central. Locate this at Platform > User > Organization > ORrg ID field.

`deployment`: The APIC deployment environment. Default value is **prod**

`environment`: The environment name you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/).

`auth.url`: The AMPLIFY login URL. Default value is **<https://login.axway.com/auth>**

`auth.realm`: The Realm used to authenticate for AMPLIFY Central. Default value is **Broker**

`auth.clientId`: The name of the Service Account you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Locate this at AMPLIFY Central > Access > Service Accounts.

`auth.privateKey`: The location of the private key file you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Absolute file path is recommended to avoid confusion.

`auth.publicKey`:  The location of the public key file you created when [preparing AMPLIFY Central](/docs/central/connect-api-manager/prepare-amplify-central/). Absolute file path is recommended to avoid confusion.  

`auth.keyPassword`: The key password to open the key. None set up by default.

`auth.timeout`: Timeout for the authentication. Default value is **10s**

`proxy_url`: The URL for the proxy for Amplify Central `<http://username:password@hostname:port>`. If empty, no proxy is defined.

`ssl` settings: By default, for connecting to AMPLIFY Central, agent uses TLS 1.2 with a predefined list of cipher suites. Refer to [SSL / TLS advanced](/docs/central/connect-api-manager/ssl-tls-advanced/) section for changing this behavior.

Once all data is gathered, this section should look like:

```
  agent:
    central:
      url: https://apicentral.axway.com
      tenantID: 68794y2
      deployment: prod
      environment: my-v7-env
      auth:
        url: https://login.axway.com/auth
        realm: Broker
        clientId: "DOSA_68732642t64545..."
        privateKey: /home/APIC-agents/private_key.pem
        publicKey: /home/APIC-agents/public_key
        keyPassword: ""
        timeout: 10s
      ssl:
#        minVersion: {CENTRAL_SSL_MINVERSION:""}
#        maxVersion: ${CENTRAL_SSL_MAXVERSION:""}
#        nextProtos: ${CENTRAL_SSL_NEXTPROTOS:[]}
#        cipherSuites: ${CENTRAL_SSL_CIPHERSUITES:[]}
#        insecureSkipVerify: ${CENTRAL_SSL_INSECURESKIPVERIFY:false}
#      proxyUrl: "http://username:password@hostname:port"
```

#### Customizing apigateway section (output.traceability.agent.apigateway)

This section helps the agent to collect the header from request/response from the API Gateway system.

`getHeaders`: Tells the agent to  call the API Gateway API to get additional transaction details (headers). Default value is **true**. If false, API Gateway config does not need to be set and no headers will be send to AMPLIFY Central.

`host`: The host that Axway API Gateway is running on. Default value is **localhost**

`port`: The port that Axway API Gateway is listening on. Default value is **8090**

`pollInterval`: The frequency in which the agent polls the logs in us, ms, s, m, h. Default value is **1m**

`auth.username`: An Axway API Gateway username with the "API Gateway operator" role.

`auth.password`: The Axway API Gateway username password in clear text.

`ssl` settings: By default, for connecting to API Gateway, the agent uses TLS 1.2 with a predefined list of cipher suites. Refer to [SSL / TLS advanced](/docs/central/connect-api-manager/ssl-tls-advanced/) section for changing this behavior.

Once all data is gathered, this section should look like:

```
    apigateway:
      getHeaders: true
      host: localhost
      port: 8090
      pollInterval: 1m
      auth:
        username: myApiGatewayOperatorUser
        password: myApiGatewayOperatorUserPassword
      ssl:
#        minVersison: ${APIGATEWAY_SSL_MINVERSION:""}
#        maxVersion: ${APIGATEWAY_SSL_MAXVERSION:""}
#        nextProtos: ${APIGATEWAY_SSL_NEXTPROTOS:[]}
#        cipherSuites: ${APIGATEWAY_SSL_CIPHERSUITES:[]}
#        insecureSkipVerify: ${APIGATEWAY_SSL_INSECURESKIPVERIFY:false}
#      proxyUrl: ${APIGATEWAY_PROXYURL:""}
```

#### Customizing apimanager section (output.traceability.agent.apimanager)

This section tells the agent which API needs to be monitor: one that has been discovered by the discovery agent (ie. has a non-empty `apicId` custom field).

`host`: The Machine name where API Manager is running. localhost value can be used, as the agent is installed on the same machine as the API Manager.

`port`: The API Manager port number (**8075** by default).

`pollInterval`: The frequency in which API Manager is polled for new endpoints. Default value is 30s.

`apiVersion`: The API Manager API version to use. Default value is **1.3**

`proxyApicIDField` (optional): The field name used to store the AMPLIFY Central identifier for the front-end proxy in API Manager. Default value is **apicId**. If you do not intend to change it, comment this property. Be aware that the field will not be visible in the API Manager front-end proxy, as it is a specific configuration. If you want to see that field or customize it, refer to Add a custom property to APIs in [Customize API Manager](/docs/apim_administration/apimgr_admin/api_mgmt_custom/index.html#customize-api-manager-data) documentation.

`auth.username`: An API Manager user the agent will use to connect to the API Manager. This user must have either an “API Manager Administrator” or “Organization administrator” role. Based on the role of this user, the agent is able to:

* discover any API from any organization (“API Manager Administrator”)  
* discovery any API from a specific organization (“Organization administrator”)

For the traceability agent to report correctly the discovered API traffic, it is recommended to use the same user as the one used for discovering APIs.

`auth.password`: The password of the API Manager user in clear text.

`ssl` settings: By default, for connecting to API Manager, the agent uses TLS 1.2 with a predefined list of cipher suites. Refer to [SSL / TLS advanced](/docs/central/connect-api-manager/ssl-tls-advanced/) section for changing this behavior.

Once all data is gathered, this section should look like:

```
    apimanager:
      host: localhost
      port: 8075
      pollInterval: 1m
      apiVersion: 1.3
      proxyApicIDField: "apicId
      auth:
        username: myAPIManagerUserName
        password: myAPIManagerUserPassword
      ssl:
#        minVersion: ${APIMANAGER_SSL_MINVERSION:""}
#        maxVersion: ${APIMANAGER_SSL_MAXVERSION:""}
#        nextProtos: ${APIMANAGER_SSL_NEXTPROTOS:[]}
#        cipherSuites: ${APIMANAGER_SSL_CIPHERSUITES:[]}
#        insecureSkipVerify: ${APIMANAGER_SSL_INSECURESKIPVERIFY:false}
#      proxyUrl: ${APIMANAGER_PROXYURL:""}
```

#### Customizing log section (logging)

The log section defines how the agent manages its logs.

`to_stderr`: (default configuration) The output is logged onto the screen.

`to_file`:  (alternate configuration) The output is logged into a file. Requires more configuration (refer to <https://www.elastic.co/guide/en/beats/filebeat/current/configuration-logging.html>).

`level`: The log level for output messages (debug, info, warn, error). Default value is **info**

Once all data is gathered, this section should look like this for standard output logging:

```
logging:
  metrics:
    enabled: false
  # Send all logging output to stderr
  to_stderr: true
  # Set log level
  level: info
  # Send all logging output to file - change value to_files: true and to_stderr: false
  to_files: false
  files:
    path: ./logs
    name: traceability_agent.log
    keepfiles: 7
    permissions: 0644
```

#### Validating your custom Traceability Agent configuration file

After customizing all the sections, your traceability_agent.yaml file should look like:

```yaml
################### Beat Configuration #########################
traceability_agent:
  inputs:
    - type: log
      paths:
        - <PATH TO>/group-X_instance-Y.log
      include_lines: ['.*"type":"transaction".*"type":"http".*']

# Send output to Central Database
output.traceability:
  enabled: true
  hosts: ${LOGSTASH_URL:ingestion-lumberjack.datasearch.axway.com:453}
  ssl:
    enabled: true
    verification_mode: none
    cipher_suites:
      - "ECDHE-ECDSA-AES-128-GCM-SHA256"
      - "ECDHE-ECDSA-AES-256-GCM-SHA384"
      - "ECDHE-ECDSA-AES-128-CBC-SHA256"
      - "ECDHE-ECDSA-CHACHA20-POLY1305"
      - "ECDHE-RSA-AES-128-CBC-SHA256"
      - "ECDHE-RSA-AES-128-GCM-SHA256"
      - "ECDHE-RSA-AES-256-GCM-SHA384"
  proxy_url: ${LOGSTASH_PROXYURL:""}
  agent:
    central:
      url: https://apicentral.axway.com
      tenantID: 68794y2
      deployment: prod
      environment: my-v7-env
      auth:
        url: https://login.axway.com/auth
        realm: Broker
        clientId: "DOSA_68732642t64545..."
        privateKey: /home/APIC-agents/private_key.pem
        publicKey: /home/APIC-agents/public_key
        keyPassword: ""
        timeout: 10s
      ssl:
#        minVersion: {CENTRAL_SSL_MINVERSION:""}
#        maxVersion: ${CENTRAL_SSL_MAXVERSION:""}
#        nextProtos: ${CENTRAL_SSL_NEXTPROTOS:[]}
#        cipherSuites: ${CENTRAL_SSL_CIPHERSUITES:[]}
#        insecureSkipVerify: ${CENTRAL_SSL_INSECURESKIPVERIFY:false}
#      proxyUrl: "http://username:password@hostname:port"
    apigateway:
      getHeaders: true
      host: localhost
      port: 8090
      pollInterval: 1m
      auth:
        username: myApiGatewayOperatorUser
        password: myApiGatewayOperatorUserPassword
      ssl:
#        minVersison: ${APIGATEWAY_SSL_MINVERSION:""}
#        maxVersion: ${APIGATEWAY_SSL_MAXVERSION:""}
#        nextProtos: ${APIGATEWAY_SSL_NEXTPROTOS:[]}
#        cipherSuites: ${APIGATEWAY_SSL_CIPHERSUITES:[]}
#        insecureSkipVerify: ${APIGATEWAY_SSL_INSECURESKIPVERIFY:false}
#      proxyUrl: ${APIGATEWAY_PROXYURL:""}
    apimanager:
      host: localhost
      port: 8075
      pollInterval: 1m
      apiVersion: 1.3
      proxyApicIDField: "apicId
      auth:
        username: myAPIManagerUserName
        password: myAPIManagerUserPassword
      ssl:
#        minVersion: ${APIMANAGER_SSL_MINVERSION:""}
#        maxVersion: ${APIMANAGER_SSL_MAXVERSION:""}
#        nextProtos: ${APIMANAGER_SSL_NEXTPROTOS:[]}
#        cipherSuites: ${APIMANAGER_SSL_CIPHERSUITES:[]}
#        insecureSkipVerify: ${APIMANAGER_SSL_INSECURESKIPVERIFY:false}
#      proxyUrl: ${APIMANAGER_PROXYURL:""}

logging:
  metrics:
    enabled: false
  # Send all logging output to stderr
  to_stderr: true
  # Set log level
  level: info
  # Send all logging output to file - change value to_files: true and to_stderr: false
  to_files: false
  files:
    path: ./logs
    name: traceability_agent.log
    keepfiles: 7
    permissions: 0644
```

### Running the Traceability Agent

Open a shell and run the following command to start up your agent:

```shell
cd /home/APIC-agents
./traceability_agent &
```

To verify if the agent is up and running, open another shell command and run:

```shell
cd /home/APIC-agents
./traceability_agent status
```
