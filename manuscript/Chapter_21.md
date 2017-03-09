# Chapter 21: Dependency Injection Revisited

Upon reflection of what we have covered in the last 20 chapters, I think there are lots of improvements that can be done. In particular, I feel that I wouldn't do justice to this book if I don't give an example of [Compiler Pass](http://symfony.com/doc/current/service_container/compiler_passes.html).

This is an advance chapter. If you skipped all the chapters and came to this chapter by chance, I recommend you to read up DI and DIC before continuing.

In this chapter, I like to introduce 2 improvements to the CMS.

a) Simplifying config.yml

b) Adding Simple User Access Control to EasyAdminBundle.

## Simplifying config.yml

Due to DI, the bundle Extension is called when the bundle is being initialised. The end result is a bunch of parameters and services that can be used and referenced throughout the application.

The app/config/config.yml is read by all bundle extensions so that relevant information relating to the bundle can be extracted. So far, there are many configuration parameters like fos, vich, doctrine ...etc. To make the installation easier, we could move all these extra configuration to elsewhere so that developers don't have to worry about them when installing the CMS and it also makes the file looks cleaner.

The trick that does that is to implement the [PrependExtensionInterface](http://symfony.com/doc/current/bundles/prepend_extension.html).

```
# src/AppBundle/DependencyInjection/AppExtension.php

namespace AppBundle\DependencyInjection;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\Extension\PrependExtensionInterface;
use Symfony\Component\HttpKernel\DependencyInjection\Extension;
use Symfony\Component\DependencyInjection\Loader\YamlFileLoader;

/**
 * This is the class that loads and manages your bundle configuration
 *
 * To learn more see {@link http://symfony.com/doc/current/cookbook/bundles/extension.html}
 */
class AppExtension extends Extension implements PrependExtensionInterface
{
    /**
     * {@inheritdoc}
     */
    public function load(array $configs, ContainerBuilder $container)
    {
        $configuration = new Configuration();
        $config = $this->processConfiguration($configuration, $configs);

        $loader = new YamlFileLoader($container, new FileLocator(__DIR__ . '/../Resources/config'));
        $loader->load('services.yml');
    }

    /**
     * http://symfony.com/doc/current/bundles/prepend_extension.html
     */
    public function prepend(ContainerBuilder $container)
    {
        // doctrine config
        $doctrine = [];
        $doctrine['orm']['resolve_target_entities']['Bpeh\NestablePageBundle\Model\PageBase'] = 'AppBundle\Entity\Page';
        $doctrine['orm']['resolve_target_entities']['Bpeh\NestablePageBundle\Model\PageMetaBase'] = 'AppBundle\Entity\PageMeta';
        $container->prependExtensionConfig('doctrine', $doctrine);

        // fos config
        $fosuser = [];
        $fosuser['db_driver'] = 'orm';
        $fosuser['firewall_name'] = 'main';
        $fosuser['user_class'] = 'AppBundle\Entity\User';
        $fosuser['from_email']['address'] = 'admin@songbird.app';
        $fosuser['from_email']['sender_name'] = 'Songbird';
        $container->prependExtensionConfig('fos_user', $fosuser);

        # Nestable page config
        $page = [];
        $page['page_entity'] = 'AppBundle\Entity\Page';
        $page['pagemeta_entity'] = 'AppBundle\Entity\PageMeta';
        $page['page_form_type'] = 'AppBundle\Form\PageType';
        $page['pagemeta_form_type'] = 'AppBundle\Form\PageMetaType';
        $container->prependExtensionConfig('bpeh_nestable_page', $page);

        # Vich config
        $vich = [];
        $vich['db_driver'] = 'orm';
        $vich['mappings']['profile_images']['uri_prefix'] = '%app.profile_image.path%';
        $vich['mappings']['profile_images']['upload_destination'] = '%kernel.root_dir%/../web/uploads/profiles';
        $vich['mappings']['profile_images']['namer'] = 'vich_uploader.namer_uniqid';
        $vich['mappings']['featured_image']['uri_prefix'] = '%app.featured_image.path%';
        $vich['mappings']['featured_image']['upload_destination'] = '%kernel.root_dir%/../web/uploads/featured_images';
        $vich['mappings']['featured_image']['namer'] = 'vich_uploader.namer_uniqid';
        $container->prependExtensionConfig('vich_uploader', $vich);

    }
}
```

my config.yml then becomes like this:

```
# app/config/config.yml

imports:
    - { resource: parameters.yml }
    - { resource: security.yml }
    - { resource: services.yml }
    - {resource: easyadmin/ }

parameters:
    locale: en
    supported_lang: [ 'en', 'fr']
    admin_path: admin
    app.profile_image.path: /uploads/profiles
    app.featured_image.path: /uploads/featured_images

framework:
    #esi:             ~
    translator:      { fallbacks: ["%locale%"] }
    secret:          "%secret%"
    router:
        resource: "%kernel.root_dir%/config/routing.yml"
        strict_requirements: ~
    form:            ~
    csrf_protection: ~
    validation:      { enable_annotations: true }
    #serializer:      { enable_annotations: true }
    templating:
        engines: ['twig']
    default_locale:  "%locale%"
    trusted_hosts:   ~
    trusted_proxies: ~
    session:
        # handler_id set to null will use default session handler from php.ini
        handler_id:  ~
    fragments:       ~
    http_method_override: true

# Twig Configuration
twig:
    debug:            "%kernel.debug%"
    strict_variables: "%kernel.debug%"
    globals:
        supported_lang: '%supported_lang%'

# Doctrine Configuration
doctrine:
    dbal:
        driver:   pdo_mysql
        host:     "%database_host%"
        port:     "%database_port%"
        dbname:   "%database_name%"
        user:     "%database_user%"
        password: "%database_password%"
        charset:  UTF8
    orm:
        auto_generate_proxy_classes: "%kernel.debug%"
        naming_strategy: doctrine.orm.naming_strategy.underscore
        auto_mapping: true

# Swiftmailer Configuration
swiftmailer:
    transport: "%mailer_transport%"
    host:      "%mailer_host%"
    username:  "%mailer_user%"
    password:  "%mailer_password%"
    spool:     { type: memory }
```

Noticed that I could have moved more parameters over to the prepend function if I want to simplify the installation further.

## Adding Simple Access Control to EasyAdminBundle

I still want to comment [javiereguiluz](https://github.com/javiereguiluz) for creating the wonderful [EasyAdminBundle](https://github.com/javiereguiluz/EasyAdminBundle). As of current, the bundle doesn't support user permissions out of the box. I believe there might be plans to include this feature in the future as it is a widely requested feature.

As an exercise, let's say that we want to customise the bundle so that we can control access to certain parts of the admin area based on the user's role and we want to do that simply by changing the easyadmin yaml files.

Let us allow all authenticated users to access the admin area rather than just ROLE_USER.

```
# app/config/security.yml
...
  access_control:
      - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
      - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
      - { path: ^/%admin_path%/, role: IS_AUTHENTICATED_FULLY }
```

The new design.yml should look like this:

```
# app/config/easyadmin/design.yml

easy_admin:
    design:
        brand_color: '#5493ca'
        assets:
            css:
                - /bundles/app/css/style.css
        menu:
          - { label: 'Dashboard', route: 'dashboard', default: true }
          - { entity: 'User', icon: 'user', role: ROLE_USER }
          - { entity: 'Page', icon: 'file', role: ROLE_ADMIN }
          - { entity: 'UserLog', icon: 'database', role: ROLE_ADMIN }
```

Noticed that we have added a new attribute called "role" to each menu item and the value (say "ROLE_ADMIN") means the mimimum permission level required to access that menu. In this case, everyone can see the dashboard, ROLE_USER and above can access the User link and only ROLE_ADMIN can see the User and UserLog link.

We are going to do something similar for all the entities yaml, starting from the page entity

```
# app/config/easyadmin/page.yml

easy_admin:
    entities:
        Page:
            class: AppBundle\Entity\Page
            label: admin.link.page_management
            role: ROLE_ADMIN
            # for new page
            new:
                fields:
                  - slug
                  - isPublished
                  - sequence
                  - parent
            edit:
                fields:
                  - slug
                  - isPublished
                  - sequence
                  - parent
                  - pageMetas
            show:
                fields:
                  - id
                  - slug
                  - isPublished
                  - sequence
                  - parent
                  - modified
                  - created
                  - pageMetas
            list:
                actions: ['show', 'edit', 'delete']
                fields:
                  - id
                  - slug
                  - isPublished
                  - sequence
                  - parent
                  - modified
        PageMeta:
            class: AppBundle\Entity\PageMeta
            role: ROLE_ADMIN
            form:
              fields:
                - page_title
                - menu_title
                - { property: 'locale', type: 'AppBundle\Form\LocaleType' }
                - { type: 'divider' }
                - { property: 'featuredImageFile', type: 'vich_image' }
                - { property: 'short_description', type: 'ckeditor' }
                - { property: 'content', type: 'ckeditor' }
                - page
```

Now, the user entity:

```
# app/config/easyadmin/user.yml

easy_admin:
    entities:
        User:
            class: AppBundle\Entity\User
            label: admin.link.user_management
            role: ROLE_USER
            # for new user
            new:
                role: ROLE_ADMIN
                fields:
                  - username
                  - firstname
                  - lastname
                  - { property: 'plainPassword', type: 'repeated', type_options: { type: 'Symfony\Component\Form\Extension\Core\Type\PasswordType', first_options: {label: 'Password'}, second_options: {label: 'Repeat Password'}, invalid_message: 'The password fields must match.'}}
                  - { property: 'email', type: 'email', type_options: { trim: true } }
                  - { property: 'imageFile', type: 'vich_image' }
                  - roles
                  - enabled
            edit:
                role: ROLE_ADMIN
                fields:
                - username
                - firstname
                - lastname
                - { property: 'plainPassword', type: 'repeated', type_options: { type: 'Symfony\Component\Form\Extension\Core\Type\PasswordType', required: false, first_options: {label: 'Password'}, second_options: {label: 'Repeat Password'}, invalid_message: 'The password fields must match.'}}
                - { property: 'email', type: 'email', type_options: { trim: true } }
                - { property: 'imageFile', type: 'vich_image' }
                - roles
                - enabled
            show:
                role: ROLE_ADMIN
                fields:
                - id
                - { property: 'image', type: 'image', base_path: '%app.profile_image.path%'}
                - username
                - firstname
                - lastname
                - email
                - roles
                - enabled
                - { property: 'last_login', type: 'datetime' }
                - modified
                - created
            list:
                role: ROLE_USER
                title: 'User Listing'
                actions: ['show']
                fields:
                  - id
                  - { property: 'image', type: 'image', base_path: '%app.profile_image.path%'}
                  - username
                  - email
                  - firstname
                  - lastname
                  - enabled
                  - roles
                  - { property: 'last_login', type: 'datetime' }
            delete:
                role: ROLE_ADMIN
```

and finally - userlog.yml. 

```
# app/config/easyadmin/userlog.yml

easy_admin:
    entities:
        UserLog:
            class: AppBundle\Entity\UserLog
            label: admin.link.user_log
            role: ROLE_ADMIN
            show:
                actions: ['list', '-edit', '-delete']
            list:
                actions: ['show', '-edit', '-delete']
```

When parameters and services are created by the extension but not yet compiled in optimised DIC, there is a chance to manipulate them. Compiler Pass exists for this purpose.

Let us tell our AppBundle to initiate its compiler pass when it is loaded by the kernel.

```
# src/AppBundle/AppBundle.php

<?php

namespace AppBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use AppBundle\DependencyInjection\Compiler\ConfigPass;

class AppBundle extends Bundle
{
	public function build(ContainerBuilder $container)
	{
		parent::build($container);
		$container->addCompilerPass(new ConfigPass());
	}
}

```

We have added a new compiler pass class called ConfigPass.php. Compiler Pass needs to extend the CompilerPassInterface.


```
# src/AppBundle/DependencyInjection/Compiler/ConfigPass.php

<?php

<?php

namespace AppBundle\DependencyInjection\Compiler;

use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class ConfigPass implements CompilerPassInterface
{
    public function process( ContainerBuilder $container ) {

        // $container->getParameterBag();
        // $container->getServiceIds();

        $config = $container->getParameter('easyadmin.config');

        // use menu to use IS_AUTHENTICATED_FULLY role by default if not set
        foreach($config['design']['menu'] as $k => $v) {
            if (!isset($v['role'])) {
                $config['design']['menu'][$k]['role'] = 'IS_AUTHENTICATED_FULLY';
            }
        }

        // update entities to use IS_AUTHENTICATED_FULLY role by default if not set
        foreach ($config['entities'] as $k => $v) {
            if (!isset($v['role'])) {
                $config['entities'][$k]['role'] = 'IS_AUTHENTICATED_FULLY';
            }
        }

        // update views to use entities role by default if not set
        foreach ($config['entities'] as $k => $v) {
            $views = ['new', 'edit', 'show', 'list', 'form', 'delete'];
            foreach ($views as $view) {
                if (!isset($v[$view]['role'])) {
                    $config['entities'][$k][$view]['role'] = $v['role'];
                }
            }
        }

        $container->setParameter('easyadmin.config', $config);

    }
}
```

What we have done here is to change the easyadmin.config parameter produced by the EasyAdminBundle. easyadmin.config is simply a bunch of arrays built based on the yaml config under app/config/easy_admin. Each for-loop adds a new key called "role" with the default "IS_AUTHENTICATED_FULLY" role if not specified by the config.

EasyAdmin dispatches lots of events. We were already subscribed to it.

```
# src/AppBundle/Resources/config/services.yml
  ...
  app.subscriber:
    class: AppBundle\EventListener\AppSubscriber
    arguments:
        - "@service_container"
    tags:
        - { name: kernel.event_subscriber }
```

We now need to add a bit more logic to the subscriber.

```
# src/AppBundle/EventListener/AppSubscriber.php

class AppSubscriber implements EventSubscriberInterface
{
    ...

    /**
     * show an error if user is not superadmin and tries to manage restricted stuff
     *
     * @param GenericEvent $event event
     * @return null
     * @throws AccessDeniedException
     */
    public function checkUserRights(GenericEvent $event)
        {

            // if super admin, allow all
            $authorization = $this->container->get('security.authorization_checker');
            $request = $this->container->get('request_stack')->getCurrentRequest()->query;
    
            if ($authorization->isGranted('ROLE_ADMIN')) {
                return;
            }
    
            $entity = $request->get('entity');
            $action = $request->get('action');
            $user_id = $request->get('id');
    
            // allow user to see and edit their own profile irregardless of permissions
                    if ($entity == 'User') {
                        // if edit and show
                        if ($action == 'edit' || $action == 'show') {
                            // check user is himself
                            if ($user_id == $this->container->get('security.token_storage')->getToken()->getUser()->getId()) {
                                return;
                            }
                        }
                    }
    
            $config = $this->container->get('easyadmin.config.manager')->getBackendConfig();
    
            // check for permission for each action
            foreach ($config['entities'] as $k => $v) {
                if ($entity == $k && !$authorization->isGranted($v[$action]['role'])) {
                    throw new AccessDeniedException();
                }
            }
        }
    ...
```

We have triggered the checkUserRights function based on a few EasyAdmin events. We have allowed the logged in user to edit his own profile irregardless of role's permission. Then, the for-loop does the magic of allowing or denying user to access different parts of the admin area based on the role key in easyadmin.config.manager service.

Note that this will work only if our AdminController dispatches the events, ie

```
# src/AppBundle/Controller/AdminController.php

...
	/**
	 * Show Page List page
	 * @return \Symfony\Component\HttpFoundation\Response
	 */
    public function listPageAction()
    {
	    $this->dispatch(EasyAdminEvents::PRE_LIST);
	    ...

    }
```

The menu display is not managed by the event subscriber. We have to add an is_granted statement before rendering the menu. See below:

```
# app/Resources/views/easy_admin/menu.html.twig

{% macro render_menu_item(item, translation_domain) %}
    {% if item.type == 'divider' %}
        {{ item.label|trans(domain = translation_domain) }}
    {% else %}
        {% set menu_params = { menuIndex: item.menu_index, submenuIndex: item.submenu_index } %}
        {% set path =
        item.type == 'link' ? item.url :
        item.type == 'route' ? path(item.route, item.params) :
        item.type == 'entity' ? path('easyadmin', { entity: item.entity, action: 'list' }|merge(menu_params)|merge(item.params)) :
        item.type == 'empty' ? '#' : ''
        %}

        {# if the URL generated for the route belongs to the backend, regenerate
           the URL to include the menu_params to display the selected menu item
           (this is checked comparing the beginning of the route URL with the backend homepage URL)
        #}
        {% if item.type == 'route' and (path starts with path('easyadmin')) %}
            {% set path = path(item.route, menu_params|merge(item.params)) %}
        {% endif %}

        <a href="{{ path }}" {% if item.target|default(false) %}target="{{ item.target }}"{% endif %}>
            {% if item.icon is not empty %}<i class="fa {{ item.icon }}"></i>{% endif %}
            <span>{{ item.label|trans(domain = translation_domain) }}</span>
            {% if item.children|default([]) is not empty %}<i class="fa fa-angle-left pull-right"></i>{% endif %}
        </a>
    {% endif %}
{% endmacro %}

{% import _self as helper %}

{% block main_menu_before %}{% endblock %}

<ul class="sidebar-menu">
    {% block main_menu %}
        {% for item in easyadmin_config('design.menu') %}
            {% if is_granted(item.role) %}
                <li class="{{ item.type == 'divider' ? 'header' }} {{ item.children is not empty ? 'treeview' }} {{ app.request.query.get('menuIndex')|default(-1) == loop.index0 ? 'active' }} {{ app.request.query.get('submenuIndex')|default(-1) != -1 ? 'submenu-active' }}">

                    {{ helper.render_menu_item(item, 'app') }}

                    {% if item.children|default([]) is not empty %}
                        <ul class="treeview-menu">
                            {% for subitem in item.children %}
                                <li class="{{ subitem.type == 'divider' ? 'header' }} {{ app.request.query.get('menuIndex')|default(-1) == loop.parent.loop.index0 and app.request.query.get('submenuIndex')|default(-1) == loop.index0 ? 'active' }}">
                                    {{ helper.render_menu_item(subitem, _entity_config.translation_domain|default('messages')) }}
                                </li>
                            {% endfor %}
                        </ul>
                    {% endif %}
                </li>
            {% endif %}
        {% endfor %}
    {% endblock main_menu %}
</ul>

{% block main_menu_after %}{% endblock %}
```

Try logging in now as test1 and you will see that the menu and entities should be access controlled.

## Adding Roles to EasyAdmin Actions

We have seen that easyadmin actions is controlled by the yml files, ie something like:

```
# app/config/easyadmin/userlog.yml
...
show:
    actions: ['list', '-edit', '-delete']
list:
    actions: ['show', '-edit', '-delete']
...
```

What if we want the actions to be "role" aware? If you look at the easyadmin twig files, you will see that it calls a Twig function "getActionsForItem" to get the actions prior to render. This gives us a chance to change the function logic by extending the class.


```
# src/AppBundle/Twig/Extension/EasyAdminTwigExtension.php

namespace AppBundle\Twig\Extension;

use JavierEguiluz\Bundle\EasyAdminBundle\Configuration\ConfigManager;
use Symfony\Component\PropertyAccess\PropertyAccessor;
use Symfony\Component\Security\Core\Authorization\AuthorizationChecker;

/**
 * Class EasyAdminTwigExtension
 * @package AppBundle\Twig\Extension
 */
class EasyAdminTwigExtension extends \JavierEguiluz\Bundle\EasyAdminBundle\Twig\EasyAdminTwigExtension
{
    private $checker;

    public function __construct(ConfigManager $configManager, PropertyAccessor $propertyAccessor, $debug = false, AuthorizationChecker $checker)
    {
        parent::__construct($configManager, $propertyAccessor, $debug);
        $this->checker = $checker;
    }

    /**
     * Overrides parent function
     *
     * @param string $view
     * @param string $entityName
     *
     * @return array
     */
    public function getActionsForItem($view, $entityName)
    {
        $entityConfig = $this->getEntityConfiguration($entityName);
        $disabledActions = $entityConfig['disabled_actions'];
        $viewActions = $entityConfig[$view]['actions'];

        $actionsExcludedForItems = array(
            'list' => array('new', 'search'),
            'edit' => array(),
            'new' => array(),
            'show' => array(),
        );
        $excludedActions = $actionsExcludedForItems[$view];

        // hid these buttons if easyadmin says so
        $actions = ['edit', 'form', 'delete', 'list', 'show'];
        foreach ($actions as $action) {
            if (isset($entityConfig[$action]['role']) && !$this->checker->isGranted($entityConfig[$action]['role'])) {
                array_push($excludedActions, $action);
            }
        }
        
        return array_filter($viewActions, function ($action) use ($excludedActions, $disabledActions) {
            return !in_array($action['name'], $excludedActions) && !in_array($action['name'], $disabledActions);
        });
    }
}
```

And we have to remember to call our new twig class in services.yml

```
# src/AppBundle/Resources/config/services.yml
  ...
  app.twig.extension:
    class: AppBundle\Twig\Extension\EasyAdminTwigExtension
    arguments:
        - "@easyadmin.config.manager"
        - "@property_accessor"
        - "%kernel.debug%"
        - "@security.authorization_checker"
    tags:
      - { name: twig.extension }
```

One thing to remember though is that we have to load our AppBundle after EasyAdminbundle so that our app.twig.extension can override the easyadmin.twig.extension service of EasyAdminBundle

```
# app/AppKernel.php
...
        $bundles = [
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\SecurityBundle\SecurityBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),
            new Symfony\Bundle\MonologBundle\MonologBundle(),
            new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            new Doctrine\Bundle\DoctrineBundle\DoctrineBundle(),
            new Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle(),
            new Vich\UploaderBundle\VichUploaderBundle(),
            // init my fosuser
            new FOS\UserBundle\FOSUserBundle(),
            new Doctrine\Bundle\MigrationsBundle\DoctrineMigrationsBundle(),
            new JavierEguiluz\Bundle\EasyAdminBundle\EasyAdminBundle(),
            new Bpeh\NestablePageBundle\BpehNestablePageBundle(),
            new Ivory\CKEditorBundle\IvoryCKEditorBundle(),
            new AppBundle\AppBundle(),
            new AppBundle\User(),
            new AppBundle\Page(),
...            
```

I have disabled the "edit" action for all users, so the edit button will not show even if the user is himself. For the sake of simplicity, let us change the layout header link to use edit action instead.

```
# app/Resources/EasyAdminBundle/views/default/layout.html.twig
...
<a href="{{ path('easyadmin') }}/?entity=User&action=edit&id={{ app.user.id }}">{{ app.user.username|default('user.unnamed'|trans(domain = 'EasyAdminBundle')) }}</a>
...
```

## Cleaning up

We are close to the end of the chapter. Let us clean up all our code using php-cs-fixer (Still remember this?)

```
-> vendor/friendsofphp/php-cs-fixer/php-cs-fixer fix src/
-> vendor/friendsofphp/php-cs-fixer/php-cs-fixer fix src/
# finally optimising composer
-> ./scripts/optimize_composer
```

## Update BDD (Optional)

We have updated some business rules. Users can now see and do what they are allowed in the admin area based on their role in the easyadmin yaml config files. Its time to ensure we update our tests to reflect these changes.

## Summary

In this chapter, we have cleaned up config.yml and provided a custom solution (Using compiler pass and event listeners) to make EasyAdmin support user permissions in the admin area. It was a huge effort but not yet a full solution. However, it should make life easy for people who wants to configure admin permissions easily.

## Exercises

* Think of another way to make EasyAdmin support user permissions.

* Write your test and make sure everything passes (Optional)

* Can you implement [autowiring](http://symfony.com/doc/current/components/dependency_injection/autowiring.html) in services.yml? What are the pros and cons of using autowiring?

## References

* [Prepend Config](http://symfony.com/doc/current/bundles/prepend_extension.html)

* [Dependency Injection Component](https://symfony.com/doc/current/components/dependency_injection.html)

* [Service Container](http://symfony.com/doc/current/service_container.html)

* [Tagging Symfony Services](http://thorpesystems.com/blog/tagging-symfony-services/)

* [Autowiring](http://symfony.com/doc/current/components/dependency_injection/autowiring.html)
