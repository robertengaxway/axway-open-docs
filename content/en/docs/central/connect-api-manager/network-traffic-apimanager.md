---
title: Network traffic - API Manager
linkTitle: Network traffic - API Manager
draft: false
weight: 120
description: Traffic is always initiated by the Agent to API Manager, API
  Gateway, and AMPLIFY Central. No sessions are ever initiated back to the
  Agent.
---

## Data destinations

The destination for:

* Agent Authentication data is `login.axway.com`
* API definition (Swagger or WSDL) and API documentation data is `apicentral.axway.com`
* API Event data, the transaction summary and headers, is `ingestion-lumberjack.datasearch.axway.com`
* Subscription notification for getting platform user information is `platform.axway.com`

## Communication ports

All outbound traffic is sent over SSL via TCP / UDP.

Open the following ports so that agents can communicate to the AMPLIFY platform:

**Outbound**:

| Host                                      | Port | Protocol | Data                               |
| ----------------------------------------- | ---- | -------- | ---------------------------------- |
| apicentral.axway.com                      | 443  | HTTPS    | API definitions, Subscription info |
| login.axway.com                           | 443  | HTTPS    | Authentication                     |
| ingestion-lumberjack.datasearch.axway.com | 453  | TCP      | API event data                     |
| platform.axway.com                        |      | HTTPS    | Platform user info                 |

Other ports which may need to be opened so that the Agent may monitor API Gateway / Manager are:

**Internal**:

| Host             | Port           | Protocol | Data                                                                                                   |
| ---------------- | -------------- | -------- | ------------------------------------------------------------------------------------------------------ |
| API Manager Host | 8075 (default) | HTTPS    | API Discovery                                                                                          |
| API Gateway Host | 8090 (default) | HTTPS    | API Transaction Header data (see [APIGATEWAY GETHEADERS](/docs/central/traceability-agent-variables/)) |

**Inbound (used for the agent status server)**:

| Host          | Port           | Protocol | Data                                                               |
| ------------- | -------------- | -------- | ------------------------------------------------------------------ |
| Agent Host(s) | 8989 (default) | HTTPS    | Serves the status of the agent and its dependencies for monitoring |

## Subscription notifications

SMTP and/or a webhook URL can be set up to send subscription notifications on both subscribe and unsubscribe actions.  You can find the configuration to set up the SMTP or webhook URL here [Deploy your agents](https://docs.axway.com/bundle/axway-open-docs/page/docs/central/connect-api-manager/deploy-your-agents/index.html)

## Using proxies

If your network policy restricts outbound traffic such that outbound traffic must pass through a proxy. A proxy can be configured in the configuration file of the agents.

### HTTP/HTTPS Proxy

Use a HTTP/HTTPS Proxy for communication to the AMPLIFY Platform.  This configuration is set for both the [Discovery](/docs/central/discovery-agent-variables/) and [Traceability Agents](/docs/central/traceability-agent-variables/).

### SOCKS5 Proxy

Use a SOCKS5 Proxy for communication to the AMPLIFY Platform when sending API Traffic Events.  This configuration is set only for [Traceability Agents](/docs/central/traceability-agent-variables/).

### Proxy authentication

Both proxy types will use one of two authentication mechanisms, none or username/password authentication. The username authentication is specified within the URL `http://{user}:{pass}@{proxy}:{port}`.

## Validation

### Direct Connection

**Connecting to AMPLIFY Central and Login hosts:**

```shell
curl -s -o /dev/null -w "%{http_code}"  https://apicentral.axway.com
```

```shell
curl -s -o /dev/null -w "%{http_code}"  https://login.axway.com
```

A return of **"200"** validates the connection was established.

**Connecting to AMPLIFY Central Event Traffic host:**

```shell
curl ingestion-lumberjack.datasearch.axway.com:453
```

A return of **"curl: (52) Empty reply from server"** validates the connection was established.

### Connection via Proxy

**Connecting to AMPLIFY Central and Login hosts:**

```shell
curl -x {{proxy_host}}:{{proxy_port}} -s -o /dev/null -w "%{http_code}"  https://apicentral.axway.com
```

```shell
curl -x {{proxy_host}}:{{proxy_port}} -s -o /dev/null -w "%{http_code}"  https://login.axway.com
```

A return of **"200"** validates the connection was established.

**Connecting to AMPLIFY Central Event Traffic host:**

```shell
curl -x socks5://{{proxy_host}}:{{proxy_port}} ingestion-lumberjack.datasearch.axway.com:453
```

A return of **"curl: (52) Empty reply from server"** validates the connection was established.

## Troubleshooting

### Curl connection to ingestion-lumberjack.datasearch.axway.com

* **Error:**

  ```shell
  curl: (6) Could not resolve host: ingestion-lumberjack.datasearch.axway.com
  ```

    * **Cause:** The host making the call can’t resolve the ingestion-lumberjack DNS name.

    * **Possible Resolution:** Tell curl to resolve the hostname on the proxy:

    ```shell
    curl -x socks5h://{{proxy_host}}:{{proxy_port}} ingestion-lumberjack.datasearch.axway.com
    ```
* **Error:**

  ```shell
  curl: (7) No authentication method was acceptable.
  ```

    * **Cause:** The SOCKS proxy server expected an authentication type other than what was specified.

    * **Possible Resolution:** Provide authentication to the proxy:

    ```shell
    socks5://{{username}}:{{password}}@{{proxy_host}}:{{proxy_port}}
    ```

    The Agents only support the use of username/password authentication method for SOCKS connections.
