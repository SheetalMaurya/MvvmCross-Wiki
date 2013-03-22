TODO - work in progress!

We started with the goal of creating an app to help calculate what tip to leave in a restaurant

We had a plan to produce a UI based on this concept:

![Sketch](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Sketch.png)

To satisfy this we built a 'Core' Portable Class Library project which contained:

* our 'business logic' - `ICalculation`
* our ViewModel - `TipViewModel`
* our `App` which contains the application wiring, including the start instructions.

We then added our first User Interface - for Xamarin.Android:

![Android](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Android_Styled.png)

We then added our second User Interface - for Xamarin.iOS:

![v1](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Touch_Sim.png)

For our next project, let's shift to WindowsPhone.

To create an WindowsPhone MvvmCross UI, you can use the Visual Studio project template wizards, but here we'll instead build up a new project 'from empty', just as we did for the Core, Android and iOS projects.

Obviously, to work with WindowsPhone, we will need to switch back to working on the PC with Visual Studio

## Create a new WindowsPhone Project

Add a new project to your solution - a 'WindowsPhone' application with name `TipCalc.UI.WP`

Within this, you'll find the normal WP application constructs:

TODO
* the Resources folder
* the info.plist 'configuration' information
* the AppDelegate.cs class
* the Main.cs class
* the MyViewController.cs class

## Delete MainPage.xaml

No-one really needs a `MainPage` :)

## Add references

### Add references to CoreCross and MvvmCross - PCL versions

Add references to the new project for the portable libraries:

* **Cirrious.CrossCore.dll** 
   - core interfaces and concepts including Trace, IoC and Plugin management
* **Cirrious.MvvmCross.dll** 
   - Mvvm classes - including base classes for your MvxActivity views
* **Cirrious.MvvmCross.Plugins.Json.dll**
   - Adds a PCL Newtonsoft.JSON.Net implementation - our WindowsPhone UI application will use this to navigate between Pages

Normally these will be found in a folder path like *{SolutionRoot}/Libs/Mvx/Portable/*

### Add references to CoreCross and MvvmCross - WindowsPhone specific versions

Add references to the new project for the WindowsPhone specific libraries:

* **Cirrious.CrossCore.WindowsPhone.dll** 
* **Cirrious.MvvmCross.WindowsPhone.dll** 

Each of these extends the functionality of its PCL counterpart with WP specific additions.

Normally these will be found in a folder path like *{SolutionRoot}/Libs/Mvx/WindowsPhone/*

Also, within that same folder you need to add:

### Add a reference to TipCalc.Core.csproj

Add a reference to your `TipCalc.Core` project - the project we created in the last step which included:

* your `Calculation` service, 
* your `TipViewModel` 
* your `App` wiring.

## Add a Setup class

Just as we said during the Android and iOS construction *Every MvvmCross UI project requires a `Setup` class*

This class sits in the root namespace (folder) of our UI project and performs the initialisation of the MvvmCross framework and your application, including:

  * the Inversion of Control (IoC) system
  * the MvvmCross data-binding
  * your `App` and its collection of `ViewModel`s
  * your UI project and its collection of `View`s

Most of this functionality is provided for you automatically. Within your WindowsPhone UI project all you have to supply are:

- your `App` - your link to the business logic and `ViewModel` content
- some initialisation for the Json.Net plugin and for the navigation mechanism

For `TipCalc` here's all that is needed in Setup.cs:

    using System;
    using TipCalc.Core;
    using Cirrious.MvvmCross.Touch.Views.Presenters;
    using Cirrious.MvvmCross.WindowsPhone.Platform;
    
    namespace TipCalc.UI.WindowsPhone
    {
	    public class Setup : MvxPhoneSetup
	    {
		    public Setup(PhoneApplicationFrame rootFrame)
                       : base(rootFrame)
		    {
			    return new App();
		    }

		    protected override Cirrious.MvvmCross.ViewModels.IMvxApplication CreateApp ()
		    {
			    return new App();
		    }

        protected override void InitializeDefaultTextSerializer()
        {
            Cirrious.MvvmCross.Plugins.Json.PluginLoader.Instance.EnsureLoaded(true);
        }
	    }
    }

## Modify the App.xaml.cs to use Setup

Your `App.xaml.cs` provides the WindowsPhone 'main application' object - an object which owns the User Interface and receives some callbacks from the operating system during some key events in your application's lifecycle.

To modify this `App.xaml.cs` for MvvmCross, we need to:

* modify the constructor so that it creates and starts 'Setup'

            var setup = new Setup(RootFrame);
            setup.Initialize();

* add a private field - just a boolean flag which we will set after we have done one navigation

        private bool _hasDoneFirstNavigation = false;

* modify the Application_Launching callback so that we can intercept the first navigation, can cancel it and can delegate the initial navigation to `IMvxAppStart` instead.

        // Code to execute when the application is launching (eg, from Start)
        // This code will not execute when the application is reactivated
        private void Application_Launching(object sender, LaunchingEventArgs e)
        {
            RootFrame.Navigating += (navigatingSender, navigatingArgs) =>
            {
                if (_hasDoneFirstNavigation)
                    return;

                navigatingArgs.Cancel = true;
                _hasDoneFirstNavigation = true;
                var appStart = this.GetService<IMvxAppStart>();
                RootFrame.Dispatcher.BeginInvoke(appStart.Start);
            };
        }

After you've done this your code might look like:

	using System.Collections.Generic;
	using System.Linq;
	using System.Net;
	using System.Windows;
	using System.Windows.Controls;
	using System.Windows.Documents;
	using System.Windows.Input;
	using System.Windows.Media;
	using System.Windows.Media.Animation;
	using System.Windows.Navigation;
	using System.Windows.Shapes;
	using Cirrious.CrossCore.IoC;
	using Cirrious.MvvmCross.ViewModels;
	using Microsoft.Phone.Controls;
	using Microsoft.Phone.Shell;

	using BestSellers;

	namespace BestSellers.WindowsPhone
	{
		public partial class App 
			: Application
				
		{
			/// <summary>
			/// Provides easy access to the root frame of the Phone Application.
			/// </summary>
			/// <returns>The root frame of the Phone Application.</returns>
			public PhoneApplicationFrame RootFrame { get; private set; }
			
			/// <summary>
			/// Constructor for the Application object.
			/// </summary>
			public App()
			{
				System.Diagnostics.Debug.WriteLine("Application Constructor");
				
				// Global handler for uncaught exceptions. 
				UnhandledException += Application_UnhandledException;
				
				// Standard Silverlight initialization
				InitializeComponent();
				
				// Phone-specific initialization
				InitializePhoneApplication();
				
				// Show graphics profiling information while debugging.
				if (System.Diagnostics.Debugger.IsAttached)
				{
					// Display the current frame rate counters.
					Application.Current.Host.Settings.EnableFrameRateCounter = true;
					
					// Show the areas of the app that are being redrawn in each frame.
					//Application.Current.Host.Settings.EnableRedrawRegions = true;
					
					// Enable non-production analysis visualization mode, 
					// which shows areas of a page that are handed off to GPU with a colored overlay.
					//Application.Current.Host.Settings.EnableCacheVisualization = true;
					
					// Disable the application idle detection by setting the UserIdleDetectionMode property of the
					// application's PhoneApplicationService object to Disabled.
					// Caution:- Use this under debug mode only. Application that disables user idle detection will continue to run
					// and consume battery power when the user is not using the phone.
					PhoneApplicationService.Current.UserIdleDetectionMode = IdleDetectionMode.Disabled;
				}
				
				var setup = new Setup(RootFrame);
				setup.Initialize();
			}
			
			private bool _hasDoneFirstNavigation = false;
			
			// Code to execute when the application is launching (eg, from Start)
			// This code will not execute when the application is reactivated
			private void Application_Launching(object sender, LaunchingEventArgs e)
			{
				System.Diagnostics.Debug.WriteLine("Application_Launching");
				
				RootFrame.Navigating += (navigatingSender, navigatingArgs) =>
				{
					if (_hasDoneFirstNavigation)
						return;
					
					navigatingArgs.Cancel = true;
					_hasDoneFirstNavigation = true;
					var applicationStart = Mvx.Resolve<IMvxAppStart>();
					RootFrame.Dispatcher.BeginInvoke(() => applicationStart.Start());
				};
			}
			
			// Code to execute when the application is activated (brought to foreground)
			// This code will not execute when the application is first launched
			private void Application_Activated(object sender, ActivatedEventArgs e)
			{
				System.Diagnostics.Debug.WriteLine("Application_Activated");
			}
			
			// Code to execute when the application is deactivated (sent to background)
			// This code will not execute when the application is closing
			private void Application_Deactivated(object sender, DeactivatedEventArgs e)
			{
				System.Diagnostics.Debug.WriteLine("Application_Deactivated");
			}
			
			// Code to execute when the application is closing (eg, user hit Back)
			// This code will not execute when the application is deactivated
			private void Application_Closing(object sender, ClosingEventArgs e)
			{
				System.Diagnostics.Debug.WriteLine("Application_Closing");
			}
			
			// Code to execute if a navigation fails
			private void RootFrame_NavigationFailed(object sender, NavigationFailedEventArgs e)
			{
				if (System.Diagnostics.Debugger.IsAttached)
				{
					// A navigation has failed; break into the debugger
					System.Diagnostics.Debugger.Break();
				}
			}
			
			// Code to execute on Unhandled Exceptions
			private void Application_UnhandledException(object sender, ApplicationUnhandledExceptionEventArgs e)
			{
				if (System.Diagnostics.Debugger.IsAttached)
				{
					// An unhandled exception has occurred; break into the debugger
					System.Diagnostics.Debugger.Break();
				}
			}
			
			#region Phone application initialization
			
			// Avoid double-initialization
			private bool phoneApplicationInitialized = false;
			
			// Do not add any additional code to this method
			private void InitializePhoneApplication()
			{
				if (phoneApplicationInitialized)
					return;
				
				// Create the frame but don't set it as RootVisual yet; this allows the splash
				// screen to remain active until the application is ready to render.
				RootFrame = new PhoneApplicationFrame();
				RootFrame.Navigated += CompleteInitializePhoneApplication;
				
				// Handle navigation failures
				RootFrame.NavigationFailed += RootFrame_NavigationFailed;
				
				// Ensure we don't initialize again
				phoneApplicationInitialized = true;
			}
			
			// Do not add any additional code to this method
			private void CompleteInitializePhoneApplication(object sender, NavigationEventArgs e)
			{
				// Set the root visual to allow the application to render
				if (RootVisual != RootFrame)
					RootVisual = RootFrame;
				
				// Remove this handler since it is no longer needed
				RootFrame.Navigated -= CompleteInitializePhoneApplication;
				
				System.Diagnostics.Debug.WriteLine("CompleteInitializePhoneApplication");
			}
			
			#endregion
		}
	}


## Add your View

### Create an initial Page

Create a Views folder

It is **important** on WindowsPhone

Within this, add a new 'Windows Phone Page' and call it `TipView`

This will generate:

* TipView.xaml
* TipView.cs

### Edit the XAML layout

Double click on the XAML file

This will open the XAML editor within Visual Studio.

I won't go into much depth at all here about how to use the XAML or do the Windows data-binding. I'm assuming most readers are already coming from at least a little XAML background.

To add the XAML here, simply edit the `ContentPanel` to include:

* a `StackPanel` container
* some `TextBlock` static text
* a bound `TextBox` for the `SubTotal`
* a bound `Slider` for the `Generosity`
* a bound `TextBlock` for the `Tip`

        <!--ContentPanel - place additional content here-->
        <Grid x:Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0">
            <StackPanel>
                <TextBlock
                    Text="SubTotal"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBox 
                    Text="{Binding SubTotal, Converter={StaticResource FloatValueConverter}, Mode=TwoWay}" 
                    />

                <TextBlock
                    Text="Generosity"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <Slider 
                    Value="{Binding Generosity,Mode=TwoWay}" 
                    SmallChange="1" 
                    LargeChange="10" 
                    Minimum="0" 
                    Maximum="100" />
                
                <TextBlock
                    Text="Tip"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBlock 
                    Text="{Binding Tip}" 
                    />
            </StackPanel>
        </Grid>
    </Grid>

**Note** that in XAML, `OneWay` binding is generally the default. So that is why `TwoWay` is explicitly stated in `Value="{Binding Generosity,Mode=TwoWay}"`

### Link the TipView to the TipViewModel

Open the TipView.cs file.

To link `TipView` to `TipViewModel` create a `public new TipViewModel ViewModel` property - exactly as you did in Xamarin.Android and Xamarin.iOS:

    public new TipViewModel ViewModel
    {
        get { return (TipViewModel) base.ViewModel; }
        set { base.ViewModel = value; }
    }


Altogether this looks like:

        using Cirrious.MvvmCross.WindowsPhone.Views;
	using TipView.Core;

	namespace TipView.UI.WP
	{
	    public partial class TipView : MvxPhonePage
	    {
	        public new TipViewModel ViewModel
	        {
	            get { return (TipViewModel)base.ViewModel; }
	            set { base.ViewModel = value; }
	        }

	        public TipView()
	        {
	            InitializeComponent();
	        }
	    }
	}

### Binding in Xamarin.iOS

You will no doubt have noticed that data-binding in iOS looks very different to the way it looked in Android - and to what you may have expected from XAML.

This is because the XIB format used in iOS is a lot less human manipulable and extensible than the XML formats used in Android AXML and Windows XAML - so it makes more sense to use C# rather than the XIB to register our bindings.

Within this section of the tutorial all of our iOS bindings look like:

		this.Bind (this.TipLabel, (TipViewModel vm) => vm.Tip ); 

what this line means is:

* bind the `TipLabel`'s default binding property - which happens to be a property called `Text`
* to the `ViewModel`'s Tip property

As with Android, this will be a `TwoWay` binding by default - which is different to what XAML developers may expect to see.

If you had wanted to specify the `TipLabel` property to use instead of relying on the default, then you could have done this with:

		this.Bind (this.TipLabel, label => label.Text, (TipViewModel vm) => vm.Tip ); 

In later topics we'll cover more on binding in iOS, including more on binding to non-default fields; other code-based binding code mechanisms; custom bindings; using `ValueConverter`s; and creating bound sub-views.

## The iOS UI is complete!

At this point you should be able to run your application.

When it starts... you should see:

![v1](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Touch_Sim.png)

This seems to work perfectly, although you may notice that if you tap on the `SubTotal` property and start entering text, then you cannot afterwards close the keyboard.

This is a View concern - it is a UI problem. So we can fix it just in the iOS UI code - in this View. For example, to fix this here, you can add a gesture recognizer to the end of the `ViewDidLoad` method like:

	View.AddGestureRecognizer(new UITapGestureRecognizer(() => {
		this.SubTotalTextField.ResignFirstResponder();
	}));
        
## Moving on...

There's more we could do to make this User Interface nicer and to make the app richer... but for this first application, we will leave it here for now.

Let's move on to Windows!