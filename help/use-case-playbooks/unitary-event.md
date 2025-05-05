---
title: 単一イベント
description: こちらは、ジャーニー検証の「[!UICONTROL 単一イベント]」タイプをシミュレートするための説明ページです。
exl-id: 314f967c-e10f-4832-bdba-901424dc2eeb
source-git-commit: 194667c26ed002be166ab91cc778594dc1f09238
workflow-type: tm+mt
source-wordcount: '839'
ht-degree: 100%

---

# 単一イベント

## 従うべき手順 {#steps-to-follow}

>[!CONTEXTUALHELP]
>id="marketerexp_sampledata_unitaryevent"
>title="使用方法"
>abstract="詳しくは、リンクを参照してください"

>[!IMPORTANT]
>
>以下の手順は&#x200B;**[!UICONTROL プレイブック]**&#x200B;ごとに異なる場合があるので、常に各&#x200B;**[!UICONTROL プレイブック]**&#x200B;のサンプルデータセクションを参照してください。

## 前提条件

* プレイブックを使用して、**[!UICONTROL ジャーニー]**、**[!UICONTROL スキーマ]**、**[!UICONTROL セグメント]**、**[!UICONTROL メッセージ]**&#x200B;などのインスタンスアセットを作成します。

* 作成したアセットは、`Bill Of Material` ページに表示されます

<!-- TODO: attached image needs to change once postman is removed from UI -->
![部品表ページ](../assets/bom-page.png)

>[!TIP]
>
>ターミナルを使用して cURL を実行する場合は、cURL を実行する前に変数値を設定できるので、個々の cURL でこれらの値を置き換える必要はありません。
>例えば、`ORG_ID=************@AdobeOrg` を設定すると、シェルはすべての `$ORG_ID` をその値に自動的に置き換えるので、以下の cURL を変更せずにコピー、ペーストおよび実行できます。
>
> このドキュメント全体で次の変数が使用されます
>
> ACCESS_TOKEN
>
> API_KEY
>
> ORG_ID
>
> SANDBOX_NAME
>
> PROFILE_SCHEMA_REF
>
> PROFILE_DATASET_NAME
>
> PROFILE_DATASET_ID
>
> JOURNEY_ID
>
> PROFILE_BASE_CONNECTION_ID
>
> PROFILE_SOURCE_CONNECTION_ID
>
> PROFILE_TARGET_CONNECTION_ID
>
> PROFILE_INLET_URL
>
> CUSTOMER_MOBILE_NUMBER
>
> CUSTOMER_FIRST_NAME
>
> CUSTOMER_LAST_NAME
>
> EMAIL
>
> EVENT_SCHEMA_REF
>
> EVENT_DATASET_NAME
>
> EVENT_DATASET_ID
>
> EVENT_BASE_CONNECTION_ID
>
> EVENT_SOURCE_CONNECTION_ID
>
> EVENT_TARGET_CONNECTION_ID
>
> EVENT_INLET_URL
>
> TIMESTAMP
>
> UNIQUE_EVENT_ID

## IMS トークンの取得

1. [Experience Platform API の認証とアクセス](https://experienceleague.adobe.com/docs/experience-platform/landing/platform-apis/api-authentication.html?lang=ja)ドキュメントに従ってアクセストークンを生成してください。

## プレイブックで作成したジャーニーの公開

ジャーニーを公開するには 2 つの方法があります。いずれかを選択できます。

1. **AJO UIの使用** - `Bill Of Material Page` のジャーニーリンクをクリックします。これにより、ジャーニーページにリダイレクトされ、そこで「**[!UICONTROL 公開]**」ボタンをクリックすると、ジャーニーが公開されます。

   ![ジャーニーの目的](../assets/journey-object.png)

1. **cURL の使用**

   1. ジャーニーを公開します。応答には、次の手順でジャーニー公開ステータスを取得するために必要なジョブ ID が含まれます。

      ```bash
      curl --location --request POST "https://journey-private.adobe.io/authoring/jobs/journeyVersions/$JOURNEY_ID/deploy" \
      --header "Accept: */*" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-api-key: $API_KEY" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" 
      ```

   1. ジャーニーの公開には時間がかかる場合があるので、ステータスを確認するには、`response.status` が `SUCCESS` になるまで、cURL の下で実行します。ジャーニーの公開に時間がかかる場合は、10～15 秒待機する必要があります。

      ```bash
      curl --location "https://journey-private.adobe.io/authoring/jobs/$JOB_ID" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-api-key: $API_KEY" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json"
      ```

## 顧客プロファイルを取得

>[!TIP]
>
>メールプロバイダーがプラスメールをサポートしている場合は、メールに `+<variable>` を追加することで、同じメールアドレスを再利用できます。例えば、`usertest@email.com` は、`usertest+v1@email.com` または `usertest+24jul@email.com` として再利用できます。これは、同じメール ID を使用しながら、毎回新しいプロファイルを作成するのに役立ちます。
>
>P.S：プラスメールは設定可能な機能で、メールプロバイダーによってサポートされる必要があります。テストで使用する前に、このようなアドレスでメールを受信できるかどうかを確認してください。

1. 初めてのユーザーは、**[!DNL customer dataset]** と **[!DNL HTTP Streaming Inlet Connection]** を作成する必要があります。
1. 既に **[!DNL customer dataset]** と **[!DNL HTTP Streaming Inlet Connection]** を作成している場合は、手順 `5` に進んでください。
1. 以下の cURL を実行して、顧客プロファイルデータセットを作成します。

   ```bash
   curl --location "https://platform.adobe.io/data/foundation/catalog/dataSet" \
   --header "Authorization: Bearer $ACCESS_TOKEN" \
   --header "x-gw-ims-org-id: $ORG_ID" \
   --header "x-sandbox-name: $SANDBOX_NAME" \
   --header "x-api-key: $API_KEY" \
   --header "Content-Type: application/json" \
   --data '{
       "name": "'$PROFILE_DATASET_NAME'",
       "schemaRef": {
           "id": "'$PROFILE_SCHEMA_REF'",
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

   応答の形式は、`"@/dataSets/<PROFILE_DATASET_ID>"` です。

1. 次の手順に従って **[!DNL HTTP Streaming Inlet Connection]** を作成します。
   1. ベース接続を作成します。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/connections?Cache-Control=no-cache" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
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

      応答からベース接続 ID を取得し、次の cURL で `PROFILE_BASE_CONNECTION_ID` の代わりに使用します

   1. ソース接続を作成します。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/sourceConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY" \
      --data '{
          "name": "AbandonedCartProduct_Source_ConnectionForCustomerProfile_1694458318",
          "description": "Marketer Playground Playbook-Validation Customer Profile Source Connection 1",
          "baseConnectionId": "'$PROFILE_BASE_CONNECTION_ID'",
          "connectionSpec": {
              "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
              "version": "1.0"
          }
      }'
      ```

      応答からソース接続 ID を取得し、`PROFILE_SOURCE_CONNECTION_ID` の代わりに使用します

   1. ターゲット接続を作成します。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/targetConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY" \
      --data '{
          "name": "AbandonedCartProduct_Target_ConnectionForCustomerProfile_1694458407",
          "description": "Marketer Playground Playbook-Validation Customer Profile Target Connection 1",
          "data": {
              "format": "parquet_xdm",
              "schema": {
                  "version": "application/vnd.adobe.xed-full+json;version=1",
                  "id": "'$PROFILE_SCHEMA_REF'"
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
      }'
      ```

      応答からターゲット接続 ID を取得し、`PROFILE_TARGET_CONNECTION_ID` の代わりに使用します

   1. データフローを作成します。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/flows" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY" \
      --data '{
          "name": "AbandonedCartProduct_Dataflow_ForCustomerCustomerProfile_1694460528",
          "description": "Marketer Playground Playbook-Validation Customer Profile Dataflow 1",
          "flowSpec": {
              "id": "d8a6f005-7eaf-4153-983e-e8574508b877",
              "version": "1.0"
          },
          "sourceConnectionIds": [
              "'$PROFILE_SOURCE_CONNECTION_ID'"
          ],
          "targetConnectionIds": [
              "'$PROFILE_TARGET_CONNECTION_ID'"
          ]
      }'
      ```

   1. ベース接続を取得します。結果には、プロファイルデータの送信に必要な inletUrl が含まれます。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/connections/$PROFILE_BASE_CONNECTION_ID" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "Content-Type: application/json" \
      --header "x-api-key: $API_KEY"
      ```

      応答から inletUrl を取得し、`PROFILE_INLET_URL` の代わりに使用します

1. この手順では、ユーザーは `PROFILE_DATASET_ID` と `PROFILE_INLET_URL` の値を設定する必要があります。設定しない場合は、手順 `3` または `4` をそれぞれ参照してください。
1. 顧客を取得するには、以下の cURL の `CUSTOMER_MOBILE_NUMBER`、`CUSTOMER_FIRST_NAME`、`CUSTOMER_LAST_NAME` および `EMAIL` を置き換える必要があります。

   1. `CUSTOMER_MOBILE_NUMBER` は、携帯電話番号（`+1 000-000-0000` など）になります
   1. `CUSTOMER_FIRST_NAME` は、ユーザーの名になります
   1. `CUSTOMER_LAST_NAME` は、ユーザーの姓になります
   1. `EMAIL` は、ユーザーのメールアドレスになります。これは、新しいプロファイルを取得できるように個別のメール ID を使用するために重要です。

1. 最後に、cURL を実行して、顧客プロファイルを取得します。検証するチャネルに基づいて、`body.xdmEntity.consents.marketing.preferred` を `email`、`sms` または `push` に更新します。また、対応する `val` を `y` に設定します。

   ```bash
   curl --location "$PROFILE_INLET_URL?synchronousValidation=true" \
   --header 'Content-Type: application/json' \
   --data-raw '{
       "header": {
           "schemaRef": {
               "id": "'$PROFILE_SCHEMA_REF'",
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
                   "id": "'$PROFILE_SCHEMA_REF'",
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

## ジャーニートリガーイベントの取得

1. 初めてのユーザーは、**[!DNL event dataset]** と **[!DNL HTTP Streaming Inlet Connection for events]** を作成する必要があります
1. 既に **[!DNL event dataset]** と **[!DNL HTTP Streaming Inlet Connection for events]** を作成している場合は、手順 `5` に進んでください。
1. 以下の cURL を実行して、イベントデータセットを作成します。

   ```bash
   curl --location "https://platform.adobe.io/data/foundation/catalog/dataSet" \
   --header "Authorization: Bearer $ACCESS_TOKEN" \
   --header "x-gw-ims-org-id: $ORG_ID" \
   --header "x-sandbox-name: $SANDBOX_NAME" \
   --header "x-api-key: $API_KEY" \
   --header "Content-Type: application/json" \
   --data '{
       "name": "'$EVENT_DATASET_NAME'",
       "schemaRef": {
           "id": "'$EVENT_SCHEMA_REF'",
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

   応答の形式は、`"@/dataSets/<EVENT_DATASET_ID>"` です。

1. 次の手順に従って **[!DNL HTTP Streaming Inlet Connection for events]** を作成します。
   <!-- TODO: Is the name unique? If so, we may need to generate and provide in variables.txt-->
   1. ベース接続を作成します。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/connections?Cache-Control=no-cache" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Base_ConnectionForAEPDemoSchema_1694461448",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Base Connection 1",
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

      応答からベース接続 ID を取得し、`EVENT_BASE_CONNECTION_ID` の代わりに使用します

   1. ソース接続を作成します。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/sourceConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Source_ConnectionForAEPDemoSchema_1694461464",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Source Connection 1",
          "baseConnectionId": "'$EVENT_BASE_CONNECTION_ID'",
          "connectionSpec": {
              "id": "bc7b00d6-623a-4dfc-9fdb-f1240aeadaeb",
              "version": "1.0"
          }
      }'
      ```

      応答からソース接続 ID を取得し、`EVENT_SOURCE_CONNECTION_ID` の代わりに使用します

   1. ターゲット接続を作成します。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/sourceConnections" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Target_ConnectionForAEPDemoSchema_1694802667",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Target Connection 1",
          "data": {
              "format": "parquet_xdm",
              "schema": {
                  "version": "application/vnd.adobe.xed-full+json;version=1",
                  "id": "'$EVENT_SCHEMA_REF'"
              },
              "properties": null
          },
          "connectionSpec": {
              "id": "c604ff05-7f1a-43c0-8e18-33bf874cb11c",
              "version": "1.0"
          },
          "params": {
              "dataSetId": "'$EVENT_DATASET_ID'"
          }
      }'
      ```

      応答からターゲット接続 ID を取得し、`EVENT_TARGET_CONNECTION_ID` の代わりに使用します

   1. データフローを作成します。

      ```bash
      curl --location "https://platform.adobe.io/data/foundation/flowservice/flows" \
      --header "Authorization: Bearer $ACCESS_TOKEN" \
      --header "x-gw-ims-org-id: $ORG_ID" \
      --header "x-sandbox-name: $SANDBOX_NAME" \
      --header "x-api-key: $API_KEY" \
      --header "Content-Type: application/json" \
      --data '{
          "name": "AbandonedCartProduct_Dataflow_ForCustomerAEPDemoSchema_1694461564",
          "description": "Marketer Playground Playbook-Validation AEP Demo Schema Dataflow 1",
          "flowSpec": {
              "id": "d8a6f005-7eaf-4153-983e-e8574508b877",
              "version": "1.0"
          },
          "sourceConnectionIds": [
              "'$EVENT_SOURCE_CONNECTION_ID'"
          ],
          "targetConnectionIds": [
              "'$EVENT_TARGET_CONNECTION_ID'"
          ]
      }'
      ```

   1. ベース接続を取得します。結果には、プロファイルデータの送信に必要な inletUrl が含まれます。

   ```bash
   curl --location "https://platform.adobe.io/data/foundation/flowservice/connections/$EVENT_BASE_CONNECTION_ID" \
       --header "Authorization: Bearer $ACCESS_TOKEN" \
       --header "x-gw-ims-org-id: $ORG_ID" \
       --header "x-sandbox-name: $SANDBOX_NAME" \
       --header "x-api-key: $API_KEY" \
       --header "Content-Type: application/json" 
   ```

   応答から inletUrl を取得し、`EVENT_INLET_URL` の代わりに使用します

1. この手順では、ユーザーは `EVENT_DATASET_ID` と `EVENT_INLET_URL` の値を設定する必要があります。設定しない場合は、手順 `3` または `4` をそれぞれ参照してください。
1. イベントを取得するには、ユーザーは以下の cURL のリクエスト本文にある時間変数 `TIMESTAMP` を変更する必要があります。

   1. `body.xdmEntity` をダウンロードしたイベント JSON の内容に置き換えます。
   1. `TIMESTAMP` では、イベントの発生時刻に UTC タイムゾーンのタイムスタンプ（例：`2023-09-05T23:57:00.071+00:00`）を使用します。
   1. 変数 `UNIQUE_EVENT_ID` に一意の値を設定します。

   ```bash
   curl --location "$EVENT_INLET_URL?synchronousValidation=true" \
   --header 'Content-Type: application/json' \
   --data-raw '{
       "header": {
           "schemaRef": {
               "id": "'$EVENT_SCHEMA_REF'",
               "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
           },
           "imsOrgId": "'$ORG_ID'",
           "datasetId": "'$EVENT_DATASET_ID'",
           "source": {
               "name": "Streaming dataflow - 8/31/2023 9:04:25 PM"
           }
       },
       "body": {
           "xdmMeta": {
               "schemaRef": {
                   "id": "'$EVENT_SCHEMA_REF'",
                   "contentType": "application/vnd.adobe.xed-full+json;version=1.0"
               }
           },
           "xdmEntity": {
               "endUserIDs": {
                   "_experience": {
                       "aaid": {
                           "id": "'$EMAIL'"
                       },
                       "emailid": {
                           "id": "'$EMAIL'"
                       }
                   }
               },
               "_experience": {
                   "analytics": {
                       "customDimensions": {
                           "eVars": {
                           "eVar235": "AC11147"
                           }
                       }
                   }
               },
               "_id": "'$UNIQUE_EVENT_ID'",
               "commerce": {
                   "productListAdds": {
                       "value": 11498
                   }
               },
               "eventType": "commerce.productListAdds",
               "productListItems": [
                   {
                       "_id": "ACS1620",
                       "SKU": "P1",
                       "_experience": {
                           "analytics": {
                           "customDimensions": {
                               "eVars": {
                                   "eVar1": "Pants"
                               }
                           }
                           }
                       },
                       "currencyCode": "USD",
                       "name": "Sample value",
                       "priceTotal": 30841.13,
                       "product": "https://ns.adobe.com/xdm/common/uri",
                       "productAddMethod": "Sample value",
                       "quantity": 1
                   },
                   {
                       "_id": "ACS1729",
                       "SKU": "P2",
                       "_experience": {
                           "analytics": {
                               "customDimensions": {
                                   "eVars": {
                                       "eVar1": "Galliano"
                                   }
                               }
                           }
                       },
                       "currencyCode": "USD",
                       "name": "Sample value",
                       "priceTotal": 20841.13,
                       "product": "https://ns.adobe.com/xdm/common/uri",
                       "productAddMethod": "Sample value",
                       "quantity": 2
                   }
               ],
               "timestamp": "'$TIMESTAMP'",
               "web": {
                   "webInteraction": {
                       "URL": "https://experienceleague.adobe.com/docs/experience-platform/edge/data-collection/collect-commerce-data.html?lang=ja",
                       "name": "Sample value",
                       "region": "Sample value"
                   },
                   "webPageDetails": {
                       "URL": "https://experienceleague.adobe.com/docs/experience-platform/edge/data-collection/collect-commerce-data.html?lang=ja",
                       "isErrorPage": false,
                       "isHomePage": false,
                       "name": "Sample value",
                       "pageViews": {
                           "id": "Sample value",
                           "value": 1
                       },
                       "server": "Sample value",
                       "siteSection": "Sample value",
                       "viewName": "Sample value"
                   },
                   "webReferrer": {
                   "URL": "Sample value",
                   "type": "internal"
                   }
               }
           }
       }
   }'
   ```

## 最終検証

**[!DNL Ingest the Customer Profile]** の手順 `8` で使用する選択した優先チャネルでメッセージを受信する必要があります

* `customer_country_code` と `customer_mobile_no` で優先チャネルが `sms` の場合は、`SMS`
* `email` で優先チャネルが `email` の場合は、`Email`

または、`Journey Report` をオンにすることもできます。オンにするには、`Bill of Materials page` で `Journey Object` をクリックします。これにより、`Journey Details page` にリダイレクトされます。

公開されたジャーニーについては、ユーザーは「**[!UICONTROL レポートを表示]**」ボタンを取得する必要があります
![ジャーニーレポートページ](../assets/journey-report-page.png)


## クリーンアップ

`Journey` の複数のインスタンスを同時に実行しないでください。検証のみを目的とする場合は、検証が完了したらジャーニーを停止してください。
