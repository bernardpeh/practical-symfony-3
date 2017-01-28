# Chapter 8: Doctrine Fixtures and Migrations

As of now, we could create and manage users via the command line (scripts/console fos:user:xxx) or using the basic CRUD UI that we have created. What if we messed up the data or if we want to reset the data with certain values confidently? How can we do that efficiently? We need an automation mechanism to create consistent schema and dummy data.

## Install DoctrineFixturesBundle

Install via composer

```
-> scripts/composer require doctrine/doctrine-fixtures-bundle ^2.3 --dev
```

Now that the data-fixtures-bundle is installed, we can update the kernel.

```
# app/AppKernel.php

...
if (in_array($this->getEnvironment(), array('dev', 'test'))) {
    ...
    $bundles[] = new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle();
    ...
}
...
```

We register the bundle under the array('dev', 'test') environment because we don't need this bundle in the production environment.

To prove that the install is successful, we should have a new entry in the console

```
-> scipts/console | grep fixtures
  doctrine:fixtures:load               Load data fixtures to your database.
```

## Create User Fixtures

Let's create the the data fixtures directory structure

```
-> mkdir -p src/AppBundle/DataFixtures/ORM
```

Now create the class. We are going to create 3 users. One super admin, 3 test users, ie test1, test2 and test3.

The username and password for the 3 users are as follows:

```
# in this format, username:password
admin:admin
test1:test1
test2:test2
test3:test3
```

**Remember these 3 users credentials** as we will be using them a lot throughout the whole book.

Now the actual fixtures class:

```
# src/AppBundle/DataFixtures/ORM/LoadUserData.php

namespace AppBundle\DataFixtures\ORM;

use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
use Doctrine\Common\Persistence\ObjectManager;
use Symfony\Component\DependencyInjection\ContainerAwareInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;

class LoadUserData extends AbstractFixture implements OrderedFixtureInterface, ContainerAwareInterface
{

    /**
     * @var ContainerInterface
     */
    private $container;

    /**
     * {@inheritDoc}
     */
    public function setContainer(ContainerInterface $container = null)
    {
        $this->container = $container;
    }

    /**
     * {@inheritDoc}
     */
    public function load(ObjectManager $manager)
    {
        $userManager = $this->container->get('fos_user.user_manager');

        // add admin user
        $admin = $userManager->createUser();
        $admin->setUsername('admin');
        $admin->setEmail('admin@songbird.app');
        $admin->setPlainPassword('admin');
        $userManager->updatePassword($admin);
        $admin->setEnabled(1);
        $admin->setFirstname('Admin Firstname');
        $admin->setLastname('Admin Lastname');
        $admin->setRoles(array('ROLE_SUPER_ADMIN'));
        $userManager->updateUser($admin);

        // add test user 1
        $test1 = $userManager->createUser();
        $test1->setUsername('test1');
        $test1->setEmail('test1@songbird.app');
        $test1->setPlainPassword('test1');
        $userManager->updatePassword($test1);
        $test1->setEnabled(1);
        $test1->setFirstname('test1 Firstname');
        $test1->setLastname('test1 Lastname');
        $userManager->updateUser($test1);

        // add test user 2
        $test2 = $userManager->createUser();
        $test2->setUsername('test2');
        $test2->setEmail('test2@songbird.app');
        $test2->setPlainPassword('test2');
        $userManager->updatePassword($test2);
        $test2->setEnabled(1);
        $test2->setFirstname('test2 Firstname');
        $test2->setLastname('test2 Lastname');
        $userManager->updateUser($test2);

        // add test user 3
        $test3 = $userManager->createUser();
        $test3->setUsername('test3');
        $test3->setEmail('test3@songbird.app');
        $test3->setPlainPassword('test3');
        $userManager->updatePassword($test3);
        $test3->setEnabled(0);
        $test3->setFirstname('test3 Firstname');
        $test3->setLastname('test3 Lastname');
        $userManager->updateUser($test3);

        // use this reference in data fixtures elsewhere
        $this->addReference('admin_user', $admin);
    }

    /**
     * {@inheritDoc}
     */
    public function getOrder()
    {
        // load user data
        return 1;
    }
}
```

Now, let us insert the fixtures by running the command line

```
-> scripts/console doctrine:fixtures:load -n
```

The "-n" option simply answer yes when prompted for data purging. Try it without the "-n" option for yourself. Verify that the data is inserted by running a simple query

```
-> scripts/mysql "select * from user"
```

The nice thing about creating fixtures is that you learn a lot about the Entity when you insert the data. Take the FOSUserBundle for example, you need to know about the userManager in order to create encrypted passwords correctly. This knowledge is valuable when writing test cases.

This line shows the power of a modern day framework:

```
$userManager = $this->container->get('fos_user.user_manager');
```

We are trying to use the userManager class using the fos_user.user_manager service. Where is this class?

```
-> scripts/console debug:container | grep fos_user.user_manager
 fos_user.user_manager                            FOS\UserBundle\Doctrine\UserManager
 
 # you can know a great deal about this service from
-> scripts/console debug:container fos_user.user_manager

Information for Service "fos_user.user_manager"
===============================================

 ------------------ ------------------------------------- 
  Option             Value                                
 ------------------ ------------------------------------- 
  Service ID         fos_user.user_manager                
  Class              FOS\UserBundle\Doctrine\UserManager  
  Tags               -                                    
  Public             yes                                  
  Synthetic          no                                   
  Lazy               yes                                  
  Shared             yes                                  
  Abstract           no                                   
  Autowired          no                                   
  Autowiring Types   -                                    
 ------------------ ------------------------------------- 
```

So basically, we are instantiating FOS\UserBundle\Doctrine\UserManager without including the class and we do it as and when we want it. This is called [Lazy Loading](https://en.wikipedia.org/wiki/Lazy_loading). Traditionally, we would require the class and use the "new" keyword, something like this:
 
 ```
 require Class.php
 $myClass = new Class();
 ```
 
 Remember we talked about [services](http://symfony.com/doc/current/book/service_container.html) in the previous chapter? We will see a lot more of these in the later chapters.

## Doctrine Migrations

Doctrine migrations allow us to migrate db changes easily. This is important especially when we want to make changes to production db. For example, if production db is a few versions behind, do we upgrade the db sequentially and safely? 

Let us start the installation:

```
-> scripts/composer require doctrine/doctrine-migrations-bundle "^1.0"
```

and update AppKernel

```
# symfony/app/AppKernel.php

public function registerBundles()
{
    $bundles = array(
        //...
        new Doctrine\Bundle\MigrationsBundle\DoctrineMigrationsBundle(),
    );
}
```

We also need to configure it.

```
# app/config/config.yml
doctrine_migrations:
    dir_name: "%kernel.root_dir%/../src/AppBundle/DoctrineMigrations"
    namespace: AppBundle\DoctrineMigrations
    table_name: migration_versions
    name: AppBundle Migrations
```

If the installation is successful, you should see some new migrations commands added:

```
 ./scripts/console | grep migration
 doctrine:migrations:diff                Generate a migration by comparing your current database to your mapping information.
   doctrine:migrations:execute             Execute a single migration version up or down manually.
   doctrine:migrations:generate            Generate a blank migration class.
   doctrine:migrations:latest              Outputs the latest version number
   doctrine:migrations:migrate             Execute a migration to a specified version or the latest available version.
   doctrine:migrations:status              View the status of a set of migrations.
   doctrine:migrations:version             Manually add and delete migration versions from the version table.
```

and

```
-> scripts/console doctrine:migrations:status

 == Configuration

    >> Name:                                               AppBundle Migrations
    >> Database Driver:                                    pdo_mysql
    >> Database Name:                                      songbird
    >> Configuration Source:                               manually configured
    >> Version Table Name:                                 migration_versions
    >> Version Column Name:                                version
    >> Migrations Namespace:                               AppBundle\DoctrineMigrations
    >> Migrations Directory:                               AppBundle/DoctrineMigrations
    >> Previous Version:                                   Already at first version
    >> Current Version:                                    0
    >> Next Version:                                       Already at latest version
    >> Latest Version:                                     0
    >> Executed Migrations:                                0
    >> Executed Unavailable Migrations:                    0
    >> Available Migrations:                               0
    >> New Migrations:                                     0
```

Its the first time we are using it, so we need to generate the initial migration class

```
-> scripts/console doctrine:migrations:generate
   Generated new migration class to "/var/www/symfony/app/../src/AppBundle/DoctrineMigrations/Version20170128004532.php"
```

Look at "Version20170128004532.php" and you won't see much in there.

## Create Script to Reset Schema and Fixtures

Every time we want to work cleanly, we want to be able to run a script to reset the database and insert dummy records. Let us create a script called resetapp that resides in scripts dir.

```
# scripts/resetapp

#!/bin/bash
rm -rf var/cache/*
# scripts/console cache:clear --no-warmup
scripts/console doctrine:database:drop --force
scripts/console doctrine:database:create
scripts/console doctrine:schema:create
scripts/console doctrine:fixtures:load -n
```

Make sure the script is executable

```
-> chmod u+x ./scripts/resetapp
```

Now we can run the test

```
-> ./scripts/resetapp
./scripts/resetapp 
                                                            
...
  > purging database
  > loading [1] AppBundle\DataFixtures\ORM\LoadUserData
```

## Update runtest script

The runtest script can now call the scripts/resetapp script to have a cleaner start before running the test

```
# scripts/runtest

#!/bin/bash
scripts/resetapp
vendor/bin/codecept run acceptance $@
```

What is "$@"? In bash, it means putting in the command line options that was passed into the runtest script. We can now execute only the RemovalTest like so:

```
# remember to scripts/start_phantomjs in a new terminal if not done.

-> scripts/runtest AppBundleCest.php:RemovalTest

...
Acceptance Tests (1) 
...
Time: 25.43 seconds, Memory: 11.50MB

OK (1 test, 1 assertion)
```

## Summary

In this chapter we learned how to install the doctrine fixtures and migrations bundle. We also created a fixture class for our user bundle. We then upgraded our runtest script to reset the db and load the fixtures before running the test.

Remember to commit all your changes before moving on.

## References

* [DoctrineFixturesBundle](http://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html)

* [DoctrineMigrationsBundle](http://symfony.com/doc/current/bundles/DoctrineMigrationsBundle/index.html)