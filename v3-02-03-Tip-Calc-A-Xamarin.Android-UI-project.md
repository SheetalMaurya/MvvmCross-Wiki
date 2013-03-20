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

### Add a reference to TipCalc.Core.csproj

Add a reference to your `TipCalc.Core` project - the project we created in the last step which includes your `Calculation` service, your `TipViewModel` and your `App` application wiring.

## Add the MvvmCross Android binding resource file

This file can be found in

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

### Add the AXML

This tutorial doesn't attempt to give an introduction to Android XML layout.

Instead all I'll say here is the bear minimum. If you are new to Android, then you can find out more about Android XML from lots of places including the official documentation at: http://developer.android.com/guide/topics/ui/declaring-layout.html

To achieve the basic layout:

 TODO - START AGAIN HERE TOMORROW

### Add the C#

Create a Views folder within your TipCalc.UI.Droid project

Within this folder create a new C# class - `TipView`

This class:
- must inherit from MvxActivity
- must inflate the UI from XML

 TODO - START AGAIN HERE TOMORROW
