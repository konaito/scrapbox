# [cline/cline](https://github.com/cline/cline)

このリポジトリ全体は **Visual Studio Code 拡張機能** として機能する「Cline」という AI アシスタント（LLM ベース）のプロジェクトです。VS Code 上でユーザーが高度な生成系 AI と対話し、以下のような「アシスタントにコードを書かせたり、ターミナルコマンドを実行させたり、Web ブラウザ統合など外部ツールを使わせたりする」ことを可能にする仕組みが一式そろっています。

---

## 全体像

1. **VS Code 拡張機能の本体**:  
   `src/` フォルダに拡張機能としてのロジックが集約されています。  
   - たとえば `extension.ts` は VS Code 拡張のエントリーポイントになっていて、コマンド登録・イベントフック・Webview のセットアップなどを行います。  
   - `core/`, `integrations/`, `services/` などのディレクトリで、AI メッセージ処理、ターミナル実行、ブラウザ操作、ファイル操作（検索・置換・読み書き）など「ツール実装」が管理されています。  
   - 「AI プロバイダごとの API 呼び出しロジック」も `src/api/providers/` 内にまとまっており、Anthropic / OpenAI / Bedrock / Vertex / Gemini / Mistral / DeepSeek など様々な LLM プロバイダと連携するようになっています。

2. **Webview UI（React）**:  
   `webview-ui/` フォルダには、React + TypeScript で構築されたフロントエンド（チャット画面、設定画面など）があります。拡張機能側でこのフロントエンドを VS Code Webview 上に注入し、ユーザーとやりとりを行います。  
   - `App.tsx` を中心とした通常の React SPA 的構造ですが、VS Code との通信には `vscode.postMessage()` / `onmessage` を使うかたちになっており、`utils/vscode.ts` で `acquireVsCodeApi()` を薄くラップしています。  
   - `components/` 以下でチャット画面や設定画面、MCP（Model Context Protocol）関連 UI などが分割管理され、ユーザーに AI との対話やツールの操作を行わせる仕組みを実現しています。

3. **設定ファイル / CI / GitHub Workflow**:  
   - `.github/workflows/test.yml` などにより、CI（lint やテスト）の自動実行が設定されています。  
   - `esbuild.js` や `tsconfig.json` など、TypeScript + esbuild を使ったビルド構成になっています。  
   - `webview-ui` は Create React App ベースのビルドを行い、ビルド結果を拡張機能に組み込む仕組みです。

4. **API プロバイダとのやりとり**:  
   `src/api/` 配下のファイルでは、Anthropic, OpenAI, AWS Bedrock, Google Vertex, あるいは Ollama, Mistral, LmStudio など様々なモデルとやりとりするためのクラスやユーティリティが定義されています。  
   - 各クラス（例: `AnthropicHandler`, `OpenAiHandler`, `VertexHandler` など）が共通のインターフェース `ApiHandler` を実装しており、ユーザーが VS Code 側の設定画面（Webview）でどのプロバイダを使うか選択すると、その設定に応じて動的にインスタンス化するような設計になっています。  
   - いずれもストリーミング応答（部分的なレスポンスを逐次受け取る）をサポートしており、VS Code のチャット画面にリアルタイムで出力を反映します。

5. **AI メッセージとツール呼び出し（Tool Use）**:  
   Cline はただのチャットボットではなく、LLM に「ファイルを編集する」「ターミナルコマンドを実行する」「ブラウザを操作する」などの命令を出させ、その処理結果を再度 LL

M に返す……といういわゆる「ツール呼び出し型」の AI アシスタントです。  
   - `src/core/assistant-message/` 付近のコードで AI が生成する特殊な XML ライクな構文（たとえば `<read_file><path>…</path></read_file>`）をパースし、実際のファイル操作 API を呼ぶ仕組みです。  
   - `integrations/` や `services/` には各機能が実装されており、AI が生成した命令をここが受けて実行してくれます。

6. **MCP (Model Context Protocol) 対応**:  
   `docs/mcp/` や `src/services/mcp/`, `src/shared/mcp.ts` などに MCP (Model Context Protocol) という、モデルが外部ツールを定義して呼び出すための仕組みが含まれています。Cline は MCP サーバーをローカルに構築して拡張することも想定しており、ブラウザ以外の多様なツール連携を柔軟に行えます。

7. **自動承認/保護機能・権限確認**:  
   AI が危険なコマンド（プロジェクトを消す、ネットワークに勝手にアクセスするなど）を実行しないように、ユーザーには権限の承認プロンプトが提示される仕組みがあります。  
   - `AutoApprovalSettings`（`src/shared/AutoApprovalSettings.ts`）を見ると分かるように、どの操作を自動承認にするか、手動承認にするかを設定できます。

8. **ユニットテスト / Lint / Prettier**:  
   `/src/**/*.test.ts` や `.eslintrc.json`、`.prettierrc.json` で構成され、CI 上で実行されるようになっています。  
   - テストは Mocha + Should (アサーションライブラリ) + vscode テストの仕組みで動きます。

---

## プロジェクトの「本質」

### 1. **VS Code 拡張としての AI アシスタントフレームワーク**
このプロジェクトは、ユーザーが VS Code 上で AI（LLM）に対しプロンプトを投げるだけではなく、

- **ファイルを読んで修正提案する**  
- **ユーザーの許可を得てコードを実際に上書きする**  
- **ターミナルを開き、ビルドやテスト、依存関係のインストールを実行する**  
- **ブラウザ（Puppeteer）連携でスクリーンショットを撮ったり、URL のテキストを解析したり**  
- **MCP（Model Context Protocol）による外部ツールやサーバー機能の拡張**  

といった「開発作業を支援するエージェント」として非常に包括的な構造になっています。

### 2. **拡張機能内での UI は React + TypeScript 製 Webview**
VS Code 拡張の多くは Webview を使って UI を組むことが多いですが、Cline は Create React App スタイルでしっかり SPA 化されており、`webview-ui/` 以下で通常の React アプリをビルドしてバンドルしたものを Webview に埋め込んでいます。  
- これにより、モダンなフロントエンドのやり方でチャット画面や設定画面、履歴管理などを実装しやすくしています。  
- Redux や `useContext` 的な状態管理も使えるので、UI ロジックを柔軟にカスタマイズしやすい構造です。

### 3. **LLM のストリーミング API / メッセージブロック変換**
Cline はチャットボットのように「ユーザーが発言 -> LLM からストリーミング応答 -> UI に流す」という流れを実現しつつ、さらに「応答中に `<tool_use>` ブロックが来れば、そのツールを同期的/非同期的に実行し、その結果をまた LLM に返す」といった「Chain-of-Thought」的な動的やり取りにも対応しています。  
- この部分は `src/api/transform/` や `src/core/assistant-message/parse-assistant-message.ts` あたりで「特定フォーマットの応答をパースして、VS Code 側の操作につなげる」仕組みが書かれています。

---

## まとめ

**「Cline」は、VS Code 上で利用できる AI アシスタント拡張機能のフルスタック実装** です。  
- **メインのロジック** … `src/` フォルダ：VS Code 拡張のエントリーポイント、LLM プロバイダ連携、ツール呼び出し実装  
- **フロントエンド** … `webview-ui/` フォルダ：React で書かれたチャット UI、設定画面など  
- **多彩な LLM/API に対応** … Anthropic, OpenAI, AWS Bedrock, Vertex AI, Gemini, Mistral, DeepSeek, Ollama, LM Studio 等  
- **ツール呼び出し** … ファイル編集、検索、ターミナル実行、ブラウザ操作、MCP 連携など  
- **開発用セットアップ** … TypeScript + esbuild + tsconfig、lint/format/test、GitHub Actions で CI/CD  
- **セキュリティや自動承認設定** … 危険な操作はユーザー承認を要求する仕組み

要するに本質的には「VS Code 拡張として多言語モデルを複合利用し、開発タスクを AI エージェントに実行させるためのフレームワーク」であり、フロントエンド（Webview）＋バックエンド（拡張機能本体）＋複数 API 連携を総合的に実装したものです。
