# call , bind , apply
令this指向传递的第一个参数，如果第一个参数为null，undefined或是不传，则指向全局变量
1、call、apply、bind 是Function.prototype下的方法，是每个函数都包含的方法，用于改变函数运行时上下文
2、apply和call方法的第一个参数都是特定的作用域，第二个参数不同，call的参数是从第二个开始罗列，apply是将参数以数组形式作为第二个参数
3、bind()方法调用并改变函数运行时上下文后，返回一个新的函数，供我们需要时再调用
手写apply,bind,call原理：利用对象调用函数时，内部this指向该对象的特性改变this
https://juejin.im/post/5d2ddd9be51d4556d86c7b79#heading-10
过程：用的不是function就返回或者抛出错误等临界条件另做判断
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
call的实现即将args改为...args接收多个参数，或用arguments实现
bind的实现返回一个函数可以传参数，同时多判断一个new操作符
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
