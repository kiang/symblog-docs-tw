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

Swift Mailer is already configured out of the box to work in the Symfony2 Standard
Distribution, however we need to configure some settings regarding sending methods,
and credentials. Open up the parameters file located at ``app/config/parameters.ini`` and
find the settings prefixed with ``mailer_``.

.. code-block:: text

    mailer_transport="smtp"
    mailer_host="localhost"
    mailer_user=""
    mailer_password=""

Swift Mailer provides a number of methods for sending emails, including using an
SMTP server, using a local install of sendmail, or even using a GMail account.
For simplicity we will use a GMail account. Update the parameters with the following,
substituting your username and password where necessary.

.. code-block:: text

    mailer_transport="gmail"
    mailer_encryption="ssl"
    mailer_auth_mode="login"
    mailer_host="smtp.gmail.com"
    mailer_user="your_username"
    mailer_password="your_password"

.. warning::

    Be careful if you are using a Version Control System (VCS) like Git for
    your project, especially if your repository is publicly accessible as your
    GMail username and password will be committed to the repository and will be
    available for anybody to see. You should make sure the file
    ``app/config/parameters.ini`` is added to the ignore list of your VCS. A common
    approach to this problem is to suffix the file name of the file
    that has sensitive information such as ``app/config/parameters.ini`` with ``.dist``.
    You then provide sensible defaults for the settings in this file and add the
    actual file, i.e. ``app/config/parameters.ini`` to you VCS ignore list.
    You can then deploy the ``*.dist`` file with your project and allow the developer to
    remove the ``.dist`` extension and fill in the required settings.

Update the controller
~~~~~~~~~~~~~~~~~~~~~

Update the ``Page`` controller located at
``src/Blogger/BlogBundle/Controller/PageController.php``
with the content below.

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

When you have used the Swift Mailer library to create a ``Swift_Message`` instance,
that can be sent as an email.

.. note::

    As the Swift Mailer library does not use namespaces, we need to
    prefix the Swift Mailer class with a ``\``. This tells PHP
    to escape back to the
    `global space <http://www.php.net/manual/en/language.namespaces.global.php>`_.
    You will need to prefix all classes and functions that are not
    namespaced with a ``\``. If you did not place this prefix before the
    ``Swift_Message`` class PHP would look for the class in the
    current namespace, which in this example is
    ``Blogger\BlogBundle\Controller``, causing an error to be thrown.

We have also set a ``flash`` message on the session. Flash messages are messages
that persist for exactly one request. After that they are
automatically cleaned up by Symfony2. The ``flash`` message will be displayed in the
contact template to inform the user the enquiry has been sent. As ``flash`` message
only persist for exactly one request, they are perfect for notifying the user of
the success of the previous actions.

To display the ``flash`` message we need to update the contact template
located at ``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig``.
Update the content of the template with the following.

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

This checks to see if a ``flash`` message with the identifier
'blogger-notice' is set and outputs it.

Register webmaster email
~~~~~~~~~~~~~~~~~~~~~~~~

Symfony2 provides a configuration system that we can use to define our own
settings. We will use this system to set the webmaster email address rather
than hard coding the address in the controller above. That way we can easily
reuse this value in other places without code duplication. Further, when your
blog has generated so much traffic the enquiries become too much for you
to deal with, you can easily update the email address to pass the emails
onto your assistant. Create a new file at
``src/Blogger/BlogBundle/Resources/config/config.yml`` and paste in the
following.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/config.yml
    parameters:
        # Blogger contact email address
        blogger_blog.emails.contact_email: contact@email.com

When defining parameters it is good practice to break the parameter name into a number
of components. The first part should be a lower cased version of the bundle name
using underscores to separate words. In our example we have transformed the
bundle ``BloggerBlogBundle`` into ``blogger_blog``. The remaining part of the
parameter name can contain any number of parts separated by the . (period) character.
This allows us to logically group parameters together.

In order for the Symfony2 application to use the new parameters, we need to import
the config into the main application config file located at ``app/config/config.yml``.
To achieve this, update the ``imports`` directive at the top of the file to the following.

.. code-block:: yaml

    # app/config/config.yml
    imports:
        # .. existing import here
        - { resource: @BloggerBlogBundle/Resources/config/config.yml }

The import path is the physical location of the file on disk. The
``@BloggerBlogBundle`` directive will resolve to the path of the
``BloggerBlogBundle`` which is ``src/Blogger/BlogBundle``.

Finally let's update the contact action to use the parameter.

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

    As the config file is imported at the top of the application configuration file
    we can easily override any of the imported parameters in the application.
    For example, adding the following to the bottom of ``app/config/config.yml``
    would override the bundle set value for the parameter.

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            # Blogger contact email address
            blogger_blog.emails.contact_email: assistant@email.com

    These customisation allow for the bundle to provide sensible defaults for values
    where the application can override them.

.. note::

    While its easy to create bundle configuration parameters using this method
    Symfony2 also provides a method where you
    `expose a Semantic Configuration <http://symfony.com/doc/current/cookbook/bundles/extension.html>`_
    for a bundle. We will explore this method later in the tutorial.

Create the Email template
~~~~~~~~~~~~~~~~~~~~~~~~~

The body of the email is set to render a template. Create this template at
``src/Blogger/BlogBundle/Resources/views/Page/contactEmail.txt.twig`` and add
the following.

.. code-block:: text

    {# src/Blogger/BlogBundle/Resources/views/Page/contactEmail.txt.twig #}
    A contact enquiry was made by {{ enquiry.name }} at {{ "now" | date("Y-m-d H:i") }}.

    Reply-To: {{ enquiry.email }}
    Subject: {{ enquiry.subject }}
    Body:
    {{ enquiry.body }}

The content of the email is just the enquiry the user submitted.

You may have also noticed the extension of this template is different to the other
templates we have created. It uses the extension ``.txt.twig``. The first part
of the extension, ``.txt`` specifies the format of the file to generate.
Common formats here include, .txt, .html, .css, .js, .xml and .json. The last part of the
extension specifies which templating engine to use, in this case Twig. An extension
of ``.php`` would use PHP to render the template instead.

When you now submit an enquiry, an email will be sent to the address set in the
``blogger_blog.emails.contact_email`` parameter.

.. tip::

    Symfony2 allows us to configure the behavior of the Swift Mailer library
    while operating in different Symfony2 environments. We can already see this
    in use for the ``test`` environment. By default, the Symfony 2 Standard
    Distribution configures Swift Mailer to not send emails when running in the ``test``
    environment. This is set in the test configuration file located at
    ``app/config/config_test.yml``.

    .. code-block:: yaml

        # app/config/config_test.yml
        swiftmailer:
            disable_delivery: true

    It could be useful to duplicate this functionality for the ``dev`` environment.
    After all, you don't want to accidentally send an email to the wrong email address
    while developing. To achieve this, add the above configuration to the
    ``dev`` configuration file located at ``app/config/config_dev.yml``.

    You may be wondering how you can now test that the emails are being sent, and
    more specifically the content of them, seeing as they will no longer be delivered
    to an actual email address. Symfony2 has a solution for this via the developer
    toolbar. When an email is sent an email notification icon will appear in the toolbar
    that has all the information about the email that Swift Mailer would have delivered.

    .. image:: /_static/images/part_2/email_notifications.jpg
        :align: center
        :alt: Symfony2 toolbar show email notifications

    If you perform a redirect after sending an email, like we do for the contact form,
    you will need to set the ``intercept_redirects`` setting in ``app/config/config_dev.yml``
    to true in order to see the email notification in the toolbar.

    We could have instead configured Swift Mailer to send all emails to a specific
    email address in the ``dev`` environment by placing the following
    setting in the ``dev`` configuration file located at ``app/config/config_dev.yml``.

    .. code-block:: yaml

        # app/config/config_dev.yml
        swiftmailer:
            delivery_address:  development@symblog.dev

Conclusion
----------

We have demonstrated the concepts behind creating one of the most fundamental part of any
website: forms. Symfony2 comes complete with an excellent Validator and Form library
that allows us to separate validation logic out of the form so it can be used
by other parts of the application (such as the Model). We were also introduced to
setting custom configuration settings that can be read into our application.

Next we will look at a big part of this tutorial, The Model. We will introduce
Doctrine 2 and use it to define the blog Model. We will also build the show blog
page and explore the concept of Data fixtures .
