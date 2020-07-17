---
title: Prepare AMPLIFY Central
linkTitle: Prepare AMPLIFY Central
draft: false
weight: 10
description: >-
  Learn how to represent AWS API Gateway inside AMPLIFY Central by using an
  environment,

  secure the connection between AMPLIFY Central and the agents using a Service Account.
---

## Before you start

* Read [AMPLIFY Central AWS API Gateway connected overview] (/docs/central/connect-aws-gateway/)
* Verify that @axway/amplify-central-cli version is at minimum 0.1.3-dev.3 (Get the [CLI](/docs/central/cli_central/cli_install/))
    * Check the installed version with `amplify central -v`
* openssl installed

## Objectives

Learn how to create a Service Account and an environment for AWS API Gateway within AMPLIFY Central.

## Create a Service Account

In order to secure the connection between agents and AMPLIFY Central, a Service Account is required.

A Service Account authenticates your agents without requiring any user information but uses a public/private key pair.

1. Generate a private and public key pair:

    ```
    openssl genpkey -algorithm RSA -out ./aws_agent_private_key.pem -pkeyopt rsa_keygen_bits:2048
    openssl rsa -pubout -in ./aws_agent_private_key.pem -out ./aws_agent_public_key.pem
    openssl rsa -pubout -in ./aws_agent_private_key.pem -out ./aws_agent_public_key.der -outform der
    (optional) base64 ./public_key.der > ./public_key
    ```

    {{< alert title="Note" color="primary" >}}The public key can be either of type .der format or of type base64 encoded of the .der format.{{< /alert >}}

2. Create a new Service Account user in AMPLIFY Central using the key pair from above. You may name this Service Account for instance AWS-EMEA-Agent. For additional information, see [Service account authentication and authorization](/docs/central/cli_getstarted/index.html). There is no need to download the Service Account JSON-File.

## Create an environment

Create an environment object in AMPLIFY Central that represents the effective API Gateway environment. Depending on your needs, you can create as many environments as required.

Each discovered API or Traffic is associated to this environment and eases the filtering.

You can create your environment using either the UI or CLI.

### Create environment using the UI

Create an environment in **AMPLIFY Central UI > Topology > Environments > create** and give it a relevant name. It is not necessary to have a real environment at this point, but it is important to have an environmentID. You can find this ID in your environment details in the UI.

Example:

```
https:/<AMPLIFY Central URL>/topology/environments/**e4e08e926cb4b22d016cb5f1f0a20019**
```

**Bold** characters are your environmentID.

### Create environment using the CLI

Examples:

```
amplify central config set --client-id <DOSA account name>
amplify central create environment <name> -f <filename>
amplify central create env <name> -o json
```

Options:

```
-o, --output = yaml | json
-f, --file = (filename.yml, filename.yaml, or filename.json)
```

#### Sample environment file

```json
[
  {
    "apiVersion": "v1alpha1",
    "group": "management",
    "title": "AWS us-east",
    "name": "aws-us-east",
    "kind": "Environment",
    "attributes": {
      "createdBy": "json",
      "randomNum": "1"
    },
    "tags": [
      "cli",
      "axway"
    ],
    "spec": {
      "description": "AWS environment US-EAST"
    }
  }
]
```

For information, see [Manage an environment using AMPLIFY CLI](/docs/central/cli_central/cli_environments/)