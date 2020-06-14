## this
this指的是当前正在执行或调用该函数的对象的值。this值的变化取决于我们使用它的上下文和我们在哪里使用它。
区分是否严格模式（严格模式下禁止this指向全局对象，部分window指向是变成undefined）
Reference 类型：只存在于规范里的抽象类型，描述语言的底层行为逻辑才存在的，由base（属性所在的对象或者就是 EnvironmentRecord）、name、strict组成
ECMAScript规范解读：通过判断()左侧MemberExpression的返回的结果是不是一个Reference类型（是否调用 GetValue，调用后返回的将是具体的值，而不再是一个 Reference）
是Reference：1、如果IsPropertyReference(MemberExpression)，如果base value 是一个对象，结果返回 true，如果base value是Environment Record，返回undefined，this指向这个结果
不是Reference：this 的值为 undefined，非严格模式下隐式转换为全局对象
var value = 1;
var foo = {
  value: 2,
  bar: function () {
    return this.value;
  }
}
// 因为使用了 GetValue，所以返回的不是 Reference 类型，this 为 undefined
console.log((false || foo.bar)()); // 1

实际场景：
1、作为对象调用时，指向该对象 obj.b(); // 指向obj
2、作为函数调用, var b = obj.b; b(); // 指向全局window
3、作为构造函数调用 var b = new Fun(); // this指向当前实例对象
4、作为call与apply调用 obj.b.apply(object, []); // this指向当前的object
5、事件监听函数中的this指向被绑定的dom元素，注意需要动态绑定this，不能使用箭头函数
6、HTML标签的属性中是可能写JS的，这种情况下this指代该HTML元素
7、箭头函数比较特殊，没有自己的this，它使用封闭执行上下文(函数或是global，变量的作用域链)的 this 值。
8、计时器回调函数中的this指向window，如果是箭头函数是封闭执行上下文(函数或是global，变量的作用域链)的 this 值

## call,bind,apply
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
