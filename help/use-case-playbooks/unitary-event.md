---
title: 単一イベント
description: これは、「[!UICONTROL 単一イベント]「 」タイプのジャーニー検証。
source-git-commit: 0a52611530513fc036ef56999fde36a32ca0f482
workflow-type: tm+mt
source-wordcount: '749'
ht-degree: 0%

---


# 単一イベント

## 従う手順 {#steps-to-follow}

>[!CONTEXTUALHELP]
>id="marketerexp_sampledata_unitaryevent"
>title="使用方法"
>abstract="詳しくは、リンクを参照してください"

>[!IMPORTANT]
>
>これらの手順は、 **[!UICONTROL プレイブック]** そのため、必ず各 **[!UICONTROL プレイブック]**.

## 前提条件

* Postmanソフトウェアをインストールしておく必要があります
* プレイブックを使用して、次のようなインスタンスアセットを作成します。 **[!UICONTROL ジャーニー]**, **[!UICONTROL スキーマ]**, **[!UICONTROL セグメント]**, **[!UICONTROL メッセージ]** など

作成されたアセットはに表示されます `Bill Of Material` ページ

![「部品表」ページ](../assets/bom-page.png)

## 必要なコレクションでPostmanを準備

1. 訪問 **[!UICONTROL ユースケースプレイブック]** アプリケーション。
1. それぞれの **[!UICONTROL プレイブック]** 訪問するカード **[!UICONTROL プレイブック]** 詳細ページ。
1. 訪問 **[!UICONTROL 部品表]** ページで **[!UICONTROL サンプルデータ]** 」セクションに入力します。
1. をダウンロードします。 `postman.json` UI の各ボタンをクリックすることで、
1. インポート `postman.json` （内） **[!DNL Postman Software]**.
1. この検証用に、専用のPostman環境を作成します ( 例： `Adobe <PLAYBOOK_NAME>`) をクリックします。

## IMS トークンを取得

>[!NOTE]
>
>すべての環境変数では大文字と小文字が区別されるので、常に正確な変数名を使用してください。

1. 後に続いてください [Experience PlatformAPI の認証とアクセス](https://experienceleague.adobe.com/docs/experience-platform/landing/platform-apis/api-authentication.html) アクセストークンを生成するためのドキュメント。
1. 次の名前の環境変数にアクセストークンの値を保存 `ACCESS_TOKEN`.
1. その他の認証関連の値（例： ）を保存 `API_KEY`, `IMS_ORG` および `SANDBOX_NAME` （環境変数内）。

>[!IMPORTANT]
>
>Postmanから API を実行する前に、必要なすべての環境変数を追加する必要があります。

## プレイブックで作成したジャーニーを公開する

ジャーニーを公開する方法は 2 つあります。次のいずれかを選択できます。

1. **AJO UI の使用**  — のジャーニーリンクをクリック `Bill Of Material Page`をクリックすると、ジャーニーページにリダイレクトされます。 **[!UICONTROL 公開]** ボタンとジャーニーが公開されます。

   ![ジャーニーオブジェクト](../assets/journey-object.png)

1. **Postman API の使用**

   1. トリガー **[!DNL Publish Journey]** からのリクエスト **[!DNL Journey Publish]** > **[!DNL Queue journey publish job]**.
   1. ジャーニーの公開には時間がかかる場合があるので、ステータスを確認するには、「ジャーニーの公開ステータスを確認」API を実行し、 `response.status` 次に該当 `SUCCESS`に設定する場合は、必ず 10～15 秒待ってください（ジャーニーの公開に時間がかかる場合）。

   >[!NOTE]
   >
   >すべての環境変数では大文字と小文字が区別されるので、常に正確な変数名を使用してください。

## 顧客プロファイルの取り込み

>[!TIP]
>
>同じ E メールアドレスを再利用するには、以下を追加します。 `+<variable>` を電子メールに追加します（例： ）。 `usertest@email.com` ～として再利用できる `usertest+v1@email.com` または `usertest+24jul@email.com`. これは、毎回新しいプロファイルを持つが、同じ電子メール ID を使用している場合に役立ちます。

1. 初めてのユーザーは、 **[!DNL customer dataset]** および **[!DNL HTTP Streaming Inlet Connection]**.
1. 既に **[!DNL customer dataset]** および **[!DNL HTTP Streaming Inlet Connection]**、スキップして手順に進んでください `5`.
1. トリガー **[!DNL Customer Profile Ingestion]** > **[!DNL Create Customer Profile InletId]** > **[!DNL Create Dataset]** を作成します。 **[!DNL customer dataset]**；これは `CustomerProfile_dataset_id` postman 環境変数内。
1. 作成 **[!DNL HTTP Streaming Inlet Connection]**、でPostman API を使用する **[!DNL Customer Profile Ingestion > Create Customer Profile InletId]**.

   1. `CustomerProfile_dataset_id` は、postman 環境変数で使用できる必要があります（使用できない場合は、手順を参照してください）。 `3`.
   1. トリガー **[!DNL `CREATE Base Connection`]** から [!DNL create base connection].
   1. トリガー **[!DNL `CREATE Source Connection`]** から [!DNL create source connection].
   1. トリガー **[!DNL `CREATE Target Connection`]** から [!DNL create target connection].
   1. トリガー **[!DNL `CREATE Dataflow`]** から [!DNL create dataflow].
   1. トリガー **[!DNL `GET Base Connection`]** — これは自動的に保存されます `CustomerProfile_inlet_id` （postman 環境変数内）

1. この手順では、 `CustomerProfile_dataset_id` および `CustomerProfile_inlet_id` postman 環境変数内。そうでない場合は、手順を参照してください。 `3` または `4` それぞれ。
1. 顧客を取り込むには、ユーザーが `customer_country_code`, `customer_mobile_no`, `customer_first_name`, `customer_last_name` および `email` postman 環境変数内。

   1. `customer_country_code` はモバイル番号の国コードです。例： `91` または `1`
   1. `customer_mobile_no` の場合は、モバイル番号 ( 例： `9987654321`
   1. `customer_first_name` はユーザーの名です。
   1. `customer_last_name` はユーザーの姓になります
   1. `email` はユーザーの電子メールアドレスになります。これは、新しいプロファイルを取り込むために、個別の電子メール id を使用することが重要です。

1. Postmanリクエストの更新 **[!DNL Customer Ingestion]** > **[!DNL Customer Streaming Ingestion]** 顧客の優先チャネルを変更する場合（デフォルト） [!DNL `email`] がリクエストで設定されている。

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

1. 優先チャネルを次に変更： `sms` または `push` を設定し、それぞれのチャネル値を `y` および `n` を他の値（例： ）に置き換えます。

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

1. 最終トリガー **[!DNL `Customer Profile Ingestion > Customer Profile Streaming Ingestion`]** 顧客プロファイルを取り込みます。

## イベントを取り込み

1. ユーザーが初めて **[!DNL event dataset]** および **[!DNL HTTP Streaming Inlet Connection for events]**
1. 既に **[!DNL event dataset]** および **[!DNL HTTP Streaming Inlet Connection for events]**、スキップして手順に進んでください `5`.
1. トリガー **[!DNL `Schemas Data Ingestion > AEP Demo Schema Ingestion > Create AEP Demo Schema InletId > Create Dataset`]** を作成します。 **[!DNL event dataset]**&#x200B;を返す場合は、 `AEPDemoSchema_dataset_id` postman 環境変数内
1. 作成 **[!DNL HTTP Streaming Inlet Connection for events]**、でPostman API を使用する **[!DNL Schemas Data Ingestion]** > **[!DNL AEP Demo Schema Ingestion]** > **[!DNL Create AEP Demo Schema InletId]**.

   1. `AEPDemoSchema_dataset_id` は、postman 環境変数で使用できる必要があります（使用できない場合は、手順を参照してください）。 `3`
   1. トリガー **[!DNL `CREATE Base Connection`]** から [!DNL create base connection]
   1. トリガー **[!DNL `CREATE Source Connection`]** から [!DNL create source connection]
   1. トリガー **[!DNL `CREATE Target Connection`]** から [!DNL create target connection]
   1. トリガー **[!DNL `CREATE Dataflow`]** から [!DNL create dataflow]
   1. トリガー **[!DNL `GET Base Connection`]** — これは自動的に保存されます `AEPDemoSchema_inlet_id` postman 環境変数内

1. この手順では、 `AEPDemoSchema_dataset_id` および `AEPDemoSchema_inlet_id` postman 環境変数で、そうでない場合は step を参照してください。 `3` または `4` それぞれ
1. イベントを取り込むには、ユーザーが時間変数を変更する必要があります `timestamp` ～の要請本文で **[!DNL Schemas Data Ingestion]** > **[!DNL AEP Demo Schema Ingestion]** > **[!DNL AEP Demo Schema Streaming Ingestion]** 郵便配達人で

   1. `timestamp` イベントが発生した時刻には、現在のタイムスタンプを使用します。例： `2023-07-21T16:37:52+05:30` 必要に応じてタイムゾーンを調整します。

1. トリガー **[!DNL Schemas Data Ingestion > AEP Demo Schema Ingestion > AEP Demo Schema Streaming Ingestion]** イベントを取り込み、ジャーニーをトリガーする

## 最終検証

選択した優先チャネル ( **[!DNL Ingest the Customer Profile]** step `8`

* `SMS` （優先チャネルの場合） `sms` オン `customer_country_code` および `customer_mobile_no`
* `Email` （優先チャネルの場合） `email` オン `email`

または、 `Journey Report`をクリックして、 `Journey Object` オン `Bill of Materials page` 次にリダイレクトされます。 `Journey Details page`.

公開されたジャーニーのユーザーは、 **[!UICONTROL レポートを表示]** ボタン
![ジャーニーレポートページ](../assets/journey-report-page.png)


## クリーンアップ

の複数のインスタンスを持たないでください。 `Journey` 同時に実行する場合は、ジャーニーが検証が完了した後でのみ検証される場合は、検証を停止してください。
