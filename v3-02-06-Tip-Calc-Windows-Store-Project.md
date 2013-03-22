We started with the goal of creating an app to help calculate what tip to leave in a restaurant

We had a plan to produce a UI based on this concept:

![Sketch](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Sketch.png)

To satisfy this we built a 'Core' Portable Class Library project which contained:

* our 'business logic' - `ICalculation`
* our ViewModel - `TipViewModel`
* our `App` which contains the application wiring, including the start instructions.

We've then three User Interfaces - for Xamarin.Android, Xamarin.iOS and WindowsPhone:

![Android](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Android_Styled.png) ![v1](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Touch_Sim.png) ![v1](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_WP_Emu.png)

For our next project, let's shift to WindowsStore.

To create a WindowsStore MvvmCross UI, you can use the Visual Studio project template wizards, but here we'll instead build up a new project 'from empty', just as we did for the Core, Android, iOS and WindowsPhone projects.

Obviously, to work with WindowsStore, you will need to be working on the PC with Visual Studio

## Create a new WindowsStore Project

Add a new project to your solution - a 'Blank App (XAML)' application with name `TipCalc.UI.WindowsStore`

Within this, you'll find the normal WindowsStore application constructs:

* the 'Assets' folder
* the 'Common' folder
* the 'Properties' folder with just the 'AssemblyInfo' file
* the App.Xaml 'application' object
* the MainPage.Xaml and MainPage.Xaml.cs files that define the default Page for this app
* the 'Package.appxmanifest' configuration file
* the debug private key for your development

## Delete MainPage.xaml

No-one really needs a `MainPage` :)

## Add references

### Add references to CoreCross and MvvmCross - PCL versions

Add references to the new project for the portable libraries:

* **Cirrious.CrossCore.dll** 
   - core interfaces and concepts including Trace, IoC and Plugin management
* **Cirrious.MvvmCross.dll** 
   - Mvvm classes - including base classes for your views and viewmodels

Normally these will be found in a folder path like *{SolutionRoot}/Libs/Mvx/Portable/*

### Add references to CoreCross and MvvmCross - WindowsStore specific versions

Add references to the new project for the WindowsStore specific libraries:

* **Cirrious.CrossCore.WindowsStore.dll** 
* **Cirrious.MvvmCross.WindowsStore.dll** 

Each of these extends the functionality of its PCL counterpart with WindowsStore specific additions.

Normally these will be found in a folder path like *{SolutionRoot}/Libs/Mvx/WindowsStore/*

Also, within that same folder you need to add:

### Add a reference to TipCalc.Core.csproj

Add a reference to your `TipCalc.Core` project - the project we created in the last step which included:

* your `Calculation` service, 
* your `TipViewModel` 
* your `App` wiring.

## Add a Setup class

Just as we said during the Android, iOS and WO construction *Every MvvmCross UI project requires a `Setup` class*

This class sits in the root namespace (folder) of our UI project and performs the initialisation of the MvvmCross framework and your application, including:

  * the Inversion of Control (IoC) system
  * the MvvmCross data-binding
  * your `App` and its collection of `ViewModel`s
  * your UI project and its collection of `View`s

Most of this functionality is provided for you automatically. Within your WindowsStore UI project all you have to supply is:

- your `App` - your link to the business logic and `ViewModel` content

For `TipCalc` here's all that is needed in Setup.cs:

    using Cirrious.MvvmCross.ViewModels;
    using Cirrious.MvvmCross.WindowsStore.Platform;
    using Windows.UI.Xaml.Controls;

    namespace TipCalc.UI.WindowsStore
    {
        public class Setup : MvxStoreSetup
        {
            public Setup(Frame rootFrame) : base(rootFrame)
            {
            }

            protected override IMvxApplication CreateApp()
            {
                return new Core.App();
            }
        }
    }

## Modify the App.xaml.cs to use Setup

Your `App.xaml.cs` provides the WindowsStore 'main application' object - an object which owns the User Interface and receives some callbacks from the operating system during some key events in your application's lifecycle.

To modify this `App.xaml.cs` for MvvmCross, we need to:

* modify the `OnLaunched` callback

 * remove these lines

                if (!rootFrame.Navigate(typeof(MainPage), args.Arguments))
                {
                    throw new Exception("Failed to create initial page");
                }
 

 * add these lines to allow it to create `Setup`, and to then initiate the `IMvxAppStart` `Start` navigation

            var setup = new Setup(RootFrame);
            setup.Initialize();

            var start = Mvx.Resolve<IMvxAppStart>();
            start.Start();

To do this, you will need to add these `using` lines:

    using Cirrious.CrossCore.IoC;
    using Cirrious.MvvmCross.ViewModels;


## Add your View

### Create an initial Page

Create a Views folder

Within this folder, add a new 'Basic Page' and call it `TipView.xaml`

You will be asked if you want to add the missing 'Common' files automatically in order to support this 'Basic Page' - answer **Yes**

The page will generate:

* TipView.xaml
* TipView.xaml.cs

Within Common you will also have new files added:

* BindableBase.cs
* BooleanNegationConverter.cs
* BooleanToVisibilityConverter.cs
* LayoutAwarePage.cs
* RichTextColumns.cs
* SuspensionManager.cs

### Convert LayoutAwarePage into an MvvmCross base view

Change `LayoutAwarePage` so that it inherits from `MvxStorePage`

Change:

    public class LayoutAwarePage : Page

to:

    public class LayoutAwarePage : MvxStorePage

This requires the addition of:

    using Cirrious.MvvmCross.WindowsStore.Views;

### Persuade LayoutAwarePage to cooperate more reasonably with the `MvxStorePage` base class

Either remove the `region`:

        #region Process lifetime management
        
        // all sorts of 'stuff' including
        //  OnNavigatedTo
        //  OnNavigatedFrom
        //  LoadState
        //  SaveState

        #endregion 

Or change the `OnNavigatedTo` and `OnNavigatedFrom` methods so that they call their base class implementations:

    base.OnNavigatedTo(e);

and 

    base.OnNavigatedFrom(e);


### Turn TipView into the MvvmCross View for TipViewModel

Open the TipView.cs file.

To link `TipView` to `TipViewModel` create a `public new TipViewModel ViewModel` property - exactly as you did in Xamarin.Android, Xamarin.iOS and WindowsPhone:

    public new TipViewModel ViewModel
    {
        get { return (TipViewModel) base.ViewModel; }
        set { base.ViewModel = value; }
    }

Remove the `LoadState` and `SaveState` methods.

Altogether this looks like:

    using TipCalc.Core.ViewModels;
    using TipCalc.UI.WindowsStore.Common;

    namespace TipCalc.UI.WindowsStore.Views
    {
        public sealed partial class TipView : LayoutAwarePage
        {
            public new TipViewModel ViewModel
            {
                get { return (TipViewModel)base.ViewModel; }
                set { base.ViewModel = value; }
            }

            public TipView()
            {
                this.InitializeComponent();
            }
        }
    }

### Edit the XAML layout

Double click on the XAML file

This will open the XAML editor within Visual Studio.

Just as with the WindowsPhone, I won't go into much depth at all here about how to use the XAML or do the Windows data-binding. I'm assuming most readers are already coming from at least a little XAML background.

To add the XAML user interface for our tip calculator, we will add a `ContentPanel` Grid just above the existing `<VisualStateManager.VisualStateGroups>` XAML node.

This `Content Panel` will include **almost** exactly the same XAML as we added to the WindowsPhone example - only the `Style` attributes are removed:

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
                    />
                <TextBox 
                    Text="{Binding SubTotal, Mode=TwoWay}" 
                    />

                <TextBlock
                    Text="Generosity"
                    />
                <Slider 
                    Value="{Binding Generosity,Mode=TwoWay}" 
                    SmallChange="1" 
                    LargeChange="10" 
                    Minimum="0" 
                    Maximum="100" />
                
                <TextBlock
                    Text="Tip"
                    />
                <TextBlock 
                    Text="{Binding Tip}" 
                    />
            </StackPanel>
        </Grid>

**Note** that in XAML, `OneWay` binding is generally the default. To provide TwoWay binding we explicitly add `Mode` to our binding expressions: e.g. `Value="{Binding Generosity,Mode=TwoWay}"`

In the designer, this will look like:

![Designer](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Store_Designer.png)

## The Store UI is complete!

At this point you should be able to run your application.

When it starts... you should see:

![Designer](https://raw.github.com/slodge/MvvmCross/v3/v3Tutorial/Pictures/TipCalc_Store_Emu.png)

This seems to work perfectly, although you may notice that if you edit the value in the `SubTotal` TextBox then you rest of the display does not correctly update.

This is a View concern - it is a UI problem. So we can fix it just in the WindowsStore UI code - just as we did in the WindowsPhone example.

## Moving on...

There's more we could do to make this User Interface nicer and to make the app richer... but for this first application, we will leave it here for now.

Let's move on to one final piece of Windows!