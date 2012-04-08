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

Homepage
~~~~~~~~

While the test for the about page was simple, it has outlined the basic principles
of functional testing the website pages.

 1. Create the client
 2. Request a page
 3. Check the response

This is a simple overview of the process, in fact there are a number of other
steps we could also do such as clicking links and populating and submitting
forms.

Lets create a method to test the homepage. We know the homepage is available
via the URL ``/`` and that is should display the latest blog posts. Add a new
method ``testIndex()`` to the ``PageControllerTest`` class located at
``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php`` as shown below.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testIndex()
    {
        $client = static::createClient();

        $crawler = $client->request('GET', '/');

        // Check there are some blog entries on the page
        $this->assertTrue($crawler->filter('article.blog')->count() > 0);
    }

You can see the same steps are taken as with the tests for the about page.
Run the test to ensure everything is working as expected.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Lets now take the testing a bit further. Part of functional testing involves being
able to replicate what a user would do on the site. In order for users to move
between pages on your website they click links. Lets simulate this action now
to test the links to the show blog page work correctly when the blog title is clicked.
Update the ``testIndex()`` method in the ``PageControllerTest`` class with the following.

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

The first thing we do it use the ``Crawler`` to extract the text within the first
blog title link. This is done using the filter ``article.blog h2 a``. This filter
is used return the ``a`` tag within the ``H2`` tag of the ``article.blog``
article. To understand this better, have a look at the markup used on the homepage
for displaying blogs.

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

You can see the filter ``article.blog h2 a`` structure in place in the homepage
markup. You'll also notice that there is more than one ``<article class="blog">`` in
the markup, meaning the ``Crawler`` filter will return a collection. As we only want
the first link, we use the ``first()`` method on the collection. Finally we use
the ``text()`` method to extract the link text, in this case it will be the text
``A day with Symfony2``. Next, the blog title link is clicked to navigate to the
blog show page. The client ``click()`` method takes a link object and returns the
``Response`` in a ``Crawler`` instance. You should by now be noticing that the
``Crawler`` object is a key part to functional testing.

The ``Crawler`` object now contains the Response for the blog show page. We need
to test that the link we navigated took us to the right page. We can use the
``$blogTitle`` value we retrieved earlier to check this against the title in the
Response.

Run the tests to ensure that navigation between the homepage and the blog show
pages is working correctly.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Now you have an understanding of how to navigate through the website pages
when functional testing, lets move onto testing forms.

Testing the Contact Page
~~~~~~~~~~~~~~~~~~~~~~~~

Users of symblog are able to submit contact enquiries by completing the form on
the contact page ``http://symblog.dev/contact``. Lets test that submissions
of this form work correctly. First we need to outline what should happen when
the form is successfully submitted (successfully submitted in this case means
there are no errors present in the form).

 1. Navigate to contact page
 2. Populate contact form with values
 3. Submit form
 4. Check email was sent to symblog
 5. Check response to client contains notification of successful contact

So far we have explored enough to be able to complete steps 1 and 5 only. We will
now look at how to test the 3 middle steps.

Add a new method ``testContact()`` to the ``PageControllerTest`` class located at
``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php``.

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

We begin in the usual fashion, making a request to the ``/contact`` URL, and
checking the page contains the correct ``H1`` title. Next we use the ``Crawler``
to select the form submit button. The reason we select the button and not the
form is that a form may contain multiple buttons that we may want to click
independently. From the selected button we are able to retrieve the form. We are
able to set the form values using the array subscript notation ``[]``.
Finally the form is passed to the client ``submit()`` method to actually
submit the form. As usual, we receive a ``Crawler`` instance back. Using the
``Crawler`` response we check to ensure the flash message is present in the returned
response. Run the test to check everything is functioning correctly.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

The tests failed. We are given the following output from PHPUnit.

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Controller\PageControllerTest::testContact
    Failed asserting that <integer:0> matches expected <integer:1>.

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php:53

    FAILURES!
    Tests: 3, Assertions: 5, Failures: 1.

The output is informing us that the flash message could not be found in the
response from the form submit. This is because when in the ``test`` environment,
redirects are not followed. When the form is successfully validated in the
``PageController`` class a redirect happens. This redirect is not being
followed; We need to explicitly say that the redirect should be followed. The
reason redirects are not followed is simple, you may want to check the current
response first. We will demonstrate this soon to check the email was sent.
Update the ``PageControllerTest`` class to set the client to follow the
redirect.

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

No when you run the PHPUnit tests they should pass. Lets now look at the final
step of checking the contact form submission process, step 4, checking an email
was sent to symblog. We already know that emails will not be delivered in the
``test`` environment due to the following configuration.

.. code-block:: yaml

    # app/config/config_test.yml

    swiftmailer:
        disable_delivery: true

We can test the emails were sent using the information gathered by the web profiler.
This is where the importance of the client not following redirects comes in. The
check on the profiler needs to be done before the redirect happens, as the information
in the profiler will be lost. Update the ``testContact()`` method with the following.

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

After the form submit we check to see if the profiler is available as it may have
been disabled by a configuration setting for the current environment.

.. tip::

    Remember tests don't have to be run in the ``test`` environment, they could be
    run on the ``production`` environment where things like the profiler wont be
    available.

If we are able to get the profiler we make a request to retrieve the
``swiftmailer`` collector. The ``swiftmailer`` collector works behind the scenes
to gather information about how the emailing service is used. We can use this to
get information regarding which emails have been sent.

Next we use the ``getMessageCount()`` method to check that 1 email was sent. This
maybe enough to ensure that at least an email is going to be sent, but it doesn't verify
that the email will be sent to the correct location. It could be very embarrassing
or even damaging for emails to be sent to the wrong email address. To check this
isn't the case we verify the email to address is correct.

Now re run the tests to check everything is working correctly.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Testing Adding Blog Comments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Lets now use the knowledge we have gained from the previous tests on the contact page
to test the process of submitting a blog comment.
Again we outline what should happen when the form is successfully
submitted.

 1. Navigate to a blog page
 2. Populate comment form with values
 3. Submit form
 4. Check new comment is added to end of blog comment list
 5. Also check sidebar latest comments to ensure comment is at top of list

Create a new file located at
``src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php``
and add in the following.

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

We jump straight in this time with the entire test. Before we begin dissecting the code,
run the tests for this file to ensure everything is working correctly.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php

PHPUnit should inform you that the 1 test was executed successfully. Looking at
the code for the ``testAddBlogComment()`` we can see things begin in the usual
format, creating a client, requesting a page and checking the page we are on is
correct. We then proceed to get the add comment form, and submit the form. The
way we populate the form values is slightly different than the previous version.
This time we use the 2nd argument of the client ``submit()`` method to pass in
the values for the form.

.. tip::

    We could also use the Object Oriented interface to set the values of the form fields.
    Some examples are shown below.

    .. code-block:: php

        // Tick a checkbox
        $form['show_emal']->tick();
        
        // Select an option or a radio
        $form['gender']->select('Male');

After submitting the form, we request the client should follow the redirect so we
can check the response. We use the ``Crawler`` again to get the last blog comment, which
should be the one we just submitted. Finally we check the latest comments in the
sidebar to check the comment is also the first one in the list.

Blog Repository
~~~~~~~~~~~~~~~

The last part of the functional testing we will explore in this chapter is
testing a Doctrine 2 repository. Create a new file located at
``src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php`` and add the
following content.

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

As we want to perform tests that require a valid connection to the database
we extend the ``WebTestCase`` again as this allows us to bootstrap the Symfony2
Kernel. Run this test for this file using the following command.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php

Code Coverage
-------------

Before we move on lets quickly touch on code coverage. Code coverage gives us an
insight into which parts of the code are executed when the tests are run. Using
this we can see the parts of our code that have no tests run on them, and
determine if we need to write test for them.

To output the code coverage analysis for your application run the following

.. code-block:: bash

    $ phpunit --coverage-html ./phpunit-report -c app/

This will output the code coverage analysis to the folder ``phpunit-report``.
Open the ``index.html`` file in your browser to see the analysis output.

See the `Code Coverage Analysis <http://www.phpunit.de/manual/current/en/code-coverage-analysis.html>`_
chapter in the PHPUnit documentation for more information.

Conclusion
----------

We have covered a number of key areas with regards to testing. We have explored
both unit and functional testing to ensure our website is functioning correctly.
We have seen how to simulate browser requests and how to use the Symfony2 ``Crawler``
class to check the Response from these requests.

Next we will look at the Symfony2 security component, and more specifically how to use it
for user management. We will also integrate the FOSUserBundle ready for us to work on the
symblog admin section.
