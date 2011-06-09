## I3: Tagging

In this iteration we’ll add the ability to tag articles for  
organization and navigation. Tagging is similar to some blogs  
having a “Posted Under” feature. For example, if I wrote a blog  
post about our training, I might tag it as “Wheels”, “Training”,  
and “ColdFusion”.

First we need to think more about what a tag is and how it’ll  
relate to the Article model. If you’re not familiar with tags,  
they’re commonly used in blogs to assign the article to one or more  
categories. For instance, if I write an article about a feature in  
ColdFusion on Wheels, I might want it tagged with all of these  
categories: ColdFusion, Wheels, programming, and features. That way  
if one of my readers is looking for more articles about one of  
those topics they can click on the tag and see a list of my  
articles with that tag.

### Understanding the Relationship

What is a tag? We need to figure that out before we can create the  
model. First, a tag must have a relationship to an article so they  
can be connected. A single tag, like “ColdFusion” for instance,  
should be able to relate to **many** articles. On the other side of  
the relationship, the article might have multiple tags (like  
“ColdFusion”, “Wheels”, and “programming” as above) - so it’s also  
a **many** relationship. Articles and tags have a **many-to-many**  
relationship.

Many-to-many relationships are tricky because we’re using an SQL  
database. If an Article “has many” tags, then we would put the  
foreign key `articleid` inside the `tags` table - so then a Tag  
would “belong to” an Article. But a tag can connect to **many**  
articles, not just one. We can’t model this relationship with just  
the `articles` and `tags` tables.

When we start thinking about the database modeling, there are a few  
ways to achieve this setup. One way is to create a “join table”  
that just tracks which tags are connected to which articles.  
Traditionally this table would be named `articlestags` and Rails  
would express the relationships by saying the Article model  
`hasAndBelongsToMany` Tags, while the Tag model  
`hasAndBelongsToMany` Articles but Wheels doesn’t have this  
relationship yet.

Even if Wheels did have `hasAndBelongsToMany`, most of the time  
this isn’t the best way to really model the relationship. The  
connection between the two models usually has value of its own, so  
we should promote it to a real model. For our purposes, we’ll  
introduce a model named “Tagging” which is the connection between  
Articles and Tags. The relationships will setup like this:

- An Article `hasMany` Taggings  
- A Tag `hasMany` Taggings  
- A Tagging `belongsTo` an Article and `belongsTo` a Tag

### Making Models

With those relationships in mind, let’s design the new models:

- Tag **** `name` A string  
- Tagging  
 **** `tagid` Integer holding the foreign key of the related Tag****  
 `articleid` Integer holding the foreign key of the related Article

Note there are no changes necessary to Article because the foreign  
key is stored in the Tagging model. So now lets generate these two  
models with the DBMigrations plugin:

The tags migration should look like this:

```cfm

<cfcomponent extends="plugins.dbmigrate.Migration" hint="creates tags tables">  
 <cffunction name="up" >  
 <cfscript >  
 t = createTable (‘tags’);  
 t = t.string (‘name’);  
 t.timestamps ();  
 t.create ();  
 </cfscript >  
 </cffunction >  
 <cffunction name="down" >  
 <cfscript >  
 dropTable (‘tags’);  
 </cfscript >  
 </cffunction >  
</cfcomponent >

```

The taggings migration should look like this:

```cfm

<cfcomponent extends="plugins.dbmigrate.Migration" hint="creates taggings table">  
 <cffunction name="up">  
 <cfscript>  
 t = createTable (‘taggings’);  
 t.integer (‘tagid’);  
 t.integer (‘articleid’);  
 t.create ();  
 </cfscript>  
 </cffunction>  
 <cffunction name="down">  
 <cfscript>  
 dropTable (‘taggings’);  
 </cfscript>  
 </cffunction>  
</cfcomponent>

```

Now run both migrations. Currently DBMigrations plugin doesn’t have  
a command for composite keys, so we will have to run this sql  
command seperately.

```sql

ALTER TABLE jsbloggers.taggings ADD PRIMARY KEY (tagid,articleid);

```

### Expressing Relationships

Now that we created our tables we will need to create our models  
and tell Wheels about the relationships between them. For each of  
the files below, create in if not already then add these lines:

In `/models/Article.cfc`

```cfm

<cfset hasMany("taggings") />

```

Then in `/models/Tagging.cfc`

```cfm

<cfset belongsTo("article") />  
<cfset belongsTo("tag") />

```

In `/models/Tag.cfc`

```cfm

<cfset hasMany("taggings") />

```

Now we can get the many-to-many association that we set up above.  
Here is how we will include the related tables from the tagging:

```cfm

<cfset data = model("tagging").findAll(include="article,tag",where="articleid=3") />

```

After Wheels had been around for awhile, developers were finding  
this kind of relationship very common. In practical usage, if I had  
an object named `article` and I wanted to find its Tags, I’d have  
to run code like this:

```cfm

<cfset article = model("article").findByKey(key=3,include="taggings") />  
 <cfdump var="#article.taggings#">  
 <cfabort>

```

That’s a pain since we don’t have `tags` yet. The solution was to  
augment the relationship with “shortcut”. We’ll update our  
relationship now to the Article and Tag classes:

In `/models/article.cfc`

```cfm

<cfset hasMany(name="taggings",shortcut="tags") />

```

In `/models/tag.cfc`

```cfm

<cfset hasMany(name="taggings",shortcut="articles") />

```

Now we can get the many-to-many association that we set up above.  
How we will include the related tables from the tagging is still  
the same `<cfset article =
model("article").findByKey(key=3,include="taggings") /\>`.

Now if we have an object like `article` we can just ask for  
`article.tags()` or, conversely, if we have an object named `tag`  
we can ask for `tag.articles()`.

Lets create an example to test our relationship with the  
[shortcut]()

```cfm

<cffunction name="six">  
 <cfset article = model("article").findByKey(key=3,include="taggings") />  
 <cfdump var="#article.tags()#">

<cfset tag = model("tag").findByKey(key=1,include="taggings") />  
 <cfdump var="#tag.articles()#">  
 <cfabort>  
</cffunction>

```

You should see in the articles with their tags in the first dump  
and tags with their articles.

### An Interface for Tagging Articles

The first interface we’re interested in is within the article  
itself. When I write an article, I want to have a text box where I  
can enter a new tag. When I save the article, my app should  
associate my article with the tags with the tag, creating them if  
necessary.

Adding the text field will take place in the file  
`/views/articles/\_form.cfm`. Add this code underneath the body  
text area like this:

```cfm

<h3>
Tag this post under:

</h3>
#textFieldTag (name=‘newTag’, label=‘New Tag’)#

```

This is the first time we used `textFieldTag`, it builds and  
returns a string containing a text field form control based on the  
supplied name. An Article doesn’t have a property named `newTag` —  
we made it up. In order for us to add a new tag, we need to add a  
method to the `article.cfc` file like this:

```cfm

<cffunction name="newTag">  
 <cfif len(trim(params.newTag))>  
 <cfset Tag = model("Tag").findOne(tagid=Tag.id,articleid=article.id)/>

<cfif not isObject(Tag)>  
 <cfset Tag = model("Tag").create(name=params.newTag)/>  
 </cfif>

<cfset Tagging = model("Tagging").create(tagid=Tag.id,articleid=article.id) />  
 </cfif>  
</cffunction>

```

Your form should now show up and there’s a text box at the bottom  
named “New Tag”. Enter content for a new article and in the tag  
enter `ColdFusion`. Click SAVE and you’ll get a successful  
message.

### Not So Fast

Did it really work? It’s hard to tell. Let’s jump into the Examples  
and have a look. Replace the key with the article, you added the  
tag to.

```cfm

<cffunction name="seven">  
 <cfset a = model("article").findByKey(key=3,include="taggings") />  
 <cfdump var="#a.tags()#" />  
 <cfabort>  
 </cffunction>

```

I bet the Examples Seven reported that `a.tags` had `0` tags — an  
empty query. So we didn’t generate an error, but we didn’t create  
any tags either.

We will need to call our `newTag` method within our `create` and  
`update` actions by adding `<cfset newTag()/\>`.

```cfm

<cffunction name="create">  
 <cfset article = model("article").create(params.article)/>

<cfset newTag()/>

<cfset flashInsert(message="Article '#article.title#' was created.")/>  
 <cfset redirectTo(action="index")>  
</cffunction>

<cffunction name="update">  
 <cfset article = model("Article").findByKey(params.article.id)/>  
 <cfset article.update(params.article)/>

<cfset newTag()/>

<cfset redirectTo(action="index")/>  
</cffunction>

```

Now try adding the tag again and verify it is present in Examples  
Seven. And you’ll see the Tag is associated with the Article.

### Adding Tags to our Display

According to our work in the Examples, articles can now have tags,  
but we haven’t done anything to display them in the article pages.  
Let’s start with `/views/articles/show.cfm`. Right below the line  
that displays the `article.title`, add this line:

```cfm

<h3>
Tags

</h3>
<cfif article.hasTaggings() EQ "YES">  
 <cfloop query="tags">  
 #linkTo (text=tags.name, controller=“tags”, action=“show”, key=tags.id)#<br>  
 </cfloop>  
 <cfelse>  
 None  
 </cfif>

```

These lines should loop and output through all the `tags` for the  
article.

Refresh your view and…BOOM:

```cfm  
Attribute validation error for tag cfloop.

The value of the attribute query, which is currently tags, is invalid.  
```

The `show` page is trying to use `tags`, but the action doesn’t  
know anything about our Tags variable. We need to create the `tags`  
vaiable in the `show` action of the `/controllers/Article.cfc`.  
Open up `/controllers/Article.cfc` and add this code `<cfset tags =
article.tags(order="name")/\>` inside the `show` method.

Now refresh your view and you should see your tags showing up on  
the individual article pages.

### Showing Tags in the New And Edit

We need an easy way to display already existing tags and allow  
selecting or deselecting them. Wheels has a `hasManyCheckBox` form  
helper. It is used as a shortcut to output the proper form elements  
for an association. First we need to add `<cfset tags =
model("Tag").findAll(order="name")/\>` to our `new` and `edit`  
actions in `/controllers/Article.cfc`. Then we can use the  
`hasManyCheckBox` in `views/articles/\_form.cfm` under the @  

<h3>
Tag this post under:  

</h3>
```` .

```cfm

<h3>Tag this post under:</h3>
<cfloop query="tags">
 #hasManyCheckBox(
  label=tags.name,
  objectName="article",
  association="taggings",
  keys="#tags.id#,#article.key()#"
 )#
</cfloop>

```

In our  ````new@ action, we need to mode the exisitn code slightly  
ending up with this:

```cfm

<cffunction name="new">  
 <cfset var newTagging = arrayNew(1)/>  
 <cfset tag = model("tag").new()/>  
 <cfset tags = model("Tag").findAll(order="name")/>  
 <cfset newTagging[1] = model("tagging").new()/>  
 <cfset article = model("article").new(taggings=newTagging)/>  
</cffunction>

```

The first line creates an array in our local scope. We did this  
since we don’t need the variable in the views. The next new line  
`<cfset newTagging[1] = model("tagging").new()/\>` uses that  
variable and creates a new Tagging in it. Then when creating our  
new article object we pass in the `newTaggings` array as taggings.  
Let’s dump the `article` and abort the process when creating a new  
article to see this object. Be sure to remove the dump and abort  
after examining the output.

Create an Article with some tags and then edit it. You should see  
all the existing tags and any already associated with the article  
when editing.

### Avoiding Repeated Tags

Lets Tag an acticle with “Glee Club”. Then lets update the Article  
and you’ll see “Glee Club” checked. What happens if we type in  
“Glee Club” again in the “New Tag” input. What is happenning? How  
would we solve this?

It prevents duplicates and allows you to remove tags from the edit  
form. Test it out and make sure things are working!

### Listing Articles by Tag

The links for our tags are showing up, but if you click on them  
you’ll get our old friend, the “Wheels.ViewNotFound”. Createyour  
`/controllers/Tags.cfc` and add a `show` method like this:

```cfm

<cfcomponent extends="Controller" output="false">

<cffunction name="show">  
 <cfset tags = model("tag").findByKey(key=params.key,include="taggings") />  
 <cfset articles = tags.articles() />  
</cffunction>

</cfcomponent>

```

Then create a file `/views/tags/show.cfm` like this:

```cfm

<cfoutput>  

<h1>
Articles Tagged with #tags.name#

</h1>
<ul>
<cfloop query="articles" >  

<li>
#linkTo (text=articles.title, controller=“articles”, action=“show”, key=articles.articleid)#<br>

</li>
</cfloop>  

</ul>
</cfoutput>

```

Refresh your view and you should see a list of articles with that  
tag. Keep in mind that there might be some abnormalities from  
articles we tagged. For any article with issues, try going to it’s  
`edit` screen, saving it, and things should be fixed up. If you  
wanted to clear out all taggings you could do `<cfset
model("Tagging").deleteAll()/\>` from your Examples.

### Listing All Tags

We’ve built the `show` action, but the reader should also be able  
to browse the tags available at `http://wheels.local/tags/`. I  
think you can do this on your own. Create an `index` action in your  
`tags.cfc` and an `index.cfm` in the corresponding views folder.  
Look at your `Articles.cfc` and Article `index.cfm` if you need  
some clues.

If that’s easy, try creating a `delete` method in your `tags.cfc`  
and adding a delete link to the tag list. If you do this, change  
the association in your `tag.cfc` so that it says `hasMany
:taggings, :dependent =\> :delete`. That’ll prevent orphaned  
Tagging objects from hanging around.

With that, a long Iteration 3 is complete!
