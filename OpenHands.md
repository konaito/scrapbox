# [All Hands AI/OpenHands](https://github.com/All-Hands-AI/OpenHands)

以下は、本リポジトリの要点（「本質」）を日本語でまとめたものです。主にどのフォルダが何のために存在し、どのようなコンポーネントが連携しているかを解説します。

---

## 全体像

- **Python（Poetry）+ Node.js（npm/React）** を組み合わせた大規模なAIエージェントプロジェクトです。  
- **バックエンド（`openhands/` ディレクトリ）** は Python 製で、エージェントの中核的なロジックを担当。  
- **フロントエンド（`frontend/` ディレクトリ）** は React/TypeScript 製で、ユーザーがブラウザからエージェントを操作したり、実行状況を可視化できるUIを提供。  
- **その他** Dockerコンテナ設定（`containers/`）や評価用ベンチマーク（`evaluation/`）など、エコシステム全体を整備する多くの補助フォルダが存在します。

---

## フォルダごとの概要

1. **ルート直下の設定ファイル**
   - **`pyproject.toml` / `poetry.lock`**: Pythonパッケージ管理ツール（Poetry）のプロジェクト設定。  
   - **`package.json` / `package-lock.json`**: Node.jsでの開発支援用（ESLintやTypeScriptなど）の依存関係管理。  
   - **`Makefile` / `build.sh` / `docker-compose.yml`**: ビルドやDocker関連のコマンドをまとめたスクリプト・ファイル。  
   - **`.gitignore`, `.dockerignore`**: GitやDockerで無視するファイルのパターン。  
   - **`.github/`**: GitHub ActionsなどCI/CD用のワークフロー設定、Issue/PRテンプレートなど。  
   - **`LICENSE`**: ライセンス文書。  
   - **`README.md`**: プロジェクト全体のメインREADME。

2. **`containers/`**
   - **`app/`, `dev/`, `runtime/`など**: DockerイメージをビルドするためのDockerfileやスクリプトを格納。  
   - **`e2b-sandbox/`**: [E2B](https://github.com/e2b-dev/e2b) というサンドボックスサービス向けのDocker設定。

3. **`dev_config/`**
   - Pythonの開発用ツール設定（`mypy.ini`, `ruff.toml`, `.pre-commit-config.yaml`など）。  
   - プロジェクト内で一貫したリントや型チェックを行うための設定がまとまっている。

4. **`docs/`**
   - [Docusaurus](https://docusaurus.io/) でドキュメントを生成するためのフォルダ。  
   - **`modules/`** などに技術文書やAPIリファレンスを配置。  
   - **`i18n/`**: 多言語翻訳（フランス語・中国語など）のJSONやMDファイル。  
   - **`plugins/`, `src/components/`**: Docusaurus向けのカスタムプラグインやReactコンポーネント。

5. **`evaluation/`**
   - エージェントのベンチマーク・評価用ハーネス群。  
   - **`benchmarks/`**: さまざまな評価タスク（コード修正テストや論理推論ベンチマークなど）の実装。  
   - **`integration_tests/`, `regression/`**: 統合テストや回帰テストをまとめたディレクトリ。  
   - 大規模かつ多角的にAIエージェントを評価する仕組みがある。

6. **`frontend/`**
   - React + TypeScript製のフロントエンド。  
   - **`src/`** 配下にコンポーネントやHook、Contextが整理されており、UIロジックが完結する。  
   - **`__tests__/`** や**`tests/`**では単体テストや統合テストを実行。  
   - ViteやESLint、PostCSSなどの設定ファイル（`vite.config.ts`, `.prettierrc.json` など）も含む。

7. **`microagents/`**
   - 「小規模エージェント（Micro-Agent）」を定義するMarkdownや設定ファイル。  
   - **`knowledge/`**: 「知識系」エージェント（何か特定のキーワードに反応するようなもの）。  
   - **`tasks/`**: 「タスク系」エージェント（PRコメントを処理する等、明確な入力/出力を持つもの）。
   - これらは`openhands/`内部でロードされる仕組み。

8. **`openhands/`**（バックエンドの中心）
   - **`agenthub/`**: 主なエージェント（`CodeActAgent`, `BrowsingAgent`, `DelegatorAgent` など）や「マイクロエージェント」のメインロジック。  
   - **`controller/`**: エージェントのメイン制御（アクションとオブザベーションのループ管理など）。  
   - **`core/`**: CLI (`cli.py`)や設定読み込み (`config/`)、ループ (`loop.py`)、ログ、メッセージ定義など基盤コード。  
   - **`events/`**: イベント（行動や観測結果）を定義・シリアライズする仕組み。  
   - **`llm/`**: 大規模言語モデル(Large Language Model)をコールするための抽象化クラス（OpenAIやGoogle等）。  
   - **`memory/`**: 対話履歴を保持したり要約（コンデンス）する仕組み。  
   - **`microagent/`**: `microagents/`フォルダと連動して、YAML等からマイクロエージェントを読み込む。  
   - **`resolver/`**: GitHub IssueやPull Requestを「解決」するためのロジック（パッチの自動生成など）。  
   - **`runtime/`**: Dockerコンテナやリモート環境でコマンドを実行する「実行サンドボックス」機能。  
   - **`security/`**: セキュリティチェックやアナライザ（不正操作を検出する仕組み）。  
   - **`server/`**: FastAPI + Socket.IO でリアルタイム通信を行うサーバー。  
   - **`storage/`**: ファイルやセッションデータを保存するインターフェース（ローカルやS3、GCSなど）。  

9. **`tests/`**
   - Python側ユニットテスト。  
   - `runtime/` や `resolver/` などサブフォルダに分かれている。  
   - **`tests/unit/`** などでピンポイントテストを行う。

---

## 開発の流れ・利用の仕方

1. **セットアップ**
   - Pythonサイド: `poetry install`  
   - フロントエンド: `cd frontend && npm install`

2. **起動・ビルド**
   - 個別に `poetry run` でバックエンドを起動したり、`npm run dev` (または `npm run build`) でフロントエンドを動かす。
   - 全体をDockerでまとめる場合: `docker-compose up` など。

3. **テスト**
   - Python: `poetry run pytest`  
   - Reactフロント: `npm test`  
   - ベンチマーク系: `evaluation/benchmarks/` 下のシェルスクリプトを実行。

4. **CI/CD**
   - `.github/workflows/` のGitHub Actionsがテストやビルド、イメージプッシュ、ドキュメント公開などを自動化。

---

### まとめ

- **`openhands/`** がAIエージェントのコア。  
- **`frontend/`** がReactベースのUI。  
- **`evaluation/`** で多面的なベンチマークや回帰テストを行い、エージェントの精度や正確性を検証。  
- **`microagents/`** で細分化された知識やタスクを拡張。  
- DockerやGitHub Actionsなどを使い、**AIエージェントのビルド・テスト・デプロイのフローが一貫管理**されている。

このように、大規模AIプロジェクトを統合的に扱うための仕組み（フロント/バックエンド、サンドボックス、ベンチマーク、セキュリティチェックなど）がバランスよく整理されているのが本リポジトリの「本質」といえます。
