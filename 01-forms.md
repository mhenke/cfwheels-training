## I1: Form-Based Workflow

We've created an articles page, but it isn't a viable long-term solution. The users of our app will expect to add content through a web interface. In this iteration we'll create an "HTML" form to submit the article, then all the backend processing to get it into the database.

### Creating the New Action and View

Previously we setup the ```<cfset addRoute(name="home", pattern="", controller="Articles", action="index") />``` route in _/config/routes.cfm_, and told Wheels we were going to follow the RESTful conventions for this model named Article. Following this convention, the URL for creating a new article would be "http://localhost:8301/index.cfm/articles/new". Enter the url into your browser and see what comes up.

```cfm  
Wheels.ViewNotFound  
Could not find the view page for the new action in the Articles controller.

Suggested action

Create a file named new.cfm in the views/articles directory (create the directory as well if it doesn't already exist).  
```

This is an error message we've seen before. The router went looking for an action named "new" inside the "Articles.cfc" and didn't find it.

So first let's create the action. Open _/controllers/Articles.cfc_ and add this method structure, making sure it's **inside** the "cfcomponent" instruction, but **outside** the existing _index_ action:

```cfm
<cffunction name="new">

</cffunction>
```

Create a new file _/views/Articles/new.cfm_ with these contents:

```cfm
<h1>
Create a New Article
</h1>
```

Refresh your browser and you should just see the heading "Create a New Article".

### Writing a Form

It's not very impressive so far -- we need to add a form to the "new.cfm" so the user can enter in the article title and body.

Because we're following the RESTful conventions, Wheels can take care of many of the details. Inside the "new.cfm", enter this code below your header:

```cfm
<cfoutput>  
 #errorMessagesFor("article")#  
 #startFormTag(action="create")#  
 #textField(objectName="article", property="title", label="Title")#  
 #textArea(objectName="article", property="body", label="Body")#  
 #submitTag()#  
 #endFormTag()#  
</cfoutput>
```

What is all that? Let's look at it piece by piece:

* "errorMessagesFor" is a Wheels helper instruction which builds and returns a list containing all the error messages(if any).
* "startFormTag" is a Wheels helper instruction which takes an action parameter, in this case "create", The first line basically says "Make an opening form tag with the name of the action included in the URL."  
* The "textField" helper creates a single-line text field control

*The "objectName" parameter which says for this input use the object named "Article" to build the form control
*The "property" parameter create the name of the property to use in the form control 
*The "label" parameter creates an HTML label for the form control, this is good usability practice and will have some other benefits for us later  

* The "textArea" helper creates a text area field form control  
* The "submitTag" helper creates a submit button labeled "Save changes"  
* The "endFormTag" helper creates the ending form tag

Refresh your browser and you'll see this:

```cfm  
Wheels.ObjectNotFound  
Wheels tried to find the model object article for the form helper, but it does not exist.

Error location

Line 3 in viewsarticlesnew.cfm

2: <cfoutput>  
3: #errorMessagesFor("article")#  
4: #startFormTag(action="create")#  
5: #textField(objectName="article", property="title", label="Title")#  
6: #textArea(objectName="article", property="body", label="Body")#  
7: #submitTag()#  
```

What's Wheels trying to tell us? In our "new.cfm" on line #3 there was an error about "Wheels.ObjectNotFound". Somewhere in line #4 we're working with an object that doesn't exist.

And since there's only one object in line #3, it makes it pretty obvious -- the problem is we started talking about a thing named "Article" without ever creating that thing. Wheels uses some of the **reflection** techniques we talked about earlier in order to setup the form. Remember in the "Examples.cfc" when we called "Article.new()" to create a new object? We need to create the blank object for it.

Go into your "Articles.cfc", and **inside** the _new_ action, add this line:

```cfm
<cfset article = model("Article").new() />
```
Refresh your browser and your form should come up. Enter in a title, somebody text, and click "Save changes".

### The Create Action

You're old friend pops up again…

```cfm  
Wheels.ViewNotFound  
Could not find the view page for the create action in the Articles controller.

Suggested action

Create a file named create.cfm in the views/articles directory(create the directory as well if it doesn't already exist).  
```

When we loaded the form, we accessed the _new_ action, but when the form is submitted to the application, following the REST convention, it goes to a _create_ action. We set this _action_ in the **startFormTag** instruction. We need to create the action. Inside your _Articles.cfc_ add the _create_ action(again, **inside** the "cfcomponent" instruction, but **outside** the other actions):

```cfm
<cffunction name="create">  
 <cfset article = model("Article").new(params.article) />  
 <cfset article.save() />  
 <cfset redirectTo(action="index") />  
</cffunction>
```

This method says…

* Create an object named "Article" and send in the parameter "params.article"  
* The line ```<cfset article.save() />``` saves the object to the database, just like we did in the examples.  
* Finally, the "redirectTo" tells Wheels we don't want to follow the **conventions** and render a view for this action. Once the previous steps are done, we want to bounce to the list of all articles. The _renderPage(action="index")_ resolve to [http://localhost:8301/index.cfm/articles/index.](http://localhost:8301/index.cfm/articles/index.)

Go back in your browser so you get to the form with the sample data you entered and click "Save changes". You should then bounce to the full articles list with your new article added.

#### Params Structure

We snuck in a structure called _params_ in the _create_ action. Wheels simplifies the request handling for you using the _params_ structure. It combines the **url** and **form** scopes. This is something that most CFML frameworks do as well. 

Wheels makes the form data available inside the variable named "params". If we're to look at "params" as a data structure, it'd be a structure with only a key called "Article". The value of the pair is another structure with keys "title" and "body". The values for those keys are the data we entered into the text boxes on the form. So when "Article.new" is called and the structure "params.article" is passed in, the _new_ action looks for the value with key "title" and puts that into the Article's "title" attribute. Then it looks for the value for key "body" and puts that into the article's "body" attribute.  

### Adding Navigation to the Index

Right now our article list is very plain with only the Title showing. Let's add some links. Open your _/views/articles/index.cfm_ and…

Replace the title display, _#articles.title#_ with _#linkTo(text=title, action="show", key=id)#_

This _linkTo_ helper says…

* Create link with the text of the articles title which points to a page where we'll show just that article.
* the text attribute is what the link will display
* the action attribute is where the link will go
* the key attribute also tells the link to include another url parameter

* Add this code at the very bottom of  _/views/articles/index.cfm_:

```cfm
<cfoutput>  
<p>
#linkTo(text="Create a New Article", action="new")#
</p>
</cfoutput>
```

This code uses the Wheels _linkTo_ helper, it tells Wheels we want a link with the text "Create a New Article" that points to the action "new"(which the router handles for us).

Refresh your browser and you should now see a list of the article titles that are linked somewhere and a link at the bottom to "Create a New Article". Test the create link and it takes you to the new article form. Then go back to the article list and click one of the article titles.

### Creating the Show Action

Tired of this error message yet? Go to your "Articles.cfc" and add a method like this:

```cfm
<cffunction name="show">

</cffunction>
```

Let's pause here before creating the view template.

Look at the URL: "http://localhost:8301/index.cfm/articles/show/1". When we added the "linkTo" in the index and pointed it to the "show" for this "Article", the router created this URL. Following the RESTful convention, this URL goes to a **SHOW** action which would display the Article with ID number "1". Your URL might have a different number depending on which article title you clicked in the Article listing page.

**So what do we want to do when the user clicks an article title?**

Find the article, and then display a page with its title and body. We'll use the number on the end of the URL to find the article in the database. The router will send us this number in the variable "params.key". Inside the _show_ action that we just created, add this line:

```cfm
<cfset article = model("Article").findByKey(params.key) />
```

Now create the file _/views/articles/show.cfm_ and add this code:

```cfm
<cfoutput>  
<h2>#article.title#</h2>
<p>#article.body#</p>
#linkTo(text="<< Back to Articles List", controller="articles", action="index")#  
</cfoutput>
```

Refresh your browser and your article should show up along with a link back to the index.

### But You Never Make Mistakes!

We can create articles and we can display them, but when we eventually deliver this to less perfect people than us, they're going to make mistakes. Right now there's no way to edit an article once it's been created. There's also no way to remove an article.  

Let's add those functions.

Look at your "index.cfm" and replace the whole @<li>@ segment so it looks like this:

```cfm
<li>
<b>#linkTo(text=title, action="show", key=id)#</b><br/>  
 <i>Actions:  
 #linkTo(text="edit", action="edit", key=id)#,  
 #linkTo(text="remove", action="delete", key=id, confirm="Remove the article '#title#'?")#   
 </i>   
</i>
```

The first link we added, _for edit_, is pretty similar to what we've done before -- creating a link with the text "edit" pointing to the address "edit", which is defined by the router, and editing the key named **id**.

The second one is a little more complex. So this link will have the text "remove", will point to the "delete". We've also added a "confirm" parameter. If a link has a "confirm", then Wheels will generate some Javascript which will pop up a box when the link is clicked that contains the text in the "confirm". Here we're setting the message to check that the user wants to remove the article and including the article's title in the message.

Refresh your browser and you should see "edit" and "remove" links for each article. Click the EDIT link for your first article.

### Creating an Edit Action & View

The router is expecting to find an action in "Articles.cfc" named "edit", so let's add this:

```cfm
<cffunction name="edit">  
 <cfset article = model("Article").findByKey(params.key) />  
</cffunction>
```

All the _edit_ action is really going to do is find the article to be edited, and then display the editing form. If you refresh after adding that "edit" action you'll see the template missing error. 

Create a file "/views/articles/edit.cfm" but **hold on before you type anything**. Below is what the edit form should look like:

```cfm
<h1>
Edit an Article

</h1>
<cfoutput>  
 #errorMessagesFor("article")#  
 #startFormTag(action="update")#  
 #textField(objectName="article", property="title", label="Title")#  
 #textArea(objectName="article", property="body", label="Body")#  
 #submitTag(value="Update")#  
 #endFormTag()#  
</cfoutput>
```

In the ColdFusion and Wheels communities there is a mantra of "Don't Repeat Yourself" -- but that's exactly what I've done here.
  
This view is basically the same as the _new.cfm_ -- the only changes are the H1 and the submit button now has a value parameter. It will display in the button form control. We can abstract this form into a single file called a **partial**, then reference this partial from both _new.cfm_ and _edit.cfm_.

#### Creating an Partial

Create a file **/views/articles/_form.cfm** and, **yes**, it has to have the underscore at the beginning of the filename. Go into your _/views/articles/new.cfm_ and **CUT** all the text from and including the _cfoutput_ line all the way to its ending _cfoutput_. The only thing left will be your H1 line. Now go back to the **_form.cfm** and paste the form code. Change the "startFormTag" to ```#startFormTag(action=myaction)#```. We will also change the text on the "submit" button to say "Save" so it makes sense both when creating a new article and editing and existing one. This is done by adding _value="Save"_ to the _submitTag()_.

Now return to the _new.cfm_ and add the following code at the bottom of that view:

```cfm
<cfoutput>#includePartial("form")#</cfoutput>
```

Also for editing we need to have a hidden field for the article's "id". So please add this above the _submitTag_ in our partial.

```cfm
<cfif not article.isNew()>  
#hiddenFieldTag(name="key", value=article.id)#
</cfif>
```

Hold on, what is the **article.isNew()** and **hiddenFieldTag**? Well, our _article_ object has built in methods. The _isNew()_ returns true or false based on if the _article_ object is new. The _hiddenFieldTag_ is like the other help instructions but this one builds the html based on the supplied **name**. The other helper tags not ending with **Tag** build and return a string based on the supplied _objectName_ and _property_.

Then create your _edit.cfm_ file, copy and paste the _new.cfm_ into the _edit.cfm_, and change the h1 tag to "Edit an Article".

We will need to add ```<cfset myaction="create" />``` to the "new" action in the _/controllers/Articles.cfc_ along with in the "edit" action ```<cfset myaction="update" />```

Go back to your articles list and try creating a new article -- it should work just fine. Try editing an article and you should see the form with the existing article's data -- it works **OK** until you click _Save_.

The router is looking for an action named _update_. Just like the _new_ action sends its form data to the _create_ action, the _edit_ action sends its form data to the _update_ action. In fact, within our "Articles.cfc", the _update_ action will look very similar to _create_ action.

```cfm
<cffunction name="update">  
 <cfset article = model("Article").findByKey(params.key) />  
 <cfset article.update(params.article) />  
 <cfset redirectTo(action="show",key=params.key) />  
</cffunction>
```

The new bits here are the _update_ action and **key** in our _redirect_. The _update_ action works very similar to when we called the _new_ action and passed in the structure of form data. When we call **update** on the _Article_ object and pass in the data from the form, it changes the values in the object to match the values submitted with the form. And saves the object to the database and redirect to the articles list. The **key** in _redirectTo_ tells Wheels to show the same article being update after **updated**. 

Now try editing and saving some of your articles.

### Creating a Delete Action

Next, click the **REMOVE** link for an article and hit OK. You can see the router is expecting a "delete" action. Go into "Articles.cfc" and add a _delete_ action like this:

```cfm
<cffunction name="delete">  
 <cfset article = model("Article").findByKey(params.key) />  
 <cfset article.delete() />  
 <cfset redirectTo(action="index") />  
</cffunction>
```

Here we're doing a _findByKey_ based on _params.key_ like we did in the _show_ action. We call that object's _delete_ action, then redirect back to the articles list.

Try it out in your browser.

### Adding a Flash

It would be nice, though, if we gave the user some kind of status message about the operation that took place. When we create an article the message might say "Article "the-article-title" was created", or "Article "the-article-title" was removed" for the remove action. We can accomplish this with a special object called the "flash".

Wheels creates the object named **flash**, so we don't need to do anything to set it up. We can start by integrating it into our _index.cfm_ by adding this line at the very top:

```cfm
<div class="flash">
<p>
<cfoutput>#flash("message")#</cfoutput>

</p>
</div>
```
This outputs the value stored in the **flash** object with the key **message**. If you refresh your articles list you won't see anything because we haven't stored a message in there yet. Look at _Articles.cfc_ and add this line right after the _save_ in your _create_ action:

```cfm
<cfset flashInsert(message="Article '#article.title#' was created.") />
```

Then go to your articles list, create another sample article, and when you click create you should see the flash message at the top of your view.

Here's something cool about how Wheels handles the **flash** -- hit your browser's REFRESH button while looking at the articles list. See how the flash disappears? Once you display the message in a flash Wheels clears it out. That's why it's perfect for status messages like this.

Similarly, add a flash message into your _delete_ action and confirm it shows up when an article is removed. Then add one to your _update_ action and _show_ view that'll display when an article is edited.

And, finally, you're done with I1!