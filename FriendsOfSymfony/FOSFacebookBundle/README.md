はじめに
============

このバンドルは、Facebook PHP SDKとJS SDKの統合を可能にします。
さらにSymfony2の認証プロバイダも提供いたします。
これはユーザーがFacebookを通じてSymfony2アプリケーションにログイン出来るようにするためです。
またさらにカスタムユーザープロバイダのサポートによって、
Facebookログインと、FOSUserBundleで提供されるデータベースソリューションのような他のデータソースとの統合も可能です。
例えばFOSUserBundleで提供されるデータベースの

ユーザーのログを取るには、次のような複数のステップが必要です:

  1. ユーザーはFacebookにログインしていなければならない
  2. ユーザーはFacebookのアカウントとあなたのサイトをコネクトしなければならない
  3. 一度ユーザが 1. と 2. を終えたら、ログインを開始しなければならない

ステップ 1. と 2. には、2つのオプションがあります:

  1. Facebookのアプリケーション設定で"OAuth Migration"を選択する
  2. Facebookログインボタンを追加する: このやり方では、ステップ 3. を扱うためにJSコードが必要です
  3. FOSFacebookBundleにFacebookログインページへリダイレクトさせる

最初のファイヤーウォール設定の最初のプロバイダををFOSFacebookBUndleに設定して
ユーザーが認証していない状態で認証が必要なページにアクセスした場合、
さきほどの2つのオプションのうち後者が自動的に発生します。

Facebookのドキュメントも参照してください。
https://developers.facebook.com/docs/guides/web/

またSecurityBundleの公式ドキュメントも参照してください。
特に設定の詳細のところです。
http://symfony.com/doc/2.0/book/security/authentication.html


インストール
==============

  1. このバンドルとFacebook PHP SDKを ``vendor/`` ディレクトリに追加してください:
      * vendorsスクリプトを使う場合

        次の数行を ``deps`` ファイルに追加::

            [FOSFacebookBundle]
                git=git://github.com/FriendsOfSymfony/FOSFacebookBundle.git
                target=/bundles/FOS/FacebookBundle
            
            [FacebookSDK]
                git=git://github.com/facebook/php-sdk.git
                target=/facebook

        vendorsスクリプトを実行:

            ./bin/vendors install

      * git submoduleを使う場合

            $ git submodule add git://github.com/FriendsOfSymfony/FOSFacebookBundle.git vendor/bundles/FOS/FacebookBundle
            $ git submodule add git://github.com/facebook/php-sdk.git vendor/facebook

  2. FOS名前空間をautoloaderに追加:

          // app/autoload.php
          $loader->registerNamespaces(array(
                'FOS' => __DIR__.'/../vendor/bundles',
                // your other namespaces
          ));

  3. このバンドルをアプリケーションのカーネルに追加

          // app/ApplicationKernel.php
          public function registerBundles()
          {
              return array(
                  // ...
                  new FOS\FacebookBundle\FOSFacebookBundle(),
                  // ...
              );
          }
          
  4. 次のルート(route)をアプリケーションに追加し、実際のコントローラのアクションを指定
          
          #application/config/routing.yml
          _security_check:
              pattern:  /login_check
          _security_logout:
              pattern:  /logout

          #application/config/routing.xml
          <route id="_security_check" pattern="/login_check" />
          <route id="_security_logout" pattern="/logout" />     

  5. 設定ファイルの中の`facebook` サービスを設定:

          # application/config/config.yml
          fos_facebook:
              file:   %kernel.root_dir%/../vendor/facebook/src/base_facebook.php
              alias:  facebook
              app_id: 123456879
              secret: s3cr3t
              cookie: true
              permissions: [email, user_birthday, user_location]

          # application/config/config.xml
          <fos_facebook:api
              file="%kernel.root_dir%/../vendor/facebook/src/base_facebook.php"
              alias="facebook"
              app_id="123456879"
              secret="s3cr3t"
              cookie="true"
          >
                <permission>email</permission>
                <permission>user_birthday</permission>
                <permission>user_location</permission>
          </fos_facebook:api>

     設定の中に `file` 値を含めなかった場合、`BaseFacebook`クラスの自動読み込みを
     アプリケーションに設定しなければならないでしょう。

  6. `security component`を使いたい場合はこの設定を追加:

          # application/config/config.yml
          security:
              factories:
                  - "%kernel.root_dir%/../vendor/bundles/FOS/FacebookBundle/Resources/config/security_factories.xml"

              providers:
                  fos_facebook:
                      id: fos_facebook.auth

              firewalls:
                  public:
                      # since anonymous is allowed users will not be forced to login
                      pattern:   ^/.*
                      fos_facebook:
                          app_url: "http://apps.facebook.com/appName/"
                          server_url: "http://localhost/facebookApp/"
                      anonymous: true
                      logout:
                          handlers: ["fos_facebook.logout_handler"]

              access_control:
                  - { path: ^/secured/.*, role: [IS_AUTHENTICATED_FULLY] } # This is the route secured with fos_facebook
                  - { path: ^/.*, role: [IS_AUTHENTICATED_ANONYMOUSLY] }

     これが動作するためには、ルーティングに `/secured/` を追加しなければなりません。例えば...
     
              _facebook_secured:
                  pattern: /secured/
                  defaults: { _controller: AcmeDemoBundle:Welcome:index }

  7. お好みでカスタムユーザープロバイダクラスを定義し、プロバイダとして使うかログインのためのパスとして定義します

          # application/config/config.yml
          security:
              factories:
                    - "%kernel.root_dir%/../vendor/bundles/FOS/FacebookBundle/Resources/config/security_factories.xml"

              providers:
                  # choose the provider name freely
                  my_fos_facebook_provider:
                      id: my.facebook.user   # see "Example Customer User Provider using the FOS\UserBundle" chapter further down

              firewalls:
                  public:
                      pattern:   ^/.*
                      fos_facebook:
                          app_url: "http://apps.facebook.com/appName/"
                          server_url: "http://localhost/facebookApp/"
                          login_path: ^/login
                          check_path: ^/login_check$
                          default_target_path: /
                          provider: my_fos_facebook_provider
                      anonymous: true
                      logout:
                          handlers: ["fos_facebook.logout_handler"]
    
 
          # application/config/config_dev.yml 
          security:
              firewalls:
                  public:
                      fos_facebook:
                          app_url: "http://apps.facebook.com/appName/"
                          server_url: "http://localhost/facebookApp/app_dev.php/" 

  8. お好みでセキュアな特定のURLでアクセスコントロールを使います


          # application/config/config.yml
          security:
              # ...
              
              access_control:
                  - { path: ^/facebook/,           role: [ROLE_FACEBOOK] }
                  - { path: ^/.*,                  role: [IS_AUTHENTICATED_ANONYMOUSLY] }
       
    `ROLE_FACEBOOK` ロール(role)は、ユーザークラスに追加されなければなりません(Acme\MyBundle\Entity\User::setFBData()を参照)
    > アクセスコントロールルールの順番が大事です!

JavsScript SDKのセットアップ
-----------------------------

Facebook JavaScript SDKを読み込み、サービスコンテナから来たパラメータで初期化するための、
テンプレートヘルパーが同梱されています。Facebook JavaScript環境をセットアップするために、
レイアウトの `body` 開始タグの直後に次の行を追加してください:

      <body>
          <!-- inside a php template -->
          <?php echo $view['facebook']->initialize(array('xfbml' => true)) ?>
          <!-- inside a twig template -->
          {{ facebook_initialize({'xfbml': true}) }}

XFBMLマークアップをサイト内に追加したい場合は、
たぶん `html` 開始タグの中に名前空間も宣言したほうが良いでしょう。

      <html xmlns:fb="http://www.facebook.com/2008/fbml">

テンプレートの中にログインボタンを挿入
------------------------------------------

テンプレートの１つに次のコードを追加するだけです:

    <!-- inside a php template -->
    <?php echo $view['facebook']->loginButton(array('autologoutlink' => true)) ?>
    <!-- inside a twig template -->
    {{ facebook_login_button({'autologoutlink': true}) }}

このやり方はログインとFacebookへのコネクトだけが扱われます。
ユーザーがSymfony2アプリケーションにログインするステップには、
発動させる必要もあります。これをするためには、たいていは、
単に "auth.login" イベントに申請し、 "check_path"にリダイレクトさせます:

    <script>
      FB.Event.subscribe('auth.login', function(response) {
        window.location = "{{ path('_security_check') }}";
      });
    </script>

上記の設定にマッチさせるために、
"_security_cehck" ルート(route)は "/login_check" パターンに繋げる必要があります。

また、ログアウトアクションのトリガーを必要とします。
そのためには、 "logout" ルートにリダイレクトするために、
"auth.logout" に予約します。

    <script>
      FB.Event.subscribe('auth.logout', function(response) {
        window.location = "{{ path('_security_logout') }}";
      });
    </script>


FOS\UserBundleを使ったカスタムユーザープロバイダの例
-------------------------------------------------------

次の設定はカスタムユーザープロバイダのためにserviceに追加する必要があります。
これは config.yml の "provider" セクションにプロバイダIDをセットします:

    services:
        my.facebook.user:
            class: Acme\MyBundle\Security\User\Provider\FacebookProvider
            arguments:
                facebook: "@fos_facebook.api"
                userManager: "@fos_user.user_manager"
                validator: "@validator"
                container: "@service_container"

    <?php

    namespace Acme\MyBundle\Security\User\Provider;

    use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
    use Symfony\Component\Security\Core\Exception\UnsupportedUserException;
    use Symfony\Component\Security\Core\User\UserProviderInterface;
    use Symfony\Component\Security\Core\User\UserInterface;
    use \BaseFacebook;
    use \FacebookApiException;

    class FacebookProvider implements UserProviderInterface
    {
        /**
         * @var \Facebook
         */
        protected $facebook;
        protected $userManager;
        protected $validator;

        public function __construct(BaseFacebook $facebook, $userManager, $validator)
        {
            $this->facebook = $facebook;
            $this->userManager = $userManager;
            $this->validator = $validator;
        }

        public function supportsClass($class)
        {
            return $this->userManager->supportsClass($class);
        }

        public function findUserByFbId($fbId)
        {
            return $this->userManager->findUserBy(array('facebookID' => $fbId));
        }

        public function loadUserByUsername($username)
        {
            $user = $this->findUserByFbId($username);

            try {
                $fbdata = $this->facebook->api('/me');
            } catch (FacebookApiException $e) {
                $fbdata = null;
            }

            if (!empty($fbdata)) {
                if (empty($user)) {
                    $user = $this->userManager->createUser();
                    $user->setEnabled(true);
                    $user->setPassword('');
                    $user->setAlgorithm('');
                }

                // TODO use http://developers.facebook.com/docs/api/realtime
                $user->setFBData($fbdata);

                if (count($this->validator->validate($user, 'Facebook'))) {
                    // TODO: the user was found obviously, but doesnt match our expectations, do something smart
                    throw new UsernameNotFoundException('The facebook user could not be stored');
                }
                $this->userManager->updateUser($user);
            }

            if (empty($user)) {
                throw new UsernameNotFoundException('The user is not authenticated on facebook');
            }

            return $user;
        }

        public function refreshUser(UserInterface $user)
        {
            if (!$this->supportsClass(get_class($user)) || !$user->getFacebookId()) {
                throw new UnsupportedUserException(sprintf('Instances of "%s" are not supported.', get_class($user)));
            }

            return $this->loadUserByUsername($user->getFacebookId());
        }
    }

最後に、User モデルに getFaceookId() と setFBData() メソッドを追加する必要もあります。
次の例では、 "firstname" と "lastname" プロパティも追加しています:

    <?php

    namespace Acme\MyBundle\Entity;

    use FOS\UserBundle\Entity\User as BaseUser;

    class User extends BaseUser
    {
        /**
         * @var string
         */
        protected $firstname;

        /**
         * @var string
         */
        protected $lastname;

        /**
         * @var string
         */
        protected $facebookID;

        /**
         * @return string
         */
        public function getFirstname()
        {
            return $this->firstname;
        }

        /**
         * @param string $firstname
         */
        public function setFirstname($firstname)
        {
            $this->firstname = $firstname;
        }

        /**
         * @return string
         */
        public function getLastname()
        {
            return $this->lastname;
        }

        /**
         * @param string $lastname
         */
        public function setLastname($lastname)
        {
            $this->lastname = $lastname;
        }

        /**
         * Get the full name of the user (first + last name)
         * @return string
         */
        public function getFullName()
        {
            return $this->getFirstName() . ' ' . $this->getLastname();
        }

        /**
         * @param string $facebookID
         * @return void
         */
        public function setFacebookID($facebookID)
        {
            $this->facebookID = $facebookID;
            $this->setUsername($facebookID);
            $this->salt = '';
        }

        /**
         * @return string
         */
        public function getFacebookID()
        {
            return $this->facebookID;
        }

        /**
         * @param Array
         */
        public function setFBData($fbdata)
        {
            if (isset($fbdata['id'])) {
                $this->setFacebookID($fbdata['id']);
                $this->addRole('ROLE_FACEBOOK');
            }
            if (isset($fbdata['first_name'])) {
                $this->setFirstname($fbdata['first_name']);
            }
            if (isset($fbdata['last_name'])) {
                $this->setLastname($fbdata['last_name']);
            }
            if (isset($fbdata['email'])) {
                $this->setEmail($fbdata['email']);
            }
        }
    }

> **TIP** Translated Info: 2011/09/18 uechoco a327853ae2
