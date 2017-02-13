# Chapter 4: The Testing Framework Part 1 (Optional)

This chapter talks about [Codeception](http://codeception.com/). Feel free to skip it if you already have a testing framework in place.

No application is complete without going through a rigorous testing process. Software Testing is a big topic by itself.

Today, many developers know [TDD](https://en.wikipedia.org/wiki/Test-driven_development) and [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development). Test First Development ensures that your software is reliable but requires a lot of patience and extra work to implement it correctly. Think of it like a quality control process. The more checks you have, the less bugs your have. Of course, you can cost cut by not having checks and hope that your product is still bug free. This is quite unlikely especially if the software is complex.

Personally, I prefer to write user stories and scenarios first rather than spending time coding the tests. Think of them as pseudocode. Once we have the user stories and scenarios defined, we will jump in and code functionality A. When functionality A is completed, we will code the test cases and ensure they pass before moving on. We will repeat the cycle for functionality B before moving on to functionality C. The idea is to not break existing functionalities while adding on new functionalities.

Everyone's testing approach is different. You could implement your own approach.

There are many frameworks for acceptance testing. [Behat](http://docs.behat.org/) and [Mink](http://mink.behat.org/) are the industrial standard at the moment. In this book, we will be using [Codeception](http://codeception.com/) to write acceptance tests in most cases. We will also be writing some functional test in phpunit.

## Installation

```
-> cd symfony
# only thing about running docker is that for anything relating to db connection,
# we need to execute commands in the docker instance. we will create a wrapper for this in the future
-> docker-compose exec php composer require codeception/codeception --dev
```

The "--dev" means we only need this in dev mode. If everything is working, you will see composer adding the dependency in composer.json

```
# symfony/composer.json

# add the codeception line under require-dev
"require-dev": {
    ...
    "codeception/codeception": "^2.2"
},
```

Now we can initialise codeception

```
-> vendor/bin/codecept bootstrap
```

Let us configure the acceptance test.

```
# symfony/tests/acceptance.suite.yml
class_name: AcceptanceTester
modules:
    enabled:
        - WebDriver:
            url: 'http://songbird.app'
            host: 172.25.0.5
            port: 4444
            browser: phantomjs
            window_size: 1024x768
            capabilities:
                unexpectedAlertBehaviour: 'accept'
                webStorageEnabled: true
        - \Helper\Acceptance
```

Acceptance Testing is like Black Box Testing - We try to simulate real users interacting with our app. We ignore the inner workings of the code and only care if it works from the end user's point of view.

Here, we are using the headless browser - [phantomjs](http://phantomjs.org) to connect to the webserver at 172.25.0.5 (see the docker-compose.yml file). Codeception by default comes with PhpBrowser which doesn't support javascript. [Selenium](http://www.seleniumhq.org/) is slow but is the veteran when comes to acceptance testing. Feel free to switch to selenium if you encounter problems.

We can now generate the acceptance actions based on the updated acceptance suite:

```
-> vendor/bin/codecept build

# we will now get all the codecept libraries for free

Building Actor classes for suites: acceptance, functional, unit
 -> AcceptanceTesterActions.php generated successfully. 0 methods added
\AcceptanceTester includes modules: WebDriver, \Helper\Acceptance
 -> FunctionalTesterActions.php generated successfully. 0 methods added
\FunctionalTester includes modules: \Helper\Functional
 -> UnitTesterActions.php generated successfully. 0 methods added
\UnitTester includes modules: Asserts, \Helper\Unit
```

## The First Test

We know that the default Symfony comes with the AppBundle example. Let us now test the bundle by creating a test suite for it.


```
-> vendor/bin/codecept generate:cest acceptance AppBundle
```

The auto generated Cest class should look like this:

```
# symfony/tests/acceptance/AppBundleCest.php

class AppBundleCest
{
    public function _before(AcceptanceTester $I)
    {
    }

    public function _after(AcceptanceTester $I)
    {
    }

   ...
}
```

Let us write our own test. All new Symfony installation homepage should have a successful message.


```
# symfony/tests/acceptance/AppBundleCest.php
...
# replaced tryToTest function with InstallationTest function
public function InstallationTest(AcceptanceTester $I)
{
    $I->wantTo('Check if Symfony is installed successfully.');
    $I->amOnPage('/');
    $I->see('Welcome to');
}
```

We have been running the codecept command from the host machine. That is fine but we should really be running the command in the php docker container. In the symfony dir, we need to softlink the .env file as we are going to run docker commands in that dir. If not, you will get a bunch of environment variables not found error.

```
# in the symfony dir
-> ln -s ../.env
```

Now run the test:

```
-> docker-compose exec php vendor/bin/codecept run acceptance AppBundleCest

Codeception PHP Testing Framework v2.2.8
Powered by PHPUnit 5.7.5 by Sebastian Bergmann and contributors.

Acceptance Tests (1) ------------------------------------------------------------------------
Testing acceptance
✔ AppBundleCest: Check if symfony is installed successfully. (4.82s)
---------------------------------------------------------------------------------------------


Time: 5.86 seconds, Memory: 13.50MB

OK (1 test, 1 assertion)
```

Some files such as images are binary. We need to tell git not to convert the line endings (google for it if interested)

```
# .gitattributes

...
# Denote all files that are truly binary and should not be modified.
*.png binary
*.jpg binary
```

Don't forget to commit your code before moving on to the next chapter.

```
-> git add symfony
-> git commit -m"added codeception and created basic test"
# update remote repo so you dont lose it
-> git push -u origin my_chapter4
```

## Summary

In this chapter, we discussed the importance of testing and touched on TDD and BDD. In our context, we will be mainly writing BDD tests. We installed codeception and wrote a simple acceptance test to tests the default symfony home page.

## Exercises (Optional)

* Try configure codeception to allow the running of different acceptance testing profiles. Can you test with PhpBrowser or selenium easily? Do you see any benefit of doing that? See [advanced codeception](http://codeception.com/docs/07-AdvancedUsage) for help.

## Resources

* [TDD](https://en.wikipedia.org/wiki/Test-driven_development)

* [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development)

* [Codeception documentation](http://codeception.com/docs)



