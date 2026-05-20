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
  ├── operations/   日常業務（議事録・メール）
  ├── agents/       学会発表（抄録・スライド）
  └── projects/     本格的な研究・個別案件
```

- **secretary** は常設の窓口。TODO・相談・メモを直接担当する
- **operations** は毎回同じ種類の定型作業（議事録・メール）
- **agents** は専門知識が必要な学会発表業務（抄録・スライド）
- **projects** は案件ごとに専用フォルダで管理する個別作業（論文要約・研究プロジェクト）

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
- 日常業務（メール、議事録）→ `operations/` を作成
- 学会発表（抄録、スライド）→ `agents/` を作成
- 本格的な研究（アイデア整理、論文作成）→ `projects/` を作成

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
├── operations/                 ← 「日常業務」選択時のみ作成
│   ├── CLAUDE.md
│   ├── minutes/                ← 議事録
│   │   └── CLAUDE.md
│   └── mail/                   ← メール下書き
│       └── CLAUDE.md
├── agents/                     ← 「学会発表」選択時のみ作成
│   ├── CLAUDE.md
│   ├── abstract/               ← 学会抄録エージェント
│   │   └── CLAUDE.md
│   └── slide/                  ← スライドエージェント
│       └── CLAUDE.md
└── projects/                   ← 「本格的な研究」選択時のみ作成
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

## 6. セキュリティ設定

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
  - 認証情報・秘密鍵などの機密情報が外部に流出するリスク
  - 入力データや内部情報が第三者サーバーに送信されるリスク
  - 意図しない外部コードが自動実行されるリスク
  - システム設定やファイルが意図せず変更されるリスク

---

## 7. インストール方法


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

## 8. 医療情報・プライバシールール

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

## 9. 対象チーム

- 外科・内科などの診療チーム
- 臨床研究グループ
- 学会・論文執筆チーム
- 学術・教育チーム
