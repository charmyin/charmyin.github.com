---
layout: post
title: "Js context introduction"
description: "Js context introduction"
category: [js]
tags: [js, tutorial]
---

---------------------------------------

**Origin**  [https://www.youtube.com/watch?v=fjJoX9F_F5g](https://www.youtube.com/watch?v=fjJoX9F_F5g)

###This in window context

![WindowThis]({{ site.url }}/assets/images/2014-11-01/window-this.jpg)


----------------------------------------

###This In Global Function

![This In Global Function]({{ site.url }}/assets/images/2014-11-01/thisInGlobalFunction.jpg)

----------------------------------------

###What is this

![What is this]({{ site.url }}/assets/images/2014-11-01/3whatisthis.jpg)

----------------------------------------

###This in Object

![This in Object]({{ site.url }}/assets/images/2014-11-01/4thisInObj.jpg)

----------------------------------------

###Three methods to change context

![Three methods to change context]({{ site.url }}/assets/images/2014-11-01/5threeMethodsToChangeContext.jpg)

**Method 1**

      var obj = {
        foo : function(){
          console.log(this === window);
        }
      };
      //call
      obj.foo.call(window);

**Method 2**

      var obj = {
        foo : function(){
          console.log(this === window);
        }
      };
      //call
      obj.foo.apply(window);

**Method 3**

      var obj = {
        foo : function(){
          console.log(this === window);
        }
      };
      //call
      var windowFoo = obj.foo.bind(window);
      windowFoo();

----------------------------------------

###Bind to change Context

![Bind to change Context]({{ site.url }}/assets/images/2014-11-01/6BindToChangeContext.jpg)

----------------------------------------

###Element context in event

![Element context in event]({{ site.url }}/assets/images/2014-11-01/7ElementContextEvent.jpg)

----------------------------------------

###Get parent context by reference variable**

![Get parent context by reference variable]({{ site.url }}/assets/images/2014-11-01/8pointToparentContext.jpg)

----------------------------------------

###Get parent context by methd bind

![Get parent context by methd bind]({{ site.url }}/assets/images/2014-11-01/8pointToParentContext2.jpg)
