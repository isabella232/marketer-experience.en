---
title: Unitary event
description: This is an intruction page for simulating the `Unitary event` type of Journey validation.
---

# Steps to follow

>[!IMPORTANT]
>
>These instructions might change across **[!UICONTROL Playbook]** so please always refer the Sample data section of respective **[!UICONTROL Playbook]** 

## Prerequisite

* You must have the Postman software installed
* Use Playbook to create the instances assets like **[!UICONTROL Journey]**, **[!UICONTROL Schemas]**, **[!UICONTROL Segments]**, **[!UICONTROL Messages]** etc.\
Created assets will be shown on `Bill Of Material` Page
![Bill Of Material Page](../assets/bom-page.png)


## Prepare Postman with required collection

1. Visit **[!UICONTROL Use Case Playbook]** application
2. Click on the respective **[!UICONTROL Playbook]** card to visit **[!UICONTROL Playbook]** details page
3. Visit **[!UICONTROL Bill Of Material]** page and find the **[!UICONTROL Sample data]** section
4. Download the `postman.json` by clicking the respective buttons on the UI
5. Import `postman.json` in the **[!DNL Postman Software]**
6. Create a dedicated Postman Environment for this validation e.g. `Adobe <PLAYBOOK_NAME>`

## Fetch IMS Token

>[!NOTE]
>
>All Environment variables are case sensitive so please always use the exact variable name.

1. Please follow [Authenticate and access Experience Platform APIs](https://experienceleague.adobe.com/docs/experience-platform/landing/platform-apis/api-authentication.html) documentation to generate the Access Token.
2. Store the Access Token value in Environment variables named `ACCESS_TOKEN`.
3. Store other authentication related values like `API_KEY`, `IMS_ORG` and `SANDBOX_NAME` in Environment variables

>[!IMPORTANT]
>
>Before executing any API from Postman, make sure all required Environment variables must be added

## Publish the Journey created by Playbook

There are 2 ways to publish the journey, you may opt any of them:
1. **Using the AJO UI** - click on the Journey link on `Bill Of Material Page`, this will redirect you to Journey page there you can click on **[!UICONTROL Publish]** button and Journey would be published.
![Journey Object](../assets/journey-object.png)
2. **Using the Postman API**
    1. Store the `JOURNEY_VERSION_ID` in Environment variables
    2. Trigger **[!DNL Publish Journey]** request from **[!DNL `Journey Publish > Queue journey publish job`]**
    3. Journey publish might take some time, so in order to check the status execute the Check Journey publish status API, until the `resonse.status` is `SUCCESS`, make sure to wait 10-15 seconds if journey publish takes time.
    >[!NOTE]
    >
    >All Environment variables are case sensitive so please always use the exact variable name.

## Ingest the Customer Profile

>[!TIP]
>
>You can reuse the same email address by appending `+<variable>` into your email e.g. `usertest@email.com` can be resued as `usertest+v1@email.com` or `usertest+24jul@email.com`. This would be helpful to have a fresh profile each time, but still using the same email id.

1. First time user need to create the **[!DNL customer dataset]** and **[!DNL HTTP Streaming Inlet Connection]**
2. If you already have created the **[!DNL customer dataset]** and **[!DNL HTTP Streaming Inlet Connection]**, please skip to the step `5`.
3. Trigger **[!DNL `Customer Ingestion > Create Customer InletId > Create Dataset`]** to create **[!DNL customer dataset]**, this will store a `customer_dataset_id` in postman environment variables
4. Create **[!DNL HTTP Streaming Inlet Connection]**, use Postman APIs under **[!DNL Customer Ingestion > Create Customer InletId]**
    1. `customer_dataset_id` must be available in postman environment variables, if not, refer step `3`
    2. Trigger **[!DNL `CREATE Base Connection`]** to [!DNL create base connection]
    3. Trigger **[!DNL `CREATE Source Connection`]** to [!DNL create source connection]
    4. Trigger **[!DNL `CREATE Target Connection`]** to [!DNL create target connection]
    5. Trigger **[!DNL `CREATE Dataflow`]** to [!DNL create dataflow]
    6. Trigger **[!DNL `GET Base Connection`]**- this will automatically store `customer_inlet_id` in the postman environment variables
5. At this step you must have `customer_dataset_id` and `customer_inlet_id` in postman environment variables, if not, please refer step `3` or `4` respectively
6. To ingest customer, user need to store `customer_country_code`, `customer_mobile_no`, `customer_firstname`, `customer_lastname` and `customer_email_id` in postman environment variables.
    1. `customer_country_code` would be the country code of mobile number e.g. `91` or `1`
    2. `customer_mobile_no` would be mobile number e.g. `9987654321`
    3. `customer_firstname` would be the first name of user
    4. `customer_lastname` would be the last name of user
    5. `customer_email_id` would be the email address of user, this is crucial to use distinct email id so that a fresh profile can be ingested.
7. Update the Postman request **[!DNL `Customer Ingestion > Customer Streaming Ingestion`]** to change the preferred channel of customer by default [!DNL `email`] is configured in request.

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

change preferred channel to `sms` or `push` and make respective channel value to `y` and `n` to other values, e.g.

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

8. Trigger **[!DNL `Customer Ingestion > Customer Streaming Ingestion`]** to ingest the customer profile

## Ingest Event

1. First time user need to create the **[!DNL event dataset]** and **[!DNL HTTP Streaming Inlet Connection for events]**
2. If you already have created the **[!DNL event dataset]** and **[!DNL HTTP Streaming Inlet Connection for events]**, please skip to the step `5`.
3. Trigger **[!DNL `Events Ingestion > Create Event InletId > Create Dataset`]** to create **[!DNL customer dataset]**, this will store a `evt_dataset_id` in postman environment variables
4. Create **[!DNL HTTP Streaming Inlet Connection for events]**, use Postman APIs under **[!DNL Events Ingestion > Create Event InletId]**
    1. `evt_dataset_id` must be available in postman environment variables, if not, refer step `3`
    2. Trigger **[!DNL `CREATE Base Connection`]** to [!DNL create base connection]
    3. Trigger **[!DNL `CREATE Source Connection`]** to [!DNL create source connection]
    4. Trigger **[!DNL `CREATE Target Connection`]** to [!DNL create target connection]
    5. Trigger **[!DNL `CREATE Dataflow`]** to [!DNL create dataflow]
    6. Trigger **[!DNL `GET Base Connection`]**- this will automatically store `evt_inlet_id` in the postman environment variables
5. At this step you must have `evt_dataset_id` and `evt_inlet_id` in postman environment variables, if not, please refer step `3` or `4` respectively
6. To ingest event, user need to store `event_timestamp` in postman environment variables.
    1. `event_timestamp` would the time of event occurance, use the currrent timestamp e.g. `2023-07-21T16:37:52+05:30`
7. Trigger **[!DNL `Events Ingestion > Insert Event into Adobe Analytics`]** to ingest the event, so that journey can be triggered

## Final Validation

You must receive a message on your selected preferred channel used in **[!DNL Ingest the Customer Profile]** step `8`
* `SMS` if preferred channel is `sms` on `customer_country_code` and `customer_mobile_no`
* `Email` if preferred channel is `email` on `customer_email_id`

Alternatively you can check `Journey Report`, to check it click on `Journey Object` on `Bill of Materials page` this will redirect you to `Journey Details page`.
For any published Journey user must get a **[!UICONTROL View report]** button
![Journey Report Page](../assets/journey-report-page.png)