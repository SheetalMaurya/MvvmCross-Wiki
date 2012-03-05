I took the TipCalc code from: https://github.com/xamarin/mobile-samples/tree/master/TipCalc - and ported it into the Tutorial project.

![TipCalc MvvmCross](https://github.com/slodge/MvvmCross/raw/master/Sample%20-%20Tutorial/Help/TipCalc.png)

For the sake of readability, I simplified the algorithm a bit - I removed Tax from the calculation.

So the ViewModel became:

```
    public class TipViewModel
        : MvxViewModel
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
    }
```

And the views for each platform were then very simple to code:

###iOS:

UI designed in XIB with binding done as:
```
        public override void ViewDidLoad ()
        {
            base.ViewDidLoad ();

            this.AddBindings(
                new Dictionary<object, string>()
                    {
                        { TipValueLabel, "{'Text':{'Path':'TipValue'}}" },
                        { TotalLabel, "{'Text':{'Path':'Total'}}" },
                        { TipPercentText, "{'Text':{'Path':'TipPercent','Converter':'Int','Mode':'TwoWay'}}" },
                        { TipPercentSlider, "{'Value':{'Path':'TipPercent','Converter':'IntToFloat','Mode':'TwoWay'}}" },
                        { SubTotalText, "{'Text':{'Path':'SubTotal','Converter':'Float','Mode':'TwoWay'}}" },
                    });
        }
```

###Android:

UI and binding done in AXML:

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
        android:id="@+id/TipPercent"
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

So the Activity C# is just:
```
    [Activity]
    public class TipView
        : MvxBindingActivityView<TipViewModel>
    {
        protected override void OnViewModelSet()
        {
            SetContentView(Resource.Layout.Page_TipView);
        }
    }
```

###WP7:

All the UI and binding in XAML:

```
            <StackPanel>
                <TextBlock
                    Text="SubTotal"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBox 
                    Text="{Binding SubTotal, Converter={StaticResource FloatValueConverter}, Mode=TwoWay}" 
                    />

                <TextBlock
                    Text="Tip"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBox 
                    Text="{Binding TipPercent, Converter={StaticResource IntValueConverter}, Mode=TwoWay}" 
                    />

                <Slider Value="{Binding TipPercent,Mode=TwoWay}" SmallChange="1" LargeChange="10" Minimum="0" Maximum="100" />
                
                <TextBlock
                    Text="Tip"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBlock 
                    Text="{Binding TipValue, Converter={StaticResource FloatValueConverter}}" 
                    />

                <TextBlock
                    Text="Total"
                    Style="{StaticResource PhoneTextSubtleStyle}"
                    />
                <TextBlock 
                    Text="{Binding Total, Converter={StaticResource FloatValueConverter}}" 
                    />
            </StackPanel>
```