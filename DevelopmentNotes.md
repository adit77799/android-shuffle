

# Introduction #

This is a collection of the notes I made for myself when developing Shuffle. Since development is sporadic, it helps remind me what everything does, make note of gotchas and include useful links in one place.

# Android #

## Roboguice 2 ##

I've started using Roboguice 2 to pick up the fragment support. Fairly stable but found a couple of bugs.

Broadcast receivers aren't injected correctly, so I had to create a local patch for the time being - http://code.google.com/p/roboguice/issues/detail?id=150

I also had issues injecting views into fragments in a `ViewPager`. I have many instances of the same class of Fragment. After navigating a couple of times, none the the views are injected correctly. This appears to be an issue in `ViewListener` where the `viewMembersInjectors` map is getting corrupted. You end up with entries where a fragment key is pointing to an `ArrayList` of `ViewMemberInjectors` for a different fragment - http://code.google.com/p/roboguice/issues/detail?id=177

## ActionBar ##

I've also been toying with the idea of using [ActionBarSherlock](http://beta.abs.io/) to get full ICS like action bar on older phones. The action bar works so well on ICS I really don't want to have to build something else that does something similar. I'm a bit hesitant about using something like this that could easily introduce a bunch of bugs to Shuffle that are out of my control, especially since it's still in beta.

Currently experimenting with extending the [ActionBarCompat](http://developer.android.com/resources/samples/ActionBarCompat/index.html) sample project instead. As mentioned above, I don't want to have a dependency on something so critical to the app. So instead will use standard ActionBar for Honeycomb+ and compat menu for the rest. I have an easier that than the Sherlock devs since I only need to support my use cases and not the whole API.

## ViewPager ##

I'm relying heavily on `ViewPager` in the app, and having a few teething issues with the way it messes with the Fragment lifecycle. Basically, I want the current fragment to set the menus and title when it's visible. Behind the scenes the pager creates the currently selected fragment and those around it and attaches them to the activity. This means you can't assume your fragment is actually visible when onResume is called.
Instead I need to rely on the `menuVisibility` and `userVisibleHint` properties on the fragment to tell if the fragment is actually visible.
These are only set for all but the first visible fragment so I need to override `setUserVisibleHint` and `onResume` (and checking `getUserVisibleHint` in this case) to tell when a fragment is visible. If so - I update the title of the window and force an update of the menus for pre-Honeycomb devices (since these only update lazily on prepareMenus currently).

# Javascript #

These are the libraries I'm using currently...

  * **[jquery](http://jquery.com/)** - like everyone else...
  * **[d3](http://d3js.org/)** - because it's awesome and I prefer the concise api to jquery. Despite the big push for svg on the site, it also handles html dom elements very nicely too
  * **[q](https://github.com/kriskowal/q)** - for promises. These are invaluable when dealing with coordinating mulitple asynchronous operations.
  * **[bootstrap](http://twitter.github.com/bootstrap/)** - because I'm not a designer and love how well this library works on any browser I care about
  * **[flight](http://twitter.github.com/flight/)** - I like the strict separation between visual components, but don't agree with using events for everything. The mixin api is very nicely done.
  * **[knockout](http://knockoutjs.com/)** - originally used for bindings, but in process of phasing it out as I don't like the way it pollutes the dom and data binding is so easy to do with d3 or jquery anyway I don't see the need for it
  * **[sammy](http://sammyjs.org/)** - for handling deep linking via hashtags
  * **[requirejs](http://requirejs.org)** - AMD is great. I treat it like a dependency injection library and that has worked very nicely
  * **[mousetrap](http://craig.is/killing/mice)** - for keyboard shortcuts


# Google App Engine datastore and objectify-appengine #

The [Google App Engine DataStore](http://code.google.com/appengine/docs/java/datastore/) is a NoSQL high performance scalable database. [Objectify](http://code.google.com/p/objectify-appengine/) is a straight forward API for the Google App Store. Similar to the Python GAE Datastore API.

## Terminology ##

### Entity ###

Similar to a table in relational databases. Different types of entities are stored separately. Unlike relational databases, there's no schema.  Each record can store different types of data. Unlike relational databases, you can also store lists of values in a single entity. Here's a list of the [supported data types](http://code.google.com/appengine/docs/java/datastore/entities.html#Properties_and_Value_Types).
Note for strings, if you need to store more than 500 characters, you need to use [Text](http://code.google.com/appengine/docs/java/javadoc/com/google/appengine/api/datastore/Text.html) type.


### Queries ###

[Queries](http://code.google.com/appengine/docs/java/datastore/queries.html) are how you retrieve entities that match a given criteria. Queries are much more restrictive than SQL select statements. You limit your selection by supplying one or more filters to your query.
Queries are limited in the following ways
  * They do not support joins between different entity types
  * They only support ANDing of filters
  * You can only filter on properties that are indexed
  * Not equal filters are converted into < and > queries that are merged
  * IN filters are expanded into a separate query for each value in the set, then merged

== Application specific queries

Shuffle on Android has a number of queries that join 2 or more tables together. These need to be rethought to work with the App Engine datastore.

### Finding inbox values ###

Actions are shown in the Inbox if they have neither a project or a context set. Since actions can only have one project, we can simply check if the project property is null. However, we now support multiple contexts for actions, so with DataStore how do you tell if a list property is empty? The same way - check for null. See http://vaclavb.blogspot.com/2010/02/lists-and-nulls-in-google-app-engine.html
**Doesn't work** - filtering by null matches nothing for Keys (either single as for projects or list of). The alternative is to add an extra 'derived' indexed property on actions to indicate if it should be shown in the inbox.

### Finding Top Actions ###

Top Actions are active, incomplete, undeleted actions that either...
  1. Have no project
  1. Are the top action in a sequential project
  1. Are in a parallel project

As mentioned above, you can't do ORs with a single query. Similarly I can't do joins to check the parallel property on the project.

**Option A** - fetch all actions sorted by order. Fetch all projects with parallel set to false. Manually iterate throw the list including ones that have no project, have a parallel project or is the first action seen for a in-sequence project. This is wasteful as all actions need to be fetched and iterated over. This will make paging difficult.

**Option B** - Add an extra property to the action to say whether it is a 'top action'. Keep this property up to date when
  1. The project property changes on the task. Fetch the new project to see what type it is. Fetch all tasks in the project sorted by order.
  1. When the parallel property changes on a project. Fetch all actions for this project (sorted by order if not parallel). If parallel is true, set topTask property to true on all actions. If parallel is false, set topTask to true for first action, false for the rest.
  1. When two actions are swapped within a project. Simply swap the value of topTask for each task too.
  1. When active, deleted or completed properties change on any entity

This makes the top actions list query very efficient at the expense of increasing the cost of changing a project assigned to an action or changing the type of a project.

### Handling nested projects ###

Lots of behaviour to think of here. What happens when deleting a nested project. What to show in the top actions page. How to convert a task to/from a project.

### Finding active and deleted actions ###

Actions inherit their active and deleted status from their contexts and projects.
An action is active if it's on active flag is true, either it has not project or it's project active flag is true and either it has no contexts or at least one of it's contexts has an active flag set to true. Otherwise it is inactive.
An action is deleted if it's deleted flag is true, it has a project with a deleted flag set to true, or all of it's contexts have a deleted flag set to true.

Will have to add extra properties on the action and update these when any of these things happen.
  1. The active or deleted property changes on an action. Recalculate the derived properties.
  1. The active or deleted property changes on a project. Fetch all actions under this project and set project active/deleted flag on each.
  1. The active or deleted property changes on a context. Fetch all actions under this context. Add or remove 1 from active/deleted count for each action.

### Synchronising with device ###

An efficient way of determining if an entity has changed since the last sync is to use a cursor set at the last position of the entity list sorted by modification date. Everything after that cursor position has changed since the last sync. See http://code.google.com/appengine/docs/java/datastore/queries.html#Query_Cursors

## Handling complex queries ##

The only sane way I can think of for handling complex queries is to effectively denormalize the data - pushing additional derived properties onto the Task class. These are then indexed and make for very cheap straight forward queries. The downside is this extra state needs to be stored (extra DB storage and IO to and from the DB), plus this state needs to be maintained. There's quite a few calculations that need to be performed, based on a large set of operations, so this needs a lot of testing. I suspect I will also need to use a  [Task Queue](http://code.google.com/appengine/docs/java/taskqueue/) to either correct bugs, or clean up state after big operations like restore from backup file.

So currently the queries that require derived properties are:
  * **Inbox** - all actions that have no contexts and no projects
  * **Top Actions** - active, incomplete, undeleted actions that either a) have no project b) are in a parallel project or c) are the top active, incomplete in a sequential project
  * **Active filter** (applicable to most views) - Actions that are active, and either have no context or have at least one active context and either have no project or have an active project
  * **Deleted filter** (applicable to most views) - Action that are deleted themselves, or have all assigned contexts are deleted, or have a project that is deleted

| **Operation** | **Inbox** | **Top Actions** | **Active filter** | **Deleted filter** |
|:--------------|:----------|:----------------|:------------------|:-------------------|
| New task      | x         | x               | x                 | x                  |
| Swap task order |           | x               |                   |                    |
| Change task project | x         | x               | x                 | x                  |
| Change task context | x         | x               | x                 | x                  |
| Change task active |           | x               | x                 |                    |
| Change task deleted |           | x               |                   | x                  |
| Change project active |           | x               | x                 |                    |
| Change project deleted |           | x               |                   | x                  |
| Change project parallel |           | x               |                   |                    |
| Change context active |           | x               | x                 |                    |
| Change context deleted |           | x               |                   | x                  |

Top Actions is the most 'sensitive' query, and also has the biggest impact, since a single change can end up affecting multiple entities.

To listen and react on these changes, we could...
  1. Add extra code to the save operations on TaskService, ContextService and ProjectService classes. These would need to fetch a new instance of the entity using Objectify and compare it to the one passed in, then react accordingly, potentially calling into other services or DAOs to make changes. This would result in at least one extra DB call each time any entity is modified.
  1. Add extra code to Task, Context and Project in setters to perform the checks and react accordingly. This would require these classes to know about each others DAOs (or services).
  1. Add subclasses to Task, Context and Project that hold transient flags to indicate properties have changed and persisted state used to filter by for these queries. The DAOs then check these flags and update the state when any of these operations occur.

The 3rd option is the most appealing at this stage.

