# [JavaScript Generator Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) 生成器对象 #

> 1. Iterable Interface
> 2. Iterator Interface
> 3. IteratorResult Interface
> 4. Generator
> 5. `yield`、`yield*`
> 6. generator + promise 应用
> 7. 总结

自定义迭代器是一个有用的工具，但由于需要显式地维护其内部状态，因此需要谨慎地创建。

而 [ES6](http://es6.ruanyifeng.com/) 引入的生成器函数为我们提供了一个强大的选择：它允许你定义一个包含自有迭代算法的函数， 同时它可以自动维护自己的状态。 

在正式说 Generator 之前，我们先了解一下什么是 *`Iterable`* 和 *`Iterator`* 接口。

**1、 The Iterable Interface**

*`Iterable`* 接口包含一个必需的属性 **`@@iterator`**。

<table align="center">
  <caption><p><b>Iterable Interface Required Properties</b></p></caption>
  <thead>
    <tr>
      <th width="110px">Propert</th>
      <th>Value</th>
      <th>Requirements</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code><i>@@iterator</i></code></td>
      <td>一个返回 <code><i>Iterator</i></code> 对象的函数.</td>
      <td>返回的对象必须符合 <code><i>Iterator</i></code> 接口.</td>
    </tr>
  </tbody>
</table>

**2、The Iterator Interface**

*`Iterator`* 接口包含一个必需属性 **next** 和两个可选属性 **return**、**throw**。

<table>
  <caption><p><b>Iterator Interface Required Properties</b></p></caption>
  <thead>
    <tr>
      <th width="100px">Propert</th>
      <th width="200px">Value</th>
      <th>Requirements</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>next</th>
      <td>一个返回 <code><i>IteratorResult</i></code> 对象的函数。</td>
      <td>返回的对象必须符合 <code><i>IteratorResult</i></code> 接口。如果上一次对 <code><i>Iterator</i></code> <b>next</b> 方法的调用返回了一个 <code>done: true</code> 的 <code><i>IteratorResult</i></code> 对象。那么所有后续对该对象 <b>next</b> 方法的调用也应该返回一个 <code>done: true</code> 的 <code><i>IteratorResult</i></code> 对象。</td>
    </tr>
  </tbody>
</table>

**next** 函数还可以传递参数，但它们的解释和有效性取决于目标 *`Iterator`* 。

<table align="center">
  <caption><p><b>Iterator Interface Optional Properties</b></p></caption>
  <thead>
    <tr>
      <th width="100px">Propert</th>
      <th width="200px">Value</th>
      <th>Requirements</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>return</th>
      <td>一个返回 <code><i>IteratorResult</i></code> 对象的函数.</td>
      <td>返回的对象必须符合 <code><i>IteratorResult</i></code> 接口。调用这个方法，是想告诉 <code><i>Iterator</i></code> 对象，调用者不再打算对 <code><i>Iterator</i></code> 执行 <b>next</b> 方法的调用。返回的 <code><i>IteratorResult</i></code> 对象通常有一个 <code>done: true</code> 的属性和一个 <b>value</b> 属性，它的值是作为参数传递给 <b>return</b> 方法的值。</td>
    </tr>
    <tr>
      <th>throw</th>
      <td>一个返回 <code><i>IteratorResult</i></code> 对象的函数.</td>
      <td>返回的对象必须符合 <code><i>IteratorResult</i></code> 接口。调用这个方法，是想告诉 <code><i>Iterator</i></code> 对象，调用者检测到一个错误情况。参数可用于识别错误条件，通常是一个异常对象。典型的响应是抛出作为参数传递的值。如果方法没有 <b>throw</b>，即没有抛出异常，返回的 <code><i>IteratorResult</i></code> 对象通常会有一个 <code>done: true</code> 的属性。</td>
    </tr>
  </tbody>
</table>

**3、 The IteratorResult Interface**

*`IteratorResult`* 一般包含 `true` 和 `false` 两个属性

<table align="center">
  <caption><p><b>IteratorResult Interface Properties</b></p></caption>
  <thead>
    <tr>
      <th width="100px">Propert</th>
      <th width="110px">Value</th>
      <th>Requirements</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>done</th>
      <td>Either <b>true</b> or <b>false</b></td>
      <td>这是一个 <code><i>iterator</i></code> <b>next</b> 方法调用的结果状态。如果是 <code><i>iterator</i></code> 的最后一次调用 <code>done: true</code>。否则， <code>done: false</code>, 并且有一个可用的 value 值。如果 <b>done</b> 属性（自身或者继承）不存在，则被认为 false。</td>
    </tr>
    <tr>
      <th>value</th>
      <td>Any <br/> ECMAScript <br/>language <br/> value.</td>
      <td>如果 <b>done</b> 的值是 <b>false</b>，这个值是当前迭代元素的值。如果 <b>done</b> 的值是 <b>true</b>，且 <b>value</b> 值存在的话，那么这个值是 <code><i>iterator</i></code> 的返回值。 如果迭代器没有返回值, <b>value</b> 的值是 <b>undefined</b>。在这种情况下，如果没有继承显示的 value 属性，那么这个 value 属性在 <code><i>IteratorResult</i></code> 接口的对象中可能是不存在的</td>
    </tr>
  </tbody>
</table>

```javascript
var myIterator = {
  [Symbol.iterator]() {
    let index = 0
    return {
      next() {
        return {
          done: index > 3,
          value: index++
        }
      }
    }
  }
}
```

现在我们定义了一个可迭代对象，那么我们怎么去使用它呢？

> Tips：当一个对象需要被迭代的时候，它的 ` @@iterator` 方法被调用，然后返回一个用于在迭代中获得值的迭代器。

所以，我们这样去做

```javascript
var iterator = myIterator[Symbol.iterator]();

iterator.next(); // { value: 0, done: false }
iterator.next(); // { value: 1, done: false }
iterator.next(); // { value: 2, done: false }
iterator.next(); // { value: 3, done: false }
iterator.next(); // { value: 4, done: true }
```

我们知道，ES6 中新增了 *`for..of`* 、`...` 、`解构赋值` 语句，都是可以用来处理迭代行为的。我们尝试在 `myIterator` 上使用它：

```javascript
for (let i of myIterator) console.log(i);
// 0
// 1
// 2
// 3

[...myIterator];
// [0, 1, 2, 3]

const [a, b, c, d] = myIterator
a; // 0
b; // 1
c; // 2
d; // 3
```

JavaScript 中已经实现的可迭代对象：
- 内置对象
    1. `Array`
    2. `TypeArray`
    3. `Map`
    4. `Set`
    5. `String`
- 其他可迭代对象
    1. `HTMLCollection`
    2. `NodeList`
    3. `Arguments`（数组参数集合）

**4、Generator**

ok，了解了 *`Iterable`* 和 *`Iterator`* 接口，我们来看一下什么是 **Generator Object** ？

 **Generator 对象是由 *`generator function`* 返回的，并且符合 *`Iterable`* 和 *`Iterator`* 接口。**

语法：

```javascript
function* gen() { 
  yield 1;
  yield 2;
  yield 3;
}
// 一旦生成器函数已定义，可以通过构造一个迭代器来使用它。
var g = gen(); // "Generator { }"
typeof g.next; // "function", because it has a next method, so it's an iterator
typeof g[Symbol.iterator]; // "function", because it has an @@iterator method, so it's an iterable

g.next(); // {value: 1, done: false}
g.next(); // {value: 2, done: false}
g.next(); // {value: 3, done: false}
g.next(); // {value: undefined, done: true}
```

可以看出，Generator 既是一个迭代器对象又是一个可迭代对象。我们还可以这样使用它：

```javascript
[...g]; // [1, 2, 3]

for (let i of g) console.log(i);
// 1
// 2
// 3

const [a, b, c] = g
a; // 1
b; // 2
c; // 3
```

根据上面的例子，我们实现一个从 start 到 end 的一个连续数字数组。

```javascript
function* gen1(start, end) {
  let index = start;
  while (index <= end) {
    yield index++;
  }
}
var g1 = gen1(1, 10);
[...g1]; // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

**5、yield、yield***

到了这里，有些同学可能比较疑惑，*`yield`* 是个什么东东？

***`yield`* 是一个 JavaScipt 关键字，用来暂停和恢复一个生成器函数。**

语法：

```javascript
[rv] = yield [expression]; // [] 表示可选
```
- `expression` 定义生成器函数返回的值，如果省略，则返回 *`undefined`*
- `rv` 返回传递给生成器的 **next** 方法的可选值，以恢复其执行。

*`yield`* 使生成器函数执行暂停，*`yield`* 后面的表达式的值返回给生成器的调用者。它可以被认为是一个基于生成器的版本的 *`return`* 关键字。

*`yield`* 实际返回一个 *`IteratorResult`* 对象，它有两个属性，**value** 和 **done**。**value** 属性是对 *`yield`* 表达式求值的结果，而 **done** 是 **false**，表示生成器函数尚未完全完成。

一旦遇到 `yield` 表达式，生成器的代码将被暂停运行，直到生成器的 **next** 方法被调用。每次调用生成器的 **next** 方法时，生成器都会恢复执行，直到达到以下某个值：
- *`yield`* 导致生成器再次暂停并返回生成器的新值。下一次调用 **next** 时，在 *`yield`* 之后紧接着的语句继续执行。
- *`throw`* 用于从生成器中抛出异常。这让生成器完全停止执行。
- 到达生成器函数的结尾。在这种情况下，生成器的执行结束，并且返回一个 **done** 为 **true**，**value** 为 **undefined** 的 *`IteratorResult`* 对象。
- 遇到 *`return`* 语句。在这种情况下，生成器的执行结束，并将 *`IteratorResult`* 返回给调用者，其 **value** 值是由 *`return`* 语句指定的，并且 **done** 为 **true**。

如果将参数传递给生成器的 **next** 方法，则该值将成为生成器当前 *`yield`* 操作返回的值。不过，正如上面所述，参数的存在是否有意义，取决于目标 *`Iterator`*。

>  Tips：**next** 方法接受一个参数用于修改生成器内部状态。传递给 **next** 的参数值会被 *`yield`* 接收。要注意的是，传给第一个 *`next`* 的值会被忽略。

```javascript
function* gen() {
  yield 1;
  var a = yield 2;
  yield a * 2;
}
var g = gen();
```
我们声明了一个变量 **a** 用来接收 *`yield`* 表达式的返回值，而 *`yield`* 的返回值是由 **next** 函数传递的，上面的语句看起来好像应该在第二次调用 **next** 的时候传递参数，并在第三次调用后 `* 2` 输出，我们来试一下，到底是不是这样的呢？

```javascript
g.next();   // { value: 1, done: false }
g.next(10); // { value: 2, done: false }
g.next();   // { value: NaN, done: false }
g.next();   // { value: undefined, done: true }
```

Oops ! NaN ? 不是我们期望的结果，那么内部怎么去做的呢？

在分析之前，我们先理解下面这条语句：
```javascript
var a = yield 2;
```
猜想一下，假使这条语句执行了两个动作：

1. 一条 *`return`* 语句，即 `return 2;`
2. 一条赋值语句（`=`）即 `var a = yield;`

所以，当迭代器遇到 `var a = yield 2;` 时，我们是否可以这样解释：**迭代器在 动作 1 之后，动作 2 之前暂停了。** ok，开始分析

```javascript
function* gen() {
  yield 1;
  var a = yield 2;
  yield a * 2;
}
var g = gen();
```
当我们完成了上面的代码，它只是给我们生成了一个迭代器，但是还没有启动。（运动员已经预备备了，就等发令枪了。）

```javascript
g.next(); // { value: 1, done: false }
```

**First，** 第一次调用 **next()**，迭代器开始启动。（发令枪响了，开始跑吧。不一样的是，咱有红绿灯，不是你想跑想跑就能跑。）当遇到了 `yield 1;` ，迭代器暂停了，停在的  *`yield`* （红灯）**前**。代码只执行到 **1**，即 `return 1;` 。

```javascript
g.next(10); // { value: 2, done: false }
```

**Second，** 第二次调用 **next(10)**，迭代器恢复。（绿灯亮了，警察叔叔放行了，继续跑吧！哎，不行，这样回去好像交不了差 `[当然这只是你的错觉]` ，于是你对警察叔叔说：警察叔叔，你看我辣么可爱，可不可以给我一只小红旗呀？警察叔叔有求必应，这个时候你得到了一只小红旗 **10**。）

当遇到了 `var a = yield 2;` ，迭代器暂停了，停在的 `var a = yield;` （红灯）前。代码只执行到 **2**，即 `return 2;` 。所以，在这里我们得不到我们期望的值。

在这里 **10** 作为参数传递给 **next** 是没有意义的。因为我们根本用不到它。如果你非要这样做，我只能说可以但没必要。

```javascript
g.next(); // { value: NaN, done: false }
```

**Third，** 第三次调用 **next()**，为什么返回 `{ value: NaN, done: false }` 呢？其实很简单，首先，代码会执行 `var a = yield;` 并将传递给 **next()** 的参数作为值赋给 **a**，可是我们什么也没有传入，所以是 **a** 的值 `undefined` 。
而 `undefined * 2` ，得到的是 `NaN` 。所以，在这里我们也得不到我们期望的值。

所以，我们应该在这里执行 **next(10)**，而不是在第二步执行。

综上所述，我们的猜想是正确的，在这个例子中，我们想要在函数内准确接收到我们传递的值，应该这样顺序执行：

```javascript
function* gen() {
  yield 1;
  var a = yield 2;
  yield a * 2;
}
var g = gen();

g.next();   // { value: 1, done: false }
g.next();   // { value: 2, done: false }
g.next(10); // { value: 20, done: false }
g.next();   // { value: undefined, done: true }
```

为了加深理解，下面是一个 `斐波那契数列生成器`，它使用了 `next(true)` 来重新启动序列，有兴趣的同学可以看一下。

```javascript
function* fibonacci() {
  var fn1 = 0;
  var fn2 = 1;
  while (true) {
    var current = fn1;
    fn1 = fn2;
    fn2 = current + fn1;
    var reset = yield current;
    if (reset) {
      fn1 = 0;
      fn2 = 1;
    }
  }
}

var sequence = fibonacci();
console.log(sequence.next().value);     // 0
console.log(sequence.next().value);     // 1
console.log(sequence.next().value);     // 1
console.log(sequence.next().value);     // 2
console.log(sequence.next().value);     // 3
console.log(sequence.next().value);     // 5
console.log(sequence.next().value);     // 8
console.log(sequence.next(true).value); // 0
console.log(sequence.next().value);     // 1
console.log(sequence.next().value);     // 1
console.log(sequence.next().value);     // 2
```

如果你搞懂了 *`yield`*， *`yield*`* 对你来说，简直是小菜一碟，我们在这里只是提一嘴。提供几个例子供大家参考。

**`yield*` 表达式用于委托给另一个 `generator` 或 可迭代对象。**

语法

```javascript
yield* [[expression]];
```

- `expression` 一个返回可迭代对象的表达式。

Examples：

```javascript
function* g1() {
  yield 2;
  yield 3;
  yield 4;
}

function* g2() {
  yield 1;
  yield* g1();
  yield 5;
}

var iterator = g2();
console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: 4, done: false}
console.log(iterator.next()); // {value: 5, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```


```javascript
function* g3() {
  yield* [1, 2];
  yield* '34'; // String 一个类数组对象
  yield* Array.from(arguments);
}

var iterator = g3(5, 6);

console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: "3", done: false}
console.log(iterator.next()); // {value: "4", done: false}
console.log(iterator.next()); // {value: 5, done: false}
console.log(iterator.next()); // {value: 6, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```

```javascript
function* g4() {
  yield* [1, 2, 3];
  return 'foo';
}

var result;

function* g5() {
  result = yield* g4();
}

var iterator = g5();

console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: undefined, done: true}, 
                              // g4() returned {value: 'foo', done: true} at this point

console.log(result);          // "foo"
```

好的，上面我们已经对 `generator` 、 `yield` 、`yield*` 做了一些知识储备，也对 `generator` 的工作机制做了一下简单分析。

那么，`generator` 究竟有哪些实际应用场景呢？请继续往下看


**6、Generator + Promise 应用**

思考下面这段代码：

**code 1**

```javascript
function getAjaxData() {
  var result = $.ajax({url: 'http://xxx', dataType: 'json'})
  console.log(result);
}
```

我们期望拿到的是异步请求到的结果数据，就像这样：

```javascript
{
  success: true,
  code: 200,
  data: {}
}
// 或者这样
{
  success: false,
  message: 'java.lang.NullPointerException'
}
```

然而，我们都知道，这是错误的，我们只能这样写（但都逃不过链式调用和回调函数）：

```javascript
$.ajax({url: 'http://xxx', dataType: 'json'})
.done(res => {
  var result = res
})
.fail(err => {})

// 或者
$.ajax({
  url: 'http://xxx',
  dataType: 'json',
  success() {
    var result = res
  },
  error() {}
})
```

有没有什么办法可以帮助我们实现 **code 1** 中的写法呢？答案是肯定的，是时候让我们的好朋友 `Generator` 上场了！

```javascript
// 声明一个辅助函数 Async，接收一个 generator 作为参数
function Async(gen) {
  var g = gen();
  
  function next(val) {
    const iterRst = g.next(val);
    if (iterRst.done) {
      return iterRst.value;
    } else {
      iterRst.value.then(res => {
        next(res);
      })
    }
  }
  
  next();
}

function* getAjaxData() {
  var result1 = yield Promise.resolve(1);
  console.log(result1);
  var result2 = yield Promise.resolve(2);
  console.log(result2);
}

Async(getAjaxData); // 控制台依次输出：1, 2
```

Perfect！我们终于得到了我们想要的结果了。（Generator 大法好，呱唧呱唧。）

`Generator` 和 `Promise` 配合使用，为我们提供了一种解决异步操作的新方法。

如果你了解 [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) 、[await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) 的语法，你会发现两者十分相似：

```javascript
async function gen() {
  const result1 = await Promise.resolve(1);
  console.log(result1);
  const result2 = await Promise.resolve(2);
  console.log(result2);
};

gen(); // 控制台依次输出：1, 2
```

**7、总结**
1. Generator 对象是由 `generator function` 返回的，既是一个迭代器对象，也是一个可迭代对象。
2. 我们可以声明一个 `generator function` 来定义我们自己的迭代行为，并通过构造一个实例来使用它。
3. 我们通常搭配 `yield`、`yield*` 来使用 `generator function`，可以解锁更多操作。
4. ES6 为我们提供了一些处理迭代行为的语法， `for...of`、`...`、`解构赋值`。
5. `Generator` 和 `Promise` 配合使用，为我们提供了一种解决异步操作的新方法，这也是 `async` 、`await` 实现原理。
