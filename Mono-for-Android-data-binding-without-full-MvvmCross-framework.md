This article introduces how to use databinding for MonoDroid within Android XML without using the full mvvmcross framework.

This article follows on from https://github.com/slodge/MvvmCross/wiki/MonoTouch.Dialog-data-binding-without-full-MvvmCross-framework and is based on the same TipCalc example.

The source for this sample is in: https://github.com/slodge/MvvmCross/tree/master/Sample%20-%20SimpleDialogBinding/SimpleBinding/SimpleDroid

##Set up - resources

In order to let the Android resource builder know about our databinding attributes we include the `MvxBindingAttributes.xml` file in our project and we reference it using:

```
      xmlns:local="http://schemas.android.com/apk/res/SimpleDroid.SimpleDroid"
```

##Set up - code

Just as with the Touch example, we  need a small stub of setup code in order to initialise the databinding system - especially the IoC framework.

This code is a singleton in Setup.cs and we check it from every Activity::OnCreate() call.

```
    public class Setup 
        : MvxSimpleAndroidBindingSetup
    {
        private static Setup _singleton;

        public static void EnsureInitialised(Context applicationContext)
        {
            if (_singleton != null)
                return;
            _singleton = new Setup(applicationContext);
            _singleton.Initialize();
        }

        private Setup(Context applicationContext)
            : base(applicationContext, typeof(Converters.Converters))
        {
        }
    }
```

##ViewModel

As with the Touch example, we use a simple TipViewModel which inherits directly from INotifyPropertyChanged:

```
    public class TipViewModel
        : INotifyPropertyChanged
    {
        private float _tipValue;
        public float TipValue
        {
            get { return _tipValue; }
            private set { _tipValue = value; FirePropertyChanged("TipValue"); }
        }

        private float _total;
        public float Total
        {
            get { return _total; }
            private set { _total = value; FirePropertyChanged("Total"); }
        }

        private float _subTotal;
        public float SubTotal
        {
            get { return _subTotal; }
            set { _subTotal = value; FirePropertyChanged("SubTotal"); Recalculate(); }
        }

        private int _tipPercent;
        public int TipPercent
        {
            get { return _tipPercent; }
            set { _tipPercent = value; FirePropertyChanged("TipPercent"); Recalculate(); }
        }

        public TipViewModel()
        {
            SubTotal = 60.0f;
            TipPercent = 12;
            Recalculate();
        }

        private void Recalculate()
        {
            TipValue = ((int)Math.Round(SubTotal * TipPercent)) / 100.0f;
            Total = TipValue + SubTotal;
        }

        #region INotifyPropertyChanged

        public event PropertyChangedEventHandler PropertyChanged;

        private void FirePropertyChanged(string whichProperty)
        {
            // take a copy - see RoadWarrior's answer on http://stackoverflow.com/questions/282653/checking-for-null-before-event-dispatching-thread-safe/282741#282741
            var handler = PropertyChanged;

            if (handler != null)
                handler(this, new PropertyChangedEventArgs(whichProperty));
        }

        #endregion
    }
```


##View

The C# for our Activity is very simple - the Activity just has responsibility for:

- creating a ViewModel
- checking Setup is initialised
- requesting the Content is inflated

```
    [Activity(Label = "SimpleDroid", MainLauncher = true, Icon = "@drawable/icon")]
    public sealed class MainActivity : MvxSimpleBindingActivity
    {
        public MainActivity()
        {
            ViewModel = new TipViewModel();
        }

        protected override void OnCreate(Bundle bundle)
        {
            Setup.EnsureInitialised(ApplicationContext);

            base.OnCreate(bundle);

            // Set our view from the "main" layout resource
            SetContentView(Resource.Layout.Main);
        }
    }
```

The binding itself is then done in the layout XML - each `local:MvxBind` attribute includes JSON describing how the bind works:

```
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
      xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:local="http://schemas.android.com/apk/res/Tutorial.UI.Droid"
      android:fillViewport="true"
      android:layout_height="fill_parent"
      android:layout_width="fill_parent">
  <TableLayout
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:stretchColumns="0,1">
    <TableRow>
      <TextView
        android:text="Subtotal:"
        android:layout_column="0"
        android:gravity="right"
      />
      <EditText
        android:hint="Subtotal"
        android:gravity="left"
        android:inputType="numberDecimal"
        android:maxLines="1"
        android:numeric="decimal"
        local:MvxBind="{'Text':{'Path':'SubTotal','Converter':'Float'}}"
      />
    </TableRow>
    <TableRow>
      <TextView
            android:text="Tip Percent:"
            android:layout_column="0"
            android:gravity="right"
      />
      <EditText
        android:text="15"
        android:gravity="left"
        android:inputType="number"
        android:maxLines="1"
        android:numeric="decimal"
        local:MvxBind="{'Text':{'Path':'TipPercent','Converter':'Int'}}"
      />
    </TableRow>
    <TableRow>
      <SeekBar
          android:max="100"
          android:progress="15"
          android:layout_span="2"
          local:MvxBind="{'Progress':{'Path':'TipPercent'}}"
      />
    </TableRow>
    <TableRow>
      <TextView
            android:text="Tip Value:"
            android:layout_column="0"
            android:gravity="right"
      />
      <EditText
        android:enabled="false"
        android:hint="Tip Value"
        android:gravity="left"
        android:inputType="number"
        android:maxLines="1"
        android:numeric="decimal"
        local:MvxBind="{'Text':{'Path':'TipValue','Converter':'Float'}}"
      />
    </TableRow>
    <TableRow>
      <TextView
        android:text="Total:"
        android:layout_column="0"
        android:gravity="right"
      />
      <EditText
        android:enabled="false"
        android:hint="Total"
        android:gravity="left"
        android:inputType="number"
        android:maxLines="1"
        android:numeric="decimal"
        local:MvxBind="{'Text':{'Path':'Total','Converter':'Float'}}"
      />
    </TableRow>
  </TableLayout>
</ScrollView>
```


##Summary

As with the Touch example, this shows the the mvvm cross databinding can be very easily used independently of the rest of the mvvmcross framework. 

As with Touch:

- the amount of learning required to get this working is pretty small - much smaller than the learning needed for the full mvvmcross framework
- there are some small inefficiencies in this approach, but overall the approach works pretty quickly and efficiently.
- if you are looking to develop cross platform code, then I still recommend investing the time in a full mvvmcross development environment - the time invested now will pay off in the longer term.


