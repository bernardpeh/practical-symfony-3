# Chapter 19: The Page Manager Part 2

In this chapter, we are going to integrate NestablePageBundle with EasyAdminBundle. We are also going to improve the cms by integrating a wysiwyg editor (ckeditor) and create a custom locale dropdown.

## Define User Stories

**19. Page Management**

|**Story Id**|**As a**|**I**|**So that I**|
|19.1|an admin|want to manage pages|update them anytime.|
|19.2|test1 user|don't want to manage pages|don't breach security|

**Story ID 19.1: As an admin, I want to manage pages, so that I can update them anytime.**

|**Scenario Id**|**Given**|**When**|**Then**|
|19.11|List Pages|I go to page list url|I can see 2 elements under the about slug|
|19.12|Show Contact Us Page|I go to contact_us page|I should see the word "contact_us" and the word "Created"|
|19.13|Reorder home|I drag and drop the home menu to under the about menu|I should see "reordered successfully message" in the response and see 3 items under the about menu|
|19.14|edit home page meta|I go to edit homepage url and update the menu title of "Home" to "Home1" and click update|I should see the menu updated to home1|
|19.15|Create and delete test page|go to page list and click "Add new page" and fill in details and click "Create" button, go to newly created test page and create 2 new test meta. Delete one testmeta and then delete the whole test page|I should see the first pagemeta being created and deleted. Then see the second testmeta being deleted when the page is being deleted.|
|19.16|Delete Contact Us Page|go to contact us page and click "delete"|I should see that the contact us page and its associate meta being deleted.|
|19.17|Create new page with existing locale|go to page list and click "Add new pagemeta" and fill in details, select locale as en, page as home and click "Create" button|I should see an exception.|

**Story ID 19.2: As test1 user, I don't want to manage pages, so that I don't breach security.**

|**Scenario Id**|Given**|**When**|**Then**|
|19.21|List pages|I go to the page management url|I should get a access denied message|
|19.22|show about us page|I go to show about us url|I should get a access denied message|
|19.23|edit about us page|I go to edit about us url|I should get a access denied message|

## Adding new image field to PageMeta Entity

Let us add a new field called featuredImage to the PageMeta entity. We will configure Vich uploader to do the job.

```
# src/AppBundle/Entity/PageMeta.php

namespace AppBundle\Entity;

use Bpeh\NestablePageBundle\Model\PageMetaBase;
use Doctrine\ORM\Mapping as ORM;
use Vich\UploaderBundle\Mapping\Annotation as Vich;
use Symfony\Component\HttpFoundation\File\File;

/**
 * PageMeta
 *
 * @ORM\Table(name="pagemeta")
 * @ORM\Entity(repositoryClass="AppBundle\Repository\PageMetaRepository")
 * @ORM\HasLifecycleCallbacks()
 * @Vich\Uploadable
 *
 */
class PageMeta extends PageMetaBase
{
    /**
     * @var integer
     *
     * @ORM\Column(name="id", type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    protected $id;

    /**
     * @ORM\Column(type="string", length=255, nullable=true)
     * @var string
     */
    private $featuredImage;

    /**
     * @Vich\UploadableField(mapping="featured_image", fileNameProperty="featuredImage")
     * @var File
     */
    private $featuredImageFile;

    /**
     * Get id
     *
     * @return integer
     */
    public function getId()
    {
        return $this->id;
    }

    /**
     * @param File|null $image
     */
    public function setFeaturedImageFile(File $image = null)
    {
        $this->featuredImageFile = $image;

        if ($image) {
            $this->setModified(new \DateTime());
        }
    }

    /**
     * @return File
     */
    public function getFeaturedImageFile()
    {
        return $this->featuredImageFile;
    }

    /**
     * @param $image
     */
    public function setFeaturedImage($image)
    {
        $this->featuredImage = $image;
    }

    /**
     * @return string
     */
    public function getFeaturedImage()
    {
        return $this->featuredImage;
    }

    /**
     * @return string
     */
    public function __toString()
    {
        return $this->getLocale().': '.$this->getMenuTitle();
    }
}
```

Let us update config.yml

```
# app/config/config.yml
parameters:
    locale: en
    supported_lang: [ 'en', 'fr']
    admin_path: admin
    app.profile_image.path: /uploads/profiles
    app.featured_image.path: /uploads/featured_images
...
vich_uploader:
    db_driver: orm
    mappings:
        profile_images:
            uri_prefix: '%app.profile_image.path%'
            upload_destination: '%kernel.root_dir%/../web/uploads/profiles'
            namer: vich_uploader.namer_uniqid
        featured_image:
            uri_prefix: '%app.featured_image.path%'
            upload_destination: '%kernel.root_dir%/../web/uploads/featured_images'
            namer: vich_uploader.namer_uniqid
```

## Installing CKEditor

We will now install CKEditor

```
-> ./scripts/composer require egeloen/ckeditor-bundle
```

then enable the bundle

```
# app/AppKernel.php
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        return array(
            // ...
            new Ivory\CKEditorBundle\IvoryCKEditorBundle(),
        );
    }
}
```

## Integration with EasyAdminBundle

There is still some effort to get BpehNestablePageBundle integrate properly with EasyAdminBundle. The reason is because the big difference in controller logic between the 2 bundles.

Let us assume that we not going to use the PageController.php and PageMetaController.php except the reorder route

The new routing.yml as follows:

```
# app/config/routing.yml

admin:
  resource: "@AppBundle/Controller/AdminController.php"
  type:     annotation

locale:
  resource: "@AppBundle/Controller/LocaleController.php"
  type:     annotation

bpeh_page_reorder:
  path: /admin/reorder
  defaults:
    _controller: BpehNestablePageBundle:Page:reorder

# FOS user bundle default routing
fos_user_security:
  resource: "@FOSUserBundle/Resources/config/routing/security.xml"

fos_user_resetting:
  resource: "@FOSUserBundle/Resources/config/routing/resetting.xml"
  prefix: /resetting

easy_admin_bundle:
  resource: "@AppBundle/Controller/AdminController.php"
  type:     annotation
  prefix:   /%admin_path%
```

We now need to add actions to the AdminController. The new AdminController should look like this:

```
# src/AppBundle/Controller/AdminController.php

namespace AppBundle\Controller;

use JavierEguiluz\Bundle\EasyAdminBundle\Controller\AdminController as BaseAdminController;
use JavierEguiluz\Bundle\EasyAdminBundle\Event\EasyAdminEvents;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use AppBundle\Entity\PageMeta;

class AdminController extends BaseAdminController
{
    /**
     * @Route("/dashboard", name="dashboard")
     *
     * @param Request $request
     * @return \Symfony\Component\HttpFoundation\Response
     *
     */
    public function dashboardAction(Request $request)
    {
        return $this->render('@EasyAdmin/default/dashboard.html.twig');
    }

    /**
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function showUserAction()
    {
        $this->dispatch(EasyAdminEvents::PRE_SHOW);
        $id = $this->request->query->get('id');
        $easyadmin = $this->request->attributes->get('easyadmin');
        $entity = $easyadmin['item'];

        $fields = $this->entity['show']['fields'];

        if (!$this->isGranted('ROLE_SUPER_ADMIN')) {
            unset($fields['created']);
        }

        $deleteForm = $this->createDeleteForm($this->entity['name'], $id);

        return $this->render($this->entity['templates']['show'], array(
            'entity' => $entity,
            'fields' => $fields,
            'delete_form' => $deleteForm->createView(),
        ));
    }

    /**
     * when edit user action
     *
     * @return Response|\Symfony\Component\HttpFoundation\RedirectResponse|\Symfony\Component\HttpFoundation\Response
     */
    protected function editUserAction()
    {
        $this->dispatch(EasyAdminEvents::PRE_EDIT);
        $id = $this->request->query->get('id');
        $easyadmin = $this->request->attributes->get('easyadmin');
        $entity = $easyadmin['item'];

        if ($this->request->isXmlHttpRequest() && $property = $this->request->query->get('property')) {
            $newValue = 'true' === strtolower($this->request->query->get('newValue'));
            $fieldsMetadata = $this->entity['list']['fields'];

            if (!isset($fieldsMetadata[$property]) || 'toggle' !== $fieldsMetadata[$property]['dataType']) {
                throw new \RuntimeException(sprintf('The type of the "%s" property is not "toggle".', $property));
            }

            $this->updateEntityProperty($entity, $property, $newValue);

            return new Response((string)$newValue);
        }

        $fields = $this->entity['edit']['fields'];

        $editForm = $this->createEditForm($entity, $fields);
        if (!$this->isGranted('ROLE_SUPER_ADMIN')) {
            $editForm->remove('enabled');
            $editForm->remove('roles');
            $editForm->remove('locked');
        }

        $deleteForm = $this->createDeleteForm($this->entity['name'], $id);

        $editForm->handleRequest($this->request);
        if ($editForm->isValid()) {
            $this->preUpdateUserEntity($entity);
            $this->em->flush();

            $refererUrl = $this->request->query->get('referer', '');

            return !empty($refererUrl)
                ? $this->redirect(urldecode($refererUrl))
                : $this->redirect($this->generateUrl('easyadmin', array('action' => 'show', 'entity' => $this->entity['name'], 'id' => $id)));
        }

        return $this->render($this->entity['templates']['edit'], array(
            'form' => $editForm->createView(),
            'entity_fields' => $fields,
            'entity' => $entity,
            'delete_form' => $deleteForm->createView(),
        ));
    }

    public function createNewUserEntity()
    {
        return $this->get('fos_user.user_manager')->createUser();
    }

    public function prePersistUserEntity($user)
    {
        $this->get('fos_user.user_manager')->updateUser($user, false);
    }

    public function preUpdateUserEntity($user)
    {
        $this->get('fos_user.user_manager')->updateUser($user, false);
    }

    /**
     * Show Page List page
     *
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function listPageAction()
    {
        $rootMenuItems = $this->getDoctrine()->getRepository('AppBundle\Entity\Page')->findParent();

        return $this->render('@EasyAdmin/Page/list.html.twig', array(
            'tree' => $rootMenuItems,
        ));
    }

    /**
     * save page meta
     *
     * @param PageMeta $pageMeta
     */
    public function prePersistPageMetaEntity(PageMeta $pageMeta)
    {
        if ( $this->em->getRepository('AppBundle\Entity\PageMeta')->findPageMetaByLocale( $pageMeta->getPage(), $pageMeta->getLocale() ) ) {
            throw new \RuntimeException($this->get('translator')->trans('one_locale_per_pagemeta_only', array(), 'BpehNestablePageBundle') );
        }

    }

    /**
     * edit page meta
     *
     * @return Response|\Symfony\Component\HttpFoundation\RedirectResponse|\Symfony\Component\HttpFoundation\Response
     */
    protected function editPageMetaAction()
    {
        $this->dispatch(EasyAdminEvents::PRE_EDIT);

        $id = $this->request->query->get('id');
        $easyadmin = $this->request->attributes->get('easyadmin');
        $entity = $easyadmin['item'];

        // get id before submission
        $pageMeta = $this->em->getRepository('AppBundle\Entity\PageMeta')->find($id);
        $origId = $pageMeta->getPage()->getId();
        $origLocale = $pageMeta->getLocale();

        if ($this->request->isXmlHttpRequest() && $property = $this->request->query->get('property')) {
            $newValue = 'true' === strtolower($this->request->query->get('newValue'));
            $fieldsMetadata = $this->entity['list']['fields'];

            if (!isset($fieldsMetadata[$property]) || 'toggle' !== $fieldsMetadata[$property]['dataType']) {
                throw new \RuntimeException(sprintf('The type of the "%s" property is not "toggle".', $property));
            }

            $this->updateEntityProperty($entity, $property, $newValue);

            return new Response((string) $newValue);
        }

        $fields = $this->entity['edit']['fields'];

        $editForm = $this->createEditForm($entity, $fields);
        $deleteForm = $this->createDeleteForm($this->entity['name'], $id);

        $editForm->handleRequest($this->request);
        if ($editForm->isValid()) {
            $this->dispatch(EasyAdminEvents::PRE_UPDATE, array('entity' => $entity));

            // if page and local is the same, dont need to check locale count
            if ($origLocale == $entity->getLocale() && $origId == $entity->getPage()->getId()) {
                // all good
            }
            elseif ( $this->em->getRepository('AppBundle\Entity\PageMeta')->findPageMetaByLocale( $pageMeta->getPage(), $pageMeta->getLocale(), true ) ) {
                throw new \RuntimeException($this->get('translator')->trans('one_locale_per_pagemeta_only', array(), 'BpehNestablePageBundle') );
            }

            $this->em->flush();

            $this->dispatch(EasyAdminEvents::POST_UPDATE, array('entity' => $entity));

            $refererUrl = $this->request->query->get('referer', '');

            return !empty($refererUrl)
                ? $this->redirect(urldecode($refererUrl))
                : $this->redirect($this->generateUrl('easyadmin', array('action' => 'list', 'entity' => $this->entity['name'])));
        }

        $this->dispatch(EasyAdminEvents::POST_EDIT);

        return $this->render($this->entity['templates']['edit'], array(
            'form' => $editForm->createView(),
            'entity_fields' => $fields,
            'entity' => $entity,
            'delete_form' => $deleteForm->createView(),
        ));
    }
}
```

Let us create the list view. We have to recreate it because we are extending it from the easyadmin layout instead.

```
-> mkdir -p app/Resources/EasyAdminBundle/views/Page
-> touch app/Resources/EasyAdminBundle/views/Page/list.html.twig
```

and the contents of list.html.twig
```
# app/Resources/EasyAdminBundle/views/Page/list.html.twig

{% extends '@EasyAdmin/default/list.html.twig' %}

{% block head_stylesheets %}
    {{ parent() }}
    <link rel="stylesheet" href="{{ asset('bundles/bpehnestablepage/css/styles.css ') }}">
{% endblock %}

{% block head_javascript %}
    {{ parent() }}
    <script src="{{ asset('bundles/bpehnestablepage/js/jquery.nestable.js') }}"></script>
    <script>
        $(function() {

            var before = null, after = null;

            $('.dd').nestable({
                afterInit: function ( event ) { }
            });

            $('.dd').nestable('collapseAll');
            before = JSON.stringify($('.dd').nestable('serialize'));
            $('.dd').on('dragEnd', function(event, item, source, destination, position) {

                id = item.attr('data-id');
                parentId = item.closest('li').parent().closest('li').attr('data-id');

                // if parent id is null of if parent id and id is the same, it is the top level.
                parentId = (parentId == id || typeof(parentId)  === "undefined") ?  '' : parentId;

                after = JSON.stringify($('.dd').nestable('serialize'));

                token = '{{ csrf_token("bpeh_page_reorder") }}';

                if (before != after) {
                    $.ajax({
                        type: "POST",
                        url: "{{ path('bpeh_page_reorder') }}",
                        data: { id: id, parentId: parentId, position: position, csrf: token },
                        success: function (data, dataType) {
                            if (data.success) {
                                $('.alert').addClass('alert-success');
                            }
                            else {
                                $('.alert').addClass('alert-danger');
                            }
                            $('.alert').html(data.message);
                            $('.alert').fadeTo( 0 , 1, function() {});
                            $('.alert').fadeTo( 4000 , 0, function() {});
                        },

                        error: function (XMLHttpRequest, textStatus, errorThrown) {
                            console.log(XMLHttpRequest);
                        }
                    });
                    before = after;
                }
            });
        });
    </script>
{% endblock %}


{% block main %}
    <div class="alert alert-dismissable">
        {{ 'flash_reorder_instructions' | trans({}, 'BpehNestablePageBundle') }}
    </div>

    <button type="button" onclick="$('.dd').nestable('expandAll')">{{ 'expand_all'|trans({}, 'BpehNestablePageBundle') }}</button>
    <button type="button" onclick="$('.dd').nestable('collapseAll')">{{ 'collapse_all'|trans({}, 'BpehNestablePageBundle') }}</button>
    <button onclick=window.location="{{ path('easyadmin') }}?entity=Page&action=new">{{ 'new_page'|trans({}, 'BpehNestablePageBundle') }}</button>
    	<button onclick=window.location="{{ path('easyadmin') }}?entity=PageMeta&action=new">{{ 'new_pagemeta'|trans({}, 'BpehNestablePageBundle') }}</button>
    <div id="nestable" class="dd">
        <ol class="dd-list">
            {% include "@BpehNestablePage/Page/tree.html.twig" with { 'tree':tree } %}
        </ol>
    </div>
{% endblock main %}
```

and don't forget about tree.html.twig

```
# app/Resources/EasyAdminBundle/views/Page/tree.html.twig

{% for v in tree %}
    <li class='dd-item' data-id='{{ v.getId() }}'>
        <div class='dd-handle'>
            <a class="dd-nodrag" href="{{ path('easyadmin') }}?entity={{ _entity_config.name }}&action=edit&{{ _entity_config.primary_key_field_name }}={{ v.getId() }}">{{ v.getSlug() }}</a>
        </div>

        {% set children = v.getChildren()|length %}
        {% if children > 0 %}
            <ol class='dd-list'>
                {% include "@EasyAdmin/BpehNestablePageBundle:Page:tree.html.twig" with { 'tree':v.getChildren() } %}
            </ol>
        {% endif %}
    </li>
{% endfor %}
```

Finally - the easyadmin config. We do not want to display the pagemeta menu.

```
# app/config/easyadmin/design.yml

easy_admin:
    design:
        brand_color: '#337ab7'
        assets:
            css:
              - /bundles/app/css/style.css
        menu:
          - { entity: 'User', icon: 'user' }
          - { entity: 'Page', icon: 'file' }
          - { entity: 'UserLog', icon: 'database' }
```

and

```
# app/config/easyadmin/page.yml

easy_admin:
    entities:
        Page:
            class: AppBundle\Entity\Page
            label: admin.link.page_management
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

Noticed the new field type we have used, ie ckeditor, vich_image, and AppBundle\Form\LocaleType. EasyAdminBundle has internal support for ckeditor and vich_image but AppBundle\Form\LocaleType is our own custom form selector which will be discussed in the next section.

## Creating Custom Locale Selector Form Type

If you are looking at the pagemeta page, say http://songbird.app:8000/app_dev.php/admin/?entity=PageMeta&action=new for example, you should have noticed by now that user can enter anything under the locale textbox. What if we want to load only the languages that we defined in the config file (ie, english and french)? It is a good idea to create our own dropdown.

```
# src/AppBundle/Form/LocaleType.php

namespace AppBundle\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;

class LocaleType extends AbstractType
{
    private $localeChoices;

    public function __construct(array $localeChoices)
    {
    	foreach ($localeChoices as $v) {
            $this->localeChoices[$v] = $v;
        }
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'choices' => $this->localeChoices,
        ));
    }

    public function getParent()
    {
        return ChoiceType::class;
    }

}
```

The array localChoices is passed into the constructor. This class can be lazy loaded if we define it in service.yml

```
# src/AppBundle/Resources/config/services.yml
...
  app.form.type.locale:
    class: AppBundle\Form\LocaleType
    arguments:
      - "%supported_lang%"
    tags:
      - { name: form.type }
...
```

See how we pass the supported_lang config variable into the class? Now, go to any pagemeta new or edit page (ie http://songbird.app:8000/app_dev.php/admin/?entity=PageMeta&action=new for example) and you should see the locale dropdown updated to only 2 enties.

Let us update the translation files

```
# src/AppBundle/Resources/translations/app.en.xlf
...
<trans-unit id="7">
    <source>admin.page_management</source>
    <target>Page Management</target>
</trans-unit>
..
```

and the french version

```
# src/AppBundle/Resources/translations/app.fr.xlf
...
<trans-unit id="7">
    <source>admin.page_management</source>
    <target>Gestion de la page</target>
</trans-unit>
...
```

There were db changes. remember to run doctrine migrations

```
-> ./scripts/console doctrine:migrations:diff
```

## Update BDD Tests (Optional)

Let us create the cest files,

```
-> ./vendor/bin/codecept generate:cest -c src/AppBundle acceptance As_An_Admin/IWantToManagePages
-> ./vendor/bin/codecept generate:cest -c src/AppBundle acceptance As_Test1_User/IDontWantToManagePages
```

Create the test cases from the scenarios above and make sure all your tests passes before moving on.

Remember to commit all your code before moving on to the next chapter.

## Summary

In this chapter, we have extended our NestablePageBundle in EasyAdmin. We have installed CKEditor in our textarea and created a customised locale dropdown based on values from our config.yml file. Our CMS is looking more complete now.

## Exercises

* From the debug toolbar, update the missing translations.

* TinyMCE is also a widely used WYSIWYG editor. How do you integrate it in Sonata Media?

* What if you want to add a new user field to the Page Management System? What is going to happen to the page if the user is deleted? 

* Can you make inserting pagemeta easier for every new page added? This just shows how much thought one person need to put when creating a software.

## References

* [Create custom form type](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html)

* [EasyAdmin Templating](https://github.com/javiereguiluz/EasyAdminBundle/blob/master/Resources/doc/book/3-list-search-show-configuration.md)

* [CKEditor Bundle](https://github.com/egeloen/IvoryCKEditorBundle)

* [Adding Wysiwyg Editor](https://github.com/javiereguiluz/EasyAdminBundle/blob/master/Resources/doc/tutorials/wysiwyg-editor.md)