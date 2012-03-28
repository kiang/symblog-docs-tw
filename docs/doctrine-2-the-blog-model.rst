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

    We have used a number of the Symfony2 command line task now, and in true
    command line task format they all provide help by specifying the ``--help``
    option. To see the help details for the ``doctrine:schema:create`` task,
    run the following

    .. code-block:: bash

        $ php app/console doctrine:schema:create --help

    The help information will be output showing the usage, and available
    options. Most tasks come with a number of options that can be set to
    customise the running of the task.

Integrating the Model with the View. Showing a blog entry
---------------------------------------------------------

Now we have the ``Blog`` entity created, and the database updated to reflect this,
we can start integrating the model into the view. We will start by building the
show page of our blog.

The Show Blog Route
~~~~~~~~~~~~~~~~~~~

We begin by creating a route for the blog ``show`` action. A blog will be identified
by its unique ID, so this ID will need to be present in the URL. Update the
``BloggerBlogBundle`` routing located at ``src/Blogger/BlogBundle/Resources/config/routing.yml``
with the following

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_blog_show:
        pattern:  /{id}
        defaults: { _controller: BloggerBlogBundle:Blog:show }
        requirements:
            _method:  GET
            id: \d+

As the blog ID must be present in the URL, we have specified an ``id`` placeholder.
This means URLs like ``http://symblog.co.uk/1`` and ``http://symblog.co.uk/my-blog``
will match this route. However, we know the blog ID must be a integer (it's defined this
way in the entity mappings) so we can add a constraint that specifies this route
only matches when the ``id`` parameter contains an integer. This is achieved with the
``id: \d+`` route requirement. Now only the first URL example of the previous would match,
``http://symblog.co.uk/my-blog`` would no longer match this route. You can also
see a matching route will execute the ``show`` action of the ``BloggerBlogBundle``
``Blog`` controller. This controller is yet to be created.

The Show Controller Action
~~~~~~~~~~~~~~~~~~~~~~~~~~

The glue between the Model and the View is the controller, so this is where we
will begin creating the show page. We could add the ``show`` action to our existing
``Page`` controller but as this page is concerned with showing ``blog`` entities
it would be better suited in its own ``Blog`` controller.

Create a new file located at ``src/Blogger/BlogBundle/Controller/BlogController.php``
and paste in the following.

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

We have created a new Controller for the ``Blog`` entity and defined the ``show`` action.
As we specified a ``id`` parameter in the ``BloggerBlogBundle_blog_show`` routing
rule, it will be passed in as an argument to the ``showAction`` method. If we had specified
more parameters in the routing rule, they would also be passed in as separate arguments.

.. tip::

    The controller actions will also pass over an object of
    ``Symfony\Component\HttpFoundation\Request`` if you specify this as a parameter.
    This can be useful when dealing with forms. We have already used a form
    in chapter 2, but we did not use this method as we used one of the
    ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` helper methods as follows.

    .. code-block:: php

        // src/Blogger/BlogBundle/Controller/PageController.php
        public function contactAction()
        {
            // ..
            $request = $this->getRequest();
        }

    We could have instead written this as follows.

    .. code-block:: php

        // src/Blogger/BlogBundle/Controller/PageController.php

        use Symfony\Component\HttpFoundation\Request;

        public function contactAction(Request $request)
        {
            // ..
        }
    
    Both methods achieve the same task. If your controller did not extend the
    ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` helper class
    you would not be able to use the first method.

Next we need to retrieve the ``Blog`` entity from the database. We first
use another helper method of the ``Symfony\Bundle\FrameworkBundle\Controller\Controller``
class to get the Doctrine2 Entity Manager. The job of the
`Entity Manager <http://www.doctrine-project.org/docs/orm/2.0/en/reference/working-with-objects.html>`_
is to handle the retrieval and persistence of objects to and from the database. We
then use the ``EntityManager`` object to get the Doctrine2 ``Repository`` for the
``BloggerBlogBundle:Blog`` entity. The syntax specified here is simply
a short cut that can be used with Doctrine 2 instead of specifying the full
entity name, i.e. ``Blogger\BlogBundle\Entity\Blog``. With the repository object
we call the ``find()`` method passing in the ``$id`` argument.
This method will retrieve the object by its primary key.

Finally we check that an entity was found, and pass this entity over to the view.
If no entity was found a ``createNotFoundException`` is thrown. This will
ultimately generate a ``404 Not Found`` response.

.. tip::

    The repository object gives you access to a number of useful helper methods
    including

    .. code-block:: php

        // Return entities where 'author' matches 'dsyph3r'
        $em->getRepository('BloggerBlogBundle:Blog')->findBy(array('author' => 'dsyph3r'));

        // Return one entity where 'slug' matches 'symblog-tutorial'
        $em->getRepository('BloggerBlogBundle:Blog')->findOneBySlug('symblog-tutorial');

    We will create our own custom Repository classes in the next chapter
    when we require more complex queries.

The View
~~~~~~~~

Now we have built the ``show`` action for the ``Blog`` controller we can focus
on displaying the ``Blog`` entity. As specified in the ``show`` action the
template ``BloggerBlogBundle:Blog:show.html.twig`` will be rendered. Let's create
this template located at ``src/Blogger/BlogBundle/Resouces/views/Blog/show.html.twig``
and paste in the following.

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

As you'd expect we begin by extending the ``BloggerBlogBundle`` main layout.
Next we override the page title with the title of the blog. This will
be useful for SEO as the page title of the blog is more descriptive
than the default title that is set. Lastly we override
the body block to output the ``Blog`` entity conent. We use the ``asset`` function
again here to render the blog image. The blog images should be placed in the
``web/images`` folder.

CSS
...

In order to ensure the blog show page looks beautiful, we need to add some styling.
Update the stylesheet located at ``src/Blogger/BlogBundle/Resouces/public/css/blog.css``
with the following.

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

    If you are not using the symlink method for referencing bundle assets into the
    ``web`` folder you must re-run the assets install task now to copy over the
    changes to your CSS.

    .. code-block:: bash

        $ php app/console assets:install web


As we have now built the controller and the view for the ``show`` actions
lets have a look at the show page. Point your browser to
``http://symblog.dev/app_dev.php/1``. Not the page you were expecting?

.. image:: /_static/images/part_3/404_not_found.jpg
    :align: center
    :alt: Symfony2 404 Not Found Exception

Symfony2 has generated a ``404 Not Found`` response. This is because we
have no data in our database, so no entity with ``id`` equal to 1 could be found.

You could simply insert a row into the blog table of your database, but we will use
a much better method; Data Fixtures.

Data Fixtures
-------------

We can use fixtures to populate the database with some sample/test data. To do this
we use the Doctrine Fixtures extension and bundle. The Doctrine Fixtures
extension and bundle do not come with the Symfony2 Standard Distribution, we need to
manually install them. Fortunately this is an easy task. Open up the deps file located
in the project root and add the Doctrine fixtures extension and bundle to it as
follows.

.. code-block:: text

    [doctrine-fixtures]
        git=http://github.com/doctrine/data-fixtures.git

    [DoctrineFixturesBundle]
        git=http://github.com/symfony/DoctrineFixturesBundle.git
        target=/bundles/Symfony/Bundle/DoctrineFixturesBundle

Next update the vendors to reflect these changes.

.. code-block:: bash

    $ php bin/vendors install

This will pull down the latest version of each of the repositories from Github and
install them to the required location.

.. note::

    If you are using a machine that does not have Git installed you will need to manually
    download and install the extension and bundle.

    doctrine-fixtures extension: `Download <https://github.com/doctrine/data-fixtures>`_
    the current version of the package from GitHub and extract to the following location
    ``vendor/doctrine-fixtures``.

    DoctrineFixturesBundle: `Download <https://github.com/symfony/DoctrineFixturesBundle>`_
    the current version of the package from GitHub and extract to the following location
    ``vendor/bundles/Symfony/Bundle/DoctrineFixturesBundle``.

Next update the ``app/autoloader.php`` file to register the new namespace.
As DataFixtures are also in the ``Doctrine\Common`` namespace they must be placed above the existing
``Doctrine\Common`` directive as they specify a new path. Namespaces are checked from top
to bottom so more specific namespaces need to be registered before less specific ones.

.. code-block:: php

    // app/autoloader.php
    // ...
    $loader->registerNamespaces(array(
    // ...
    'Doctrine\\Common\\DataFixtures'    => __DIR__.'/../vendor/doctrine-fixtures/lib',
    'Doctrine\\Common'                  => __DIR__.'/../vendor/doctrine-common/lib',
    // ...
    ));

Now let's register the ``DoctrineFixturesBundle`` in the kernel located at
``app/AppKernel.php``

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

Blog Fixtures
~~~~~~~~~~~~~

We are now ready to define some fixtures for our blogs. Create a fixture file at
``src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php`` and add the following content:

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

The fixture file demonstrates a number of important features when using Doctrine 2,
including how to persist entities to the database.

Let's look at how we create one blog entry.

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

We start by creating an object of ``Blog`` and setting some values for its
members. At this point Doctrine 2 knows nothing about the ``Entity`` object. It's
only when we make a call to ``$manager->persist($blog1)`` that we instruct
Doctrine 2 to start managing this entity object. The ``$manager`` object here
is an instance of the ``EntityManager`` object we saw earlier when retrieving
entites from the database. It is important to note that while
Doctrine 2 is now aware of the entity object, it is still not persisted to the
database. A call to ``$manager->flush()`` is required for this. The flush
method causes Doctrine 2 to actually interact with the database and action all the
entities it is managing. For best performance you should group Doctrine 2
operations together and flush all the actions in one go. This is how we have
done so in our fixtures. We create each entity, ask Doctrine 2 to manage it and
then flush all operations at the end.

.. tip:

    You may have noticed the setting of the ``created`` and ``updated`` members. This is not
    an ideal way to set these fields as you'd expect them to be set automatically
    when an object is created or updated. Doctrine 2 provides a way for us to achieve this
    which we will explore shortly.

Loading the fixtures
~~~~~~~~~~~~~~~~~~~~

We are now ready to load the fixtures into the database.

.. code-block:: bash

    $ php app/console doctrine:fixtures:load

If we have a look at the show page at ``http://symblog.dev/app_dev.php/1``
you should see a blog the blog entry.

.. image:: /_static/images/part_3/blog_show.jpg
    :align: center
    :alt: The symblog blog show page

Try changing the ``id`` parameter in the URL to 2. You should see the next blog entry
being shown.

If you have a look at the URL ``http://symblog.dev/app_dev.php/100`` you should see
a ``404 Not Found`` exception being thrown. You'd expect this as there is no ``Blog``
entity with an ID of 100. Now try the URL ``http://symblog.dev/app_dev.php/symfony2-blog``.
Why don't we get a ``404 Not Found`` exception? This is because the ``show`` action is
never executed. The URL fails to match any route in the application because of the
``\d+`` requirement we set on the ``BloggerBlogBundle_blog_show`` route. This
is why you see a ``No route found for "GET /symfony2-blog"`` exception.

Timestamps
----------

Finally in this chapter we will look at the 2 timestamp members on the ``Blog`` entity;
``created`` and ``updated``. The functionality for these 2 members is commonly referred to as
the ``Timestampable`` behavior. These members hold the time the blog was created and
the time the blog was last updated. As we don't want to have to manually set these fields
each time we create or update a blog, we can use Doctrine 2 to help us.

Doctrine 2 comes with an
`Event System <http://www.doctrine-project.org/docs/orm/2.0/en/reference/events.html>`_
that provides
`Lifecycle Callbacks <http://www.doctrine-project.org/docs/orm/2.0/en/reference/events.html#lifecycle-callbacks>`_.
We can use these callback events to register our entities to be notified of events
during the entity lifetime. Some example of events we can be notified about
include before an update happens, after a persist happens and after a remove happens.
In order to use Lifecycle Callbacks on our entity we need to register the entity for them.
This is done using metadata on the entity. Update the ``Blog`` entity located at
``src/Blogger/BlogBundle/Entity/Blog.php`` with the following.

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

Now let's add a method in the ``Blog`` entity that registers for the ``preUpdate``
event. We also add a constructor to set default values for the ``created`` and
``updated`` members.

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

We register the ``Blog`` entity to be notified on the ``preUpdate`` event to set the
``updated`` member value. Now when you re-run the load fixtures task you will notice the
``created`` and ``updated`` members are set automatically.

.. tip::

    As timestampable members are such a common requirement for entities, there is
    a bundle available that supports them. The
    `StofDoctrineExtensionsBundle <https://github.com/stof/StofDoctrineExtensionsBundle>`_
    provides a number of useful Doctrine 2 extensions including Timestampable,
    Sluggable, and Sortable.

    We will look at integrating this bundle later in the tutorial. The eager
    ones among you can check the
    `cookbook <http://symfony.com/doc/current/cookbook/doctrine/common_extensions.html>`_
    for a chapter on this topic.

Conclusion
----------

We have covered a number of concepts for dealing with models in Doctrine 2.
We also looked at defining Data fixtures to provide us will an easy way to get
suitable test data into our application duration development and testing.

Next we will look at extending the model further by adding the comment entity.
We will start to construct the homepage and create a custom Repository to do this.
We will also introduce the concept of Doctrine Migrations and how forms
interact with Doctrine 2 to allow comments to be posted for a blog.
