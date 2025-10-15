1. はじめに
2. [SAMLによるSSO](summary-for-sso.md)
3. [構築準備](summary-for-setup)
4. 環境構築
	1. [OpenJDK の準備](prepare-jdk.md)
	2. [SSL証明書の準備](prepare-openssl.md)
	3. [データベースの準備](create-db)
	4. Keycloakの環境構築
	5. Keycloakの初期設定
	6. レルムの作成
5. MCの設定
6. Keycloakの設定
7. SAML連携確認
8. Appendix

## 凡例等・ドキュメントの見方

設定ファイルの変更前後の値を表示する例

```diff linenums="98"
-     修正前の値
+     修正後の値
```

任意の値、変数を意味する場合の例

```
http://{任意の値、変数}/mc
```

## Keycloak とは

![logo](image/home/keycloak_logo_200px.svg)

Keycloak とは、オープンソースのアイデンティティ・アクセス・マネジメント（IAM）ソフトウェアです。

Keycloak は、Web アプリケーションやマイクロサービスに対して、ユーザーの認証や認可、シングルサインオン（SSO）機能を提供します。Keycloak は、OAuth 2.0、OpenID Connect、SAML、Kerberos など、多数の認証プロトコルに対応しています。

## 概要

本資料は ManagementConsole と同一の環境に Keycloak を構築し、SAML 認証を行うための設定手順書です。  
最低限の動作を確認するための設定であるため、実際に運用環境として構築する場合には下記に挙げた情報も参考にしてください。
Keycloak の version は 20.0.3 を利用します。画面の更新が頻繁にあるようなので、実際に環境構築を行う際には適宜読み替が必要となります。

Keycloak は自己署名を利用した TLS で構築しているため、アクセス時にアラートが出ます。

## SAML 認証について

- 参考資料
  - https://www.splashtop.co.jp/knowhow/26/

- Keycloak 公式ドキュメント
  - https://www.keycloak.org/documentation
    - 最新版（英語のみ）
  - https://keycloak-documentation.openstandia.jp/
    - 日本語ドキュメント（旧 ver のみ）
