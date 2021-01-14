# JavaScript 封装和继承

## 一、构造函数模式封装对象
JavaScript 是一种基于对象（object-based）的语言，你遇到的所有的东西几乎都是对象。但是，它又不是一种真正的面向对象编程（OOP）的语言，因为它的语法中没有真正的 class（类）。说到这里，可能会有人要说 ECMAScript2015（ES6）已经引入了 class（类）这个概念，但你如果仔细去看 ES6 中的 class 的话，那你就应该明白，ES6 的 class 也只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。

那么，我们应该怎么把 property 和 method 封装成一个对象呢？

为了解决从原型对象生成实例的问题，Javascript提供了一个构造函数（Constructor）模式。

所谓"构造函数"，其实就是一个普通函数，但是内部使用了this变量。对构造函数使用new运算符，就能生成实例，并且this变量会绑定在实例对象上。

比如我们要创建一个关于猫的原型对象:
```
function Cat(name,color){
    this.name = name;
    this.color = color;
}
```

现在我们就可以生成实例对象了：
```
var cat1 = new Cat("大毛"， "黄色");
```

构造函数很好用，但是存在一个内存浪费的问题

请看，我们现在为Cat对象添加一个不变的属性type（种类），再添加一个方法eat（吃）。那么，原型对象Cat就变成了下面这样：
```
function Cat(name,color){
    this.name = name;
    this.color = color;
    this.type = "猫";
    this.eat = function(){
        console.log("吃老鼠");
    } 
}
```

然后再实例化对象的时候，对于每一个实例对象，type属性和eat()方法都是一模一样的内容，每一次生成一个实例，都必须为重复的内容，多占用一些内存。这样既不环保，也缺乏效率。


为了解决这个问题，就需要让 `type` 和 `eat()` 只在内存中生成一次，然后所有实例都指向那个内存地址。


Javascript规定，每一个构造函数都有一个prototype属性，指向另一个对象。这个对象的所有属性和方法，都会被构造函数的实例继承。

这意味着，我们可以把那些不变的属性和方法，直接定义在prototype对象上。
```
function Cat(name,color){
    this.name = name;
    this.color = color;
}

Cat.prototype.type = "猫";
Cat.prototype.eat = function(){
    console.log("吃老鼠");
}
```

这时所有实例的type属性和eat()方法，其实都是同一个内存地址，指向prototype对象，因此就提高了运行效率。


## 二、构造函数的继承
文章的第一部分介绍了如何“封装”数据和方法，以及如何从原型对象生成实例，那第二部分就开始介绍对象之间继承的方法。

比如，现在有一个“动物”对象的构造函数：
```
function Animal(){
    this.species = "动物";
}
```

还有一个“猫”对象的构造函数
```
function Cat(name,color){
    this.name = name;
    this.color = color;
}
```

怎样才能使“猫”继承“动物”呢？

### 1.借用构造函数继承

第一种方法也是最简单的方法，使用call或apply方法，将父对象的构造函数绑定在子对象上，即在子对象构造函数中加一行：
```
function Animal() {
    this.species = "动物";
}
function Cat(name, color) {
    Animal.apply(this, arguments);
    this.name = name;
    this.color = color;
}

var cat1 = new Cat("大毛", "黄色");
console.log(cat1.species);    // 动物
```

但是这种继承方法有个问题就是不能访问原型上的属性和方法

### 2.使用 prototype（原型链继承）
```
Cat.prototype = new Animal();
//任何一个prototype对象都有一个constructor属性，指向它的构造函数.如果没有"Cat.prototype = new Animal();"这一行，Cat.prototype.constructor是指向Cat的；加了这一行以后，Cat.prototype.constructor指向Animal。每一个实例也有一个constructor属性，默认调用prototype对象的constructor属性
Cat.prototype.constructor = Cat;//手动纠正，将Cat.prototype对象的constructor值改回Cat
var cat1 = new Cat("大毛","黄色");
alert(cat1.species); // 动物
```

此方法中我们手动纠正了 constructor 属性，这是很重要的，再以后的编程中务必要遵守，即如果替换了 prototype 对象，那么，下一步必然是为新的prototype对象加上constructor属性，并将这个属性指回原来的构造函数。
```
o.prototype = {};
o.prototype.constructor = o;
```

### 3.直接继承 prototype
```
// 先改写 Animal 对象
function Animal(){}
Animal.prototype.species = "动物";

// 然后，将Cat的prototype对象，然后指向Animal的prototype对象，这样就完成了继承。
Cat.prototype = Animal.prototype;
Cat.prototype.constructor = Cat;//修正原型上的constructor属性，指回原来的构造函数
var cat1 = new Cat("大毛","黄色");
alert(cat1.species); // 动物
```

与前一种方法相比，这样做的优点是效率比较高（不用执行和建立Animal的实例了），比较省内存。缺点是 Cat.prototype和Animal.prototype现在指向了同一个对象，那么任何对Cat.prototype的修改，都会反映到Animal.prototype

所以，上面这一段代码其实是有问题的。请看第二行：
```
Cat.prototype.constructor = Cat;
```
这一句实际上把 Amimal.prototype 对象的 constructor 属性也改掉了

### 4. 利用空对象作为中介

刚说到了"直接继承 prototype"存在一些缺点，我们可以利用一个空对象来作为中介解决。

```
var F = function(){};
F.prototype = Animal.prototype;
Cat.prototype = new F();
Cat.prototype.constructor = Cat;
```

F是空对象，所以几乎不占内存。这时，修改Cat的prototype对象，就不会影响到Animal的prototype对象。

我们将上面的方法，封装成一个函数，便于使用：
```
function extend(Child, Parent){
    var F = function(){};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.prototype.constructor = Child;
    Child.uber = Parent.prototype;
    // 最后一行意思是为子对象设一个uber属性，这个属性直接指向父对象的prototype属性。（uber是一个德语词，意思是"向上"、"上一层"。）这等于在子对象上打开一条通道，可以直接调用父对象的方法。这一行放在这里，只是为了实现继承的完备性，纯属备用性质。
}

// 使用此方法
extend(Cat, Animal);
var cat1 = new Cat("大毛"， 黄色);
```

### 5.组合式继承

调用父类构造函数，继承父类属性，通过将父类实例作为子类原型，实现函数复用

```
function Cat(name, color) {
    Animal.apply(this, arguments);
    this.name = name;
    this.color = color;
}

Cat.prototype = new Animal();
Cat.prototype.constructor = Cat;
```

但是这种方法有一个缺点：由于调用了两次父类，所以产生了两份实例


### 6.继承组合继承

通过寄生的方式（利用空对象作为中介），来修复组合式继承的不足，实现完美的继承，我们来封装两个对象，实现继承，模拟真实场景：
```
//父类
function People(name,age){
  this.name = name || 'wangxiao'
  this.age = age || 27
}
//父类方法
People.prototype.eat = function(){
  return this.name + this.age + 'eat sleep'
}
//子类
function Woman(name,age){
  //继承父类属性
  People.call(this,name,age)
}
//继承父类方法
(function(){
  // 创建空类
  var Super = function(){};
  Super.prototype = People.prototype;
  //父类的实例作为子类的原型
  Woman.prototype = new Super();
})();
//修复构造函数指向问题
Woman.prototype.constructor = Woman;
var womanObj = new Woman();
```