h1. Preparation for JSBloggers Project

While it's possible to sit and watch the material presented during
class, to receive the maximum benefit you should have your own
system to work on. You will need to prepare your laptop system in
advance of the class so we can cover the maximum amount of
material.

h2. Installing Servers

This class will make use of the Adobe ColdFusion 9 application
server for processing the ColdFusion Markup Language (CFML) pages
that you'll be creating. Our application will also store data in a
relational database using the MySQL database server.

h3. Install MySQL Database

-   Download and install MySQL 5.5 or newer. You can find the
    downloads by going to "MySQL Downloads page":http://www.mysql.com
-   Run the installer and select the options as follows:
    *\* Accept the End-User License Agreement (EULA)*\* Select Custom
    Setup Type
    *\* On the Custom Setup click the "Next" button to select all components*\*
    Click the "Install" button to install the server
    *\* Note: On Windows Vista & Windows 7 computers you'll be prompted if you want to allow this application to make changes to your computer, click the "Yes" button*\*
    On the MySQL Enterprise advertisement screen click the "Next"
    button
    *\* On the MySQL Enterprise monitoring service advertisement screen click the "Next" button*\*
    Check the "Launch the MySQL Instance Configuration Wizard"
    checkbox, then click the "Finish" button
    *\* Note: On Windows Vista & Windows 7 computers you'll be prompted if you want to allow this application to make changes to your computer, click the "Yes" button*\*
    On the "MySQL Instance Configuration Wizard screen" click the
    "Next" button
    *\* Select the "Detailed Configuration" radio button, then click the "Next" button*\*
    On the "Server Type" screen select "Developer Machine" radio
    button, then click the "Next" button
    *\* On the "Database usage" screen, select the "Transactional Database Only" radio button, then click the "Next" button*\*
    On the "Select Drive" screen, click the "Next" button
    *\* On the "Concurrent Connections" screen select "Decision Support (DSS)/OLAP" radio button, then click the "Next" button*\*
    On the "Networking" screen, select the following options:
    **\* Check "Enable TCP/IP Networking"**\* Set the port number to
    "3306" **\* Check "Add firewall exception for this port"**\* Check
    "Enable Strict Mode" *\** Click the "Next" button
-   On the "Default character set" screen, select the "Best Support
    for Multilingualism" radio button, then click the "Next" button
    *\* On the "Set Windows Options" screen, select the following options: **\* Check the "Install as Windows Service" checkbox***
    Set the "Service Name" to "MySQL"
    **\* Check the "Launch MySQL automatically" checkbox**\* Check the
    "Include Bin Directory in Windows PATH" checkbox
    **\* Click the "Next" button** On the "Security Options" screen,
    select the following options:
    **\* Check the "Modifiy Security Settings" checkbox**\* Set the
    root password to "cfwheels" in all lowercase, no quotes
    **\* Check the "Enable root access from remote machines" checkbox**\*
    Uncheck the "Create An Anonymous Account" checkbox
    **\* Click the "Next" button** Click the "Execute" button

h3. Install Adobe ColdFusion 9

-   Adobe ColdFusion 9
    *\* Download and install "http://www.adobe.com/products/coldfusion/":http://www.adobe.com/products/coldfusion/ as multiple instances*\*
    Follow the instructions in the installation wizard and let it run
    to completion. Ensure you select Multiserver configuration.
    **\* Presentation / Slides for "Installing Multiple Versions of ColdFusion Together":http://www.cfgothchic.com/blog/post.cfm/my-cfmeetup-presentation-installing-multiple-versions-of-coldfusion**
    Name the instance **cfwheels101**
    *\* Test at "http://localhost:8301/cfide/administrator/":http://localhost:8301/cfide/administrator/*\*
    Path should be C:\JRun4\servers\cfwheels101  
    \*\* Our web root is "C:\JRun4\servers\cfwheels101"

h3. Install Apache ( optional )

-   Apache
    *\* Download and install the "Win32 Binary without crypto (no mod\_ssl) (MSI Installer)":http://apache.tradebit.com/pub//httpd/binaries/win32/httpd-2.2.17-win32-x86-no\_ssl.msi*\*
    We will modify three files: httpd.conf, httpd-vhosts.conf, and
    hosts

First uncomment these lines by deleting the hash (@\#@) in front of
the line and save the file.

-   C:\Program Files (x86)\Apache Software
    Foundation\Apache2.2\conf\httpd.conf

```cfm

LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
Include conf/extra/httpd-vhosts.conf

```

Next add this code to the bottom the the next file,
@httpd-vhosts.conf@.

-   C:\Program Files (x86)\Apache Software
    Foundation\Apache2.2\conf\extra\httpd-vhosts.conf

```cfm

<VirtualHost *:80>
 ServerName wheels.local

 ProxyRequests Off
  <Proxy *>
  Order deny,allow
  Allow from all
  </Proxy>

 ProxyPass / http://wheels.local:8301/
 ProxyPassReverse / http://wheels.local:8301/
</VirtualHost>

```

Finally, open the last file, @hosts@, and add this line at the
top.

-   C:\Windows\System32\drivers\etc\hosts add this:

```cfm

127.0.0.1   wheels.local

```

Now instead of going to
"http://localhost:8301/":http://localhost:8301/, you can get to our
ColdFusion server instance via
"http://wheels.local/":http://wheels.local/.

h3. Install ColdFusion on Wheels

-   ColdFusion on Wheels
    *\* Download "http://cfwheels.org/download/latest-version":http://cfwheels.org/download/latest-version*\*
    Extract and copy the files to "C:\JRun4\servers\cfwheels101" \*\*
    Test at "http://wheels.local/":http://wheels.local/ and you should
    see a Congratulations page

h4. Install Plugin Manager

-   Plugin Manager *\* Download*\* Move the downloaded zip archive
    file into "C:\ColdFusion9\wwwroot"
    *\* Reload CFWheels "http://wheels.local/":http://wheels.local/*\*
    You should see *Plugin Manager* under Plugins

h4. Install DBMigrate Plugin

-   DBMigrate
    *\* Download "http://code.google.com/p/cfwheels-dbmigrate/":http://code.google.com/p/cfwheels-dbmigrate/*\*
    Move the downloaded zip into "C:\JRun4\servers\cfwheels101\plugins"
    *\* Reload CFWheels "http://wheels.local/":http://wheels.local/*\*
    You should see *DBMigrate* under Plugins

h4. Install Scaffold Plugin

-   Scaffold
    *\* Download "http://cfwheels.org/plugins/listing/9":http://cfwheels.org/plugins/listing/9*\*
    Move the downloaded zip into "C:\JRun4\servers\cfwheels101\plugins"
    *\* Reload Wheels "http://wheels.local/":http://wheels.local/*\*
    You should see Scaffold under Plugins

h3. Install MySQL Server

-   Windows \*\* Download and install
    "http://dev.mysql.com/downloads/mirror.php?id=401202":http://dev.mysql.com/downloads/mirror.php?id=401202

h4. Create JSBloggers Database

-   Select Start --\> MySQL - MySQL 5.1 Server --\> MySQL Command
    Line Client \*\* Type your password then "create database
    jsbloggers;" and press return

h2. Client Tools

h3. Install ColdFusion Builder

-   Windows \*\* Download and install as standalone
    "http://www.adobe.com/products/coldfusion/cfbuilder/features/":http://www.adobe.com/products/coldfusion/cfbuilder/features/

h2. Add JSBloggers connection

-   Add DataSource through the ColdFusion Administrator
    *\* Name: JSBloggers*\* Driver: MySQL \*\* URL:
    jdbc:mysql://localhost:3306/

h2. Setup Git and Github

h3. Create a free Github account

-   Sign up for a free "Github
    account":https://github.com/signup/free

h3. Install Git and Connect to Github

-   "Setup Git and Github":http://help.github.com/win-set-up-git/



