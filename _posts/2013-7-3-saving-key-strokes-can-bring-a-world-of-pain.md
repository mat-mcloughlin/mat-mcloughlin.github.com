---
layout: post
title: Saving Key Strokes can bring a World of Pain
post_url: http://mat-mcloughlin.github.io
---

Developers seem to fall into two schools of thought on this one. Limit the amount of code you write or leverage the syntax to assure no mistakes are made. I've flirted between the two, but right now I'm heavily in favor of the latter approach. 

What am I talking about? Well two things in particular. Firstly the use of chaining declarations in JavaScript and secondly omitting curly braces in single line if statements.

Both of these things have caught me out recently. Let me give you some examples:

## Chaining Declarations in JavaScript

{% highlight js linenos %}
function SomeObject() {
    var span = document.createElement('span'),
        li = document.createElement('li'),
        div = document.createElement('div');
        label = document.createElement('label'),
        input = document.createElement('input');
}
{% endhighlight %}

Can you spot the mistake. I accidentally used a semi-colon instead of a comma when declaring my variables. This is bad news. It means that everything after the semi-colon became a global variable and therefore there could be contamination between my two object instances. This wasn't picked up by my IDE and took a second pair of eyes to find it (thanks Rob). 

This could have all been easily avoided if I had just declared everything individually as below:

{% highlight js linenos %}
function SomeObject() {
    var span = document.createElement('span');
    var li = document.createElement('li');
    var div = document.createElement('div');
    var label = document.createElement('label');
    var input = document.createElement('input');
}
{% endhighlight %}

It doesn't look as tidy but it certainly will help avoid any pitfalls.

## Omitting curly braces on single line if statements
This caught me out too. Consider this example:

{% highlight csharp linenos %}
function void ClearStuff(bool shouldIClear) {
    if (shouldIClear)
        Foo = null;
}
{% endhighlight %}

If somebody decided they want to clear out Bar as well its possible they would just add the line in below not noticing the lack of braces:

{% highlight csharp linenos %}
function void ClearStuff(bool shouldIClear) {
    if (shouldIClear)
        Foo = null;
        Bar = null;
}
{% endhighlight %}

And this code isn't going to work as expected. Its easily done and will compile so could easily go unnoticed. Better to include the braces so when somebody comes along to modify the code, its clear where the if statement starts and ends:

{% highlight csharp linenos %}
function void ClearStuff(bool shouldIClear) {
    if (shouldIClear)
    {
        Foo = null;
    }
}
{% endhighlight %}

Some may argue that the code isn't as readable. I say its about limiting pitfalls.