# Chapter 8: Fixtures, Fixtures, Fixtures

As of now, we could create and manage users via the command line (app/console fos:user:xxx) or using the basic CRUD UI that we have created. What if we messed up the data or if we want to reset the data with certain values. How can we do that efficiently? We need an automation mechanism to create consistent dummy data.

## Install DoctrineFixturesBundle

Install via composer

```
-> composer doctrine/doctrine-fixtures-bundle ^2.3 --dev
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
-> app/console | grep fixtures
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
-> app/console doctrine:fixtures:load -n
```

The "-n" option simply answer yes when prompted for data purging. Try it without the "-n" option for yourself. Verify that the data is inserted by logging into adminer and check the user table

```
http://adminer.app/
```

The nice thing about creating fixtures is that you learn a lot about the Entity when you insert the data. Take the FOSUserBundle for example, you need to know about the userManager in order to create encrypted passwords correctly. This knowledge is valuable when writing test cases.

This line shows the power of a modern day framework:

```
$userManager = $this->container->get('fos_user.user_manager');
```

We are trying to use the userManager class using the fos_user.user_manager service. Where is this class?

```
-> app/console debug:container | grep fos_user.user_manager
 fos_user.user_manager                            FOS\UserBundle\Doctrine\UserManager
```

So basically, we are instantiating FOS\UserBundle\Doctrine\UserManager without including the class. Remember we talked about [services](http://symfony.com/doc/current/book/service_container.html) in the previous chapter? We will sell a lot more of these in action in the later chapters.

## Create Script to Reset Fixtures

Every time we want to work cleanly, we want to be able to run a script to reset the database and insert dummy records. Let us create a script called resetapp that resides in scripts dir.

```
# scripts/resetapp

#!/bin/bash
rm -rf app/cache/*
# app/console cache:clear --no-warmup
app/console doctrine:database:drop --force
app/console doctrine:database:create
app/console doctrine:schema:create
app/console doctrine:fixtures:load -n
```

I prefer to use the "rm" command to delete all caches more than the app/console cache:clear command. I leave it up to you to decide which one you prefer. Make sure the script is executable

```
-> chmod u+x ./scripts/resetapp
```

Now we can run the test

```
-> ./scripts/resetapp
Dropped database for connection named `songbird`
Created database `songbird` for connection named default
ATTENTION: This operation should not be executed in a production environment.

Creating database schema...
Database schema created successfully!
  > purging database
  > loading [1] Songbird\UserBundle\DataFixtures\ORM\LoadUserData
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
# remember to ./scripts/start_selenium if not done.

-> ./scripts/runtest AppBundleCest.php:RemovalTest

/scripts/runtest AppBundleCest.php:RemovalTest
Dropped database for connection named `songbird`
Created database `songbird` for connection named default
ATTENTION: This operation should not be executed in a production environment.

Creating database schema...
Database schema created successfully!
  > purging database
  > loading [1] AppBundle\DataFixtures\ORM\LoadUserData
Codeception PHP Testing Framework v2.2.2
Powered by PHPUnit 4.8.26 by Sebastian Bergmann and contributors.
...

Time: 14.11 seconds, Memory: 10.50Mb

OK (1 test, 1 assertion)
```

## Summary

In this chapter we learned how to install the fixtures bundle and created a fixture class for our user bundle. We then upgraded our test script to load the fixtures and run the test automatically.

Remember to commit all your changes before moving on.

## References

* [DoctrineFixturesBundle](http://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html)