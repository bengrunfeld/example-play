# Play Quick Start Guide

This guide will walk you through deploying a Play application on Deis.

## Prerequisites

* A [User Account](http://docs.deis.io/en/latest/client/register/) on a [Deis Controller](http://docs.deis.io/en/latest/terms/controller/).
* A [Deis Formation](http://docs.deis.io/en/latest/gettingstarted/concepts/#formations) that is ready to host applications

If you do not yet have a controller or a Deis formation, please review the [Deis installation](http://docs.deis.io/en/latest/installation/) instructions.

## Setup your workstation

* Install [RubyGems](http://rubygems.org/pages/download) to get the `gem` command on your workstation
* Install [Foreman](http://ddollar.github.com/foreman/) with `gem install foreman`
* Install [Play](http://www.playframework.com/documentation/2.2.x/Installing)

## Clone your Application

If you want to use an existing application, no problem.  You can also use the Deis sample application located at <https://github.com/opdemand/example-play>.  Clone the example application to your local workstation:

    $ git clone https://github.com/opdemand/example-play.git
    $ cd example-play

## Prepare your Application

To use a Play application with Deis, you will need to conform to 3 basic requirements:

 1. Use [Play Dependencies](http://www.playframework.com/documentation/1.2.1/dependency) to manage dependencies
 2. Use [Foreman](http://ddollar.github.com/foreman/) to manage processes
 3. Use [Environment Variables](https://help.ubuntu.com/community/EnvironmentVariables) to manage configuration inside your application

If you're deploying the example application, it already conforms to these requirements.

#### 1. Use Play Dependencies to manage dependencies

Play requires that you explicitly declare your dependencies using a [conf/dependencies.yml](http://www.playframework.com/documentation/1.2.1/dependency) file. Here is a very basic example:

	# Application dependencies
	
	require:
	    - play 1.2.4

You can then install dependencies on your local workstation with `play dependencies`:

	[info] :: delivering :: exampleapp#exampleapp_2.10;1.0-SNAPSHOT :: 1.0-SNAPSHOT :: integration :: Mon Nov 04 13:21:33 MST 2013
	[info] 	delivering ivy file to /Users/bengrunfeld/Desktop/OpDemand/repos/example-play/exampleApp/target/scala-2.10/ivy-1.0-SNAPSHOT.xml
	
	Here are the resolved dependencies of your application:
	
	+-------------------------------------------------------------------+--------------------------------------------------------+------------------------------+
	| Module                                                            | Required by                                            | Note                         |
	+-------------------------------------------------------------------+--------------------------------------------------------+------------------------------+
	| com.typesafe.play:play-cache_2.10:2.2.0                           | exampleapp:exampleapp_2.10:1.0-SNAPSHOT                | As play-cache_2.10.jar       |
	+-------------------------------------------------------------------+--------------------------------------------------------+------------------------------+
	| net.sf.ehcache:ehcache-core:2.6.6                                 | com.typesafe.play:play-cache_2.10:2.2.0                | As ehcache-core.jar          |

Note: You can test locally using `play run`.

#### 2. Use Foreman to manage processes

Deis relies on a [Foreman](http://ddollar.github.com/foreman/) `Procfile` that lives in the root of your repository.  This is where you define the command(s) used to run your application.  Here is an example `Procfile`:

	web: target/universal/stage/bin/exampleapp -Dhttp.port=$PORT

This tells Deis to run `web` workers using the command `target/universal/stage/bin/exampleapp -Dhttp.port=$PORT`. You can test this locally by running `foreman start`.

	13:27:11 web.1  | started with pid 41249
	13:27:14 web.1  | [info] Loading project definition from /Users/bengrunfeld/Desktop/OpDemand/repos/example-play/exampleApp/project
	13:27:15 web.1  | [info] Set current project to exampleApp (in build file:/Users/bengrunfeld/Desktop/OpDemand/repos/example-play/exampleApp/)
	13:27:15 web.1  | 
	13:27:16 web.1  | --- (Running the application from SBT, auto-reloading is enabled) ---
	13:27:16 web.1  | 
	13:27:16 web.1  | [info] play - Listening for HTTP on /0:0:0:0:0:0:0:0%0:9000

You should now be able to access your application locally at <http://localhost:9000>.

#### 3. Use Environment Variables to manage configuration

Deis uses environment variables to manage your application's configuration. For example, your application listener must use the value of the `PORT` environment variable. The following code snippet demonstrates how this can work inside your application:

    int port = System.getenv("PORT");
    if (port == null){ port = 5000;)}

## Create a new Application

Per the prerequisites, we assume you have access to an existing Deis formation. If not, please review the Deis [installation instuctions](http://docs.deis.io/en/latest/gettingstarted/installation/).

Use the following command to create an application on an existing Deis formation.

    $ deis create --formation=<formationName> --id=<appName>
	Creating application... done, created <appName>
	Git remote deis added
    
If an ID is not provided, one will be auto-generated for you.

## Deploy your Application

Use `git push deis master` to deploy your application.

	$ git push deis master
	Counting objects: 40, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (32/32), done.
	Writing objects: 100% (40/40), 40.64 KiB, done.
	Total 40 (delta 4), reused 0 (delta 0)
	       Play 2.x - Java app detected
	-----> Installing OpenJDK 1.6...done
	-----> Building app with sbt
	-----> Running: sbt clean compile stage			
Once your application has been deployed, use `deis open` to view it in a browser. To find out more info about your application, use `deis info`.

## Scale your Application

To scale your application's [Docker](http://docker.io) containers, use `deis scale` and specify the number of containers for each process type defined in your application's `Procfile`. For example, `deis scale web=8`.

	$ deis scale web=8
	Scaling containers... but first, coffee!
	done in 18s
	
	=== <appName> Containers

	--- web: `target/universal/stage/bin/exampleapp -Dhttp.port=$PORT`
	web.1 up 2013-11-06T23:24:54.875Z (dev-runtime-1)
	web.2 up 2013-11-06T23:27:46.850Z (dev-runtime-1)
	web.3 up 2013-11-06T23:27:46.867Z (dev-runtime-1)
	web.4 up 2013-11-06T23:27:46.884Z (dev-runtime-1)
	web.5 up 2013-11-06T23:27:46.902Z (dev-runtime-1)
	web.6 up 2013-11-06T23:27:46.922Z (dev-runtime-1)
	web.7 up 2013-11-06T23:27:46.942Z (dev-runtime-1)
	web.8 up 2013-11-06T23:27:46.964Z (dev-runtime-1)


## Configure your Application

Deis applications are configured using environment variables. The example application includes a special `POWERED_BY` variable to help demonstrate how you would provide application-level configuration. 

	$ curl -s http://yourapp.yourformation.com
	Powered by Deis
	$ deis config:set POWERED_BY=Play
	=== <appName>
	POWERED_BY: Play
	$ curl -s http://yourapp.yourformation.com
	Powered by Play

`deis config:set` is also how you connect your application to backing services like databases, queues and caches. You can use `deis run` to execute one-off commands against your application for things like database administration, initial application setup and inspecting your container environment.

	$ deis run ls -la
	total 80
	drwxr-xr-x 13 root root 4096 Nov  6 23:24 .
	drwxr-xr-x 57 root root 4096 Nov  6 23:27 ..
	-rw-r--r--  1 root root  141 Nov  6 23:13 .gitignore
	drwxr-xr-x  3 root root 4096 Nov  6 23:13 .ivy2
	drwxr-xr-x  6 root root 4096 Nov  6 23:13 .jdk
	drwxr-xr-x  2 root root 4096 Nov  6 23:24 .profile.d
	-rw-r--r--  1 root root  252 Nov  6 23:24 .release
	drwxr-xr-x  4 root root 4096 Nov  6 23:24 .sbt_home
	drwxr-xr-x  2 root root 4096 Nov  6 23:17 .settings
	-rw-r--r--  1 root root   61 Nov  6 23:13 Procfile
	-rw-r--r--  1 root root 7373 Nov  6 23:13 README.md
	drwxr-xr-x  4 root root 4096 Nov  6 23:13 app
	-rw-r--r--  1 root root  149 Nov  6 23:13 build.sbt
	drwxr-xr-x  2 root root 4096 Nov  6 23:13 conf
	drwxr-xr-x  4 root root 4096 Nov  6 23:24 project
	drwxr-xr-x  5 root root 4096 Nov  6 23:13 public
	-rw-r--r--  1 root root   25 Nov  6 23:13 system.properties
	drwxr-xr-x  3 root root 4096 Nov  6 23:24 target
	drwxr-xr-x  2 root root 4096 Nov  6 23:13 test
	
## Troubleshoot your Application

To view your application's log output, including any errors or stack traces, use `deis logs`.

    $ deis logs
	Nov  6 23:25:09 ip-172-31-11-82 rental-yearling[web.1]: Picked up JAVA_TOOL_OPTIONS:  -Djava.rmi.server.useCodebaseOnly=true
	Nov  6 23:25:09 ip-172-31-11-82 rental-yearling[web.1]: Play server process ID is 13
	Nov  6 23:25:11 ip-172-31-11-82 rental-yearling[web.1]: [#033[37minfo#033[0m] play - Application started (Prod)
	Nov  6 23:25:11 ip-172-31-11-82 rental-yearling[web.1]: [#033[37minfo#033[0m] play - Listening for HTTP on /0:0:0:0:0:0:0:0:10151

## Additional Resources

* [Get Deis](http://deis.io/get-deis/)
* [GitHub Project](https://github.com/opdemand/deis)
* [Documentation](http://docs.deis.io/)
* [Blog](http://deis.io/blog/)
