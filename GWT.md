# Introduction #

I started off the website in GWT and got a reasonable way into it. We then started doing some cutting edge html5 development at work which made me want to have a crack at doing it in straight javascript/html/css. That was definitely the right decision - my productivity has improved massively, it opened the door to using lots of very decent javascript libraries and I think the website will be a lot better because of it.

# GWT #

## Terminology ##

### Module ###
A GWT configuration unit. Configured with a ModuleName.ui.xml file that lists modules it inherits from and the EntryPoint for this module (can have multiple).
http://code.google.com/webtoolkit/doc/latest/DevGuideOrganizingProjects.html

### Deferred Bindings ###
Depending on browser, load different code. Not generally used explicitly in user projects.

### Request Factory ###
Alternative to GWT-RPC for data oriented services that makes CRUD easy. Maps entities to client side objects. On client side, you create interfaces for domain objects (proxies) and server API (request contexts). The framework then creates implementations for these (on Android and GWT). For entities, it keeps track of what properties have changed and only sends those up when saving.

`RequestFactory` instance can be reused many times in your GWT module
`RequestContext` (extended by your impl) is short lived. Several calls can be setup, but fire can only be called once.
Immutable proxies can be reused between requests. To edit one, call requestContext.edit(proxy).
Mutable proxies (e.g. those returned from edit or create) can only be used in the current context.
See http://stackoverflow.com/questions/5641868/clarify-how-gwt-requestfactory-and-requestcontext-work
Default RequestFactory server side error handler swallows the stack trace - see http://cleancodematters.wordpress.com/2011/05/29/improved-exceptionhandling-with-gwts-requestfactory/

See http://code.google.com/p/google-web-toolkit/wiki/RequestFactoryMovingParts for a good description of the flow client to server.

When a call such as save(taskProxy) is sent from the client, the changes are sent up the server. It uses the ObjectifyLocator to create a load a Task from the datastore with the same id (properties are set directly not via setters), then replays the changes made on the client (ie calls to setters). This is then passed into the save method of TaskService.
If an entity refers to another entity, Objectify will load the whole object graph, but only send down the original entity (unless the client adds with("foo") qualifiers to the request.

### Code Splitting ###
Load different parts of the website separately. Improves startup time.
http://code.google.com/webtoolkit/doc/latest/DevGuideCodeSplitting.html

### Client Bundle ###
Some things are never cached (.nocache.js) others are cached forever (.cache.html), and then there's everything else. ClientBundle moves stuff into the cache forever category.
Recommended way to load resources from the server. (strongly typed and efficient)
http://code.google.com/webtoolkit/doc/latest/DevGuideClientBundle.html

Create the bundle in your app using `GWT.create(MyBundle.class)`
Then `foo.myCss().ensureInjected()` loads the CSS (safe to call many times - can be done in static initialiser in widget)

### Styles ###
Recommended is to use CssResource in Client bundles as you get lots of fancy extras and it's efficient
http://code.google.com/webtoolkit/doc/latest/DevGuideClientBundle.html#CssResource

# gwt-platform #

## Introduction ##
See http://code.google.com/p/gwt-platform/wiki/GettingStarted

MVP framework used for the GWT based website. Separates the view from business logic,

## Page construction ##
Page made up of 3 classes - View, Presenter and Proxy.

  1. **Presenter** contains View and Proxy interfaces. Where your business logic should be.
  1. **View** contains methods implemented by the actual View
  1. **Proxy** is generally implemented for you. It listens for events that would cause this view to be loaded. Lets you do lazy loading. You can annotate it to indicate this view is behind a split point. You can also annotate it with NameToken to indicate the page can use browser history - so can navigate using either the URL or a Hyperlink widget + back/forward buttons.

## Presenter Lifecycle ##
The presenter has a number of lifecycle callbacks. Need to always call super methods 1st.
onBind - right after presenter is constructed. Good place to add handlers(?) to the view. If you use registerHandler these will be automatically cleaned up.
onUnbind - destroying the presenter. Undo what you did in onBind except registerHandler which is cleaned up automatically.
onReveal - presenter becomes visible
onHide - presenter becomes invisible
onReset - user navigates to a page with this presenter. Called regardless of whether it was already visible.

## Nested Presenters ##
A page may be made up of several presenters each with it's own view. This is achieved with nested presenters. The composite presenter defines event Types annotated with @ContentSlot for each of the places where child presenters go. The composite view overrides setInSlot to place child views in the appropriate place. Child presenters override revealInParent and file the appropriate typed event that corresponds with the slot they should go in.
There's help to to tabbed presenters.

## Presenter widgets ##
Presenter (which extends PresenterWidget) are designed to be singletons. One view used for the whole app. Presenter widgets are widgets that can be instantiated multiple times. They don't have a proxy since it's not needed - only a view. Parent presenter must add the widget themselves, injecting in con structure and calling setInSlot in onReveal.

Presenter widgets can be used for dialog boxes - either associated with a few, or global to the app.

## Blocking presenters ##
Gatekeeper to prevent navigation to a page

## Calling Presenter from View ##
In other words, injecting PM into view. By defining an interface on the presenter, you can assign this to the view after creation to make callbacks. Use [UIHandler](http://code.google.com/webtoolkit/doc/latest/DevGuideUiBinder.html#Simple_binding) annotation to wire up View to events from its widgets.

## Steps for adding a new view ##
  1. Create `FooPresenter.java` and `FooView.java` in same package (see others how for how these files look)
  1. Create `FooView.ui.xml` in same package in `src/main/resources`
  1. If you want it bookmarkable, add a new token and getter to `NameTokens.java`
  1. Bind the new presenter, view and proxy in `ClientModule.java`
  1. Add a getter for the new preseter in `ClientGinjector.java`

## Passing data round on the client ##

The app uses RequestFactory to make asynchronous RPC calls to the server. Messages also need to be passed around on the client, ideally in a loosely coupled manner.

Examples of occurrences that to be handled
| **Event** | **Action** | **Mechanism** |
|:----------|:-----------|:--------------|
| A new task was created and persisted successfully | Add task list, if visible, or task count | Try `EntityProxyChange PERSIST`  event  |
| A task was updated | If visible check if it should still be visible. If so update it. | Try `EntityProxyChange UPDATE`  event |
| A task was deleted | If visible, remove from list and update counts. | Try `EntityProxyChange DELETE`  event |
| A reset event occurred (restore from backup or delete everything) | Reload all entities in all views (including nav and drop downs. Show lists from first page. | Custom event  |
| User wishes to create a new task/context/project | Navigate to editor screen | Use `PlaceManager` to reveal new `PlaceRequest` |
| User wishes to edit an existing task/context/project | Navigate to editor screen | Use `PlaceManager` to reveal new `PlaceRequest` passing id as parameter |

## Open question - how to support data binding? ##

How to supply entites to a view where:
  1. They may or may not be immediately available
  1. They can change over time

So how to support supplying them immediately, initially after a response, at any point after a change event. Really need some form of data binding.