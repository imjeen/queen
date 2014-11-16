---
layout: post
title: JavaScript函数节流与函数去抖
date: 2014.11.15
categories: update
---

## 概念

- 函数去抖（debounce）：让一个函数在一定间隔内没有被调用时，才开始执行被调用方法。
- 函数节流（throttle）：是让一个函数无法在很短的时间间隔内连续调用，当上一次函数执行后过了规定的时间间隔，才能进行下一次该函数的调用。
    函数节流的出发点，就是让一个函数不要执行得太频繁，减少一些过快的调用来节流。浏览器aip功能： [requestAnimationFrame](http://www.w3cfuns.com/blog-5416113-5397832.html "requestAnimationFrame")

两个方法都是用来提升前端性能，减轻浏览器压力。

## 应用

理解起来有点费力，通过应用来理解就轻松了。通常，我们会在有用户交互参与的地方添加事件，而往往这种事件会被频繁触发。

想象一下窗口的resize事件或者是一个元素的onmousemove事件，resize会在改变浏览器大小事连续触发、onmousemove会在鼠标移动时被连续触发，如果你的回调过重，你可能使浏览器死掉。
想象一下你需要在用户输入一段文字时对文字进行处理，你监听文字改变，每一次改变都会调用一次回调函数，其实我需要的是在用户输入停下来的时候去处理一次。
射击游戏中你希望1s中之内只能发射一颗子弹，而不是用户每按一次发射就发射。
类似的应用还有很多，throttle和debounce的区别就是在频繁的回调中，throttle以一定频率运行，而debounce在频繁回调之后运行。总的来说就是过滤频繁触发的事件回调，使其在真正需要的时候执行，两者根据应用场景自行选择。

## 实现

说了这么多，怎么使用debounce和throttle功能呢，伟大的 http://underscorejs.org 给我们实现好了这两个方法，这两个方法的实现都是不依赖于其他underscore方法的，所以我们可以轻易的添加到其他JavaScript库中，比如jQuery。

### debounce -- setTimeout
函数去抖的基本思想是：对需要去抖的函数做包装，使用闭包记录timeout，第一次回调给函数设置 setTimeout定时器，只要在wait时间内，后一次的回调会clearTimeout取消前一次回调的执行。

    ```
    _.debounce = function(func, wait, immediate) {
        var timeout, result;
        return function() {
          var context = this, args = arguments;
          var later = function() {
            timeout = null;
            if (!immediate) result = func.apply(context, args);
          };
          var callNow = immediate && !timeout;
          clearTimeout(timeout);
          timeout = setTimeout(later, wait);
          if (callNow) result = func.apply(context, args);
          return result;
        };
    };
    ```

### throttle -- setInterval
函数节流的基本思想是:无视浏览器的回调，自己按一定频率执行代码。

    ```
    _.throttle = function(func, wait) {
        var context, args, timeout, result;
        var previous = 0;
        var later = function() {
          previous = new Date;
          timeout = null;
          result = func.apply(context, args);
        };
        return function() {
          var now = new Date;
          var remaining = wait - (now - previous);
          context = this;
          args = arguments;
          if (remaining <= 0) {
            clearTimeout(timeout);
            timeout = null;
            previous = now;
            result = func.apply(context, args);
          } else if (!timeout) {
            timeout = setTimeout(later, remaining);
          }
          return result;
        };
    };
    ```

## 参考
1. [如何稀释onscroll事件](http://segmentfault.com/q/1010000000707337)
2. [js能否给事件触发设置时间间隔条件？](http://segmentfault.com/q/1010000000714176)
3. [浅谈javascript的函数节流](http://www.alloyteam.com/2012/11/javascript-throttle/)
4. [JavaScript函数节流与函数去抖](http://www.cnblogs.com/friskfly/p/3175077.html)