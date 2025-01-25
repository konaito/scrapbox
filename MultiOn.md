#[MultiOn](https://github.com/MULTI-ON/cookbook)

以下のリポジトリは、**[MultiOn](https://www.multion.ai)** というウェブ操作用AIエージェントの活用例やサンプルコードをまとめた「Cookbook（レシピ集）」です。フォルダごとに用途が異なり、Notebook形式のサンプルからPythonプロジェクトまで、さまざまなユースケースが含まれています。以下で各ディレクトリと主要ファイルの概要を解説します。

---

## ルート構成

```
├── .github
│   └── ISSUE_TEMPLATE
│       └── bug_report.md
├── .gitignore
├── LICENSE
├── README.md
├── accommodation
│   └── accommodation.ipynb
├── coding-agent
│   ├── README.md
│   ├── agent.py
│   ├── app.py
│   ├── prompts.py
│   └── pyproject.toml
├── competitor-research
│   ├── README.md
│   ├── agent.py
│   ├── main.py
│   ├── prompts.py
│   └── pyproject.toml
├── internet-of-agents
│   ├── README.md
│   ├── manager.py
│   ├── pyproject.toml
│   ├── social_manager.py
│   ├── type.py
│   ├── viz.py
│   └── worker.py
├── job-search
│   ├── README.md
│   ├── agent.py
│   ├── main.py
│   ├── prompts.py
│   └── pyproject.toml
├── news-digest
│   └── news_digest.ipynb
├── personalized-travel-agent
│   └── mem0_travel_agent.ipynb
├── recursive-scraping
│   └── recursive_scraping.ipynb
├── restaurant-booking
│   └── restaurant_booking.ipynb
└── scraping
    └── scrape_linkedin.ipynb
```

---

## 共通・補助ファイル

- **`.github/ISSUE_TEMPLATE/bug_report.md`**  
  GitHubでバグ報告をするときに使われるテンプレートMarkdownファイル。OSやブラウザ、発生した問題の詳細などの記入欄がある。

- **`.gitignore`**  
  Pythonの仮想環境や一時ファイルなど、Gitで追跡しないファイルやフォルダを定義している。

- **`LICENSE`**  
  MIT License のライセンス文書。誰でもソフトウェアの利用・改変・再配布ができるライセンス形態。

- **`README.md`**  
  リポジトリ全体の概要説明。MultiOnの紹介や、このCookbookに含まれる各レシピへのリンクなどがまとめられている。

---

## Notebookフォルダとサンプル

### accommodation
- **`accommodation.ipynb`**  
  Airbnbでの宿泊施設検索や予約に関するサンプルNotebook。  
  - **MultiOn** の `step` と `retrieve` を使い、ブラウザ操作や情報取得を自動化している。

### news-digest
- **`news_digest.ipynb`**  
  ニュース記事の収集と簡単な分析を行うNotebook。ニュースを検索し、記事をスクレイピングして要約する例を示している。

### personalized-travel-agent
- **`mem0_travel_agent.ipynb`**  
  [Mem0](https://mem0.ai) という外部サービスと連携し、ユーザーの旅行嗜好データを活用する「パーソナライズ旅行エージェント」のサンプルNotebook。  
  - ユーザーの過去メモを参照しながら旅行提案を行う例を示す。

### recursive-scraping
- **`recursive_scraping.ipynb`**  
  Hacker Newsのようなサイトを再帰的にスクレイピング（親ページだけでなく、リンク先コメントなど深層ページにも移動して情報取得）するサンプルNotebook。  
  - ThreadPoolExecutorと組み合わせて並列処理を行う例がある。

### restaurant-booking
- **`restaurant_booking.ipynb`**  
  OpenTableでレストラン予約を自動化する例Notebook。日程や人数、場所を指定して予約ページでの操作をエージェントに任せるフローを紹介。

### scraping
- **`scrape_linkedin.ipynb`**  
  LinkedInのプロフィール情報をスクレイピングするサンプルNotebook。ログイン後に検索結果を自動でたどり、プロフィール詳細を取得する実装例。

---

## Pythonプロジェクト別フォルダ

### coding-agent
エージェントを用いてコーディングタスクを自動化するプロジェクト。

- **`README.md`**  
  Poetryでのセットアップ手順や、Gradioを利用した実行方法の説明。
- **`agent.py`**  
  「プログラマー／リサーチャー／ノートテイカー」3つのマルチエージェントを統括し、ユーザーのコーディング要求に応じてブラウザ操作やメモ取りなどを行うクラス `DevOn` を定義。
- **`app.py`**  
  Gradioインターフェースを使ったWebアプリケーションのエントリポイント。ユーザーがチャット形式で命令を与えると、`DevOn` クラスが処理を進める。
- **`prompts.py`**  
  システムプロンプトやエージェント用の定型メッセージ（プログラマー用、リサーチャー用など）がまとめられている。
- **`pyproject.toml`**  
  Poetryの設定ファイル。依存パッケージやメタデータが書かれている。

### competitor-research
競合リサーチを行うエージェントの例。

- **`README.md`**  
  プロジェクト概要と実行手順。PoetryやOpenAI・MultiOnのAPIキー設定など。
- **`agent.py`**  
  `Analyst` クラスを定義。マルチエージェントを用いて競合企業を探し、解析結果を「State」にまとめるフローを実装。
- **`main.py`**  
  プログラム実行のエントリポイント。`analyst.run(...)` を呼び出し競合調査を実行する。
- **`prompts.py`**  
  システムメッセージ、ツール呼び出しのJSONスキーマなどを定義。
- **`pyproject.toml`**  
  Poetryの設定ファイル。

### internet-of-agents
インターネット上で多数のAIエージェントを動かす「Internet of Agents」的な並列処理の事例。

- **`README.md`**  
  使い方や流れの説明。Chrome拡張を使う設定、.envファイルへのAPIキー記入など。
- **`manager.py`**  
  いくつかのタスクをまとめて並列実行する「ManagerAgent」を定義。`prefect` を使ってフローを管理している。
- **`social_manager.py`**  
  LinkedInやX(Twitter), Facebookに投稿して宣伝をするマネージャーの例を実行するスクリプト。
- **`type.py`**  
  タスクのデータ型定義（Pydanticのモデル）。`Task`, `TaskList`など。
- **`viz.py`**  
  `graphviz` を使ってタスク一覧を可視化するサンプルコード。
- **`worker.py`**  
  個別のウェブ操作（`perform_task`）を行うワーカー用クラス。`MultiOn` のセッション開始→ステップ実行→終了処理を担当。
- **`pyproject.toml`**  
  Poetryの設定ファイル。

### job-search
求人検索と応募を行うエージェントの事例。

- **`README.md`**  
  概要とセットアップ手順。
- **`agent.py`**  
  `Agent` クラスが定義され、`browse` や `retrieve` などのツールを使ってジョブ情報を収集し、Stateに進捗を貯める仕組みがある。
- **`main.py`**  
  実行スクリプト。`agent.run(...)` を呼んで求人検索と応募を自動化する。
- **`prompts.py`**  
  こちらでもツール呼び出し用のJSONスキーマやシステムプロンプトを定義。
- **`pyproject.toml`**  
  Poetryの設定ファイル。

---

## 各Notebookの概要

1. **accommodation/accommodation.ipynb**  
   Airbnbで宿泊先を検索して予約する流れのサンプル。
2. **news-digest/news_digest.ipynb**  
   ニュースサイトで記事を取得・要約するサンプル。
3. **personalized-travel-agent/mem0_travel_agent.ipynb**  
   ユーザーの履歴情報をMem0から取得して、個人に合わせた旅行プランを提案するデモ。
4. **recursive-scraping/recursive_scraping.ipynb**  
   Hacker Newsなどの掲示板のリンクを再帰的にたどってスクレイピングを行う例。
5. **restaurant-booking/restaurant_booking.ipynb**  
   OpenTableを使い、レストランの検索と予約を自動化する例。
6. **scraping/scrape_linkedin.ipynb**  
   LinkedInにログインし、プロフィール情報を検索・取得するサンプル。

---

## まとめ
- **MultiOn** は「自然言語でのウェブ操作やスクレイピング」が可能なプラットフォームで、本リポジトリには具体的な活用例が数多く収録されています。
- **Notebook形式のサンプル** では、特定のウェブサイトやユースケースに応じたステップ実行や構造化データの抽出（`retrieve`）を体験できます。
- **各Pythonプロジェクト** では、より高度なエージェントの連携例（プログラミング支援、競合調査、求人応募、など）や複数タスクの並列実行が含まれています。
- **拡張性・再利用性** を考慮した設計がされており、Poetryを使って環境を整えれば、自分のAPIキーを設定するだけでサンプルがすぐに動くようになっています。

興味のあるフォルダやNotebookを選んで試してみると、MultiOnの使い方や可能性を一通り学習できます。さらに発展させれば、独自のAIエージェントや自動化ツールの構築が可能になるでしょう。
