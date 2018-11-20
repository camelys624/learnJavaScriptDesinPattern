# 深入理解JavaScript

### (5)强大的原型和原型链

JavaScript不包含传统的类继承模型，而是使用prototype原型模型.

虽然这经常被当作是JavaScript的缺点被提及，其实基于原型的继承模型比传统的类继承还要强大。

#### 原型使用方式1：

通过对Calculator对象的prototype属性赋值对象字面量来设定Calculator对象的原型.

``` js
let Calculator = function (decimalDigits, tax) {
  this.decimalDigits = decimalDigits;
  this.tax = tax;
};

Calucator.prototype = {
  add: function (x, y) {
    return x + y;
  },
  subtract: function (x, y) {
    return x - y;
  }
};
```

#### 原型使用方式2：

在赋值原型prototype的时候使用function立即执行的表达式来赋值:

``` js
Claculator.prototype = function () {  }();
```

它可以封装私有的function,通过return 的形式暴露出简单的使用名称，以达到public/private的效果：

``` js
Calculator.prototype = function () {
  add = function (x, y) {
    return x + y;
  },
  subtract = function (x, y) {
    return x - y;
  }
  return {
    add: add,
    subtract: subtract
  }
}();
```

#### 分步声明:

上述使用原型的时候，有一个限制就是一次性设置了原型对象，下面是分步设置原型的每个属性。

``` js
let BaseCalculator = function () {
  // 为每个示例都声明一个小数位数
  this.decimalDigits = 2;
};

// 使用原型给BaseCalculator扩展2个对象方法
BaseCalculator.prototype.add = function (x, y) {
  return x + y;
};

BaseCalculator.prototype.subtract = function (x, y) {
  return x - y;
};
```

BaseCalculator构造函数里面初始化了一个小数位数的属性decimalDigits，然后通过原型属性设置两个function，分别是add(x, y)和subtract(x, y)，当然可以使用上面两种方式的任意一种(**更希望是第二种**),我们的主要目的是将BaseCalculator对象设置到真正的Calculator的原型上.

``` js
let Calculator = function () {
  // 为每个实例都声明一个税收数字
  this.tax = 5;
};

Calculator.prototype = new BaseCalculator();
```

我们可以看到Calculator的原型是指向到BaseCalculator的一个实例上，目的是让Calculator继承它的那两个方法，而且，还有一点需要说的是，由于它的原型是BaseCalculatorde的一个实例，所以不管创建多少个Calculator对象实例，他们的原型指向的都是同一个实例。

``` js
let calc = new Calucator();
alert(calc.add(1, 1));
// BaseCalculator里声明的decimalDigits属性，在Calculator里是可以访问到的
alert(calc.decimalDigits);
```

上面的代码运行之后，我们可以暗道因为Calculator的原型是指向BaseCalculator的实例上的，所以可以访问它的decimalDigits。

如果不想让Calculator访问BaseCalculator的构造函数里声明的属性值。

``` js
let Calculator = function () {
  this.tax = 5;
}

Calucator.prototype = BaseCalculator.prototype;
```

通过将BaseCalculator的原型赋给Calculator的原型，这样在Calculator的实例上就访问不到那个decimalDigits值了，如果范文如下代码，就会报错：

``` js
let calc = new Calculator();
alert(calc.add(1, 1));
alert(calc.decimalDigits);  // undefined
```

#### 重写原型

在使用第三方JS类库的时候，往往有时候它们定义的原型方法是不能满足我们的需要，我们可以重写他们的原型中的一个或多个属性或function，我们可以通过继续声明同样的add代码的形式来达到覆盖重写前面的add功能.

``` js
// 覆盖前面的Calculator的add() function
Calculator.prototype.add = function (x, y) {
  return x + y + this.tax;
};

let calc = new Calculator();
alert(calc.add(1, 1)) // -> 7
```

这样，我们计算得出的结果就比原来多出了一个tax的值，**重写的代码需要放在最后，这样才能覆盖前面的代码**

### 原型链

``` js
function Foo() {
    this.value = 42;
}
Foo.prototype = {
    method: function() {}
};

function Bar() {}

// 设置Bar的prototype属性为Foo的实例对象
Bar.prototype = new Foo();
Bar.prototype.foo = 'Hello World';

// 修正Bar.prototype.constructor为Bar本身
Bar.prototype.constructor = Bar;

var test = new Bar() // 创建Bar的一个新实例
```

它的原型链如下所示：

``` js
// 原型链
test [Bar的实例]
    Bar.prototype [Foo的实例]
        { foo: 'Hello World' }
        Foo.prototype
            {method: ...};
            Object.prototype
                {toString: ... /* etc. */};
```

可以看到，他只包含父类的prototype原型属性上的方法，并不包含父类自带的方法。

#### 属性查找

当查找一个对象的属性时，JavaScript会向上遍历原型链，直到找到给定名称的属性为止，到查找到达到原型链的顶部-也就是Object.prototype-但是任然没有找到指定的属性，就会返回undefind。

``` js
function  foo() {
  this.add = function (x, y) {
    return x + y;
  }
}

foo.prototype.add = function (x, y) {
  return x + y + 10;
}

Object.prototype.subtract = function (x, y) {
  return x - y;
};

let f = new foo();
alert(f.add(1, 2)); // -> 3, 不是13
aler(f.subtract(1, 2)); // -> -1
```

属性在查找的时候是先查找自身的属性，如果没有再查找原型，再没有，再往上走，一直查到Object的原型上，所以在某种层面上说，用for in语句遍历属性的时候，效率也是个问题。(**可以通过下面的hasOwnProperty函数破解**)。

还有一点需要注意的是，我们可以赋值任何类型的对象到原型上，但是不能赋值**原子类型**的值，比如下面的代码是无效的：

``` js
function Foo () {}
Foo.prototype = 1;  // 无效
```

#### hasOwnProperty函数

hasOwnProperty是Object.prototype的一个方法，它是个好东西，它能判断一个对象是否包含自定义属性而不是圆形上的属性，因为hasOwnProperty是JavaScript中唯一一个处理属性但是不查找原型链的函数。

``` js
// 修改Object.prototype
Object.prototype.bar = 1;
let foo = {goo: undefined};

foo.bar;  // -> 1
'bar' in foo; // -> true

foo.hasOwnProperty('bar');  // -> false
foo.hasOwnProperty('goo');  // -> true
```

只有hasOwnProperty可以正确得到预期的结果，这在遍历对象的属性时会很有用，没有其他方法可以用来排除原型链上的属性，而不是定义在对象自身上的属性。

但是有个恶心的地方是：JavaScript不会保护hasOwnProperty被非法占用，因此如果一个对象碰巧存在这个属性，就需要使用外部的hasOwnProperty函数来获取正确的结果。

``` js
let foo = {
  hasOwnProperty: function () {
    return false;
  },
  bar: 'Here be dragons'
};

foo.hasOwnProperty('bar');  // 总是返回false

// 使用{}对象的hasOwnProperty,并将其上下设置为foo
({}).hasOwnProperty.call(foo, 'bar'); // true
```
