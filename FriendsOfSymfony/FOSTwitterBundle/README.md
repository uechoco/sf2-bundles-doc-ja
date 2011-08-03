TODO
====
    1. テストの更新

インストール
============

  1. このバンドルとアブラハム・ウィリアムの Twitter ライブラリを Git サブモジュールとしてプロジェクトに追加 

          $ git submodule add git://github.com/FriendsOfSymfony/FOSTwitterBundle.git vendor/bundles/FOS/TwitterBundle
          $ git submodule add git://github.com/kertz/twitteroauth.git vendor/twitteroauth

> **Note:** kertz/twitteroauth は FOSTwitterBundle に対応するためにパッチが充てられています。

  2. プロジェクトのオートローダー bootstrap スクリプトに、 `FOS` 名前空間を登録

          //app/autoload.php
          $loader->registerNamespaces(array(
                // ...
                'FOS'    => __DIR__.'/../vendor/bundles',
                // ...
          ));

  3. アプリケーションのカーネルにこのバンドルを追加 

          //app/AppKernel.php
          public function registerBundles()
          {
              return array(
                  // ...
                  new FOS\TwitterBundle\FOSTwitterBundle(),
                  // ...
              );
          }

  4. YAML の設定ファイルに、 `twitter` サービスを設定 

            #app/config/config.yml
            fos_twitter:
                file: %kernel.root_dir%/../vendor/twitteroauth/twitteroauth/twitteroauth.php
                consumer_key: xxxxxx
                consumer_secret: xxxxxx
                callback_url: http://www.example.com/login_check

  5. security コンポーネントを使うために、次のような設定を追加:

            #app/config/config.yml
            security:
                factories:
                  - "%kernel.root_dir%/../vendor/bundles/FOS/TwitterBundle/Resources/config/security_factories.xml"
                providers:
                    fos_twitter:
                        id: fos_twitter.auth
                firewalls:
                    secured:
                        pattern:   /secured/.*
                        fos_twitter: true
                    public:
                        pattern:   /.*
                        anonymous: true
                        fos_twitter: true
                        logout: true
                access_control:
                    - { path: /.*, role: [IS_AUTHENTICATED_ANONYMOUSLY] }

Twitter @Anywhere を使う
------------------------

> **Note:** Security コンポーネントを Twitter @Anywhere と連携させるためには、成功したクライアント認証の上に設定済みの確認パスに対して、リクエストを送る必要があります (設定例は https://gist.github.com/1021384 を参照)。

Twitter @Anywhere を使うためのテンプレートヘルパーが含まれています。
このヘルパーを使うためには、まずは DOM の先頭の方で `->setup()` メソッドを呼びます。

        <!-- php テンプレートの内部 -->
          <?php echo $view['twitter_anywhere']->setup() ?>
        </head>

        <!-- twig テンプレートの内部 -->
          {{ twitter_anywhere_setup() }}
        </head>

一度その呼出しが済んだら、 JavaScript のキューを作ることができます。
キューになっているのは、ライブラリが実際に読み込まれるときに一度だけ実行されるようにするためです。

        <!-- php テンプレートの中身 -->
        <span id="twitter_connect"></span>
        <?php $view['twitter_anywhere']->setConfig('callbackURL', 'http://www.example.com/login_check') ?>
        <?php $view['twitter_anywhere']->queue('T("#twitter_connect").connectButton()') ?>

        <!-- twig テンプレートの中身 -->
        <span id="twitter_connect"></span>
        {{ twitter_anywhere_setConfig('callbackURL', 'http://www.example.com/login_check') }}
        {{ twitter_anywhere_queue('T("#twitter_connect").connectButton()') }}

最後に、 DOM の最後の方で `->initialize()` メソッドを呼び出します。

        <!-- inside a php template -->
          <?php $view['twitter_anywhere']->initialize() ?>
        </body>

        <!-- inside a twig template -->
        {{ twitter_anywhere_initialize() }}
        </body>

### Twitter @Anywhere の設定

テンプレートヘルパーを使って設定することができます。setConfig() メソッドを使います。

## 高度な使い方

FOSFacebookBundle のカスタムユーザープロバイダの作り方に関するドキュメントを御覧ください。

> **TIP** Translation Info: 2011/08/03 uechoco 32429e0c
