#

*created @ Sat Sep 29 2018*    @JX

最近做了一个关于使用setTimeout(), rAF()和CSS3制作动画的一些比较，自己收获也蛮多。

#### 基本的概念
1. 屏幕的刷新频率60HZ 以及 浏览器渲染刷新频率 60fps
  
      一般的显示器的刷新频率是60H，指的是在1s内刷新60次；而浏览器渲染刷新频率60fps指的是1s内刷新
      60帧，即60次。一副静态的图片，帧率是0fps，但是显示器的刷新还是60HZ，而不是0，这个概念需要区分。

      但是并不是所有的浏览器的刷新频率都是60fps（frame per second），也不是所有的屏幕的刷新频率
      都是60HZ，假如不一样的话，会出现什么问题呢？假如浏览器的刷新频率比屏幕的快，那么当屏幕还没到
      刷新时间点时，GPU已经准备好了刷新的新的界面，这样子就会导致画面不流畅，有撕裂感；反过来就会出现动画缓慢的效果。
      
      所以为了解决这个不同步问题，屏幕在刷新之前会发出一个垂直同步信号，浏览器接收这个信号，再进行渲染，这样
      每次都会在屏幕两次刷新之间准备好，一般就不会有问题；但是对GPU频率很高的PC游戏来说，这个不是好的解决方案，
      因为会降低刷新频率，导致游戏页面非常卡顿，影响体验。但是对网页来说，其实已经足够了。
    
 2. 浏览器的时钟是不准确的，不同浏览器有很大的差别，即使同一个浏览器在不同的环境（电池电量，cpu等）下也会不一样
     
     一般而言，chrome浏览器的时钟是4.8ms，IE8差不多为15.6ms，所以使用setTimeout(callback, t)来异步执行回调函数，执行的
     时机可能并不如你想象的那样，因为这个时间本身就不是1ms或者0.1ms来量度的；
     
 3. Update UI Rendering是在event loop执行完一个task之后
 
     js是单线程的，当js在执行的时候，ui渲染线程得不到执行的时间片段，所以必须要等到js执行完之后，在执行
     页面UI的渲染更新，具体可以看看[EVENT LOOP PROCESS MODEL](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model)
     
 4. Web render flow
 
    ![如图](https://raw.githubusercontent.com/xiezhm3/xiezhm3.github.io/6e62ba484c7203b29add14e3f922310e8eab8f40/assets/img/web-render-flow.jpg)
    
    ![图2](https://raw.githubusercontent.com/xiezhm3/xiezhm3.github.io/6e62ba484c7203b29add14e3f922310e8eab8f40/assets/img/web-rendering-flow.jpg)

#### setTimeout(cb, t)

  60fps，即16.7ms刷新一帧，所有的js操作以及UI渲染刷新都要在这个时间间隔之内完成，这样才能保证动画非常流畅。
  
  但是使用setTimeout，前面我们知道，受限于浏览器时钟的不准确性，并不能在16.7ms时刚好执行回调函数，假如浏览器的时钟是15.6ms，
  那么就需要两个时钟来计算16.7ms，这样中间就会有15.6 * 2 - 16.7 = 14.5ms的时间差，积累之后，后面的帧肯定就是会乱了，导致动画效果非常差
  
  上面讨论的前提是 setTimeout的异步回调函数在执行的时候，没有同步js代码的干扰下。
  
  
#### rAF()
  
setTimeout的问题是不知道浏览器在什么时候进行刷新，以及浏览器的时间是不准确的，而window提供一个原生API winsdow.requestAnimationFrame
函数给我们在页面刷新之前执行想要的操作。

如此便可以不用考虑浏览器刷新时机的问题了。这里不讨论rAF具体如何使用，以后会写相关的笔记来记录，留个坑先。

rAF解决了时机问题，但是，使用rAF不恰当操作页面的时候，比如频繁的改变元素的高度等会引起重流重绘的操作非常多的时候，浏览器这个时候为了能让动画
流畅，唯一能做的就是降帧，比如30fps，这样动画虽然慢，但是至少看起来是流畅的；

当然，可以不在一个rAF里面一次性做太多的DOM操作，可以把不需要那么快执行的放到下一帧或者下下帧去操作，从而使得引起重流的时间减少，但是其实这样做
本质还是没有改变，就是会引起重流重绘，只是可以去优化。

#### CSS3 transform

根据浏览器渲染流程图我们可以看出，假如我们做动画的时候，不造成reflow，那么就会大大提高性能。而利用CSS3制作动画，性能高于使用js的hack点就在于，
使用CSS3 transform的时候，该节点会独立成为一个composite layer，这样在发生动画的时候，不会对其它节点造成影响，不会reflow，
性能提高非常大。

关于composite layer这个知识点，也会有一篇文章专门讲一下，留个坑先。

但是过多的使用transform，会导致cpu和内存占用大，原因是cpu比之前需要管理更多的layer，这样自然会消耗更多的资源。

#### Summary

其实CSS3底层运用的也是类似rAF机制，不过没有深入了解过，也不好讨论。又留了个坑。

目前CSS3支持的动画效果可以从[ANIMATE](https://daneden.github.io/animate.css/)上看到，支持的种类其实也是有限的。所以，
一般而言，考虑到一些js动画库，性能方面其实已经优化的非常好了，比如[velocity.js](http://velocityjs.org)更是宣称自己的性能高于CSS3，
所以在制作复杂的动画的时候，js是首选，但是一般的动画可以用CSS3，毕竟不需要引入多一个库，就是最大的优点。

只是很浅显的记录了一些了解的内容，有纰漏的地方等以后了解深入了再回来修改。欢迎提issue ：）

##### References

1. [Youtube](https://www.youtube.com/watch?v=cCOL7MC4Pl0)
    
2. [Infoq](http://www.infoq.com/cn/articles/javascript-high-performance-animation-and-page-rendering)
    
3. [Event loop process model](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model)
    
4. [Google web developer](https://developers.google.com/web/fundamentals/design-and-ux/animations/css-vs-javascript?hl=zh-cn)
