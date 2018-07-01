# JavaScript之this指向

## 前言

对于JavaScript的this，我们需要的知道的是：

> this对象是在函数执行时基于函数的运行环境动态绑定的。

这一点对于我们理解this对象的指向尤为重要，也是这一特性，导致同一个函数可能由于你调用方式的不同，this对象的指向也发生改变。比如

```javascript
var name = 'jack';
var person = {
  name: 'bob',
  getName: function () {
    return this.name;
  }
}

// 方式一
console.log(person.getName());

// 方式二
var getName = person.getName;
console.log(getName());
```

执行一下上边的代码，我们会发现，两次打印的值并不一样。同一个函数，由于执行方法的不同，导致得到不同的结果。。

在不考虑箭头函数的情况下，一般函数的调用我们可以分为四种情况。


## 作为对象的方法被调用

当函数作为对象的方法被调用时，this一般指向对象本身。

```javascript
var person = {
  name: 'bob',
  getName: function () {
    console.log(this);
    return this.name;
  }
};
person.getName();

var obj = {
  name: 'obj',
  obj1: {
    name: 'obj1',
    obj2: {
      name: 'obj2',
      getName: function () {
        console.log(this);
        return this.name;
      }
    }
  }
}
obj.obj1.obj2.getName();
```

需要注意的是，当多层嵌套时，this指向他的直接调用者，本例中指向`obj2`

## 作为普通函数被调用

js天生适合函数式编程，我们可以编写一个函数，让他四处传递，作为普通函数被直接调用的情况，在JavaScript中尤为常见。此时this对象的指向为全局对象。

```javascript
var func1 = function () {
  console.log('fucn1',this);
};
function func2() {
  console.log('func2',this);
}

func1();

func2();

setTimeout(function(){console.log('settimeout',this)});

(function () {
  console.log('立即执行函数',this);
  var func3 = function () {
    console.log('func3', this);
  }
  func3();
})();

var obj = {
  _func: function () {
    var func5 = function () {
      console.log('func5', this);
    };
    func5();
  }
}
obj._func();
```

执行上述代码发现，所有打印的this都是指向window全局对象的（运行环境为浏览器），作为普通函数被调用时，函数的this对象均指向全局对象。

`func1`和`func2`不用多说，属于函数最基本的调用形式。对于setTimeout而言，这里给出的例子是一个匿名的回调函数，他的内部会将这个回调函数作为普通函数直接执行，所以也属于这种情况。立即执行函数的效果其实和`func1`或`func2`一样，只是写法上更加简洁，而内部的`func3`，也是作为普通函数被调用的。

到这里就可以解答前言中给出的两个例子：

```javascript
var name = 'jack';
var person = {
  name: 'bob',
  getName: function () {
    return this.name;
  }
}

// 方式一
console.log(person.getName());

// 方式二
var getName = person.getName;
console.log(getName());
```

对于方式一而言，没有疑问，函数作为对象的调用者指向对象本身。第二种情况此时函数被`getName`引用，即使申明时函数是作为对象属性的，但是this指向只能由执行时的上下文环境决定，故方式二中`getName`函数是作为普通函数被执行，返回的是全局的`name`，即`jack`。

## 作为构造函数被调用

new关键字在JavaScript中只类似原型链的语法糖，通过new关键字会生成一个新的对象，而此时的this指向的就是即将出生的对象。

```javascript
var Person = function (name, age) {
  this.name = name;
  this.age = age;
}

var jack = new Person('jack', 10);
```

this指向新生成的对象，即`jack`。

## apply、call和bind的调用

很多时候，我们需要明确的this指向，甚至动态的改变或者绑定this对象。`apply`、`call`和`bind`就可以动态的绑定this对象。

```javascript
var name = 'jack';
var person = {
  name: 'bob',
  getName: function () {
    return this.name;
  }
}

// 方式一

var getName = person.getName;
console.log(getName.apply(person)); // 'bob'

// 方式二
console.log(person.getName.call()); // 'jack'，默认绑定全局对象
```

[更多关于apply、call和bind的用法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

## 小结

到这里大部分关于this对象的问题（箭头函数除外），基本都能解决了。合理的使用this对象，会给我们的编程带来许多惊喜，但是由于其独特的性质，使用的时候也不能掉以轻心。
