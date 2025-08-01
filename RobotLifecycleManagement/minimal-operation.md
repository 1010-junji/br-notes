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
C:\Users\ore> cd C:\Program Files\BizRobo Basic 11.5.0.5\bin

C:\Program Files\BizRobo Basic 11.5.0.5\bin>Synchronizer.exe
--> Wrapper がコンソールとして開始しました
JVM 起動中…
WrapperManager: Initializing...
2025-07-29 14:53:03,267  WARN com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] No settings given, falling back on defaults (use -h to for help)
2025-07-29 14:53:03,271  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set mc_url = http://localhost:8080/ManagementConsole
2025-07-29 14:53:03,271  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set shared_secret = null
2025-07-29 14:53:03,271  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set shared_secret_file = null
2025-07-29 14:53:03,272  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set interval = 3
2025-07-29 14:53:03,272  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set private_key = C:\Users\ore\.ssh\id_rsa
2025-07-29 14:53:03,272  INFO com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] Set no_host_key = false
2025-07-29 14:53:03,272 ERROR com.kapowtech.synchronizer.runtime.Settings - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.runtime.Settings] shared_secret or shared_secret_file is missing in default values
<-- Wrapper Stopped
```

どうやらデフォルト値が代わりに設定されるものの、`shared_secret` も `shared_secret_file` も null であるため、そのまま処理が終了します。

次に、以下の通りパラメータを指定して起動します。

**パラメータを指定して実行**
```bash
C:\Program Files\BizRobo Basic 11.5.0.5\bin>Synchronizer.exe -c ^
  --mc-url http://localhost:8080 ^
  --shared-secret fxxxxxxxxxxxxxAZP83ts5-KZCmat-WeuKz5NBBqwEmdDxxxxxxxxxg ^
  --interval 10 --no-host-key false ^
  --private-key p:\dummy
```

- `-c` により実行時パラメータとして指定する。
- `--mc-url` は各自の `Management Console` を指定
- `--shared-secret` は `Management Console` の（管理 > サービス認証 > シンクロナイザー）から共有シークレットを取得して指定[^1]
- `--interval` は、`Synchronizer` が同期する間隔。30秒～60秒ぐらいの間隔でもよさそう。
- `--no-host-key` については、今回 Git連携ツールと接続しないためどちらでもいいが、デフォルト値を採用
- `--private-key` についても、今回 Git連携ツールと接続しないため内容についてはダミー値を設定。ただし、設定必須項目のため省略はできない。

[^1]: 共有シークレットを取得するためには管理者アカウントで `Management Console` にログインする必要があります。

また、`Synchronizer` と連携する `Management Console` 側のリポジトリの設定を以下に示します。

![[project.pj_ua.repository_top.png]]


```bash
C:\Program Files\BizRobo Basic 11.5.0.5\bin>Synchronizer.exe -c ^
  --mc-url http://localhost:8080 ^
  --shared-secret fxxxxxxxxxxxxxAZP83ts5-KZCmat-WeuKz5NBBqwEmdDxxxxxxxxxg ^
  --interval 10 --no-host-key false ^
  --private-key p:\dummy

--> Wrapper がコンソールとして開始しました
JVM 起動中…
WrapperManager: Initializing...
2025-07-30 17:45:30,464  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] started Synchronizer for http://localhost:8070 with a poll interval of 10s
2025-07-30 17:45:31,346  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua using com.kapowtech.synchronizer.git.GitSynchronizer
2025-07-30 17:45:31,813  INFO com.kapowtech.synchronizer.git.GitSynchronizer - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.git.GitSynchronizer] cloned remote C:\RobotLifecycleManagement into local repository C:\Users\oishi\AppData\Local\Kofax RPA\11.5.0.5_549\Data\Synchronizer\pj_ua_localhost_RobotLifecycleManagement_develop (b47774543aa6e8e97ec90a438428755d8773dad5)
2025-07-30 17:45:32,227  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] Missing unknown 17f5107aaf9041f927f2e0bf7710caf2f6181269. About to reset pj_ua and to try once more.
2025-07-30 17:45:32,453  INFO com.kapowtech.synchronizer.git.GitSynchronizer - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.git.GitSynchronizer] cloned remote C:\RobotLifecycleManagement into local repository C:\Users\oishi\AppData\Local\Kofax RPA\11.5.0.5_549\Data\Synchronizer\pj_ua_localhost_RobotLifecycleManagement_develop (b47774543aa6e8e97ec90a438428755d8773dad5)
2025-07-30 17:45:32,933  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua 8f95476d370e537e8d1896aa091d73adfbf8e46d
2025-07-30 17:45:32,934  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua_prod using com.kapowtech.synchronizer.git.GitSynchronizer
2025-07-30 17:45:33,042  INFO com.kapowtech.synchronizer.git.GitSynchronizer - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.git.GitSynchronizer] cloned remote C:\RobotLifecycleManagement into local repository C:\Users\oishi\AppData\Local\Kofax RPA\11.5.0.5_549\Data\Synchronizer\pj_ua_prod_localhost_RobotLifecycleManagement_prod (b47774543aa6e8e97ec90a438428755d8773dad5)
2025-07-30 17:45:33,056  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] Missing unknown b814041f7cd7a6fcae380650cd154846735f508b. About to reset pj_ua_prod and to try once more.
2025-07-30 17:45:33,193  INFO com.kapowtech.synchronizer.git.GitSynchronizer - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.git.GitSynchronizer] cloned remote C:\RobotLifecycleManagement into local repository C:\Users\oishi\AppData\Local\Kofax RPA\11.5.0.5_549\Data\Synchronizer\pj_ua_prod_localhost_RobotLifecycleManagement_prod (b47774543aa6e8e97ec90a438428755d8773dad5)
2025-07-30 17:45:33,243  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua_prod b47774543aa6e8e97ec90a438428755d8773dad5
2025-07-30 17:45:43,288  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua using com.kapowtech.synchronizer.git.GitSynchronizer
2025-07-30 17:45:43,390  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua 8f95476d370e537e8d1896aa091d73adfbf8e46d
2025-07-30 17:45:43,391  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua_prod using com.kapowtech.synchronizer.git.GitSynchronizer
2025-07-30 17:45:43,445  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua_prod b47774543aa6e8e97ec90a438428755d8773dad5
2025-07-30 17:45:53,485  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua using com.kapowtech.synchronizer.git.GitSynchronizer
2025-07-30 17:45:53,569  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua 8f95476d370e537e8d1896aa091d73adfbf8e46d
2025-07-30 17:45:53,569  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua_prod using com.kapowtech.synchronizer.git.GitSynchronizer
2025-07-30 17:45:53,627  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua_prod b47774543aa6e8e97ec90a438428755d8773dad5
2025-07-30 17:46:03,675  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua using com.kapowtech.synchronizer.git.GitSynchronizer
2025-07-30 17:46:03,774  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua 8f95476d370e537e8d1896aa091d73adfbf8e46d
2025-07-30 17:46:03,774  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua_prod using com.kapowtech.synchronizer.git.GitSynchronizer
2025-07-30 17:46:03,833  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua_prod b47774543aa6e8e97ec90a438428755d8773dad5
CTRL-C トラップされました。シャットダウン中。
CTRL-C トラップされました。即時のシャットダウンを強制中。
JVM はリクエストに応じて終了しませんでした、中止をします。
JVM はリクエストに応じて中止しました。
<-- Wrapper Stopped
```

**実行内容**
1. パラメータ条件に沿って Synchronizer 起動
	- http://localhost:8070 の MC に対して 10秒間隔で監視
2. MC内の設定に沿って監視開始（Synchronizerが有効化されているプロジェクトに対して監視実行）
	- プロジェクト `pj_ua` のGitとの同期開始
	- プロジェクトで設定した `URL/パス` にベア リポジトリ作成
	- 作成したローカルリポジトリにプロジェクトで設定したブランチ（`develop`）を作成
3. ベア リポジトリをSynchronizer実行ユーザーのアプリケーションデータフォルダ内にクローンして個別のローカルリポジトリを作成
	- `%LOCALAPPDATA%\Kofax RPA\11.5.0.5_549\Data\Synchronizer\pj_ua_localhost_RobotLifecycleManagement_develop`
4. 10秒間隔でのリポジトリ監視開始

> [!INFORMATION]
> リポジトリ用のフォルダは事前に作成しておかなくても `MC` のプロジェクトに設定した値をもとに Synchronizer が自動的に作成します。
> クローンしたレポジトリには同期対象のオブジェクトもファイルとして自動生成されます。これは、MCではこれらのオブジェクト情報をDB内に格納していますが、Gitによる更新情報をファイルベースで感知するために複製しているものと思われます。







> [!QUESTION]
> 現行仕様ではフォーマット上 `--private-key` が必要ですが、実在にかかわらずパス形式で指定されていればOK。
> 上の例ではドライブもフォルダも存在しない値（`p:\dummy`）を指定していますが、ローカル環境のみの運用の場合、問題ありません。
> よくある設定値としては `%USERPROFILE%\.ssh\id_rsa`


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
