FOSUserBundle のコマンドラインツール
================================

FOSUserBundle はアプリケーションユーザーの管理を手助けするためのたくさんのコマンドラインツールを提供しています。
利用可能なコマンドのタスクは以下のとおりです:

1. ユーザーの作成
2. ユーザーのアクチベーション
3. ユーザーのデアクチベーション
4. ユーザーの昇格
5. ユーザーの降格
6. ユーザーパスワードの変更

**Note:**

```
これらのコマンドを使用する前に、FOSUserBundle を正しくインストールして設定して置かなければなりません。
```

### ユーザーの作成

アプリケーションに新しいユーザーを作成するために `fos:user:create` コマンドが使えます。
このコマンドは3つの引数、作成するユーザーの `username`、`email`、`password` を取ります。

例として、username が `testuser`、email が `test@example.com`、password が `p@assword` というユーザーを作成したいときは、
以下のようにコマンドを実行します。

``` bash
$ php app/console fos:user:create testuser test@example.com p@ssword
```

もし必要な引数のいくつかがコマンドに渡らなかった場合は、対話プロンプトが入力を促します。
例えば、以下のようなコマンドを走らせた場合は、作成したいユーザーの `email` と `password` の入力を求められます。

``` bash
$ php app/console fos:user:create testuser
```

また、このコマンドは2つのオプションを受け付けます。
`--super-admin` と `--inactive` です。

`--super-admin` オプションを明示すると、ユーザー作成時にスーパーアドミンとしてのフラグが立ちます。
スーパーアドミンはアプリケーションのあらゆる部分にアクセスすることができます。
以下がその例です:

``` bash
$ php app/console fos:user:create adminuser --super-admin
```

`--inactive` オプションを明示すると、アクチベーションしない限りログイン出来ないユーザーを作成することができます。

``` bash
$ php app/console fos:user:create testuser --inactive
```

### ユーザーのアクチベーション

`fos:user:activate` コマンドは非アクティブユーザーをアクチベートします。
アクティベーションしたいユーザーの `username` だけがコマンドに必要な引数です。
`username` が指示されなかった場合は、対話プロンプトが入力を促します。
コマンドの使い方をいかに示します:

``` bash
$ php app/console fos:user:activate testuser
```

### ユーザーのデアクチベーション

`fos:user:deactivate` コマンドは、ユーザーをデアクチベートします。
アクチベーションコマンドと同様に、デアクチベートしたいユーザーの `username` だけを引数にとります。
`username` が明示されなかったときは、対話プロンプトが入力を促します。
以下にコマンドの使い方を示します:

``` bash
$ php app/console fos:user:deactivate testuser
```

### ユーザーの昇格

`fos:user:promote` コマンドは、ユーザーにロールを追加したり、ユーザーをスーパーアドミンにしたりできます。

ユーザーにロールを追加したい場合は、第１引数としてユーザーの `username` を渡し、
第２引数としてユーザーに追加したい `role` を指定します。

``` bash
$ php app/console fos:user:promote testuser ROLE_ADMIN
```

`username` の後ろに `--super` オプションを与えると、ユーザーをスーパーアドミンに昇格させることができます。

``` bash
$ php app/console fos:user:promote testuser --super
```

引数をコマンドに与えなかった場合は、対話プロンプトが入力を促します。

**Note:** `role` 引数と同時に `--super` オプションを指定することはできません。 

### ユーザーの降格

`fos:user:demote` コマンドは、昇格コマンドと似ていますが、ユーザーにロールを追加する代わりにロールを削除します。
また、ユーザーのスーパーアドミン状態を取りやめることも出来ます。

ユーザーからロールを削除したい場合は、第１引数に `username` を指定し、第２引数に削除したい `role` を指定します。

``` bash
$ php app/console fos:user:demote testuser ROLE_ADMIN
```

ユーザーのスーパーアドミン状態を取りやめるには、 `username` を指定すると共に `--super` オプションを指定します。

``` bash
$ php app/console fos:user:demote testuser --super
```

引数が指定されなかった場合は、対話プロンプトが入力を促します。

**Note:** `role` 引数と同時に `--super` オプションを指定することはできません。 

### ユーザーパスワードの変更

`fos:user:change-password` コマンドは、ユーザーのパスワードを簡単に変更する方法を提供しています。
コマンドは、パスワードを変更したいユーザーの `username` と新しい `password` の2つの引数を取ります。

``` bash
$ php app/console fos:user:change-password testuser newp@ssword
```

`password` 引数を指定しない場合は、対話プロンプトが入力を促します。

> **TIP** Translated Info: 2011/10/16 uechoco a1f17666	

