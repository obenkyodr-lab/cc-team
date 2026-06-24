# cc-team

Claude Code 用プラグイン。`/team` を実行するだけで秘書AIが起動し、TODO管理・メール下書き・議事録作成・学会抄録・スライド整理などを自然な会話でこなせる。医療・研究・教育チームの業務を想定して設計されており、患者情報・個人情報への配慮ルールを最初から組み込んでいる。

---

## まず試す

1. Claude Code でプロジェクトフォルダを開く
2. `/team` を実行する
3. 3問の質問（チームの名前・種類・よく行う業務）に答える → 自動でフォルダが生成される
4. 「おはよう」または `/morning` を実行する → TODO・メール・予定が一覧表示される
5. 「今日やることを整理して」と依頼する → `secretary/todos/YYYY-MM-DD.md` に保存される

**「動いた」と感じたら、あとは話しかけるだけ。**

---

## できること

- **モーニングルーティン（`/morning`）** — TODO・メール・今日の予定を一覧表示。Gmail・Calendar連携済みなら自動分類まで実行
- **TODO管理** — 話しかけるだけで今日のタスクを整理・追記
- **0_inboxトレイ** — ファイルを `0_inbox/` に置いて「入れました」と言うだけで自動処理・振り分け
- **振り分け早見表** — 依頼の種類ごとに振り分け先と参照すべきCLAUDE.mdを明示。作業ミスを防ぐ
- **メール下書き** — 状況を説明するだけで件名・本文を作成
- **議事録作成** — 会議メモや書き起こしテキストから構造化された議事録を生成
- **学会抄録** — 背景・方法・結果・結論を整理、文字数調整にも対応
- **スライド整理** — 講演・学会発表のスライド構成と読み原稿を作成
- **論文要約** — PDFや論文テキストを構造化要約
- **研究プロジェクト管理** — 案件ごとに専用フォルダを作成して成果物を整理
- **社長室（CEO）管理** — YouTube要約・読書メモ・思考整理をceoフォルダに自動保存
- **二層ログ** — 各フォルダの詳細ログ＋秘書の月次ログで作業履歴を自動管理
- **Gmail連携**（オプション） — メールを自動分類・返信下書きを作成（[詳細](docs/gmail.md)）

---

## インストール

`~/.claude/settings.json` に以下を追加する:

```json
{
  "extraKnownMarketplaces": [
    {
      "url": "https://raw.githubusercontent.com/obenkyodr-lab/cc-team/main/.claude-plugin/marketplace.json",
      "name": "cc-team"
    }
  ],
  "enabledPlugins": {
    "team@cc-team": true
  }
}
```

詳しいセットアップ手順 → [docs/setup.md](docs/setup.md)

---

## 生成される構造

`/team` を初めて実行したとき、以下のフォルダが**すべて自動生成**される。

```
.team/
├── CLAUDE.md                  ← チームプロフィール・運営ルール
├── 0_inbox/                   ← ファイル投入トレイ（「入れました」で自動処理）
├── secretary/                 ← 窓口。TODO・メモ・相談はすべてここに
│   ├── todos/                 ← 日次TODO（YYYY-MM-DD.md）
│   ├── notes/                 ← メモ・アイデア・壁打ち・意思決定ログ
│   └── logs/                  ← セッションログ（月1ファイル）
├── operations/                ← 繰り返し発生する定型業務
│   ├── minutes/               ← 議事録
│   │   └── logs/              ← 議事録作業の詳細ログ
│   └── mail/                  ← メール下書き
│       └── logs/              ← メール作業の詳細ログ
├── agents/                    ← 専門業務エージェント
│   ├── abstract/              ← 学会抄録
│   │   └── logs/              ← 抄録作業の詳細ログ
│   └── slide/                 ← スライド
│       └── logs/              ← スライド作業の詳細ログ
├── projects/                  ← 研究・個別案件（プロジェクトごとにサブフォルダ）
│   ├── project-a/             ← 例：プロジェクトごとにサブフォルダを作成
│   │   ├── overview.md        ← 目標・進捗
│   │   └── log.md             ← このプロジェクトの作業ログ
│   └── project-b/
│       ├── overview.md
│       └── log.md
└── ceo/                       ← 個人の情報収集・学習・思考整理
    ├── videos/                ← YouTube・動画要約（_index.mdにインデックス）
    └── notes/                 ← 読書メモ・記事要約・人脈メモ
```

運用しながら「PM部署を追加して」「研究エージェントを追加して」と話しかけると、必要なフォルダが追加されていく。

---

## 振り分けルール

依頼の種類によって、秘書が自動的に適切なフォルダへ振り分ける。作業前に必ず「作業前に読むCLAUDE.md」列のファイルを読んでから着手する。

| 依頼の種類 | 振り分け先 | 作業前に読むCLAUDE.md |
|---|---|---|
| 議事録作成 | `operations/minutes/` | `operations/minutes/CLAUDE.md` |
| メール下書き・返信 | `operations/mail/` | `operations/mail/CLAUDE.md` |
| 学会抄録 | `agents/abstract/` | `agents/abstract/CLAUDE.md` |
| スライド・発表準備 | `agents/slide/` | `agents/slide/CLAUDE.md` |
| データ解析 | `agents/analyst/` | `agents/analyst/CLAUDE.md` |
| 論文要約・文献整理 | `agents/summarizer/` | `agents/summarizer/CLAUDE.md` |
| 研究デザイン・RQ設計 | `agents/designer/` | `agents/designer/CLAUDE.md` |
| 論文執筆 | `agents/author/` | `agents/author/CLAUDE.md` |
| 査読・コメント対応 | `agents/reviewer/` | `agents/reviewer/CLAUDE.md` |
| 優先順位・進捗管理 | `pm/` | `pm/CLAUDE.md` |
| 経費・請求書管理 | `finance/` | `finance/CLAUDE.md` |
| 個人メモ・学習・動画要約 | `ceo/` | `ceo/CLAUDE.md` |
| 研究案件・個別プロジェクト | `projects/[研究名]/` | `projects/CLAUDE.md` |

---

## スキル一覧

| スキル | 起動方法 | 概要 |
|---|---|---|
| `/team` | `/team` と入力 | メインスキル。秘書AIが起動し、すべての業務を受け付ける |
| `/morning` | `/morning` または「おはよう」 | モーニングルーティン。TODO・メール・予定を一括表示 |

---

## 安全性について

初回セットアップ時に `.claude/settings.json` が自動生成され、以下の設定が適用される:

- **操作範囲を `.team/` 以下に限定** — それ以外へのアクセスは都度確認が必要
- **危険コマンドをブロック** — `rm -rf`・`sudo`・`curl | bash`・`git push --force` など
- **承認不要操作を自動設定** — `.team/` 内のフォルダ作成・移動・`.md` ファイルの作成・編集は承認なしで実行

自動生成される `settings.json` の `autoApprove` 設定:

```json
{
  "permissions": {
    "autoApprove": [
      "Write([CWD]/.team/**/*.md)",
      "Edit([CWD]/.team/**/*.md)",
      "Bash(mkdir [CWD]/.team/**)",
      "Bash(mkdir -p [CWD]/.team/**)",
      "Bash(mv [CWD]/.team/**)"
    ]
  }
}
```

詳細な設定値 → [docs/security.md](docs/security.md)

---

## 医療情報・個人情報に関する注意

### このツールでやらないこと

- 患者個人情報をクラウドAIに入力すること
- AI出力を未確認のまま臨床・研究・対外文書に使うこと
- 医学的判断をAIに委ねること

### 情報の正確性ルール

不明な情報（ID・番号・固有名詞・数値など）は推測・捏造しない。確認できない値は `NA` または空欄のままにし、ユーザーに確認を促す。

### 免責事項

本ツールは現状有姿（as-is）で提供されており、いかなる明示または黙示の保証も行いません。本ツールの使用によって生じた以下の損害について、作者および関係者は一切の責任を負いません。

**認証情報・機密情報の漏洩**
APIキー・パスワード・秘密鍵などの認証情報をファイルやプロンプトに含めた結果として生じた漏洩・不正利用・課金被害は、すべて利用者の責任となります。認証情報はコードやMarkdownファイルに直接記載しないでください。

**PCシステムへの影響**
本ツールの設定ミス・操作ミス・想定外の動作によって生じたファイルの消失、システム設定の変更、ソフトウェアの破損その他PC環境への影響について、作者は責任を負いません。実行前に重要データのバックアップを取ることを強く推奨します。

**患者データ・個人情報の漏洩**
患者氏名・ID・診断名・治療内容その他の個人情報をクラウドベースのAI（Claude等）に入力することで生じた情報漏洩について、作者は一切の責任を負いません。患者データを扱う場合は、インターネット非接続のローカルLLMを使用するか、所属機関のプライバシーポリシーおよび関連法令（個人情報保護法・医療法等）に従って利用者自身が安全性を確保してください。

**AI出力の正確性**
本ツールを通じてAIが生成した文書・数値・判断内容の正確性は保証されません。臨床判断・研究成果・対外文書への適用は、必ず利用者自身が内容を確認・責任を持って行ってください。

**本ツールの使用はすべて利用者自身の判断と責任において行われるものとします。**

---

## 詳細ドキュメント

| ドキュメント | 内容 |
|---|---|
| [docs/setup.md](docs/setup.md) | 詳しいインストール手順・初回設定 |
| [docs/security.md](docs/security.md) | セキュリティ設定の詳細（permissions / sandbox / deny） |
| [docs/gmail.md](docs/gmail.md) | Gmail連携の手順 |
| [docs/examples.md](docs/examples.md) | 具体的な使い方の例 |
