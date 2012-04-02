[Part 4] - 關於 Comments Model：新增評論、Doctrine資料庫與搬遷
=====================================================================================

概要
--------

這一章會基於上一章定義的部落格 model 做延伸，我們會建立評論 model ，用來處理部落格文章的評論。
我們會介紹與建立兩個 models 之間的關聯，通常一篇部落格文章可以包含多篇評論。我們會使用 Doctrine 2
查詢精靈與 Doctrine 2 資料庫類別從資料庫取得資料。我們也會嘗試使用 Doctrine 2 的搬遷功能，這個功能
提供一個程式化的方式來佈署資料庫異動。在這一章結束後，你會完成與部落格 model 連結在一起的評論 model
，我們也會建立首頁與提供使用者發表評論到部落格文章的功能。


關於首頁
------------

我們先開始建立首頁，一般部落格流行在這裡顯示每篇部落格文章的摘要，從最新的排到最舊的，完整的部落格
文章會在點選連結後進入部落格展示頁。由於我們已經建立首頁的網址路徑、 controller 與 view ，我們只要
簡單的更新它就好。

取得部落格文章：查詢 model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

為了要顯示部落格文章，我們需要從資料庫取得他們，Doctrine 2 提供了
`Doctrine 查詢語言 <http://www.doctrine-project.org/docs/orm/2.1/en/reference/dql-doctrine-query-language.html>`_
(DQL) 以及一個
`查詢精靈 <http://www.doctrine-project.org/docs/orm/2.1/en/reference/query-builder.html>`_
來達成這個目的（你也可以在 Doctrine 2 直接使用原始的 SQL ，不過不建議使用這個方法，因為這樣就無法
享受 Doctrine 2 給我們的資料抽象化功能）。我們會使用 ``QueryBuilder`` ，因為它提供一個物件導向方式
給我們去產生 DQL ，我們可以用來查詢資料庫。現在更新位於 ``src/Blogger/BlogBundle/Controller/PageController.php``
``Page`` controller 中的 ``index`` 方法來從資料庫取得部落格文章。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        public function indexAction()
        {
            $em = $this->getDoctrine()
                       ->getEntityManager();
    
            $blogs = $em->createQueryBuilder()
                        ->select('b')
                        ->from('BloggerBlogBundle:Blog',  'b')
                        ->addOrderBy('b.created', 'DESC')
                        ->getQuery()
                        ->getResult();
    
            return $this->render('BloggerBlogBundle:Page:index.html.twig', array(
                'blogs' => $blogs
            ));
        }
        
        // ..
    }

我們從 ``EntityManager`` 取得一個 ``QueryBuilder`` 實體開始，這讓我們透過 ``QueryBuilder`` 提供的
許多方法開始建立查詢。完整的可用方法可以參考 ``QueryBuilder`` 的手冊，建議可以從
`輔助方法 <http://www.doctrine-project.org/docs/orm/2.1/en/reference/query-builder.html#helper-methods>`_.
開始。我們使用了包括 ``select()`` 、 ``from()`` 與 ``addOrderBy()`` 方法，就之前與 Doctrine 2 互動
的經驗，我們可以使用縮寫方式 ``BloggerBlogBundle:Blog`` 來參照 ``Blog`` 實體（需要記住是這跟
``Blogger\BlogBundle\Entity\Blog`` 一樣意思）。當我們完成指定查詢條件，我們呼叫 ``getQuery()`` 方法
來傳回一個 ``DQL`` 實例。我們還無法從 ``QueryBuilder`` 物件取得結果，我們一定要先將它轉換為一個 ``DQL``
實例， ``DQL`` 實例提供了 ``getResult()`` 方法來傳回 ``Blog`` 實體的集合。稍候我們會看到 ``DQL`` 實例
提供了 `許多方法 <http://www.doctrine-project.org/docs/orm/2.1/en/reference/dql-doctrine-query-language.html#query-result-formats>`_
來取得結果，包含 ``getSingleResult()`` 與 ``getArrayResult()`` 。

關於 View
........

現在我們有一些 ``Blog`` 資料，我們需要顯示他們。用下面內容去取代 ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig``
的首頁樣板

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        {% for blog in blogs %}
            <article class="blog">
                <div class="date"><time datetime="{{ blog.created|date('c') }}">{{ blog.created|date('l, F j, Y') }}</time></div>
                <header>
                    <h2><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}">{{ blog.title }}</a></h2>
                </header>
        
                <img src="{{ asset(['images/', blog.image]|join) }}" />
                <div class="snippet">
                    <p>{{ blog.blog(500) }}</p>
                    <p class="continue"><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}">Continue reading...</a></p>
                </div>
        
                <footer class="meta">
                    <p>Comments: -</p>
                    <p>Posted by <span class="highlight">{{blog.author}}</span> at {{ blog.created|date('h:iA') }}</p>
                    <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
                </footer>
            </article>
        {% else %}
            <p>There are no blog entries for symblog</p>
        {% endfor %}
    {% endblock %}

我們在這裡加入了一個 Twig 控制結構 ``for..else..endfor`` ，如果你過去沒使用過樣板引擎，你也應該熟悉下面這樣的程式碼：

.. code-block:: php

    <?php if (count($blogs)): ?>
        <?php foreach ($blogs as $blog): ?>
            <h1><?php echo $blog->getTitle() ?><?h1>
            <!-- rest of content -->
        <?php endforeach ?>
    <?php else: ?>
        <p>There are no blog entries</p>
    <?php endif ?>

Twig 的 ``for..else..endfor`` 控制結構可以更簡潔的達成這個任務，在首頁樣板中大部分程式碼都是關於以 HTML 格式輸出部落格
資訊，不過有些事情我們需要留意。首先，我們使用了 Twig 的 ``path`` 函式來產生顯示部落格的網址，由於顯示頁面網址會需要部落格
的 ``ID`` ，我們需要將它傳給 ``path`` 函式作為參數，就像這樣：

.. code-block:: html
    
    <h2><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}">{{ blog.title }}</a></h2>
    
接著我們用 ``<p>{{ blog.blog(500) }}</p>`` 輸出部落格內容，傳入的參數 ``500`` 是我們希望從函式接收到的部落格文章最大長度，
要讓這個功能運作，我們需要更新 Doctrine 2 之前為我們產生的 ``getBlog`` 方法，它被放在 ``src/Blogger/BlogBundle/Entity/Blog.php``
的 ``Blog`` 實體中。
.

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php
    public function getBlog($length = null)
    {
        if (false === is_null($length) && $length > 0)
            return substr($this->blog, 0, $length);
        else
            return $this->blog;
    }

由於一般 ``getBlog`` 行為應該會傳回整篇部落格文章，我們將 ``$length`` 參數預設值設定為 ``null`` ，如果傳入 ``null`` 就會
傳回整篇部落格文章。

如果你用瀏覽器打開 ``http://symblog.dev/app_dev.php/`` ，你應該會看到顯示最新部落格文章的首頁，也應該能夠點選個別文章的
標題或 'continue reading...' 連結來檢視整篇文章。

.. image:: /_static/images/part_4/homepage.jpg
    :align: center
    :alt: symblog homepage

雖然我們可以直接在 controller 中查詢資料，但那並不是最好的方式，基於下面幾個理由，我們最好將查詢工作放到 controller 以外的
地方。

    1. 我們在應用程式裡會無法在其他地方重複使用這個查詢，除非要複製整個 ``QueryBuilder`` 的程式碼。
    2. 如果我們複製了 ``QueryBuilder`` 程式碼，我們會需要在未來查詢需求改變時做多個修改。
    3. 將查詢與 controller 分開可以讓我們獨立測試查詢。

Doctrine 2 提供了資料庫類別來協助這個部份。

Doctrine 2 資料庫
-----------------------

我們在之前建立部落格顯示頁的章節已經介紹過 Doctrine 2 資料庫類別，我們用 ``Doctrine\ORM\EntityRepository`` 類別預設版本的
 ``find()`` 方法來從資料庫取得資料。由於我們想要建立一個自訂查詢，我們需要建立一個自訂的資料庫類別， Doctrine 2 可以在這裡
提供一些幫助。更新放在 ``src/Blogger/BlogBundle/Entity/Blog.php`` 的 ``Blog`` 實體後設資料。


.. code-block:: php
    
    // src/Blogger/BlogBundle/Entity/Blog.php
    /**
     * @ORM\Entity(repositoryClass="Blogger\BlogBundle\Repository\BlogRepository")
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..
    }

你可以看到我們在這個實體的關聯位置指定了 ``BlogRepository`` 類別的命名空間位置，在更新了 ``Blog`` 實體在 Doctrine 2 的後設
資料後，我們需要像下面這樣傳回 ``doctrine:generate:entities`` 的結果。

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger
    
Doctrine 2 會在 ``src/Blogger/BlogBundle/Repository/BlogRepository.php`` 建立 ``BlogRepository`` 的殼類別。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Repository/BlogRepository.php
    
    namespace Blogger\BlogBundle\Repository;

    use Doctrine\ORM\EntityRepository;

    /**
     * BlogRepository
     *
     * This class was generated by the Doctrine ORM. Add your own custom
     * repository methods below.
     */
    class BlogRepository extends EntityRepository
    {

    }

這個 ``BlogRepository`` 類別繼承了  ``EntityRepository`` 類別，藉此提供之前使用的 ``find()`` 方法。我們可以更新
``BlogRepository`` 類別，將 ``QueryBuilder`` 程式碼從 ``Page`` controller 移動到這裡。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Repository/BlogRepository.php

    namespace Blogger\BlogBundle\Repository;

    use Doctrine\ORM\EntityRepository;

    /**
     * BlogRepository
     *
     * This class was generated by the Doctrine ORM. Add your own custom
     * repository methods below.
     */
    class BlogRepository extends EntityRepository
    {
        public function getLatestBlogs($limit = null)
        {
            $qb = $this->createQueryBuilder('b')
                       ->select('b')
                       ->addOrderBy('b.created', 'DESC');

            if (false === is_null($limit))
                $qb->setMaxResults($limit);

            return $qb->getQuery()
                      ->getResult();
        }
    }

我們已經建立了 ``getLatestBlogs`` 方法來傳回最新的部落格文章，就像是在 controller 中使用的 ``QueryBuilder``
程式碼。在資料庫類別我們透過 ``createQueryBuilder()`` 方法直接存取 ``QueryBuilder`` ，我們也加入一個預設的參數
 ``$limit`` ，藉此限制傳回的資料數量。查詢的結果跟在 controller 中沒有兩樣。你也許注意到我們不需要透過 ``from()``
方法來指定要使用的實體，因為我們是在 ``BlogRepository`` 中操作，它已經與 ``Blog`` 產生關聯。如果我們看到
``EntityRepository`` 類別中的 ``createQueryBuilder`` 方法實做方式，我們可以看到它幫我們呼叫了 ``from()`` 方法。

.. code-block:: php
    
    // Doctrine\ORM\EntityRepository
    public function createQueryBuilder($alias)
    {
        return $this->_em->createQueryBuilder()
            ->select($alias)
            ->from($this->_entityName, $alias);
    }

最後我們更新 ``Page`` controller 的 ``index`` 方法來使用 ``BlogRepository`` 。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        public function indexAction()
        {
            $em = $this->getDoctrine()
                       ->getEntityManager();
                       
            $blogs = $em->getRepository('BloggerBlogBundle:Blog')
                        ->getLatestBlogs();
                       
            return $this->render('BloggerBlogBundle:Page:index.html.twig', array(
                'blogs' => $blogs
            ));
        }
        
        // ..
    }

現在當你重新整理首頁應該會看到跟之前顯示的沒兩樣，我們所做的只是重構我們的程式碼，讓正確的類別執行正確的工作。

更多關於 Model 的部份：建立評論實體
----------------------------------------------

文章在部落格這股風潮只佔了一半的重要性，我們還需要讓讀者能夠評論文章，這些文章需要被保留與連結 ``Blog`` 實體，
因為一篇文章可以包含多個評論。

我們開始定義 ``Comment`` 實體類別的基礎，建立一個檔案在 ``src/Blogger/BlogBundle/Entity/Comment.php`` 並且
放入下面內容：

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Comment.php

    namespace Blogger\BlogBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity(repositoryClass="Blogger\BlogBundle\Repository\CommentRepository")
     * @ORM\Table(name="comment")
     * @ORM\HasLifecycleCallbacks()
     */
    class Comment
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
        protected $user;

        /**
         * @ORM\Column(type="text")
         */
        protected $comment;

        /**
         * @ORM\Column(type="boolean")
         */
        protected $approved;
        
        /**
         * @ORM\ManyToOne(targetEntity="Blog", inversedBy="comments")
         * @ORM\JoinColumn(name="blog_id", referencedColumnName="id")
         */
        protected $blog;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $created;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $updated;

        public function __construct()
        {
            $this->setCreated(new \DateTime());
            $this->setUpdated(new \DateTime());
            
            $this->setApproved(true);
        }

        /**
         * @ORM\preUpdate
         */
        public function setUpdatedValue()
        {
           $this->setUpdated(new \DateTime());
        }
    }

在這裡看到大部分的程式碼在上一個章節都提過，不過我們這裡使用後設資料去設定連結 ``Blog`` 實體。由於一個評論只會
針對一篇文章，我們設定 ``Comment`` 實體的連結屬於  ``Blog`` 實體，我們以指定一個 ``ManyToOne`` 連結對象為
``Blog`` 實體，以及相反的連結可以透過 ``comments`` 存取。要建立相反的連結，我們需要更新 ``Blog`` 實體，讓
Doctrine 2 知道一篇文章可以包含許多評論，所以更新 ``src/Blogger/BlogBundle/Entity/Blog.php`` 的 ``Blog``
實體來加入下面對映。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    namespace Blogger\BlogBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Doctrine\Common\Collections\ArrayCollection;

    /**
     * @ORM\Entity(repositoryClass="Blogger\BlogBundle\Repository\BlogRepository")
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..
        
        /**
         * @ORM\OneToMany(targetEntity="Comment", mappedBy="blog")
         */
        protected $comments;
        
        // ..
        
        public function __construct()
        {
            $this->comments = new ArrayCollection();
            
            $this->setCreated(new \DateTime());
            $this->setUpdated(new \DateTime());
        }
        
        // ..
    }

在這裡有一些異動需要說明，首先我們加入了後設資料到屬性 ``$comments`` ，記得在上一個章節我們在這個屬性沒有加入
任何後設資料，因為我們不希望 Doctrine 2 保留它。這還是一樣，只是我們想要 Doctrine 2 能夠將相關的 ``Comment``
資料放入這個屬性，這就是後設資料的目的。其次， Doctrine 2 要求我們 ``$comments`` 屬性預設必須是一個
``ArrayCollection`` 物件，我們在 ``constructor`` 進行。也需要注意在 ``use`` 語法中匯入了 ``ArrayCollection``
類別。

我們現在已經建立了 ``Comment`` 實體、更新了 ``Blog`` 實體，接著我們讓 Doctrine 2 產生存取器，像之前一樣執行下面
Doctrine 2 的指令就可以。

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger
    
現在兩個實體應該都有最新、正確的存取器方法，你也會注意到多了 ``src/Blogger/BlogBundle/Repository/CommentRepository.php``
這個 ``CommentRepository`` 類別，如同我們在後設資料所指定的。

最後我們需要更新資料庫來反應這些實體的異動，我們可以接著執行 ``doctrine:schema:update`` 指令來做到，不過這裡我們
要介紹 Doctrine 2 搬遷 。

.. code-block:: bash

    $ php app/console doctrine:schema:update --force

Doctrine 2 搬遷
-------------------

Doctrine 2 搬遷外掛與軟體包並不存在於 Symfony2 標準版本，我們需要像是之前處理資料裝置外掛與軟體包一樣手動安裝它們
，請打開放在專案根目錄的檔案 ``deps`` ，並且像下面這樣新增 Doctrine 2 搬遷外掛與軟體包。

.. code-block:: text
    
    [doctrine-migrations]
        git=http://github.com/doctrine/migrations.git

    [DoctrineMigrationsBundle]
        git=http://github.com/symfony/DoctrineMigrationsBundle.git
        target=/bundles/Symfony/Bundle/DoctrineMigrationsBundle

接著更新 vendors to 來反應這些異動。

.. code-block:: bash

    $ php bin/vendors install

這會從 Github 下載每個函式庫的最新版本並且安裝到需要的位置。

.. note::

    如果你使用的電腦沒有安裝 Git ，你會需要手動下載與安裝這個外掛與軟體包。

    doctrine-migrations 外掛：從 GitHub `下載 <http://github.com/doctrine/migrations>`_ 目前版本並且解壓縮到
    ``vendor/doctrine-migrations`` 。

    DoctrineMigrationsBundle: 從 GitHub `下載 <http://github.com/symfony/DoctrineMigrationsBundle>`_ 目前版本
    並且解壓縮到 ``vendor/bundles/Symfony/Bundle/DoctrineMigrationsBundle`` 。

接著更新檔案 ``app/autoload.php`` 來註冊新的命名空間，由於 Doctrine 2 搬遷也在 ``Doctrine\DBAL`` 命名空間，他們
必須被放在既有的 ``Doctrine\DBAL`` 設定，因為他們指定一個新的路徑。命名空間是由上而下檢查，所以特定的命名空間需要
在非特定的之前註冊。

.. code-block:: php

    // app/autoload.php
    // ...
    $loader->registerNamespaces(array(
    // ...
    'Doctrine\\DBAL\\Migrations' => __DIR__.'/../vendor/doctrine-migrations/lib',
    'Doctrine\\DBAL'             => __DIR__.'/../vendor/doctrine-dbal/lib',
    // ...
    ));

接著在核心檔案 ``app/AppKernel.php`` 註冊這個軟體包。

.. code-block:: php

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Symfony\Bundle\DoctrineMigrationsBundle\DoctrineMigrationsBundle(),
            // ...
        );
        // ...
    }

.. warning::

    Doctrine 2 搬遷函式庫還在開發階段，所以現在還不建議將它用在正式主機上。

我們現在已經準備好更新資料庫來反應這些實體異動，這個過程有兩個步驟，第一我們需要讓 Doctrine 2 搬遷來比對實體與目前
資料庫結構之間的差異，這是透過指令 ``doctrine:migrations:diff`` 完成。接著我們需要基於上一步的異動細節來執行搬遷
操作，這是透過 ``doctrine:migrations:migrate`` 指令。

執行下面兩個指令來更新資料庫結構。

.. code-block:: bash

    $ php app/console doctrine:migrations:diff
    $ php app/console doctrine:migrations:migrate

你的資料庫現在會同步最新的實體異動以及加入新的評論資料表。

.. note::

    你也會發現資料庫多了一個新的資料表 ``migration_versions`` ，它保存了搬遷的版本編號，讓搬遷指令可以知道目前資料
    庫的版本為何。
    
.. tip::

    Doctrine 2 搬遷用來更新正式資料庫非常方便，因為它可以透過程式化的方式進行，這表示我們可以將這個指令整合到佈署程
    式中，這樣一來我們在佈署應用程式的新版本時就可以自動更新資料庫。 Doctrine 2 搬遷也允許我們還原異動，因為每個建立
    的搬遷都有一個 ``up`` 與 ``down`` 方法，要還原到上個版本需要像下面這樣指定希望還原到哪個版本。
    
    .. code-block:: bash
    
        $ php app/console doctrine:migrations:migrate 20110806183439
        
資料裝置：再次了解
-------------------------

現在我們已經建立了 ``Comment`` 實體，接著為它建立一些裝置，每次建立一個實體之後就加入一些裝置是個不錯的習慣。我們知道
一個評論必須有一個相關的 ``Blog`` 實體，因為後設資料裡面是這樣設定的，不過建立 ``Comment`` 實體的裝置時，我們會需要指
定 ``Blog`` 實體，這樣子我們就可以直接更新這個檔案來加入 ``Comment`` 資料。現在也許還容易控制，不過如果我們後面開始加
入會員、文章類別與完整功能的其他實體到我們的軟體包，比較建議為 ``Comment`` 實體裝置建立一個新檔案，這個方法的問題會出
在我們如何從文章裝置中存取 ``Blog`` 資料。

幸運的是，我們可以輕易做到，只要在一個裝置檔案設定參照到其他物件，讓其他裝置可以存取。用下面內容更新放在
``src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php`` 的 ``Blog`` 實體裝置 ``DataFixtures`` 。這個異動需要
注意的第方式 ``AbstractFixture`` 的延伸與 ``OrderedFixtureInterface`` 的實做，也要注意匯入那些類別的兩個新 use 語法。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php

    namespace Blogger\BlogBundle\DataFixtures\ORM;

    use Doctrine\Common\DataFixtures\AbstractFixture;
    use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
    use Doctrine\Common\Persistence\ObjectManager;
    use Blogger\BlogBundle\Entity\Blog;

    class BlogFixtures extends AbstractFixture implements OrderedFixtureInterface
    {
        public function load(ObjectManager $manager)
        {
            // ..

            $manager->flush();

            $this->addReference('blog-1', $blog1);
            $this->addReference('blog-2', $blog2);
            $this->addReference('blog-3', $blog3);
            $this->addReference('blog-4', $blog4);
            $this->addReference('blog-5', $blog5);
        }

        public function getOrder()
        {
            return 1;
        }
    }

我們用 ``addReference()`` 方法來新增參照到文章，第一個參數是一個參照識別字元，我們可以用它在後面取得對應物件。最後我們
必須實做 ``getOrder()`` 方法來指定裝置的載入順序，文章必須在評論之前載入，所以我們傳回 1 。

評論裝置
~~~~~~~~~~~~~~~~

我們現在已經準備好為 ``Comment`` 實體定義一些裝置，建立一個裝置檔案到 ``src/Blogger/BlogBundle/DataFixtures/ORM/CommentFixtures.php``
並且放入下面內容：

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/DataFixtures/ORM/CommentFixtures.php
    
    namespace Blogger\BlogBundle\DataFixtures\ORM;
    
    use Doctrine\Common\DataFixtures\AbstractFixture;
    use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
    use Doctrine\Common\Persistence\ObjectManager;
    use Blogger\BlogBundle\Entity\Comment;
    use Blogger\BlogBundle\Entity\Blog;
    
    class CommentFixtures extends AbstractFixture implements OrderedFixtureInterface
    {
        public function load(ObjectManager $manager)
        {
            $comment = new Comment();
            $comment->setUser('symfony');
            $comment->setComment('To make a long story short. You can\'t go wrong by choosing Symfony! And no one has ever been fired for using Symfony.');
            $comment->setBlog($manager->merge($this->getReference('blog-1')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('David');
            $comment->setComment('To make a long story short. Choosing a framework must not be taken lightly; it is a long-term commitment. Make sure that you make the right selection!');
            $comment->setBlog($manager->merge($this->getReference('blog-1')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Anything else, mom? You want me to mow the lawn? Oops! I forgot, New York, No grass.');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('Are you challenging me? ');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:15:20"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Name your stakes.');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:18:35"));
            $manager->persist($comment);
            
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('If I win, you become my slave.');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:22:53"));
            $manager->persist($comment);
            
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Your SLAVE?');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:25:15"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('You wish! You\'ll do shitwork, scan, crack copyrights...');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:46:08"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('And if I win?');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 10:22:46"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('Make it my first-born!');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 11:08:08"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Make it our first-date!');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-24 18:56:01"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('I don\'t DO dates. But I don\'t lose either, so you\'re on!');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-25 22:28:42"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Stanley');
            $comment->setComment('It\'s not gonna end like this.');
            $comment->setBlog($manager->merge($this->getReference('blog-3')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Gabriel');
            $comment->setComment('Oh, come on, Stan. Not everything ends the way you think it should. Besides, audiences love happy endings.');
            $comment->setBlog($manager->merge($this->getReference('blog-3')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Mile');
            $comment->setComment('Doesn\'t Bill Gates have something like that?');
            $comment->setBlog($manager->merge($this->getReference('blog-5')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Gary');
            $comment->setComment('Bill Who?');
            $comment->setBlog($manager->merge($this->getReference('blog-5')));
            $manager->persist($comment);
    
            $manager->flush();
        }
    
        public function getOrder()
        {
            return 2;
        }
    }
        
As with the modifications we made the ``BlogFixtures`` class, the ``CommentFixtures``
class also extends the ``AbstractFixture`` class and  implements the ``OrderedFixtureInterface``.
This means we must also implement the ``getOrder()`` method. This time we set the
return value to 2, ensuring these fixtures will be loaded after the blog fixtures.

We can also see how the references to the ``Blog`` entities we created earlier
are being used.

.. code-block:: php

    $comment->setBlog($manager->merge($this->getReference('blog-2')));

We are now ready to load the fixtures into the database.

.. code-block:: bash

    $ php app/console doctrine:fixtures:load
    
Displaying Comments
-------------------

We can now display the comments related to each blog post. We begin by
updating the ``CommentRepository`` with a method to retrieve the latest approved
comments for a blog post.

Comment Repository
~~~~~~~~~~~~~~~~~~

Open the ``CommentRepository`` class located at
``src/Blogger/BlogBundle/Repository/CommentRepository.php`` and replace its
content with the following.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Repository/CommentRepository.php

    namespace Blogger\BlogBundle\Repository;

    use Doctrine\ORM\EntityRepository;

    /**
     * CommentRepository
     *
     * This class was generated by the Doctrine ORM. Add your own custom
     * repository methods below.
     */
    class CommentRepository extends EntityRepository
    {
        public function getCommentsForBlog($blogId, $approved = true)
        {
            $qb = $this->createQueryBuilder('c')
                       ->select('c')
                       ->where('c.blog = :blog_id')
                       ->addOrderBy('c.created')
                       ->setParameter('blog_id', $blogId);
            
            if (false === is_null($approved))
                $qb->andWhere('c.approved = :approved')
                   ->setParameter('approved', $approved);
                   
            return $qb->getQuery()
                      ->getResult();
        }
    }
    
The method we have created will retrieve comments for a blog post. To do this
we need to add a where clause to our query. The where clause uses a named parameter
that is set using the ``setParameter()`` method. You should always use parameters
instead of setting the values directly in the query like so
    
.. code-block:: php

    ->where('c.blog = ' . blogId)

In this example the value of ``$blogId`` will not be sanitized and could leave the
query open to an SQL injection attack.

Blog Controller
---------------

Next we need to update the ``show`` action of the ``Blog`` controller to retrieve
the comments for the blog. Update the ``Blog`` controller located at
``src/Blogger/BlogBundle/Controller/BlogController.php`` with the following.

.. code-block:: php
    
    // src/Blogger/BlogBundle/Controller/BlogController.php
    
    public function showAction($id)
    {
        // ..

        if (!$blog) {
            throw $this->createNotFoundException('Unable to find Blog post.');
        }
        
        $comments = $em->getRepository('BloggerBlogBundle:Comment')
                       ->getCommentsForBlog($blog->getId());
        
        return $this->render('BloggerBlogBundle:Blog:show.html.twig', array(
            'blog'      => $blog,
            'comments'  => $comments
        ));
    }

We use the new method on the ``CommentRepository`` to retrieve the approved comments
for the blog. The ``$comments`` collection is also passed into the template.

Blog show template
~~~~~~~~~~~~~~~~~~

Now we have a list of comments for the blog we can update the blog show template
to display the comments. We could simply place the rendering of the comments
directly in the blog show template, but as comments are their own entity, it would
be better to separate the rendering into another template, and include that
template. This would allow us to reuse the comment rendering template elsewhere in the
application. Update the blog show template located at
``src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig`` with the
following.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig #}
    
    {# .. #}
    
    {% block body %}
        {# .. #}
    
        <section class="comments" id="comments">
            <section class="previous-comments">
                <h3>Comments</h3>
                {% include 'BloggerBlogBundle:Comment:index.html.twig' with { 'comments': comments } %}
            </section>
        </section>
    {% endblock %}
    
You can see the use of a new Twig tag, the ``include`` tag. This will include the
content of the template specified by ``BloggerBlogBundle:Comment:index.html.twig``.
We can also pass over any number of arguments to the template. In this case, we need
to pass over a collection of ``Comment`` entities to render.

Comment show template
~~~~~~~~~~~~~~~~~~~~~

The ``BloggerBlogBundle:Comment:index.html.twig`` we are including above does
not exist yet so we need to create it. As this is just a template, we don't need
to create a route or a controller for this, we only need the template file. Create
a new file located at ``src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig``
and paste in the following.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig #}
    
    {% for comment in comments %}
        <article class="comment {{ cycle(['odd', 'even'], loop.index0) }}" id="comment-{{ comment.id }}">
            <header>
                <p><span class="highlight">{{ comment.user }}</span> commented <time datetime="{{ comment.created|date('c') }}">{{ comment.created|date('l, F j, Y') }}</time></p>
            </header>
            <p>{{ comment.comment }}</p>
        </article>
    {% else %}
        <p>There are no comments for this post. Be the first to comment...</p>
    {% endfor %}

As you can see we iterate over a collection of ``Comment`` entities and display
the comments. We also introduce one of the other nice Twig functions, the ``cycle``
function. This function will cycle through the values in the array you
pass it as each iteration of the loop progresses. The current loop iteration value
is obtained via the special ``loop.index0`` variable. This keeps a count of the
loop iterations, starting at 0. There are a number of other
`special variables <http://www.twig-project.org/doc/templates.html#for>`_
available when we are within a loop code block. You may also notice the setting
of an HTML ID to the ``article`` element. This will allow us to later create
permalinks to created comments.

Comment show CSS
~~~~~~~~~~~~~~~~

Finally lets add some CSS to keep the comments looking stylish. Update the stylesheet
located at ``src/Blogger/BlogBundle/Resorces/public/css/blog.css`` with the following.

.. code-block:: css

    /** src/Blogger/BlogBundle/Resorces/public/css/blog.css **/
    .comments { clear: both; }
    .comments .odd { background: #eee; }
    .comments .comment { padding: 20px; }
    .comments .comment p { margin-bottom: 0; }
    .comments h3 { background: #eee; padding: 10px; font-size: 20px; margin-bottom: 20px; clear: both; }
    .comments .previous-comments { margin-bottom: 20px; }

.. note::

    If you are not using the symlink method for referencing bundle assets into the
    ``web`` folder you must re-run the assets install task now to copy over the
    changes to your CSS.

    .. code-block:: bash

        $ php app/console assets:install web
        
If you now have a look at one of the blog show pages, eg
``http://symblog.dev/app_dev.php/2`` you should see the blog comments output.

.. image:: /_static/images/part_4/comments.jpg
    :align: center
    :alt: symblog show blog comments
    
Adding Comments
---------------

The last part of the chapter will add the functionality for users to add
comments to blog post. This will be possible via a form on the blog show page. We
have already been introduced to creating forms in Symfony2 when we created the
contact form. Rather than creating the comment form manually, we can use Symfony2
to do this for us. Run the following task to generate the ``CommentType`` class for
the ``Comment`` entity.

.. code-block:: bash
    
    $ php app/console generate:doctrine:form BloggerBlogBundle:Comment
    
You'll notice again here, the use of the short hand version to specify the
``Comment`` entity.

.. tip::

    You may have noticed the task ``doctrine:generate:form`` is also available.
    This is the same task just namespaced differently.
    
The generate form task has created the ``CommentType`` class located at
``src/Blogger/BlogBundle/Form/CommentType.php``.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Form/CommentType.php
    
    namespace Blogger\BlogBundle\Form;
    
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;
    
    class CommentType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder
                ->add('user')
                ->add('comment')
                ->add('approved')
                ->add('created')
                ->add('updated')
                ->add('blog')
            ;
        }
    
        public function getName()
        {
            return 'blogger_blogbundle_commenttype';
        }
    }

We have already explored what is happening here in the previous ``EnquiryType``
class. We could begin by customising this class now, but lets move onto displaying
the form first. 

Displaying the Comment Form
~~~~~~~~~~~~~~~~~~~~~~~~~~

As we want the user to add comments from the show blog page, we could create the
form in the ``show`` action of the ``Blog`` controller and render the form
directly in the ``show`` template. However, it would be better to separate this
code as we did with displaying the comments. The difference between showing
the comments and displaying the comment form is the comment form needs
processing, so this time a controller is required. This introduces a method
slightly different to the above where we just included a template.

Routing
~~~~~~~

We need to create a new route to handle the processing of submitted forms. Add
a new route to  the routing file located at
``src/Blogger/BlogBundle/Resources/config/routing.yml``.

.. code-block:: yaml

    BloggerBlogBundle_comment_create:
        pattern:  /comment/{blog_id}
        defaults: { _controller: BloggerBlogBundle:Comment:create }
        requirements:
            _method:  POST
            blog_id: \d+
        
The controller
~~~~~~~~~~~~~~

Next, we need to create the new ``Comment`` controller we have referenced above.
Create a file located at ``src/Blogger/BlogBundle/Controller/CommentController.php`` and
paste in the following.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/CommentController.php
    
    namespace Blogger\BlogBundle\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Blogger\BlogBundle\Entity\Comment;
    use Blogger\BlogBundle\Form\CommentType;
    
    /**
     * Comment controller.
     */
    class CommentController extends Controller
    {
        public function newAction($blog_id)
        {
            $blog = $this->getBlog($blog_id);
            
            $comment = new Comment();
            $comment->setBlog($blog);
            $form   = $this->createForm(new CommentType(), $comment);
    
            return $this->render('BloggerBlogBundle:Comment:form.html.twig', array(
                'comment' => $comment,
                'form'   => $form->createView()
            ));
        }
    
        public function createAction($blog_id)
        {
            $blog = $this->getBlog($blog_id);
            
            $comment  = new Comment();
            $comment->setBlog($blog);
            $request = $this->getRequest();
            $form    = $this->createForm(new CommentType(), $comment);
            $form->bindRequest($request);
    
            if ($form->isValid()) {
                // TODO: Persist the comment entity
    
                return $this->redirect($this->generateUrl('BloggerBlogBundle_blog_show', array(
                    'id' => $comment->getBlog()->getId())) .
                    '#comment-' . $comment->getId()
                );
            }
    
            return $this->render('BloggerBlogBundle:Comment:create.html.twig', array(
                'comment' => $comment,
                'form'    => $form->createView()
            ));
        }
        
        protected function getBlog($blog_id)
        {
            $em = $this->getDoctrine()
                        ->getEntityManager();
    
            $blog = $em->getRepository('BloggerBlogBundle:Blog')->find($blog_id);
    
            if (!$blog) {
                throw $this->createNotFoundException('Unable to find Blog post.');
            }
            
            return $blog;
        }
       
    }
    
We create 2 actions in the ``Comment`` controller, one for ``new`` and one for
``create``. The ``new`` action is concerned with displaying the comment form,
the ``create`` action is concerned with processing the submission of the comment
form. While this may seem like a big chuck of code, there is nothing new here,
everything was covered in chapter 2 when we created the contact form. However,
before moving on make sure you fully understand what is happening in the
``Comment`` controller.

Form Validation
~~~~~~~~~~~~~~~

We don't want users to be able to submit blogs comments with blank ``user`` or
``comment`` values. To achieve this we look back to the validators we were
introduced to in part 2 when creating the enquiry form. Update the ``Comment``
entity located at ``src/Blogger/BlogBundle/Entity/Comment.php`` with the
following.

.. code-block:: php
    
    <?php
    // src/Blogger/BlogBundle/Entity/Comment.php
    
    // ..
    
    use Symfony\Component\Validator\Mapping\ClassMetadata;
    use Symfony\Component\Validator\Constraints\NotBlank;
    
    // ..
    class Comment
    {
        // ..
        
        public static function loadValidatorMetadata(ClassMetadata $metadata)
        {
            $metadata->addPropertyConstraint('user', new NotBlank(array(
                'message' => 'You must enter your name'
            )));
            $metadata->addPropertyConstraint('comment', new NotBlank(array(
                'message' => 'You must enter a comment'
            )));
        }
        
        // ..
    }

The constraints ensure that both the user and comment members must not be blank.
We have also set the ``message`` option for both constraints to override the
default ones. Remember to add the namespace for ``ClassMetadata`` and
``NotBlank`` as shown above.

The view
~~~~~~~~

Next we need to create the 2 templates for the ``new`` and ``create`` controller
actions. First create  a new file
located at ``src/Blogger/BlogBundle/Resources/views/Comment/form.html.twig``
and paste in the following.

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/Comment/form.html.twig #}
    
    <form action="{{ path('BloggerBlogBundle_comment_create', { 'blog_id' : comment.blog.id } ) }}" method="post" {{ form_enctype(form) }} class="blogger">
        {{ form_widget(form) }}
        <p>
            <input type="submit" value="Submit">
        </p>
    </form>

The purpose of this template is simple, It just renders the comment form. You'll
also notice the ``action`` of the form is to ``POST`` to the new route we created
``BloggerBlogBundle_comment_create``.

Next lets add the template for the ``create`` view. Create a new file located at
``src/Blogger/BlogBundle/Resources/views/Comment/create.html.twig``
and paste in the following.

.. code-block:: html

    {% extends 'BloggerBlogBundle::layout.html.twig' %}
    
    {% block title %}Add Comment{% endblock%}
    
    {% block body %}
        <h1>Add comment for blog post "{{ comment.blog.title }}"</h1>
        {% include 'BloggerBlogBundle:Comment:form.html.twig' with { 'form': form } %}    
    {% endblock %}

As the ``create`` action of the ``Comment`` controller deals with processing
the form, it also needs to be able to display it, as there could be errors in the
form. We reuse the ``BloggerBlogBundle:Comment:form.html.twig`` to render the
actual form to prevent code duplication.

Now lets update the blog show template to render the add blog form. Update the
template located at ``src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig``
with the following.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig #}
    
    {# .. #}
    
    {% block body %}
    
        {# .. #}
        
        <section class="comments" id="comments">
            {# .. #}
            
            <h3>Add Comment</h3>
            {% render 'BloggerBlogBundle:Comment:new' with { 'blog_id': blog.id } %}
        </section>
    {% endblock %}

We use another new Twig tag here, the ``render`` tag. This tag will render
the contents of a controller into the template. In our case we render the 
contents of the ``BloggerBlogBundle:Comment:new`` controller action.

If you now have a look at one of the blog show pages, such as
``http://symblog.dev/app_dev.php/2`` you'll notice a Symfony2 exception is thrown.

.. image:: /_static/images/part_4/to_string_error.jpg
    :align: center
    :alt: toString() Symfony2 Exception
    
This exception is being thrown by the ``BloggerBlogBundle:Blog:show.html.twig``
template. If we look at line 25 of the ``BloggerBlogBundle:Blog:show.html.twig``
template we can see its the following line showing that the problem actually exists
in the process of embedding the ``BloggerBlogBundle:Comment:create`` controller.

.. code-block:: html

    {% render 'BloggerBlogBundle:Comment:create' with { 'blog_id': blog.id } %}
    
If we look at the exception message further it gives us some more information
about the nature of why the exception was caused.

    Entities passed to the choice field must have a "__toString()" method defined

This is telling us that a choice field that we are trying to render doesn't have
a ``__toString()`` method set for the entity the choice field is associated with.
A choice field is a form element that gives the user a number of choices,
such as a ``select`` (drop down) element. You maybe wondering where are we rendering
a choice field in the comment form? If you look at the comment form template again you will notice
we render the form using the ``{{ form_widget(form) }}`` Twig function. This
function outputs the entire form in its basic form. So lets go back to the class
the form is created from, the ``CommentType`` class. We can see that a number of
fields are being added to the form via the ``FormBuilder`` object. In particular
we are adding a ``blog`` field.

If you remember from chapter 2, we spoke about how the ``FormBuilder`` will try
to guess the field type to output based on metadata related to the field. As we
setup a relationship between ``Comment`` and ``Blog`` entities, the
``FormBuilder`` has guessed the comment should be a ``choice`` field, which
would allow the user to specify the blog post to attach the comment to. That is
why we have a ``choice`` field in the form, and why the Symfony2 exception is
being thrown. We can fix this problem by implementing the ``__toString()``
method in the ``Blog`` entity.

.. code-block:: php
    
    // src/Blogger/BlogBundle/Entity/Blog.php
    public function __toString()
    {
        return $this->getTitle();
    }

.. tip::

    The Symfony2 error messages are very informative when describing the problem
    that has occurred. Always read the error messages as they will usually make
    the process of debug a lot easier. The error messages also provide a full
    stack trace so you can see the steps that were taking to cause the error.
    
Now when you refresh the page you should see the comment form output. You will
also notice that some undesirable fields have been output such as ``approved``,
``created``, ``updated`` and ``blog``. This is because we did not customise
the generated ``CommentType`` class earlier.

.. tip::

    The fields being rendered all seem to be output as the correct type of fields.
    The ``user`` fields is an ``text`` field, the ``comment`` field is a ``textarea``,
    the 2 ``DateTime`` fields are a number of ``select`` fields allowing us to specify the
    time, etc.
    
    This is because of the ``FormBuilders`` ability to guess the type of field
    the member it is rendering requires. It is able to do this based on the metadata
    you provide. As we have specified quite specific metadata for the ``Comment``
    entity, the ``FormBuilder`` is able to make accurate guesses of the field types.
    
Lets now update this class located at
``src/Blogger/BlogBundle/Form/CommentType.php`` to output only the fields we
need. 

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Form/CommentType.php
    
    // ..
    class CommentType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder
                ->add('user')
                ->add('comment')
            ;
        }
    
        // ..
    }

Now when you refresh the page only the user and comment fields are output. If
you were to submit the form now, the comment would not actually be saved to the
database. That's because the form controller does nothing with the ``Comment`` entity
if the form passes validation. So how do we persist the ``Comment`` entity to the database.
You have already seen how to do this when creating ``DataFixtures``. Update the
``create`` action of the ``Comment`` controller to persist the ``Comment`` entity
to the database.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/CommentController.php
    
    // ..
    class CommentController extends Controller
    {
        public function createAction($blog_id)
        {
            // ..
            
            if ($form->isValid()) {
                $em = $this->getDoctrine()
                           ->getEntityManager();
                $em->persist($comment);
                $em->flush();
                    
                return $this->redirect($this->generateUrl('BloggerBlogBundle_blog_show', array(
                    'id' => $comment->getBlog()->getId())) .
                    '#comment-' . $comment->getId()
                );
            }
        
            // ..
        }
    }

Persisting the ``Comment`` entity is as simple as a call to ``persist()`` and ``flush()``.
Remember, the form just deals with PHP objects, and Doctrine 2 manages and persists
these objects. There is no direct connection between submitting a form, and
the submitted data being persisted to the database.

You should now be able to add comments to the blog posts.

.. image:: /_static/images/part_4/add_comments.jpg
    :align: center
    :alt: symblog add blog comments
    
Conclusion
----------

We have made good progress in this chapter. Our blogging website is starting to
function more like you'd expect. We now have the basics of the homepage created
and the comment entity. User can now post comments on blogs and read comments
left by other user. We saw how to create fixtures that could be referenced
across multiple fixture files and used Doctrine 2 Migrations to keep the database
schema inline with the entity changes.

Next we will look at building the sidebar to include The Tag Cloud and Recent
Comments. We will also extend Twig by creating our own custom filters. Finally
we will look at using the Assetic asset library to assist us in managing our
assets.
    
