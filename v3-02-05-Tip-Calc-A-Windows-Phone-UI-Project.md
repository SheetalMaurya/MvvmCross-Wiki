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

Add a new project to your solution - a 'Windows Phone App' application with name `TipCalc.UI.WP`

For target operating system, you can choose 7.1 or 8.0 - your choice.

Within this, you'll find the normal WP application constructs:

* the App.Xaml 'application' object
* the 'Properties' folder with its AppManifest.xml and WMAppManifest.xml 'configuration' files
* the MainPage.Xaml and MainPage.Xaml.cs files that define the default Page for this app
* some icons

## Delete MainPage.xaml

No-one really needs a `MainPage` :)

## Add references

### Add references to CoreCross and MvvmCross - PCL versions

Add references to the new project for the portable libraries:

* **Cirrious.CrossCore.dll** 
   - core interfaces and concepts including Trace, IoC and Plugin management
* **Cirrious.MvvmCross.dll** 
   - Mvvm classes - including base classes for your views and viewmodels
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
- some initialisation 
  - for the Json.Net plugin 
  - for the navigation mechanism

For `TipCalc` here's all that is needed in Setup.cs:

    using Cirrious.MvvmCross.ViewModels;
    using Microsoft.Phone.Controls;
    using Cirrious.MvvmCross.WindowsPhone.Platform;

    namespace TipCalc.UI.WP
    {
        public class Setup : MvxPhoneSetup
        {
            public Setup(PhoneApplicationFrame rootFrame)
                : base(rootFrame)
            {
            }

            protected override IMvxApplication CreateApp()
            {
                return new Core.App();
            }

            protected override IMvxNavigationSerializer CreateNavigationSerializer()
            {
                Cirrious.MvvmCross.Plugins.Json.PluginLoader.Instance.EnsureLoaded(true);
                return new MvxJsonNavigationSerializer();
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

    using System.Windows;
    using System.Windows.Navigation;
    using Cirrious.CrossCore.IoC;
    using Cirrious.MvvmCross.ViewModels;
    using Microsoft.Phone.Controls;
    using Microsoft.Phone.Shell;

    namespace TipCalc.UI.WP
    {
        public partial class App : Application
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
                RootFrame.Navigating += (navigatingSender, navigatingArgs) =>
                {
                    if (_hasDoneFirstNavigation)
                        return;

                    navigatingArgs.Cancel = true;
                    _hasDoneFirstNavigation = true;
                    var appStart = Mvx.Resolve<IMvxAppStart>();
                    RootFrame.Dispatcher.BeginInvoke(() => appStart.Start());
                };
            }

            // Code to execute when the application is activated (brought to foreground)
            // This code will not execute when the application is first launched
            private void Application_Activated(object sender, ActivatedEventArgs e)
            {
            }

            // Code to execute when the application is deactivated (sent to background)
            // This code will not execute when the application is closing
            private void Application_Deactivated(object sender, DeactivatedEventArgs e)
            {
            }

            // Code to execute when the application is closing (eg, user hit Back)
            // This code will not execute when the application is deactivated
            private void Application_Closing(object sender, ClosingEventArgs e)
            {
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
            }

            #endregion
        }
    }

## Add your View

### Create an initial Page

Create a Views folder

It is **important** on WindowsPhone, that this folder is called `Views` - the MvvmCross framework looks for this name by default on WindowsPhone.

Within this folder, add a new 'Windows Phone Portrait Page' and call it `TipView.xaml`

This will generate:

* TipView.xaml
* TipView.xaml.cs

### Turn TipView into the MvvmCross View for TipViewModel

Open the TipView.cs file.

To change TipView from a `PhonePage` into an MvvmCross view, change it so that it inherits from `MvxPhonePage`

    public partial class TipView : MvxPhonePage

To link `TipView` to `TipViewModel` create a `public new TipViewModel ViewModel` property - exactly as you did in Xamarin.Android and Xamarin.iOS:

    public new TipViewModel ViewModel
    {
        get { return (TipViewModel) base.ViewModel; }
        set { base.ViewModel = value; }
    }


Altogether this looks like:

    using Cirrious.MvvmCross.WindowsPhone.Views;
    using TipCalc.Core.ViewModels;

    namespace TipCalc.UI.WP.Views
    {
        public partial class TipView : MvxPhonePage
        {
            public new TipViewModel ViewModel
            {
                get { return (TipViewModel) base.ViewModel; }
                set { base.ViewModel = value; }
            }

            public TipView()
            {
                InitializeComponent();
            }
        }
    }

### Edit the XAML layout

Double click on the XAML file

This will open the XAML editor within Visual Studio.

I won't go into much depth at all here about how to use the XAML or do the Windows data-binding. I'm assuming most readers are already coming from at least a little XAML background.

To make the XAML inheritance match the `MvxPhonePage` inheritance, change the outer root node of the Xaml file from:

    <phone:PhoneApplicationPage 
        ... >
        <!-- content -->
    </phone:PhoneApplicationPage>

to:

    <views:MvxPhonePage
        xmlns:views="clr-namespace:Cirrious.MvvmCross.WindowsPhone.Views;assembly=Cirrious.MvvmCross.WindowsPhone"
        ... >
        <!-- content -->
    </views:MvxPhonePage>

To then add the XAML user interface for our tip calculator, we wi;l edit the `ContentPanel` to include:

* a `StackPanel` container, into which we add:
  * some `TextBlock` static text
  * a bound `TextBox` for the `SubTotal`
  * a bound `Slider` for the `Generosity`
  * a bound `TextBlock` for the `Tip`

This will produce XAML like:

        <Grid x:Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0">
            <StackPanel>
                <TextBlock
                    Text="SubTotal"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBox 
                    Text="{Binding SubTotal, Mode=TwoWay}" 
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

**Note** that in XAML, `OneWay` binding is generally the default. To provide TwoWay binding we explicitly add `Mode` to our binding expressions: e.g. `Value="{Binding Generosity,Mode=TwoWay}"`

In the designer, this will look like:

![Designer](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_WP_Designer.png)

## The WP UI is complete!

At this point you should be able to run your application.

When it starts... you should see:

![v1](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_WP_Emu.png)

This seems to work perfectly, although you may notice that if you edit the value in the `SubTotal` TextBox then you rest of the display does not correctly update.

This is a View concern - it is a UI problem. So we can fix it just in the WindowsPhone UI code - in this View. For example, to fix this here, you can add the 'Coding4Fun' toolkit from Nuget and then use their `UpdateSourceOnChange` attached property to resolve the issue

     coding4fun:TextBinding.UpdateSourceOnChange="True"
        
## Moving on...

There's more we could do to make this User Interface nicer and to make the app richer... but for this first application, we will leave it here for now.

Let's move on to even more Windows!