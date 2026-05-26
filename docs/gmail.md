# Gmail 連携

メールの自動チェック・返信下書きの作成が可能になる。

---

## シナリオ1: Claude と同じ Google アカウントの Gmail（簡単）

1. Claude Code **デスクトップアプリ**を開く
2. 左下の設定アイコン → **Integrations** → Gmail を有効化
3. Google アカウントで認証する
4. 完了。「メールを確認して」と話しかけて動作確認する

---

## シナリオ2: 別の Google アカウントの Gmail を使う場合

`@monsoft/mcp-gmail` パッケージを使って接続する。

### Step 1: Google Cloud Console で認証情報を作成

1. [Google Cloud Console](https://console.cloud.google.com) にアクセス
2. 新規プロジェクトを作成（例: `gmail-mcp`）
3. 「APIとサービス」→「ライブラリ」→ **Gmail API** を検索して有効化
4. 「認証情報」→「OAuthクライアントIDを作成」→ アプリの種類: **デスクトップアプリ**
5. 作成された JSON ファイルをダウンロード

### Step 2: 認証ファイルを保存

```bash
mkdir -p ~/.gmail-mcp
mv ~/Downloads/client_secret_*.json ~/.gmail-mcp/gmail-oauth.json
```

### Step 3: MCP サーバーを登録

```bash
claude mcp add-json gmail-team '{"type":"stdio","command":"npx","args":["-y","@monsoft/mcp-gmail","--oauth-path","~/.gmail-mcp/gmail-oauth.json","--credentials-path","~/.gmail-mcp/gmail-token.json"]}'
```

### Step 4: 初回認証

```bash
npx @monsoft/mcp-gmail --oauth-path ~/.gmail-mcp/gmail-oauth.json --credentials-path ~/.gmail-mcp/gmail-token.json
```

ブラウザが開いたら、連携したい Gmail アカウントでログインする。「認証完了」と表示されたら成功。

### Step 5: Claude Code を再起動して動作確認

「メールを確認して」と話しかけて動作確認する。

---

## トークン切れ時の対応

「メールが取得できない」「認証エラー」が出た場合は、Step 4 と同じコマンドを再実行してブラウザで再ログインする。

---

## 自動メールチェックの設定

Gmail 連携後、「毎回自動でメールをチェックして」と秘書に伝えると、セッション開始時（または朝の挨拶時）に自動でメールを取得・分類・表示するようになる。

| 分類 | 内容 | 対応 |
|---|---|---|
| 🔴 要返信 | 個人・企業からの直接メッセージ | 返信案を作成して Gmail 下書き保存。TODOに追加 |
| 🟡 要対応 | 書類提出・フォーム回答・期限付き手続き | TODOに追加 |
| 🟢 情報把握 | 学会案内・論文通知・お知らせ | 口頭共有のみ |
| ⚪ 不要 | ニュースレター・営業メール | 報告しない |

CC・BCCの全体配信メールは返信不要なことが多いため、慎重に判断する。
