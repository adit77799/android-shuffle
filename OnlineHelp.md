

# Views #

## Inbox ##
This is the best place to add your actions. The **Inbox** is where all actions start off before they are processed. It's a good idea to keep your Inbox as slim as possible. Doing so, does not mean you have to actually perform the actions. It simply means you've analyzed each action and done one of the following:
  * discard actions that no longer consider important
  * perform the action if it doesn't require much time
  * assigned the action to a project or one or more contexts
  * re-categorized the action as a project, if it requires many steps to complete
Once a project or context is assigned to an action, it will automatically be removed from the Inbox.

## Projects ##
Now that you've organized your actions, you can view them in a structured manner. The **Project** perspective lists all your projects, and allows you to drill-down into each project to view its actions. In addition to the standard adding, editing and deleting of actions, the project view also supports rearranging actions in the order you intend to complete them. This step is important for the Next Actions perspective discussed below.

For some projects it makes sense for actions to completed one after the other in strict order. For other
projects, actions can be completed in any order. This is configurable when creating or editing a project.

## Contexts ##
You may optionally assign one or more contexts to each of your actions. **Contexts** are like tags for your actions. They can be locations, categories or whatever makes sense to you. Shuffle comes with a few standard contexts to get you started: _At home_, _At  work_, _Online_, _Errands_, _Contact_ and _Read_. Feel free to modify or delete the existing contexts, or add new ones that suit your situation.

The context view is especially useful when you're intending on getting something done, and need to see what actions are applicable to your current situation. For instance, you're out and about in town, so you check your Errands context and see you need to buy an umbrella and pick up the dry cleaning on your way home.

## Next Actions ##
**Next actions** is the best place to look to determine what needs to be done. It shows the following actions
  * those without a project
  * the first action for sequential projects
  * all actions for parallel projects

This list quickly shows you the most important actions for you to get started on now.

## Due Actions ##
**Due actions** lets you keep track of actions you have assigned an explicit due date. There are 3 views - _Today_, _This Week_ and _This Month_ each showing actions due for the given time scale. Overdue actions are shown in all views.

## Custom ##
**Custom** by default shows all actions. Change the view settings to whatever you want to create your own personalized view.

## Tickler ##
**Tickler** shows actions you've decided to delay for whatever reason. By default it includes actions that are set to start in the future. You can change this (via View Settings) to whatever you want.

For instance, you may change it to show all inactive actions (actions assigned to inactive projects or contexts).

# Tips #

Some handy Shuffle features you may not know about:
  1. In almost all views, swipe left or right to navigate between similar views.
  1. All lists have View Settings to let you pick exactly what you want to see. For instance whether to show completed or deleted actions.
  1. Enabled the Quick Add Bar via View Settings on any view to quickly add actions, projects and contexts in place.
  1. Hitting the Back button saves before closing.
  1. Create shortcuts on your Home Screens for various Shuffle views and actions
  1. Add a widget to your Home Screen to get a quick view on any list
  1. In Global Settings > Search, enable Shuffle for actions to appear in Quick Search Box queries

# Entity lifecycle #

Actions, projects and contexts all have active and deleted flags. Additionally, actions have a completed flag.

All entities are **active** by default. You may wish to make an entity inactive if you don't intend to tackle it any time soon. It still exists, but by default, you won't see it in your views.

When you **delete** an entity, it is flagged as deleted internally and by default will not be visible in any views. You can undelete an entity by changing your View Settings to show deleted items and editing the value. If you wish to permanently delete an entity, you can do so from the Settings menu.

Actions pick up some of their state from the project and contexts they are assigned to. An action is considered active if its project (if any) is active and _at least_ one of its contexts (if any) are active. An action is considered deleted if either it is marked as deleted or its project (if any) is deleted. When a context is marked as deleted its association with all actions is removed. In other words, deleting a context will never delete an action.

# Settings #
A number of aspects of Shuffle are user-configurable. The **Settings** screen is available from all views via the menu.
  * **Calendar**  - Select a calendar to use when integrating actions with Google Calendar.
  * **Synchronize Settings**  - Configure a website you'd like your data to be synchronized with.
  * **Backup to SD Card** - Save all your actions, contexts and projects to your SD Card. You can them back this up to your computer.
  * **Restore from SD Card** - Restores all your data from a backup. No attempt is made to avoid duplicate actions, so it's best to restore onto a clean Shuffle instance.
  * **Delete completed actions** - Permanently delete all completed actions.
  * **Delete everything!** - Permanently delete all actions, projects and custom contexts.

### Synchronizing with Tracks ###

In the settings menu there is an option called “Tracks settings”. Pressing the Tracks settings option allows you to enter the settings required to synchronize with Tracks.
To synchronize with Tracks you need to enter the URL for the Tracks installation, your username and password. Currently there is no support for synchronizing with Tracks over HTTPs.

Once you have entered the settings you can check them with the check settings button.

You can set up the synchronization to run as a background process. This means that Shuffle with synchronize with Tracks at the given interval.
Once you have entered settings for synchronizing with Tracks, a Synchronize button appears in the menu at the top-level menu in Shuffle, allowing you to synchronize manually.

#### Trouble-shooting ####
If you are hosting your own Tracks and you cannot authenticate with your settings, but you can log in to Tracks using them. Then double check if Apache is sending the authentication headers to Tracks. For more information see [Tracks known issues](http://www.getontracks.org/wiki/Known-Issues/) and [Shuffle issue regarding this error](https://code.google.com/p/android-shuffle/issues/detail?id=94&can=1&colspec=ID%20Type%20Status%20Priority%20Size%20Stars%20Summary)

# Acknowledgements #
Thanks to the team behind the Tango Desktop Project, an open source collection of icons used in Shuffle.
Thanks also to David Gibb for providing some great feedback on the design, Morten Nielson for the Tracks synch work, and to Gudrun for tolerating my nights and weekends spent tinkering with Android.

For feedback or bug reports, please check http://code.google.com/p/android-shuffle/issues/list