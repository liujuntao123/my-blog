---
title: 趣味探索promise的秘密
date: 2018-05-17 19:19:46
tags: 
- promise
- 异步编程 
categories: 深入JS
---

网上也有很多关于promise的讲解，但感觉都停留在理论层面，对于读者来说，看了一遍，似乎已经全部明白了，但真要用起来，发现还是无从下手。因此，本文旨在用人话为小伙伴们解释一下promise到底真正怎么用。
最近的项目里包含了大量的表单交互，必然免不了各种异步请求，promise使用相当频繁。我想，是时候来与小伙伴们分享一下我遇到的各种promise的坑了。

> promise干嘛用的？----一般用于异步操作。
为什么能用来异步操作？----因为能等一个函数执行完之后执行下一个函数操作。
懂了吗？----懂了
那好，用一个我看看----额，那个。。到底怎么写来着。。
你不是都懂了吗？----是啊，道理我都懂啊，但为什么就是不会用呢。。

哈哈，这就是我最初使用promise的深刻感受，明明那些概念都懂，语法也并不复杂，可就是实际用的时候用不出来。如果你也有这种感受，那么，恭喜你，来对地方了，跟着我继续探索promise的秘密吧！
## promise和then的配套使用
我们来想象这么一个生活情境：

> 开学了，幼儿园里的小朋友们排排坐，老师说：小朋友们，让我们来介绍自己的名字好不好呀，那么，从第一排的小红开始，依次往下介绍，好，我们开始吧！
小红站起来说，大家好，我叫小红，
接着小强站起来说，大家好，我叫小强
。
。
接着，轮到小明了，小明有个缺点，就是一紧张就口吃，只听他红着脸，站起来说：大。。大。。大。。大。。家。。好。。好。。
还没说完呢，后面的急性子小刚就站了起来，说：大家好，我叫小刚
说完，小刚后面的同学就接着站了起来，继续介绍自己。
最后，最后一排的同学也介绍完自己，坐下去了。此时，到家听到小明的声音：我叫。。叫。。叫。。。叫。。叫。。
此时，老师也不耐烦了，说：行了，小明，坐下吧
可怜的小明，就这样，连一个自我介绍都没有完成。。。

好了，亲，记住上面的故事了吗？我们开始学习promise。本来大家都好好的，按顺序自我介绍，挺好。但毕竟人生不如意十之八九，谁能想到小明有些口吃呢，所以小刚就抢着自我介绍了，你可能想说，这都怪小刚，可人家小刚还委屈呢，老师说的，按照座位来的，既然小明已经开始自我介绍了，那我就开始嘛，谁知道他这么慢呢。
对，小明就是这个异步操作，异步请求和接受服务器相应都是需要时间的，但是人家正常的js程序不管这些，一条语句执行了，下一条语句就会接着执行，不会去管上一条语句又没有完成，程序员你也没说要必须等上一条语句执行完才能执行下一条啊。
这样的问题在哪里呢，问题在最后，老师统计班上同学姓名的时候，会直接把小明漏掉，因为统计的时候，他都没有自我介绍完啊！也就是说，你后面的语句需要用到前面异步的返回值的时候，就会接收不到！
故事还没完呢，接下来听第二段：

> 第二年，小明班上换了一个班主任，这回小明学聪明了，提前就跟老师说明了他的情况，于是老师想了个办法。
老师说，同学们，我们今年的自我介绍，换一种方式，每一位同学在自我介绍之前，必须要先说出前一位同学的名字，才能进行自我介绍，这样小刚就算再性急，也必须要等小明自我介绍完了才能开始自我介绍。
多么聪明的老师啊，这样，任何同学，都能做完一个完整的自我介绍了！

老师的做法，就是promise。后一个同学必须要等钱一个同学说完，因为他需要前一个同学的名字。来看promise的语法：

``` js
new Promise((resolve,reject)=>{
// dosomething(异步操作);
	resolve(value)
})
```
把老师的做法翻译成js语句就是这样：
``` js
var m=new Promise((resolve,reject)=>{
    console.log('小明开始自我介绍')
    setTimeout(()=>{//用定时器模拟的异步操作(小明口吃)
        var name='小明'
        resolve(name)
    },3000)
})
m.then(model=>{
    var preName=model
    var name='小刚'
    console.log(preName)
    console.log(name)
})

```
执行结果如下：

![QQ截图20170208152427.jpg](http://upload-images.jianshu.io/upload_images/4050018-2fcdb92847858667.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你会发现，不光是小明在等待了3秒后才打印出来，小刚也一样是等待了3秒后才打印出来，这充分说明了promise的特点，同时我都不用再解释，相信then的用法相信小伙伴们也能很轻易的学会了！



## Promise.all的应用场合

接下来，说说Promise.all的应用场合，同样，上故事：

> 开学第二天，老师需要统计学生们的个人信息，于是让班长小红给每人发了一张个人信息表，让他们填写，并嘱咐她，一定要等所有的同学都填完了，再收上来。小红心里偷偷的笑了，肯定老师已经知道小华写字特别慢了。。

天啊，这到底是怎样一个班级，口吃的、写字慢的。。。为老师默哀一秒钟。。。
咳咳，说正事。其实老师的嘱咐，便是Promise.all的做法，Promise.all语法如下：

``` js
Promise.all(arr)
```

arr是一个包含Promise对象元素的数组，可能这么说还是有点抽象，那接下来把老师的做法翻译成js：
``` js
var xiaoming=new Promise((resolve,reject)=>{
    setTimeout(()=>{
        var info={name:'xiaoming',age:'5'}
        resolve(info)
    },2000)
})
var xiaogang=new Promise((resolve,reject)=>{
    setTimeout(()=>{
        var info={name:'xiaogang',age:'6'}
        resolve(info)
    },1000)
})
var xiaohua=new Promise((resolve,reject)=>{
    setTimeout(()=>{
        var info={name:'xiaohua',age:'7'}
        resolve(info)
    },5000)
})
var arr=[xiaoming,xiaogang,xiaohua]
Promise.all(arr).then(model=>{
    console.log(model)
})
```

结果如你所料，在等待了5秒后，控制台打印出了三个人的信息：


![QQ截图20170208154348.png](http://upload-images.jianshu.io/upload_images/4050018-e79d17705ee6304b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


Promise.all的用法也就一目了然了，它会让多个互不影响的Promise里的语句同时执行，等待所有的Promise语句全部执行完，然后执行then里的操作，返回给then的model是所有Promise里的resolve的值组成的一个数组。

## promise的异常处理

同志们，接下来，嘿嘿嘿，继续讲故事：
> 在填写个人信息的时候，小智的通知看到他填的年龄是8岁，于是指着他说：哈哈哈，你个智障，8岁还读幼儿园。。。小智恼羞成怒，和同桌大打了一架，打架中途，个人信息表也被同桌给撕了。这下子，班长小红苦恼了，老师可是说，要等每个人都写完再把他们所有的信息表收上来，这可怎么办啊。。。

好了，我们暂且先不要管这个班里为什么会有这么多奇葩。我们来看逻辑，本来异步请求正常执行完，接下来就执行then里的resolve，那现在如果向服务器请求失败了，又会怎么样呢，总不能还是执行吧，直接看代码：
``` js
var xiaozhi=new Promise((resolve,reject)=>{
    setTimeout(()=>{
        var info={name:'xiaozhi',age:'8'}
        var reason='打架,小智的信息表被撕了'
        if(reason){
            info=null
            reject(reason) 
        }else{
            resolve(info)
        }
        
    },5000)
})
xiaozhi.then(model=>{
    console.log(model)
},error=>{
    console.log('提交个人信息表失败')
    console.log('原因是'+error)
})
```

这段代码执行结果如下:


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4050018-eab2c9517852f81c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


小智因为这个事情，被老师狠狠的批评了一顿，他不禁想，如果时光能够倒流，当初没有打这个架该多好啊。小伙伴们，你应该明白我的意思了，不打架，reason就是undefined,就会进入else里的语句，就能正常的输出给then的model了。实际情况正是如此，你很难保证你的请求就一定能够成功响应，如果没有成功请求到数据，你就应该拦住它，执行请求失败的情况的语句。
可能有的小伙伴会错误的理解为，失败那不就是语句执行错误吗？这里一定要主要，这里的失败是指异步请求的失败，并不是js语句的执行失败。换句话说，这里的失败，是你自己定义的失败，如果你非要把请求成功定义为失败，那也会照样进入reject，而不会执行resolve。

## Promise.all的异常处理
Promise.all的异常处理是什么样子呢，看代码：
``` js
var xiaoming=new Promise((resolve,reject)=>{
    setTimeout(()=>{
        var info={name:'xiaoming',age:'5'}
        resolve(info)
    },2000)
})
var xiaogang=new Promise((resolve,reject)=>{
    setTimeout(()=>{
        var info={name:'xiaogang',age:'6'}
        resolve(info)
    },1000)
})
var xiaohua=new Promise((resolve,reject)=>{
    setTimeout(()=>{
        var info={name:'xiaohua',age:'7'}
        resolve(info)
    },5000)
})
var xiaozhi=new Promise((resolve,reject)=>{
    setTimeout(()=>{
        var info={name:'xiaozhi',age:'8'}
        var reason='打架,小智的信息表被撕了'
        if(reason){
            info=null
            reject(reason) 
        }else{
            resolve(info)
        }
    },1000)
})

var arr=[xiaoming,xiaogang,xiaohua,xiaozhi]
Promise.all(arr).then(model=>{
    console.log(model)
},error=>{
    console.log('收上来失败')
})
```
执行结果如下：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4050018-07fa0b30753b96d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


1秒的时候，打印出“收上来失败”这句话，就好比
> 小红把除了小智的其他人的信息表全部收上来了给老师，老师一看，少了小智的，说：少了小智的，重新发下去吧，这信息表不全，收上来我也不要。

看到了吧，Promise.all就是保证所有的Promise都是成功的，只要有一个失败，它就执行reject，并且其他的哪怕是成功的Promise结果也都全部舍弃。这叫，一个都不能少！

怎么样，小伙伴们，看了这篇文章，是否如同剥开了Promise身上的那层薄纱，将她看了个仔细呢(咳咳，别想歪。。。)。
#### 如果觉得文章不错，请在下方点赞支持我哦！

### 参考资料
> [MDN官方文档--Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)