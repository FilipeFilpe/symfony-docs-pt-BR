.. index::
   single: Templating

Criando e usando Templates
============================

Como voc� sabe o :doc:`controller </book/controller>` � respons�vel por
controlar cada requisi��o que venha de uma aplica��o Symfony2. Na realidade,
o controller delega muito do trabalho pesado para outros lugares ent�o aquele
c�digo pode ser testado e reusado. Quando um controller precisa gerar HTML,
CSS e qualquer outro conte�do, ele entrega o trabalho para a engine de template.
Nesse cap�tulo, voc� ir� aprendder como escrever templates poderosas que podem ser
usada para retornar conte�do para o usu�rio, preencher corpo de e-mail, e mais. Voc� 
ir� aprender atalhos, maneiras espertas de extender templates e como reusar c�digo
de template.

.. index::
   single: Templating;O que � um template?

Templates
---------

Um template � simplesmente um arquivo de texto que pode gerar qualquer formato baseado em texto 
(HTML, XML, CSV, LaTeX ...). O tipo mais familiar de template � um 
template em *PHP* - um arquivo de texto analisado pelo PHP que cont�m uma mistura de texto e c�digo PHP::

    <!DOCTYPE html>
    <html>
        <head>
            <title>Welcome to Symfony!</title>
        </head>
        <body>
            <h1><?php echo $page_title ?></h1>

            <ul id="navigation">
                <?php foreach ($navigation as $item): ?>
                    <li>
                        <a href="<?php echo $item->getHref() ?>">
                            <?php echo $item->getCaption() ?>
                        </a>
                    </li>
                <?php endforeach; ?>
            </ul>
        </body>
    </html>

.. index:: Twig; Introdu��o

Mas Symfony2 empacota at� mesmo uma linguagem muito poderosa de template chamada `Twig`_.
Twig permit a voc� escrever templates consisos e leg�veis que s�o mais amig�veis
para web designers e, e de uma certa forma, mais poderosos que templates de PHP:

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
        <head>
            <title>Welcome to Symfony!</title>
        </head>
        <body>
            <h1>{{ page_title }}</h1>

            <ul id="navigation">
                {% for item in navigation %}
                    <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
                {% endfor %}
            </ul>
        </body>
    </html>

Twig degine dois tipos de sintaxe especiais:

* ``{{ ... }}``: "Diga algo": exibe uma vari�vel ou o resultado de uma
  express�o para o template;
  
* ``{% ... %}``: "Fa�a algo": uma **tag** que controla a l�gica do
  template; ela � usada para executar certas senten�as como for-loops por exemplo.

.. note::

   H� uma terceira sintaxe usada para criar coment�rios ``{# this is a comment #}``.
   Essa sintaxe pode ser usada atrav�s de m�ltiplas linhas, parecidas com a sintaxe
   equivalente em PHP ``/* comment */``.

Twig tamb�m cont�m **filtros**, que modificam conte�do antes de serem interpretados.
O seguinte filtro transforma a vari�vel ``title`` toda em letra mai�scula antes de interpret�-la:

.. code-block:: jinja

    {{ title | upper }}

Twig vem com uma longa lista de `tags`_ e `filtros`_ que est�o dispon�veis
por padr�o. Voc� pode at� mesmo `adicionar suas pr�prias extens�es`_ para o Twig quando precisar.

.. tip::

    Registrar uma extens�o Twig � t�o f�cil quanto criar um novo servi�o e atribuir tag
    nele com ``twig.extension`` :ref:`tag<reference-dic-tags-twig-extension>`.

Como voc� ver� atrav�s da documenta��o, Twig tamb�m suporta fun��es
e nova fun��es podem ser facilmente adicionadas. Por exemplo. a seguinte fun��o usa
uma tag padr�o ``for``  e a fun��o ``cycle`` para ent�o imprimir dez tags div, alternando
entre classes ``odd`` e ``even``:

.. code-block:: html+jinja

    {% for i in 0..10 %}
      <div class="{{ cycle(['odd', 'even'], i) }}">
        <!-- some HTML here -->
      </div>
    {% endfor %}

Durante este cap�tulo, exemplos de template ser�o mostrados tanto em Twig como PHP.

.. sidebar:: Por que Twig?
    
    Templates Twig s�o feitas para serem simples e n�o ir�o processar tags PHP. Isto
    � pelo design: o sistema de template do Twig � feito para expressar apresenta��o,
    n�o l�gica de programa. Quanto mais voc� usa Twig, mais voc� ir� apreciar e beneficiar
    desta distin��o. E claro, voc� ser� amado por web designers de todos os lugares.
    
    Twig pode tamb�m fazer coisas que PHP n�o pode, como por exemplo heran�a verdadeira de template
    (Templates do Twig compilam classes PHP que herdam uma da outra),
    controle de espa�o em branco, caixa de areia, e a inclus�o de fun��es personalizadas
    e filtros que somente afetam templates. Twig cont�m pequenos recursos
    que fazem escrita de templates mais f�cil e mais concisa. Considere o seguinte 
    exemplo, que combina um loop com uma senten�a l�gia ``if``:
    
    .. code-block:: html+jinja
    
        <ul>
            {% for user in users %}
                <li>{{ user.username }}</li>
            {% else %}
                <li>No users found</li>
            {% endfor %}
        </ul>

.. index::
   pair: Twig; Cache

Cache do Template Twig
~~~~~~~~~~~~~~~~~~~~~

Twig � r�pido. Cada template Twig � compilado para uma classe nativa PHP
que � processada na execu��o. As classes compiladas s�o localizadas no
diret�rio ``app/cache/{environment}/twig`` (onde ``{environment}`` � o
ambiente, como ``dev`` ou ``prod``), e em alguns casos pode ser �til durante 
a depura��o. Veja :ref:`environments-summary` para mais informa��es de
ambientes.

Quando o modo ``debug`` � abilitado (comum no ambiente ``dev``), um template
Twig ser� automaticamente recompilado quando mudan�as s�o feitas nele. Isso
signitica que durante o desenvolvimento voc� pode alegremente fazer mudan�as para um template Twig
e inst�ntaneamente ver as mudan�as sem precisar se preocupar sobre limpar qualquer
cache.

Quando o modo ``debug`` � desabilitado (comum no ambiente ``prod``), entretanto,
voc� deve limpar o cache do diret�rio Twig para que ent�o os templates Twig
se regenerem. Lembre de fazer isso quando distribuir sua aplica��o.

.. index::
   single: Templating; Inheritance

Heran�a e Layouts de Template
--------------------------------

Mais frequentemente que n�o, templates compartilham elementos comuns em um projeto, como o
header, footer, sidebar ou outros. Em Symfony2,  n�s gostamos de pensar sobre esse
problema de forma diferente: um template pode ser decorado por outro. Isso funciona
exatemente da mesma forma como classes PHP: heran�a de template permite voc� construir
um "layout" de template base que contenha todos os elementos comuns de seu site
definidos como **blocos** (pense em "classe PHP com m�todos base"). Um template filho
pode extender o layout base e sobrepor os blocos (pense "subclasse PHP 
que sobreponha certos m�todos de sua classe pai").

Primeiro, construa um arquivo de layout de base:

.. configuration-block::

    .. code-block:: html+jinja

        {# app/Resources/views/base.html.twig #}
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title>{% block title %}Test Application{% endblock %}</title>
            </head>
            <body>
                <div id="sidebar">
                    {% block sidebar %}
                    <ul>
                        <li><a href="/">Home</a></li>
                        <li><a href="/blog">Blog</a></li>
                    </ul>
                    {% endblock %}
                </div>

                <div id="content">
                    {% block body %}{% endblock %}
                </div>
            </body>
        </html>

    .. code-block:: php

        <!-- app/Resources/views/base.html.php -->
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title><?php $view['slots']->output('title', 'Test Application') ?></title>
            </head>
            <body>
                <div id="sidebar">
                    <?php if ($view['slots']->has('sidebar'): ?>
                        <?php $view['slots']->output('sidebar') ?>
                    <?php else: ?>
                        <ul>
                            <li><a href="/">Home</a></li>
                            <li><a href="/blog">Blog</a></li>
                        </ul>
                    <?php endif; ?>
                </div>

                <div id="content">
                    <?php $view['slots']->output('body') ?>
                </div>
            </body>
        </html>

.. note::
    
    Apesar da discuss�o sobre heran�a de template ser em termos do Twig,
    a filosofia � a mesma entre templates Twig e PHP.

Este template define o esqueleto do documento base HTML de um p�gina simples
de duas colunas. Neste exemplo, tr�s �reas ``{% block %}`` s�o definidas (``title``,
``sidebar`` e ``body``). Cada bloco pode ser sobreposto por um template filho
ou largado com sua implementa��o padr�o. Esse template poderia tamb�m ser processado
diretamente. Neste caso os blocos ``title``, ``sidebar`` e ``body`` blocks deveriam
simplesmente reter os valores padr�o neste template.

Um template filho poderia ser como este:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/Blog/index.html.twig #}
        {% extends '::base.html.twig' %}

        {% block title %}My cool blog posts{% endblock %}

        {% block body %}
            {% for entry in blog_entries %}
                <h2>{{ entry.title }}</h2>
                <p>{{ entry.body }}</p>
            {% endfor %}
        {% endblock %}

    .. code-block:: php

        <!-- src/Acme/BlogBundle/Resources/views/Blog/index.html.php -->
        <?php $view->extend('::base.html.php') ?>

        <?php $view['slots']->set('title', 'My cool blog posts') ?>

        <?php $view['slots']->start('body') ?>
            <?php foreach ($blog_entries as $entry): ?>
                <h2><?php echo $entry->getTitle() ?></h2>
                <p><?php echo $entry->getBody() ?></p>
            <?php endforeach; ?>
        <?php $view['slots']->stop() ?>

.. note::
    
   O template pai � idenficado por uma sintaxe especial de string
   (``::base.html.twig``) que indica que o template reside no diret�rio
   ``app/Resources/views`` do projeto. Essa conven��o de nomeamento �
   explicada inteiramente em :ref:`template-naming-locations`.

A chave para heran�a template � a tag  ``{% extends %}``. Ela avisa
a engine de template para primeiro avaliar o template base, que configura
o layout e define v�rios blocos. O template filho � ent�o processado,
ao ponto que os blocos  ``title`` e ``body`` do template pai sejam substitu�dos
por aqueles do filho. Dependendo do valor de ``blog_entries``, a
sa�da poderia parecer com isso::

    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
            <title>My cool blog posts</title>
        </head>
        <body>
            <div id="sidebar">
                <ul>
                    <li><a href="/">Home</a></li>
                    <li><a href="/blog">Blog</a></li>
                </ul>
            </div>

            <div id="content">
                <h2>My first post</h2>
                <p>The body of the first post.</p>

                <h2>Another post</h2>
                <p>The body of the second post.</p>
            </div>
        </body>
    </html>

Perceba que como o template filho n�o definiu um bloco ``sidebar``, o
valor do template pai � usado no lugar. Conte�do dentro de uma tag  ``{% block %}``
em um template pai � sempre usado por padr�o.

Voc� pode usar muitos n�veis de heran�a quanto quiser. Na pr�xima sess�o,
um modelo comum de heran�a de tr�s n�veis ser� explicado assim como os templates
s�o organizados dentro de um projeto Symfony2.

Quando trabalhar com heran�a de template, aqui est�o algumas dicas para guardar na cabe�a:

* Se voc� usa ``{% extends %}`` em um template, ele deve ser a primeira tag 
  naquele template.
  
* Quanto mais tags ``{% block %}`` voc� tiver no template base, melhor.
  Lembre, templates filhos n�o precisam definir todos os blocos pai, ent�o
  criar tantos blocos em seus templates base quanto voc� quiser e dar a cada um
  padr�o sensato. Quanto mais blocos seus templates base tiverem, mais flex�vel
  seu layout ser�.

* Se voc� achar voc� mesmo duplicando conte�do em um certo n�mero de templates, isto provavelmente
  significa que voc� deveria mover aquele conte�do para um ``{% block %}`` no template pai.
  Em alguns casos, uma solu��o melhor pode ser mover o conte�do para um novo template
  e ``incluir`` ele (veja :ref:`including-templates`).

* Se voc� precisa obter o conte�do de um bloco do template pai, voc�
  pode usar a fun��o ``{{ parent() }}``. Isso � �til se voc� quiser adicionar
  ao conte�do de um bloco pai ao inv�s de sobrepor ele::
  
    .. code-block:: html+jinja

        {% block sidebar %}
            <h3>Table of Contents</h3>
            ...
            {{ parent() }}
        {% endblock %}

.. index::
   single: Templating; Conven��es de Nomea��o
   single: Templating; Localiza��o de Arquivos

.. _template-naming-locations:

Nomea��o de Template e Localiza��es
-----------------------------

Por padr�o, templates podem residir em duas localiza��es diferentes:

* ``app/Resources/views/``: O diret�rio de aplica��o de ``views`` pode conter
  templates bases para toda a aplica��o(ex: os layout de sua aplica��o) assim como
  os tempates que sobrep�em templates de pacote (veja :ref:`overriding-bundle-templates`);
  
  application-wide base templates (i.e. your application's layouts) as well as
  templates that override bundle templates (see
  :ref:`overriding-bundle-templates`);
  

* ``path/to/bundle/Resources/views/``: Cada pacote possui as templates dela no diret�rio 
  ``Resources/views`` (e sub-diret�rios). A maioria dos templates ir� residir dentro de
  um pacote.
  
Symfony2 usa a sintaxe de string **bundle**:**controller**:**template** para
templates. Isso permite v�rios tipos diferente de template, cada um residindo
em uma localiza��o especifica:

* ``AcmeBlogBundle:Blog:index.html.twig``: Esta sintaxe � usada para especificar um
  template para uma p�gina espec�fica. As tr�s partes do string, cada uma separada
  por dois pontos, (``:``), signitca o seguinte:

    * ``AcmeBlogBundle``: (*bundle*) o template reside entro de
      ``AcmeBlogBundle`` (e.g. ``src/Acme/BlogBundle``);

    * ``Blog``: (*controller*) indica que o template reside dentro do
       sub-diret�rio ``Blog`` de ``Resources/views``;

    * ``index.html.twig``: (*template*) o verdadeiro nome do arquivo �
      ``index.html.twig``.

  Assumindo que o ``AcmeBlogBundle`` reside em ``src/Acme/BlogBundle``, o
  atalho final para o layout seria ``src/Acme/BlogBundle/Resources/views/Blog/index.html.twig``.

* ``AcmeBlogBundle::layout.html.twig``: Essa sintaxe refere ao template base que
  � espec�fica para ``AcmeBlogBundle``. Since the middle, "controller",
  portion is missing (e.g. ``Blog``), the template lives at
  ``Resources/views/layout.html.twig`` inside ``AcmeBlogBundle``.

* ``::base.html.twig``: Esta sintaxe refere a uma template base para toda a aplica��o ou
  layout. Perceba que a string come�a com dois sinais de dois pontos (``::``), significando
  que ambas as partes *bundle*  *controller* est�o faltando. Isto significa
  que o template n�o � localizado em qualquer pacote, mas sim na raiz do 
  diret�rio ``app/Resources/views/``.

Na se��o :ref:`overriding-bundle-templates`, voc� ir� descobrir como cada
template reside dentro do ``AcmeBlogBundle``, por exemplo, pode ser sobreposto
ao colocar um template de mesmo nome no diret�rio 
``app/Resources/AcmeBlogBundle/views/``. Isso d� o poder de sobrepor templates de qualquer pacote pago.

.. tip::
    
    Esperan�osamente, a sintaxe de nomea��o de template parece familiar - � a mesma conven��o
    para nomea��o usada para referir para :ref:`controller-string-syntax`.

Sufixo de Template 
~~~~~~~~~~~~~~~

O formato **bundle**:**controller**:**template** de cada template especifica
*onde* o arquivo de template est� localizado. Cada nome de template tamb�m tem duas express�es
que especificam o *formato* e *engine* para aquela template.

* **AcmeBlogBundle:Blog:index.html.twig** - formato HTML, engine Twig

* **AcmeBlogBundle:Blog:index.html.php** - formato HTML, engine PHP

* **AcmeBlogBundle:Blog:index.css.twig** - formato CSS, engine Twig

Por padr�o, qualquer template Symfony2 ou pode ser escrito em Twig ou em PHP, and
the last part of the extension (e.g. ``.twig`` or ``.php``) specifies which
of these two *engines* should be used. The first part of the extension,
(e.g. ``.html``, ``.css``, etc) is the final format that the template will
generate. Unlike the engine, which determines how Symfony2 parses the template,
this is simply an organizational tactic used in case the same resource needs
to be rendered as HTML (``index.html.twig``), XML (``index.xml.twig``),
or any other format. For more information, read the :ref:`template-formats`
section.

.. note::
    
   As "engines" dispon�veis podem ser configurados e at� mesmo ter novas engines adicionadas.
   Veja :ref:`Configura��o de Template<template-configuration>` para mais detalhes.

.. index::
   single: Templating; Tags and Helpers
   single: Templating; Helpers

Tags and Helpers
----------------

You already understand the basics of templates, how they're named and how
to use template inheritance. The hardest parts are already behind you. In
this section, you'll learn about a large group of tools available to help
perform the most common template tasks such as including other templates,
linking to pages and including images.

Symfony2 comes bundled with several specialized Twig tags and functions that
ease the work of the template designer. In PHP, the templating system provides
an extensible *helper* system that provides useful features in a template
context.

We've already seen a few built-in Twig tags (``{% block %}`` & ``{% extends %}``)
as well as an example of a PHP helper (``$view['slots']``). Let's learn a
few more.

.. index::
   single: Templating; Including other templates

.. _including-templates:

Including other Templates
~~~~~~~~~~~~~~~~~~~~~~~~~

You'll often want to include the same template or code fragment on several
different pages. For example, in an application with "news articles", the
template code displaying an article might be used on the article detail page,
on a page displaying the most popular articles, or in a list of the latest
articles.

When you need to reuse a chunk of PHP code, you typically move the code to
a new PHP class or function. The same is true for templates. By moving the
reused template code into its own template, it can be included from any other
template. First, create the template that you'll need to reuse.

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/views/Article/articleDetails.html.twig #}
        <h2>{{ article.title }}</h2>
        <h3 class="byline">by {{ article.authorName }}</h3>

        <p>
          {{ article.body }}
        </p>

    .. code-block:: php

        <!-- src/Acme/ArticleBundle/Resources/views/Article/articleDetails.html.php -->
        <h2><?php echo $article->getTitle() ?></h2>
        <h3 class="byline">by <?php echo $article->getAuthorName() ?></h3>

        <p>
          <?php echo $article->getBody() ?>
        </p>

Including this template from any other template is simple:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/Article/list.html.twig #}
        {% extends 'AcmeArticleBundle::layout.html.twig' %}

        {% block body %}
            <h1>Recent Articles<h1>

            {% for article in articles %}
                {% include 'AcmeArticleBundle:Article:articleDetails.html.twig' with {'article': article} %}
            {% endfor %}
        {% endblock %}

    .. code-block:: php

        <!-- src/Acme/ArticleBundle/Resources/Article/list.html.php -->
        <?php $view->extend('AcmeArticleBundle::layout.html.php') ?>

        <?php $view['slots']->start('body') ?>
            <h1>Recent Articles</h1>

            <?php foreach ($articles as $article): ?>
                <?php echo $view->render('AcmeArticleBundle:Article:articleDetails.html.php', array('article' => $article)) ?>
            <?php endforeach; ?>
        <?php $view['slots']->stop() ?>

The template is included using the ``{% include %}`` tag. Notice that the
template name follows the same typical convention. The ``articleDetails.html.twig``
template uses an ``article`` variable. This is passed in by the ``list.html.twig``
template using the ``with`` command.

.. tip::

    The ``{'article': article}`` syntax is the standard Twig syntax for hash
    maps (i.e. an array with named keys). If we needed to pass in multiple
    elements, it would look like this: ``{'foo': foo, 'bar': bar}``.

.. index::
   single: Templating; Embedding action

.. _templating-embedding-controller:

Embedding Controllers
~~~~~~~~~~~~~~~~~~~~~

In some cases, you need to do more than include a simple template. Suppose
you have a sidebar in your layout that contains the three most recent articles.
Retrieving the three articles may include querying the database or performing
other heavy logic that can't be done from within a template.

The solution is to simply embed the result of an entire controller from your
template. First, create a controller that renders a certain number of recent
articles:

.. code-block:: php

    // src/Acme/ArticleBundle/Controller/ArticleController.php

    class ArticleController extends Controller
    {
        public function recentArticlesAction($max = 3)
        {
            // make a database call or other logic to get the "$max" most recent articles
            $articles = ...;

            return $this->render('AcmeArticleBundle:Article:recentList.html.twig', array('articles' => $articles));
        }
    }

The ``recentList`` template is perfectly straightforward:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/views/Article/recentList.html.twig #}
        {% for article in articles %}
          <a href="/article/{{ article.slug }}">
              {{ article.title }}
          </a>
        {% endfor %}

    .. code-block:: php

        <!-- src/Acme/ArticleBundle/Resources/views/Article/recentList.html.php -->
        <?php foreach ($articles as $article): ?>
            <a href="/article/<?php echo $article->getSlug() ?>">
                <?php echo $article->getTitle() ?>
            </a>
        <?php endforeach; ?>

.. note::

    Notice that we've cheated and hardcoded the article URL in this example
    (e.g. ``/article/*slug*``). This is a bad practice. In the next section,
    you'll learn how to do this correctly.

To include the controller, you'll need to refer to it using the standard string
syntax for controllers (i.e. **bundle**:**controller**:**action**):

.. configuration-block::

    .. code-block:: html+jinja

        {# app/Resources/views/base.html.twig #}
        ...

        <div id="sidebar">
            {% render "AcmeArticleBundle:Article:recentArticles" with {'max': 3} %}
        </div>

    .. code-block:: php

        <!-- app/Resources/views/base.html.php -->
        ...

        <div id="sidebar">
            <?php echo $view['actions']->render('AcmeArticleBundle:Article:recentArticles', array('max' => 3)) ?>
        </div>

Whenever you find that you need a variable or a piece of information that
you don't have access to in a template, consider rendering a controller.
Controllers are fast to execute and promote good code organization and reuse.

.. index::
   single: Templating; Linking to pages

Linking to Pages
~~~~~~~~~~~~~~~~

Creating links to other pages in your application is one of the most common
jobs for a template. Instead of hardcoding URLs in templates, use the ``path``
Twig function (or the ``router`` helper in PHP) to generate URLs based on
the routing configuration. Later, if you want to modify the URL of a particular
page, all you'll need to do is change the routing configuration; the templates
will automatically generate the new URL.

First, link to the "_welcome" page, which is accessible via the following routing
configuration:

.. configuration-block::

    .. code-block:: yaml

        _welcome:
            pattern:  /
            defaults: { _controller: AcmeDemoBundle:Welcome:index }

    .. code-block:: xml

        <route id="_welcome" pattern="/">
            <default key="_controller">AcmeDemoBundle:Welcome:index</default>
        </route>

    .. code-block:: php

        $collection = new RouteCollection();
        $collection->add('_welcome', new Route('/', array(
            '_controller' => 'AcmeDemoBundle:Welcome:index',
        )));

        return $collection;

To link to the page, just use the ``path`` Twig function and refer to the route:

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ path('_welcome') }}">Home</a>

    .. code-block:: php

        <a href="<?php echo $view['router']->generate('_welcome') ?>">Home</a>

As expected, this will generate the URL ``/``. Let's see how this works with
a more complicated route:

.. configuration-block::

    .. code-block:: yaml

        article_show:
            pattern:  /article/{slug}
            defaults: { _controller: AcmeArticleBundle:Article:show }

    .. code-block:: xml

        <route id="article_show" pattern="/article/{slug}">
            <default key="_controller">AcmeArticleBundle:Article:show</default>
        </route>

    .. code-block:: php

        $collection = new RouteCollection();
        $collection->add('article_show', new Route('/article/{slug}', array(
            '_controller' => 'AcmeArticleBundle:Article:show',
        )));

        return $collection;

In this case, you need to specify both the route name (``article_show``) and
a value for the ``{slug}`` parameter. Using this route, let's revisit the
``recentList`` template from the previous section and link to the articles
correctly:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/views/Article/recentList.html.twig #}
        {% for article in articles %}
          <a href="{{ path('article_show', { 'slug': article.slug }) }}">
              {{ article.title }}
          </a>
        {% endfor %}

    .. code-block:: php

        <!-- src/Acme/ArticleBundle/Resources/views/Article/recentList.html.php -->
        <?php foreach ($articles in $article): ?>
            <a href="<?php echo $view['router']->generate('article_show', array('slug' => $article->getSlug()) ?>">
                <?php echo $article->getTitle() ?>
            </a>
        <?php endforeach; ?>

.. tip::

    You can also generate an absolute URL by using the ``url`` Twig function:

    .. code-block:: html+jinja

        <a href="{{ url('_welcome') }}">Home</a>

    The same can be done in PHP templates by passing a third argument to
    the ``generate()`` method:

    .. code-block:: php

        <a href="<?php echo $view['router']->generate('_welcome', array(), true) ?>">Home</a>

.. index::
   single: Templating; Linking to assets

Linking to Assets
~~~~~~~~~~~~~~~~~

Templates also commonly refer to images, Javascript, stylesheets and other
assets. Of course you could hard-code the path to these assets (e.g. ``/images/logo.png``),
but Symfony2 provides a more dynamic option via the ``assets`` Twig function:

.. configuration-block::

    .. code-block:: html+jinja

        <img src="{{ asset('images/logo.png') }}" alt="Symfony!" />

        <link href="{{ asset('css/blog.css') }}" rel="stylesheet" type="text/css" />

    .. code-block:: php

        <img src="<?php echo $view['assets']->getUrl('images/logo.png') ?>" alt="Symfony!" />

        <link href="<?php echo $view['assets']->getUrl('css/blog.css') ?>" rel="stylesheet" type="text/css" />

The ``asset`` function's main purpose is to make your application more portable.
If your application lives at the root of your host (e.g. http://example.com),
then the rendered paths should be ``/images/logo.png``. But if your application
lives in a subdirectory (e.g. http://example.com/my_app), each asset path
should render with the subdirectory (e.g. ``/my_app/images/logo.png``). The
``asset`` function takes care of this by determining how your application is
being used and generating the correct paths accordingly.

Additionally, if you use the ``asset`` function, Symfony can automatically
append a query string to your asset, in order to guarantee that updated static
assets won't be cached when deployed. For example, ``/images/logo.png`` might
look like ``/images/logo.png?v2``. For more information, see the :ref:`ref-framework-assets-version`
configuration option.

.. index::
   single: Templating; Including stylesheets and Javascripts
   single: Stylesheets; Including stylesheets
   single: Javascripts; Including Javascripts

Including Stylesheets and Javascripts in Twig
---------------------------------------------

No site would be complete without including Javascript files and stylesheets.
In Symfony, the inclusion of these assets is handled elegantly by taking
advantage of Symfony's template inheritance.

.. tip::

    This section will teach you the philosophy behind including stylesheet
    and Javascript assets in Symfony. Symfony also packages another library,
    called Assetic, which follows this philosophy but allows you to do much
    more interesting things with those assets. For more information on 
    using Assetic see :doc:`/cookbook/assetic/asset_management`.


Start by adding two blocks to your base template that will hold your assets:
one called ``stylesheets`` inside the ``head`` tag and another called ``javascripts``
just above the closing ``body`` tag. These blocks will contain all of the
stylesheets and Javascripts that you'll need throughout your site:

.. code-block:: html+jinja

    {# 'app/Resources/views/base.html.twig' #}
    <html>
        <head>
            {# ... #}

            {% block stylesheets %}
                <link href="{{ asset('/css/main.css') }}" type="text/css" rel="stylesheet" />
            {% endblock %}
        </head>
        <body>
            {# ... #}

            {% block javascripts %}
                <script src="{{ asset('/js/main.js') }}" type="text/javascript"></script>
            {% endblock %}
        </body>
    </html>

That's easy enough! But what if you need to include an extra stylesheet or
Javascript from a child template? For example, suppose you have a contact
page and you need to include a ``contact.css`` stylesheet *just* on that
page. From inside that contact page's template, do the following:

.. code-block:: html+jinja

    {# src/Acme/DemoBundle/Resources/views/Contact/contact.html.twig #}
    {# extends '::base.html.twig' #}

    {% block stylesheets %}
        {{ parent() }}
        
        <link href="{{ asset('/css/contact.css') }}" type="text/css" rel="stylesheet" />
    {% endblock %}
    
    {# ... #}

In the child template, you simply override the ``stylesheets`` block and 
put your new stylesheet tag inside of that block. Of course, since you want
to add to the parent block's content (and not actually *replace* it), you
should use the ``parent()`` Twig function to include everything from the ``stylesheets``
block of the base template.

You can also include assets located in your bundles' ``Resources/public`` folder.
You will need to run the ``php app/console assets:install target [--symlink]``
command, which moves (or symlinks) files into the correct location. (target
is by default "web").

.. code-block:: html+jinja

   <link href="{{ asset('bundles/acmedemo/css/contact.css') }}" type="text/css" rel="stylesheet" />

The end result is a page that includes both the ``main.css`` and ``contact.css``
stylesheets.

.. index::
   single: Templating; The templating service

Configuring and using the ``templating`` Service
------------------------------------------------

The heart of the template system in Symfony2 is the templating ``Engine``.
This special object is responsible for rendering templates and returning
their content. When you render a template in a controller, for example,
you're actually using the templating engine service. For example:

.. code-block:: php

    return $this->render('AcmeArticleBundle:Article:index.html.twig');

is equivalent to

.. code-block:: php

    $engine = $this->container->get('templating');
    $content = $engine->render('AcmeArticleBundle:Article:index.html.twig');

    return $response = new Response($content);

.. _template-configuration:

The templating engine (or "service") is preconfigured to work automatically
inside Symfony2. It can, of course, be configured further in the application
configuration file:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            # ...
            templating: { engines: ['twig'] }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:templating>
            <framework:engine id="twig" />
        </framework:templating>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            // ...
            'templating'      => array(
                'engines' => array('twig'),
            ),
        ));

Several configuration options are available and are covered in the
:doc:`Configuration Appendix</reference/configuration/framework>`.

.. note::

   The ``twig`` engine is mandatory to use the webprofiler (as well as many
   third-party bundles).

.. index::
    single; Template; Overriding templates

.. _overriding-bundle-templates:

Overriding Bundle Templates
---------------------------

The Symfony2 community prides itself on creating and maintaining high quality
bundles (see `Symfony2Bundles.org`_) for a large number of different features.
Once you use a third-party bundle, you'll likely need to override and customize
one or more of its templates.

Suppose you've included the imaginary open-source ``AcmeBlogBundle`` in your
project (e.g. in the ``src/Acme/BlogBundle`` directory). And while you're
really happy with everything, you want to override the blog "list" page to
customize the markup specifically for your application. By digging into the
``Blog`` controller of the ``AcmeBlogBundle``, you find the following::

    public function indexAction()
    {
        $blogs = // some logic to retrieve the blogs

        $this->render('AcmeBlogBundle:Blog:index.html.twig', array('blogs' => $blogs));
    }

When the ``AcmeBlogBundle:Blog:index.html.twig`` is rendered, Symfony2 actually
looks in two different locations for the template:

#. ``app/Resources/AcmeBlogBundle/views/Blog/index.html.twig``
#. ``src/Acme/BlogBundle/Resources/views/Blog/index.html.twig``

To override the bundle template, just copy the ``index.html.twig`` template
from the bundle to ``app/Resources/AcmeBlogBundle/views/Blog/index.html.twig``
(the ``app/Resources/AcmeBlogBundle`` directory won't exist, so you'll need
to create it). You're now free to customize the template.

This logic also applies to base bundle templates. Suppose also that each
template in ``AcmeBlogBundle`` inherits from a base template called
``AcmeBlogBundle::layout.html.twig``. Just as before, Symfony2 will look in
the following two places for the template:

#. ``app/Resources/AcmeBlogBundle/views/layout.html.twig``
#. ``src/Acme/BlogBundle/Resources/views/layout.html.twig``

Once again, to override the template, just copy it from the bundle to
``app/Resources/AcmeBlogBundle/views/layout.html.twig``. You're now free to
customize this copy as you see fit.

If you take a step back, you'll see that Symfony2 always starts by looking in
the ``app/Resources/{BUNDLE_NAME}/views/`` directory for a template. If the
template doesn't exist there, it continues by checking inside the
``Resources/views`` directory of the bundle itself. This means that all bundle
templates can be overridden by placing them in the correct ``app/Resources``
subdirectory.

.. _templating-overriding-core-templates:

.. index::
    single; Template; Overriding exception templates

Overriding Core Templates
~~~~~~~~~~~~~~~~~~~~~~~~~

Since the Symfony2 framework itself is just a bundle, core templates can be
overridden in the same way. For example, the core ``TwigBundle`` contains
a number of different "exception" and "error" templates that can be overridden
by copying each from the ``Resources/views/Exception`` directory of the
``TwigBundle`` to, you guessed it, the
``app/Resources/TwigBundle/views/Exception`` directory.

.. index::
   single: Templating; Three-level inheritance pattern

Three-level Inheritance
-----------------------

One common way to use inheritance is to use a three-level approach. This
method works perfectly with the three different types of templates we've just
covered:

* Create a ``app/Resources/views/base.html.twig`` file that contains the main
  layout for your application (like in the previous example). Internally, this
  template is called ``::base.html.twig``;

* Create a template for each "section" of your site. For example, an ``AcmeBlogBundle``,
  would have a template called ``AcmeBlogBundle::layout.html.twig`` that contains
  only blog section-specific elements;

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/layout.html.twig #}
        {% extends '::base.html.twig' %}

        {% block body %}
            <h1>Blog Application</h1>

            {% block content %}{% endblock %}
        {% endblock %}

* Create individual templates for each page and make each extend the appropriate
  section template. For example, the "index" page would be called something
  close to ``AcmeBlogBundle:Blog:index.html.twig`` and list the actual blog posts.

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/Blog/index.html.twig #}
        {% extends 'AcmeBlogBundle::layout.html.twig' %}

        {% block content %}
            {% for entry in blog_entries %}
                <h2>{{ entry.title }}</h2>
                <p>{{ entry.body }}</p>
            {% endfor %}
        {% endblock %}

Notice that this template extends the section template -(``AcmeBlogBundle::layout.html.twig``)
which in-turn extends the base application layout (``::base.html.twig``).
This is the common three-level inheritance model.

When building your application, you may choose to follow this method or simply
make each page template extend the base application template directly
(e.g. ``{% extends '::base.html.twig' %}``). The three-template model is
a best-practice method used by vendor bundles so that the base template for
a bundle can be easily overridden to properly extend your application's base
layout.

.. index::
   single: Templating; Output escaping

Output Escaping
---------------

When generating HTML from a template, there is always a risk that a template
variable may output unintended HTML or dangerous client-side code. The result
is that dynamic content could break the HTML of the resulting page or allow
a malicious user to perform a `Cross Site Scripting`_ (XSS) attack. Consider
this classic example:

.. configuration-block::

    .. code-block:: jinja

        Hello {{ name }}

    .. code-block:: php

        Hello <?php echo $name ?>

Imagine that the user enters the following code as his/her name::

    <script>alert('hello!')</script>

Without any output escaping, the resulting template will cause a JavaScript
alert box to pop up::

    Hello <script>alert('hello!')</script>

And while this seems harmless, if a user can get this far, that same user
should also be able to write JavaScript that performs malicious actions
inside the secure area of an unknowing, legitimate user.

The answer to the problem is output escaping. With output escaping on, the
same template will render harmlessly, and literally print the ``script``
tag to the screen::

    Hello &lt;script&gt;alert(&#39;helloe&#39;)&lt;/script&gt;

The Twig and PHP templating systems approach the problem in different ways.
If you're using Twig, output escaping is on by default and you're protected.
In PHP, output escaping is not automatic, meaning you'll need to manually
escape where necessary.

Output Escaping in Twig
~~~~~~~~~~~~~~~~~~~~~~~

If you're using Twig templates, then output escaping is on by default. This
means that you're protected out-of-the-box from the unintentional consequences
of user-submitted code. By default, the output escaping assumes that content
is being escaped for HTML output.

In some cases, you'll need to disable output escaping when you're rendering
a variable that is trusted and contains markup that should not be escaped.
Suppose that administrative users are able to write articles that contain
HTML code. By default, Twig will escape the article body. To render it normally,
add the ``raw`` filter: ``{{ article.body | raw }}``.

You can also disable output escaping inside a ``{% block %}`` area or
for an entire template. For more information, see `Output Escaping`_ in
the Twig documentation.

Output Escaping in PHP
~~~~~~~~~~~~~~~~~~~~~~

Output escaping is not automatic when using PHP templates. This means that
unless you explicitly choose to escape a variable, you're not protected. To
use output escaping, use the special ``escape()`` view method::

    Hello <?php echo $view->escape($name) ?>

By default, the ``escape()`` method assumes that the variable is being rendered
within an HTML context (and thus the variable is escaped to be safe for HTML).
The second argument lets you change the context. For example, to output something
in a JavaScript string, use the ``js`` context:

.. code-block:: js

    var myMsg = 'Hello <?php echo $view->escape($name, 'js') ?>';

.. index::
   single: Templating; Formats

.. _template-formats:

Template Formats
----------------

Templates are a generic way to render content in *any* format. And while in
most cases you'll use templates to render HTML content, a template can just
as easily generate JavaScript, CSS, XML or any other format you can dream of.

For example, the same "resource" is often rendered in several different formats.
To render an article index page in XML, simply include the format in the
template name:

* *XML template name*: ``AcmeArticleBundle:Article:index.xml.twig``
* *XML template filename*: ``index.xml.twig``

In reality, this is nothing more than a naming convention and the template
isn't actually rendered differently based on its format.

In many cases, you may want to allow a single controller to render multiple
different formats based on the "request format". For that reason, a common
pattern is to do the following:

.. code-block:: php

    public function indexAction()
    {
        $format = $this->getRequest()->getRequestFormat();
    
        return $this->render('AcmeBlogBundle:Blog:index.'.$format.'.twig');
    }

The ``getRequestFormat`` on the ``Request`` object defaults to ``html``,
but can return any other format based on the format requested by the user.
The request format is most often managed by the routing, where a route can
be configured so that ``/contact`` sets the request format to ``html`` while
``/contact.xml`` sets the format to ``xml``. For more information, see the
:ref:`Advanced Example in the Routing chapter <advanced-routing-example>`.

To create links that include the format parameter, include a ``_format``
key in the parameter hash:

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ path('article_show', {'id': 123, '_format': 'pdf'}) }}">
            PDF Version
        </a>

    .. code-block:: html+php

        <a href="<?php echo $view['router']->generate('article_show', array('id' => 123, '_format' => 'pdf')) ?>">
            PDF Version
        </a>

Final Thoughts
--------------

The templating engine in Symfony is a powerful tool that can be used each time
you need to generate presentational content in HTML, XML or any other format.
And though templates are a common way to generate content in a controller,
their use is not mandatory. The ``Response`` object returned by a controller
can be created with our without the use of a template:

.. code-block:: php

    // creates a Response object whose content is the rendered template
    $response = $this->render('AcmeArticleBundle:Article:index.html.twig');

    // creates a Response object whose content is simple text
    $response = new Response('response content');

Symfony's templating engine is very flexible and two different template
renderers are available by default: the traditional *PHP* templates and the
sleek and powerful *Twig* templates. Both support a template hierarchy and
come packaged with a rich set of helper functions capable of performing
the most common tasks.

Overall, the topic of templating should be thought of as a powerful tool
that's at your disposal. In some cases, you may not need to render a template,
and in Symfony2, that's absolutely fine.

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/templating/PHP`
* :doc:`/cookbook/controller/error_pages`

.. _`Twig`: http://twig.sensiolabs.org
.. _`Symfony2Bundles.org`: http://symfony2bundles.org
.. _`Cross Site Scripting`: http://en.wikipedia.org/wiki/Cross-site_scripting
.. _`Output Escaping`: http://twig.sensiolabs.org
.. _`tags`: http://twig.sensiolabs.org/doc/tags/index.html
.. _`filters`: http://twig.sensiolabs.org/doc/templates.html#filters
.. _`add your own extensions`: http://twig.sensiolabs.org/doc/advanced.html
