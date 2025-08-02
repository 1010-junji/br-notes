Robot Lifecycle Management（以下 RLM）は、BizRobo! の運用管理機能として備わった Git ベースの構成管理・デプロイ機構です。ロボット、スニペット、リソースなどの資産を Git リポジトリで一元管理し、開発環境／テスト環境・本番環境の間で安全に同期・昇格（Promote）・ロールバックを行えるようにすることで、複数の開発者による並行開発やガバナンス、監査対応を強化します。

<br/>

## 1. Robot Lifecycle Management とは？

- **定義**
    
    - RLM は「Management Console 上のオブジェクト」と「Git リポジトリ」の状態を Synchronizer 経由で双方向比較・同期する仕組みで、Git に準拠した完全なバージョン履歴を保持します。
        
- **同期対象オブジェクト**
    
    - robots / types / snippets / resources / schedules / OAuth 設定 を含む複数種のワークオブジェクトを取り扱えます。(何を同期対象とするかについては、 `Management Console` のプロジェクト管理機能から選択します。) 
        
- **Git プロトコル**
    
    - HTTP(S), SSH, git いずれの転送プロトコルにも対応しており、オンプレでもクラウドでも同じ手順で運用できます。([公式のドキュメント](https://my.bizrobo.com/product/list/?typeCode=Document)ではクラウドを前提とした記述しかないので、ここではオンプレでの簡易環境を前提として進めます。)

<br>

> [!IMPORTANT]
> **RLMとして説明されている誤解**  
> 
> RLMは「ロボット開発」の資材管理を Gitと連携して行うものではありません。開発・編集作業を経て本番環境で動かす、またはManagement Console に登録されたスケジュールや認可情報を、Git の機能を活用して開発用環境と連携したり、オブジェクトのバージョン管理をする機能です。
> 
> Design Studioで扱う作成中のロボットファイルはRLMでバージョン管理する対象ではないため、別途専用に Git リポジトリを作成したうえで 管理することを推奨します。

<br>


## 2. RLM を構成する主な要素

> GitはBizRobo!には同梱されていないため、別途インストールする必要があります。

| 要素                              | 役割                                                                                                                | 補足                                                                                                                 |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Bare Git Repository**         | RLMにおいてブランチごとの履歴が完全に残る「原本リポジトリ」                                                                                   | 空の bare リポジトリを先に用意するのが推奨。                                                                                          |
| **Management Console（開発用/本番用）** | ロボット実行・運用を管理する機能                                                                                                  | 開発用/本番用のブランチごとに `Management Console` を分ける構成が推奨。                                                                    |
| **Synchronizer（開発用／本番用）**       | `Management Console` と `Git` を定期同期する機能                                                                            | RESTを通じて周期的に`Management Console`の状態を監視し、差分を発見したら`Git`と同期。                                                          |
| **Promotion Manager（人の役割）**     | 開発用環境で承認されたコミットを本番用環境へ昇格、または本番環境で問題が見つかったコミットをロールバックする運用の管理者                                                      | `Promotion Manager`用に `Bare Git Repository` から clone したリポジトリに対して、Git を直接操作することで実行。 SourceTree など GUI クライアントの利用も推奨。 |
| **Git 連携ツール**                   | `Promotion Manager` がプルリクエストの承認をしたり、ローカルリポジトリから連携されたオブジェクトをCI/CDパイプラインで処理させるためのツール。GitHub, Bitbucket, 自社 Git サーバー | CI/CDパイプラインによって「開発環境」依存のリソースファイルを「本番環境」用に書き換えたり、ファイル内を精査して本番環境に上げてはいけない情報が混ざっていないかのチェックを実施可能。                      |

![RLM を構成する主な要素](images/rlm.drawio.svg)

<br>

## 3. 用途・目的・代表ユースケース

> Chat GPTにより生成したコンテンツを転載。いずれもGitHubやGitLabへの連携を前提とし、その機能を十二分に発揮している例

- **大規模開発のガバナンス強化**
	 - 数百～数千体のロボットを Git で一括管理し、変更履歴・責任追跡・ロールバックを即時実行できる。
          
- **CI/CD パイプラインの構築**
     - GitHub Actions などと組み合わせて自動テスト → 自動デプロイを行い、DevOps 的な高速リリースを実現。
          
- **規制・監査対応**
    - 各コミットに誰がいつ何を変更したかが残るため、金融・医療など厳格な監査要件を満たしやすい。
          
- **マルチ環境運用**
    - 開発・検証・本番のManagement Consoleをブランチで対応づけ、同期ポリシーを変更するだけで Hotfix を安全に展開。
          
- **災害復旧 (DR)**
    - Git リポジトリがバックアップになるため、Management Console 障害時も `git clone`＋同期で即復旧が可能。
          

<br>


## 4. 導入フローとアクションの簡易例

> [!NOTE]
> Git 連携ツールは使用せず、最低限 RLM の動作を体感するための簡易フロー。Git 連携ツールの代わりに Promotion Manager 用にローカルブランチを clone し、Git クライアントから直接リポジトリを操作する（Pull、Merge、Push）ことでリポジトリ連携を実現する。

1. **ブランチ戦略を決定**（`prod=本番用`, `develop=開発用` の２ブランチでの運用とする。）[^1]
2. **Bare リポジトリを作成**し、`Promotion Manager` 用にローカルリポジトリを clone
3. **開発用・本番用の Management Console で Git URL(/ファイルパス) とブランチ名を設定**（管理 > プロジェクト > リポジトリ）
4. **Synchronizer を CLI で設定保存** [^2]
    
   ```cmd
    REM BizRobo! v11.5 を前提に記載（パラメータは利用するバージョンにより異なります。）
    Synchronizer.exe -c ^
      --mc-url http://localhost:8070 ^
      --shared-secret <secret> ^
      --interval 10 --no-host-key true ^
      --private-key p:\dummy -s
    ```
    
    - Git連携ツールは使用しないため、`--private-key` はダミーの値で問題ありません。
    - `-s` で `synchronizer.settings` を保存し終了。（v11.5の場合、`%LOCALAPPDATA%\Kofax RPA\11.5.0.5_549\Configuration` にファイルが生成されます。）
        
5. **同期確認** –  リポジトリ設定をしたプロジェクトに登録されているロボットの `Management Console` 上でのリビジョン番号が `local` ではなく、`1c0c0c15a06....` という形式の番号に更新されていることを確認。
6. **昇格／ロールバック 手順を定義** 
    - 開発用環境の更新を確認し、本番用リポジトリにマージ（origin/develop -[pull]-> develop -[merge]-> prod -[push]-> origin/prod）
    - 本番環境の異常を確認し、コミットをリバート（prod -[revert]-> prod -[push]-> origin/prod, prod -[merge]→ develop -[push]-> origin/develop）
7. **モニタリング／ログ** – Management Console の リポジトリ ビューと Git ログを突き合わせ、想定外の差分が無いか日次でチェック。

  
<br>

## 5. 技術的背景（なぜこう動くのか）

- **Git によるオブジェクト管理**
    
    - Management Console 内部の XML/JSON/Binary オブジェクトをファイル化してコミットするため、Git の差分・マージ機能をそのまま活用できます。([ロボット ライフサイクル マネジメントを使用した基本的な設定](https://docshield.tungstenautomation.com/RPA/ja_JA/11.5.0-nlfihq5gwr/help/rpa_help/All_Shared/c_gettingstarted.html))
        
- **Synchronizer の動作**
    
    - `--interval` で指定した秒数ごとに Management Console API を呼び出し、リポジトリとオブジェクト SHA を比較して差分をプッシュ/プルします。([同期の開始](https://docshield.tungstenautomation.com/RPA/ja_JA/11.5.0-nlfihq5gwr/help/rpa_help/All_Shared/c_startsynch.html))
        
- **OAuth 認証と SSH 鍵**
    
    - Management Console とは OAuth、リポジトリとは（Git連携ツールを使用する場合） SSH または HTTPS 認証を併用し、パイプラインを完全自動化しても資格情報を安全に保持可能です。([同期の開始](https://docshield.tungstenautomation.com/RPA/ja_JA/11.5.0-nlfihq5gwr/help/rpa_help/All_Shared/c_startsynch.html))
        
- **推奨ブランチ戦略**
    
    - prod を必ず本番に対応させ、`--no-ff` マージで履歴を残すことで「いつ、どのコミットが本番に上がったか」を Git 単体で追跡できるようになります。([ブランチ選択の方針](https://docshield.tungstenautomation.com/RPA/ja_JA/11.5.0-nlfihq5gwr/help/rpa_help/All_Shared/c_select_branchingstrategy.html))
        

<br>

## 6. BizRobo!製品ラインナップ別 利用の動機付け

### BizRobo! Basic利用者

- 開発用 Management Console から本番用 Management Console へのオブジェクト移行のチェックシートを用いた一連の作業がいくつかの単純なコマンド操作で自動化できる。（そしてその変更の記録をすべて履歴として記録できる）

- Management Console を開発用・本番用で分離して運用している場合[^3]、開発用環境 → 本番用環境への連携について内部統制上の業務分掌が確保できる。（本番環境のプロジェクトを`読み取り専用` に設定することで、RLM以外からの Management Console へのオブジェクト登録を禁止できるため、Gitを用いて一貫したアクセスコントロールとトレーサビリティが確保可能）

- 誤ったオブジェクトの登録（間違ってスケジュールを消してしまった、不具合のあるロボットを承認してしまったなど）が判明した場合、Gitを通じてRevert命令を実行することで、即座にもれなく指定した時点の状態（リビジョン）に本番環境の状態を切り戻すことができる。

	**Git連携ツールを導入により得られるベネフィット**
	- 開発担当者から運用管理者へプルリクエストを送信することにより、運用管理者のレビュー・承認を得た段階で自動的にオブジェクトが本番環境にリリースされる。
	- 本番環境リリースに伴う CI/CDパイプラインに様々な処理を組み込んでおくことで、人が見落としそうなチェック作業[^4]や、本番環境専用のデータ変換[^5] するといった自動化処理を組み込み可能であり、管理の工数削減と品質向上を期待できる。




### BizRobo! Lite利用者

- サービスが前提とする利用規模においては、RLMで得られるメリットは少ない。
- 複数 Management Console を運用することがないため、開発用環境 → 本番用環境への連携についても発生しない。
- Management Console 端末に `Bare Git Repository` と clone したローカルリポジトリを用意し、SourceTree などのGitクライアントを用いてBizRobo! の資材管理を行うことができる。 

	**考えられるベネフィット**
	- Management Console間の連携は発生しないものの、Design Studioからアップロードされたロボットファイル等のバージョン管理やスケジュール設定変更の履歴管理などをGitのヒストリーとして管理することができる。また、Git のヒストリー情報をもとに、特定のリビジョンに安全に切り戻すことができる。 
  

### BizRobo! mini利用者

- RLMは利用できません。

<br>

[^1]: BizRobo! Basic ユーザーで、大規模に運用している場合には Git連携ツールを用いてロボットのリリース管理を行うことは非常に有効だと思います。またそのような場合には、`prod=本番用`, `develop=開発用` だけでなく、テスト環境用のブランチを追加することでより厳密なバージョン管理を行うことが可能です。([ブランチ選択の方針](https://docshield.tungstenautomation.com/RPA/ja_JA/11.5.0-nlfihq5gwr/help/rpa_help/All_Shared/c_select_branchingstrategy.html))

[^2]: パラメータの値は BizRobo! のバージョンごとに違うため、対象バージョンの公式ドキュメントを確認してください。

[^3]: Management Console自体は1台であっても、クラスターにより論理的に分離している場合も含みます。

[^4]: ロボットファイルのバージョンをチェックし、現在のBizRobo!バージョンより一定数古い場合にはアラートを発生させるなど

[^5]: ロボットが参照しているURLに開発環境用のURLが残っている場合には、URLを変更して担当者に通知したうえで、本番環境へ登録するなど


