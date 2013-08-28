In order to test objects that make use of MvvmCross infrastructure, like ViewModels and the IoC container, it is necessary to propertly setup the environment. 

You'll need to (at least) add references to the following assemblies:
* `Cirrious.CrossCore`
* `Cirrious.MvvmCross`
* `Cirrious.MvvmCross.Test.Core`

The last one (`Cirrious.MvvmCross.Test.Core`) is the key, as it is defines the class `MvxIoCSupportingTest` which is used to do the heavy work. 

## Test class declaration and setup
Make your test classe inherit from `MvxIoCSupportingTest`, and then call the `Setup` method somewhere:

```csharp
using Cirrious.MvvmCross.Test.Core;

[TestFixture]
public class FooTestCase : MvxIoCSupportingTest
{
	protected UnityAutoMoqContainer container;

	public FooTestCase()
	{
		base.Setup(); // from MvxIoCSupportingTest
	}
}
```

## Registering objects and additional setup
Now that you have the bare bones for your test to work, you can use the `Ioc` property to register any singleton or regular types within MvvmCross. Also, there's a special method named `AdditionalSetup()` which can be overriden to automatically do post setup initialisation: 

```csharp
protected override void AdditionalSetup() 
{
	Ioc.RegisterSingleton<IFooService>(new MockFooService());
}
```

## Links and other references
* [MvvmCross: Enable Unit-testing](http://blog.fire-development.com/2013/06/29/mvvmcross-enable-unit-testing/). It is a blog post with some extra details. Please note that the `MvvmCrossTestSetup` used there is the same of `MvxIoCSupportingTest` that ships with MvvmCross itself

* The [Twitter Search](https://github.com/slodge/MvvmCross-Tutorials/tree/master/Sample%20-%20TwitterSearch) example has a [test project](https://github.com/slodge/MvvmCross-Tutorials/tree/master/Sample%20-%20TwitterSearch/TwitterSearch.Test) which can be used as reference as well

* There is a [video tutorial](http://slodge.blogspot.co.uk/2013/06/n29-testing-n1-days-of-mvvmcross.html) on testing