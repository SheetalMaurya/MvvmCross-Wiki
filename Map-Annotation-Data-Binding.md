Subscribing to changing collections is one of the corner-stones of Data-Binding and relies on a little knowledge of the `INotifyCollectionChanged` interface.

Within the MvvmCross source, there are a few example classes which show how to subscribe to collections and their change notifications - e.g. [MvxViewGroupExtensions.cs][1] in Droid and [MvxTableViewSource.cs][2] in Touch

The core to the technique is to create an `Adapter` or `Source` object which listens for changes either in the whole list or to parts of the list and which takes action accordingly.

The same type of approach applies for maps with multiple - but markers - although we don't yet have any helper classes for this.

----

Without actually having a Mac or iOS device to hand, here are roughly the steps I'd take to create a wrapper...

Assuming I had a Model object like:

    public class House
    {
        public double Lat { get; set; }
        public double Lng { get; set; }
        public string Name { get; set; }
    }

Inside a ViewModel like:

    public class FirstViewModel : MvxViewModel
    {
        public ObservableCollection<House> HouseList { get; set; }
    }

With this done, then in the View we can create an annotation class for each House - e.g. something like:

    public class HouseAnnotation : MKAnnotation
    {
        public HouseAnnotation(House house)
        {
            // Todo - the details of actually using the house here.
            // in theory you could also data-bind to the house too (e.g. if it's location were to move...)
        }

        public override CLLocationCoordinate2D Coordinate { get; set; }
    }

We could then create a `HouseAnnotationManager` who's responsibility would be to manage the translation of changes in the `HouseList` being mapped to changes in the annotations being displayed on the map.

To do this, we would give the manager methods to:

1. Create a single annotation:

        private MKAnnotation CreateAnnotation(House house)
        {
            return new HouseAnnotation(house);
        }

2. Add an annotation to the map (and to a local lookup table)

        private void AddAnnotationFor(House house)
        {
            var annotation = CreateAnnotation(house);
            _annotations[house] = annotation;
            _mapView.AddAnnotation(annotation);
        }

3. Remove an annotation from the map (and from a local lookup table)

        private void RemoveAnnotationFor(House house)
        {
            var annotation = _annotations[house];
            _mapView.RemoveAnnotation(annotation);
            _annotations.Remove(house);
        }

4. Do the same actions for lists:

        private void AddAnnotations(IList newItems)
        {
            foreach (House house in newItems)
            {
                AddAnnotationFor(house);
            }
        }

        private void RemoveAnnotations(IList oldItems)
        {
            foreach (House house in oldItems)
            {
                RemoveAnnotationFor(house);
            }
        }

5. Respond to `INotifyCollection` changes:

        private void OnItemsSourceCollectionChanged(object sender, NotifyCollectionChangedEventArgs e)
        {
            switch (e.Action)
            {
                case NotifyCollectionChangedAction.Add:
                    AddAnnotations(e.NewItems);
                    break;
                case NotifyCollectionChangedAction.Remove:
                    RemoveAnnotations(e.OldItems);
                    break;
                case NotifyCollectionChangedAction.Replace:
                    RemoveAnnotations(e.OldItems);
                    AddAnnotations(e.NewItems);
                    break;
                case NotifyCollectionChangedAction.Move:
                    // not interested in this
                    break;
                case NotifyCollectionChangedAction.Reset:
                    ReloadAllAnnotations();
                    break;
                default:
                    throw new ArgumentOutOfRangeException();
            }
        }

6. Respond to whole list changes:

        // MvxSetToNullAfterBinding isn't strictly needed any more 
        // - but it's nice to have for when binding is torn down
        [MvxSetToNullAfterBinding]
        public virtual IEnumerable<House> ItemsSource
        {
            get { return _itemsSource; }
            set { SetItemsSource(value); }
        }

        protected virtual void SetItemsSource(IEnumerable<House> value)
        {
            if (_itemsSource == value)
                return;

            if (_subscription != null)
            {
                _subscription.Dispose();
                _subscription = null;
            }
            _itemsSource = value;
            if (_itemsSource != null && !(_itemsSource is IList))
                MvxBindingTrace.Trace(MvxTraceLevel.Warning,
                                      "Binding to IEnumerable rather than IList - this can be inefficient, especially for large lists");

            ReloadAllAnnotations();

            var newObservable = _itemsSource as INotifyCollectionChanged;
            if (newObservable != null)
            {
                _subscription = newObservable.WeakSubscribe(OnItemsSourceCollectionChanged);
            }
        }

With this all written, then your ViewModel can have a private `_manager` field and can create and data-bind it as:

            _manager = new HouseAnnotationManager(myMapView);

            var set = this.CreateBindingSet<FirstView, FirstViewModel>();
            set.Bind(_manager).To(vm => vm.HouseList);
            set.Apply();

-----

Overall, this might look something like: https://gist.github.com/slodge/6070386

-----

The same basic approach should also work in Android - although in Android you'll also have to fight a battle against the setup - Ant, Google Play v2 and all that jazz.

-----

If you wanted to do further map manipulations - e.g. changing the map center and zoom when a house is added, then this can obviously be done from within overrides of methods such as AddAnnotation within your manager.

  [1]: https://github.com/slodge/MvvmCross/blob/v3/Cirrious/Cirrious.MvvmCross.Binding.Droid/Views/MvxViewGroupExtensions.cs
  [2]: https://github.com/slodge/MvvmCross/blob/v3/Cirrious/Cirrious.MvvmCross.Binding.Touch/Views/MvxTableViewSource.cs