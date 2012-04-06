[Part 2] - 聯絡頁面：驗證、表單與發信
=======================================================

概要
--------

現在我們有基本的網頁樣板，現在就來讓其中一個頁面開始運作。我們會從最簡單的一頁開始，也就是聯絡頁。
在這個章節結束後，你會有一個聯絡頁讓使用者發送資訊給網頁管理員，這些諮詢會直接發出信件給網頁管理員。

在這個章節會展示下面幾個主題：

1. 驗證
2. 表單
3. 設定軟體包設定值

聯絡頁面
------------

網址路徑
~~~~~~~

如同在上一個章節我們建立關於頁面時，我們是從定義聯絡頁的網址路徑開始的。開啟 ``BloggerBlogBundle``
的網址路徑檔 ``src/Blogger/BlogBundle/Resources/config/routing.yml`` 並且附加下面的網址路徑規則。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_contact:
        pattern:  /contact
        defaults: { _controller: BloggerBlogBundle:Page:contact }
        requirements:
            _method:  GET

在這裡沒有新東西，規則符合樣式 ``/contact`` 與 HTTP ``GET`` 方法時會執行 ``BloggerBlogBundle`` 中
``Page`` controller 的 ``contact`` 方法。

Controller
~~~~~~~~~~

接著我們在 ``BloggerBlogBundle`` 的 ``Page`` Controller 加入聯絡頁的方法，檔案在
``src/Blogger/BlogBundle/Controller/PageController.php`` 。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    // ..
    public function contactAction()
    {
        return $this->render('BloggerBlogBundle:Page:contact.html.twig');
    }
    // ..

現在這個方法非常簡單，只是產生聯絡頁畫面，我們後面會回來處理這裡。

View
~~~~

在 ``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig`` 建立聯絡頁樣板並且加入下面
內容

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}Contact{% endblock%}

    {% block body %}
        <header>
            <h1>Contact symblog</h1>
        </header>

        <p>Want to contact symblog?</p>
    {% endblock %}

這個樣板也很簡單，它延伸了 ``BloggerBlogBundle`` 版面樣板，覆蓋標題區塊來設定自訂標題，並且定義一些
內容區塊的文字。

連結到這一頁
~~~~~~~~~~~~~~~~~~~

最後我們需要更新放在 ``app/Resources/views/base.html.twig`` 的應用程式樣板，放入連結指向聯絡頁。

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    {% block navigation %}
        <nav>
            <ul class="navigation">
                <li><a href="{{ path('BloggerBlogBundle_homepage') }}">Home</a></li>
                <li><a href="{{ path('BloggerBlogBundle_about') }}">About</a></li>
                <li><a href="{{ path('BloggerBlogBundle_contact') }}">Contact</a></li>
            </ul>
        </nav>
    {% endblock %}

如果用瀏覽器打開 ``http://symblog.dev/app_dev.php/`` 並且點選導覽列的聯絡連結，你應該可以看到一個
非常基本的聯絡頁面，現在我們已經設定好聯絡頁，接著要開始處理聯絡表單。這會切割為兩個獨立的部份，就是
驗證與表單。在我們開始介紹驗證與表單的概念前，我們需要思考要如何處理聯絡的資料。

聯絡實體
--------------

我們開始建立一個類別來代表來自使用者的一個諮詢，我們想要納入一些基本的資訊，像是名稱、主旨與諮詢內容。
建立一個檔案 ``src/Blogger/BlogBundle/Entity/Enquiry.php`` 並且貼入下面內容。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Enquiry.php

    namespace Blogger\BlogBundle\Entity;

    class Enquiry
    {
        protected $name;

        protected $email;

        protected $subject;

        protected $body;

        public function getName()
        {
            return $this->name;
        }

        public function setName($name)
        {
            $this->name = $name;
        }

        public function getEmail()
        {
            return $this->email;
        }

        public function setEmail($email)
        {
            $this->email = $email;
        }

        public function getSubject()
        {
            return $this->subject;
        }

        public function setSubject($subject)
        {
            $this->subject = $subject;
        }

        public function getBody()
        {
            return $this->body;
        }

        public function setBody($body)
        {
            $this->body = $body;
        }
    }

如同你看到的，這個類別只是定義一些受保護的屬性與他們的存取器，在這裡我們沒有定義屬性的驗證或是屬性與表
單元素的關聯，我們晚點會回來看這裡。


.. note::

    在這裡要簡短介紹一下 Symfony2 命名空間的用法，我們建立的一些實體類別設定了命名空間為
    ``Blogger\BlogBundle\Entity`` ，由於 Symfony2 的自動載入功能支援
    `PSR-0 標準 <http://groups.google.com/group/php-standards/web/psr-0-final-proposal?pli=1>`_
    ，命名空間會直接對應到軟體包的資料夾結構，而 ``Enquiry`` 實體類別會被放在
    ``src/Blogger/BlogBundle/Entity/Enquiry.php`` ，這樣可以確保 Symfony2 能夠正確的自動載入這個類別。

    至於 Symfony2 自動載入功能如何知道命名空間 ``Blogger`` 可以在 ``src`` 資料夾找到，這就需要感謝我們
    在 ``app/autoloader.php`` 的自動載入設定。

    .. code-block:: php

        // app/autoloader.php
        $loader->registerNamespaceFallbacks(array(
            __DIR__.'/../src',
        ));

    這個語法會註冊一個任何尚未註冊命名空間的最終處理，由於沒有註冊 ``Blogger`` 這個命名空間， Symfony2
    自動載入器會在 ``src`` 資料夾找尋需要的檔案。

    自動載入與命名空間在 Symfony2 是一個非常強大的概念，如果你遇到 PHP 無法找到類別的錯誤，通常這就是你
    在命名空間或資料夾結構出了錯誤，同時也要檢查這個命名空間是不是已經像上面那樣在自動載入器註冊，千萬
    不要嘗試直接用 PHP 的 ``require`` 或 ``include`` 來 ``修正`` 這個問題。

表單
-----

接著我們會建立表單， Symfony2 附帶了一個非常強大的表單架構，這可以讓處理表單這個沉悶的工作變得容易。如同所有
Symfony2 元件一樣，它可以在專案中被用在 Symfony2 以外的地方， `表單元件原始碼 <https://github.com/symfony/Form>`_
可以透過 Github 下載。我們會開始建立一個 ``AbstractType`` 類別來代表諮詢表單，我們可以直接在 controller 中
建立這個表單而不需要使用這個類別，不過將表單分離到獨立的類別讓我們在應用程式中可以重複使用這個表單。總之，
controller 應該要簡潔，它的目的是在 Model 與 View 之間作為膠水。

EnquiryType
~~~~~~~~~~~

建立一個檔案在 ``src/Blogger/BlogBundle/Form/EnquiryType.php`` 並且貼入下面內容。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Form/EnquiryType.php

    namespace Blogger\BlogBundle\Form;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class EnquiryType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('name');
            $builder->add('email', 'email');
            $builder->add('subject');
            $builder->add('body', 'textarea');
        }

        public function getName()
        {
            return 'contact';
        }
    }

這個 ``EnquiryType`` 類別使用了 ``FormBuilder`` 類別，當我們要建立表單時 ``FormBuilder`` 類別就是你最好的
朋友，它可以基於欄位的後設資料簡化定義欄位的操作。由於我們的 Enquiry 實體非常簡單，我們還沒定義任何後設資料，
所以 ``FormBuilder`` 預設都會將欄位類型設定為文字框，這在大部分欄位都合適，除了在內容欄位我們希望使用
``textarea`` ，還有信箱欄位我們希望運用 HTML5 新加入的 email 輸入類型。

.. note::

    一個需要注意的第方式， ``getName`` 方法需要傳回一個唯一的識別字串。

在 controller 建立表單
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

現在我們已經定義了 ``Enquiry`` 實體與 ``EnquiryType`` ，我們可以更新聯繫方法來使用它們。用下面內容取代
``src/Blogger/BlogBundle/Controller/PageController.php`` 。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    public function contactAction()
    {
        $enquiry = new Enquiry();
        $form = $this->createForm(new EnquiryType(), $enquiry);

        $request = $this->getRequest();
        if ($request->getMethod() == 'POST') {
            $form->bindRequest($request);

            if ($form->isValid()) {
                // Perform some action, such as sending an email

                // Redirect - This is important to prevent users re-posting
                // the form if they refresh the page
                return $this->redirect($this->generateUrl('BloggerBlogBundle_contact'));
            }
        }

        return $this->render('BloggerBlogBundle:Page:contact.html.twig', array(
            'form' => $form->createView()
        ));
    }

我們開始先建立一個 ``Enquiry`` 實體的實例，這個實體代表一個聯繫的資料。接著我們建立實際的表單，指定之前
建立的 ``EnquiryType`` ，然後傳給查詢實體物件。這個 ``createForm`` 方法會透過兩個藍圖來建立代表的表單。

由於這個 controller 方法會處理顯示與送出的表單，我們需要檢查 HTTP 方法。送出的表單通常會透過 ``POST``
方法，我們這個表單也沒有例外。如果請求的方法是 ``POST`` ，會呼叫 ``bindRequest`` 來傳送送出的資料給我們
``$enquiry`` 物件的屬性，在這裡 ``$enquiry`` 物件現在保留了使用者送出的資料。

接著我們驗證一下表單內容，只是我們還沒指定任何驗證器，所以表單的內容會直接通過檢查。

最後我們指定一個要顯示的樣板，需要注意我們現在也將表單在 view 的代表傳送給樣板，這個物件讓我們可以在 view
產生表單。

因為我們在 controller 使用了兩個新類別，我們需要匯入命名空間。用下面內容更新
``src/Blogger/BlogBundle/Controller/PageController.php`` ，這些語法應該要放在現有的 ``use`` 下面。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/PageController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    // Import new namespaces
    use Blogger\BlogBundle\Entity\Enquiry;
    use Blogger\BlogBundle\Form\EnquiryType;

    class PageController extends Controller
    // ..

產生表單
~~~~~~~~~~~~~~~~~~

感謝 Twig 提供的方法讓產生表單非常容易， Twig 提供一個分層系統來產生表單，讓你可以為整個實體產生表單，或
是獨立的錯誤與元素，就看你需要在哪個層次進行客製化。

為了展示 Twig 方法的強大，我們用下面的程式碼來產生整個表單。

.. code-block:: html

    <form action="{{ path('BloggerBlogBundle_contact') }}" method="post" {{ form_enctype(form) }}>
        {{ form_widget(form) }}

        <input type="submit" />
    </form>

這對於雛型或是簡單的表單很實用，只是它在需要延伸客製時就會有所侷限，而這個情形在表單經常發生。

在我們的聯絡表單中，我們會選擇中間的方法。用下面內容替代
``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig`` 的樣板程式碼。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}Contact{% endblock%}

    {% block body %}
        <header>
            <h1>Contact symblog</h1>
        </header>

        <p>Want to contact symblog?</p>

        <form action="{{ path('BloggerBlogBundle_contact') }}" method="post" {{ form_enctype(form) }} class="blogger">
            {{ form_errors(form) }}

            {{ form_row(form.name) }}
            {{ form_row(form.email) }}
            {{ form_row(form.subject) }}
            {{ form_row(form.body) }}

            {{ form_rest(form) }}

            <input type="submit" value="Submit" />
        </form>
    {% endblock %}

如同你所看到，我們使用 4 個新的 Twig 方法來產生表單。

第一個方法是 ``form_enctype`` ，設定了表單內容類型，這在表單處理檔案上傳時就會必須設定。我們的表單用不到
這個方法，不過養成習慣在所有表單加入它是建好事，也許未來你就會需要加入檔案上傳欄位。檔案上傳的除錯如果是
因為沒有設定內容類型，經常會讓人想破頭。

第二個方法 ``form_errors`` 會在表單驗證失敗時產生錯誤訊息。

第三個方法 ``form_row`` 輸出每個表單欄位的整個元素，這包含了欄位的錯誤、欄位標籤與實際的欄位元素。

最後我們使用 ``form_rest`` 方法，在表單最後使用這個方法會比較安全，它會產生你也許遺忘的欄位，像是隱藏欄位
以及 Symfony2 表單 CSRF 權杖。

.. note::

    跨網站偽造請求 (CSRF) 在 Symfony2 手冊的
    `表單章節 <http://symfony.com/doc/current/book/forms.html#csrf-protection>`_
    有詳細說明。


裝飾表單
~~~~~~~~~~~~~~~~

如果你看到 ``http://symblog.dev/app_dev.php/contact`` 的聯絡表單，你會發現它看起來不是很舒服。我們加入
一些風格來改善畫面。由於這些風格是針對我們部落格軟體包的表單，我們會在軟體包中建立這個風格表。請建立一個
檔案在 ``src/Blogger/BlogBundle/Resources/public/css/blog.css`` 並且貼入下面內容。

.. code-block:: css

    .blogger-notice { text-align: center; padding: 10px; background: #DFF2BF; border: 1px solid; color: #4F8A10; margin-bottom: 10px; }
    form.blogger { font-size: 16px; }
    form.blogger div { clear: left; margin-bottom: 10px; }
    form.blogger label { float: left; margin-right: 10px; text-align: right; width: 100px; font-weight: bold; vertical-align: top; padding-top: 10px; }
    form.blogger input[type="text"],
    form.blogger input[type="email"]
        { width: 500px; line-height: 26px; font-size: 20px; min-height: 26px; }
    form.blogger textarea { width: 500px; height: 150px; line-height: 26px; font-size: 20px; }
    form.blogger input[type="submit"] { margin-left: 110px; width: 508px; line-height: 26px; font-size: 20px; min-height: 26px; }
    form.blogger ul li { color: #ff0000; margin-bottom: 5px; }


我們需要讓應用程式知道我們想使用這個風格表，我們可以直接匯入這個風格表到聯絡樣板，不過由於其他樣板在後面也
會用到這個風格表，比較建議將它匯入到我們在第一章建立的 ``BloggerBlogBundle`` 版面，開啟位於
``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` 的 ``BloggerBlogBundle`` 並且替換下面內容。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    {% extends '::base.html.twig' %}

    {% block stylesheets %}
        {{ parent() }}
        <link href="{{ asset('bundles/bloggerblog/css/blog.css') }}" type="text/css" rel="stylesheet" />
    {% endblock %}

    {% block sidebar %}
        Sidebar content
    {% endblock %}

你可以看到我們定義了一個風格表區塊來覆蓋父樣板定義的同樣區塊，不過需要注意的是呼叫 ``parent`` 方法部份，這
會匯入放在 ``app/Resources/base.html.twig`` 父樣板中的風格表區塊內容，讓我們可以附加新的風格，畢竟我們並不
想替換已經存在的風格表。

為了要讓 ``asset`` 方法正確連結到相關資源，我們需要將軟體包資源複製或是連結到應用程式的 ``web`` 資料夾，這
可以這樣子做。

.. code-block:: bash

    $ php app/console assets:install web --symlink

.. note::

    如果你使用的作業系統不支援符號連結，像是 Windows ，你會需要拿掉符號連結選項。

    .. code-block:: bash

        php app/console assets:install web

    這個方法會實際從軟體包的 ``public`` 資料夾複製資源到應用程式的 ``web`` 資料夾，因為檔案是真的複製過去，
    你會需要在每次異動軟體包資源時執行這個指令。

現在如果重新整理畫面會看到表單有比較漂亮的風格了。

.. image:: /_static/images/part_2/contact.jpg
    :align: center
    :alt: symblog contact form

.. tip::

    除了 ``asset`` 方法提供了我們在使用資源需要的功能，其實還有一個更好的方法。 `Kris Wallsmith <https://github.com/kriswallsmith>`_
    製作了一個函式庫 `Assetic <https://github.com/kriswallsmith/assetic>`_ ，預設就放在 Symfony2 標準版本中。
    這個函式庫提供比標準 Symfony2 方式更好的資源管理效果， Assetic 讓我們在資源中執行過濾器來自動整合、最小
    化與壓縮，它還能在圖片檔案執行壓縮過濾器。 Assetic 讓我們在軟體包的公開資料夾直接參考資源，而不需要執行
    ``assets:install`` 指令，我們在後面章節會做更多介紹。

提交失敗
-----------------

如果動作快一點，你也許已經試著送出表單，然後看到 Symfony2 的錯誤訊息。

.. image:: /_static/images/part_2/post_error.jpg
    :align: center
    :alt: No route found for "POST /contact": Method Not Allowed (Allow: GET, HEAD)

這個錯誤告訴我們沒有網址路徑符合 ``/contact`` 的 HTTP POST 方法，目前網址路徑只接受 GET 與 HEAD 請求。這是
因為我們設定了這個網址路徑需要透過 GET 方式。

我們來更新 ``src/Blogger/BlogBundle/Resources/config/routing.yml`` 的網址路徑來接受 POST 請求。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_contact:
        pattern:  /contact
        defaults: { _controller: BloggerBlogBundle:Page:contact }
        requirements:
            _method:  GET|POST

.. tip::

    你也許想知道為什麼網址路徑在指定 GET 時還允許 HEAD 方法，因為 HEAD 就是一個 GET 請求，不過只會傳回
    HTTP Headers 。

現在當你上傳表單時，應該會以預期的方式運作，雖然預期還沒實際做那麼多。這個頁面會將你引導回聯絡表單。

驗證
----------

Symfony2 的驗證器讓我們執行資料驗證的工作，在處理表單送過來的資料時驗證是常見的任務。這個驗證在資料送入
資料庫時也需要執行， Symfony2 驗證器讓我們從可能用到的元件中分離驗證邏輯，像是表單元件或資料庫元件。這個
方法表示我們對一個物件會有一組驗證規則。

讓我們開始更新放在 ``src/Blogger/BlogBundle/Entity/Enquiry.php`` 的 ``Enquiry`` 實體來指定一些驗證器，
先確認你在檔案最上方加入了那 5 個 ``use`` 語法。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Enquiry.php

    namespace Blogger\BlogBundle\Entity;

    use Symfony\Component\Validator\Mapping\ClassMetadata;
    use Symfony\Component\Validator\Constraints\NotBlank;
    use Symfony\Component\Validator\Constraints\Email;
    use Symfony\Component\Validator\Constraints\MinLength;
    use Symfony\Component\Validator\Constraints\MaxLength;

    class Enquiry
    {
        // ..

        public static function loadValidatorMetadata(ClassMetadata $metadata)
        {
            $metadata->addPropertyConstraint('name', new NotBlank());

            $metadata->addPropertyConstraint('email', new Email());

            $metadata->addPropertyConstraint('subject', new NotBlank());
            $metadata->addPropertyConstraint('subject', new MaxLength(50));

            $metadata->addPropertyConstraint('body', new MinLength(50));
        }

        // ..

    }

要定義驗證器，我們需要實做靜態方法 ``loadValidatorMetadata`` ，它提供我們一個 ``ClassMetadata`` 物件，
我們可以使用這個物件來為實體屬性設定正確的條件。第一個語法在 ``name`` 屬性套用了 ``NotBlank`` 條件，這
個 ``NotBlank`` 驗證器跟字面上看來一樣簡單，它只會在驗證的資料不是空白時傳回 ``true`` 。接著我們設定
``email`` 屬性的驗證， Symfony2 驗證器服務提供了一個針對
`emails <http://symfony.com/doc/current/reference/constraints/Email.html>`_ 的驗證器，它甚至可以檢查
MX 記錄檔來確保網址是正確的。在 ``subject`` 屬性我們想要設定一個 ``NotBlank`` 與一個 ``MaxLength`` 條件
，你可以針對一個屬性設定數個你想要使用的驗證器。

`驗證器條件 <http://symfony.com/doc/current/reference/constraints.html>`_ 的完整清單可以在 Symfony2
參考文件找到，當然你也能夠
`建立自訂驗證器 <http://symfony.com/doc/current/cookbook/validation/custom_constraint.html>`_ 。

現在當你送出聯絡表單，你所送出的資料會被送到驗證限制，試著輸入一個錯誤的信箱，你應該可以看到一個錯誤訊息
告訴你信箱是錯誤的。每個驗證器提供了一個預設訊息，如果需要可以直接覆寫。要修改信箱驗證的輸出訊息可以這樣
子做：

.. code-block:: php

    $metadata->addPropertyConstraint('email', new Email(array(
        'message' => 'symblog does not like invalid emails. Give me a real one!'
    )));

.. tip::

    如果你使用了一個支援 HTML5 的瀏覽器（通常是有），它會用 HTML5 訊息提醒必需要符合這個條件。這是用戶端
    驗證， Symfony2 會基於你的 ``Entity`` 後設資料設定適合的 HTML5 條件。在 email 元素你可以看到輸出的
    HTML 像這樣。

    .. code-block:: html

        <input type="email" value="" required="required" name="contact[email]" id="contact_email">

    它使用了新的 HTML5 輸入類型之一，包括 email 與設定 required 屬性。用戶端驗證的好處是它不需要在驗證表
    單時與伺服器來來回回，不過不能夠只靠用戶端的驗證，你應該要驗證所有送到伺服器端的資料，因為使用者可以輕
    易跳過用戶端的驗證。

發送信件
-----------------

我們的聯絡表單是讓使用者發出諮詢，不過現在還沒有發生，我們更新 controller 來送出信件給網站管理員。 Symfony2
包含了完整的 `Swift Mailer <http://swiftmailer.org/>`_ 函式庫來寄送郵件，它非常強大，我們只會介紹這個函式
庫的一些表面工夫。

設定 Swift Mailer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Swift Mailer 在 Symfony2 標準版原本就設定好可以運作，不過我們還是需要設定一些發送信件的參數與認證資訊。開啟
設定檔 ``app/config/parameters.ini`` 找到 ``mailer_`` 開頭的設定。

.. code-block:: text

    mailer_transport="smtp"
    mailer_host="localhost"
    mailer_user=""
    mailer_password=""

Swift Mailer 提供許多發送信件的方式，包括使用 SMTP 伺服器、使用主機裝好的 sendmail 或甚至透過一個 GMail 帳號。
為了要簡單化，我們使用一個 GMail 帳號，請像下面這樣更新參數，記得調整為你自己的帳號、密碼。

.. code-block:: text

    mailer_transport="gmail"
    mailer_encryption="ssl"
    mailer_auth_mode="login"
    mailer_host="smtp.gmail.com"
    mailer_user="your_username"
    mailer_password="your_password"

.. warning::

    如果你有使用版本控制系統 (VCS) ，請一定要注意，特別是在你的檔案庫可以公開存取時，因為你的 GMail 帳號與密碼會
    被放進檔案庫，任何人都看得到。你需要確認 ``app/config/parameters.ini`` 是否已經加入 VCS 的忽略清單。一個常見
    的解決方法是在放有敏感資訊的檔案名稱後面加入一些綴字，像是在 ``app/config/parameters.ini`` 後面加 ``.dist``
    。接著在這個檔案提供一些預設的設定，把實際的設定檔 ``app/config/parameters.ini`` 加入到 VCS 的忽略清單。接著
    可以佈署 ``*.dist`` 檔案到專案中，讓開發者移除 ``.dist`` 副檔名然後填入需要的設定。

更新 controller
~~~~~~~~~~~~~~~~~~~~~

用下面內容更新放在 ``src/Blogger/BlogBundle/Controller/PageController.php`` 的 ``Page`` controller 。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php

    public function contactAction()
    {
        // ..
        if ($form->isValid()) {

            $message = \Swift_Message::newInstance()
                ->setSubject('Contact enquiry from symblog')
                ->setFrom('enquiries@symblog.co.uk')
                ->setTo('email@email.com')
                ->setBody($this->renderView('BloggerBlogBundle:Page:contactEmail.txt.twig', array('enquiry' => $enquiry)));
            $this->get('mailer')->send($message);

            $this->get('session')->setFlash('blogger-notice', 'Your contact enquiry was successfully sent. Thank you!');

            // Redirect - This is important to prevent users re-posting
            // the form if they refresh the page
            return $this->redirect($this->generateUrl('BloggerBlogBundle_contact'));
        }
        // ..
    }

當你使用 Swift Mailer 函式庫建立了一個 ``Swift_Message`` 實例，就可以開始發信。

.. note::

    由於 Swift Mailer 函式庫並沒有使用命名空間，我們需要在 Swift Mailer 類別名稱前面放一個 ``\`` ，讓 PHP 回到
    `全域空間 <http://www.php.net/manual/en/language.namespaces.global.php>`_ 。所有沒使用命名空間的類別與函式都
    需要在前面加個 ``\`` ，如果在 ``Swift_Message`` 類別前面沒有加這個符號， PHP 會將這個類別視為使用目前的命名空間
    ，像這裡就是 ``Blogger\BlogBundle\Controller`` ，就會造成一個錯誤。

我們也在 session 設定了一個 ``flash`` 訊息，閃光訊息是只會保留一個請求週期的訊息，在這之後就會自動被 Symfony2 清除。
這個 ``flash`` 訊息會在聯絡樣板顯示，提醒使用者諮詢的內容已經送出。由於 ``flash`` 訊息只保留一個請求週期，它們非常適
合用來提醒使用者上個操作已經成功。

要顯示 ``flash`` 訊息，我們需要更新位於 ``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig`` 的聯絡
樣板，用下面內容去取代。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig #}

    {# rest of template ... #}
    <header>
        <h1>Contact symblog</h1>
    </header>

    {% if app.session.hasFlash('blogger-notice') %}
        <div class="blogger-notice">
            {{ app.session.flash('blogger-notice') }}
        </div>
    {% endif %}

    <p>Want to contact symblog?</p>

    {# rest of template ... #}

這會檢查 ``flash`` 訊息中是否有識別字元為 'blogger-notice' 的訊息，並且將它輸出。

註冊網站管理員信箱
~~~~~~~~~~~~~~~~~~~~~~~~

Symfony2 提供一個設定系統，我們可以用來定義自己的設定。我們會用這個系統來設定網站管理員的信箱，而不是直接寫入到上面的
controller ，這樣一來我們就可以輕易的重複運用這個數值在其他地方，而不需要複製重複的程式碼。進一步的，當你的部落格流量
非常大且諮詢數量多到你無法處理，你可以輕易更新這個信箱來將它轉給你的助理。建立一個檔案放在
``src/Blogger/BlogBundle/Resources/config/config.yml`` 並且貼入下面內容。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/config.yml
    parameters:
        # Blogger contact email address
        blogger_blog.emails.contact_email: contact@email.com

定義參數時，建議可以將參數名稱切割為多個元件，第一個部份是小寫的軟體包名稱，使用底線來區隔多個單字。在我們的範例中，我們
將 ``BloggerBlogBundle`` 轉換為 ``blogger_blog`` ，而其他部份的參數名稱用多個以小數點 . 字元分隔，這讓我們可以在邏輯上
把參數組合在一起。

要讓 Symfony2 應用使用新參數，我們需要將設定匯入到主要的應用程式設定，檔案放在 ``app/config/config.yml`` 。要做到這個部
份，更新檔案最上面的 ``imports`` 指令成下面這樣。

.. code-block:: yaml

    # app/config/config.yml
    imports:
        # .. existing import here
        - { resource: @BloggerBlogBundle/Resources/config/config.yml }

匯入路徑是檔案的實體位置， ``@BloggerBlogBundle`` 指令會解決 ``BloggerBlogBundle`` 對應到路徑 ``src/Blogger/BlogBundle``
之類的問題。

最後讓我們更新聯繫方法來使用這個參數。

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php

    public function contactAction()
    {
        // ..
        if ($form->isValid()) {

            $message = \Swift_Message::newInstance()
                ->setSubject('Contact enquiry from symblog')
                ->setFrom('enquiries@symblog.co.uk')
                ->setTo($this->container->getParameter('blogger_blog.emails.contact_email'))
                ->setBody($this->renderView('BloggerBlogBundle:Page:contactEmail.txt.twig', array('enquiry' => $enquiry)));
            $this->get('mailer')->send($message);

            // ..
        }
        // ..
    }

.. tip::

    由於設定檔案在應用程式設定檔最上方匯入，我們在應用程式中可以輕易覆蓋任何匯入的參數，例如將下面內容加入到 ``app/config/config.yml``
    最下面會覆蓋這個參數在軟體包的設定。

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            # Blogger contact email address
            blogger_blog.emails.contact_email: assistant@email.com

    這些客製允許軟體包提供一些敏感資料的預設值，讓應用程式可以覆寫過去。

.. note::

    雖然用這個方法可以輕易建立軟體包設定參數， Symfony2 還提供軟體包一個方法
    `揭露一個語意設定 <http://symfony.com/doc/current/cookbook/bundles/extension.html>`_
    ，我們在後面的教學會介紹它。

建立信件樣板
~~~~~~~~~~~~~~~~~~~~~~~~~

信件內容設定為透過一個樣板產生，建立一個樣板在 ``src/Blogger/BlogBundle/Resources/views/Page/contactEmail.txt.twig`` 並且放入
下面內容。

.. code-block:: text

    {# src/Blogger/BlogBundle/Resources/views/Page/contactEmail.txt.twig #}
    A contact enquiry was made by {{ enquiry.name }} at {{ "now" | date("Y-m-d H:i") }}.

    Reply-To: {{ enquiry.email }}
    Subject: {{ enquiry.subject }}
    Body:
    {{ enquiry.body }}

信件內容只是使用者送出的諮詢內容。

你也許會注意到這個樣板的副檔名跟我們之前建立的不一樣，它使用 ``.txt.twig`` 作為副檔名。副檔名的第一部份 ``.txt`` 指定了這個檔案要
產生的格式，常見的格式包括 .txt 、 .html 、 .css 、 .js 、 .xml 與 .json ，而副檔名的最後則是指定要使用的樣板引擎，這裡是使用 Twig
。副檔名如果是 ``.php`` 就會改使用 PHP 來產生樣板。

現在當你送出一個諮詢，一封信件就會寄送到 ``blogger_blog.emails.contact_email`` 參數所設定的信箱。

.. tip::

    Symfony2 允許我們在不同的 Symfony2 環境設定 Swift Mailer 函式庫的行為，我們已經看到這個用在 ``test`` 環境。 Symfony 2 標準版
    設定的 Swift Mailer 是不會在執行 ``test`` 環境時發出信件，這是設定在 ``app/config/config_test.yml`` 的環境中。

    .. code-block:: yaml

        # app/config/config_test.yml
        swiftmailer:
            disable_delivery: true

    在 ``dev`` 環境複製這個功能很實用，畢竟你不會想在開發過程中意外送出信件到錯誤的信箱。要做到這樣可以將上面的設定加入到
    ``app/config/config_dev.yml`` 的 ``dev`` 部份。

    你也許想知道現在如何測試信件是否寄出，特別是它們的內容為何，因為它們不會再實際發送信件到信箱。 Symfony2 有一個解決方案是透過開發
    工具列，當有信件送出時，工具列會出現一個信件提醒圖示，它會包含 Swift Mailer 送出信件的所有資訊。

    .. image:: /_static/images/part_2/email_notifications.jpg
        :align: center
        :alt: Symfony2 toolbar show email notifications

    如果你在發送信件後執行頁面引導，就像我們在這個聯絡表單做的一樣，你會需要在 ``app/config/config_dev.yml`` 設定 ``intercept_redirects``
    為 true ，這樣才能在工具列看到信件提醒。

    我們也可以設定 Swift Mailer 在 ``dev`` 環境時將所有信件寄到指定信箱，只要將下面內容放到 ``app/config/config_dev.yml`` 的 ``dev``
    設定中。

    .. code-block:: yaml

        # app/config/config_dev.yml
        swiftmailer:
            delivery_address:  development@symblog.dev

結論
----------

我們已經展示了建立任何網站都會需要的基礎，表單，背後的概念。 Symfony2 提供了非常好用的驗證與表單函式庫，讓我們可以將驗證邏輯從表單獨立
出來，讓它可以被用在應用程式的其他部份（像是 Model）。我們也介紹了自訂參數設定如何在我們的應用中使用。

接著我們會介紹這個教學的重點，也就是 Model 。我們會介紹 Doctrine 2 並且用它來定義部落格 Model 。我們也會建立檢視部落格頁面以及探討資料
裝置的概念。
page and explore the concept of Data fixtures .
