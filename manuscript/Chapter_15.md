# Chapter 15: Logging User Activities

A proper CMS needs a logging mechanism. We are talking about the admin area, not the front end. If something happens, we need to know what was done and what happened? We can log user activities in a file system but it is not very efficient. File system is good for logging errors - see [monolog](http://symfony.com/doc/current/cookbook/logging/monolog.html). Ideally, we need a database solution.

## Define User Stories

After the user logs in, we want to record the username, current_url, previous_url, CRUD action, data on every page that the user visits. These data should be recorded in a new table. When the user is deleted, we do not want the logs associated with the user to be deleted, therefore the 2 tables are not related.

There is a popular [loggable doctrine extension](https://github.com/Atlantic18/DoctrineExtensions/blob/master/doc/loggable.md) that we can use. However, it is easy enough to built one for ourselves.

**15. Logging User Activitiy**

|**Story Id**|**As a**|**I**|**So that I**|
|15.1|admin user|want to manage user logs|check on user activity anytime.|
|15.2|test1 user|don't want to manage user logs|don't breach security|

**Story ID 15.1: As an admin, I want to manage user logs, so that I can check on user activity anytime.**

|**Scenario Id**|**Given**|**When**|**Then**|
|15.1.1|List user log|I click on user log on the left menu|I should see more than 1 row in the table|
|15.1.2|Show user log 1|I go to the first log entry|I should see the text "/admin/dashboard"|

**Story ID 15.2: As test1 user, I don't want to manage user logs, so that I don't breach security.**

|**Scenario Id**|**Given**|**When**|**Then**|
|15.2.1|List user log|I go to the user log url|I should get an access denied message|
|15.2.2|Show log 1|I go to the show log id 1 url|I should get an access denied message|
|15.2.3|Edit log 1|I go to the edit log id 1 url|I should get an access denied message|

## Implementation

We will create a new entity called UserLog. The UserLog entity should have the following fields: id, username, current_url, referrer, action, data, created.

```
-> ./scripts/console doctrine:generate:entity --entity=AppBundle:UserLog --format=annotation --fields="username:string(255) current_url:text referrer:text(nullable=true) action:string(255) data:text(nullable=true) created:datetime" --no-interaction
```

Again, don't memorise this command. You can find out more about this command using

```
-> ./scripts/console doctrine:generate:entity --help
```

or from the [online documentation](http://symfony.com/doc/current/bundles/SensioGeneratorBundle/commands/generate_doctrine_entity.html)

In the entity, note that we are populating the username field from the user entity but not creating a constraint between the 2 entities. The reason for that is that when we delete the user, we still want to keep the user entries. We haven't really gone through doctrine yet. You can read more about association mapping [here](http://doctrine-orm.readthedocs.org/projects/doctrine-orm/en/latest/reference/association-mapping.html) if we want them to be related. We will touch on doctrine again in the later chapters.

```
# src/AppBundle/Entity/UserLog.php

<?php

/**
 * UserLog
 *
 * @ORM\Table(name="user_log")
 * @ORM\Entity(repositoryClass="AppBundle\Repository\UserLogRepository")
 */
class UserLog
{
    /**
     * @var int
     *
     * @ORM\Column(name="id", type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     * @var string
     *
     * @ORM\Column(name="username", type="string", length=255)
     */
    private $username;

    /**
     * @var string
     *
     * @ORM\Column(name="current_url", type="text")
     */
    private $current_url;

    /**
     * @var string
     *
     * @ORM\Column(name="referrer", type="text", nullable=true)
     */
    private $referrer;

    /**
     * @var string
     *
     * @ORM\Column(name="action", type="string", length=255)
     */
    private $action;

    /**
     * @var string
     *
     * @ORM\Column(name="data", type="text", nullable=true)
     */
    private $data;

    /**
     * @var \DateTime
     *
     * @ORM\Column(name="created", type="datetime")
     */
    private $created;


    /**
     * Get id
     *
     * @return int
     */
    public function getId()
    {
        return $this->id;
    }

    /**
     * Set username
     *
     * @param string $username
     *
     * @return UserLog
     */
    public function setUsername($username)
    {
        $this->username = $username;

        return $this;
    }

    /**
     * Get username
     *
     * @return string
     */
    public function getUsername()
    {
        return $this->username;
    }

    /**
     * Set currentUrl
     *
     * @param string $currentUrl
     *
     * @return UserLog
     */
    public function setCurrentUrl($currentUrl)
    {
        $this->current_url = $currentUrl;

        return $this;
    }

    /**
     * Get currentUrl
     *
     * @return string
     */
    public function getCurrentUrl()
    {
        return $this->current_url;
    }

    /**
     * Set referrer
     *
     * @param string $referrer
     *
     * @return UserLog
     */
    public function setReferrer($referrer)
    {
        $this->referrer = $referrer;

        return $this;
    }

    /**
     * Get referrer
     *
     * @return string
     */
    public function getReferrer()
    {
        return $this->referrer;
    }

    /**
     * Set action
     *
     * @param string $action
     *
     * @return UserLog
     */
    public function setAction($action)
    {
        $this->action = $action;

        return $this;
    }

    /**
     * Get action
     *
     * @return string
     */
    public function getAction()
    {
        return $this->action;
    }

    /**
     * Set data
     *
     * @param string $data
     *
     * @return UserLog
     */
    public function setData($data)
    {
        $this->data = $data;

        return $this;
    }

    /**
     * Get data
     *
     * @return string
     */
    public function getData()
    {
        return $this->data;
    }

    /**
     * Set created
     *
     * @param \DateTime $created
     *
     * @return UserLog
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
}
```

Next, we will intercept the kernel.request event.

```
# src/AppBundle/EventListener/AppSubscriber.php

...
use Symfony\Component\HttpKernel\Event\GetResponseEvent;
use Symfony\Component\HttpKernel\KernelEvents;
...
    public static function getSubscribedEvents()
    {
        // return the subscribed events, their methods and priorities
        return array(
            EasyAdminEvents::PRE_LIST => 'checkUserRights',
            EasyAdminEvents::PRE_EDIT => 'checkUserRights',
            EasyAdminEvents::PRE_SHOW => 'checkUserRights',
            FOSUserEvents::RESETTING_RESET_SUCCESS => 'redirectUserAfterPasswordReset',
            KernelEvents::REQUEST => 'onKernelRequest'
        );
    }
    ...

    /**
     * We will log request to db on every url change
     * 
     * @param GetResponseEvent $event
     */
    public function onKernelRequest(GetResponseEvent $event)
    {
        $request = $event->getRequest();
        $current_url = $request->server->get('REQUEST_URI');
        // ensures we track admin only.
        $admin_path = $this->container->getParameter('admin_path');

        // only log admin area and only if user is logged in. Dont log search by filter
        if (!is_null($this->container->get('security.token_storage')->getToken()) && preg_match('/\/'.$admin_path.'\//', $current_url)
            && ($request->query->get('filter') === null) && !preg_match('/\/userlog\//', $current_url)) {

            $em = $this->container->get('doctrine.orm.entity_manager');
            $log = new UserLog();
            $log->setData(json_encode($request->request->all()));
            $log->setUsername($this->container->get('security.token_storage')->getToken()->getUser()
                ->getUsername());
            $log->setCurrentUrl($current_url);
            $log->setReferrer($request->server->get('HTTP_REFERER'));
            $log->setAction($request->getMethod());
            $log->setCreated(new \DateTime('now'));
            $em->persist($log);
            $em->flush();
        }
    }
        ...
```

Let us create the new menu.

```
# app/config/easyadmin/userlog.yml

easy_admin:
    entities:
        UserLog:
            class: AppBundle\Entity\UserLog
            label: admin.link.user_log
            show:
                  actions: ['list', '-edit', '-delete']
            list:
                actions: ['show', '-edit', '-delete']

```
and the translation.

```
# src/AppBundle/Resources/translations/app.en.xlf
...
        <trans-unit id="6">
            <source>admin.link.user_log</source>
            <target>User Log</target>
        </trans-unit>
...
```

and the french version

```
# src/AppBundle/Resources/translations/app.fr.xlf
...
        <trans-unit id="6">
            <source>admin.link.user_log</source>
            <target>Connexion utilisateur</target>
        </trans-unit>
...
```

Now reset the db, re-login again, click on the user log menu and you will see the new menu on the left.

There were db changes, let us chapter the change.

```
./scripts/console doctrine:migrations:diff
```

and we can reset the db now.

```
-> ./scripts/resetapp
```

## Update BDD (Optional)

Let us create the cest files.

```
-> vendor/bin/codecept generate:cest acceptance As_An_Admin/IWantToManageUserLog -c src/AppBundle
-> vendor/bin/codecept generate:cest acceptance As_Test1_User/IDontWantToManageUserLog -c src/AppBundle
```

*Tip: The assert module is very useful.*

Let us add the assert module

```
# src/AppBundle/Tests/acceptance.suite.yml
...
        - Asserts:
        ...
```

Let us rebuild the libraries

```
-> vendor/bin/codecept build -c src/AppBundle/
```

Again, I will leave you to write the bdd tests. The more detail your scenario is, the better the test coverage will be. Get all the test to pass and remember to commit everything before moving on to the next chapter.

## Summary

In this chapter, we created a new entity called UserLog and used the kernel request event to inject the required request data into the database.

## Exercises

* Modify the UserLog entity such that deleting the user in the User entity will delete the associated user entries in the UserLog entity. (optional)

* What are the pros and cons of allowing CRUD actions on log entries?

* Can you use doctrine loggable extension to achieve what was achieved here? (optional)

* Can you implement automated entity logging using [Traits](http://php.net/manual/en/language.oop5.traits.php)?

## References

* [Symfony Events](http://symfony.com/doc/current/reference/events.html)

* [Doctrine Extensions](http://symfony.com/doc/current/cookbook/doctrine/common_extensions.html)

* [Doctrine Association Mapping](http://doctrine-orm.readthedocs.org/projects/doctrine-orm/en/latest/reference/association-mapping.html)