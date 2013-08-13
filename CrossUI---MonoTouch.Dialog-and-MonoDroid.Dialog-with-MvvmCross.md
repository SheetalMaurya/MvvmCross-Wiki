**THIS ARTICLE IS VERY MUCH IN DRAFT - UNCOMPLETED**
**THIS ARTICLE IS VERY MUCH IN DRAFT - UNCOMPLETED**
**THIS ARTICLE IS VERY MUCH IN DRAFT - UNCOMPLETED**

The MvvmCross repo contains forks of the MonoTouch.Dialog and MonoDroid.Dialog projects.

These projects enable 'form' user interfaces to be programmatically created and displayed, mainly based on underlying `UITableView` and `ListView` display techniques.

Generally speaking:

- The 'forms' are made up of `Element`s. 
- A `RootElement` describes the entire 'form'. 
- Inside this Root, are a number of `Section`s - each one providing a display grouping of functionality. 
- Inside each `Section` individual `Element`s provide functionality display and/or edit options including:
  - `StringElement` - a static text Caption and Value pair
  - `EntryElement` - a static text Caption, along with an edit text input Value
  - `BooleanElement` - a static text Caption, along with a checkbox or on/off switch Value
  - `DateElement` - a static text Caption, along with a date display Value which presents a datepicker when tapped
  - `TimeElement` - a static text Caption, along with a time display Value which presents a timepicker when tapped
  - `FloatElement` - a static text Caption, along with a numeric slider Value editor
  - `RootElement`, `RadioGroup` and `RadioElement` combinations which allow single-Value selection from a list of options.
  - beyond this 'standard list', custom UIView/View elements can be added
  
  
### MvvmCross' Fork

The original .Dialog source code can be found at:

- TODO - Miguel's MT.D
- TODO - original MD.D

The main change in the MvvmCross forks of these projects are that the MvvmCross versions provide in-line editing/update/binding of editable Values.

- in 'standard' MonoTouch.Dialog, Cells are provided and filled using `GetCell()` calls
- in the 'MvvmCross' .Dialog projects:
  - new Cells are provided using `GetCellImpl()` calls
  - each `Element` stores and tracks the `CurrentAttachedCell`
  - the `CurrentAttachedCell` can be updated at any time - typically inside the methods:
  - `UpdateCellDisplay()` - update everything
  - `UpdateCaptionDisplay()` - update the Caption   
  - `UpdateDetailDisplay()` - update the Detail
  

### Using Dialog in Projects

A good sample project to see Dialog in use is the  DialogExamples code in TODO.

Further, using the MvvmCross Dialog code is shown in the video N=2x - TODO

The basic setup is to:

- for Android, ensure your UI project references the nuget package TODO - this will add references to:
  - CrossUI.Core
  - CrossUI.Droid
  - Cirrious.MvvmCross.Dialog.Droid
- for iOS, ensure your UI project references the nuget package TODO
  - CrossUI.Core
  - CrossUI.Touch
  - Cirrious.MvvmCross.Dialog.Touch
  
**Note:** The Android `CrossUI.Droid` assembly contains embedded Android resources used by the Dialog code. In recent versions Xamarin.Android has added the ability to import these resources directly into your UI project (see discussion on TODO-Xamarin-forums-link). However, there are still occasional reports of problems with this technique (eg see TODO-Stackoverflow). If you encounter such an issue, please report it, but you can also hopefully workaround the problem by manually adding the resource files directly to your UI project.

Once you have added the dialog Assemblies to your project, then you also need to modify the base class of your App's `Setup`:

   - `MvxTouchSetup` -> `MvxTouchDialogSetup`
   - `MvxDroidSetup` -> `MvxDroidDialogSetup`
   
With these setup steps done, you can then use Dialog based UIViewControllers or Activities like:

TODO - copy samples from MvxTutorials (not up-to-date on the Mac - doh!)

As you can see, binding for Dialog projects generally uses an 'inline Fluent; syntax. To bind the individual elements:

- we first create an initial `MvxInlineBindingTarget<TViewModel>` 
- and then, for each `Element`, we can bind the Element properties to this inline target

###Adding new Elements to Dialog

Todo - would be good to work out how to do this!

###Using the UIViewElement and ViewElement

Todo - would be good to work out how to do this!

###Adding Bindable Sections

Todo - this is done in the DialogExamples sample

###Customising the look and feel of iOS Elements

Todo - link to NicW's blog

###Customising the look and feel of Droid Elements

Brand new content needed here - basically every control has a `layout_name` and you can use that `layout_name` to competely customse the layout of the Element.

For example, a custom `EntryElement` might look like: TODO

###AutoViews

Need to introduce these - maybe use an old blog post.

Only describe these at high level only.

Would be fab to have a dynamic form example too?