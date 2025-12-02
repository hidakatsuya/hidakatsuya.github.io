---
title: Rails のセキュリティガイドを読んだ
---

改めて Railsガイドの [セキュリティガイド](https://railsguides.jp/security.html) を読んで、気になった点や知らなかった点をまとめた。

## 3. セッション

- `bin/rails secret` で一意な秘密鍵を生成できる
  - Railsアプリケーションでシークレットキーとして使うための、暗号学的に安全なランダム文字列を生成する
- Railsでは暗号化済み cookie と署名済み cookie があり、それらの salt を同一にしてはいけない
  - 暗号化済み: `cookies.encrypted[:foo] = "bar"`
  - 署名済み: `cookies.signed[:foo] = "bar"`
- ついでに、`cookies.permanent[:foo] = "bar"` というのもある。デフォルトで期限が20年になる
- 暗号化/署名済みcookieの設定をローテーションできる（以下詳細）

### 暗号化/署名済みcookieの設定のローテーション

例えば、署名済みcookieのダイジェストをSHA1からSHA256に変更する場合

```ruby
Rails.application.config.action_dispatch.signed_cookie_digest = "SHA256"
```

後は古いSHA1ダイジェストのローテーションを追加すれば、既存のcookieが自動的にSHA256ダイジェストにアップグレードされる

```ruby
Rails.application.config.action_dispatch.cookies_rotations.tap do |cookies|
  cookies.rotate :signed, digest: "SHA1"
end
```

## 4. クロスサイトリクエストフォージェリ（CSRF）

- GET と POST を適切に使うことが対策の一つ。W3C は要求している
- POST は、ブラウザが「安全ではない操作（命令や状態変更）」を伴うリクエストを、外部から簡単にトリガーできないように制限できるため
  - GET で命令を実行していると、`<img>` の `src` にターゲットのアプリケーションに対するURLを埋め込むことができてしまう
  - 例: `<img src="http://www.webapp.com/project/1/destroy">`
- W3C は HTTP の GET と POST を選択する際のチェックリストを提供している
  - GET
    - そのやりとりが基本的に問い合わせである場合（クエリ、読み出し操作、検索などの安全な操作）
  - POST（以下のいずれか）
    - そのやりとりが基本的に命令である場合
    - そのやりとりによってユーザーにわかる形でリソースのステートが変わる場合（サービスへの申し込みなど）
    - そのやりとりによって生じる結果の責任をユーザーが負う場合

## 7. インジェクション

### 許可リスト方式と禁止リスト方式

- 基本的に許可リスト方式を推奨
  - セキュリティに関連するアクションでは、`before_action` に `only: [...]` ではなく `except: [...]` を指定すること。その方が将来コントローラにアクションを追加するときにセキュリティチェックを忘れずに済む
  - クロスサイトスクリプティング（XSS）対策では、`<script>` を削除する禁止リスト方式ではなく、たとえば `<strong>` だけを許可する許可リスト方式を使うこと
  - ユーザー入力データは訂正せず拒否する
    - そうでないと、以下のような攻撃が成立する
    - `"<sc<script>ript>".gsub("<script>", "")`

### クロスサイトスクリプティング（XSS）

* XSS は cookie を取得することが目的の一つである
* Cookie に `HttpOnly` フラグをつけることで、その cookie は JavaScript の `document.cookie` APIにはアクセスできない。これは XSS を緩和する
  * Railsで `cookies[]` で設定するクッキーは、デフォルトで `httponly: true` がセットされる
  * https://api.rubyonrails.org/classes/ActionDispatch/Cookies.html

### CSSインジェクション

* 古いブラウザ（IE6-8など）は CSS インジェクションによって、CSSを通してJSを埋め込むことができた
* `<div style="background:url('javascript:alert(1)')">`

### コマンドインジェクション

* コマンドインジェクションの一つの対策として、`system()` の複数パラメータでの呼び出しを使う対策がある
  * `system()` は引数が一つの文字列の場合は、シェル経由（`/bin/sh -c "..."`）で実行する
  * `system()` は引数が複数の文字列の場合は、シェル経由ではなく、文字列をシェルとして解釈せずに実行する
* `Kernel#open` は、`|` で始まる引数を渡すとOSコマンドを実行できる
  * `puts open('| ls') { |f| f.read }`
  * `File.open`, `IO.open`, `URI#open` を使う

### ヘッダーインジェクション

例えば、以下のように指定のアドレスにリダイレクトする実装があるとする

```ruby
redirect_to params[:referer]
```

悪意のあるユーザーが以下のように `Location` ヘッダーを注入する

```
http://www.yourapplication.com/controller/action?referer=path/at/your/app%0d%0aLocation:+http://www.malicious.tld
```

`%0d%0a` は `\r\n` がURLエンコードされたものなので、以下のように解釈されてリダイレクトしてしまう

```
HTTP/1.1 302 Moved Temporarily
(...)
Location: http://www.malicious.tld
```

* ヘッダーインジェクションの攻撃経路は、ヘッダーへの CRLF 文字のインジェクションに基づいている
* Rails では `redirect_to` メソッドの `Location` フィールドからこれらの文字をエスケープするようになった
* しかし、ユーザー入力を使ってヘッダーを生成する場合は、自前で `CRLF` 文字のエスケープが必要

### DNSリバインディングとHostヘッダー攻撃

* リクエスト中の `Host: evil.example.com` の `Host` を、システム側がそのまま信用して使うことで起こる
* 例えば、ある Rails アプリのリセット機能で、リセット機能の URL を `request.host` を使って構築していると、攻撃者が被害者のメールアドレスを使って `Host: evil.example.com` を含むリクエストをリセット機能に送信すると、被害者には `https://evil.example.com/reset_password?token=token` のような URL を含むメールが送信されてしまう
* `Rails.application.config.hosts << "product.com"` を設定することで、不正な `Host` のリクエストを 403 で拒否することができる
* HTTP/1.1 では `Host` フィールドが必須。HTTP/2,HTTP/3 では必須ではなくなったが、代わりに `:authority` に同等の情報が必要であり、依然として問題が発生する可能がある
* Hostヘッダー攻撃は、他にも Open Redirect 問題も引き起こす

## 8. 安全でないクエリ生成

```ruby
unless params[:token].nil?
  user = User.find_by_token(params[:token])
  user.reset_password!
end
```

* 上記のようなコードでは、`params[:token]` が `[nil]`, `[nil, nil, ...]`, `['foo', nil]` のとき、nil チェックがバイパスされ、SQLに `IS NULL` や `IN ('foo', NULL)` が混入する可能性がある
* これを防ぐために、デフォルトで `config.perform_deep_munge` オプションが有効になっている
* `deep_munge` は次のように、一部の値を `nil` に置き換える

| JSON | パラメータ |
| -- | -- |
| `{ "person": null }` |	`{ :person => nil }` |
| `{ "person": [] }` |	`{ :person => [] }` |
| `{ "person": [null] }` |	`{ :person => [] }` |
| `{ "person": [null, null, ...] }` |	`{ :person => [] }` |
| `{ "person": ["foo", null] }` |	`{ :person => ["foo"] }` |

## 9. HTTPセキュリティヘッダー

* X-Frame-Options:
  * ブラウザがページを `<frame>` `<iframe>` `<embed>` または `<object>` タグでレンダリングしてよいかどうかを示す
  * このヘッダはデフォルトで `SAMEORIGIN` に設定されており、同一ドメイン内でのみフレームのレンダリングを許可
* X-Content-Type-Options:
  * ブラウザがコンテンツのMIMEタイプを推測してレンダリングしないように指示する
  * デフォルトは `nosniff` で、ブラウザがコンテンツのMIMEタイプを推測してレンダリングしないように指示する
* Strict-Transport-Security (HSTS)
  * ブラウザがHTTPSのみで通信するように指示する
  * `config.force_ssl = true` にすると、このヘッダーがレスポンスに追加される
* Permission-Policy
  * 実験的なヘッダーだが、DSLが用意されている
    ```ruby
    Rails.application.config.permissions_policy do |policy|
      policy.camera      :none
      policy.gyroscope   :none
      policy.microphone  :none
      policy.usb         :none
      policy.fullscreen  :self
      policy.payment     :self, "https://secure.example.com"
    end
    ```
