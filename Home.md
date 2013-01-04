Welcome to the MvvmCross wiki!

## MvvmCross vNext

Sorry... this Wiki is currently out of date... updating it for vNext is on my TODO list...

If you would like to help, then please get in touch :)

## MvvmCross

Here are some of the latest examples:

- https://github.com/slodge/MvvmCross/wiki/New-Samples--TwitterSearch-and-Conference

![sql bits](http://i.imgur.com/vfWen.png)

Maybe you are looking for the Tutorial:

- https://github.com/slodge/MvvmCross/wiki/Tutorial-Part-1
- https://github.com/slodge/MvvmCross/wiki/Tutorial-Part-2:--PullToRefresh
- https://github.com/slodge/MvvmCross/wiki/Tutorial-Part-3:-TipCalc

## Just binding

Or maybe you just want binding on your MonoTouch.Dialog classes?

- https://github.com/slodge/MvvmCross/wiki/MonoTouch.Dialog-data-binding-without-full-MvvmCross-framework

Or maybe you just want Android data binding?

- https://github.com/slodge/MvvmCross/wiki/Mono-for-Android-data-binding-without-full-MvvmCross-framework

## Some useful links...

For setting up your development environment to support portable libraries, see the steps in http://slodge.blogspot.co.uk/2012/09/mvvmcross-vnext-portable-class.html

For a video of a talk I've given a couple of times about end-to-end C# development, see http://slodge.blogspot.co.uk/2012/11/azure-to-wpmonodroidmonotouch-video-of.html

http://vimeo.com/39019207 provides a video of me talking about this project (with poor noise quality - sorry...)

For more presentations (without sound) see http://www.slideshare.net/cirrious/

MvvmCross made it briefly to Channel9 - see http://slodge.blogspot.co.uk/2012/06/mvvmcross-on-channel9.html

MvvmCross was at BUILD 2012 - see http://channel9.msdn.com/Events/Build/2012/3-004

For International inspiration, see:

    http://www.slideshare.net/Runegri/kryssplatform-mobilutvikling
    http://www.slideshare.net/dan_ardelean/mvvmcross-da-windows-phone-a-windows-8-passando-per-android-e-ios
    http://www.e-naxos.com/Blog/post/Strategie-de-developpement-Cross-Platform-Partie-2.aspx

Below are some key posts from http://slodge.blogspot.com , from http://stackoverflow.com/questions/tagged/mvvmcross and beyond...

###Portable Class Libraries

Portable Class Library problems - "defined in an assembly that is not referenced" http://stackoverflow.com/questions/13197546/mvvmcross-vnext-observablecolletion-is-defined-in-an-assembly-that-is-not-refer

Working around the profile1-only problem in Mono builds - http://slodge.blogspot.co.uk/2012/10/mvvm-monotouch-back-up-and-running.html

More on PCL setup for MonoTouch/MonoDroid - http://slodge.blogspot.co.uk/2012/09/mvvmcross-vnext-portable-class.html

Xamarin reported bugs that @slodge is involved with  - mostly PCL - https://bugzilla.xamarin.com/buglist.cgi?email2=me%40slodge.com&emailassigned_to2=1&emailcc2=1&emailreporter2=1&emailtype2=substring&list_id=33065&query_format=advanced&order=bug_id%20DESC&query_based_on=

###General

Experiences and Limitations - http://stackoverflow.com/questions/10224376/mvvmcross-experiences-hindsight-limitations

Getting Help - http://slodge.blogspot.co.uk/2012/10/how-to-ask-questions-about-mvvmcross.html

Setting up a fork - basic GitHub steps - http://slodge.blogspot.co.uk/2012/09/heres-how-i-setup-fork.html

###Navigation and Architecture

Passing in variables to ViewModels - http://stackoverflow.com/questions/10192505/passing-on-variables-from-viewmodel-to-another-view-mvvmcross 

MethodAccessException - http://stackoverflow.com/questions/10227237/methodaccessexception-when-passing-variables-from-viewmodel-to-viewmodel-on-wp7

Providing different views for iPhone and iPad - http://stackoverflow.com/questions/10297260/mvvmcross-sharing-viewmodels-for-multiple-views

Replacing the default ViewModelLocator - http://stackoverflow.com/questions/10411735/how-to-replace-mvxdefaultviewmodellocator-in-mvvmcross-application

Passing in Services as constructor parameters - http://stackoverflow.com/questions/10511853/constructor-in-viewmodel

Problems in release mode only - http://stackoverflow.com/questions/11973993/mvxexception-in-release-mode-for-android

Hunting for memory leaks - http://stackoverflow.com/questions/12494804/helping-the-gc-in-mono-droid-using-mvvmcross

One pattern for error reporting - http://stackoverflow.com/questions/10411352/what-is-the-suggested-way-to-handle-exceptions-during-in-a-mvvmcross-application and also at http://stackoverflow.com/questions/11053535/alerts-or-popups-in-mvvmcross

Returning Results from ViewModels - http://www.gregshackles.com/2012/11/returning-results-from-view-models-in-mvvmcross/

###Testing

Testing ViewModels - http://slodge.blogspot.co.uk/2012/10/testing-viewmodels-in-mvvmcross.html

###Binding

JSON Binding introduction - http://stackoverflow.com/questions/10688603/how-do-i-bind-multiple-properties-in-an-android-layout-element

Custom Favorites Button Binding - http://stackoverflow.com/questions/10495137/mvvmcross-bindings-in-android
Custom binding for text color - http://stackoverflow.com/questions/10700445/in-mvvmcross-how-do-i-do-custom-bind-properties

Binding MonoTouch UIViewController member variables - http://stackoverflow.com/questions/10929779/monotouch-mvvmcross-binding-to-instance-variables

Two way bindings for integers - http://stackoverflow.com/questions/11231624/mvvmcross-monotouch-c-sharp-binding-int-property-mode-twoway

Binding doesn't work on a real device - http://stackoverflow.com/questions/11349864/mvvmcross-monotouch-fail-to-bind-properties-on-a-real-ipad-but-it-works-on-th

Custom binding for a Touch event - http://stackoverflow.com/questions/11815893/mvvmcross-touch-command-binding-in-android

ListItem Property Binding - http://stackoverflow.com/questions/12825121/binding-a-property-to-a-mvxbindablelistview-in-android

Displaying images - http://stackoverflow.com/questions/12876406/how-to-bind-an-image-src-to-resource-drawable-image-with-mvvmcross and http://stackoverflow.com/questions/12925056/binding-to-path-imageview-in-mvvmcross-for-monodroid-android

What do I do without CommandParameters? http://stackoverflow.com/questions/12892672/mvvmcross-vnext-monodroid-commandparameter-similar-to-wp7

Why doesn't Touch work - http://stackoverflow.com/questions/12950879/mvvmcross-button-command-binding-not-firing

Working around ICommand CanExecute problems - http://stackoverflow.com/questions/12951733/how-to-use-canexecute-with-mvvmcross

Debugging binding problems - http://stackoverflow.com/questions/12973594/mvxbinderror-10-40-exception-thrown-during-the-view-binding

###Droid

Using Tabs/TabActivity - http://stackoverflow.com/questions/10243672/how-to-use-a-monodroid-tabactivity-using-mvvmcross-framework

AutoComplete - http://stackoverflow.com/questions/10550829/autocomplete-mvvm-and-java-castings-without-using-java-lang-object-on-viewmodel

Removing the splash screen - http://stackoverflow.com/questions/10584385/how-do-i-initialize-the-mvvmcross-framework-without-a-splash-activity

Handling Back - http://stackoverflow.com/questions/10621593/monodroid-mvvmcross-handle-back-button-pressed-on-a-tabhost

I want a simple View without a ViewModel - http://stackoverflow.com/questions/10684174/navigate-to-a-view-without-viewmodel-in-mvvmcross

Fragment support - current not done - http://stackoverflow.com/questions/10698638/mvvmcross-viewtyperesolver-doesnt-resolve-tag-fragment-or-custom-type

Using custom Intents - http://stackoverflow.com/questions/12564272/making-mono-cross-platform-support-for-task-intent

Why won't my project include the MvxBindingAttributes.xml file - http://stackoverflow.com/questions/12596553/attributes-from-mvxbindingattributes-are-not-added-with-mvvmcross

Using ICommands within ListItems - http://stackoverflow.com/questions/12682082/mvvmcross-changing-viewmodel-within-a-mvxbindablelistview

Receiving broadcast actions - http://stackoverflow.com/questions/12687149/where-to-listen-for-broadcast-action-with-mvvmcross

Displaying Selection - http://stackoverflow.com/questions/12738516/does-mvvmcross-have-a-way-to-change-the-selector-when-clicking-imagebutton-on-li

Using custom controls in XML - http://stackoverflow.com/questions/12934654/how-to-create-views-actions-listener-for-mvxitemtemplate

Responding to checkbox changes - http://stackoverflow.com/questions/13120574/mvvmcross-vnext-checkbox-checkedchange-event-to-a-command-with-monodroid

Integrating a custom Activity - http://stackoverflow.com/questions/13201174/insert-a-monogame-view-inside-mvvmcross-monodroid-activity

Binding a spinner - http://blog.ostebaronen.dk/2012/08/mvvmcross-binding-spinner.html

###Touch

Modal presentation (and custom presenters) - http://stackoverflow.com/questions/10512853/how-do-i-specify-a-view-to-be-pushed-as-modal-in-mvvmcross

Using Tabs/UITabBarController - http://stackoverflow.com/questions/10981985/mvvmcross-monotouch-uitabbarcontroller-cant-access-navigation-bar

Customising the Navigation Bar (UINavigationController) - http://stackoverflow.com/questions/11011218/mvvmcross-uinavigationcontroller-customise-navigationbar

Modal Views - with a UINavigationController - http://stackoverflow.com/questions/11037119/mvvmcross-using-a-modal-viewcontroller-from-a-tab

Displaying multiple modal views - http://stackoverflow.com/questions/11041605/why-does-mvxmodalsupporttouchviewpresenter-in-mvvmcross-only-support-one-modal-v

Using a DateTimeElement in MT.Dialog - http://stackoverflow.com/questions/11105125/mvvmcross-null-exception-when-using-datetimeelement-in-monotouch-dialog

How to do a "combobox"/Uipickerview in Touch http://stackoverflow.com/questions/12160239/mvvmcross-binding-lists-in-monotouch (also see https://github.com/slodge/MvvmCross/pull/25)

Problems with MT.D Element redrawing - http://stackoverflow.com/questions/12249758/mvvmcross-twoway-bindings-with-monotouch-dialog

###WP7/WP8

Intercepting Back key presses - http://stackoverflow.com/questions/10515990/wp7-mvvmcross-detect-requestclose-or-backkeypressed-inside-viewmodels

Using live tiles - http://stackoverflow.com/questions/11566460/mvvmcross-and-wp7-secondary-tile

###Services

Getting Location - http://stackoverflow.com/questions/10317025/monodroid-locationmanager-requestlocationupdates-gives-java-lang-illegalargument

Distance between 2 points - http://stackoverflow.com/questions/10331836/finding-the-distance-between-two-geo-locations-when-using-the-mvvmcross-framewor

Why no speed and bearing :( - http://stackoverflow.com/questions/11582345/why-is-mvvmcross-mvxgeolocationwatcher-not-returning-speed-or-bearing

Integrating VideoView from Xamarin.Mobile - http://stackoverflow.com/questions/13016103/mvvmcross-vnext-monodroid-use-a-videoview-inside-a-plugin

###Plugins/IoC

How to write a Conventional plugin - http://stackoverflow.com/questions/12842727/mvvmcross-conventional-plugin-bypass

How to inject platform specific services - http://stackoverflow.com/questions/12564272/making-mono-cross-platform-support-for-task-intent

Another introduction to Conventional plugins - http://stackoverflow.com/questions/12931618/mvvmcross-vnext-merge-plugins-with-monodroid

Changes to IoC in vNext - http://slodge.blogspot.co.uk/2012/10/changes-to-ioc-in-vnext.html

[[SQLite Plugin]]

###"Simple Projects"

Multiple View/ViewModel sample - http://stackoverflow.com/questions/10430481/how-do-i-use-multiple-viewmodels-with-simple-binding
