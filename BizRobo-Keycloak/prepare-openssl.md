# SSL証明書の準備

Keycloak において TLS 通信を有効化するために SSL 証明書（`.crt`）及び秘密鍵（`.key`）をご用意ください。

---

事前に証明書をお持ちでない場合は以下の手順に沿って生成してください。ただし、これらのファイルは Keycloak を構築するサーバ上で生成する **必要はありません** 。

以降、下記バージョンの OpenSSL を利用して自己証明書を生成します。

**OpenSSLのバージョン確認**
```powershell
openssl version
# OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
```

> [!WARNING] 本番環境構築時の注意
>
 >   Keycloak を本番環境にて利用際には自己証明書ではなく認証局（CA）により発行されたサーバー証明書を利用することを推奨します。

## OpenSSL の有効化

作業者の Windows 10/11 PC で作業することを前提に、WSL 上の OpenSSL を利用します。

> [!NOTE] OpenSSL の準備について
>
 >   本手順書で利用するOpenSSLのバージョンについては前述のとおりですが、Keycloakの環境構築においては特段バージョンや実装の定めはありません。
>
 >   既に秘密鍵、サーバ証明書を生成する環境をお持ちの場合、ソフトウェアにこだわりのある場合にはそちらをご利用いただいて問題ありません。

1.  PowerShell を **管理者権限** で起動し、以下のコマンドを実行します。
    ```powershell linenums="1" title="Powershellで実行"
    wsl -l -v
    ```

    以下の様に表示されれば既に WSL はインストール済みです。
    ```powershell linenums="2"
    NAME                   STATE           VERSION
    * Ubuntu                 Running         2
    ```

2.  WSL がインストールされていない場合には、以下のコマンドを実行します。
    ```powershell linenums="1" title="Powershellで実行"
    wsl --install
    ```

    インストール完了後、Windows を再起動します。

3.  Windows 再起動後、WSL が自動起動しユーザー名とパスワードを求められます。 Windows のログインアカウントとは別に WSL のユーザーアカウントを設定します。[^1]

[^1]: 正確には WSL で有効にした Linux ディストリビューションのユーザーアカウントです。

## 自己証明書の作成

以下の手順に従い、OpenSSL を使用して HTTPS 通信に必要なサーバー証明書を作成します。

1.  任意の方法で WSL を起動し、コンソールから以降の操作を行います。
2.  秘密鍵を生成します。
    ```wsl linenums="1" title="wslで実行"
    openssl genpkey -out secret.key \
                    -algorithm RSA \
                    -pkeyopt rsa_keygen_bits:2048
    ```

* **genpkey**: 秘密鍵を生成するためのコマンドです。
	* **-out secret.key**: 生成された秘密鍵を secret.key という名前のファイルに保存するよう指定しています。>
* **-algorithm RSA**: RSA アルゴリズムを使用して鍵を生成することを指定しています。
* **-pkeyopt rsa_keygen_bits:2048**: RSA 鍵の長さを 2048 ビットに設定しています。

3.  秘密鍵ファイルからサーバー鍵ファイルを作成します。
    ```wsl linenums="1" title="wslで実行"
    openssl pkey -in secret.key -out server.key -traditional
    ```

- **pkey**: 秘密鍵を操作するためのコマンドです。
- **-in secret.key**: secret.key を入力ファイルに指定します。
- **-out server.key**: server.key を出力ファイルに指定します。
- **-traditional**: 秘密鍵を古いバージョンのopensslと互換性が確保された形式でエクスポートします。

4.  新しい SSL 証明書署名リクエスト（CSR）を作成します。

    ```wsl linenums="1" title="wslで実行"
    openssl req -new -key server.key > server.csr
    ```

- **-new**: 新しい CSR を作成することを指示します。
- **-key server.key**: 作成した秘密鍵（server.key）を使用して、CSR を作成します。
- **> server.csr**: CSR を server.csr という名前でファイルに出力します。

5.  CSR から自己署名証明書（.crt）を生成します
    ```wsl linenums="1" title="wslで実行"
    openssl x509 -req -in server.csr -signkey server.key \
                      -out server.crt -days 3650
    ```

- **-req**: 証明書署名要求を使用することを指定します。
- **-in server.csr**: 署名するためのCSRファイルを指定します。
- **-signkey server.key**: 秘密鍵（.key）を指定して署名します。
- **-out server.crt**: 署名された証明書ファイルの名前とパスを指定します。
- **-days 3650**: 証明書が有効である期間を日単位で指定します（ここでは10年間）。

 6.  以下のファイルがカレントディレクトリに出力されていることを確認します。

```bash
bizrobo@HOST-NAME:/mnt/c/Users/xxx/Documents/Keycloak$ ls -l
total 4
-rwxrwxrwx 1 bizrobo bizrobo 1704 Apr 23 23:51 secret.key
-rwxrwxrwx 1 bizrobo bizrobo 1399 Apr 23 23:53 server.crt
-rwxrwxrwx 1 bizrobo bizrobo 1127 Apr 23 23:53 server.csr
-rwxrwxrwx 1 bizrobo bizrobo 1679 Apr 23 23:51 server.key
```
