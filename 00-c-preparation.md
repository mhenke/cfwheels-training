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

* Download [http://code.google.com/p/cfwheels-dbmigrate/](http://code.google.com/p/cfwheels-dbmigrate/)
* Move the downloaded zip into "C:\\JRun4\\servers\\cfwheels101\\plugins"
* Reload CFWheels [http://wheels.local/](http://wheels.local/)
* You should see **DBMigrate** under Plugins

### Install MySQL Server

#### Windows

* Download and install [http://dev.mysql.com/downloads/mirror.php?id=401202](http://dev.mysql.com/downloads/mirror.php?id=401202)

#### Create JSBloggers Database

* Select Start --> MySQL - MySQL 5.1 Server --> MySQL Command Line Client
* Type your password then "create database jsbloggers;" and press return

### Add JSBloggers connection

Add DataSource through the ColdFusion Administrator

* Name: JSBloggers
* Driver: MySQL
* URL: jdbc:mysql://localhost:3306/

## Client Tools

### Install ColdFusion Builder or CFEclipse

#### Windows and CFBuilder

* Download and install as standalone [http://www.adobe.com/products/coldfusion/cfbuilder/features/ (http://www.adobe.com/products/coldfusion/cfbuilder/features/)

## Setup Git and Github

* Sign up for a free [Github account](https://github.com/signup/free)
* [Setup Git and Github](http://help.github.com/win-set-up-git/)