## Building a new MvvmCross application

We start with WP7 or with MonoDroid... mainly because we find the Visual Studio tooling is the easiest to use, especially with Resharper installed.

### Create a new empty Solution

In this case we will just call it "Tutorial"

![Add solution](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/1.png)

Add the existing projects Cirrious.Mvvm.Android and Cirrious.Mvvm.Binding.Android to your solution (later this step will be replaced with simlpy adding assembly references instead of source code inconclusion)


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

Also, to make the converters easier to identify, add a simple static class like:

```
namespace Tutorial.Core.Converters
{
    public static class Converters
    {
        public static readonly StringLengthValueConverter StringLength = new StringLengthValueConverter();
        public static readonly StringReverseValueConverter StringReverse = new StringReverseValueConverter();
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

Add references to the new project for: 

- Cirrious.MvvmCross.Android
- Cirrious.MvvmCross.Binding.Android
- Tutorial.Core

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

            var staticFiller = new MvxStaticBasedValueConverterRegistryFiller(registry);
            staticFiller.AddStaticFieldConverters(typeof(Converters));
        }
```

That's it... the app should now run and you should be able to tap down into the detail view which will show you something like:

![One text property bound to 4 Views](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/6.png)
![After some editing](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/7.png)

And when you edit the text in the edit field then you should find all the other fields auto-update: