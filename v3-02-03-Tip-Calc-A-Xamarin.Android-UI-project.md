We started with the goal of creating an app to help calculate what tip to leave in a restaurant

We had a plan to produce a UI based on this concept:


TODO - picture/sketch

To satisfy this we built a 'Core' Portable Class Library project which contained:

* our 'business logic' - `ICalculation`
* our ViewModel - `TipViewModel`
* our `App` which contains the application wiring, including the start instructions.

So we're now ready to add out first User Interface.

So... let's start with Android.

To create an Android MvvmCross UI, you can use the Visual Studio project template wizards, but here we'll instead build up a new project 'from empty', just as we did for the Core project.

## Create a new Android UI Project

Add a new project to your solution - a 'Xamarin.Android' application with name `TipCalc.UI.Droid`

Within this, you'll find the normal Android application constructs:

* the Assets folder
* the Resources folder
* the Activity1.cs

## Delete Activity1.cs

No-one really needs an `Activity1` :)

Also, delete `Main.axml` in the /resources/Layout folder.

## Add references

### Add references to CoreCross, Binding and MvvmCross - PCL versions

Add references to the new project for the portable libraries:

* **Cirrious.CrossCore.dll** 
   - core interfaces and concepts including Trace, IoC and Plugin management
* **Cirrious.MvvmCross.Binding.dll** 
   - DataBinding classes - which you'll mainly use from 
* **Cirrious.MvvmCross.dll** 
   - Mvvm classes - including base classes for your MvxActivity views
* **Cirrious.MvvmCross.Plugins.Json.dll** 
   - Adds a PCL Newtonsoft.JSON.Net implementation - our Android UI application will use this to navigate between Activities (pages)

Normally these will be found in a folder path like *{SolutionRoot}/Libs/Mvx/Portable/*

### Add references to CoreCross, Binding and MvvmCross - Droid specific versions

Add references to the new project for the Xamarin.Android specific libraries:

* **Cirrious.CrossCore.Droid.dll** 
* **Cirrious.MvvmCross.Binding.Droid.dll** 
* **Cirrious.MvvmCross.Droid.dll** 

Each of these extends the functionality of its PCL counterpart with Android specific additions.

Normally these will be found in a folder path like *{SolutionRoot}/Libs/Mvx/Droid/*

Also, within that same folder you need to add:

* **System.Windows.dll** - *Android version* 
   - This adds some PCL adaptation - some 'type forwarding' allowing PCL libraries that need to access things like `System.Windows.ICommand` to work on Xamarin.Android


### Add a reference to TipCalc.Core.csproj

Add a reference to your `TipCalc.Core` project - the project we created in the last step which includes your `Calculation` service, your `TipViewModel` and your `App` application wiring.


## Add the MvvmCross Android binding resource file

This file can be found in https://github.com/slodge/MvvmCross/tree/v3/Cirrious/Cirrious.MvvmCross.Binding.Droid/ResourcesToCopy

It needs to be copied to the `/Resources/Values` folder in your project.

The contents of this file are very simple - they just declare some XML extensions to enable declarative data-binding:

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	  <declare-styleable name="MvxBinding">
		<attr name="MvxBind" format="string"/>
		<attr name="MvxLang" format="string"/>
	  </declare-styleable>
	  <declare-styleable name="MvxListView">
		<attr name="MvxItemTemplate" format="string"/>
		<attr name="MvxDropDownItemTemplate" format="string"/>
	  </declare-styleable>
	  <item type="id" name="MvxBindingTagUnique"/>
	  <declare-styleable name="MvxImageView">
		<attr name="MvxSource" format="string"/>
	  </declare-styleable>
	</resources>

We'll cover the nodes and attributes within this XML file more in later topics. For this topic the only one we will use is the core data-binding attribute: `MvxBind`

## Add a Setup class

Every MvvmCross UI project requires a `Setup` class.

This class sits in the root namespace (folder) of our UI project and performs the initialisation of the MvvmCross framework, including:

  * the Inversion of Control (IoC) system
  * the MvvmCross data-binding
  * your `App` and its collection of ViewModels
  * your UI project and its collection of Views

Most of this functionality is provided for you automatically. Within your Droid UI project all you have to supply are:

- your `App` - your link to the business logic and ViewModel content
- some initialisation for the Json.Net plugin and for the navigation mechanism

For `TipCalc` here's all that is needed in Setup.cs:

    public class Setup
        : MvxAndroidSetup
    {
        public Setup(Context applicationContext)
            : base(applicationContext)
        {
        }

        protected override IMvxApplication CreateApp()
        {
            return new TipCalc.Core.App();
        }

        protected override IMvxNavigationRequestSerializer CreateNavigationRequestSerializer()
        {
            Cirrious.MvvmCross.Plugins.Json.PluginLoader.Instance.EnsureLoaded();
            return new MvxJsonNavigationSerializer();
        }
    }

**Note:** You may wonder why the Json Navigation is initialized within your Setup code, while so much else initialization is done automatically for you. The reason behind this lies in the effort MvvmCross makes to minimize dependencies on external projects. By not including JSON.Net as a reference within MvvmCross, this enables the application developer to choose a completely different serialization mechanism if you want to - e.g. you are free to choose `ServiceStack.Text`, `System.Xml.Serialization` or even some custom binary serializer.

## Add your View

### Add the Android Layout XML (AXML)

This tutorial doesn't attempt to give an introduction to Android XML layout.

Instead all I'll say here is the bare minimum. If you are new to Android, then you can find out more about Android XML from lots of places including the official documentation at: http://developer.android.com/guide/topics/ui/declaring-layout.html. If you are coming from a XAML background - you are a *XAMLite* - then I'll include some simple XAML-AXML comparisons to help you out.

To achieve the basic layout:

- we'll add a new AXML file and we'll edit it using either the Xamarin Android designer or the Visual Studio XML editor - the designer gives us a visual display, while the VS editro *sometimes* gives us XML Intellisense.

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent">
    </LinearLayout>

- we'll add a local app namespace - **http://schemas.android.com/apk/res/TipCalc.UI.Droid** - this is just like adding a namespace in XAML.

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:local="http://schemas.android.com/apk/res/TipCalc.UI.Droid"
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent">
    </LinearLayout>

- notice that this 'layout' is already by default a vertical `LinearLayout` - for XAMLites, this is like a `StackPanel`

- within this layout we'll add some `TextView`s to provide some static text labels - for XAMLites, these are "like `TextBlock`s

        <TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="SubTotal" />
        <TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="Generosity" />
        <TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="Tip to leave" />

- we'll also add a short, wide `View` with a yellow background to provide a small amount of chrome:

        <View
            android:layout_width="fill_parent"
            android:layout_height="1dp"
            android:background="#ffff00" />

- we'll add some `View`s for data display and entry, and we'll databind these `View`s to properties in our `TipViewModel` 

  - an `EditText` for text data entry of the SubTotal - for XAMLites, this is a `TextBox`

        <EditText
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            local:MvxBind="Text SubTotal" />

  - a `SeekBar` for touch/slide entry of the Generosity - for XAMLites, this is like a `ProgressBar`

        <SeekBar
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:max="40"
            local:MvxBind="Progress Generosity" />

- we'll add a `TextView` to disply the Tip that results from the calculation:

        <TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            local:MvxBind="Text Tip" />

Put together, this looks like:

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:local="http://schemas.android.com/apk/res/TipCalc.UI.Droid"
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent">
        <TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="SubTotal" />
        <EditText
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            local:MvxBind="Text SubTotal" />
        <TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="Generosity" />
        <SeekBar
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:max="40"
            local:MvxBind="Progress Generosity" />
        <View
            android:layout_width="fill_parent"
            android:layout_height="1dp"
            android:background="#ffff00" />
        <TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="Tip to leave" />
        <TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            local:MvxBind="Text Tip" />
    </LinearLayout>

### About the data-binding syntax 

Each of the data-binding blocks within our first sample looks similar:

    local:MvxBind="Text SubTotal"

What this means is:

* data-bind the property `Text` on the `View`
* to the property `SubTotal` on the `DataContext` (which in this case will be a `TipViewModel`)

In later topics, we'll return to show you many more options for data-binding, including `ValueConverter`s, but for now all our binding uses this simple `View_Property ViewModel_Property` syntax

### Add the View class

Create a Views folder within your TipCalc.UI.Droid project

Within this folder create a new C# class - `TipView`

This class will:

- inherit from `MvxActivity`
- be marked with the Xamarin.Android `Activity` attribute, marking it as the `MainLauncher` for the project
- provide a `new ViewModel` Property to specify the type of ViewModel it expects - the `TipViewModel`
- use `OnViewModelSet` to inflate its `ContentView` from the AXML - this will use a resource identifier generated by the Android and Xamarin tools. 

As a result this class is very simple:

    [Activity(MainLauncher=true)]
    public class TipCalcView : MvxActivity
    {
        public new TipViewModel ViewModel
        {
            get { return (TipViewModel)base.ViewModel; }
            set { base.ViewModel = value; }
        }

        protected override void OnViewModelSet()
        {
            SetContentView(Resource.Layout.Page_Tip);
        }
    }

## The Android UI is complete!

At this point you should be able to run your application.

When it starts... you should see:



There's obviously more you could do to make this User Interface nicer and to make the app richer... but for this first application, we will leave it here and move on to Xamarin.iOS and to Windows