FOSUserBundleを始める
==================================

Symfony2のセキュリティコンポーネントは柔軟なセキュリティフレームワークを提供していて、
設定・データベース・その他思い通りのものからユーザーを読み込むことができます。
FOSUserBundleはこのコンポーネントの上に構築され、
データベース内にユーザー情報を簡単に素早く貯めこむことができます。

システムの中のユーザーをデータベースに永続化し、またデータベースから読み取る必要があるのでしたら、
このドキュメントはちょうど良いでしょう。

## 前提条件

### 翻訳

このバンドルで提供されるデフォルトのテキストを使いたい場合は、
設定の中でトランスレータを有効化しなければなりません。

```
# app/config/config.yml

framework:
    translator: ~
```

翻訳についての詳細は、[Symfony documentation](http://symfony.com/doc/2.0/book/translation.html) をチェックして下さい。

## インストール

インストールは簡単な(保証します!)8つの手順です:

1. FOSUserBundle をダウンロード
2. Autoloader の設定
3. Bundle の有効化
4. User クラスの作成
5. アプリケーションの security.yml の設定
6. FOSUserBundle の設定
7. FOSUserBundle のルーティングをインポート
8. データベーススキーマの更新

### ステップ1: FOSUserBundle のダウンロード

結局のところ、FOSUserBudnle ファイルを
`vendor/bundles/FOS/UserBundle` ディレクトリにダウンロードしなくてはなりません。

これには幾つかの方法がありますが、お好みで選んでください。
最初の方法は、Symfony2の標準的な方法です。

**vendorsスクリプトを使う**

`deps`ファイルに次の行を追加してください:

```
[FOSUserBundle]
    git=git://github.com/FriendsOfSymfony/FOSUserBundle.git
    target=bundles/FOS/UserBundle
```

バンドルをダウンロードするためにvendorsスクリプトを実行しましょう

``` bash
$ php bin/vendors install
```

**submodulesを使う**

代わりにgit submoduleを使うほうが好みであれば、次のように実行してください:

``` bash
$ git submodule add git://github.com/FriendsOfSymfony/FOSUserBundle.git vendor/bundles/FOS/UserBundle
$ git submodule update --init
```

### ステップ2: Autoloaderの設定

`FOS` 名前空間を autoloader に追加します:

``` php
<?php
// app/autoload.php

$loader->registerNamespaces(array(
    // ...
    'FOS' => __DIR__.'/../vendor/bundles',
));
```

### ステップ3: bundleの有効化

最後に、カーネルの中でバンドルを有効化します:

``` php
<?php
// app/AppKernel.php

public function registerBundles()
{
    $bundles = array(
        // ...
        new FOS\UserBundle\FOSUserBundle(),
    );
}
```

### ステップ4: Userクラスの作成

このバンドルの目標は、ある `User` クラスをデータベース(MySql, MongoDB, CouchDBなど)に
永続化させることです。最初の作業は、アプリケーションに`User`クラスを作成することです。
このクラスは思うがままに動作させることができます: 
例えば、あなたが便利だと思うプロパティやメソッドを追加できます。
これはまさしく *あなたの* `User` クラスです。

このクラスには２つだけ必要なことがあります。
これはFOSUserBundleの機能すべてを活用するために必要なことです。

1. バンドル内のベース`User`クラスの１つを拡張しなければなりません
2. `id` フィールドを持たなければなりません

次の節では、Doctrine ORM・MongoDB ODM・CouchDB ODMなどにどのようにユーザー情報を貯めこむかによって
どのように `User` クラスが見えるのかの例をお見せします。

`User` クラスはアプリケーション内のどのバンドルの中に存在していてもかまいません。
例えば、あなたが`Acme`カンパニーで働いているならば、
`AcmeUserBundle`という名のバンドルを作成し、`User` クラスをそこに配置するでしょう。

**注意**

> もし User クラスの __construct()メソッドをオーバーライドしているならば、
> parent::__construct() を呼び出してください。ベースのUserクラスは、
> 幾つかのフィールドの初期化を親のコンストラクタに依存しているためです。

**a) Doctrine ORM User クラス**

Doctrine ORMによってユーザーの永続化をするならば、
`User`クラスはバンドルの`Entity`名前空間に置くべきで、
このように始まるでしょう:

``` php
<?php
// src/Acme/UserBundle/Entity/User.php

namespace Acme\UserBundle\Entity;

use FOS\UserBundle\Entity\User as BaseUser;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="fos_user")
 */
class User extends BaseUser
{
    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    protected $id;

    public function __construct()
    {
        parent::__construct();
        // your own logic
    }
}
```

**Note:**

> `User`はSQLの予約語であるため、テーブル名として使えません。

**b) MongoDB User クラス**

Doctrine MongoDB ODMによってユーザーを永続化する場合、
`User`クラスはバンドルの`Document`名前空間に置くべきで、
このように始まります:

``` php
<?php
// src/Acme/UserBundle/Document/User.php

namespace Acme\UserBundle\Document;

use FOS\UserBundle\Document\User as BaseUser;
use Doctrine\ODM\MongoDB\Mapping\Annotations as MongoDB;

/**
 * @MongoDB\Document
 */
class User extends BaseUser
{
    /**
     * @MongoDB\Id(strategy="auto")
     */
    protected $id;

    public function __construct()
    {
        parent::__construct();
        // your own logic
    }
}
```

**c) CouchDB User クラス**

Doctrine CouchDB ODMによってユーザーを永続化する場合は、
`User`クラスはバンドルの`Document`名前空間に置くべきで、
このように始まります:

``` php
<?php
// src/Acme/UserBundle/Document/User.php

namespace Acme\UserBundle\Document;

use FOS\UserBundle\Document\User as BaseUser;
use Doctrine\ODM\CouchDB\Mapping as CouchDB;

/**
 * @CouchDB\Document
 */
class User extends BaseUser
{
    /**
     * @CouchDB\Id
     */
    protected $id;

    public function __construct()
    {
        parent::__construct();
        // your own logic
    }
}
```

### ステップ5: アプリケーションの security.yml の設定

FOSUserBundleを使うにあたり、Symfony のセキュリティコンポーネントのために、
`security.yml` ファイルの中でそのことを伝えなければなりません。
`security.yml` ファイルは、アプリケーションの基本的なセキュリティ設定が含まれている場所です。

アプリケーションの中でFOSUserBundleを使うために必要な最小設定の例を次に示します:

``` yaml
# app/config/security.yml
security:
    providers:
        fos_userbundle:
            id: fos_user.user_manager

    firewalls:
        main:
            pattern: ^/
            form_login:
                provider: fos_userbundle
            logout:       true
            anonymous:    true

    access_control:
        - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/, role: ROLE_ADMIN }

    role_hierarchy:
        ROLE_ADMIN:       ROLE_USER
        ROLE_SUPER_ADMIN: ROLE_ADMIN
```

`providers`セクションの下に、`fos_userbundle`エイリアスを用いて
バンドルのパッケージされたユーザープロバイダサービスを有効化します。
バンドルのユーザープロバイダサービスのidは`fos_user.user_manager`です。

次に、`firewalls` セクションを見てみましょう。
ここに `main` という名前のファイアウォールを定義しています。
`form_login` を明記することで、
リクエストが認証を必要としているユーザーを導くこのファイアウォールに作られるときはいつでも、
ユーザーは自身の認証情報で入ることの出来るフォームにリダイレクトされる、
ということをSymfony2 に対して伝えることができます。
ファイアウォールを認証プロセスの一部として使うためのプロバイダとして、
早い段階で定義したユーザープロバイダを特定していることは驚くに値しない。
（原文：Next, take a look at examine the `firewalls` section. Here we have declared a
firewall named `main`. By specifying `form_login`, you have told the Symfony2
framework that any time a request is made to this firewall that leads to the
user needing to authenticate himself, the user will be redirected to a form
where he will be able to enter his credentials. It should come as no surprise
then that you have specified the user provider we declared earlier as the
provider for the firewall to use as part of the authentication process.）

**Note:**

> この例ではフォームのログイン機構を使っているにもかかわらず、
> FOSUserBundleのユーザープロバイダは他の多くの認証メソッドと互換性があります。
> 他の認証メソッドについて詳しい情報は、Symfony2のセキュリティコンポーネントのドキュメントを読んでください。

`access_control`セクションは、
アプリケーションの特定の部分にアクセスしようとしているユーザーにどんな認証情報が必要かを明示する場所です。
ログインフォームやユーザーの作成やパスワードのリセットのためのすべてのルートで認証されていないユーザーでも
利用可能でああるが、バンドルでセキュアにしたいページと同じファイアウォールを使用することを、バンドルは必要とします。
これが、`/login`というパターンにマッチするリクエストや`/register`や`/resetting`で始まるリクエストが
不特定ユーザーでも使用可能であることを明示している理由です。
また、`/admin` で始まるどんなリクエストも `ROLE_ADMIN` ロール(role) を持つユーザーを必要とすることも明示しています。
（原文：The `access_control` section is where you specify the credentials necessary for
users trying to access specific parts of your application. The bundle requires
that the login form and all the routes used to create a user and reset the password
be available to unauthenticated users but use the same firewall as
the pages you want to secure with the bundle. This is why you have specified that
the any request matching the `/login` pattern or starting with `/register` or
`/resetting` have been made available to anonymous users. You have also specified
that any request beginning with `/admin` will require a user to have the
`ROLE_ADMIN` role.）

`security.yml`の設定の詳しい情報は、 Symfony2 のセキュリティコンポーネントの[ドキュメント](http://symfony.com/doc/current/book/security.html)
を参照してください。

**Note:**

> `main` という名前によく注目してください。FOSUserBudnleが設定されているファイアウォールに与えています。
> 次のステップでFOSUserBundleの設定を刷るときにこれを使うでしょう。

### ステップ6: FOSUserBundle の設定

今やアプリケーションの `security.yml` をFOSUserBundleが動くようにきっちりと設定しているので、
次のステップでは、アプリケーションの特定のニーズと連携するためのバンドルの設定をします。

次の設定を 使用しているデータストアの形式に一致する`config.yml` ファイルに追加してください。

``` yaml
# app/config/config.yml
fos_user:
    db_driver: orm # other valid values are 'mongodb', 'couchdb'
    firewall_name: main
    user_class: Acme\UserBundle\Entity\User
```

あるいは、XMLが好みであれば:

``` xml
# app/config/config.xml
<!-- app/config/config.xml -->

<!-- other valid 'db-driver' values are 'mongodb' and 'couchdb' -->
<fos_user:config
    db-driver="orm"
    firewall-name="main"
    user-class="Acme\UserBundle\Entity\User"
/>
```

バンドルを使うのに必要な設定値はたったの3つです:

* 使用しているデータストアの種類 (`orm`, `mongodb`, or `couchdb`)
* ステップ5で設定したファイアウォールの名前
* ステップ2で作成した `User` クラスの完全限定クラス名(FQCN: fully qualified class name) 

### ステップ7: FOSUserBundle のルーティングをインポート

バンドルを有効化して設定しましたので、
残ったやるべき事はFOSUserBundleのルーティングファイルをインポートすることだけです。

ルーティングファイルをインポートすることで、
ログインやユーザー作成などのページを作る準備をすることになります。

YAMLで:

``` yaml
# app/config/routing.yml
fos_user_security:
    resource: "@FOSUserBundle/Resources/config/routing/security.xml"

fos_user_profile:
    resource: "@FOSUserBundle/Resources/config/routing/profile.xml"
    prefix: /profile

fos_user_register:
    resource: "@FOSUserBundle/Resources/config/routing/registration.xml"
    prefix: /register

fos_user_resetting:
    resource: "@FOSUserBundle/Resources/config/routing/resetting.xml"
    prefix: /resetting

fos_user_change_password:
    resource: "@FOSUserBundle/Resources/config/routing/change_password.xml"
    prefix: /change-password
```

XMLがおこのみであれば:

``` xml
<!-- app/config/routing.xml -->
<import resource="@FOSUserBundle/Resources/config/routing/security.xml"/>
<import resource="@FOSUserBundle/Resources/config/routing/profile.xml" prefix="/profile" />
<import resource="@FOSUserBundle/Resources/config/routing/registration.xml" prefix="/register" />
<import resource="@FOSUserBundle/Resources/config/routing/resetting.xml" prefix="/resetting" />
<import resource="@FOSUserBundle/Resources/config/routing/change_password.xml" prefix="/change-password" />
```

**Note:**

> 組み込みのemail昨日(アカウントの確認やパスワードのリセットアカウントの確認やパスワードのリセット)を使うため、
> SwiftmailerBundleを有効化し、設定しておく必要があります。

### ステップ8: データベーススキーマの更新

バンドルが設定されたので、最後にデータベーススキーマを更新します。
これはステップ2で作成した`User`クラスの新しいエンティティを追加するためです。

ORMに対して、次のコマンドを実行します。

``` bash
$ php app/console doctrine:schema:update --force
```

MongoDBユーザーであれば、インデックスを作成するために次のコマンドを実行できます。

``` bash
$ php app/console doctrine:mongodb:schema:create --index
```

これで `http://app.com/app_dev.php/login` にアクセスするとログインが出来ます！

### 次のステップ

FOSUserBundleの基本的なインストールと設定が完了したので、
バンドルの高度な機能や使い方について学ぶ準備が整いました。

また次のドキュメントも利用できます。

1. [Overriding Templates](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/overriding_templates.md)
2. [Overriding Controllers](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/overriding_controllers.md)
3. [Command Line Tools](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/command_line_tools.md)
4. [Command Line Tools](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/command_line_tools.md)
5. [Supplemental Documenation](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/supplemental.md)
6. [Configuration Reference](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/configuration_reference.md)

> **TIP** Translation Info: 2011/09/19 uechoco 1eea035
