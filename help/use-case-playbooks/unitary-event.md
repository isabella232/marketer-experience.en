---
title: Unitary event
description: This is an intruction page for simulating the '[!UICONTROL Unitary event]' type of Journey validation.
---

# Unitary event

## Steps to follow {#steps-to-follow}

>[!CONTEXTUALHELP]
>id="marketerexp_sampledata_unitaryevent"
>title="How to use?"
>abstract="Please follow the link for more details"

>[!IMPORTANT]
>
>These instructions might change across **[!UICONTROL Playbook]** so please always refer the Sample data section of respective **[!UICONTROL Playbook]**.

## Prerequisite

* Use Playbook to create the instances assets like **[!UICONTROL Journey]**, **[!UICONTROL Schemas]**, **[!UICONTROL Segments]**, **[!UICONTROL Messages]** etc.

Created assets will be shown on `Bill Of Material` Page

<!-- TODO: attached image needs to change once postman is removed from UI -->
![Bill Of Material Page](../assets/bom-page.png)

## Prepare your terminal with required variables

1. Visit **[!UICONTROL Use Case Playbook]** application.
1. Click on the respective **[!UICONTROL Playbook]** card to visit **[!UICONTROL Playbook]** details page.
1. Visit **[!UICONTROL Bill Of Material]** page and find the **[!UICONTROL Sample data]** section.
1. Download the `variables.txt` by clicking the respective button on the UI.
1. Export all the variables with values in your terminal window.

## Fetch IMS Token

>[!NOTE]
>
>All Environment variables are case sensitive so please always use the exact variable name.

1. Please follow [Authenticate and access Experience Platform APIs](https://experienceleague.adobe.com/docs/experience-platform/landing/platform-apis/api-authentication.html) documentation to generate the Access Token.
1. Set the Access Token value in a variables named `ACCESS_TOKEN` in current shell.

>[!IMPORTANT]
>
>Before executing any cURLs, make sure all required environment variables are set.

## Publish the Journey created by Playbook

There are 2 ways to publish the journey; you may choose any of them:

1. **Using the AJO UI** - click on the Journey link on `Bill Of Material Page`; this will redirect you to Journey page there you can click on **[!UICONTROL Publish]** button and Journey would be published.

   ![Journey Object](../assets/journey-object.png)

1. **Using cURL**

    1. Publish the journey. The response will contain job id needed in next step to fetch journey publish status.

        ``` bash
        curl --location --request POST 'https://journey-private.adobe.io/authoring/jobs/journeyVersions/$JOURNEY_ID/deploy' \
        --header "Accept: */*" \
        --header "Authorization: Bearer $ACCESS_TOKEN" \
        --header "Content-Type: application/json" \
        --header "x-api-key: voyager_ui" \
        --header "x-gw-ims-org-id: $ORG_ID" \
        --header "x-sandbox-name: $SANDBOX_NAME"
        ```

    <!-- TODO: How to get and set JOB_ID? -->
    1. Journey publish might take some time, so in order to check the status execute below cURL, until the `response.status` is `SUCCESS`, make sure to wait 10-15 seconds if journey publish takes time.

        ```bash
        curl --location 'https://journey-private.adobe.io/authoring/jobs/$JOB_ID' \
        --header 'Authorization: Bearer $ACCESS_TOKEN' \
        --header 'Content-Type: application/json' \
        --header 'x-api-key: voyager_ui' \
        --header 'x-gw-ims-org-id: $ORG_ID' \
        --header 'x-sandbox-name: $SANDBOX_NAME'
        ```

    >[!NOTE]
    >
    >All Environment variables are case sensitive so please always use the exact variable name.

## Ingest the Customer Profile

>[!TIP]
>
>If your email provider supports plus emails, you can reuse the same email address by appending `+<variable>` into your email e.g. `usertest@email.com` can be resued as `usertest+v1@email.com` or `usertest+24jul@email.com`. This would be helpful to have a fresh profile each time, but still using the same email id.
>
>P.S: Plus emails are a configurable feature which needs to be supported by email provider. Please check if you are able to receive emails on such addresses before using them in test.

1. First-time user needs to create the **[!DNL customer dataset]** and **[!DNL HTTP Streaming Inlet Connection]**.
1. If you already have created the **[!DNL customer dataset]** and **[!DNL HTTP Streaming Inlet Connection]**, please skip to the step `5`.
1. Create a customer profile dataset by executing the below cURL.

    ```bash
    curl --location 'https://platform.adobe.io/data/foundation/catalog/dataSet' \
    --header "Authorization: Bearer $ACCESS_TOKEN" \
    --header "x-gw-ims-org-id: $ORG_ID" \
    --header "x-sandbox-name: $SANDBOX_NAME" \
    --header "x-api-key: platform_exim" \
    --header "Content-Type: application/json" \
    --data '{
        "name": "Customer_Profile",
        "schemaRef": {
            "id": "'$SCHEMA_REF'",
            "contentType": "application/vnd.adobe.xed-full-notext+json; version=1"
        },
        "tags": {
            "unifiedProfile": [
            "enabled:true"
            ],
            "unifiedIdentity": [
            "enabled:true"
            ]
        },
        "fileDescription": {
            "persisted": true,
            "containerFormat": "parquet",
            "format": "parquet"
        }
    }'
    ```

    The response will be of the format `"@/dataSets/<dataset_id>"`. 
    Set the variable in the terminal window as `PROFILE_DATASET_ID=<dataset_id>` using the `dataset_id` from the response. This will be used in later cURLs.

1. Create **[!DNL HTTP Streaming Inlet Connection]** with help of following steps.
    1. Create a base connection.

        ```bash
        curl --location 'https://platform.adobe.io/data/foundation/flowservice/connections?Cache-Control=no-cache' \
        --header "Authorization: Bearer $ACCESS_TOKEN" \
        --header "x-gw-ims-org-id: $ORG_ID" \
        --header "x-sandbox-name: $SANDBOX_NAME" \
        --header "Content-Type: application/json" \
        --header "x-api-key: platform_exim" \
        --data '{
            "name": "AbandonedCartProduct_Base_ConnectionForCustomerProfile_1694458293",
            "description": "Marketer Playground Playbook-Validation Customer Profile Base Connection 1",
            "auth": {
                "specName": "Streaming Connection",
                "params": {
                    "dataType": "xdm"
                }
            },
            "connectionSpec": {
                "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
                "version": "1.0"
            }
        }'
        ```

        <!-- TODO: Provide instructions to set base connection id to a variable -->
    1. Create source connection. Use the id from the result of step 1 to get base connection id.

        ```bash
        curl --location 'https://platform.adobe.io/data/foundation/flowservice/sourceConnections' \
        --header "Authorization: Bearer $ACCESS_TOKEN" \
        --header "x-gw-ims-org-id: $ORG_ID" \
        --header "x-sandbox-name: $SANDBOX_NAME" \
        --header "Content-Type: application/json" \
        --header "x-api-key: platform_exim" \
        --data '{
            "name": "AbandonedCartProduct_Source_ConnectionForCustomerProfile_1694458318",
            "description": "Marketer Playground Playbook-Validation Customer Profile Source Connection 1",
            "baseConnectionId": $BASE_CONNECTION_ID,
            "connectionSpec": {
                "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
                "version": "1.0"
            }
        }'
        ```

    1. Create target connection.

        ```bash
        curl --location 'https://platform.adobe.io/data/foundation/flowservice/targetConnections' \
        --header "Authorization: Bearer $ACCESS_TOKEN" \
        --header "x-gw-ims-org-id: $ORG_ID" \
        --header "x-sandbox-name: $SANDBOX_NAME" \
        --header "Content-Type: application/json" \
        --header "x-api-key: platform_exim" \
        --data '{
            "name": "AbandonedCartProduct_Target_ConnectionForCustomerProfile_1694458407",
            "description": "Marketer Playground Playbook-Validation Customer Profile Target Connection 1",
            "data": {
                "format": "parquet_xdm",
                "schema": {
                    "version": "application/vnd.adobe.xed-full+json;version=1",
                    "id": "'$SCHEMA_REF'"
                },
                "properties": null
            },
            "connectionSpec": {
                "id": "c604ff05-7f1a-43c0-8e18-33bf874cb11c",
                "version": "1.0"
            },
            "params": {
                "dataSetId": "'$PROFILE_DATASET_ID'"
            }
        }
        ```

        <!-- TODO : Might need to convert below connection ids to variables" -->
    1. Create a dataflow.

        ```bash
        curl --location 'https://platform.adobe.io/data/foundation/flowservice/flows' \
        --header "Authorization: Bearer $ACCESS_TOKEN" \
        --header "x-gw-ims-org-id: $ORG_ID" \
        --header "x-sandbox-name: $SANDBOX_NAME" \
        --header "Content-Type: application/json" \
        --header "x-api-key: platform_exim" \
        --data '{
            "name": "AbandonedCartProduct_Dataflow_ForCustomerCustomerProfile_1694460528",
            "description": "Marketer Playground Playbook-Validation Customer Profile Dataflow 1",
            "flowSpec": {
                "id": "d8a6f005-7eaf-4153-983e-e8574508b877",
                "version": "1.0"
            },
            "sourceConnectionIds": [
                "69c08c38-c756-44e5-9a14-d6e042944bbd"
            ],
            "targetConnectionIds": [
                "1a1dca2f-a603-4741-8911-3fa170536a58"
            ]
        }'
        ```

    1. Get Base Connection. The result will contain inlet id URL required to send profile data.

        ```bash
        curl --location 'https://platform.adobe.io/data/foundation/flowservice/connections/e6527218-042b-4c45-98cd-753db9f4a7e3' \
        --header "Authorization: Bearer $ACCESS_TOKEN" \
        --header "x-gw-ims-org-id: $ORG_ID" \
        --header "x-sandbox-name: $SANDBOX_NAME" \
        --header "Content-Type: application/json" \
        --header "x-api-key: platform_exim"
        ```

1. At this step you must have `CustomerProfile_dataset_id` in variable `PROFILE_DATASET_ID` and `CustomerProfile_inlet_id` in variable `PROFILE_INLET_ID`; if not, please refer step `3` or `4` respectively.
1. To ingest customer, user need to store `customer_country_code`, `customer_mobile_no`, `customer_first_name`, `customer_last_name` and `email` in terminal window shell variables.

    1. `CUSTOMER_MOBILE_NUMBER` would be mobile number e.g. `+1 000-000-0000`
    1. `CUSTOMER_FIRST_NAME` would be the first name of user
    1. `CUSTOMER_LAST_NAME` would be the last name of user
    1. `EMAIL` would be the email address of user, this is crucial to use distinct email id so that a fresh profile can be ingested.

1. Finally execute the curl to ingest customer profile. Update `body.xdmEntity.consents.marketing.preferred` to `email`, `sms`, or `push` based on the channels you intend to verify. Also set corresponding `val` to `y`.

    ```bash
    curl --location 'https://dcs.adobedc.net/collection/ee492764395a581d74ffda72e27736b8a8b3e58d8b86d48fe174111e2bd0e668?synchronousValidation=true' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "header": {
            "schemaRef": {
                "id": "'$SCHEMA_REF'",
                "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
            },
            "imsOrgId": "'$ORG_ID'",
            "datasetId": "'$PROFILE_DATASET_ID'",
            "source": {
                "name": "Streaming dataflow - 1694460605"
            }
        },
        "body": {
            "xdmMeta": {
                "schemaRef": {
                    "id": "'$SCHEMA_REF'",
                    "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
                }
            },
            "xdmEntity": {
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
            },
            "mobilePhone": {
                "number": "'$CUSTOMER_MOBILE_NUMBER'",
                "status": "active"
            },
            "person": {
                "name": {
                "firstName": "'$CUSTOMER_FIRST_NAME'",
                "lastName": "'$CUSTOMER_LAST_NAME'"
                }
            },
            "personalEmail": {
                "address": "'$EMAIL'"
            },
            "testProfile": false
            }
        }
    }'
    ```

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
