# AGENTS.md: NotebookLM 対話型ストーリー管理ガイド

このリポジトリは、**Google NotebookLM（Gemini Notebook）を活用した対話型ロールプレイ／物語作成環境**の設定・ソース資料を管理・同期するためのワークスペースです。

エージェントはユーザーからの指示を受け、設定ファイルの作成からNotebookLMへの同期・ノートブック管理を自律的に行います。

---

## 1. ディレクトリ構造

```
byNotebook/
├── .agents/
│   └── mcp_config.json              # MCP連携設定 (notebooklm-mcp)
├── worlds/
│   ├── _template/                   # ★ 新規世界観作成用のひな形
│   │   ├── system_prompt.md         # チャット設定用プロンプト（進行ルール・出力形式）
│   │   ├── world_setting.md         # 世界観・舞台背景
│   │   └── character_sheets.md      # 登場人物設定シート
│   └── <world_name>/                # 各世界観ごとの管理フォルダ
│       ├── config.json              # ノートブックID等のメタデータ
│       ├── system_prompt.md         # チャットカスタム指示
│       ├── world_setting.md         # 世界観設定
│       ├── character_sheets.md      # キャラ設定
│       └── prologue.md              # プロローグ / エピソード
└── AGENTS.md                        # 本ファイル
```

---

## 2. 新規世界観ノートの作成・同期手順（標準ワークフロー）

ユーザーから「新しい世界を作って」「〇〇という物語ノートを作成して」と指示された場合、エージェントは以下の手順を実行してください。

### Step 1: ローカル設定ファイルの作成
1. `worlds/<world_name>/` ディレクトリを作成。
2. `worlds/_template/` を参考に、ユーザーの要望に合わせた以下のファイルを作成する：
   - `system_prompt.md`（以下の必須ルールを盛り込んだチャット指示）:
     - プレイヤーの入力形式（地の文、`＠キャラ名:` によるなりきり発言、未入力時の代行展開許可）
     - スローペース制御（通常は数秒〜数分、完結・移動・指示時のみスキップ＋着地時減速）
     - 間接描写（視線、沈黙、吐息、仕草、周囲の空気感の描写）
     - **キャラクターごとの個別知識・情報格差の厳守（他者の秘密や裏の出来事を知った気になって喋る全知化の防止）**
     - オウム返し・要約癖の防止
     - 過去の記憶・約束の想起（長期記憶ロアシートとの連動）
     - 【情景・間接描写】【キャラセリフ】【次の状況/問いかけ】の出力フォーマット
   - `world_setting.md`（世界観、用語、ルール）
   - `character_sheets.md`（主人公・ヒロイン・NPCの性格・口調例）
   - `memories_and_lore.md`（長期記憶・関係性ステータス・情報格差リスト）
   - `prologue.md`（初期シチュエーション）

### Step 2: NotebookLM にノートブックを作成
PowerShellで以下のコマンドを実行し、新規ノートブックを作成して `notebook_id` を取得する。

```powershell
nlm create notebook "物語を紡ぐ：<タイトル>"
```

### Step 3: `config.json` を保存
取得した `notebook_id` を `worlds/<world_name>/config.json` に保存する。

```json
{
  "notebook_id": "<取得したnotebook_id>",
  "title": "物語を紡ぐ：<タイトル>",
  "created_at": "YYYY-MM-DD",
  "description": "<概要>"
}
```

### Step 4: 設定資料（Markdown）をソースとしてアップロード
作成した設定ファイルをソースとして登録する（`--wait` を付与して処理完了を待つ）。

```powershell
nlm source add <notebook_id> --file .\worlds\<world_name>\world_setting.md --title "世界観設定：<タイトル>" --wait
nlm source add <notebook_id> --file .\worlds\<world_name>\character_sheets.md --title "登場人物設定：<キャラ名>" --wait
nlm source add <notebook_id> --file .\worlds\<world_name>\prologue.md --title "プロローグ：<シーン名>" --wait
```

### Step 5: チャット設定（システムプロンプト）の反映
`system_prompt.md` の内容をチャットのカスタムゴールとして登録する。

```powershell
$prompt = Get-Content -Raw .\worlds\<world_name>\system_prompt.md
nlm chat configure <notebook_id> --goal custom --prompt "$prompt" --response-length longer
```

### Step 6: 完了報告
作成完了後、ノートブックIDとブラウザ用URLをユーザーに案内する：
- URL: `https://notebooklm.google.com/notebook/<notebook_id>`

---

## 3. 既存ノートブックの更新・ログ追加手順

物語が進んで新たな設定・エピソードが追加された場合：

1. ローカルの `worlds/<world_name>/` 内のファイルを編集、または `story_log_chX.md` などの新ファイルを作成。
2. `nlm source add <notebook_id> --file <ファイルパス> --title "<タイトル>" --wait` を実行して反映。
3. 必要に応じてチャットプロンプト（`nlm chat configure`）を更新。

---

## 4. トラブルシューティング

* **認証エラー（401 / Profile not found）が発生した場合**:
  - `nlm login --check` で認証状態を確認。
  - 認証が切れている場合は、ユーザーにターミナルで `nlm login` を実行してGoogleログインするよう依頼する。
