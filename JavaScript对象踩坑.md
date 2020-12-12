# 重看JavaScript--对象踩坑

## 对象常用的两种定义方式
### 使用 new Object() 方式定义对象
```
let obj = new Object({'name': 'haha'})
```

### 使用对象字面量定义对象（即使用 {} 语法糖结构定义对象）
```
let obj = {'name': 'haha'}
```
其中需要特别注意的是
* 对象的key，也就是属性名是一个字符串，不是标识符，可以包含任意字符
* 引号可以省略，但是就算是省略了，属性名也还是字符串
* 所有省略了引号的属性名，都会自动变成字符串，如下面代码块，科学计数法被换算成了数值，然后变成了字符串，16进制也被换算成了10进制，然后变成了字符串
```
let obj = {
    1: 'a',
    3.2: 'b',
    1e2: 'c',
    1e-2: 'd',
    .234: 'f',
    0xFF: 'g'
}
console.log(Object.keys(obj)) 
// ["1", "100", "255", "3.2", "0.01", "0.234"]
```

## 用变量作对象的属性名
很多时候我们想用一个变量作为对象的属性名，但是直接把变量的名字作为属性名写进入发现，其实并没有把这个变量的值用作属性名
```
let name = 'firstName' + 'secondName'
let obj1 = {name: 'haha'}
console.log(obj1) // {"name": "haha"}
```
我们可以看到，直接把变量名作为属性名发现，变量名会直接变成字符串用作属性名。

但是加上 `[]` 就可以把变量用作属性名了
```
let obj2 = {[name]: 'haha'}
console.log(obj2) // {"firstsecond": "haha"}
```

## 对象的增删改查
### 删除属性
* 使用 `delete obj.xxx` 或 `delete obj['xxx']` 即可删除对象 obj 的 xxx 属性
* 判断属性是否在对象上
  * `'xxx' in obj` 包括原型对象里的
  * `obj.hasOwnProperty('xxx')` 返回一个布尔值，指示对象自身属性中是否具有 'xxx' 

### 查看对象所有属性
* `Object.keys(obj)` 用于查看自身所有属性
* `console.dir(obj)` 查看自身+原型对象里的属性

### 查看对象的属性的值
* `obj['key']`
* `obj.key`
* 注意，`obj[key]` 这个指的是变量 key 的值作为对象的key，而不是 'key' 作为对象的key

### 修改对象属性
* 直接赋值
  ```
  let obj = {name: 'haha'}
  obj.name = 'xixi'
  obj['name'] = 'hehe'
  ```
* 批量赋值
  ```
  // 直接赋值，会覆盖掉之前 obj 里的属性
  Object.assign(obj, {name: 18, gender: 'male'})
  ```

## 原型
### 每个对象都有原型
* 原型里存着对象的一些共有属性
* 比如 obj 的原型就是一个对象
* `obj.__proto__` 存着这个对象的地址
* 这个对象里有 toString / constructor / valueOf 等属性

### 对象的原型也是对象
* 所有对象的原型也有原型
* obj = {} 的原型即为所有对象的原型
* 这个原型包含所有对象的共有属性，是对象的根
* 这个原型也有原型，是 null

### 修改后增加原型上的属性
* 无法通过自身修改或者增加原型上的属性
  ```
  let obj = {}, obj2 = {}
  obj.toString = 'xxx' // 只会在 obj 自身属性中添加一个 toString 属性，值为 xxx
  obj2.toString 还是在原型上
  ```

* 一定要修改原型上的属性
  ```
  obj.__proto__toString = 'xxx'
  ```
  或者使用
  ```
  Object.prototype.toString = 'xxx'
  ```

### 修改原型
* 不推荐使用 `__proto__`
  ```
  let obj = {name: 'haha'}
  let common = {gender: 'male'}
  obj.__proto__ = common
  ```

* 推荐使用 `Object.create`
  ```
  // 先创建一个原型为 common 的对象 obj
  let obj = Object.create(common)
  // 再在 obj 上添加属性
  obj.name = 'haha'
  ```