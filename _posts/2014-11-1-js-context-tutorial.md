---
layout: post
title: "Reuse functions by prototype and apply in js"
description: "Reuse functions by prototype and apply in js"
category: [js]
tags: [js, tutorial]
---

---------------------------------------
**In this example, I reuse the pop function of Array.prototype to pop the first element of input arguments**

    var calculate = function(){
        var fn = Array.prototype.pop.apply(arguments);
        console.log(fn);
        return fn.apply(null, arguments);
    };

    var sum = function(){
        var total = 0;
        for(var i=0, l=arguments.length; i<l; i=i+1){
            total = total +arguments[i];
        }
        return total;
    };

    var diff = function(x, y){
        return x-y;
    };


    var sumResult = calculate(1,2,3,4,5, sum),
        diffResult = calculate(1,2, diff);

    alert(sumResult);
    alert(diffResult);
