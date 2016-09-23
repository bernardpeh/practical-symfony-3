# Chapter 10: BDD With Codeception (Optional)

This chapter is optional, feel free to skip it if you already have your own testing framework in place.

Behavioural-Driven-Development (BDD) is best used as [integration testing](https://en.wikipedia.org/wiki/Integration_testing). It is the concept of writing tests based on user's behaviour. The way users interact with the software defines the requirements for the software. Once we know the requirements, we are able to write tests and simulate user's interaction with the software.

In BDD, each user's requirement (user story) can be created using the following template:

```
As a ...
I (don't) want to ...
So that ...
```

We can then further breakdown the story into scenarios. For each scenario, we define the "When" (user's action) and the "Then" (acceptance criteria).

```
Given scenario
When ...
Then ...
```

It is a good idea to create a matrix for user stories and test scenarios to fully capture user's requirement as part of the functional specifications.

## User Stories

Let us define the user stories for this chapter. We will define the user stories before each chapter from now on.

**User Story 10: User Management**

|**Story Id**|**As a**|**I**|**So that I**|
|10.1|test1 user|want to login|can access admin functions|
|10.2|admin user|want to login|can access admin functions|
|10.3|test3 user|don't want to login|can prove that this account is disabled|
|10.4|test1 user|want to manage my own profile|can update it any time|
|10.5|test1 user|dont't want to manage other profiles|don't breach security|
|10.6|admin user|want to manage all users|can control user access of the system|

## User Scenarios

We will break the individual story down with user scenarios.

**Story ID 10.1: As a test1 user, I want to login, so that I can access admin functions.**

|**Scenario Id**|**Given**|**When**|**Then**|
|10.1.1|Wrong login credentials|I login with the wrong credentials|I should see an error message|
|10.1.2|See my dashboard content|I login correctly|I should see Access Denied|
|10.1.3|Logout successfully|I go to the logout url|I should be redirected to the home page|
|10.1.4|Access admin url without logging in|go to admin url without logging in|I should be redirected to the login page|

**Story ID 10.2: As a admin user, I want to login, so that I can access admin functions.**

|**Scenario Id**|**Given**|**When**|**Then**|
|10.2.1|Wrong login credentials|I login with the wrong credentials|I should see an error message|
|10.2.2|See my dashboard content|I login correctly|I should see the text User Management|
|10.2.3|Logout successfully| go to the logout url|I should be redirected to the home page|
|10.2.4|Access admin url without logging in|go to admin url without logging in|I should be redirected to the login page|

**Story ID 10.3: As a test3 user, I don't want to login successfully, so that I can prove that this account is disabled.**

|**Scenario Id**|**Given**|**When**|**Then**|
|10.3.1|Account disabled|I login with the right credentials|I should see an "account disabled" message|

**Story ID 10.4: As a test1 user, I want to manage my profile, so that I can update it any time.**

|**Scenario Id**|Given**|**When**|Then**|
|10.4.1|Show my profile|I go to "/admin/?action=show&entity=User&id=2"|I should see test1@songbird.app|
|10.4.2|Hid uneditable fields|I go to "/admin/?action=edit&entity=User&id=2"|I should not see enabled, locked and roles fields|
|10.4.3|Update Firstname Only|I go to "/admin/?action=edit&entity=User&id=2" And update firstname only And Submit|I should see content updated|
|10.4.4|Update Password Only|I go to "/admin/?action=edit&entity=User&id=2" And update password And Submit And Logout And Login Again|I should see content updated And be able to login with the new password|

**Story ID 10.5: As a test1 user, I don't want to manage other profiles, so that I don't breach security.**

|**Scenario Id**|**Given**|**When**|**Then**|
|10.5.1|List all profiles|I go to "/admin/?action=list&entity=User" url|I should get an "access denied" error.|
|10.5.2|Show test2 profile|I go to "/admin/?action=show&entity=User&id=3"|I should get an "access denied" error.|
|10.5.3|Edit test2 user profile|I go to "/admin/?action=edit&entity=User&id=3"|I should get an "access denied" error|
|10.5.4|See admin dashboard content|I login correctly|I should not see User Management Text|

**Story ID 10.6: As an admin user, I want to manage all users, so that I can control user access of the system.**

|**Scenario Id**|**Given**|**When**|**Then|
|10.6.1|List all profiles|I go to "/admin/?action=list&entity=User" url|I should see a list of all users in a table|
|10.6.2|Show test3 user|I go to "/admin/?action=show&entity=User&id=4" url|I should see test3 user details|
|10.6.3|Edit test3 user|I go to "/admin/?action=edit&entity=User&id=4" url And update lastname|I should see test3 lastname updated on the "List all users" page|
|10.6.4|Create and Delete new user|I got to "/admin/?action=new&entity=User" And fill in the required fields And Submit And Delete the new user|I should see the new user created and deleted again in the listing page.|

## Creating the Cest Class

Since we have already deleted the test directory, let us create a new test dir.

```
-> cd src/AppBundle
-> ../../vendor/bin/codecept bootstrap
-> ../../vendor/bin/codecept build
```

and update the acceptance file again

```
# src/AppBundle/Tests/acceptance.suite.yml
class_name: AcceptanceTester
modules:
    enabled:
        - WebDriver:
            url: 'http://songbird.app'
            browser: chrome
            window_size: 1024x768
            capabilities:
                unexpectedAlertBehaviour: 'accept'
                webStorageEnabled: true
        - \Helper\Acceptance
```

Codeception is really flexible in the way we create the test scenarios. Take User Story 1 for example, we will break the user story down into directories and the scenario into cest class:

```
-> vendor/bin/codecept generate:cest acceptance As_Test1_User/IWantToLogin -c src/AppBundle
-> vendor/bin/codecept generate:cest acceptance As_An_Admin/IWantToLogin -c src/AppBundle
-> vendor/bin/codecept generate:cest acceptance As_Test3_User/IDontWantTologin -c src/AppBundle
-> vendor/bin/codecept generate:cest acceptance As_Test1_User/IWantToManageMyOwnProfile -c src/AppBundle
-> vendor/bin/codecept generate:cest acceptance As_Test1_User/IDontWantToManageOtherProfiles -c src/AppBundle
-> vendor/bin/codecept generate:cest acceptance As_An_Admin/IWantToManageAllUsers -c src/AppBundle
```

We will create a common class in the bootstrap and define all the constants we need for the test.

```
# src/AppBundle/Tests/acceptance/_bootstrap.php

define('ADMIN_USERNAME', 'admin');
define('ADMIN_PASSWORD', 'admin');
define('TEST1_USERNAME', 'test1');
define('TEST1_PASSWORD', 'test1');
define('TEST2_USERNAME', 'test2');
define('TEST2_PASSWORD', 'test2');
// test3 Account is disabled? See data fixtures to confirm.
define('TEST3_USERNAME', 'test3');
define('TEST3_PASSWORD', 'test3');


class Common
{
	public static function login(AcceptanceTester $I, $user, $pass)
    {
        $I->amOnPage('/login');
        $I->fillField('_username', $user);
        $I->fillField('_password', $pass);
        $I->click('_submit');
    }
}
```

Let us try creating story 10.6

```
# src/AppBundle/Tests/acceptance/As_An_Admin/IWantToManageAllUsersCest.php

namespace As_An_Admin;
use \AcceptanceTester;
use \Common;

class IWantToManageAllUsersCest
{
    public function _before(AcceptanceTester $I)
    {
    }

    public function _after(AcceptanceTester $I)
    {
    }

    protected function login(AcceptanceTester $I)
    {
        Common::login($I, ADMIN_USERNAME, ADMIN_PASSWORD);
    }

    /**
     * Scenario 10.6.1
     * @before login
     */
    public function listAllProfiles(AcceptanceTester $I)
    {
        $I->amOnPage('/admin/?action=list&entity=User');
        $I->canSeeNumberOfElements('//table/tbody/tr',4);
    }
}
```
Noticed the [xpath](https://msdn.microsoft.com/en-us/library/ms256086(v=vs.110).aspx) selector?

```
//table/tbody/tr
```

This is the xpath for the show button. How do we know where it is located? We can inspect the elements with the developer tool (available in many browser).

You also noticed that the login class is protected rather than public. Protected class won't be executed when we run the "runtest" command but we can use it as a pre-requisite when testing listAppProfiles scenario for example, ie the @before login annotation.

listAllProfiles function goes to the user listing page and checks for 4 rows in the table. How do I know about the amOnPage and canSeeNumberOfElements functions? Remembered you ran the command "/bin/codecept build" before? This command generates the AcceptanceTester class to be used in the Cest class. All the functions of the AcceptanceTester class can be found in the "src/AppBundle/Tests/_support/_generated/AcceptanceTesterActions.php" class.

In the test, I used the user listing url directly rather than clicking on the "User Management" link. *Simulating user clicks should be the way to go because you are simulating user behaviour*. We will update the test again once we work on the UI updated.

Let us update the runtest script

```
# scripts/runtest

#!/bin/bash

scripts/resetapp
vendor/bin/codecept run acceptance $@ -c src/AppBundle
```

and update the gitignore path

```
# .gitignore
...
src/AppBundle/Tests/_output/*
```

Then, run the test only for scenario 10.6.1

```
# remember to start selenium server in a separate terminal
-> scripts/start_selenium
# switch to a new terminal and run the test
-> scripts/runtest As_An_Admin/IWantToManageAllUsersCest.php:listAllProfiles
...
OK (1 test, 1 assertion)
```

Looking good, what if the test fails and you want to look at the logs? The log files are all in the "Tests/_output/" directory.

Let us write another test for scenario 10.6.2. We will simulate clicking on test3 show button and check the page is loading fine.

```
# src/AppBundle/Tests/acceptance/As_An_Admin/IWantToManageAllUsersCest.php
...
   /**
    * Scenario 10.6.2
    * @before login
    */
   public function showTest3User(AcceptanceTester $I)
   {
       // go to user listing page
       $I->amOnPage('/admin/?action=list&entity=User');
       // click on show button
       $I->click('Show');
       $I->waitForText('test3@songbird.app');
       $I->canSee('test3@songbird.app');
   }
...
```

run the test now

```
-> scripts/runtest As_An_Admin/IWantToManageAllUsersCest.php:showTest3User
```

and you should get a success message.

We will now write the test for scenario 10.6.3

```
# src/AppBundle/tests/acceptance/As_An_Admin/IWantToManageAllUsersCest.php
...
    /**
     * Scenario 10.6.3
     * @before login
     */
    public function editTest3User(AcceptanceTester $I)
    {
        // go to user listing page
        $I->amOnPage('/admin/?action=list&entity=User');
        // click on edit button
        $I->click('Edit');
        // check we are on the right url
        $I->canSeeInCurrentUrl('/admin/?action=edit&entity=User');
        $I->fillField('//input[@value="test3 Lastname"]', 'lastname3 updated');
        // update
        $I->click('//button[@type="submit"]');
        // go back to listing page
        $I->amOnPage('/admin/?action=list&entity=User');
        $I->canSee('lastname3 updated');
        // now revert username
        $I->amOnPage('/admin/?action=edit&entity=User&id=4');
        $I->fillField('//input[@value="lastname3 updated"]', 'test3 Lastname');
        $I->click('//button[@type="submit"]');
        $I->amOnPage('/admin/?action=list&entity=User');
        $I->canSee('test3 Lastname');
    }
...
```

Run the test now to make sure everything is ok before moving on.

```
-> scripts/runtest As_An_Admin/IWantToManageAllUsersCest.php:editTest3User
```

and scenario 10.6.4

```
# src/AppBundle/tests/acceptance/As_An_Admin/IWantToManageAllUsersCest.php
...
   /**
    * Scenario 10.6.4
    * @before login
    */
   public function createAndDeleteNewUser(AcceptanceTester $I)
   {
       // go to create page and fill in form
       $I->amOnPage('/admin/?action=new&entity=User');
       $I->fillField('//input[contains(@id, "_username")]', 'test4');
       $I->fillField('//input[contains(@id, "_email")]', 'test4@songbird.app');
       $I->fillField('//input[contains(@id, "_plainPassword_first")]', 'test4');
       $I->fillField('//input[contains(@id, "_plainPassword_second")]', 'test4');
       // submit form
       $I->click('//button[@type="submit"]');
       // go back to user list
       $I->amOnPage('/admin/?entity=User&action=list');
       // i should see new test4 user created
       $I->canSee('test4@songbird.app');

       // now delete user
       // click on edit button
       $I->click('Delete');
       // wait for model box and then click on delete button
       $I->waitForElementVisible('//button[@id="modal-delete-button"]');
       $I->click('//button[@id="modal-delete-button"]');
       // I can no longer see test4 user
       $I->cantSee('test4@songbird.app');
   }
...
```

createNewUser test is a bit longer. I hope the comments are self explainatory.

Let's run the test just for this scenario

```
-> scripts/runtest As_An_Admin/IWantToManageAllUsersCest.php:createAndDeleteNewUser
```

Feeling confident? We can run all the test together now

```
-> scripts/runtest

Dropped database for connection named `songbird`
Created database `songbird` for connection named default
ATTENTION: This operation should not be executed in a production environment.

Creating database schema...
Database schema created successfully!
  > purging database
  > loading [1] AppBundle\DataFixtures\ORM\LoadUserData
Codeception PHP Testing Framework v2.1.1
Powered by PHPUnit 4.7.7 by Sebastian Bergmann and contributors.

Acceptance Tests (9) -----------------------------------------------------
Testing acceptance
✔ IWantToLoginCest: Try to test (0.00s)
✔ IWantToManageAllUsersCest: List all profiles (5.89s)
✔ IWantToManageAllUsersCest: Show test3 user (2.35s)
✔ IWantToManageAllUsersCest: Edit test3 user (6.33s)
✔ IWantToManageAllUsersCest: Create and delete new user (6.29s)
✔ IDontWantToManageOtherProfilesCest: Try to test (0.00s)
✔ IWantToLoginCest: Try to test (0.00s)
✔ IWantToManageMyOwnProfileCest: Try to test (0.00s)
✔ IDontWantTologinCest: Try to test (0.00s)
--------------------------------------------------------------------------
```

Want more detail output? Try this

```
-> ./scripts/runtest --steps
```

How about with debug mode

```
-> ./scripts/runtest -d
```

Tip: If you are using mac and got "too many open files" error, you need to change the ulimit to something bigger

```
-> ulimit -n 2048
```

Add this to your ~/.bash_profile if you want to change the limit everytime you open up a shell.

If your machine is slow, sometimes it might take too long before certain text or element is being detected. In that case, use the "waitForxxx" function before the assert statement, like so

```
# wait for element to be loaded first
# you can see all the available functions in src/AppBundle/Tests/_support/_generated/AcceptanceTesterActions.php
$I->waitForElement('//div[contains(@class, "alert-success")]');
# now we can do the assert statement
$I->canSeeElement('//div[contains(@class, "alert-success")]');
```

We have only written the BDD tests for user story 10.6. Are you ready to write acceptance tests for the other user stories?

Writing tests can be a boring process but essential if you want your software to be robust. A tip to note is that every scenario must have a closure so that it is self-contained. The idea is that you can run a test scenario by itself without affecting the rest of the scenarios. For example, if you change a password in a scenario, you have to remember to change it back so that you can run the next test without worrying that the password has been changed. Alternatively, you could reset the db after every test but this would make running all the tests longer.  There are also other ways you can achieve this. How can you do it such that it doesn't affect performance?

The workflow in this book is just one of many ways to write BDD tests. At the time of writing, many people uses [behat](http://docs.behat.org/en/v3.0/).

## Summary

In this chapter, we wrote our own CEST classes based on different user stories and scenarios. We are now more confident that we have a way to test Songbird's user management functionality as we add more functionalities in the future.

Remember to commit your changes before moving on the next chapter.

## Exercises

* Write acceptance test for User Stories 10.1, 10.2, 10.3, 10.4 and 10.5.

* (Optional) Can you think of other business rules for user management? Try adding your own CEST.

## References

* [More BDD Readings](https://en.wikipedia.org/wiki/Behavior-driven_development)

* [User Story](https://en.wikipedia.org/wiki/User_story)

* [integration testing](https://en.wikipedia.org/wiki/Integration_testing)
