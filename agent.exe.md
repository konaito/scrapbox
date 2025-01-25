#[agent.exe](https://github.com/corbt/agent.exe)

以下は、このリポジトリに含まれているディレクトリ・ファイル構成と、それぞれの役割や目的についての解説です。Electron + React + TypeScript（Electron React Boilerplate由来の構成）をベースにしているため、それに関連する典型的なファイルやスクリプトが含まれています。

---

## ルート直下のファイル

### 1. **.editorconfig**
エディタやIDEで共通のフォーマット設定を共有するための設定ファイルです。インデント幅・文字コード・行末コードなどの規則が書かれています。

### 2. **.env.example**
AnthropicのAPIキーのサンプルなど、環境変数を定義するための例示ファイルです。実際に使用する際は `.env` とリネームして中身を埋める運用が想定されています。

### 3. **.eslintignore**
ESLint による静的解析を行わないファイルやディレクトリを指定しています。`node_modules` やビルド成果物 (`.erb/dll/`, `release/app/dist` など) が無視対象に含まれます。

### 4. **.eslintrc.js**
ESLint（JavaScript/TypeScriptのLintツール）の設定ファイルです。ルールやパーサ、プラグインなどが定義されています。

### 5. **.gitattributes**
Gitの属性を指定するファイルです。特定の拡張子をバイナリとして扱うよう設定したり、改行コードの扱いなどを指定しています。

### 6. **.gitignore**
Gitで追跡させないファイル・フォルダを指定します。`node_modules` やビルド成果物、OS固有ファイル (`.DS_Store` など) が除外対象になっています。

### 7. **.vscode/**
VSCodeで開発する際の設定ファイルを配置しています。  
- **extensions.json**: 推奨拡張機能のリスト  
- **launch.json**: デバッグ時の設定（Electron: Main / Rendererなど）  
- **settings.json**: ワークスペースレベルのVSCode設定

### 8. **CODE_OF_CONDUCT.md**
プロジェクトへのコントリビューター向けの行動規範(Code of Conduct)が記載されています。コミュニティ活動時の基本的なルールが示されます。

### 9. **LICENSE**
MITライセンス等、ライセンスの文面が書かれています。本プロジェクトはElectron React Boilerplate由来のMITライセンスですが、`package.json` によるとApache-2.0としても表記があるため、どちらも確認が必要なようです。

### 10. **README.md**
このリポジトリのトップページに表示される説明文書です。セットアップ方法や動機、使い方、既知の問題などの概要が書かれています。

### 11. **package-lock.json / package.json**
- `package.json` は、プロジェクトの依存関係やスクリプト、Electronビルダーの設定などが一括で管理されるファイル。  
- `package-lock.json` は、依存パッケージの正確なバージョンなどをロックするためのファイルです。

### 12. **tsconfig.json**
TypeScriptのコンパイラオプションを定義しています。  
`"outDir": ".erb/dll"` など、Electron React Boilerplateの構成特有の出力先指定があります。

### 13. **assets/**  
アプリのアイコン類やmac用のentitlementsファイル(署名に必要)などが入っています。`icon.png`/`.icns` や さまざまな解像度のアイコン画像、`entitlements.mac.plist` 等が含まれます。

### 14. **release/**
Electronでパッケージングしたものを配置するためのディレクトリです。  
- **app/**: アプリの実行ファイルが配置される場所。  
  - `package.json` (軽量版) など、ビルド後に必要なものが置かれる。

---

## `.erb` ディレクトリ

Electron React Boilerplate（ERB）由来の設定やスクリプトがまとまっているフォルダです。

### **.erb/configs/**

- **webpack.config.\*.ts**: Webpackの設定ファイル群です。  
  - `webpack.config.base.ts`: 他の特定ビルド向け設定の共通ベースとなるファイル  
  - `webpack.config.main.*.ts`: メインプロセスのビルド設定  
  - `webpack.config.preload.dev.ts`: プリロードスクリプトの開発向け設定  
  - `webpack.config.renderer.*.ts`: レンダラープロセスのビルド設定  
  - `webpack.config.eslint.ts`: ESLint用Webpack設定  
  - `webpack.paths.ts`: パス管理用の共通ファイル (ソースパス、出力先ディレクトリなどを一括管理)

- **.eslintrc**  
  追加的なESLint設定です。`.erb/configs` 配下でのみ適用されるよう指定している場合があります。

### **.erb/img/**  
Electron React Boilerplateのロゴやバナー画像が入ったフォルダ。

### **.erb/mocks/**  
テスト用にモックファイル (`fileMock.js`) を置くためのフォルダ。

### **.erb/scripts/**  
ビルドや開発フローに関わるスクリプトが多数置かれています。  
- **check-build-exists.ts**: メイン/レンダラープロセスのビルドが済んでいるか検証  
- **check-native-dep.js**: ネイティブ依存パッケージが正しく`release/app`側にのみ入っているかチェック  
- **check-node-env.js**: Webpack等の設定において、`NODE_ENV` が想定通りか検証  
- **check-port-in-use.js**: devサーバが使用するポートが既に使われていないかチェック  
- **clean.js**: ビルド結果を削除するためのスクリプト  
- **delete-source-maps.js**: ソースマップを削除するスクリプト  
- **electron-rebuild.js**: ネイティブ依存がある際に `electron-rebuild` を実行するスクリプト  
- **link-modules.ts**: `release/app` の `node_modules` をソース側からシンボリックリンクするためのスクリプト（ERBの典型）

---

## `.github/` ディレクトリ

GitHub ActionsやIssueテンプレート、設定ファイル等が置かれるディレクトリ。

- **workflows/**  
  - **codeql-analysis.yml**: GitHub CodeQLを使用したセキュリティスキャンの設定  
  - **publish.yml**: GitHubリポジトリへのリリース自動アップロード設定 (Electron Builderによりビルド&リリースするなど)  
  - **test.yml**: CI上でテスト・Lintなどを行うワークフロー

- **config.yml** / **stale.yml**  
  - `config.yml`: プルリクやIssueの必須ヘッダー指定などの設定例  
  - `stale.yml`: 長期間放置のIssueを自動でステータス変更(ステール化)したり、クローズする設定

---

## `src` ディレクトリ

アプリのメインロジックが格納される。ERB標準では `src/main` (メインプロセス) と `src/renderer` (レンダラープロセス) に分かれる構成。

### 1. **src/__tests__/\***  
Jestなどでテストを行うためのテストファイルがまとめられるディレクトリ。

### 2. **src/main/**

Electronのメインプロセスで動くコード。

- **main.ts**  
  エントリーポイント(メインプロセス側)です。ウィンドウの生成やIPC設定、AutoUpdaterの初期化等を行っています。

- **menu.ts**  
  Electronアプリのメニューバーを生成するためのコードがまとめられています。macOS / Windows でのメニュー差異もここで定義。

- **preload.ts**  
  プリロードスクリプト。`contextBridge.exposeInMainWorld` を使い、レンダラープロセスから扱うAPIを安全に注入する仕組みです。

- **store/\***  
  AIエージェントとのやりとりに使うZustand (Vanilla版) のストア定義や、Anthropic APIへのアクセスロジック、Nut.jsによるマウス/キーボード操作などが書かれています。  
  - `create.ts`: Zustandのストア生成およびアクションを定義  
  - `runAgent.ts`: Claude（Anthropic API）との会話をループしながら、指示内容を解析しNut.jsで操作を実行するフロー  
  - `extractAction.ts`: Claudeのメッセージから「どのツール（何の操作）を呼んだか」を抽出する処理  
  - `types.ts`: ストアで使用する型定義 (`AppState`, `NextAction`など)

- **util.ts**  
  一部の共通処理(HTMLパス生成など)が書かれている。

- **window.ts**  
  `BrowserWindow` の生成やフェードイン・フェードアウトなどのウィンドウ管理を行うユーティリティです。スクリーンサイズの取得、アニメーション表示ロジックなどをまとめています。

### 3. **src/renderer/**

Reactコンポーネントやフロントエンド（レンダラープロセス）側のソース。

- **App.tsx**  
  メインとなるReactコンポーネント。Chakra UIを使用したUIや、ユーザーの入力テキスト（命令文）、AI操作状態の表示などを行っています。

- **RunHistory.tsx**  
  Claude側からどんなアクションが行われたかを表示する履歴コンポーネントです。

- **hooks/useStore.ts**  
  `zutron`(Zustand + IPCを同期するためのライブラリ) を使ってメインプロセスとストアを同期し、Reactで利用できるようにするHookが定義されています。

- **index.ejs / index.tsx**  
  HTMLのテンプレート (EJS) と、そのルートにReactをrenderするエントリーポイント。  
  Electron + React での画面描画の基本。

- **global.d.ts / preload.d.ts**  
  TypeScriptで、`window.electron` や `window.zutron` などの型定義を追加するための宣言ファイル。

---

## 補足

- **npm scripts (package.json内)**  
  `npm run start` や `npm run build` など、Electronアプリ開発でよく使うタスクが多数定義されています。  
  - `start` は開発用。 Webpackでレンダラーを起動し、Electronメインを立ち上げる。  
  - `build` は本番ビルド。 `webpack.config.main.prod.ts` / `renderer.prod.ts` 等で最適化。  
  - `start:main`, `start:renderer`, `start:preload` などに分割されており、同時起動には `concurrently` が利用される。  
  - `build:dll` は Electron React Boilerplate特有の DLL ビルド。 依存パッケージバンドルを分割してビルド時間を削減する手法。
  - `postinstall` や `clean.js` などもERB標準の流れ。

- **Anthropic + Nut.js + Electron** という特徴的な組み合わせで、Electronウィンドウを最前面に出したり隠したりしつつ、Nut.jsでマウス・キーボード操作を自動化している部分がユニークです。

---

以上が、ディレクトリ/ファイル構成の概要と役割の説明です。  
Electron React Boilerplate(ERB)ベースの典型構造を踏襲しながら、AnthropicのAPIとNut.jsによる自動操作ロジックが追加されているのが大きな特徴になっています。
