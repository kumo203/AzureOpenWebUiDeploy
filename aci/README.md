# Open WebUI on Azure Container Instances（検証・使い捨て用）

このディレクトリのテンプレートは **動作検証専用** です。リポジトリルートの `azuredeploy.json`（Azure Container
Apps + Azure Files永続化）とは別物で、次の点が異なります。

- **永続化なし**: `/app/backend/data`（SQLiteのDB、アップロードファイル、キャッシュなど）はコンテナのローカル
  (エフェメラル)ディスクに置かれます。コンテナの再作成・削除でデータは消えます。
- Azure Files (SMB) をマウントしないため、Open WebUIのデフォルトDBであるSQLiteのファイルロックが正しく機能し、
  ルート版で発生した `database is locked` エラーを回避できます。
- **HTTPのみ**: ACIにはApp Service/Container Appsのようなビルトインのingress/HTTPS終端がなく、Open WebUI自体も
  HTTPSを話さないため、`http://<dnsNameLabel>.<region>.azurecontainer.io:8080` でのHTTPアクセスになります。
  ブラウザの「HTTPSを常に使用する」設定(Chrome/EdgeのHTTPS-First、FirefoxのHTTPS-Onlyモード等)が有効だと
  `https://`に自動で書き換えられて接続できないことがあるため、その場合は該当設定を一時的に無効にするか
  例外に追加してください。

まずはこの構成でコンテナイメージが問題なく起動・応答するかを確認し、その後永続化（外部DB等）を含む本番構成の
検討に進むための土台として使ってください。

## 主なパラメータ

| パラメータ | 説明 | デフォルト |
|---|---|---|
| `containerGroupName` | コンテナグループ名 | `openwebui-aci-<一意な文字列>` |
| `dnsNameLabel` | 公開URL用DNSラベル（リージョン内で一意） | `openwebui-aci-<一意な文字列>` |
| `containerImage` | コンテナイメージ | `ghcr.io/open-webui/open-webui:main` |
| `cpuCores` | 割り当てるvCPU数 | `1` |
| `memoryInGB` | 割り当てるメモリ(GB) | `2` |
| `restartPolicy` | 終了時の再起動ポリシー (`Always`/`Never`/`OnFailure`) | `Never` |
| `webuiSecretKey` | セッション署名キー (`WEBUI_SECRET_KEY`) | 空（永続化されないため実質使い捨て） |
| `webuiAuth` | ログイン認証の有効化 (`WEBUI_AUTH`) | `true` |
| `enableSignup` | 新規サインアップ許可 (`ENABLE_SIGNUP`) | `true` |
| `openAiApiKey` | OpenAI互換APIキー (`OPENAI_API_KEY`) | 空（任意。デプロイ後にAdmin Settings > Connectionsからも設定可能） |
| `openAiApiBaseUrl` | OpenAI互換APIのベースURL (`OPENAI_API_BASE_URL`) | `https://api.openai.com/v1` |
| `enableOllamaApi` | Ollama連携の有効化 (`ENABLE_OLLAMA_API`) | `false` |
| `ollamaBaseUrl` | Ollamaサーバー URL (`OLLAMA_BASE_URL`) | 空 |

## デプロイ

```bash
az group create -n <リソースグループ名> -l japaneast

az deployment group create \
  --resource-group <リソースグループ名> \
  --template-file aci/azuredeploy.json \
  --parameters aci/azuredeploy.parameters.json
```

## ログ確認・後片付け

```bash
az container logs -g <リソースグループ名> -n <containerGroupName>
```

検証が終わったら、コンテナグループまたはリソースグループごと削除してください。

```bash
az container delete -g <リソースグループ名> -n <containerGroupName> --yes
# もしくは検証専用のリソースグループごと削除
az group delete -n <リソースグループ名> --yes --no-wait
```
