# [私家版]  OpenTelemetry ドキュメント日本語化プロジェクト

OpenTelemetry documents Japanese translation poject

!!このレポジトリの内容は私家版であり、OpenTelemtryプロジェクトとは無関係です!!

## 翻訳対象ドキュメント (Target repositories)

* [community](https://github.com/open-telemetry/community)
  * 67d8f20a8b0cb49e3fd9033545c3f9e5865de7cf (2021-02-11バージョン)
* [specification](https://github.com/open-telemetry/opentelemetry-specification)
  * v1.3.0

## 翻訳に取り掛かる前に

### 訳語について

翻訳プロジェクトへようこそ。この翻訳プロジェクトでは、全体的に翻訳に統一感を出すための工夫をしています。

* 訳語一覧: [wikiのDictionary](https://github.com/open-telemetry/docs-ja/wiki/Dictionary) で英語と日本語の対訳の一覧を作成しているので、翻訳時にこちらに揃えてください。
* 助動詞について: 原文はBCP 14に準ずると[明記しているので](https://github.com/open-telemetry/opentelemetry-specification#notation-conventions-and-compliance)、日本語訳もそれに従います。
  * 参照: https://www.ipa.go.jp/security/rfc/RFC2119JA.html

**TODO: textlintなどの方法である程度自動化する**

### 機械翻訳の使用について

機械翻訳を利用して翻訳作業を行うことは技術の正しい利用方法だと思います。
しかしながら、サービスによっては次のような問題があるため注意が必要です。

* 翻訳結果の著作権（例: DeepL無料版、Google翻訳など）
* 翻訳結果自体の品質

したがって、機械翻訳を利用する場合には、上記の懸念が明確にないと確認できる状況以外では、機械翻訳による翻訳結果をそのままレポジトリにpushすることは避けてください。（各種サービスがCNCFのCLAにサインしたわけでもないため）

## コントリビューションワークフロー

1. Issueより対象のドキュメントに対し作業に取り掛かる旨コメントしてください。
2. 作業ドキュメントは原文の英語ドキュメントをコピーし、翻訳を進めるにあたりそれをコメントアウトする形で勧めてください。([参考](https://raw.githubusercontent.com/open-telemetry/docs-ja/master/specification/library-guidelines.md))
4. 作業中は `WIP` をつけたブランチを作成しPRを早めに出してください。
5. Reviewerがレビューを行ってapproveされるまで修正をお願いします。
6. ApproveされたものはCODEOWNERSの誰かがsquash mergeします。

もし翻訳等で質問がある場合はIssueでコメントしてください。

## チャット

翻訳プロジェクトのチャットはこちらのdiscordの `#otel-docs-ja` チャンネルで行っています。

https://discord.gg/vYKuzFK

## 翻訳者ミーティング

よりオープンなプロジェクトにするために、オープンな定期ミーティングを計画しています。
