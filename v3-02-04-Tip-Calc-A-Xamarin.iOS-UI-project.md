We started with the goal of creating an app to help calculate what tip to leave in a restaurant

We had a plan to produce a UI based on this concept:

![Sketch](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Sketch.png)

To satisfy this we built a 'Core' Portable Class Library project which contained:

* our 'business logic' - `ICalculation`
* our ViewModel - `TipViewModel`
* our `App` which contains the application wiring, including the start instructions.

We then added our first User Interface - for Xamarin.Android:

![Android](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Android_Styled.png)

For our next project, let's shift to Xamarin.iOS.

To create an iPhone MvvmCross UI, you can use the Visual Studio project template wizards, but here we'll instead build up a new project 'from empty', just as we did for the Core and Android projects.

Also, to work with iPhone, for now we will switch to working on the Mac with Xamarin Studio

## Create a new iOS UI Project

Add a new project to your solution - a 'Xamarin.iOS' iPhone application with name `TipCalc.UI.Touch`

Within this, you'll find the normal iOS application constructs:

* the Resources folder
* the info.plist 'configuration' information
* the AppDelegate.cs class
* the Main.cs class
* the MyViewController.cs class

## Delete MyViewController.cs

No-one really needs an `MyViewController` :)

Also, delete `MyViewController.xib` (if there is one)

## Add references

### Add references to CoreCross, Binding and MvvmCross - PCL versions

Add references to the new project for the portable libraries:

* **Cirrious.CrossCore.dll** 
   - core interfaces and concepts including Trace, IoC and Plugin management
* **Cirrious.MvvmCross.Binding.dll** 
   - DataBinding classes - which you'll mainly use from 
* **Cirrious.MvvmCross.dll** 
   - Mvvm classes - including base classes for your MvxActivity views

Normally these will be found in a folder path like *{SolutionRoot}/Libs/Mvx/Portable/*

### Add references to CoreCross, Binding and MvvmCross - iOS specific versions

Add references to the new project for the Xamarin.iOS specific libraries:

* **Cirrious.CrossCore.Touch.dll** 
* **Cirrious.MvvmCross.Binding.Touch.dll** 
* **Cirrious.MvvmCross.Touch.dll** 

Each of these extends the functionality of its PCL counterpart with iOS specific additions.

Normally these will be found in a folder path like *{SolutionRoot}/Libs/Mvx/Touch/*

Also, within that same folder you need to add:

### Add a reference to TipCalc.Core.csproj

Add a reference to your `TipCalc.Core` project - the project we created in the last step which included:

* your `Calculation` service, 
* your `TipViewModel` 
* your `App` wiring.

## Add a Setup class

Just as we said during the Android construction *Every MvvmCross UI project requires a `Setup` class*

This class sits in the root namespace (folder) of our UI project and performs the initialisation of the MvvmCross framework and your application, including:

  * the Inversion of Control (IoC) system
  * the MvvmCross data-binding
  * your `App` and its collection of `ViewModel`s
  * your UI project and its collection of `View`s

Most of this functionality is provided for you automatically. Within your iOS UI project all you have to supply are:

- your `App` - your link to the business logic and `ViewModel` content

For `TipCalc` here's all that is needed in Setup.cs:

    using System;
    using Cirrious.MvvmCross.Touch.Platform;
    using TipCalc.Core;
    using Cirrious.MvvmCross.Touch.Views.Presenters;

    namespace TipCalc.UI.Touch
    {
	    public class Setup : MvxTouchSetup
	    {
		    public Setup (MvxApplicationDelegate appDelegate, IMvxTouchViewPresenter presenter)
			    : base(appDelegate, presenter)
		    {
		    }

		    protected override Cirrious.MvvmCross.ViewModels.IMvxApplication CreateApp ()
		    {
			    return new App();
		    }
	    }
    }

## Modify the AppDelegate to use Setup

Your `AppDelegate` provides a set of callback that iOS uses to inform you about events in your application's lifecycle.

To use this `AppDelegate` within MvvmCross, we need to:

* modify it so that it inherits from `MvxApplicationDelegate` instead of `UIApplicationDelegate`

        public partial class AppDelegate : MvxApplicationDelegate

* modify it so that the method that is called on startup (FinishedLaunching) does some UI application setup:

   * create a new presenter - this is the class that will determine how Views are shown - for this sample, we choose a 'standard' one:

			var presenter = new MvxTouchViewPresenter(this, window);

   * create and call Initialize on a `Setup`:

			var setup = new Setup(this, presenter);
			setup.Initialize();

   * with `Setup` completed, use the `Mvx` Inversion of Control container in order to find and `Start` the `IMvxAppStart` object:

			var startup = Mvx.Resolve<IMvxAppStart>();
			startup.Start();

Together, this looks like:

    using System;
    using System.Collections.Generic;
    using System.Linq;

    using MonoTouch.Foundation;
    using MonoTouch.UIKit;
    using Cirrious.MvvmCross.Touch.Platform;
    using Cirrious.MvvmCross.Touch.Views.Presenters;
    using Cirrious.MvvmCross.ViewModels;
    using Cirrious.CrossCore.IoC;

    namespace TipCalc.UI.Touch
    {
        [Register("AppDelegate")]
        public partial class AppDelegate : MvxApplicationDelegate
        {
            UIWindow window;

            public override bool FinishedLaunching(UIApplication app, NSDictionary options)
            {
                window = new UIWindow(UIScreen.MainScreen.Bounds);

			    var presenter = new MvxTouchViewPresenter(this, window);

			    var setup = new Setup(this, presenter);
			    setup.Initialize();

			    var startup = Mvx.Resolve<IMvxAppStart>();
			    startup.Start();

                window.MakeKeyAndVisible();

                return true;
            }
        }
    }


## Add your View

### Create an initial UIViewController

Create a Views folder

Within this, add a new 'iPhone UIViewController' called `TipView`

This will generate:

* TipView.cs
* TipView.designer.cs
* TipView.xib

### Edit the XIB layout

Double click on the XIB file

This will open the XIB editor within xCode.

Just as we did with Android, I won't go into depth here about how to use the XIB iOS editor - instead I'll just cover the bare basics, and I'll also try to provide some comparisons for those familiar with XAML.

Drag/drop from the 'Object Library' to add:

* some UILabels for showing static text - these are like `TextBlock`s
* a UITextField for editing the SubTotal - this is like a `TextBox`
* a UISlider for editing the Generosity - this is like a `ProgressBar`
* a UILabel for showing the Tip result  - this is like a `TextBlock`

Using drag and drop, uou should be able to quite quickly generate a design similar to:

![design](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Touch_Design.png)

### Create 'outlets' within the XIB editor

Once you have your UI drawn, you can then link those UI displayed fields to ObjectiveC variables called outlets. After you have done this, then the Xamarin tools will then map those ObjectiveC fields to C# properties back in your iOS app.

To start doing this, you need to open the 'Assistant Editor' from menu option 'View' -> 'Assistant Editor' -> 'Show Assistant Editor' within xCode.

Once you have done this, then you can ctrl-click (right click) on each of the 3 SubTotal, Generosity and Tip fields in turn. 

![outlet](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Touch_Outlet.png)

For each of them:

- ctrl-click the UI field
- this will 'pop up' a window listing the 'outlets and actions' available for this field
- find the one marked 'New Referencing Outlet' 
- click on the circle to the right of 'New Referencing Outlet' and drag that to the Assistant Editor.
- drop the field on the assistant editor
- it will then ask you to provide a name for your outlet option

Following this process you should be able to create three ObjectiveC variables for the three fields:

* `SubTotalTextField`
* `GenerositySlider`
* `TipLabel`

With this done, save your xCode changes (using the File Menu) and then exit xCode.


### Edit TipView.cs

Back in Xamarin Studio, you should now see that the Xamarin products have updated the TipView.designer.cs file - it will now contain three `[Outlet]` properties with those same three names

* `SubTotalTextField`
* `GenerositySlider`
* `TipLabel`

Close the TipView.designer.cs file

Open TipView.cs

Because we want our `TipView` to be not only a `UIViewController` but also an Mvvm `View`, then change the inheritance of `TipView` so that it inherits from `MvxViewController`

    public class TipView : MvxViewController

Now, to link `TipView` to `TipViewModel` create a `public new TipViewModel ViewModel` property - exactly as you did in Xamarin.Android:

    public new TipViewModel ViewModel
    {
        get { return (TipViewModel) base.ViewModel; }
        set { base.ViewModel = value; }
    }

To add the data-binding code, go to the `ViewDidLoad` method in your TipView and add some binding methods:

	public override void ViewDidLoad ()
	{
		base.ViewDidLoad ();
		
		this.Bind (this.TipLabel, (TipViewModel vm) => vm.Tip ); 
		this.Bind (this.SubTotalTextField, (TipViewModel vm) => vm.SubTotal );
		this.Bind (this.GenerositySlider, (TipViewModel vm) => vm.Generosity );
	}
   
What this code does is to generate 'in code' the same type of binding information as we generated 'in XML' in Android.

Altogether this looks like:

	using System;
	using System.Drawing;
	using MonoTouch.Foundation;
	using MonoTouch.UIKit;
	using Cirrious.MvvmCross.Touch.Views;
	using Cirrious.MvvmCross.Binding.BindingContext;
	using TipCalc.Core;

	namespace TipCalc.UI.Touch
	{
		public partial class TipView : MvxViewController
		{
			public new TipViewModel ViewModel
			{
				get { return (TipViewModel)base.ViewModel; }
				set { base.ViewModel = value; }
			}

			public TipView () : base ("TipView", null)
			{
			}
			
			public override void ViewDidLoad ()
			{
				base.ViewDidLoad ();
				
				this.Bind (this.TipLabel, (TipViewModel vm) => vm.Tip ); 
				this.Bind (this.SubTotalTextField, (TipViewModel vm) => vm.SubTotal );
				this.Bind (this.GenerositySlider, (TipViewModel vm) => vm.Generosity );
			}
		}
	}

### Binding in Xamarin.iOS

You will no doubt have noticed that data-binding in iOS looks veru different to the way it looked in Android - and to what you may have expected from XAML.

This is because the XIB format used in iOS is a lot less human manipulable and extensible than the XML formats used in Android AXML and Windows XAML - so it makes more sense to use C# rather than the XIB to register our bindings.

Within this section of the tutorial all of our iOS bindings look like:

		this.Bind (this.TipLabel, (TipViewModel vm) => vm.Tip ); 

what this line means is:

* bind the `TipLabel`'s default binding property - which happens to be a property called `Text`
* to the `ViewModel`'s Tip property

As with Android, this will be a `TwoWay` binding by default - which is different to what XAML developers may expect to see.

If you had wanted to specify the `TipLabel` property to use instead of relying on the default, then you could have done this with:

		this.Bind (this.TipLabel, label => label.Text, (TipViewModel vm) => vm.Tip ); 

In later topics we'll cover more on binding in iOS, including more on binding to non-default fields; other binding code mechanisms; and using `ValueConverter`s.

## The iOS UI is complete!

At this point you should be able to run your application.

When it starts... you should see:

![v1](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Touch_Sim.png)

This seems to work perfectly, although you may notice that when you tap on the `SubTotal` property and start entering text, then there is no way later to close the keyboard.

To work around this, you can add a gesture recognizer to the `ViewDidLoad` method:

	View.AddGestureRecognizer(new UITapGestureRecognizer(() => {
		this.SubTotalTextField.ResignFirstResponder();
	}));
        
## Moving on...

There's more we could do to make this User Interface nicer and to make the app richer... but for this first application, we will leave it here for now.

Let's move on to Windows!