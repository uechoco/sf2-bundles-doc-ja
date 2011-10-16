標準の FOSUserBundle フォームのオーバーライド
================================================

## Form Type のオーバーライド

FOSUserBundle に含まれる標準のフォームは、
新規ユーザーの登録やプロフィールの更新、パスワードの変更や他にも様々な機能を提供しています。
これらのフォームはバンドルの標準クラスやコントローラで上手く動くようになっています。
しかし、`User` クラスにもっとプロパティを追加し始めたり、登録フォームに幾つかのオプションを追加したいと思った時には、
バンドルのフォームをオーバーライドする必要があることに気づくでしょう。

`Acme\UserBundle\Entity\User` というクラス名で ORM ユーザークラスを作成していると仮定します。
このクラスには、 `name` プロパティを追加していることにします。
これは、ユーザーの username や email address と同様に name を保存しようとしているからです。
これで、ユーザーがあなたのサイトで登録をする時に、username や email、password と同様に name も入力するようになりました。
以下に、 `$name` プロパティとそのバリデータの例を示します。

``` php
// src/Acme/UserBundle/Entity/User.php
<?php

use FOS\UserBundle\Entity\User as BaseUser;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

class User extends BaseUser
{
    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    protected $id;
    
    /**
     * @ORM\Column(type="string", length="255")
     *
     * @Assert\NotBlank(message="Please enter your name.", groups={"Registration", "Profile"})
     * @Assert\MinLength(limit="3", message="The name is too short.", groups={"Registration", "Profile"})
     * @Assert\MaxLength(limit="255", message="The name is too long.", groups={"Registration", "Profile"})
     */
    protected $name;

    // ...
}
```

**Note:**

```
標準では、 Registration バリデーショングループは、新規ユーザーの登録をバリデーションする時に使われます。
設定でこの値を上書きしていない限りは、name プロパティには Registration という名前でバリデーションを追加してください。
```

標準の登録フォームで登録しようとした時に、新しく追加した `name` プロパティが
フォームの要素に含まれていないことに気づくでしょう。
カスタムフォームタイプを作成し、それを使うことをバンドルに設定する必要があります。

最初のステップは、あなたのバンドルの中に新しいフォームタイプを作成することです。
以下のクラスでは、ベースとなる FOSUserBundle の `RegistrationFormType` を継承し、
`name` フィールドを追加しています。

``` php
// src/Acme/UserBundle/Form/Type/RegistrationFormType.php
<?php

namespace Acme\UserBundle\Form\Type;

use Symfony\Component\Form\FormBuilder;
use FOS\UserBundle\Form\Type\RegistrationFormType as BaseType;

class RegistrationFormType extends BaseType
{
    public function buildForm(FormBuilder $builder, array $options)
    {
        parent::buildForm($builder, $options);

        // add your custom field
        $builder->add('name');
    }

    public function getName()
    {
        return 'acme_user_registration';
    }
}
```

これでカスタムフォームタイプが作成できましたので、
このクラスをサービスとして宣言し、タグ付をしなければなりません。
このタグは、`name` 属性は `form.type` という値でなければなりません。
また `alias` 属性はフォームタイプクラスの `getName` メソッドが返す文字列と同じでなければなりません。
この `alias` 属性は、カスタムフォームを使いたいということをバンドルに知らせるための FOSUserBundle の設定において、
何を使うのかを表しています。

以下に、XML形式で定義したフォームタイプのサービス定義の例を示します。

``` xml
<!-- src/Acme/UserBundle/Resources/config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>

<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>

        <service id="acme_user.registration.form.type" class="Acme\UserBundle\Form\Type\RegistrationFormType">
            <tag name="form.type" alias="acme_user_registration" />
            <argument>%fos_user.model.user.class%</argument>
        </service>

    </services>

</container>
```

あるいは、YAMLがお好みの場合:

``` yaml
# src/Acme/UserBundle/Resources/config/services.yml
services:
    acme_user.registration.form.type:
        class: Acme\UserBundle\Form\Type\RegistrationFormType
        arguments: [%fos_user.model.user.class%]
        tags:
            - { name: form.type, alias: acme_user_registration }
```

**Note:**

```
フォームタイプのサービス設定の中で、
コンストラクタの引数として、`fos_user.model.user.class` コンテナパラメータを指定しています。
フォームタイプクラスでコンストラクタを最定義しない限りは、
FOSUserBundle のフォームタイプで必要なのでこの引数を入れる必要があります。
```

最後に、FOSUserBundle の設定を更新しなければなりません。
これは標準のフォームタイプの代わりにカスタムフォームタイプを使いたいからです。
以下に、YAMLで registration のフォームタイプを変更する設定を示します。

``` yaml
# app/config/config.yml
fos_user:
    # ...
    registration:
        form:
            type: acme_user_registration
```

フォームタイプのサービス設定タグにあった `alias` の値がどのように使われるかというと、
バンドルの設定において、 FOSUserBundle に対してカスタムフォームタイプを使うことを知らせるために使っています。

## フォームハンドラのオーバーライド

FOSUserBundle のフォームハンドラによって提供されている標準機能をオーバーライドする方法は2つあります。
最も簡単な方法は、ハンドラの `onSuccess` メソッドをオーバーライドする方法です。
`onSuccess` メソッドは、フォームが束縛(bound)され、バリデートされた後に呼ばれます。

2つ目の方法は、`process` メソッドをオーバーライドする方法です。
`process` メソッドのオーバーライドは、フォームをバインドしてバリデーションする時により高度な機能が必要になった時にだけ必要になるでしょう。

ユーザー登録が成功した後に幾つかの機能を追加したいと仮定しましょう。
まずはじめに、`FOS\UserBundle\Form\Handler\RegistrationFormHandler` を継承した新しいクラスを作成し、
protected な `onSuccess` メソッドをオーバーライドします。

``` php
// src/Acme/UserBundle/Form/Handler/RegistrationFormHandler.php
<?php

namespace Acme\UserBundle\Form\Handler;

use FOS\UserBundle\Form\Handler\RegistrationFormHandler as BaseHandler;
use FOS\UserBundle\Model\UserInterface;

class RegistrationFormHandler extends BaseHandler
{
    protected function onSuccess(UserInterface $user, $confirmation)
    {
        // Note: if you plan on modifying the user then do it before calling the 
        // parent method as the parent method will flush the changes

        parent::onSuccess($user, $confirmation);

        // otherwise add your functionality here
    }
}
```

**Note:**

```
親クラスの onSuccess クラスを呼ばなければ、
FOSUserBundle のハンドラがサブミッションの成功時に標準で実行するロジックが動作しません。
```

また、ハンドラの `process` メソッドをオーバーライドする方法を選択することも出来ます。
`process` メソッドをオーバーライドする方法を選択した場合は、
サブミッションの成功時に必要とされるロジックの中で実装されているものと同じように、
フォームのデータをバインドしてバリデーションする責任が発生します。

``` php
// src/Acme/UserBundle/Form/Handler/RegistrationFormHandler.php
<?php

namespace Acme\UserBundle\Form\Handler;

use FOS\UserBundle\Form\Handler\RegistrationFormHandler as BaseHandler;

class RegistrationFormHandler extends BaseHandler
{
    public function process($confirmation = false)  
    {
        $user = $this->userManager->createUser();
        $this->form->setData($user);

        if ('POST' == $this->request->getMethod()) {
            $this->form->bindRequest($this->request);
            if ($this->form->isValid()) {
              
                // do your custom logic here

                return true;
            }
        }

        return false;
    }
}
```

**Note:**

```
process メソッドはサブミッションの成功時に true を、失敗時には false を返さなければなりません。
```

ここまででカスタムフォームハンドラクラスを作成し実装していますので、
コンテナでサービスとして設定しなければなりません。
フォームハンドラをサービスとして設定するXMLの例をいかに示します。

``` xml
<!-- src/Acme/UserBundle/Resources/config/services.xml -->
<?xml version="1.0" encoding="UTF-8" ?>

<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>

        <service id="acme_user.form.handler.registration" class="Acme\UserBundle\Form\Handler\RegistrationFormHandler" scope="request" public="false">
            <argument type="service" id="fos_user.registration.form" />
            <argument type="service" id="request" />
            <argument type="service" id="fos_user.user_manager" />
            <argument type="service" id="fos_user.mailer" />
        </service>

    </services>

</container>
```

あるいは、YAMLが好みであれば:

``` yaml
# src/Acme/UserBundle/Resources/config/services.yml
services:
    acme_user.form.handler.registration:
        class: Acme\UserBundle\Form\Handler\RegistrationFormHandler
        arguments: ["@fos_user.registration.form", "@request", "@fos_user.user_manager", "@fos_user.mailer"]
        scope: request
        public: false
```

ここではクラスのコンストラクタに引数として他のサービスを挿入していますが、
これらの引数はベースの FOSUserBundle フォームハンドラクラスで必要になるためです。

これでコンテナで新しいフォームハンドラの設定ができましたので、
残っている作業は FOSUserBundle の設定を更新するだけです

``` yaml
# app/config/config.yml
fos_user:
    # ...
    registration:
        form:
            handler: acme_user.form.handler.registration
```

設定したサービスの `id` がバンドルの設定でどのように使用されるかというと、
FOSUserBundle にカスタムフォームハンドラを使うことを伝えるために使います。

ここまでで、ユーザがサイトに登録する時に、
フォームのサブミッションを取り扱うようになりました。

> **TIP** Translated Info: 2011/10/16 uechoco 5ef597cb94
