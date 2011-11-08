## I0: Up and Running

Part of the reason ColdFusion on Wheels became popular quickly is it takes a lot of the hard work off your hands, and it's especially true in starting up a project. Wheels practices the idea of "sensible defaults" and tries to, with a couple simple actions; create a working application ready for your customization.

### Setting the Stage

First we need to make sure everything is setup and installed. See the [Preparation for Wheels Projects](/resources/CFWheels-jumpstart/preparation/) page for instructions on setting up and verifying your ColdFusion, Wheels, and add-ons.

With that done, we need to create new project in Eclipse. Open Eclipse and…

* File --> New--> ColdFusion Project  
* Project Name: JSBlogger
* Uncheck Use Default Location in Project Location put "C:/JRun4/servers/cfwheels101" 
* Click Finish

Eclipse will then create a ColdFusion project for you and automatically open the project.

We need to change the Wheels datasource convention. Wheels assumes our datasource connection is the folder name Wheels resides in but for our case it is not "cfwheels101" but "JSBloggers". We could have easily named our ColdFusion server instance but I wanted to show how we can easily override a Wheels convention.

Press **Ctrl+Shift+R** while in Eclipse. This is the Open Resource window. I use this a lot when coding in Eclipse. Sometimes I highlight a file name in code and press **Ctrl+Shift+R**, other times I type in the file name like we will do next. Type in _settings.cfm_, and select "settings.cfm -- /JSBloggers/config". This will open that file for us. In the file at the bottom, type ```<cfset set(dataSourceName="JSBloggers") />``` to tell Wheels to use this as are datasource. Also add ```<cfset set(URLRewriting="Partial") />``` to tell Wheels to use Partial for URL Rewriting. Then reload Wheels and you should see under DataSource, JSBloggers. To reload Wheels, you can 1) add ```?reload=true``` to the url or 2) click on the "Reload" link in the Wheels debug section. Reloading refreshes any cached items.

Our blog will be centered on "articles" so we'll need a table in the database to store all the articles and a model to allow our Wheels app to work with that table.

### Working with the Database

Wheels can uses migration files to perform modifications to the database with the **DBMigrate** plugin. Almost any modification you can make to a DB can be done through a migration. The killer feature about Wheels migrations is they're generally database agnostic. When developing applications a person might use MySQL as we are in this tutorial, but when deploying to their production server it is running Oracle. Many others choose Microsoft SQL Server. It doesn't matter -- the same migrations will work on all of them! This is an example of how Wheels takes some of the painful work off your hands. You write your migrations once, and then run them against almost any database.

#### What is a migration?

Migrations are a convenient way for you to alter your database in a structured and organized manner. Migrations are meant to be **symmetric**. Whatever a migration changes inside the ```up``` method should be **undone** with the ```down``` method. Frequently in development you'll think you want the database to look one way, and then realize you need something different. You can create a migration to make your changes to the database, start building, and then revert as necessary.

Let's generate a "Create table" migration for our articles table. In the browser select, DBMigration link in the Wheels debugging section and open it in a new tab (This will make switching between plugins and our code easier). And complete the page like:

* Template: Create table  
* Migration prefix: Timestamp (e.g. 20110406075307)  
* Migration description: create articles table  
* Click create

On the page, you should see text like "The migration 20110406195353_create_articles_table.cfc file was created" showing the **create** was successful. Two new sections appear also on the page: **Migrate** and **Available migrations**. Under **Available migrations** you should see something like "20110406195353 **create_articles_table (create articles table)** not migrated". This is saying a migration script exists but hasn't been run.

Let's open _/db/migrate/(some\time*stamp)*\create\articles*table.cfc_ and take a look. You may need to refresh the project to see any new resources. Press **Ctrl+Shift+R** again, and type "*create" and select our newly created migration file. The astrick is a wildcard in the Resource view.First please note the filename is our comment with underlines instead of spaces and a timestamp of when the migration was created. Migrations need to be ordered, so the timestamp serves to keep them in chronologic order.  

Inside the file, you'll see two methods: ```up``` and ```down```.

```cfm
<cfcomponent extends="plugins.dbmigrate.Migration" hint="create articles table">  
 <cffunction name="up">  
 <cfscript>  
 t = createTable("[tableName]");  
 t.timestamps();  
 t.create();  
 </cfscript>  
 </cffunction>  
 <cffunction name="down">  
 <cfscript>  
 dropTable("[tableName]");  
 </cfscript>  
 </cffunction>  
</cfcomponent>
```

Inside the ```up``` method you'll see the generator has placed a call to the ```createTable``` method, passed "[tableName]" as a parameter, and created a block with the variable "t" referencing the table that's created. Let's change "[tableName]" to "[articles]". We can tell "t" what kind of columns we want in the "articles" table.

But first you might be wondering "**What is "t.timestamps" doing there?**". The generator inserted that line. It will create three columns inside our table titled **createdat** for when created, **updatedat** for last updated, and **deletedat** for soft deletes. Wheels will manage these columns for us, so when an article is created it's **createdat** and **updatedat** are automatically set. Each time we make a change to the article, the **updatedat** will automatically be …uhhh… updated. Very handy.

Well, what kind of fields does our Article need to have? Since migrations make it easy to add or change columns later, we don't need to think of **EVERYTHING** right now, we just need a few to get us rolling. Here's a starter set:

* title(a "string")  
* body(a "text")

So add these into your ```up``` so it looks like this:

```cfm
<cffunction name="up">  
 <cfscript>  
 t = createTable("articles");  
 t.string("title");  
 t.text("body");  
 t.timestamps();  
 t.create();  
 </cfscript>  
</cffunction>
```

That's it! You might be wondering, what is the "text" type? This is an example of relying on the Wheels database adapters to make the right call. For some databases, large text fields are stored as "varchar", while other's like Postgres use a "text" type. The database adapter will figure out the best choice for us depending on the configured database -- we don't have to worry about it.

Now our ```up``` migration is done. You might wonder, what about the ```down```? Didn't I say migrations need to be symmetric? If we added something to the ```up``` it is **generally** the case we need to undo the same change in the ```down```. However, when the migration is creating a table, the ```down``` can just drop that table regardless of what columns are inside of it. That's what the generator has setup for us here, where it says "dropTable("[tableName]");" replace "[tableName]" with "articles".

```cfm
<cffunction name="down">  
 <cfscript>  
 dropTable("articles");
 </cfscript>  
</cffunction>
```

Save that migration cfc file, switch over to your browser, and click "go" under the **Migrate** section:

The "All non-migrated" option finds all migrations in the _/db/migrate/_ folder, determines which migrations have and have not been run yet, and then runs them. Pretty much it says "**look in your set of cfcs for the database in _/db/migrate/_** and _run the migrations not yet ran_."

In this case we had one migration to run and it should print some output like this to your browser under **Migration Results**:

```cfm  
Migrating from 0 up to 20110406195353.

------ 20110406195353createarticles\table --------  
Created table articles  
```

The migration page tells you the plugin ran the migration. As I said before, **DBMigrate** keeps track of which migrations **have** and **have not** been run. Try running "All non-migrated" again now, and see what happens.

We've now created the "articles" table in the database and can start working on our **Article** model. We can view the **SQL** under **/db/sql**. We can see the new table exists and its structure in MySQL by running these command in the prompt:
* USE JSBloggers;
* SHOW TABLES;
* DESCRIBE articles;

### Working with a Model

Another awesome feature of working with Wheels is the **ORM**. **ORM** stands for Object-Relational Mapping. It allows you to map objects in your application to records in your database tables. This can simplify your development process and the **ORM** makes it very easy to do modifications, searches, and other data operations. Wheels automatically convert all the inputs we passed into to the ORM methods to _cfqueryparam_ for people interested in sql injection protection. So let's create a **Controller** to demonstrate some simple interactions with our **Model**. 

In the Eclipse menu:

* File --> New--> ColdFusion Component  
* Source Location: /JSBloggers/controllers 

* By clicking Browse…
* Component Name: Examples  
* Extends: Controller  
* Click Finish

Our new component will open automatically and we'll add some code in between the ```cfcomponent``` instructions. Let's try some experiments… To view our first experiment, we'll go to http://localhost:8301/index.cfm/examples/one. So type these instructions and observe the results when running the page:

```cfm
<cffunction name="one">  
 <cfset time = now() />  
 <cfdump var="#time#" />  
 <cfset articles = model("article").findAll() />  
 <cfdump var="#articles#" />  
 <cfset article = model("article").new() />  
 <cfdump var="#article#" />  
 <cfabort>  
</cffunction>
```

* The first line was to demonstrate we can do anything in our **Controller** we previously did during "CFML in 100 minutes". 
* The third line referenced the **Article** model and called the ```findAll``` method which returns a query of all articles in the database -- so far an empty result. 
* The fifth line created a new article object and returns it. The object is not saved to the database; it only exists in memory. Property names and values can be passed in either using named arguments or as a structure to the properties argument.

All the information about the **Article** model is in the file _/models/Article.cfc_, so let's open that now. Did you find it, hopefully not since it doesn't exist. We are following Wheels **conventions** so everything is **assumed** and it isn't needed yet. We'll create it next because eventually we'll need to define some model settings.

### Creating the Article Model

We'll create an **Article Model** to represent the **Articles** table. In Eclipse, right-click on the _models_ folder and select New --> File. In the file name input type _Article.cfc_ and press **Finish**. Type this in the blank file:

```cfm
<cfcomponent extends="Model" output="false">
	
</cfcomponent>
```

* _/models/Article.cfc_ : The file will hold the model code

Not very impressive, right? There are no attributes defined inside the model, so how does Wheels know an Article should have a "title", a "body", etc? It queries the database, looks at the articles table, and assumes whatever columns the table has should probably be the attributes accessible through the model.

#### Let's look at the Articles table

In the command prompt, run: _Show articles;_

You created most of the columns in your migration file, but what about the "id"? Every table you create with a migration will automatically have an "id" column which serves as the table's primary key. When you want to find a specific article, you'll look it up in the articles table by its unique ID number. Wheels and the database work together to make sure that these IDs are unique, usually using a special column type in the DB like "serial".

Back to the ```<cfset article = model("article").new() />``` instruction. in the controller. The ```new()``` method doesn't change values in the database until we explicitly call the ```save``` method on an object. 

 Let's create a sample article and you'll see how to use ```save``` method. In our previous example the **Article** object didn't have the attributes "id", "title", "body", "createdat", and "updatedat". Enter each of the following lines to add them:

```cfm
<cffunction name="two">  
 <cfset a.title = "Sample Article Title" />  
 <cfset a.body = "This is the text for my article, woo hoo!" />  
 <cfset article = model("article").new(a) />  
 <cfset article.save() />  
 <cfset articles = model("article").findAll(select="body",order="id",where="title='Sample Article Title'") />  
 <cfdump var="#articles#" />  
 <cfabort>  
</cffunction>
```

Now you'll see the ```findAll()``` command gave you back an query object holding the one article we created and saved. We added some parameters to the command. The _select_ determines how the SELECT clause for the query used to return data will look. The _order_ maps to the ORDER BY clause of the query. The _where_ argument maps to the WHERE clause of the query. You can review all the parameters on the Wheels website, cfwheels.org. http://cfwheels.org/docs/1-1/function/findAll

Go ahead and **create 3 more sample articles** in one request under a method called "moreSamples".

### Moving Towards a Web Interface -- Setting up the Route

We've created a few articles through the Examples controller, but we really don't have a web application until we have a better web interface. Let's get that started. We said "Wheels uses an **MVC** architecture", and we've worked with the Model, now we need a View and Controller.

When a Wheels application gets a request from a web browser, it first goes to the **router**. The router decides what the request is trying to do and what resources it is trying to interact with. The router dissects a request based on the address it is requesting. Let's open the router's configuration file, _/config/routes.cfm_.

Inside this file you'll see ```<cfset addRoute(name="home", pattern="", controller="wheels", action="wheels") />```. This is the default route and it directs to the Congratulations page, you saw when we first loaded our application. Let's replace it with:

```cfm
<cfset addRoute(name="home", pattern="", controller="Articles", action="index") />
```

This line tells Wheels to do a lot of work. It declares that we have a resource named "home" and the router should expect requests to follow the **RESTful** model of web interaction (**RE**presentational State Transfer). The details don't matter to you right now, but just know that when you make a request like "http://localhost:8301/" and the router will know you're looking for a **index** action of the **Articles** controller.

Now the router knows how to handle requests about articles, it needs a place to actually send those requests, the **Controller**.

### Creating the Articles Controller

We're going to use the Scaffold plugin again. In your browser, click the "Scaffold" link under Plugins in the Wheels debugging section.

Type **article** for the Object name and select **Controller** for Type. We'll leave the Template as default and click _Generate_. The page should refresh and "**File "controllers/Articles.cfc" created.**" should appear.

The output should show the generator created this file for you:

* _/controllers/Articles.cfc_: The controller file itself

Let's open up the controller file, _/controllers/Articles.cfc_. You'll see the **Scaffold** plugin generated a lot of code. It created these actions for us: "index", "show", "new", "edit", "create", "update", and "delete".

Any additional code we add to the controller must go **between** the beginning and ending ```cfcomponent``` instructions, outside any ```cffunction``` instructions.

We have a working controller for CRUD (create, read, update, and delete) but we didn't learn anything so let's comment out everything between the beginning and ending ```cfcomponent``` instructions. We can review this generated code later if you want.

### Defining the Index Action

The first action we want to talk about is the ```index```. This is what the app will send back when a user requests "http://localhost:8301/index.cfm/Articles/" -- following the RESTful conventions, this should be a list of the articles. So when the router sees this request come in, it tries to call the ```index``` action inside **Articles** controller. It goes to the ```index``` **action** which gets all our articles then renders the ```index``` **view**:

```cfm
<cffunction name="index">  
 <cfset articles = model("Article").findAll() /> 
 <cfset renderPage(action="index")>
</cffunction>
```

We can actually remove the renderPage instruction because Wheels **conventions** will **assume** the ```view ``` is the same name as the ```action```. We would use it if we wanted to **override** the assumption. Lets remove the _renderPage_ instruction.

#### Passing Action variables to Views

What scope is "articles"? Usually in a "cfc" we will "var" scope variables to stop data clashing issues. The ```var``` instruction marks the variable as a "local variable". We specifically want the list of articles to be accessible from both the controller and the view we're about to create. In order for "articles" variable to be visible in both places, it has to be in the variables scope. If we had named it "var articles", the local variable would only be available within the ```index``` action of the controller.

Let's load "http://localhost:8301/". You'll notice our updated default route is mapping the code to the ```index``` action of the **Articles** controller but we are getting an error since the view doesn't exist.

```cfm  
Wheels.ViewNotFound  
Could not find the view page for the index action in the Articles controller.

Suggested action

Create a file named index.cfm in the views/articles directory (create the directory as well if it doesn't already exist).  
```

### Creating the Index View

The error message is pretty helpful. It tells us Wheels is looking for a template in _/views/Articles/_ but it can't find one named _index.cfm_. Wheels has **assumed** our ```index``` action in the controller should have a corresponding _index.cfm_ view template in the views folder. We didn't have to put any code in the controller to tell it what view we wanted, Wheels figures it out.

Let's create that view template now. In the left pane of your Eclipse window, expand the "JSBloggers" project so you can see "views", then expand "views". Right-click on the "views" folder, select "New" then "Folder" and, in the popup, name the folder "articles". Next repeat the process, but select "File" and, in the popup, name the file _index.cfm_.

Now you're looking at a blank file. Enter in this view template code which is a mix of HTML and what are called CFML tags:

```cfm
<h1>
Listing articles

</h1>
<ul>
<cfoutput query="articles">  

<li>
<b>#articles.title#</b><br/> 
</li>
</cfoutput>  

</ul>
```

CFML is a templating language that allows us to mix CFML into our HTML. There are only a few things to know about CFML:

* An CFML clause starts with ```<cf..>``` and ends with ```</cf..>``` 
* If the clause started with ```<!---``` and ends with ```--->```, the result of the code in between will be commented out (not ran)
* If the clause started with ```<cfoutput>``` and ends with ```</cfoutput>```, the result of the CFML code will be output in place of the instructions

Save the file and refresh your web browser. You should see a listing of the articles you created in the Examples. We've got the start of a web application!