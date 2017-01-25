# Chapter 1: Survival Skills

Without a doubt, the 2 biggest Symfony resources on the web at the moment are "The Book" and "The Cookbook", both can be downloaded from [Symfony Documentation Page](http://symfony.com/doc/current/index.html). Cudos to Fabien and the team behind the books, making Symfony one of the best documented frameworks out there. Having said that, the content in these 2 books are hard to digest and almost impossible to follow unless you have good foundation in Object Oriented Programming. There are a lot to go through. Even if you are have the skills, you will need enough determination to read them. Even if you finish reading them, you still need to have enough practical experience to digest the theory.

I hope there is really a simple formula to become a Symfony ninja overnight...

## The Tools You Need

You will need to equip yourself before diving in. Ideally, you have

* A good computer. I recommend a modern day *Mac* not more than 4 years old with at least 100G of free space to setup development environment. Mac is fast becoming the new standard for coding. Linux is fine. If you insist in windows, make sure you have command line - [cygwin](https://www.cygwin.com/) is a good option.
* Good foundation in programming. Experience with Object Oriented Programming and relational databases is recommended.
* Understand Dependency Injection (DI). Fabien wrote a good article about [DI](http://fabien.potencier.org/what-is-dependency-injection.html). DI is the heartbeat of Symfony and most modern day framework.
* Good source control knowledge, especially Git and Git Flow.
* Basic HTML, CSS and Javascript knowledge.
* Basic Stylesheet Pre-processor language like LESS or SASS.
* Basic Linux command line knowledge.
* A good IDE. There are lots of them out there. [Sublime Text](www.sublimetext.com) is OK but [PHP Storm](https://www.jetbrains.com/phpstorm/) is way better for serious Symfony development.

I hope the list doesn't scare you to get started.

## Using the Command Line

I suggest you to get comfortable with the command line. Many modern day frameworks use command line to automate tasks. In this book, I'll be using a lot of command line but I suggest you not to memorise them. Always type "app/console" to see the options and then narrow in from there.

For example, your app/console might look like this (doesn't matter if it doesn't at this point):

```
-> bin/console

...

Available commands:
  help                                 Displays help for a command
  list                                 Lists commands
 assetic
  assetic:dump                         Dumps all assets to the filesystem
  assetic:watch                        Dumps assets to the filesystem as their source files are modified
 assets
  assets:install                       Installs bundles web assets under a public web directory
 cache
  cache:clear                          Clears the cache
  cache:warmup                         Warms up an empty cache
 config
  config:debug                         Dumps the current configuration for an extension
  config:dump-reference                Dumps the default configuration for an extension
 container
  container:debug                      Displays current services for an application
 debug
  debug:config                         Dumps the current configuration for an extension
  debug:container                      Displays current services for an application
  debug:event-dispatcher               Displays configured listeners for an application
  debug:router                         Displays current routes for an application
  debug:swiftmailer                    Displays current mailers for an application
  debug:translation                    Displays translation messages information
  debug:twig                           Shows a list of twig functions, filters, globals and tests
 doctrine
  doctrine:cache:clear-metadata        Clears all metadata cache for an entity manager
  doctrine:cache:clear-query           Clears all query cache for an entity manager
  doctrine:cache:clear-result          Clears result cache for an entity manager
  doctrine:database:create             Creates the configured database
  doctrine:database:drop               Drops the configured database
  doctrine:ensure-production-settings  Verify that Doctrine is properly configured for a production environment.
  doctrine:generate:crud               Generates a CRUD based on a Doctrine entity
  doctrine:generate:entities           Generates entity classes and method stubs from your mapping information
  doctrine:generate:entity             Generates a new Doctrine entity inside a bundle
  doctrine:generate:form               Generates a form type class based on a Doctrine entity
  doctrine:mapping:convert             Convert mapping information between supported formats.
  doctrine:mapping:import              Imports mapping information from an existing database
  doctrine:mapping:info
  doctrine:query:dql                   Executes arbitrary DQL directly from the command line.
  doctrine:query:sql                   Executes arbitrary SQL directly from the command line.
  doctrine:schema:create               Executes (or dumps) the SQL needed to generate the database schema
  doctrine:schema:drop                 Executes (or dumps) the SQL needed to drop the current database schema
  doctrine:schema:update               Executes (or dumps) the SQL needed to update the database schema to match the current mapping metadata.
  doctrine:schema:validate             Validate the mapping files.
 fos
  fos:user:activate                    Activate a user
  fos:user:change-password             Change the password of a user.
  fos:user:create                      Create a user.
  fos:user:deactivate                  Deactivate a user
  fos:user:demote                      Demote a user by removing a role
  fos:user:promote                     Promotes a user by adding a role
 generate
  generate:bundle                      Generates a bundle
  generate:controller                  Generates a controller
  generate:doctrine:crud               Generates a CRUD based on a Doctrine entity
  generate:doctrine:entities           Generates entity classes and method stubs from your mapping information
  generate:doctrine:entity             Generates a new Doctrine entity inside a bundle
  generate:doctrine:form               Generates a form type class based on a Doctrine entity
 init
  init:acl                             Mounts ACL tables in the database
 lint
  lint:twig                            Lints a template and outputs encountered errors
  lint:yaml                            Lints a file and outputs encountered errors
 orm
  orm:convert:mapping                  Convert mapping information between supported formats.
 router
  router:debug                         Displays current routes for an application
  router:dump-apache                   [DEPRECATED] Dumps all routes as Apache rewrite rules
  router:match                         Helps debug routes by simulating a path info match
 security
  security:check                       Checks security issues in your project dependencies
  security:encode-password             Encodes a password.
 server
  server:run                           Runs PHP built-in web server
  server:start                         Starts PHP built-in web server in the background
  server:status                        Outputs the status of the built-in web server for the given address
  server:stop                          Stops PHP's built-in web server that was started with the server:start command
 swiftmailer
  swiftmailer:debug                    Displays current mailers for an application
  swiftmailer:email:send               Send simple email message
  swiftmailer:spool:send               Sends emails from the spool
 translation
  translation:debug                    Displays translation messages information
  translation:update                   Updates the translation file
 twig
  twig:debug                           Shows a list of twig functions, filters, globals and tests
  twig:lint                            Lints a template and outputs encountered errors
 yaml
  yaml:lint                            Lints a file and outputs encountered errors
```

Wow, that is a lot but don't worry, you will get used to the important ones after finishing the book.

## Selling Your Soul to the Demon

Many people use [web frameworks](http://symfony.com/why-use-a-framework) to create internet applications nowadays. A framework speeds up web development by giving you automation tools to create commonly used features like user management system, forms, pages, menus...etc. This means that you can create these features easily without knowing how they work. It is like buying a car without knowing how the car works. This is all good until if you want to customise the inner components or repair it. You could get someone to customise the car (hire a developer) or DIY.

If you are a developer, there is value in learning how to built a CMS with Symfony. While building the CMS, you learn how to configure and customise all the bundles to make them work together. As the builder, you will know where to start troubleshooting when things go wrong.

Let's get the ball rolling...

## Summary

This is a short chapter. We discussed the basic skills required to learn a modern day framework like Symfony. You were mentally prepared and warned about the pros and cons of using a framework.


## References

* [git](https://git-scm.com/)

* [gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow/)

