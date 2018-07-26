title: Symbol类型及使用案例
date: 2016-05-21 14:45:23
tags: [ES6]
---

**摘要**：ES6 为 JavaScript 引入了一种新的基本类型：Symbol，它由全局 Symbol() 函数创建，每次调用 Symbol()函数，都会返回一个唯一的 Symbol。因为每个 Symbol 值都是唯一的，因此该值不与其它任何值相等 

ES6 为 JavaScript 引入了一种新的基本类型：Symbol，它由全局 Symbol() 函数创建，每次调用 Symbol()函数，都会返回一个唯一的 Symbol。

```javascript
let symbol1 = Symbol();
let symbol2 = Symbol();

console.log( symbol1 === symbol2 );
false
```

因为每个 Symbol 值都是唯一的，因此该值不与其它任何值相等。
Symbol 是 JavaScript 中的新原始类型。
```javascript
console.log( typeof symbol1 );
"symbol"
```

Symbol 充当唯一的对象键。
```javascript
let myObject = { 
    publicProperty: 'Value of myObject[ "publicProperty" ]'
};

myObject[ symbol1 ] = 'Value of myObject[ symbol1 ]';
myObject[ symbol2 ] = 'value of myObject[ symbol2 ]';

console.log( myObject );
Object
    publicProperty: "Value of myObject[ "publicProperty" ]"
    Symbol(): "Value of myObject[ symbol1 ]"
    Symbol(): "value of myObject[ symbol2 ]"
    __proto__: Object

console.log( myObject[ symbol1 ] );
Value of myObject[ symbol1 ]
```

当控制台打印`myObject`时，你能看到两个 Symbol 值都存储在对象中。`"Symbol()"` 是调用`toString()`的返回值，此值表示控制台中存在 Symbol 键。如果我们想访问正确的 Symbol，可以检索相应的值。

Symbol 键的属性不会在对象的 JSON 中显示，也不会在 for-in 循环和`Object.keys`中被枚举出来：
```javascript
JSON.stringify( myObject )
"{"publicProperty":"Value of myObject[ \"publicProperty\" ] "}"
 
for( var prop in myObject ) {
    console.log( prop, myObject[prop] );
}
publicProperty Value of myObject[ "publicProperty" ] 

console.log( Object.keys( myObject ) );
["publicProperty"]
```

即使 Symbol 键的属性没有在上述案例中出现，这些属性在严格意义上也不是完全私有的。`Object.getOwnPropertySymbols`提供了一种检索对象的 Symbol 键的方法。

```javascript
Object.getOwnPropertySymbols(myObject)
[Symbol(), Symbol()]
 
myObject[ Object.getOwnPropertySymbols(myObject)[0] ]
"Value of myObject[ symbol1 ]"
```
如果你使用 Symbol 键来表示私有变量，要确保不要用`Object.getOwnPropertySymbols`来检索可能私有化的属性。在这种情况下，`Object.getOwnPropertySymbols`的唯一使用情况就是测试和调试。

只要你遵循上述规则，从代码开发的角度来看，对象键值是私有的，但在实际情况中，其他人仍能访问你的私有值。

虽然 Symbol 键不能被`for...of`，扩展运算符和`Object.keys`枚举，但它们仍被包含在浅拷贝里：
```javascript
clonedObject = Object.assign( {}, myObject );

console.log( clonedObject );
Object
    publicProperty: "Value of myObject[ "publicProperty" ]"
    Symbol(): "Value of myObject[ symbol1 ]"
    Symbol(): "value of myObject[ symbol2 ]"
    __proto__: Object
```

正确命名 Symbol 对指明其用途至关重要，如果你需要额外的语义指导，还可在 Symbol 上附上一个描述。Symbol 的描述体现在 Symbol 的字符串值中。
```javascript
let leftNode = Symbol( 'Binary tree node' );
let rightNode = Symbol( 'Binary tree node' );

console.log( leftNode )
Symbol(Binary tree node)
```

始终提供 Symbol 的描述，并始终保持描述的唯一性。如果用 Symbol 访问私有属性，请将其描述视为变量名。

如果你将相同的描述传递给两个 Symbol，它们的值仍不相同。
```javascript
console.log( leftNode === rightNode );
false
```

### **全局 Symbol 注册表**
ES6 有一个用于创建 Symbol 的全局资源：Symbol 注册表，它为字符串和 Symbol 提供了一对一的关系。注册表使用 `Symbol.for( key )`返回 Symbol。

当出现`key1 === key2`时就会有`Symbol.for( key1 ) === Symbol.for( key2 )`。这种对应关系甚至是跨 service worker 和 iframe 的。
```javascript
let privateProperty1 = Symbol.for( 'firstName' );
let privateProperty2 = Symbol.for( 'firstName' );

myObject[ privateProperty1 ] = 'Dave';
myObject[ privateProperty2 ] = 'Zsolt';

console.log( myObject[ privateProperty1 ] );
// Zsolt
```

因为 Symbol 注册表中的 Symbol 值和字符串之间有一一对应的关系，所以我们也可以检索字符串键。使用`Symbol.keyFor`方法。

```javascript
Symbol.keyFor( privateProperty1 );
 "firstName"

Symbol.keyFor( Symbol() );
 undefined
```

### **Symbol 作为半私有属性键**
即使 Symbol 不能使属性私有，它们也能用作带有私有属性的符号。你可以使用 Symbol 来分隔公有和私有属性的枚举，Symbol 能使它更清楚。
```javascript
const _width = Symbol('width');
class Square {
    constructor( width0 ) {
        this[_width] = width0;
    }
    getWidth() {
        return this[_width];
    }
}
```


只要你能隐藏`_width`就行了，隐藏`_width`的方法之一是创建闭包：
```javascript
let Square = (function() {

    const _width = Symbol('width');

    class Square {
        constructor( width0 ) {
            this[_width] = width0;
        }
        getWidth() {
            return this[_width];
        }
    }
 
    return Square;  

} )();
```

这样做的好处是，他人很难访问到我们对象的私有`_width`值，而且也能很好地区分，哪些属性是公有的，哪些属性是私有的。但这种方法的缺点也很明显：

- 通过调用`Object.getOwnPropertySymbols`，我们可以使用 Symbol 键。
- 如果要写很多的代码，这会使得开发者的体验不佳，访问私有属性不像 Java 或 TypeScript 那样方便。

如果你要用 Symbol 来表示私有字段，那你需要表明哪些属性不能被公开访问，如若有人试图违背这一规则，理应承担相应的后果。

### **创建枚举类型**
枚举允许你定义具有语义名称和唯一值的常量。假定 Symbol 的值不同，它们能为枚举类型提供*好的值。

```javascript
const directions = {
    UP   : Symbol( 'UP' ),
    DOWN : Symbol( 'DOWN' ),
    LEFT : Symbol( 'LEFT' ),
    RIGHT: Symbol( 'RIGHT' )
};
```

### **避免名称冲突**
当使用 Symbol 作为变量时，我们不必建立可用标识符的全局注册表，也不必费心思想标识符名字，只需要创建一个 Symbol 就行了。
外部库的做法也是这样。

### **知名 Symbol**
[这里有一些比较常用的 Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol#Well-known_symbols)，用以访问和修改内部 JavaScript 行为。你可以用它们重新定义内置方法。运算符和循环。

### **演练**
**演练1.**用下划线来表示字段的私有，有什么利弊？用这种方法和 Symbol 比较。
```javascript
let mySquare {
    _width: 5,
    getWidth() { return _width; }
}
```

**利：**
- 开发者体验佳
- 不会造成复杂的代码结构

**弊：**
- 属性仅被表示为私有，在实践中并不是私有的，容易被破解
- 不同于 Symbol，这种方式的公有和私有属性没有很好地区分，私有属性出现在对象的公有接口中，它们使用能被扩展运算符，`Object.keys`和`for..of`循环枚举。

**演练2.** 模拟 JavaScript 中的私有字段。
**解决方案：**当涉及到构造函数时，可以使用`var`, `let`, 或 `const`在构造函数中声明私有成员。
```javascript
function F() {
   let privateProperty = 'b';
   this.publicProperty = 'a';
}

let f = new F();

// f.publicProperty returns 'a'
// f.privateProperty returns undefined 
```

为了对类使用相同的方法，我们必须放置方法定义：在可访问私有属性的作用域中的构造函数方法中使用私有属性的方法。我们将使用`Object.assign`来达到此目的。（灵感来自[Managing private data of ES6 classes](http://www.2ality.com/2016/01/private-data-classes.html)）
```javascript
class C {
    constructor() {
        let privateProperty = 'a';
        Object.assign( this, {
            logPrivateProperty() { console.log( privateProperty ); }
        } );
    }
}

let c = new C();
c.logPrivateProperty();
```

字段`privateProperty`在对象`c`中不可访问。
该解决方案也适用于我们扩展 C 类。

### 原文
http://www.phpchina.com/portal.php?mod=view&aid=40399