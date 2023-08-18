---
title: Unitary event
description: This is an intruction page for simulating the '[!UICONTROL Unitary event]' type of Journey validation.
---

# Unitary event

>[!IMPORTANT]
>
>These instructions might change across **[!UICONTROL Playbook]** so please always refer the Sample data section of respective **[!UICONTROL Playbook]**.

## Prerequisite

* You must have the Postman software installed
* Use Playbook to create the instances assets like **[!UICONTROL Journey]**, **[!UICONTROL Schemas]**, **[!UICONTROL Segments]**, **[!UICONTROL Messages]** etc.

Created assets will be shown on `Bill Of Material` Page

![Bill Of Material Page](../assets/bom-page.png)

## Prepare Postman with required collection

1. Visit **[!UICONTROL Use Case Playbook]** application.
1. Click on the respective **[!UICONTROL Playbook]** card to visit **[!UICONTROL Playbook]** details page.
1. Visit **[!UICONTROL Bill Of Material]** page and find the **[!UICONTROL Sample data]** section.
1. Download the `postman.json` by clicking the respective buttons on the UI.
1. Import `postman.json` in the **[!DNL Postman Software]**.
1. Create a dedicated Postman Environment for this validation (e.g. `Adobe <PLAYBOOK_NAME>`).

## Fetch IMS Token

>[!NOTE]
>
>All Environment variables are case sensitive so please always use the exact variable name.

1. Please follow [Authenticate and access Experience Platform APIs](https://experienceleague.adobe.com/docs/experience-platform/landing/platform-apis/api-authentication.html) documentation to generate the Access Token.
1. Store the Access Token value in Environment variables named `ACCESS_TOKEN`.
1. Store other authentication related values like `API_KEY`, `IMS_ORG` and `SANDBOX_NAME` in Environment variables.

>[!IMPORTANT]
>
>Before executing any API from Postman, make sure all required Environment variables must be added.

## Publish the Journey created by Playbook

There are 2 ways to publish the journey; you may choose any of them:

1. **Using the AJO UI** - click on the Journey link on `Bill Of Material Page`; this will redirect you to Journey page there you can click on **[!UICONTROL Publish]** button and Journey would be published.

   ![Journey Object](../assets/journey-object.png)

1. **Using the Postman API**

    1. Trigger **[!DNL Publish Journey]** request from **[!DNL Journey Publish]** > **[!DNL Queue journey publish job]**.
    1. Journey publish might take some time, so in order to check the status execute the Check Journey publish status API, until the `response.status` is `SUCCESS`, make sure to wait 10-15 seconds if journey publish takes time.

    >[!NOTE]
    >
    >All Environment variables are case sensitive so please always use the exact variable name.

## Ingest the Customer Profile

>[!TIP]
>
>You can reuse the same email address by appending `+<variable>` into your email e.g. `usertest@email.com` can be resued as `usertest+v1@email.com` or `usertest+24jul@email.com`. This would be helpful to have a fresh profile each time, but still using the same email id.

1. First-time user needs to create the **[!DNL customer dataset]** and **[!DNL HTTP Streaming Inlet Connection]**.
1. If you already have created the **[!DNL customer dataset]** and **[!DNL HTTP Streaming Inlet Connection]**, please skip to the step `5`.
1. Trigger **[!DNL Customer Profile Ingestion]** > **[!DNL Create Customer Profile InletId]** > **[!DNL Create Dataset]** to create **[!DNL customer dataset]**; this will store a `CustomerProfile_dataset_id` in postman environment variables.
1. Create **[!DNL HTTP Streaming Inlet Connection]**, use Postman APIs under **[!DNL Customer Profile Ingestion > Create Customer Profile InletId]**.

    1. `CustomerProfile_dataset_id` must be available in postman environment variables, if not, refer step `3`.
    1. Trigger **[!DNL `CREATE Base Connection`]** to [!DNL create base connection].
    1. Trigger **[!DNL `CREATE Source Connection`]** to [!DNL create source connection].
    1. Trigger **[!DNL `CREATE Target Connection`]** to [!DNL create target connection].
    1. Trigger **[!DNL `CREATE Dataflow`]** to [!DNL create dataflow].
    1. Trigger **[!DNL `GET Base Connection`]**- this will automatically store `CustomerProfile_inlet_id` in the postman environment variables.

1. At this step you must have `CustomerProfile_dataset_id` and `CustomerProfile_inlet_id` in postman environment variables; if not, please refer step `3` or `4` respectively.
1. To ingest customer, user need to store `customer_country_code`, `customer_mobile_no`, `customer_first_name`, `customer_last_name` and `email` in postman environment variables.

    1. `customer_country_code` would be the country code of mobile number e.g. `91` or `1`
    1. `customer_mobile_no` would be mobile number e.g. `9987654321`
    1. `customer_first_name` would be the first name of user
    1. `customer_last_name` would be the last name of user
    1. `email` would be the email address of user, this is crucial to use distinct email id so that a fresh profile can be ingested.

1. Update the Postman request **[!DNL Customer Ingestion]** > **[!DNL Customer Streaming Ingestion]** to change the preferred channel of customer; by default [!DNL `email`] is configured in request.

   ```js
   "consents": {
       "marketing": {
           "preferred": "email",
           "email": {
               "val": "y"
           },
           "push": {
               "val": "n"
           },
           "sms": {
               "val": "n"
           }
       }
   }
   ```

1. Change preferred channel to `sms` or `push` and make respective channel value to `y` and `n` to other values, e.g.

   ```js
   "consents": {
       "marketing": {
           "preferred": "sms",
           "email": {
               "val": "n"
           },
           "push": {
               "val": "n"
           },
           "sms": {
               "val": "y"
           }
       }
   }
   ```

1. Finally Trigger **[!DNL `Customer Profile Ingestion > Customer Profile Streaming Ingestion`]** to ingest the customer profile.

## Ingest Event

1. First time user need to create the **[!DNL event dataset]** and **[!DNL HTTP Streaming Inlet Connection for events]**
1. If you already have created the **[!DNL event dataset]** and **[!DNL HTTP Streaming Inlet Connection for events]**, please skip to the step `5`.
1. Trigger **[!DNL `Schemas Data Ingestion > AEP Demo Schema Ingestion > Create AEP Demo Schema InletId > Create Dataset`]** to create **[!DNL event dataset]**, this will store a `AEPDemoSchema_dataset_id` in postman environment variables
1. Create **[!DNL HTTP Streaming Inlet Connection for events]**, use Postman APIs under **[!DNL Schemas Data Ingestion]** > **[!DNL AEP Demo Schema Ingestion]** > **[!DNL Create AEP Demo Schema InletId]**.

    1. `AEPDemoSchema_dataset_id` must be available in postman environment variables, if not, refer step `3`
    1. Trigger **[!DNL `CREATE Base Connection`]** to [!DNL create base connection]
    1. Trigger **[!DNL `CREATE Source Connection`]** to [!DNL create source connection]
    1. Trigger **[!DNL `CREATE Target Connection`]** to [!DNL create target connection]
    1. Trigger **[!DNL `CREATE Dataflow`]** to [!DNL create dataflow]
    1. Trigger **[!DNL `GET Base Connection`]**- this will automatically store `AEPDemoSchema_inlet_id` in the postman environment variables

1. At this step you must have `AEPDemoSchema_dataset_id` and `AEPDemoSchema_inlet_id` in postman environment variables, if not, please refer step `3` or `4` respectively
1. To ingest event, user need to change the time variable `timestamp` in request body of **[!DNL Schemas Data Ingestion]** > **[!DNL AEP Demo Schema Ingestion]** > **[!DNL AEP Demo Schema Streaming Ingestion]** in postman.

    1. `timestamp` would the time of event occurance, use the currrent timestamp e.g. `2023-07-21T16:37:52+05:30` adjust the time zone as per your need.

1. Trigger **[!DNL Schemas Data Ingestion > AEP Demo Schema Ingestion > AEP Demo Schema Streaming Ingestion]** to ingest the event, so that journey can be triggered

## Final Validation

You must receive a message on your selected preferred channel used in **[!DNL Ingest the Customer Profile]** step `8`

* `SMS` if preferred channel is `sms` on `customer_country_code` and `customer_mobile_no`
* `Email` if preferred channel is `email` on `email`

Alternatively you can check `Journey Report`, to check it click on `Journey Object` on `Bill of Materials page` this will redirect you to `Journey Details page`.

For any published Journey user must get a **[!UICONTROL View report]** button
![Journey Report Page](../assets/journey-report-page.png)


## Clean up

Please do not have the multiple instances of `Journey` running simultaneously, kindly stop the Journey if it is only for validation once the validation is completed.
