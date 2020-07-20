---
title: Install AMPLIFY Central CLI
linkTitle: Install AMPLIFY Central CLI
weight: 90
date: 2020-05-29T00:00:00.000Z
description: Learn how to install the AMPLIFY CLI and authorize it to use the AMPLIFY Central APIs. This enables you to integrate the CLI into your DevOps pipeline.
---

## Before you start

* You will need a basic understanding of OAuth authorization ([RFC 6749](https://tools.ietf.org/html/rfc6749)) and JWT ([RFC 7523](https://tools.ietf.org/html/rfc7523))
* You will need an administrator account for AMPLIFY Central ([Managing Accounts](https://docs.axway.com/bundle/AMPLIFY_Dashboard_allOS_en/page/managing_accounts.html))

## Install AMPLIFY CLI and AMPLIFY Central CLI

1. Install `Node.js 8 LTS` or later (`Node.js 11` and later is not supported).
2. Run the following command to install AMPLIFY CLI:

    ```
    [sudo] npm install -g @axway/amplify-cli
    ```

    {{< alert title="Note" color="primary" >}}Use `sudo` on Mac OS X or Linux if you do not own the directory where npm installs packages to. On Windows, you do not need to run as     Administrator as npm installs packages into your AppData directory.{{< /alert >}}

3. Run AMPLIFY package manager to install AMPLIFY Central CLI:

    ```
    amplify pm install @axway/amplify-central-cli
    ```

4. Run AMPLIFY package manager list command to view available packages.

    ```
    amplify pm list
    AMPLIFY CLI, version 1.4.0
    Copyright (c) 2018, Axway, Inc. All Rights Reserved.
    NAME                           | INSTALLED VERSIONS             | ACTIVE VERSION
    @axway/amplify-central-cli     | 0.1.4,0.1.3-dev.10             | 0.1.4
    ```

All the development versions of AMPLIFY Central CLI can be found at [NPM install of AMPLIFY Central CLI](https://www.npmjs.com/package/@axway/amplify-central-cli). To install a specific development version, run the following command:

```
amplify pm install @axway/amplify-central-cli@0.1.3-dev.10
```

## Authorize your CLI to use the AMPLIFY Central APIs

Before using the AMPLIFY Central APIs you must first authorize your CLI, so you can use it, for example, as part of your DevOps pipeline.
There are two ways to authorize your CLI:

* [Use your AMPLIFY Platform login credentials](/docs/central/cli_central/cli_install#login-with-your-amplify-platform-credentials).
* [Use a service account](/docs/central/cli_central/cli_install#authenticate-and-authorize-your-service-account).

### Log in with your AMPLIFY Platform credentials

To use Central CLI to log in with your AMPLIFY Platform credentials, run the following command and used `apicentral` as the client identifier:

```
amplify auth login --client-id apicentral
```

Enter valid credentials (email address and password) on the dialog box displayed.
An "Authorization Successful" message is displayed, and you can go back to the Central CLI.

If you are a member of multiple AMPLIFY organizations, you may have to choose an organization.

To check that your client identifier is set correctly to `apicentral`, run:

```
amplify central config list
{ 'client-id': 'apicentral' }
```

If the client identifier is not set to `apicentral`, set the client identifier for future operations::

```
amplify central config set --clientid=apicentral
```

You have completed the authorization of your CLI with your AMPLIFY Platform credentials.

### Authenticate and authorize your service account

To use the Central CLI, your service account must authenticate with AMPLIFY Platform and it must be authorized to use the AMPLIFY Central APIs.

To support DevOps service interactions, AMPLIFY Central uses the OAuth 2.0 client credentials flow with JWT:

* Create an RSA public private key pair for your DevOps service account.
* Use the public key to register the service account with AMPLIFY Platform to obtain a client ID.
* Use the client ID and private key to authenticate with AMPLIFY Platform to obtain a JWT.
* Use the JWT to make authorized API calls to AMPLIFY Central.

#### Generate an RSA key pair

To authorize a service account with AMPLIFY Platform, you must have a public and private key pair in RSA format. To create this key pair, use `openssl` as follows:

```
$ openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048
..............................................................+++
.........................+++

user@test123 ~/test
$ openssl rsa -pubout -in private_key.pem -out public_key.pem
writing RSA key

user@test123 ~/test
$ ls
private_key.pem  public_key.pem
```

Alternatively, you can create this key pair using `openssh` and the _PKCS8_ format as follows:

```
# private key generation
ssh-keygen -t rsa -b 2048 -m PEM
# public key generation
ssh-keygen -f <public_key_name> -e -m PKCS8
```

#### Create a service account

Log in to AMPLIFY Central UI as an administrator, and create a service account for your CLI. Add the public key that you created earlier. When the account is created, copy the client identifier from the **Client ID** field.

Watch the animation to learn how to do this in AMPLIFY Central UI.

![Create service account in AMPLIFY Central UI](/Images/central/service_account_animation.gif)

#### Authorize the service account with AMPLIFY platform

To authorize the service account with AMPLIFY platform, log in to AMPLIFY CLI using the following command:

```
amplify auth login --client-id DOSA_105cf15d051c432c8cd2e1313f54c2da --secret-file ~/test/private_key.pem

AMPLIFY CLI, version 1.4.0
Copyright (c) 2018, Axway, Inc. All Rights Reserved.

You are logged into 8605xxxxxxx28 as DOSA_5ed74d68defxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx604.

This account has been set as active.
```

#### Set the active service account

To set the service account client identifier for future operations:

```
amplify central config set --client-id DOSA_105cf15d051c432c8cd2e1313f54c2da
```

To view the saved configuration:

```
amplify central config list
{ 'client-id': 'DOSA_105cf15d051c432c8cd2e1313f54c2da' }
```

## Review

You have learned how to install the AMPLIFY CLI and how to register or link it to a service account, or to the AMPLIFY Platform account. Now, you can integrate the AMPLIFY CLI into your DevOps pipeline to automate actions in AMPLIFY Central.
