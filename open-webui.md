# [open-webui](https://github.com/open-webui/open-webui)

以下のリポジトリは大きく分けて「**バックエンド（Python / FastAPI）**」「**フロントエンド（Svelte / TypeScript）**」「**コンテナ関連（Docker / docker-compose / Kubernetes）**」「**テスト関連（Cypress など）**」の4つに整理されており、加えて GitHub ワークフローや設定ファイルなどが複数含まれています。ざっくり全体像をまとめると、**LLM を扱う Web UI をオープンソースで開発・運用するためのフルスタック構成**です。以下のポイントが「本質情報（プロジェクトの中核）」となります。

---

## 1. バックエンド：`backend/`
- **Python + FastAPI** による API サーバおよびソケット通信（WebSocket 等）。
- `open_webui/` ディレクトリ配下に実装が集中しており、フォルダ構成は以下のように整理：
  - **`main.py`**：FastAPI アプリのエントリポイント。
  - **`routers/`**：API エンドポイントがテーマ別に分割（`auths.py`, `chats.py`, `files.py`, etc.）。
  - **`models/`**：SQLAlchemy 等を用いた DB モデル定義。
  - **`retrieval/`**：外部検索やベクタ DB との連携処理（`chroma`, `qdrant`, `pgvector` など）。
  - **`internal/db.py`** や **`migrations/`**：Alembic を用いた DB マイグレーション管理。
  - **`utils/`**：権限管理や OAuth、マルチメディア処理などの共通関数。
  - **`tasks.py`**：バックグラウンドジョブや非同期タスクの管理。
- **LLM やチャットの履歴、ファイルアップロード、ツール連携**など、「ユーザの会話やデータ管理に関する API 機能」が実装されている。

### データベースや永続化
- Alembic でのテーブルマイグレーションが多数存在（`migrations/versions/`）。
- `backend/data/` フォルダをマウントして DB・ファイルなどの永続化を行う構成を想定している。

---

## 2. フロントエンド：`src/`
- **SvelteKit + TypeScript + Tailwind CSS** で構築された SPA（Single Page Application）。
- `/src/lib/` 配下に各種コンポーネント群（チャット画面、管理画面、UI 部品、アイコンなど）が細かく分割。
- 国際化対応（`i18n/` ディレクトリに多数の言語 JSON）。
- `/routes/` 以下に画面単位のレイアウトやページが整理されており、`(app)/` フォルダがメインのアプリ画面に該当。

### 主なフロントエンド要素
- **チャット UI**（`src/lib/components/chat/`）  
  メッセージ入力、モデル切り替え、レスポンスのレンダリング、音声入力など。
- **管理画面**（`src/lib/components/admin/`）  
  ユーザ管理、モデル管理、評価、設定などバックエンド API を操作するための GUI。
- **知識ベース管理**（`src/lib/components/workspace/Knowledge/` など）  
  ドキュメントアップロードやベクタ検索連携などのフロント。

---

## 3. コンテナ・インフラ関連：`Dockerfile`, `docker-compose.*`, `kubernetes/`
- **`Dockerfile`**：フロントエンド・バックエンドをまとめてビルドし、WebUI を起動するためのイメージ定義。
- **`docker-compose.yaml`** および **`docker-compose.*.yaml`**：  
  - CPU/GPU 用、Ollama（ローカル LLM）の接続用、データ永続化用など複数の構成ファイルが用意されている。
  - `docker-compose.api.yaml` や `docker-compose.gpu.yaml` など用途別に上書きしやすい形。
- **`kubernetes/manifest/`**：K8s 環境へのデプロイマニフェスト（Namespace, Deployment, Service, PVC など）。
  - GPU 対応や Ollama の StatefulSet も含まれる。

### Ollama 連携
- `ollama` コンテナを立ち上げてローカル LLM を利用可能にするための設定が多数見受けられる (`OLLAMA_BASE_URL` や GPU/AMD 対応など)。

---

## 4. テストおよびスクリプト：`cypress/`, `scripts/`, `Makefile` etc.
- **`cypress/`**：エンドツーエンドテスト（E2E）を実行するための設定とテストコード。  
  フロント側の動作確認やドキュメントアップロード・チャット動作などを自動化している。
- **`scripts/prepare-pyodide.js`**：Pyodide（Python を WebAssembly 化）関連の準備スクリプトらしきもの。
- **`Makefile`, `run.sh`, `start.sh`, `run-compose.sh`** など：ローカルでの立ち上げやビルド、コンテナ実行を助けるスクリプト群。

---

## 5. その他のポイント
- **`.env.example`** に API キーや音声設定、Ollama の接続 URL などの環境変数テンプレート。
- **`.github/workflows/`** で **CI/CD**（テスト、ビルド、リリース）を自動化。  
  `build-release.yml`, `release-pypi.yml`, `docker-build.yaml` などがある。
- **多言語対応**：`src/i18n/locales/` に非常に多くの言語 JSON があるため、UI の国際化 (i18n) に力を入れている。
- **ライセンス** は `LICENSE` ファイル参照（MIT or Apache などを確認）。

---

## 要約
- **「LLM を用いたチャット/知識ベース/ツール連携を行うオープンソース Web UI」** がこのリポジトリの本質です。  
- バックエンドは FastAPI (Python) で AI や DB、外部検索との連携を担当し、  
- フロントエンドは SvelteKit (TypeScript) でチャット画面や管理画面を提供する構成。  
- Docker もしくは Kubernetes 上での **コンテナ実行** が前提となっており、Ollama などローカル LLM や GPU も含めた運用を想定。  
- Cypress 等でテストを自動化しつつ、GitHub Actions で CI/CD パイプラインを整えている。

つまり、このレポジトリをクローンすれば、Python + SvelteKit + Docker を組み合わせた **チャット UI・文書アップロード・権限管理など含むフルスタックの LLM WebUI** が構築できる、というのが「本質情報」です。
