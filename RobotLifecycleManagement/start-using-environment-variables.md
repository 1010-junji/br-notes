## 環境変数 `-e` を使用した Synchronizerの起動方法

Docker 利用時の `README` ファイルからの抽出ですが、下記でも示すように Docker を使わない場合であっても環境変数による起動は可能です。

> [!IMPORTANT]  
> Synchronizer を起動する際のパラメータには機密性の高い情報も含まれるため、平文のまま環境変数に設定することは好ましくありません。利用方法としては、「本番環境は `-s` で指定した設定値を使うものの、デバッグ等で試験的に動作を切り替えたい場合に、`-e` を指定して起動する。」のがいいと思います。

### 環境変数一覧
#### `SYNCHRONIZER_MC_URL`

プロトコルとポート番号を含む、Management Console に接続するための URL。

#### `SYNCHRONIZER_SHARED_SECRET`

Management Console の初期同期に使用する共有シークレット（平文）。

#### `SYNCHRONIZER_SHARED_SECRET_FILE`

Management Console の初期同期で使用する共有シークレットを含むファイルのパス。

#### `SYNCHRONIZER_INTERVAL`

同期処理の実行間隔（秒単位）。0 以下の値、または数値以外を指定した場合、Synchronizer は一度だけ実行されて終了します。

#### `SYNCHRONIZER_PRIVATE_KEY`

リモートリポジトリに接続するための SSH 秘密鍵が格納されたファイルのパス。ローカルリポジトリに接続する場合はこの属性は無視されますが、値の指定自体は必要です。

#### `SYNCHRONIZER_NO_HOST_KEY`

SSH 接続時の厳密なホストキー検証を無効にします。

### 動作例

環境変数のうち、下記4つを設定して起動する例を示します。
- SYNCHRONIZER_MC_URL
- SYNCHRONIZER_SHARED_SECRET
- SYNCHRONIZER_PRIVATE_KEY
- SYNCHRONIZER_INTERVAL

```shell
:: 設定されている環境変数を確認
C:\Program Files\BizRobo Basic 11.5.0.5\bin>echo %SYNCHRONIZER_MC_URL%
http://localhost:8070

C:\Program Files\BizRobo Basic 11.5.0.5\bin>echo %SYNCHRONIZER_SHARED_SECRET%
fOLLg7xexxxxxxxx............................................H6JrdGg

C:\Program Files\BizRobo Basic 11.5.0.5\bin>echo %SYNCHRONIZER_PRIVATE_KEY%
p:\dummy

C:\Program Files\BizRobo Basic 11.5.0.5\bin>echo %SYNCHRONIZER_INTERVAL%
30

:: `-e` オプションをつけて実行
C:\Program Files\BizRobo Basic 11.5.0.5\bin>Synchronizer.exe -e
--> Wrapper がコンソールとして開始しました
JVM 起動中…
WrapperManager: Initializing...
2025-08-18 09:25:54,917  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] started Synchronizer for http://localhost:8070 with a poll interval of 30s
2025-08-18 09:25:55,894  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua using com.kapowtech.synchronizer.git.GitSynchronizer
2025-08-18 09:25:56,865  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua d173db336a5abbcf400b0c2a714314b80d2051e5
2025-08-18 09:25:56,866  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] starting the synchronization of pj_ua_prod using com.kapowtech.synchronizer.git.GitSynchronizer
2025-08-18 09:25:57,041  INFO com.kapowtech.synchronizer.Main - [EVENT UNSPECIFIED -> /com.kapowtech.synchronizer.Main] done synchronizing pj_ua_prod d173db336a5abbcf400b0c2a714314b80d2051e5
:
```

起動時の `-e` 付け忘れにご注意ください。