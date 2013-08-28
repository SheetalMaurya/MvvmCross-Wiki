MvvmCross provides a plugin system to allow developers to provide IoC-injectable functionality at run-time.

This functionality can include both portable and platform-specific code, and can easily be substituted for mock implementations during tests.

Plugins are really just a layer on top of MvvmCross's IoC/Service Resolution implementation - they use a filename-based convention to make it easier to share cross-platform blocks of functionality.

For more background on the MvvmCross IoC and Service Resolution APIs, see [Service Location and Inversion of Control](https://github.com/slodge/MvvmCross/wiki/Service-Location-and-Inversion-of-Control).

### General Introductions

This article is long and detailed.

For more gentle introductions to using and building plugins, see:

- the N+1 video on `Location` and `Messenger` - http://slodge.blogspot.co.uk/2013/05/n9-getting-message-n1-days-of-mvvmcross.html
- the N+1 video on IoC and Plugins - http://slodge.blogspot.co.uk/2013/06/n31-injection-platform-specific.html


### FAQ: Why use lots of small plugins?

This question is asked quite frequently.

Why does MvvmCross contain lots of small plugins rather than just including the functionality within the core package? I quite frequently hear the (valid) complaint that Mvx would be easier to use if there weren't so many individual assemblies to reference and so many namespaces to use. 

The motivation for using lots of separate small modules for this is partly to do with good software architecture, and partly to do with performance.

On the architectural side, providing small self-contained (tightly-coupled) plugins as individual modules makes them easier to write, easier to modify, easier to test and easier to replace.

On the performance side, making the plugins optional means that less unnecessary code is required in apps both at build time and at startup time. If an app doesn't need a module - for example, geo-location - then it simply doesn't reference that plugin.

As for the main complaint - about the referencing of so many assemblies - I do agree that the need to reference additional plugins can make 'getting started' with MvvmCross more difficult than the v1 of MvvmCross where all plugin-type functionality was baked into a single assembly for each platform. However, I hope this difficulty can be lessened through the use of package managers like nuget and the Xamarin component store, and I believe that once over the 'getting started' hump, then the plugins deliver significant benefits which were worth any initial pain.

Further, because MvvmCross itself is so heavily based on replaceable, extensible plugins, I hope that this will encourage others to author, modify and share additional components and extensions, and that these components will be shareable with other platforms beyond MvvmCross.

### How plugins are loaded

Each plugin load is normally initiated from a bootstrap class. These bootstrap classes are added in your UI projects and generally look something like:

    public class ColorPluginBootstrap
        : MvxPluginBootstrapAction<Color.PluginLoader>
    {
    }

During the early stages of `Initialise` within the `Setup` of your app, MvvmCross uses Reflection on your UI Assembly to enumerate these BootStrap actions. For each one found, MvvmCross creates an instance and then calls `Run` on that instance. For an `MvxPluginBootstrapAction` this causes the Bootstrap to look for the `IMvxPluginManager` and to request the load of a specific plugin.

**Note:** on AoT platforms (i.e. MonoTouch/Xamarin.iOS) the bootstrap class used is slightly different - to assist the AoT process, it references classes from both the PCL and the platform specific Assemblies for the plugin like:

    public class ColorPluginBootstrap
        : MvxLoaderPluginBootstrapAction<Color.PluginLoader, Color.Touch.Plugin>
    {
    }


The `IMvxPluginManager` is supplied by each MvvmCross platform. The work the `PluginManager` does to load a plugin is:

- it gets hold of the 'Core' Portable Class Library `Assembly` for the plugin by loading that Assembly (by file name) into the App's AppDomain.
- it uses Reflection to look for the static `PluginLoader.Instance` within the loaded `Assembly`
- this `Instance` may support `IMvxConfigurablePluginLoader` - if it does, then the PluginManager will look for a configuration for the PluginLoader in the UI projects `Setup`.
- this `Instance` will always support `IMvxPluginLoader`. This interface has a single method `void EnsureLoaded()` - which the MvvmCross plugin manager calls.
- the implementation of `EnsureLoaded()` is specific to each  plugin, but generally they perform actions like:
  - sometimes, especially for plugins which are PCL only, they use the `Mvx` IoC service registry to directly register implementations of interfaces
  - sometimes, especially for plugins which have native extensions to the PCL core, `EnsureLoaded` will call back to the PluginManager to ask for the platform-specific assemblies to be loaded using code like:
 
	       var manager = Mvx.Resolve<IMvxPluginManager>();
	       manager.EnsurePlatformAdaptionLoaded<PluginLoader>();
 
  - sometimes, the plugins will also register with IoC for Callbacks when parts of MvvmCross are started. For example, the Color plugin calls  `Mvx.CallbackWhenRegistered<IMvxValueConverterRegistry>(callback)` in order to get notified when the ValueConverterRegistry is available (the Color plugin uses this callback to register its Color ValueConverters - see [Mvx Color Value Conversion](https://github.com/slodge/MvvmCross/wiki/Value-Converters#the-mvx-color-valueconverters)). 
- if the manager is called back with an `EnsurePlatformAdaptionLoaded<TPlugin>()` call, then the manager will attempt to load up a platform-specific Assembly:

  -  it finds this using the core filename plus an extension. For example, if the core PCL plugin Assembly is called `Acme.MyPlugin.dll` then the Manager knows that the Droid adaption for that plugin will be called `Acme.MyPlugin.Droid.dll`
  - within this loaded platform specific Assembly, the Manager looks for a class named `Plugin` which implements the `IMvxPlugin` interface
  - when it finds `Plugin` it then creates a new instance of that class.
  - if the instanciated `Plugin` supports `IMvxConfigurablePlugin` then the PluginManager will look for a configuration from `Setup` for the Plugin.
  - the PluginManager then calls `Load()` on this instance.  
  - during this `Load()` call, the Plugin itself typically registers platform specific implementations of services with the `Mvx` IoC container.

This sounds complicated, but in practice writing a plugin is quite straight-forward - it just requires plugin developers to follow the Assembly naming convention and to add the small `PluginLoader` and `Plugin` standard classes. 

### Writing a plugin

#### Writing a pure-PCL plugin

If you want to write a plugin which only contains portable code, then you only need to provide a single PCL assembly.

For example, to create a plugin which provides an `INumberGenerator` service, then you could create a single PCL Assembly containing:

    public interface INumberGenerator
    {
        double Next();
    }
    
    public class NumberGenerator : INumberGenerator
    {
        private readonly Random _random = new Random();
        
        public double Next()
        {
            return _random.Next();
        }
    }
    
    public class PluginLoader
        : IMvxPluginLoader
    {
        public static readonly PluginLoader Instance = new PluginLoader();

        private bool _loaded;

        public void EnsureLoaded()
        {
            if (_loaded)
            {
                return;
            }

            _loaded = true;
            Mvx.RegisterType<INumberGenerator, NumberGenerator>();
        }
    }

    
#### Writing a PCL+native plugin

If you want to write a plugin which contains both portable and platform specific code, then you need to provide a PCL assembly, plus a platform specific assembly for each platform you are supporting.

For example, to create a plugin which provides an `IVibrate` service, then you could first create the PCL Assembly - say `Acme.Vibrate.dll` - containing:

    public interface IVibrate
    {
        void Shake();
    }

    public class PluginLoader
        : IMvxPluginLoader
    {
        public static readonly PluginLoader Instance = new PluginLoader();

        public void EnsureLoaded()
        {
            var manager = Mvx.Resolve<IMvxPluginManager>();
            manager.EnsurePlatformAdaptionLoaded<PluginLoader>();
        }
    }
    
Then on each platform you would provide a platform specific implementation - e.g. on Android you could provide an Assembly `Acme.Vibrate.Droid.dll` containing:

    public class DroidVibrate : IVibrate
    {
        public void Shake()
        {
            var globals = Mvx.Resolve<Cirrious.CrossCore.Droid.IMvxAndroidGlobals>();
            var vibrator = globals.ApplicationContext
                                  .GetSystemService(Context.VibratorService)
                                  as Vibrator;
            vibrator.Vibrate(100L);
        }
    }
    
    public class Plugin
        : IMvxPlugin
    {
        public void Load()
        {
            Mvx.RegisterSingleton<IVibrate>(new DroidVibrate());
        }
    }    

**Note:** the filename (and corresponding namespace) used is important to ensuring the plugin can be loaded. If the core plugin is `Acme.Vibrate.dll`, then the platform extensions should be:

- Android -`Acme.Vibrate.Droid.dll` 
- iOS - `Acme.Vibrate.Touch.dll` 
- WindowsPhone - `Acme.Vibrate.WindowsPhone.dll` 
- WindowsStore - `Acme.Vibrate.WindowsStore.dll` 
- Wpf - `Acme.Vibrate.Wpf.dll` 

#### Writing a configurable plugin

To make a plugin configurable, you can optionally either change the PluginLoader or the Plugin implementations to support `IMvxConfigurablePluginLoader` or `IMvxConfigurablePlugin`

These two interfaces both provide a configuration method:

    void Configure(IMvxPluginConfiguration configuration);

In order to call this `Configure` method, the MvvmCross `PluginManager` will ask for a configuration using the `GetPluginConfiguration` method in your app's `Setup` class.

For example, if we wanted to provide a configuration for the Droid Acme.Vibrate plugin module we could do this using:

    public class DroidVibrate : IVibrate
    {
        public long ShakeDurationInMs { get; set; }
        
        public void Shake()
        {
            var globals = Mvx.Resolve<Cirrious.CrossCore.Droid.IMvxAndroidGlobals>();
            var vibrator = globals.ApplicationContext
                                  .GetSystemService(Context.VibratorService)
                                  as Vibrator;
            vibrator.Vibrate(ShakeDurationInMs);
        }
    }
    
    public class DroidVibrateConfiguration 
        : IMvxPluginConfiguration
    {
        public ShakeDurationInMs { get; set;}
    }
    
    public class Plugin
        : IMvxPlugin
    {
        private _shakeDurationInMs = 100L;
        
        public void Configure(IMvxPluginConfiguration configuration)
        {
            if (configuration == null)
                return;
                
            var droidConfiguration = (DroidVibrateConfiguration)configuration;
            _shakeDurationInMs = droidConfiguration.ShakeDurationInMs;
        }

        public void Load()
        {
        	  var instance = new DroidVibrate();
        	  instance.ShakeDurationInMs = _shakeDurationInMs;
            Mvx.RegisterSingleton<IVibrate>(instance);
        }
    }    

This configuration could then be provided in `Setup` as:

        protected override IMvxPluginConfiguration GetPluginConfiguration(Type plugin)
        {
            if (plugin == typeof(Acme.Vibrate.Droid.Plugin))
            {
                return new Acme.Vibrate.Droid.DroidVibrateConfiguration()
                {
                    ShakeDurationInMs = 10L
                };
            }
            
            return null;
        }


### Distributing a plugin

The easiest mechanism for distributing the plugin is to use a nuget package. 

The nuget packaging helps ensure the correct references are added to each UI project type and assists with adding the bootstrap files too.

For our `Acme.Vibrate` plugin, a sample nuget package file might be:

    <?xml version="1.0" encoding="utf-8"?>
	 <package xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
	 	<metadata>
			<id>Acme.Vibrate</id>
			<version>1.0</version>
			<title>Acme Vibration (mvvmcross plugin)</title>
			<authors>Acme</authors>
			<owners>User</owners>
			<licenseUrl>http://opensource.org/licenses/ms-pl.html</licenseUrl>
			<projectUrl>https://github.com/Acme/Acme.Vibrate</projectUrl>
			<requireLicenseAcceptance>false</requireLicenseAcceptance>
			<description>This package contains the 'Vibrate' plugin for MvvmCross v3.</description>
			<tags>mvvm mvvmcross xamarin monodroid monotouch wpf windows8 winrt netcore wp wpdev windowsphone</tags>
			<dependencies>
			  <dependency id="MvvmCross.HotTuna.CrossCore" version="3.0.10" />
			</dependencies>
		</metadata>
		<files>		
			<!-- PCL -->
			<file src="..\bin\Release\Acme.Vibrate.Color.dll" target="lib\portable-win+net45+MonoAndroid16+MonoTouch40+sl40+wp71\Acme.Vibrate.dll" />

	    	<!-- wpf  -->
			<file src="..\bin\Release\Acme.Vibrate.dll" target="lib\net45\Acme.Vibrate.dll" />
			<file src="..\bin\Release\Acme.Vibrate.Wpf.dll" target="lib\net45\Acme.Vibrate.Wpf.dll" />
	        <file src="BootstrapContent\VibratePluginBootstrap.cs.pp" target="content\net45\Bootstrap\VibratePluginBootstrap.cs.pp" />
					
			<!-- store -->
			<file src="..\bin\Release\Acme.Vibrate.Color.dll" target="lib\netcore45\Acme.Vibrate.Color.dll" />
			<file src="..\bin\Release\Acme.Vibrate.WindowsStore.dll" target="lib\netcore45\Acme.Vibrate.WindowsStore.dll" />
	        <file src="BootstrapContent\VibratePluginBootstrap.cs.pp" target="content\netcore45\Bootstrap\VibratePluginBootstrap.cs.pp" />
	        
			<!-- phone -->
			<file src="..\bin\Release\Acme.Vibrate.dll" target="lib\sl4-wp71\Acme.Vibrate.dll" />
			<file src="..\bin\Release\Acme.Vibrate.WindowsPhone.dll" target="lib\sl4-wp71\Acme.Vibrate.WindowsPhone.dll" />
	        <file src="BootstrapContent\VibratePluginBootstrap.cs.pp" target="content\sl4-wp71\Bootstrap\VibratePluginBootstrap.cs.pp" />
		
			<!-- droid -->
			<file src="..\bin\Release\Acme.Vibrate.dll" target="lib\MonoAndroid16\Acme.Vibrate.dll" />
			<file src="..\bin\Release\Acme.Vibrate.Droid.dll" target="lib\MonoAndroid16\Acme.Vibrate.Droid.dll" />
	        <file src="BootstrapContent\VibratePluginBootstrap.cs.pp" target="content\MonoAndroid16\Bootstrap\VibratePluginBootstrap.cs.pp" />

			<!-- touch -->
			<file src="..\bin\Release\Acme.Vibrate.dll" target="lib\MonoTouch40\Acme.Vibrate.dll" />
			<file src="..\bin\Release\Acme.Vibrate.dll" target="lib\MonoTouch40\Acme.Vibrate.Touch.dll" />
	        <file src="TouchBootstrapContent\VibratePluginBootstrap.cs.pp" target="content\MonoTouch40\Bootstrap\VibratePluginBootstrap.cs.pp" />
			
		</files>
	</package>

where the two bootstrap files are:

    // Bootstrap/VibratePluginBootstrap.cs
    using Cirrious.CrossCore.Plugins;

	 namespace $rootnamespace$.Bootstrap
	 {
	    public class VibratePluginBootstrap
	        : MvxPluginBootstrapAction<Acme.Vibrate.PluginLoader>
	    {
	    }
	 }

and

    // TouchBootstrap/VibratePluginBootstrap.cs
    using Cirrious.CrossCore.Plugins;

    namespace $rootnamespace$.Bootstrap
    {
	    public class VibratePluginBootstrap
	        : MvxLoaderPluginBootstrapAction<Acme.Vibrate.PluginLoader, Acme.Vibrate.Plugin>
	    {
	    }
    }


## Existing MvvmCross plugins

Cirrious - the team behind MvvmCross - has provided a number of plugins for use with the platform.

These plugins are 'normal plugins' - none of them use hidden APIs or have any special access to MvvmCross internals.  In the plugin architecture, all plugins are equal - third party plugins have exactly the same opportunities and capabilities as the 'official' MvvmCross plugins.

All of the MvvmCross plugins exist because they've been required for individual customer projects. Because of this 'specific use case' history, the APIs of some of the plugins are not always general purpose - instead they do sometimes have their 'app-specific-quirks'.

Please do consider using the Cirrious MvvmCross plugins in your apps - but also please also consider writing your own or writing changes and extensions to these plugins to meet your organisation's and your own app's specific needs - "The App Is The King". 

Plugins are there to offer flexibility - so please don't feel limited to the 'official' plugin set.

### Accelerometer

The `Accelerometer` plugin provides access to a platforms accelerometer using a singleton implementing the API:

    public interface IMvxAccelerometer
    {
        void Start();
        void Stop();
        bool Started { get; }
        MvxAccelerometerReading LastReading { get; }
        event EventHandler<MvxValueEventArgs<MvxAccelerometerReading>> ReadingAvailable;
    }
    
This plugin is available on all of Android, iOS, WindowsPhone, WindowsStore and Wpf.

The Wpf implementation is an empty implementation - so it's not really supported there.

All implementations are 'intro level' only implementations. You may find that you need to add additional code to, for example, get consistent x,y,z readings across all platforms or to handle errors on devices where the accelerometer is not currently available.

Current advice (August 2013): if your app requires accelerometer support, consider this plugin as a good open source starting point - but do test the implementation on several devices to ensure it suits your app - consider developing the code further (and consider contributing your changes back to improving this plugin too!).

### Bookmarks

The `Bookmarks` plugin provides a simple 'live tile' service for WindowsPhone only.

    public interface IMvxBookmarkLibrarian
    {
        bool HasBookmark(string uniqueName);

        bool AddBookmark(Type viewModelType, string uniqueName, MvxBookmarkMetadata metadata,
                         IDictionary<string, string> navigationArgs);

        bool UpdateBookmark(string uniqueName, MvxBookmarkMetadata metadata);
    }
    
Current advice (August 2013): if your app requires cross-platform live-tile support, consider this plugin as an open source example only.    

### Color

The `Color` plugin provides native implementations for conversion from the cross-platform `MvxColor` to platform-specific `Color` implementations.

    public interface IMvxNativeColor
    {
        object ToNative(MvxColor mvxColor);
    }

This plugin is available on all platforms.

This plugin also provides a number of useful Color-outputting ValueConverters - see [Mvx Color Value Conversion](https://github.com/slodge/MvvmCross/wiki/Value-Converters#the-mvx-color-valueconverters) for information on these. If you wish to create your own Color ValueConverters, then this plugin provides the base classes `MvxColorValueConverter` and `MvxColorValueConverter<T>`.

The Android version of this plugin also registers some Binding Targets for use with Color 

- `BackgroundColor` for any Android `View` 
- `TextColor` for any Android `TextView`

### DownloadCache

The `DownloadCache` plugin provides a managed disk and memory cache for downloaded files - especially images.

The DownloadCache plugin is only available for iOS and Android, although others have ported this to Windows platforms too.

For the DownloadCache plugin to fully work, implementations must be registered for:

- `IMvxTextSerializer`
- `IMvxFileStore`

One relatively easiest way to register implementations for these interfaces is to load the Json and File plugins.

The most common use of the DownloadCache plugin is for downloading images using in `MvxImageView` in Android and iOS. This is managed automatically using the `IMvxImageHelper<Bitmap>` and `<IMvxImageHelper<UIImage>` helpers.

The caches used by these helpers are configured `MvxDownloadCacheConfiguration` configuration classes. The default configurations store up to 500 files for up to 7 days, and will maintain up to 4MB of up to 30 images in RAM. To override the default values, provide custom settings in `GetPluginConfiguration` in your `Setup` class.

Known issues:

- this cache is a complicated implementation, has been well tested in apps, but is poorly unit tested currently
- one user has reported MonoTouch download issues in certain network conditions - these problems seem to be related to known MonoTouch issues - see [StackOverflow Q&A on this](http://stackoverflow.com/questions/17238809/mvxdynamicimagehelper-unreliable). That first StackOverflow post has a suggested workaround - overriding the `IMvxHttpFileDownloader` registration with an implementation which uses the `UIImage.LoadFromData` method.

### Email

The `Email` plugin provides a cross-platform implementation for sending emails:

    public interface IMvxComposeEmailTask
    {
        void ComposeEmail(string to, string cc, string subject, string body, bool isHtml);
    }
    
The Email plugins is supported on all platforms.

The implementation on Windows Store and Wpf is very simplistic - using only 'mailto:' url-open requests.

To send an email you can use:

    Mvx.Resolve<IMvxComposeEmailTask>()
       .ComposeEmail("me@slodge.com", 
                     string.Empty, 
                     "MvvmCross Email",
                     "I <3 MvvmCross",
                     false);
                   
### FieldBinding

The `FieldBinding` plugin is part of the `Rio` binding extensions for MvvmCross.

The FieldBinding plugin is a pure PCL plugin - it contains only PCL assemblies.

When FieldBinding is loaded, then MvvmCross data-binding:

- can use fields as well as properties for binding.
- can use `INotifyChanged` for dynamic fields.

An example, Rio-based ViewModel using both FieldBinding and MethodBinding is:

    public class FirstViewModel
       : MvxViewModel
    {
        private readonly IDataStore _dataStore;
        
        public FirstViewModel(IDataStore dataStore)
        {
            _dataStore = dataStore;
        }
        
        public void Init(int id)
        {
            var person = _dataStore.Get<Person>(id);
            Id.Value = id;
            FirstName.Value = person.FirstName;
            LastName.Value = person.LastName;
        }
        
        public readonly INC<int> Id = new NC<int>();
        public readonly INC<string> FirstName = new NC<string>();
        public readonly INC<string> LastName = new NC<string>();
        
        public void Save()
        {
            var person = _dataStore.Get<Person>(id);
            person.FirstName = FirstName.Value;
            person.LastName = LastName.Value;
            _dataStore.Update(person);
            Close(this);
        }
    }

The field in this class could be accessed using Android syntax:

    <TextView
        android:layout_width='fill_parent'
        android:layout_height='wrap_content'
        local:MvxBind='Text FirstName' />

    <TextView
        android:layout_width='fill_parent'
        android:layout_height='wrap_content'
        local:MvxBind='Text LastName' />

For more on Rio FieldBinding see the [N=36 video](http://slodge.blogspot.co.uk/2013/07/n36-rio-binding-carnival.html)

### File

The `File` plugin provides cross-platform access to a File Store API:

    public interface IMvxFileStore
    {
        bool TryReadTextFile(string path, out string contents);
        bool TryReadBinaryFile(string path, out Byte[] contents);
        bool TryReadBinaryFile(string path, Func<Stream, bool> readMethod);
        void WriteFile(string path, string contents);
        void WriteFile(string path, IEnumerable<Byte> contents);
        void WriteFile(string path, Action<Stream> writeMethod);
        bool TryMove(string from, string to, bool deleteExistingTo);
        bool Exists(string path);
        bool FolderExists(string folderPath);
        string PathCombine(string items0, string items1);
        string NativePath(string path);

        void EnsureFolderExists(string folderPath);
        IEnumerable<string> GetFilesIn(string folderPath);
        void DeleteFile(string path);
        void DeleteFolder(string folderPath, bool recursive);
    }

This plugin is implemented on all platforms - except WindowsStore where the `Folder` APIs are currently unimplemented.

By defautlt, the plugin reads and writes files in paths relative to:

- Android - `Context.FilesDir`
- iOS - `Environment.SpecialFolder.MyDocuments`
- WindowsPhone - app-specific isolated storage
- WindowsStore - `Windows.Storage.ApplicationData.Current.LocalFolder.Path`
- Wpf - `Environment.SpecialFolder.ApplicationData`

Note: while it works, the use of a synchronous API for File IO on WindowsStore applications is slightly 'naughty'. It's likely that an asynchronous version of the IMvxFileStore interface will be provided in the near future.

### Json

The `Json` plugin provides a wrapper for a PCL version of the NewtonSoft Json.Net library allowing it to support:

    public interface IMvxJsonConverter 
        : IMvxTextSerializer
    {
    }

    public interface IMvxTextSerializer
    {
        T DeserializeObject<T>(string inputText);
        string SerializeObject(object toSerialise);
        object DeserializeObject(Type type, string inputText);
    }

The Serialize and Deserialize implementations used are 'default' Json.Net implementations with `Formatting.None` specified along with the settings:

    new JsonSerializerSettings
    {
        Converters = new List<JsonConverter>
            {
                new MvxEnumJsonConverter(),
            },
        DateFormatHandling = DateFormatHandling.IsoDateFormat,
    }
 
The version of Json.Net referenced is an old PCL version - 4.5.4.14825

Json.Net is a popular library and it may be that the use of this old PCL version will conflict with another library your app is using. If this is the case, then it should be relatively straight-forward to **not** load the Json plugin, and to instead build your own implementations of `IMvxTextSerializer` and `IMvxJsonConverter`.

The Json plugin can optionally be requested **not** to register Json as `IMvxTextSerializer` using the configuration in Setup.cs:


        protected override IMvxPluginConfiguration GetPluginConfiguration(Type plugin)
        {
            if (plugin == typeof(Cirrious.MvvmCross.Plugins.Json.PluginLoader))
            {
                return new Cirrious.MvvmCross.Plugins.Json.MvxJsonConfiguration()
                {
                    RegisterAsTextSerializer = false
                };
            }
            
            return null;
        }

The Json plugin can be used:

        public class ExampleObject
        {
            public string Name { get; set; }
            public DateTime DateOfBirth { get; set; }
        }

        var serializer = Mvx.Resolve<IMvxJsonConverter>();
        
        var exampleObject = new ExampleObject()
        {
             Name = "Fred bloggs",
             DateOfBirth = new DateTime(1972,7,13)
        };
        
        var jsonText = serializer.SerializeObject(exampleObject);
        
        var deserialized = serializer.DeserializeObject<ExampleObject>(jsonText);
        
Note that other Json libraries are available for .Net - in particular, several people have recommended the ServiceStack.Text library which is available on many platforms.

### JsonLocalisation

The `JsonLocalisation` plugin provides a number of support classes to help load Json language text files for internationalization (i18n).

The `JsonLocalisation` plugin is a single PCL Assembly and isn't really a typical plugin - it doesn't itself register any singletons or services with the MvvmCross IoC container.

For advice on using the JsonLocalisation library, see:

- the Babel sample - https://github.com/slodge/MvvmCross-Tutorials/tree/master/Babel
- the N+1 video N=21 which discusses how i18n is built - http://slodge.blogspot.co.uk/2013/05/n21-internationalisation-i18n-n1-days.html

Notes:

- the standard JsonLocalisation implementation relies on the `ResourceLoader` and `Json` plugins for loading Json files from the application package contents.
- several alternative Localisation implementations have been suggested including:
  - using Microsoft Resx files - possibly while also using Vernacular from Rdio. For a detailed blog post on using Resx files in MvvmCross, see http://opendix.blogspot.co.uk/2013/05/using-resx-files-for-localization-in.html
  - using EmbeddedResources to store the Json files in the Core PCL assembly (see https://github.com/slodge/MvvmCross/issues/55)
  - using a single JSON file or a single CSV (Comma Separated Variables) file to store all languages in one single file (again see https://github.com/slodge/MvvmCross/issues/55)
  
  we *hope* that some of these alternatives will become open source realities in the future
  

### Location

The `Location` plugin provides access to GeoLocation (typically GPS) functionality via the API:

    public interface IMvxGeoLocationWatcher
    {
        void Start(
            MvxGeoLocationOptions options, 
            Action<MvxGeoLocation> success, 
            Action<MvxLocationError> error);
        void Stop();
        bool Started { get; }
    }

The `Location` plugin is implemented on all platforms EXCEPT Wpf.

Because of the `Action` based nature of the `IMvxGeoLocationWatcher` API, it's generally best **not** to use this interface directly inside ViewModels, but instead to use the API in a singleton service which can then send Messages to your ViewModels.

An example implementation of such a service is:

    public class LocationService
        : ILocationService
    {
        private readonly IMvxGeoLocationWatcher _watcher;
        private readonly IMvxMessenger _messenger;

        public LocationService(IMvxGeoLocationWatcher watcher, IMvxMessenger messenger)
        {
            _watcher = watcher;
            _messenger = messenger;
            _watcher.Start(new MvxGeoLocationOptions(), OnLocation, OnError);
        }

        private void OnLocation(MvxGeoLocation location)
        {
            var message = new LocationMessage(this,
                                              location.Coordinates.Latitude,
                                              location.Coordinates.Longitude
                );

            _messenger.Publish(message);
        }

        private void OnError(MvxLocationError error)
        {
            Mvx.Error("Seen location error {0}", error.Code);
        }
    }

For a good walk-through of using the location plugin, including using it in tandem with the MvvmCross messenger, see both N=8 and N=9 in N+1 videos of MvvmCross - [N+1 videos](https://github.com/slodge/MvvmCross/wiki/N-1-Videos-Of-MvvmCross)


Notes:

- the `MvxGeoLocationOptions` object passed into the Start method provides a number of options like `EnableHighAccuracy` - not all of these options are well implemented on all platforms
- the default implementation is a good 'general' module if you just need 'location information' in your app. If your app requires more - e.g. control over time and distance tracking, geo-fencing, etc - then consider building your own plugin (or injecting your own service from the UI project on each platform). The source code for the Location plugin should provide you with a good starting place for this.
- it's not unusual for Android developers to hit issues with location detection on different phones and on different Android - check StackOverflow and Issues for questions and answers - e.g. https://github.com/slodge/MvvmCross/issues/360
- the MvvmCross coordinates object - MvxCoordinates - does not currently come with any built-in maths operations. Algorithms for some common coordinate operations can be found on (for example) http://slodge.blogspot.co.uk/2012/04/calculating-distance-between-latlng.html.

### Messenger

The MvvmCross `Messenger` plugin provides an Event aggregation Messenger which is biased towards using Weak references for event subscription.

The `IMvxMessenger` API includes:

- publishing methods:
  - `Publish`
- subscription methods:
  - `Subscribe`, `SubscribeOnMainThread`, `SubscribeOnThreadPoolThread`, `Unsubscribe`
- subscription state observation methods:
  - `HasSubscriptionsFor`, `HasSubscriptionsForTag`, `CountSubscriptionsFor`, `CountSubscriptionsForTag`, `GetSubscriptionTagsFor`
- clearup methods:
  - `RequestPurge`, `RequestPurgeAll`  



The basic use of the `Messenger` is:

- define one or more Message classes for communication between components. These should inherit from `MvxMessage` - e.g.:

	    public class LocationMessage
	        : MvxMessage
	    {
	        public LocationMessage(object sender, double lat, double lng) 
	            : base(sender)
	        {
	            Lng = lng;
	            Lat = lat;
	        }
	
	        public double Lat { get; private set; }
	        public double Lng { get; private set; }
	    }

- define the classes which will create and send these Messages - e.g. a `LocationService` might create and send `LocationMessage`s using

       var message = new LocationMessage(
           this,
           location.Coordinates.Latitude,
           location.Coordinates.Longitude
           );

       _messenger.Publish(message);

- define the classes which will subscribe to and receive these messages. Each of these classes must call one of the `Subscribe` methods on the `IMvxMessenger` and **must store the returned token**. For example part of a ViewModel receivin `LocationMessage`s might look like:

	    public class LocationViewModel 
			: MvxViewModel
	    {
	        private readonly MvxSubscriptionToken _token;
	
	        public LocationViewModel(IMvxMessenger messenger)
	        {
	            _token = messenger.Subscribe<LocationMessage>(OnLocationMessage);
	        }
	
	        private void OnLocationMessage(LocationMessage locationMessage)
	        {
	            Lat = locationMessage.Lat;
	            Lng = locationMessage.Lng;
	        }
	        
	        // remainder of ViewModel
	    }

The three different options for subscribing for messages differ only in terms of which thread messages will be passed back on:

- `Subscribe` - messages will be passed directly on the `Publish` thread. These subscriptions have the lowest processing overhead - messages will always be received synchronously whenever they are published. You should use this type of subscription if you already know which type of thread the Publish will be called on and if you have a good understanding on the resource and UI usage of your message handler.
- `SubscribeOnMainThread` - any message published on a background thread will be marshalled to the main UI thread.  This type of subscription is ideal if your message handler needs to perform some resource-unintensive task which involves interacting with the UI.
- `SubscribeOnThreadPoolThread` - messages will always be queued for thread pool processing. This always involves an asynchonous post - even if the message is published on an existing ThreadPool thread. This type of subscription is ideal if your message handler needs to perform some resource-intensive task as it won't block the UI, nor the message publisher.

All subscription methods have two additional parameters:

- `MvxReference reference = MvxReference.Weak` - specify `MvxReference.Strong` if you would prefer to use `Strong` references - in this case, the Messenger will keep a strong reference to the callback method and Garbage Collection will not be able to remove the subscription.
- `string tag = null` - an optional `tag` which allows code to inspect what Message listeners are currently listening for - see 'observe the current subscription status' below.

Subscriptions can be cancelled at any time using the `Unsubscribe` method on the `IMvxMessenger` or by calling `Dispose()` on the subscription token.

However, in many cases, `Unsubscribe`/`Dispose` is never called. Instead listeners rely on the `WeakReference` implementation of the  `MvxSubscriptionToken` to clear up the subscription when objects go out of scope and Garbage Collection occurs.

This GC-based unsubscription will occur whenever the subscription token returned from `Subscribe` is Garbage Collected - so if the token is **not** stored, then unsubscription may occur immediately - e.g. in this method

     public void MayNotEverReceiveAMessage()
     {
         var token = _messenger.Subscribe<MyMessage>((message) => {
             Mvx.Trace("Message received!");
         });
         // token goes out of scope now 
         // - so will be garbage collected *at some point*
         // - so trace may never get called
     }

For any code wishing to observe the current subscription status on any message type (including subscriptions that have been requested with a named string `tag`) then this can be done:

- using the `HasSubscriptionsFor` and `CountSubscriptionsFor` methods
- by subscribing for `MvxSubscriberChangeMessage` messages - the Messenger itself publishes these `MvxSubscriberChangeMessage` messages whenever subscriptions are made, are removed or have expired.

	    public class MvxSubscriberChangeMessage : MvxMessage
	    {
	        public Type MessageType { get; private set; }
	        public int SubscriberCount { get; private set; }
	
	        public MvxSubscriberChangeMessage(object sender, Type messageType, int countSubscribers = 0) 
	            : base(sender)
	        {
	            SubscriberCount = countSubscribers;
	            MessageType = messageType;
	        }
	    }

These mechanisms allow you to author singleton services which can adapt their resource requirements according to the current needs of the app. 

For example, suppose you have a service which tracks stock prices using calls to a web service. Individual clients might subscribe to Messages from this service for individual stock codes. The stock service can track when subscribers are present for each stock code and can then adjust which network calls it makes.

### MethodBinding

The `MethodBinding` plugin is part of the `Rio` binding extensions for MvvmCross.

The MethodBinding plugin is a pure PCL plugin - it contains only a PCL assembly.

When MethodBinding is loaded, then MvvmCross data-binding:

- can use public methods as well as `ICommand` properties for action/command binding.

An example, Rio-based ViewModel using both FieldBinding and MethodBinding is:

    public class FirstViewModel
       : MvxViewModel
    {
        private readonly IDataStore _dataStore;
        
        public FirstViewModel(IDataStore dataStore)
        {
            _dataStore = dataStore;
        }
        
        public void Init(int id)
        {
            var person = _dataStore.Get<Person>(id);
            Id.Value = id;
            FirstName.Value = person.FirstName;
            LastName.Value = person.LastName;
        }
        
        public readonly INC<int> Id = new NC<int>();
        public readonly INC<string> FirstName = new NC<string>();
        public readonly INC<string> LastName = new NC<string>();
        
        public void Save()
        {
            var person = _dataStore.Get<Person>(id);
            person.FirstName = FirstName.Value;
            person.LastName = LastName.Value;
            _dataStore.Update(person);
            Close(this);
        }
    }
    
The `Save` method in this class could be accessed using Android syntax:

    <Button
        android:layout_width='fill_parent'
        android:layout_height='wrap_content'
        android:text='Save'
        local:MvxBind='Click Save' />
       

For more on Rio MethodBinding see N=36 on http://slodge.blogspot.co.uk/2013/07/n36-rio-binding-carnival.html

### Network

The original purpose of the `Network` plugin was to provide `IMvxReachability` **on iOS only**

    public interface IMvxReachability
    {
        bool IsHostReachable(string host);
    }

**Note:** this interface is currently implemented on iOS only, although some contributors are working on other platforms (e.g. for Android see https://github.com/slodge/MvvmCross/issues/362)

Since this original purpose, Network has now further been expanded to provide a simple Rest implementation - and this is available on Droid, Touch, WindowsPhone, WindowsStore and Wpf.

The Rest client is mainly implemented using the `IMvxRestClient` and `IMvxJsonRestClient` interfaces

    public interface IMvxRestClient
    {
        void ClearSetting(string key);
        void SetSetting(string key, object value);

        void MakeRequest(MvxRestRequest restRequest, Action<MvxRestResponse> successAction,
                         Action<Exception> errorAction);

        void MakeRequest(MvxRestRequest restRequest, Action<MvxStreamRestResponse> successAction,
                         Action<Exception> errorAction);
    }

    public interface IMvxJsonRestClient
    {
        Func<IMvxJsonConverter> JsonConverterProvider { get; set; }

        void MakeRequestFor<T>(MvxRestRequest restRequest, Action<MvxDecodedRestResponse<T>> successAction,
                               Action<Exception> errorAction);
    }

These are supported by a small set of `Request` and `Response` classes:

- `MvxRestRequest`, `MvxStreamRestRequest`, `MvxJsonRestRequest`, `MvxStringRestRequest`, `MvxStringRestRequest`, `MvxMultiPartFormRestRequest` and `MvxWwwFormRequest`
- `MvxRestResponse`, `MvxStreamRestResponse`, `MvxJsonRestResponse`

To make a simple fixed url Rest request, you can use:

    var request = new MvxRestRequest("http://myService.org/things/list");
    var client = Mvx.Resolve<IMvxRestClient>();
    client.MakeRequest(request,
         (MvxStreamRestResponse response) => {
             // do something with the response.StatusCode and response.Stream
         },
         error => {
         	   // do something with the error
         });

To use the Json APIs, you must have an `IMvxJsonConverter` implementation available - one way to get this is to load the Json plugin. With this in place, a simple Json upload with Json response might look like:

    var request = new MvxJsonRestRequest<Person>("http://myService.org/things/add")
    {
        Body = person
    };
    
    var client = Mvx.Resolve<IMvxJsonRestClient>();
    client.MakeRequestFor<PersonAddResult>(request,
         (MakeDecodedRestResponse<PersonAddResult> response) => {
             // do something with the response.StatusCode and response.Result
         },
         error => {
         	   // do something with the error
         });

Note:

- This Rest module is a 'light-weight' implementation which works for many simple Rest web services.
- For more advanced web service requirements, consider extending the classes offered here or consider importing other more established Rest libraries such as RestSharp (http://restsharp.org/).

### PhoneCall

The `PhoneCall` plugin provides implementations of:

    public interface IMvxPhoneCallTask
    {
        void MakePhoneCall(string name, string number);
    }

The PhoneCall plugin is available on Android, iOS, and WindowsPhone with a Skype-based implementation on WindowsStore.

The PhoneCall plugin is very simple - e.g. it doesn't provide any detection of whether or not a phone call is currently possible - e.g. for flight mode or for iPod-type devices without cell connectivity.

Sample using the PhoneCall plugin include:

- CustomerManagement - https://github.com/slodge/MvvmCross-Tutorials/tree/master/Sample%20-%20CustomerManagement
- Conference - https://github.com/slodge/MvvmCross-Tutorials/tree/master/Sample%20-%20CirriousConference


### PictureChooser

The `PictureChooser` plugin provides implementations of:

    public interface IMvxPictureChooserTask
    {
        void ChoosePictureFromLibrary(int maxPixelDimension, int percentQuality, Action<Stream> pictureAvailable,
                                      Action assumeCancelled);

        void TakePicture(int maxPixelDimension, int percentQuality, Action<Stream> pictureAvailable,
                         Action assumeCancelled);
    }

This is available on Android, iOS, WindowsPhone and WindowsStore. 

This interface is designed for single use only - i.e. each time you require a picture you should request a new `IMvxPictureChooserTask` instance.

The interface can be used as:

    var task = Mvx.Resolve<IMvxPictureChooserTask>();
    task.ChoosePictureFromLibrary(500, 90,
           stream => {
               // use the stream
               // expect the stream to be disposed after immediately this method returns.
           },
           () => {
               // perform any cancelled operation
           });

**Note:** Using this interface well on Android is very difficult.

The reason for this is because of Android's Activity lifecyle. The Android lifecycle means that the image that may be returned to a different View and ViewModel than the one that requested it. This is partly because camera apps generally use a lot of RAM (raw camera images are large files) - so while th camera app is capturing you image, then Android may look to free up additional RAM by killing your app's Activity.

If you want to use this `IMvxPictureChooserTask` effectively and reliably on Android then you really need to call this API via a service class, to use Messaging to pass the returned message back to a ViewModel and to implement 'tombstoning' support for that ViewModel.

There is a simple demo for `IMvxPictureChooserTask` in [PictureTaking](https://github.com/slodge/MvvmCross-Tutorials/tree/master/PictureTaking) - however, this simple demo doesn't currently show this full Android technique. 

**Note:** On WindowsPhone, an additional implementation is available:

    public interface IMvxCombinedPictureChooserTask
    {
        void ChooseOrTakePicture(int maxPixelDimension, int percentQuality, Action<Stream> pictureAvailable,
                                 Action assumeCancelled);
    }
 
Client code can test for the availability of this interface using:

     var isAvailable = Mvx.CanResolve<IMvxCombinedPictureChooserTask>();

or:

     IMvxCombinedPictureChooserTask combined;
     var isAvailable = Mvx.TryResolve(out combined);
     
Finally, the `PictureChooser` plugin also provides an "InMemoryImage" ValueConverter - `MvxInMemoryImageValueConverter`. This value converter allows images to be decoded from byte arrays for use on-screen.

The "InMemoryImage" ValueConverter can be seen in use in the PictureTaking sample - see https://github.com/slodge/MvvmCross-Tutorials/tree/master/PictureTaking.

### Reflection

The `Reflection` Plugin was intended to help fill in some of the gaps for the portable Reflection API set.

However, the gaps appear to be genuine (WindowsPhone and especially WindowsStore have different API subsets) - so this plugin is currently not very useful.

Current advice (August 2013): there's no reason to use this plugin.


### ResourceLoader

The `ResourceLoader` plugin provides access to files bundled within the app package.

- On Android, this is for files bundled in the `Assets` folder and marked as Build Action of `AndroidAsset`
- On iOS and Windows, this is for files bundled with a Build Action of `Content` 

On several platforms, the ResourceLoader plugin requires an `IMvxFileStore` is available. One easy way to supply this is to load the `File` plugin.

The main interface supplied by this plugin is:

    public interface IMvxResourceLoader
    {
        bool ResourceExists(string resourcePath);
        string GetTextResource(string resourcePath);
        void GetResourceStream(string resourcePath, Action<Stream> streamAction);
    }
    
For a text file 'Hello.txt' bundled in a folder 'Foo', this can be called as:    

    var loader = Mvx.Resolve<IMvxResourceLoader>();
    var contents = loader.GetTextResource("Foo/Hello.txt");

Samples using the ResourceLoader plugin include:

- Babel - JsonLocalisation - see https://github.com/slodge/MvvmCross-Tutorials/tree/master/Babel
- Conference - the sessions are loaded from Json resources - see https://github.com/slodge/MvvmCross-Tutorials/tree/master/Sample%20-%20CirriousConference

### Share

The `Share` plugin provides implementations of:

    public interface IMvxShareTask
    {
        void ShareShort(string message);
        void ShareLink(string title, string message, string link);
    }
    
This plugin is available on Android, iOS, and WindowsPhone. On WindowsStore, sharing is done more by UI-based sharing (swipe in from the right).

On Android, sharing is done using general Share/Send Intents. This could be improved in future implementations.
On WindowsPhone, sharing is done via the OS level share task. 
On iOS, currently only sharing by linked Twitter account is supported. There is code available to extend this to Facebook - see https://github.com/slodge/MvvmCross/issues/188.

A sample using the Share plugin is:

- Conference - see https://github.com/slodge/MvvmCross-Tutorials/tree/master/Sample%20-%20CirriousConference

### SoundEffects

The `SoundEffects` plugin is only supported on WindowsPhone currently.

It uses the ResourceLoader plugin and allows small SoundEffect files to be played via:

    public interface IMvxSoundEffectLoader
        : IMvxResourceObjectLoader<IMvxSoundEffect>
    {
    }

    public interface IMvxSoundEffect
        : IMvxResourceObject, IDisposable
    {
        IMvxSoundEffectInstance CreateInstance();
    }

    public interface IMvxSoundEffectInstance
        : IDisposable
    {
        void Play();
        void Stop();
    }

Current advice (August 2013): this plugin isn't really useful outside of WindowsPhone today - hopefully some project will come along soon that needs this implemented on more platforms.

### Sqlite

The `Sqlite` plugin provide local Sqlite storage via the a modified version of the Sqlite-net library.

The version of this library MvvmCross forked from is at https://github.com/praeclarum/sqlite-net/.

In SQLite-net, database entities are mapped into RAM using ORM classes like:

    public class CollectedItem
    {
        [PrimaryKey, AutoIncrement]
        public int Id { get; set; }
        
        public string Caption { get; set; }
        public string Notes { get; set; }

        public DateTime WhenUtc { get; set; }
        
        public bool LocationKnown { get; set; }
        public double Lat { get; set; }
        public double Lng { get; set; }

        public string ImagePath { get; set; }
    }

The MvvmCross plugin version allows local databases to be opened/created using the interface:

    public interface ISQLiteConnectionFactory
    {
        ISQLiteConnection Create(string address);
    }

This can be used:

    var factory = Mvx.Resolve<ISQLiteConnectionFactory>();
    // open or create the database
    var connection = factory.Create("mydb.sql");
    // ensure our tables exist
    connection.CreateTable<CollectedItem>();
    
Once opened/created, SQLite database connections can be used like:

    // Create
    connection.Insert(item);
    
    // Update
    connection.Update(item);
    
    // Delete
    connection.Delete(item)l
    
    // Select
    var item = connection.Table<CollectedItem>.FirstOrDefault(x => x.Id == 42);

For more on the capabilities and use of sqlite-net, see https://github.com/praeclarum/sqlite-net/.

For samples using sqlite-net, see:

- N=10 - KittensDb - http://slodge.blogspot.co.uk/2013/05/n10-sqlite-persistent-data-storage-n1.html
- N=12 to N=17 - CollectABull - http://slodge.blogspot.co.uk/2013/05/n12-collect-bull-full-app-part-1-n1.html
 
The file location used by the sqlite-net `factory.Create` call is not ideal:

- Android and iOS - `Environment.SpecialFolder.Personal`
- WindowsPhone - IsolatedStorage root
- WindowsStore - Windows.Storage.ApplicationData.Current.LocalFolder
- Wpf - `Directory.GetCurrentDirectory()`

This may need changing and unifying in future versions of the plugin. To avoid these default locations, providing absolute paths is possible on some platforms.

Use of the Sqlite plugin on WindowsStore (and to some extent Wpf) is complicated by Windows native x86, x64 and ARM Sqlite.dll versions. For some discussion on this, see https://github.com/slodge/MvvmCross/issues/307

The future of this plugin is that we do want to reintegrate with the core Sqlite-net version - see https://github.com/praeclarum/sqlite-net/issues/135.

Until this reintegration occurs, we aren't actively extending this plugin - we don't want to diverge from the core platform if we can help it.

When this reintegration occurs, we do hope to include Sqlite-net async support.

### ThreadUtils

The ThreadUtils plugin provides a trivial cross-platform access to a `Thread.Sleep` implementation.

This is really only used for demos.

Current advice (August 2013): there's no real reason to use this plugin in production apps.

### Visibility

The `Visibility` plugin provides native implementations for the `MvxVisibility` enumeration.

`Visibility` is available on all platforms.

The `Visibility` functionality is generally not used directlt - instead it's generally used within ValueConverters used in Data-Binding. The plugin includes a couple of general purpose Value Converters - for more on these, see [The Mvx Visibility ValueConverters](https://github.com/slodge/MvvmCross/wiki/Value-Converters#the-mvx-visibility-valueconverters).

### WebBrowser

The `WebBrowser` plugin provides cross platform support for showing web pages using the external web browser using:

    public interface IMvxWebBrowserTask
    {
        void ShowWebPage(string url);
    }

This plugin is available on all of Android, iOS, WindowsPhone and WindowsStore.

## Other Plugins

### OrangeBit
Bitmap Plugin - Android and WindowsStore - https://github.com/orangebit/mvvmcross-bitmap-plugin/

### James Montemagno 
Settings - Android, iOS, WindowsPhone and WindowsStore - https://github.com/ceton/Mvx.Plugins.Settings

### GoodVibrations

This sample project is in https://github.com/slodge/MvvmCross-Tutorials/tree/master/GoodVibrations

This includes a simple Vibration implementation on Android, iOS and WindowsPhone.

### BallControl

The BallControl sample provides several examples of Plugins.

Most notably, BallControl includes a sample Bluetooth plugin for controlling Sphero Bluetooth robots.

Unfortunately, the source code for BallControl has gotten a little 'uncared for' recently - the update to v3 hasn't really been fully completed. However, the v3 branch does at least illustrate a real app with real plugins - see https://github.com/slodge/BallControl/tree/hotTuna.
