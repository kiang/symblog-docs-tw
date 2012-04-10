[Part 6] - 測試：使用 PHPUnit 進行單元與功能測試
====================================================

概要
--------

到目前為止，我們已經試過了許多 Symfony2 開發的核心概念基礎，在我們開始加入新功能前，該是時候來介紹測試
部份。我們會看看如何針對個別方法進行單元測試以及確保多個元件可以一起正確運作的功能性測試。我們會介紹
PHP 測試函式庫 `PHPUnit <http://www.phpunit.de/manual/current/en/>`_ ，因為它是 Symfony2 測試功能的
核心。由於測試是一個很大的主題，在後面章節也會談到。在這個章節完成後，你會完成一些測試程式，包括單元測試
以及功能性測試。你會用到模擬的瀏覽器請求、將資料放入表單以及檢查過的回應來確保網頁可以正確輸出，你也會看
到測試程式在應用程式基礎上涵蓋了多大範圍。

在 Symfony2 的測試
-------------------

`PHPUnit <http://www.phpunit.de/manual/current/en/>`_ 已經自然形成了 PHP 測試程式的一個標準，所以學著
用它對於所有 PHP 專案都有幫助，也別忘了這個章節大部分主題都沒有區分程式語言，所以可以轉移到其他語言使用。

.. tip::

    如果你計畫寫一個自己的開放原始碼 Symfony2 軟體包，如果你的軟體包經過了完整的測試（與製作文件），預期
    會有比較多人感興趣。可以看看 `Symfony2Bundles <http://symfony2bundles.org/>`_ 上面已經存在的
    Symfony2 軟體包。

單元測試
~~~~~~~~~~~~

單元測試的目的是確保程式碼功能的個別單元在獨立使用時的正確性，在一個像 Symfony2 這樣物件導向的程式架構，
一個單元可能是一個類別與他的方法。例如，我們可以設計測試程式給 ``Blog`` 與 ``Comment`` 實體類別。在設計
單元測試時，測試案例應該要被設計為不需要依靠其他程式就能獨立運作，例如測試案例 B 的結果不應該依賴測試案例
A 的結果。單元測試時有個實用的方法是模擬物件，讓你可以輕易針對依賴外部程式的功能進行單元測試，透過這個方式
可以讓你模擬一個方法的執行，而不需要實際執行它，例如一個類別打包了外部 API ，我們要針對它做單元測試時，我
們可以模擬傳輸層的請求方法來傳回我們指定的結果，而不需要實際接觸外部 API 。單元測試並不針對應用程式功能相
關元件能夠一起運作，這會在下一個主題的功能性測試中討論。

功能性測試
~~~~~~~~~~~~~~~~~~

功能性測試檢查應用程式中不同元件結合在一起的情況，像是網址路徑、 controllers 與 views 。功能性測試有點像
是你自己會透過瀏覽器手動進行的測試，像是請求首頁、點選部落格連結以及檢查是否顯示了正確的文章。功能性測試讓
你可以將這個程序自動化， Symfony2 提供了完整且實用的類別來協助你進行功能性測試，包括一個 ``Client`` 可以
用來請求頁面、送出表單與 DOM ``Crawler`` 來解讀來自用戶端的 ``Response`` 。

.. tip::

    許多軟體開發流程是測試導向的，像是測試驅動開發 (TDD) 與行為驅動開發 (BDD) 。不過這些已經超過本篇教學的
    範圍，你可以參考 `everzet <https://twitter.com/#!/everzet>`_ 設計的 BDD 函式庫
    `Behat <http://behat.org/>`_ ，已經有一個 Symfony2 `BehatBundle <http://docs.behat.org/bundle/index.html>`_
    可以讓你將 Behat 整合到自己的 Symfony2 專案。

PHPUnit
-------

如同上面提到的， Symfony2 的測試程式是以 PHPUnit 設計，你需要安裝 PHPUnit 來執行這些測試，以及我們在這個章
節產生的測試程式。完整的 `安裝教學 <http://www.phpunit.de/manual/current/en/installation.html>`_ 可以參考
PHPUnit 官方網站的文件。要執行 Symfony2 的測試，你需要安裝 PHPUnit 3.5.11 或更新版本， PHPUnit 是一個非常
大的測試函式庫，所以有需要的時候可以參考官方文件的說明。

主張
~~~~~~~~~~

設計測試程式的目的是檢查實際測試結果是否與預期測試結果相同，在 PHPUnit 有許多主張方法協助你處理這個部份。部份
你會用到的常見主張方法列在下面。

.. code-block:: php

    // Check 1 === 1 is true
    $this->assertTrue(1 === 1);

    // Check 1 === 2 is false
    $this->assertFalse(1 === 2);

    // Check 'Hello' equals 'Hello'
    $this->assertEquals('Hello', 'Hello');

    // Check array has key 'language'
    $this->assertArrayHasKey('language', array('language' => 'php', 'size' => '1024'));

    // Check array contains value 'php'
    $this->assertContains('php', array('php', 'ruby', 'c++', 'JavaScript'));

完整的
`主張方法 <http://www.phpunit.de/manual/current/en/writing-tests-for-phpunit.html#writing-tests-for-phpunit.assertions>`_
可以參考 PHPUnit 文件。

執行 Symfony2 測試
----------------------

在我們開始設計一些測試程式前，我們先看看如何執行在 Symfony2 中的測試。 PHPUnit可以設定為透過一個設定檔執行，在
我們的 Symfony2 專案，這個檔案放在 ``app/phpunit.xml.dist`` 。由於這個檔案副檔名是 ``.dist`` ，你需要複製它的
內容到一個新檔案 ``app/phpunit.xml`` 。

.. tip::

    如果你使用了像是 Git 這樣的 VCS ，你需要將新增的 ``app/phpunit.xml`` 檔案加入到 VCS 忽略清單。

如果你打開 PHPUnit 的設定檔，會看到下面內容。

.. code-block:: xml

    <!-- app/phpunit.xml -->
    
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/*/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

這些設定有一些資料夾是我們測試程式的一部分，執行 PHPUnit 時它會檢查上面的資料夾來執行測試，你也可以傳一些額外的
指令列參數給 PHPUnit 來在指定資料夾執行，而不是這個測試程式。後面會介紹這個部份。

你也會注意到這個設定指定了放在 ``app/bootstrap.php.cache`` 的引導程序檔案，這個檔案是讓 PHPUnit 可以取得測試
環境設定。

.. code-block:: xml

    <!-- app/phpunit.xml -->
    
    <phpunit
        bootstrap                   = "bootstrap.php.cache" >

.. tip::

    關於在 PHPUnit 使用 XML 檔案進行設定部份可以參考
    `PHPUnit 手冊 <http://www.phpunit.de/manual/current/en/organizing-tests.html#organizing-tests.xml-configuration>`_.

執行目前測試
-------------------------

由於我們在第一章使用了一個 Symfony2 產生器指令來建立 ``BloggerBlogBundle`` ，它同時也為 ``DefaultController``
類別建立了一個 controller 測試，我們可以在專案根目錄用下面指令執行這個測試， ``-c`` 選項指定了 PHPUnit 應該要從
``app`` 目錄載入它的設定。

.. code-block:: bash

    $ phpunit -c app

測試完成後你應該會被提醒測試有錯誤，如果你看看放在 ``src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php``
的 ``DefaultControllerTest`` 類別，會看到下面內容。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class DefaultControllerTest extends WebTestCase
    {
        public function testIndex()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/hello/Fabien');

            $this->assertTrue($crawler->filter('html:contains("Hello Fabien")')->count() > 0);
        }
    }

這是 Symfony2 為 ``DefaultController`` 類別產生的功能性測試，如果你還記得我們在第一章提到的，這個 Controller 有
一個方法會處理 ``/hello/{name}`` 的請求。由於我們已經刪除了這個 controller ，所以上面的測試會產生錯誤。可以試著
在瀏覽器打開網址 ``http://symblog.dev/app_dev.php/hello/Fabien`` ，你應該會被提醒這個網址路徑不存在，因為上面
的測試對同樣網址發出一個請求，它應該會取得同樣的回應，這就是為什麼測試出現錯誤。功能性測試在這個章節佔很大的篇幅，
我們後面會做詳細介紹。

由於 ``DefaultController`` 類別已經刪除了，你也可以刪除這個測試類別，也就是刪除放在
``src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php`` 的 ``DefaultControllerTest`` 類別。

單元測試
------------

如同之前提到的，單元測試的目的是獨立測試應用程式的個別單元，設計單元測試時建議在測試資料夾下重現軟體包的目錄結構，
例如如果你想要測試放在 ``src/Blogger/BlogBundle/Entity/Blog.php`` 的 ``Blog`` 實體類別，測試檔案就可以放在
``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php`` ，一個資料夾的範例就像下面這樣。

.. code-block:: text

    src/Blogger/BlogBundle/
                    Entity/
                        Blog.php
                        Comment.php
                    Controller/
                        PageController.php
                    Twig/
                        Extensions/
                            BloggerBlogExtension.php
                    Tests/
                        Entity/
                            BlogTest.php
                            CommentTest.php
                        Controller/
                            PageControllerTest.php
                        Twig/
                            Extensions/
                                BloggerBlogExtensionTest.php

注意，每個測試檔案的名稱在最後都有 Test 。

測試 Blog 實體 - Slugify 方法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我們從測試 ``Blog`` 實體的替代網址方法開始，設計一些測試來確保這個方法可以正確運作。建立一個新檔案在
``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php`` 並且貼入下面內容。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    namespace Blogger\BlogBundle\Tests\Entity;

    use Blogger\BlogBundle\Entity\Blog;

    class BlogTest extends \PHPUnit_Framework_TestCase
    {

    }

我們已經建立了一個 ``Blog`` 實體的測試類別，注意檔案的位置遵循了上面提到的資料夾結構，這個 ``BlogTest`` 類別繼承
了 ``PHPUnit_Framework_TestCase`` 這個基礎 PHPUnit 類別，所有基於 PHPUnit 設計的測試都會是這個類別的一個子項目。
記得在上一個章節提到過，在 ``PHPUnit_Framework_TestCase`` 類別名稱前需要加一個 ``\`` ，因為這個類別被宣告為 PHP
開放命名空間。

現在我們有測試 ``Blog`` 實體的骨架類別，就開始來設計一個測試案例。在 PHPUnit 中，測試案例就是測試類別中名稱前面有
``test`` 的方法，像是 ``testSlugify()`` 。用下面內容更新放在 ``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php``
的 ``BlogTest`` 類別

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    class BlogTest extends \PHPUnit_Framework_TestCase
    {
        public function testSlugify()
        {
            $blog = new Blog();

            $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        }
    }

這是一個非常簡單的測試案例，它產生 ``Blog`` 實體的實例，接著針對 ``slugify`` 方法執行一個 ``assertEquals()``。
``assertEquals()`` 方法取得兩個必要參數，預期的結果以及實際結果。另外選填的第三個參數可以放入在測試案例錯誤時要顯
示的訊息。

讓我們執行新的單元測試，執行下面指令。

.. code-block:: bash

    $ phpunit -c app

你應該會看到下面輸出。

.. code-block :: bash

    PHPUnit 3.5.11 by Sebastian Bergmann.

    .

    Time: 1 second, Memory: 4.25Mb

    OK (1 test, 1 assertion)

PHPUnit 的輸出很簡單，開始是顯示 PHPUnit 的資訊以及輸出一些 ``.`` 來代表每次執行的測試數量。在我們的例子中，我們
只有執行一個測試，所以只有輸出一個 ``.`` 。最後的描述告訴我們測試的結果，在我們的 ``BlogTest`` 只有執行一個測試，
其中只有一個主張。如果你的指令模式支援彩色輸出，你會發現最後一行是綠色的，表示執行正確。接著更新 ``testSlugify()``
方法來看測試失敗時會怎麼樣。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a day with symfony2', $blog->slugify('A Day With Symfony2'));
    }

重新執行上一個單元測試，應該會看到下面輸出。

.. code-block :: bash

    PHPUnit 3.5.11 by Sebastian Bergmann.

    F

    Time: 0 seconds, Memory: 4.25Mb

    There was 1 failure:

    1) Blogger\BlogBundle\Tests\Entity\BlogTest::testSlugify
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -a day with symfony2
    +a-day-with-symfony2

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Entity/BlogTest.php:15

    FAILURES!
    Tests: 1, Assertions: 2, Failures: 1.

這次的輸出就有點複雜了，我們可以看到執行測試時的 ``.`` 變成了 ``F`` ，這告訴我們測試出錯。如果你的測試包含錯誤，你
也會看到 ``E`` 字元輸出。我們看到 ``Blogger\BlogBundle\Tests\Entity\BlogTest::testSlugify`` 方法錯誤，因為預期的
數值與實際的數值不一樣，如果你的指令列支援彩色輸出，你會發現最後一行用紅色顯示，表示測試有錯誤發生。修正
``testSlugify()`` 方法，這樣子測試就能夠執行成功。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a-day-with-symfony2', $blog->slugify('A Day With Symfony2'));
    }

在繼續加入更多測試給 ``slugify()`` 方法。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a-day-with-symfony2', $blog->slugify('A Day With Symfony2'));
        $this->assertEquals('hello-world', $blog->slugify('Hello    world'));
        $this->assertEquals('symblog', $blog->slugify('symblog '));
        $this->assertEquals('symblog', $blog->slugify(' symblog'));
    }

現在我們已經測試了 ``Blog`` 實體的替代網址方法，我們需要確保 ``Blog`` 的 ``$title`` 屬性更新時， ``Blog`` 的
``$slug`` 會正確被設定。 在 ``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php`` 的 ``BlogTest`` 加入下面方法。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSetSlug()
    {
        $blog = new Blog();

        $blog->setSlug('Symfony2 Blog');
        $this->assertEquals('symfony2-blog', $blog->getSlug());
    }

    public function testSetTitle()
    {
        $blog = new Blog();

        $blog->setTitle('Hello World');
        $this->assertEquals('hello-world', $blog->getSlug());
    }

我們開始測試 ``setSlug`` 方法來確保 ``$slug`` 屬性在更新時能夠正確放入替代網址，接著我們檢查 ``Blog`` 實體的
``setTitle`` 呼叫時能夠正確更新 ``$slug`` 屬性。

執行這些測試來檢驗 ``Blog`` 實體能否正常運作。

測試 Twig 外掛
~~~~~~~~~~~~~~~~~~~~~~~~~~

在上一個章節，我們建立了一個 Twig 外掛來轉換一個 ``\DateTime`` 實體變成字串來處理一個時段的持續時間，在
``src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php`` 建立一個檔案並且貼入下面內容。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php

    namespace Blogger\BlogBundle\Tests\Twig\Extensions;

    use Blogger\BlogBundle\Twig\Extensions\BloggerBlogExtension;

    class BloggerBlogExtensionTest extends \PHPUnit_Framework_TestCase
    {
        public function testCreatedAgo()
        {
            $blog = new BloggerBlogExtension();

            $this->assertEquals("0 seconds ago", $blog->createdAgo(new \DateTime()));
            $this->assertEquals("34 seconds ago", $blog->createdAgo($this->getDateTime(-34)));
            $this->assertEquals("1 minute ago", $blog->createdAgo($this->getDateTime(-60)));
            $this->assertEquals("2 minutes ago", $blog->createdAgo($this->getDateTime(-120)));
            $this->assertEquals("1 hour ago", $blog->createdAgo($this->getDateTime(-3600)));
            $this->assertEquals("1 hour ago", $blog->createdAgo($this->getDateTime(-3601)));
            $this->assertEquals("2 hours ago", $blog->createdAgo($this->getDateTime(-7200)));

            // Cannot create time in the future
            $this->setExpectedException('\InvalidArgumentException');
            $blog->createdAgo($this->getDateTime(60));
        }

        protected function getDateTime($delta)
        {
            return new \DateTime(date("Y-m-d H:i:s", time()+$delta));
        }
    }

這個類別設定的跟之前一樣，建立一個 ``testCreatedAgo()`` 方法來測試 Twig 外掛。在這個測試案例我們介紹了另一個 PHPUnit
方法 ``setExpectedException()`` ，這個方法應該要在你預期方法執行前被呼叫來產生一個例外。我們知道 Twig 外掛的
``createdAgo`` 方法無法處理未來的日期，同時也會產生一個 ``\Exception`` 。這個 ``getDateTime()`` 方法只是一個用來建立
``\DateTime`` 實例的輔助方法，差異是它前面沒有 ``test`` ，所以 PHPUnit 不會試著將它當測試案例執行。我們可以像之前一樣
簡單執行測試，不過我們也可以告訴 PHPUnit 來執行特定目錄的測試（像是它的子目錄）或檔案，執行下面指令。

.. code-block:: bash

    $ phpunit -c app src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php

這只會執行 ``BloggerBlogExtensionTest`` 這個檔案的測試， PHPUnit 會告訴我們測試失敗，輸出像這樣。

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Twig\Extension\BloggerBlogExtensionTest::testCreatedAgo
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -0 seconds ago
    +0 second ago

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php:14

我們預期第一個主張會傳回 ``0 seconds ago`` ，不過實際上沒有，second 這個字不是複數。讓我們來修正這個放在
``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php`` 的 Twig 外掛。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        // ..

        public function createdAgo(\DateTime $dateTime)
        {
            // ..
            if ($delta < 60)
            {
                // Seconds
                $time = $delta;
                $duration = $time . " second" . (($time === 0 || $time > 1) ? "s" : "") . " ago";
            }
            // ..
        }

        // ..
    }

重新執行 PHPUnit 測試，你應該會看到第一個主張正確的通過，不過我們的測試案例還是有錯，所以來檢查下面的輸出。

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Twig\Extension\BloggerBlogExtensionTest::testCreatedAgo
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -1 hour ago
    +60 minutes ago

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php:18

我們可以看到第五個主張錯誤的細節（留意輸出最下方的 18 ，這告訴我們主張錯誤的部份是在哪個檔案的那一行），看看這個測試案例
，我們可以看到這個 Twig 外掛的功能不正確，它應該要傳回 1 hour ago ，但實際上卻傳回 60 minutes ago 。如果我們檢查這個
``BloggerBlogExtension`` Twig 外掛的程式碼可以看到原因，我們在時間的比對用了包含語法，也就是說我們用了 ``<=`` 而不是
``<`` ，這個問題經常會花多個小時來找尋。更新這個放在 ``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php``
的 Twig 外掛來修正問題。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        // ..

        public function createdAgo(\DateTime $dateTime)
        {
            // ..

            else if ($delta < 3600)
            {
                // Mins
                $time = floor($delta / 60);
                $duration = $time . " minute" . (($time > 1) ? "s" : "") . " ago";
            }
            else if ($delta < 86400)
            {
                // Hours
                $time = floor($delta / 3600);
                $duration = $time . " hour" . (($time > 1) ? "s" : "") . " ago";
            }

            // ..
        }

        // ..
    }

現在我們透過下面指令重新執行所有測試。

.. code-block:: bash

    $ phpunit -c app

這個指令執行所有測試，所有的測試也都成功通過。雖然我們只有設計少數單元測試，你應該可以感受到寫程式時測試的強大與重要性
。當出現上面這樣的小錯誤，它們仍然是錯誤，測試也可以幫助我們在未來新功能加入專案時不會造成舊有功能的錯誤。到這裡就是單
元測試的結論了，我們會在後面章節看到更多的單元測試，你現在就可以試著加入一些自己覺得缺少的單元測試來測試功能

功能性測試
------------------

現在我們已經設計了一些單元測試，接著來看如何測試多個元件一起運作。功能性測試的第一個部份會介紹模擬瀏覽器請求來測試產生
的回應。

測試關於頁面
~~~~~~~~~~~~~~~~~~~~~~

我們開始測試 ``PageController`` 類別的關於頁面，由於關於頁面非常單純，所以是個適合作為開始的地方。建立一個檔案在
``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php`` 並且貼入下面內容。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class PageControllerTest extends WebTestCase
    {
        public function testAbout()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/about');

            $this->assertEquals(1, $crawler->filter('h1:contains("About symblog")')->count());
        }
    }

我們在大略看過 ``DefaultControllerTest`` 類別時就有見過一個非常相像的 Controller 測試，這是用來測試 symblog 關於頁面
，檢查 ``About symblog`` 這個字串是否出現在產生的 HTML ，指定是在 ``H1`` 標籤中。這個 ``PageControllerTest`` 類別沒有
繼承我們在單元測試提到的 ``\PHPUnit_Framework_TestCase`` ，而是繼承 ``WebTestCase`` 這個類別，他是 Symfony2 Framework
軟體包的一部分。

在之前提過， PHPUnit 測試類別必須繼承 ``\PHPUnit_Framework_TestCase`` ，不過當我們在多個測試案例都需要額外或常見功能時
，將這些封裝為它自己的類別然後在你的測試類別中繼承它是很實用的，這個 ``WebTestCase`` 就是這樣子做，它提供一些實用的方法
來在 Symfony2 中執行功能性測試，可以看一下放在 ``vendor/symfony/src/Symfony/Bundle/FrameworkBundle/Test/WebTestCase.php``
的 ``WebTestCase`` ，你會看到這個類別實際上繼承了 ``\PHPUnit_Framework_TestCase`` 類別。

.. code-block:: php

    // vendor/symfony/src/Symfony/Bundle/FrameworkBundle/Test/WebTestCase.php

    abstract class WebTestCase extends \PHPUnit_Framework_TestCase
    {
        // ..
    }

如果你看一下 ``WebTestCase`` 類別的 ``createClient()`` 方法，你可以看到它建立一個 Symfony2 核心的實例，繼續看下去你也會
注意到 ``environment`` 被設定為 ``test`` （除非被一個 ``createClient()`` 的參數覆蓋），這就是我們在上一章提到的 ``test``
環境。

看回到我們的測試類別，我們可以看到 ``createClient()`` 方法在測試開始執行時被呼叫，我們接著在用戶端呼叫 ``request()`` 方法
來針對網址 ``/about`` 模擬一個瀏覽器的 HTTP GET 請求（這就很像你透過瀏覽器訪問 ``http://symblog.dev/about`` ）。這個請求
傳回一個 ``Crawler`` 物件，它包含了 ``Response`` 。這個 ``Crawler`` 類別非常實用，因為它讓我們解析傳回的 HTML ，我們用這
個 ``Crawler`` 實例來檢查傳回 HTML 中的 ``H1`` 標籤是否包含文字 ``About symblog`` 。你會發現即使我們是繼承 ``WebTestCase`
類別，我們依舊使用之前提到的主張方法（記得 ``PageControllerTest`` 類別還是 ``\PHPUnit_Framework_TestCase`` 的子項目）。

我們用下面指令執行 ``PageControllerTest`` ，設計測試時，我們通常只執行目前處理中檔案的測試。當你要測試的對象變多，執行個別
測試會變成需要大量時間的工作。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

你應該會看到歡迎訊息 ``OK (1 test, 1 assertion)`` ，這讓我們知道執行了一個測試（ ``testAbout()`` ），以及一個主張（
``assertEquals()`` ）。

試著修改文字 ``About symblog`` 為 ``Contact`` ，然後重新執行測試。這個測試現在應該會錯物，因為找不到 ``Contact`` ，會造成
``asertEquals`` 變成 false 。

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Controller\PageControllerTest::testAbout
    Failed asserting that 0 matches expected 1.

在繼續之前把文字改回 ``About symblog`` 。

這個 ``Crawler`` 實例用來讓你解析 HTML 或 XML 文件（這表示 ``Crawler`` 只會在回應是 HTML 或 XML 時運作），我們可以用 ``Crawler``
來解析產生的回應，可以使用的方法包括 ``filter()`` 、 ``first()`` 、 ``last()`` 與 ``parents()`` ，如果你曾經使用過
`jQuery <http://jquery.com/>`_ ，你應該會對於 ``Crawler`` 感覺很熟悉。 ``Crawler`` 支援的解析方法完整清單可以參考 Symfony2
手冊的 `測試 <http://symfony.com/doc/current/book/testing.html#traversing>`_ 章節，我們接著會介紹更多 ``Crawler`` 的功能。

首頁
~~~~~~~~

雖然關於頁面的測試很簡單，它已經概要介紹了網頁功能性測試的基本原則。

 1. 建立用戶端
 2. 請求一個頁面
 3. 檢查回應

這是一個簡單的處理概要，事實上還有許多期它步驟我們可以做，像是點選連結、填寫與送出表單等。

我們來建立一個方法測試首頁。我們知道首頁的網址是 ``/`` ，它應該會顯示最新的部落格文章。在檔案
``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php`` 的 ``PageControllerTest`` 類別建立一個新方法 ``testIndex()``
並且放入下面內容。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testIndex()
    {
        $client = static::createClient();

        $crawler = $client->request('GET', '/');

        // Check there are some blog entries on the page
        $this->assertTrue($crawler->filter('article.blog')->count() > 0);
    }

你可以看到跟測試關於頁面一樣的步驟，執行它來看看是否能夠如預期般運作。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

接著做進一步的測試，部份功能性測試包含能夠複製使用者在網站的操作，使用者為了在頁面間移動會點選連結，我們就試著模擬這個操作來測
試當部落格標題點選後顯示部落格頁面的連結是否正確。用下面內容更新 ``PageControllerTest`` 類別的 ``testIndex()`` 方法。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testIndex()
    {
        // ..

        // Find the first link, get the title, ensure this is loaded on the next page
        $blogLink   = $crawler->filter('article.blog h2 a')->first();
        $blogTitle  = $blogLink->text();
        $crawler    = $client->click($blogLink->link());

        // Check the h2 has the blog title in it
        $this->assertEquals(1, $crawler->filter('h2:contains("' . $blogTitle .'")')->count());
    }

我們首先使用 ``Crawler`` 來解析第一個部落格標題連結的文字，這透過過濾器 ``article.blog h2 a`` 完成。這個過濾器會傳回
``article.blog`` 在 ``H2`` 標籤中的 ``a`` 標籤，要更了解這個部份，可以看看首頁用來顯示部落格的原始碼。

.. code-block:: html

    <article class="blog">
        <div class="date"><time datetime="2011-09-05T21:06:19+01:00">Monday, September 5, 2011</time></div>
        <header>
            <h2><a href="/app_dev.php/1/a-day-with-symfony2">A day with Symfony2</a></h2>
        </header>

        <!-- .. -->
    </article>
    <article class="blog">
        <div class="date"><time datetime="2011-09-05T21:06:19+01:00">Monday, September 5, 2011</time></div>
        <header>
            <h2><a href="/app_dev.php/2/the-pool-on-the-roof-must-have-a-leak">The pool on the roof must have a leak</a></h2>
        </header>

        <!-- .. -->
    </article>

你可以看到過濾器的 ``article.blog h2 a`` 結構就在首頁原始碼中，你也會發現不只一個 ``<article class="blog">`` ，這表示
``Crawler`` 過濾器會傳回一個集合，由於我們只需要第一個連結，我們在這個集合使用 ``first()`` 方法。最後我們使用 ``text()`` 方法
來解析連結文字，在這裡應該是 ``A day with Symfony2`` 。接著部落格標題的連結被點選來引導到部落格顯示頁，用戶端的 ``click()``
方法會接受一個連結物件，然後以 ``Crawler`` 實例傳回一個 ``Response`` ，你現在應該會注意到 ``Crawler`` 物件在功能性測試裡面是一
個很重要的部份。

``Crawler`` 物件現在包含部落格顯示頁的回應，我們需要測試引導的連結是否讓我們在正確的頁面，我們可以使用之前取得的 ``$blogTitle``
數值來檢查回應中的標題。

執行測試來確認首頁與部落格顯示頁的引導是否正確。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

現在你應該了解如何在網站頁面間透過引導來進行功能性測試，接著我們會看到表單的測試。

測試聯絡頁面
~~~~~~~~~~~~~~~~~~~~~~~~

symblog 的使用者可以在聯絡頁面 ``http://symblog.dev/contact`` 透過填寫表單送出聯繫諮詢，我們來測試這個表單送出過程是否正確。首
先我們需要列出在表單成功送出過程的概要（成功送出在這裡意思是表單沒有錯誤資料）。

 1. 引導到聯絡頁面
 2. 在表單輸入數值
 3. 送出表單
 4. 檢查信件是否送到 symblog
 5. 檢查用戶端的回應是否包含成功聯繫的提醒

到目前為止我們只知道如何進行第 1 步與第 5 步，我們現在要看看如何測試中間的 3 個步驟。

在檔案 ``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php`` 中的類別 ``PageControllerTest`` 新增一個方法
``testContact()`` 。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        $client = static::createClient();

        $crawler = $client->request('GET', '/contact');

        $this->assertEquals(1, $crawler->filter('h1:contains("Contact symblog")')->count());

        // Select based on button value, or id or name for buttons
        $form = $crawler->selectButton('Submit')->form();

        $form['blogger_blogbundle_enquirytype[name]']       = 'name';
        $form['blogger_blogbundle_enquirytype[email]']      = 'email@email.com';
        $form['blogger_blogbundle_enquirytype[subject]']    = 'Subject';
        $form['blogger_blogbundle_enquirytype[body]']       = 'The comment body must be at least 50 characters long as there is a validation constrain on the Enquiry entity';

        $crawler = $client->submit($form);

        $this->assertEquals(1, $crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count());
    }

我們用同樣的形式開始，建立一個請求到網址 ``/contact`` ，然後檢查頁面是否包含正確的 ``H1`` 標題。接著我們使用 ``Crawler`` 來找到
表單送出按鈕。為什麼我們找按鈕而不是表單？理由是一個表單也許包含多個按鈕，我們會想要個別點選。從找到的按鈕可以取得表單，我們可以透
過陣列下標語法 ``[]`` 來設定表單數值。最後表單會傳送給用戶端的 ``submit()`` 方法來實際將表單送出。我們一樣會接收到一個 ``Crawler``
實例，使用這個 ``Crawler`` 回應來檢查是否有快閃訊息出現在回應中。執行這個測試來確認所有功能都正確運作。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

這個測試失敗了，我們會看到 PHPUnit 產生下面的輸出。

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Controller\PageControllerTest::testContact
    Failed asserting that <integer:0> matches expected <integer:1>.

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php:53

    FAILURES!
    Tests: 3, Assertions: 5, Failures: 1.

這個輸出告訴我們在表單送出後的回應找不到快閃訊息，這是因為在 ``test`` 環境中並不會發生轉頁。在表單於 ``PageController`` 類別通過
檢查後會發生一個轉頁，這個轉頁沒有發生，我們需要明確的表示應該要進行轉頁。至於為什麼轉頁沒有發生，很簡單，你也許想要先檢查目前的回
應，我們很快就會在檢查信件是否送出時展示這個部份。更新 ``PageControllerTest`` 類別來設定用戶端去進行轉頁。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        // ..

        $crawler = $client->submit($form);

        // Need to follow redirect
        $crawler = $client->followRedirect();

        $this->assertEquals(1, $crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count());
    }

現在你執行 PHPUnit 測試時應該會通過，我們現在來看檢查聯絡表單送出程序的最後一步，第 4 步，檢查信件是否送給 symblog 。我們已經知道
在 ``test`` 環境信件不會被送出，因為下面設定。

.. code-block:: yaml

    # app/config/config_test.yml

    swiftmailer:
        disable_delivery: true

我們可以透過取得網頁配置器的資訊來測試信件是否送出，這就是為什麼用戶端不進行轉頁的重點，配置器的檢查必需要在轉頁發生前進行，因為在轉
頁後配置器的資訊會消失。用下面內容來更新 ``testContact()`` 方法。

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        // ..

        $crawler = $client->submit($form);

        // Check email has been sent
        if ($profile = $client->getProfile())
        {
            $swiftMailerProfiler = $profile->getCollector('swiftmailer');

            // Only 1 message should have been sent
            $this->assertEquals(1, $swiftMailerProfiler->getMessageCount());

            // Get the first message
            $messages = $swiftMailerProfiler->getMessages();
            $message  = array_shift($messages);

            $symblogEmail = $client->getContainer()->getParameter('blogger_blog.emails.contact_email');
            // Check message is being sent to correct address
            $this->assertArrayHasKey($symblogEmail, $message->getTo());
        }

        // Need to follow redirect
        $crawler = $client->followRedirect();

        $this->assertTrue($crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count() > 0);
    }

在表單送出後我們檢查配置器是否存在，因為它在目前環境有可能設定為停用。

.. tip::

    記住測試並不一定要在 ``test`` 執行，它們可以在 ``production`` 環境執行，這時候像配置器這樣的東西並不會存在。

如果我們可以取得配置器，我們建立一個請求來取得 ``swiftmailer`` 集合器，這個 ``swiftmailer`` 集合器在後端運作，用來取得信件服務的使
用狀況，我們可以藉此取得信件是否送出的資訊。

接著我們使用 ``getMessageCount()`` 方法來檢查是否有 1 個訊息送出，這對於確認至少一個信件是否送出也許足夠，不過它並沒有檢查信件是否
被送到正確的位置，如果信件被送到錯誤的信箱也許會有點麻煩或甚至造成損害，我們在這裡並不會檢查信箱是否正確。

現在我們執行測試來檢查是否正確運作。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

測試新增部落格評論
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我們現在用已經在聯絡頁面提過的知識來測試發布部落格評論的流程，一樣的，我們需要列出表單成功送出會經過的流程。

 1. 引導到部落格頁面
 2. 在表單輸入數值
 3. 送出表單
 4. 檢查新的評論是否被加入到部落格評論清單的最後
 5. 也檢查邊框區塊的最新評論是否在清單最上方

在 ``src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php`` 建立一個新檔案並且貼入下面內容。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class BlogControllerTest extends WebTestCase
    {
        public function testAddBlogComment()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/1/a-day-with-symfony');

            $this->assertEquals(1, $crawler->filter('h2:contains("A day with Symfony2")')->count());

            // Select based on button value, or id or name for buttons
            $form = $crawler->selectButton('Submit')->form();

            $crawler = $client->submit($form, array(
                'blogger_blogbundle_commenttype[user]'          => 'name',
                'blogger_blogbundle_commenttype[comment]'       => 'comment',
            ));

            // Need to follow redirect
            $crawler = $client->followRedirect();

            // Check comment is now displaying on page, as the last entry. This ensure comments
            // are posted in order of oldest to newest
            $articleCrawler = $crawler->filter('section .previous-comments article')->last();

            $this->assertEquals('name', $articleCrawler->filter('header span.highlight')->text());
            $this->assertEquals('comment', $articleCrawler->filter('p')->last()->text());

            // Check the sidebar to ensure latest comments are display and there is 10 of them

            $this->assertEquals(10, $crawler->filter('aside.sidebar section')->last()
                                            ->filter('article')->count()
            );

            $this->assertEquals('name', $crawler->filter('aside.sidebar section')->last()
                                                ->filter('article')->first()
                                                ->filter('header span.highlight')->text()
            );
        }
    }

我們這次直接跳到整個測試，在我們開始剖析程式碼前，執行這個檔案的測試來確認功能正確。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php

PHPUnit 應該會告訴你有 1 個測試成功執行，看看 ``testAddBlogComment()`` 的程式碼，我們可以看到一樣的形式，建立一個用戶端、請求一個
頁面以及檢查我們所在頁面是否正確。我們接著取得新增評論表單、送出表單，填寫表單數值的方法跟之前有點不同，這次我們使用用戶端
``submit()`` 方法的第 2 個參數來傳入表單的數值。

.. tip::

    我們也可以透過物件導向的介面來設定表單欄位的數值，下面是一些範例。

    .. code-block:: php

        // Tick a checkbox
        $form['show_emal']->tick();
        
        // Select an option or a radio
        $form['gender']->select('Male');

在表單送出後，我們要求用戶端去進行轉頁，這樣我們才能檢查回應。我們再次使用 ``Crawler`` 來取得最新部落格評論，它應該是我們剛剛送出的
那一筆。最後我們檢查邊框區塊的最新評論，這個評論是否也在清單中的第一筆。

部落格儲藏庫
~~~~~~~~~~~~~~~

在這一章功能性測試我們要介紹的最後一個部份是測試 Doctrine 2 的儲藏庫，建立一個檔案在
``src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php`` 並且放入下面內容。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php

    namespace Blogger\BlogBundle\Tests\Repository;

    use Blogger\BlogBundle\Repository\BlogRepository;
    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class BlogRepositoryTest extends WebTestCase
    {
        /**
         * @var \Blogger\BlogBundle\Repository\BlogRepository
         */
        private $blogRepository;

        public function setUp()
        {
            $kernel = static::createKernel();
            $kernel->boot();
            $this->blogRepository = $kernel->getContainer()
                                           ->get('doctrine.orm.entity_manager')
                                           ->getRepository('BloggerBlogBundle:Blog');
        }

        public function testGetTags()
        {
            $tags = $this->blogRepository->getTags();

            $this->assertTrue(count($tags) > 1);
            $this->assertContains('symblog', $tags);
        }

        public function testGetTagWeights()
        {
            $tagsWeight = $this->blogRepository->getTagWeights(
                array('php', 'code', 'code', 'symblog', 'blog')
            );

            $this->assertTrue(count($tagsWeight) > 1);

            // Test case where count is over max weight of 5
            $tagsWeight = $this->blogRepository->getTagWeights(
                array_fill(0, 10, 'php')
            );

            $this->assertTrue(count($tagsWeight) >= 1);

            // Test case with multiple counts over max weight of 5
            $tagsWeight = $this->blogRepository->getTagWeights(
                array_merge(array_fill(0, 10, 'php'), array_fill(0, 2, 'html'), array_fill(0, 6, 'js'))
            );

            $this->assertEquals(5, $tagsWeight['php']);
            $this->assertEquals(3, $tagsWeight['js']);
            $this->assertEquals(1, $tagsWeight['html']);

            // Test empty case
            $tagsWeight = $this->blogRepository->getTagWeights(array());

            $this->assertEmpty($tagsWeight);
        }
    }

由於我們要執行測試時需要一個正確的資料庫連線，我們再次繼承 ``WebTestCase`` ，因為這讓我們能夠啟用 Symfony2 核心。用下面指令來執行
這個測試。

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php

程式碼涵蓋範圍
-------------

在繼續之前，我們快速的看過程式碼涵蓋範圍。程式碼涵蓋範圍讓我們可以觀察測試執行時哪個部份的程式碼被執行，透過這個功能我們可以看到程式
碼的哪些部份沒有相關測試，並且決定我們是否要為這些部份設計測試。

要輸出應用程式的程式碼涵蓋範圍分析就執行下面指令

.. code-block:: bash

    $ phpunit --coverage-html ./phpunit-report -c app/

這會將程式碼涵蓋範圍分析輸出到 ``phpunit-report`` 資料夾，透過瀏覽器打開 ``index.html`` 就可以看到分析輸出。

更多的資訊可以參考 PHPUnit 手冊的 `程式碼涵蓋範圍分析 <http://www.phpunit.de/manual/current/en/code-coverage-analysis.html>`_ 章節。

結論
----------

我們已經提到一些關於測試重要的地方，我們也介紹了單元與功能性測試來確保我們網站功能正確執行。我們已經看到如何模擬瀏覽器請求以及如何使用
Symfony2 的 ``Crawler`` 類別檢查這些請求的回應。

接著我們會看到 Symfony2 的安全元件，特別是如何將它用在使用者管理上。我們也會在 symblog 管理區結合已經做好的 FOSUserBundle 。
