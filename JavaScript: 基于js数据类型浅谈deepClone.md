## 基于js数据类型浅谈deepClone

谈到deepClone，对于一些刚入手前端的小伙伴可能会发现一些问题，举个简单的例子：

```
let a = {apple: 1}
let b = a
b.apple = 2
console.log(a.apple) // 2
```
我们会发现，修改b的时候，a也被修改了，这种情况下在我们一般的项目中，不是我们想要的结果，如何保持a不变呢，这时候就会需要深拷贝这个东西。在这之前呢，有个东西你必须先熟悉，那就是js的数据类型，这对了解深拷贝是至关重要的。

### 一、数据类型

在JS中，数据类型分为基本数据类型和引用数据类型两种，分别存在内存中的栈和堆中。
+ 栈（stack）中主要存放一些基本类型的变量和对象的引用（包含引用类型的值的变量并不是对象本身，而是一个指向该对象的指针，复制该对象的话，实际上是复制该对象的指针），其优势是存取速度比堆要快，并且栈内的数据可以共享，但缺点是存在栈中的数据大小与生存期必须是确定的，缺乏灵活性。
+ 堆（heap）用于复杂数据类型（引用类型）分配空间，例如数组对象、object对象；它是运行时动态分配内存的，因此存取速度较慢。

#### 1.基本数据类型

基本数据类型的值直接存储在栈内存中；栈有一个很重要的特殊性，就是存在栈中的数据可以共享。例如下面的代码定义两个变量，变量的值都是数字类型。

```
var a=3; 
var b=3;
```
JavaScript将首先处理 var a=3;,会在栈中创建一个变量为a引用，然后查找栈中是否有3这个值，如果没有找到，就将3存放进来，然后将a指向3。接着处理 var b=3;,在创建为b的引用变量后，查找栈中是否有3这个值，因为此时栈中已经存在了3，便将b直接指向3。这样，就出现了a与b同时指向3的情况。此时，如果再令a=4，那么JavaScript解释引擎会重新搜查栈中是否有4这个值，如果有，则将4存放进来，并令a指向4；如果已经有了，则直接将a指向这个地址。因此a值的改变不会影响到b的值。

#### 2.引用数据类型

引用类型在栈内存中仅仅存储了一个引用，而真正的数据存储在堆内存中；

具体的例子，在开头的例子中，就属于引用数据类型的现象。对象是引用类型的值，对于引用类型来说，我们将 a 赋予 b 的时候，我们其实仅仅只是将 a 存储在栈堆中的的引用赋予了 b ，而两个对象此时指向的是在堆内存中的同一个数据，所以当我们修改任意一个值的时候，修改的都是堆内存中的数据，而不是引用，所以只要修改了，同样引用的对象的值也自然而然的发生了改变。

这种情况，就是一个浅拷贝，就是只拷贝对象的引用，而不深层次的拷贝对象的值，多个对象指向堆内存中的同一对象，任何一个修改都会使得所有对象的值修改，因为它们公用一条数据。

### 二、深拷贝

深拷贝作用在引用类型上，例如：Object，Array；深拷贝不会拷贝引用类型的引用，而是将引用类型的值全部拷贝一份，形成一个新的引用类型。

下面介绍几种简易的方式：

#### 1.最low版：JSON.stringify()以及JSON.parse()

```
var obj1 = {
    a: 1,
    b: 2
}
var objString = JSON.stringify(obj1);
var obj2 = JSON.parse(objString);
obj2.a = 3;
console.log(obj1.a);  // 1
console.log(obj2.a); // 3
```

此方法不可以拷贝 undefined ， function， RegExp 等等类型的。

#### 2.Object.assign(target, obj)

```
 var obj1 = {
    a: 1,
    b: 2
}
var obj2 = Object.assign({}, obj1);
obj2.b = 3;
console.log(obj1.b); // 2
console.log(obj2.b); // 3
```
此方法不可以用于多层嵌套的对象。

#### 3.使用扩展运算符 {...}

```
var obj1 = {
    a: 1,
    b: 2
}
var obj2 = {...obj1};
obj2.b = 3;
console.log(obj1.b); // 2
console.log(obj2.b); // 3
```

#### 4.递归函数
我们可以递归去复制所有层级属性，以实现深拷贝：

```
function deepClone(obj, clone) {
  //判断拷贝的要进行深拷贝的是数组还是对象，是数组的话进行数组拷贝，对象的话进行对象拷贝
  const toString = Object.prototype.toString
  toString.call(obj) === '[object Array]' ? clone = clone || [] : clone = clone || {}
  for (const i in obj) {
    if (typeof obj[i] === 'object' && obj[i]!==null) {
      // 要考虑深复制问题了
      if (Array.isArray(obj[i])) {
        // 这是数组
        clone[i] = []
      } else {
        // 这是对象
        clone[i] = {}
      }
      deepClone(obj[i], clone[i])
    } else {
      clone[i] = obj[i]
    }
  }
  return clone
}

// 使用方法：
let a = deepClone(b)
```