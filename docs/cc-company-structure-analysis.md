# cc-company プラグイン構造分析

作成日: 2026-05-20
目的: cc-team プラグイン自作のためのリファレンス

---

## 1. リポジトリ全体構成

```
cc-company/                          ← GitHubリポジトリルート
├── .claude-plugin/                  ← マーケットプレース定義（リポジトリレベル）
├── .github/workflows/               ← CI/CD（公開・テスト自動化）
├── docs/                            ← ドキュメント
├── packages/dashboard/              ← Webダッシュボード（npx cc-company-dashboard）
├── plugins/company/                 ← プラグイン本体（複数プラグインの場合は複数フォルダ）
│   ├── .claude-plugin/
│   │   └── plugin.json              ← プラグインメタ情報
│   └── skills/company/              ← スキル定義（コマンド名がフォルダ名）
│       ├── SKILL.md                 ← スキルのメインロジック（AIへの指示書）
│       └── references/              ← 補助コンテキストファイル
│           ├── claude-md-template.md
│           └── departments.md
├── tests/                           ← テスト
├── package.json
└── README.md
```

---

## 2. キーファイルの役割

### 2-1. `plugins/company/.claude-plugin/plugin.json`

プラグインのメタ情報。Claude Codeがプラグインを識別するために使用。

```json
{
  "name": "company",
  "description": "秘書から始める仮想組織スキル。3ステップで即運用開始、部署は必要に応じて自然に増える。",
  "version": "2.0.0"
}
```

**ポイント:**
- `name`: スキルのID。`/company:company` の2番目の `company` に対応
- `version`: セマンティックバージョニング。キャッシュパスに使われる（`~/.claude/plugins/cache/cc-company/company/2.0.0/`）

---

### 2-2. `plugins/company/skills/company/SKILL.md`

**最重要ファイル。スキル全体のロジックをAIへの指示（マークダウン）で記述する。**

構成:
```
# スキル名

## いつ使うか（トリガー条件）

## ワークフロー
  ### Step 1: 検出とモード判定
  ### Step 2: オンボーディング（AskUserQuestion使用）
  ### Step 3: ファイル生成

## 運営モード（起動後の動作）
  - 基本フロー
  - 秘書が直接対応するもの
  - 部署への振り分け基準
  - MCP連携の提案

## 運用ルール（ファイル命名・追記ルールなど）

## ファイル参照（referencesへのポインタ）

## 重要な注意事項
```

**設計思想:**
- コードではなくマークダウンで「AIへの指示」を書く
- ステートマシン的に「モード」で分岐（オンボーディング / 運営 / マイグレーション）
- `AskUserQuestion` ツールでインタラクティブなオンボーディングを実現
- `references/` ファイルを参照して動的にテンプレートを生成

---

### 2-3. `references/claude-md-template.md`

`.company/CLAUDE.md` を生成するためのテンプレート。

変数プレースホルダー:
| 変数 | 内容 |
|------|------|
| `{{BUSINESS_TYPE}}` | Q1（事業・活動）の回答 |
| `{{GOALS_AND_CHALLENGES}}` | Q2（目標・困りごと）の回答 |
| `{{CREATED_DATE}}` | 作成日 |
| `{{ADDITIONAL_DEPARTMENTS}}` | 追加部署のディレクトリツリー |
| `{{DEPARTMENT_TABLE_ROWS}}` | 追加部署のテーブル行 |
| `{{PERSONALIZATION_NOTES}}` | ユーザー状況に応じたメモ |

---

### 2-4. `references/departments.md`

**2つのセクションを持つ複合ファイル:**

1. **`_template.md` 群**: 各部署フォルダに配置するファイルテンプレート（TODO・Inbox・メモ・請求書など）
2. **`CLAUDE.md` テンプレート群**: 各部署の振る舞い・ルールを定義するCLAUDE.md

対応部署（デフォルト）:
- secretary（常設）
- pm / research / marketing / engineering / finance / sales / creative / hr
- 汎用テンプレート（カスタム部署用フォールバック）

---

## 3. Claude Codeとのインテグレーション仕組み

### 3-1. マーケットプレース登録（ユーザー側設定）

```json
// ~/.claude/settings.json
{
  "extraKnownMarketplaces": {
    "cc-company": {
      "source": { "source": "github", "repo": "Shin-sibainu/cc-company" }
    }
  },
  "enabledPlugins": {
    "company@cc-company": true
  }
}
```

### 3-2. キャッシュパス

```
~/.claude/plugins/cache/{marketplace-id}/{plugin-name}/{version}/
例: ~/.claude/plugins/cache/cc-company/company/2.0.0/
```

### 3-3. スキル呼び出し

- `/company` または `/company:company` でSKILL.mdが実行される
- `{marketplace-id}:{skill-name}` が完全修飾名

---

## 4. データフロー（スキル実行時）

```
ユーザー: /company
    ↓
SKILL.md 読み込み
    ↓
Step 1: .company/ 存在チェック
    ↓
[初回] AskUserQuestion（Q1〜Q3）
    ↓
references/claude-md-template.md 読み込み
references/departments.md 読み込み
    ↓
.company/CLAUDE.md 生成（変数置換）
.company/secretary/ フォルダ群 生成
    ↓
[2回目以降] .company/CLAUDE.md 読み込み → 運営モード
    ↓
ユーザーの入力を判断 → 秘書が対応 or 部署フォルダに書き込み
```

---

## 5. cc-teamの現状構造（2026-05-20時点）

cc-teamはcc-companyとは**異なるアーキテクチャ**を採用している。

### 5-1. 設計コンセプトの違い

| 項目 | cc-company | cc-team |
|------|-----------|---------|
| **ユースケース** | 個人の仮想組織・タスク管理 | 医療・学術業務の実務支援 |
| **起動コマンド** | `/company` | `/aiteam` |
| **ロジック集約** | SKILL.md 1ファイルに全ワークフロー | Secretary + Agent + SKILL に分散 |
| **状態管理** | `.company/` フォルダ | `projects/` フォルダ（案件単位） |
| **ルーティング** | SKILL.md 内で直接判断 | `secretary/routing.md` で明示的に定義 |
| **参照ファイル** | `references/` にテンプレート集約 | `shared/` に共通ルール（全Agent参照） |
| **プラグイン化** | 完成・公開済み (v2.0.0) | 未完成（スタブ状態） |

### 5-2. cc-teamの実際のフォルダ構成

```
cc-team/                              ← GitHubリポジトリ（.git管理済み）
├── plugin/
│   └── commands/
│       └── aiteam.md                 ← /aiteam コマンドのエントリーポイント（スタブ）
├── secretary/
│   ├── agent.md                      ← 秘書エージェント（完成）
│   └── routing.md                    ← 振り分けルール表（完成）
├── agents/
│   ├── abstract/
│   │   └── agent.md                  ← 学会抄録エージェント（スタブ）
│   └── slide/
│       └── agent.md                  ← スライド構成エージェント（スタブ）
├── operations/
│   ├── minutes/SKILL.md              ← 議事録作成（スタブ）
│   ├── mail/SKILL.md                 ← メール処理（スタブ）
│   ├── abstract/SKILL.md             ← 学会抄録作成（スタブ）
│   └── slide/SKILL.md                ← スライド構成作成（スタブ）
├── projects/
│   └── _project_template/
│       ├── PROJECT.md                ← 案件テンプレート（スタブ）
│       └── pm_agent.md               ← PMエージェント（スタブ）
├── shared/
│   ├── privacy_rules.md              ← プライバシールール（スタブ）
│   └── style_guide.md                ← 文体・出力ガイド（スタブ）
└── docs/
    └── cc-company-structure-analysis.md  ← この文書
```

### 5-3. cc-teamのデータフロー

```
ユーザー: /aiteam <依頼>
    ↓
plugin/commands/aiteam.md（エントリー）
    ↓
secretary/agent.md（受付・分類）
    ↓ routing.md を参照
    ├── 定型業務 → operations/<name>/SKILL.md
    └── 判断必要 → agents/<name>/agent.md → operations/<name>/SKILL.md
         ↑ 共通参照: shared/privacy_rules.md, shared/style_guide.md
         ↑ 案件固有: projects/<案件名>/PROJECT.md
```

---

## 6. cc-teamをClaude Codeプラグインにするために必要なもの

cc-companyのプラグイン構造と照合した**ギャップ分析**:

### 6-1. 必須: Claude Codeプラグイン定義ファイル

cc-companyの `plugins/company/.claude-plugin/plugin.json` に相当するものが**ない**。

**必要な追加:**
```json
// plugins/team/.claude-plugin/plugin.json
{
  "name": "team",
  "description": "AIチームで医療・学術業務を支援するフレームワーク",
  "version": "1.0.0"
}
```

### 6-2. 必須: SKILL.md（エントリーポイントの整理）

現在の `plugin/commands/aiteam.md` はほぼ空のスタブ。  
cc-companyの `skills/company/SKILL.md` のように、**完全なワークフロー指示**を書く必要がある。

**現状の `aiteam.md`（3行のみ）:**
```
# aiteam コマンド
AIチームへの指示を受け付けるエントリーポイント。
秘書エージェントにルーティングし、適切な担当エージェントへ振り分ける。
```

**必要な内容:**
- いつ使うか（トリガー条件）
- ワークフロー（Secretary → routing → Agent/SKILL）
- 各ファイルの読み込み順序
- エラー時・未分類時の対応

### 6-3. 必須: マーケットプレース定義

リポジトリルートの `.claude-plugin/` ディレクトリ（未作成）。  
ユーザーの `settings.json` にマーケットプレース登録することでインストール可能になる。

### 6-4. スタブ状態のファイル（中身を書く必要あり）

| ファイル | 優先度 | 内容 |
|---------|-------|------|
| `plugin/commands/aiteam.md` | 🔴 最高 | エントリーポイント（フル実装が必要） |
| `operations/minutes/SKILL.md` | 🔴 高 | 議事録生成の具体的な手順 |
| `operations/mail/SKILL.md` | 🔴 高 | メール下書きの具体的な手順 |
| `shared/privacy_rules.md` | 🟡 中 | 医療情報取り扱いルール |
| `shared/style_guide.md` | 🟡 中 | 文体・フォーマットルール |
| `agents/abstract/agent.md` | 🟡 中 | 抄録エージェントの詳細指示 |
| `agents/slide/agent.md` | 🟡 中 | スライドエージェントの詳細指示 |
| `operations/abstract/SKILL.md` | 🟢 低 | 抄録生成手順 |
| `operations/slide/SKILL.md` | 🟢 低 | スライド生成手順 |
| `projects/_project_template/PROJECT.md` | 🟢 低 | 案件テンプレート |

---

## 7. 推奨実装ロードマップ

### Phase 1: プラグイン骨格（最優先）
- [ ] `plugins/team/.claude-plugin/plugin.json` 作成
- [ ] `plugin/commands/aiteam.md` をフル実装（secretary → routing → 処理フロー全体）
- [ ] `.claude-plugin/` ルート定義作成

### Phase 2: コア機能（MVP）
- [ ] `operations/minutes/SKILL.md` 実装（議事録：最も使用頻度が高い）
- [ ] `operations/mail/SKILL.md` 実装（メール処理）
- [ ] `shared/privacy_rules.md` 実装（医療情報保護）
- [ ] `shared/style_guide.md` 実装

### Phase 3: エージェント（拡張）
- [ ] `agents/abstract/agent.md` 実装
- [ ] `agents/slide/agent.md` 実装
- [ ] `operations/abstract/SKILL.md` 実装
- [ ] `operations/slide/SKILL.md` 実装

### Phase 4: プラグイン公開
- [ ] GitHubリポジトリ公開
- [ ] `settings.json` へのマーケットプレース登録テスト
- [ ] README.md にインストール手順追記
