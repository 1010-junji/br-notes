# データベースの準備

Keycloak の情報を保存するデータベース領域を作成します。

本手順書においては MySQL8 を使用してデータベース領域を作成します。データベース自体のインストール、セットアップは済ませたうえで以降の手順に進んでください。

## Keycloak 用データベースの作成

MySQL client に root ユーザでログインし、以下の手順を実行します。

1.  Keycloak 用のデータベースを作成します。

**MySQL clientで実行:**
```sql
CREATE DATABASE keycloak CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
```

| 項目           | 値       |
| -------------- | -------- |
| データベース名 | keycloak |

2.  作成したデータベースを確認します。

**MySQL clientで実行:**
```sql
SHOW DATABASES;
```

以下の様に `keycloak` データベースが表示されていれば OK です。

```sql
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| keycloak           |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

## データベースアクセスアカウントの作成

> [!WARNING] パスワードの設定について
>
 >   本番環境を構築する際には、十分な強度の文字列をパスワードとして設定し、厳密に保管してください。

1.  Keycloak 用のデータベースにアクセスするためのアカウントを作成します。

**MySQL clientで実行:**
```sql
CREATE USER 'keycloak'@'localhost' IDENTIFIED BY 'password';
```

| 項目       | 値       | 備考                               |
| ---------- | -------- | ---------------------------------- |
| ユーザ名   | keycloak |                                    |
| パスワード | password | 本番環境構築時には変更してください |


2.  作成したアカウントを確認します。

**MySQL clientで実行:**
```sql
SELECT USER, HOST FROM mysql.user;
```

以下の様に `keycloak` ユーザが表示されていれば OK です。

```sql
mysql> SELECT USER, HOST FROM mysql.user;
+------------------+-----------+
| USER             | HOST      |
+------------------+-----------+
| keycloak         | localhost |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
5 rows in set (0.00 sec)
```

## アカウントへの権限付与

1.  作成した Keycloak アクセス用アカウントに権限を付与します。

**MySQL clientで実行:**
```sql
GRANT ALL PRIVILEGES ON keycloak.* TO 'keycloak'@'localhost';
```

> [!NOTE] 接続ホストの指定について
>
>本手順書では Keycloak 本体と DB は同一サーバ上に構築する前提で解説しているため、`@localhost` を指定していますが、Keycloak と DB のサーバを別々で用意する際には `@IPアドレス/ホスト名`で指定ください。
>
> また、 `@'%'` と指定することにより Keycloak はどのホストからでも接続を許可されます。ただしこの指定はセキュリティ上の問題を引き起こす可能性があるため、推奨されません。[^1]

[^1]: アプリケーションサーバを冗長化しており複数のサーバーから単一のDBに接続する際には、`@'198.51.100.0/255.255.255.0'` の様にネットマスクを使い範囲指定をしたり、接続元ホストのIP/ホスト名を変えて複数回同一ユーザの権限を登録することで対応します。


2.  作成したアカウントを確認します。

**MySQL clientで実行:**
```sql
SHOW GRANTS FOR keycloak@localhost;
```

以下の様に付与した権限が表示されていれば OK です。

```sql
mysql> SHOW GRANTS FOR keycloak@localhost;
+----------------------------------------------------------------+
| Grants for keycloak@localhost                                  |
+----------------------------------------------------------------+
| GRANT USAGE ON *.* TO `keycloak`@`localhost`                   |
| GRANT ALL PRIVILEGES ON `keycloak`.* TO `keycloak`@`localhost` |
+----------------------------------------------------------------+
2 rows in set (0.00 sec)
```

## Keycloak 用データベースの削除

環境の初期化が必要な場合には、以下の手順で Keycloak のデータベースおよびユーザーアカウント・権限を削除します。

**MySQL clientで実行:**
```sql
DROP USER keycloak@localhost;
DROP DATABASE keycloak;
```

- ユーザーアカウントを削除することにより、権限も一緒に削除されます。
