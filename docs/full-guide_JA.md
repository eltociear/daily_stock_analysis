# 設定・デプロイ完全ガイド

本ドキュメントは AI 株式分析システムの完全な設定ガイドであり、高度な機能や特殊なデプロイ方法を必要とするユーザー向けです。

> クイックスタートガイドは [README_EN.md](README_EN.md) にあります。本ドキュメントは高度な設定を扱います。

## プロジェクト構成

```
daily_stock_analysis/
├── main.py              # メインエントリポイント
├── src/                 # コアビジネスロジック
│   ├── analyzer.py      # AI アナライザー
│   ├── config.py        # 設定管理
│   ├── notification.py  # メッセージプッシュ通知
│   └── ...
├── data_provider/       # マルチソースデータアダプター
├── bot/                 # Bot インタラクションモジュール
├── api/                 # FastAPI バックエンドサービス
├── apps/dsa-web/        # React フロントエンド
├── docker/              # Docker 設定
├── docs/                # プロジェクトドキュメント
└── .github/workflows/   # GitHub Actions
```

## 目次

- [プロジェクト構成](#プロジェクト構成)
- [GitHub Actions の設定](#github-actions-の設定)
- [環境変数一覧（完全版）](#環境変数一覧完全版)
- [Docker デプロイ](#docker-デプロイ)
- [ローカルデプロイ](#ローカルデプロイ)
- [定時タスクの設定](#定時タスクの設定)
- [通知チャネルの設定](#通知チャネルの設定)
- [データソースの設定](#データソースの設定)
- [高度な機能](#高度な機能)
- [バックテスト](#バックテスト)
- [ローカル WebUI 管理画面](#ローカル-webui-管理画面)

---

## GitHub Actions の設定

### 1. このリポジトリを Fork する

右上の `Fork` ボタンをクリックします。

### 2. Secrets を設定する

Fork したリポジトリ → `Settings` → `Secrets and variables` → `Actions` → `New repository secret`

<div align="center">
  <img src="assets/secret_config.png" alt="GitHub Secrets Configuration" width="600">
</div>

#### AI モデルの設定（少なくとも 1 つを設定）

| Secret 名 | 説明 | 必須 |
|------------|------|:----:|
| `ANSPIRE_API_KEYS` | [Anspire](https://open.anspire.cn/?share_code=QFBC0FYC) の API キー。1 つのキーで人気の LLM と中国語に最適化された Web 検索を利用でき、本プロジェクト向けの無料枠があります | 推奨 |
| `AIHUBMIX_KEY` | [AIHubMix](https://aihubmix.com/?aff=CfMq) の API キー。1 つのキーで複数のモデルファミリーを利用でき、本プロジェクト向けにチャージ 10% 割引があります | 推奨 |
| `GEMINI_API_KEY` | [Google AI Studio](https://aistudio.google.com/) から無料キーを取得 | オプション |
| `ANTHROPIC_API_KEY` | Anthropic Claude API キー | オプション |
| `OPENAI_API_KEY` | OpenAI 互換 API キー（DeepSeek、Qwen などに対応） | オプション |
| `OPENAI_BASE_URL` | OpenAI 互換 API エンドポイント（例: `https://api.deepseek.com`） | オプション |
| `OPENAI_MODEL` | モデル名（例: `deepseek-v4-flash`） | オプション |

> *注: モデルキーまたはチャネルを少なくとも 1 つ設定してください。Anspire または AIHubMix は 1 キーでマルチモデルにアクセスできる最もシンプルな出発点です。利用可能な AI モデルキーやモデルチャネルが設定されていない場合、起動時の検証が明確なエラーを報告します。

#### 通知チャネル（複数設定でき、すべてのチャネルが通知を受信します）

> 通知チャネルのマトリクス、最小構成／高度な構成のキー分割、生成される Actions マッピング、`--check-notify` CLI の動作、Web のワンクリック通知テスト、ローカル／Docker／GitHub Actions／デスクトップのセットアップ注意事項は [Notification Guide](notifications.md) にまとめられています。

| Secret 名 | 説明 | 必須 |
|------------|------|:----:|
| `WECHAT_WEBHOOK_URL` | WeChat Work Webhook URL | オプション |
| `FEISHU_WEBHOOK_URL` | Feishu Webhook URL | オプション |
| `FEISHU_WEBHOOK_SECRET` | Feishu Webhook の署名シークレット（「署名」セキュリティが有効な場合に必須） | オプション |
| `FEISHU_WEBHOOK_KEYWORD` | Feishu Webhook のキーワード（「キーワード」セキュリティが有効な場合に必須） | オプション |
| `TELEGRAM_BOT_TOKEN` | Telegram Bot Token（@BotFather から取得） | オプション |
| `TELEGRAM_CHAT_ID` | Telegram Chat ID | オプション |
| `TELEGRAM_MESSAGE_THREAD_ID` | Telegram トピック ID（トピックへの送信用） | オプション |
| `DISCORD_WEBHOOK_URL` | Discord Webhook URL（[作成方法](https://support.discord.com/hc/en-us/articles/228383668)） | オプション |
| `DISCORD_BOT_TOKEN` | Discord Bot Token（Webhook とどちらか一方を選択） | オプション |
| `DISCORD_MAIN_CHANNEL_ID` | Discord Channel ID（Bot 使用時に必須） | オプション |
| `DISCORD_INTERACTIONS_PUBLIC_KEY` | Discord 公開鍵（受信 Interaction/Webhook の署名検証時のみ必須） | オプション |
| `SLACK_BOT_TOKEN` | Slack Bot Token（推奨、画像アップロードに対応。両方設定時は Webhook より優先） | オプション |
| `SLACK_CHANNEL_ID` | Slack Channel ID（Bot 使用時に必須） | オプション |
| `SLACK_WEBHOOK_URL` | Slack Incoming Webhook URL（テキストのみ、画像非対応） | オプション |
| `EMAIL_SENDER` | 送信者メールアドレス（例: `xxx@qq.com`） | オプション |
| `EMAIL_PASSWORD` | メールの認証コード（ログインパスワードではありません） | オプション |
| `EMAIL_RECEIVERS` | 受信者メールアドレス（カンマ区切り、空欄の場合は自分自身に送信） | オプション |
| `EMAIL_SENDER_NAME` | 送信者の表示名 | オプション |
| `STOCK_GROUP_N` / `EMAIL_GROUP_N` | メールルーティンググループ（Issue #268）: `STOCK_GROUP_N` は `STOCK_LIST` のサブセットであるべきで、メール受信者にのみ影響し、分析範囲や他チャネルには影響しません | オプション |
| `PUSHPLUS_TOKEN` | PushPlus Token（[取得はこちら](https://www.pushplus.plus)、中国向けプッシュサービス） | オプション |
| `SERVERCHAN3_SENDKEY` | ServerChan v3 Sendkey（[取得はこちら](https://sc3.ft07.com/)、モバイルアプリのプッシュサービス） | オプション |
| `ASTRBOT_URL` | AstrBot Webhook URL | オプション |
| `ASTRBOT_TOKEN` | AstrBot の Bearer Token（任意） | オプション |
| `NTFY_URL` | ntfy トピックの完全なエンドポイント。トピックパスを含む必要があります。例: `https://ntfy.sh/my-topic` | オプション |
| `NTFY_TOKEN` | ntfy の Bearer Token（任意） | オプション |
| `GOTIFY_URL` | Gotify サーバーのベース URL。`/message` を含めないでください。送信側が `/message` を付加します | オプション |
| `GOTIFY_TOKEN` | `X-Gotify-Key` ヘッダーで送信される Gotify アプリケーショントークン | オプション |
| `CUSTOM_WEBHOOK_URLS` | カスタム Webhook（DingTalk などに対応、カンマ区切り） | オプション |
| `CUSTOM_WEBHOOK_BEARER_TOKEN` | カスタム Webhook 用の Bearer Token（認証付き Webhook 用） | オプション |
| `CUSTOM_WEBHOOK_BODY_TEMPLATE` | AstrBot、NapCat、または特殊なペイロードを持つセルフホストサービス向けのカスタム Webhook JSON ボディテンプレート | オプション |
| `WEBHOOK_VERIFY_SSL` | この設定を読み取る Webhook 形式の通知リクエストに対する HTTPS 証明書検証（デフォルト true）。自己署名証明書の場合は false に設定。警告: 無効化には深刻なセキュリティリスク（MITM）があり、信頼できる内部ネットワークでのみ使用してください | オプション |

> *注: 少なくとも 1 つのチャネルを設定してください。複数のチャネルはすべて通知を受信します。起動時の検証では、ペアになっていない Telegram／メールのフィールドや、`http://` または `https://` で始まらない一般的な Webhook URL を報告します。
>
> 本リポジトリのデフォルト `00-daily-analysis.yml` は固定の Secret／Variable 名のみをエクスポートします。`STOCK_GROUP_1` や `EMAIL_GROUP_1` のような任意の番号付き環境変数はジョブに自動注入されないため、Fork したワークフローの `env:` マッピングを明示的に拡張しない限り、株式ワークフローでグループ化メールルーティングは利用できません。Actions は現在、`CUSTOM_WEBHOOK_BODY_TEMPLATE`、`WEBHOOK_VERIFY_SSL`、`FEISHU_WEBHOOK_SECRET`、`FEISHU_WEBHOOK_KEYWORD`、`PUSHPLUS_TOPIC`、`NTFY_URL`、`NTFY_TOKEN`、`GOTIFY_URL`、`GOTIFY_TOKEN`、P3 通知ルートキー、および P4 通知ノイズ制御キーをマッピングします。`MARKDOWN_TO_IMAGE_CHANNELS` と `MERGE_EMAIL_NOTIFICATION` は、デフォルトワークフローのマッピング外にある動作トグルのままです。

#### プッシュ動作の設定

| Secret 名 | 説明 | 必須 |
|------------|------|:----:|
| `SINGLE_STOCK_NOTIFY` | 単一銘柄プッシュモード: `true` に設定すると各銘柄の分析直後に即時プッシュします | オプション |
| `REPORT_TYPE` | レポートタイプ: `simple`（簡潔）、`full`（完全）、`brief`（3〜5 文）。Docker 推奨: `full` | オプション |
| `REPORT_LANGUAGE` | レポートの出力言語: `zh`（デフォルト、中国語）／`en`（英語）。プロンプト指示、テンプレート、通知フォールバック、Web レポートビューの固定文言も更新します。同梱の `00-daily-analysis.yml` は既にこの変数をマッピングしているため、Actions Secrets/Variables で設定すればそのまま動作します | オプション |
| `REPORT_SHOW_LLM_MODEL` | 通知レポートのフッターに分析で使用した LLM モデルを表示するかどうか。デフォルトは `true`。`false` に設定するとランタイムのモデルメタデータを隠します。このスイッチは表示のみに影響し、provider／model／Base URL、LiteLLM ルーティング、ランタイムのモデル保存／移行／クリーンアップ動作は変更しません。 | オプション |
| `REPORT_TEMPLATES_DIR` | Jinja2 テンプレートディレクトリ（プロジェクトルートからの相対パス、デフォルト `templates`） | オプション |
| `REPORT_RENDERER_ENABLED` | Jinja2 テンプレートレンダリングを有効化（デフォルト `false`、リグレッションなし） | オプション |
| `REPORT_INTEGRITY_ENABLED` | レポート整合性チェックを有効化。フィールド欠落時にリトライまたはプレースホルダーを使用（デフォルト `true`） | オプション |
| `REPORT_INTEGRITY_RETRY` | 整合性リトライ回数（デフォルト `1`、`0` = プレースホルダーのみ） | オプション |
| `REPORT_HISTORY_COMPARE_N` | 履歴シグナル比較件数。`0` で無効（デフォルト）、`>0` で有効 | オプション |
| `ANALYSIS_DELAY` | API レート制限を回避するための、銘柄分析と大引け振り返りの間の遅延（秒）。例: `10` | オプション |
| `SAVE_CONTEXT_SNAPSHOT` | 分析履歴の `context_snapshot` を永続化するかどうか。デフォルトは `true`。`false` に設定するか `--no-context-snapshot` を使うと、完全なスナップショットの永続化を停止します | オプション |
| `NOTIFICATION_REPORT_CHANNELS` | 単一銘柄、日次集計、大引け振り返り、マージプッシュ、Feishu ドキュメント成功通知のレポートルートチャネル。空の場合は設定済みの全チャネル | オプション |
| `NOTIFICATION_ALERT_CHANNELS` | EventMonitor 通知のアラートルートチャネル。空の場合は設定済みの全チャネル | オプション |
| `NOTIFICATION_SYSTEM_ERROR_CHANNELS` | 予約済みの system_error ルートチャネル。P3 では自動的なシステムエラー生成元は追加されません。空の場合は設定済みの全チャネル | オプション |
| `NOTIFICATION_DEDUP_TTL_SECONDS` | 重複排除 TTL（秒）。`0` で重複排除を無効化。同一の安定した重複排除キーは TTL 内で 1 回のみ送信されます | オプション |
| `NOTIFICATION_COOLDOWN_SECONDS` | クールダウンウィンドウ（秒）。`0` でクールダウンを無効化。同一のクールダウンキーはウィンドウ内でレート制限されます | オプション |
| `NOTIFICATION_QUIET_HOURS` | サイレント時間帯を `HH:MM-HH:MM` 形式で指定。日をまたぐ範囲に対応。空でサイレント時間を無効化 | オプション |
| `NOTIFICATION_TIMEZONE` | サイレント時間用の IANA タイムゾーン。例: `Asia/Shanghai`。空の場合は `TZ` またはローカルシステムのタイムゾーンに従います | オプション |
| `NOTIFICATION_MIN_SEVERITY` | 最小重大度: `info`、`warning`、`error`、`critical`。空で現在の動作を維持 | オプション |
| `NOTIFICATION_DAILY_DIGEST_ENABLED` | 予約済みの日次ダイジェストフラグ。現在の実装ではダイジェストの送信や永続化は行いません | オプション |

> 互換性に関する注記: `REPORT_SHOW_LLM_MODEL` は従来のデフォルト表示動作（`true`）を維持し、レポートフッターのレンダリングのみを変更します。provider／model／Base URL、LiteLLM ルーティング、ランタイムのモデル永続化／移行／クリーンアップのセマンティクスは変更しません。ロールバックは変数を削除するか `true` に戻します。

> `REPORT_LANGUAGE` はレポート本文とレポートページの固定文言にのみ影響します。Web UI のシェル言語（ナビゲーション、ログイン、設定、シェルラベル、共有コントロール）は意図的に独立しており、ブラウザの `localStorage` に `dsa.uiLanguage` として保存されます。
> UI 言語の解決順序は次のとおりです: localStorage の明示的な値（`zh` または `en`） -> ブラウザの言語（`navigator.languages` / `navigator.language`） -> デフォルト `zh`。

#### その他の設定

| Secret 名 | 説明 | 必須 |
|------------|------|:----:|
| `STOCK_LIST` | ウォッチリストの銘柄コード。例: `600519,300750,002594` | ✅ |
| `ANSPIRE_API_KEYS` | [Anspire AI Search](https://aisearch.anspire.cn/)。中国語コンテンツに最適化されており、同じキーは Anspire LLM フォールバックのシナリオにも使用できます（モデル例: `Doubao-Seed-2.0-lite`） | 推奨 |
| `SERPAPI_API_KEYS` | [SerpAPI](https://serpapi.com/baidu-search-api?utm_source=github_daily_stock_analysis) の検索エンジン結果。リアルタイムの金融ニュース用 | 推奨 |
| `TAVILY_API_KEYS` | [Tavily](https://tavily.com/) Search API（ニュース検索用） | オプション |
| `BOCHA_API_KEYS` | [Bocha Search](https://open.bocha.cn/) Web Search API（中国語検索に最適化、AI 要約に対応、複数キーはカンマ区切り） | オプション |
| `BRAVE_API_KEYS` | [Brave Search](https://brave.com/search/api/) API（プライバシー重視、米国株ニュースの補強、複数キーはカンマ区切り） | オプション |
| `MINIMAX_API_KEYS` | [MiniMax](https://platform.minimax.io/) Coding Plan の Web 検索（構造化された検索結果） | オプション |
| `SEARXNG_BASE_URLS` | SearXNG のセルフホストインスタンス（無料枠のフォールバック、settings.yml で format: json を有効化）。空の場合はアプリが公開インスタンスを自動検出します | オプション |
| `SEARXNG_PUBLIC_INSTANCES_ENABLED` | `SEARXNG_BASE_URLS` が空のとき、`searx.space` から公開 SearXNG インスタンスを自動検出（デフォルト `true`） | オプション |
| `TUSHARE_TOKEN` | [Tushare Pro](https://tushare.pro/weborder/#/login?reg=834638) Token | オプション |
| `TICKFLOW_API_KEY` | CN 大引け振り返りの指数強化用 [TickFlow](https://tickflow.org) API キー。プランがユニバースクエリに対応している場合は市場ブレッドスでも TickFlow を使用します | オプション |

#### ✅ 最小構成の例

すぐに始めるには、最低限以下が必要です:

1. **AI モデル**: `ANSPIRE_API_KEYS`（LLM と検索を 1 キーで）、`AIHUBMIX_KEY`（複数モデルファミリーを 1 キーで）、`GEMINI_API_KEY`、または `OPENAI_API_KEY`
2. **通知チャネル**: 少なくとも 1 つ。例: `WECHAT_WEBHOOK_URL` または `EMAIL_SENDER` + `EMAIL_PASSWORD`
3. **銘柄リスト**: `STOCK_LIST`（必須）
4. **検索 API**: `ANSPIRE_API_KEYS` または `SERPAPI_API_KEYS`（ニュースとセンチメント検索のために推奨）

> この 4 項目を設定すれば準備完了です！

### 3. Actions を有効化する

1. Fork したリポジトリに移動します
2. 上部の `Actions` タブをクリックします
3. プロンプトが表示されたら、`I understand my workflows, go ahead and enable them` をクリックします

### 4. 手動テスト

1. `Actions` タブに移動します
2. 左側で `Daily Stock Analysis` ワークフローを選択します
3. 右側の `Run workflow` ボタンをクリックします
4. 実行モードを選択します
5. 緑色の `Run workflow` をクリックして確定します

### 5. 完了！

デフォルトのスケジュール: 毎週平日 **18:00（北京時間）** に自動実行されます。

---

## 環境変数一覧（完全版）

### AI モデルの設定

> 詳細: [LLM Config Guide](LLM_CONFIG_GUIDE_EN.md)（3 層構成、チャネル、Vision、Agent、トラブルシューティング）。
> Issue #1306 の互換性に関する注記: この変更は既存の大引け振り返り出力を履歴パス経由で永続化・公開するのみで、モデル名、provider、base URL、LiteLLM クリーンアップルール、`.env` ランタイム移行のセマンティクスは変更しません。ロールバックはこの変更セットを取り消すことです。ランタイム互換性のリファレンスは `requirements.txt`（`litellm` 制約）、`docs/LLM_CONFIG_GUIDE_EN.md`、および `tests/test_analysis_api_contract.py`、`tests/test_analysis_history.py`、`tests/test_market_review.py` のリグレッションテストです。公式リファレンス: [LiteLLM OpenAI-compatible](https://docs.litellm.ai/docs/providers/openai_compatible)、[OpenAI Chat Completion API](https://platform.openai.com/docs/api-reference/chat)。

| 変数 | 説明 | デフォルト | 必須 |
|--------|------|--------|:----:|
| `LITELLM_MODEL` | プライマリモデル。形式は `provider/model`（例: `gemini/gemini-3.1-pro-preview`）。推奨 | - | No |
| `AGENT_LITELLM_MODEL` | Agent 専用のプライマリモデル（任意）。空の場合はプライマリモデルを継承し、プレフィックスなしの名前は `openai/<model>` に正規化されます | - | No |
| `LITELLM_FALLBACK_MODELS` | フォールバックモデル（カンマ区切り） | - | No |
| `LLM_CHANNELS` | チャネル名（カンマ区切り）。`LLM_{NAME}_*` と併用。[LLM Config Guide](LLM_CONFIG_GUIDE_EN.md) を参照 | - | No |
| `LITELLM_CONFIG` | 高度なモデルルーティング YAML のパス（上級者向け） | - | No |
| `ANSPIRE_API_KEYS` | [Anspire](https://open.anspire.cn/?share_code=QFBC0FYC) API キー。LLM ゲートウェイと検索を 1 キーで | - | オプション |
| `AIHUBMIX_KEY` | [AIHubMix](https://aihubmix.com/?aff=CfMq) API キー。複数モデルファミリーを 1 キーで | - | オプション |
| `GEMINI_API_KEY` | Google Gemini API キー | - | オプション |
| `GEMINI_MODEL` | プライマリモデル名（レガシー、`LITELLM_MODEL` を推奨） | `gemini-3.1-pro-preview` | No |
| `GEMINI_MODEL_FALLBACK` | フォールバックモデル（レガシー） | `gemini-3-flash-preview` | No |
| `ANTHROPIC_API_KEY` | Anthropic Claude API キー | - | オプション |
| `OPENAI_API_KEY` | OpenAI 互換 API キー | - | オプション |
| `OPENAI_BASE_URL` | OpenAI 互換 API エンドポイント | - | オプション |
| `OLLAMA_API_BASE` | Ollama ローカルサービスのアドレス（例: `http://localhost:11434`）。[LLM Config Guide](LLM_CONFIG_GUIDE_EN.md) を参照 | - | オプション |
| `OPENAI_MODEL` | OpenAI モデル名（レガシー） | `gpt-5.5` | オプション |

> *注: `ANSPIRE_API_KEYS`、`AIHUBMIX_KEY`、`GEMINI_API_KEY`、`ANTHROPIC_API_KEY`、`OPENAI_API_KEY`、`OLLAMA_API_BASE`、または `LLM_CHANNELS` / `LITELLM_CONFIG` のうち少なくとも 1 つを設定してください。`ANSPIRE_API_KEYS` と `AIHUBMIX_KEY` は `OPENAI_BASE_URL` なしで自動的に適合されます。

### 通知チャネルの設定

通知のベースライン、診断、デプロイの注意事項については [Notification Guide](notifications.md) を参照してください。

| 変数 | 説明 | 必須 |
|--------|------|:----:|
| `WECHAT_WEBHOOK_URL` | WeChat Work Bot の Webhook URL | オプション |
| `FEISHU_WEBHOOK_URL` | Feishu Bot の Webhook URL | オプション |
| `FEISHU_WEBHOOK_SECRET` | Feishu Bot の署名シークレット（署名セキュリティが有効な Webhook Bot のみ） | オプション |
| `FEISHU_WEBHOOK_KEYWORD` | Feishu Bot のキーワード（キーワードセキュリティが有効な Webhook Bot のみ） | オプション |
| `TELEGRAM_BOT_TOKEN` | Telegram Bot Token | オプション |
| `TELEGRAM_CHAT_ID` | Telegram Chat ID | オプション |
| `TELEGRAM_MESSAGE_THREAD_ID` | Telegram トピック ID | オプション |
| `DISCORD_WEBHOOK_URL` | Discord Webhook URL | オプション |
| `DISCORD_BOT_TOKEN` | Discord Bot Token（Webhook とどちらか一方を選択） | オプション |
| `DISCORD_MAIN_CHANNEL_ID` | Discord Channel ID（Bot 使用時に必須） | オプション |
| `DISCORD_INTERACTIONS_PUBLIC_KEY` | Discord 公開鍵（受信 Interaction/Webhook の署名検証時のみ必須） | オプション |
| `DISCORD_MAX_WORDS` | Discord の文字数制限（未アップグレードのサーバーではデフォルト 2000） | オプション |
| `SLACK_BOT_TOKEN` | Slack Bot Token（推奨、画像アップロードに対応。両方設定時は Webhook より優先） | オプション |
| `SLACK_CHANNEL_ID` | Slack Channel ID（Bot 使用時に必須） | オプション |
| `SLACK_WEBHOOK_URL` | Slack Incoming Webhook URL（テキストのみ、画像非対応） | オプション |
| `EMAIL_SENDER` | 送信者メールアドレス | オプション |
| `EMAIL_PASSWORD` | メールの認証コード（ログインパスワードではありません） | オプション |
| `EMAIL_RECEIVERS` | 受信者メールアドレス（カンマ区切り、空欄の場合は自分自身に送信） | オプション |
| `EMAIL_SENDER_NAME` | 送信者の表示名 | オプション |
| `STOCK_GROUP_N` / `EMAIL_GROUP_N` | メールルーティンググループ（Issue #268）: `STOCK_GROUP_N` は `STOCK_LIST` の範囲内にとどめるべきで、メール受信者のみを変更します | オプション |
| `CUSTOM_WEBHOOK_URLS` | カスタム Webhook（カンマ区切り） | オプション |
| `CUSTOM_WEBHOOK_BEARER_TOKEN` | カスタム Webhook の Bearer Token | オプション |
| `WEBHOOK_VERIFY_SSL` | この設定を読み取る Webhook 形式の通知リクエストに対する HTTPS 証明書検証（デフォルト true）。自己署名証明書の場合は false に設定。警告: 無効化には深刻なセキュリティリスクがあります | オプション |
| `PUSHOVER_USER_KEY` | Pushover User Key | オプション |
| `PUSHOVER_API_TOKEN` | Pushover API Token | オプション |
| `NTFY_URL` | ntfy トピックの完全なエンドポイント。トピックパスを含む必要があります。例: `https://ntfy.sh/my-topic` | オプション |
| `NTFY_TOKEN` | ntfy の Bearer Token（任意） | オプション |
| `GOTIFY_URL` | Gotify サーバーのベース URL。`/message` を含めません | オプション |
| `GOTIFY_TOKEN` | `X-Gotify-Key` で送信される Gotify アプリケーショントークン | オプション |
| `PUSHPLUS_TOKEN` | PushPlus Token（中国向けプッシュサービス） | オプション |
| `SERVERCHAN3_SENDKEY` | ServerChan v3 Sendkey | オプション |
| `ASTRBOT_URL` | AstrBot Webhook URL | オプション |
| `ASTRBOT_TOKEN` | AstrBot の Bearer Token（任意） | オプション |
| `NOTIFICATION_REPORT_CHANNELS` | レポートルートチャネル（カンマ区切り）。許可される値: wechat,feishu,telegram,email,pushover,ntfy,gotify,pushplus,serverchan3,custom,discord,slack,astrbot | オプション |
| `NOTIFICATION_ALERT_CHANNELS` | アラートルートチャネル（カンマ区切り）。空の場合は設定済みの全チャネルを維持 | オプション |
| `NOTIFICATION_SYSTEM_ERROR_CHANNELS` | 予約済みの system_error ルートチャネル（カンマ区切り）。空の場合は設定済みの全チャネルを維持 | オプション |
| `NOTIFICATION_DEDUP_TTL_SECONDS` | 重複排除 TTL（秒）。`0` で重複排除を無効化 | オプション |
| `NOTIFICATION_COOLDOWN_SECONDS` | クールダウンウィンドウ（秒）。`0` でクールダウンを無効化 | オプション |
| `NOTIFICATION_QUIET_HOURS` | サイレント時間帯を `HH:MM-HH:MM` 形式で指定。日をまたぐ範囲に対応 | オプション |
| `NOTIFICATION_TIMEZONE` | サイレント時間のタイムゾーン。例: `Asia/Shanghai`。空の場合は `TZ` またはローカルシステムのタイムゾーンに従います | オプション |
| `NOTIFICATION_MIN_SEVERITY` | 最小重大度: info、warning、error、critical。空で現在の動作を維持 | オプション |
| `NOTIFICATION_DAILY_DIGEST_ENABLED` | 予約済みの日次ダイジェストフラグ。まだダイジェストは送信しません | オプション |

> 注: デフォルトの `00-daily-analysis.yml` GitHub Actions ワークフローは固定の変数名のみをマッピングします。`STOCK_GROUP_N` / `EMAIL_GROUP_N` のような任意の番号付き変数を自動的にインポートすることはありません。したがってこの機能は、ローカル `.env`、Docker、またはそれらの変数を明示的に注入する任意のランタイムで機能します。

#### Feishu クラウドドキュメントの設定（オプション、メッセージ切り捨て問題を解決）

| 変数 | 説明 | 必須 |
|--------|------|:----:|
| `FEISHU_APP_ID` | Feishu App ID | オプション |
| `FEISHU_APP_SECRET` | Feishu App Secret | オプション |
| `FEISHU_FOLDER_TOKEN` | Feishu クラウドドライブのフォルダトークン | オプション |

> Feishu クラウドドキュメントのセットアップ手順:
> 1. [Feishu Developer Console](https://open.feishu.cn/app) でアプリを作成
> 2. GitHub Secrets を設定
> 3. グループを作成し、アプリの Bot を追加
> 4. クラウドドライブのフォルダにグループをコラボレーターとして追加（管理権限付き）
>
> 注: `FEISHU_APP_ID` / `FEISHU_APP_SECRET` は Feishu アプリモード、クラウドドキュメント、または Stream Bot モード用です。これら単体ではグループ Webhook 通知を有効化しません。シンプルなグループプッシュ通知には、まず `FEISHU_WEBHOOK_URL` を使用してください。
>
> 補足: `FEISHU_APP_ID`、`FEISHU_APP_SECRET`、`FEISHU_CHAT_ID` をすべて設定すると、グループ Webhook に依存せず Feishu App Bot のアクティブ通知チャネルを有効化できます。`FEISHU_RECEIVE_ID_TYPE` のデフォルトは `chat_id` で、P2P 配信には `open_id` を設定します。これは Feishu OpenAPI Bot のセッションルートを使用し、グループ Webhook パスから独立しています。

### 検索サービスの設定

| 変数 | 説明 | 必須 |
|--------|------|:----:|
| `ANSPIRE_API_KEYS` | Anspire Open API キー（検索と LLM フォールバック例で共有。可用性はアカウント／モデルの権限に依存し、A 株分析を効果的に強化できます） | 推奨 |
| `SERPAPI_API_KEYS` | SerpAPI の検索エンジン結果。リアルタイムの金融ニュース用 | 推奨 |
| `TAVILY_API_KEYS` | Tavily Search API キー | オプション |
| `BOCHA_API_KEYS` | Bocha Search API キー（中国語に最適化） | オプション |
| `BRAVE_API_KEYS` | Brave Search API キー（米国株に最適化） | オプション |
| `MINIMAX_API_KEYS` | MiniMax Coding Plan の Web 検索（構造化結果） | オプション |
| `SOCIAL_SENTIMENT_API_KEY` | 株式センチメント API キー（Reddit / X / Polymarket、米国株向けオプション） | オプション |
| `SOCIAL_SENTIMENT_API_URL` | 株式センチメント API エンドポイント（デフォルト `https://api.adanos.org`） | オプション |
| `SEARXNG_BASE_URLS` | SearXNG のセルフホストインスタンス（無料枠のフォールバック、settings.yml で format: json を有効化）。空の場合はアプリが公開インスタンスを自動検出します | オプション |
| `SEARXNG_PUBLIC_INSTANCES_ENABLED` | `SEARXNG_BASE_URLS` が空のとき、`searx.space` から公開 SearXNG インスタンスを自動検出（デフォルト `true`） | オプション |

> 動作に関する注記: 検索とソーシャルセンチメントはオプションの強化サービスです。いずれかのサービスの初期化に失敗した場合、システムは警告をログに記録し、そのステージをスキップして優雅にデグレードします。コア分析フローはブロックされません。

### データソースの設定

| 変数 | 説明 | デフォルト | 必須 |
|--------|------|--------|:----:|
| `TUSHARE_TOKEN` | Tushare Pro Token | - | オプション |
| `TICKFLOW_API_KEY` | TickFlow API キー。設定時、CN 大引け振り返りの指数は TickFlow を優先し、市場ブレッドスはプランがユニバースクエリに対応している場合のみ優先します | - | オプション |
| `ENABLE_REALTIME_QUOTE` | リアルタイム相場を有効化（無効の場合、過去の終値を分析に使用） | `true` | オプション |
| `ENABLE_REALTIME_TECHNICAL_INDICATORS` | 日中リアルタイムテクニカル: 有効時はリアルタイム価格で MA5/MA10/MA20 と強気トレンドを計算（Issue #234）。無効時は前日終値を使用。 | `true` | オプション |
| `ENABLE_CHIP_DISTRIBUTION` | チップ分布分析を有効化（この API は不安定なため、クラウドデプロイでは無効を推奨）。GitHub Actions ユーザーは有効化するために Repository Variables で `ENABLE_CHIP_DISTRIBUTION=true` を設定する必要があります。ワークフローではデフォルト無効です。 | `true` | オプション |
| `ENABLE_EASTMONEY_PATCH` | Eastmoney API パッチ: Eastmoney API が頻繁に失敗する場合（例: RemoteDisconnected、接続クローズ）は `true` を推奨。NID トークンとランダムな User-Agent を注入してレート制限の確率を下げます。 | `false` | オプション |
| `REALTIME_SOURCE_PRIORITY` | リアルタイム相場ソースの優先順位（カンマ区切り）。例: `tencent,akshare_sina,efinance,akshare_em` | .env.example を参照 | オプション |
| `ENABLE_FUNDAMENTAL_PIPELINE` | ファンダメンタルズ集約のマスタースイッチ。無効時は `not_supported` ブロックのみを返し、元の分析パイプラインは変更しません。 | `true` | オプション |
| `FUNDAMENTAL_STAGE_TIMEOUT_SECONDS` | ファンダメンタルズステージの総レイテンシ予算（秒） | `8.0` | オプション |
| `FUNDAMENTAL_FETCH_TIMEOUT_SECONDS` | 単一の機能ソース呼び出しのタイムアウト（秒） | `3.0` | オプション |
| `FUNDAMENTAL_RETRY_MAX` | ファンダメンタルズ機能のリトライ回数（初回試行を含む） | `1` | オプション |
| `FUNDAMENTAL_CACHE_TTL_SECONDS` | ファンダメンタルズ集約のキャッシュ TTL（秒）。短いキャッシュで API の繰り返し取得を削減。 | `120` | オプション |
| `FUNDAMENTAL_CACHE_MAX_ENTRIES` | ファンダメンタルズキャッシュの最大エントリ数（TTL 内で時間順に退避） | `256` | オプション |

> **動作に関する注記:**
> - **A 株**: `valuation/growth/earnings/institution/capital_flow/dragon_tiger/boards` で集約された機能を返します。
> - **ETF**: 利用可能な項目を返し、欠落している機能を `not_supported` としてマークします。全体として元のフローには影響しません。
> - **米国株／香港株**: yfinance アダプター経由で `valuation/growth/earnings/belong_boards`（`info.sector`/`info.industry` 由来）を返します。`institution/capital_flow/dragon_tiger/boards` は、現時点でオフショアのデータフィードが存在しないため `not_supported` のままです。yfinance が利用できない、または空のペイロードを返す場合は完全な `not_supported` ブロックにフォールバックします。それでも fail-open です。
> - 例外はすべて fail-open ロジックを使用し、エラーをログに記録するだけで、メインのテクニカル／ニュース／チップパイプラインには影響しません。
> - **フィールド契約**:
>   - `fundamental_context.belong_boards` = 銘柄の関連ボードリスト。A 株は AkShare のボード所属、米国株／香港株は yfinance の `info.sector`/`info.industry` 由来。利用不可の場合は `[]`。
>   - `fundamental_context.boards.data` = `sector_rankings`（セクターの騰落ランキング、構造 `{top, bottom}`。現時点で米国株／香港株には提供されません）。
>   - `fundamental_context.earnings.data.financial_report.currency` = 財務諸表の通貨（`info.financialCurrency`。香港 ADR はここで CNY を報告することが一般的です）。
>   - `fundamental_context.earnings.data.dividend.currency` = 取引／配当の通貨（`info.currency`。香港 ADR は財務諸表通貨が CNY でもここでは HKD を使用します）。レンダラーは単一のグローバル通貨を仮定せず、各ブロック自身の通貨を読み取ります。
>   - `fundamental_context.earnings.data.dividend.ttm_dividend_yield_pct` = `ttm_cash_dividend_per_share / latest_price * 100`。両辺とも取引通貨。TTM 現金配当または最新価格が利用できない場合のみ、`info.trailingAnnualDividendYield`（小数）または `info.dividendYield`（既にパーセントのパススルー）にフォールバックします。
>   - `get_stock_info.belong_boards` = 個別銘柄が所属するセクターのリスト。
>   - `get_stock_info.boards` は互換性のためのエイリアスで、値は `belong_boards` と同一です（削除はメジャーバージョン更新でのみ検討）。
>   - `get_stock_info.sector_rankings` は `fundamental_context.boards.data` と一貫します。
>   - `AnalysisReport.details.belong_boards` = 構造化レポート詳細内の関連ボードリスト。
>   - `AnalysisReport.details.sector_rankings` = ボード連動表示のための構造化レポート詳細内のセクターランキング。
> - **セクターランキング** は固定のフォールバック順序を使用します: グローバル優先順位と一貫します。
> - **タイムアウト制御** は `best-effort` のソフトタイムアウトです: ステージは予算に基づいて素早くデグレードして実行を継続しますが、基盤となるサードパーティのネットワーク呼び出しのハード中断は保証しません。
> - `FUNDAMENTAL_STAGE_TIMEOUT_SECONDS=8.0` は新規追加されたファンダメンタルズステージの目標予算を示し、厳格なハード SLA ではありません。Windows、Docker、またはレート制限された無料データソースでは `12-15s` まで引き上げる場合があります。
> - ハード SLA については、タイムアウトタスクを強制終了するために、将来のバージョンで分離された子プロセス実行にアップグレードしてください。

### その他の設定

| 変数 | 説明 | デフォルト |
|--------|------|--------|
| `STOCK_LIST` | ウォッチリストの銘柄コード（カンマ区切り） | - |
| `MAX_WORKERS` | 並行スレッド数 | `3` |
| `MARKET_REVIEW_ENABLED` | 大引け振り返りを有効化 | `true` |
| `MARKET_REVIEW_REGION` | 大引け振り返りの対象地域: cn（A 株）、hk（香港株）、us（米国株）、both（3 市場すべて） | `cn` |
| `MARKET_REVIEW_COLOR_SCHEME` | 大引け振り返りの指数変動の色スタイル: `green_up` = 上昇が緑／下落が赤（デフォルト）、`red_up` = 上昇が赤／下落が緑 | `green_up` |
| `SCHEDULE_ENABLED` | 定時タスクを有効化 | `false` |
| `SCHEDULE_TIME` | 定時実行の時刻 | `18:00` |
| `SCHEDULE_RUN_IMMEDIATELY` | スケジューラモード起動時に一度すぐに実行する。未設定の場合はレガシーの `RUN_IMMEDIATELY` ランタイムオーバーライドに従い続けます | `true` |
| `RUN_IMMEDIATELY` | 非スケジューラ起動時に一度すぐに実行する。`SCHEDULE_RUN_IMMEDIATELY` が未設定の場合のレガシーフォールバックとしても機能します | `true` |
| `LOG_DIR` | ログディレクトリ | `./logs` |
| `SAVE_CONTEXT_SNAPSHOT` | 分析履歴の `context_snapshot` を永続化する。false の場合、新しい履歴レコードは enhanced_context、market_phase_summary、AnalysisContextPack 概要、診断スナップショットを保存しませんが、現在実行のプロンプト要約は有効のままです | `true` |

> 動作に関する注記:
> - `TICKFLOW_API_KEY` が設定されている場合、CN 大引け振り返りはまずメイン指数で TickFlow を試します。市場ブレッドスは、現在の TickFlow プランがユニバースクエリに対応している場合のみ TickFlow を試します。
> - TickFlow の動作はキーベースではなく機能ベースです: 制限のあるプランでもメイン CN 指数を強化でき、`CN_Equity_A` ユニバースクエリに対応するプランは市場ブレッドスも強化します。
> - 公式クイックスタートは `quotes.get(universes=["CN_Equity_A"])` を記載していますが、オンラインのスモークテストで 2 つの追加的な実環境制約が確認されました: ユニバースアクセスはプラン権限に依存すること、そして `quotes.get(symbols=[...])` にはリクエストごとのシンボル数上限があることです。
> - TickFlow は現在 `change_pct` / `amplitude` を比率値として返します。この統合はそれらをプロジェクトのパーセント規約に正規化し、AkShare / Tushare / efinance のセマンティクスに一致させます。
> - スケジューラモードでは、ランタイム環境が `RUN_IMMEDIATELY` を明示的に設定しているが `SCHEDULE_RUN_IMMEDIATELY` を設定していない場合、スケジューラは永続化された `.env` エイリアス値に引き戻されるのではなく、レガシーのランタイムオーバーライドを継承し続けます。
> - CN 大引け振り返りレポートは現在、市場シグナル、指数詳細、セクター Top テーブル、ニュースカタリスト、翌セッション計画、リスクのセクションを持つ大引け後のワークステーションレイアウトを使用します。市場シグナルは、ターミナルや通知クライアント間で一貫してレンダリングされるよう、ブロックバーの代わりに `66/100 (constructive, risk-on)` のようなプレーンテキストのスコアを使用します。ニュースカタリストは、混在言語のノイズを減らすために、検索スニペットではなく見出し、ソース、リンクのみをリストします。データソースが欠落した場合は、影響を受けるブロックのみを省略または簡略化してデグレードします。
> - 個別銘柄の分析、リアルタイム相場の優先順位、セクターランキングのフォールバックは変更されません。

---

## Docker デプロイ

イメージは実行時に `/app/static` 配下のビルド済みフロントエンドアセットを使用するため、実行中の `server` コンテナは `apps/dsa-web` のソースツリーやランタイムの `npm` を必要としません。Docker デプロイ後に WebUI を開けない場合は、まずコンテナ内に `/app/static/index.html` が存在するか確認してください。

公式イメージレジストリ:

- GHCR: `ghcr.io/zhulinsen/daily_stock_analysis:<tag>`
- Docker Hub: `<DOCKERHUB_USERNAME>/daily_stock_analysis:<tag>`（公開者の `DOCKERHUB_USERNAME` シークレットで駆動。公式リリースは `zhulinsen/daily_stock_analysis` を使用）

### クイックスタート

```bash
# 1. リポジトリをクローン
git clone https://github.com/ZhuLinsen/daily_stock_analysis.git
cd daily_stock_analysis

# 2. 環境変数を設定
cp .env.example .env
vim .env  # API キーと設定を記入

# 3. コンテナを起動
docker-compose -f ./docker/docker-compose.yml up -d server     # Web サービスモード（推奨、API と WebUI を提供）
docker-compose -f ./docker/docker-compose.yml up -d analyzer   # 定時タスクモード
docker-compose -f ./docker/docker-compose.yml up -d            # 両方のモードを起動

# 4. WebUI にアクセス
# http://localhost:8000

# 5. ログを確認
docker-compose -f ./docker/docker-compose.yml logs -f server
```

### 公式イメージを直接実行する

ターゲットマシンにソースツリーを保持したくない場合は、公開イメージを直接実行できます:

```bash
# Web/API モード
docker pull zhulinsen/daily_stock_analysis:latest
docker run -d \
  --name dsa-server \
  --env-file .env \
  -p 8000:8000 \
  -v "$(pwd)/data:/app/data" \
  -v "$(pwd)/logs:/app/logs" \
  -v "$(pwd)/reports:/app/reports" \
  zhulinsen/daily_stock_analysis:latest \
  python main.py --serve-only --host 0.0.0.0 --port 8000

# 定時タスクモード
docker run -d \
  --name dsa-analyzer \
  --env-file .env \
  -v "$(pwd)/data:/app/data" \
  -v "$(pwd)/logs:/app/logs" \
  -v "$(pwd)/reports:/app/reports" \
  zhulinsen/daily_stock_analysis:latest
```

固定されたデプロイやロールバックを容易にするには、`latest` を `v3.13.0` のような具体的なバージョンタグに置き換えてください。

### 実行モードの説明

| コマンド | 説明 | ポート |
|------|------|------|
| `docker-compose -f ./docker/docker-compose.yml up -d server` | Web サービスモード。API と WebUI を提供 | 8000 |
| `docker-compose -f ./docker/docker-compose.yml up -d analyzer` | 定時タスクモード。毎日自動実行 | - |
| `docker-compose -f ./docker/docker-compose.yml up -d` | 両方のモードを同時に起動 | 8000 |

### Docker Compose の設定

`docker-compose.yml` は YAML アンカーを使用して設定を再利用します:

```yaml
version: '3.8'

x-common: &common
  build:
    context: ..
    dockerfile: docker/Dockerfile
  restart: unless-stopped
  env_file:
    - ../.env
  environment:
    - TZ=Asia/Shanghai
  volumes:
    - ../data:/app/data
    - ../logs:/app/logs
    - ../reports:/app/reports
    - ../strategies:/app/strategies:ro

services:
  # 定時タスクモード
  analyzer:
    <<: *common
    container_name: stock-analyzer

  # FastAPI モード
  server:
    <<: *common
    container_name: stock-server
    command: ["python", "main.py", "--serve-only", "--host", "0.0.0.0", "--port", "${API_PORT:-8000}"]
    ports:
      - "${API_PORT:-8000}:${API_PORT:-8000}"
```

### `.env` とボリュームマッピング

`docker run` と Compose の両方で、起動時の環境注入とランタイムのファイル書き込みを分離しておきます:

- 環境注入: `--env-file .env` または Compose の `env_file`
  これは `.env` のキー/値ペアをコンテナプロセスの環境に渡します。
- ランタイム設定の書き込み: ホストの `.env` を単一ファイルとしてコンテナの `.env` パスにバインドマウントしないでください。Docker はターゲットをマウントポイントとして扱うため、設定保存時に使用される `os.replace()` のアトミック更新が `Device or resource busy` で失敗することがあります。代替のインプレース書き込みも権限で失敗することがあります。

デフォルトの Compose と `docker run` の例は、起動設定の注入に `env_file` / `--env-file` のみを使用し、ホストの `.env` ファイルをコンテナにマウントしなくなりました。アクティブな `.env` ファイルにキーが含まれていない場合、WebUI の設定ページは起動時に注入されたプロセス環境変数から同じキーを表示するフォールバックを行うため、Docker ユーザーは先にインポートしなくても注入された設定を確認できます。生の `.env` エクスポートには引き続きアクティブな設定ファイルの内容のみが含まれます。

WebUI から保存されたランタイム設定は、デフォルトでコンテナローカルの設定ファイルに書き込まれ、ホストの `.env` への書き戻しとは異なります。コンテナを削除または再作成した後も、起動時には注入された `.env` ファイルが使用されます。永続的なランタイム設定が必要な場合は、単一ファイルの `.env` バインドマウントを使う代わりに、`ENV_FILE` を `/app/data/runtime.env` のような書き込み可能なデータボリュームのファイルに向けてください。なお、起動時の `env_file`、`--env-file`、`docker run -e`、または Compose の `environment:` に同名の値が残っていると、再起動時にランタイムファイルを上書きできます。WebUI で保存した値を優先させたい場合は、それらの起動時オーバーライドを更新または削除してください。

推奨されるホストマッピング:

- `./data:/app/data` ランタイムデータとデータベースファイル用
- `./logs:/app/logs` ログ用
- `./reports:/app/reports` 生成されたレポート用
- `./strategies:/app/strategies:ro` カスタムストラテジー YAML ファイル用

公式 Docker イメージは起動時に `/app/data`、`/app/logs`、`/app/reports` のマウントを自動的に作成して所有権を修正し、その後コンテナ内の非 root の `dsa` ユーザー（UID/GID `1000:1000`）に権限を降格します。通常の Docker / Compose デプロイでは、ホスト側で手動の `chown` や `chmod` は不要です。

`--user` または Compose の `user:` でランタイムユーザーを上書きする場合、読み取り専用マウント、rootless Docker、NFS、または `chown` をブロックする別のストレージ環境を使用する場合、自動修復は適用されないことがあります。その場合は、実際のランタイムユーザーが `data`、`logs`、`reports` に書き込めることを確認するか、書き込み可能なボリュームを使用してください。

オプションの静的アセットオーバーライド:

- `./static:/app/static:ro`

### よく使うコマンド

```bash
# 実行状態を確認
docker-compose -f ./docker/docker-compose.yml ps

# ログを確認
docker-compose -f ./docker/docker-compose.yml logs -f server

# サービスを停止
docker-compose -f ./docker/docker-compose.yml down

# イメージを再ビルド（コード更新後）
docker-compose -f ./docker/docker-compose.yml build --no-cache
docker-compose -f ./docker/docker-compose.yml up -d server
```

### 手動イメージビルド

```bash
docker build -f docker/Dockerfile -t stock-analysis .
docker run -d \
  --name dsa-server-local \
  --env-file .env \
  -p 8000:8000 \
  -v "$(pwd)/data:/app/data" \
  -v "$(pwd)/logs:/app/logs" \
  -v "$(pwd)/reports:/app/reports" \
  stock-analysis \
  python main.py --serve-only --host 0.0.0.0 --port 8000
```

---

## ローカルデプロイ

### 依存関係のインストール

```bash
# Python 3.10+ 推奨
pip install -r requirements.txt

# または conda を使用
conda create -n stock python=3.10
conda activate stock
pip install -r requirements.txt
```

Windows PowerShell で、Python や pip が依然としてシステムのデフォルトコードページを使用している場合は、最初の依存関係インストールや環境チェックの前に UTF-8 を有効化してください。これにより、ターミナル出力やサードパーティツールが非 ASCII テキストで失敗するのを防ぎます:

```powershell
$env:PYTHONUTF8='1'
$env:PYTHONIOENCODING='utf-8'
python -m pip install -r requirements.txt
python scripts/check_env.py --config
```

### コマンドライン引数

```bash
python main.py                        # 完全分析（銘柄 + 大引け振り返り）
python main.py --market-review        # 大引け振り返りのみ
python main.py --no-market-review     # 銘柄分析のみ
python main.py --stocks 600519,300750 # 銘柄を指定
python main.py --dry-run              # データ取得のみ、AI 分析なし
python main.py --no-notify            # 通知を送信しない
python main.py --schedule             # 定時タスクモード
python main.py --debug                # デバッグモード（詳細ログ）
python main.py --workers 5            # 並行数を指定
```

---

## 定時タスクの設定

### GitHub Actions のスケジュール

`.github/workflows/00-daily-analysis.yml` を編集します:

```yaml
schedule:
  # UTC 時間、北京時間 = UTC + 8
  - cron: '0 10 * * 1-5'   # 月曜から金曜 18:00（北京時間）
```

よく使う時刻のリファレンス:

| 北京時間 | UTC cron 式 |
|---------|----------------|
| 09:30 | `'30 1 * * 1-5'` |
| 12:00 | `'0 4 * * 1-5'` |
| 15:00 | `'0 7 * * 1-5'` |
| 18:00 | `'0 10 * * 1-5'` |
| 21:00 | `'0 13 * * 1-5'` |

### ローカル定時タスク

```bash
# スケジュールモードを起動（デフォルト 18:00 実行）
python main.py --schedule

# または crontab を使用
crontab -e
# 追加: 0 18 * * 1-5 cd /path/to/project && python main.py
```

> 注: スケジュールモードは各実行の前に保存された `STOCK_LIST` を再読み込みします。`--stocks` も渡した場合、将来の定時実行が起動時のスナップショットに固定されることはありません。一時的な銘柄リストを分析したい場合は、通常の単発実行を使用してください。
>
> `python main.py --schedule`、`python main.py --serve --schedule`、または同等のローカルモードで組み込みスケジューラを起動した場合、WebUI から新しい `SCHEDULE_TIME` を保存すると、プロセスを再起動せずに次回のスケジューラポーリングで日次ジョブが再バインドされます。以前のトリガー時刻は、新しいものと並存させずに削除されます。

### Market Phase ベースライン（Issue #1386 P0）

P0 は内部の market-phase 推論ベースラインを追加するのみです。既存の日次大引け後レポート、取引日スキップ動作、有効取引日の解決、API、Web、Bot、Agent、または GitHub Actions のデフォルトは変更しません。フェーズ推論は P1+ のコンテキスト契約の準備です。`exchange-calendars` が利用できない、またはカレンダー参照が失敗した場合、フェーズは `unknown` を返します。既存の取引日フィルターと有効日ヘルパーは、現在の fail-open 動作を維持します。

フェーズラベルは通常セッションの状態を表します:

| フェーズ | 意味 |
| --- | --- |
| `premarket` | 通常セッション開始前。時間外相場が取得されたことを意味しません |
| `intraday` | 通常セッション内かつ昼休みや引け間際ウィンドウの外 |
| `lunch_break` | 市場カレンダーが提供する昼休みウィンドウ。昼休みのない市場はこのフェーズをスキップします |
| `closing_auction` | 引け間際のヒューリスティックウィンドウ: CN は 3 分、HK は 10 分、US は 5 分。これは完全な取引所オークションモデルではありません |
| `postmarket` | 通常セッション終了後。時間外相場が取得されたことを意味しません |
| `non_trading` | 現在の市場ローカル日付が取引セッションではない |
| `unknown` | 不明な市場、カレンダー利用不可、またはカレンダーエラーのため、フェーズを確実に推論できない |

現在のエントリポイントベースライン:

- 通常の銘柄分析、Agent 分析、Web 手動分析、Bot の `/analyze` / `/ask`、スケジュールモード、GitHub Actions は、引き続き既存の分析パスと大引け後リキャップの文言を使用します。P0 はプロンプトや出力スキーマを自動的に切り替えません。
- 大引け振り返りは引き続き `MARKET_REVIEW_REGION` と取引日フィルタリングに従います。market phase ラベルは消費しません。
- 混在市場のウォッチリストは、シンボルの市場ごとにフェーズを推論すべきです。集約レポートで一貫しないフェーズを表示することは P1+ に委ねられます。

既知の問題ベースライン:

- 日中の実行が、未完了の日中データを完全な日次リキャップのように記述する場合があります。
- 出力が、現在の日中観測ではなく「本日のリキャップ／明日に注目」に焦点を当て続ける場合があります。
- 相場のタイムスタンプ、ソース、キャッシュ、stale 状態が、まだフェーズコンテキストに統一されていません。
- 昼休み、引け間際、強制的な非取引日の実行が、まだプロンプトやレポート構造で明示されていません。

P0 はこのベースラインをパイプライン / Agent / API / Web / Bot に接続せず、レポートスキーマを変更せず、アラートのテクニカル指標の部分バー処理を変更せず、設定キーを追加しません。

### ランタイム Market Phase コンテキスト（Issue #1386 P1a）

P1a は内部の `market_phase_context` を構築し、通常の銘柄分析パイプライン、レガシー Agent コンテキスト、マルチエージェントの `ctx.meta` を通じて渡します。コンテキストには、市場、フェーズ、市場ローカル日付、有効な日足バー日付、取引日 / 市場開場 / 部分バーの三状態フラグ、ベストエフォートの開場/閉場分の推定、および `unknown_market`、`calendar_unavailable`、`calendar_error` などのデグレード警告コードが含まれます。

P1a 自体は、プロンプトの文言、API/Web/Bot のパラメータ、レポートスキーマ、安定した履歴/タスクステータスのメタデータ、または相場の鮮度/データ品質のセマンティクスを変更しません。通常の履歴スナップショットと Agent 履歴スナップショットは、このランタイム専用フィールドを除去します。P1b は永続的なメタデータとタスクステータス表示の契約を定義するために残されています。

### Market Phase 低感度メタデータ（Issue #1386 P1b）

P1b は P1a のランタイム `market_phase_context` を、安定した低感度の公開 `market_phase_summary` に射影し、`analysis_history.context_snapshot` のトップレベルに保存します。履歴詳細、同期分析レスポンス、完了した `/api/v1/analysis/status/{task_id}` レスポンスは、同じ market-phase メタデータを `report.meta.market_phase_summary` で返します。完了したタスクステータスはトップレベルの `TaskStatus` フィールドを追加せず、`status.result.report.meta.market_phase_summary` を通じてのみ公開します。

`market_phase_summary` には、市場、フェーズ、市場ローカル時刻、セッション日付、有効な日足バー日付、取引日 / 市場開場 / 部分バーのフラグ、開場/閉場分の推定、トリガーソース、分析意図、警告コードのみが含まれます。完全な `market_phase_context` を公開せず、相場の鮮度、フォールバック、stale、またはデータ品質スコアリングのフィールドも追加しません。`report.details.analysis_context_pack_overview` は引き続き #1389 の入力データブロック品質概要です。API の `details.context_snapshot` はトップレベルの `market_phase_summary` と `analysis_context_pack_overview` を除去するため、生のスナップショットはこれらの安定した公開フィールドを重複させません。`SAVE_CONTEXT_SNAPSHOT=false` の場合、完全な `analysis_history.context_snapshot` は永続化されません。古い履歴レコードに要約がない場合、フィールドは空になりますが、レポートは引き続き読み込まれます。

P1b はプロンプトを変更せず、`analysis_phase` リクエストパラメータを追加せず、Web のフェーズラベルやレンダリングを追加せず、pending/processing の TaskPanel 状態、進行中の SSE イベント、Bot、通知、`market_review`、または P3 の日中データ品質フィールドを対象にしません。

### Market Phase プロンプト注入（Issue #1386 P2-min）

P2-min は、既に `market_phase_context` を受け取る分析パスに対して、ランタイムの market phase を LLM が読める形のプロンプトセクションにレンダリングし始めます。通常分析、単一 Agent、マルチエージェントのプロンプトは、現在のフェーズ、市場ローカル時刻、再利用可能な最新の完全日足バー日付、および最小限のフェーズ制約を確認できるようになりました: 寄り付き前の実行は本日の値動きを既に起きたものとして記述してはならず、日中 / 昼休み / 引け間際の実行は最新の日足バーを未完了の可能性があるものとして扱い、大引け後の実行は完全セッションのリキャップスタイルを維持でき、非取引または不明なフェーズは保守的であるべきです。

P2-min は依然として API/Web/Bot のパラメータを追加せず、フェーズを履歴/タスクステータス/レポートメタデータに永続化せず、レポート JSON スキーマを変更せず、完全な相場鮮度、フォールバック、stale、またはデータ品質の契約を導入しません。P1a パイプラインを通らずに `market_phase_context` を構築する Bot/API の直接 Agent エントリポイントは、以前の動作を維持します。エントリポイントの伝播と可視ラベルは後の P4+ の作業に残されています。

### 日中データパケットとリアルタイム品質管理（Issue #1386 P3）

P3 は通常分析パスにリアルタイム相場の品質メタデータを追加しますが、依然として `analysis_phase` パラメータを追加せず、API/Web/Bot のフェーズエントリポイントを変更せず、レポート JSON スキーマを変更せず、#1389 P5 のデータ品質スコアリングやモデル信頼度の制限を実装しません。リアルタイム相場は `fetched_at`、`provider_timestamp`、`is_stale`、`stale_seconds`、`fallback_from` を持つ場合があります。`fetched_at` はシステムの取得時刻で、`provider_timestamp` はプロバイダーが実際に相場タイムスタンプを返した場合にのみ設定されます。プロバイダー時刻が利用できない場合、システムは鮮度を捏造せず、`stale_seconds` / `is_stale` は空のままです。

ソース全体のフォールバックセマンティクスは固定されています: `source` は実際に成功したプロバイダートークンを保持し、`fallback_from` は現在の試行で失敗した最優先のソース全体を記録します。プライマリソースが成功し、後のプロバイダーが欠落フィールドを補完するだけの場合、`fallback_from` は設定されません。`AnalysisContextBuilder` はこれらの上流アーティファクトをマッピングするだけで、追加の取得を行わず、品質スコアリングも行いません。相場ブロックのステータスは `STALE > FALLBACK > AVAILABLE` の順に収束します。リアルタイム価格が `today` を上書きする場合、パイプラインは `is_partial_bar`、`is_estimated`、`estimated_fields`、`realtime_source`、および相場メタデータをマークします。`daily_bars` ブロックは引き続きストレージ内の完全な日足バーウィンドウを表します。部分/推定マーカーはテクニカルブロックにのみ入ります。鮮度スコアリング、日中キャッシュ TTL 階層、Agent ツールレベルの再利用、API/Web 表示は引き続きフォローアップとなります。

### 分析フェーズのエントリポイントとタスクキューのパススルー（Issue #1386 P4a）

P4a は `analysis_phase=auto|premarket|intraday|postmarket` リクエストパラメータ（デフォルト `auto`）を追加し、API 呼び出し元が現在の分析のフェーズを明示的にオーバーライドできるようにします。このパラメータは `POST /api/v1/analysis/analyze`、非同期タスクキュー、`AnalysisService`、通常分析パイプライン、market-phase コンテキスト構築を通じて配線されます。Web フロントエンドの型と API マッピングはこのフィールドを受け入れますが、このフェーズではページセレクターを追加しません。Bot、スケジュール、GitHub Actions、DB マイグレーションは対象外のままです。

`analysis_phase` はリクエストされたオーバーライド値で、最終レポートのフェーズは `report.meta.market_phase_summary.phase` のままです。非同期で受理されたレスポンス、インメモリのタスクステータス、タスクリストレスポンス、SSE ペイロードは、リクエストされたフェーズをエコーします。DB 履歴フォールバックは永続化されたフェーズフィールドを追加しないため、古いレコードでは依然として空のまま返る場合があります。重複検出は引き続き銘柄のみなので、異なるフェーズで送信された同じ銘柄は、進行中の重複タスクとして扱われます。

market-phase コンテキスト構築は、依然としてレガシーの内部 `analysis_intent` 引数をサポートします: `analysis_phase` が `auto` のままの場合のみ、`auto` 以外の `analysis_intent` がこの実行のリクエストされたフェーズとして正規化されます。外部呼び出し元は `analysis_phase` を優先すべきです。

`auto` は既存のカレンダー推論を維持します。`auto` 以外の値は、フェーズのみをオーバーライドし、`is_trading_day`、`is_market_open_now`、`is_partial_bar`、`minutes_to_open`、`minutes_to_close` を再計算します。オーバーライドは実際の `market_local_time` や `effective_daily_bar_date` を書き換えません。現在の日付が取引セッションでない、またはカレンダーがセッションをサポートできない場合、分のフィールドは空になることがあります。

### Web フェーズラベル（Issue #1386 P4b）

P4b はフェーズオーバーライドセレクターを追加せずに、Web の可視性スライスを完成させます。進行中の TaskPanel は P4a がエコーした、リクエストされた `analysis_phase` のみを表示します。現在のタスクパネル UI では、`auto` はリクエストされた自動フェーズ（`请求阶段: 自动阶段`）として明示的にラベル付けされ、最終的に推論されたフェーズとしては提示されません。最終レポートページは `report.meta.market_phase_summary.phase` から実際の market phase をレンダリングし、`is_partial_bar=true` の場合は `Partial bar` マーカーを表示します。

データ品質の可視性は、引き続き `report.details.analysis_context_pack_overview.data_quality` と既存の `AnalysisContextSummary` コンポーネントを再利用します。Web UI は低感度のデータ品質要約とともにフェーズラベルを表示するだけで、完全な `AnalysisContextPack`、プロンプト要約、生のペイロード、または除去されたスナップショットの内部を公開しません。履歴リストのフィールド、Bot、スケジュール、GitHub Actions、Desktop、通知要約、高度なフェーズオーバーライド UI は、引き続きフォローアップの作業です。

### AnalysisContextPack プロンプト要約（Issue #1389 P3）

P3 は通常分析と Agent の初期プロンプトに、低感度の `AnalysisContextPack` 要約を注入します。パイプラインは、既に取得した相場、日足バー、トレンド、チップ、ファンダメンタルズ、ニュース、market-phase のアーティファクトからパックを構築し、`analysis_context_pack_summary` を下流に渡します。この新しいパック要約セクションでは、LLM は subject、version、データブロックの status/source/warnings/missing reason、ニュース結果件数のみを参照し、完全な `news.content`、`trend_result`、チップ、またはファンダメンタルズの生ペイロードをそのセクションを通じて見ることはありません。Agent パスでは、パイプラインは履歴プリフェッチ後に `storage.get_analysis_context()` を一度読み取って日足バーステータスを駆動し、その読み取りに使用可能なコンテキストがない場合にのみ `daily_bars_missing` をマークします。既存の `news_context`、Agent がプリフェッチした JSON、`enhanced_context` の生ペイロードチャネルは、P3 以前の動作を維持し、この要約によって置き換えられたりサニタイズされたりすることはありません。

P3 自体は API/Web/Bot のパラメータを追加せず、フィールドを履歴/タスクステータス/レポートメタデータに永続化せず、レポート JSON スキーマを変更せず、履歴、通知、Web サーフェスを通じて完全なパックを公開しません。パックデータの Agent ツールレベルの再利用と P5 のデータ品質スコアリングは、後のフェーズに残されています。

### AnalysisContextPack 低感度可視性（Issue #1389 P4）

P4 は `report.details.analysis_context_pack_overview` を追加します。履歴詳細と完了した `/api/v1/analysis/status/{task_id}` レスポンスは、永続化された `context_snapshot` から同じ低感度概要を読み取ります。同期分析レスポンスも、永続化されたばかりの `analysis_history.context_snapshot` から概要を抽出するため、`SAVE_CONTEXT_SNAPSHOT=false` の場合、新しいレコードでこのフィールドが保証されません。Web レポートページは、Strategy と News の後に折りたたまれたデータブロック要約をレンダリングし、ヘッダーに available/missing 件数、ゼロでない他ステータス件数、トリガーソースを表示し、展開後にデータブロックの status、source、warnings、missing reason、ステータス件数、ニュース結果件数を表示します。API の `details.context_snapshot` はトップレベルの `analysis_context_pack_overview` を除去するため、生のスナップショットパネルが公開概要を重複させません。

概要には、完全なパック、`analysis_context_pack_summary` プロンプト文字列、`items.value`、ニュース本文、`trend_result`、チップ、またはファンダメンタルズの生ペイロードは含まれません。`SAVE_CONTEXT_SNAPSHOT=false` の場合、完全な `analysis_history.context_snapshot` は永続化されないため、新しい履歴レコードは概要を提供できません。概要のない古いレコードは引き続き空のフィールドを返し、レポートは引き続き読み込まれます。このフェーズは pending/processing の TaskPanel、進行中の SSE イベント、通知要約、Bot/Desktop 固有のレンダリング、`market_review` 概要、またはデータ品質スコアリングを対象にしません。

### AnalysisContextPack データ品質スコアリングとプロンプト制限（Issue #1389 P5）

P5 は、`PACK_VERSION = "1.0"` を変更せず、データソースを追加せず、レポート JSON スキーマを変更することなく、`AnalysisContextPack` に軽量なデータ品質スコアリングとモデルが読める形のデータ制限を追加します。`ContextFieldStatus` は現在 `fetch_failed` を含み、これはフィールドまたはデータブロックがこの実行で明示的に取得に失敗したことのみを意味します。最初のマッピングは `fundamental_context.status == "failed"` のみを `fetch_failed` に変換し、空のニュース、未設定の検索、リアルタイム相場の欠落、チップデータの欠落は既存の `missing` / `not_supported` セマンティクスを維持します。

`DataQuality` は現在 `overall_score`、`level`、`block_scores`、`limitations` を含み、古い `warnings` / `metadata` フィールドを保持します。スコアリングは 6 ブロックに固定されています: `quote`、`daily_bars`、`technical`、`news`、`fundamentals`、`chip`。補助的な欠落ブロックは再正規化されません。コアブロックがデグレードしている場合、プロンプトの `Data Limitations` セクションはモデルに高い信頼度を返さないよう指示します。欠落した補助ブロックは、対応する分析セクションのみを制約し、強気または弱気と解釈してはなりません。このセクションは `format_analysis_context_pack_prompt_section()` によって生成されるため、通常分析、単一 Agent、マルチエージェントのパスは、生のペイロード、ニュース本文、生のトレンド値、シークレット、トークン、または Webhook を公開せずに、同じ低感度要約を再利用します。

履歴詳細、同期分析レスポンス、完了タスクステータスレスポンスは、引き続き `report.details.analysis_context_pack_overview` のみを公開します。P5 は score、level、block_scores、limitations を持つネストした `data_quality` オブジェクトを追加するだけで、`warnings` を重複させません。Web レポートページはデフォルトで折りたたまれたままで、ヘッダーに品質スコア/レベルを追加し、展開後に limitations と `fetch_failed` ステータスを表示します。API の `details.context_snapshot` は引き続きトップレベルの `analysis_context_pack_overview` を除去します。

### AnalysisContextPack のドキュメント、移行、ロールバック（Issue #1389 P6）

P6 はドキュメントと設定可視性のクロージャのみです。パックのランタイム動作を追加せず、パックの有効化/無効化フィーチャーフラグを追加せず、`PACK_VERSION = "1.0"` を変更せず、API パラメータを追加せず、レポート JSON スキーマを変更せず、データベースマイグレーションを実行しません。完全な契約、フィールド状態、低感度可視性、編集境界、移行ノート、ロールバックパスについては、[AnalysisContextPack トピックドキュメント](analysis-context-pack.md) を参照してください。

`SAVE_CONTEXT_SNAPSHOT` は既存の環境変数です。P6 はそれを `.env.example`、設定レジストリ、Web 設定ヘルプを通じて公開するだけです。デフォルトは `true` です。`false` に設定した場合、または CLI が `--no-context-snapshot` を使用した場合、新しい履歴レコードは `enhanced_context`、`market_phase_summary`、`analysis_context_pack_overview`、診断スナップショット、生のスナップショットフィールドを含む完全な `analysis_history.context_snapshot` を永続化しなくなります。この設定は、現在実行の `AnalysisContextPack` 構築を無効化せず、プロンプトから低感度の `analysis_context_pack_summary` を削除せず、レポート JSON スキーマや API リクエストパラメータを変更しません。

ランタイムのパックマスタースイッチはありません。P3〜P5 のパックプロンプト要約、概要、またはデータ品質統合を無効化するには、リリースロールバックまたはコードロールバックが必要です。`analysis_context_pack_overview` / `data_quality` のない古い履歴レコードは、引き続き空のフィールドを返し、読み取り可能なままです。

### 日中の意思決定ガードレールと品質チェック（Issue #1386 P5）

P5 は、個別銘柄の分析レポートに `dashboard.phase_decision` 配下のフェーズ対応の意思決定ブロックを追加します: `phase_context`、`action_window`、`immediate_action`、`watch_conditions`、`next_check_time`、`confidence_reason`、`data_limitations`。これは履歴の `raw_result` に保存される後方互換のレポート JSON 追加です。`analysis_phase` API パラメータを追加せず、Web のフェーズエントリポイントを変更せず、設定を追加せず、デフォルトの大引け後日次レビュー動作を変更しません。

通常分析と Agent 分析は、現在の `market_phase_summary` と `analysis_context_pack_overview.data_quality` を使用して、履歴が保存される前に軽量なガードレールを適用します。コアの quote / daily_bars / technical データが stale、フォールバック、欠落、fetch_failed、部分、または推定の場合、高信頼度の結論は上限が設けられます。寄り付き前、非取引、または不明なフェーズは、高信頼度の日中の売買アクションを出してはなりません。日中、昼休み、引け間際の出力は、主結論とアクションフィールドで「after today's close」や「focus tomorrow」のような大引け後リキャップの文言がスキャンされ、明らかな違反はフェーズ安全な待機/監視の文言に置き換えられます。ガードレールは低感度の `phase_context` とデータ制限を埋めるだけで、監視条件や次回チェック時刻を捏造しません。通知要約、アラート、保有銘柄、バックテスト連携は、引き続き後の P6 の作業です。

### アラート、ポートフォリオ、履歴の連携（Issue #1386 P6）

P6 は、アラート、ポートフォリオ、履歴、バックテスト、通知にわたって既存の `market_phase_summary` と `analysis_context_pack_overview` を再利用します。新しいフェーズ/パックプロトコルを導入せず、データベースマイグレーションを必要としません。アラートトリガー行は、既存のテキストの `diagnostics` フィールドを引き続き使用します。診断を JSON として表現できる場合、ワーカーは `analysis_visibility.market_phase_summary`、`analysis_visibility.analysis_context_pack_overview`、`analysis_visibility.source` をトリガー行にマージします。レガシーのプレーンテキスト診断は引き続き読み取り可能です。Alert API の派生フィールドは空のままで、`analysis_visibility_source=legacy_text` となります。

アラートのフェーズ要約はトリガー時のコンテキストから生成されます: シンボルターゲットは銘柄の市場を推論し、`target_scope=market` は `cn|hk|us` 地域を直接使用し、単一の市場にマッピングできないアカウントレベルのターゲットは `unknown` にフォールバックする場合があります。パック概要は、評価器が提供する概要、または過去 30 日以内の最近の低感度履歴スナップショットからのみ取得されます。データが欠落している場合は `null` を返します。アラートワーカーはパックを捏造せず、軽量な LLM 分析を自動的に実行しません。公開ソース値は `alert_trigger_market_context`、`analysis_history_snapshot`、`evaluator_snapshot`、`legacy_text`、または `null` です。

ポートフォリオページは、`POST /api/v1/portfolio/positions/{symbol}/analysis` に裏付けられたポジションごとの手動分析アクションを追加します。リクエストは `account_id`、`analysis_phase=auto|premarket|intraday|postmarket`、`force` を受け入れます。ゼロでない現在の保有銘柄のみ送信でき、保有がない場合は 404 を返し、`account_id` なしで複数アカウントに保有される同じシンボルは `400 ambiguous_position_account` を返します。エンドポイントは既存の非同期受理 / 重複セマンティクスを維持し、`force` はリフレッシュ動作のみを制御し、進行中の重複検出をバイパスしません。バックエンドは、低感度の `portfolio_context` のみを内部的にパイプラインとオプションのコンテキストパック `portfolio` ブロックに渡します。そのブロックは既存の 6 つのデータ品質ウェイトに影響せず、タスクリストや SSE ペイロードを通じて公開されません。

履歴リスト、同一銘柄履歴、StockBar 項目、詳細は `context_snapshot` から `market_phase_summary` を抽出します。古い行、スナップショット欠落、またはパース失敗の場合は `null` を返します。バックテスト結果項目は現在 `market_phase` と `market_phase_summary` を含み、結果/パフォーマンス/要約クエリは `analysis_phase=premarket|intraday|postmarket|unknown` をサポートします。統計は `intraday`、`lunch_break`、`closing_auction` を intraday に折り込み、`non_trading`、欠落、無効な値を unknown に折り込みます。フェーズフィルタリングされたバックテストクエリは、リポジトリを通じて結果とスナップショットをバッチ読み取りし、ページネーション前にバケット化し、要約診断に `phase_breakdown` と `raw_phase_counts` を公開します。

通知要約は 1 つの公開フォーマットヘルパーを使用し、フェーズラベル、トリガーソース、部分バー警告、データ品質レベル、最初の 2 つの limitations のみを含みます。生のコンテキストパック、プロンプト、ニュース本文、または機密性の高いポートフォリオの詳細は出力しません。Web の Alerts、Portfolio、History、StockBar、Backtest の各ページは、新しいフェーズバッジ、品質要約、フェーズフィルター、内訳を表示します。

### ドキュメント、設定、移行に関するノート（Issue #1386 P7）

P7 は寄り付き前 / 日中 / 大引け後分析に関するユーザー向けドキュメントのクロージャのみです。ランタイム動作、設定キー、API パラメータ、データベースマイグレーション、Web フェーズオーバーライドセレクター、Bot フェーズパラメータ、または GitHub Actions の日中ワークフローを追加しません。デフォルトの日次大引け後分析、デフォルトの GitHub Actions 実行、既存のスケジュール動作は変更されません。

推奨される使い方:

| シナリオ | 推奨される使い方 | 注記 |
| --- | --- | --- |
| 寄り付き前 | 寄り付き計画と監視条件を構築する | 本日のまだ取引されていない値動きを事実として記述しないこと。直近の完全な取引日、オーバーナイト情報、寄り付きトリガーに焦点を当てる。 |
| 日中 / 昼休み / 引け間際 | ライブ状態、リスク、機会アラートを確認する | 現在価格、リアルタイム相場の鮮度、部分バー、データ制限、次の監視条件に焦点を当てる。これは完全な大引け後レビューを置き換えるものではない。 |
| 大引け後 | 完全なレビューと翌日計画を維持する | 完全な取引日のセマンティクスを使用し、デフォルトの日次分析シナリオに最も近い。 |

エントリポイントと可視性:

| エントリポイント | フェーズの動作 |
| --- | --- |
| `POST /api/v1/analysis/analyze` | `analysis_phase=auto|premarket|intraday|postmarket` をサポート。省略時はデフォルト `auto`。 |
| Web のメイン分析 / 再分析 / ポートフォリオ手動分析 | 現在フェーズオーバーライドセレクターはありません。フロントエンドはデフォルトで `auto` を使用し、進行中のタスクパネルはリクエストされたフェーズを表示し、最終レポートページは最終的なフェーズラベルを表示します。 |
| Bot / CLI / スケジュール / デフォルト GitHub Actions | `analysis_phase` を渡さず、引き続き `auto` 推論を使用し、デフォルトの大引け後動作は変更されません。 |
| 履歴 / バックテスト / 通知 / アラート | 公開の `market_phase_summary` と低感度の `analysis_context_pack_overview` のみを消費します。完全なパック、プロンプト要約、ニュース本文、または機密性の高いポートフォリオの詳細は公開しません。 |

`analysis_phase` はリクエストされたオーバーライド値で、最終レポートのフェーズは `report.meta.market_phase_summary.phase` のままです。`analysis_phase` を省略する古い呼び出し元は互換性を維持します。`market_phase_summary` や `analysis_context_pack_overview` のない古い履歴行は空のフィールドを返し、引き続き正常に読み込まれます。バックテストクエリは `analysis_phase=premarket|intraday|postmarket|unknown` フィルタリングをサポートし、P6 は昼休みと引け間際のフェーズを intraday に折り込みます。

`SAVE_CONTEXT_SNAPSHOT=false` または CLI の `--no-context-snapshot` は、新しい履歴行に対して完全な `context_snapshot` の永続化を停止するだけなので、新しい履歴は永続化されたフェーズ要約 / パック概要 / 診断スナップショットデータを公開しなくなります。現在実行の `AnalysisContextPack` 構築は無効化せず、プロンプトから低感度の `analysis_context_pack_summary` を削除せず、レポート JSON スキーマを変更しません。古い大引け後の文言により近い出力が必要な呼び出し元は、一時的に `analysis_phase=postmarket` を固定できます。P0〜P6 のフェーズ/パックのランタイム統合を完全に削除するには、リリースロールバックまたはコードロールバックが必要です。

---

## 通知チャネルの設定

通知チャネルのマトリクスと `--check-notify` CLI の詳細は [Notification Guide](notifications.md) に記載されています。

### WeChat Work

1. WeChat Work のグループチャットで「グループ Bot」を追加します
2. Webhook URL をコピーします
3. `WECHAT_WEBHOOK_URL` を設定します

### Feishu

> ⚠️ **重要な区別**: `FEISHU_WEBHOOK_SECRET`（Webhook 署名シークレット）と `FEISHU_APP_SECRET`（Feishu App Secret）はまったく異なる 2 つの設定変数であり、互換的に使用できません。

**最小限の実用構成（セキュリティ制限なし）:**

```env
FEISHU_WEBHOOK_URL=https://open.feishu.cn/open-apis/bot/v2/hook/your_hook_token
```

**ステップごとのセットアップ:**

1. **対象の Feishu グループでカスタム Bot を作成**:
   - グループを開く → 設定アイコン（右上）をタップ → **グループ Bot** → **Bot を追加** → **カスタム Bot**
   - Bot の名前を入力し、生成された **Webhook URL**（形式: `https://open.feishu.cn/open-apis/bot/v2/hook/...`）をコピーします
2. コピーした URL を `FEISHU_WEBHOOK_URL` に設定します。
3. Bot の **セキュリティ設定** を確認し、追加オプションが有効な場合は対応する設定を追加します:
   - **追加のセキュリティなし**: `FEISHU_WEBHOOK_URL` のみが必要です。
   - **署名検証が有効**: Feishu に表示されるシークレットを `FEISHU_WEBHOOK_SECRET` にコピーします。**両側を同時に有効化または無効化する必要があります** — Feishu で署名がオンなのに `FEISHU_WEBHOOK_SECRET` が欠落している（またはその逆）場合、すべてのリクエストが拒否されます。
   - **キーワードが有効**: まったく同じキーワードを `FEISHU_WEBHOOK_KEYWORD` にコピーします。アプリは自動的にすべてのメッセージの先頭に付加します。レポートテンプレートを変更する必要はありません。
   - **IP 許可リストが有効**: ランタイム（ローカル / Docker / GitHub Actions はそれぞれ異なる IP を持ちます）のアウトバウンド IP が許可リストにあることを確認してください。
4. `FEISHU_APP_ID` / `FEISHU_APP_SECRET` は Feishu アプリ / Stream Bot / クラウドドキュメントのフローのみのためのものです。これらはグループ Webhook 通知を**トリガーせず**、`FEISHU_WEBHOOK_URL` の代わりに単独で使用してはなりません。
5. `FEISHU_APP_ID` / `FEISHU_APP_SECRET` を `FEISHU_CHAT_ID` とともに設定すると、Feishu App Bot はグループ Webhook なしで、指定したチャットまたはユーザーに直接通知をプッシュできます。`FEISHU_RECEIVE_ID_TYPE` のデフォルトは `chat_id` で、P2P 配信には `open_id` を設定します。これは Feishu OpenAPI Bot のセッションルートを使用し、グループ Webhook パスから独立しています。
6. App Bot の送信パスは、`requirements.txt` に既に記載されている既存の `lark-oapi>=1.0.0` 依存関係を再利用します。標準的なソースインストール、Docker、GitHub Actions の日次ワークフロー、デスクトップビルドはすべて `pip install -r requirements.txt` でインストールします。リファレンス: [Feishu message create OpenAPI](https://open.feishu.cn/document/server-docs/im-v1/message/create)、[lark-oapi PyPI](https://pypi.org/project/lark-oapi/)、[SDK リポジトリ](https://github.com/larksuite/oapi-sdk-python)。

**よくある失敗の原因:**
- `FEISHU_APP_ID` / `FEISHU_APP_SECRET` のみを設定し、`FEISHU_WEBHOOK_URL` も App Bot のアクティブ配信ターゲット `FEISHU_CHAT_ID` も設定していない
- Bot で署名セキュリティが有効だが、`FEISHU_WEBHOOK_SECRET` がローカルで設定されていない（または誤って `FEISHU_APP_SECRET` を設定している）
- Bot でキーワードセキュリティが有効だが、`FEISHU_WEBHOOK_KEYWORD` がローカルで設定されていない
- Bot が対象グループに追加されていない、またはグループ権限が投稿をブロックしている
- Feishu IP 許可リストが有効で、ランタイム IP が許可リストにない
- メッセージ内容が長すぎる: Feishu にはメッセージごとの長さ制限があり、システムはメッセージを自動分割します。単一ドキュメントで全内容を得るには、Feishu クラウドドキュメント（`FEISHU_APP_ID` / `FEISHU_APP_SECRET` / `FEISHU_FOLDER_TOKEN`）を設定してください

図解付きの完全なトラブルシューティングガイドについては、[docs/bot/feishu-bot-config.md](bot/feishu-bot-config.md) を参照してください。

### Telegram

1. @BotFather と会話して Bot を作成します
2. Bot Token を取得します
3. Chat ID を取得します（@userinfobot 経由）
4. `TELEGRAM_BOT_TOKEN` と `TELEGRAM_CHAT_ID` を設定します
5. （オプション）トピックに送信するには、`TELEGRAM_MESSAGE_THREAD_ID` を設定します（トピックリンクから取得）

### Email

1. メールの SMTP サービスを有効化します
2. 認証コードを取得します（ログインパスワードではありません）
3. `EMAIL_SENDER`、`EMAIL_PASSWORD`、`EMAIL_RECEIVERS` を設定します

サポートされているメールプロバイダー:
- QQ Mail: smtp.qq.com:465
- 163 Mail: smtp.163.com:465
- Gmail: smtp.gmail.com:587

**異なる銘柄グループを異なるメール受信者に送信する**（Issue #268、オプション）:
`STOCK_GROUP_N` と `EMAIL_GROUP_N` を設定して、異なる銘柄グループを異なる受信箱にルーティングします。`STOCK_LIST` は依然として実際の分析範囲を定義するため、各 `STOCK_GROUP_N` は `STOCK_LIST` のサブセットであるべきです。これはメール受信者のみを変更します。Telegram、WeChat、Webhook、その他のチャネルは引き続き `STOCK_LIST` 全体の完全なレポートを受信します。大引け振り返りメールは、設定済みのすべてのグループ受信者に送信されます。

> GitHub Actions の制限: 2026-03-29 時点で、リポジトリのデフォルト `00-daily-analysis.yml` は任意の番号付き `STOCK_GROUP_N` / `EMAIL_GROUP_N` 変数を自動インポートしません。ワークフローの `env:` ブロックを拡張せずにリポジトリの Secrets / Variables に追加するだけでは、ランタイムプロセスに到達しません。

```bash
STOCK_LIST=600519,300750,002594,AAPL
STOCK_GROUP_1=600519,300750
EMAIL_GROUP_1=user1@example.com
STOCK_GROUP_2=002594,AAPL
EMAIL_GROUP_2=user2@example.com
```

### カスタム Webhook

任意の POST JSON Webhook をサポートします。以下を含みます:
- DingTalk Bot
- Discord Webhook
- Slack Webhook
- Bark（iOS プッシュ）
- セルフホストサービス

`CUSTOM_WEBHOOK_URLS` を設定し、複数の場合はカンマで区切ります。

AstrBot、NapCat、またはセルフホストサービスがカスタムリクエストボディを必要とする場合は、
`CUSTOM_WEBHOOK_BODY_TEMPLATE` を設定します。これはグローバルテンプレートであり、
Bark、Slack、Discord などの URL 自動検出ペイロードの前にレンダリングされます。レンダリングされた値が
JSON オブジェクトでない場合、DSA はデフォルトのペイロードにフォールバックします。改行や引用符が有効な JSON を
維持するよう、`$content_json` / `$title_json` を優先してください:

```env
CUSTOM_WEBHOOK_BODY_TEMPLATE={"msg_type":"text","content":$content_json}
```

使用可能なプレースホルダー: `$content_json`、`$content`、`$title_json`、`$title`。
生の `$content` / `$title` は JSON エスケープされないため、引用符や改行によって
テンプレートが無効になりフォールバックがトリガーされる可能性があります。

Bark はカスタム Webhook のベースラインのままで、`BARK_*` 設定は不要です。
Bark エンドポイントは `CUSTOM_WEBHOOK_URLS` に設定します。グローバルテンプレートで
Bark を使用する場合は、Bark ボディを明示的に含めてください:

```env
CUSTOM_WEBHOOK_URLS=https://api.day.app/YOUR_BARK_KEY
```

```env
CUSTOM_WEBHOOK_BODY_TEMPLATE={"title":$title_json,"body":$content_json,"group":"stock"}
```

NapCat / OneBot の例は、実際のエンドポイント、`user_id`、
または `group_id` に合わせて調整する必要があります:

```env
CUSTOM_WEBHOOK_BODY_TEMPLATE={"user_id":123456,"message":$content_json}
```

### ntfy / Gotify

ntfy と Gotify は第一級の通知チャネルです。テキスト / JSON
のみを送信し、Markdown から画像への変換は使用しません。

ntfy は完全なトピックエンドポイントを使用します。最後のパスセグメントが
トピックとして扱われます:

```env
NTFY_URL=https://ntfy.sh/my-topic
NTFY_TOKEN=
```

Gotify はサーバーのベース URL を使用します。送信側は固定の `/message` API を付加し、
アプリケーショントークンを `X-Gotify-Key` ヘッダーで送信します。`GOTIFY_URL` は
リバースプロキシのパスプレフィックスを含むことができますが、`/message` を含めてはなりません:

```env
GOTIFY_URL=https://gotify.example
GOTIFY_TOKEN=app-token
```

```env
# 実際のリクエスト URL: https://example.com/gotify/message
GOTIFY_URL=https://example.com/gotify
GOTIFY_TOKEN=app-token
```

`NTFY_URL` と `GOTIFY_URL` は、2 つのサービスが異なる API を公開しているため、
意図的に異なる URL セマンティクスを使用します: ntfy のトピックはエンドポイントの一部であり、
Gotify は `/message` を固定のサーバー API として使用します。

### Discord

Discord は 2 つのプッシュ方法をサポートします:

**方法 1: Webhook（推奨、シンプル）**

1. Discord のチャネル設定で Webhook を作成します
2. Webhook URL をコピーします
3. 環境変数を設定します:

```bash
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/xxx/yyy
```

**方法 2: Bot API（より多くの権限が必要）**

1. [Discord Developer Portal](https://discord.com/developers/applications) でアプリケーションを作成します
2. Bot を作成し Token を取得します
3. Bot をサーバーに招待します
4. Channel ID を取得します（開発者モードでチャネルを右クリック）
5. 環境変数を設定します:

```bash
DISCORD_BOT_TOKEN=your_bot_token
DISCORD_MAIN_CHANNEL_ID=your_channel_id
```

Discord への通知送信だけでなく、Discord のスラッシュコマンド / Interaction コールバックを受信する必要がある場合は、`Discord Developer Portal -> General Information -> Public Key` から公開鍵もコピーして設定します:

```bash
DISCORD_INTERACTIONS_PUBLIC_KEY=your_public_key
```

この公開鍵がないと、受信した Discord Webhook リクエストは拒否されます。

### Slack

Slack は 2 つのプッシュ方法をサポートします。両方を設定した場合、テキストと画像が同じチャネルに届くようにするため、Bot API が優先されます:

**方法 1: Bot API（推奨、画像アップロードに対応）**

1. Slack App を作成: https://api.slack.com/apps → Create New App
2. Bot Token Scopes を追加: `chat:write`、`files:write`
3. ワークスペースにインストールして Bot Token（xoxb-...）を取得します
4. Channel ID を取得: チャネル詳細 → 下部の channel ID をコピー
5. 環境変数を設定します:

```bash
SLACK_BOT_TOKEN=xoxb-...
SLACK_CHANNEL_ID=C01234567
```

**方法 2: Incoming Webhook（簡単なセットアップ、テキストのみ）**

1. Slack App 管理ページで Incoming Webhook を作成します
2. Webhook URL をコピーします
3. 環境変数を設定します:

```bash
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/T.../B.../xxx
```

### Pushover（iOS/Android プッシュ）

[Pushover](https://pushover.net/) は iOS と Android をサポートするクロスプラットフォームのプッシュサービスです。

1. Pushover アカウントを登録し、アプリをダウンロードします
2. [Pushover Dashboard](https://pushover.net/) から User Key を取得します
3. Application を作成して API Token を取得します
4. 環境変数を設定します:

```bash
PUSHOVER_USER_KEY=your_user_key
PUSHOVER_API_TOKEN=your_api_token
```

機能:
- iOS/Android に対応
- 通知の優先度とサウンド設定に対応
- 個人利用に十分な無料枠（月 10,000 メッセージ）
- メッセージは 7 日間保持

---

## データソースの設定

システムはデフォルトで AkShare（無料）を使用し、他のデータソースもサポートします:

### AkShare（デフォルト）
- 無料、設定不要
- データソース: Eastmoney スクレイパー

### Tushare Pro
- Token を取得するために登録が必要
- より安定、より包括的なデータ
- `TUSHARE_TOKEN` を設定

### Baostock
- 無料、設定不要
- バックアップデータソースとして使用

### YFinance
- 無料、設定不要
- 米国株／香港株データに対応
- 米国株の過去データとリアルタイムデータはどちらも YFinance を専用に使用し、akshare の米国株調整問題によるテクニカル指標エラーを回避します

### Longbridge
- 米国株／香港株向けのオプションのフォールバックで、主に YFinance が見逃す可能性のあるフィールドを補完するために使用
- 新規統合では Longbridge OAuth 2.0 を使用すべきです: client id は `LONGBRIDGE_OAUTH_CLIENT_ID` から、または Legacy Access Token が設定されていない場合は `LONGBRIDGE_APP_KEY` から読み取られます。SDK トークンキャッシュを生成するには、対話的なマシンで `python scripts/generate_longbridge_oauth_token.py --client-id <client_id>` を一度実行します
- GitHub Actions / Docker のヘッドレス実行では、ローカルの `~/.longbridge/openapi/tokens/<client_id>` ファイルを base64 化し、`LONGBRIDGE_OAUTH_TOKEN_CACHE_B64` として保存します
- OAuth ランタイムサポートには SDK API の `OAuthBuilder` と `Config.from_oauth` が必要です。Linux/Docker 環境が古い SDK のみインストールできる場合、アプリは明確な警告をログに記録して Longbridge をスキップし、YFinance / AkShare フォールバックは利用可能なまま維持します
- レガシー API キーは `LONGBRIDGE_APP_KEY`、`LONGBRIDGE_APP_SECRET`、`LONGBRIDGE_ACCESS_TOKEN` で引き続きサポートされます。この Access Token はレガシー API キーの認証情報であり、OAuth アクセストークンではありません
- オプションの調整値: `LONGBRIDGE_STATIC_INFO_TTL_SECONDS`（デフォルト `86400`）と `LONGBRIDGE_CONNECTION_COOLDOWN_SECONDS`（デフォルト `15`）
- 認証情報がない場合、オプションの Longbridge フェッチャーはインスタンス化されません
- `client is closed`、`context closed`、`connection closed` などのランタイムエラーが発生した場合、Longbridge は短いクールダウンウィンドウに入り、米国株／香港株の日足またはリアルタイムリクエストは、リクエストごとに再接続するのではなく、自動的に YFinance / AkShare にフォールバックします

---

## 高度な機能

### 香港株のサポート

香港株のコードには `hk` プレフィックスを使用します:

```bash
STOCK_LIST=600519,hk00700,hk01810
```

香港株の日足履歴は、香港株の日足データをサポートしない efinance、pytdx、baostock などの組み込みプロバイダーをスキップし、香港株シンボルと非香港市場データの不一致を回避します。AkShare/Tushare/YFinance/Longbridge は引き続き香港株のフォールバックパスを提供します。Longbridge が接続クールダウンウィンドウ内にある場合、ルートは一時的にそれをスキップし、残りの香港株対応フォールバックで続行します。

### マルチモデル切り替え

複数のモデルを設定すると、システムが自動的に切り替えます:

```bash
# Gemini（プライマリ）
GEMINI_API_KEY=xxx
GEMINI_MODEL=gemini-3.1-pro-preview

# OpenAI 互換（バックアップ）
OPENAI_API_KEY=xxx
OPENAI_BASE_URL=https://api.deepseek.com
OPENAI_MODEL=deepseek-v4-flash
# deepseek-chat / deepseek-reasoner は引き続き互換ですが、DeepSeek は 2026/07/24 以降これらを非推奨とマークします
```

### 高度なモデルルーティング（LiteLLM による）

[LLM Config Guide](LLM_CONFIG_GUIDE_EN.md) を参照してください。ほとんどのユーザーはプライマリモデル、フォールバックモデル、チャネルの観点で考えるだけで十分です。このセクションは、基盤となる [LiteLLM](https://github.com/BerriAI/litellm) ルーティング機能に直接アクセスしたい上級ユーザー向けです。別途 Proxy サービスは不要です。

**2 層メカニズム**: 同一モデルのマルチキーローテーション（Router）とクロスモデルフォールバックは独立しています。

**マルチキー + クロスモデルフォールバックの例**:

```env
# プライマリ: 3 つの Gemini キーがローテーション。Router は 429 で切り替え
GEMINI_API_KEYS=key1,key2,key3
LITELLM_MODEL=gemini/gemini-3.1-pro-preview

# クロスモデルフォールバック: すべてのプライマリキーが失敗したら Claude → GPT を試す
# ANTHROPIC_API_KEY、OPENAI_API_KEY が必要
LITELLM_FALLBACK_MODELS=anthropic/claude-sonnet-4-6,openai/gpt-5.4-mini
```

> ⚠️ `LITELLM_MODEL` には provider プレフィックス（例: `gemini/`、`anthropic/`、`openai/`）を含める必要があります。レガシーの `GEMINI_MODEL`（プレフィックスなし）は `LITELLM_MODEL` が設定されていない場合にのみ使用されます。

**Vision モデル（画像からの銘柄コード抽出）**: [LLM Config Guide - Vision](LLM_CONFIG_GUIDE_EN.md#41-vision-model-image-stock-code-extraction) を参照してください。

### デバッグモード

```bash
python main.py --debug
```

ログファイルの場所:
- 通常ログ: `logs/stock_analysis_YYYYMMDD.log`
- デバッグログ: `logs/stock_analysis_debug_YYYYMMDD.log`

デバッグログはアプリ自身の DEBUG メッセージを保持しますが、LiteLLM の内部はデフォルトで `WARNING` となり、ストリーミング生成中のトークンレベルのサードパーティノイズを回避します。LiteLLM の内部を一時的に調べるには、`.env` で `LITELLM_LOG_LEVEL=DEBUG` を設定してください。

### SQLite 書き込みの安定性

ファイルベースの SQLite データベースに対して、アプリは接続起動時に `WAL` を有効化し、`busy_timeout` を設定するようになりました。`save_daily_data()` も `(code, date)` でのバッチアトミック upsert を使用し、一括書き込みや並行コールバック時のロック競合を削減します。

`.env` で動作を調整できます:

| 変数 | デフォルト | 説明 |
|----------|---------|-------------|
| `SQLITE_WAL_ENABLED` | `true` | ファイルベースの SQLite で `journal_mode=WAL` を有効化 |
| `SQLITE_BUSY_TIMEOUT_MS` | `5000` | SQLite のロック待機タイムアウト（ミリ秒） |
| `SQLITE_WRITE_RETRY_MAX` | `3` | `database is locked` / `database table is locked` エラーの最大リトライ回数 |
| `SQLITE_WRITE_RETRY_BASE_DELAY` | `0.1` | 指数バックオフ書き込みリトライの基本バックオフ遅延（秒） |

---

## 意思決定の実行可能性

個別銘柄レポートは、サポート/レジスタンス、出来高/チップのコンテキスト、主力資金フロー、リスクイベントで操作アドバイスを較正します。これにより、1 日の値動きやスコア閾値のみによって生じる売買の直接的な反転を減らします。価格がサポートとレジスタンスの間にあり、資金フローが不明確な場合、レポートは保有、レンジ内監視、洗い場監視などの中立的で実行可能な文言を優先します。買いの判断には、サポート確認、または出来高/資金フロー確認を伴う有効なレジスタンスブレイクアウトが必要です。売り/減らす判断には、サポート崩壊、継続的な流出、または明確に高まったリスクが必要です。
この後処理の更新は、アドバイス文言と安定性ロジックを調整するのみで、設定された LLM モデル/provider ルーティングのセマンティクス（LiteLLM、provider、API モデル設定を含む）は変更しません。
互換性チェックの結果: 意思決定の操作性とランタイム後処理パスは変更されますが、モデル/provider/API の設定と永続化のセマンティクスは変更されません。互換性の境界は現在、analysis/pipeline/agent の意図推論と安定化マッピングにあります。
検証の経緯: ランタイム動作は `src/analyzer.py`、`src/core/pipeline.py`、`src/core/backtest_engine.py`、`src/report_language.py`、`src/agent` の意思決定パスモジュールに実装されています（対応するテストは `tests/test_backtest_engine.py`、`tests/test_analyzer_news_prompt.py`、`tests/test_decision_stability.py`、`tests/test_agent_pipeline.py`）。`src/config.py` や永続化コードパスでのランタイム設定フィールドや設定クリーンアップロジックの追加/削除は行いません。

### 意思決定アクション分類（#1390 P0）

個別銘柄レポートは、既存の自由テキスト `operation_advice` を維持し、Web 履歴、StockBar、同一銘柄履歴、バックテスト結果行での構造化表示のためにオプションの `action` / `action_label` フィールドを追加します。`decision_type` はレガシーの `buy|hold|sell` 統計契約のままで、空の `action` は既存の `decision_type` 推論チェーンを書き換えません。

| `action` | よくあるソーステキスト | `decision_type` ブリッジ |
| --- | --- | --- |
| `buy` | `strong_buy`, `强烈买入`, `buy`, `买入`, `布局`, `建仓` | `buy` |
| `add` | `add`, `加仓`, `增持`, `accumulate` | `buy` |
| `hold` | `hold`, `持有`, `持有观察`, `洗盘观察` | `hold` |
| `watch` | `watch`, `观望`, `等待`, `wait` | `hold` |
| `reduce` | `reduce`, `减仓`, `trim` | `sell` |
| `sell` | `sell`, `卖出`, `清仓`, `strong_sell`, `强烈卖出` | `sell` |
| `avoid` | `avoid`, `回避`, `规避`, `不建议买入`, `避免买入`, `do not buy` | `hold` |
| `alert` | `alert`, `风险预警`, `警惕`, `触发告警`, `risk alert` | `hold` |

表の `decision_type` ブリッジは、8 状態のアクション分類とレガシーの 3 状態統計契約の間の互換性を文書化するのみです。#1390 P0 は `action` を既存の `decision_type` に自動的に書き戻しません。上流が明示的な `action` と意味的に異なる `decision_type` の両方を送る場合、レガシー統計、バックテスト、古いレポートセマンティクスは引き続き `decision_type` / 既存の推論チェーンに従います。`action/action_label` は構造化表示メタデータのままです。

不明または曖昧なアドバイスは `watch` や `hold` に強制されず、空の `action/action_label` を返します。Web 履歴カード、StockBar、同一銘柄履歴ドロワー、バックテスト結果行は、古いレコードに `action/action_label` がない場合、表示専用のフォールバックとして `operation_advice` を使用します。そのフォールバックは UI ラベルにのみ影響し、安定した API アクションや将来のシグナルアセットではありません。Web が `action` と `action_label` の両方を受け取った場合、まず現在の UI 言語で `action` からラベルをレンダリングします。API の `action_label` は、`action` がない場合の非 Web クライアントや互換表示のための、レポート言語の表示メタデータのままです。大引け振り返りやその他の非銘柄レポートは取引 `action` 値を出さず、`operation_advice` テキストのみを維持します。`dashboard.phase_decision.immediate_action` は market-phase ガードレールのレポートブロックに属し、#1390 P0 の 8 状態アクション導出には使用されません。最終的な market phase は引き続き `report.meta.market_phase_summary.phase` から取得されます。

#1390 P0 は、将来のシグナルアセットフィールドを現在のレポート要約、履歴リスト、StockBar 行、バックテストレスポンスにフラット化しません。#1390 P1 は現在、独立した `DecisionSignal` リソースを通じて、`horizon`、`plan_quality`、`status` などのより細かい計画フィールドを保持します。既存のレポート契約を変更せず、履歴をバックフィルせず、設定を追加しません。

### 意思決定シグナルアセット（#1390 P1）

`DecisionSignal` は、AI の推奨をクエリ可能、重複排除済み、ステータス更新可能なシグナルアセットとして永続化するための独立したバックエンドリソースです。`operation_advice` を置き換えず、レガシーの `decision_type=buy|hold|sell` 契約を拡張せず、既存のレポートからまだ自動抽出しません。P2 より前は、シグナルは明示的な API またはサービス呼び出しを通じてのみ書き込まれます。

コアフィールドには、`stock_code`、`stock_name`、`market`、`source_type`、`source_agent`、`source_report_id`、`trace_id`、`market_phase`、`trigger_source`、`action`、`action_label`、`confidence`、`score`、`horizon`、`entry_low`、`entry_high`、`stop_loss`、`target_price`、`invalidation`、`watch_conditions`、`reason`、`risk_summary`、`catalyst_summary`、`evidence`、`data_quality_summary`、`plan_quality`、`status`、`expires_at`、`created_at`、`updated_at`、`metadata` が含まれます。`action` は 8 状態のアクション分類を再利用し、`market_phase` は market phase 列挙を再利用します。`source_type` は `analysis|agent|alert|market_review|manual` をサポートし、`status` は `active|expired|invalidated|closed|archived` をサポートし、`horizon` は `intraday|1d|3d|5d|10d|swing|long` をサポートします。

`confidence` は `0.0-1.0`、`score` は `0-100` で、過去の `sentiment_score` とは別です。価格計画フィールドの `entry_low`、`entry_high`、`stop_loss`、`target_price` は有限の正の数でなければなりません。`entry_low` と `entry_high` の両方が存在する場合、`entry_low <= entry_high` が必要です。`plan_quality` は `complete|partial|minimal|unknown` をサポートします: 有効な明示的値はそのまま保存され、そうでない場合はサービスが計算します。エントリレンジ（`entry_low` または `entry_high`）は 1 スロットとして数えられ、`stop_loss`、`target_price`、`invalidation`、`watch_conditions` はそれぞれ 1 スロットとして数えられます。2 スロットで `partial`、4 つ以上で `complete`、十分なスロットのない action/reason で `minimal` となります。

新しい API エンドポイント:

- `POST /api/v1/decision-signals`: シグナルを作成または重複排除し、HTTP 200 で `{ item, created }` を返します。重複排除は、`source_report_id` が存在する場合は `(source_report_id, source_type, market, stock_code, action, horizon, market_phase)` を、`trace_id` のみが存在する場合は `(trace_id, source_type, market, stock_code, action, horizon, market_phase)` を使用します。どちらのソース識別子も持たないシグナルは重複排除されません。`source_type` はソースの名前空間なので、manual/pre-report の弱参照は、実際の分析にバインドされたシグナルに対して重複排除されません。`NULL` の `horizon` と `NULL` の `market_phase` は、同じ `NULL` 次元に対してのみ重複排除されます。異なるソースタイプ、市場、horizon、または market phase は、別々のシグナルとして永続化される場合があります。同じソースキーが期限切れシグナルに一致し、新しいリクエストが将来の `expires_at` を持つ active の場合、既存の行はその場でリフレッシュされ、引き続き `created=false` を返します。P1 は並行冪等性を保証しません。
- `GET /api/v1/decision-signals`: `market`、`stock_code`、`action`、`market_phase`、`source_type`、`source_report_id`、`trace_id`、`trigger_source`、`status`、時間範囲、`holding_only`、`account_id` を使用したページネーションクエリ。
- `GET /api/v1/decision-signals/{signal_id}`: 1 つのシグナルを取得。存在しない ID は 404 を返します。
- `PATCH /api/v1/decision-signals/{signal_id}/status`: 有効なステータスとオプションの `metadata` を更新します。`metadata` が提供された場合、保存されたメタデータオブジェクト全体を置き換え、複雑な状態マシンは強制されません。
- `GET /api/v1/decision-signals/latest/{stock_code}`: 銘柄の最新の active シグナルを返します。デフォルト `limit=1`。

読み取りパスは、リスト、詳細、latest クエリの前に、`expires_at` を過ぎた active シグナルを遅延的に期限切れにします。既に期限切れの active シグナルを作成すると `expired` として保存されます。同一ソースの期限切れシグナルは、将来の `expires_at` を持つ active データを再投稿することによってのみ延長でき、`PATCH /status` は `expires_at` を受け入れません。`closed|invalidated|archived` のシグナルは作成パスによって再アクティブ化されません。時間フィールドは保存と比較のために UTC のナイーブな datetime に正規化されます。タイムゾーン対応の入力は UTC に変換されて `tzinfo` が除去され、ナイーブな入力は UTC として扱われ、API レスポンスは引き続きタイムゾーンサフィックスのない ISO 文字列を返します。銘柄コードは `market` によって決定的に正規化されます: `600519`、`SH600519`、`600519.SH` などの CN バリアントは同じ保存コードに一致し、`00700`、`HK00700`、`00700.HK` などの HK バリアントは `HK00700` に一致し、US ティッカーは大文字化されます。`holding_only=true` は、active アカウント下で `quantity > 0` のキャッシュされた `portfolio_positions` 行のみを読み取り、保有された `(market, stock_code)` でシグナルを一致させます。オプションで active な `account_id` でスコープできます。ポートフォリオスナップショットのリプレイは呼び出しません。キャッシュが存在しない場合は空の結果を返し、呼び出し元はまずポートフォリオスナップショット API を通じてキャッシュをリフレッシュすべきです。

`source_report_id` は nullable で、既存の履歴行を参照する必要はありません。履歴レコードの削除は、実際に削除された ID に `source_report_id` が一致する `source_type=analysis` の履歴バインドシグナルのみを明示的に削除するため、`manual/agent/alert/market_review` の弱参照シグナルは、ID の衝突だけで削除されることはありません。リストエンドポイントは `source_report_id` と `trace_id` の型付きフィルターをサポートします。`task_id` や `alert_trigger_id` などの後続の関連付けフィールドは、P1 では `metadata` に保存すべきです。P1 はそれらの専用カラムや型付きフィルターを追加せず、後の統合フェーズに延期します。JSON フィールド、長いテキストフィールド、公開短テキストフィールド（`stock_name/source_agent/trigger_source/action_label`）は、機密キー、Bearer 値、Authorization/Cookie ヘッダーまたは代入、トークン様文字列、その他の機密代入、Webhook URL、URL のユーザー情報、機密のクエリまたはフラグメントパラメータを持つ URL を編集するシグナル固有のサニタイザーによって、永続化前にサニタイズされます。通常のエビデンス URL はソースのトレーサビリティのために保持され、長いテキストは診断の 300 文字切り捨てを使用しません。`trace_id` は同一ソースのアイデンティティフィールドです。編集される機密の認証情報を含む場合、API はロスのある編集済み値を保存する代わりにリクエストを拒否します。

これらのエンドポイントは既存の `/api/v1/*` 管理者認証ミドルウェアを継承します: `ADMIN_AUTH_ENABLED=true` の場合、呼び出し元は有効な管理者セッション Cookie を送信する必要があります。DecisionSignal は別個の認証スキームを追加しません。

## バックテスト

バックテストモジュールは、過去の AI 分析レコードを実際の値動きに対して自動的に検証し、分析推奨の精度を評価します。

### 仕組み

1. クールダウン期間（デフォルト 14 日）を過ぎた `AnalysisHistory` レコードを選択します
2. 分析日以降の日足バーデータ（フォワードバー）を取得します
3. 操作アドバイスから期待される方向を推論し、実際の値動きと比較します
4. 損切り/利確のヒット条件を評価し、執行リターンをシミュレートします
5. 全体および銘柄ごとのパフォーマンス指標に集約します

### 操作アドバイスのマッピング

| 操作アドバイス | ポジション | 期待される方向 | 勝利条件 |
|-----------------|----------|-------------------|---------------|
| 買い / 加仓 / 強い買い | long | up | リターン >= 中立バンド |
| 売り / 減仓 / 強い売り | cash | down | 下落 >= 中立バンド |
| 保有 / 保有観察 / レンジ内監視 / 洗い場監視 / 保有・監視 | long | not_down | 大幅な下落なし |
| 待機 / 観望 | cash | flat | 価格が中立バンド内 |

### 設定

`.env` で以下の変数を設定します（すべてオプション、デフォルトあり）:

| 変数 | デフォルト | 説明 |
|----------|---------|-------------|
| `BACKTEST_ENABLED` | `true` | 日次分析後にバックテストを自動実行するかどうか |
| `BACKTEST_EVAL_WINDOW_DAYS` | `10` | 評価ウィンドウ（取引日） |
| `BACKTEST_MIN_AGE_DAYS` | `14` | 不完全なデータを避けるため、N 日より古いレコードのみバックテスト |
| `BACKTEST_ENGINE_VERSION` | `v1` | エンジンバージョン。ロジック更新時に結果を区別するために使用 |
| `BACKTEST_NEUTRAL_BAND_PCT` | `2.0` | 中立バンドの閾値（%）。±2% をレンジ内として扱う |

### 自動実行

バックテストは日次分析フロー完了後に自動的にトリガーされます（非ブロッキング。失敗しても通知には影響しません）。API 経由で手動トリガーすることもできます。

### 評価指標

| 指標 | 説明 |
|--------|------|
| `direction_accuracy_pct` | 方向予測の精度（期待される方向が実際と一致） |
| `win_rate_pct` | 勝率（勝ち / (勝ち + 負け)、中立を除く） |
| `avg_stock_return_pct` | 平均銘柄リターン率 |
| `avg_simulated_return_pct` | 平均シミュレート執行リターン（SL/TP の決済を含む） |
| `stop_loss_trigger_rate` | 損切りトリガー率（SL が設定されたレコードのみカウント） |
| `take_profit_trigger_rate` | 利確トリガー率（TP が設定されたレコードのみカウント） |

---

## ローカル WebUI 管理画面

WebUI と FastAPI API は同じサービスプロセスを共有します。起動後、ブラウザワークスペースを使用して、設定管理、手動分析、タスク進捗、過去のレポート、バックテスト、ポートフォリオ管理、スマートインポートを行います。認証、クラウドサーバーアクセス、API 使用の詳細は以下で説明します。

### FastAPI API サービス

FastAPI は、設定管理と分析トリガーのための RESTful API サービスを提供します。

### 起動方法

| コマンド | 説明 |
|------|------|
| `python main.py --serve` | API サービスを起動 + 完全分析を一度実行 |
| `python main.py --serve-only` | API サービスのみを起動、手動で分析をトリガー |

### 機能

- **設定管理** - ウォッチリストの表示/変更
- **UI 言語切り替え** - ログインページ、シェル/ナビゲーション、設定ページ、共有コントロールで UI 言語（`zh`/`en`）を切り替え。このスイッチは `REPORT_LANGUAGE` から独立しています。
- **クイック分析** - API 経由で銘柄分析をトリガー。Home ページには、Docker/サーバーモードでバックグラウンドの大引け振り返りを開始する Market Review ボタンもあります
- **ストラテジー選択** - Home ページは分析ストラテジースキルの明示的選択をサポート。`skills` が省略された場合、分析はサーバーのデフォルトストラテジーを使用するため、レガシークライアントは既存の動作を維持します
- **初回セットアップヒント** - Home ページは読み取り専用のセットアップステータスを読み取り、プライマリ LLM チャネルやウォッチリストなどの必須項目が欠落している場合にユーザーを Settings に誘導します
- **リアルタイム進捗** - 分析タスクのステータスがリアルタイムで更新され、並列タスクをサポート。通常の銘柄分析パスは現在、LLM ステージ中に LiteLLM ストリーミングを優先し、タスク SSE を通じてより細かい `message/progress` 更新をプッシュします
- **復元可能な AlphaSift スクリーニング** - Screening ページは AlphaSift の作業をバックグラウンドタスクとして送信し、ステータスをポーリングします。これにより、ページに戻ると、スナップショット、相場、または LLM 呼び出しが遅い場合にフィードバックを失う代わりに、アクティブなタスク進捗または最終結果が復元されます
- **Market Review の可視性** - Market Review をクリックした後、API は `task_id` を返し、UI は `GET /api/v1/analysis/status/{task_id}` をポーリングして進捗を表示します。完了/失敗状態は明示的にレンダリングされ、失敗メッセージは UI のエラー領域に直接表示されます。
- **大引け振り返り履歴の専用エントリ** - 大引け振り返り履歴は専用の履歴エントリで表示され、通常の銘柄履歴から分離されます。`stock_code=MARKET` と `report_type=market_review` を使用して、大引け振り返りレコードのみを表示・リプレイします。
- **大引け振り返り履歴のリプレイ** - 大引け振り返りの結果は `report_type=market_review` で永続化され、新しい分析実行を再トリガーせずに、履歴リスト/詳細または Markdown エンドポイントから直接再オープンできます。
- **入力データブロックの可視性** - 通常分析レポートは、履歴詳細、同期レスポンス、完了タスクステータスを通じて低感度の `AnalysisContextPack` 概要を公開します。Web レポートページは、Strategy と News の後にデータブロック要約を折りたたんで表示し、展開時にブロックステータス、ソース、欠落理由、フォールバック要約を利用できます。
- **銘柄相談のフォローアップコンテキスト** - 履歴レポートから銘柄相談を開いた場合、フォローアップメッセージはアクティブな `stock_code/stock_name` を送信し続けます。既存のチャットを再オープンすると、読み込まれたユーザーメッセージから基準銘柄を復元でき、比較スタイルのプロンプトは現在の銘柄コンテキストを上書きしません。
- **バックテスト検証** - 過去の分析精度を評価し、方向勝率とシミュレートリターンをクエリ
- **API ドキュメント** - Swagger UI は `/docs` でアクセス

### 製品動作に関する注記

この機能の製品動作は次のとおりです:

- UI 言語はレポート言語から独立しています: `dsa.uiLanguage`（ブラウザ永続化）はシェル/ログイン/設定のテキストを制御し、`REPORT_LANGUAGE` はレポートテキストとレポートページの固定文言（`zh`/`en`）を制御します。
- `dsa.uiLanguage` はローカル永続化 -> ブラウザ言語 -> デフォルト `zh` に従います。
- この変更はリクエストスコープのレポート言語オーバーライドパラメータを追加するのみで、`provider`、`model`、`base_url`、または移行/クリーンアップ動作は変更しません。
- PR レベルの検証出力、スクリーンショット、コマンドログは、この使用ガイドではなく PR 説明で維持されます。

### API エンドポイント

| エンドポイント | メソッド | 説明 |
|------|------|------|
| `/api/v1/analysis/analyze` | POST | 銘柄分析をトリガー |
| `/api/v1/analysis/market-review` | POST | バックグラウンドの大引け振り返りをトリガー。リクエストボディで `{"send_notification": true}` を渡せます。`main.py --market-review` や Bot コマンドと同じ `GeminiAnalyzer/SearchService/NotificationService` の構築セマンティクスを共有します |
| `/api/v1/analysis/tasks` | GET | タスクリストをクエリ |
| `/api/v1/analysis/tasks/stream` | GET (SSE) | リアルタイムのタスク更新を購読 |
| `/api/v1/analysis/status/{task_id}` | GET | タスクステータスをクエリ |
| `/api/v1/alphasift/screen/tasks` | POST | AlphaSift スクリーニングのバックグラウンドタスクを送信（先に `ALPHASIFT_ENABLED` を有効化する必要があります） |
| `/api/v1/alphasift/screen/tasks/{task_id}` | GET | AlphaSift スクリーニングタスクのステータスと完了結果をクエリ |
| `/api/v1/history` | GET | 分析履歴をクエリ |
| `/api/v1/history/{record_id}/diagnostics` | GET | 過去レポート実行の診断要約とサニタイズされたコピーテキストをクエリ |
| `/api/v1/decision-signals` | POST | 意思決定シグナルを明示的に作成または重複排除し、`{ item, created }` を返す |
| `/api/v1/decision-signals` | GET | 銘柄、市場、action、phase、source、status、時間範囲、キャッシュのみの保有銘柄フィルターを使用したページネーション意思決定シグナルクエリ |
| `/api/v1/decision-signals/{signal_id}` | GET | 1 つの意思決定シグナルを取得し、読み取り前に遅延期限切れを適用 |
| `/api/v1/decision-signals/{signal_id}/status` | PATCH | 意思決定シグナルのステータスとオプションのメタデータを更新 |
| `/api/v1/decision-signals/latest/{stock_code}` | GET | 銘柄の最新の active 意思決定シグナルをクエリ |
| `/api/v1/usage/summary?period=today|month|all` | GET | 呼び出しタイプとモデルでグループ化した LLM 呼び出し回数とトークン使用量をクエリ |
| `/api/v1/backtest/run` | POST | バックテストをトリガー |
| `/api/v1/backtest/results` | GET | バックテスト結果をクエリ（ページネーション） |
| `/api/v1/backtest/performance` | GET | 全体のバックテストパフォーマンスを取得 |
| `/api/v1/backtest/performance/{code}` | GET | 銘柄ごとのバックテストパフォーマンスを取得 |
| `/api/health` | GET | ヘルスチェック |
| `/docs` | GET | API Swagger ドキュメント |

> 注: `POST /api/v1/analysis/analyze` は `async_mode=false` のとき 1 銘柄のみをサポートします。バッチの `stock_codes` には `async_mode=true` が必要です。非同期の `202` レスポンスは、1 銘柄の場合は単一の `task_id` を、バッチリクエストの場合は `accepted` / `duplicates` の要約を返します。
> 注: `POST /api/v1/analysis/analyze` は `skills` をストラテジー ID の配列として受け入れます。省略された場合はサーバーのデフォルトが使用されます。レガシーフィールドの `strategies` も後方互換のために引き続き受け入れられます。
> 注: `POST /api/v1/analysis/analyze` は `analysis_phase=auto|premarket|intraday|postmarket` を受け入れ、デフォルトは `auto` です。`auto` 以外はこの実行のフェーズと派生フェーズフラグのみをオーバーライドし、実際の取引カレンダーのタイムスタンプは書き換えません。受理されたレスポンス、インメモリのタスクステータス、タスクリスト、SSE はリクエストされたフェーズをエコーし、最終レポートのフェーズは `report.meta.market_phase_summary.phase` のままです。
> 注: `POST /api/v1/analysis/analyze` は `report_language=zh|en`（レガシー互換のエイリアス `reportLanguage`）を受け入れます。省略された場合はグローバルの `REPORT_LANGUAGE` にフォールバックします。このパラメータはリクエストスコープのみで、この実行のレポート出力言語に影響し、レスポンスの `report.meta.report_language` を含みます。
> 注: Web Home ページは明示的なストラテジーセレクターを公開します。ユーザーが選択しない場合、`skills` は送信されずレガシー動作が維持されます。選択された場合、このエンドポイントに渡され、タスクステータス/履歴スナップショットに永続化されます。
> 注: `POST /api/v1/analysis/market-review` は CLI/Bot の大引け振り返り（`GeminiAnalyzer(config=...)`、検索セットアップ、プロンプト/レンダリングパイプライン）と同じランタイム設定パスに従います。provider 互換性パスは `litellm_model` と `llm_model_list` を優先し、それらが設定されていない場合は既存のレガシーキー（`GEMINI_*`、`OPENAI_*`、`ANTHROPIC_*`、`DEEPSEEK_*`）にフォールバックします。provider 名、Base URL、LiteLLM ルーティングのセマンティクスはそれ以外は変更されません。
> 注: `POST /api/v1/analysis/market-review` は、そのリクエストのレポート言語を設定するために `report_language=zh|en` / `reportLanguage` も受け入れます。省略された場合はグローバルの `REPORT_LANGUAGE` にフォールバックします。Bot/CLI/手動の `/market-review` 呼び出しは引き続きグローバル設定を使用し、リクエストレベルのオーバーライドを持ちません。
> 注: `POST /api/v1/analysis/market-review` は明示的な Web/デスクトップトリガーであり、大引け振り返りタスクを直接送信します。`TRADING_DAY_CHECK_ENABLED=true` や設定された市場がその日に閉場していても短絡しません。定時ジョブ、GitHub Actions の手動実行、CLI のデフォルトは、`--force-run` またはワークフローの `force_run` が使用されない限り、引き続き取引日ゲートに従います。
> 監査ノート: 優先順位とフォールバックは `src/config.py` の `Config._load_from_env()`（`LITELLM_CONFIG` > `LLM_CHANNELS` > レガシー）によって定義されます。リグレッションカバレッジは `tests/test_llm_channel_config.py`（設定ソースのパース）と `tests/test_market_review_runtime.py`（共有ランタイムアセンブリ）にあります。エンドポイントロックはプロセス/ホストレベルのみです。マルチインスタンスデプロイでは、依然として外部の分散冪等性制御が必要です。
> 注: `/api/v1/analysis/market-review` が完了すると、レポートは `report_type=market_review` で永続化されます。分析を再実行せずに直接表示するには、`/api/v1/history` と `/api/v1/history/{record_id}`（または Markdown 履歴エンドポイント）を開きます。
> 注: `/api/v1/analysis/market-review` のレスポンスと永続化された履歴には、`market_scope`、`sections`、`sectors`、`news`、`market_light`、`indices` などのフィールドを持つ構造化された `market_review_payload` が含まれます。Web レンダリングと履歴詳細は同じ構造を使用し、構造が利用できない場合のみ生の `markdown_report` にフォールバックします。
> 注: `market_review_payload.breadth` はブレッドスデータが本当に利用可能な場合のみ出力されます。使用可能なブレッドスのない市場/フィードでは、フィールドは省略され、UI は `No data` を表示すべきです（誤解を招くゼロ値ではなく）。
> 注: `/api/v1/analysis/market-review` が `task_id` を返すと、WebUI は `GET /api/v1/analysis/status/{task_id}` をポーリングします。UI は明確な `pending/processing` 進捗をレンダリングし、ステータスが `completed` になると完了フィードバックを表示し、`failed` の場合は `error` の内容を表面化します。
> 注: 通常の銘柄履歴との混在を避けるため、`stock_code=MARKET&report_type=market_review` を指定した `GET /api/v1/history` で大引け振り返りのみの履歴をフィルタします。
> 注: `GET /api/v1/history/{record_id}/diagnostics` は履歴主キー ID または `query_id` のいずれかを受け入れ、`normal/degraded/failed/unknown` の要約、主要なパイプラインコンポーネント、サニタイズされた `copy_text` を返します。`context_snapshot.diagnostics` のない古いレポートは、通常のレポート読み取りに影響することなく `unknown` を返します。
> 注: `GET /api/v1/history` のリスト要約は、同一銘柄履歴のために `stock_code` でページネーションでき、オプションのトレンド、要約、モデル、分析時の価格/変動フィールドを含むようになりました。永続化されたスナップショットのない古い行は空の値を返します。Web レポートページの「History Trend」ドロワーはこのエンドポイントを再利用します。
> Issue #1520 の互換性に関する注記: ここで返される `model`/`model_used` は、各レコードの読み取り専用の過去スナップショットメタデータで、トレンドドロワー/履歴表示にのみ使用されます。分析パスのランタイムモデル/モデルプロバイダー/base URL の解決、設定移行、またはクリーンアップのセマンティクスを変更しません。ロールバックはこのコミットを取り消すことです。履歴クエリ、API レスポンス形状、UI ドロワーの消費は互換性を維持します。
> 注: 履歴詳細、同期分析レスポンス、完了タスクステータスレスポンスは、`report.details.analysis_context_pack_overview` で低感度の入力データブロック概要を公開します。同期分析レスポンスは永続化されたばかりの `analysis_history.context_snapshot` に依存するため、`SAVE_CONTEXT_SNAPSHOT=false` の場合、新しいレコードは概要を保証しません。`details.context_snapshot` はそのトップレベルフィールドを除去し、完全な `AnalysisContextPack` やプロンプト要約を返しません。
> 注: `POST /api/v1/agent/chat` と `POST /api/v1/agent/chat/stream` は、サーバー側の stock-scope 解決後にのみ、フロントエンドが提供する `context.stock_code` をアクティブな銘柄相談の基準として使用します。各ターンは `maintain`、`switch`、または `compare` に分類されます: 変更のないフォローアップは現在の銘柄に対してのみ銘柄スコープのツールを呼び出せます。明示的な切り替えは古い銘柄要約とプリフェッチコンテキストをクリアします。compare/vs/difference などの比較プロンプトは、現在の銘柄を書き換えずに、そのターンで明示的に言及されたコードを許可します。モデルが TTM、PE、MACD、KDJ などの金融略語、移動平均プロンプトの `MA` などのコンテキスト指標トークン、または SH/SZ/BJ/HK/SS などの取引所フラグメントで銘柄ツールを呼び出そうとした場合、バックエンドはその銘柄ツールを実行する代わりに、再試行不可の `stock_scope_violation` ツール結果を返します。ツール名は正確なレジストリ名でのみ解決されます。provider の名前空間やサフィックスは既存のツールにルーティングされません。

> 互換性監査のエビデンス:
> - 公式リファレンス: LiteLLM OpenAI-compatible provider ドキュメント <https://docs.litellm.ai/docs/providers/openai_compatible>、OpenAI Chat API <https://platform.openai.com/docs/api-reference/chat/create>、DeepSeek API ドキュメント <https://api-docs.deepseek.com/>。
> - 依存関係の境界: このリポジトリは現在 `litellm>=1.80.10,!=1.82.7,!=1.82.8,<2.0.0` を固定しています（`requirements.txt` を参照）。このパスの互換性リグレッションは、その依存関係ウィンドウ下で検証されました。
> - 検証可能なテスト:
>   - `tests/test_llm_channel_config.py`（設定の優先順位と provider/base URL のマッピング）
>   - `tests/test_market_review_runtime.py`（`build_market_review_runtime` の共有アセンブリパス）
>   - `tests/test_analysis_api_contract.py`（`/api/v1/analysis/market-review` の契約とタスクステータスフロー）
> - ロールバックパス: リグレッションが現れた場合、過去の `LITELLM_MODEL`、`LITELLM_FALLBACK_MODELS`、レガシーの `GEMINI_*` / `OPENAI_*` / `ANTHROPIC_*` / `DEEPSEEK_*` を復元するか、`POST /api/v1/system/config/import` を通じてデスクトップバックアップをインポートして再起動します。ランタイムでは、`LITELLM_CONFIG` / `LLM_CHANNELS` をクリアしてレガシーフォールバックを強制することもできます。

> 進捗ストリームに関する注記: `GET /api/v1/analysis/tasks/stream` は現在、`task_created / task_started / task_completed / task_failed` に加えて `task_progress` を出力します。通常分析パスは、相場準備、ニュース取得、コンテキストアセンブリ、LLM 生成、レポート永続化にわたって `progress` と `message` を更新します。ストリーミングチャンクはサーバー側でのみ蓄積されます。履歴は最終的な JSON が正常にパースされた後にのみ永続化されます。最初のチャンクの前にストリーミングが利用できない場合、システムは以前の非ストリームリクエストにフォールバックします。部分的な出力が既に到着した後にストリームが失敗した場合、システムはまず同じモデルで非ストリームを再試行し、その後、元の順序（プライマリ + フォールバックリスト）で既存のフォールバックモデルを通じて続行します。
> 進捗コールバックが失敗した場合、分析フローは続行され、例外は SSE 配信のギャップのトラブルシューティングを助けるために警告レベルでログに記録されるようになりました。

> 注: この動作は、詳細なランタイム SSE/フォールバック動作であり、したがって README には含まれないため、完全ガイド（`full-guide*.md`）に文書化されています。

**使用例**:
```bash
# ヘルスチェック
curl http://127.0.0.1:8000/api/health

# 分析をトリガー（A 株）
curl -X POST http://127.0.0.1:8000/api/v1/analysis/analyze \
  -H 'Content-Type: application/json' \
  -d '{"stock_code": "600519"}'

# ストラテジーリストを渡す（オプション）
curl -X POST http://127.0.0.1:8000/api/v1/analysis/analyze \
  -H 'Content-Type: application/json' \
  -d '{"stock_code": "600519", "skills": ["bull_trend", "growth_quality"]}'

# タスクステータスをクエリ
curl http://127.0.0.1:8000/api/v1/analysis/status/<task_id>

# 本日の LLM 使用量をクエリ
curl "http://127.0.0.1:8000/api/v1/usage/summary?period=today"

# バックテストをトリガー（全銘柄）
curl -X POST http://127.0.0.1:8000/api/v1/backtest/run \
  -H 'Content-Type: application/json' \
  -d '{"force": false}'

# バックテストをトリガー（特定の銘柄）
curl -X POST http://127.0.0.1:8000/api/v1/backtest/run \
  -H 'Content-Type: application/json' \
  -d '{"code": "600519", "force": false}'

# 全体のバックテストパフォーマンスをクエリ
curl http://127.0.0.1:8000/api/v1/backtest/performance

# 銘柄ごとのバックテストパフォーマンスをクエリ
curl http://127.0.0.1:8000/api/v1/backtest/performance/600519

# ページネーションされたバックテスト結果
curl "http://127.0.0.1:8000/api/v1/backtest/results?page=1&limit=20"
```

### カスタム設定

デフォルトポートを変更するか、LAN アクセスを許可します:

```bash
python main.py --serve-only --host 0.0.0.0 --port 8888
```

### サポートされる銘柄コード形式

| タイプ | 形式 | 例 |
|------|------|------|
| A 株 | 6 桁の数字 | `600519`, `000001`, `300750` |
| BSE（北京） | 8/4/92 プレフィックス、6 桁。`BJ` プレフィックスまたは `.BJ` サフィックスに対応 | `920748`, `BJ920493`, `920493.BJ` |
| 香港株 | hk + 5 桁の数字 | `hk00700`, `hk09988` |

### 注意事項

- ブラウザアクセス: `http://127.0.0.1:8000`（または設定したポート）
- 分析完了後、通知は設定済みのチャネルに自動的にプッシュされます
- この機能は GitHub Actions 環境では自動的に無効化されます

---

## FAQ

### Q: プッシュメッセージが切り捨てられますか？
A: WeChat Work/Feishu にはメッセージ長の制限があり、システムは既にメッセージを自動分割しています。完全な内容を得るには、Feishu クラウドドキュメント機能を設定してください。

### Q: データ取得に失敗しましたか？
A: AkShare はスクレイピングメカニズムを使用しており、一時的にレート制限される場合があります。システムにはリトライメカニズムが設定されているため、通常は数分待って再試行するだけです。

### Q: ウォッチリストに銘柄を追加するには？
A: `STOCK_LIST` 環境変数を変更し、複数のコードをカンマで区切ります。

### Q: GitHub Actions が実行されませんか？
A: Actions が有効になっているか、cron 式が正しいか（UTC 時間であることに注意）を確認してください。

---

## ポートフォリオ Web に関するノート

### `/portfolio` でのポートフォリオアカウントアーカイブ

- `/portfolio` のアカウントツールバーは、既存の `DELETE /api/v1/portfolio/accounts/{account_id}` エンドポイントを通じて、選択した単一アカウントを削除できます。
- アカウント削除はソフトデリート/アーカイブのセマンティクスを使用します。アーカイブされたアカウントは、デフォルトのアカウントリスト、ポートフォリオスナップショット、リスク要約、入力フォーム、イベントリストから非表示になります。
- 過去の取引、現金台帳、コーポレートアクション、日次スナップショットの行は物理的に削除されません。Web UI から特定の台帳行を修正するには、そのアカウントをアーカイブする前にその行を削除してください。

### `/portfolio` での手動 FX リフレッシュ

- Web `/portfolio` ページの FX ステータスカードには手動リフレッシュアクションが含まれます。
- ボタンは既存の `POST /api/v1/portfolio/fx/refresh` エンドポイントを呼び出し、スナップショット/リスクデータのみを再読み込みします。
- 上流の FX 取得が失敗した場合、ページはリフレッシュ後も stale のままになる可能性があり、フォールバック結果をインラインで説明します。
- `PORTFOLIO_FX_UPDATE_ENABLED=false` の場合、リフレッシュ API は明示的な無効化ステータスを返し、ページはリフレッシュ可能なペアが存在しないことを示唆する代わりに、オンライン FX リフレッシュが無効であることを表示します。
- ポートフォリオスナップショットの `positions[]` には、`price_source`、`price_date`、`price_stale`、`price_available` などの価格メタデータが含まれます。本日のスナップショットはまずリアルタイム相場を試し、リアルタイム相場が利用できないか非正の場合は、`as_of` 以前の最新の過去終値にフォールバックします。過去の `as_of` スナップショットは過去終値のセマンティクスを維持し、原価を現在価格として静かに扱うことはなくなりました。価格欠落のポジションは `price_available=false` でマークされ、時価/含み損益の合計から除外されます。

## Agent ツールのデータキャッシュと永続化

- `get_daily_history` はまずローカルの `stock_daily` 日足バーキャッシュの再利用を試みます。キャッシュが新鮮で、ダッシュボードのデフォルトである 30 レコード以上を含む場合、別の外部データソースリクエストを回避します。
- Agent がローカルキャッシュに含まれるより多くの日数を要求した場合、ツールは利用可能なレコードを返し、レスポンスに `partial_cache=true`、`requested_days`、`actual_records` をマークします。
- キャッシュが欠落しているか stale の場合、ツールは元のデータソース取得パスを維持します。成功した取得はベストエフォートで `stock_daily` に書き戻され、書き込み失敗は Agent レスポンスをブロックしません。
- `search_stock_news` と `search_comprehensive_intel` は、既存の URL / フォールバックキーの重複排除ロジックを再利用して、成功した結果をベストエフォートで `news_intel` に永続化します。
- `get_realtime_quote` は `stock_daily` をリアルタイム相場キャッシュとして使用せず、日中相場を日足バーテーブルに書き込みません。リアルタイム相場のキャッシュが必要な場合は、専用のリアルタイムストアを使用すべきです。

## Agent イベントモニター

`AGENT_EVENT_MONITOR_ENABLED=true` の場合、スケジュールモードは `AGENT_EVENT_MONITOR_INTERVAL_MINUTES` 分ごとにアラートワーカーを実行します。ワーカーは Alert API を通じて作成された有効なルールを読み取り、`AGENT_EVENT_ALERT_RULES_JSON` のレガシールールも引き続きサポートします。トリガーされたアラートは引き続き既存の通知チャネルを通じて送られます。Alert API / Web の永続化ルールは、価格、変動率、出来高、日足テクニカル指標、`watchlist`、`portfolio_holdings`、`portfolio_account`、`market` Market Light のターゲットをサポートします。レガシー JSON は引き続き 3 つの基本ルールタイプのみをサポートします。

> 互換性とロールバックに関する注記: このセクションは現在の Event Monitor のルール動作（`price_change_percent` を含む）を文書化するもので、モデル名、provider、Base URL、LiteLLM、`OPENAI_*`、`DEEPSEEK_*`、`GEMINI_*` 設定などの外部モデル/provider API のセマンティクスは変更しません。
> レガシー JSON は自動的に移行、削除、または書き換えされません。バックグラウンドアラートワーカーをロールバックするには、`AGENT_EVENT_MONITOR_ENABLED`/関連ルール設定をクリアまたは無効化してください。

| `alert_type` | 方向 | 閾値 | 説明 |
| --- | --- | --- | --- |
| `price_cross` | `above` / `below` | `price` | 現在価格が固定閾値を越える |
| `price_change_percent` | `up` / `down` | `change_pct` | 日中変動率が閾値に達する |
| `volume_spike` | - | `multiplier` | 最新の出来高が直近 20 日平均をこの倍率で超える |
| `ma_price_cross` | `above` / `below` | `window` | 日足終値が MA(window) をエッジクロスする |
| `rsi_threshold` | `above` / `below` | `period`, `threshold` | RSI が閾値をエッジクロスする |
| `macd_cross` | `bullish_cross` / `bearish_cross` | `fast_period`, `slow_period`, `signal_period` | DIF/DEA がエッジでゴールデン/デッドクロスする |
| `kdj_cross` | `bullish_cross` / `bearish_cross` | `period`, `k_period`, `d_period` | K/D がエッジでゴールデン/デッドクロスする |
| `cci_threshold` | `above` / `below` | `period`, `threshold` | CCI が閾値をエッジクロスする |
| `portfolio_stop_loss` | `mode=near|breach` | - | アカウントレベルの損切り接近または突破 |
| `portfolio_concentration` | - | - | アカウントレベルのシンボル集中度 |
| `portfolio_drawdown` | - | - | アカウントレベルの最大ドローダウンアラート |
| `portfolio_price_stale` | - | - | stale または欠落したポートフォリオ価格 |
| `market_light_status` | - | `statuses` | 現在の Market Light ステータスが設定された `red/yellow` リストに一致する |
| `market_light_score_drop` | - | `min_drop` | Market Light スコアが前取引日から少なくとも閾値だけ下落する |

例:

```env
AGENT_EVENT_MONITOR_ENABLED=true
AGENT_EVENT_MONITOR_INTERVAL_MINUTES=5
AGENT_EVENT_ALERT_RULES_JSON=[{"stock_code":"600519","alert_type":"price_cross","direction":"above","price":1800},{"stock_code":"300750","alert_type":"price_change_percent","direction":"down","change_pct":3.0},{"stock_code":"000858","alert_type":"volume_spike","multiplier":2.5}]
```

ワーカーは、評価履歴として `triggered`、`skipped`、`degraded`、`failed` の行を `alert_triggers` に書き込みます。通常の非トリガーチェックは履歴を書き込みません。DB 永続化ルールでは、`triggered` 履歴は `rule_id + target + data_source + data_timestamp` でベストエフォートで重複排除されます: 同じデータポイントへの繰り返しヒットは最も早いトリガー行を再利用し、`data_timestamp` のないレコードは重複排除されません。実際のトリガーはチャネルごとの試行を `alert_notifications` に書き込み、Alert API の永続化ルールはビジネスクールダウン状態を `alert_cooldowns` に書き込みます。永続化されたクールダウンの読み取りが失敗した場合、ワーカーは DB 障害中の繰り返し通知を避けるため、一時的にインプロセスのフィンガープリントガードにフォールバックします。レガシーの `AGENT_EVENT_ALERT_RULES_JSON` ルールは引き続きインプロセスのフィンガープリントサプレッサーを使用し、永続化されたクールダウン状態を書き込みません。通知インフラの `notification_noise.py` ガードは独立したままです。Web のルールリストは、ブラウザローカルのタイムゾーンパースの代わりに、バックエンドが提供する `cooldown_active` フラグを使用して、ルールがクールダウン中かどうかを判断します。

テクニカル指標ルールは日足終値のエッジトリガーのみを使用します。部分バー処理はサーバーローカル時刻 + 16:00 のヒューリスティックであり、市場カレンダーの精度を実装していません。`watchlist` ルールはワーカー実行ごとに `STOCK_LIST` をリフレッシュして展開し、`portfolio_holdings` はシンボルの重複排除を伴ってゼロでないスナップショットポジションを展開し、`portfolio_account` はアカウントレベルの集約評価のためにポートフォリオリスクサービスを再利用します。`market` ルールは `cn|hk|us` のターゲットのみを受け入れ、構造化された `MarketLightSnapshot` データを使用します。`trade_date` は現在の市場概要から取得され、`data_quality=unavailable` はトリガーをスキップし、非取引日は取引日ゲートによってスキップされ、`market_light_score_drop` は取引日間でのみスコアを比較します。WebUI の「Alerts」ページは、永続化ルールの管理、ワンショットのドライランテストの実行、トリガー履歴、通知試行、読み取り専用のクールダウン状態の表示ができます。バッチルールのクールダウンは親ルールの要約で、子ターゲットのクールダウン詳細はトリガー履歴を通じて確認できます。詳細な境界については [Real-Time Alert Center](alerts.md) を参照してください。

---

さらに質問がある場合は、[Issue を提出](https://github.com/ZhuLinsen/daily_stock_analysis/issues) してください
