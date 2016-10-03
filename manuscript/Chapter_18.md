# Chapter 18: Making Your Bundle Reusable

We have created a page bundle in the previous chapter. It's not perfect but let's say we want to share it with everyone. How do we do that? Be warned, we need lots of refactoring in the code to make it sharable.

This is a long chapter. Its is a good process to go through because it makes you pause and think. If you already know the process and want to skip through, simple clone the [NestablePageBundle from github](https://github.com/bernardpeh/NestablePageBundle) and follow the installation instructions in the readme file. Then, jump over to the next chapter.

## Creating a separate repository

First of all, let us create a readme file.

```
-> cd src/Songbird/NestablePageBundle
-> touch readme.md
```

Update the readme file.

Let us create the composer.json file for this repo. We will do a simple one

```
-> composer init
```

Follow the prompts. You might need to read up on software licensing. [MIT license](https://en.wikipedia.org/wiki/MIT_License) is becoming really popular. The sample composer.json might look like this:

```
{
    "name": "Yourname/nestable-page-bundle",
    "description": "your description",
    "type": "symfony-bundle",
    "require": {
        "symfony/symfony": "~3.0"
    },
    "require-dev": {
        "doctrine/doctrine-fixtures-bundle": "~2.0"
    },
    "autoload": {
        "psr-4": { "Songbird\NestablePageBundle\": "" }
    },
    "license": "MIT",
    "authors": [
        {
            "name": "your name",
            "email": "your_email@your_email.xx"
        }
    ]
}
```

Note that we have to add the "autoload" component so that Symfony can autoload the namespace post installation. <a href="https://getcomposer.org/doc/04-schema.md#autoload">PS-4</a> is the default standard at the time of writing. Next, let us create the license in a text file

```
-> touch LICENSE
```

copy the [MIT LICENSE](http://opensource.org/licenses/MIT) and update the LICENSE file.

Init the repo

```
-> cd src/Songbird/NestablePageBundle
-> git init .
-> git add .
-> git commit -m"init commit"
```

In [github](http://github.com) (create a new acct if not done), create a new repo. Let's call it NestablePageBundle for example. Once you have created the new repo, you should see instructions on how to push your code.

```
-> git remote add origin git@github.com:your_username/NestablePageBundle.git
-> git push -u origin master
```

Let us give our first release a version number using the [semantic versioning](http://semver.org) convention.

```
-> git tag 0.1.0
-> git push --tags
```

Your repository is now available for the public to pull.

## Updating Application composer.json

Note that this composer.json is different from the one that we have just created. If we add our repo to [packagist](https://packagist.org), we could install our bundle like any other bundles by using the "composer require" command. I was afraid that anyone reading this tutorial might submit their test bundle to packagist, so I thought it would be a better idea to install the bundle from git instead. Let's use github for the sake of illustration.

```
# composer.json
...
    "repositories": [
        {
            "type": "git",
            "url": "https://github.com/your_name/NestablePageBundle"
        }
    ],
...
    "require": {
        ...
        "your_name/nestable-page-bundle": ">0.1.0"
    }
...
```

Note that the bundle name is "nestable-page-bundle" under the "require" section. Why not use NestablePageBundle following Symfony's convention? Remember the composer.json file that you have created previously? "nestable-page-bundle" is the name of the bundle as specified in that composer file.

Now lets run composer update and see what happens

```
-> cd ../../../
-> composer update
...

  - Installing your_name/nestable-page-bundle (0.1.0)
    Downloading: 100%
```

At this point, look at the vendor directory and you will see your bundle being installed in there. That's a good start.

## Renaming SongbirdNestablePageBundle

Let us do some cleaning up. We no longer need the src/Songbird/NestablePageBundle since we have installed the bundle under vendor dir.

```
git rm -rf src/Songbird/
```
Let us check if the route is still there.

```
-> app/console debug:router | grep songbird
...
songbird_page            GET      ANY    ANY  /songbird_page/
songbird_page_list       GET      ANY    ANY  /songbird_page/list
songbird_page_reorder    POST     ANY    ANY  /songbird_page/reorder
```

Woah!! We have already deleted src/Songbird/NestablePageBundle and we should expect to see some errors. Why are the songbird routes still there?

We have a problem. The namespace "Songbird" is no longer relevant in vendor/your-name/nestable-page-bundle since the bundle is already decoupled from Songbird CMS. We want to change the bundle's filename and namespace so that it is more intuitive. How do we do that?

Let us re-download the repo and do some mass restructuring

```
-> cd vendor/your-name
-> rm -rf nestable-page-bundle
-> git clone git@github.com:your_name/NestablePageBundle.git nestable-page-bundle
-> cd nestable-page-bundle
```

There is no quick way for this, some bash magic helps

```
# Your-Initial can be something short but has to be unique
# let us change the namespace
-> find . -type f | grep -v .git/ | while read s; do sed -i '' 's/Songbird\NestablePageBundle/{your-initial}\NestablePageBundle/g' $s ; done
# change the bundle name
-> find . -type f | grep -v .git/ | while read s; do sed -i '' 's/SongbirdNestablePage/{your-initial}NestablePage/g' $s ; done
-> find . -type f | grep -v .git/ | while read s; do sed -i '' 's/songbird_/{your_initial}_/g' $s ; done
-> find . -type f | grep -v .git/ | while read s; do sed -i '' 's/songbirdnestable/{your_initial}nestable/g' $s ; done
```

That should save us 90% of the time. Then visually walk through all the files and rename whatever that was not renamed by the bash commands.

Lastly, rename the bundle file

```
-> git mv SongbirdNestablePageBundle.php {your-initial}NestablePageBundle.php
-> cd DependencyInjection
-> git mv SongbirdNestablePageExtension.php BpehNestablePageExtension.php
-> cd ../Resources/translations
-> git mv SongbirdNestablePageBundle.en.xlf {your-initial}NestablePageBundle.en.xlf
-> git mv SongbirdNestablePageBundle.fr.xlf {your-initial}NestablePageBundle.fr.xlf
```

Now, here is the question. How do we test our changes without committing to git and re-run composer update? We can update our entry in vendor/composer/autoload_psr4.php

```
# vendor/composer/autoload_psr4.php
...
    # 'Songbird\NestablePageBundle\' => array($vendorDir . '/{your-name}/nestable-page-bundle'),
    '{your-initial}\NestablePageBundle\' => array($vendorDir . '/{your-name}/nestable-page-bundle'),
...
```

Now, let us update AppKernel

```
# app/config/AppKernel.php
# new SongbirdNestablePageBundleSongbirdNestablePageBundle(),
new {your-initial}NestablePageBundle{your-initial}NestablePageBundle(),
```

and routing

```
# app/config/routing.yml

# songbird_nestable_page:
#     resource: "@SongbirdNestablePageBundle/Controller/"
#    type:     annotation
#    prefix:   /

{your-initial}_nestable_page:
    resource: "@{your-initial}/NestablePageBundle/Controller/"
    type:     annotation
    prefix:   /
```

My initial is bpeh, let us check that the routes are working.

```
app/console debug:router | grep bpeh
bpeh_page                GET      ANY    ANY  /bpeh_page/
bpeh_page_list           GET      ANY    ANY  /bpeh_page/list
bpeh_page_reorder        POST     ANY    ANY  /bpeh_page/reorder
...
```

We can now install the assets.

```
-> ./scripts/assetsinstall
```

Now go your new page list url and do a quick test. In my case,

```
http://songbird.app/app_dev.php/bpeh_page/list
```

Looks like it is working. How can we be sure? Remember our functional tests?

```
-> phpunit -c app vendor/{your-name}/nestable-page-bundle/
...
Time: 29.04 seconds, Memory: 88.50Mb

OK (5 tests, 14 assertions)
```

Remember to commit your code before moving to the next chapter. Up your nestablepagebundle tags to 0.2.0 or something else since there were major changes.

## Making the Bundle Extensible

When this bundle is initialised in AppKernel.php, running "app/console doctrine:schema:create will create the default tables. We should be able to extend this bundle and modify the entity name and methods easily. The war is not over. There are still lots to be done!!

Let us clean up the AppKernel and Route.

```
# app/AppKernel.php
...
// new {your-inital}NestablePageBundle{your-initial}NestablePageBundle(),
...
```

and in routing.yml

```
# app/config/routing.yml

# {your-initial}_nestable_page:
# resource: "@{your-initial}NestablePageBundle/Controller/"
# type:     annotation
# prefix:   /
```

and refocus our attention to the NestablePageBundle:

```
-> cd vendor/{your-initial}/NestablePageBundle
```

First of all, we need to make Page and PageMeta entities extensible. We will move the entities to the Model directory, making the entities abstract. 

I'll be using my initial (bpeh) from now onwards to make life easier when referencing paths.

```
# vendor/bpeh/nestable-page-bundle/Model/PageBase.php

namespace Bpeh\NestablePageBundle\Model;

use Doctrine\ORM\Mapping as ORM;

/**
 * Page
 *
 */
abstract class PageBase
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
     * @var string
     *
     * @ORM\Column(name="slug", type="string", length=255, unique=true)
     */
    protected $slug;

    /**
     * @var boolean
     *
     * @ORM\Column(name="isPublished", type="boolean", nullable=true)
     */
    protected $isPublished;

    /**
     * @var integer
     *
     * @ORM\Column(name="sequence", type="integer", nullable=true)
     */
    protected $sequence;

    /**
     * @var \DateTime
     *
     * @ORM\Column(name="modified", type="datetime")
     */
    protected $modified;

    /**
     * @var \DateTime
     *
     * @ORM\Column(name="created", type="datetime")
     */
    protected $created;


    /**
     * @ORM\ManyToOne(targetEntity="Bpeh\NestablePageBundle\Model\PageBase", inversedBy="children")
     * @ORM\JoinColumn(name="parent_id", referencedColumnName="id", onDelete="CASCADE")}
     * @ORM\OrderBy({"sequence" = "ASC"})
     */
    protected $parent;

    /**
     * @ORM\OneToMany(targetEntity="Bpeh\NestablePageBundle\Model\PageBase", mappedBy="parent")
     * @ORM\OrderBy({"sequence" = "ASC"})
     */
    protected $children;

    /**
     * @ORM\OneToMany(targetEntity="Bpeh\NestablePageBundle\Model\PageMetaBase", mappedBy="page", cascade={"persist"}))
     */
    protected $pageMetas;

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
     * Set slug
     *
     * @param string $slug
     * @return Page
     */
    public function setSlug($slug)
    {
        $this->slug = $slug;

        return $this;
    }

    /**
     * Get slug
     *
     * @return string
     */
    public function getSlug()
    {
        return $this->slug;
    }

    /**
     * Set isPublished
     *
     * @param boolean $isPublished
     * @return Page
     */
    public function setIsPublished($isPublished)
    {
        $this->isPublished = $isPublished;

        return $this;
    }

    /**
     * Get isPublished
     *
     * @return boolean
     */
    public function getIsPublished()
    {
        return $this->isPublished;
    }

    /**
     * Set sequence
     *
     * @param integer $sequence
     * @return Page
     */
    public function setSequence($sequence)
    {
        $this->sequence = $sequence;

        return $this;
    }

    /**
     * Get sequence
     *
     * @return integer
     */
    public function getSequence()
    {
        return $this->sequence;
    }

    /**
     * Set modified
     *
     * @param \DateTime $modified
     * @return Page
     */
    public function setModified($modified)
    {
        $this->modified = $modified;

        return $this;
    }

    /**
     * Get modified
     *
     * @return \DateTime
     */
    public function getModified()
    {
        return $this->modified;
    }

    /**
     * Set created
     *
     * @param \DateTime $created
     * @return Page
     */
    public function setCreated($created)
    {
        $this->created = $created;

        return $this;
    }

    /**
     * Get created
     *
     * @return \DateTime
     */
    public function getCreated()
    {
        return $this->created;
    }
    /**
     * Constructor
     */
    public function __construct()
    {
        $this->children = new \Doctrine\Common\Collections\ArrayCollection();
        $this->pageMetas = new \Doctrine\Common\Collections\ArrayCollection();
    }

    /**
     * @ORM\PrePersist
     */
    public function prePersist()
    {
        // update the modified time
        $this->setModified(new \DateTime());

        // for newly created entries
        if ($this->getCreated() == null) {
            $this->setCreated(new \DateTime('now'));
        }
        $this->created = new \DateTime();
    }

    /**
     * Set parent
     *
     * @param \Bpeh\NestablePageBundle\Model\PageBase $parent
     * @return Page
     */
    public function setParent(\Bpeh\NestablePageBundle\Model\PageBase $parent = null)
    {
        $this->parent = $parent;

        return $this;
    }

    /**
     * Get parent
     *
     * @return \Bpeh\NestablePageBundle\Model\PageBase
     */
    public function getParent()
    {
        return $this->parent;
    }

    /**
     * Add children
     *
     * @param \Bpeh\NestablePageBundle\Model\PageBase $children
     * @return Page
     */
    public function addChild(\Bpeh\NestablePageBundle\Model\PageBase $children)
    {
        $this->children[] = $children;

        return $this;
    }

    /**
     * Remove children
     *
     * @param \Bpeh\NestablePageBundle\Model\Page $children
     */
    public function removeChild(\Bpeh\NestablePageBundle\Model\PageBase $children)
    {
        $this->children->removeElement($children);
    }

    /**
     * Get children
     *
     * @return \Doctrine\Common\Collections\Collection
     */
    public function getChildren()
    {
        return $this->children;
    }

    /**
     * Add pageMetas
     *
     * @param \Bpeh\NestablePageBundle\Model\PageMetaBase $pageMetas
     * @return Page
     */
    public function addPageMeta(\Bpeh\NestablePageBundle\Model\PageMetaBase $pageMetas)
    {
        $this->pageMetas[] = $pageMetas;

        return $this;
    }

    /**
     * Remove pageMetas
     *
     * @param \Bpeh\NestablePageBundle\Model\PageMetaBase $pageMetas
     */
    public function removePageMeta(\Bpeh\NestablePageBundle\Model\PageMetaBase $pageMetas)
    {
        $this->pageMetas->removeElement($pageMetas);
    }

    /**
     * Get pageMetas
     *
     * @return \Doctrine\Common\Collections\Collection
     */
    public function getPageMetas()
    {
        return $this->pageMetas;
    }

    /**
     * convert object to string
     * @return string
     */
    public function __toString()
    {
        return $this->slug;
    }
}

```
Note that we have changed all variables to protected to allow inheritance. The references to PageBase has also been changed.

To make our bundle flexible, we also need to allow user to specify their own child entities, form type and templates to use.

```
# vendor/bpeh/nestable-page-bundle/DependencyInjection/Configuration.php

namespace Bpeh\NestablePageBundle\DependencyInjection;

use Symfony\Component\Config\Definition\Builder\TreeBuilder;
use Symfony\Component\Config\Definition\ConfigurationInterface;

/**
 * This is the class that validates and merges configuration from your app/config files
 *
 * To learn more see {@link http://symfony.com/doc/current/cookbook/bundles/extension.html#cookbook-bundles-extension-config-class}
 */
class Configuration implements ConfigurationInterface
{
    /**
     * {@inheritdoc}
     */
    public function getConfigTreeBuilder()
    {
        $treeBuilder = new TreeBuilder();
        $rootNode = $treeBuilder->root('bpeh_nestable_page');
        // Here you should define the parameters that are allowed to
        // configure your bundle. See the documentation linked above for
        // more information on that topic.
        $rootNode
            ->children()
                ->scalarNode('page_entity')->defaultValue('Bpeh\NestablePageBundle\PageTestBundle\Entity\Page')->end()
                ->scalarNode('pagemeta_entity')->defaultValue('Bpeh\NestablePageBundle\PageTestBundle\Entity\PageMeta')->end()
                ->scalarNode('page_form_type')->defaultValue('Bpeh\NestablePageBundle\PageTestBundle\Form\PageType')->end()
                ->scalarNode('pagemeta_form_type')->defaultValue('Bpeh\NestablePageBundle\PageTestBundle\Form\PageMetaType')->end()
	            ->scalarNode('page_view_list')->defaultValue('BpehNestablePageBundle:Page:list.html.twig')->end()
		        ->scalarNode('page_view_edit')->defaultValue('BpehNestablePageBundle:Page:edit.html.twig')->end()
		        ->scalarNode('page_view_show')->defaultValue('BpehNestablePageBundle:Page:show.html.twig')->end()
		        ->scalarNode('page_view_new')->defaultValue('BpehNestablePageBundle:Page:new.html.twig')->end()
		        ->scalarNode('pagemeta_view_new')->defaultValue('BpehNestablePageBundle:PageMeta:new.html.twig')->end()
		        ->scalarNode('pagemeta_view_edit')->defaultValue('BpehNestablePageBundle:PageMeta:edit.html.twig')->end()
		        ->scalarNode('pagemeta_view_index')->defaultValue('BpehNestablePageBundle:PageMeta:index.html.twig')->end()
		        ->scalarNode('pagemeta_view_show')->defaultValue('BpehNestablePageBundle:PageMeta:show.html.twig')->end()
            ->end()
        ;
        return $treeBuilder;
    }
}

```

and the extension

```
# vendor/bpeh/nestable-page-bundle/DependencyInjection/BpehNestablePageExtension.php

namespace Bpeh\NestablePageBundle\DependencyInjection;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\Config\FileLocator;
use Symfony\Component\HttpKernel\DependencyInjection\Extension;
use Symfony\Component\DependencyInjection\Loader\YamlFileLoader;

/**
 * This is the class that loads and manages your bundle configuration
 *
 * To learn more see {@link http://symfony.com/doc/current/cookbook/bundles/extension.html}
 */
class BpehNestablePageExtension extends Extension
{
    /**
     * {@inheritdoc}
     */
    public function load(array $configs, ContainerBuilder $container)
    {
        $configuration = new Configuration();
        $config = $this->processConfiguration($configuration, $configs);

        $container->setParameter( 'bpeh_nestable_page.page_entity', $config[ 'page_entity' ]);
        $container->setParameter( 'bpeh_nestable_page.pagemeta_entity', $config[ 'pagemeta_entity' ]);
        $container->setParameter( 'bpeh_nestable_page.page_form_type', $config[ 'page_form_type' ]);
        $container->setParameter( 'bpeh_nestable_page.pagemeta_form_type', $config[ 'pagemeta_form_type' ]);
	    $container->setParameter( 'bpeh_nestable_page.page_view_list', $config[ 'page_view_list' ]);
	    $container->setParameter( 'bpeh_nestable_page.page_view_new', $config[ 'page_view_new' ]);
	    $container->setParameter( 'bpeh_nestable_page.page_view_edit', $config[ 'page_view_edit' ]);
	    $container->setParameter( 'bpeh_nestable_page.page_view_show', $config[ 'page_view_show' ]);
	    $container->setParameter( 'bpeh_nestable_page.pagemeta_view_index', $config[ 'pagemeta_view_index' ]);
	    $container->setParameter( 'bpeh_nestable_page.pagemeta_view_edit', $config[ 'pagemeta_view_edit' ]);
	    $container->setParameter( 'bpeh_nestable_page.pagemeta_view_new', $config[ 'pagemeta_view_new' ]);
	    $container->setParameter( 'bpeh_nestable_page.pagemeta_view_show', $config[ 'pagemeta_view_show' ]);
	    $loader = new YamlFileLoader($container, new FileLocator(__DIR__ . '/../Resources/config'));
	    $loader->load('services.yml');
    }
}

```

Now in config.yml, anyone can define the page and pagemeta entities themselves.

We also need to initialise the new config parameters when the controllers are loaded. We will do that via the controller event listener.

```
# vendor/bpeh/nestable-page-bundle/Resources/config/services.yml

services:

  bpeh_nestable_page.init:
    class: Bpeh\NestablePageBundle\EventListener\ControllerListener
    tags:
      - { name: kernel.event_listener, event: kernel.controller, method: onKernelController}

```

and in the controller listener class

```
# vendor/bpeh/nestable-page-bundle/EventListener/ControllerListener.php

namespace Bpeh\NestablePageBundle\EventListener;

use Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException;
use Symfony\Component\HttpKernel\Event\FilterControllerEvent;
use Bpeh\NestablePageBundle\Controller\PageController;
use Bpeh\NestablePageBundle\Controller\PageMetaController;

class ControllerListener
{

    public function onKernelController(FilterControllerEvent $event)
    {
        $controller = $event->getController();

        /*
         * controller must come in an array
         */
        if (!is_array($controller)) {
            return;
        }

        if ($controller[0] instanceof PageController || $controller[0] instanceof PageMetaController) {
            $controller[0]->init();
        }
    }
}

```

The Page Controller can now use the parameters as defined in config.yml to load the entities and form types.

```
# vendor/bpeh/nestable-page-bundle/Controller/PageController.php

namespace Bpeh\NestablePageBundle\Controller;

use Bpeh\NestablePageBundle\Entity\Page;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Method;
use Symfony\Component\HttpFoundation\JsonResponse;

/**
 * Page controller.
 *
 * @Route("/bpeh_page")
 */
class PageController extends Controller
{

    private $entity;

	private $entity_meta;

    private $page_form_type;

	private $page_view_list;

	private $page_view_new;

	private $page_view_edit;

	private $page_view_show;

    public function init()
    {
    	$this->entity = $this->container->getParameter('bpeh_nestable_page.page_entity');
	    $this->entity_meta = $this->container->getParameter('bpeh_nestable_page.pagemeta_entity');
        $this->page_form_type = $this->container->getParameter('bpeh_nestable_page.page_form_type');
	    $this->page_view_list = $this->container->getparameter('bpeh_nestable_page.page_view_list');
	    $this->page_view_new = $this->container->getparameter('bpeh_nestable_page.page_view_new');
	    $this->page_view_edit = $this->container->getparameter('bpeh_nestable_page.page_view_edit');
	    $this->page_view_show = $this->container->getparameter('bpeh_nestable_page.page_view_show');
    }

	/**
	 * Lists all Page entities.
	 *
	 * @Route("/", name="bpeh_page")
	 * @Method("GET")
	 *
	 * @return \Symfony\Component\HttpFoundation\RedirectResponse
	 */
    public function indexAction()
    {
	    return $this->redirect($this->generateUrl('bpeh_page_list'));
    }

	/**
	 * Lists all nested page
	 *
	 * @Route("/list", name="bpeh_page_list")
	 * @Method("GET")
	 *
	 * @return array
	 */
    public function listAction()
    {
    	$em = $this->getDoctrine()->getManager();
        $rootMenuItems = $em->getRepository($this->entity)->findParent();

        return $this->render($this->page_view_list, array(
            'tree' => $rootMenuItems,
        ));
    }

	/**
	 * reorder pages
	 *
	 * @Route("/reorder", name="bpeh_page_reorder")
	 * @Method("POST")
	 *
	 * @param Request $request
	 *
	 * @return JsonResponse
	 */
    public function reorderAction(Request $request)
    {
	    $em = $this->getDoctrine()->getManager();
	    // id of affected element
	    $id = $request->get('id');
	    // parent Id
	    $parentId = ($request->get('parentId') == '') ? null : $request->get('parentId');
	    // new sequence of this element. 0 means first element.
	    $position = $request->get('position');

	    $result = $em->getRepository($this->entity)->reorderElement($id, $parentId, $position);

	    return new JsonResponse(
		    array('message' => $this->get('translator')->trans($result[0], array(), 'BpehNestablePageBundle')
		    , 'success' => $result[1])
	    );

    }

	/**
	 * Creates a new Page entity.
	 *
	 * @Route("/new", name="bpeh_page_new")
	 * @Method({"GET", "POST"})
	 *
	 * @param Request $request
	 *
	 * @return array|\Symfony\Component\HttpFoundation\RedirectResponse
	 */
    public function newAction(Request $request)
    {
	    $page = new $this->entity();
	    $form = $this->createForm($this->page_form_type, $page);
	    $form->handleRequest($request);

	    if ($form->isSubmitted() && $form->isValid()) {
		    $em = $this->getDoctrine()->getManager();
		    $em->persist($page);
		    $em->flush();

		    return $this->redirectToRoute('bpeh_page_show', array('id' => $page->getId()));
	    }

	    return $this->render($this->page_view_new, array(
		    'page' => $page,
		    'form' => $form->createView(),
	    ));
    }

	/**
	 * Finds and displays a Page entity.
	 *
	 * @Route("/{id}", name="bpeh_page_show")
	 * @Method("GET")
	 *
	 * @param Request $request
	 *
	 * @return array
	 */
	public function showAction(Request $request)
	{
		$em = $this->getDoctrine()->getManager();

		$page = $em->getRepository($this->entity)->find($request->get('id'));

		$pageMeta = $em->getRepository($this->entity_meta)->findPageMetaByLocale($page,$request->getLocale());

		$deleteForm = $this->createDeleteForm($page);

		return $this->render($this->page_view_show, array(
			'page' => $page,
			'pageMeta' => $pageMeta,
			'delete_form' => $deleteForm->createView(),
		));

	}

	/**
	 * Displays a form to edit an existing Page entity.
	 *
	 * @Route("/{id}/edit", name="bpeh_page_edit")
	 * @Method({"GET", "POST"})
	 *
	 * @param Request $request
	 *
	 * @return array|\Symfony\Component\HttpFoundation\RedirectResponse
	 */
	public function editAction(Request $request)
	{
		$em = $this->getDoctrine()->getManager();
		$page = $em->getRepository($this->entity)->find($request->get('id'));
		$deleteForm = $this->createDeleteForm($page);
		$editForm = $this->createForm($this->page_form_type, $page);
		$editForm->handleRequest($request);

		if ($editForm->isSubmitted() && $editForm->isValid()) {
			$em = $this->getDoctrine()->getManager();
			$em->persist($page);
			$em->flush();

			return $this->redirectToRoute('bpeh_page_edit', array('id' => $page->getId()));
		}

		return $this->render($this->page_view_edit, array(
			'page' => $page,
			'edit_form' => $editForm->createView(),
			'delete_form' => $deleteForm->createView(),
		));

	}

	/**
	 * Deletes a Page entity.
	 *
	 * @Route("/{id}", name="bpeh_page_delete")
	 * @Method("DELETE")
	 *
	 * @param Request $request
	 *
	 * @return \Symfony\Component\HttpFoundation\RedirectResponse
	 */
	public function deleteAction(Request $request)
	{
		$em = $this->getDoctrine()->getManager();
		$page = $em->getRepository($this->entity)->find($request->get('id'));
		$form = $this->createDeleteForm($page);
		$form->handleRequest($request);

		if ($form->isSubmitted() && $form->isValid()) {
			$em = $this->getDoctrine()->getManager();
			$em->remove($page);
			$em->flush();
		}

		return $this->redirectToRoute('bpeh_page_list');
	}

	/**
	 * Creates a form to delete a Page entity.
	 *
	 * @return \Symfony\Component\Form\Form The form
	 */
	private function createDeleteForm(Page $page)
	{
		return $this->createFormBuilder()
		            ->setAction($this->generateUrl('bpeh_page_delete', array('id' => $page->getId())))
		            ->setMethod('DELETE')
		            ->getForm()
			;
	}

}
```

Likewise for PageMeta Controller

```
# vendor/bpeh/nestable-page-bundle/Controller/PageMetaController.php

namespace Bpeh\NestablePageBundle\Controller;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Method;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Bpeh\NestablePageBundle\Entity\PageMeta;

/**
 * PageMeta controller.
 *
 * @Route("/bpeh_pagemeta")
 */
class PageMetaController extends Controller
{

	private $entity;

	private $entity_meta;

	private $page_meta_form_type;

	private $pagemeta_view_index;

	private $pagemeta_view_new;

	private $pagemeta_view_edit;

	private $pagemeta_view_show;

	public function init()
	{
		$this->entity = $this->container->getParameter('bpeh_nestable_page.page_entity');
		$this->entity_meta = $this->container->getParameter('bpeh_nestable_page.pagemeta_entity');
		$this->page_meta_form_type = $this->container->getParameter('bpeh_nestable_page.pagemeta_form_type');
		$this->pagemeta_view_index = $this->container->getparameter('bpeh_nestable_page.pagemeta_view_index');
		$this->pagemeta_view_new = $this->container->getparameter('bpeh_nestable_page.pagemeta_view_new');
		$this->pagemeta_view_edit = $this->container->getparameter('bpeh_nestable_page.pagemeta_view_edit');
		$this->pagemeta_view_show = $this->container->getparameter('bpeh_nestable_page.pagemeta_view_show');
	}

	/**
	 * Lists all PageMeta entities.
	 *
	 * @Route("/", name="bpeh_pagemeta_index")
	 * @Method("GET")
	 */
	public function indexAction()
	{
		$em = $this->getDoctrine()->getManager();

		$pageMetas = $em->getRepository($this->entity_meta)->findAll();

		return $this->render($this->pagemeta_view_index, array(
			'pageMetas' => $pageMetas,
		));

	}

	/**
	 * Creates a new PageMeta entity.
	 *
	 * @Route("/new", name="bpeh_pagemeta_new")
	 * @Method({"GET", "POST"})
	 */
	public function newAction(Request $request)
	{
		$pageMeta = new $this->entity_meta();
		$form = $this->createForm($this->page_meta_form_type, $pageMeta);
		$form->handleRequest($request);

		if ($form->isSubmitted() && $form->isValid()) {
			$em = $this->getDoctrine()->getManager();

			if ( $em->getRepository( $this->entity_meta )->findPageMetaByLocale( $pageMeta->getPage(), $pageMeta->getLocale() ) ) {
				$this->get('session')->getFlashBag()->add( 'error', $this->get('translator')->trans('one_locale_per_pagemeta_only', array(), 'BpehNestablePageBundle') );
			} else {
				$em->persist( $pageMeta );
				$em->flush();
				return $this->redirectToRoute( 'bpeh_pagemeta_show', array( 'id' => $pageMeta->getId() ) );
			}
		}

		return $this->render($this->pagemeta_view_new, array(
			'pageMeta' => $pageMeta,
			'form' => $form->createView(),
		));

	}

	/**
	 * Finds and displays a PageMeta entity.
	 *
	 * @Route("/{id}", name="bpeh_pagemeta_show")
	 * @Method("GET")
	 */
	public function showAction(Request $request)
	{
		$em = $this->getDoctrine()->getManager();

		$pageMeta = $em->getRepository($this->entity_meta)->find($request->get('id'));

		$deleteForm = $this->createDeleteForm($pageMeta);

		return $this->render($this->pagemeta_view_show, array(
			'pageMeta' => $pageMeta,
			'delete_form' => $deleteForm->createView(),
		));
	}

	/**
	 * Displays a form to edit an existing PageMeta entity.
	 *
	 * @Route("/{id}/edit", name="bpeh_pagemeta_edit")
	 * @Method({"GET", "POST"})
	 */
	public function editAction(Request $request)
	{
		$em = $this->getDoctrine()->getManager();
		$pageMeta = $em->getRepository($this->entity_meta)->find($request->get('id'));
		$origId = $pageMeta->getPage()->getId();
		$origLocale = $pageMeta->getLocale();

		$deleteForm = $this->createDeleteForm($pageMeta);
		$editForm = $this->createForm($this->page_meta_form_type, $pageMeta);
		$editForm->handleRequest($request);

		if ($editForm->isSubmitted() && $editForm->isValid()) {

			$error = false;

			// if page and local is the same, dont need to check locale count
			if ($origLocale == $pageMeta->getLocale() && $origId == $pageMeta->getPage()->getId()) {
				// all good
			}
			elseif ( $em->getRepository( $this->entity_meta )->findPageMetaByLocale( $pageMeta->getPage(), $pageMeta->getLocale(), true ) ) {
				$this->get('session')->getFlashBag()->add( 'error', $this->get('translator')->trans('one_locale_per_pagemeta_only', array(), 'BpehNestablePageBundle') );
				$error = true;
			}

			// if everything is successful
			if (!$error) {
				$em->persist( $pageMeta );
				$em->flush();
				return $this->redirectToRoute( 'bpeh_pagemeta_edit', array( 'id' => $pageMeta->getId() ) );
			}
		}

		return $this->render($this->pagemeta_view_edit, array(
			'pageMeta' => $pageMeta,
			'edit_form' => $editForm->createView(),
			'delete_form' => $deleteForm->createView(),
		));
	}

	/**
	 * Deletes a PageMeta entity.
	 *
	 * @Route("/{id}", name="bpeh_pagemeta_delete")
	 * @Method("DELETE")
	 */
	public function deleteAction(Request $request)
	{
		$em = $this->getDoctrine()->getManager();
		$pageMeta = $em->getRepository($this->entity_meta)->find($request->get('id'));
		$form = $this->createDeleteForm($pageMeta);
		$form->handleRequest($request);

		if ($form->isSubmitted() && $form->isValid()) {
			$em = $this->getDoctrine()->getManager();
			$em->remove($pageMeta);
			$em->flush();
		}

		return $this->redirectToRoute('bpeh_pagemeta_index');
	}

	/**
	 * Creates a form to delete a PageMeta entity.
	 *
	 * @param PageMeta $pageMetum The PageMeta entity
	 *
	 * @return \Symfony\Component\Form\Form The form
	 */
	private function createDeleteForm(PageMeta $pageMeta)
	{
		return $this->createFormBuilder()
		            ->setAction($this->generateUrl('bpeh_pagemeta_delete', array('id' => $pageMeta->getId())))
		            ->setMethod('DELETE')
		            ->getForm()
			;
	}
}
```

We also need to refactor PageMetaRepository because findPageMetaByLocale can now return either an object or scalar value.

```
# vendor/bpeh/nestable-page-bundle/Entity/PageMetaRepository.php

namespace Bpeh\NestablePageBundle\Entity;

use Bpeh\NestablePageBundle\Model\PageBase;

/**
 * PageMetaRepository
 *
 * This class was generated by the Doctrine ORM. Add your own custom
 * repository methods below.
 */
class PageMetaRepository extends \Doctrine\ORM\EntityRepository {

	/**
	 * find page locale and return object or counter
	 *
	 * @param \Bpeh\NestablePageBundle\Entity\Page $page
	 * @param $locale
	 * @param bool $count
	 *
	 * @return mixed
	 */
	public function findPageMetaByLocale( PageBase $page, $locale, $count = false ) {

		$qb = $this->createQueryBuilder( 'pm' );

		if ( $count ) {
			$qb->select( 'count(pm.id)' );
		}

		$query = $qb->where( 'pm.locale = :locale' )
	      ->andWhere( 'pm.page = :page' )
	      ->setParameter( 'locale', $locale )
	      ->setParameter( 'page', $page )
	      ->getQuery();

		if ( $count ) {
			return $query->getSingleScalarResult();
		}

		return $query->getOneOrNullResult();

	}
}
```

There are other stuff to be done

* Create the translations.
* Move all the related views from app/resources/views to vendor/bpeh/nestable-page-bundle/views
* Update functional tests.

Once you are happy with it, give it a new tag and commit your changes again.

The bundle is now ready to be extended.


## Extending BpehNestablePageBundle

If you are already lost, I've created a [demo bundle](https://github.com/bernardpeh/NestablePageBundle) and you can install the demo bundle and test out it for yourself.

Let us extend BpehNestablePageBundle by copying that bundle.

```
-> cd src/AppBundle
-> cp ../../vendor/bpeh/nestable-page-bundle/PageTestBundle/PageTestBundle.php .
# let us name it page bundle
-> mv PageTestBundle.php Page.php
-> cp ../../vendor/bpeh/nestable-page-bundle/PageTestBundle/Controller/*.php Controller/
-> cp ../../vendor/bpeh/nestable-page-bundle/PageTestBundle/Entity/*.php Entity/
# let us move the pagerepository.php to the right dir
-> mv Entity/PageRepository.php Repository/
-> mv Entity/PageMetaRepository.php Repository/
-> cp -a ../../vendor/bpeh/nestable-page-bundle/PageTestBundle/Form .
-> cp ../../vendor/bpeh/nestable-page-bundle/PageTestBundle/DataFixtures/ORM/LoadPageData.php DataFixtures/ORM/
```

Let us call this bundle PageBundle to keep it simple.

```
# src/AppBundle/Page.php

namespace AppBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class Page extends Bundle
{
	// use a child bundle
	public function getParent()
	{
		return 'BpehNestablePageBundle';
	}
}
```

Let us configure the Entities

```
# src/AppBundle/Entity/Page.php

namespace AppBundle\Entity;

use Bpeh\NestablePageBundle\Model\PageBase;
use Doctrine\ORM\Mapping as ORM;

/**
 * PageMeta
 *
 * @ORM\Table(name="page")
 * @ORM\Entity(repositoryClass="AppBundle\Repository\PageRepository")
 * @ORM\HasLifecycleCallbacks()
 */
class Page extends PageBase
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
     * Get id
     *
     * @return integer
     */
    public function getId()
    {
        return $this->id;
    }

}
```

and PageMeta.php

```
# src/AppBundle/Entity/PageMeta.php

namespace AppBundle\Entity;

use Bpeh\NestablePageBundle\Model\PageMetaBase;
use Doctrine\ORM\Mapping as ORM;

/**
 * PageMeta
 *
 * @ORM\Table(name="pagemeta")
 * @ORM\Entity(repositoryClass="AppBundle\Repository\PageMetaRepository")
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
     * Get id
     *
     * @return integer
     */
    public function getId()
    {
        return $this->id;
    }

}
```

Let us update the PageRepository.php

```
# src/AppBundle/Repository/PageRepository.php

namespace AppBundle\Repository;

use Bpeh\NestablePageBundle\Entity\PageRepository as BasePageRepository;

/**
 * PageRepository
 *
 */
class PageRepository extends BasePageRepository
{

}
```

and PageMetaRepository.php

```
# src/AppBundle/Repository/PageRepository.php

namespace AppBundle\Repository;

use Bpeh\NestablePageBundle\Entity\PageMetaRepository as BasePageMetaRepository;

/**
 * PageRepository
 *
 */
class PageMetaRepository extends BasePageMetaRepository
{

}
```

Let us update the PageController to have a route which is easier to use

```
# src/AppBundle/Controller/PageController.php

namespace AppBundle\Controller;

use Bpeh\NestablePageBundle\Controller\PageController as BaseController;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

/**
 * Page controller.
 * @Route("/page")
 */
class PageController extends BaseController
{

}

```

Now PageMetaController.php

```
# src/AppBundle/Controller/PageMetaController.php
namespace AppBundle\Controller;

use Bpeh\NestablePageBundle\Controller\PageMetaController as BaseController;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

/**
 * PageMeta controller.
 * @Route("/pagemeta")
 */
class PageMetaController extends BaseController
{

}

```

Its time to update the basic forms

```
# src/AppBundle/Form/PageType.php

namespace AppBundle\Form;

use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Bpeh\NestablePageBundle\Form\PageType as BasePageType;

class PageType extends BasePageType
{

	/**
	 * @param FormBuilderInterface $builder
	 * @param array $options
	 */
	public function buildForm(FormBuilderInterface $builder, array $options)
	{
		parent::buildForm($builder,$options);
	}

	/**
	 * @param OptionsResolver $resolver
	 */
	public function configureOptions(OptionsResolver $resolver)
	{
		$resolver->setDefaults(array(
			'data_class' => 'AppBundle\Entity\Page',
		));
	}
}
```

and PageMetaType.php

```
# src/AppBundle/Form/PageMetaType.php


namespace AppBundle\Form;

use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\FormBuilderInterface;
use Bpeh\NestablePageBundle\Form\PageMetaType as BasePageMetaType;

class PageMetaType extends BasePageMetaType
{

	/**
	 * @param FormBuilderInterface $builder
	 * @param array $options
	 */
	public function buildForm(FormBuilderInterface $builder, array $options)
	{
		parent::buildForm($builder,$options);
	}

	/**
	 * @param OptionsResolver $resolver
	 */
	public function configureOptions(OptionsResolver $resolver)
	{
		$resolver->setDefaults(array(
			'data_class' => 'AppBundle\Entity\PageMeta'
		));
	}
}
```

Let us confirm the new routes are working...

```
-> app/console debug:router | grep page
     bpeh_page                        GET        ANY      ANY    /page/
     bpeh_page_list                   GET        ANY      ANY    /page/list
     bpeh_page_reorder                POST       ANY      ANY    /page/reorder
     bpeh_page_new                    GET|POST   ANY      ANY    /page/new
     bpeh_page_show                   GET        ANY      ANY    /page/{id}
     bpeh_page_edit                   GET|POST   ANY      ANY    /page/{id}/edit
     bpeh_page_delete                 DELETE     ANY      ANY    /page/{id}
     bpeh_pagemeta_index              GET        ANY      ANY    /pagemeta/
     bpeh_pagemeta_new                GET|POST   ANY      ANY    /pagemeta/new
     bpeh_pagemeta_show               GET        ANY      ANY    /pagemeta/{id}
     bpeh_pagemeta_edit               GET|POST   ANY      ANY    /pagemeta/{id}/edit
     bpeh_pagemeta_delete             DELETE     ANY      ANY    /pagemeta/{id}
```

Looks good. Its time to update config.yml

```
...
    orm:
        auto_generate_proxy_classes: "%kernel.debug%"
        naming_strategy: doctrine.orm.naming_strategy.underscore
        auto_mapping: true
        resolve_target_entities:
            Bpeh\NestablePageBundle\Model\PageBase: AppBundle\Entity\Page
            Bpeh\NestablePageBundle\Model\PageMetaBase: AppBundle\Entity\PageMeta
...
# Nestable Page Configuration
bpeh_nestable_page:
    page_entity: AppBundle\Entity\Page
    pagemeta_entity: AppBundle\Entity\PageMeta
    page_form_type: AppBundle\Form\PageType
    pagemeta_form_type: AppBundle\Form\PageMetaType
    # Customise the template if you want.
    # page_view_list: YourBundle:list.html.twig
    # page_view_new: YourBundle:new.html.twig
    # page_view_edit: YourBundle:edit.html.twig
    # page_view_show: YourBundle:show.html.twig
    # pagemeta_view_index: YourBundle:index.html.twig
    # pagemeta_view_new: YourBundle:new.html.twig
    # pagemeta_view_edit: YourBundle:edit.html.twig
    # pagemeta_view_show: YourBundle:show.html.twig
```

Finally, init the new bundle in AppKernel.php

```
# app/AppKernel.php

...
    new Bpeh\NestablePageBundle\BpehNestablePageBundle(),
    new AppBundle\Page()
...
```

If everything is working, the new url should be working.

```
songbird.app/app_dev.php/page
```


## Summary

In this chapter, we have created a new repo for the NestablePageBundle. We have updated composer to pull the bundle from the repo and auto-loaded it according to the PSR-4 standard. We learned the hard way of creating a non-extensible bundle with the wrong namespace and then mass renaming it again. Making the entities extensible was a massive job and required a lot of refactoring in our code.

We have done so much to make NestablePageBundle as decoupled as possible. Still, there was lots of room for improvement. Was it worth the effort? Definitely!

## Exercises

* Delete the whole vendor directory and try doing a composer update. Did anything break?
* Update the functional test.

## References

* [Define relationship between abstract classes and interfaces](http://symfony.com/doc/current/doctrine/resolve_target_entity.html)
* [Short guide to licenses](http://www.smashingmagazine.com/2010/03/a-short-guide-to-open-source-and-similar-licenses)
* [Software licenses at a glance](http://tldrlegal.com)
* [Composer Schema](https://getcomposer.org/doc/04-schema.md)
* [Composer versioning](https://getcomposer.org/doc/articles/versions.md)