## I2.1: Adding Comments

Most blogs allow the reader to interact with the content by posting
comments. Let's add some simple comment functionality.

### Designing the Comment Model

First, we need to brainstorm what a comment *is*...what kinds of data
does it have...

-   It's attached to an article
-   It has an author name
-   It usually has an author email address
-   It usually has an optional URL
-   It has a body

With that understanding, let's create a Comment model. Switch over to
Eclipse and right click on the models folder, then select New --\> File
and enter "Comment.cfc" for File name. The file will be created and
automatically open, then you can add this code to it:

~~~~ {lang="cfm"}
 

~~~~

Pretty simple, huh?

### Setting up the Migration

Open the **DBMigrate** plugin and fill it out with the following and
click "create":

-   Select template? Create table
-   Migration description: creates comments table

Open the file that the generator created,
`/db/migrate/some-timestamp_creates_comments_table.cfc`. Inside the
`self.up` you need to add one line for each of the pieces of data we
just brainstormed. It'll start off with these...

~~~~ {lang="cfm"}
t.integer('articleid');t.string('authorname');
~~~~

Then keep adding lines creating strings named `authoremail`,
`authorurl`, and a text field named `body`.

Once that's complete, go to the **DBMigrate** plugin and run the
migration by clicking `go`.

### Relationships

The power of SQL databases is the ability to express relationships
between elements of data. We can join together the information about an
order with the information about a customer. Or in our case here, join
together an article (in the articles table) with its comments (in the
comments table). We do this by using foreign keys.

Foreign keys are a way of marking one-to-one and one-to-many
relationships. An article might have zero, five, or one hundred
comments. But a comment only belongs to one article. These objects have
a one-to-many relationship - one article connects to many comments.

Part of the big deal with Wheels is it makes working with these
relationships very easy. When we created the migration for comments we
started with an `integer` field named `articleid`. The Wheels convention
is, for a one-to-many relationship, the objects on the "many" end should
have a foreign key referencing the "one" object. And the foreign key
should be titled with the name of the "one" object, then "id". So in
this case one article has many comments, so each comment has a field
named `articleid` which tracks which article they belong to. Similarly,
a store's customer might have many orders, so each order would have a
`customerid` specifying which customer they belong to.

Following this convention will get us a lot of functionality "for free."
Open your `/models/comment.cfc` and add the middle line so it looks like
this:

~~~~ {lang="cfm"}
  <cfset belongsTo("article") />
 
~~~~

A comment relates to a single article, it "belongs to" an article. We
then want to declare the other side of the relationship inside
`/models/article.cfc` like this:

~~~~ {lang="cfm"}
 <cfset hasMany("comments") />
~~~~

Wheels now know an article "has many" comments, and a comment "belongs
to" an article. We have explained to Wheels these objects have a
one-to-many relationship.

### Testing in Examples

Let's use the `/controllers/Examples.cfc` to test how this relationship
works in code. Open the `Examples.cfc` file and paste in the following
instructions and observe the output at at
[http://wheels.local/index.cfm/Examples/three](http://wheels.local/index.cfm/Examples/three)
:

~~~~ {lang="cfm"}
 <cfset a = model("article").findByKey(key=3,  include="comments") />  <cfdump var="#model("comment").new()#" />  
~~~~

When you called the `comments` method on object `a`, it gave you back a
blank array because that article doesn't have any comments. When you
executed `model("comment").new()` it gave you back a blank Comment
object. But, if you look closely, when you did `a.newComment()` the
comment object you got back wasn't quite blank - it has the `articleid`
field already filled in with the ID number of article `a`.

Try creating a few comments for that article like this:

~~~~ {lang="cfm"}
 <cfset a = model("article").findByKey(key=3,include="comments") /><cfset c = a.newComment() />
 <cfset c.authorname = "Daffy Duck" />
 <cfset c.authorurl = "http://daffyduck.com" />
 <cfset c.body = "I think this article is thhh-thhh-thupid!" />
 <cfset c.save() /><cfset d = a.createComment(authorname = "Chewbacca", body = "RAWR!") />
 

~~~~

For the first comment, `c`, I used a series of commands like we've done
before. For the second comment, `d`, I used the `create` method. When
you use `new` it doesn't go to the database until you call `save`. With
`create` you usually pass in the attributes then the object is created,
those attributes set, and the object saved to the database all in one
step.

Now you've created a few comments, try executing `a.comments` again.

~~~~ {lang="cfm"}
 <cfset a = model("article").findByKey(key=3,include="comments") />  
~~~~

Did your comments all show up? Great. Now we need to integrate them into
the article display.

## I2.1: Adding Comments (continued)

### Displaying Comments for an Article

We want to display any comments underneath their parent article. Because
we've setup the relationships between those models, this is very easy.
Open `/views/articles/show.cfm` and add the following lines right before
the link to the articles list:

~~~~ {lang="cfm"}
Comments#includePartial(article.comments)#
~~~~

This says we want to render a partial named "comment" and we want to do
it once for each element in the collection `article.comments`. We saw in
the Examples that when we call the `comments` method on an article we'll
get back an array of it's associated comment objects. So this render
line will pass each element of that array one at a time into the partial
named "comment". Now we need to create the partial
`/views/articles/_comment.cfm` and add this code:

~~~~ {lang="cfm"}
  Comment by #arguments.comment.authorname#  #arguments.comment.body#
~~~~

With that in place, try clicking on the articles and find the one where
you created the comments. Did they show up? What happens when an article
doesn't have any comments?

### Web-Based Comment Creation

Good start, but our users (hopefully) can't get into the Examples code
to create their comments. We'll need to create a web interface. We'll go
through some of the same steps we did when creating the web interface
for creating articles.

Let's start with the form. The comment form should be embedded into the
article's `show` template. So let's add this code right above the "<<
Back to Articles List" in the articles `show.cfm`

~~~~ {lang="cfm"}
#includePartial("comment_form")#
~~~~

Obviously this is expecting a file `/views/articles/_comment_form.cfm`,
so create the file and add this content for now:

~~~~ {lang="cfm"}
Post a Comment(Comment form will go here)
~~~~

Look at an article in your browser to make sure the partial is showing
up. Then we can start figuring out the details of the form.

Ok, now look at your `Articles.cfc` in the `new` method. Remember how we
had to create a blank Article object so Wheels could figure out which
fields an article has? We need to do the same thing before we create a
form for the comment. But when we view the article and display the
comment form we're not running the article's `new` method, we're running
the `show` method. So we'll need to create a blank Comment object inside
that `show` method like this:

~~~~ {lang="cfm"}
<cfset comment = article.newComments() />
~~~~

This is just like we did it in the Examples. Now we can create a form
inside our `_comment_form.cfm` partial like this:

~~~~ {lang="cfm"}
Post a Comment #errorMessagesFor("comment")# #startFormTag(controller="comments",action="create")#  #hiddenField(objectName='comment', property='articleid')#  #textField(objectName='comment', property='authorname')#  #textField(objectName='comment', property='authoremail')#  #textField(objectName='comment', property='authorurl')#  #textArea(objectName='comment', property='body')#  #submitTag(value="Create Comment")# #endFormTag()#
~~~~

The only new thing here is the hidden field helper. This hidden field
will hold the ID of the article to help when creating the comment
object.

Save then refresh in your web browser and... well... you'll get an error
like this:

~~~~ {lang="cfm"}
Wheels.ViewNotFoundCould not find the view page for the create action in the Comments controller.Suggested actionCreate a file named create.cfm in the views/comments directory (create the directory as well if it doesn't already exist).
~~~~

The `startFormTag` helper is trying to build the form so that it submits
to `/comments/create`, but we haven't created the Comments controller
yet.

### Creating a Comments Controller

Just like we needed an `Articles.cfc` to manipulate our Articles, we'll
need a `Comments.cfc` to manipulate our Comments. Create it to add this
code:

~~~~ {lang="cfm"}
 
 
 
~~~~

The first action we're interested in first is `create`. You can cheat by
looking at the `create` method in your `Articles.cfc`. For your
`Comments.cfc`, everything should be the same just replace article with
comment. Then the `redirectTo` is a little different, use this:

~~~~ {lang="cfm"}
<cfset redirectTo(controller="article",action="index")>
~~~~

Test out your form to create another comment now - and it should work!

#### Comment Validation

During creating this training, I noticed a "bug" with the `renderPage`
and `#includePartial(article.comments)#` combination. It seems
`includePartial` used this way doesn't allow another controller to be
called so, I copied `/views/articles/_comment.cfm` to `/views/comments`.
This allows error messages to appear along with the form being populated
again. Another fix could be to do a `redirectTo`. I also had to change
the `redirectTo(back=true)` to
`<cfset redirectTo(controller="articles",action="show",key=comment.articleid`
this way if an error happens, then a successful comment, we get back to
the appropriate place.

### Cleaning Up

We've got some decent comment functionality, but there are a few things
we should add and tweak.

#### Validation for Articles and Comments

TODO: add section for validation of articles and comments

#### Form Labels

The comments form looks a little silly will all the inputs on one line
and with "Authorname" and "Authorurl" and such. It should probably say
"Your Name" and "Your URL (optional)", right? To change the text that
the label helper prints out, you just pass in the desired text as a
second parameter, like this:

~~~~ {lang="cfm"}
#textArea(objectName='comment', property='authorname', label='Your Name')#
~~~~

Change your `_comment_form.cfm` so it prints out "Your Name", "Your
Email Address", "Your URL (optional)", and "Your Comment". Then refresh
the page and look at the html generated. It should look like this:

~~~~ {lang="cfm"}
Your Name
~~~~

Nicer, shouldn't the label be around "Author Name"? And we are still
missing the wrapping `<p>` tag along with a `<br>` between the label and
input. To accomplish this we will set some defaults for a our helper
functions in `config/settings.cfm`. Open the file and add these lines:

~~~~ {lang="cfm"}
<cfset set(functionName="textField", labelPlacement='before', prependToLabel="", prepend="", append="") /><cfset set(functionName="textArea", labelPlacement='before', prependToLabel="", prepend="", append="") />
~~~~

We are saying place the label before the input, and prepend `<p>` to the
label, then prepend `<br>` to the input, and finally append `</p>` to
the input. Confused? Hopefully, not :-)

Refresh, and you should see html source like this:

~~~~ {lang="cfm"}
 Authorname 
~~~~

#### Comments Count

Let's make it so where the view template has the "Comments" header it
displays how many comments there are, like "Comments ( 3 )". Open up
your article's `show.cfm` and change the comments header so it looks
like this:

~~~~ {lang="cfm"}
Comments ( #article.commentCount()# )
~~~~

#### Add Timestamp to the Comment Display

We should add something about when the comment was posted. Wheels has a
really neat helper named `distanceOfTimeInWords` which takes two dates
and creates a text description of their difference like "32 minutes
later", "3 months later", and so on. You can use it in your
`_comment.cfm` partial like this:

~~~~ {lang="cfm"}
Posted #distanceOfTimeInWords(arguments.comment.createdat, now())# later
~~~~

With that, you're done with I2!
