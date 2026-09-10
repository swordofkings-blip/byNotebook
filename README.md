# 📖 byNotebook - NotebookLM 対話型ロールプレイ環境

> **Google NotebookLM × AIエージェントで創る、設定破綻ゼロの対話型ライトノベル生成環境**

AIロールプレイや対話小説でよくある**「途中で設定を忘れる」「キャラ崩壊」「不自然な知ったかぶり（全知化）」「オウム返しの多用」**を、Google NotebookLMの強力なRAG技術と専用プロンプトアーキテクチャによって解決した対話型ノベル環境です。

---

## 🌟 主な特徴

* 🧠 **設定忘れ・キャラ崩壊ゼロ**: NotebookLM（Gemini）の超高精度RAGにより、長編ストーリーでも設定や過去の約束を忠実に再現。
* 🎭 **知ったかぶり（全知化）の完全防止**:
  * **名前アンロック**: 初対面の人物は「銀髪の少女」等の見た目で描写され、名乗られて初めて名前が表示。
  * **認知動線の義務化**: 相手の居場所や秘密を最初から知っている体で動かず、「どこにいるの？（質問）」や「端末で検索（調査）」といった動線を必ず描写。
* 💬 **掛け合いの臨場感（インターリーブ描写）**: 複数行のセリフや長文を打ってもオウム返しせず、セリフとセリフの間にNPCの仕草やリアクションを挟み込んでテンポよく進行。
* ⚡ **特殊スラッシュコマンド**:
  * `/日記`（ヒロインの秘密の日記）
  * `/心理`（胸の内の葛藤・本音）
  * `/幕間`（主人公の見ていない裏の事件）
  * `/イラスト`（作中シーンのアニメ調画像プロンプト）
* 🚀 **ひな形テンプレート完備**: SF、学園ラブコメ、ファンタジーなど、好きな世界観のファイルを置くだけですぐに新しい物語を構築可能。

👉 **[実際の長文プレイログを見る (DEMO_PLAY_LOG.md)](./DEMO_PLAY_LOG.md)**

---

## 📁 ディレクトリ構造

```
byNotebook/
├── .agents/
│   └── mcp_config.json              # MCP連携設定 (notebooklm-mcp)
├── worlds/
│   ├── _template/                   # ★ 新規世界観作成用のひな形
│   │   ├── system_prompt.md         # チャット設定用プロンプト
│   │   ├── world_setting.md         # 世界観・舞台背景
│   │   ├── character_sheets.md      # 登場人物設定シート
│   │   └── memories_and_lore.md     # 相互認知マップ・情報格差ロア
│   └── helloworld/                  # 検証・実証済みサンプルワールド (SF・アンドロイド)
│       ├── config.json              # ノートブックID等のメタデータ
│       ├── system_prompt.md
│       ├── world_setting.md
│       ├── character_sheets.md
│       ├── memories_and_lore.md
│       └── prologue.md
├── user/
│   └── writing_guidelines.md        # ユーザー向け執筆・カスタマイズガイド
├── ONE_PAGER.md                     # プロジェクト概要ペライチ
├── DEMO_PLAY_LOG.md                 # 実際の対話実演ログ
└── README.md                        # 本ファイル
```

---

## 🚀 クイックスタート

### 1. 前提条件のインストール
NotebookLMをCLI/MCPから操作するツールをインストールします。

```bash
npm install -g notebooklm-mcp-cli
```

### 2. Google認証
ターミナルで以下を実行し、Googleアカウントでログインします。

```bash
nlm login
```

### 3. 新しい世界観を作る
1. `worlds/_template/` をコピーして `worlds/<あなたの世界名>/` を作成します。
2. キャラ設定や世界観を編集します。
3. AIコーディングアシスタント（Cursor / Windsurf / Antigravity等）に以下のように依頼するだけで、自動でNotebookLMにノートブックが作成・同期されます：

> 「`worlds/<あなたの世界名>/` の設定で新しいNotebookLMノートブックを作って同期して」

---

## 📚 詳しいドキュメント
* **[執筆・設定カスタマイズガイド (user/writing_guidelines.md)](./user/writing_guidelines.md)**: ON/OFFスイッチ、相互認知マップの書き方、長期記憶の同期運用テクニックなど
* **[エージェント運用手順書 (AGENTS.md)](./AGENTS.md)**: AIエージェントが自律的にノートを作成・同期するためのコマンド手順書

---

## 📜 ライセンス
MIT License
