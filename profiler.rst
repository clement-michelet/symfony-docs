Profiler
========

The profiler is a powerful **development tool** that gives detailed information
about the execution of any request.

.. caution::

    **Never** enable the profiler in production environments
    as it will lead to major security vulnerabilities in your project.

Installation
------------

In applications using :ref:`Symfony Flex <symfony-flex>`, run this command to
install the ``profiler`` :ref:`Symfony pack <symfony-packs>` before using it:

.. code-block:: terminal

    $ composer require --dev symfony/profiler-pack

Now, browse any page of your application in the development environment to let
the profiler collect information. Then, click on any element of the debug
toolbar injected at the bottom of your pages to open the web interface of the
Symfony Profiler, which will look like this:

.. image:: /_images/profiler/web-interface.png
   :align: center
   :class: with-browser

.. note::

    The debug toolbar is only injected into HTML responses. For other kinds of
    contents (e.g. JSON responses in API requests) the profiler URL is available
    in the ``X-Debug-Token-Link`` HTTP response header. Browse the ``/_profiler``
    URL to see all profiles.

.. versionadded:: 6.3

    Profile garbage collection was introduced in Symfony 6.3.

.. note::

    To limit the storage used by profiles on disk, they are probabilistically
    removed after 2 days.

Accessing Profiling Data Programmatically
-----------------------------------------

Most of the time, the profiler information is accessed and analyzed using its
web-based interface. However, you can also retrieve profiling information
programmatically thanks to the methods provided by the ``profiler`` service.

When the response object is available, use the
:method:`Symfony\\Component\\HttpKernel\\Profiler\\Profiler::loadProfileFromResponse`
method to access to its associated profile::

    // ... $profiler is the 'profiler' service
    $profile = $profiler->loadProfileFromResponse($response);

When the profiler stores data about a request, it also associates a token with it;
this token is available in the ``X-Debug-Token`` HTTP header of the response.
Using this token, you can access the profile of any past response thanks to the
:method:`Symfony\\Component\\HttpKernel\\Profiler\\Profiler::loadProfile` method::

    $token = $response->headers->get('X-Debug-Token');
    $profile = $profiler->loadProfile($token);

.. tip::

    When the profiler is enabled but not the web debug toolbar, inspect the page
    with your browser's developer tools to get the value of the ``X-Debug-Token``
    HTTP header.

The ``profiler`` service also provides the
:method:`Symfony\\Component\\HttpKernel\\Profiler\\Profiler::find` method to
look for tokens based on some criteria::

    // gets the latest 10 tokens
    $tokens = $profiler->find('', '', 10, '', '', '');

    // gets the latest 10 tokens for all URL containing /admin/
    $tokens = $profiler->find('', '/admin/', 10, '', '', '');

    // gets the latest 10 tokens for local POST requests
    $tokens = $profiler->find('127.0.0.1', '', 10, 'POST', '', '');

    // gets the latest 10 tokens for requests that happened between 2 and 4 days ago
    $tokens = $profiler->find('', '', 10, '', '4 days ago', '2 days ago');

Data Collectors
---------------

The profiler gets its information using some services called "data collectors".
Symfony comes with several collectors that get information about the request,
the logger, the routing, the cache, etc.

Run this command to get the list of collectors actually enabled in your app:

.. code-block:: terminal

    $ php bin/console debug:container --tag=data_collector

You can also :ref:`create your own data collector <profiler-data-collector>` to
store any data generated by your app and display it in the debug toolbar and the
profiler web interface.

.. _profiler-timing-execution:

Timing the Execution of the Application
---------------------------------------

If you want to measure the time some tasks take in your application, there's no
need to create a custom data collector. Instead, use the built-in utilities to
:ref:`profile Symfony applications <profiling-applications>`.

.. tip::

    Consider using a professional profiler such as `Blackfire`_ to measure and
    analyze the execution of your application in detail.

.. _enabling-the-profiler-programmatically:

Enabling the Profiler Programmatically or Conditionally
-------------------------------------------------------

Symfony Profiler can be enabled and disabled programmatically. You can use the ``enable()``
and ``disable()`` methods of the :class:`Symfony\\Component\\HttpKernel\\Profiler\\Profiler`
class in your controllers to manage the profiler programmatically::

    use Symfony\Component\HttpKernel\Profiler\Profiler;
    // ...

    class DefaultController
    {
        // ...

        public function someMethod(?Profiler $profiler)
        {
            // $profiler won't be set if your environment doesn't have the profiler (like prod, by default)
            if (null !== $profiler) {
                // if it exists, disable the profiler for this particular controller action
                $profiler->disable();
            }

            // ...
        }
    }

In order for the profiler to be injected into your controller you need to
create an alias pointing to the existing ``profiler`` service:

.. configuration-block::

    .. code-block:: yaml

        # config/services_dev.yaml
        services:
            Symfony\Component\HttpKernel\Profiler\Profiler: '@profiler'

    .. code-block:: xml

        <!-- config/services_dev.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <services>
                <service id="Symfony\Component\HttpKernel\Profiler\Profiler" alias="profiler"/>
            </services>
        </container>

    .. code-block:: php

        // config/services_dev.php
        use Symfony\Component\HttpKernel\Profiler\Profiler;

        $container->setAlias(Profiler::class, 'profiler');

.. _enabling-the-profiler-conditionally:

Enabling the Profiler Conditionally
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Instead of enabling the profiler programmatically as explained in the previous
section, you can also enable it when a certain condition is met (e.g. a certain
parameter is included in the URL):

.. code-block:: yaml

    # config/packages/dev/web_profiler.yaml
        framework:
            profiler:
                collect: false
                collect_parameter: 'profile'

This configuration disables the profiler by default (``collect: false``) to
improve the application performance; but enables it for requests that include a
query parameter called ``profile`` (you can freely choose this query parameter name).

In addition to the query parameter, this feature also works when submitting a
form field with that name (useful to enable the profiler in ``POST`` requests)
or when including it as a request attribute.

Updating the Web Debug Toolbar After AJAX Requests
--------------------------------------------------

`Single-page applications`_ (SPA) are web applications that interact with the
user by dynamically rewriting the current page rather than loading entire new
pages from a server.

By default, the debug toolbar displays the information of the initial page load
and doesn't refresh after each AJAX request. However, you can set the
``Symfony-Debug-Toolbar-Replace`` header to a value of ``1`` in the response to
the AJAX request to force the refresh of the toolbar::

    $response->headers->set('Symfony-Debug-Toolbar-Replace', 1);

Ideally this header should only be set during development and not for
production. To do that, create an :doc:`event subscriber </event_dispatcher>`
and listen to the :ref:`kernel.response <component-http-kernel-kernel-response>`
event::


    use Symfony\Component\EventDispatcher\EventSubscriberInterface;
    use Symfony\Component\HttpKernel\Event\ResponseEvent;
    use Symfony\Component\HttpKernel\KernelInterface;

    // ...

    class MySubscriber implements EventSubscriberInterface
    {
        public function __construct(
            private KernelInterface $kernel,
        ) {
        }

        // ...

        public function onKernelResponse(ResponseEvent $event)
        {
            if (!$this->kernel->isDebug()) {
                return;
            }

            $request = $event->getRequest();
            if (!$request->isXmlHttpRequest()) {
                return;
            }

            $response = $event->getResponse();
            $response->headers->set('Symfony-Debug-Toolbar-Replace', 1);
        }
    }

.. _profiler-data-collector:

Creating a Data Collector
-------------------------

The Symfony Profiler obtains its profiling and debug information using some
special classes called data collectors. Symfony comes bundled with a few of
them, but you can also create your own.

A data collector is a PHP class that implements the
:class:`Symfony\\Component\\HttpKernel\\DataCollector\\DataCollectorInterface`.
For convenience, your data collectors can also extend from the
:class:`Symfony\\Bundle\\FrameworkBundle\\DataCollector\\AbstractDataCollector`
class, which implements the interface and provides some utilities and the
``$this->data`` property to store the collected information.

The following example shows a custom collector that stores information about the
request::

    // src/DataCollector/RequestCollector.php
    namespace App\DataCollector;

    use Symfony\Bundle\FrameworkBundle\DataCollector\AbstractDataCollector;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;

    class RequestCollector extends AbstractDataCollector
    {
        public function collect(Request $request, Response $response, \Throwable $exception = null)
        {
            $this->data = [
                'method' => $request->getMethod(),
                'acceptable_content_types' => $request->getAcceptableContentTypes(),
            ];
        }
    }

These are the method that you can define in the data collector class:

:method:`Symfony\\Component\\HttpKernel\\DataCollector\\DataCollectorInterface::collect` method:
    Stores the collected data in local properties (``$this->data`` if you extend
    from ``AbstractDataCollector``). If you need some services to collect the
    data, inject those services in the data collector constructor.

    .. caution::

        The ``collect()`` method is only called once. It is not used to "gather"
        data but is there to "pick up" the data that has been stored by your
        service.

    .. caution::

        As the profiler serializes data collector instances, you should not
        store objects that cannot be serialized (like PDO objects) or you need
        to provide your own ``serialize()`` method.

:method:`Symfony\\Component\\HttpKernel\\DataCollector\\DataCollectorInterface::reset` method:
    It's called between requests to reset the state of the profiler. By default
    it only empties the ``$this->data`` contents, but you can override this method
    to do additional cleaning.

:method:`Symfony\\Component\\HttpKernel\\DataCollector\\DataCollectorInterface::getName` method:
    Returns the collector identifier, which must be unique in the application.
    By default it returns the FQCN of the data collector class, but you can
    override this method to return a custom name (e.g. ``app.request_collector``).
    This value is used later to access the collector information (see
    :doc:`/testing/profiling`) so you may prefer using short strings instead of FQCN strings.

The ``collect()`` method is called during the :ref:`kernel.response <component-http-kernel-kernel-response>`
event. If you need to collect data that is only available later, implement
:class:`Symfony\\Component\\HttpKernel\\DataCollector\\LateDataCollectorInterface`
and define the ``lateCollect()`` method, which is invoked right before the profiler
data serialization (during :ref:`kernel.terminate <component-http-kernel-kernel-terminate>` event).

.. note::

    If you're using the :ref:`default services.yaml configuration <service-container-services-load-example>`
    with ``autoconfigure``, then Symfony will start using your data collector after the
    next page refresh. Otherwise, :ref:`enable the data collector by hand <data_collector_tag>`.

Adding Web Profiler Templates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The information collected by your data collector can be displayed both in the
web debug toolbar and in the web profiler. To do so, you need to create a Twig
template that includes some specific blocks.

First, add the ``getTemplate()`` method in your data collector class to return
the path of the Twig template to use. Then, add some *getters* to give the
template access to the collected information::

    // src/DataCollector/RequestCollector.php
    namespace App\DataCollector;

    use Symfony\Bundle\FrameworkBundle\DataCollector\AbstractDataCollector;

    class RequestCollector extends AbstractDataCollector
    {
        // ...

        public static function getTemplate(): ?string
        {
            return 'data_collector/template.html.twig';
        }

        public function getMethod()
        {
            return $this->data['method'];
        }

        public function getAcceptableContentTypes()
        {
            return $this->data['acceptable_content_types'];
        }
    }

In the simplest case, you want to display the information in the toolbar
without providing a profiler panel. This requires to define the ``toolbar``
block and set the value of two variables called ``icon`` and ``text``:

.. code-block:: html+twig

    {# templates/data_collector/template.html.twig #}
    {% extends '@WebProfiler/Profiler/layout.html.twig' %}

    {% block toolbar %}
        {% set icon %}
            {# this is the content displayed as a panel in the toolbar #}
            <svg xmlns="http://www.w3.org/2000/svg"> ... </svg>
            <span class="sf-toolbar-value">Request</span>
        {% endset %}

        {% set text %}
            {# this is the content displayed when hovering the mouse over
               the toolbar panel #}
            <div class="sf-toolbar-info-piece">
                <b>Method</b>
                <span>{{ collector.method }}</span>
            </div>

            <div class="sf-toolbar-info-piece">
                <b>Accepted content type</b>
                <span>{{ collector.acceptableContentTypes|join(', ') }}</span>
            </div>
        {% endset %}

        {# the 'link' value set to 'false' means that this panel doesn't
           show a section in the web profiler #}
        {{ include('@WebProfiler/Profiler/toolbar_item.html.twig', { link: false }) }}
    {% endblock %}

.. tip::

    Symfony Profiler icons are selected from `Tabler icons`_, a large and open
    source collection of SVG icons. It's recommended to also use those icons for
    your own profiler panels to get a consistent look.

.. tip::

    Built-in collector templates define all their images as embedded SVG files.
    This makes them work everywhere without having to mess with web assets links:

    .. code-block:: twig

        {% set icon %}
            {{ include('data_collector/icon.svg') }}
            {# ... #}
        {% endset %}

If the toolbar panel includes extended web profiler information, the Twig template
must also define additional blocks:

.. code-block:: html+twig

    {# templates/data_collector/template.html.twig #}
    {% extends '@WebProfiler/Profiler/layout.html.twig' %}

    {% block toolbar %}
        {% set icon %}
            {# ... #}
        {% endset %}

        {% set text %}
            <div class="sf-toolbar-info-piece">
                {# ... #}
            </div>
        {% endset %}

        {{ include('@WebProfiler/Profiler/toolbar_item.html.twig', { 'link': true }) }}
    {% endblock %}

    {% block head %}
        {# Optional. Here you can link to or define your own CSS and JS contents. #}
        {# Use {{ parent() }} to extend the default styles instead of overriding them. #}
    {% endblock %}

    {% block menu %}
        {# This left-hand menu appears when using the full-screen profiler. #}
        <span class="label">
            <span class="icon"><img src="..." alt=""/></span>
            <strong>Request</strong>
        </span>
    {% endblock %}

    {% block panel %}
        {# Optional, for showing the most details. #}
        <h2>Acceptable Content Types</h2>
        <table>
            <tr>
                <th>Content Type</th>
            </tr>

            {% for type in collector.acceptableContentTypes %}
            <tr>
                <td>{{ type }}</td>
            </tr>
            {% endfor %}
        </table>
    {% endblock %}

The ``menu`` and ``panel`` blocks are the only required blocks to define the
contents displayed in the web profiler panel associated with this data collector.
All blocks have access to the ``collector`` object.

.. note::

    The position of each panel in the toolbar is determined by the collector
    priority, which can only be defined when :ref:`configuring the data collector by hand <data_collector_tag>`.

.. note::

    If you're using the :ref:`default services.yaml configuration <service-container-services-load-example>`
    with ``autoconfigure``, then Symfony will start displaying your collector data
    in the toolbar after the next page refresh. Otherwise, :ref:`enable the data collector by hand <data_collector_tag>`.

.. _data_collector_tag:

Enabling Custom Data Collectors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you don't use Symfony's default configuration with
:ref:`autowire and autoconfigure <service-container-services-load-example>`
you'll need to configure the data collector explicitly:

.. configuration-block::

    .. code-block:: yaml

        # config/services.yaml
        services:
            App\DataCollector\RequestCollector:
                tags:
                    -
                        name: data_collector
                        # must match the value returned by the getName() method
                        id: 'App\DataCollector\RequestCollector'
                        # optional template (it has more priority than the value returned by getTemplate())
                        template: 'data_collector/template.html.twig'
                        # optional priority (positive or negative integer; default = 0)
                        # priority: 300

    .. code-block:: xml

        <!-- config/services.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <services>
                <service id="App\DataCollector\RequestCollector">
                    <!-- the 'template' attribute has more priority than the value returned by getTemplate() -->
                    <tag name="data_collector"
                        id="App\DataCollector\RequestCollector"
                        template="data_collector/template.html.twig"
                    />
                    <!-- optional 'priority' attribute (positive or negative integer; default = 0) -->
                    <!-- priority="300" -->
                </service>
            </services>
        </container>

    .. code-block:: php

        // config/services.php
        namespace Symfony\Component\DependencyInjection\Loader\Configurator;

        use App\DataCollector\RequestCollector;

        return function(ContainerConfigurator $container): void {
            $services = $container->services();

            $services->set(RequestCollector::class)
                ->tag('data_collector', [
                    'id' => RequestCollector::class,
                    // optional template (it has more priority than the value returned by getTemplate())
                    'template' => 'data_collector/template.html.twig',
                    // optional priority (positive or negative integer; default = 0)
                    // 'priority' => 300,
                ]);
        };

.. _`Single-page applications`: https://en.wikipedia.org/wiki/Single-page_application
.. _`Blackfire`: https://blackfire.io/docs/introduction?utm_source=symfony&utm_medium=symfonycom_docs&utm_campaign=profiler
.. _`Tabler icons`: https://github.com/tabler/tabler-icons
