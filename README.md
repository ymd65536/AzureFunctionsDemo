# Azure Template Repo README

## Blob Event Grid トリガーのローカル検証

このワークスペースでは、サンプルを `functions-quickstart-dotnet-azd-eventgrid-blob`
に配置します。このフォルダーは Git 管理対象外のため、別の環境ではサンプルの取得も必要です。
以下はサンプルが配置済みの場合の手順です。

現在のプロジェクトは .NET 10 を対象としています。`.NET 10 SDK`、Azure Functions
Core Tools v4 と、VS Code の推奨拡張機能（Azurite、Azure Storage、REST Client）が必要です。
`azurite` コマンドが PATH に存在しなくても、Azurite 拡張機能から起動できます。

### 1. Azurite と入力 Blob を準備する

1. VS Code のコマンドパレットで `Azurite: Start` を実行します。
   Blob / Queue / Table の各サービスを起動してください。
2. Azure Storage 拡張機能のローカルエミュレーターに、次のコンテナーを作成します。
   - `unprocessed-pdf`
   - `processed-pdf`
3. サンプルの `data/PerksPlus.pdf` を `unprocessed-pdf` にアップロードします。
   Blob 名は大文字・小文字を含めて `PerksPlus.pdf` に合わせてください。

### 2. Functions ホストを起動する

サンプルの `src/local.settings.json` を次の設定にします。
Azure の接続文字列ではなく、ローカルエミュレーターを使用します。

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
    "PDFProcessorSTORAGE": "UseDevelopmentStorage=true"
  }
}
```

ワークスペースのルートから次を実行し、ターミナルを開いたままにします。

```bash
cd functions-quickstart-dotnet-azd-eventgrid-blob/src
func start
```

すでにポート 7071 でホストが起動している場合、二重に起動する必要はありません。
別のターミナルから以下で状態を確認できます。

```bash
curl --fail-with-body http://localhost:7071/admin/host/status
```

`"state":"Running"` が起動成功の目安です。

### 3. Event Grid イベントを手動送信する

**Azurite は Event Grid イベントを自動送信しません。**
このサンプルは `BlobTriggerSource.EventGrid` を使用するため、
PDF をアップロードするだけでは関数は実行されません。

ルートの [test.http](./test.http) を開き、POST リクエストの `Send Request` を実行します。
このリクエストはローカルの Blob webhook に CloudEvent を送信し、
Azure Event Grid からの通知を模擬します。

- `@blobName` は、Azurite にアップロード済みの Blob 名と一致させてください。
- `data.url` は Azurite の URL を使用します。Azure 上のサンプル URL とは異なり、
  `http://127.0.0.1:10000/devstoreaccount1/...` になります。
- 別のポートで Functions を起動した場合は `@host` を変更してください。

レスポンスの `202 Accepted` はイベントの受理を示しますが、処理完了の保証ではありません。
Functions の実行ログと、`processed-pdf/processed-PerksPlus.pdf` の生成を確認してください。
このサンプルは出力 Blob が存在するとコピーをスキップします。
新しい処理を検証するには、別名の PDF をアップロードして `@blobName` も変更してください。

この検証の対象はイベント受信後の Blob 処理です。図の Event Grid サブスクリプション、
Private Endpoint、VNet、マネージド ID は Azurite では再現されず、Azure 上での検証が別途必要です。
公式手順は [Azure Functions を使用して BLOB ストレージ イベントに応答する](https://learn.microsoft.com/ja-jp/azure/azure-functions/scenario-blob-storage-events?tabs=linux&pivots=programming-language-csharp)
を参照してください。

## Azure Developer CLIのセットアップ

以下のコマンドを実行して、Azure Developer CLI (azd) をインストールします。

```bash
curl -fsSL https://aka.ms/install-azd.sh | bash
```

インストール方法は[公式ドキュメント](https://learn.microsoft.com/ja-jp/azure/developer/azure-developer-cli/install-azd)を参照してください。

### Azure Developer CLIの動作確認

以下のコマンドでAzure Developer CLIのバージョンを確認します。

```bash
azd version
# azd version 1.35.0 (commit 170ebb858071da353cc9bf8657a377bff268ba36) (stable)
```

### Azure Developer CLIでログインする

環境変数 `AZURE_TENANT_ID`が設定されている場合は、以下のコマンドでログインします。

```bash
azd auth login --tenant-id $AZURE_TENANT_ID
```

環境変数が設定されていない場合は、以下のコマンドでログインします。

```bash
azd auth login
```

## Azure CLIをセットアップする

以下のコマンドを実行して、Azure CLIをインストールします。

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

インストール方法は[公式ドキュメント](https://learn.microsoft.com/ja-jp/cli/azure/install-azure-cli-linux?pivots=apt)を参照してください。

## Azure CLIでログインする

環境変数 `AZURE_TENANT_ID`が設定されている場合は、以下のコマンドでログインします。

```bash
az login --tenant $AZURE_TENANT_ID
```

### Azure CLIの動作確認

以下のコマンドでAzure CLIのバージョンとアカウント情報を確認します。

```bash
az version
az account list
```

## GitHub Codespacesの設定

`.env`でシークレットを管理する場合、以下のコマンドでCodespacesにシークレットを設定します。

```bash
gh secret set --app codespaces -f .env
```

シークレットの一覧を確認するには、以下のコマンドを実行します。

```bash
gh secret list --app codespaces
```

単一のシークレットを設定するには、以下のコマンドを使用します。

```bash
gh secret set --app codespaces SECRET_NAME
```

シークレットの削除は以下のコマンドで行います。

```bash
gh secret delete --app codespaces SECRET_NAME
```
