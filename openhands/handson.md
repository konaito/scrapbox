# OpenHands ハンズオン資料

## 1. プロジェクト概要

**OpenHands** は、大規模言語モデル（LLM）を用いて以下のようなことを行うフレームワーク／ツール群です:

- Python バックエンド（FastAPI + Socket.IO）と React+TypeScript フロントエンドのモノリポジトリ構成
- **LLM を活用したエージェント**（CodeActAgent、BrowsingAgent、MicroAgent など）を実行し、対話形式でソースコードを編集したり、ツール呼び出しを行ったりできる
- Docker や E2B sandbox、Remote（SSH）など **複数の実行環境** に対応
- **eval/benchmarks** ディレクトリで、いろいろなベンチマークを通してエージェントを評価可能
- microagents ディレクトリに定義される「マイクロエージェント」を拡張することで、タスクごとに特化したコマンドやプロンプトを追加できる

本ハンズオンでは、**ローカル環境で Python バックエンドとフロントエンドを一通り動かしてみる** シナリオを説明します。最終的に「どの LLM モデルを使うのがよいか」を簡単にまとめています。

---

## 2. 事前準備

1. **Git, Docker（推奨）, Python3.10+**  
   - Python3.12 でも OK ですが、仮想環境上で動かす場合は pyenv 等でバージョンを合わせてください。
2. **Node.js v22**  
   - `.nvmrc` によって Node.js のバージョンが 22 と指定されています。  
   - `nvm` を使って指定バージョンをインストール＆使用するか、他の方法で Node.js 22 系を用意します。

---

## 3. リポジトリのクローンとインストール

```bash
# GitHub からクローン
git clone https://github.com/All-Hands-AI/OpenHands.git
cd OpenHands

# Python パッケージのセットアップ (Poetry で)
# pyproject.toml / poetry.lock に従って依存を取得
poetry install  --with dev --with test

# フロントエンドのセットアップ (npm or yarn)
cd frontend
npm install
cd ..

# Docker Compose でバックエンド・DB など立ち上げる場合 (任意)
# docker-compose up -d
```

- Poetry が入っていない方は [公式ドキュメント](https://python-poetry.org/docs/) を参照してインストールします。
- `poetry install` に `--with dev --with test` を付けると開発・テスト系の依存もすべて導入されます。
- フロントエンドは `vite` による開発サーバを起動可能 (`npm run dev`) です。

---

## 4. ローカル開発モードで起動してみる

### 4.1 バックエンド（Python）の起動

OpenHands は **FastAPI + Socket.IO** の構成です。
ルートにある `openhands/server/listen.py` などがエントリポイントとなります。

最もシンプルには、以下で起動可能です:

```bash
# 仮想環境へ入る
poetry shell
# または source $(poetry env info --path)/bin/activate

# FastAPI+SocketIO サーバーを起動
python -m openhands.server.listen

# (ポート3000等で待受する設定になっている場合は別途 --port=xxx が必要な場合も)
```

あるいは `make start` などの Make ターゲットが存在すればそちらを利用してください（プロジェクトの Makefile をご確認ください）。

### 4.2 フロントエンド（React+TypeScript）の起動

```bash
cd frontend
npm run dev
```

デフォルトでは `http://localhost:5173` などで立ち上がるはずです。  
ブラウザでアクセスし、OpenHands の GUI が表示されれば OK です。

- バックエンドが `http://localhost:3000` で立ち上がっている想定の場合、フロントエンドの `.env` or `vite.config.ts` などで `VITE_BACKEND_BASE_URL` が `localhost:3000` に設定されているか確認してください。
- Docker Compose で一括起動するやり方もあります。`docker-compose.yml` や `containers/dev/compose.yml` などを参照すると、フロントエンドとバックエンドの両方がコンテナで走る設定があるかもしれません。

---

## 5. CodeActAgent を CLI 上で試す（最小ハンズオン）

GUI ではなく、CLI（ターミナル）上でもエージェントを動かせます。以下は最も単純な例です：

```bash
poetry run python openhands/core/main.py \
    -t "Hello World を出力するシンプルスクリプトを作って" \
    -c CodeActAgent \
    -m openai:gpt-3.5-turbo
```

- `-t` はタスク（ユーザーからの命令）。
- `-c` は利用するエージェントのクラス（この例だと `CodeActAgent`）。
- `-m` は LLM モデル指定（後述）。

実行すると、LLM との対話に基づいてエージェントがファイル編集コマンド（write）を実行したり、bash コマンド（run）を実行する形で進行するログがターミナルに表示されます。

もし「セキュリティ確認」が有効な場合（`confirmation_mode = true` など config が設定されている時）は、何かコマンドを実行するたびに「実行していい？」と尋ねる観測(Observation)が起こるかもしれません。その場合は `yes` / `no` などで答える運用です。

---

## 6. LLM モデルは何を使えばいいか？

OpenHands は [Litellm](https://github.com/litellm/litellm) を使用しているため、**モデル指定** は基本的に以下のような書式が使えます（例）:

- `openai:gpt-4`  
  → OpenAI の GPT-4 API を使う  
- `openai:gpt-3.5-turbo`  
  → OpenAI の GPT-3.5 Turbo  
- `google:chat-bison`  
  → Google PaLM API  
- `anthropic:claude-instant-1`  
  → Anthropic Claude Instant  
- `local:something`  
  → ローカルモデル (openHands のローカル LLM 実装)
- `litellm_proxy/anthropic.claude-3.5-sonnet`  
  → LiteLLM Proxy 経由のモデル呼び出し

### (1) 一般的な推奨

- **OpenAI GPT-4** または **GPT-3.5**  
  手軽で、エージェントが複雑なタスクをこなせる確率が高いです。  
  → ただし OpenAI API キーが必要です。[ここ](https://platform.openai.com) から取得

- **Anthropic Claude**  
  オープンベータの APIキーを取得できるならこちらも選択肢。  
  特に多ステップの対話タスクに強い場合があります。

### (2) 非 OpenAI オプション

- **Google PaLM**: `google:chat-bison` など  
- **Local LLM**: 例えば GPT4All, llama.cpp を使ったモデルをローカルで実行する仕組みも一部サポートがありますが、初期セットアップはやや複雑です。

### (3) Litellm Proxy（複数モデルまとめ）

- もし [litellm proxy](https://docs.litellm.ai/docs/proxy/quick_start) を自前で立てるなら、`-m litellm_proxy/<MODEL_NAME>` の形で指定すると、OpenAI, Anthropic, Azure など色々なプロバイダを一元的に呼び出せます。

### (4) `.env` / GUI 設定

フロントエンド（GUI）経由で使うときは、**Settings** ページで

1. `LLM Provider`  
2. `Model`  
3. `API Key`  

を入力し、`Advanced Options` で `Custom Model` を指定することもできます。  
（例: openai/gpt-4, openrouter/anthropic/claude-2 など）

---

## 7. より実践的な流れ

1. **プロジェクトをクローンし、Poetry と npm のセットアップ**  
2. **フロントエンド + バックエンドを起動**  
3. **GUI にアクセス** → 新規プロジェクトを作成 → LLM 設定で API キー入力 (OpenAI など)
4. **実際に対話** → 「ソースコードをアップロードして、修正案を提案してもらう」など
5. **GitHub リポジトリを接続** → Pull Request の更新や Issue に対応したりが可能 (Settings で GitHub 連携要)

---

## 8. まとめ

- OpenHands は Python + TypeScript の大規模なモノリポジトリで、LLM エージェントを活用した **自動コード操作** や **ツール呼び出し** をサポートするフレームワークです。
- ハンズオンとしては **Poetry で依存を入れ、ローカルサーバを起動 → フロントエンド（Vite）でアクセス** するとスムーズに試せます。
- **LLM モデル** は OpenAI の GPT-3.5 / GPT-4 が最も導入が簡単で汎用的に高性能です。  
  また、Anthropic / Google PaLM / OpenRouter / Local LLM などにも対応可能です。
- さらに実践的に使うには「Docker で統合起動」や「CI/CD でエージェント実行」「GitHub Issue/PullRequest Resolver 機能」といった発展的な機能もあります。

ぜひ試してみてください。わからない点があれば、`docs/` ディレクトリ以下の Docusaurus ドキュメントや、`CONTRIBUTING.md`, `Development.md` もご参照ください。
