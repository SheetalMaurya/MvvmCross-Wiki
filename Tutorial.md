## Building a new MvvmCross application

We start with a MonoDroid project on Windows... mainly because we find the Visual Studio tooling is the easiest to use, especially with Resharper installed.

The source for the finished tutorial is in: https://github.com/slodge/MvvmCross/tree/master/Sample%20-%20Tutorial/Tutorial

## Create a new empty Solution

In this case we will just call it "Tutorial"

![Add solution](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/1.png)

Add the existing projects Cirrious.Mvvm.Android and Cirrious.Mvvm.Binding.Android to your solution (later this step will be replaced with simlpy adding assembly references instead of source code inconclusion)

## Android

### Build the "Core" project - the Assembly containing the ViewModels

Start a new MonoDroid Class Library project - in this case we create it with name Tutorial.Core, and then rename it afterwards to Tutorial.Core.Droid

![Create Droid Library project](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/2.png)

Add a reference for Cirrious.Mvvm.Android to this new class library.

Delete Class1.cs

Create a new folder and call it ViewModels

Create a new class in the ViewModels and call it MainMenuViewModel. 

In this class:
- remove all the Android specific using statements - e.g. `using Android.App;`
- add MvxViewModel as a base class
- add a public property which will be the list of menu items available

We ended up with a class like:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Cirrious.MvvmCross.ViewModels;


namespace Tutorial.Core.ViewModels
{
    public class MainMenuViewModel
        : MvxViewModel
    {
        public List<Type> Items { get; set; } 
    }
}
```

Obviously right now we're missing any Items, so let's add one. Create a new ViewModel class ViewModels/Lessons/SimpleTextPropertyViewModel.cs

In this class:
- remove all the Android specific using statements - e.g. `using Android.App;`
- add MvxViewModel as a base class
- add a public property which will be the text we are going to play with - this time making it a private field backed property and including a FirePropertyChanged notification

We ended up with a class like:


```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Cirrious.MvvmCross.ViewModels;

namespace Tutorial.Core.ViewModels.Lessons
{
    public class SimpleTextPropertyViewModel
        : MvxViewModel
    {
        private string _theText;
        public string TheText
        {
            get { return _theText; }
            set { _theText = value; FirePropertyChanged("TheText"); }
        }

        public SimpleTextPropertyViewModel()
        {
            TheText = "Hello MvvmCross";
        }
    }
}
```

Now, because SimpleText is a bit too simple... let's add some value converters to our project - these will be used later by the view to make things "more exciting"

To do this, create a folder called Converters and add these 2 ValueConverters:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Cirrious.MvvmCross.Converters;

namespace Tutorial.Core.Converters
{
    public class StringLengthValueConverter
        : MvxBaseValueConverter
    {
        public override object Convert(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
        {
            var stringValue = value as string;
            if (string.IsNullOrEmpty(stringValue))
                return 0;
            return stringValue.Length;
        }
    }
}
```
and

```
using System;
using System.Linq;
using Cirrious.MvvmCross.Converters;

namespace Tutorial.Core.Converters
{
    public class StringReverseValueConverter
        : MvxBaseValueConverter
    {
        public override object Convert(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
        {
            var stringValue = value as string;
            if (string.IsNullOrEmpty(stringValue))
                return string.Empty;
            return new string(stringValue.Reverse().ToArray());
        }
    }
}
```

Also, to make the converters easier to identify, add a simple class like:

```
namespace Tutorial.Core.Converters
{
    public class Converters
    {
        public readonly StringLengthValueConverter StringLength = new StringLengthValueConverter();
        public readonly StringReverseValueConverter StringReverse = new StringReverseValueConverter();
    }
}
```



Back in MainMenuViewModel you can now add this SimpleTextPropertyViewModel class to your list:

```
        public MainMenuViewModel()
        {
            Items = new List<Type>()
                        {
                            typeof(Lessons.SimpleTextPropertyViewModel)
                        };
        }
```

And we also want a way to navigate to the new ViewModel. To do that, add an IMvxCommand to your MainMenuViewModel class:
```
        public IMvxCommand ShowItemCommand
        {
            get
            {
                return new MvxRelayCommand<Type>((type) => this.RequestNavigate(type));
            }
        }
```

With this in place, all we need left to do in the Core project is to create the main application class and a "start object".

For the start object, create a new folder called ApplicationObjects and place StartApplicationObject.cs inside it - all this class does is to provide a `Start` method:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Cirrious.MvvmCross.Interfaces.ViewModels;
using Cirrious.MvvmCross.ViewModels;
using Tutorial.Core.ViewModels;

namespace Tutorial.Core.ApplicationObjects
{
    public class StartApplicationObject 
        : MvxApplicationObject
        , IMvxStartNavigation
    {
        public void Start()
        {
            this.RequestNavigate<MainMenuViewModel>();
        }

        public bool ApplicationCanOpenBookmarks
        {
            get { return true; }
        }
    }
}
```

Now in the root of the Tutorial.Core.Droid project, create an App object. In more complex projects this App might have a lot to build and own on application startup, but for our Tutorial here, the App has only to create and register the start object:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Cirrious.MvvmCross.Application;
using Cirrious.MvvmCross.ExtensionMethods;
using Cirrious.MvvmCross.Interfaces.ServiceProvider;
using Cirrious.MvvmCross.Interfaces.ViewModels;
using Tutorial.Core.ApplicationObjects;

namespace Tutorial.Core
{
    public class App
        : MvxApplication
        , IMvxServiceProducer<IMvxStartNavigation>
    {
        public App()
        {
            Title = "MvvmCross Tutorial";

            var startApplicationObject = new StartApplicationObject();
            this.RegisterServiceInstance<IMvxStartNavigation>(startApplicationObject);
        }
    }
}
```

### Build the "Droid UI" project - the User Interface specifically for Droid

Add a new project to your solution - a Mono for Android application with name Tutorial.UI.Droid

![Add project](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/3.png)

Add references to the new project for: 

- Cirrious.MvvmCross.Android
- Cirrious.MvvmCross.Binding.Android
- Tutorial.Core.Droid

Add a link to the /Resources/Values to the resource file Cirrious.MvvmCross.Binding.Android/ResourcesToCopy/MvxBindingAttribute.xml - then make sure this file type is set to AndroidResource. Doing this copies in the required identifiers into your local project resources, enabling the Android SDK to generate some unique integer identifier for them.

Now, lets create some Views...

First the resources:
In Resources/Layout, add a new file "ListItem_ViewModel.axml" - this is the template for each individual ListItem:
```
<?xml version="1.0" encoding="utf-8"?>
<TextView 
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:local="http://schemas.android.com/apk/res/Tutorial.UI.Droid"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="6dip"
        android:layout_marginTop="6dip"
        android:textAppearance="?android:attr/textAppearanceLarge"
        local:MvxBind="{'Text':{'Path':'Name'}}"
        />
```

Within this, notice that this is fairly normal Android XML - except that we have added a local namespace for our Tutorial Assembly and we have added a Binding attribute `local:MvxBind="{'Text':{'Path':'Name'}}"` - what this binding attribute contains is some simple JSON which tells the Mvx framework to fill the TextView's Text property from the .Name property of whatever object is attached.

With that item template done, now add Page_MainMenuView.axml - which will show our main menu... In the XML for this, type:
```
<?xml version="1.0" encoding="utf-8"?>
<cirrious.mvvmcross.binding.android.views.MvxBindableListView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:local="http://schemas.android.com/apk/res/Tutorial.UI.Droid"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    local:MvxBind="{'ItemsSource':{'Path':'Items'},'ItemClick':{'Path':'ShowItemCommand'}}"
    local:MvxItemTemplate="@layout/listitem_viewmodel"
  />
```

As before, this is fairly normal android XML, excpept that we now have two new local attributes:

- local:MvxBind which this time contains a JSON object with two binding instructions - the first for ItemsSource, the second for ItemClick.
- local:MvxItemTemplate which contains a reference to the ListItem_ViewModel.axml file from above. Notice that the capitalization on this reference is a bit unusual - sometimes Android just insists you use lower case.

At this point, we'll save our detail view (SimpleTextProperyView) for a bit later, and instead write the C# for the MainMenu. To do this, within the project, add a new folder called Views

Within Views, add a new class called MainMenuView, inherit this class from a MvxBindingActivityView templated on a MainMenuViewModel, and then override the OnViewModelSet() handler in order to inflate the XML we just created:

```
using Android.App;
using Cirrious.MvvmCross.Binding.Android.Views;
using Tutorial.Core.ViewModels;

namespace Tutorial.UI.Droid.Views
{
    [Activity]
    public class MainMenuView
        : MvxBindingActivityView<MainMenuViewModel>
    {
        protected override void OnViewModelSet()
        {
            SetContentView(Resource.Layout.Page_MainMenuView);
        }
    }
}
```

With that done, we now need to add a little housekeeping code to "boot" our application.

Add a Setup.cs class at the root level - this file basically contains the code to create an App instance (from the Core project) and to map ViewModels to Views:
```
using System;
using System.Collections.Generic;
using System.Reflection;
using Android.Content;
using Cirrious.MvvmCross.Android.Platform;
using Cirrious.MvvmCross.Application;
using Cirrious.MvvmCross.Binding.Android;
using Tutorial.Core;
using Tutorial.Core.ViewModels;
using Tutorial.UI.Droid.Views;

namespace Tutorial.UI.Droid
{
    public class Setup
        : MvxBaseAndroidBindingSetup
    {
        public Setup(Context applicationContext)
            : base(applicationContext)
        {
        }

        protected override MvxApplication CreateApp()
        {
            return new App();
        }

        protected override IDictionary<Type, Type> GetViewModelViewLookup()
        {
            return new Dictionary<Type, Type>()
                       {
                           {typeof(MainMenuViewModel),typeof(MainMenuView)}
                       };
        }

        public override string ExecutableNamespace
        {
            get { return "Tutorial.UI.Droid"; }
        }

        public override Assembly ExecutableAssembly
        {
            get { return GetType().Assembly; }
        }
    }
}
```

Rename Activity1 to SplashScreenActivity, and change the code inside the class to:
```
using Android.App;
using Android.OS;
using Cirrious.MvvmCross.Android.Platform;
using Cirrious.MvvmCross.Android.Views;

namespace Tutorial.UI.Droid
{
    [Activity(Label = "Tutorial.UI.Droid", MainLauncher = true, Icon = "@drawable/icon")]
    public class SplashScreenActivity 
        : MvxBaseSplashScreenActivity
    {
        public SplashScreenActivity()
            : base(Resource.Layout.SplashScreen)
        {
        }

        protected override MvxBaseAndroidSetup CreateSetup()
        {
            return new Setup(ApplicationContext);
        }

        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);
        }
    }
}
```

where SplashScreen.axml looks like:
```
<?xml version="1.0" encoding="utf-8"?>
<TextView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent" 
    android:layout_height="fill_parent"
    android:gravity="center"
    android:text="@string/Loading"
    />
```

At this point you can run the app and see a single item list... not very exciting, and when you click on the list item, you'll generate a KeyNotFoundException, because we have no View registered for our SimpleTextPropertyViewModel

![SplashScreen...](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/4.png)
![A list with one item](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/5.png)

So... let's add a view - starting with the XML:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:local="http://schemas.android.com/apk/res/Tutorial.UI.Droid"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical"
  >
  <TextView
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="Current text"
    />
  <TextView
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:textAppearance="?android:attr/textAppearanceLarge"
    local:MvxBind="{'Text':{'Path':'TheText'}}"
    />
  <TextView
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="Length"
    />
  <TextView
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:textAppearance="?android:attr/textAppearanceLarge"
    local:MvxBind="{'Text':{'Path':'TheText','Converter':'StringLength'}}"
    />
  <TextView
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="In reverse"
    />
  <TextView
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:textAppearance="?android:attr/textAppearanceLarge"
    local:MvxBind="{'Text':{'Path':'TheText','Converter':'StringReverse'}}"
    />
  <TextView
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="Editable"
    />
  <EditText
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:textAppearance="?android:attr/textAppearanceLarge"
    local:MvxBind="{'Text':{'Path':'TheText'}}"
    />
</LinearLayout>
```

Note that this XML contains lots of views, that many of them are bound to TheText as we've seen before, except that now we are also sometimes using our ValueConverters to change what is displayed.

To enable this XML to be loaded, add an Activity for our View:

```
using Android.App;
using Cirrious.MvvmCross.Binding.Android.Views;
using Tutorial.Core.ViewModels.Lessons;

namespace Tutorial.UI.Droid.Views.Lessons
{
    [Activity]
    public class SimpleTextPropertyView
        : MvxBindingActivityView<SimpleTextPropertyViewModel>
    {
        protected override void OnViewModelSet()
        {
            SetContentView(Resource.Layout.Page_SimpleTextPropertyView);
        }
    }
}
```

and then change Setup.cs in 2 ways:

- first add our View/ViewModel pair to the lookup list:

```
        protected override IDictionary<Type, Type> GetViewModelViewLookup()
        {
            return new Dictionary<Type, Type>()
                       {
                           {typeof(MainMenuViewModel),typeof(MainMenuView)},
                           {typeof(SimpleTextPropertyViewModel),typeof(SimpleTextPropertyView)}
                       };
        }
```

- second override the FillValueConverters method in order to register our Value Converters with the Mvx framework:

```
        protected override void FillValueConverters(Cirrious.MvvmCross.Binding.Interfaces.Binders.IMvxValueConverterRegistry registry)
        {
            base.FillValueConverters(registry);

            var filler = new MvxInstanceBasedValueConverterRegistryFiller(registry);
            filler.AddFieldConverters(typeof(Converters));
        }
```

That's it... the app should now run and you should be able to tap down into the detail view which will show you something like:

![One text property bound to 4 Views](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/6.png)

And when you edit the text in the edit field then you should find all the other fields auto-update:

![After some editing](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/7.png)



## Touch

In Android and WP7, XML-driven RelativeLayouts, LinearLayouts, StackPanels and Grids are the norm for self-adjusting UI layout. However, in iOS we are encouraged more to use XIB files to achieve pixel-perfect UIs and the XIB files aren't (in my opinion) quite as programmer friendly.

This does mean that the C# code for the Touch tutorial is a bit more verbose and technical than the Android and WP7 samples, but the underlying View and Databinding principles still apply.

On of the techniques used in this tutorial, and indeed in many of the MvvmCross samples, is to use a branch of MonoTouch.Dialog to achieve a "StackPanel-like" effect. I personally find MonoTouch.Dialog much easier to use than coding with XIB files or coding directly to raw UIViews or UITableViews. When, however, I do want or need to use other UI approaches for a project, then MvvmCross still supports them, as Mvx navigation works at the UIViewController level and as Mvx binding can work on any target which supports public Properties and public events.

### Build the "Core" project - the Assembly containing the ViewModels

The code for the Core in Touch is 100% identical to the code for the Android Core.

To build this

- Add the Cirrious.MvvmCross.Touch, Cirrious.MvvmCross.Binding.Touch, Cirrious.MvvmCross.Dialog.Touch and our special MonoTouch.Dialog projects to your solution
- Create a new MonoTouch class library project.
- Delete the Class1.cs file
- Then do some "unload-finder or windows explorer based copy and paste-reload" magic to get the new .csproj file into the /Tutorial.Core directory and loaded into your solution
- Then in the solution in MonoDevelop or Visual Studio, copy and paste all the files from the Core Android project into your Core Touch project
- Then add a reference to Cirrious.MvvmCross.Touch
- And that's it - it's the same code!

### Build the "MonoTouch UI" project - the User Interface specifically for iOS/MonoTouch

Add a new project to your solution - an iPhone single view application with name Tutorial.UI.Touch

![Add project](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/t1.png)

Add references to the new project for: 

- Cirrious.MvvmCross.Touch
- Cirrious.MvvmCross.Binding.Touch
- Cirrious.MvvmCross.Dialog.Touch
- MonoTouch.Dialog (our local version)
- Tutorial.Core.Touch

Then delete the default generated ViewController and its XIB

Now, lets create some Views...

First create the Views folder.

Then create a class MainMenuView.cs. This class needs to inherit from a table view controller and needs to have access to a MainMenuViewModel - so the base class is MvxTouchTableViewController<MainMenuViewModel>.

We then also need to add some basic binding support to the class:

```
	public class MainMenuView
		: MvxTouchTableViewController<MainMenuViewModel>
		, IMvxServiceConsumer<IMvxBinder>
	{
		private readonly List<IMvxUpdateableBinding> _bindings;

        public MainMenuView(MvxShowViewModelRequest request)
            : base(request)
		{
			_bindings = new List<IMvxUpdateableBinding>();
		}
		
		protected override void Dispose (bool disposing)
		{
			if (disposing)
			{
                _bindings.ForEach(x => x.Dispose());
			}
			
			base.Dispose(disposing);
		}
```

Beyond this, the binding itself happens in ViewDidLoad() - in this class we setup binding for both a Table Delegate and a Table Source:

```
        public override void ViewDidLoad()
        {
            base.ViewDidLoad();

            Title = "Views";

            var tableDelegate = new MvxBindableTableViewDelegate();
            tableDelegate.SelectionChanged += (sender, args) => ViewModel.ShowItemCommand.Execute(args.AddedItems[0]);
            var tableSource = new TableViewDataSource(TableView);

            var binder = this.GetService<IMvxBinder>();
            _bindings.AddRange(binder.Bind(ViewModel, tableDelegate, "{'ItemsSource':{'Path':'Items'}}"));
            _bindings.AddRange(binder.Bind(ViewModel, tableSource, "{'ItemsSource':{'Path':'Items'}}"));

            TableView.Delegate = tableDelegate;
            TableView.DataSource = tableSource;
            TableView.ReloadData();
        }
```

Finally, the actual TableSource we use in this class, along with the UITableCellView for this table - are setup as nested classes:

- The Source:
```
        public class TableViewDataSource : MvxBindableTableViewDataSource
        {
            static readonly NSString CellIdentifier = new NSString("TableViewCell");

            public TableViewDataSource(UITableView tableView)
                : base(tableView)
            {
            }

            protected override UITableViewCell GetOrCreateCellFor(UITableView tableView, NSIndexPath indexPath, object item)
            {
                var reuse = tableView.DequeueReusableCell(CellIdentifier);
                if (reuse != null)
                    return reuse;

                var toReturn = new TableViewCell(UITableViewCellStyle.Subtitle, CellIdentifier)
                                   {Accessory = UITableViewCellAccessory.DisclosureIndicator};
                return toReturn;
            }
        }
```

The cell:

```
        public class TableViewCell
            : MvxBindableTableViewCell
        {
            public static readonly MvxBindingDescription[] BindingDescriptions
                = new[]
                  {
                      new MvxBindingDescription()
                          {
                              TargetName = "TitleText",
                              SourcePropertyPath = "Name"
                          },
                      new MvxBindingDescription()
                          {
                              TargetName = "DetailText",
                              SourcePropertyPath = "FullName"
                          },
                  };

            // if you don't want to code the MvxBindingDescription, then a string could instead be used:
            //public const string BindingText = @"{'TitleText':{'Path':'Name'},'DetailText':{'Path':'FullName'}}";

            public TableViewCell(UITableViewCellStyle cellStyle, NSString cellIdentifier)
                : base(BindingDescriptions, cellStyle, cellIdentifier)
            {
            }
        }
```

For the second View, the SimpleTextPropertyView we use MonoTouch.Dialog as the base class. This makes the code much, much more straightforward than for the previous MainMenuView:

```
    public class SimpleTextPropertyView
         : MvxTouchDialogViewController<SimpleTextPropertyViewModel>
    {
        public SimpleTextPropertyView(MvxShowViewModelRequest request) 
            : base(request, UITableViewStyle.Grouped, null, true)
        {
        }

        public override void ViewDidLoad()
        {
            base.ViewDidLoad();

            this.NavigationItem.SetLeftBarButtonItem(new UIBarButtonItem("Cancel", UIBarButtonItemStyle.Bordered, null), false);
            this.NavigationItem.LeftBarButtonItem.Clicked += delegate
            {
                ViewModel.BackCommand.Execute();
            };

            this.Root = new RootElement("Simple Text Property")
                            {
                                new Section("Display")
                                    {
                                        new StringElement("Current").Bind(this, "{'Value':{'Path':'TheText'}}"),
                                        new StringElement("Length").Bind(this, "{'Value':{'Path':'TheText','Converter':'StringLength'}}"),
                                        new StringElement("Reversed").Bind(this, "{'Value':{'Path':'TheText','Converter':'StringReverse'}}"),
                                    },
                                new Section("Editing")
                                    {
                                        new EntryElement("Edit").Bind(this, "{'Value':{'Path':'TheText'}}"),
                                    },
                            };
        }
    }
```

Note that for our iOS View, we have to manually add a Cancel/Back button - this is because iOS does not provide a physical Back Button at the hardware level.

With these views coded, then there's just a little housekeeping to do:

1. Create a Setup class: to create the App object; to provide the View-ViewModel mapping; and to register our converters.

```
    public class Setup
        : MvxTouchDialogBindingSetup
    {
        public Setup(MvxApplicationDelegate applicationDelegate, IMvxTouchViewPresenter presenter)
            : base(applicationDelegate, presenter)
        {
        }

        #region Overrides of MvxBaseSetup

        protected override MvxApplication CreateApp()
        {
            var app = new App();
            return app;
        }

        protected override IDictionary<Type, Type> GetViewModelViewLookup()
        {
            return new Dictionary<Type, Type>()
                       {
                            { typeof(MainMenuViewModel), typeof(MainMenuView)},
                            { typeof(SimpleTextPropertyViewModel), typeof(SimpleTextPropertyView)},
                       };
        }
		
		protected override void FillValueConverters(Cirrious.MvvmCross.Binding.Interfaces.Binders.IMvxValueConverterRegistry registry)
        {
            base.FillValueConverters(registry);

            var filler = new MvxInstanceBasedValueConverterRegistryFiller(registry);
            filler.AddFieldConverters(typeof(Converters));
        }

        #endregion
    }
```

2. Change the appdelegate class so that it creates and starts our setup.cs process

```
    // The UIApplicationDelegate for the application. This class is responsible for launching the 
    // User Interface of the application, as well as listening (and optionally responding) to 
    // application events from iOS.
    [Register("AppDelegate")]
    public partial class AppDelegate
        : MvxApplicationDelegate
        , IMvxServiceConsumer<IMvxStartNavigation>
    {
        // class-level declarations
        UIWindow window;

        //
        // This method is invoked when the application has loaded and is ready to run. In this 
        // method you should instantiate the window, load the UI into it and then make the window
        // visible.
        //
        // You have 17 seconds to return from this method, or iOS will terminate your application.
        //
        public override bool FinishedLaunching(UIApplication app, NSDictionary options)
        {
            window = new UIWindow(UIScreen.MainScreen.Bounds);

            // initialize app for single screen iPhone display
            var presenter = new MvxTouchSingleViewsPresenter(this, window);
            var setup = new Setup(this, presenter);
            setup.Initialize();

            // start the app
            var start = this.GetService<IMvxStartNavigation>();
            start.Start();

            return true;
        }
    }
```

With that house-keeping done our app should now run:

![MainMenu](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/t2.png)

And we should be able to see databinding of the initial values:

![MonoTouch.Dialog binding](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/t3.png)

And we should be able to see databinding of dynanmic changes too:

![Changing as we type](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/t4.png)

## WP7

For most WP7 engineers this will be quite simple and straight-forward - as MVVM is a standard thing to use on WP7/Silverlight apps.

### Build the "Core" project - the Assembly containing the ViewModels

The code for the Core is 100% identical to the code for the Android and Touch Cores.

To build this:

- Add the Cirrious.MvvmCross.WindowsPhone project to your solution
- Create a new WindowsPhone class library project.

![Add project](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/w1.png)

- Delete the Class1.cs file
- Then do some "unload-windows explorer based copy and paste-reload" magic to get the new .csproj file into the /Tutorial.Core directory and loaded into your solution
- Then in the solution in Visual Studio, copy and paste all the files from the Core Android project into your Core WindowsPhone project
- Then add a reference to Cirrious.MvvmCross.WindowsPhone
- And that's it - it's the same code!

### Build the "Windows Phone UI" project - the User Interface specifically for Windows Phone

Add a new project to your solution - a WindowsPhone application with name Tutorial.UI.WindowsPhone

![Add solution](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/w2.png)

Add references to the new project for: 

- Cirrious.MvvmCross.WindowsPhone
- Tutorial.Core.WindowsPhone

Now, lets create some Views...

First create the Views folder.

Then inside View, add a new Page called MainMenuView.xaml/MainMenuView.xaml.cs

To make this View inherit from the base MvxPhonePage class and to give it access to a ViewModel, change the cs to:

```
using Cirrious.MvvmCross.WindowsPhone.Views;
using Tutorial.Core.ViewModels;

namespace Tutorial.UI.WindowsPhone.Views
{
    public partial class MainMenuView : BaseMainMenuView
    {
        public MainMenuView()
        {
            InitializeComponent();
        }
    }

    public class BaseMainMenuView : MvxPhonePage<MainMenuViewModel> { }
}
```

To then make the .xaml declarations match the .cs, modify the xaml:

- so that it includes a reference to: `xmlns:Views="clr-namespace:Tutorial.UI.WindowsPhone.Views" `
- so that the root xml object is now `<Views:BaseMainMenuView .... > ... </Views:BaseMainMenuView>`

Now to add the content for this ViewModel, simply add a databound ListBox:

```
            <ListBox ItemsSource="{Binding Items}" x:Name="TheListBox">
                <ListBox.ItemTemplate>
                    <DataTemplate>
                        <TextBlock Text="{Binding}" Margin="12" FontSize="24" TextWrapping="Wrap">
                            <i:Interaction.Triggers>
                                <i:EventTrigger EventName="Tap">
                                    <commandbinding:MvxEventToCommand Command="{Binding Path=DataContext.ShowItemCommand, ElementName=TheListBox}" CommandParameter="{Binding}" />
                                </i:EventTrigger>
                            </i:Interaction.Triggers>
                        </TextBlock>
                    </DataTemplate>
                </ListBox.ItemTemplate>
            </ListBox>
```

where the additional namespaces imported are:

```
    xmlns:commandbinding="clr-namespace:Cirrious.MvvmCross.WindowsPhone.Commands;assembly=Cirrious.MvvmCross.WindowsPhone"
    xmlns:i="clr-namespace:System.Windows.Interactivity;assembly=System.Windows.Interactivity"
```

With that done, we again need to add some housekeeping:

- add a Setup.cs class to create the Core.App object and to map the Views and ViewModels together.

```
using System;
using System.Collections.Generic;
using Cirrious.MvvmCross.Application;
using Cirrious.MvvmCross.WindowsPhone.Platform;
using Microsoft.Phone.Controls;
using Tutorial.Core.ViewModels;
using Tutorial.UI.WindowsPhone.Views;

namespace Tutorial.UI.WindowsPhone
{
    public class Setup
        : MvxBaseWindowsPhoneSetup
    {
        public Setup(PhoneApplicationFrame rootFrame)
            : base(rootFrame)
        {
        }

        protected override MvxApplication CreateApp()
        {
            var app = new Core.App();
            return app;
        }

        protected override IDictionary<Type, Type> GetViewModelViewLookup()
        {
            return new Dictionary<Type, Type>()
                       {
                            { typeof(MainMenuViewModel), typeof(MainMenuView)},
                       };
        }
    }
}
```

- create this Setup class somewhere at the bottom of the App.xaml.cs constructor:

```
            var setup = new Setup(RootFrame);
            setup.Initialize();
```

- again in App.xaml.cs, setup an interception handler for the very first navigation:

```
        private bool _onceOnlyNavigation = false;

        // Code to execute when the application is launching (eg, from Start)
        // This code will not execute when the application is reactivated
        private void Application_Launching(object sender, LaunchingEventArgs e)
        {
            RootFrame.Navigating += (navigatingSender, navigatingArgs) =>
                                        {
                                            if (_onceOnlyNavigation)
                                                return;

                                            navigatingArgs.Cancel = true;
                                            _onceOnlyNavigation = true;
                                            var applicationStart = this.GetService<IMvxStartNavigation>();
                                            RootFrame.Dispatcher.BeginInvoke(applicationStart.Start);
                                        };
        }
```

- to make this compile, you will need to add inheritance on the App.xaml class to the interface `IMvxServiceConsumer<IMvxStartNavigation>` and you will need to include some using statements:

```
using Cirrious.MvvmCross.ExtensionMethods;
using Cirrious.MvvmCross.Interfaces.ServiceProvider;
using Cirrious.MvvmCross.Interfaces.ViewModels;
```

That's it - your app should now build and run the 1-item list page, although obviously if you tap on the list you can't yet drill down...

![Add solution](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/w3.png)

Note: this list is even uglier than the Droid example - I'm sure it could be made prettier - e.g. by using more views in the DataTemplate and/or a Converter to improve the text content.

So... now let's add the drill down view - the view for the SimpleTextPropertyViewModel.

To do this, add a new PhonePage to the project, then edit the csharp to look like:

```
using Cirrious.MvvmCross.WindowsPhone.Views;
using Tutorial.Core.ViewModels.Lessons;

namespace Tutorial.UI.WindowsPhone.Views.Lessons
{
    public partial class SimpleTextPropertyView : BaseSimpleTextPropertyView
    {
        public SimpleTextPropertyView()
        {
            InitializeComponent();
        }
    }

    public class BaseSimpleTextPropertyView : MvxPhonePage<SimpleTextPropertyViewModel> { }
}
```

and change the xaml root element to the appropriate base type `<Lessons:BaseSimpleTextPropertyView`

Now, add some xaml content to support the ViewModel binding:

```
            <StackPanel>
                <TextBlock
                    Text="Current text"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBlock 
                    Text="{Binding TheText}" 
                    />

                <TextBlock
                    Text="Length"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBlock 
                    Text="{Binding TheText, Converter={StaticResource StringLengthValueConverter}}" 
                    />
                
                <TextBlock
                    Text="In Reverse"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBlock 
                    Text="{Binding TheText, Converter={StaticResource StringReverseValueConverter}}" 
                    />

                <TextBlock
                    Text="Editable:"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBox
                    Text="{Binding TheText, Mode=TwoWay}" 
                    />
            </StackPanel>
```

Now, just the house keeping to go:

- add the Converters as a resource to the App.xaml:
```
    <Application.Resources>
        <Converters:StringReverseValueConverter x:Key="StringReverseValueConverter"/>
        <Converters:StringLengthValueConverter x:Key="StringLengthValueConverter"/>
    </Application.Resources>
```

where the namespace import is: `xmlns:Converters="clr-namespace:Tutorial.Core.Converters;assembly=Tutorial.Core.WindowsPhone"`

- and update the ViewModel-View map in Setup.cs:
```
         protected override IDictionary<Type, Type> GetViewModelViewLookup()
        {
            return new Dictionary<Type, Type>()
                       {
                            { typeof(MainMenuViewModel), typeof(MainMenuView)},
                            { typeof(SimpleTextPropertyViewModel), typeof(SimpleTextPropertyView)},
                       };
        }
```

That's it - the app should now run...

![Add solution](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/w4.png)

And the ViewModel text is now editable (although the way the WP7 TextBox works uses focus as the Change trigger)

![Add solution](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/w5.png)