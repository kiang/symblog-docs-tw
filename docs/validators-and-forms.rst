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

Now we have defined the ``Enquiry`` entity and ``EnquiryType``, we can update the contact action to
use them. Replace the content of the contact action located at
``src/Blogger/BlogBundle/Controller/PageController.php`` with the following.

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

We begin by creating an instance of the ``Enquiry`` entity. This entity represent
the data of a contact enquiry. Next we create the actual form. We specify the
``EnquiryType`` we created earlier, and pass in our enquiry entity object. The
``createForm`` method is able to use these 2 blueprints to create a form representation.

As this controller action will deal with displaying and processing the submitted form, we
need to check the HTTP method. Submitted forms are usually sent via ``POST``, and our
form will be no exception. If the request method is ``POST``, a call to ``bindRequest``
will transform the submitted data back to the members of our ``$enquiry`` object. At
this point the ``$enquiry`` object now holds a representation of what the user submitted.

Next we make a check to see if the form is valid. As we have specified no validators
at the point, the form will always be valid.

Finally we specify the template to render. Note that we are now also passing
over a view representation of the form to the template. This object allow us to
render the form in the view.

As we have used 2 new classes in our controller we need to import the namespaces.
Update the controller file located at ``src/Blogger/BlogBundle/Controller/PageController.php``
with the following. The statements should be placed under the existing ``use`` statement.

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

Rendering the form
~~~~~~~~~~~~~~~~~~

Thanks to Twig's methods rendering forms is very simple. Twig provides a
layered system for form rendering that allows you to render the form as one entire
entity, or as individual errors and elements, depending on the level of customisation
you require.

To demonstrate the power of Twig's methods we can use the following snippet
to render the entire form.

.. code-block:: html

    <form action="{{ path('BloggerBlogBundle_contact') }}" method="post" {{ form_enctype(form) }}>
        {{ form_widget(form) }}

        <input type="submit" />
    </form>

While this is very useful for prototyping and simple forms it has its limitations
when extended customisations are needed, which is often the case with forms.

For our contact form, we will opt for the middle ground. Replace the template
code located at ``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig``
with the following.

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

As you can see, we use 4 new Twig methods to render the form.

The first method ``form_enctype`` sets the form content type. This must be set
when your form deals with file uploads. Our form doesn't have any use for this
method but its always good practice to use this on all forms in the event
that you may add file uploads in the future. Debugging a form that handles file
uploads that has no content type set can be a real head scratcher!

The second method ``form_errors`` will render any errors the form has in the event
that validation failed.

The third method ``form_row`` outputs the entire elements related to each form field.
This include any errors for the field, the field label and the actual field
element.

Finally we use the ``form_rest`` method. Its always a safe bet to use the method at the
end of the form to render any fields you may have forgotten, including hidden
fields and the Symfony2 Form CSRF token.

.. note::

    Cross-site request forgery (CSRF) is explained in details in the
    `Forms chapter <http://symfony.com/doc/current/book/forms.html#csrf-protection>`_
    of the Symfony2 book.


Styling the form
~~~~~~~~~~~~~~~~

If you view the contact form now via ``http://symblog.dev/app_dev.php/contact``
you will notice it doesn't look that appealing. Lets add some styles to improve
this look. As the styles are specific to the form within our Blog bundle
we will create the styles in a new stylesheet within the bundle itself. Create a
new file located at ``src/Blogger/BlogBundle/Resources/public/css/blog.css`` and
paste in the following content.

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


We need to let the application know we want to use this stylesheet. We could
import the stylesheet into the contact template but as other templates will
also use this stylesheet later, it makes sense to import it into the
``BloggerBlogBundle`` layout we created in chapter 1. Open up the
``BloggerBlogBundle`` layout located at
``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` and replace
with the following content.

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

You can see we have defined a stylesheet block to override the stylesheet block
defined in the parent template. However, its important to notice the call to the
``parent`` method. This will import the content from the stylesheets block in
the parent template located at ``app/Resources/base.html.twig``, allowing us
to append our new stylesheet. After all, we don't want to replace the existing stylesheets.

In order for the ``asset`` function to correctly link up the the resource we need to
copy over or link the bundle resources into the applications ``web`` folder. This can
be done with the following

.. code-block:: bash

    $ php app/console assets:install web --symlink

.. note::

    If you are using an Operating System that doesn't support symlinks such as
    Windows you will need to drop the symlink option as follows.

    .. code-block:: bash

        php app/console assets:install web

    This method will actually copy the resources from the bundles ``public`` folder into the
    applications ``web`` folder. As the files are actually copied, you will need to run
    this task every time you make a change to a bundles public resource.

Now if you refresh the contact page the form will be beautifully styled.

.. image:: /_static/images/part_2/contact.jpg
    :align: center
    :alt: symblog contact form

.. tip::

    While the ``asset`` function provides the functionality we require to use
    resources, there is a better alternative for this. The
    `Assetic <https://github.com/kriswallsmith/assetic>`_ library by
    `Kris Wallsmith <https://github.com/kriswallsmith>`_ is bundled
    with the Symfony2 standard distribution by default. This library
    provides asset management far beyond the standard Symfony2 capabilities.
    Assetic allows us to run filters on assets to automatically combine,
    minify and gzip them. It can also run compression filters on images. Assetic
    further allows us to reference resources directly within the bundles public
    folder without having to run the ``assets:install`` task. We will explore the
    use of Assetic in later chapters.

Failure to submit
-----------------

The eager ones among you may have already tried to submit the form to be
greeted with a Symfony2 error.

.. image:: /_static/images/part_2/post_error.jpg
    :align: center
    :alt: No route found for "POST /contact": Method Not Allowed (Allow: GET, HEAD)

This error is telling us that there is no route to match ``/contact``
for the HTTP method POST. The route only accepts GET and HEAD requests.
This is because we configured our route with the method requirement of
GET.

Lets update the contact route located at
``src/Blogger/BlogBundle/Resources/config/routing.yml`` to also allow POST requests.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_contact:
        pattern:  /contact
        defaults: { _controller: BloggerBlogBundle:Page:contact }
        requirements:
            _method:  GET|POST

.. tip::

    You maybe wondering why the route would allow the HEAD method when only
    GET was specified. This is because HEAD is a GET request
    but only the HTTP Headers are returned.

Now when you submit the form it should function as expected, although expected
doesn't actually do much yet. The page will just redirect you back to the contact
form.

Validators
----------

The Symfony2 validator allows us to perform the task of data validation. Validation
is a common task when dealing with data from forms. Validation also needs to be
performed on data before it is submitted to a database. The Symfony2 validator
allows us to separate our validation logic away from the components that may use it,
such as the Form component or the Database component. This approach means we have one
set of validation rules for an object.

Let's begin by updating the ``Enquiry`` entity located at
``src/Blogger/BlogBundle/Entity/Enquiry.php`` to specify some Validators.
Ensure you add the 5 new ``use`` statements at the top of the file.

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

To define the validators we must implement the static method
``loadValidatorMetadata``. This provides us with an object of
``ClassMetadata``. We can use this object to set property constraints
on our entity members. The first statement applies the ``NotBlank``
constraint to the ``name`` member. The ``NotBlank`` validator is as simple
as it sounds, it will only return ``true`` if the value it is validating is
not empty. Next we setup validation for the ``email`` member. The Symfony2
Validator service provides a validator for
`emails <http://symfony.com/doc/current/reference/constraints/Email.html>`_
that will even check MX records to ensure the domain is valid. On the ``subject``
member we want to set a ``NotBlank`` and a ``MaxLength`` constraint.
You can apply as many validators to a member as you wish.

A full list of
`validator constrains <http://symfony.com/doc/current/reference/constraints.html>`_
is available in the Symfony2 reference documents. It is also possible to
`create custom validators <http://symfony.com/doc/current/cookbook/validation/custom_constraint.html>`_.

Now when you submit the contact form, your submitted data will be passed through the
validation constraints. Try typing in an invalid email address. You
should see an error message informing you that the email address is invalid. Each
validator provides a default message that can be overridden if required.
To change the message output by the email validator you would do the following.

.. code-block:: php

    $metadata->addPropertyConstraint('email', new Email(array(
        'message' => 'symblog does not like invalid emails. Give me a real one!'
    )));

.. tip::

    If you are using a browser that supports HTML5 (it is likely you are)
    you will be prompted with HTML5 messages enforcing certain constraints.
    This is client side validation and Symfony2 will set suitable HTML5 constraints
    based on your ``Entity`` metadata. You can see this on the email element. The
    output HTML is

    .. code-block:: html

        <input type="email" value="" required="required" name="contact[email]" id="contact_email">

    It has used one of the new HTML5 Input type fields, email and has set the required
    attribute. Client side validation is great in that it doesn't require a round
    trip to the server to validate the form. However, client side validation
    should not be used alone. You should always validate submitted data server
    side as it's quite easy for a user to by-pass the client side validation.

Sending the email
-----------------

While our contact form will allow users to submit enquiries, nothing actually happens
with them yet. Let's update the controller to send an email to the blog webmaster.
Symfony2 comes complete with the `Swift Mailer <http://swiftmailer.org/>`_
library for sending emails. Swift Mailer is a very powerful library,
we will only scratch the surface of what this library can perform.

Configure Swift Mailer settings
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
