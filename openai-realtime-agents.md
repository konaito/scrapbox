# [openai/openai-realtime-agents][https://github.com/openai/openai-realtime-agents]

以下がこのリポジトリの**本質的なポイント**です。詳細なコードの解説はすでに述べた通りですが、特に押さえておくべき重要事項を簡潔にまとめます。

---

## 1. 「複数エージェント」と「転送(transfer)」の仕組み
- **同じアプリ内に複数のエージェント(LLM)を定義**し、ユーザーとの会話内容に応じて適切なエージェントに切り替えられる仕組み。
- エージェント間の切り替えは `transferAgents` という Function Calling を使い、**LLM が「別エージェントに転送しろ」と判断して JSON を返す→クライアントで実際の切り替え処理**を行う。
- `downstreamAgents` 配列で「どのエージェントに転送が可能か」を宣言すると、`utils.ts` の `injectTransferTools()` で各エージェントに転送用ツール(`transferAgents`)が自動追加される。

## 2. リアルタイム音声ストリーミング (WebRTC) + LLM
- **OpenAI Realtime API** を用いた WebRTC 接続により、ブラウザと OpenAI 間で音声を双方向ストリーミング。
- **音声認識**（Whisper）・**音声合成**（TTS）を行い、ユーザー音声 → テキスト → LLM → テキスト → 音声 の流れをリアルタイムにやり取りする。
- `createRealtimeConnection()` で WebRTC の `offer` を作成し、OpenAI Realtime API から `answer` を受け取って接続完了。  
  - `RTCDataChannel` を通して **ツール呼び出しなどのイベント**を JSON 形式で双方向に送受信する。

## 3. ツール(Function)呼び出しによる拡張
- LLM が会話中に「他のAPIやDBを参照したい」と判断すると、**Function Calling** 形式の JSON を返す（例: `authenticate_user_information`, `lookupOrders` など）。
- フロントエンド（`useHandleServerEvent`）がその JSON を受け取り、該当のツールロジック(`toolLogic`)を実行。  
  - 実行結果をまた LLM に返すことで、LLM がさらに会話を続行。
- 例:  
  1. 返品フローで「注文情報を検索したい」→ `lookupOrders` 関数を呼ぶ JSON を LLM が出力  
  2. 実際の JavaScript 関数(`toolLogic.lookupOrders`)が実行され、結果を LLM に返す  
  3. LLM が続けて返品可否や手続き方法をユーザーに案内

## 4. 「会話ログ(Transcript)」と「イベントログ(Events)」の分離
- ユーザー/アシスタントのメッセージなど、実際の**会話の流れ**は `TranscriptContext` で管理し、`Transcript.tsx` で表示。
- **デバッグ用のイベントログ**（クライアント送信イベント/サーバー受信イベントなど）は `EventContext` で保持し、`Events.tsx` に表示。  
  - どのような JSON が行き来しているかを可視化し、Function Calling のデータなどを確認可能。

## 5. `agentConfigs/` の構成が核心
- このフォルダに**エージェントごとの命令文(instructions)、性格、利用ツール(tools)、下流の転送先(downstreamAgents)などを定義**し、アプリ起動時にまとめて読み込む。
- 各エージェントごとに `instructions` に行動指針やステートマシン風のテキストを記述して、LLM がそのルールに従うよう促す。
- エージェントが最終的に使えるツールは、`tools` と `injectTransferTools()` によって付与された `transferAgents` のみ。**LLM は記載されていないツールや機能は基本的に使えない**ようにする設計。

---

## まとめ

1. **複数エージェント**をシナリオ別に切り替えられる機能が本質。  
2. 音声の双方向ストリーミング（Whisper/TTS）と**Function Calling** を組み合わせることで、**「リアルタイム会話 + ツール呼び出し + エージェント転送」**を実現。  
3. **エージェントの性格やステートマシン、ツール定義**などは `agentConfigs/` に集約されていて、ここが最も重要かつ拡張の鍵となる。  

これらの仕組みにより、音声ベースでユーザーと会話しながら、複雑な認証フロー・返品処理・カスタマーサポートなどを行えるデモとして機能しているのがこのプロジェクトの本質です。
