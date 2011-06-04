## I5: Authentication

Authentication is an important part of almost any web application and
there are several approache's to take. We will create one using a
`filter`. `Filters` tell Wheels to run a function before an action is
run or after an action has been run. You can also specify multiple
functions and actions.

### Creating a People table

We will need a place to store users for authentication so lets create a
migration for a people table. You should be an old hat at this. Here is
what the final migration for a People table should look like:

~~~~ {lang="cfm"}
          t = createTable('people');    t.string('firstname');    t.string('lastname');    t.string('email');    t.string('password');    t.timestamps();    t.create();   addRecord(table='people',id=1,email='admin@gmail.com',password='#Hash("admin")#');
    
  
  
    
    dropTable('people');
    
  

~~~~

### Creating the Person Model

We will create a Model for our users called `Person.cfc`. It will
validate several of our properties.

~~~~ {lang="cfm"}
 <!-- Validations --->
  <cfset validatesPresenceOf("firstName,lastName,email")>
  <cfset validatesFormatOf(property="email", type="email")>
  <cfset validatesLengthOf(property="firstName", maximum=30)>
  <cfset validatesLengthOf(property="lastName", maximum=50)>
  <cfset validatesConfirmationOf("password")>
~~~~

### Creating a Login view

Create a new folder and file `/views/main/login.cfm` with this code:

~~~~ {lang="cfm"}
Login<cfif isdefined("error")> #flash("error")##errorMessagesFor("user")##startFormTag(controller="main", action="signin")##textField(label="Email", objectName="user", property="email")#
 #passwordField(label="Password", objectName="user", property="password")#
  #submitTag(value="Sign In")#
 #endFormTag()#
~~~~

Nothing special here. Lets try to load our login page at
`http://wheels.local/index.cfm/main/login` If its there, you're ready to
go!

### Creating our Main Controller

We will make a simple controller with the following actions: `login`,
`logout`, and `signin`. The `signin` action creates a session structure
called user if the email and password is correct.

~~~~ {lang="cfm"}
  <cfset user = model("person").new()>
 
  <cfset StructDelete(session, "user")>
  <cfset redirectTo(controller="main", action="login")>
 
  <cfset user = model("person").findOne(where="email='#params.user.email#' AND password='#hash(params.user.password)#'")> <cfif IsObject(user)>
   <cfset session.user.id = user.id>
   <cfset redirectTo(controller="articles", action="index")>
  
   <cfset user = model("person").new(email=params.user.email)>
   <cfset flashInsert(error="The email and password that you entered is not valid.")>
   <cfset renderPage(action="login")>
  
 
~~~~

Now login with **admin@gmail.com** and **admin** and you should be
directed to our listing page.

### Securing Our Application

It looks like we can login, but we don't have a security check. Were
just going to use one layer of security for the appa person who is
logged in has access to all the commands and pages, while a person who
isn't logged in can only post comments, view articles, and try to login.

Open the `/controllers/Controller.cfc` and add this code between the
`cfcomponent` instructions:

~~~~ {lang="cfm"}
 <cfif StructKeyExists(session, "user")>  <cfset loggedInUser = model("person").findByKey(session.user.id)>   <cfset redirectTo(controller="main", action="login")> 
~~~~

You can add other functions to `/controllers/Controller.cfc` and make
them globally available in all your controllers like we did for
`checkLogin`.

Then we define an action to check if the person is logged in. If not, we
redirect to the login page. If true this filter create a variable called
LoggedInUser and then allow the requested action to be rendered.

The first thing we need to do is sprinkle `checkLogin` on most of our
controllers:

-   In `tags.cfc` , we don't have any actions that need to be protected.
-   In `comments.cfc` , we never implemented `index` and `delete` , but
    just in case we do lets allow unauthenticated users to only access
-   In `Articles.cfc` authentication should be required for `new` ,
    `create` , `edit` , `update` and `delete`. Figure out how to write
    the before filter using either `only` or `except`

Did you know how? If not here is the code to add the filters. In the
`/controllers/Comments.cfc`:

~~~~ {lang="cfm"}
      <cfset filters(through="checkLogin", except="create")>   
~~~~

Next we add our filter to the `/controllers/Articles.cfc` in the `init`
method.

~~~~ {lang="cfm"}
<cfset filters(through="checkLogin", only="new,create,edit,update,delete")>
~~~~

This will run our `checkLogin` action in `/controllers/Controller.cfc`
*before* these actions: `new`, `create`, `edit`, `update` and `delete`.

With that in place, try accessing `/article/new` when you logged in and
when your logged out.

Now our app is pretty secure, but we should hide all those edit, delete,
and new article links from unauthenticated users.

Open `/views/articles/index.cfm` and find the section where we output
the Actions. Wrap that whole section in an `if` clause like this:

~~~~ {lang="cfm"}
<cfif StructKeyExists(session, "user")>  Actions:    #linkTo(text='edit', action='edit', key=id)#,  #linkTo(text='remove', action='delete', key=id, confirm="Remove the article '#title#'?")#  
~~~~

Look at the article listing in your browser when you're logged out and
make sure those links disappear.

### Better Navigation

We still have to hide the Create a New Article link but the navigation
has been bothering me. Let's try to clean it up a little more. Remove
any cases of "<< Back to Articles List" and "Create a New Article". Then
add this code to `/views/layout.cfm`.

~~~~ {lang="cfm"}
  <cfif "show,edit,new,login" contains params.action >  #linkTo(text="<< Back to Articles List", controller="articles", action="index")#  <cfif StructKeyExists(session, "user")>  #linkTo(text="Create a New Article", action="new")#  #linkTo(text="Logout", controller="main", action="logout")# <cfelseif params.action NEQ "login">  #linkTo(text="Login", controller="main", action="login")# 
~~~~

If you look at the `show` view template, you'll see that we never added
an edit link! Let's quick add that link now, but protect it to only show
up when a user is logged in and they are on the `article` controller and
`show` action.

~~~~ {lang="cfm"}
<cfif params.controller EQ "articles" and params.action EQ "show">#linkTo(text='Edit Current Article', action='edit', key=params.key)#
~~~~

Your basic authentication is done, and Iteration 5 is complete!
