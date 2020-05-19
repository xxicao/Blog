# call , bind , apply
作用：令 this 指向传递的第一个参数，如果第一个参数为 null ，undefined 或是不传，则指向全局对象。
#### 注意：
- call、apply、bind 是 Function.prototype 下的方法，是每个函数都包含的方法，用于改变函数运行时上下文。
- apply 和 call 方法的第一个参数都是特定的作用域，第二个参数不同，call 的参数是从第二个开始罗列，apply 是将参数以数组形式作为第二个参数。
- `bind()`方法调用并改变函数运行时上下文后，返回一个新的函数，供我们需要时再调用。

### 手写apply,bind,call
#### Apply
原理：利用对象调用函数时，内部 this 指向该对象的特性来改变 this ，当调用者不是 function 等临界条件先不做判断
```javascript
Function.prototype.myApply = function (context, args) {
    //这里默认不传或者null和undefined就是给window,也可以用es6给参数设置默认参数
    context = context || window;
    args = args ? args : [];
    //给context新增一个独一无二的属性以免覆盖原有属性
    const key = Symbol();
    // this指向myApply的调用函数，即我们要改变指向的函数，为上下文context设置一个属性，值为this即调用函数
    context[key] = this;
    //通过隐式绑定的方式调用函数，类似于obj.say()，此时函数内部this指向context
    const result = context[key](...args);
    //删除添加的属性
    delete context[key];
    //返回函数调用的返回值
    return result;
}
```
#### Call
call 的实现将 args 改为`...args`接收多个参数即可，或用 arguments 实现
#### Bind
bind 的实现返回一个函数可以传参数，同时多判断一个 new 操作符
```javascript
Function.prototype.myBind = function (context, ...args) { 
    // 取调用函数，需要改变上下文的函数
    const fn = this;
    args = args ? args : [] ;
    return function newFn(...newFnArgs) { 
    // 此处this是new的时候的构造函数内部this
    if (this instanceof newFn) { 
        return new fn(...args, ...newFnArgs) 
    } 
        return fn.apply(context, [...args,...newFnArgs]) 
    } ;
}
```

