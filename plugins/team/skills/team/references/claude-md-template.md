# CLAUDE.md 生成テンプレート

組織構築時に `.team/CLAUDE.md` を生成するためのテンプレート。
`{{...}}` の変数はオンボーディングデータで置換する。

---

## テンプレート

````markdown
# cc-team - AI実務支援システム

## チームプロフィール

- **チームの種類・活動**: {{TEAM_TYPE}}
- **目標・課題**: {{GOALS_AND_CHALLENGES}}
- **作成日**: {{CREATED_DATE}}

## 組織構成

```
.team/
├── CLAUDE.md
├── secretary/              ← 常設
│   ├── CLAUDE.md
│   ├── inbox/
│   ├── todos/
│   ├── notes/
│   └── logs/
├── operations/             ← 日常業務（選択時のみ）
│   ├── CLAUDE.md
│   ├── minutes/
│   │   └── CLAUDE.md
│   └── mail/
│       └── CLAUDE.md
├── agents/                 ← 学会発表（選択時のみ）
│   ├── CLAUDE.md
│   ├── abstract/
│   │   └── CLAUDE.md
│   └── slide/
│       └── CLAUDE.md
└── projects/               ← 本格的な研究（選択時のみ）
    └── CLAUDE.md
```

{{ADDITIONAL_OPERATIONS}}

## 業務一覧

| 業務 | フォルダ | 役割 |
|------|---------|------|
| 秘書室 | secretary | 窓口・相談役。TODO管理、壁打ち、メモ。常設。 |
| 議事録 | operations/minutes | 会議メモ・Plaudテキストから議事録を生成。 |
| メール | operations/mail | メール下書き・返信案作成。 |
| 学会抄録 | agents/abstract | 学会抄録の作成・文字数調整。医学フォーマット対応。 |
| スライド | agents/slide | 講演・学会発表用スライド構成・原稿作成。 |
| 研究プロジェクト | projects | 論文要約・研究案件。案件ごとに専用フォルダ。 |
{{OPERATION_TABLE_ROWS}}

## 運営ルール

### 秘書が窓口
- ユーザーとの対話は常に秘書が担当する
- 秘書は丁寧だが親しみやすい口調で話す
- 壁打ち、相談、雑談、何でも受け付ける
- 専門業務が必要な場合、秘書が直接該当業務フォルダに書き込む

### 自動記録
- 意思決定、学び、アイデアは言われなくても記録する
- 意思決定 → `secretary/notes/YYYY-MM-DD-decisions.md`
- 学び → `secretary/notes/YYYY-MM-DD-learnings.md`
- アイデア → `secretary/inbox/YYYY-MM-DD.md`

### 同日1ファイル
- 同じ日付のファイルがすでに存在する場合は追記する。新規作成しない

### 日付チェック
- ファイル操作の前に必ず今日の日付を確認する

### ファイル命名規則
- **日次ファイル**: `YYYY-MM-DD.md`
- **トピックファイル**: `kebab-case-title.md`

### TODO形式
```markdown
- [ ] タスク内容 | 優先度: 高/通常/低 | 期限: YYYY-MM-DD
- [x] 完了タスク | 完了: YYYY-MM-DD
```

### コンテンツルール
1. 迷ったら `secretary/inbox/` に入れる
2. 既存ファイルは上書きしない（追記のみ）
3. 追記時はタイムスタンプを付ける

## パーソナライズメモ

{{PERSONALIZATION_NOTES}}
````

---

## 変数リファレンス

| 変数 | ソース | 説明 |
|------|--------|------|
| `{{TEAM_TYPE}}` | Q1 | チームの種類・活動 |
| `{{GOALS_AND_CHALLENGES}}` | Q2 | 目標・困りごと |
| `{{CREATED_DATE}}` | 自動 | 組織構築日 |
| `{{ADDITIONAL_OPERATIONS}}` | 業務追加時 | 追加業務のディレクトリツリー（初期は空） |
| `{{OPERATION_TABLE_ROWS}}` | 業務追加時 | 追加業務のテーブル行（初期は空） |
| `{{PERSONALIZATION_NOTES}}` | Q1+Q2 | チームの状況に応じたメモ |

---

## 業務説明スニペット

業務追加時に `{{OPERATION_TABLE_ROWS}}` に追記するデータ:

| 業務 | フォルダ | 説明 |
|------|---------|------|
| 議事録 | minutes | 会議メモ・文字起こしから議事録を生成。決定事項・ToDoを抽出。 |
| メール | mail | メール下書き・返信案作成。依頼・お礼・日程調整に対応。 |
| 学会抄録 | abstract | 背景・目的・方法・結果・結論の整理。日英抄録・文字数調整。 |
| スライド | slide | 講演・発表スライドの構成作成。原稿・メッセージ生成。 |
| PM | pm | プロジェクト進捗・マイルストーン・スケジュール管理。 |
| リサーチ | research | 文献調査・情報収集・トピック整理。 |
