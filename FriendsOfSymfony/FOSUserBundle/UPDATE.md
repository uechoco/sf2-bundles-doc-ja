バンドルの更新方法
===========================

このドキュメントでは、あるFOSUSerBundleバンドルのバージョンから新しいバージョンに
アップグレードする方法を説明します。
バンドルの公式APIを使っている時にしておく必要のある変更だけを取り扱います。
コア部分をハックした場合は、時系列に近い形でしたがっておくべきです。

* The methods relative to the groups have been removed from `FOS\UserBundle\Model\UserInterface`
  and a new `FOS\UserBundle\Model\GroupableInterface` has been added (implemented
  by `FOS\UserBundle\Model\User`).

* The User class of the bundle does not contain the timestampable fields anymore
  as they were not used by the bundle.

* The `fos:user:changePassword` command has been renamed to `fos:user:change-password`.

* The way to configure the forms has been refactored to give more flexibility:

    * The configuration of the type now accepts the name of the type. You can
      register your own type by creating a tagged service in the container:

        <tag name="form.type" alias="acme_custom_type" />

    * The configuration of the handler now accepts a service id.

* The form classes have been moved to subnamespaces to keep them organized.

* The ACL implementation using JMSSecurityExtraBundle which was broken
  since Symfony beta2 has been removed.

* The Twig block has been renamed from `content` to `fos_user_content`.

> **TIP** Translated Info: 2011/09/19 uechoco 1642f98d
