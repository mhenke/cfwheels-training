# Preparation for JSBloggers Project

There are a few extra steps of preparation when working with Wheels projects:

## Installing Servers

### Install Adobe ColdFusion 9

* Download and install [http://www.adobe.com/products/coldfusion/](http://www.adobe.com/products/coldfusion/) as multiple instances
* Follow the instructions in the installation wizard and let it run to completion. Ensure you select Multiserver configuration.
* Presentation / Slides for [Installing Multiple Versions of ColdFusion Together](http://www.cfgothchic.com/blog/post.cfm/my-cfmeetup-presentation-installing-multiple-versions-of-coldfusion)
* Name the instance **cfwheels101**
* Test at [http://localhost:8301/cfide/administrator/](http://localhost:8301/cfide/administrator/)
* File Path to the Web root should be C:\\JRun4\\servers\\cfwheels101\\

### Install ColdFusion on Wheels

* Download [http://cfwheels.org/download/latest-version](http://cfwheels.org/download/latest-version)
* Extract and copy the files to "C:\\JRun4\\servers\\cfwheels101"
* Test at [http://wheels.local/](http://wheels.local/) and you should see a Congratulations page

#### Install DBMigrate Plugin

* Download [http://cfwheels.org/plugins/listing/28](http://cfwheels.org/plugins/listing/28)
* Move the downloaded zip into the plugins folder. Example: "C:\\JRun4\\servers\\cfwheels101\\plugins"
* Reload CFWheels [http://wheels.local/](http://wheels.local/)
* You should see **DBMigrate** under Plugins

#### Install Scaffold Plugin

* Download [http://cfwheels.org/plugins/listing/9](http://cfwheels.org/plugins/listing/9)
* Move the downloaded zip into the plugins folder. Example: "C:\\JRun4\\servers\\cfwheels101\\plugins"
* Reload CFWheels [http://wheels.local/](http://wheels.local/)
* You should see **DBMigrate** under Plugins

### Install MySQL Server

#### Windows

* Download and install [http://dev.mysql.com/downloads/mirror.php?id=401202](http://dev.mysql.com/downloads/mirror.php?id=401202)
* Remember the password you enter when installing.

#### Create JSBloggers Database

Mysql commands can span several lines. Do not forget to end your mysql command with a semicolon. You may need to usethe "MySQL Command Line" in the start menu if the normal command prompt doesn't work when issuing the following commands.

In the command prompt type:

* mysql -u root -p

* mysql> CREATE DATABASE JSBloggers;
* mysql> exit

### Add JSBloggers connection

Add DataSource through the ColdFusion Administrator

* Driver: MySQL (4/5)
* Name: JSBloggers
* Database: JSBloggers
* Server: localhost
* User name: root
* Password: Password entered when installing MySQL

## Client Tools

### Install ColdFusion Builder or CFEclipse

#### Editor

* Download and install as standalone http://www.adobe.com/products/coldfusion/cfbuilder/features/
OR
* Download and install Eclipse, then add CFEclipse

## Setup Git and Github

* Sign up for a free [Github account](https://github.com/signup/free)
* [Setup Git and Github](http://help.github.com/win-set-up-git/)