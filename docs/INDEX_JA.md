# 日本語ドキュメントインデックス

これはプロジェクトドキュメントのエントリポイントです。README ではプロジェクト概要とクイックスタートを扱い、詳細なセットアップ、設定、デプロイ、機能の使い方、トラブルシューティングのドキュメントは以下にリンクしています。

> 中国語のドキュメントは [docs/INDEX.md](INDEX.md) を参照してください。

## 目的から選ぶ

| やりたいこと | まず読む | 次に読む |
| --- | --- | --- |
| プロジェクトの概要を理解する | [README (EN)](README_EN.md) | [完全ガイド (EN)](full-guide_EN.md) |
| プロジェクトを初めて実行する | [README (EN)](README_EN.md) | [完全ガイド (EN)](full-guide_EN.md) |
| モデルプロバイダーを設定する | [LLM 設定ガイド (EN)](LLM_CONFIG_GUIDE_EN.md) | [プロバイダー設定ガイド](llm-providers.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） |
| 通知を設定する | [通知ベースライン](notifications.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） | [完全ガイド (EN)](full-guide_EN.md) |
| サーバーやクラウドプラットフォームにデプロイする | [デプロイガイド (EN)](DEPLOY_EN.md) | [クラウド WebUI デプロイ](deploy-webui-cloud.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ）、[Zeabur デプロイ](docker/zeabur-deployment.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） |
| Bot / IM 連携を使う | [Bot コマンド (EN)](bot-command_EN.md) | [Bot プラットフォームドキュメント](bot/) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） |
| 実行時の問題をトラブルシューティングする | [FAQ (EN)](FAQ_EN.md) | [変更履歴](CHANGELOG.md) |
| コードやドキュメントに貢献する | [コントリビューションガイド (EN)](CONTRIBUTING_EN.md) | [API 仕様](architecture/api_spec.json) |

## はじめに

| ドキュメント | 内容 |
| --- | --- |
| [README (EN)](README_EN.md) | プロジェクト概要、主要機能、クイックスタート、サンプル出力 |
| [完全ガイド (EN)](full-guide_EN.md) | 環境セットアップ、実行モード、設定、デプロイ手順、よくある問題 |
| [FAQ (EN)](FAQ_EN.md) | よくある設定・モデル・通知・デプロイ・実行時の問題 |
| [変更履歴](CHANGELOG.md) | リリースノート、機能変更、移行に関する注意 |

## 設定

| ドキュメント | 内容 |
| --- | --- |
| [LLM 設定ガイド (EN)](LLM_CONFIG_GUIDE_EN.md) | モデルプロバイダー、3 ティア構成、Web 設定、一般的なモデルのセットアップ |
| [プロバイダー設定ガイド](llm-providers.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） | プロバイダープリセット、GitHub Actions マッピング、エラーカテゴリ、診断 |
| [LiteLLM YAML 例](examples/litellm_config.example.yaml) | LiteLLM のマルチプロバイダー設定例 |
| [通知ベースライン](notifications.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） | WeChat Work、Feishu、Telegram、Discord、Slack、メールなどの通知チャネル |
| [Tushare 銘柄リストガイド](TUSHARE_STOCK_LIST_GUIDE.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） | Tushare 銘柄リストの設定と使用上の注意 |

## 使い方トピック

| ドキュメント | 内容 |
| --- | --- |
| [Bot コマンド (EN)](bot-command_EN.md) | Bot コマンド、Webhook、プラットフォーム連携、コールバックの挙動 |
| [Bot プラットフォームドキュメント](bot/) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） | Feishu、DingTalk、Discord などの Bot 設定スクリーンショットと注意 |
| [リアルタイムアラートセンター](alerts.md) <sub><sub>![P4 Badge](https://img.shields.io/badge/P4-yellow?style=flat)</sub></sub>（中国語のみ） | EventMonitor ベースライン、Web ルール管理、通知試行、クールダウン状態、フェーズ境界 |
| [分析コンテキストパックの契約・ランタイム消費・可視性](analysis-context-pack.md) <sub><sub>![P6 Badge](https://img.shields.io/badge/P6-orange?style=flat)</sub></sub>（中国語のみ） | AnalysisContextPack の初期スコープ境界、フィールド品質ステータス、P1/P2 内部契約、P3 プロンプトサマリー消費、P4 履歴/API/Web 低センシティビティ可視性、P5 データ品質スコアリング、P6 移行/ロールバックの注意、およびソースアンカー。完全ガイドには #1386 のマーケットフェーズ分析、移行、ロールバックのエントリポイントが追加されています |
| [画像抽出プロンプト](image-extract-prompt.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） | 画像から銘柄情報を抽出するためのプロンプトと境界 |
| [OpenClaw Skill 連携](openclaw-skill-integration.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） | OpenClaw / Skill の外部連携に関する注意 |

## デプロイとパッケージング

| ドキュメント | 内容 |
| --- | --- |
| [デプロイガイド (EN)](DEPLOY_EN.md) | サーバーデプロイ、Docker、systemd、Supervisor などのオプション |
| [クラウド WebUI デプロイ](deploy-webui-cloud.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） | クラウドサーバーでの WebUI アクセスとデプロイの注意 |
| [Zeabur デプロイ](docker/zeabur-deployment.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） | Zeabur プラットフォームへのデプロイ |
| [デスクトップパッケージング](desktop-package.md) <sub><sub>![P2 Badge](https://img.shields.io/badge/P2-yellow?style=flat)</sub></sub>（中国語のみ） | Electron デスクトップアプリと Web 成果物のパッケージング |

## リファレンスと開発

| ドキュメント | 内容 |
| --- | --- |
| [API 仕様](architecture/api_spec.json) | FastAPI の OpenAPI 成果物 |
| [コントリビューションガイド (EN)](CONTRIBUTING_EN.md) | Issue、プルリクエスト、テスト、ドキュメント同期、協業の期待事項 |

## 言語

| ドキュメント | 内容 |
| --- | --- |
| [中国語ドキュメントインデックス](INDEX.md) | 中国語ドキュメントのエントリポイント |
| [繁体字中国語 README](README_CHT.md) | 繁体字中国語のプロジェクト概要とクイックスタート |

## 中国市場用語集

| 用語 | 意味 |
| --- | --- |
| **A 株** | 上海証券取引所または深セン証券取引所に上場し、人民元建てで取引される株式 |
| **北向き資金フロー（Northbound）** | Stock Connect プログラムを通じた外国人投資家による純買い／純売りの資金フロー |
| **龍虎榜（Dragon-Tiger List）** | 売買が活発な銘柄と上位の売買シートを開示する SSE/SZSE の日次開示 |
| **チップ分布（Chip distribution）** | 発行済み株式の取得コストベースの分布。支持線・抵抗線の推定によく用いられる |
| **Tushare** | トークンを必要とする中国の金融データ API |
| **AkShare** | オープンソースの Python マーケットデータライブラリ |
| **Baostock** | A 株のヒストリカルデータ向けの無料 Python SDK |
| **WeChat Work** | Webhook 通知に対応した Tencent のエンタープライズメッセージングプラットフォーム |
| **Feishu** | Webhook 通知に対応した ByteDance のエンタープライズコラボレーションプラットフォーム |
| **PushPlus / ServerChan** | 中国のモバイルプッシュ通知サービス |
