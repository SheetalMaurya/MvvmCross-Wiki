MvvmCross application's are normally structured with:

* one shared 'core' Portable Class Library (PCL) project 
  * containing as much code as possible: models, view models, services, converters, etc
* one UI project per platform 
  * each containing the bootstrap and view-specific code for that platform

Normally, you start development from the core project - and that's exactly what we'll do here.

To create the core, you can use the Visual Studio project template wizards, but here we'll instead build up a new project 'from empty'.

## Create the new Portable Class Library

Using Visual Studio, create your new PCL using the File|New Project wizard.

Call it something like TipCalc.Core.csproj

When asked to choose platforms, select all of WindowsPhone, WindowsStore (.Net4.5), Xamarin.Android and Xamarin.iOS - this will ensure that the PCL is in **Profile104**. This profile defines a small subset of .Net that contains parts of the assemblies for:

* mscorlib
* System.Core
* System.Net
* System.Runtime.Serialization
* System.ServiceModel
* System.Windows
* System.Xml
* System.Xml.Linq
* System.Xml.Serialization

Importantly for us this Profile104 includes everything we need to build our Mvvm applications.

## Delete Class1.cs

No-one really needs a `Class1` :)

## Add references to the CrossCore and MvvmCross assemblies

Use 'Add references' to add links to the 2 Portable Class Libraries.

* **Cirrious.CrossCore.dll** 
   - core interfaces and concepts including Trace, IoC and Plugin management
* **Cirrious.MvvmCross.dll** 
   - Mvvm classes - including base classes for your MvxApplication and your MvxViewModels

Normally these will be found in a folder path like *{SolutionRoot}/Libs/Mvx/Portable/*

## Add the Tip Calculation service

Create a folder called 'Services'

Within this folder create a new Interface which will be used for calculating tips:

    public interface ICalculation
    {
        double TipAmount(double subTotal, int generosity);
    }

Within this folder create an implementation of this interface:

    public class Calculation : ICalculation
    {
        public double TipAmount(double subTotal, int generosity)
        {
            return subTotal * ((double)generosity))/100.0;
        }
    }

This provides us with some simple business logic for our app

## Add the ViewModel

At a sketch level, we want a user interface that:

* uses:
  * our calculation service to calculate the tip
* has inputs of:
  * the current bill (the subTotal)
  * a feeling for how much tip we'd like to leave (the generosity)
* has output displays of:
  * the calculated tip to leave

To represent this user interface we need to build a 'model' for the user interface - which is, of course, a 'ViewModel'

Within MvvmCross, all ViewModels should inherit from `MvxViewModel`.

So now create a ViewModels folder in our project, and in this folder add a new `TipViewModel` class like:

    using Cirrious.MvvmCross.ViewModels;

    namespace TipCalc.Core
    {
        public class TipViewModel : MvxViewModel
        {
            private readonly ICalculation _calculation;
            public TipViewModel(ICalculation calculation)
            {
                _calculation = calculation;
            }

            public override void Start()
            {
                _subTotal = 100;
                _generosity = 10;
                Recalcuate();
                base.Start();
            }

            private double _subTotal;

            public double SubTotal
            {
                get { return _subTotal; }
                set { _subTotal = value; RaisePropertyChanged(() => SubTotal); Recalcuate(); }
            }

            private int _generosity;

            public int Generosity
            {
                get { return _generosity; }
                set { _generosity = value; RaisePropertyChanged(() => Generosity); Recalcuate(); }
            }

            private double _tip;

            public double Tip
            {
                get { return _tip; }
                set { _tip = value; RaisePropertyChanged(() => Tip);}
            }

            private void Recalcuate()
            {
                Tip = _calculation.TipAmount(SubTotal, Generosity);
            }
        }
    }

For many of you, this `TipViewModel` will already make sense to you. If it does then **skip ahead** to 'Create the Application'. If not, then here are some simple explanations:

* the `TipViewModel` is constructed with an `ICalculation` service

        private readonly ICalculation _calculation;
       
        public TipViewModel(ICalculation calculation)
        {
            _calculation = calculation;
        }

* after construction, the `TipViewModel` will be started - during this it sets some initial values.

        public override void Start()
        {
            // set some start values
            SubTotal = 100.0;
            Generosity = 10;
            Recalculate();
        }

* the view data held within the `TipViewModel` is exposed through properties. 
  * Each of these properties is backed by a private member variable
  * Each of these properties has a get and a set 
  * The set accessor for `Tip` is marked private
  * All of the set accessors call `RaisePropertyChanged` to tell the base `MvxViewModel` that the data has changed
  * The `SubTotal` and `Generosity` set accessors also call `Recalculate()`

            private double _subTotal;
            public double SubTotal
            {
                get { return _subTotal; }
                set {  _subTotal = value; RaisePropertyChanged(() => SubTotal); Recalculate(); }
            }

            private int _generosity;
            public int Generosity
            {
                get { return _generosity; }
                set {  _generosity = value; RaisePropertyChanged(() => Generosity); Recalculate(); }
            }

            private double _tip;
            public double Tip
            {
                get { return _tip; }
                private set {  _tip = value; RaisePropertyChanged(() => Tip); }
            }

* The `Recalculate` method uses the `_calculation` service to update `Tip` from the current values in `SubTotal` and `Generosity`

        private void Recalculate()
        {
            Tip = _calculation.TipAmount(SubTotal, Generosity);
        }

## Add the App(lication)

With our `Calculation` service and `TipViewModel` defined, we now just need to add the main `App` code.

This code:

* will sit in a single class within the root folder of our PCL core project. 
* this class will inherits from the `MvxApplication` class
* this class is normally just called `App`
* this class is responsible for providing:
  * registration of which interfaces and implementations the app uses
  * registration of which `ViewModel` the `App` will show when it starts
  * control of how `ViewModel`s are located - although most applications normally just use the default implementation of this supplied by the base `MvxApplication` class.

'Registration' here means creating an 'Inversion of Control' - IoC - record for an interface. This IoC record tells the MvvmCross framework what to do when anything asks for an instance of that interface.

For our Tip Calculation app:

* we register the `Calculation` class to implement the `ICalculation` service

             Mvx.RegisterType<ICalculation, Calculation>();

  this line tells the MvvmCross framework that whenever any code requests an `ICalculation` reference, then the framework should create a new instance of `Calculation`

* we want the app to start with the `TipViewModel`

             var appStart = new MvxAppStart<TipViewModel>();
             Mvx.RegisterSingleton<IMvxAppStart>(appStart);

  this line tells the MvvmCross framework that whenever any code requests an `IMvxAppStart` reference, then the framework should return that same `appStart` instance.

So here's what App.cs looks like:

    using Cirrious.CrossCore.IoC;
    using Cirrious.MvvmCross.ViewModels;

    namespace TipCalc.Core
    {
        public class App : MvxApplication
        {
            public App()
            {
                Mvx.RegisterType<ICalculation,Calculation>();
                Mvx.RegisterSingleton<IMvxAppStart>(new MvxAppStart<TipViewModel>());
            }
        }
    }

## Note: What is 'Inversion of Control'?

We won't go into depth here about what IoC - Inversion of Control - is.

Instead, we will just say that:

* within each MvvmCross application, there is a single special object - a `singleton`
* This `singleton` lives within the `Mvx` static class.
* The application startup code can use the `Mvx.Register` methods in order to specify what will implement `interface`s during the lifetime of the app.
* After this has been done, then later in the life when any code needs an `interface` implementation, then it can request one using the `Mvx.Resolve` methods.

One common pattern that is seen is 'constructor injection':

* Our `TipViewModel` uses this pattern. 
* It presents a constructor like: `public TipViewModel(ICalculation calculation)`. 
* When the app is running a part of the MvvmCross framework called the `ViewModelLocator` is used to find and create `ViewModel`s
* when a `TipViewModel` is needed, the `ViewModelLocator` uses a call to `Mvx.IocConstruct` to create one.
* This `Mvx.IocConstruct` call creates the `TipViewModel` using the `ICalculation` implementation that it finds using `Mvx.Resolve`

This is obviously only a very brief introduction.

If you would like to know more, please see look up some of the excellent tutorials out there on the Internet - like http://joelabrahamsson.com/inversion-of-control-an-introduction-with-examples-in-net/

## The Core project is complete :)

Just to recap the steps we've followed:

1. We created a new PCL project using Profile104

2. We added references to two PCL libraries - CrossCore and MvvmCross

3. We added a `ICalculation` interface and implementation pair

4. We added a `TipViewModel` which:
  * inherited from `MvxViewModel`
  * used `ICalculation` 
  * presented a number of public properties each of which called `RaisePropertyChanged`


5. We added an `App` which:
  * inherited from `MvxApplication`
  * registered the `ICalculation`/`Calculation` pair
  * registered a special start object for `IMvxAppStart`

These are the same steps that you need to go through for every new MvvmCross application.

## Moving on

Next we'll start looking at how to add a first UI to this MvvmCross application.