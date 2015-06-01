# Couchbase Connect Mobile Workshop Android

In this workshop, you will learn how to use Couchbase Lite along with Sync Gateway to build a ToDos app with a guest account mode, offline-first capabilities and syncing different ToDo lists to Sync Gateway.

This paper will guide you through the steps to build the application and know all the tips and tricks to building apps with a great look and feel using Couchbase Mobile.

## 30 minutes: Couchbase Mobile Presentation

	paragraph + link to download the slide deck
	what's the problem we're fixing
	what if you had to code that from scratch

## 30 minutes: Couchbase Lite in-depth

	paragraph + link to download the slide deck
	show all the steps
	explain what's a recycler view

## 90 minutes: Hands on Couchbase Lite

### Getting started

Let us get the ToDoLite [repository][1] cloned to your computer.  First open up Terminal all and run the commands below:  

## Get the code

```
$ git clone https://github.com/couchbaselabs/ToDoLite-Android.git
$ cd ToDoLite-Android
$ git submodule init && git submodule update
```

We also added the submodules required for our project and now we are ready to begin.  You should now be at the step below and see something like:
![][image-13]

The ToDoLite-Android folder now will contain the files below:
![][image-14]

Within Android Studio, click:  File>New>Import Project 
![][image-15]

Locate the ToDoLite-Android folder and import the project:  
![][image-16]

Run the app now to see Couchbase Lite working within the ToDoLite Android app.  Click on the green button to build the app.  
![][image-17]

Choose to launch the app on the device or using the Android emulator:
![][image-18]

When complete, you will have a mobile app that looks like below:
![][image-19]

This application has three main screens, the drawer to display the List, the Main screen to display the Tasks in a particular screen and finally the Share screen to share a List with other users.

![][image-1]

Every step of the tutorial are saved to a branch on the GitHub repository. If you find yourself in trouble and want to skip a step or catch up, you can just check out to the next branch. For example, to start checkout on `workshop/starter`:

	git checkout workshop/starter

In the source code, you will find comments to help locate where the missing code is meant to go. For example:

	// WORKSHOP STEP 1: missing method to save a new List doc

### Introduction

The topics below are the fundamental aspects of Couchbase Mobile. If you understand all of them and their purposes, you’ll be in a very good spot after reading this tutorial.

- Document: the primary entity stored in a database.
- Revision: with every change to a document, we get a new revision.
- View: persistent index of documents in a database, which you then query to find data.
- Query: the action of looking up results from a view’s index.
- Attachment: stores data associated with a document, but are not part of the document’s JSON object.

Throughout this tutorial, we will refer to the logs in LogCat to check that things are working as expected. You can filter logs on the `ToDoLite` Tag name and `com.couchbase.todolite` package name. Create a new Filter Configuration.

![][image-2]

### ToDoLite Data Model

In ToDoLite, there are 3 types of documents: a profile, a list and a task. The task document holds a reference to the list it belongs to and a list has an owner and a members array.

![][image-3]

### STEP 0: Create a database

Open `Application.java`, notice there is a property called database of type CBLDatabase. We will use this property throughout the application to get access to our database.

In `Application.java`:

- set up a new instance of the manager
- create a new database called name `todos`
- set the user id to your name
- use the Profile’s createNewProfile class method to create a new Profile document with the user you chose above
- log the properties of the Profile document to the Console

Launch the app and log the properties of the Profile document to LogCat:

![]()

### STEP 1: Working with HashMap\<String, Object\>

You will learn how to save documents and consequently revisions as well.

In Couchbase Lite a document’s body takes the form of a JSON object - a collection a key/value pairs where the values can be different types of data such as numbers, strings, arrays or even nested objects.

Open `document/List.java` and add the necessary code in the `createNewList` method to persist a List document to a Couchbase Lite database. Instantiate a new HashMap and save a few properties:
- `type` » the document type `list`.
- `title` » parameter that’s passed to the function.
- `created_at` » the `currentTimeString` variable.
- `members` » an empty `ArrayList`.

Create a new document using the `createDocument` method available on the [database][2] object.

With a new document, use the `putProperties` method passing in the HashMap. This method persists the document to the database.

Don’t forget to return the saved document.

Now let’s test this method is working as expected. Open `MainActivity.java` and navigate to the `createNewList` method, in the onClick listener call the `List.createNewList` method passing in the database, title and currentUserId.

Finally, add a log statement to check that the document was saved.

Run the app and create a couple lists. Nothing will display in the UI just yet but you see the Log statement you added above. In the next section, you will learn how to query those documents.

![][image-5]

The solution is on the `workshop/saving_list_document` branch.

### STEP 2: Creating Views

Couchbase views enable indexing and querying of data.

The main component of a view is its **map function**. This function is written in the same language as your app—most likely Objective-C or Java—so it’s very flexible. It takes a document's JSON as input, and emits (outputs) any number of key/value pairs to be indexed. The view generates a complete index by calling the map function on every document in the database, and adding each emitted key/value pair to the index, sorted by key.

You will find the `queryListsInDatabase` method in `List.java` and the objective is to add the missing code to index the List documents. The emit function will emit the List title as key and null as the value.

In pseudo code, the map function will look like:

	var type = document.type;
	if document.type == "list"
	    emit(document.title, null)

The solution is on the `workshop/create_views` branch.

### STEP 3: Query Views

A query is the action of looking up results from a view's index. In Couchbase Lite, queries are objects of the Query class. To perform a query you create one of these, customise its properties (such as the key range or the maximum number of rows) and then run it. The result is a QueryEnumerator, which provides a list of QueryRow objects, each one describing one row from the view's index.

Now you have created the view to index List documents, you can query it. In `MainActivity.java`, add the missing code to the `setupTodoLists` method to run the query.

Iterate on the result and print the title of every List document. If you saved List documents in Step 1, you should now see the titles in LogCat.

![][image-6]

The solution is on the `workshop/query_views` branch.

At this point, we could pass the result enumerator to a ArrayAdapter or RecyclerViewAdapter to display the lists on screen. However, we will jump slightly ahead of ourselves and use a Live Query to have Reactive UI capabilities.

### STEP 4: A Recycler View meets a Live Query

Couchbase Lite provides live queries. Once created, a live query remains active and monitors changes to the view's index, notifying observers whenever the query results change. Live queries are very useful for driving UI components like lists.

We will use the query to populate a Recycler View with those documents. To have the UI automatically update when new documents are indexed, we will use a Live Query.

Open `LiveQueryRecyclerAdapter.java` and let’s discuss the methods in this file:

![][image-7]

There are a few things to note here that you will see over and over again when using View Queries with UI classes. The constructor takes a LiveQuery as the second parameter. We subsequently use the `addChangeListener` method to register a listener for changes to the view result (also called an `enumerator`). That’s great because it means the adapter will get notified when it needs to redraw the Recycler View.

Next up, open `ListAdapter.java`:

![][image-8]

The responsibility of this class is to bind the data from the document to the `viewHolder`. In particular, the `onCreateViewHolder` creates the view holder.

Now we understand the mechanics from Query » LiveQueryRecyclerAdapter » ListAdapter, we can use it to display with the Query we wrote in Step 4.

### STEP 5: Using the ListsAdapter

Back in `setupTodoLists` of `MainActivity.java`, we will need to make slight changes to accommodate for a live query instead of a simple query. There is a `liveQuery` property on the Main Activity class that we can use in `setupTodoLists`:

- Initialise the liveQuery with the query from Step 4 (all queries have a `toLiveQuery` method we can use to convert the query into a Live Query).
- Create a new `listAdapter` variable of type ListAdapter and pass in the liveQuery object.
- Click events on a row are handled by this class, use the `setOnItemClickListener` method passing in `this` as the argument.
- Use the `setAdapter` method on the `recyclerView` variable to wire up the adapter to the Recycler View.

Run the app on the simulator and start creating ToDo lists, you can see they are persisted and displayed in the Drawer.

![][image-9]

The solution is on the `workshop/persist_task_document` branch.

### STEP 6: Persist the Task document

Open `Task.java` and find the `createTask` method. Similarly to Step 1 & 2, complete the body of the function to persist the HashMap of properties in a document.

Instantiate a new HashMap and add the following properties:
- `type` » the type of document, in this case `task`.
- `title` » the title parameter passed in.
- `checked` » a boolean to track if a task has been completed, the default is `Boolean.FALSE`.
- `created_at` » the `currentTimeString` variable.
- `list_id` » the listId parameter passed in .

So far, we’ve added valid JSON types similarly to Step 1. 

	need steps on where to call it

![][image-10]

However, a Task document can have an image. In Couchbase Lite, all binary properties of documents are called attachments. The Document api doesn’t allow to save an attachment. To do so, we’ll have to go one step further and use the underlying Revision api.

### STEP 7: Working with Attachments and Revisions

To create a Revision, we must first create a Document:

- Create a new variable named `document` of type Document using the `createDocument` method.
- In turn, create a new variable name revision of type Revision with the document’s `createRevision` method.
- Call the `setUserProperties` passing in the properties HashMap. In this context, user properties represent any property except the `_id` and `rev`, those two properties are important to save the revision as we’ll see in a bit. If we called the `setProperties`, the `_id` and `rev` would get deleted in the process.
- If an image was passed in, use the `setAttachment` method on the revision to save it as attachment.
- Call `revision.save()` and this will create the new revision with the image attachment.

Run the app and you should now be able to attach images to tasks:

![][image-11]

The solution is on the `workshop/attachments_and_revisions` branch.

## 30 minutes: Sync Gateway in-depth

The goal is to add the sync feature to our application. The speaker will go through the steps to install Sync Gateway and get it running with Couchbase Server.

Then, we will all attempt to connect to the same instance of Sync Gateway running [here][3].

## 30 minutes: Hands-on, Replications

### STEP 8: Replications without authentication

In `MainActivity.java`, create a new method called `startReplications` to create the push/pull replications:

- Initialise a new NSURL object. The string url for this tutorial is `http://todolite-syncgateway.cluster.com`.
- Initialise the pull replication with the `createPullReplication` method.
- Initialise the push replication with the `createPushReplication  ` method.
- Set the continuous property to true on both replications.
- Call the `start` method on each replication.

Finally, call the `startReplications` method in the `onCreate` method.

If you run the app, nothing is saved to the Sync Gateway. That’s because we disabled the GUEST account in the configuration file.  You can see the 401 HTTP errors in the console:

	Gif TBA

The solution is on the `workshop/replication` branch.

In the next section, you will add user authentication with Sync Gateway. You can choose to use Facebook Login or Basic Authentication for this workshop.

### STEP 9: Sync Gateway Basic Authentication

Currently, the functionality to create a user with a username/password is not implemented in ToDoLite-iOS or ToDoLite-Android. But you can create one using the ToDoLite-Web app, the demo app is available at `http://todolite-web.herokuapp.com` and is connecting to the same Sync Gateway instance.

Create a new user account on the [signup page](). 

Back in the iOS app in AppDelegate.m, refactor the `startReplications` method to provide a username and password:

- Rename the `startReplications` method to take the login credentials as arguments `startReplicationsWithBasicAuth(String username, String password)`.
- Refactor the method to use those credentials to instantiate a new `authenticator` of type Authenticator.
- Wire up the authenticator to the replications using the `setAuthenticator` method.
- Call the refactored method in the `onCreate` method.

Notice in LogCat that the documents are now syncing to Sync Gateway.

	Gif TBA

The solution is on the `workshop/replication_basic_auth` branch.

### STEP 10: Sync Gateway Facebook Authentication

	add paragraph to get the facebook token

If you logged into the app with Facebook then the access token should be saved to the SharedPreferences and we can retrieve it using the `preferences.getLastReceivedFbAccessToken()` method.

- Rename the `startReplications` method to take the Facebook access token as argument `startReplicationsWithFacebook(String accessToken)`.
- Refactor the method to use the access token to instantiate a new `authenticator` of type Authenticator.
- Wire up the authenticator to the replications using the `setAuthenticator` method.
- Call the refactored method in the `onCreate` method.

Notice in LogCat that the documents are now syncing to Sync Gateway.

	Gif TBA

## 30 minutes: Data orchestration with Sync Gateway

So far, you’ve learned how to use the Replication and Authenticator classes to authenticate as a user with Sync Gateway. The last component we will discuss is the Sync Function. It’s part of Sync Gateway’s configuration file and defines the access rules for users.

## 30 minutes: Hands-on, Data orchestration

### STEP 11: The Share View

As we saw in the presentation, a List document is mapped to a channel to which the Tasks are also added. The List document has a `members` property of type ArrayList holding the ids of the users to share the list with.

All Profile documents are mapped to the `profiles` channel and all users have access to it.

That way, we can display all the user Profiles and let the user pick who to share the List with. Remember earlier we used a Recycler View to display a Query result. This time, we will use the ListView api.

Similarly to the LiveQuery for the RecyclerView, the `LiveQueryAdapter.java` serves as the glue between the LiveQuery change events and the ListView api to redraw the results.

![][image-12] 

The UserAdapter class inherits from this class. In the `onCreate` method of the ShareActivity:

- Create a new variable called `query` of type Query and the `getQuery`.
- The `getQuery` takes the database and user id as parameters.
- Initialise the `mAdapter` property passing in the live query.
- Wire up the adapter to the ListView.

The `UserAdapter` is an inner class to serve as the adapter to populate the ListView. But the `getView` method is missing some code to bind the data to the item view.

Where the code is missing add the following:
- Initialise a new `user` variable of type `Document` using the `getItem` method.
- Set the text property on the `textView` to the `name` property of the document.

The `mCurrentList` property of type document refers to the List Document that was selected, check if the user id is in the array. If it’s the case then set the checked property of `checkBox` to true.

	Gif to show the checkmark displaying

The solution is on the `populating_list_items` branch.

### STEP 12: Sharing a List

Now we will use a click listener on the `checkBox` object to toggle the data and update the UI.

Setup the listener class inline and call the `List.addMemberToList` and `List.removeMemberFromList` accordingly.

Both methods update the List document according to wether it should add or remove the User from the members array property.

Next time a push replication (or immediately if it’s continuous) occurs, Sync Gateway will update the access to this List channel to reflect the change in the data model.

	Gif to show the properties updating

The solution is on the `workshop/final` branch.

## The End

Congratulations on building the main features of ToDoLite. Now you have a deeper understanding of Couchbase Lite and how to use the sync features with Sync Gateway you can start using the SDKs in your own apps.

### Optional: Working with the Sync Gateway REST API

[1]:	https://github.com/couchbaselabs/ToDoLite-Android
[2]:	http://developer.couchbase.com/mobile/develop/references/couchbase-lite/couchbase-lite/database/index.html
[3]:	#


[image-1]:	http://i.gyazo.com/a5d4774bdc4ed02afe77f3841be5db18.gif
[image-2]:	http://i.gyazo.com/daf65b5f80afe626877348635aefcead.gif
[image-3]:	http://f.cl.ly/items/0r2I3p2C0I041G3P0C0C/Model.png
[image-5]:	http://i.gyazo.com/332190d6d46fb059d7f0953bb938321f.gif
[image-6]:	http://i.gyazo.com/71c39cfdc9ed1aa5c90b1521906a92ef.gif
[image-7]:	http://cl.ly/image/3w0m352S0k0s/Screen%20Shot%202015-05-27%20at%2021.28.06.png
[image-8]:	http://cl.ly/image/2b0S2E0v1F1L/Screen%20Shot%202015-05-27%20at%2021.35.30.png
[image-9]:	http://i.gyazo.com/e7faa2e8a395a12bf4ce8315372f8a71.gif
[image-10]:	http://i.gyazo.com/68dfc680dc38813aa0c6ff144697ef4c.gif
[image-11]:	http://i.gyazo.com/4b35a4bcf99bc57d3c47553b3ca973d4.gif
[image-12]:	http://cl.ly/image/2W3F001H2C3Q/Screen%20Shot%202015-05-27%20at%2023.29.26.png
[image-13]: http://cl.ly/bTt7/git%20submodule%20git%20init.png
[image-14]: http://cl.ly/bUlx/To-Do-Lite%20Android%20folder%20content.png
[image-15]: http://cl.ly/bSYg/file_import_project.png
[image-16]: http://cl.ly/bTxP/import%20project.png
[image-17]: http://cl.ly/bU5d/ToLite%20App%20in%20Android.png
[image-18]: http://cl.ly/bUZe/Run%20Android%20App.png
[image-19]: http://cl.ly/bRmh/Build%20ToDo-Lite.png