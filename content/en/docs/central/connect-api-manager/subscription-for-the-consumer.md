---
title: Subscription for the consumer
linkTitle: Subscription for the consumer
draft: false
weight: 100
description: A subscription provides the consumer, or subscriber, with the
  required security, quota and endpoint materials to correctly consume the API.
---
## Subscription workflow

1. An administrator creates an application on Axway API Manager that provides the necessary security feature (API key / OAuth...) and quota, if needed:

   * Add a custom field to the application to track the AMPLIFY Central subscription. Refer to `<API_Gateway_install_dir>/apigateway/webapps//apiportal/vordel/apiportal/app/app.config file` in the **customPropertiesConfig** section. For more details, see [Customize API Manager](https://docs.axway.com/bundle/axway-open-docs/page/docs/apim_administration/apimgr_admin/api_mgmt_custom/index.html).

       Sample application:

     ```
     customPropertiesConfig: {
             user: {
                 // custom properties...
             },
             organization: {
                 // custom properties...
             },
             application: {
                 subscriptions: {
                     label: 'Subscriptions'
                 },
             },
             api: {
                 // custom properties...
             }
         }
     ```
2. A consumer initiates the subscription in AMPLIFY Central:

   1. Open an AMPLIFY Catalog item.
   2. Click **Subscribe**.
   3. Enter the Team and API Manager Application name (created in Step 1). **Warning**: The names must match. Otherwise, the subscription will fail.  For additional information, see [Manage AMPLIFY Catalog subscriptions.](https://docs.axway.com/bundle/axway-open-docs/page/docs/catalog/manage_subscriptions/index.html)

3. The Discovery Agent receives the subscription event:

   * If subscription status: **Subscribing...**

      * Associate the API to the selected application.
      * Send back the subscription status.

   * If subscription status: **Active**

      * Subscription ID is automatically added to the **Custom** field of the application.
      * If failure, subscription status: **Subscription failed**. Refer to the Discovery Agent log for more information. You can delete the subscription and start again from Step 2.

4. The subscriber consumes the API:

   * The API can be consumed once the subscription details are received.

{{< alert title="Note" color="primary" >}}If the FrontEnd API on API Manager corresponding to the Catalog item is set to **unpublished** at the time the subscription is initiated, the Discovery Agent will receive the event, but will not allow the subscription to be completed. Instead, it will send back a subscription status of **Subscribe failed**.{{< /alert >}}

{{< alert title="Note" color="primary" >}}The API Manager application and the API must be in the same organization. Otherwise,  an error message is displayed in the Discovery Agent log.{{< /alert >}}

**Workaround**: You can grant the API access to the organization where the application belongs:

1. In the UI, select the API.
2. Expand **Manage selected**.
3. Select **Grant access**.

## Unsubscribe workflow

1. A consumer initiates unsubscribe:

   1. Open the AMPLIFY Catalog and navigate to the **Subscription** tab.
   2. Delete the subscription. For additional information, see [Manage AMPLIFY Catalog subscriptions](https://docs.axway.com/bundle/axway-open-docs/page/docs/catalog/manage_subscriptions/index.html).

2. The Discovery Agent receives the Unsubscribe event:

   * The subscription ID is removed from the application's Custom field.

## Impact on subscription when unpublishing an API

1. In API Manager, assume there is a FrontEnd API that is published, has been discovered by the Discovery Agent, and has an active subscription to it in AMPLIFY Central.
2. A user in API Manager unpublishes that API
3. The Discovery Agent discovers the change and:

   * Initiates an unsubscribe to AMPLIFY Central for that Catalog item.
   * The subscription ID is removed from the application's Custom field.
   * The subscription status is set to **Unsubscribed**.

{{< alert title="Note" color="primary" >}}Depending on the poll interval settings for the Discovery Agent, it will take a little time from when the user unpublishes the API until AMPLIFY Central shows the subscription state of **Unsubscribed**. This is because of the time it takes to discover the change on API Manager and send events back and forth between API Manager and AMPLIFY Central.{{< /alert >}}

For additional information, see [Manage AMPLIFY Catalog subscriptions](https://docs.axway.com/bundle/axway-open-docs/page/docs/central/catalog/catalog/index.html).
