# セキュリティ設定の詳細

初回セットアップ（`/team` 初回実行）時に、カレントディレクトリの `.claude/settings.json` が自動生成される。

---

## パーミッション設定（permissions）

### 許可（allow）

`.team/` 以下の操作は承認なしで実行できるようにする。

```
Read(.team/**)
Edit(.team/**)
Bash(mkdir [プロジェクトディレクトリ]/.team/**)
Bash(mkdir -p [プロジェクトディレクトリ]/.team/**)
Bash(mv [プロジェクトディレクトリ]/.team/**)
```

`.team/` 以外のディレクトリへのアクセスは都度確認が必要になる。

#### ファイルパスの書き方の注意（2026-09-27 検証）

Claude Code の `Read(...)` / `Edit(...)` のパスは、次のように解釈される。

| 書き方 | 意味 | 例 |
|---|---|---|
| `.team/**` | プロジェクトからの相対パス | ✅ 推奨 |
| `//Users/○○/.team/**` | 絶対パス（スラッシュ2つ） | ✅ 効く |
| `~/○○/.team/**` | ホームからのパス | ✅ 効く |
| `/Users/○○/.team/**` | **設定ファイルからの相対パス**（絶対パスではない） | ❌ 効かない |

- `Edit(...)` はファイルを作る・書き換える全ツール（Write を含む）に効く。`Write(...)` というルールはファイルの許可判定に使われない
- `autoApprove` という項目は Claude Code に存在しないため、書いても効果がない。承認なしにしたい操作は `allow` に書く
- 同じ理由で、`deny` にパスを書くときも `/Users/...` の形では禁止が効かない。`~/Documents/**` のように書く

#### v1.4.1 以前でセットアップした場合の更新方法

v1.4.1 以前は `Read([CWD]/.team/**)`（`/Users/...` の形）と `Write(...)`・`autoApprove` を使っていたため、`.team/` 内の操作でも毎回確認が出ていた。プロジェクトの `.claude/settings.json` を開き、`permissions` を次のように直す（`deny` と `sandbox` はそのまま）。

1. `allow` の `Read(/Users/…/.team/**)` を `Read(.team/**)`、`Edit(/Users/…/.team/**)` を `Edit(.team/**)` に書き換える
2. `allow` の `Write(...)` の行を削除する
3. `autoApprove` のブロックを丸ごと削除する
4. Claude Code を再起動する

### 禁止（deny）

以下の操作はブロックされる:

| 禁止コマンド | 理由 |
|---|---|
| `Read(**/.env)` | APIキーなど環境変数ファイルの読み取りを禁止 |
| `Bash(rm *)` / `Bash(rm -rf *)` / `Bash(rm -r*)` | ファイル削除を禁止（単一ファイル・再帰いずれも）。Claudeに削除権限を与えず、削除は `0_inbox/trash/` への `mv` で代替する |
| `Bash(rmdir *)` / `Bash(unlink *)` | ディレクトリ削除・リンク解除を禁止 |
| `Bash(sudo *)` | 管理者権限コマンドを禁止 |
| `Bash(chmod 777 *)` | 全開放パーミッション変更を禁止 |
| `Bash(dd *)` / `Bash(mkfs *)` | ディスク書き込み・フォーマットを禁止 |
| `Bash(curl * \| bash)` / `Bash(wget * \| bash)` | ダウンロード即実行（RCE攻撃）を禁止 |
| `Bash(git push --force *)` | 強制プッシュを禁止 |
| `Bash(killall *)` | プロセス強制終了を禁止 |
| `mcp__claude_ai_Gmail__trash_message` / `trash_thread` / `delete_label` | Claude Code純正Gmail連携でのメール削除・ゴミ箱移動を禁止 |
| `mcp__gmail-team__gmail_delete_email` / `gmail_move_to_trash` / `gmail_batch_delete` / `gmail_batch_move_to_trash` | `@monsoft/mcp-gmail` 経由でのメール削除・ゴミ箱移動を禁止 |

---

## サンドボックス設定（sandbox）

| 設定 | 値 | 内容 |
|---|---|---|
| enabled | true | サンドボックスを有効化 |
| autoAllowBashIfSandboxed | true | サンドボックス内の Bash は自動許可 |
| allowUnsandboxedCommands | false | サンドボックス外のコマンドを禁止 |
| allowedDomains | github.com, *.npmjs.org, registry.yarnpkg.com | ネットワークアクセスをこれらのドメインのみに制限 |

---

## CLAUDE.md に記載されるセキュリティルール

セットアップ時に生成される `.team/CLAUDE.md` には、以下の行動ルールが書き込まれる:

- ファイル削除の権限はClaudeに与えない。削除が必要なものは `0_inbox/trash/` へ移動する（`rm`系コマンドはdenyでブロック済み）
- Gmail連携時、メール・スレッドの削除・ゴミ箱移動は行わない（該当ツールはdenyでブロック済み）
- 知らないコマンドは実行前に日本語で内容を説明する
- ツール実行の許可を求める際は以下のリスクを % で提示する:
  - 機密情報・秘密鍵の外部流出リスク
  - データの外部サーバーへの送信リスク
  - 外部コードの不正実行リスク
  - システム設定の不正変更リスク

---

## 注意事項

これらの設定はリスクを**減らす**ための初期設定であり、完全な安全性を**保証するものではない**。特に患者データ・機密情報を扱う場合は、ご自身の組織のセキュリティポリシーに従って判断すること。
