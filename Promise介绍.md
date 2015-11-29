## Promise介绍

### 介绍
Promise 对象用于延迟(deferred) 计算和异步(asynchronous ) 计算.。一个Promise对象代表着一个还未完成，但预期将来会完成的操作。
Promise 对象是一个返回值的代理，这个返回值在promise对象创建时未必已知。它允许你为异步操作的成功或失败指定处理方法。 这使得异步方法可以像同步方法那样返回值：异步方法会返回一个包含了原返回值的 promise 对象来替代原返回值。
> 引自[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)

### 它解决什么问题
一个简单的示例
执行一个动画A，执行完之后再去执行另一个动画B

```
	setTimeout(function(){
		//A动画
		console.log('A');
		setTimeout(function() {
			//B动画
			console.log('B');
		},300)
	},300);
```
这里只有两个动画，如果有更多呢，就会看到一堆函数缩进

### 一种写法
> 浏览器实现方式 可以在支持Promise的版本上运行
```
var p = new Promise(function(resolve, reject){
  setTimeout(function(){
    //A动画
    console.log('A');
    resolve();
  },300);
});

p.then(function(){
  setTimeout(function() {
      //B动画
      console.log('B');
  },300);
});
```
### 另一种写法（jQuery版本）

>jQuery版本的实现

```
var deferred = $.Deferred();
setTimeout(function(){
  //A动画
  console.log('A');
  deferred.resolve();
},300);

deferred.done(function() {
  setTimeout(function() {
    //B动画
    console.log('B');
  },300)
});
```
好像从代码上来看，是多了几行的样子，但是能用这种串行的方式来写，感觉一定很爽吧

### Promise中的概念
Promise中有几个状态：
+ pending: 初始状态, 非 fulfilled 或 rejected.
+ fulfilled: 成功的操作.
+ rejected: 失败的操作.

这里从pending状态可以切换到fulfill状态（jQuery中是resolve状态），也可以从pengding切换到reject状态，这个状态切换不可逆，且fulfilled和reject两个状态之间是不能互相切换的。

[配图]


### 一个简单版本的实现
```
/**
 * simple promise 
 * @param {[type]} fun [description]
 */
function PromiseB(fun) {

    this.succArg = undefined;
    this.failArg = undefined;
    this.succCbs = [];
    this.failCbs = [];
    this._status = this.STATUS.PENDING;

    this._execFun(fun);
}

PromiseB.prototype.STATUS = {
    PENDING: 1, //挂起状态
    RESOLVE: 2, //完成状态
    REJECT: 3 //拒绝状态
};


PromiseB.prototype._isFunction = function(f) {
    return Object.prototype.toString.call(f) === '[object Function]';
};

PromiseB.prototype._exec = function(callback, arg) {
    var newcallback;


    if (this._isFunction(callback)) {
        if (callback instanceof PromiseB) {
            callback.resolve(arg);
        } else {
            newcallback = new PromiseB(callback);
            newcallback.resolve(arg);
        }
    }
};

PromiseB.prototype._execFun = function(fun) {
    var that = this;

    if (this._isFunction(fun)) {
        fun(function() {
            that.succArg = Array.prototype.slice.apply(arguments);
            that._status = that.STATUS.RESOLVE;

            that.resolve.apply(that, arguments);
        }, function() {
            that.failArg = Array.prototype.slice.apply(arguments);
            that._status = that.STATUS.REJECT;

            that.reject.apply(that, arguments);
        });
    } else {
        this.resolve(fun);
    }


};

PromiseB.prototype.resolve = function() {
    var arg = arguments,
        ret,
        callback = this.succCbs.shift();
    if (this._status === this.STATUS.RESOLVE && callback) {
        ret = callback.apply(callback, arg);
        if (!(ret instanceof PromiseB)) {
            var _ret = ret;
            ret = new PromiseB(function(resolve) {
                setTimeout(function() {
                    resolve(_ret);
                });
            });

            ret.succCbs = this.succCbs.slice();
        }
        // this._exec(callback.apply(callback, arg), arg);
    }
};

PromiseB.prototype.reject = function() {
    var arg = arguments,
        ret,
        callback = this.failCbs.shift();
    if (this._status === this.STATUS.REJECT && callback) {
        ret = callback.apply(callback, arg);
        if (!(ret instanceof PromiseB)) {
            var _ret = ret;
            ret = new PromiseB(function(resolve) {
                setTimeout(function() {
                    resolve(_ret);
                }, 200);
            });
            ret.failCbs = this.failCbs.slice();
        }
    }
};

PromiseB.prototype.then = function(s, f) {
    this.done(s);
    this.fail(f);
    return this;
};

PromiseB.prototype.done = function(fun) {
    if (this._isFunction(fun)) {
        if (this._status === this.STATUS.RESOLVE) {
            fun.apply(fun, this.succArg);
        } else {
            this.succCbs.push(fun);
        }
    }
    return this;
};

PromiseB.prototype.fail = function(fun) {
    if (this._isFunction(fun)) {
        if (this._status === this.STATUS.REJECT) {
            fun.apply(fun, this.failArg);
        } else {
            this.failCbs.push(fun);
        }
    }
    return this;
};

PromiseB.prototype.always = function(fun) {
    this.done(fun);
    this.fail(fun);
    return this;
};


```

### 总结
+ promise会让代码变得更容易维护，像写同步代码一样写异步代码
+ 了解promise的原理，写个简单的实现版本就好了
+ promise的实现方案有很多，可以看[这里](https://github.com/nodejs/node-v0.x-archive/wiki/modules#wiki-async-flow)
+ [这里](https://github.com/moonye6/PromiseB)有项目代码和几个demo，方便查看

### 相关阅读
+ [Promise - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
+ [jQuery.Callbacks()](http://www.css88.com/jqapi-1.9/jQuery.Callbacks/)
+ [如何实现一个ECMAScript 6 的promise功能](http://www.html-js.com/article/JavaScript-tips-on-how-to-implement-a-ECMAScript-6-promise-patch)