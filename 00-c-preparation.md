# Preparation for JSBloggers Project

There are a few extra steps of preparation when working with Wheels
projects:

## Installing Servers

### Install Adobe ColdFusion 9

-   Adobe ColdFusion 9
    -   Download and install
        [http://www.adobe.com/products/coldfusion/](http://www.adobe.com/products/coldfusion/)
        as multiple instances
    -   Follow the instructions in the installation wizard and let it
        run to completion. Ensure you select Multiserver configuration.
        -   Presentation / Slides for [Installing Multiple Versions of
            ColdFusion
            Together](http://www.cfgothchic.com/blog/post.cfm/my-cfmeetup-presentation-installing-multiple-versions-of-coldfusion)

    -   Name the instance **cfwheels101**
    -   Test at
        [http://localhost:8301/cfide/administrator/](http://localhost:8301/cfide/administrator/)
    -   Path should be C:\\JRun4\\servers\\cfwheels101\\
    -   Our web root is "C:\\JRun4\\servers\\cfwheels101\\"

### Install Apache ( optional )

-   Apache
    -   Download and install the [Win32 Binary without crypto (no
        mod\_ssl) (MSI
        Installer)](http://apache.tradebit.com/pub//httpd/binaries/win32/httpd-2.2.17-win32-x86-no_ssl.msi)
    -   We will modify three files: httpd.conf, httpd-vhosts.conf, and
        hosts

First uncomment these lines by deleting the hash (`#`) in front of the
line and save the file.

-   C:\\Program Files (x86)\\Apache Software
    Foundation\\Apache2.2\\conf\\httpd.conf

~~~~ {lang="cfm"}
LoadModule proxy_module modules/mod_proxy.soLoadModule proxy_http_module modules/mod_proxy_http.soInclude conf/extra/httpd-vhosts.conf
~~~~

Next add this code to the bottom the the next file, `httpd-vhosts.conf`.

-   C:\\Program Files (x86)\\Apache Software
    Foundation\\Apache2.2\\conf\\extra\\httpd-vhosts.conf

~~~~ {lang="cfm"}
<VirtualHost *:80> ServerName wheels.localProxyRequests Off
  <Proxy *>
  Order deny,allow
  Allow from all
  ProxyPass / http://wheels.local:8301/
 ProxyPassReverse / http://wheels.local:8301/

~~~~

Finally, open the last file, `hosts`, and add this line at the top.

-   C:\\Windows\\System32\\drivers\\etc\\hosts add this:

~~~~ {lang="cfm"}
127.0.0.1    wheels.local
~~~~

Now instead of going to
[http://localhost:8301/](http://localhost:8301/), you can get to our
ColdFusion server instance via
[http://wheels.local/](http://wheels.local/).

### Install ColdFusion on Wheels

-   ColdFusion on Wheels
    -   Download
        [http://cfwheels.org/download/latest-version](http://cfwheels.org/download/latest-version)
    -   Extract and copy the files to "C:\\JRun4\\servers\\cfwheels101"
    -   Test at [http://wheels.local/](http://wheels.local/) and you
        should see a Congratulations page

#### Install DBMigrate Plugin

-   DBMigrate
    -   Download
        [http://code.google.com/p/cfwheels-dbmigrate/](http://code.google.com/p/cfwheels-dbmigrate/)
    -   Move the downloaded zip into
        "C:\\JRun4\\servers\\cfwheels101\\plugins"
    -   Reload CFWheels [http://wheels.local/](http://wheels.local/)
    -   You should see **DBMigrate** under Plugins

#### Install Scaffold Plugin

-   Scaffold
    -   Download
        [http://cfwheels.org/plugins/listing/9](http://cfwheels.org/plugins/listing/9)
    -   Move the downloaded zip into
        "C:\\JRun4\\servers\\cfwheels101\\plugins"
    -   Reload Wheels [http://wheels.local/](http://wheels.local/)
    -   You should see Scaffold under Plugins

### Install MySQL Server

-   Windows
    -   Download and install
        [http://dev.mysql.com/downloads/mirror.php?id=401202](http://dev.mysql.com/downloads/mirror.php?id=401202)

#### Create JSBloggers Database

-   Select Start --\> MySQL - MySQL 5.1 Server --\> MySQL Command Line
    Client
    -   Type your password then "create database jsbloggers;" and press
        return

## Client Tools

### Install ColdFusion Builder

-   Windows
    -   Download and install as standalone
        [http://www.adobe.com/products/coldfusion/cfbuilder/features/](http://www.adobe.com/products/coldfusion/cfbuilder/features/)

## Add JSBloggers connection

-   Add DataSource through the ColdFusion Administrator
    -   Name: JSBloggers
    -   Driver: MySQL
    -   URL: jdbc:mysql://localhost:3306/

## Setup Git and Github

### Create a free Github account

-   Sign up for a free [Github account](https://github.com/signup/free)

### Install Git and Connect to Github

-   [Setup Git and Github](http://help.github.com/win-set-up-git/)

