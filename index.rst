在 Symfony2 建立一個部落格
===========================

介紹
------------

這個教學會透過建立一個完整功能的部落格網站來引導你使用 `Symfony2 <http://symfony.com/>`_ 。
這裡使用了 Symfony2 的標準版本，它包含了在建立你自己網站時需要的主要元件。這個教學切割為多個部份
，每個部份涵蓋了 Symfony2 不一樣的層面與元件。這個教學參考了 symfony 1 的
`Jobeet <http://www.symfony-project.org/jobeet/1_4/Doctrine/en/>`_ 以類似手法製作。

教學章節
~~~~~~~~~~~~~~

.. toctree::
    :maxdepth: 1

    docs/configuration-and-templating
    docs/validators-and-forms
    docs/doctrine-2-the-blog-model
    docs/extending-the-model-blog-comments
    docs/customising-the-view-more-with-twig
    docs/testing-unit-and-functional-phpunit

展示網站
------------

這個 symblog 網站可以在 `http://symblog.co.uk <http://symblog.co.uk/>`_ 瀏覽，原始碼放在
`Github <https://github.com/dsyph3r/symblog>`_ ，它遵循了這個教學的每個部份。

涵蓋範圍
--------

這個教學的目標是涵蓋你在建置 Symfony2 網站常會遇到的任務。

    1.  軟體包
    2.  Controllers
    3.  樣板 (使用 TWIG)
    4.  Model - Doctrine 2
    5.  搬遷 
    6.  資料裝置
    7.  驗證
    8.  表單
    9.  網址路徑
    10. 資源管理
    11. 發信
    12. 環境
    13. 自訂錯誤頁
    14. 安全
    15. 使用者與連線
    16. 產生增刪改查功能
    17. 快取
    18. 測試
    19. 佈署

Symfony2 可以高度客製與提供許多不同的方法來完成同樣工作，像是寫入設定的選項有 YAML 、 XML 、
PHP 或註解，建立樣板可以使用 Twig 或 PHP 。未來讓這個教學簡化，我們在設定部份會使用 YAML 與註解
，樣板則是 Twig 。 `Symfony 手冊 <http://symfony.com/doc/current/book/index.html>`_ 提供了
豐富的資源與範例說明如何使用其他方法。如果有其他人想要參與完成這個教學或是其他方式，只需要在
`Github <https://github.com/dsyph3r/symblog-docs>`_ 建立一個衍生版本，然後發出 pull requests

翻譯
------------

西班牙文
~~~~~~~

Symblog 由 `Lisper <https://twitter.com/#!/esymfony>`_ 翻譯成 `西班牙文 <http://symblog.site90.net/>`_ 。

法文
~~~~~~~

Symblog 由 `Clement Keirua <https://twitter.com/clemkeirua>`_ 翻譯為 `法文 <http://keiruaprod.fr/symblog-fr/>`_ 。

正體中文
~~~~~~~

Symblog 由 `Ricky <https://github.com/RickySu>`_ 與 `kiang <https://github.com/kiang>`_ 翻譯為 `中文 <https://github.com/kiang/symblog-docs-tw>`_ 。

作者
------

這個教學是由 `dsyph3r <http://twitter.com/#!/dsyph3r>`_ 建立

貢獻
------------

這個教學的 `原始碼 <https://github.com/dsyph3r/symblog-docs>`_ 放在 Github ，如果你想要改善或是
延伸這個教學，只要建立一個衍生版本然後發出回推請求。你也可以用 `GitHub Issue Tracker <https://github.com/dsyph3r/symblog-docs/issues>`_
發表問題。如果你對於改善這個教學的視覺設計感興趣，請
`與我聯繫 <http://twitter.com/#!/dsyph3r>`_!

Credits
-------

Special thanks to all the contributors of the
`Official Symfony2 documentation <http://symfony.com/doc/current/>`_. This
provided an invaluable resource of information.

國旗圖示來自 `famfamfam <http://www.famfamfam.com/lab/icons/flags/>`_ 。

搜尋
---------

在找一個特定主題嗎？使用 :ref:`search` 。
