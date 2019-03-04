title: Mocha—二.mocha接口
date: 2014-06-08 15:45:23

tags: [Node.js]
---

### 1.BDD行为驱动开发
---
mocha “接口” 系统允许开发者选择自身喜爱的特定领域语言风格, mocha 提供 TDD（测试驱动开发）、BDD (行为驱动开发) 和 exports 风格的接口。

BDD是“行为驱动的开发”（Behavior-Driven Development）的简称，指的是写出优秀测试的最佳实践的总称。

BDD认为，不应该针对代码的实现细节写测试，而是要针对行为写测试。BDD测试的是行为，即软件应该怎样运行。

BDD接口提供以下方法：

* describe()：测试套件
* it()：测试用例
* before()：所有测试用例的统一前置动作
* after()：所有测试用例的统一后置动作
* beforeEach()：每个测试用例的前置动作
* afterEach()：每个测试用例的后置动作
BDD的特征就是使用describe()和it() 这两个方法。

before()、after()、beforeEach()和afterEach() 是为测试做辅助的作用域，它们合起来组成了hook的概念。

### 2.方法解析
---
#### descript()
describe()方法接收两个参数：第一个参数是一个字符串，表示测试套件的名字或标题，表示将要测试什么。第二个参数是一个函数，用来实现这个测试套件。

上述中引出了一个概念：测试套件。那什么是测试套件呢？

测试套件（test suite）指的是，一组针对软件规格的某个方面的测试用例。也可以看作，对软件的某个方面的描述（describe）。结构如下：
```
describe("A suite", function() {
  // ...
});
```

#### it()
要想理解it()，首先我们要知道什么是测试用例? 测试用例（test case）指的是，针对软件一个功能点的测试，是软件测试的最基本单位。一组相关的测试用例，构成一个测试套件。

测试用例由it函数构成，它与describe函数一样，接受两个参数：第一个参数是字符串，表示测试用例的标题；第二个参数是函数，用来实现这个测试用例。

#### BDD风格用例
```
var expect = require('chai').expect;
describe('Array', function(){
  before(function(){
    console.log('在测试之前运行');
  });
  describe('#indexOf()', function(){
    it('当值不存在时应该返回 -1', function(){
      expect([1,2,3].indexOf(4)).to.equal(-1);
    });
  });
});
```

### 3.TDD测试驱动开发
---
#### TDD风格
TDD（测试驱动开发）组织方式是使用测试集（suite）和测试（test）。

每个测试集都有 setup 和 teardown 函数。这些方法会在测试集中的测试执行前执行，它们的作用是为了避免代码重复以及最大限度使得测试之间相互独立。

TDD接口：

* suite：类似BDD中 describe()
* test：类似BDD中 it()
* setup：类似BDD中 before()
* teardown：类似BDD中 after()
* suiteSetup：类似BDD中 beforeEach()
* suiteTeardown：类似BDD中 afterEach()
示例
```
var assert = require("assert");

suite('Array', function(){
  setup(function(){
    console.log('测试执行前执行');
  });
  
  suite('#indexOf()', function(){
    test('当值不存在时应该返回 -1', function(){
      assert.equal(-1, [1,2,3].indexOf(4));
    });
  });
});
```
运行mocha：

 __mocha --ui tdd *.js__ (\*表示的是文件名)

PS：mocha 默认是使用 bdd 的接口，所以在这里我们告诉mocha我们用的是tdd.

### 4.exports风格
---
exports类似于nodejs里的模块语法,关键字 before, after, beforeEach, 和 afterEach 是特殊保留的，值为对象时是一个测试套件，为函数时则是一个测试用例。

#### 示例如下：
```
module.exports = {
  before: function(){
    // ...
  }, 
  
  'Array': {
     '#indexOf()': {
        '当值不存在时应该返回 -1': function(){
          expect([1,2,3].indexOf(4)).to.equal(-1);
        }
      }
  }
};
```
#### 运行 mocha

mocha --ui exports *.js


### 5.小结
---
我们在前文中讲到了 mocha 提供 TDD（测试驱动开发）、BDD (行为驱动开发) 和 exports 风格的接口。其实还有 QUnit 和 require 风格的接口。

但是比较常用就是 BDD 和 TDD，mocha 默认的也是BDD，且 BDD 是 TDD 的一个专业版本，它指定了从业务需求的角度出发需要哪些单元测试。
