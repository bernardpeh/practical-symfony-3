# Chapter 20: The Front View

Going to "http://songbird.app:8000/" has nothing at the moment because we have so far been focusing on the the admin area and not touched the frontend. In this chapter, we will create an automatic route based on the slug and display the frontend view when the slug matches. Any route that matches "/" and "/home" will be using the index template while the rest of the pages will be using the view template.

We will create a simple home and subpages using bootstrap and use [smartmenus javascript library](http://www.smartmenus.org/) to create the top menu which will render the the submenus as well.

Lastly, we'll add a language toggle so that the page can render different languages easily. The menu and page content will be rendered based on the toggled language. To get the menu to display different languages, we will create a custom twig function (an extension called MenuLocaleTitle).

## Define User Stories

**20. Frontend**

|**Story Id**|**As a**|**I**|**So that I**|
|20.1|test3 user|want to browse the frontend|I can get the information I want.|

**Story ID 20.1: As test3 user, I want to browse the frontend, so that I can get the information I want.**

|**Scenario Id**|**Given**|**When**|**Then**|
|20.11|Home page is working|I go to the / or /home|I can see the jumbotron class and the text "SongBird CMS Demo"|
|20.12|Menus are working|I mouseover the about menu|I should see 2 menus under the about menu|
|20.13|Subpages are working|I click on contact memu|I should see the text "This project is hosted in"|
|20.14|Login menu is working|I click on login memu|I should see 2 menu items only|

## Creating the Frontend

Let create a new frontend controller

```
# src/AppBundle/Controller/FrontendController.php

<?php

namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\Request;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Method;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

class FrontendController extends Controller
{
    /**
     * @Route("/{slug}", name="app_frontend_index", requirements = {"slug" = "^((|home)$)"})
     * @Method("GET")
     * @Template()
     * @param Request $request
     *
     * @return array
     */
    public function indexAction(Request $request)
    {
        $page = $this->getDoctrine()->getRepository('AppBundle:Page')->findOneBySlug($request->get('_route_params')['slug']);
        $pagemeta = $this->getDoctrine()->getRepository('AppBundle:PageMeta')->findPageMetaByLocale($page, $request->getLocale());
        $rootMenuItems = $this->getDoctrine()->getRepository('AppBundle:Page')->findParent();

        return array(
            'pagemeta' => $pagemeta,
            'tree' => $rootMenuItems,
        );
    }


    /**
     * @Route("/{slug}", name="app_frontend_view")
     * @Template()
     * @Method("GET")
     */
    public function pageAction(Request $request)
    {

        $page = $this->getDoctrine()->getRepository('AppBundle:Page')->findOneBySlug($request->get('_route_params')['slug']);
        $pagemeta = $this->getDoctrine()->getRepository('AppBundle:PageMeta')->findPageMetaByLocale($page, $request->getLocale());
        $rootMenuItems = $this->getDoctrine()->getRepository('AppBundle:Page')->findParent();

        return array(
            'pagemeta' => $pagemeta,
            'tree' => $rootMenuItems,
        );
    }
}
```

With the new @Template annotation, the action just need to return an array rather than a response. With the new routes added, we will move the frontend routes to the last priority, so routes like /login will be executed first.

```
# app/config/routing.yml
...
fos_user_security:
  resource: "@FOSUserBundle/Resources/config/routing/security.xml"

fos_user_resetting:
  resource: "@FOSUserBundle/Resources/config/routing/resetting.xml"
  prefix: /resetting

frontend:
  resource: "@AppBundle/Controller/FrontendController.php"
  type:     annotation
```

Let us update the frontend base view.

```
# src/AppBundle/Resources/Views/base.html.twig

<!DOCTYPE HTML>
<html lang="en-US">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}{% endblock %}</title>
    {% block stylesheets %}
        <link href="{{ asset('minified/css/styles.css') }}" rel="stylesheet" />
    {% endblock %}
</head>
<body>

{% set urlPrefix = (app.environment == 'dev') ? '/app_dev.php' : '' %}

{% block body %}

    <div class="container">
        {% block topnav %}
            <ul id="top_menu" class="sm sm-clean">
                <li id="page_logo">
                    <a href="{{ urlPrefix }}" alt="songbird">
                        <img src="{{ asset('bundles/app/images/logo.png') }}" class="center-block img-responsive" alt="Songbird" />
                    </a>
                </li>
                {% if tree is defined %}
                    {% include "AppBundle:Frontend:tree.html.twig" with { 'tree':tree } %}
                {% endif %}
                <li id="frontend_lang_toggle">
                    <select id="lang" name="lang">
                        {% for lang in supported_lang %}
                            <option value="{{ lang }}">{{ lang }}</option>
                        {% endfor %}
                    </select>
                </li>
                <li id="login_link">
                    {% if is_granted("IS_AUTHENTICATED_REMEMBERED") %}
                        <a href="{{ path('fos_user_security_logout') }}">
                            {{ 'layout.logout'|trans({}, 'FOSUserBundle') }}
                        </a>
                    {% else %}
                        <a href="{{ path('fos_user_security_login') }}">
                            {{ 'layout.login'|trans({}, 'FOSUserBundle') }}
                        </a>
                    {% endif %}
                </li>
            </ul>

        {% endblock %}

        <div class="clearfix vspace"></div>

        {% block content %}{% endblock %}

        {% block footer %}
            <hr />
            <footer>
                <p class="text-center">© Songbird {{ "now" | date("Y")}}</p>
            </footer>
        {% endblock %}
    </div>

{% endblock %}

{% block script %}
    <script src="{{ asset('minified/js/javascript.js') }}"></script>
    <script>
        $(function() {
            $('#top_menu').smartmenus();
            // select the box based on locale
            $('#lang').val('{{ app.request.getLocale() }}');
            // redirect user if user change locale
            $('#lang').change(function() {
                window.location='{{ urlPrefix }}'+$(this).val()+'/locale';
            });
        });
    </script>
{% endblock %}

</body>
</html>
```

We will now create a homepage view.

```
# app/Resources/AppBundle/views/Frontend/index.html.twig

{% extends "base.html.twig" %}

{% block title %}
	{{ pagemeta.getPageTitle() }}
{% endblock %}

{% block content %}
{% if pagemeta is not null %}
	<div class="jumbotron">
		<h1 class="text-center">{{ pagemeta.getShortDescription() | raw }}</h1>
	</div>

	<div class="row-fluid">
		<div class="pull-left col-xs-3 col-md-3">
			{%  if pagemeta.featuredImage is not null %}
				<img class="featured_image" src="{{ vich_uploader_asset(pagemeta, 'featuredImageFile') }}" alt="{{ pagemeta.getShortDescription() | striptags }}"/>
			{% endif %}
		</div>
		<div class="pull-right col-xs-9 col-md-9">
			{{ pagemeta.getContent() | raw }}
		</div>
	</div>
	<div class="clearfix"></div>

{% endif %}
{% endblock %}


```

and pages view

```
# app/Resources/AppBundle/views/Frontend/page.html.twig

{% extends "base.html.twig" %}

{% block title %}
	{{ pagemeta.getPageTitle() }}
{% endblock %}

{% block content %}

	{% if pagemeta is not null %}
	<h2>{{ pagemeta.getShortDescription() | raw }}</h2>

	{%  if pagemeta.featuredImage is not null %}
		<img class="featured_image" src="{{ vich_uploader_asset(pagemeta, 'featuredImageFile') }}" alt="{{ pagemeta.getShortDescription() | striptags }}"/>
	{% endif %}

	{{ pagemeta.getContent() | raw }}

	{% endif %}

{% endblock %}


```

and lastly, recursive view for the menu

```
# app/Resources/AppBundle/views/Frontend/tree.html.twig

{% for v in tree %}

    <li>
        <a href="{{ urlPrefix }}/{{ v.getSlug() }}">{{ getMenuLocaleTitle(v.getSlug()) }}</a>
        {% set children = v.getChildren()|length %}
        {% if children > 0 %}
            <ul>
                {% include "AppBundle:Frontend:tree.html.twig" with { 'tree':v.getChildren() } %}
            </ul>
        {% endif %}
    </li>
{% endfor %}

```
Note the new getMenuLocaleTitle function in the twig. We will create a custom function usable by twig - Twig Extension.

```
# src/AppBundle/Twig/Extension/MenuLocaleTitle.php

namespace AppBundle\Twig\Extension;

/**
 * Twig Extension to get Menu title based on locale
 */
class MenuLocaleTitle extends \Twig_Extension
{
    /**
     * @var EntityManager
     */
    private $em;

	/**
	 * @var $request
	 */
	private $request;

	/**
	 * MenuLocaleTitle constructor.
	 *
	 * @param $em
	 * @param $request
	 */
    public function __construct($em, $request)
    {
        $this->em = $em;
        $this->request = $request->getCurrentRequest();
    }

	/**
	 * @return string
	 */
    public function getName()
    {
        return 'menu_locale_title_extension';
    }

	/**
	 * @return array
	 */
    public function getFunctions()
    {
        return array(
            new \Twig_SimpleFunction('getMenuLocaleTitle', array($this, 'getMenuLocaleTitle'))
        );
    }

	/**
	 * @param string $slug
	 *
	 * @return mixed
	 */
    public function getMenuLocaleTitle($slug = 'home')
    {
	    $locale = ($this->request) ? $this->request->getLocale() : 'en';
    	$page = $this->em->getRepository('AppBundle:Page')->findOneBySlug($slug);
	    $pagemeta = $this->em->getRepository('AppBundle:PageMeta')->findPageMetaByLocale($page, $locale);

    	return $pagemeta->getMenuTitle();
    }
}
```

we now need to make this class available as a service.

```
# src/AppBundle/Resources/config/services.yml
...
  menu_locale_title.twig_extension:
    class: AppBundle\Twig\Extension\MenuLocaleTitle
    arguments:
      - "@doctrine.orm.entity_manager"
      - "@request_stack"
    tags:
      - { name: twig.extension }
...
```

Since we have added a new top navbar, let us remove the SongBird logo from the login and password reset pages. Update the following pages as you see fit:

```
/songbird/symfony/app/Resources/FOSUserBundle/views/Resetting/checkEmail.html.twig
/songbird/symfony/app/Resources/FOSUserBundle/views/Resetting/request.html.twig
/songbird/symfony/app/Resources/FOSUserBundle/views/Resetting/reset.html.twig
/songbird/symfony/app/Resources/FOSUserBundle/views/Security/login.html.twig
```

Let us update bower.json to pull in smartmenus js.

```
-> bower install smartmenus --S
```

then make gulp to pull the libraries in

```
# gulpfile.js
...
// Minify JS
gulp.task('js', function () {
    return gulp.src(['bower_components/jquery/dist/jquery.js',
        'bower_components/bootstrap/dist/js/bootstrap.js',
        'bower_components/smartmenus/dist/jquery.smartmenus.js'])
        .pipe(concat('javascript.js'))
        .pipe(uglify())
        .pipe(sourcemaps.write('./'))
        .pipe(gulp.dest('web/minified/js'));
});

// Minify CSS
gulp.task('css', function () {
    return gulp.src([
        'bower_components/bootstrap/dist/css/bootstrap.css',
        'bower_components/smartmenus/dist/css/sm-core-css.css',
        'bower_components/smartmenus/dist/css/sm-clean/sm-clean.css',
        'src/AppBundle/Resources/public/less/*.less',
        'src/AppBundle/Resources/public/sass/*.scss',
        'src/AppBundle/Resources/public/css/*.css'])
        .pipe(gulpif(/[.]less/, less()))
        .pipe(gulpif(/[.]scss/, sass()))
        .pipe(concat('styles.css'))
        .pipe(uglifycss())
        .pipe(sourcemaps.write('./'))
        .pipe(gulp.dest('web/minified/css'));
});
...
```

Let us update the datafixtures as well.

```
# src/AppBundle/DataFixtures/ORM/LoadPageData.php

...
        $homemetaEN = new PageMeta();
        $homemetaEN->setPage($homepage);
        $homemetaEN->setMenuTitle('Home');
        $homemetaEN->setPageTitle('SongBird CMS Demo');
        $homemetaEN->setShortDescription('SongBird CMS Demo');
        $homemetaEN->setContent('<p>SongBird is a simple CMS built with popular bundles like FOSUserBundle and EasyAdminBundle.
            The CMS is meant to showcase Rapid Application Development with Symfony.</p>');
        copy(__DIR__.'/images/home_en.png', __DIR__.'/../../../../web/uploads/featured_images/home_en.png');
        $homemetaEN->setFeaturedImage('home_en.png');
        $manager->persist($homemetaEN);

        $homemetaFR = new PageMeta();
        $homemetaFR->setPage($homepage);
        $homemetaFR->setMenuTitle('Accueil');
        $homemetaFR->setPageTitle('SongBird CMS Démo');
        $homemetaFR->setShortDescription('SongBird CMS Démo');
        $homemetaFR->setLocale('fr');
        $homemetaFR->setContent('<p>SongBird est un simple CMS construit avec des faisceaux populaires comme FOSUserBundle et EasyAdminBundle.
            Le CMS est destinée à mettre en valeur Rapid Application Development avec Symfony .</p>');
        copy(__DIR__.'/images/home_fr.png', __DIR__.'/../../../../web/uploads/featured_images/home_fr.png');
        $homemetaFR->setFeaturedImage('home_fr.png');
        $manager->persist($homemetaFR);
```

I've added new images to the homepage. The new images are in the src/AppBundle/DataFixtures/ORM/images folder. Feel free to get the images from there.

Lastly, let us update the stylesheets. We might as well update them in scss

```
# src/AppBundle/Resources/public/sass/styles.scss

// variables
$black: #000;
$white: #fff;
$radius: 6px;
$spacing: 20px;
$font_big: 16px;
$featured_image_width: 200px;

body {
    color: $black;
    background: $white !important;
    padding-top: $spacing;
}
.vspace {
    height: $spacing;
}
.skin-black {
    .logo {
        background-color: $black;
    }
    .left-side {
        background-color: $black;
    }
}
.sidebar {
    ul {
        padding-top: $spacing;
    }
    li {
        padding-top: $spacing;
    }
}
.admin_top_left {
    padding-top: $spacing 0 0 $spacing;
}
#top_menu {
    padding: $spacing 0 $spacing;

    #page_logo {
        padding: 0 $spacing;
        margin: -$spacing/2 0;
    }

    #login_link {
        float:right;
    }

    #frontend_lang_toggle {
        float: right;
        padding: $spacing/2 $spacing;
    }
}
.sm-clean {
    border-radius: $radius;

}
// admin area
#page_logo img {
    display: inline;
    max-height: 100%;
    max-width: 50%;
}
#page_menu li {
    line-height: $spacing;
    padding-top: $spacing;
}
.navbar-brand img {
    margin-top: -8px;
}

.form-signin {
    max-width: 330px;
    padding: 15px;
    margin: 0 auto;
    .form-signin-heading {
        margin-bottom: $spacing;
    }
    .checkbox {
        margin-bottom: $spacing;
        font-weight: normal;
    }
    .form-control {
        position: relative;
        height: auto;
        box-sizing: border-box;
        padding: $spacing;
        font-size: $font_big;
        &:focus {
            z-index: 2;
        }
    }
    input[type="email"] {
        border-bottom-right-radius: 0;
        border-bottom-left-radius: 0;
    }
    input[type="password"] {
        margin-bottom: $spacing;
        border-top-left-radius: 0;
        border-top-right-radius: 0;
    }
}
.form-control {
    margin-top: $spacing;
    margin-bottom: $spacing;
}

// frontend
.featured_image {
    margin: auto;
    display: block;
    width: $featured_image_width;
}

```

We no longer need our old .css files

```
-> git rm src/AppBundle/Resources/public/css/*
```

Now run gulp and refresh the homepage and everything should renders.

```
-> gulp
```

Remember to create the featured_images dir and reset the db if not done

```
-> mkdir -p web/uploads/featured_images
-> ./scripts/resetapp
```

Go to homepage and this should be the end result.

![](images/cms_final.png)


## Update BDD (Optional)

Let us create the cest file:

```
-> vendor/bin/codecept generate:cest -c src/AppBundle acceptance As_Test3_User/IWantToViewTheFrontend
```

Write your test and make sure everything passes.

## Summary

In this chapter, we have created the frontend controllers and views. We used smartmenus to render the menus and converted our css to sass. Finally, we wrote BDD tests to make sure our frontend renders correctly. The CMS is now complete.

## Exercises

* Try extending the NestablePageBundle so that you can have multiple menus, say a top and bottom menu?

* One of the argument against using a language toggle is that it is bad for SEO. Language toggle can be good for usability. Can you think of a way to overcome the SEO issue?

## References

* [Controllers best practice](http://symfony.com/doc/current/best_practices/controllers.html)
* [Smart Menus](http://www.smartmenus.org/)
* [Twig Extension](http://symfony.com/doc/current/cookbook/templating/twig_extension.html)
