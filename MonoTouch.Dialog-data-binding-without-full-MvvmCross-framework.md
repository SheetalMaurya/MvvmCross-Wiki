I volunteered to put a demo together of how to use the MvvmCross databinding code without using the full Mvvm structure for your app.

This demo is at:

https://github.com/slodge/MvvmCross/tree/master/Sample%20-%20SimpleDialogBinding/SimpleBinding

The demo is based on TipCalc.

It uses a simplified MvvmCross setup in the AppDelegate class - this setup only passes in the IValueConverters to use in binding:

```
MvxSimpleTouchDialogBindingSetup.Initialise(typeof(Converters.Converters));
```

Because of this simplified setup, ViewModel and View creation is now left entirely up to the coder - MvvmCross just keeps entirely out of the way!

The ViewModel then used in this example is a very straight-forward `INotifyPropertyChanged` class - no `Mvx` in sight!

```
    public class TipViewModel
        : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

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


        private void FirePropertyChanged(string whichProperty)
        {
            // take a copy - see RoadWarrior's answer on http://stackoverflow.com/questions/282653/checking-for-null-before-event-dispatching-thread-safe/282741#282741
            var handler = PropertyChanged;

            if (handler != null)
                handler(this, new PropertyChangedEventArgs(whichProperty));
        }
    }
```

And finally the view is then just setup as a "normal MonoTouch.Dialog" but with extra `Bind` statements added:

```
    public class TipView : MvxSimpleTouchDialogViewController
    {
        public TipView () : base (UITableViewStyle.Grouped, null, false)
        {
            ViewModel = new TipViewModel();
        }

        public override void ViewDidLoad ()
        {
            base.ViewDidLoad ();

            this.Root = new RootElement("Tip Calc")
                            {
                                new Section("Enter Values")
                                    {
                                        new EntryElement("SubTotal", "SubTotal").Bind(this, "{'Value':{'Path':'SubTotal','Converter':'Float','Mode':'TwoWay'}}"),
                                        new EntryElement("TipPercent", "TipPercent").Bind(this, "{'Value':{'Path':'TipPercent','Converter':'Int','Mode':'TwoWay'}}"),
                                        new FloatElement(null, null, 0.0f)
                                                {
                                                    ShowCaption = false,
                                                    MinValue = 0.0f,
                                                    MaxValue = 100.0f
                                                }
                                                .Bind(this, "{'Value':{'Path':'TipPercent','Converter':'IntToFloat','Mode':'TwoWay'}}"),
                                    },
                                new Section("See the results")
                                    {
                                        new StringElement("TipValue").Bind(this, "{'Value':{'Path':'TipValue'}}"),
                                        new StringElement("Total").Bind(this, "{'Value':{'Path':'Total'}}"),
                                    },
                            };
        }
    }
```

This code as presented is a little bit inefficient - behind the scenes there is still quite a lot of Mvx code being loaded. However, for a simple binding solution if you are only working on MonoTouch, then I think it works quite well. For coders who are looking beyond MonoTouch, then I definitely recommend still trying the "full" MvvmCross experience - it's built from the ground up to allow you to share 90% of your code across platforms.
