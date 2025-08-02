[ロボット ライフサイクル マネジメント（RLM）](index.md) の導入イメージをクリアにするために、最低限度のセットアップ手順と、運用手順についてまとめます。

RLM の真価（[末尾参照](#Git-連携ツールを利用する意義)）は GitHub や GitLab などの Git 連携ツールを利用することにより発揮されますが、ここでは RLM という仕組みの理解を優先して Git の操作についてはコマンドを直接操作する形式で説明します。

<br>

## RLMのセットアップ

### 前提条件

- BizRobo! Lite での運用を想定し、`Management Console` 、`RoboServer` 、 `Synchronizer`、`Git` はすべて同じ端末にセットアップします。
- BizRobo! `v11.5.0.5` に対して RLM のセットアップを行います。
- `Synchronizer` は `RoboServer` に同梱されてインストールされるため、RLMの挙動自体に `RoboServer` は直接関与しませんが、対象の端末には `RoboServer` をインストールしておく必要があります。 
- `Git` は別途端末にインストールしましょう。
　  
	BizRobo! Lite では `Management Console` は1台のため、本番用のプロジェクト（`pj_ua_prod`: `prod` ブランチ）と開発用のプロジェクト（`pj_ua`: `develop` ブランチ）を用意して、両者を連携させる前提で記述します。[^1]

	以降の記述では、`Design Studio` で作成したロボットが `pj_ua` プロジェクトにアップロードされた状態をスタートして `Synchronizer` を起動していきます。（`pj_ua_prod` プロジェクトは空の状態）

	![構築前提の構成](images/rlm-lite.drawio.svg)

[^1]: ただし、実際 BizRobo! Lite において、このような構成をとるのは意味がありません。今回は RLM のハンズオンを想定しているためこのような構成にしていますが、Lite において RLM を活用するのであれば、`develop`リポジトリのみを用意し、いざというときの切り戻しのために、本番環境へのアップロード情報を履歴として記録しておくといった利用法が適切だと思います。

### Synchronizer の起動方法

`Synchronizer` の起動にはパラメータの設定が必要です。パラメータの仕様については [ヘルプサイト](https://docshield.tungstenautomation.com/RPA/ja_JA/11.5.0-nlfihq5gwr/help/rpa_help/All_Shared/c_startsynch.html) で内容を確認しましょう。

そのうえで、まずはパラメータを設定しないで起動するとどうなるのか見てみましょう。（単なる興味です）
`Synchronizer` がインストールされているフォルダにカレントディレクトリを移したうえで実行します。

**パラメータを付けずに起動**  
```cmd
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
```cmd
C:\Program Files\BizRobo Basic 11.5.0.5\bin>Synchronizer.exe -c ^
  --mc-url http://localhost:8080 ^
  --shared-secret fxxxxxxxxxxxxxAZP83ts5-KZCmat-WeuKz5NBBqwEmdDxxxxxxxxxg ^
  --interval 10 --no-host-key false ^
  --private-key p:\dummy -s
```

- `-c` により実行時パラメータとして指定する。
- `--mc-url` は各自の `Management Console` を指定
- `--shared-secret` は `Management Console` の（管理 > サービス認証 > シンクロナイザー）から共有シークレットを取得して指定[^2]
- `--interval` は、`Synchronizer` が同期する間隔。30秒～60秒ぐらいの間隔でもよさそう。
- `--no-host-key` については、今回 Git連携ツールと接続しないためどちらでもいいが、デフォルト値を採用
- `--private-key` についても、今回 Git連携ツールと接続しないため内容についてはダミー値を設定。ただし、設定必須項目のため省略はできない。
- `-s` によりコマンドラインに指定したパラメータをファイルに記録します。

[^2]: 共有シークレットを取得するためには管理者アカウントで `Management Console` にログインする必要があります。

> [!NOTE]
> - パラメータ設定の種類は3つ。
> 	- `-c` 実行時にキーと値のセットで指定
> 	- `-e` Docker上で `Scynchronizer` を動かす際のDockerの環境変数
> 	- なし。 `-c` と同時に `-s` オプションを付与し手実行すると、`synchronizer.settings` ファイルとしてパラメータが出力されます。そのため、`synchronizer.settings` ファイルの存在する環境においてはパラメータを指定することなく `Scynchronizer` を起動できます。
> 
> `synchronizer.settings` は `%LocalAppData%\Kofax RPA\11.5.0.5_549\Configuration` 配下に出力されます。

また、`Synchronizer` と連携する `Management Console` 側のリポジトリの設定を以下に示します。
URLに設定しているのが `Bare Git Repository` のパスです。今回は Git連携ツール を使用しないため、直接ローカルにリモートリポジトリ（という位置づけになるベア リポジトリ）を作成します。

以下のコマンドによりベア リポジトリを生成しますが、初回 `Scynchronizer` 起動時に当該のリポジトリがない場合には、`Scynchronizer` によりベア リポジトリが自動的に生成されます。

```cmd
C:\Users\ore\Desktop> mkdir C:\RobotLifecycleManagement
C:\Users\ore\Desktop> cd C:\RobotLifecycleManagement
C:\RobotLifecycleManagement> git init --bare
```

![リポジトリの設定](images/project.pj_ua.repository_top.png)

```cmd
C:\Program Files\BizRobo Basic 11.5.0.5\bin>Synchronizer.exe -c ^
  --mc-url http://localhost:8080 ^
  --shared-secret fxxxxxxxxxxxxxAZP83ts5-KZCmat-WeuKz5NBBqwEmdDxxxxxxxxxg ^
  --interval 10 --no-host-key false ^
  --private-key p:\dummy

--> Wrapper がコンソールとして開始しました
JVM 起動中…
WrapperManager: Initializing...
2025-07-30 17:45:30,464  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] started Synchronizer for http://localhost:8080 with a poll interval of 10s
2025-07-30 17:45:31,346  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua using com.kapowtech.synchronizer.git.GitSynchronizer
2025-07-30 17:45:31,813  INFO com.kapowtech.synchronizer.git.GitSynchronizer - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.git.GitSynchronizer] cloned remote C:\RobotLifecycleManagement into local repository C:\Users\ore\AppData\Local\Kofax RPA\11.5.0.5_549\Data\Synchronizer\pj_ua_localhost_RobotLifecycleManagement_develop (b47774543aa6e8e97ec90a438428755d8773dad5)
2025-07-30 17:45:32,227  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] Missing unknown 17f5107aaf9041f927f2e0bf7710caf2f6181269. About to reset pj_ua and to try once more.
2025-07-30 17:45:32,453  INFO com.kapowtech.synchronizer.git.GitSynchronizer - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.git.GitSynchronizer] cloned remote C:\RobotLifecycleManagement into local repository C:\Users\ore\AppData\Local\Kofax RPA\11.5.0.5_549\Data\Synchronizer\pj_ua_localhost_RobotLifecycleManagement_develop (b47774543aa6e8e97ec90a438428755d8773dad5)
2025-07-30 17:45:32,933  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua 8f95476d370e537e8d1896aa091d73adfbf8e46d
2025-07-30 17:45:32,934  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua_prod using com.kapowtech.synchronizer.git.GitSynchronizer
2025-07-30 17:45:33,042  INFO com.kapowtech.synchronizer.git.GitSynchronizer - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.git.GitSynchronizer] cloned remote C:\RobotLifecycleManagement into local repository C:\Users\ore\AppData\Local\Kofax RPA\11.5.0.5_549\Data\Synchronizer\pj_ua_prod_localhost_RobotLifecycleManagement_prod (b47774543aa6e8e97ec90a438428755d8773dad5)
2025-07-30 17:45:33,056  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] Missing unknown b814041f7cd7a6fcae380650cd154846735f508b. About to reset pj_ua_prod and to try once more.
2025-07-30 17:45:33,193  INFO com.kapowtech.synchronizer.git.GitSynchronizer - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.git.GitSynchronizer] cloned remote C:\RobotLifecycleManagement into local repository C:\Users\ore\AppData\Local\Kofax RPA\11.5.0.5_549\Data\Synchronizer\pj_ua_prod_localhost_RobotLifecycleManagement_prod (b47774543aa6e8e97ec90a438428755d8773dad5)
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
：
```

**実行内容**
1. `Management Console`（http://localhost:8080） に対して 10秒間隔の監視を開始
2. リポジトリの設定に基づき、ローカルリポジトリを設定数分作成
	- プロジェクト `pj_ua` 、`pj_ua_prod` のベア リポジトリを確認し、ローカルリポジトリを clone
	- ログからはわからないが、おそらくブランチも作成（`pj_ua=develop`、`pj_ua_prod=prod`）
3. 全てのリポジトリに対して、10秒間隔での監視開始


### 同期確認

`Synchronizer` が起動して同期処理を開始したのち、`Management Console` の画面からその状況を確認します。
以下図のようにプロジェクト `pj_ua` のリビジョン番号が `local` ではなく `1c0c0c15a06....` というGitで発行される形式の文字列に変わっていれば、無事 `develop` ブランチについては同期ができたといえます。

![リビジョン番号更新](repository.revision.updated.png)

この時点で `pj_ua_prod` プロジェクトにはロボットが登録されていないためリストには表示されませんが、後述の手順で `pj_ua` プロジェクトのロボットがマージされると下記のように同じリビジョン番号が付与されます。

![ロボット同期](images/repository.revision.synched.png)

> [!Important]  
> RLMによって同期される先のプロジェクト（今回の場合 `pj_ua_prod` ）については、プロジェクトのリポジトリ設定に対しては「読み取り専用」チェックを ON にすることを推奨します。
> 
> 仮に、`pj_ua_prod`が「読み取り専用」設定されておらず、間違って直接 `Management Console` にロボット等のオブジェクトを登録してしまった場合、`pj_ua` のリポジトリと状態の不整合（conflict）が発生し、`pj_ua_prod` への同期が止まってしまうこともあるので注意が必要です。[^3]

[^3]: 中身の違う同一オブジェクトが発見された場合、`develop` → `prod` へマージを行う際に conflict が発生します。また、conflict が発生しない場合でも `prod` 側だけに存在し、`develop` には存在しないオブジェクトが発生してしまうため、資材管理上もよろしくないでしょう。


<br>

## RLMを使った本番環境の運用

### 前提条件

- ロボット開発者とロボット管理者が分かれており、本番環境へのアクセスはロボット管理者に限られている場合。
- 開発・テスト環境と本番環境が `Management Console` 上でプロジェクトとして分かれており、本番用のプロジェクトは「読み取り専用」設定により、RLMからのみ更新が可能な状態とする。
- Git連携ツールは使用しないため、Gitクライアント（今回はGitに同梱されている`Git bash`）を使用する

### 本番更新の基本的な流れ


### 通常作業（基本の型）


### 本番更新の取り消し、切り戻し


### 分割更新


### コンフリクトが発生した際の対応

<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

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