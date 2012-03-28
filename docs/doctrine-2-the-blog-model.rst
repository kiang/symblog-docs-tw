[Part 3] - 關於 Blog Model： 使用 Doctrine 2 與資料裝置
=============================================================

概要
--------

這一章會開始探索 blog model，這個 model 會使用 `Doctrine 2 <http://www.doctrine-project.org/projects/orm>`_
物件關聯對映 (ORM) 實做，Doctrine 2 提供我們 PHP 物件的儲存，也提供了特有的 SQL 語法，我們稱之為
Doctrine 查詢語言 (DQL)。除了 Doctrine 2 之外，我們也會介紹測試資料的概念。資料裝置是一個技巧，用來在開發與測試
資料庫中產生適當的測試資料。這個章節完成後，你會得到一個定義好的 blog model 、更新過的資料庫來反映新的 model
以及建立一些資料裝置。你也會建立顯示部落格頁面的基礎。

Doctrine 2：關於 Model
---------------------

為了要讓我們的部落格功能運作，我們需要一個儲存資料的方式， Doctrine 2 提供一個專門處理這個用途的 ORM 函式庫。
Doctrine 2 ORM 基於一個強大的 `資料庫抽象層 <http://www.doctrine-project.org/projects/dbal>`_ 延伸，讓我們可以
透過 PHP PDO 達到資料儲存的抽象化，所以我們可以使用許多不同的儲存引擎，包含 MySQL 、 PostgreSQL 與 SQLite 。我們
會使用 MySQL 作為主要儲存引擎，但是可以輕易的替換為其他引擎。

.. tip::

    如果你對於 ORMs 不熟悉，我們針對一些基礎概念做介紹。
    來自 `Wikipedia <http://en.wikipedia.org/wiki/Object-relational_mapping>`_ 的定義：

    "物件關聯對映 (ORM, O/RM, and O/R mapping) 在電腦軟體領域是一個程式設計技巧，主要是為了將彼此不相容的系統轉換
    為物件導向程式語言，結果造成了一個 "虛擬物件資料庫" ，可以在程式設計時使用。"
    
    ORM 幫助我們將來自關聯資料庫（像是 MySQL）的資料轉換為 PHP 物件進行處理，讓我們可以將一個資料表所需要的功能封
    裝在一個物件中。舉例來說，一個使用者資料表可能包含像是帳號、密碼、姓氏、姓名、信箱與生日等欄位，在 ORM 中他們
    就會變成一個物件的屬性，讓我們可以執行像是 ``getUsername()`` 與 ``setPassword()`` 這樣的方法。 ORMs 其實還有
    比這更多的部份，它們也可以幫我們取得相關資料表的資料，可能是在取得使用者物件的同時，或是在晚點用到的時候。現在
    想想，我們的使用者可能有些相關的朋友，這可以是一個朋友資料表，儲存來自使用者資料表的主鍵，使用 ORM 我們可以執行
    像是 ``$user->getFriends()`` 方法來取得朋友資料表的資料。如果這樣還不夠，ORM 也可以處理資料的儲存，我們可以在
    PHP 建立物件，然後呼叫一個方法 ``save()`` ，讓 ORM 處理實際儲存資料到資料庫的細節。由於我們使用 Doctrine 2 ORM
    函式庫，你將會在這個教學進行的過程中越來越熟悉 ORM 是什麼。

.. note::

    由於這個教學使用 Doctrine 2 ORM 函式庫，你可以選擇改用 Doctrine 2 類別文件對映 (ODM) 函式庫，這個函式庫提供了
    許多實做，像是 `MongoDB <http://www.mongodb.org/>`_ 與 `CouchDB <http://couchdb.apache.org/>`_ 。參考
    `Doctrine 專案 <http://www.doctrine-project.org/projects>`_ 頁來取得更多資訊。

    在 `cookbook <http://symfony.com/doc/current/cookbook/doctrine/mongodb.html>`_ 也有一篇文章介紹如何在
    Symfony2 設定 ODM 。

Blog 實體
~~~~~~~~~~~~~~~

我們用建立 ``Blog`` 實體類別作為開始，在上一個章節我們在建立 ``Enquiry`` 實體時已經介紹過實體，一個實體的目的是保留
資料，正確的觀念是用一個實體來代表一筆 blog 資料。在建立一個實體時我們並沒有說資料會自動對應到資料庫，在 ``Enquiry``
實體可以看到，保留在實體的資料只是用來發送郵件給管理員。

建立一個新檔案在 ``src/Blogger/BlogBundle/Entity/Blog.php`` ，並且貼上下面內容：

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    namespace Blogger\BlogBundle\Entity;

    class Blog
    {
        protected $title;

        protected $author;

        protected $blog;

        protected $image;

        protected $tags;

        protected $comments;

        protected $created;

        protected $updated;
    }


如你所見，這是個簡單的 PHP 類別，它沒有從任何父項目延伸，也沒有存取方法，每個定義的屬性都是 protected ，所以我們無法在
操作這個類別的物件時存取它們，我們可以為這些屬性自己宣告取得與設定方法，不過 Doctrine 2 提供了一個工具可以作到，畢竟設
計存取方法並不是程式設計中最有趣的任務。

在執行這個工具前，我們需要提醒 Doctrine 2 ``Blog`` 實體應該要怎麼對映到資料庫，這些資訊會透過後設資料在 Doctrine 2 指
定，後設資料可以透過許多格式設定，包括 ``YAML`` 、 ``PHP`` 、 ``XML`` 與 ``Annotations`` ，我們在這個教學會使用
``Annotations`` 。需要特別注意的是，在實體中並非所有屬性都必須被儲存，所以我們不需要為那些屬性提供後設資料。這讓我們可
以彈性選擇我們要 Doctrine 2 對映到資料庫的屬性。用下面內容取代原本放在 ``src/Blogger/BlogBundle/Entity/Blog.php`` 的
 ``Blog`` 實體類別。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    namespace Blogger\BlogBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity
     * @ORM\Table(name="blog")
     */
    class Blog
    {
        /**
         * @ORM\Id
         * @ORM\Column(type="integer")
         * @ORM\GeneratedValue(strategy="AUTO")
         */
        protected $id;

        /**
         * @ORM\Column(type="string")
         */
        protected $title;

        /**
         * @ORM\Column(type="string", length=100)
         */
        protected $author;

        /**
         * @ORM\Column(type="text")
         */
        protected $blog;

        /**
         * @ORM\Column(type="string", length="20")
         */
        protected $image;

        /**
         * @ORM\Column(type="text")
         */
        protected $tags;

        protected $comments;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $created;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $updated;
    }

首先我們匯入與設定 Doctrine 2 ORM 對映命名空間，這讓我們可以使用 ``annotations`` 來描述實體的後設資料。這個後設資料提
供了屬性如何對映到資料庫的資訊。

.. tip::

    我們只有使用 Doctrine 2 對映類型的一小部份，完整的清單可以參考
    `對映類型 <http://www.doctrine-project.org/docs/orm/2.0/en/reference/basic-mapping.html#doctrine-mapping-types>`_
    ，這來自 Doctrine 2 網站。其他對映類型會在後面的教學介紹。

眼睛尖一點可以發現屬性 ``$comments`` 沒有附加後設資料，因為我們並不需要保存它，它只會提供一個部落格文章相關的評論集合，
在這裡可以不需要把資料庫放在心上，下面的程式碼會做個示範。

.. code-block:: php

    // Create a blog object.
    $blog = new Blog();
    $blog->setTitle("symblog - A Symfony2 Tutorial");
    $blog->setAuthor("dsyph3r");
    $blog->setBlog("symblog is a fully featured blogging website ...");

    // Create a comment and add it to our blog
    $comment = new Comment();
    $comment->setComment("Symfony2 rocks!");
    $blog->addComment($comment);

上面的程式碼展示在文章與評論類別常見的行為， ``$blog->addComment()`` 方法在程式內部可以這樣子實做：

.. code-block:: php

    class Blog
    {
        protected $comments = array();

        public function addComment(Comment $comment)
        {
            $this->comments[] = $comment;
        }
    }

 ``addComment`` 方法只是把一個新的評論加入到文章的 ``$comments`` 屬性，取得評論的方式也相當簡單。

.. code-block:: php

    class Blog
    {
        protected $comments = array();

        public function getComments()
        {
            return $this->comments;
        }
    }

如你所見， ``$comments`` 屬性只是一群 ``Comment`` 物件清單， Doctrine 2 不會異動這個邏輯，
Doctrine 2 會自動將 ``blog`` 類別有關的物件放入 ``$comments`` 屬性。

現在我們已經告訴 Doctrine 2 如何對映實體屬性，我們可以透過下面指令產生存取方法：

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger


你會注意到 ``Blog`` 實體會更新存取方法，每次我們異動實體類別的 ORM 後設資料，我們可以執行這個指令
產生其他的存取方法，這個指令不會異動實體中已經存在的方法，所以這個指令不會覆蓋舊有存取方法，重點在
於你接下來也許會客製一些預設存取方法。

.. tip::

    當我們在實體中使用 ``annotations`` ，我們可以透過 ``doctrine:mapping:convert`` 指令將對映資訊
    轉換為支援的格式，例如，下面的指令會將上面的實體轉換為 ``yaml`` 格式。

    .. code-block:: bash

        $ php app/console doctrine:mapping:convert --namespace="Blogger\BlogBundle\Entity\Blog" yaml src/Blogger/BlogBundle/Resources/config/doctrine

    這會建立一個檔案放在
    ``src/Blogger/BlogBundle/Resources/config/doctrine/Blogger.BlogBundle.Entity.Blog.orm.yml``
    ，這個檔案會包含 ``yaml`` 格式的 ``blog`` 實體對映。

關於資料庫
~~~~~~~~~~~~

建立資料庫
.....................

如果你看過這個教學的第一章，你應該已經用過網頁設定精靈來設定資料庫，如果沒有，可以直接調整
 ``app/config/parameters.ini`` 中的 ``database_*`` 選項。

現在可以用 Doctrine 2 來建立資料庫，這個指令只有建立資料庫，不會在資料庫中建立任何資料表。如果同樣
名稱的資料庫已經存在，這個指令會產生錯誤，不會去異動現有資料庫。

.. code-block:: bash

    $ php app/console doctrine:database:create

我們已經準備好在資料庫建立代表 ``Blog`` 實體的代表，我們有兩個方法可以作到，我們可以使用 Doctrine 2
schema 指令來更新資料庫，或是使用更強大的 Doctrine 2 migrations 。現在我們會使用 schema 指令，
Doctrine Migrations 會在後面章節介紹。

建立 blog 資料表
.......................

要在資料庫建立 blog 資料表可以執行下面 doctrine 指令。

.. code-block:: bash

    $ php app/console doctrine:schema:create

這會執行需要的 SQL 來產生 ``blog`` 實體的資料庫結構，你也可以在這個指令傳入選項 ``--dump-sql`` 以
輸出 SQL 取代在資料庫執行。如果檢查你的資料庫，應該可以看到已經建立了 blog 資料表，欄位如同我們所設定
的對映資訊。

.. tip::

    我們已經使用了一些 Symfony2 命令列指令，實際上命令列指令格式中都有提供說明，可以透過指定 ``--help``
    選項。例如想要看 ``doctrine:schema:create`` 指令的說明細節，可以這樣子執行

    .. code-block:: bash

        $ php app/console doctrine:schema:create --help

    說明資訊會輸出顯示用法與參數等，大部分的指令都有一些參數能夠用來客製執行的指令。

整合 Model 在 View 中，顯示一個部落格文章
---------------------------------------------------------

現在我們已經建立了 ``Blog`` 實體，資料庫也已經更新來反應這個異動，我們可以開始將 model 整合到 view 中，
我們要開始建立部落格的顯示頁。

顯示部落格的網址路徑
~~~~~~~~~~~~~~~~~~~

我們開始先為部落格的 ``show`` 方法建立一個網址路徑，一篇部落格可以透過唯一的 ID 做識別，所以這個 ID 必
需要出現在網址中。用下面的內容更新 ``BloggerBlogBundle`` 的網址路徑，檔案在
``src/Blogger/BlogBundle/Resources/config/routing.yml``

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_blog_show:
        pattern:  /{id}
        defaults: { _controller: BloggerBlogBundle:Blog:show }
        requirements:
            _method:  GET
            id: \d+

由於網址必須使用 ID ，我們指定了一個 ``id`` 替位符號，這表示像是 ``http://symblog.co.uk/1`` 與
``http://symblog.co.uk/my-blog`` 的網址會符合這個網址路徑，不過我們知道部落格的 ID 必須是數字（
在實體對映時是這樣子定義的），所以我們可以加入一個限制來指定這個網址路徑只在 ``id`` 包含一個數字
時符合，這是透過在網址路徑要求中設定 ``id: \d+`` 達到，這樣一來之前提到的網址就只有第一個符合，
``http://symblog.co.uk/my-blog`` 就不再符合這個網址路徑。你也會看到符合這個網址路徑會執行
``BloggerBlogBundle`` 中 ``Blog`` controller 的 ``show`` 方法，只是這個 controller 還沒建立。

顯示 Controller 方法
~~~~~~~~~~~~~~~~~~~~~~~~~~

Model 與 View 之間的介質就是 controller ，所以我們會在這兒建立顯示頁。我們可以將 ``show`` 方法加入到
已經存在的 ``Page`` controller ，不過因為這個頁面是用來顯示 ``blog`` 實體，將它放在自己的 ``Blog``
controller 會比較適合。

建立一個新檔案在 ``src/Blogger/BlogBundle/Controller/BlogController.php`` 並且貼入下面內容。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/BlogController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    /**
     * Blog controller.
     */
    class BlogController extends Controller
    {
        /**
         * Show a blog entry
         */
        public function showAction($id)
        {
            $em = $this->getDoctrine()->getEntityManager();

            $blog = $em->getRepository('BloggerBlogBundle:Blog')->find($id);

            if (!$blog) {
                throw $this->createNotFoundException('Unable to find Blog post.');
            }

            return $this->render('BloggerBlogBundle:Blog:show.html.twig', array(
                'blog'      => $blog,
            ));
        }
    }

我們已經為 ``Blog`` 實體建立一個新 Controller ，並且定義一個 ``show`` 方法，由於我們在網址路徑
``BloggerBlogBundle_blog_show`` 定義了一個 ``id`` 參數，它會被傳進 ``showAction`` 方法作為參數。
如果我們在網址路徑中指定了更多參數，他們會被視為獨立的參數傳入。

.. tip::

    如果你指定了 ``Symfony\Component\HttpFoundation\Request`` 作為參數，這個 controller 方法也會
    傳入該物件，這在處理表單的時候很實用，我們在第 2 章已經使用過一個表單，不過我們沒有使用這個方法，
    因為我們使用了一個來自 ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` 的輔助方法，
    像這樣：

    .. code-block:: php

        // src/Blogger/BlogBundle/Controller/PageController.php
        public function contactAction()
        {
            // ..
            $request = $this->getRequest();
        }

    我們可以將它改寫為這樣。

    .. code-block:: php

        // src/Blogger/BlogBundle/Controller/PageController.php

        use Symfony\Component\HttpFoundation\Request;

        public function contactAction(Request $request)
        {
            // ..
        }
    
    兩個方法都可以達成同樣任務，如果你的 controller 沒有繼承
    ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` 輔助類別，你就無法使用第一個方法。

接著我們需要從資料庫取得 ``Blog`` 實體，我們先用 ``Symfony\Bundle\FrameworkBundle\Controller\Controller``
的另外一個輔助方法來取得 Doctrine2 實體管理器，
`實體管理器 <http://www.doctrine-project.org/docs/orm/2.0/en/reference/working-with-objects.html>`_
是用來在表單與資料庫之間處理物件的取得與儲存，我們使用 ``EntityManager`` 物件為 ``BloggerBlogBundle:Blog``
取得 Doctrine2 ``Repository`` ，在這裡 Doctrine 2 可以指定縮寫語法來取代像是 ``Blogger\BlogBundle\Entity\Blog``
這樣的完整實體名稱，透過 repository 物件我們可以執行 ``find()`` 方法來傳入 ``$id`` 參數，這個方法會透過它的主鍵
取得對映的物件。

最後我們檢查發現找到一個實體，並且將這個實體傳遞給 view 。如果找不到實體就會丟出 ``createNotFoundException``
這個例外，這最後會產生一個 ``404 Not Found`` 回應。

.. tip::

    這個 repository 物件讓你可以存取許多實用的輔助方法，像是：

    .. code-block:: php

        // 傳回 'author' 符合 'dsyph3r' 的實體
        $em->getRepository('BloggerBlogBundle:Blog')->findBy(array('author' => 'dsyph3r'));

        // 傳回 'slug' 符合 'symblog-tutorial' 的一個實體
        $em->getRepository('BloggerBlogBundle:Blog')->findOneBySlug('symblog-tutorial');

    我們在下一個章節會建立我們自訂的 Repository 類別，這是在我們需要比較複雜的查詢時進行。

關於 View
~~~~~~~~

現在我們已經在 ``Blog`` controller 建立了 ``show`` 方法，我們可以專注在顯示 ``Blog`` 實體。由於在 ``show`` 方法
已經指定，所以會透過樣板 ``BloggerBlogBundle:Blog:show.html.twig`` 表示，我們來建立這個放在
 ``src/Blogger/BlogBundle/Resouces/views/Blog/show.html.twig`` 的樣板檔案，並且貼入下面內容。

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resouces/views/Blog/show.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}{{ blog.title }}{% endblock %}

    {% block body %}
        <article class="blog">
            <header>
                <div class="date"><time datetime="{{ blog.created|date('c') }}">{{ blog.created|date('l, F j, Y') }}</time></div>
                <h2>{{ blog.title }}</h2>
            </header>
            <img src="{{ asset(['images/', blog.image]|join) }}" alt="{{ blog.title }} image not found" class="large" />
            <div>
                <p>{{ blog.blog }}</p>
            </div>
        </article>
    {% endblock %}

如同你猜的，我們開始延伸 ``BloggerBlogBundle`` 的主要版面，接著我們用部落格標題覆寫頁面標題，這在進行 SEO 相當有
幫助，因為部落格的頁面標題會比已經設定的預設標題更具代表性。最後我們覆寫 body 區塊來輸出 ``Blog`` 實體內容。我們
在這裡再次使用 ``asset`` 方法來產生部落格圖片，這個部落格圖片應該會放在 ``web/images`` 資料夾。

CSS
...

為了要確保部落格顯示頁看起來美觀，我們需要加入一些風格。用下面內容更新位於
``src/Blogger/BlogBundle/Resouces/public/css/blog.css`` 的風格表。

.. code-block:: css

    .date { margin-bottom: 20px; border-bottom: 1px solid #ccc; font-size: 24px; color: #666; line-height: 30px }
    .blog { margin-bottom: 20px; }
    .blog img { width: 190px; float: left; padding: 5px; border: 1px solid #ccc; margin: 0 10px 10px 0; }
    .blog .meta { clear: left; margin-bottom: 20px; }
    .blog .snippet p.continue { margin-bottom: 0; text-align: right; }
    .blog .meta { font-style: italic; font-size: 12px; color: #666; }
    .blog .meta p { margin-bottom: 5px; line-height: 1.2em; }
    .blog img.large { width: 300px; min-height: 165px; }

.. note::
    如果你不是使用符號連結方式在 ``web`` 資料夾參照軟體包的資源，你現在會需要重新執行資源安裝指令來將異動過的 CSS
    複製過來。

    .. code-block:: bash

        $ php app/console assets:install web


我們現在已經建立了 ``show`` 方法的 controller 與 view ，現在可以看看這個顯示頁。將 ``http://symblog.dev/app_dev.php/1``
輸入瀏覽器的網址列，發現不是你預期的頁面？

.. image:: /_static/images/part_3/404_not_found.jpg
    :align: center
    :alt: Symfony2 404 Not Found Exception

Symfony2 產生了一個 ``404 Not Found`` 回應，這是因為資料庫沒有任何資料，所以找不到 ``id`` 等於 1 的實體。

你可以輕易的新增一筆資料到 blog 資料表，不過我們要使用一個更好的方式，資料裝置。

資料裝置
-------------

我們可以透過裝置來產生一些範例/測試資料進資料庫，我們透過 Doctrine Fixtures 外掛與軟體包做到。 Doctrine Fixtures
外掛與軟體包並沒有附在 Symfony2 標準版中，我們需要手動安裝。幸運的是，我們有個簡單的指令，開啟專案根目錄的檔案 deps
並且像下面這樣新增 Doctrine Fixtures 外掛與軟體包。

.. code-block:: text

    [doctrine-fixtures]
        git=http://github.com/doctrine/data-fixtures.git

    [DoctrineFixturesBundle]
        git=http://github.com/symfony/DoctrineFixturesBundle.git
        target=/bundles/Symfony/Bundle/DoctrineFixturesBundle

接著更新 vendors 來反應這些異動

.. code-block:: bash

    $ php bin/vendors install

這會從 Github 下載個別程式庫的最新版本，並且安裝在必要得位置。

.. note::

    如果你正在操作一個沒有安裝 Git 的機器，你會需要手動下載與安裝外掛與軟體包。

    doctrine-fixtures 外掛：從 GitHub `下載 <https://github.com/doctrine/data-fixtures>`_ 軟體目前版本並且解
    壓縮到 ``vendor/doctrine-fixtures`` 。

    DoctrineFixturesBundle: 從 GitHub `下載 <https://github.com/symfony/DoctrineFixturesBundle>`_ 軟體目前版
    本並且解壓縮到 ``vendor/bundles/Symfony/Bundle/DoctrineFixturesBundle`` 。

接著更新 ``app/autoloader.php`` 檔案來註冊新的命名空間，由於 DataFixtures 也在 ``Doctrine\Common`` 命名空間，
他們必須被放在現有 ``Doctrine\Common`` 上面，因為他們指定了一個新路徑。命名空間是從上而下檢查，所以比較多特定的
命名空間需要放在比較少特定的之前。

.. code-block:: php

    // app/autoloader.php
    // ...
    $loader->registerNamespaces(array(
    // ...
    'Doctrine\\Common\\DataFixtures'    => __DIR__.'/../vendor/doctrine-fixtures/lib',
    'Doctrine\\Common'                  => __DIR__.'/../vendor/doctrine-common/lib',
    // ...
    ));

接著在核心 ``app/AppKernel.php`` 註冊 ``DoctrineFixturesBundle`` 

.. code-block:: php

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Symfony\Bundle\DoctrineFixturesBundle\DoctrineFixturesBundle(),
            // ...
        );
        // ...
    }

部落格裝置
~~~~~~~~~~~~~

我們現在已經準備好為部落格定義一些裝置，在 ``src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php`` 建立一個
裝置檔案，並且放入下面內容：and add the following content:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php
    
    namespace Blogger\BlogBundle\DataFixtures\ORM;
    
    use Doctrine\Common\DataFixtures\FixtureInterface;
    use Doctrine\Common\Persistence\ObjectManager;
    use Blogger\BlogBundle\Entity\Blog;
    
    class BlogFixtures implements FixtureInterface
    {
        public function load(ObjectManager $manager)
        {
            $blog1 = new Blog();
            $blog1->setTitle('A day with Symfony2');
            $blog1->setBlog('Lorem ipsum dolor sit amet, consectetur adipiscing eletra electrify denim vel ports.\nLorem ipsum dolor sit amet, consectetur adipiscing elit. Morbi ut velocity magna. Etiam vehicula nunc non leo hendrerit commodo. Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque. Nulla consectetur tempus nisl vitae viverra. Cras el mauris eget erat congue dapibus imperdiet justo scelerisque. Nulla consectetur tempus nisl vitae viverra. Cras elementum molestie vestibulum. Morbi id quam nisl. Praesent hendrerit, orci sed elementum lobortis, justo mauris lacinia libero, non facilisis purus ipsum non mi. Aliquam sollicitudin, augue id vestibulum iaculis, sem lectus convallis nunc, vel scelerisque lorem tortor ac nunc. Donec pharetra eleifend enim vel porta.');
            $blog1->setImage('beach.jpg');
            $blog1->setAuthor('dsyph3r');
            $blog1->setTags('symfony2, php, paradise, symblog');
            $blog1->setCreated(new \DateTime());
            $blog1->setUpdated($blog1->getCreated());
            $manager->persist($blog1);
    
            $blog2 = new Blog();
            $blog2->setTitle('The pool on the roof must have a leak');
            $blog2->setBlog('Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque. Na. Cras elementum molestie vestibulum. Morbi id quam nisl. Praesent hendrerit, orci sed elementum lobortis.');
            $blog2->setImage('pool_leak.jpg');
            $blog2->setAuthor('Zero Cool');
            $blog2->setTags('pool, leaky, hacked, movie, hacking, symblog');
            $blog2->setCreated(new \DateTime("2011-07-23 06:12:33"));
            $blog2->setUpdated($blog2->getCreated());
            $manager->persist($blog2);
    
            $blog3 = new Blog();
            $blog3->setTitle('Misdirection. What the eyes see and the ears hear, the mind believes');
            $blog3->setBlog('Lorem ipsumvehicula nunc non leo hendrerit commodo. Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque.');
            $blog3->setImage('misdirection.jpg');
            $blog3->setAuthor('Gabriel');
            $blog3->setTags('misdirection, magic, movie, hacking, symblog');
            $blog3->setCreated(new \DateTime("2011-07-16 16:14:06"));
            $blog3->setUpdated($blog3->getCreated());
            $manager->persist($blog3);
    
            $blog4 = new Blog();
            $blog4->setTitle('The grid - A digital frontier');
            $blog4->setBlog('Lorem commodo. Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque. Nulla consectetur tempus nisl vitae viverra.');
            $blog4->setImage('the_grid.jpg');
            $blog4->setAuthor('Kevin Flynn');
            $blog4->setTags('grid, daftpunk, movie, symblog');
            $blog4->setCreated(new \DateTime("2011-06-02 18:54:12"));
            $blog4->setUpdated($blog4->getCreated());
            $manager->persist($blog4);
    
            $blog5 = new Blog();
            $blog5->setTitle('You\'re either a one or a zero. Alive or dead');
            $blog5->setBlog('Lorem ipsum dolor sit amet, consectetur adipiscing elittibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque.');
            $blog5->setImage('one_or_zero.jpg');
            $blog5->setAuthor('Gary Winston');
            $blog5->setTags('binary, one, zero, alive, dead, !trusting, movie, symblog');
            $blog5->setCreated(new \DateTime("2011-04-25 15:34:18"));
            $blog5->setUpdated($blog5->getCreated());
            $manager->persist($blog5);
    
            $manager->flush();
        }
    
    }

這個裝置檔案展示一些使用 Doctrine 2 的重要功能，包含如何儲存資料到資料庫。

看看我們怎麼建立一篇部落格。

.. code-block:: php

    $blog1 = new Blog();
    $blog1->setTitle('A day in paradise - A day with Symfony2');
    $blog1->setBlog('Lorem ipsum dolor sit d us imperdiet justo scelerisque. Nulla consectetur...');
    $blog1->setImage('beach.jpg');
    $blog1->setAuthor('dsyph3r');
    $blog1->setTags('symfony2, php, paradise, symblog');
    $blog1->setCreated(new \DateTime());
    $blog1->setUpdated($this->getCreated());
    $manager->persist($blog1);
    // ..

    $manager->flush();

我們先建立一個 ``Blog`` 物件並且為它的屬性設定一些數值，在這裡 Doctrine 2 並不知道 ``Entity`` 物件，只有在我們執行
了 ``$manager->persist($blog1)`` 時我們才指示 Doctrine 2 開始管理這個實體物件，而 ``$manager`` 物件是之前在從資料
庫取得資料時看到的 ``EntityManager`` 物件實例。需要注意的是，當 Doctrine 2 知道了實體物件，它還沒儲存到資料庫中，需要呼
叫 ``$manager->flush()`` 才可以，這個方法讓 Doctrine 2 實際與資料庫互動以及操作管理中的所有實體。為了要取得比較好的效能
，你應該要把 Doctrine 2 的操作集中，並且一次執行所有操作，這就是我們在裝置中進行的方式，我們建立個別實體、要求 Doctrine 2
去管理它，接著最後執行所有操作。

.. tip:

    你也許已經注意到 ``created`` 與 ``updated`` 屬性的設定，這並不是一個設定這些欄位的理想方法，因為你會愈其他們在物件建
    立或更新時自動設定， Doctrine 2 提供一個方法幫我們做到，我們稍候會看看這個部份。

載入裝置
~~~~~~~~~~~~~~~~~~~~

我們現在已經準備好要載入裝置到資料庫中。

.. code-block:: bash

    $ php app/console doctrine:fixtures:load

如果我們看一下 ``http://symblog.dev/app_dev.php/1`` 顯示頁，你應該可以看到一篇部落格文章。

.. image:: /_static/images/part_3/blog_show.jpg
    :align: center
    :alt: The symblog blog show page

試著修改網址中的參數 ``id`` 為 2 ，你應該會看到它顯示下一篇部落格文章。

如果你輸入網址 ``http://symblog.dev/app_dev.php/100`` ，你應該會看到它丟出一個 ``404 Not Found`` 例外，你應該會希望這個
訊息換成找不到 ID 為 100 的文章。現在試試網址 ``http://symblog.dev/app_dev.php/symfony2-blog`` ，為什麼我們不會得到一個
``404 Not Found`` 例外？這是因為 ``show`` 方法並沒有執行，這個網址無法符合應用中的任何網址路徑，因為我們在網址路徑
``BloggerBlogBundle_blog_show`` 設定了要求為 ``\d+`` ，這是為什麼你會看到一個 ``No route found for "GET /symfony2-blog"``
例外。

時間標記
----------

在這一章最後我們會看到 ``Blog`` 的兩個時間標記屬性 ``created`` 與 ``updated`` ，這兩個屬性的功能常常涉及到 ``Timestampable``
行為，這些屬性保留了部落格建立與最後更新的時間，由於我們不想要每次建立一篇部落格就手動設定這些欄位，我們可以透過 Doctrine 2
協助。

Doctrine 2 提供了一個 `事件系統 <http://www.doctrine-project.org/docs/orm/2.0/en/reference/events.html>`_ ，提供
`生命週期回叫 <http://www.doctrine-project.org/docs/orm/2.0/en/reference/events.html#lifecycle-callbacks>`_ 。我們可以使用
這些回叫事件來註冊我們的實體在生命週期中的事件被提醒，一些我們可以被提醒的事件範例，包含在更新發生前、在一個儲存發生後以及在一個
移除發生後，我們需要為他們註冊實體來使用生命週期回叫功能，這可以透過使用實體的後設資料完成，透過下面內容更新位於
``src/Blogger/BlogBundle/Entity/Blog.php`` 的 ``Blog`` 實體。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    // ..

    /**
     * @ORM\Entity
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..
    }

現在在 ``Blog`` 實體加入一個方法來註冊 ``preUpdate`` 事件，我們也新增一個建構子來設定 ``created`` 與 ``updated`` 屬性的預設值。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    // ..

    /**
     * @ORM\Entity
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..

        public function __construct()
        {
            $this->setCreated(new \DateTime());
            $this->setUpdated(new \DateTime());
        }

        /**
         * @ORM\preUpdate
         */
        public function setUpdatedValue()
        {
           $this->setUpdated(new \DateTime());
        }

        // ..
    }

我們為 ``Blog`` 註冊在 ``preUpdate`` 事件發生時被提醒來設定 ``updated`` 屬性值，現在你可以重新執行載入裝置指令，你會注意到
``created`` 與 ``updated`` 屬性會自動設定。

.. tip::

    由於時間標記屬性是常見的實體要求，有一個軟體包提供這個功能，
    `StofDoctrineExtensionsBundle <https://github.com/stof/StofDoctrineExtensionsBundle>`_
    提供了一些實用的 Doctrine 2 外掛，包含 Timestampable 、 Sluggable 與 Sortable 。

    我們會試著在這個教學後面整合這個軟體包，有心的朋友可以先看看
    `cookbook <http://symfony.com/doc/current/cookbook/doctrine/common_extensions.html>`_
    關於這個主題的章節。

結論
----------

我們已經提到一些處理 Doctrine 2 models 的概念，我們也看到定義資料裝置讓我們在開發與測試時可以應用輕易取得適當的測試資料。

接著我們會試著透過加入評論實體來延伸更多 model 功能，我們會開始建置首頁與建立一個自訂 Repository 來達到，我們也會介紹
Doctrine Migrations 的概念與表單如何與 Doctrine 2 互動來讓評論發表到一篇部落格中。
