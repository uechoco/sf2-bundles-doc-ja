標準の FOSUserBundle コントローラのオーバーライド
===================================================

FOSUserBundle に含まれている標準のコントローラは、
一般的なユースケースには十分なくらいのたくさんの機能を提供しています。
しかし、それらの機能を拡張したり、
アプリケーションの特定のニーズに適合するようなロジックを追加したりする必要があることに、
気づくことでしょう。

バンドルのコントローラをオーバーライドする最初のステップは、
FOSUserBundle を親に持つような子バンドルを作ることです。
次のコードスニペットは、FOSUserBundle の子として宣言された `AcmeUserBundle` という名前の新しいバンドルを作成しています。

``` php
// src/Acme/UserBundle/AcmeUserBundle.php
<?php

namespace Acme\UserBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class AcmeUserBundle extends Bundle
{
    public function getParent()
    {
        return 'FOSUserBundle';
    }
}
```

**Note:**

```
Symfony2 フレームワークでは、あるバンドルは１つの子バンドルしか持つことができません。
FOSUserBundle の子として別のバンドルを作ることはできないということです。
```

ここまでで新しい子バンドルを作成していますので、
オーバーライドしたいコントローラと同じ名前と場所でコントローラクラスを作成することができます。
この例では、FOSUserBundle の `RegistrationController` クラスを拡張し、
追加の機能要望のメソッドを単にオーバーライドすることで、`RegistrationController` をオーバーライドしています。

以下の例では `registerAction` メソッドをオーバーライドしています。
そのメソッドは基底コントローラのコードから使われ、新しいユーザーの登録のロギングを追加しています。

``` php
// src/Acme/UserBundle/Controller/RegistrationController.php
<?php

namespace Acme\UserBundle\Controller;

use FOS\UserBundle\Controller\RegistrationController as BaseController;

class RegistrationController extends BaseController
{
    public function registerAction()
    {
        $form = $this->container->get('fos_user.registration.form');
        $formHandler = $this->container->get('fos_user.registration.form.handler');
        $confirmationEnabled = $this->container->getParameter('fos_user.registration.confirmation.enabled');

        $process = $formHandler->process($confirmationEnabled);
        if ($process) {
            $user = $form->getData();

            /*****************************************************
             * Add new functionality (e.g. log the registration) *
             *****************************************************/
            $this->get('logger')->info(
                sprintf('New user registration: %s', $user)
            );

            if ($confirmationEnabled) {
                $this->container->get('session')->set('fos_user_send_confirmation_email/email', $user->getEmail());
                $route = 'fos_user_registration_check_email';
            } else {
                $this->authenticateUser($user);
                $route = 'fos_user_registration_confirmed';
            }

            $this->setFlash('fos_user_success', 'registration.flash.user_created');
            $url = $this->container->get('router')->generate($route);

            return new RedirectResponse($url);
        }

        return $this->container->get('templating')->renderResponse('FOSUserBundle:Registration:register.html.'.$this->getEngine(), array(
            'form' => $form->createView(),
            'theme' => $this->container->getParameter('fos_user.template.theme'),
        ));
    }
}
```

**Note:**

```
オーバーライドしたい FOSUserBundle のコントローラクラスを拡張せずに
FrameworkBundle が提供している ContainerAware や Controller クラスを代わりに拡張したい場合は、
オーバーライドしようとしている FOSUserBundle のコントローラが持つすべてのメソッドを実装しなければなりません。
```

> **TIP** Translated Info: 2011/09/24 uechoco 837707a0
