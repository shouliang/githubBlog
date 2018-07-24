title: JavaScript函数参数传递到底是值传递还是引用传递
date: 2015-04-05 22:16:53

tags: [Node.js_JS基础]
---

在传统的观念里，都认为JavaScript函数传递的是引用传递(也称之为指针传递)，也有人认为是值传递和引用传递都具备。那么JS的参数传递到底是怎么回事呢？事实上以下的演示也完全可以用于Java

首先来一个比较简单的，基本类型的传递:
```javascript
function add(num){
   num+=10;
   return num;
}
num=10;
console.log(add(num));
console.log(num);
//输出20,10
```

  对于这里的输出20,10，按照JS的官方解释就是在基本类型参数传递的时候，做了一件复制栈帧的拷贝动作，这样外部声明的变量num和函数参数的num，拥有完全相同的值，但拥有完全不同的参数地址，两者谁都不认识谁，在函数调用返回的时候弹出函数参数num栈帧。所以改变函数参数num，对原有的外部变量没有一点影响。

再来看一个较复杂的，对象引用类型的传递:
```javascript
function setName(obj){
    obj.name="ted";
}
var obj=new Object();
setName(obj);
console.log(obj.name);
//输出ted
```

  以上代码的运行的实质是:创建了一个object对象，将其引用赋给obj(在C里面就直接是一个内存地址的赋值)，然后在传递函数参数的时候，做了一件与前一个方法相同的事情，复制了一个栈帧给函数参数的obj，两者拥有相同的值(不妨将其理解为object对象的地址)，然后在setName做改变的时候，事实上是改变了object对象自身的值(在JAVA里称之为可变类)，在改变完成之后同样也要弹出函数参数obj对应的栈帧。

所以对应的输出是改变后object对象的值

  那么可能有的朋友可能会问，这样也可以理解为一个引用传递(指针传递)呀？不，这里严格的说，在和JAVA类似的语言中，已经没有了指针，在JAVA里将上述过程称之为一个从符号引用到直接引用的解析过程。在C里面，指针就是一个具有固定长度的类型(在大多数的C编译器里是2个字节)，但在JAVA类似的语言里，引用也有自己的属性和方法，只是你不能直接去访问和控制它，所以它从某种意义上也是一种对象，这种机制也很大程度的避免了内存泄露，术语称之为内存结构化访问机制。

为了证明上述观点，稍微改造下上述例子:

```javascript
function setName(obj){
	obj.name="zhangsan";
	obj=new Object();
	obj.name="lisi";
}
var obj=new Object();
setName(obj);
console.log(obj.name);
//输出zhangsan
```
  这个例子与上一个例子的唯一不同是这里将一个新的对象赋给了函数参数obj，这样函数参数obj和原有的引用obj参数，有着完全不同的值和内存地址。
