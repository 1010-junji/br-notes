1. はじめに
2. [SAMLによるSSO](summary-for-sso.md)
3. 構築準備
4. 環境構築
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

ManagementConsole と同一の環境に Keycloak を構築し、SAML 認証を行うための設定手順です。  
最低限動作での設定手順のため、情報に不足があることはご了承ください。  
Keycloak の version は 20.0.3 を利用します。画面の update が頻繁にあるようなのでこの version 以外は非対応です。

Keycloak は自己署名を利用した TLS で構築しているため、アクセス時にアラートが出ます。

## SAML 認証について

- 参考資料

  - https://www.splashtop.co.jp/knowhow/26/

- Keycloak 公式ドキュメント

  - https://www.keycloak.org/documentation

    - 最新版（英語のみ）

  - https://keycloak-documentation.openstandia.jp/
    - 日本語ドキュメント（旧 ver のみ）
