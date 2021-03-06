---
title: Node.js アプリケーションのインスツルメンテーション
kind: ドキュメント
further_reading:
  - link: serverless/installation/node
    tag: Documentation
    text: Node.js サーバーレスモニタリングのインストール
  - link: serverless/installation/ruby
    tag: Documentation
    text: Ruby サーバーレスモニタリングのインストール
---
[AWS インテグレーション][1]と [Datadog Forwarder][2] をインストールしたら、以下のいずれかの方法を選択してアプリケーションをインスツルメントし、Datadog にメトリクス、ログ、トレースを送信します。

## 構成

{{< tabs >}}
{{% tab "Serverless Framework" %}}

[Datadog Serverless Plugin][1] は、レイヤーを使用して Datadog Lambda ライブラリを関数に自動的に追加し、[Datadog Forwarder][2] を介してメトリクス、トレース、ログを Datadog に送信するように関数を構成します。

Datadog サーバーレスプラグインをインストールして構成するには、次の手順に従います。

1. Datadog サーバーレスプラグインをインストールします。
    ```
    yarn add --dev serverless-plugin-datadog
    ```
2. `serverless.yml` に以下を追加します。
    ```
    plugins:
      - serverless-plugin-datadog
    ```
3. `serverless.yml` に、以下のセクションも追加します。
    ```
    custom:
      datadog:
        flushMetricsToLogs: true
        forwarder: # The Datadog Forwarder ARN goes here.
    ```
   Datadog Forwarder ARN またはインストールの詳細については、[こちら][2]を参照してください。追加の設定については、[プラグインのドキュメント][1]を参照してください。

[1]: https://github.com/DataDog/serverless-plugin-datadog
[2]: https://docs.datadoghq.com/ja/serverless/forwarder/
{{% /tab %}}
{{% tab "AWS SAM" %}}
<div class="alert alert-warning"> このサービスは公開ベータ版です。フィードバックがございましたら、<a href="/help">Datadog サポートチーム</a>までお寄せください。</div>

[Datadog CloudFormation マクロ][1]は、SAM アプリケーションテンプレートを自動的に変換して、レイヤーを使用して Datadog Lambda ライブラリを関数に追加し、[Datadog Forwarder][2] を介してメトリクス、トレース、ログを Datadog に送信するように関数を構成します。

### Datadog CloudFormation マクロのインストール

[AWS 認証情報][3]で次のコマンドを実行して、マクロ AWS リソースをインストールする CloudFormation スタックをデプロイします。アカウントの特定のリージョンに一度だけマクロをインストールする必要があります。マクロを最新バージョンに更新するには、`create-stack` を `update-stack` に置き換えます。

```sh
aws cloudformation create-stack \
  --stack-name datadog-serverless-macro \
  --template-url https://datadog-cloudformation-template.s3.amazonaws.com/aws/serverless-macro/latest.yml \
  --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_IAM
```

マクロが表示され、使用を開始できます。

### 関数をインスツルメントする

`template.yml` で、SAM の `AWS::Serverless` 変換の**後に**、`Transform` セクションの下に以下を追加します。

```yaml
Transform:
  - AWS::Serverless-2016-10-31
  - Name: DatadogServerless
    Parameters:
      stackName: !Ref "AWS::StackName"
      nodeLayerVersion: "<LAYER_VERSION>"
      forwarderArn: "<FORWARDER_ARN>"
      service: "<SERVICE>" # オプション
      env: "<ENV>" # オプション
```

`<SERVICE>` と `<ENV>` を適切な値に置き換え、`<LAYER_VERSION>` を目的のバージョンの Datadog Lambda レイヤーに置き換え ([最新リリース][4]を参照)、`<FORWARDER_ARN>` を Forwarder ARN に置き換えます ([Forwarder のドキュメント][2]を参照)。

[マクロのドキュメント][1]に詳細と追加のパラメーターがあります。

[1]: https://github.com/DataDog/datadog-cloudformation-macro
[2]: https://docs.datadoghq.com/ja/serverless/forwarder/
[3]: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html
[4]: https://github.com/DataDog/datadog-lambda-js/releases
{{% /tab %}}
{{% tab "AWS CDK" %}}

<div class="alert alert-warning"> このサービスは公開ベータ版です。フィードバックがございましたら、<a href="/help">Datadog サポートチーム</a>までお寄せください。</div>

[Datadog CloudFormation マクロ][1]は、AWS CDK によって生成された CloudFormation テンプレートを自動的に変換して、レイヤーを使用して Datadog Lambda ライブラリを関数に追加し、[Datadog Forwarder][2] を介してメトリクス、トレース、ログを Datadog に送信するように関数を構成します。

### Datadog CloudFormation マクロのインストール

[AWS 認証情報][3]で次のコマンドを実行して、マクロ AWS リソースをインストールする CloudFormation スタックをデプロイします。アカウントの特定の地域に**一度だけ**マクロをインストールする必要があります。マクロを最新バージョンに更新するには、`create-stack` を `update-stack` に置き換えます。

```sh
aws cloudformation create-stack \
  --stack-name datadog-serverless-macro \
  --template-url https://datadog-cloudformation-template.s3.amazonaws.com/aws/serverless-macro/latest.yml \
  --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_IAM
```

マクロが表示され、使用を開始できます。

### 関数をインスツルメントする

AWS CDK アプリの `Stack` オブジェクトに `DatadogServerless` 変換と `CfnMapping` を追加します。以下の TypeScript のサンプルコードを参照してください (他の言語での使用方法も同様です)。

```typescript
import * as cdk from "@aws-cdk/core";

class CdkStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    this.addTransform("DatadogServerless");

    new cdk.CfnMapping(this, "Datadog", {
      mapping: {
        Parameters: {
          nodeLayerVersion: "<LAYER_VERSION>",
          forwarderArn: "<FORWARDER_ARN>",
          stackName: this.stackName,
          service: "<SERVICE>", // オプション
          env: "<ENV>", // オプション
        },
      },
    });
  }
}
```

`<SERVICE>` と `<ENV>` を適切な値に置き換え、`<LAYER_VERSION>` を目的のバージョンの Datadog Lambda レイヤーに置き換え ([最新リリース][3]を参照)、`<FORWARDER_ARN>` を Forwarder ARN に置き換えます ([Forwarder のドキュメント][2]を参照)。

[マクロのドキュメント][1]に詳細と追加のパラメーターがあります。

[1]: https://github.com/DataDog/datadog-cloudformation-macro
[2]: https://docs.datadoghq.com/ja/serverless/forwarder/
[3]: https://github.com/DataDog/datadog-lambda-js/releases
{{% /tab %}}
{{% tab "Datadog CLI" %}}

<div class="alert alert-warning"> このサービスは公開ベータ版です。フィードバックがございましたら、<a href="/help">Datadog サポートチーム</a>までお寄せください。</div>

Datadog CLI を使用して、CI/CD パイプラインの Lambda 関数にインスツルメンテーションを設定します。CLI コマンドは、レイヤーを使用して Datadog Lambda ライブラリを関数に自動的に追加し、メトリクス、トレース、ログを Datadog に送信するように関数を構成します。

### Datadog CLI のインストール

NPM または Yarn を使用して Datadog CLI をインストールします。

```sh
# NPM
npm install -g @datadog/datadog-ci

# Yarn
yarn global add @datadog/datadog-ci
```

### 関数をインスツルメントする

[AWS 認証情報][1]を使用して次のコマンドを実行します。`<functionname>` と `<another_functionname>` を Lambda 関数名に置き換え、`<aws_region>` を AWS リージョン名に置き換え、`<layer_version>` を目的のバージョンの Datadog Lambda レイヤーに置き換え ([最新リリース][2]を参照)、`<forwarder_arn>` を Forwarder ARN に置き換えます ([Forwarder のドキュメント][3]を参照)。

```sh
datadog-ci lambda instrument -f <functionname> -f <another_functionname> -r <aws_region> -v <layer_version> --forwarder <forwarder_arn>
```

例:

```sh
datadog-ci lambda instrument -f my-function -f another-function -r us-east-1 -v 26 --forwarder arn:aws:lambda:us-east-1:000000000000:function:datadog-forwarder
```

[CLI のドキュメント][4]に詳細と追加のパラメーターがあります。

[1]: https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-credentials-node.html
[2]: https://github.com/DataDog/datadog-lambda-js/releases
[3]: https://docs.datadoghq.com/ja/serverless/forwarder/
[4]: https://github.com/DataDog/datadog-ci/tree/master/src/commands/lambda
{{% /tab %}}
{{% tab "Custom" %}}

### Datadog Lambda ライブラリのインストール

Datadog Lambda ライブラリは、レイヤーまたは JavaScript パッケージとしてインポートすることができます。

`datadog-lambda-js` パッケージのマイナーバージョンは、常にレイヤーのバージョンに一致します。例: datadog-lambda-js v0.5.0 は、レイヤーバージョン 5 のコンテンツに一致。

#### レイヤーの使用

以下のフォーマットで、ARN を使用して Lambda 関数に[レイヤーを構成][1]します。

```
# 通常のリージョンの場合
arn:aws:lambda:<AWS_REGION>:464622532012:layer:Datadog-<RUNTIME>:<VERSION>

# 米国政府リージョンの場合
arn:aws-us-gov:lambda:<AWS_REGION>:002406178527:layer:Datadog-<RUNTIME>:<VERSION>
```

使用できる `RUNTIME` オプションは、`Node8-10`、`Node10-x`、`Node12-x` です。`VERSION` については、[最新リリース][2]を参照してください。例:

```
arn:aws:lambda:us-east-1:464622532012:layer:Datadog-Node12-x:25
```

#### パッケージの使用

**NPM**:

```
npm install --save datadog-lambda-js
```

**Yarn**:

```
yarn add datadog-lambda-js
```

[最新リリース][3]を参照。

### 関数の構成

1. 関数のハンドラーを、レイヤーを使用する場合は `/opt/nodejs/node_modules/datadog-lambda-js/handler.handler` に、パッケージを使用する場合は `node_modules/datadog-lambda-js/dist/handler.handler` に設定します。
2. 元のハンドラーに、環境変数 `DD_LAMBDA_HANDLER` を設定します。例: `myfunc.handler`。
3. 環境変数 `DD_TRACE_ENABLED` を `true` に設定します。
4. 環境変数 `DD_FLUSH_TO_LOG` を `true` に設定します。
5. オプションで、関数に `service` および `env` タグを適切な値とともに追加します。

### Datadog Forwarder をロググループにサブスクライブ

メトリクス、トレース、ログを Datadog へ送信するには、関数の各ロググループに Datadog Forwarder Lambda 関数をサブスクライブする必要があります。

1. [まだの場合は、Datadog Forwarder をインストールします][4]。
2. [DdFetchLambdaTags のオプションが有効であることを確認します][5]。
3. [Datadog Forwarder を関数のロググループにサブスクライブします][6]。


[1]: https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html
[2]: https://github.com/DataDog/datadog-lambda-layer-js/releases
[3]: https://www.npmjs.com/package/datadog-lambda-js
[4]: https://docs.datadoghq.com/ja/serverless/forwarder/
[5]: https://docs.datadoghq.com/ja/serverless/forwarder/#experimental-optional
[6]: https://docs.datadoghq.com/ja/logs/guide/send-aws-services-logs-with-the-datadog-lambda-function/#collecting-logs-from-cloudwatch-log-group
{{% /tab %}}
{{< /tabs >}}

## Datadog サーバーレスモニタリングの利用

以上の方法で関数を構成すると、[Serverless Homepage][2] でメトリクス、ログ、トレースを確認できるようになります。

カスタムメトリクスの送信または関数の手動インスツルメントをご希望の場合は、以下のコード例をご参照ください。

```javascript
const { sendDistributionMetric } = require("datadog-lambda-js");
const tracer = require("dd-trace");

// "sleep" という名前のカスタム APM スパンを送信します
const sleep = tracer.wrap("sleep", (ms) => {
  return new Promise((resolve) => setTimeout(resolve, ms));
});

exports.handler = async (event) => {
  await sleep(1000);

  // カスタムメトリクスを送信します
  sendDistributionMetric(
    "coffee_house.order_value", // メトリクス名
    12.45, // metric Value
    "product:latte", // タグ
    "order:online", // 別のタグ
  );
  const response = {
    statusCode: 200,
    body: JSON.stringify("Hello from serverless!"),
  };
  return response;
};
```

[1]: /ja/integrations/amazon_web_services/
[2]: https://app.datadoghq.com/functions