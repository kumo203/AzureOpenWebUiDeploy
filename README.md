# AzureOpenWebUiDeploy

Azure App Service (Web App for Containers) 上に [Open WebUI](https://github.com/open-webui/open-webui) をデプロイする ARM テンプレートです。

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fkumo203%2FAzureOpenWebUiDeploy%2Fmain%2Fazuredeploy.json)

## 構成

- **App Service Plan (Linux)**: デフォルト SKU は `B2`。RAG用の埋め込みモデルをコンテナ内でロードするため、`B1`(1.75GB RAM) だとメモリ不足で落ちる可能性があります。
- **Web App for Containers**: イメージは `ghcr.io/open-webui/open-webui:main` 固定ではなくパラメータ化。
- **Storage Account + Azure Files**: `/app/backend/data` を永続化するため、Web App にファイル共有をマウントします（チャット履歴・アップロードファイル・キャッシュなどが保存される場所）。

## 主なパラメータ

| パラメータ | 説明 | デフォルト |
|---|---|---|
| `webAppName` | Web App 名（URL の一部） | `openwebui-<一意な文字列>` |
| `appServicePlanSku` | App Service Plan の SKU | `B2` |
| `containerImage` | コンテナイメージ | `ghcr.io/open-webui/open-webui:main` |
| `webuiSecretKey` | セッション署名キー (`WEBUI_SECRET_KEY`) | 空（コンテナ側が `/app/backend/data` に自動生成・永続化） |
| `webuiAuth` | ログイン認証の有効化 (`WEBUI_AUTH`) | `true` |
| `enableSignup` | 新規サインアップ許可 (`ENABLE_SIGNUP`) | `true` |
| `openAiApiKey` | OpenAI互換APIキー (`OPENAI_API_KEY`) | 空（任意。デプロイ後にAdmin Settings > Connectionsからも設定可能） |
| `openAiApiBaseUrl` | OpenAI互換APIのベースURL (`OPENAI_API_BASE_URL`) | `https://api.openai.com/v1` |
| `enableOllamaApi` | Ollama連携の有効化 (`ENABLE_OLLAMA_API`) | `false` |
| `ollamaBaseUrl` | Ollamaサーバー URL (`OLLAMA_BASE_URL`) | 空 |

Azure AI Foundry のモデルエンドポイントを使う場合は `openAiApiBaseUrl` に `https://<リソース名>.services.ai.azure.com/openai/v1` のようなURLを指定してください。

## CLI でのデプロイ例

```bash
az deployment group create \
  --resource-group <リソースグループ名> \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json
```

`azuredeploy.parameters.json` の `openAiApiKey` はサンプル値です。実際のキーはコマンドラインやパラメータファイルにベタ書きせず、`az deployment group create --parameters openAiApiKey=<キー>` のように渡すか、Key Vault 参照の利用を検討してください。

## 注意事項

- APIキーなどの秘密情報は Git リポジトリにコミットしないでください。
- ローカルの Docker コンテナで使っていた API キーを誤ってチャットや設定ファイルに貼り付けた場合は、発行元（Azure AI Foundry など）で速やかにローテーションしてください。