[Part 1] - Symfony2 設定與樣板
================================================

概要
--------

這一節會介紹建立 Symfony2 網站的幾個開始步驟，我們會下載與使用 Symfony2
`標準版 <http://symfony.com/doc/current/glossary.html#term-distribution>`_,
來建立部落格軟體包以及放入主要網頁樣板。在這一節結束後你會製作出一個設定好的 Symfony2 網站，
能夠透過本地網址存取，像是 ``http://symblog.dev/`` 。這個網站會包含部落格主要的網頁結構以
及一些測試內容。

在這個章節會展示下面這些主題：

    1. 設定一個 Symfony2 應用
    2. 設定一個開發網址
    3. Symfony2 軟體包
    4. 網址路徑
    5. Controller
    6. 使用 Twig 製作樣板

下載與安裝
------------------

如同上面提到，我們會使用 Symfony2 標準版，這個版本包含了完整的核心函式庫與建立網站大多會需要的
軟體包，你可以從官方網站 `下載 <http://symfony.com/download>`_ 。這個教學不希望重複官方網站的
線上手冊說明，所以請先參考 `安裝與設定 Symfony2 <http://symfony.com/doc/current/book/installation.html>`_
這個章節來了解環境需求。線上手冊會引導你進行下載軟體包的選擇、安裝必要外部軟體包與資料夾的權限
設定等。

.. warning::

    需要特別注意安裝說明的 `設定權限 <http://symfony.com/doc/current/book/installation.html#configuration-and-setup>`_
    部份，這會說明一些設定 ``app/cache`` 與 ``app/logs`` 權限的方法，讓網頁伺服器使用者與指令
    模式使用者都可以有寫入的權限。

建立一個開發網址
-----------------------------

為了進行這個教學，我們會使用本地網址 ``http://symblog.dev/`` ，不過你可以選擇自己希望使用的網址。
這些步驟主要針對 `Apache <http://httpd.apache.org/>`_ 說明，也假設你已經在自己主機安裝並且執行了
Apache。如果你已經知道如何設定本地網址，或是使用像是 `nginx <http://nginx.net/>`_ 的網頁伺服器，
你可以跳過這個部份。

.. note::

    這些步驟是在特定 Linux 套件 Fedora 完成，因此路徑名稱等資訊可能會跟你操作中的作業系統不一樣。

先開始建立一個 Apache 的虛擬主機。找到 Apache 設定檔案，然後附加下面這些設定。請依據自己的環境調整
``DocumentRoot`` 與 ``Directory`` 路徑，Apache 設定檔案的位置與名稱會依據使用的作業系統產生差異，
在 Fedora 是放在 ``/etc/httpd/conf/httpd.conf`` ，你會需要透過 ``sudo`` 權限來編輯這個檔案。

.. code-block:: text

    # /etc/httpd/conf/httpd.conf

    NameVirtualHost 127.0.0.1

    <VirtualHost 127.0.0.1>
      ServerName symblog.dev
      DocumentRoot "/var/www/html/symblog.dev/web"
      DirectoryIndex app.php
      <Directory "/var/www/html/symblog.dev/web">
        AllowOverride All
        Allow from All
      </Directory>
    </VirtualHost>


接著新增一個網址到 ``/etc/hosts`` 主機檔案最下面，同樣的，你會需要透過 ``sudo`` 權限來編輯這個檔案。

.. code-block:: text

    # /etc/hosts
    127.0.0.1     symblog.dev

最後不要忘記重新啟動 Apache 服務，這會套用剛剛異動的設定。

.. code-block:: bash

    $ sudo service httpd restart

.. tip::

    如果你發現自己經常在建立虛擬網址，你可以透過 `Dynamic virtual hosts <http://blog.dsyph3r.com/2010/11/apache-dynamic-virtual-hosts.html>`_
    簡化這個步驟。

你現在應該可以訪問 ``http://symblog.dev/app_dev.php/`` 。

.. image:: /_static/images/part_1/welcome.jpg
    :align: center
    :alt: Symfony2 welcome page

如果這是你第一次看到 Symfony2 歡迎頁，可以花些時間看看它的展示頁，每個展示頁會提供相關程式碼來示範畫面
背後如何運作。

.. note::

    你也會發現在歡迎頁下方的一個工具列，這是開發者工具，可以提供你許多關於應用程式狀態的寶貴資訊。可以
    透過這個工具列看到的這些資訊包含頁面執行時間、記憶體用量、資料庫查詢、認證狀態等等。預設這個工具列
    只會在執行於 ``dev`` 環境時顯示，在正式環境提供這個工具列會造成很大的安全風險，因為它揭露了應用程
    式的許多細節。這個工具列的參考資訊會在介紹新功能時提到。

設定 Symfony：網頁介面
----------------------------------

Symfony2 加入了一個網頁介面來進行網站的多個設定，像是資料庫。我們在這個專案會需要一個資料庫，所以現在就
開始使用這個設定工具。

瀏覽 ``http://symblog.dev/app_dev.php/`` 並且點選設定按鈕，進入設定資料庫的細節（這個教學假設使用 MySQL
資料庫，不過你可以選擇任何可以使用的資料庫），接著在下一頁會產生一個 CSRF 權杖，你會看到 Symfony2 產生的
各種參數。注意這個頁面的說明，有時你的 ``app/config/parameters.ini`` 檔案無法直接寫入，所以你會需要複製
這些設定到位於 ``app/config/parameters.ini`` 的檔案（這些設定可以直接取代檔案中既有設定）。


軟體包：組成 Symfony2 的積木
----------------------------------

軟體包是任何 Symfony2 應用的基礎組成積木，事實上 Symfony2 框架本身也是個軟體包。軟體包讓我們可以分離功能來
提供可以重複運用的單位程式碼。它們封裝了整個為了軟體包目的所需要的東西，包含 controllers 、 model 與樣板，
以及許多資源,以及許多像是圖片與 CSS 之類的資源。我們會建立一個使用 Blogger 命名空間的軟體包來製作網站。如果
你不熟悉 PHP 的命名空間，你應該要花些時間去閱讀相關文件，因為在 Symfony2 中大量使用，每個元件都使用了命名空
間。可以參考
`Symfony2 autoloader <http://symfony.com/doc/current/cookbook/tools/autoloader.html>`_
了解 Symfony2 如何作到自動載入功能。

.. tip::

    確實了解命名空間可以協助排除一些常見問題，像是資料夾結構與命名空間結構不一致你就會遇到。

建立軟體包
~~~~~~~~~~~~~~~~~~~

為了要封裝部落格的功能，我們要建立一個 Blog 軟體包。它將會包含所有需要的檔案，這樣一來可以輕易的移植到另一個
Symfony2 應用程式。 Symfony2 提供了許多工具來協助我們執行一般操作，其中一個就是軟體包產生器。

要執行軟體包產生器請執行下面指令，畫面會顯示一些提示讓你設定軟體包應該如何配置。在這裡應該要使用每個提示的預
設值。

.. code-block:: bash

    $ php app/console generate:bundle --namespace=Blogger/BlogBundle --format=yml

產生器執行後，Symfony2 會建立基本的軟體包結構，在這裡需要注意一些重要的變動。

.. tip::

    其實你並不需要使用 Symfony2 提供的產生器工具，它們只是放在那裡來幫你，你當然可以手動建立軟體包的資料夾結構
    與檔案。雖然使用產生器並非必要的，它們的確提供了一些好處，像是它們可以快速的執行所有必要工作來讓軟體包產生
    與執行，其中一個例子就是註冊軟體包。

註冊軟體包
......................

我們的新軟體包 ``BloggerBlogBundle`` 已經註冊在核心檔案 ``app/AppKernel.php`` ，Symfony2 要求我們註冊所有應用
程式使用到的軟體包，你也會注意到一些軟體包只有在 ``dev`` 或 ``test`` 環境下註冊，在正式環境 ``prod`` 載入這些軟
體包會因為一些用不到的功能而徒增系統負擔。下面這段程式碼顯示如何註冊 ``BloggerBlogBundle`` 。

.. code-block:: php

    // app/AppKernel.php
    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
            // ..
                new Blogger\BlogBundle\BloggerBlogBundle(),
            );
            // ..

            return $bundles;
        }

        // ..
    }

網址路徑
.......

軟體包的路徑已經匯入到應用程式的主要網址路徑檔案 ``app/config/routing.yml`` 。

.. code-block:: yaml

    # app/config/routing.yml
    BloggerBlogBundle:
        resource: "@BloggerBlogBundle/Resources/config/routing.yml"
        prefix:   /

prefix 選項讓我們可以掛載整個 ``BloggerBlogBundle`` 的網址路徑，在我們的例子中已經選擇掛載在預設的 ``/`` 。
如果你想要所有的網址路徑開始是 ``/blogger`` ，可以將 prefix 改為 ``prefix: /blogger`` 。

預設結構
.................

在 ``src`` 資料夾已經建立了預設的軟體包結構，開始的是最上層 ``Blogger`` 資料夾，它直接對應到我們為軟體包設定的
 ``Blogger`` 命名空間，在這之下可以看到包含實際軟體包的 ``BlogBundle`` 資料夾，裡面的內容結構有一部份名稱就解
釋了它的用途。

預設 Controller
~~~~~~~~~~~~~~~~~~~~~~

在軟體包產生器製作的檔案中 Symfony2 建立了一個預設 controller ，我們可以透過瀏覽
 ``http://symblog.dev/app_dev.php/hello/symblog`` 來執行它，你可以看到一個簡單的歡迎頁。試著修改網址的 ``symblog``
為你所製作的名稱，我們可以藉此在比較高的層級檢驗頁面的產生。

網址路徑
......

 ``BloggerBlogBundle`` 的路徑檔案放在 ``src/Blogger/BlogBundle/Resources/config/routing.yml`` ，包含了下面的
預設網址路徑規則。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_homepage:
        pattern:  /hello/{name}
        defaults: { _controller: BloggerBlogBundle:Default:index }

網址路徑是由一個樣式與一些預設選項組成，樣式會用來檢查網址，預設選項則是指定在網址符合時應該要執行的 controller 。在樣式
 ``/hello/{name}`` 中， ``{name}`` 替位符號會對應到任意數值，因為沒有設定特別條件。網址路徑也沒有指定任何內涵、格式或
HTTP 方法，沒有指定 HTTP 方法表示來自 GET 、 POST 、 PUT 等方式的請求都會視為符合樣式。

如果網址符合所有指定的條件，就會執行預設選項中設定的 _controller ， _controller 選項指定了 controller 的邏輯名稱，讓
Symfony2 可以對應到一個指定的檔案。上面的例子會執行 ``Default`` controller 中的 ``index`` ，檔案位置在
 ``src/Blogger/BlogBundle/Controller/DefaultController.php`` 。

關於 Controller
..............

在這個例子中的 controller 非常簡單， ``DefaultController`` 繼承了 ``Controller`` ，它提供了一些有用的方法，像是下面用到的
``render`` 。由於我們的網址路徑定義了一個替位符號 ``$name`` ，它會被送到方法中作為參數。 ``index`` 方法只有呼叫 ``render``
方法來指定位於 ``BloggerBlogBundle`` 預設樣板資料夾中的 ``index.html.twig`` 樣板來顯示。樣板名稱的格式是
``bundle:controller:template`` ，在我們的例子中是 ``BloggerBlogBundle:Default:index.html.twig`` ，會對應到 ``BloggerBlogBundle``
 ``Default`` 樣板資料夾的 ``index.html.twig`` 樣板，實際上的路徑為
``src/Blogger/BlogBundle/Resources/views/Default/index.html.twig`` 。在應用與所對應的軟體包中可以在樣板顯示時指定許多不同
的樣板格式，在這個章節的後面會做介紹。

我們也透過 ``array`` 選項傳遞了變數 ``$name`` 到樣板。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/DefaultController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class DefaultController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('BloggerBlogBundle:Default:index.html.twig', array('name' => $name));
        }
    }

關於樣板 (也就是 View)
.......................

如你所見，這個樣板非常簡單，只有印出 Hello 以及接著 controller 傳送過來的參數 name 。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Default/index.html.twig #}
    Hello {{ name }}!

整理
~~~~~~~~~~~

由於我們不需要一些產生器製作出來的檔案，可以做些整理。

controller 檔案 ``src/Blogger/BlogBundle/Controller/DefaultController.php`` 可以刪除，包含樣板資料夾
``src/Blogger/BlogBundle/Resources/views/Default/`` 與其中的內容。最後移除定義在
``src/Blogger/BlogBundle/Resources/config/routing.yml`` 的網址路徑。

樣板
----------

在 Symfony2 中使用樣板預設有 `Twig <http://www.twig-project.org/>`_ 與 PHP 兩個選擇，在不同的函式庫當然
可以做不同的選擇，這要感謝 Symfony2 實做了 `Dependency Injection Container <http://symfony.com/doc/current/book/service_container.html>`_
我們會基於下面理由選擇使用 Twig 。

1. Twig 非常快，Twig 樣板會編譯成 PHP 物件，所以使用 Twig 樣板不會造成太大的負擔。
2. Twig 非常簡潔， Twig 讓我們可以透過少量程式碼執行樣板功能， PHP 在部份情況下則是會相對冗長。
3. Twig 支援樣板繼承，這是筆者個人喜愛的特色之一。樣板能夠繼承與覆寫其他樣板，讓子樣板可以修改來自父樣板的預設值。
4. Twig 非常安全， Twig 預設啟用了輸出的檢查，甚至還為匯入的樣板提供一個沙箱環境。
5. Twig 容易擴充，Twig 帶來了許多你對樣板期待的常見核心功能，而一些你預期需要的其他功能， Twig 可以輕易的延伸。

這只是 Twig 的一些好處，更多關於為什麼該用 Twig 的理由可以參考 `Twig <http://www.twig-project.org/>`_ 官方網站。

Layout Structure
~~~~~~~~~~~~~~~~

As Twig supports template inheritance, we are going to use the
`Three level inheritance <http://symfony.com/doc/current/book/templating.html#three-level-inheritance>`_
approach. This approach allows us to modify the view at 3 distinct levels within the
application, giving us plenty of room for customisations.

Main Template - Level 1
.......................

Lets start by creating our basic block level template for symblog. We need 2
files here, the template and the CSS. As Symfony2 supports
`HTML5 <http://diveintohtml5.org/>`_ we will also be using it.

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html"; charset=utf-8" />
            <title>{% block title %}symblog{% endblock %} - symblog</title>
            <!--[if lt IE 9]>
                <script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
            <![endif]-->
            {% block stylesheets %}
                <link href='http://fonts.googleapis.com/css?family=Irish+Grover' rel='stylesheet' type='text/css'>
                <link href='http://fonts.googleapis.com/css?family=La+Belle+Aurore' rel='stylesheet' type='text/css'>
                <link href="{{ asset('css/screen.css') }}" type="text/css" rel="stylesheet" />
            {% endblock %}
            <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
        </head>
        <body>

            <section id="wrapper">
                <header id="header">
                    <div class="top">
                        {% block navigation %}
                            <nav>
                                <ul class="navigation">
                                    <li><a href="#">Home</a></li>
                                    <li><a href="#">About</a></li>
                                    <li><a href="#">Contact</a></li>
                                </ul>
                            </nav>
                        {% endblock %}
                    </div>

                    <hgroup>
                        <h2>{% block blog_title %}<a href="#">symblog</a>{% endblock %}</h2>
                        <h3>{% block blog_tagline %}<a href="#">creating a blog in Symfony2</a>{% endblock %}</h3>
                    </hgroup>
                </header>

                <section class="main-col">
                    {% block body %}{% endblock %}
                </section>
                <aside class="sidebar">
                    {% block sidebar %}{% endblock %}
                </aside>

                <div id="footer">
                    {% block footer %}
                        Symfony2 blog tutorial - created by <a href="https://github.com/dsyph3r">dsyph3r</a>
                    {% endblock %}
                </div>
            </section>

            {% block javascripts %}{% endblock %}
        </body>
    </html>

.. note::

    There are 3 external files pulled into the template, 1 JavaScript and 2 CSS.
    The JavaScript file fixes the lack of HTML5 support in IE browsers pre version
    9. The 2 CSS files import fonts from
    `Google Web font <http://www.google.com/webfonts>`_.

This template marks up the main structure of our blogging website. Most
of the template consists of HTML, with the odd Twig directive. Its these
Twig directives that we will examine now.

We will start by focusing on the document HEAD. Lets look at the title:

.. code-block:: html

    <title>{% block title %}symblog{% endblock %} - symblog</title>

The first thing you'll notice is the alien ``{%`` tag. Its not HTML, and its
definitely not PHP. This is one of the 3 Twig tags. This tag is the Twig
``Do something`` tag. It is used to execute statements such as control statements and
for defining block elements. A full list of
`control structures <http://www.twig-project.org/doc/templates.html#list-of-control-structures>`_
can be found in the Twig Documentation. The Twig block we have defined in the
title does 2 things; It sets the block identifier to title, and provides a
default output between the block and endblock directives. By defining a block we
can take advantage of Twig's inheritance model. For example, on a page to
display a blog post we would want the page title to reflect the title of the
blog. We can achieve this by extending the template and overriding the title block.

.. code-block:: html

    {% extends '::base.html.twig' %}

    {% block title %}The blog title goes here{% endblock %}

In the above example we have extended the applications base template that first
defined the title block. You'll notice the template format used with the
``extends`` directive is missing the ``Bundle`` and the ``Controller`` parts,
remember the template format is ``bundle:controller:template``. By excluding the
``Bundle`` and the ``Controller`` parts we are specifiying the use of the application
level templates defined at ``app/Resources/views/``.

Next we have defined another title block and put in some
content, in this case the blog title. As the parent template already
contains a title block, it is overridden by our new one. The title would now
output as 'The blog title goes here - symblog'. This functionality provided by
Twig will be used extensively when creating templates.

In the stylesheets block we are introduced to the next Twig tag, the ``{{`` tag,
or the ``Say something`` tag.

.. code-block:: html

    <link href="{{ asset('css/screen.css') }}" type="text/css" rel="stylesheet" />

This tag is used to print the value of variable or expression. In the above example
it prints out the return value of the ``asset`` function, which provides us with
a portable way to link to the application assets, such as CSS, JavaScript, and images.

The ``{{`` tag can also be combined with filters to manipulate the output before
printing.

.. code-block:: html

    {{ blog.created|date("d-m-Y") }}

For a full list of filters check the
`Twig Documentation <http://www.twig-project.org/doc/templates.html#list-of-built-in-filters>`_.

The last Twig tag, which we have not seen in the templates is the comment tag ``{#``.
Its usage is as follows:

.. code-block:: html

    {# The quick brown fox jumps over the lazy dog #}

No other concepts are introduced in this template. It provides the main
layout ready for us to customise it as we need.

Next lets add some styles. Create a stylesheet at ``web/css/screen.css`` and add
the following content. This will add styles for the main template.

.. code-block:: css

    html,body,div,span,applet,object,iframe,h1,h2,h3,h4,h5,h6,p,blockquote,pre,a,abbr,acronym,address,big,cite,code,del,dfn,em,img,ins,kbd,q,s,samp,small,strike,strong,sub,sup,tt,var,b,u,i,center,dl,dt,dd,ol,ul,li,fieldset,form,label,legend,table,caption,tbody,tfoot,thead,tr,th,td,article,aside,canvas,details,embed,figure,figcaption,footer,header,hgroup,menu,nav,output,ruby,section,summary,time,mark,audio,video{border:0;font-size:100%;font:inherit;vertical-align:baseline;margin:0;padding:0}article,aside,details,figcaption,figure,footer,header,hgroup,menu,nav,section{display:block}body{line-height:1}ol,ul{list-style:none}blockquote,q{quotes:none}blockquote:before,blockquote:after,q:before,q:after{content:none}table{border-collapse:collapse;border-spacing:0}

    body { line-height: 1;font-family: Arial, Helvetica, sans-serif;font-size: 12px; width: 100%; height: 100%; color: #000; font-size: 14px; }
    .clear { clear: both; }

    #wrapper { margin: 10px auto; width: 1000px; }
    #wrapper a { text-decoration: none; color: #F48A00; }
    #wrapper span.highlight { color: #F48A00; }

    #header { border-bottom: 1px solid #ccc; margin-bottom: 20px; }
    #header .top { border-bottom: 1px solid #ccc; margin-bottom: 10px; }
    #header ul.navigation { list-style: none; text-align: right; }
    #header .navigation li { display: inline }
    #header .navigation li a { display: inline-block; padding: 10px 15px; border-left: 1px solid #ccc; }
    #header h2 { font-family: 'Irish Grover', cursive; font-size: 92px; text-align: center; line-height: 110px; }
    #header h2 a { color: #000; }
    #header h3 { text-align: center; font-family: 'La Belle Aurore', cursive; font-size: 24px; margin-bottom: 20px; font-weight: normal; }

    .main-col { width: 700px; display: inline-block; float: left; border-right: 1px solid #ccc; padding: 20px; margin-bottom: 20px; }
    .sidebar { width: 239px; padding: 10px; display: inline-block; }

    .main-col a { color: #F48A00; }
    .main-col h1,
    .main-col h2
        { line-height: 1.2em; font-size: 32px; margin-bottom: 10px; font-weight: normal; color: #F48A00; }
    .main-col p { line-height: 1.5em; margin-bottom: 20px; }

    #footer { border-top: 1px solid #ccc; clear: both; text-align: center; padding: 10px; color: #aaa; }

Bundle Template - Level 2
.........................

We now move onto creating the layout for the Blog bundle. Create a file located at
``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` and add the
following content.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    {% extends '::base.html.twig' %}

    {% block sidebar %}
        Sidebar content
    {% endblock %}

At a first glance this template may seem a little simple, but its simplicity is
the key. Firstly it extends the applications base template that we created earlier.
Secondly it overrides the parent sidebar block with some dummy content. As the
sidebar will be present on all pages of our blog it makes sense to perform the
customisation at this level. You may ask why don't we just put the customisation
in the application template as it will be present on all pages. This is simple,
the application knows nothing about the Bundle and shouldn't. The Bundle should
self contain all its functionality and rendering the sidebar is part of this
functionality. OK, so why don't we just place the sidebar in each of the page
templates? Again this is simple, we would have to duplicate the sidebar each
time we added a page. Further this level 2 template gives us the flexibility in
the future to add other customisations that all children templates will inherit.
For example, we may want to change the footer copy on all pages, this would be a
great place to do this.

Page Template - Level 3
.......................

Finally we are ready for the controller layout. These layouts will commonly be
related to a controller action, i.e., the blog show action will have a
blog show template.

Lets start by creating the controller for the homepage and its template. As this
is the first page we are creating we need to create the controller. Create the
controller at ``src/Blogger/BlogBundle/Controller/PageController.php`` and add
the following:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/PageController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class PageController extends Controller
    {
        public function indexAction()
        {
            return $this->render('BloggerBlogBundle:Page:index.html.twig');
        }
    }

Now create the template for this action. As you can see in the controller action
we are going to render the Page index template. Create the template at
``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig``

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        Blog homepage
    {% endblock %}

This introduces the final template format we can specify. In this example
the template ``BloggerBlogBundle::layout.html.twig`` is extended where
the ``Controller`` part of the template name is ommitted. By excluding the
``Controller`` part we are specifiying the use of the Bundle level template
created at ``src/Blogger/BlogBundle/Resources/views/layout.html.twig``.

Now lets add a route for our homepage. Update the Bundle routing config located
at ``src/Blogger/BlogBundle/Resources/config/routing.yml``.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_homepage:
        pattern:  /
        defaults: { _controller: BloggerBlogBundle:Page:index }
        requirements:
            _method:  GET

Lastly we need to remove the default route for the Symfony2 welcome screen.
Remove the ``_welcome`` route at the top of the ``dev`` routing file located at
``app/config/routing_dev.yml``.

We are now ready to view our blogger template. Point your browser to
``http://symblog.dev/app_dev.php/``.

.. image:: /_static/images/part_1/homepage.jpg
    :align: center
    :alt: symblog main template layout

You should see the basic layout of the blog, with
the main content and sidebar reflecting the blocks we have overridden in the relevant
templates.

The About Page
--------------

The final task in this part of the tutorial will be creating a static page for the
about page. This will demonstrate how to link pages together, and further enforce the
Three Level Inheritance approach we have adopted.

The Route
~~~~~~~~~

When creating a new page, one of the first tasks should be creating the route for it.
Open up the ``BloggerBlogBundle`` routing file located at
``src/Blogger/BlogBundle/Resources/config/routing.yml`` and append the following routing
rule.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_about:
        pattern:  /about
        defaults: { _controller: BloggerBlogBundle:Page:about }
        requirements:
            _method:  GET

The Controller
~~~~~~~~~~~~~~

Next open the ``Page`` controller located at
``src/Blogger/BlogBundle/Controller/PageController.php`` and add the action
to handle the about page.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        //  ..

        public function aboutAction()
        {
            return $this->render('BloggerBlogBundle:Page:about.html.twig');
        }
    }

The View
~~~~~~~~

For the view, create a new file located at
``src/Blogger/BlogBundle/Resources/views/Page/about.html.twig`` and copy in the
following content.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/about.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}About{% endblock%}

    {% block body %}
        <header>
            <h1>About symblog</h1>
        </header>
        <article>
            <p>Donec imperdiet ante sed diam consequat et dictum erat faucibus. Aliquam sit
            amet vehicula leo. Morbi urna dui, tempor ac posuere et, rutrum at dui.
            Curabitur neque quam, ultricies ut imperdiet id, ornare varius arcu. Ut congue
            urna sit amet tellus malesuada nec elementum risus molestie. Donec gravida
            tellus sed tortor adipiscing fringilla. Donec nulla mauris, mollis egestas
            condimentum laoreet, lacinia vel lorem. Morbi vitae justo sit amet felis
            vehicula commodo a placerat lacus. Mauris at est elit, nec vehicula urna. Duis a
            lacus nisl. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices
            posuere cubilia Curae.</p>
        </article>
    {% endblock %}

The about page is nothing spectacular. Its only action is to render a template file
with some dummy content. It does however bring us on to the next task.

Linking the pages
~~~~~~~~~~~~~~~~~

We now have the about page ready to go. Have a look at ``http://symblog.dev/app_dev.php/about``
to see this. As it stands there is no way for a user of your blog to view the about page,
short of typing in the full URL just like we did. As you'd expect Symfony2 provides both
sides to the routing equation. It can match routes as we have seen, and can also
generate URLs from these routes. You should always use the routing functions provided
by Symfony2. Never in your application should you be tempted to put the following.

.. code-block:: html+php

    <a href="/contact">Contact</a>

    <?php $this->redirect("/contact"); ?>

You may be wondering what's wrong with this approach, it may be the way you always
link your pages together. However, there are a number of problems with this approach.

1. It uses a hard link and ignores the Symfony2 routing system entirely. If you wanted to change
   the location of the contact page at any point you would have to find all references to the hard
   link and change them.
2. It will ignore your environment controllers. Environments is something we haven't really explained yet
   but you have been using them. The ``app_dev.php`` front controller provides us access to our application
   in the ``dev`` environment. If you were to replace the ``app_dev.php`` with ``app.php`` you will be
   running the application in the ``prod`` environment. The significance of these environments will
   be explained further in the tutorial but for now it's important to note that the hard link
   defined above does not maintain the current environment we are in as the front controller is
   not prepended to the URL.

The correct way to link pages together is with the ``path`` and ``url`` methods provided by Twig. They are
both very similar, except the ``url`` method will provide us with absolute URLs. Lets
update the main application template located at ``app/Resources/views/base.html.twig`` to link
to the about page and homepage together.

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    {% block navigation %}
        <nav>
            <ul class="navigation">
                <li><a href="{{ path('BloggerBlogBundle_homepage') }}">Home</a></li>
                <li><a href="{{ path('BloggerBlogBundle_about') }}">About</a></li>
                <li><a href="#">Contact</a></li>
            </ul>
        </nav>
    {% endblock %}

Now refresh your browser to see the Home and About page links working as expected. If you view the source
for the pages you will notice the link has been prefixed with ``/app_dev.php/``. This
is the front controller I was explaining above, and as you can see the use of ``path`` has maintained
it.

Finally lets update the logo links to redirect you back to the homepage. Update the
template located at ``app/Resources/views/base.html.twig``.

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    <hgroup>
        <h2>{% block blog_title %}<a href="{{ path('BloggerBlogBundle_homepage') }}">symblog</a>{% endblock %}</h2>
        <h3>{% block blog_tagline %}<a href="{{ path('BloggerBlogBundle_homepage') }}">creating a blog in Symfony2</a>{% endblock %}</h3>
    </hgroup>
    
Conclusion
----------

We have covered the basic areas with regards to a Symfony2 application including getting
the application configured and up and running. We have started to explore the fundamental concepts
behind a Symfony2 application, including Routing and the Twig templating engine.

Next we will look at creating the Contact page. This page is slightly more involved than the About page
as it allows users to interact with a web form to send us enquiries. The next chapter will introduce
concpets including Validators and Forms.
