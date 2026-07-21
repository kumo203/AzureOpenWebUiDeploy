# AzureOpenWebUiDeploy

Azure Container Apps 上に [Open WebUI](https://github.com/open-webui/open-webui) をデプロイする ARM テンプレートです。

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fkumo203%2FAzureOpenWebUiDeploy%2Fmain%2FConApps%2Fazuredeploy.json)

クイックに動作検証だけしたい場合は、永続化なしの使い捨てACIテンプレート ([`aci/`](../aci/README.md)) も参照してください。

## 構成

- **Container Apps (Consumption プラン)**: `Microsoft.App/managedEnvironments` + `Microsoft.App/containerApps`。App Service Plan のような専有VM(Dedicated)クォータを消費しないため、サブスクリプションによってはApp Serviceでデプロイできないケースでも動作します。
- **Log Analytics ワークスペース**: Container Apps環境のログ出力先として必須のため作成します。
- **Storage Account + Azure Files**: `/app/backend/data` を永続化するため、Container Appsの環境ストレージとしてマウントします（チャット履歴・アップロードファイル・キャッシュなどが保存される場所）。
- 複数レプリカ間でのデータ競合を避けるため `minReplicas`/`maxReplicas` はともに `1` に固定しています。

## 主なパラメータ

| パラメータ | 説明 | デフォルト |
|---|---|---|
| `appName` | Container App の名前（URL の一部） | `openwebui-<一意な文字列>` |
| `storageAccountName` | 永続データ用ストレージアカウント名（3〜24文字の小文字英数字、グローバルに一意） | `owui<一意な文字列>` |
| `containerImage` | コンテナイメージ | `ghcr.io/open-webui/open-webui:main` |
| `containerCpu` | 割り当てるvCPU数 | `1.0` |
| `containerMemory` | 割り当てるメモリ | `2Gi` |
| `webuiSecretKey` | セッション署名キー (`WEBUI_SECRET_KEY`) | 空（コンテナ側が `/app/backend/data` に自動生成・永続化） |
| `webuiAuth` | ログイン認証の有効化 (`WEBUI_AUTH`) | `true` |
| `enableSignup` | 新規サインアップ許可 (`ENABLE_SIGNUP`) | `true` |
| `openAiApiKey` | OpenAI互換APIキー (`OPENAI_API_KEY`) | 空（任意。デプロイ後にAdmin Settings > Connectionsからも設定可能） |
| `openAiApiBaseUrl` | OpenAI互換APIのベースURL (`OPENAI_API_BASE_URL`) | `https://api.openai.com/v1` |
| `enableOllamaApi` | Ollama連携の有効化 (`ENABLE_OLLAMA_API`) | `false` |
| `ollamaBaseUrl` | Ollamaサーバー URL (`OLLAMA_BASE_URL`) | 空 |

Azure AI Foundry のモデルエンドポイントを使う場合は `openAiApiBaseUrl` に `https://<リソース名>.services.ai.azure.com/openai/v1` のようなURLを指定してください。

`containerCpu`/`containerMemory` はContainer Apps Consumptionプランで有効な組み合わせを指定する必要があります（例: `0.5`/`1Gi`、`1.0`/`2Gi`、`2.0`/`4Gi` など）。RAG用の埋め込みモデルをコンテナ内でロードするため、デフォルトの `1.0`/`2Gi` を下回ると起動時にメモリ不足で落ちる可能性があります。

## CLI でのデプロイ例

```bash
az deployment group create \
  --resource-group <リソースグループ名> \
  --template-file ConApps/azuredeploy.json \
  --parameters ConApps/azuredeploy.parameters.json
```

APIキーなどの秘密情報はコマンドラインやパラメータファイルにベタ書きせず、`az deployment group create --parameters openAiApiKey=<キー>` のように渡すか、Key Vault 参照の利用を検討してください。

## 注意事項

- APIキーなどの秘密情報は Git リポジトリにコミットしないでください。
- ローカルの Docker コンテナで使っていた API キーを誤ってチャットや設定ファイルに貼り付けた場合は、発行元（Azure AI Foundry など）で速やかにローテーションしてください。
