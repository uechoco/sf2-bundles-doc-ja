前書き
============

このバンドryはTwitter PHPを統合することができます。
また、Symfony2の認証プロバイダも提供しているので、
ユーザーがTwitter経由でSymfony2アプリケーションにログインすることができます。
さらに、Twitterログインをサポートするカスタムユーザープロバイダによって、
他のデータソースと統合することもできます。
例えば、FOSUserBundleによって提供されるデータベースベースのソリューションのような
データソースです。

[![Build Status](https://secure.travis-ci.org/FriendsOfSymfony/FOSTwitterBundle.png)](http://travis-ci.org/FriendsOfSymfony/FOSTwitterBundle)
 
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

FOSUserBundleを使ったカスタムユーザープロバイダの例
-------------------------------------------------------

このプロバイダを使うために、config.ymlに新しいサービスを追加する必要があります。


``` yaml
# app/config/config.yml

        my.twitter.user:
            class: Acme\YourBundle\Security\User\Provider\TwitterProvider
            arguments:
                twitter_oauth: "@fos_twitter.api"
                userManager: "@fos_user.user_manager"
                validator: "@validator"
                session: "@session"
```

また、Userモデルクラスに新しいプロパティとメソッドをいくつか必要です。

``` php
<?php
// src/Acme/YourBundle/Entity/User.php

        /** 
         * @var string
         */
        protected $twitterID;

        /** 
         * @var string
         */
        protected $twitter_username;


        /**
         * Set twitterID
         *
         * @param string $twitterID
         */
        public function setTwitterID($twitterID)
        {
            $this->twitterID = $twitterID;
            $this->setUsername($twitterID);
            $this->salt = '';
        }

        /**
         * Get twitterID
         *
         * @return string 
         */
        public function getTwitterID()
        {
            return $this->twitterID;
        }

        /**
         * Set twitter_username
         *
         * @param string $twitterUsername
         */
        public function setTwitterUsername($twitterUsername)
        {
            $this->twitter_username = $twitterUsername;
        }

        /**
         * Get twitter_username
         *
         * @return string 
         */
        public function getTwitterUsername()
        {
            return $this->twitter_username;
        }

        
このTwitterProviderクラスを追加します。

``` php
<?php
// src/Acme/YourBundle/Security/User/Provider/TwitterProvider.php

namespace Acme\YourBundle\Security\User\Provider;

use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
use Symfony\Component\Security\Core\Exception\UnsupportedUserException;
use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\HttpFoundation\Session;
use \TwitterOAuth;
use FOS\UserBundle\Entity\UserManager;
use Symfony\Component\Validator\Validator;

class TwitterProvider implements UserProviderInterface
{
    /** 
     * @var \Twitter
     */
    protected $twitter_oauth;
    protected $userManager;
    protected $validator;
    protected $session;

    public function __construct(TwitterOAuth $twitter_oauth, UserManager $userManager,Validator $validator, Session $session)
    {   
        $this->twitter_oauth = $twitter_oauth;
        $this->userManager = $userManager;
        $this->validator = $validator;
        $this->session = $session;
    }   

    public function supportsClass($class)
    {   
        return $this->userManager->supportsClass($class);
    }   

    public function findUserByTwitterId($twitterID)
    {   
        return $this->userManager->findUserBy(array('twitterID' => $twitterID));
    }   

    public function loadUserByUsername($username)
    {
        $user = $this->findUserByTwitterId($username);


         $this->twitter_oauth->setOAuthToken( $this->session->get('access_token') , $this->session->get('access_token_secret'));

        try {
             $info = $this->twitter_oauth->get('account/verify_credentials');
        } catch (Exception $e) {
             $info = null;
        }

        if (!empty($info)) {
            if (empty($user)) {
                $user = $this->userManager->createUser();
                $user->setEnabled(true);
                $user->setPassword('');
                $user->setAlgorithm('');
            }

            $username = $info->screen_name;


            $user->setTwitterID($info->id);
            $user->setTwitterUsername($username);
            $user->setEmail('');
            $user->setFirstname($info->name);

            $this->userManager->updateUser($user);
        }

        if (empty($user)) {
            throw new UsernameNotFoundException('The user is not authenticated on twitter');
        }

        return $user;

    }

    public function refreshUser(UserInterface $user)
    {
        if (!$this->supportsClass(get_class($user)) || !$user->getTwitterID()) {
            throw new UnsupportedUserException(sprintf('Instances of "%s" are not supported.', get_class($user)));
        }

        return $this->loadUserByUsername($user->getTwitterID());
    }
}
```


最後に、Twitterから認証トークンを得るために、このようなアクティンをコントローラに追加する必要があります。

``` php

<?php
// src/Acme/YourBundle/Controller/DefaultController.php

        /** 
        * @Route("/connectTwitter", name="connect_twitter")
        *
        */
        public function connectTwitterAction()
        {   

          $request = $this->get('request');
          $twitter = $this->get('fos_twitter.service');

          $authURL = $twitter->getLoginUrl($request);

          $response = new RedirectResponse($authURL);

          return $response;

        }  

```

ユーザーがTwitterに認証を送るために、Twigテンプレートにボタンを作成できます。

```
         <a href="{{ path ('connect_twitter')}}"> <img src="/images/twitterLoginButton.png"></a> 

```

* Note: config.yml の callback URL は check_path として設定されていなければなりません。

``` yaml
# app/config/config.yml

        fos_twitter:
            ...
            callback_url: http://www.yoursite.com/twitter/login_check
```

このプロバイダを使うために、security.ymlを編集することを忘れないで下さい。


``` yaml
# app/config/security.yml

        security:
            factories:
                - "%kernel.root_dir%/../vendor/bundles/FOS/TwitterBundle/Resources/config/security_factories.xml"

            encoders:
                Symfony\Component\Security\Core\User\User: plaintext

            role_hierarchy:
                ROLE_ADMIN:       ROLE_USER
                ROLE_SUPER_ADMIN: [ROLE_USER, ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

            providers:

                my_fos_twitter_provider:
                    id: my.twitter.user 

            firewalls:
                dev:
                    pattern:  ^/(_(profiler|wdt)|css|images|js)/
                    security: false

                public:
                    pattern:  /
                    fos_twitter:
                        login_path: /twitter/login
                        check_path: /twitter/login_check
                        default_target_path: /
                        provider: my_fos_twitter_provider

                    anonymous: ~

```

> **TIP** Translation Info: 2011/12/02 uechoco 155453c937
