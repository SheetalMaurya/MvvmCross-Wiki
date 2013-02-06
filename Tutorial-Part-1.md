## Requirements

* Visual Studio 2012 or Visual Studio 2010 with PCL support (no Express versions of VS!). 
* VisualStudio plugin for MonoAndroid installed
* PCL support for MonoAndroid. Look here: http://slodge.blogspot.co.uk/2012/12/cross-platform-winrt-monodroid.html Note: You don't need MonoTouch support in VisualStudio
* WindowsPhone SDK installed
* For iOS development: A Mac with MonoTouch installed and a prepaired targets file. See here: http://slodge.blogspot.co.uk/2013/01/if-pcls-will-not-build-for-you-in.html
* The MvvmCross binaries: http://slodge.blogspot.co.uk/p/mvvmcross-binaries_7.html (You can compile them by yourself but this can be painful. This tutorial is tested with the binaries from 2013_01_28.) 

## Building a new MvvmCross application

We start with a portable class library project on Windows.

The source for the finished tutorial is in: https://skydrive.live.com/?cid=d3d039f88037aad4&id=D3D039F88037AAD4!107#cid=D3D039F88037AAD4&id=D3D039F88037AAD4%21139

## Create a new empty Solution

In this case we will just call it "Tutorial".

Go to the Windows Explorer and create a folder MvvmCross inside your Tutorial folder. 
Extract the archive BuiltInVS2012 into the folder MvvmCross.
Rename the folder bin to VS2012.
Extract the archive BuiltInMonoTouch into the folder MvvmCross.
Rename the folder bin to MonoTouch.

![Add solution](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/1.png)

## Portable Class Library

### Build the "Core" project - the Assembly containing the ViewModels

Start a new Portable Class Library project for Profile104 and name it Tutorial.Core.
Profile104 is for .Net Framework 4.5, Silverlight 4, WindowsPhone 7.5 and Mono For Android.

Add a reference for MvvmCross\VS2012\Portable\Release\Cirrious.MvvmCross.dll to this new portable class library.

Delete Class1.cs

Create a new folder and call it ViewModels

Create a new class in the ViewModels and call it MainMenuViewModel. 

In this class:
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
            set { _theText = value; RaisePropertyChanged("TheText"); }
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

            char[] arr = stringValue.ToCharArray();
            Array.Reverse(arr);
            return new string(arr);
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

And we also want a way to navigate to the new ViewModel. To do that, add an ICommand to your MainMenuViewModel class (you need the namespaces System.Windows.Input and Cirrious.MvvmCross.Commands):
```
    public ICommand ShowItemCommand
    {
        get 
        {
		return new MvxRelayCommand<Type>((type) => DoShowItem(type));
        }
    }

		public void DoShowItem(Type itemType)
		{
			this.RequestNavigate(itemType);
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

Now in the root of the Tutorial.Core project, create an App object. In more complex projects this App might have a lot to build and own on application startup, but for our Tutorial here, the App has only to create and register the start object:

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
        , IMvxServiceProducer
    {
        public App()
        {
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

- Cirrious.MvvmCross
- Cirrious.MvvmCross.Droid
- Cirrious.MvvmCross.Binding.Android
- System.Net.dll and System.Windows.dll (saved in the Droid specific MvvmCross folder!)
- Tutorial.Core

Add a file MvxBindingAttribute.xml to the /Resources/Values and fill it with [this content](https://github.com/slodge/MvvmCross/blob/vnext/Cirrious/Cirrious.MvvmCross.Binding.Droid/ResourcesToCopy/MvxBindingAttributes.xml) - then make sure this file type is set to AndroidResource. Doing this copies in the required identifiers into your local project resources, enabling the Android SDK to generate some unique integer identifier for them.

Now, lets create some Views...

First the resources:
In Resources/Layout, add a new Android Layout "ListItem_ViewModel.axml" - this is the template for each individual ListItem:
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

NOTE: In case the Android designer escapes the single quotes to &amp;apos; then you should open the .axml file with Visual Studio's XML (Text-)Editor instead.   

Within this, notice that this is fairly normal Android XML - except that we have added a local namespace for our Tutorial Assembly and we have added a Binding attribute `local:MvxBind="{'Text':{'Path':'Name'}}"` - what this binding attribute contains is some simple JSON which tells the Mvx framework to fill the TextView's Text property from the .Name property of whatever object is attached.
You can ignore the warnings about the undefined MvxBind attribute. 

With that item template done, now add Page_MainMenuView.axml - which will show our main menu... In the XML for this, type:
```
<?xml version="1.0" encoding="utf-8"?>
<Mvx.MvxBindableListView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:local="http://schemas.android.com/apk/res/Tutorial.UI.Droid"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    local:MvxBind="{'ItemsSource':{'Path':'Items'},'ItemClick':{'Path':'ShowItemCommand'}}"
    local:MvxItemTemplate="@layout/listitem_viewmodel"
  />
```

As before, this is fairly normal android XML, except that we now have two new local attributes:

- local:MvxBind which this time contains a JSON object with two binding instructions - the first for ItemsSource, the second for ItemClick.
- local:MvxItemTemplate which contains a reference to the ListItem_ViewModel.axml file from above. Notice that the capitalization on this reference is a bit unusual - sometimes Android just insists you use lower case.

At this point, we'll save our detail view (SimpleTextProperyView) for a bit later, and instead write the C# for the MainMenu. To do this, within the project, add a new folder called Views

Within Views, add a new class called MainMenuView, inherit this class from a MvxBindingActivityView templated on a MainMenuViewModel, and then override the OnViewModelSet() handler in order to inflate the XML we just created:

```
using Android.App;
using Cirrious.MvvmCross.Binding.Droid.Views;
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
using Cirrious.MvvmCross.Droid.Platform;
using Cirrious.MvvmCross.Application;
using Cirrious.MvvmCross.Binding.Droid;
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
    }
}
```

Rename Activity1 to SplashScreenActivity, and change the code inside the class to:
```
using Android.App;
using Android.OS;
using Cirrious.MvvmCross.Droid.Platform;
using Cirrious.MvvmCross.Droid.Views;

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

Define the string with the name "Loading" by editing the file Strings.xml in Resources\Values so that it looks like this:

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="Loading">Loading</string>
    <string name="ApplicationName">Tutorial.UI.Droid</string>
</resources>
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
using Cirrious.MvvmCross.Binding.Droid.Views;
using Tutorial.Core.ViewModels.Lessons;

namespace Tutorial.UI.Droid.Views
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

and then change Setup.cs to:

- override the FillValueConverters method in order to register our Value Converters with the Mvx framework (you need the namespace Tutorial.Core.Converters):

```
        protected override IEnumerable<Type> ValueConverterHolders
        {
            get { return new[] {typeof (Converters)}; }
        }
```

That's it... the app should now run and you should be able to tap down into the detail view which will show you something like:

![One text property bound to 4 Views](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/6.png)

And when you edit the text in the edit field then you should find all the other fields auto-update:

![After some editing](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/7.png)



## Touch

In Android and WP7, XML-driven RelativeLayouts, LinearLayouts, StackPanels and Grids are the norm for self-adjusting UI layout. However, in iOS we are encouraged more to use XIB files to achieve pixel-perfect UIs and the XIB files aren't (in my opinion) quite as programmer friendly.

The technique used in this tutorial is to use a branch of MonoTouch.Dialog to achieve a "StackPanel-like" effect. If you prefer, however, to create you UI using code or using a XIB file, then check out some of the other tutorials - MvvmCross supports these as the binding can work on any target which supports public Properties and public events.

### Prepare the solution for MonoTouch on Mac

Open the file Tutorial.Core.csproj in a text editor and replace this:

```   
<TargetFrameworkProfile>Profile104</TargetFrameworkProfile>
```

with:

```
<TargetFrameworkProfile Condition="'$(OS)' != 'Windows_NT'">Profile1</TargetFrameworkProfile>      
<TargetFrameworkProfile Condition="'$(OS)' == 'Windows_NT'">Profile104</TargetFrameworkProfile>
```
For more information take a look here: http://slodge.blogspot.de/2012/10/a-temporary-solution-for-profile1-only.html

For iOS you must reference the MvvmCross binaries built with MonoTouch. To do this you can change the binary files in the referenced folder. If you do it this way then you must change tghe files every time you switch between Mac and Windows.
To avoid this you can use conditional compiling for your references. 

So, replace this:

```   
<Reference Include="Cirrious.MvvmCross">
  <HintPath>..\MvvmCross\VS2012\Portable\Release\Cirrious.MvvmCross.dll</HintPath>
</Reference>
```

with:

```
<Reference Include="Cirrious.MvvmCross" Condition="'$(OS)' == 'Windows_NT'">
  <HintPath>..\MvvmCross\VS2012\Portable\Release\Cirrious.MvvmCross.dll</HintPath>
</Reference>
<Reference Include="Cirrious.MvvmCross" Condition="'$(OS)' != 'Windows_NT'">
  <HintPath>..\MvvmCross\MonoTouch\Touch\Release\Cirrious.MvvmCross.dll</HintPath>
</Reference>
```

Now the project is prepared for use on both platforms. Copy the Tutorial solution folder to your Mac and open the solution with MonoDevelop and rebuild the Tutorial.Core portable class library. 

### Build the "MonoTouch UI" project - the User Interface specifically for iOS/MonoTouch

Add a new project to your solution - an iPhone single view application with name Tutorial.UI.Touch

![Add project](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/t1.png)

Add references to the new project for: 

- Cirrious.MvvmCross (use the same file as referenced in the Core project for iOS)
- Cirrious.MvvmCross.Binding
- Cirrious.MvvmCross.Touch
- Cirrious.MvvmCross.Binding.Touch
- Cirrious.MvvmCross.Dialog.Touch (which includes a modified version of MonoTouch.Dialog)
- Cirrious.MvvmCross.Plugins.Location.Touch
- Cirrious.MvvmCross.Plugins.ThreadUtils.Touch
- Cirrious.MvvmCross.Plugins.File.Touch
- Cirrious.MvvmCross.Plugins.DownloadCache.Touch
- CrossUI.Touch
- Tutorial.Core

Then delete the default generated ViewController and its XIB

Now, lets create some Views...

First create the Views folder.

Then create a class MainMenuView.cs. This class needs to inherit from a table view controller and needs to have access to a MainMenuViewModel - so the base class is MvxBindingTouchTableViewController<MainMenuViewModel>.

We then also need to add some basic binding support to the class:

```
using System;
using System.Collections.Generic;
using Cirrious.MvvmCross.Binding.Interfaces;
using Cirrious.MvvmCross.Binding.Touch.ExtensionMethods;
using Cirrious.MvvmCross.Binding.Touch.Views;
using Cirrious.MvvmCross.ExtensionMethods;
using Cirrious.MvvmCross.Interfaces.ServiceProvider;
using Cirrious.MvvmCross.Views;
using MonoTouch.Foundation;
using MonoTouch.UIKit;
using Tutorial.Core.ViewModels;
using Cirrious.MvvmCross.Touch.Views;

namespace Tutorial.UI.Touch
{
	public class MainMenuView
		: MvxBindingTouchTableViewController<MainMenuViewModel>
	{
		private readonly List<IMvxUpdateableBinding> _bindings;

		public MainMenuView (MvxShowViewModelRequest request)
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

	}
}
```

Beyond this, the binding itself happens in ViewDidLoad() - in this class we setup binding for both a Table Delegate and a Table Source:

```
        public override void ViewDidLoad()
        {
            base.ViewDidLoad();

            Title = "Views";

            var tableSource = new TableViewSource(TableView);
            tableSource.SelectionChanged += (sender, args) => ViewModel.DoShowItem((Type)args.AddedItems[0]);
            
            this.AddBindings(
                new Dictionary<object, string>()
                    {
                        { tableSource, "{'ItemsSource':{'Path':'Items'}}" }
                    });

            TableView.Source = tableSource;
            TableView.ReloadData();
        }
```

Finally, the actual TableSource we use in this class, along with the UITableCellView for this table - are setup as nested classes:

- The Source:
```
	public class TableViewSource : MvxBindableTableViewSource
	{
		static readonly NSString CellIdentifier = new NSString("TableViewCell");
		
		public TableViewSource(UITableView tableView)
			: base(tableView)
		{
		}
		
		protected override UITableViewCell GetOrCreateCellFor(UITableView tableView, NSIndexPath indexPath, object item)
		{
			var reuse = tableView.DequeueReusableCell(CellIdentifier);
			if (reuse != null)
				return reuse;
			
			var toReturn = new TableViewCell(UITableViewCellStyle.Subtitle, CellIdentifier);
			return toReturn;
		}
	}
```

The cell:

```
        public sealed class TableViewCell
            : MvxBindableTableViewCell
        {
            public const string BindingText = @"{'TitleText':{'Path':'Name'},'DetailText':{'Path':'FullName'}}";

            // if you don't want to JSON text, then you can use MvxBindingDescription in C#, instead:
            //public static readonly MvxBindingDescription[] BindingDescriptions
            //    = new[]
            //      {
            //          new MvxBindingDescription()
            //              {
            //                  TargetName = "TitleText",
            //                  SourcePropertyPath = "Name"
            //              },
            //          new MvxBindingDescription()
            //              {
            //                  TargetName = "DetailText",
            //                  SourcePropertyPath = "FullName"
            //              },
            //      };

            public TableViewCell(UITableViewCellStyle cellStyle, NSString cellIdentifier)
                : base(BindingText, cellStyle, cellIdentifier)
            {
                Accessory = UITableViewCellAccessory.DisclosureIndicator;
            }
        }
```

For the second View, the SimpleTextPropertyView we use MonoTouch.Dialog as the base class. This makes the code much, much more straightforward than for the previous MainMenuView:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Cirrious.MvvmCross.Dialog.Touch;
using Cirrious.MvvmCross.Views;
using MonoTouch.UIKit;
using Tutorial.Core.ViewModels.Lessons;
using CrossUI.Touch.Dialog.Elements;

namespace Tutorial.UI.Touch.Views.Lessons
{
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
                ViewModel.DoClose();
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
}
```

Note that for our iOS View, we have to manually add a Cancel/Back button - this is because iOS does not provide a physical Back Button at the hardware level.

With these views coded, then there's just a little housekeeping to do:

1. Create a Setup class: to create the App object; to provide the View-ViewModel mapping; and to register our converters.

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Cirrious.MvvmCross.Application;
using Cirrious.MvvmCross.Dialog.Touch;
using Cirrious.MvvmCross.Touch.Interfaces;
using Cirrious.MvvmCross.Binding.Binders;
using Cirrious.MvvmCross.Touch.Platform;
using Tutorial.Core;
using Tutorial.Core.Converters;
using Tutorial.Core.ViewModels;
using Tutorial.Core.ViewModels.Lessons;
using Tutorial.UI.Touch.Views;
using Tutorial.UI.Touch.Views.Lessons;
using Cirrious.MvvmCross.Platform;

namespace Tutorial.UI.Touch
{
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

        protected override IEnumerable<Type> ValueConverterHolders
        {
            get { return new[] { typeof(Converters) }; }
        }

		protected override void AddPluginsLoaders(MvxLoaderPluginRegistry registry)
		{
			registry.AddConventionalPlugin<Cirrious.MvvmCross.Plugins.Location.Touch.Plugin>();
			registry.AddConventionalPlugin<Cirrious.MvvmCross.Plugins.ThreadUtils.Touch.Plugin>();
			registry.AddConventionalPlugin<Cirrious.MvvmCross.Plugins.File.Touch.Plugin>();
			registry.AddConventionalPlugin<Cirrious.MvvmCross.Plugins.DownloadCache.Touch.Plugin>();
			base.AddPluginsLoaders(registry);
		}
        #endregion
    }
}
```

2. Change the appdelegate class so that it creates and starts our setup.cs process

```
using System;
using System.Collections.Generic;
using System.Linq;
using Cirrious.MvvmCross.ExtensionMethods;
using Cirrious.MvvmCross.Interfaces.ServiceProvider;
using Cirrious.MvvmCross.Interfaces.ViewModels;
using Cirrious.MvvmCross.Touch.Platform;
using Cirrious.MvvmCross.Touch.Views;
using Cirrious.MvvmCross.Touch.Views.Presenters;
using MonoTouch.Foundation;
using MonoTouch.UIKit;

namespace Tutorial.UI.Touch
{
    // The UIApplicationDelegate for the application. This class is responsible for launching the 
    // User Interface of the application, as well as listening (and optionally responding) to 
    // application events from iOS.
    [Register("AppDelegate")]
    public partial class AppDelegate
        : MvxApplicationDelegate
        , IMvxServiceConsumer
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
            var presenter = new MvxTouchViewPresenter(this, window);
            var setup = new Setup(this, presenter);
            setup.Initialize();

            // start the app
            var start = this.GetService<IMvxStartNavigation>();
            start.Start();

            window.MakeKeyAndVisible();

            return true;
        }
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

### Build the "Windows Phone UI" project - the User Interface specifically for Windows Phone

Add a new project to your solution - a WindowsPhone application with name Tutorial.UI.WindowsPhone

![Add solution](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/w2.png)

Add references to the new project for: 

- Cirrious.MvvmCross
- Cirrious.MvvmCross.WindowsPhone
- Tutorial.Core
- System.Windows.Interactivity
- Cirrious.MvvmCross.Plugins.ThreadUtils.WindowsPhone
- Cirrious.MvvmCross.Plugins.Location.WindowsPhone
- Cirrious.MvvmCross.Plugins.Visibility.WindowsPhone
- Cirrious.MvvmCross.Plugins.Visibility
- Cirrious.MvvmCross.Plugins.Json

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
using Cirrious.MvvmCross.Platform;
using Cirrious.MvvmCross.WindowsPhone.Platform;
using Microsoft.Phone.Controls;

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

        protected override void InitializeDefaultTextSerializer()
        {
            Cirrious.MvvmCross.Plugins.Json.PluginLoader.Instance.EnsureLoaded(true);
        }

        protected override void AddPluginsLoaders(MvxLoaderPluginRegistry registry)
        {
            registry.AddConventionalPlugin<Cirrious.MvvmCross.Plugins.ThreadUtils.WindowsPhone.Plugin>();
            registry.AddConventionalPlugin<Cirrious.MvvmCross.Plugins.Location.WindowsPhone.Plugin>();
            registry.AddConventionalPlugin<Cirrious.MvvmCross.Plugins.Visibility.WindowsPhone.Plugin>();
            base.AddPluginsLoaders(registry);
        }

        protected override void InitializeLastChance()
        {
            Cirrious.MvvmCross.Plugins.Visibility.PluginLoader.Instance.EnsureLoaded();

            base.InitializeLastChance();
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

- to make this compile, you will need to add inheritance on the App.xaml class to the interface `IMvxServiceConsumer` and you will need to include some using statements:

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

You need to create special converters for WindowsPhone. Create a folder NativeConverters. Inside this folder create the classes StringLengthValueConverter and StringReverseValueConverter. They should look like this:

```
using Cirrious.MvvmCross.WindowsPhone.Platform.Converters;

namespace Tutorial.UI.WindowsPhone.NativeConverters
{
    public class StringLengthValueConverter : MvxNativeValueConverter<Core.Converters.StringLengthValueConverter>
    {
    }
}
```

```
using Cirrious.MvvmCross.WindowsPhone.Platform.Converters;

namespace Tutorial.UI.WindowsPhone.NativeConverters
{
    public class StringReverseValueConverter : MvxNativeValueConverter<Core.Converters.StringReverseValueConverter>
    {
    }
}
```

Now, just the house keeping to go:

- add the Converters as a resource to the App.xaml:
```
    <Application.Resources>
        <NativeConverters:StringReverseValueConverter x:Key="StringReverseValueConverter"/>
        <NativeConverters:StringLengthValueConverter x:Key="StringLengthValueConverter"/>
    </Application.Resources>
```

where the namespace import is: `xmlns:NativeConverters="clr-namespace:Tutorial.UI.WindowsPhone.NativeConverters"`

That's it - the app should now run...

![Add solution](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/w4.png)

And the ViewModel text is now editable (although the way the WP7 TextBox works uses focus as the Change trigger)

![Add solution](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/w5.png)