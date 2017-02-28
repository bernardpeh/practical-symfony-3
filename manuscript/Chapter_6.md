# Chapter 6: The User Management System Part 1

User Management System is the core part of any CMS. We will create this feature using the popular [FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle).

## Pre-setup

Make sure we are in the right branch. Let us branch off from the previous chapter.

```
-> git checkout -b my_chapter6
```

## Installing the FOSUserBundle

Add the bundle in composer.json

```
# in symfony
-> docker-compose exec php composer require friendsofsymfony/user-bundle ~2.0@dev
```

Now in AppKernel, we need to register the bundles

```
# app/AppKernel.php
...
public function registerBundles()
{
    $bundles = array(
        ...
        new AppBundle\AppBundle(),
        // init my fosuser
        new FOS\UserBundle\FOSUserBundle(),
        new AppBundle\User()
    );
}
```

AppBundle\User() will look for the User class in User.php (under the AppBundle namespace). We want the User class to inherit all properties of FOSUserBundle. Let us create User.php

```
# src/AppBundle/User.php

namespace AppBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class User extends Bundle
{
    // use a child bundle
    public function getParent()
    {
	return 'FOSUserBundle';
    }
}
```

Next we need to configure FOSUserBundle. Don't worry if certain directives don't make sense. It will as you progress further. Note that yaml files cannot contain tabs.

```
# app/config/config.yml
...
# turn on translator
translator:      { fallbacks: ["%locale%"] }
...

framework:
    ...
    session:
        # http://symfony.com/doc/current/reference/configuration/framework.html#handler-id
        # we have to use the system session storage because the default doesn't work with vagrant.
        handler_id:  ~
        # save_path:   "%kernel.root_dir%/../var/sessions/%kernel.environment%"
        
# fosuser config
fos_user:
    db_driver: orm
    firewall_name: main
    user_class: AppBundle\Entity\User
    from_email:
        address: admin@songbird.app
        sender_name: Songbird
```

and setup the security and firewall, your file should look like this

```
# app/config/security.yml
security:
  encoders:
          FOS\UserBundle\Model\UserInterface: bcrypt

  # http://symfony.com/doc/current/book/security.html#where-do-users-come-from-user-providers
  providers:
      fos_userbundle:
          id: fos_user.user_provider.username

  role_hierarchy:
          ROLE_ADMIN:       ROLE_USER
          ROLE_SUPER_ADMIN: ROLE_ADMIN

  firewalls:
      # disables authentication for assets and the profiler, adapt it according to your needs
      dev:
          pattern: ^/(_(profiler|wdt)|css|images|js)/
          security: false

      main:
          pattern: ^/
          form_login:
              provider: fos_userbundle
              csrf_token_generator: security.csrf.token_manager
          logout:       true
          anonymous:    true

  access_control:
          - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
          - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
          - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
          - { path: ^/admin/, role: ROLE_ADMIN }
```

## DB credentials

The db credentials are in app/config/parameters.yml. They are usually variables based on your environment. Since we are using docker, we can hard code them.

Have a look at the file if you are interested.

```
# symfony/app/config/parameters.yml

# your db host is the container in your docker environment
# run "docker network inspect songbird_mynet" to see the ip of the mysql instance.

parameters:
    database_host: 172.25.0.2
    database_port: 3306
    database_name: songbird
    database_user: root
    database_password: root
    mailer_transport: smtp
    mailer_host: 172.25.0.6:1025
    mailer_user: null
    mailer_password: null
    secret: ThisTokenIsNotSoSecretChangeIt

```

## Creating the User Entity

Have a quick read if you are unfamiliar with [doctrine and entity](http://symfony.com/doc/current/book/doctrine.html). We will be using doctrine very often in this book.

Symfony allows us to automate lots of things using command line, including the creation of entities. We will create the user entity with 2 custom fields called firstname and lastname.

```
-> docker-compose exec php bin/console generate:doctrine:entity

# You will be prompted a series of questions.

The Entity shortcut name: AppBundle:User

Configuration format (yml, xml, php, or annotation) [annotation]: annotation

New field name (press <return> to stop adding fields): firstname
Field type [string]:
Field length [255]:
Is nullable [false]: true
Unique [false]:

New field name (press <return> to stop adding fields): lastname
Field type [string]:
Field length [255]:
Is nullable [false]: true
Unique [false]:
```

You realised we have to use "docker-compose exec php" to run commands in the php container. Its ugly and we will automate that in a minute. Once you are familiar with the command line, you should be able to generate the entity and other files without prompts. We will be doing that in the future chapters.

[FOSUserBundle Groups](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/groups.md) are useful when you want to group users together. For the sake of simplicity, we won't be using this feature. However, you should be able to add this feature in easily once you are comfortable with the Symfony workflow.

Now, the entity class is generated under src/AppBundle/Entity folder. We need to extend the fosuserbundle and make the id protected because of inheritance. If you open up the file, you will see that the code has been created for you already but we still need to make some changes in order for the entity inheritance to work. Refer to comments in the code.

```
namespace AppBundle\Entity;

# add baseuser
use FOS\UserBundle\Model\User as BaseUser;
use Doctrine\ORM\Mapping as ORM;

/**
 * User extending fos user
 *
 * @ORM\Table(name="user")
 * @ORM\Entity(repositoryClass="AppBundle\Repository\UserRepository")
 */
class User extends BaseUser
{
    /**
     * Needs to be protected because of inheritance
     *
     * @var int
     *
     * @ORM\Column(name="id", type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    protected $id;

    /**
     * @var string
     *
     * @ORM\Column(name="firstname", type="string", length=255, nullable=true)
     */
    private $firstname;

    /**
     * @var string
     *
     * @ORM\Column(name="lastname", type="string", length=255, nullable=true)
     */
    private $lastname;

    /**
     * User constructor.
     */
    public function __construct()
    {
        parent::__construct();
    }

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
     * Set firstname
     *
     * @param string $firstname
     *
     * @return User
     */
    public function setFirstname($firstname)
    {
        $this->firstname = $firstname;

        return $this;
    }

    /**
     * Get firstname
     *
     * @return string
     */
    public function getFirstname()
    {
        return $this->firstname;
    }

    /**
     * Set lastname
     *
     * @param string $lastname
     *
     * @return User
     */
    public function setLastname($lastname)
    {
        $this->lastname = $lastname;

        return $this;
    }

    /**
     * Get lastname
     *
     * @return string
     */
    public function getLastname()
    {
        return $this->lastname;
    }
}
```

You will noticed all the getters and setters have already been generated for you as well. Cool!

Now, we need to configure the routes. The default routes provided by FOSUser is a good start.

```
# app/config/routing.yml
...
# FOS user bundle default routing
fos_user_security:
    resource: "@FOSUserBundle/Resources/config/routing/security.xml"

fos_user_profile:
    resource: "@FOSUserBundle/Resources/config/routing/profile.xml"
    prefix: /profile

fos_user_resetting:
    resource: "@FOSUserBundle/Resources/config/routing/resetting.xml"
    prefix: /resetting

fos_user_change_password:
    resource: "@FOSUserBundle/Resources/config/routing/change_password.xml"
    prefix: /profile
```

To check that the new routes have been installed correctly,

```
# in symfony
-> bin/console debug:router | grep fos

 fos_user_security_login           GET|POST ANY    ANY  /login
 fos_user_security_check           POST     ANY    ANY  /login_check
 fos_user_security_logout          GET      ANY    ANY  /logout
 fos_user_profile_show             GET      ANY    ANY  /profile/
 fos_user_profile_edit             GET|POST ANY    ANY  /profile/edit
 fos_user_resetting_request        GET      ANY    ANY  /resetting/request
 fos_user_resetting_send_email     POST     ANY    ANY  /resetting/send-email
 fos_user_resetting_check_email    GET      ANY    ANY  /resetting/check-email
 fos_user_resetting_reset          GET|POST ANY    ANY  /resetting/reset/{token}
 fos_user_change_password          GET|POST ANY    ANY  /profile/change-password

```

or we can use the router:match command to match the exact url and get more details

```
# in symfony
-> bin/console router:match /profile/
Route "fos_user_profile_show" matches

[router] Route "fos_user_profile_show"
Name         fos_user_profile_show
Path         /profile/
Path Regex   #^/profile/$#s
Host         ANY
Host Regex
Scheme       ANY
Method       GET
Class        Symfony\Component\Routing\Route
Defaults     _controller: FOSUserBundle:Profile:show
Requirements NO CUSTOM
Options      compiler_class: Symfony\Component\Routing\RouteCompiler
```

See how much work has done for you by inheriting the FOSUserBundle... This step allows you to use many default FOSUserBundle functionalities like password reset and user profile update without writing a single line of code! Now, let us test one of the routes by going to

```
http://songbird.app:8000/app_dev.php/login
```

![](images/default_login.png)

You should see a simple login page.

To verify that the schema is correct, let us generate it:

```
# in symfony
-> docker-compose exec php bin/console doctrine:schema:create

Creating database schema...
Database schema created successfully!
```

Let us check that the schema has indeed been created correctly.

```
-> docker-compose exec db mysql -uroot -proot songbird -e "describe user"
+-----------------------+--------------+------+-----+---------+----------------+
| Field                 | Type         | Null | Key | Default | Extra          |
+-----------------------+--------------+------+-----+---------+----------------+
| id                    | int(11)      | NO   | PRI | NULL    | auto_increment |
| username              | varchar(180) | NO   |     | NULL    |                |
| username_canonical    | varchar(180) | NO   | UNI | NULL    |                |
| email                 | varchar(180) | NO   |     | NULL    |                |
| email_canonical       | varchar(180) | NO   | UNI | NULL    |                |
| enabled               | tinyint(1)   | NO   |     | NULL    |                |
| salt                  | varchar(255) | YES  |     | NULL    |                |
| password              | varchar(255) | NO   |     | NULL    |                |
| last_login            | datetime     | YES  |     | NULL    |                |
| confirmation_token    | varchar(180) | YES  | UNI | NULL    |                |
| password_requested_at | datetime     | YES  |     | NULL    |                |
| roles                 | longtext     | NO   |     | NULL    |                |
| firstname             | varchar(255) | YES  |     | NULL    |                |
| lastname              | varchar(255) | YES  |     | NULL    |                |
+-----------------------+--------------+------+-----+---------+----------------+
```

Looks like we got the right fields. Let us now create a console wrapper to make our life easier.

## Wrapper Scripts

We now need a very simple wrapper to run the console commands. Let us create a console wrapper.

```
# in symfony/scripts/console

#!/bin/bash 
docker-compose exec php bin/console $@
```

once the console script is created, it needs to be executable.

```
# in symfony
chmod u+x scripts/console
```

Let us try some commands

```
# you should not see an error
./scripts/console debug:router
```

Let us do the same for the composer command

```
# in symfony/scripts/composer

#!/bin/bash 
docker-compose exec php composer "$@"
```

and 

```
# in symfony
chmod u+x scripts/composer
```

Finally, we will create another for the mysql command

```
# in symfony/scripts/mysql

#!/bin/bash 

MYSQL_DATABASE=`grep MYSQL_DATABASE .env | cut -d= -f 2`
MYSQL_ROOT_PASSWORD=`grep MYSQL_ROOT_PASSWORD .env | cut -d= -f 2`

docker-compose exec db mysql -uroot -p$MYSQL_ROOT_PASSWORD $MYSQL_DATABASE -e "$@"
```

now we allow executable bit to this script.

```
# in symfony
chmod u+x scripts/mysql
```

We can now use some wrapper scripts to access the php container easily. We are gearing up. Ready for more?

## Summary

In this chapter, we have installed FOSUserBundle and extended it in AppBundle. We have verified that the installation was correct by looking at the default login page and database schema. We also created some helper scripts to help accessing the docker instance a bit easier.

Remember to commit all your changes before moving on.

## Exercises (Optional)

* Try installing the UserBundle outside of Appbundle. Are there any pros and cons of doing that as compared to putting all the bundles in AppBundle?

## References

* [FOSUserBundle Doc](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/index.md)

* [FOSUserBundle Installation](https://symfony.com/doc/master/bundles/FOSUserBundle/index.html)

* [Routing](http://symfony.com/doc/current/book/routing.html)