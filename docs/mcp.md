# MCP連携（Gmail・Google Calendar）

メール・カレンダーとの連携、および接続エラー時の対処方法をまとめる。

---

## Gmail連携

メールの自動チェック・返信下書きの作成が可能になる。

### シナリオ1: Claude と同じ Google アカウントの Gmail（簡単）

1. Claude Code **デスクトップアプリ**を開く
2. 左下の設定アイコン → **Integrations** → Gmail を有効化
3. Google アカウントで認証する
4. 完了。「メールを確認して」と話しかけて動作確認する

### シナリオ2: 別の Google アカウントの Gmail を使う場合

`@monsoft/mcp-gmail` パッケージを使って接続する。

#### Step 1: Google Cloud Console で認証情報を作成

1. [Google Cloud Console](https://console.cloud.google.com) にアクセス
2. 新規プロジェクトを作成（例: `gmail-mcp`）
3. 「APIとサービス」→「ライブラリ」→ **Gmail API** を検索して有効化
4. 「認証情報」→「OAuthクライアントIDを作成」→ アプリの種類: **デスクトップアプリ**
5. 作成された JSON ファイルをダウンロード

#### Step 2: 認証ファイルを保存

```bash
mkdir -p ~/.gmail-mcp
mv ~/Downloads/client_secret_*.json ~/.gmail-mcp/gmail-oauth.json
```

#### Step 3: MCP サーバーを登録

```bash
claude mcp add-json gmail-team '{"type":"stdio","command":"npx","args":["-y","@monsoft/mcp-gmail","--oauth-path","~/.gmail-mcp/gmail-oauth.json","--credentials-path","~/.gmail-mcp/gmail-token.json"]}'
```

#### Step 4: 初回認証

```bash
npx @monsoft/mcp-gmail --oauth-path ~/.gmail-mcp/gmail-oauth.json --credentials-path ~/.gmail-mcp/gmail-token.json
```

ブラウザが開いたら、連携したい Gmail アカウントでログインする。「認証完了」と表示されたら成功。

#### Step 5: Claude Code を再起動して動作確認

「メールを確認して」と話しかけて動作確認する。

### 既知の制限：長いスレッドから一部メールが見えないことがある

スレッド検索系のツール（`search_threads`など）は、メッセージ数の多いスレッド（9通程度で確認）で**先頭5件程度しか取得できず、後半のメールを「存在しない」と誤答することがある**。

- 「このメールがあるはずなのに出てこない」と言われたら、検索結果を鵜呑みにせず、該当スレッドを`get_thread`（スレッドを個別に取得するツール）で取り直して全件確認する
- **経緯・行き違いの多いやり取りを長いスレッドから確認するときは、最初から`search_threads`だけに頼らず`get_thread`も併用する**（後から気づくより先に潰しておく方が早い）

### 自動メールチェックの設定

Gmail 連携後、「毎回自動でメールをチェックして」と秘書に伝えると、セッション開始時（または朝の挨拶時）に自動でメールを取得・分類・表示するようになる。

| 分類 | 内容 | 対応 |
|---|---|---|
| 🔴 要返信 | 個人・企業からの直接メッセージ | 返信案を作成して Gmail 下書き保存。TODOに追加 |
| 🟡 要対応 | 書類提出・フォーム回答・期限付き手続き | TODOに追加 |
| 🟢 情報把握 | 学会案内・論文通知・お知らせ | 口頭共有のみ |
| ⚪ 不要 | ニュースレター・営業メール | 報告しない |

CC・BCCの全体配信メールは返信不要なことが多いため、慎重に判断する。

---

## Google Calendar連携

1. Claude Code **デスクトップアプリ**を開く
2. 左下の設定アイコン → **Integrations** → Google Calendar を有効化
3. Google アカウントで認証する
4. 完了。「今日の予定を確認して」と話しかけて動作確認する

---

## MCP接続が切れた・エラーが出たときの対応

Gmail・Google Calendarとも、一度接続してもセッションの途中や翌日以降に**接続が切れる／エラーを返す**ことがある。この状態を「連携未設定」と混同すると、「予定が確認できない」「メールが見当たらない」という行き違いが起きる。

### 見分け方

- 「連携方法を案内される」→ そもそも未接続
- 「一覧が空で返ってくる／エラーメッセージが出る／さっきまで動いていたのに動かない」→ 接続切れの可能性が高い

### 対応手順

1. インタラクティブセッション内であれば `/mcp` コマンドでMCPサーバーの接続状態を確認する
2. CLIからは `claude mcp list` で一覧、`claude mcp get <name>` で個別のサーバー状態を確認できる
3. **claude.aiのコネクタ経由（デスクトップアプリのIntegrations）の場合**：Integrations設定画面から該当サービスを無効化→再度有効化し、再認証する
4. **`@monsoft/mcp-gmail`など独自MCPサーバー経由の場合（トークン切れ）**：Step 4と同じ初回認証コマンドを再実行してブラウザで再ログインする

「メールが確認できない」「予定が確認できない」状態に気づいたら、黙ってスキップせず、上記のどちらに該当するかをユーザーに伝えた上で再接続手順を案内する。
