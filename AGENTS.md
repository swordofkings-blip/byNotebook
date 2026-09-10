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
│   │   ├── character_sheets.md      # 登場人物設定シート
│   │   └── memories_and_lore.md     # 相互認知マップ・情報格差ロア
│   └── <world_name>/                # 各世界観ごとの管理フォルダ
│       ├── config.json              # ノートブックID等のメタデータ
│       ├── system_prompt.md         # チャットカスタム指示
│       ├── world_setting.md         # 世界観設定
│       ├── character_sheets.md      # キャラ設定
│       ├── memories_and_lore.md     # 相互認知・ロアシート
│       └── prologue.md              # プロローグ / エピソード
├── user/
│   └── writing_guidelines.md        # ユーザー向け執筆・カスタマイズガイド
├── ONE_PAGER.md                     # 友達向けペライチ紹介文
├── DEMO_PLAY_LOG.md                 # 実際の対話実演ログ
├── README.md                        # GitHub用メインREADME
└── AGENTS.md                        # 本ファイル（エージェント運用マニュアル）
```

---

## 2. 新規世界観ノートの作成・同期手順（標準ワークフロー）

ユーザーから「新しい世界を作って」「〇〇という物語ノートを作成して」と指示された場合、エージェントは以下の手順を実行してください。

### Step 1: ローカル設定ファイルの作成
1. `worlds/<world_name>/` ディレクトリを作成。
2. `worlds/_template/` を参考に、ユーザーの要望に合わせた以下のファイルを作成する：
   - `system_prompt.md`（以下の必須ルールを盛り込んだチャット指示）:
     - プレイヤーの入力形式（地の文、`＠キャラ名:` によるなりきり発言、未入力時の代行展開許可）
     - **純粋なライトノベル形式**: メタラベル（【情景描写】等）や箇条書き選択肢、引用番号（`[1]`等）の完全禁止
     - **対話の掛け合い（インターリーブ描写）**: 複数行入力時にオウム返しせず、セリフの合間に仕草・反応を挟んでテンポよく描写し、最後を手厚く展開
     - **未知の情報への言及制限（全知化の防止）**: 居場所や秘密を最初から知っている体で動かず、「尋ねる」「調べる」「委ねる」の動線を義務化
     - **名前アンロックシステム**: プレイヤー未認知のキャラは見た目の仮名（「銀髪の少女」等）で描写し、名乗られた瞬間に正式名称へアンロック
     - **誤字・脱字・タイポの自動補正**: ユーザーの誤変換や表記揺れを文脈から補完して正しい名称で出力
     - **特殊スラッシュコマンド**: `/日記`, `/登場`, `/心理`, `/幕間`, `/イラスト`, `/スキップ`
     - スローペース制御（通常は数秒〜数分、完結・移動・指示時のみスキップ＋着地時減速）
     - 衣装・装備の固定管理
   - `world_setting.md`（世界観、用語、ルール）
   - `character_sheets.md`（主人公・ヒロイン・NPCの性格・口調例）
   - `memories_and_lore.md`（相互認知マップ・情報格差リスト・初期衣装）
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
nlm source add <notebook_id> --file .\worlds\<world_name>\memories_and_lore.md --title "長期記憶・相互認知マップ：<タイトル>" --wait
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

## 3. 既存ノートブックの更新・ロア同期手順

物語が進んで新たな設定・エピソードが追加されたり、長期記憶（ロアシート）を更新する場合：

### ① プロンプト設定の更新
```powershell
$prompt = Get-Content -Raw .\worlds\<world_name>\system_prompt.md
nlm chat configure <notebook_id> --goal custom --prompt "$prompt" --response-length longer
```

### ② ロアシート（ソース）の入れ替え
古いソースを削除し、最新のロアシートをアップロードする。
```powershell
# 古いソースの削除（--confirm で確認スキップ）
nlm source delete <old_source_id> --confirm

# 新しいロアシートの追加
nlm source add <notebook_id> --file .\worlds\<world_name>\memories_and_lore.md --title "長期記憶・相互認知マップ：<タイトル>" --wait
```

---

## 4. トラブルシューティング

* **認証エラー（401 / Profile not found）が発生した場合**:
  - `nlm login --check` で認証状態を確認。
  - 認証が切れている場合は、ユーザーにターミナルで `nlm login` を実行してGoogleログインするよう依頼する。
* **ソース削除時の注意**:
  - `nlm source delete` コマンドには notebook_id ではなく **source_id** を直接指定すること（一括削除時は `--confirm` を付与）。
