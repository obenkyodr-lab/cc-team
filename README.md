# cc-team

Claude Code プラグイン。秘書が窓口となり、議事録・メール・学会抄録・スライド・論文要約などのチーム業務を支援する。

---

## 1. cc-team とは

`/team` コマンド一つで秘書エージェントが起動し、依頼内容を解析して適切な業務フォルダへ振り分ける。
ユーザーは秘書に話しかけるだけでよく、内部のフォルダ構成はフレームワークが担う。

医療・学術チーム（診療チーム、臨床研究グループ、学術・教育チームなど）を主な対象としており、患者情報・個人情報のプライバシールールを組み込んでいる。

---

## 2. 基本コンセプト

```
ユーザー
  ↓  /team で依頼
秘書（窓口・分類・交通整理）
  ↓  依頼に応じてルーティング
  ├── operations/   繰り返し発生する定型業務（議事録・メール）
  ├── projects/     目標のある個別プロジェクト（学会発表・研究・案件）
  └── agents/       専門知識が必要な場面で呼び出す専門担当AI
```

- **secretary** は常設の窓口。TODO・相談・メモを直接担当する
- **operations** は毎回同じ手順で繰り返される定型業務（議事録・メール）
- **projects** はゴールのある個別プロジェクト（学会発表準備・研究・特定案件）
- **agents** はoperationsやprojectsの中で専門知識が必要な局面で呼び出す専門担当AI（抄録作成・データ解析・論文執筆など）

初回セットアップ時にどの業務が必要かを選ぶと、対応するフォルダだけが作成される。

---

## 3. プラグイン構成

```
cc-team/
├── .claude-plugin/
│   └── marketplace.json        ← マーケットプレイス定義
├── plugins/
│   └── team/
│       ├── .claude-plugin/
│       │   └── plugin.json     ← プラグインメタデータ
│       └── skills/
│           └── team/
│               ├── SKILL.md    ← メインのAI指示書（全ワークフローを記述）
│               └── references/
│                   ├── operations.md          ← 業務別テンプレート集
│                   └── claude-md-template.md  ← .team/CLAUDE.md 生成テンプレート
└── README.md
```

---

## 4. セットアップ後に生成される .team/ 構造

`/team` を初めて実行するとオンボーディング（2問）が走り、回答に応じたフォルダのみが作成される。

### オンボーディング質問

**Q1: チームの種類**
- 診療チーム（外科・内科など）
- 臨床研究チーム
- 学術・教育チーム
- その他

**Q2: よく行う業務（複数選択可）**
- 日常業務（メール、議事録）→ `operations/` 内に `minutes/`、`mail/` を作成
- 学会発表（抄録、スライド）→ `agents/` 内に `abstract/`、`slide/` を作成
- 研究（臨床研究・論文作成など）→ `agents/` 内に `abstract/`、`slide/` を作成。運用開始後に研究パイプラインエージェントの追加を提案する

`operations/`・`agents/`・`projects/` の3フォルダは選択に関わらず常に作成される。

### 生成される構造

```
.team/
├── CLAUDE.md                   ← チームプロフィール・運営ルール（必ず作成）
├── secretary/                  ← 秘書室（常設・窓口）
│   ├── CLAUDE.md
│   ├── inbox/                  ← クイックキャプチャ
│   ├── todos/                  ← 日次TODO
│   │   └── YYYY-MM-DD.md
│   ├── notes/                  ← 壁打ち・相談メモ・意思決定ログ
│   └── logs/                   ← セッションログ（月1ファイル・最新が先頭）
│       └── YYYY-MM.md
├── operations/                 ← 日常業務フォルダ（常設）
│   ├── CLAUDE.md
│   ├── minutes/                ← 議事録（「日常業務」選択時のみ作成）
│   │   └── CLAUDE.md
│   └── mail/                   ← メール下書き（「日常業務」選択時のみ作成）
│       └── CLAUDE.md
├── agents/                     ← エージェントフォルダ（常設）
│   ├── CLAUDE.md
│   ├── abstract/               ← 学会抄録（「学会発表」または「研究」選択時のみ作成）
│   │   └── CLAUDE.md
│   └── slide/                  ← スライド（「学会発表」または「研究」選択時のみ作成）
│       └── CLAUDE.md
└── projects/                   ← 研究・個別案件フォルダ（常設）
    └── CLAUDE.md
```

---

## 5. 主な機能

### 秘書が直接対応するもの

| パターン | 対応 |
|---|---|
| TODO・タスク | 今日の `secretary/todos/YYYY-MM-DD.md` に追記・表示 |
| 壁打ち・相談 | 対話で深掘りし、まとまったら `secretary/notes/` に保存 |
| メモ | `secretary/inbox/` にタイムスタンプ付きで記録 |
| 朝の挨拶（おはよう） | 今日の未完了TODOを即座に表示 |
| 雑談 | 親しみやすく応答 |

### 業務への振り分け

| 業務 | トリガー | 保存先 |
|---|---|---|
| 議事録 | 会議メモ・書き起こしスクリプト | `operations/minutes/YYYY-MM-DD-[会議名].md` |
| メール | 返信・下書き・日程調整 | `operations/mail/YYYY-MM-DD-[宛先].md` |
| 学会抄録 | 抄録・abstract・文字数制限 | `agents/abstract/YYYY-MM-DD-[学会名].md` |
| スライド | 講演・学会発表・MR講演 | `agents/slide/YYYY-MM-DD-[講演名].md` |
| 論文要約 | 論文・PDF・要約して | `projects/[論文名]/summary.md` |
| 研究プロジェクト | 研究・RQ・プロトコル・論文執筆 | `projects/[研究名]/[成果物].md` |

### セッションログ

セッション終了時（「じゃあね」「おやすみ」「またね」など）に、`secretary/logs/YYYY-MM.md` へ実行内容を自動記録する。最新エントリーが先頭に追記される。

---

## 6. オプション業務・エージェント

初期セットアップ後に必要に応じて追加できる業務・エージェントの一覧。
「〇〇を追加して」と秘書に依頼するか、繰り返し同種の業務が発生すると自動提案される。

### 追加できる部署

| 部署 | フォルダ | 主な用途 |
|---|---|---|
| PM | `pm/` | プロジェクト進捗・マイルストーン・締め切り管理 |
| 経理 | `finance/` | 請求書・経費・売上管理 |

### 追加できる研究パイプラインエージェント

臨床研究・論文執筆を行うチーム向け。論文化のプロセスに沿って順番に追加できる。

| エージェント | フォルダ | 役割 |
|---|---|---|
| 情報収集 | `agents/researcher/` | 文献検索・検索式管理・背景情報の収集 |
| 論文要約・解釈 | `agents/summarizer/` | 論文の構造化要約と臨床的意義の解釈 |
| 研究デザイン | `agents/designer/` | PICO整理・デザイン選択・バイアス評価・統計戦略 |
| データ解析 | `agents/analyst/` | データクリーニング・前処理・統計解析支援 |
| 論文執筆 | `agents/author/` | 各セクション草稿・投稿規定への整形 |
| 論文査読 | `agents/reviewer/` | 査読コメント対応・改訂整理 |

---

## 7. セキュリティ設定

初回セットアップ（`/team` 初回実行）時に、カレントディレクトリの `.claude/settings.json` が自動生成され、以下のセキュリティ設定が適用される。

### パーミッション設定（permissions）

#### 許可（allow）

Claude のファイル操作を `.team/` 以下のディレクトリのみに限定する。

```
Read([プロジェクトディレクトリ]/.team/**)
Write([プロジェクトディレクトリ]/.team/**)
Edit([プロジェクトディレクトリ]/.team/**)
```

`.team/` 以外のディレクトリへのアクセスには都度確認が必要になる。

#### 禁止（deny）

以下の操作はブロックされる:

| 禁止コマンド | 理由 |
|---|---|
| `Read(**/.env)` | 環境変数ファイル（APIキー等）の読み取りを禁止 |
| `Bash(rm -rf *)` / `Bash(rm -r*)` | 再帰的削除を禁止 |
| `Bash(sudo *)` | 管理者権限コマンドを禁止 |
| `Bash(chmod 777 *)` | 全開放パーミッション変更を禁止 |
| `Bash(dd *)` / `Bash(mkfs *)` | ディスク書き込み・フォーマットを禁止 |
| `Bash(curl * \| bash)` / `Bash(wget * \| bash)` | ダウンロード即実行（RCE攻撃）を禁止 |
| `Bash(git push --force *)` | 強制プッシュを禁止 |
| `Bash(killall *)` | プロセス強制終了を禁止 |

### サンドボックス設定（sandbox）

| 設定 | 値 | 内容 |
|---|---|---|
| enabled | true | サンドボックスを有効化 |
| autoAllowBashIfSandboxed | true | サンドボックス内の Bash は自動許可 |
| allowUnsandboxedCommands | false | サンドボックス外のコマンドを禁止 |
| allowedDomains | github.com, *.npmjs.org, registry.yarnpkg.com | ネットワークアクセスをこれらのドメインのみに制限 |

### CLAUDE.md セキュリティルール

セットアップ時に生成される `.team/CLAUDE.md` には、以下の行動ルールが記載される:

- **ファイル削除前は必ず確認**を求める
- **知らないコマンドは日本語で内容を説明**してから実行する
- ツール実行の許可を求める際は以下のリスクを **%（パーセント）で提示**する:
  - 機密情報・秘密鍵の外部流出リスク
  - データの外部サーバーへの送信リスク
  - 外部コードの不正実行リスク
  - システム設定の不正変更リスク

---

## 8. インストール方法


### マーケットプレイスとして登録

`~/.claude/settings.json` に以下を追加する:

```json
{
  "extraKnownMarketplaces": [
    {
      "url": "https://raw.githubusercontent.com/[your-org]/cc-team/main/.claude-plugin/marketplace.json",
      "name": "cc-team"
    }
  ],
  "enabledPlugins": {
    "team@cc-team": true
  }
}
```

### ローカルで直接使う場合

このリポジトリをクローンし、`plugins/team/skills/team/` の内容を参照する。

---

## 9. 医療情報・プライバシールール

チーム業務には患者情報・個人情報が含まれることがある。以下を自動で適用する。

- 患者氏名・ID・診断名はイニシャル・症例番号で匿名化
- メール・抄録・スライドに患者情報が含まれる場合は送信前に確認を促す
- 不明な医療情報（薬剤量・エビデンス・統計値）は推測せず「要確認」とマーク
- 学会発表に患者情報が含まれる場合は倫理審査の確認を促す

### ⚠️ 患者データ解析に関する重要な注意事項

患者データをクラウドベースのAI（本ツールを含む）に入力する行為は、**情報漏洩のリスク**を伴います。
患者データの解析にこのツールを使用する場合は、**ご自身の責任**のもとでご判断ください。

**完全なローカルLLM（インターネット非接続の環境）での解析を強く推奨します。**

> This tool is not designed for processing identifiable patient data via cloud-based AI.
> Use of patient data is entirely at the user's own risk.
> We strongly recommend using fully local LLMs for any patient data analysis.

---

## 10. 対象チーム

- 診療チーム（外科・内科など）
- 臨床研究チーム
- 学術・教育チーム
- その他のチーム

---

## 11. オプション設定

初期セットアップには含まれないが、秘書に「〇〇したい」と話しかけることで設定できるオプション機能。

### 11.1 起動の自動化

現状の起動フロー: VS Code でプロジェクトを開く → ターミナルを開く → `claude` を実行 → `/team` を入力

「起動を自動化して」と秘書に伝えると、`.vscode/tasks.json` を生成する。設定後は VS Code でプロジェクトを開くだけで Claude Code が自動起動する。

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Start Team AI",
      "type": "shell",
      "command": "claude",
      "runOptions": { "runOn": "folderOpen" },
      "presentation": { "reveal": "always", "panel": "new", "focus": true }
    }
  ]
}
```

> VS Code で「タスクを自動実行してよいか」の確認ダイアログが出た場合は「許可」を選択する。

### 11.2 Gmail 連携

「Gmail を連携したい」と秘書に伝えると、サブスクリプションの Google アカウントと連携先が同じかどうかを確認して、いずれかの手順を案内する。

**シナリオ1: Claude と同じ Google アカウントの Gmail（簡単）**

Claude Code デスクトップアプリの設定 → **Integrations** → Gmail を有効化して認証するだけ。

**シナリオ2: 別の Google アカウントの Gmail**

`@monsoft/mcp-gmail` パッケージを使った設定が必要。秘書が以下の手順を案内する:

1. Google Cloud Console でプロジェクト作成・Gmail API 有効化・OAuth クライアントID（デスクトップアプリ）を作成・JSON をダウンロード
2. ダウンロードした JSON を `~/.gmail-mcp/gmail-oauth.json` に保存
3. MCP サーバーを登録:
   ```bash
   claude mcp add-json gmail-team '{"type":"stdio","command":"npx","args":["-y","@monsoft/mcp-gmail","--oauth-path","~/.gmail-mcp/gmail-oauth.json","--credentials-path","~/.gmail-mcp/gmail-token.json"]}'
   ```
4. 初回認証（ブラウザが開いて Gmail アカウントでログイン）:
   ```bash
   npx @monsoft/mcp-gmail --oauth-path ~/.gmail-mcp/gmail-oauth.json --credentials-path ~/.gmail-mcp/gmail-token.json
   ```
5. Claude Code を再起動して動作確認

**トークン切れ時の対応**

「メールが取得できない」エラーが出た場合、初回認証と同じコマンドを再実行してブラウザで再ログインする。

### 11.3 自動メールチェック

Gmail 連携後に有効にできる。「毎回自動でメールをチェックして」と秘書に伝えると設定される。

セッション開始時（または朝の挨拶時）に前回チェック以降のメールを自動取得・分類・表示する:

| 分類 | 内容 | 対応 |
|------|------|------|
| 🔴 要返信 | 個人・企業からの直接メッセージ | 返信案を作成して Gmail 下書き保存。TODO に追加 |
| 🟡 要対応 | 書類提出・フォーム回答・期限付き手続き | TODO に追加 |
| 🟢 情報把握 | 学会案内・論文通知・お知らせ | 口頭共有のみ。TODO には追加しない |
| ⚪ 不要 | ニュースレター・営業メール | 報告しない |

CC・BCC の全体配信メールは返信不要なことが多いため、慎重に判断する。
