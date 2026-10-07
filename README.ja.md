[English](README.md) · [简体中文](README.zh-CN.md) · **日本語** · [한국어](README.ko.md)

<p align="center">
  <img src="skills/dori-mode/assets/dori-avatar.png" alt="Dori" width="120">
</p>

# omo-dori-mode-experimental

Dori モードは、コーディングエージェントのセッションひとつを常駐型のメッセンジャーエージェントに変えます。あなたが話す相手は Telegram か Discord のボットひとつだけ。Dori は仕事ごとに herdr のタブでエージェントセッションを立ち上げて任せ、自分が立ち上げたセッションをすべて覚えておき、仕事が本当に終わったときだけ閉じます。PR がマージされ、issue が閉じ、バージョンが公開されたあとです。

中身はスキル(`skills/dori-mode/SKILL.md` と references)と、bun + TypeScript の小さな CLI `dori` です。実験段階なので、粗いところがあります。

## インストール

```sh
curl -fsSL https://raw.githubusercontent.com/sisyphuslabs/omo-dori-mode-experimental/main/install.sh | bash
```

リポジトリを `~/.dori/src` に取得し、スキルを `~/.agents/skills/dori-mode` にリンクし、`bun link` で `dori` を PATH に通し、設定の例を `~/.dori/config.json` にコピーします。エージェントが別の場所からスキルを読む場合は `SKILLS_DIR` を指定してください。

あとは herdr の中でエージェントを開いて「Dori mode」と言うだけです。

## 必要なもの

- [bun](https://bun.sh) 1.3 以上と git
- [herdr](https://herdr.dev):レーンが動くターミナルマルチプレクサ
- スキルを読み込むコーディングエージェント([OmO](https://github.com/code-yeongyu/oh-my-openagent) 向けに作っていますが、起動コマンドは設定で変えられます)
- PR のマージと issue のクローズを確認する `gh`(GitHub CLI)、公開バージョンを確認する `npm`
- ボット本体のための [agent-messenger](https://github.com/agent-messenger/agent-messenger)

ホスト監視は macOS ですべて動きます。Linux では負荷とディスクだけを見て、メモリとスワップは不明として扱います。

## Dori の名前

Dori は最初に、自分を何と呼べばいいかを聞いてきます。「Dori」のままでもいいし、ShipDori や WorkDori のように Dori で終わる名前でも構いません。複数動かすときに見分けやすくなります。決めた名前はボット名、メッセージの署名、モード名にそのまま使われ、次からは「ShipDori mode」の一言で戻せます。

## Slack で使うとき

Slack を選ぶと、Dori はもうひとつ質問して答えを待ちます。

- **ユーザートークン**:ワークスペースの本物のメンバーとして動きます。有料の席がひとつ必要で、その費用はあなたが払います。そのメンバーが見られるものはすべて読め、緑のオンライン表示も保てます。
- **ボットトークン**:Slack アプリとして動きます。席の費用はかかりませんが、招待されたチャンネルと、アプリに与えた権限の範囲しか見えません。

## Dori の話し方

Dori はあなたの話し方に合わせます。短く、くだけた調子で、小文字で書けば、同じように返します。どの言語でも絵文字は使いません。伝えることがいくつかあるときは、長い文章ひとつではなく短いメッセージに分けて送り、それぞれ用意できた時点ですぐに送ります。わざと間をあけることはしません。刻々と変わる進捗だけは例外で、ひとつのメッセージをその場で書き換えていきます。

## 設定

設定はすべて `~/.dori/config.json` にあり、どの項目も省略できます。たいてい決めておくのは次の項目です。

| 項目 | 意味 |
|---|---|
| `leadPane` | Dori 自身の herdr pane(`herdr pane current`)。レーンの報告はここに届きます。 |
| `laneWorkspace` | 新しいレーンのタブを開く herdr workspace |
| `defaultCwd` | レーンが始まるディレクトリで、レーンが worktree を作るリポジトリ |
| `agentCommand` | エージェントの起動コマンド。`{model}` と `{prompt}` を含む argv のリスト |
| `hooks.threadReply`, `hooks.threadDone` | メッセンジャー CLI を `{thread}` と `{text}` を含む argv のリストで。レーンの進捗投稿と完了表示に使います。 |

残り(時間、しきい値、heavy スロット数)は既定値で十分です。全項目の表は [`references/scripts.md`](skills/dori-mode/references/scripts.md) にあります。環境変数 `DORI_CONFIG`、`DORI_STATE_DIR`、`DORI_LEAD_PANE` はファイルより優先されます。

## オンボーディング

最初のセットアップで、Dori は何かを読む前にまず許可を求めます。使っているツール、何をなぜやっているのか、あなたと会社がどんなところかを知ってもいいか、と。許可があったときだけ、ツールをひとつずつ見ていきます。ツールごとに、どの連携を使い、それが何を読むのかを説明してから個別に聞きます。たとえば「Gmail とカレンダーを読む CLI で、予定とメールを見守ってもいいですか?」という具合です。断られたツールは飛ばし、断られたことも覚えておきます。

全体を通して読むだけです。わかったことはその都度メモリに書き、最後に、わかったことと足りないところを短くまとめて伝えます。手順の詳細は [`references/onboarding.md`](skills/dori-mode/references/onboarding.md) にあります。

## 依頼の振り分け

メッセージごとの扱いは Dori が自分で決めます。

- 質問、状況確認、調べもの、数回のツール呼び出しで済む小さな修正は、新しいセッションを開かずにその場で片づけます。
- PR で終わるコード作業、何段階もある仕事、長くかかる仕事や並列にできる仕事はレーンを開きます。そのリポジトリを担当していて手が空いているレーンがあれば、新しく開かずにそちらへ渡します。
- メモリ、ディスク、pane 数に余裕がないとき(`dori can-launch` が HOLD)は何も開きません。仕事を待ち行列に入れ、理由を伝えます。
- 新しい仕事は新しいスレッドで始めます。続きの話は元のスレッドに戻し、レーンが閉じていれば記録してあるセッションを開き直します。ちょっとした質問は、聞かれたその場で答えます。

## セッションレジストリ

レーンごとに `~/.dori/state/lanes/` の下に JSON ファイルがひとつできます。メッセンジャーのスレッドと herdr の pane、pane とエージェントのセッション id を結びつけ、状態(`working`、`done-claimed`、`verified-done`、`not-done`、`closed`)とその変化の履歴を残します。

`dori sync` はこの記録を実際に動いている pane と突き合わせて、ずれを教えてくれます。消えた pane、変わったセッション id、完了を証明する手段がないレーンなどです。何も削除しません。`--write` を付けると、見つけたセッション id を保存します。

## 5 分の完了フロー

レーンが完了を申告します。

```sh
dori claim-done fix-login --evidence "merged acme/app#412 (a1b2c3d)"
```

Dori には `LANE_DONE_CLAIMED` が届き、レーンには 5 分後に閉じると伝わります。その間に異議を出せます。

```sh
dori object-done fix-login --reason "changelog の項目が抜けている"
```

理由はそのままレーンに届き、レーンは直してからもう一度申告します。誰も異議を出さなければ、時間が来たところで `dori watch` がレーンを閉じます。閉じる前に `Done =` のシグナルをすべて実際に読み直し、worktree にリモートへ届いていないコミットや未コミットの変更があれば閉じません。その場合、申告は理由付きで not-done に戻ります。watcher を再起動しても、5 分の時計は最初からにはなりません。

### 何をもって完了とするか

レーンの `Done =` 行には、watcher が自分で確かめられるシグナルを書きます。
- PR のマージ
- issue のクローズ
- パッケージのバージョン公開
- PR で終わらない仕事(ローカルのセットアップ、QA、立ち上げたサービス)なら、終了コード 0 で終わるコマンド、ハッシュや JSON の値が合うファイル、期待どおりのステータスと本文で応答する URL

```
Done = command ["bun","test"] stdout~" 0 fail"; file qa/report.json json:.passed=true; url http://localhost:3000/health body~"ready"
```

watcher はレーンを閉じるときに、シェルを通さずこの確認を自分で実行し直し、レーンの言い分をそのまま信じることはありません。解釈できないシグナルは、レーンの起動時に拒否します。文法の全体は [`references/sessions.md`](skills/dori-mode/references/sessions.md) にあります。

## コマンド

| コマンド | やること |
|---|---|
| `dori launch <key> ...` | brief にレーンの footer を書き、タブを開き、エージェントを起動し、起動エラーを確認 |
| `dori adopt <key> --pane ID ...` | すでに動いているレーンを登録 |
| `dori sync [--write]` | レジストリと実際の pane を比べ、ずれを表示 |
| `dori claim-done` / `object-done` / `close` | 完了フロー |
| `dori watch` | 自動クローズの watcher。常駐モニターとして動かします |
| `dori freshness [--loop MIN]` | 静かになったレーンに声をかけ、最後の報告をスレッドに投稿 |
| `dori dead-panes [--loop MIN]` | 止まったエージェントの pane を報告 |
| `dori guard [--loop MIN]` | 負荷、メモリ、ディスク、pane 数の警告 |
| `dori heavy <label> -- <cmd>` | スロットが空いて負荷が低いときだけビルドやテストを実行 |

pane に送る文字列は必ず引数ひとつとして渡し、シェル文字列を通しません。Enter が本当に入ったかも確認します。

## ユーティリティ

CLI には、Dori に必要なメッセンジャーまわりの機能も入っています。`scripts/src/messenger/` の型付きモジュールとして取り込むこともできます。

| コマンド | やること |
|---|---|
| `dori send slack\|telegram\|discord --to T --text X [--thread ID] [--edit ID]` | メッセージの送信と編集。レート制限は待ってから送り直し、`$(` を含む文字列は拒否します |
| `dori presence slack\|discord` | アカウントをオンライン表示のまま保つ。Discord ボットはゲートウェイで、Slack のユーザーアカウントは 1 分ごとに合図を送る Web クライアント型ソケットで保ちます |
| `dori transcribe <file>` | `hooks.transcribe` のコマンドで音声メッセージを文字にする |
| `dori can-launch` | もうひとつレーンを開く余裕があるかを確かめる |
| `dori inbound slack [--loop MIN]` | Slack で Dori 宛てのものを取りこぼさない。Threads 画面の未読の返信、Dori が書き込んだスレッドの新しい返信(タグなしでも)、未読メンションのある DM とチャンネル |

コマンドはなくても、モジュールで使える機能もあります。
- Telegram:「Thinking…」から始まる `sendMessageDraft` のストリーミング、フォーラムのトピック、HTML の表
- Discord:スレッドの作成、名前の変更、アーカイブ
- Slack:ファイルのアップロード
- `typingWhile`:作業が動いているあいだ入力中の表示を出しておきます

Slack で Dori が送るメッセージは、どの関数から送ってもそのスレッドを記録します。生の API 呼び出しで投稿した親メッセージの下に、タグなしで付いた返信も届きます。メッセージイベントを聞くだけのやり方では、こうした返信を取りこぼします。

トークンは `DORI_SLACK_TOKEN`(ユーザートークンなら `DORI_SLACK_COOKIE` も)、`DORI_TELEGRAM_TOKEN`、`DORI_DISCORD_TOKEN` から読みます。

## テスト

CI はありません。テストはローカルで回します。

```sh
cd skills/dori-mode/scripts
bun install
bun test           # 偽の herdr、git、gh で振る舞いを確かめるテスト
bunx tsc --noEmit  # 型チェック
```

テストが本物の pane、リポジトリ、GitHub に触れることはありません。

## ライセンス

MIT

## OmOMeow からの移行

以前の gist で OmOMeow モードを使っていたなら、ボットはそのまま動きます。次の 4 つを変えれば Dori になります。

1. **Dori の名前を決める。**「Dori」だけでも、ShipDori や WorkDori のように Dori で終わる名前でも構いません。エージェントに「これからきみの名前は ShipDori で、これは ShipDori mode だよ」と伝えてください。それ以降は「OmOMeow mode」ではなく「ShipDori mode」がモードを呼び出す言葉になります。
2. **ボットの名前とプロフィール画像を変える。**
   - Telegram:@BotFather で `/setname` を送り、ボットを選んで新しい名前を送ります。続けて `/setuserpic` を送り、ボットを選んで新しい画像を送ります。既定の Dori の絵は [`skills/dori-mode/assets/dori-avatar.png`](skills/dori-mode/assets/dori-avatar.png) です。好きな画像に替えても構いません。ボットの名前と画像は BotFather からしか変えられません。
   - Discord:Developer Portal でアプリケーションを開きます。**Bot** ページでユーザー名とアイコンを、**General Information** でアプリ名とアイコンを変えて(同じ既定の絵が使えます)保存します。
3. **このリポジトリを入れる。** 上の 1 行インストールを実行し、エージェントに「ShipDori mode」と言えば、以前貼り付けていたプロンプトの代わりにスキルと `dori` CLI を使います。
4. **オンボーディングをする。** OmOMeow のときにやっていなければ「オンボーディングして」と言うだけです。

これまでのスレッド、トピック、メモリはそのまま残ります。
