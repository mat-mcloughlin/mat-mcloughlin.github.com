---
layout: post
title: A Basic Example of an IoC Container
post_url: http://mat-mcloughlin.github.io
image: inversion_house_artwork.jpg
---
Its bad I know. I've been using an IoC Container for while now, but never really knew how they worked. I had an idea, but it wasn't clear in my head. I got bored last night so I thought I'd have a go at writing a very simple one.

This example has no error handling and can only handle injecting classes into a single constructor. But for a proof of concept it will do nicely. 

Thinking about it, a simple IoC has to do a couple of things:

1. It needs to allow you to setup your bindings. By this I mean you should be able to tell it what concrete implementation you want returning for each interface. Some IoC containers do this automatically by scanning your assemblies, for my example you'll have to define them explicitly.
2. It then has to return an implementation when you try and resolve the interface. 
3. Any dependencies your implementation is dependent on also need to be resolved recursively.

And thats pretty much it. Sure, there are lots of additional functionality in most of the popular IoC containers but this is the core functionality. 

So to the code. The first thing I needed to do was allow you to store the bindings. I decided to store them in a dictionary of type:

{% highlight csharp linenos %}
private readonly Dictionary<Type, Func<object>> _providers;
{% endhighlight %}

So that the concrete implementation wouldn't get created until it was called for. Along with the following method to add to it

{% highlight csharp linenos %}
public void Bind<TInterface, TConcrete>() where TConcrete : TInterface
{
    _providers[typeof(TInterface)] = () => Resolve(typeof(TConcrete));
}
{% endhighlight %}
    
This allows you to pass in a interface and concrete class that inherits that interface like so:

{% highlight csharp linenos %}
var container = new Container();
container.Bind<ICar, Car>();
{% endhighlight %}

where car is:

{% highlight csharp linenos %}
class Car : ICar { }
{% endhighlight %}

That bit is simple. The next stage is to be able to resolve that type when it is called for. That means

1. Using refelection to get a reference to the class constructor.
2. Get the list of parameters for said constructor.
3. Resolve each of these parameters.
4. Invoke the constructor whilst passing in the parameters.

The resulting method is below. It could be shortend with linq and there is no error checking, however:

{% highlight csharp linenos %}
private object Resolve(Type type)
{
    var constructor = type.GetConstructors().SingleOrDefault();
    var parametersInfo = constructor.GetParameters();
    var parameters = new List<object>();

    foreach (var parameterInfo in parametersInfo)
    {
        parameters.Add(_providers[parameterInfo.ParameterType]());
    }

    return constructor.Invoke(parameters.ToArray());
}
{% endhighlight %}

Finally a public method to allow you to resolve:

{% highlight csharp linenos %}
public T Resolve<T>()
{
    return (T)_providers[typeof(T)]();
}
{% endhighlight %}

That is called as such:

{% highlight csharp linenos %}
var car = container.Resolve<ICar>();
{% endhighlight %}

And thats it. For a complete example see this [gist](https://gist.github.com/mat-mcloughlin/6240821). Like I said, there's a lot more involved in IoC containers such as [Ninject](http://www.ninject.org/) and [TinyIoC](https://github.com/grumpydev/TinyIoC) (my favourite of the moment) but the basic principle is pretty simple.
