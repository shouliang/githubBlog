title: 三.hook机制和测试技巧
date: 2014-06-10 16:45:23

tags: [Node.js_测试]
---

### 1.hook机制
---
hook 就是在测试流程的不同时段触发，比如在整个测试流程之前，或在每个独立测试之前等。

hook也可以理解为是一些逻辑，通常表现为一个函数或者一些声明，当特定的事件触发时 hook 才执行。

提供方法有：before()、beforeEach() after() 和 afterEach()。

__方法解析：__

* before()：所有测试用例的统一前置动作
* after()：所有测试用例的统一后置动作
* beforeEach()：每个测试用例的前置动作
* afterEach()：每个测试用例的后置动作
用法：
```
describe('hooks', function() {
  before(function() {
    //在执行本区块的所有测试之前执行
  });
 
  after(function() {
    //在执行本区块的所有测试之后执行
  });
 
  beforeEach(function() {
    //在执行本区块的每个测试之前都执行
  });
 
  afterEach(function() {
    //在执行本区块的每个测试之后都执行
  });
 
  //测试用例
 
});
```

### 2.描述hook
---
所有的 hook 都可以加上描述，这样可以更好地定位到测试用例中的错误。如果 hook 函数指定了名称，会在没有描述时使用函数名，例如：
```
beforeEach(function() {
  //beforeEach hook
});
 
beforeEach(function needFun() {
  //beforeEach: namedFun
});
 
beforeEach('some description', function() {
  //beforeEach:some description
});
```

补充：上述示例中 注释的内容就是对 hook 的描述。

### 3.测试占位
---
测试用例占位只要添加一个没有回调的 it() 方法即可：
```
var assert = require('assert');
 
describe('Array', function() {
 
  describe('#indexOf()', function() {
    //同步测试
    it('当值不存在时应该返回 -1', function() {
      assert.equal(-1, [1,2,3].indexOf(5));
      assert.equal(-1, [1,2,3].indexOf(0));
    });
  });
 
  describe('Array', function() {
    describe('#indexOf()', function() {
      //下面是一个挂起的测试
      it('当值不存在时应该返回 -1');
    });
  });
});
```

### 4.仅执行指定测试
---
仅执行指定测试的特性可以让你通过添加 .only() 来指定唯一要执行的测试套件或测试用例：
```
describe('Array', function(){
  describe.only('#indexOf()', function(){    
  })
})
```
或一个指定的测试用例：

```
describe('Array', function(){
  describe('#indexOf()', function(){
    it.only('当值不存在时应该返回 -1', function(){
 
    }) 
    it('当值不存在时应该返回 -1', function(){
     
    })
  })
})
```

PS:注意只能出现一个 .only()


### 5.忽略指定测试
---
该特性和 .only() 非常相似，通过添加 .skip() 你可以告诉 Mocha 忽略的测试套件或者测试用例（可以有多个）。该操作使得这些操作处于挂起的状态，这比使用注释来的要好，因为你可能会忘记把注释给取消掉。
```
describe('Array', function(){
  describe.skip('#indexOf()', function(){
    ...
  })
})
```
或一个指定的测试用例：

```
describe('Array', function(){
  describe('#indexOf()', function(){
    it.skip('当值不存在时应该返回 -1', function(){
 
    })
 
    it('当值不存在时应该返回 -1', function(){
 
    })
  })
})
```

### 6.动态生成测试
---
由于mocha 可以使用 function.prototype.call 和function 表达式定义测试套件和测试用例，所以可以动态生成测试用例。
```
var assert = require('assert');
 
function add() {
  return Array.prototype.slice.call(arguments).reduce(function(prev, curr) {
    return prev + curr;
  }, 0);
}
 
describe('add()', function() {
  var tests = [
    {args: [1, 2],       expected: 3},
    {args: [1, 2, 3],    expected: 6},
    {args: [1, 2, 3, 4], expected: 10}
  ];
 
  tests.forEach(function(test) {
    it('correctly adds ' + test.args.length + ' args', function() {
      var res = add.apply(null, test.args);
      assert.equal(res, test.expected);
    });
  });
});
```
