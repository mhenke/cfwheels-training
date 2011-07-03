## I4: Installing Plugins

In this iteration we'll learn more about plugins and how to take advantage of the many plugins available to quickly add features to your application. We have already worked with several plugins like **DBMigrations** and **Scaffolding**. We will also cover several other topics like Event Handlers, Responding with Multiple Formats, Layouts, and Sass (Syntactically Awesome StyleSheets).

### Using the **Plugin Manager**

Wheels plugins are distributed in a zip that get stored into your applications file structure. One advantage of this method is the plugin could be easily checked into your source control system along with everything you wrote in the app. The disadvantage is it made upgrading to newer versions of the plugin, and dealing with the versions at all, complicated. The **Plugin Manager** will help us with some some of these disadvantages.

Download and drop the **Plugin Manager** in the '/plugins' folder. Reload the application and we can start using the **Plugin Manager** in our application!

### Event Handlers

Once loaded, the **Plugin Manager** notices we haven't set a reloadPassword yet. For security on production we would probably have a password set so not everyone can reload our application. Most likely, we would set this in '/config/production/settings.cfm' but this instruction '<cfset set(reloadPassword="y0urPassw0rdShouldNotBeThis")>'. We will add '<cfset application.pluginManager.requirePassword = false>' to '/events/onapplicationstart.cfm'.

**Events folder?** Well, our 'Application.cfc' in Wheels includes 'wheels/functions.cfm' which has a lot of framework code. As a general recommendation we never touch any files in the wheels folder. So how do we using the 'Application.cfc' events then? The answer is to use the events folder. There is a file in there for every single event that 'Application.cfc' can trigger plus some special events for Wheels like "On Error".

### Using the **Plugin Manager** (For Real)

Now we can reload Wheels, and should see a list of all the Wheels plugins. First will be plugins we are currently using. If an upgrade exists, you will see "Version X.X.X Available: 'More Info' 'Auto Upgrade' 'Download'". We can select 'Auto Upgrade' so we won't have to worry about updates to plugins. 'Download' will download the plugin into our '/plugins/' folder.

### Responding with Multiple Formats

Changing gears, Wheels controllers provide some powerful mechanisms for responding to requests for content in XML, JSON, and other formats. We want to provide our articles in html (default), pdf, json, and word. First we will add this code to '/controllers/Articles.cfc'. This code will tell the Wheels controller to be ready to provide content in these formats:

```cfm
 <cfset provides("html,json,pdf")>
```

In our 'index' action of '/controllers/Articles.cfc' add this line '<cfset renderWith(articles)>'. Wheels can handle HTML, XML, JSON, CSV, PDF, and XLS. XML and JSON Formats can be automatic generated. Let's test the JSON response. Add '?format=json' to the end of our index page's url. It should return a JSON file. Try '?format=pdf'.

Element TEMPLATE is undefined in LOC. The error occurred in C:\\JRun4\\servers\\cfwheels\\cfusion.ear\\cfusion.war\\wheels\\controller\\provides.cfm: line 123

Not to helpful, huh. Well, we mentioned XML and JSON can be automatically generated so we'll need to provide our own custom responses for PDF. Create '/views/articles/index.pdf.cfm' and place this in it.

```cfm
Listing articles     
#articles.title#
```

Reload the page with the error and you should see a pdf with our custom content.

### Adding a Word format

In 'config/settings.cfm' add '<cfset addFormat(extension="doc", mimeType="application/msword")/>'.

Don't forget to add 'doc' to the 'provides' instruction in '/controllers/Articles.cfc'.

Copy '/views/articles/index.pdf.cfm' and delete the starting and ending 'cfdocument' instructions. This isn't very DRY but you can fix that later. Then refresh the article list in your browser. Tada!

### Sass and Compass

Sass makes CSS fun again. Sass is an extension of CSS3, adding nested rules, variables, mixins, selector inheritance, and more. Compass is an open-source CSS Authoring Framework. Compass takes the CSS supplied Blueprint and the other supported frameworks and transforms them into Sass files. These will make writing CSS and working with CSS much, much easier.

-   All the details about Sass can be found here: [http://Sass-lang.com/](http://Sass-lang.com/)
-   All the details about Compass can be found here: [http://compass-style.org/](http://compass-style.org/)
-   Site to convert CSS to Sass: [http://css2sass.heroku.com/](http://css2sass.heroku.com/)

### Installing Sass and Compass

First, you will need to grab and install Ruby. And under [Ruby on Windows](http://www.ruby-lang.org/en/downloads/) said our first option was using the [RubyInstaller](http://rubyinstaller.org/downloads/) so I went there and downloaded 'Ruby 1.8.7-p302' under RubyInstallers. After installing, open the Ruby command prompt under Start and then check to see if Ruby is there by this instruction: 'ruby -version'

Step 1 done. Now install Compass. With the Command Prompt still open type: 'gem install compass'

These steps will only be done once on each system you want to use Compass and Sass. Also since Compass will generate our CSS, we don't need to install this on environments like Production.

Now for the big finale:

1) We will setup a configuration file
2) Install Blueprint for a project, and
3) Start a watcher (this will automate the compiling of our css).

Here are the steps in detail:

1) In the webroot create a folder called Sass and create a file called 'config.rb' in it. Paste in this code in the 'config.rb' file:

```ruby
http_path = '/'
Sass_dir = 'src'
css_dir = '../stylesheets'
images_dir = '../images'
javascripts_dir = '../javascripts'
http_stylesheets_path = 'stylesheets'
http_javascripts_path = 'javascripts'
http_images_path = 'images'
environment = :development
output_style = :compressed
```

The lines you may need to modify are the css_dir, images_dir, and javascript_dir to match your folder structure. Notice it is a relative path from the location of config.rb file http_stylesheets_path, http_javascripts_path, and http_images_path are the url path to these folders.

In the Command Prompt type in the path to your webroot and the Sass folder you created like "C:\\JRun4\\servers\\cfwheels\\cfusion.ear\\cfusion.war\\Sass".

Type in and run: 'compass install blueprint/basic'

Drop down one directory in your command prompt, so you are in your webroot and type in and run: 'compass watch Sass'

Check your editor, you will need to refresh in Eclipse. You should see a '/Sass/src/' folder in your Sass folder. This is where you will be creating the Sass for your CSS. Also created are some compiled CSS files in your '/stylesheet/' location. Lets first check out the 'scss' files used to generate the stylesheets then the actual CSS.

In '/Sass/src/' you'll see a 'partials' folder. This is exactly like Wheels partials with even the naming convention beginning with underscore '_'.

### A Few Sass Examples

Were not focusing on CSS development, so here are a few styles you can copy & paste and modify to your heart's content:

```sass
$bgColor: #AAA;
$fgColor: #0000FF;
$navbar-color: #ce4dd6;

body {  
	background-color: $bgColor;  
	color: $fgColor;  
	font-size: 14px;   
	font-family: Verdana, Helvetica, Arial;  
	line-height: 24px;
}
	
select, input, textarea {  
	color: $fgColor;
}
	
label { 
	color: darken($bgColor,20);
	font-weight: bold;
}

table { 
	border-collapse:collapse; border-spacing:0; 
}

label, input[type=button], input[type=submit], button { 
	cursor: pointer; 
	margin-top: 15px; 
}
button, input, select, textarea { 
	margin: 0; 
}

a {  
	color: lighten($fgColor,10);  
	
	&:hover { 
		color: darken($fgColor,10);
		text-decoration: none;
	} 
	
	&:visited {
		color: darken($fgColor,10);
	}
}

h1, h2, h3, h4, h5 {  
	color: darken($bgColor,20);  
	padding-bottom: 10px;
}
```

But our application isn't setup to load that stylesheet yet. We need to make a change to our view templates.

### Working with Layouts

We've created about a dozen view templates between our different models. We *could* go into each of those templates and add a line like this at the top:

```cfm
#styleSheetLinkTag("screen,ie,style")#
#styleSheetLinkTag(source="print", media="print")#
```

The 'styleSheetLinkTag' would find the Sass file we just wrote. That's a lame job, imagine if we had 100 view templates. What if we want to change the name of the stylesheet later? Ugh.

Wheels and ColdFusion both emphasize the idea of D.R.Y. 'Don't Repeat Yourself'. In the area of view templates, we can achieve this by creating a **layout**. A layout is a special view template that wraps other views. Look in your navigation pane for '/views/layout.cfm', open that file. In this layout we'll put the view code that we want to render for every view template in the application. This code will go between the 'head' tags.

```cfm
#styleSheetLinkTag("screen,ie,style")#
#styleSheetLinkTag(source="print", media="print")#
```

Now refresh your article listing page and you should see the styles take effect. Whatever code is in the individual view template gets inserted into the layout where you see the 'includeContent()'. Using layouts makes it easy to add site-wide elements like navigation, sidebars, and so forth.