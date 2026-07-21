# AzureOpenWebUiDeploy

Azure 上に [Open WebUI](https://github.com/open-webui/open-webui) をデプロイする ARM テンプレート集です。

以下の Deploy ボタンから、**Azure Container Instances (ACI)** 上に検証用の Open WebUI をワンクリックでデプロイできます。

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fkumo203%2FAzureOpenWebUiDeploy%2Fmain%2Faci%2Fazuredeploy.json)

## 構成一覧

| ディレクトリ | プラン | 用途 |
|---|---|---|
| [`aci/`](aci/README.md) | Azure Container Instances | 永続化なしの検証・使い捨て用。上の Deploy ボタンはこちらを使用します。 |
| [`ConApps/`](ConApps/README.md) | Azure Container Apps | Azure Files でデータを永続化する本番向け構成。 |

用途に応じて各ディレクトリの README を参照してください。

## 注意事項

- APIキーなどの秘密情報は Git リポジトリにコミットしないでください。
- ローカルの Docker コンテナで使っていた API キーを誤ってチャットや設定ファイルに貼り付けた場合は、発行元（Azure AI Foundry など）で速やかにローテーションしてください。
