[Part 4] - 關於 Comments Model：新增評論、Doctrine貯藏庫與搬遷
=====================================================================================

概要
--------

這一章會基於上一章定義的部落格 model 做延伸，我們會建立評論 model ，用來處理部落格文章的評論。
我們會介紹與建立兩個 models 之間的關聯，通常一篇部落格文章可以包含多篇評論。我們會使用 Doctrine 2
查詢精靈與 Doctrine 2 貯藏庫類別從資料庫取得資料。我們也會嘗試使用 Doctrine 2 的搬遷功能，這個功能
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

Doctrine 2 提供了貯藏庫類別來協助這個部份。

Doctrine 2 貯藏庫
-----------------------

我們在之前建立部落格顯示頁的章節已經介紹過 Doctrine 2 貯藏庫類別，我們用 ``Doctrine\ORM\EntityRepository`` 類別預設版本的
 ``find()`` 方法來從資料庫取得資料。由於我們想要建立一個自訂查詢，我們需要建立一個自訂的貯藏庫類別， Doctrine 2 可以在這裡
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
程式碼。在貯藏庫類別我們透過 ``createQueryBuilder()`` 方法直接存取 ``QueryBuilder`` ，我們也加入一個預設的參數
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
        
跟我們在 ``BlogFixtures`` 類別做的異動一樣， ``CommentFixtures`` 類別也繼承了 ``AbstractFixture`` 類別與實做
``OrderedFixtureInterface`` 。這表示我們也必須實做 ``getOrder()`` 方法，這次我們將傳回的值設定為 2 ，這樣可以確保
這些裝置在部落格裝置之後載入。

我們也可以看看我們之前建立的 ``Blog`` 資料可以如何使用。

.. code-block:: php

    $comment->setBlog($manager->merge($this->getReference('blog-2')));

我們現在已經準備好將裝置載入資料庫中。

.. code-block:: bash

    $ php app/console doctrine:fixtures:load
    
顯示評論
-------------------

我們現在可以在每一篇文章顯示相關的評論，先更新 ``CommentRepository`` ，透過一個方法來取得一篇文章最新通過審核的評論。

評論貯藏庫
~~~~~~~~~~~~~~~~~~

開啟位於 ``src/Blogger/BlogBundle/Repository/CommentRepository.php`` 的 ``CommentRepository`` 類別，用下面內容取代
原有程式：

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
    
這個我們建立的方法會取得一篇文章的評論，要這麼做我們需要加入一個 where 條件到查詢中，這個 where 條件使用一個透過
``setParameter()`` 方法設定的特定參數。你應該要使用參數而非直接在查詢中設定數值，像這樣：
    
.. code-block:: php

    ->where('c.blog = ' . blogId)

在這個例子， ``$blogId`` 的數值不會經過過濾，而且可能造成查詢存在著 SQL 插入攻擊風險。

部落格 Controller
---------------

接著我們需要更新 ``Blog`` controller 的 ``show`` 方法來取得文章的評論。用下面內容更新放在
``src/Blogger/BlogBundle/Controller/BlogController.php`` 的 ``Blog`` controller 。

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

我們在 ``CommentRepository`` 使用新方法來取得通過審核的評論， ``$comments`` 集合也會傳給樣板。

部落格顯示樣板
~~~~~~~~~~~~~~~~~~

現在我們有這個部落格的評論清單，我們可以更新部落格顯示樣板來顯示評論，我們可以直接在部落格顯示樣板中放入評論的顯示，
不過由於評論有著他們自己的實體，比較建議將顯示的部份分離為另外一個樣板，然後引用進來，這可以讓我們在應用程式中重複運
用評論顯示樣板。用下面內容更新位於 ``src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig`` 的顯示樣板。

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
    
你可以看到使用了一個新的 Twig 標籤， ``include`` 標籤。這會引用 ``BloggerBlogBundle:Comment:index.html.twig``
所指定的樣板內容，我們也可以傳任意數量的參數給樣板。在這個例子中，我們需要傳過去一個 ``Comment`` 資料的集合來顯示。

評論顯示樣板
~~~~~~~~~~~~~~~~~~~~~

我們在上面引用的 ``BloggerBlogBundle:Comment:index.html.twig`` 還不存在，所以我們需要建立它。由於這只是個樣板，
我們不需要為它建立一個網址路徑或 controller ，我們只需要樣板檔案。用下面內容建立一個檔案在
``src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig`` ：

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

如你所見，我們迭代了一個 ``Comment`` 資料集合並且顯示評論，我們也使用了一個好用的 Twig 方法 ``cycle`` ，這個方法會
在每次迴圈進行迭代時循環使用你傳進來的陣列數值，目前迴圈迭代數值是透過特別的變數 ``loop.index0`` 取得，這個變數保留了
一個迴圈迭代的計數器，從 0 開始。還有很多其他的 `特別變數 <http://www.twig-project.org/doc/templates.html#for>`_
可以用在迴圈程式碼區塊中。你也許也注意到為 ``article`` 元素設定的 HTML 編號，這可以讓我們稍候建立評論的永久連結。

評論顯示 CSS
~~~~~~~~~~~~~~~~

最後我們加一些 CSS 來讓評論看起來有點風格，用下面內容更新位於 ``src/Blogger/BlogBundle/Resorces/public/css/blog.css``
的檔案。

.. code-block:: css

    /** src/Blogger/BlogBundle/Resorces/public/css/blog.css **/
    .comments { clear: both; }
    .comments .odd { background: #eee; }
    .comments .comment { padding: 20px; }
    .comments .comment p { margin-bottom: 0; }
    .comments h3 { background: #eee; padding: 10px; font-size: 20px; margin-bottom: 20px; clear: both; }
    .comments .previous-comments { margin-bottom: 20px; }

.. note::

    如果你不是使用符號連結方法來在 ``web`` 目錄參照軟體包資源，你現在需要重新執行資源安裝指令來複製在 CSS 的異動。

    .. code-block:: bash

        $ php app/console assets:install web
        
如果你現在打開其中一篇文章的顯示頁面，像是 ``http://symblog.dev/app_dev.php/2`` ，你應該可以看到部落格評論的輸出。

.. image:: /_static/images/part_4/comments.jpg
    :align: center
    :alt: symblog show blog comments
    
新增評論
---------------

這個章節的最後一部分會加入一個功能讓使用者把評論加到文章中，這可以透過一個部落格顯示頁的表單處理。我們在建立聯絡表單
時已經介紹過如何使用 Symfony2 表單，與其手動建立評論表單，我們這次讓 Symfony2 幫我們完成。執行下面指令來為 ``Comment``
實體產生 ``CommentType`` 類別。

.. code-block:: bash
    
    $ php app/console generate:doctrine:form BloggerBlogBundle:Comment
    
你會發現我們在這裡使用縮寫版本來指定 ``Comment`` 實體。

.. tip::

    你也許注意到還有一個 ``doctrine:generate:form`` 指令，這是同樣的指令，只是使用了不一樣的命名空間。
    
產生表單的指令建立了 ``CommentType`` 類別在 ``src/Blogger/BlogBundle/Form/CommentType.php`` 。

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

在上一個 ``EnquiryType`` 類別中我們已經看過類似這裡的作法，我們可以從這裡的客製開始，不過先從顯示表單開始。

顯示評論表單
~~~~~~~~~~~~~~~~~~~~~~~~~~

我們想要讓使用者可以在部落格顯示頁面新增評論，我們可以將表單建立在 ``Blog`` controller 的  ``show`` 方法，然後直接
在 ``show`` 樣板產生表單。不過比較建議將程式碼分離，就像在顯示評論時一樣，顯示評論與顯示評論表單的差異在，評論表單需
要進一步處理，所以這次需要一個 controller 。這裡介紹的方式跟上面只有引用樣板的方式有點差異。

網址路徑
~~~~~~~

我們需要建立一個新的網址路徑來處理送出的表單，新增下面的網址路徑到檔案
``src/Blogger/BlogBundle/Resources/config/routing.yml`` 。

.. code-block:: yaml

    BloggerBlogBundle_comment_create:
        pattern:  /comment/{blog_id}
        defaults: { _controller: BloggerBlogBundle:Comment:create }
        requirements:
            _method:  POST
            blog_id: \d+
        
關於 controller
~~~~~~~~~~~~~~

接著我們需要建立上面參照的新 ``Comment`` controller ，建立一個檔案在
``src/Blogger/BlogBundle/Controller/CommentController.php`` 並且貼入下面內容。

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
    
我們在 ``Comment`` controller 建立了兩個方法，一個是 ``new`` ，另一個是 ``create`` 。 ``new`` 方法是用來顯示評論表
單，而 ``create`` 方法是用來處理評論表單送出的資料。雖然看起來有一堆程式碼，但這裡沒有新東西，所有的東西在第二章介紹
聯絡表單時就提過，不過在繼續往下看之前，請確認你完全了解 ``Comment`` controller 發生了什麼事。

表單驗證
~~~~~~~~~~~~~~~

我們不希望使用者提供的評論中 ``user`` 或 ``comment`` 是空白的，要處理這個部份我們可以回頭看第二部份介紹查詢表單時使用
的驗證器，用下面內容更新位於 ``src/Blogger/BlogBundle/Entity/Comment.php`` 的 ``Comment`` 實體。

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

這裡的限制確保了使用者與評論屬性不會是空白的，我們也在兩個限制中設定了 ``message`` 選項來取代預設值，記住要像上面這樣
加入命名空間 ``ClassMetadata`` 與 ``NotBlank`` 。

關於 view
~~~~~~~~

接著我們需要建立兩個樣板給 ``new`` 與 ``create`` 方法，先建立一個檔案在 ``src/Blogger/BlogBundle/Resources/views/Comment/form.html.twig``
並且貼入下面內容。

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/Comment/form.html.twig #}
    
    <form action="{{ path('BloggerBlogBundle_comment_create', { 'blog_id' : comment.blog.id } ) }}" method="post" {{ form_enctype(form) }} class="blogger">
        {{ form_widget(form) }}
        <p>
            <input type="submit" value="Submit">
        </p>
    </form>

這個樣板的目的很單純，只是要顯示評論表單。你也會注意到表單的 ``action`` 會將資料 ``POST`` 到我們建立的新網址路徑
``BloggerBlogBundle_comment_create`` 。

接著我們新增 ``create`` 的樣板，建立一個檔案在 ``src/Blogger/BlogBundle/Resources/views/Comment/create.html.twig``
並且貼入下面內容。

.. code-block:: html

    {% extends 'BloggerBlogBundle::layout.html.twig' %}
    
    {% block title %}Add Comment{% endblock%}
    
    {% block body %}
        <h1>Add comment for blog post "{{ comment.blog.title }}"</h1>
        {% include 'BloggerBlogBundle:Comment:form.html.twig' with { 'form': form } %}    
    {% endblock %}

由於 ``Comment`` controller 的 ``create`` 方法會處理表單資料，它也需要能夠顯示，因為可能在表單會有錯誤。我們重複使用
``BloggerBlogBundle:Comment:form.html.twig`` 來顯示實際的表單，藉此避免重複的程式碼。

現在可以更新部落格顯示樣板來產生新增部落格表單，更新位於 ``src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig``
的樣板，放入下面內容。

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

我們在這裡使用了另一個新的 Twig 標籤 ``render`` ，這個標籤會產生一個 controller 的內容來放入樣板，在我們的例子中，我
們產生了 ``BloggerBlogBundle:Comment:new`` controller 方法的內容。

如果你現在看看其中一個部落格顯示頁面，像是 ``http://symblog.dev/app_dev.php/2`` ，你會發現出現了一個 Symfony2 例外。

.. image:: /_static/images/part_4/to_string_error.jpg
    :align: center
    :alt: toString() Symfony2 Exception
    
這個例外是由 ``BloggerBlogBundle:Blog:show.html.twig`` 樣板產生，如果我們看到 ``BloggerBlogBundle:Blog:show.html.twig``
樣板的第 25 行，我們會看到下面這行，這表示問題實際上存在於嵌入 ``BloggerBlogBundle:Comment:create`` controller 的處理
中。

.. code-block:: html

    {% render 'BloggerBlogBundle:Comment:create' with { 'blog_id': blog.id } %}
    
如果我們再仔細看例外訊息，它提供了一些關於例外發生原因的理由。

    Entities passed to the choice field must have a "__toString()" method defined

這告訴我們一個我們試著顯示的選擇欄位沒有為關聯的實體設定 ``__toString()`` 方法，一個選擇欄位是一個提供使用者一些選項的表單元素
，想是 ``select`` (下拉選單)元素，你也許想知道我們在評論表單的哪裡產生一個選擇欄位，如果你再仔細看看評論表單樣板，你會注意到我們
透過 Twig 方法 ``{{ form_widget(form) }}`` 來產生表單，這個方法用基本格式輸出整個表單。所以讓我們回到建立表單的來源類別
``CommentType`` ，我們可以看到一些欄位透過 ``FormBuilder`` 物件加入到表單，特別的是我們新增了一個 ``blog`` 欄位。

如果你還記得第二章，我們提到 ``FormBuilder`` 會如何猜測欄位輸出類型，也就是欄位的相關後設資料。因為我們設定了關聯在 ``Comment``
與 ``Blog`` 實體間， ``FormBuilder`` 已經猜到評論也許是一個 ``choice`` 欄位，可以讓使用者指定要附加評論的對象，這是為什麼我們在
表單中有一個 ``choice`` 欄位，也是為什麼 Symfony2 例外會發生。我們可以在 ``Blog`` 實體實做 ``__toString()`` 方法來修正這個問題。

.. code-block:: php
    
    // src/Blogger/BlogBundle/Entity/Blog.php
    public function __toString()
    {
        return $this->getTitle();
    }

.. tip::

    Symfony2 的錯誤訊息在描述已經發生的問題時提供許多資訊，記得仔細看錯誤訊息，它們通常可以讓除錯的過程簡單許多。錯誤訊息也會提供一個
    完整的堆疊追蹤，所以你可以看到造成這個問題發生的詳細過程。
    
現在當你重新整理網頁，你應該可以看到評論表單輸出。你也會注意到一些不希望出現的欄位在輸出中，像是 ``approved`` 、 ``created`` 、 ``updated``
與 ``blog`` ，這是因為我們還沒客製之前產生的 ``CommentType`` 類別。

.. tip::

    顯示的欄位似乎都輸出了正確的欄位邢台， ``user`` 欄位是一個 ``text`` 類型， ``comment`` 欄位則是一個 ``textarea`` ，兩個 ``DateTime``
    欄位是一些 ``select`` 欄位讓我們指定時間等資訊。
    
    這是因為 ``FormBuilders`` 能夠猜測屬性在顯示時需要的欄位類型，它是透過你提供的後設資料做到。由於我們已經指定了 ``Comment`` 實體詳細的
    後設資料， ``FormBuilder`` 就能夠正確的猜出欄位類型。
    
現在我們來更新位於 ``src/Blogger/BlogBundle/Form/CommentType.php`` 的這個類別，只輸出我們需要的欄位。

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

現在當你重新整理頁面，只會輸出使用者與評論欄位。如果你現在送出表單，評論並不會真的被儲存到資料庫中，因為表單的 controller 還沒有為 ``Comment``
實體做任何事。如果表單通過了檢驗，我們應該要如何將 ``Comment`` 保存到資料庫？你已經在建立 ``DataFixtures`` 看過怎麼做，更新 ``Comment``
controller 的 ``create`` 方法來儲存 ``Comment`` 實體到資料庫。

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

保存 ``Comment`` 實體很簡單，只要呼叫 ``persist()`` 與 ``flush()`` 。記得，這個表單只處理 PHP 物件， Doctrine 2 負責管理與保存這些物件。
在送出的表單與送出資料被保存到資料庫之兼併沒有直接的連結。

你現在應該可以新增評論到文章中。

.. image:: /_static/images/part_4/add_comments.jpg
    :align: center
    :alt: symblog add blog comments
    
結論
----------

我們在這個章節有了不錯的進展，我們的部落格網站開始像你預期一樣運作。我們現在建立了基本的首頁與評論實體，使用者可以發表評論到文章以及閱讀其他
使用者留下的評論，我們看到如何建立裝置在多個裝置檔案間參照，並且使用了 Doctrine 2 搬遷來讓資料庫結構連結到實體異動。

接下來我們會看看建立一個選單列來包含標籤雲與最新評論，我們也會建立自訂的過濾器來延伸 Twig 功能，最後我們會看看如何使用 Assetic 資源函式庫來
幫助我們管理我們的資源。
    
