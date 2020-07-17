---
title: Prepare AMPLIFY Central
linkTitle: Prepare AMPLIFY Central
draft: false
weight: 20
description: Learn how to create an environment and Service Account for Axway
  API Gateway within AMPLIFY Central.
---

## Before you start

* Read [AMPLIFY Central and Axway API Manager connected overview](/docs/central/connect-api-manager/)
* You will need a basic knowledge of Axway API Manager
* Verify that @axway/amplify-central-cli version is at minimum 0.1.3-dev.3

## Objectives

Learn how to create a Service Account and an environment for Axway API Gateway within AMPLIFY Central.

## Create a Service Account

Create a Service Account in AMPLIFY Central.

1. Generate a private and public key pair:

    ```
    openssl genpkey -algorithm RSA -out ./private_key.pem -pkeyopt rsa_keygen_bits:2048

    openssl rsa -pubout -in ./private_key.pem -out ./public_key.pem

    openssl rsa -pubout -in ./private_key.pem -out ./public_key.der -outform der
    (optional) base64 ./public_key.der > ./public_key
    ```

    {{< alert title="Note" color="primary" >}}The public key can be either of type .der format or of type base64 encoded of the .der format.{{< /alert >}}

2. Create a new Service Account user in API Central UI using the key pair from above. For additional information, see [Create a service account](/docs/central/cli_central/cli_install/).

## Create an environment

Create an environment object in AMPLIFY Central that represents the effective Axway API Gateway environment. Depending on your needs, you can create as many environments as required.

Each discovered API or Traffic is associated to this environment and eases the filtering.

You can create your environment using either the UI, API or CLI.

### Create environment using the UI

Create an environment in **AMPLIFY Central UI > Topology > Environments > create** and give it a relevant name. It is not necessary to have a real environment at this point, but it is important to have an environment name. You can find this environment name in your environment details in the UI.

Example:

```
https:/<AMPLIFY Central URL>/topology/environments/apigtw-v77
```

### Create environment using the CLI

Examples:

```
amplify central config set --client-id <DOSA account name>
amplify central create -f <filename>
amplify central create env <name> -o json
```

Options:

```
-o, --output = yaml | json

-f, --file = (filename.yml, filename.yaml, or filename.json)
```

Sample file:

```
---
group: management
apiVersion: v1alpha1
kind: Environment
name: My beautifull environment name
title: Any usefull title
metadata:
  id: e4e084a66f86a7ea016f8c2ba1a40005
  audit:
    createTimestamp: '2020-01-09T21:17:47.302+0000'
    createUserId: DOSA_91cdec76c1084d86a6ee48f19bc
    modifyTimestamp: '2020-01-09T21:17:47.302+0000'
    modifyUserId: DOSA_91cdec76c1084d86a6ee48f19bc
  resourceVersion: '6'
  references: []
attributes:
  attr1: value1
  attr2: value2
tags:
  - Testing
  - another tag
spec:
  description: My wonderfull description to help me.
  icon:
    contentType: image/png
    data: "[optional base64 encoded image]"
```

For information, see [Manage an environment using AMPLIFY CLI](/docs/central/cli_central/cli_environments/).
