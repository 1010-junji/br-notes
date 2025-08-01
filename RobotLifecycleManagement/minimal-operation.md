[ロボット ライフサイクル マネジメント（RLM）](index.md) の導入イメージをクリアにするために、最低限度のセットアップ手順と、運用手順についてまとめます。

RLM の真価（[末尾参照](#Git-連携ツールを利用する意義)）は GitHub や GitLab などの Git 連携ツールを利用することにより発揮されますが、ここでは RLM という仕組みの理解を優先して Git の操作についてはコマンドを直接操作する形式で説明します。

<br>

## RLMのセットアップ

### 前提条件

- BizRobo! Lite での運用を想定し、`Management Console` 、`RoboServer` 、 `Synchronizer`、`Git` はすべて同じ端末にセットアップします。
- BizRobo! `v11.5.0.5` に対して RLM のセットアップを行います。
- `Synchronizer` は `RoboServer` に同梱されてインストールされるため、RLMの挙動自体に `RoboServer` は直接関与しませんが、環境条件として入れています。 
- `Git` は別途端末にインストールしておく必要があります。

### Synchronizer の起動方法

`Synchronizer` の起動にはパラメータの設定が必要です。パラメータの仕様については [ヘルプサイト](https://docshield.tungstenautomation.com/RPA/ja_JA/11.5.0-nlfihq5gwr/help/rpa_help/All_Shared/c_startsynch.html) で内容を確認しましょう。

そのうえで、まずはパラメータを設定しないで起動するとどうなるのか見てみましょう。（単なる興味です）
`Synchronizer` がインストールされているフォルダにカレントディレクトリを移したうえで実行します。

**パラメータを付けずに起動**  

```bash
C:\Program Files\BizRobo Basic 11.5.0.5\bin>Synchronizer.exe
--> Wrapper がコンソールとして開始しました
JVM 起動中…
WrapperManager: Initializing...
2025-07-29 14:53:03,267  WARN com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] No settings given, falling back on defaults (use -h to for help)
2025-07-29 14:53:03,271  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set mc_url = http://localhost:8080/ManagementConsole
2025-07-29 14:53:03,271  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set shared_secret = null
2025-07-29 14:53:03,271  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set shared_secret_file = null
2025-07-29 14:53:03,272  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set interval = 3
2025-07-29 14:53:03,272  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set private_key = C:\Users\oishi\.ssh\id_rsa
2025-07-29 14:53:03,272  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set no_host_key = false
2025-07-29 14:53:03,272 ERROR com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] shared_secret or shared_secret_file is missing in default values
<-- Wrapper Stopped
```

**実行内容**
1. 設定有無確認（パラメータ/環境変数/synchronizer.settings）
2. デフォルト値の割り当て起動
3. 必須項目不足によるエラー停止

> [!NOTICE]
> 必須項目が設定されていないと結局シャットダウンされるのに、なぜデフォルト値を設定？
> （デフォルト設定される値はすべて必須項目のため、デフォルト値が活用されることはない。。）

> [!QUESTION]
> - パラメータの設定方法についてはドキュメントあり。
> - 環境変数への設定はDocker環境を想定したIFであり、今回は考慮しない。
> - `synchronizer.settings`への設定方法はパラメータ設定の内容を `-s` で保存。ただ、バイナリ形式なのか、設定状況の確認ができない。（仕様）





<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

> [!TIP]  
> #### Git 連携ツールを利用する意義
>
>Git は単体でも強力なバージョン管理システムですが、GitHub や GitLab といった連携ツール（ホスティングサービス）と組み合わせることで、その真価を最大限に引き出すことができます。特にチームでの開発や、厳格な運用管理が求められる場面では、これらのツールの利用が大きな効果をもたらします。
>
>#### 1. 変更内容の可視化とレビュー文化の醸成
>
>- **プルリクエスト（マージリクエスト）機能**:
>     - 開発ブランチから本番ブランチへの変更を取り込む際に、必ず第三者のレビューと承認を挟むワークフローを構築できます。
>     - コードや設定の変更点についてチーム内で議論し、品質を高め、ナレッジを共有する文化が根付きます。
>     - RLM の文脈では、「開発環境で承認されたロボット」を本番環境へ昇格させる際の公式な承認プロセスとして機能します。
>
>#### 2. CI/CD パイプラインによる自動化
> 
>- **テストとデプロイの自動化**:
>     - プルリクエストが作成されたり、特定のブランチにマージされたりしたタイミングをトリガーに、自動テストやビルド、本番環境へのデプロイといった一連のプロセスを自動実行できます。
>     - RLM と連携させることで、以下のような高度な自動化が可能です。
> - **静的解析**: ロボットファイル内に開発環境用の設定値（URL やファイルパスなど）が残っていないか自動でチェックする。
> - **自動変換**: 開発環境用のリソースを、本番環境用に自動で書き換えてからデプロイする。
> - **デプロイ承認**: テストがすべて成功した場合にのみ、マージ（本番反映）を許可する。
>
>#### 3. ガバナンスとセキュリティの強化
>
>- **厳格なアクセスコントロール**:
>     - リポジトリやブランチごとに、誰が読み書きできるか、誰がマージできるかといった権限を細かく設定できます。
>     - 本番環境（`prod`ブランチなど）を保護し、権限のないユーザーによる意図しない変更を防ぎます。
>- **監査証跡の確保**:
>     - 「誰が、いつ、何を、なぜ変更したか」がプルリクエストの履歴として明確に残るため、監査対応が容易になります。
>- **セキュリティスキャン**:
>     - コードに埋め込まれたパスワードや API キーなどの機密情報を自動で検知し、漏洩を未然に防ぐ機能も利用できます。
>
>#### 4. 堅牢なバックアップとチームの知見集約
>
>- **中央リポジトリとしての役割**:
>     - クラウド上で堅牢に管理されるリポジトリは、ローカル環境や社内サーバーの障害に強い、信頼性の高いバックアップとなります。
>- **Issue や Wiki による情報集約**:
>     - 開発中の課題やバグを管理する Issue（課題管理）機能や、プロジェクトのドキュメントをまとめる Wiki 機能も提供されており、ロボット開発・運用に関する知見をチーム全体で集約・共有するプラットフォームとして活用できます。
