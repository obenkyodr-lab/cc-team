# cc-team

Claude Code上で動作するAI実務支援フレームワーク。  
秘書エージェントを窓口として、複数のエージェントとスキルを組み合わせて日常業務を処理する。

---

## 1. cc-teamとは

`/aiteam` コマンド一つで秘書エージェント（Secretary）が起動し、依頼内容を解析して適切なエージェントや操作手順に振り分ける。ユーザーは秘書に話しかけるだけでよく、内部の処理フローはフレームワークが担う。

AIworkspace/cc-team を独立したGitHubリポジトリとして管理し、チームや別環境でも再利用できる形を目指す。

---

## 2. 基本コンセプト

```
ユーザー
  ↓  /aiteam で依頼
Secretary（窓口・分類・交通整理）
  ↓  依頼に応じてルーティング
Agents（判断主体）          Operations/SKILL（再利用可能な作業手順）
  ↓  必要なSKILLを呼び出す          ↑
  └─────────────────────────────────┘
```

- **Secretary** は何でも受け取り、どこへ渡すかだけを決める
- **Agents** は「どう処理するか」を判断し、SKILLを組み合わせて実行する
- **SKILL** は独立した手順書として複数のエージェントから呼び出せる

---

## 3. フォルダ構成

```
cc-team/
├── plugin/
│   └── commands/
│       └── aiteam.md           # /aiteam スラッシュコマンドの定義
├── secretary/
│   ├── agent.md                # 秘書エージェント
│   └── routing.md              # ルーティングロジック
├── agents/
│   ├── abstract/
│   │   └── agent.md            # 学会抄録エージェント
│   └── slide/
│       └── agent.md            # スライド構成エージェント
├── operations/
│   ├── minutes/
│   │   └── SKILL.md            # 議事録作成
│   ├── mail/
│   │   └── SKILL.md            # メール処理
│   ├── abstract/
│   │   └── SKILL.md            # 学会抄録作成
│   └── slide/
│       └── SKILL.md            # スライド構成作成
├── projects/
│   └── _project_template/      # 案件テンプレート
│       ├── PROJECT.md
│       └── pm_agent.md
└── shared/
    ├── privacy_rules.md        # プライバシールール（全エージェント共通）
    └── style_guide.md          # 文体・出力ガイドライン
```

---

## 4. 各フォルダの役割

| フォルダ | 役割 |
|---|---|
| `plugin/commands/` | `/aiteam` コマンドのエントリーポイント。Claude Codeスラッシュコマンドとして動作する |
| `secretary/` | 依頼を受け取り、内容を分類してエージェントまたはSKILLへ振り分ける窓口 |
| `agents/` | 判断主体となるエージェント群。複雑な処理や複数ステップを伴う業務を担当する |
| `operations/` | 再利用可能な作業手順（SKILL.md）を格納する。単純・定型的な処理はここで完結させる |
| `projects/` | 案件単位でコンテキスト・成果物・進捗を管理する |
| `shared/` | プライバシールール・文体ガイドなど、全エージェントが参照する共通ルール |

---

## 5. 基本ワークフロー

1. Claude Codeのチャットで `/aiteam` を実行
2. Secretaryが依頼内容を読み取り、担当を判断する
3. 定型業務 → 対応するSKILLを直接呼び出して実行
4. 判断・調整が必要な業務 → 担当Agentを経由してSKILLを呼び出す
5. 結果をユーザーへ返却

**MVP対応済み機能：**

| 機能 | 担当SKILL |
|---|---|
| 議事録作成 | `operations/minutes/SKILL.md` |
| メール処理 | `operations/mail/SKILL.md` |
| 学会抄録作成 | `operations/abstract/SKILL.md` |
| スライド構成作成 | `operations/slide/SKILL.md` |

---

## 6. agent.md と SKILL.md の使い分け

| | agent.md | SKILL.md |
|---|---|---|
| **置き場所** | `agents/<name>/` | `operations/<name>/` |
| **役割** | 判断・調整・複数ステップの統括 | 単一の作業手順を定義する部品 |
| **呼び出し元** | Secretary または他のAgent | Secretary または Agent |
| **向いている処理** | 状況によって出力が変わる業務、複数SKILLの組み合わせ | 毎回ほぼ同じ手順で実行できる定型業務 |
| **例** | 抄録エージェント（要件ヒアリング→構成判断→出力） | 議事録SKILL（入力→フォーマット→出力） |

> **判断基準：** 「手順書に書き切れるか？」 → 書けるならSKILL。状況判断が必要ならAgent。

---

## 7. 今後の拡張予定

- Secretaryのルーティング精度の向上（曖昧な依頼への対応）
- 新規SKILL追加（報告書作成、論文要約、週報生成など）
- プロジェクト管理エージェント（pm_agent）の実装
- エージェント間連携（マルチエージェントフロー）の実装
- secretaryによる自動ログ・履歴管理
