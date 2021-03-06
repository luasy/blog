# JS 规范



为了让阅读者更好的理解和识别，对于要进行规范的范式使用标识符进行一定的划分。

`e` ：表示 `eslint`，我们将会去 `eslint` 中去实现。

`p` ：表示 `person`，需要开发者了解的。(ps: 当然，所有的规则开发者都了解更佳。)

`u` ：表示 `unknown`，表示当前规则时候有现有 `eslint` 规则。

`t` ：表示 `tool`, 最好能通过三方库的方式来引解决(三方库可以是内部自己的库)。

## 声明

- `e` 1.1 所有的赋值都用`const`，避免使用`var`。eslint: [`prefer-const`](https://eslint.org/docs/rules/prefer-const.html), [`no-const-assign`](https://eslint.org/docs/rules/no-const-assign.html)

> 为什么？这确保您不会再被修改的声明被重新赋值，这可能会导致bug和难以理解的代码。

```js
// bad
var a = 1;
var b = 2;

// good
const a = 1;
const b = 2;
```

- `e` 2.2 如果你一定要对参数重新赋值，那就用`let`，而不是`var`.  eslint: [`no-var`](https://eslint.org/docs/rules/no-var.html)

> 为什么？ `let` 是一个块级作用于而不是想 `var` 是一个函数作用域。

```js
// bad
var count = 1;
if (true) {
  count += 1;
}

// good, use the let.
let count = 1;
if (true) {
  count += 1;
}
```

- `p` 2.3 注意，`let` 和 `const` 都是块级作用域。

```js
// const 和 let 都只存在于它定义的那个块级作用域
{
  let a = 1;
  const b = 1;
}
console.log(a); // ReferenceError
console.log(b); // ReferenceError
```



## Objects

- `e` 2.1 使用字面值创建对象. eslint: [`no-new-object`](http://eslint.org/docs/rules/no-new-object.html)

- `p` 2.2 当创建一个带有动态属性名的对象时，用计算后属性名

> Why? 这可以使你将定义的所有属性放在对象的一个地方.

```js
function getKey(k) {
  return `a key named ${k}`;
}

// bad
const obj = {
  id: 5,
  name: 'San Francisco',
};
obj[getKey('enabled')] = true;

// good getKey('enabled')是动态属性名
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};
```

- `e` 2.3 用对象方法简写. eslint: [`object-shorthand`](http://eslint.org/docs/rules/object-shorthand.html)

```js
// bad
const atom = {
  value: 1,

  addValue: function (value) {
    return atom.value + value;
  },
};

// good
const atom = {
  value: 1,

  // 对象的方法
  addValue(value) {
    return atom.value + value;
  },
};
```

-  `e` 3.4 用属性值缩写. eslint: [`object-shorthand`](http://eslint.org/docs/rules/object-shorthand.html)

> Why? 这样写的更少且更可读

```js
const lukeSkywalker = 'Luke Skywalker';

// bad
const obj = {
  lukeSkywalker: lukeSkywalker,
};

// good
const obj = {
  lukeSkywalker,
};
```

- `u` 3.5 将你的所有缩写放在对象声明的开始.

> Why? 这样也是为了更方便的知道有哪些属性用了缩写.

```js
const anakinSkywalker = 'Anakin Skywalker';
const lukeSkywalker = 'Luke Skywalker';

// bad
const obj = {
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  lukeSkywalker,
  episodeThree: 3,
  mayTheFourth: 4,
  anakinSkywalker,
};

// good
const obj = {
  lukeSkywalker,
  anakinSkywalker,
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  episodeThree: 3,
  mayTheFourth: 4,
};
```

- `e` 3.6 只对那些无效的标示使用引号 `''`. eslint: [`quote-props`](http://eslint.org/docs/rules/quote-props.html)

> Why? 通常我们认为这种方式主观上易读。他优化了代码高亮，并且也更容易被许多JS引擎压缩。

```js
// bad
const bad = {
  'foo': 3,
  'bar': 4,
  'data-blah': 5,
};

// good
const good = {
  foo: 3,
  bar: 4,
  'data-blah': 5,
};
```

- `t` 3.7 不要直接调用`Object.prototype`上的方法，如`hasOwnProperty`, `propertyIsEnumerable`, `isPrototypeOf`。

> Why? 在一些有问题的对象上， 这些方法可能会被屏蔽掉 - 如：`{ hasOwnProperty: false }` - 或这是一个空对象`Object.create(null)`

```js
// bad
console.log(object.hasOwnProperty(key));

// good
console.log(Object.prototype.hasOwnProperty.call(object, key));

// best
const has = Object.prototype.hasOwnProperty; // 在模块作用内做一次缓存
/* or */
import has from 'has'; // https://www.npmjs.com/package/has
// ...
console.log(has.call(object, key));
```

- `p` 3.8 对象浅拷贝时，更推荐使用扩展运算符[就是 `···`运算符]，而不是`Object.assign`。获取对象指定的几个属性时，用对象的rest解构运算符[也是`…`运算符]更好。
  - 这一段不太好翻译出来， 大家看下面的例子就懂了。^.^

> 注意：建议在拷贝时用，但如是深拷贝，还是使用三方库更佳。此外，如果是赋值操作，使用第一个没问题。

```js
// very bad
const original = { a: 1, b: 2 };
const copy = Object.assign(original, { c: 3 }); // this mutates `original` ಠ_ಠ
delete copy.a; // so does this

// bad
const original = { a: 1, b: 2 };
const copy = Object.assign({}, original, { c: 3 }); // copy => { a: 1, b: 2, c: 3 }

// good es6扩展运算符 ...
const original = { a: 1, b: 2 };
// 浅拷贝
const copy = { ...original, c: 3 }; // copy => { a: 1, b: 2, c: 3 }

// rest 赋值运算符
const { a, ...noA } = copy; // noA => { b: 2, c: 3 }
```



## Arrays

- `e` 4.1 用字面量赋值。 eslint: [`no-array-constructor`](http://eslint.org/docs/rules/no-array-constructor.html)

```js
// bad
const items = new Array();

// good
const items = [];
```



- `p`  4.2 用[Array#push](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/push) 代替直接向数组中添加一个值。

解释：更具语义化。

```js
const someStack = [];

// bad
someStack[someStack.length] = 'abracadabra';

// good
someStack.push('abracadabra');
```

- `p`  4.3  用扩展运算符做数组浅拷贝，类似上面的对象浅拷贝

```js
// bad
const len = items.length;
const itemsCopy = [];
let i;

for (i = 0; i < len; i += 1) {
	itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```



- `p`  4.4 用 `...` 运算符而不是[`Array.from`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/from)来将一个可迭代的对象转换成数组。

```js
const foo = document.querySelectorAll('.foo');

// good
const nodes = Array.from(foo);

// best
const nodes = [...foo];
```



- `p`  4.5 用 [`Array.from`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/from) 去将一个类数组对象转成一个数组。

```js
const arrLike = { 0: 'foo', 1: 'bar', 2: 'baz', length: 3 };

// bad
const arr = Array.prototype.slice.call(arrLike);

// good
const arr = Array.from(arrLike);
```



- `p`  4.6 用 [`Array.from`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/from) 而不是 `...` 运算符去做map遍历。 因为这样可以避免创建一个临时数组。

```js
// bad
const baz = [...foo].map(bar);

// good
const baz = Array.from(foo, bar);
```



- `e` 4.7 在数组方法的回调函数中使用 return 语句。 如果函数体由一条返回一个表达式的语句组成， 并且这个表达式没有副作用， 这个时候可以忽略return，详见 [7.2](https://github.com/lin-123/javascript#arrows--implicit-return). eslint: [`array-callback-return`](http://eslint.org/docs/rules/array-callback-return)

```js
// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});

// good 函数只有一个语句
[1, 2, 3].map(x => x + 1);

// bad - 没有返回值， 因为在第一次迭代后acc 就变成undefined了
[[0, 1], [2, 3], [4, 5]].reduce((acc, item, index) => {
  const flatten = acc.concat(item);
  acc[index] = flatten;
});

// good
[[0, 1], [2, 3], [4, 5]].reduce((acc, item, index) => {
  const flatten = acc.concat(item);
  acc[index] = flatten;
  return flatten;
});

// bad
inbox.filter((msg) => {
  const { subject, author } = msg;
  if (subject === 'Mockingbird') {
    return author === 'Harper Lee';
  } else {
    return false;
  }
});

// good
inbox.filter((msg) => {
  const { subject, author } = msg;
  if (subject === 'Mockingbird') {
  	return author === 'Harper Lee';
  }

	return false;
});
```



- `u`  4.8 如果一个数组有很多行，在数组的 `[` 后和 `]` 前断行。 请看下面示例

```js
// bad
const arr = [
  [0, 1], [2, 3], [4, 5],
];

const objectInArray = [{
  id: 1,
}, {
  id: 2,
}];

const numberInArray = [
  1, 2,
];

// good
const arr = [[0, 1], [2, 3], [4, 5]];

const objectInArray = [
  {
    id: 1,
  },
  {
    id: 2,
  },
];

const numberInArray = [
  1,
  2,
];
```



## Destructuring


- `e` 5.1 用对象的解构赋值来获取和使用对象某个或多个属性值。 eslint: [`prefer-destructuring`](https://eslint.org/docs/rules/prefer-destructuring)

> Why? 解构保存了这些属性的临时值/引用

```js
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;

  return `${firstName} ${lastName}`;
}

// good
function getFullName(user) {
  const { firstName, lastName } = user;
  return `${firstName} ${lastName}`;
}

// best
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}
```



- `p` 5.2 用数组解构.

```js
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```



- `p` 5.3 多个返回值用对象的解构，而不是数据解构。

> Why? 你可以在后期添加新的属性或者变换变量的顺序而不会打破原有的调用

```js
  // bad
  function processInput(input) {
    // 然后就是见证奇迹的时刻
    return [left, right, top, bottom];
  }
  
  // 调用者需要想一想返回值的顺序
  const [left, __, top] = processInput(input);
  
  // good
  function processInput(input) {
    // oops， 奇迹又发生了
    return { left, right, top, bottom };
  }
  
  // 调用者只需要选择他想用的值就好了
  const { left, top } = processInput(input);
```



## Strings



- `e` 6.1 对string用单引号 `''` 。 eslint: [`quotes`](https://eslint.org/docs/rules/quotes.html)

  ```js
  // bad
  const name = "Capt. Janeway";
  
  // bad - 样例应该包含插入文字或换行
  const name = `Capt. Janeway`;
  
  // good
  const name = 'Capt. Janeway';
  ```



- `u` 6.2 超过100个字符的字符串不应该用string串联成多行。

  > Why? 被折断的字符串工作起来是糟糕的而且使得代码更不易被搜索。

  ```js
  // bad
  const errorMessage = 'This is a super long error that was thrown because \
  of Batman. When you stop to think about how Batman had anything to do \
  with this, you would get nowhere \
  fast.';
  
  // bad
  const errorMessage = 'This is a super long error that was thrown because ' +
    'of Batman. When you stop to think about how Batman had anything to do ' +
    'with this, you would get nowhere fast.';
  
  // good
  const errorMessage = 'This is a super long error that was thrown because of Batman. When you stop to think about how Batman had anything to do with this, you would get nowhere fast.';
  ```



- `e` 6.3 用字符串模板而不是字符串拼接来组织可编程字符串。 eslint: [`prefer-template`](https://eslint.org/docs/rules/prefer-template.html) [`template-curly-spacing`](https://eslint.org/docs/rules/template-curly-spacing)

  > Why? 模板字符串更具可读性、语法简洁、字符串插入参数。

  ```js
  // bad
  function sayHi(name) {
    return 'How are you, ' + name + '?';
  }
  
  // bad
  function sayHi(name) {
    return ['How are you, ', name, '?'].join();
  }
  
  // bad
  function sayHi(name) {
    return `How are you, ${ name }?`;
  }
  
  // good
  function sayHi(name) {
    return `How are you, ${name}?`;
  }
  ```



- `e` 6.4 永远不要在字符串中用`eval()`，他就是潘多拉盒子。 eslint: [`no-eval`](https://eslint.org/docs/rules/no-eval)



- `e` 6.5 不要使用不必要的转义字符。eslint: [`no-useless-escape`](http://eslint.org/docs/rules/no-useless-escape)

  > Why? 反斜线可读性差，所以他们只在必须使用时才出现哦

  ```js
  // bad
  const foo = '\'this\' \i\s \"quoted\"';
  
  // good
  const foo = '\'this\' is "quoted"';
  
  //best
  const foo = `my name is '${name}'`;
  ```

## Functions



- [7.1](https://github.com/lin-123/javascript#functions--declarations) 用命名函数表达式而不是函数声明。eslint: [`func-style`](http://eslint.org/docs/rules/func-style)

  > 函数表达式： const func = function () {}

  > 函数声明： function func() {}

  > Why? 函数声明时作用域被提前了，这意味着在一个文件里函数很容易（太容易了）在其定义之前被引用。这样伤害了代码可读性和可维护性。如果你发现一个函数又大又复杂，这个函数妨碍这个文件其他部分的理解性，这可能就是时候把这个函数单独抽成一个模块了。别忘了给表达式显示的命名，不用管这个名字是不是由一个确定的变量推断出来的，这消除了由匿名函数在错误调用栈产生的所有假设，这在现代浏览器和类似babel编译器中很常见 ([Discussion](https://github.com/airbnb/javascript/issues/794))

  > Why? 这一段还不理解这种错误发生的场景，所以只能直译过来了， 另附[原文](https://github.com/airbnb/javascript#functions--declarations) Why? Function declarations are hoisted, which means that it’s easy - too easy - to reference the function before it is defined in the file. This harms readability and maintainability. If you find that a function’s definition is large or complex enough that it is interfering with understanding the rest of the file, then perhaps it’s time to extract it to its own module! Don’t forget to explicitly name the expression, regardless of whether or not the name is inferred from the containing variable (which is often the case in modern browsers or when using compilers such as Babel). This eliminates any assumptions made about the Error’s call stack. ([Discussion](https://github.com/airbnb/javascript/issues/794))

  ```
  // bad
  function foo() {
    // ...
  }
  
  // bad
  const foo = function () {
    // ...
  };
  
  // good
  // lexical name distinguished from the variable-referenced invocation(s)
  // 函数表达式名和声明的函数名是不一样的
  const short = function longUniqueMoreDescriptiveLexicalFoo() {
    // ...
  };
  ```



- [7.2](https://github.com/lin-123/javascript#functions--iife) 把立即执行函数包裹在圆括号里。 eslint: [`wrap-iife`](http://eslint.org/docs/rules/wrap-iife.html)

  > Why? immediately invoked function expression = IIFE Why? 一个立即调用的函数表达式是一个单元 - 把它和他的调用者（圆括号）包裹起来，在括号中可以清晰的地表达这些。 Why? 注意：在模块化世界里，你几乎用不着 IIFE

  ```
  // immediately-invoked function expression (IIFE)
  (function () {
    console.log('Welcome to the Internet. Please follow me.');
  }());
  ```



- [7.3](https://github.com/lin-123/javascript#functions--in-blocks) 不要在非函数块（if、while等等）内声明函数。把这个函数分配给一个变量。浏览器会允许你这样做，但浏览器解析方式不同，这是一个坏消息。【详见`no-loop-func`】 eslint: [`no-loop-func`](http://eslint.org/docs/rules/no-loop-func.html)



- [7.4](https://github.com/lin-123/javascript#functions--note-on-blocks) **Note:** 在ECMA-262中 [块 `block`] 的定义是： 一系列的语句； 但是函数声明不是一个语句。 函数表达式是一个语句。

  ```
  // bad
  if (currentUser) {
    function test() {
      console.log('Nope.');
    }
  }
  
  // good
  let test;
  if (currentUser) {
    test = () => {
      console.log('Yup.');
    };
  }
  ```



- [7.5](https://github.com/lin-123/javascript#functions--arguments-shadow) 不要用`arguments`命名参数。他的优先级高于每个函数作用域自带的 `arguments` 对象， 这会导致函数自带的 `arguments` 值被覆盖

  ```
  // bad
  function foo(name, options, arguments) {
    // ...
  }
  
  // good
  function foo(name, options, args) {
    // ...
  }
  ```



- [7.6](https://github.com/lin-123/javascript#es6-rest) 不要使用`arguments`，用rest语法`...`代替。 eslint: [`prefer-rest-params`](http://eslint.org/docs/rules/prefer-rest-params)

  > Why? `...`明确你想用那个参数。而且rest参数是真数组，而不是类似数组的`arguments`

  ```
  // bad
  function concatenateAll() {
    const args = Array.prototype.slice.call(arguments);
    return args.join('');
  }
  
  // good
  function concatenateAll(...args) {
    return args.join('');
  }
  ```



- [7.7](https://github.com/lin-123/javascript#es6-default-parameters) 用默认参数语法而不是在函数里对参数重新赋值。

  ```
  // really bad
  function handleThings(opts) {
    // 不， 我们不该改arguments
    // 第二： 如果 opts 的值为 false, 它会被赋值为 {}
    // 虽然你想这么写， 但是这个会带来一些细微的bug
    opts = opts || {};
    // ...
  }
  
  // still bad
  function handleThings(opts) {
    if (opts === void 0) {
      opts = {};
    }
    // ...
  }
  
  // good
  function handleThings(opts = {}) {
    // ...
  }
  ```



- [7.8](https://github.com/lin-123/javascript#functions--default-side-effects) 默认参数避免副作用

  > Why? 他会令人迷惑不解， 比如下面这个， a到底等于几， 这个需要想一下。

  ```
  var b = 1;
  // bad
  function count(a = b++) {
    console.log(a);
  }
  count();  // 1
  count();  // 2
  count(3); // 3
  count();  // 3
  ```



- [7.9](https://github.com/lin-123/javascript#functions--defaults-last) 把默认参数赋值放在最后

  ```
  // bad
  function handleThings(opts = {}, name) {
    // ...
  }
  
  // good
  function handleThings(name, opts = {}) {
    // ...
  }
  ```



- [7.10](https://github.com/lin-123/javascript#functions--constructor) 不要用函数构造器创建函数。 eslint: [`no-new-func`](http://eslint.org/docs/rules/no-new-func)

  > Why? 以这种方式创建函数将类似于字符串 eval()，这会打开漏洞。

  ```
  // bad
  var add = new Function('a', 'b', 'return a + b');
  
  // still bad
  var subtract = Function('a', 'b', 'return a - b');
  ```



- [7.11](https://github.com/lin-123/javascript#functions--signature-spacing) 函数签名部分要有空格。eslint: [`space-before-function-paren`](http://eslint.org/docs/rules/space-before-function-paren) [`space-before-blocks`](http://eslint.org/docs/rules/space-before-blocks)

  > Why? 统一性好，而且在你添加/删除一个名字的时候不需要添加/删除空格

  ```
  // bad
  const f = function(){};
  const g = function (){};
  const h = function() {};
  
  // good
  const x = function () {};
  const y = function a() {};
  ```



- [7.12](https://github.com/lin-123/javascript#functions--mutate-params) 不要改参数. eslint: [`no-param-reassign`](http://eslint.org/docs/rules/no-param-reassign.html)

  > Why? 操作参数对象对原始调用者会导致意想不到的副作用。 就是不要改参数的数据结构，保留参数原始值和数据结构。

  ```
  // bad
  function f1(obj) {
    obj.key = 1;
  };
  
  // good
  function f2(obj) {
    const key = Object.prototype.hasOwnProperty.call(obj, 'key') ? obj.key : 1;
  };
  ```



- [7.13](https://github.com/lin-123/javascript#functions--reassign-params) 不要对参数重新赋值。 eslint: [`no-param-reassign`](http://eslint.org/docs/rules/no-param-reassign.html)

  > Why? 参数重新赋值会导致意外行为，尤其是对 `arguments`。这也会导致优化问题，特别是在V8里

  ```
  // bad
  function f1(a) {
    a = 1;
    // ...
  }
  
  function f2(a) {
    if (!a) { a = 1; }
    // ...
  }
  
  // good
  function f3(a) {
    const b = a || 1;
    // ...
  }
  
  function f4(a = 1) {
    // ...
  }
  ```



- [7.14](https://github.com/lin-123/javascript#functions--spread-vs-apply) 用`spread`操作符`...`去调用多变的函数更好。 eslint: [`prefer-spread`](http://eslint.org/docs/rules/prefer-spread)

  > Why? 这样更清晰，你不必提供上下文，而且你不能轻易地用`apply`来组成`new`

  ```
  // bad
  const x = [1, 2, 3, 4, 5];
  console.log.apply(console, x);
  
  // good
  const x = [1, 2, 3, 4, 5];
  console.log(...x);
  
  // bad
  new (Function.prototype.bind.apply(Date, [null, 2016, 8, 5]));
  
  // good
  new Date(...[2016, 8, 5]);
  ```



- [7.15](https://github.com/lin-123/javascript#functions--signature-invocation-indentation) 调用或者书写一个包含多个参数的函数应该像这个指南里的其他多行代码写法一样： 每行值包含一个参数，每行逗号结尾。

  ```
  // bad
  function foo(bar,
               baz,
               quux) {
    // ...
  }
  
  // good 缩进不要太过分
  function foo(
    bar,
    baz,
    quux,
  ) {
    // ...
  }
  
  // bad
  console.log(foo,
    bar,
    baz);
  
  // good
  console.log(
    foo,
    bar,
    baz,
  );
  ```

**⬆ back to top**

## Arrow Functions



- [8.1](https://github.com/lin-123/javascript#arrows--use-them) 当你一定要用函数表达式（在回调函数里）的时候就用箭头表达式吧。 eslint: [`prefer-arrow-callback`](http://eslint.org/docs/rules/prefer-arrow-callback.html), [`arrow-spacing`](http://eslint.org/docs/rules/arrow-spacing.html)

  > Why? 他创建了一个`this`的当前执行上下文的函数的版本，这通常就是你想要的；而且箭头函数是更简洁的语法

  > Why? 什么时候不用箭头函数： 如果你有一个相当复杂的函数，你可能会把这个逻辑移出到他自己的函数声明里。

  ```
  // bad
  [1, 2, 3].map(function (x) {
    const y = x + 1;
    return x * y;
  });
  
  // good
  [1, 2, 3].map((x) => {
    const y = x + 1;
    return x * y;
  });
  ```



- [8.2](https://github.com/lin-123/javascript#arrows--implicit-return) 如果函数体由一个没有副作用的[表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Expressions)语句组成，删除大括号和return。否则，继续用大括号和 `return` 语句。 eslint: [`arrow-parens`](https://eslint.org/docs/rules/arrow-parens.html), [`arrow-body-style`](https://eslint.org/docs/rules/arrow-body-style.html)

  > Why? 语法糖，当多个函数链在一起的时候好读

  ```
  // bad
  [1, 2, 3].map(number => {
    const nextNumber = number + 1;
    `A string containing the ${nextNumber}.`;
  });
  
  // good
  [1, 2, 3].map(number => `A string containing the ${number}.`);
  
  // good
  [1, 2, 3].map((number) => {
    const nextNumber = number + 1;
    return `A string containing the ${nextNumber}.`;
  });
  
  // good
  [1, 2, 3].map((number, index) => ({
    [index]: number
  }));
  
  // 表达式有副作用就不要用隐式return
  function foo(callback) {
    const val = callback();
    if (val === true) {
      // Do something if callback returns true
    }
  }
  
  let bool = false;
  
  // bad
  // 这种情况会return bool = true, 不好
  foo(() => bool = true);
  
  // good
  foo(() => {
    bool = true;
  });
  ```



- [8.3](https://github.com/lin-123/javascript#arrows--paren-wrap) 万一表达式涉及多行，把他包裹在圆括号里更可读。

  > Why? 这样清晰的显示函数的开始和结束

  ```
  // bad
  ['get', 'post', 'put'].map(httpMethod => Object.prototype.hasOwnProperty.call(
      httpMagicObjectWithAVeryLongName,
      httpMethod
    )
  );
  
  // good
  ['get', 'post', 'put'].map(httpMethod => (
    Object.prototype.hasOwnProperty.call(
      httpMagicObjectWithAVeryLongName,
      httpMethod
    )
  ));
  ```



- [8.4](https://github.com/lin-123/javascript#arrows--one-arg-parens) 如果你的函数只有一个参数并且函数体没有大括号，就删除圆括号。否则，参数总是放在圆括号里。 注意： 一直用圆括号也是没问题，只需要配置 [“always” option](https://eslint.org/docs/rules/arrow-parens#always) for eslint. eslint: [`arrow-parens`](https://eslint.org/docs/rules/arrow-parens.html)

  > Why? 这样少一些混乱， 其实没啥语法上的讲究，就保持一个风格。

  ```
  // bad
  [1, 2, 3].map((x) => x * x);
  
  // good
  [1, 2, 3].map(x => x * x);
  
  // good
  [1, 2, 3].map(number => (
    `A long string with the ${number}. It’s so long that we don’t want it to take up space on the .map line!`
  ));
  
  // bad
  [1, 2, 3].map(x => {
    const y = x + 1;
    return x * y;
  });
  
  // good
  [1, 2, 3].map((x) => {
    const y = x + 1;
    return x * y;
  });
  ```



- [8.5](https://github.com/lin-123/javascript#arrows--confusing) 避免箭头函数(`=>`)和比较操作符（`<=, >=`）混淆. eslint: [`no-confusing-arrow`](http://eslint.org/docs/rules/no-confusing-arrow)

  ```
  // bad
  const itemHeight = (item) => item.height <= 256 ? item.largeSize : item.smallSize;
  
  // bad
  const itemHeight = (item) => item.height >= 256 ? item.largeSize : item.smallSize;
  
  // good
  const itemHeight = (item) => (item.height <= 256 ? item.largeSize : item.smallSize);
  
  // good
  const itemHeight = (item) => {
    const { height, largeSize, smallSize } = item;
    return height <= 256 ? largeSize : smallSize;
  };
  ```



- [8.6](https://github.com/lin-123/javascript#whitespace--implicit-arrow-linebreak) 在隐式return中强制约束函数体的位置， 就写在箭头后面。 eslint: [`implicit-arrow-linebreak`](https://eslint.org/docs/rules/implicit-arrow-linebreak)

  ```
  // bad
  (foo) =>
    bar;
  
  (foo) =>
    (bar);
  
  // good
  (foo) => bar;
  (foo) => (bar);
  (foo) => (
     bar
  )
  ```

**⬆ back to top**

## Classes & Constructors



- [9.1](https://github.com/lin-123/javascript#constructors--use-class) 常用`class`，避免直接操作`prototype`

  > Why? `class`语法更简洁更易理解

  ```
  // bad
  function Queue(contents = []) {
    this.queue = [...contents];
  }
  Queue.prototype.pop = function () {
    const value = this.queue[0];
    this.queue.splice(0, 1);
    return value;
  };
  
  
  // good
  class Queue {
    constructor(contents = []) {
      this.queue = [...contents];
    }
    pop() {
      const value = this.queue[0];
      this.queue.splice(0, 1);
      return value;
    }
  }
  ```



- [9.2](https://github.com/lin-123/javascript#constructors--extends) 用`extends`实现继承

  > Why? 它是一种内置的方法来继承原型功能而不打破`instanceof`

  ```
  // bad
  const inherits = require('inherits');
  function PeekableQueue(contents) {
    Queue.apply(this, contents);
  }
  inherits(PeekableQueue, Queue);
  PeekableQueue.prototype.peek = function () {
    return this.queue[0];
  }
  
  // good
  class PeekableQueue extends Queue {
    peek() {
      return this.queue[0];
    }
  }
  ```



- [9.3](https://github.com/lin-123/javascript#constructors--chaining) 方法可以返回`this`来实现方法链

  ```
  // bad
  Jedi.prototype.jump = function () {
    this.jumping = true;
    return true;
  };
  
  Jedi.prototype.setHeight = function (height) {
    this.height = height;
  };
  
  const luke = new Jedi();
  luke.jump(); // => true
  luke.setHeight(20); // => undefined
  
  // good
  class Jedi {
    jump() {
      this.jumping = true;
      return this;
    }
  
    setHeight(height) {
      this.height = height;
      return this;
    }
  }
  
  const luke = new Jedi();
  
  luke.jump()
    .setHeight(20);
  ```



- [9.4](https://github.com/lin-123/javascript#constructors--tostring) 写一个定制的toString()方法是可以的，只要保证它是可以正常工作且没有副作用的

  ```
  class Jedi {
    constructor(options = {}) {
      this.name = options.name || 'no name';
    }
  
    getName() {
      return this.name;
    }
  
    toString() {
      return `Jedi - ${this.getName()}`;
    }
  }
  ```



- [9.5](https://github.com/lin-123/javascript#constructors--no-useless) 如果没有具体说明，类有默认的构造方法。一个空的构造函数或只是代表父类的构造函数是不需要写的。 eslint: [`no-useless-constructor`](http://eslint.org/docs/rules/no-useless-constructor)

  ```
  // bad
  class Jedi {
    constructor() {}
  
    getName() {
      return this.name;
    }
  }
  
  // bad
  class Rey extends Jedi {
    // 这种构造函数是不需要写的
    constructor(...args) {
      super(...args);
    }
  }
  
  // good
  class Rey extends Jedi {
    constructor(...args) {
      super(...args);
      this.name = 'Rey';
    }
  }
  ```



- [9.6](https://github.com/lin-123/javascript#classes--no-duplicate-members) 避免重复类成员。 eslint: [`no-dupe-class-members`](http://eslint.org/docs/rules/no-dupe-class-members)

  > Why? 重复类成员会默默的执行最后一个 —— 重复本身也是一个bug

  ```
  // bad
  class Foo {
    bar() { return 1; }
    bar() { return 2; }
  }
  
  // good
  class Foo {
    bar() { return 1; }
  }
  
  // good
  class Foo {
    bar() { return 2; }
  }
  ```



- [9.7](https://github.com/lin-123/javascript#classes--methods-use-this) 除非外部库或框架需要使用特定的非静态方法，否则类方法应该使用`this`或被做成静态方法。 作为一个实例方法应该表明它根据接收者的属性有不同的行为。eslint: [`class-methods-use-this`](https://eslint.org/docs/rules/class-methods-use-this)

  ```
  // bad
  class Foo {
    bar() {
      console.log('bar');
    }
  }
  
  // good - this 被使用了 
  class Foo {
    bar() {
      console.log(this.bar);
    }
  }
  
  // good - constructor 不一定要使用this
  class Foo {
    constructor() {
      // ...
    }
  }
  
  // good - 静态方法不需要使用 this
  class Foo {
    static bar() {
      console.log('bar');
    }
  }
  ```

**⬆ back to top**

## Modules



- [10.1](https://github.com/lin-123/javascript#modules--use-them) 用(`import`/`export`) 模块而不是无标准的模块系统。你可以随时转到你喜欢的模块系统。

  > Why? 模块化是未来，让我们现在就开启未来吧。

  ```
  // bad
  const AirbnbStyleGuide = require('./AirbnbStyleGuide');
  module.exports = AirbnbStyleGuide.es6;
  
  // ok
  import AirbnbStyleGuide from './AirbnbStyleGuide';
  export default AirbnbStyleGuide.es6;
  
  // best
  import { es6 } from './AirbnbStyleGuide';
  export default es6;
  ```



- [10.2](https://github.com/lin-123/javascript#modules--no-wildcard) 不要用import通配符， 就是 `*` 这种方式

  > Why? 这确保你有单个默认的导出

  ```
  // bad
  import * as AirbnbStyleGuide from './AirbnbStyleGuide';
  
  // good
  import AirbnbStyleGuide from './AirbnbStyleGuide';
  ```



- [10.3](https://github.com/lin-123/javascript#modules--no-export-from-import) 不要直接从import中直接export

  > Why? 虽然一行是简洁的，有一个明确的方式进口和一个明确的出口方式来保证一致性。

  ```
  // bad
  // filename es6.js
  export { es6 as default } from './AirbnbStyleGuide';
  
  // good
  // filename es6.js
  import { es6 } from './AirbnbStyleGuide';
  export default es6;
  ```



- [10.4](https://github.com/lin-123/javascript#modules--no-duplicate-imports) 一个路径只 import 一次。 eslint: [`no-duplicate-imports`](http://eslint.org/docs/rules/no-duplicate-imports)

  > Why? 从同一个路径下import多行会使代码难以维护

  ```
  // bad
  import foo from 'foo';
  // … some other imports … //
  import { named1, named2 } from 'foo';
  
  // good
  import foo, { named1, named2 } from 'foo';
  
  // good
  import foo, {
    named1,
    named2,
  } from 'foo';
  ```



- [10.5](https://github.com/lin-123/javascript#modules--no-mutable-exports) 不要导出可变的东西 eslint: [`import/no-mutable-exports`](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/no-mutable-exports.md)

  > Why? 变化通常都是需要避免，特别是当你要输出可变的绑定。虽然在某些场景下可能需要这种技术，但总的来说应该导出常量。

  ```
  // bad
  let foo = 3;
  export { foo }
  
  // good
  const foo = 3;
  export { foo }
  ```



- [10.6](https://github.com/lin-123/javascript#modules--prefer-default-export) 在一个单一导出模块里，用 `export default` 更好。 eslint: [`import/prefer-default-export`](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/prefer-default-export.md)

  > Why? 鼓励使用更多文件，每个文件只做一件事情并导出，这样可读性和可维护性更好。

  ```
  // bad
  export function foo() {}
  
  // good
  export default function foo() {}
  ```



- [10.7](https://github.com/lin-123/javascript#modules--imports-first) `import` 放在其他所有语句之前。 eslint: [`import/first`](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/first.md)

  > Why? 让`import`放在最前面防止意外行为。

  ```
  // bad
  import foo from 'foo';
  foo.init();
  
  import bar from 'bar';
  
  // good
  import foo from 'foo';
  import bar from 'bar';
  
  foo.init();
  ```



- [10.8](https://github.com/lin-123/javascript#modules--multiline-imports-over-newlines) 多行import应该缩进，就像多行数组和对象字面量

  > Why? 花括号与样式指南中每个其他花括号块遵循相同的缩进规则，逗号也是。

  ```
  // bad
  import {longNameA, longNameB, longNameC, longNameD, longNameE} from 'path';
  
  // good
  import {
    longNameA,
    longNameB,
    longNameC,
    longNameD,
    longNameE,
  } from 'path';
  ```



- [10.9](https://github.com/lin-123/javascript#modules--no-webpack-loader-syntax) 在import语句里不允许Webpack loader语法 eslint: [`import/no-webpack-loader-syntax`](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/no-webpack-loader-syntax.md)

  > Why? 一旦用Webpack语法在import里会把代码耦合到模块绑定器。最好是在`webpack.config.js`里写webpack loader语法

  ```
  // bad
  import fooSass from 'css!sass!foo.scss';
  import barCss from 'style!css!bar.css';
  
  // good
  import fooSass from 'foo.scss';
  import barCss from 'bar.css';
  ```

**⬆ back to top**

## Iterators and Generators



- [11.1](https://github.com/lin-123/javascript#iterators--nope) 不要用遍历器。用JavaScript高级函数代替`for-in`、 `for-of`。 eslint: [`no-iterator`](http://eslint.org/docs/rules/no-iterator.html) [`no-restricted-syntax`](http://eslint.org/docs/rules/no-restricted-syntax)

  > Why? 这强调了我们不可变的规则。 处理返回值的纯函数比副作用更容易。

  > Why? 用数组的这些迭代方法： `map()` / `every()` / `filter()` / `find()` / `findIndex()` / `reduce()` / `some()` / ... , 用对象的这些方法 `Object.keys()` / `Object.values()` / `Object.entries()` 去产生一个数组， 这样你就能去遍历对象了。

  ```
  const numbers = [1, 2, 3, 4, 5];
  
  // bad
  let sum = 0;
  for (let num of numbers) {
    sum += num;
  }
  sum === 15;
  
  // good
  let sum = 0;
  numbers.forEach(num => sum += num);
  sum === 15;
  
  // best (use the functional force)
  const sum = numbers.reduce((total, num) => total + num, 0);
  sum === 15;
  
  // bad
  const increasedByOne = [];
  for (let i = 0; i < numbers.length; i++) {
    increasedByOne.push(numbers[i] + 1);
  }
  
  // good
  const increasedByOne = [];
  numbers.forEach(num => increasedByOne.push(num + 1));
  
  // best (keeping it functional)
  const increasedByOne = numbers.map(num => num + 1);
  ```



- [11.2](https://github.com/lin-123/javascript#generators--nope) 现在不要用generator

  > Why? 它在es5上支持的不好



- [11.3](https://github.com/lin-123/javascript#generators--spacing) 如果你一定要用，或者你忽略[我们的建议](https://github.com/lin-123/javascript#generators--nope), 请确保它们的函数签名空格是得当的。 eslint: [`generator-star-spacing`](http://eslint.org/docs/rules/generator-star-spacing)

  > Why? `function` 和 `*` 是同一概念关键字 - `*`不是`function`的修饰符，`function*`是一个和`function`不一样的独特结构

  ```
  // bad
  function * foo() {
    // ...
  }
  
  // bad
  const bar = function * () {
    // ...
  }
  
  // bad
  const baz = function *() {
    // ...
  }
  
  // bad
  const quux = function*() {
    // ...
  }
  
  // bad
  function*foo() {
    // ...
  }
  
  // bad
  function *foo() {
    // ...
  }
  
  // very bad
  function
  *
  foo() {
    // ...
  }
  
  // very bad
  const wat = function
  *
  () {
    // ...
  }
  
  // good
  function* foo() {
    // ...
  }
  
  // good
  const foo = function* () {
    // ...
  }
  ```

**⬆ back to top**

## Properties



- [12.1](https://github.com/lin-123/javascript#properties--dot) 访问属性时使用点符号. eslint: [`dot-notation`](http://eslint.org/docs/rules/dot-notation.html)

  ```
  const luke = {
    jedi: true,
    age: 28,
  };
  
  // bad
  const isJedi = luke['jedi'];
  
  // good
  const isJedi = luke.jedi;
  ```



- [12.2](https://github.com/lin-123/javascript#properties--bracket) 当获取的属性是变量时用方括号`[]`取

  ```
  const luke = {
    jedi: true,
    age: 28,
  };
  
  function getProp(prop) {
    return luke[prop];
  }
  
  const isJedi = getProp('jedi');
  ```



- [12.3](https://github.com/lin-123/javascript#es2016-properties--exponentiation-operator) 做幂运算时用幂操作符 `**` 。 eslint: [`no-restricted-properties`](https://eslint.org/docs/rules/no-restricted-properties).

  ```
  // bad
  const binary = Math.pow(2, 10);
  
  // good
  const binary = 2 ** 10;
  ```

**⬆ back to top**

## Variables



- [13.1](https://github.com/lin-123/javascript#variables--const) 用`const`或`let`声明变量。不这样做会导致全局变量。 我们想要避免污染全局命名空间。首长这样警告我们。 eslint: [`no-undef`](http://eslint.org/docs/rules/no-undef) [`prefer-const`](http://eslint.org/docs/rules/prefer-const)

  ```
  // bad
  superPower = new SuperPower();
  
  // good
  const superPower = new SuperPower();
  ```



- [13.2](https://github.com/lin-123/javascript#variables--one-const) 每个变量都用一个 `const` 或 `let `。 eslint: [`one-var`](http://eslint.org/docs/rules/one-var.html)

  > Why? 这种方式很容易去声明新的变量，你不用去考虑把`;`调换成`,`，或者引入一个只有标点的不同的变化。这种做法也可以是你在调试的时候单步每个声明语句，而不是一下跳过所有声明。

  ```
  // bad
  const items = getItems(),
      goSportsTeam = true,
      dragonball = 'z';
  
  // bad
  // (compare to above, and try to spot the mistake)
  const items = getItems(),
      goSportsTeam = true;
      dragonball = 'z';
  
  // good
  const items = getItems();
  const goSportsTeam = true;
  const dragonball = 'z';
  ```



- [13.3](https://github.com/lin-123/javascript#variables--const-let-group) `const`放一起，`let`放一起

  > Why? 在你需要分配一个新的变量， 而这个变量依赖之前分配过的变量的时候，这种做法是有帮助的

  ```
  // bad
  let i, len, dragonball,
      items = getItems(),
      goSportsTeam = true;
  
  // bad
  let i;
  const items = getItems();
  let dragonball;
  const goSportsTeam = true;
  let len;
  
  // good
  const goSportsTeam = true;
  const items = getItems();
  let dragonball;
  let i;
  let length;
  ```



- [13.4](https://github.com/lin-123/javascript#variables--define-where-used) 在你需要的地方声明变量，但是要放在合理的位置

  > Why? `let` 和 `const` 都是块级作用域而不是函数级作用域

  ```
  // bad - unnecessary function call
  function checkName(hasName) {
    const name = getName();
  
    if (hasName === 'test') {
      return false;
    }
  
    if (name === 'test') {
      this.setName('');
      return false;
    }
  
    return name;
  }
  
  // good
  function checkName(hasName) {
    if (hasName === 'test') {
      return false;
    }
  
    // 在需要的时候分配
    const name = getName();
  
    if (name === 'test') {
      this.setName('');
      return false;
    }
  
    return name;
  }
  ```



- [13.5](https://github.com/lin-123/javascript#variables--no-chain-assignment) 不要使用链接变量分配。 eslint: [`no-multi-assign`](https://eslint.org/docs/rules/no-multi-assign)

  > Why? 链接变量分配创建隐式全局变量。

  ```
  // bad
  (function example() {
    // JavaScript 将这一段解释为
    // let a = ( b = ( c = 1 ) );
    // let 只对变量 a 起作用; 变量 b 和 c 都变成了全局变量
    let a = b = c = 1;
  }());
  
  console.log(a); // undefined
  console.log(b); // 1
  console.log(c); // 1
  
  // good
  (function example() {
    let a = 1;
    let b = a;
    let c = a;
  }());
  
  console.log(a); // undefined
  console.log(b); // undefined
  console.log(c); // undefined
  
  // `const` 也是如此
  ```



- [13.6](https://github.com/lin-123/javascript#variables--unary-increment-decrement) 不要使用一元自增自减运算符（`++`， `--`）. eslint [`no-plusplus`](http://eslint.org/docs/rules/no-plusplus)

  > Why? 根据eslint文档，一元增量和减量语句受到自动分号插入的影响，并且可能会导致应用程序中的值递增或递减的无声错误。 使用`num + = 1`而不是`num ++`或`num ++`语句来表达你的值也是更有表现力的。 禁止一元增量和减量语句还会阻止您无意地预增/预减值，这也会导致程序出现意外行为。

  ```
    // bad
  
    const array = [1, 2, 3];
    let num = 1;
    num++;
    --num;
  
    let sum = 0;
    let truthyCount = 0;
    for (let i = 0; i < array.length; i++) {
      let value = array[i];
      sum += value;
      if (value) {
        truthyCount++;
      }
    }
  
    // good
  
    const array = [1, 2, 3];
    let num = 1;
    num += 1;
    num -= 1;
  
    const sum = array.reduce((a, b) => a + b, 0);
    const truthyCount = array.filter(Boolean).length;
  ```



- [13.7](https://github.com/lin-123/javascript#variables--linebreak) 在赋值的时候避免在 `=` 前/后换行。 如果你的赋值语句超出 [`max-len`](https://eslint.org/docs/rules/max-len.html)， 那就用小括号把这个值包起来再换行。 eslint [`operator-linebreak`](https://eslint.org/docs/rules/operator-linebreak.html).

  > Why? 在 `=` 附近换行容易混淆这个赋值语句。

  ```
  // bad
  const foo =
    superLongLongLongLongLongLongLongLongFunctionName();
  
  // bad
  const foo
    = 'superLongLongLongLongLongLongLongLongString';
  
  // good
  const foo = (
    superLongLongLongLongLongLongLongLongFunctionName()
  );
  
  // good
  const foo = 'superLongLongLongLongLongLongLongLongString';
  ```



- [13.8](https://github.com/lin-123/javascript#variables--no-unused-vars) 不允许有未使用的变量。 eslint: [`no-unused-vars`](https://eslint.org/docs/rules/no-unused-vars)

  > Why? 一个声明了但未使用的变量更像是由于重构未完成产生的错误。这种在代码中出现的变量会使阅读者迷惑。

  ```
  // bad
  
  var some_unused_var = 42;
  
  // 写了没用
  var y = 10;
  y = 5;
  
  // 变量改了自己的值，也没有用这个变量
  var z = 0;
  z = z + 1;
  
  // 参数定义了但未使用
  function getX(x, y) {
      return x;
  }
  
  // good
  function getXPlusY(x, y) {
    return x + y;
  }
  
  var x = 1;
  var y = a + 2;
  
  alert(getXPlusY(x, y));
  
  // 'type' 即使没有使用也可以可以被忽略， 因为这个有一个 rest 取值的属性。
  // 这是从对象中抽取一个忽略特殊字段的对象的一种形式
  var { type, ...coords } = data;
  // 'coords' 现在就是一个没有 'type' 属性的 'data' 对象
  ```

**⬆ back to top**

## Hoisting



- [14.1](https://github.com/lin-123/javascript#hoisting--about) `var`声明会被提前到他的作用域的最前面，它分配的值还没有提前。`const` 和 `let`被赋予了新的调用概念[时效区 —— Temporal Dead Zones (TDZ)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let)。 重要的是要知道为什么 [typeof不再安全](http://es-discourse.com/t/why-typeof-is-no-longer-safe/15).

  ```
  // 我们知道这个不会工作，假设没有定义全局的notDefined
  function example() {
    console.log(notDefined); // => throws a ReferenceError
  }
  
  // 在你引用的地方之后声明一个变量，他会正常输出是因为变量作用域上升。
  // 注意： declaredButNotAssigned的值没有上升
  function example() {
    console.log(declaredButNotAssigned); // => undefined
    var declaredButNotAssigned = true;
  }
  
  // 解释器把变量声明提升到作用域最前面，
  // 可以重写成如下例子， 二者意义相同
  function example() {
    let declaredButNotAssigned;
    console.log(declaredButNotAssigned); // => undefined
    declaredButNotAssigned = true;
  }
  
  // 用 const， let就不一样了
  function example() {
    console.log(declaredButNotAssigned); // => throws a ReferenceError
    console.log(typeof declaredButNotAssigned); // => throws a ReferenceError
    const declaredButNotAssigned = true;
  }
  ```



- [14.2](https://github.com/lin-123/javascript#hoisting--anon-expressions) 匿名函数表达式和 `var` 情况相同

  ```
  function example() {
    console.log(anonymous); // => undefined
  
    anonymous(); // => TypeError anonymous is not a function
  
    var anonymous = function () {
      console.log('anonymous function expression');
    };
  }
  ```



- [14.3](https://github.com/lin-123/javascript#hoisting--named-expresions) 已命名函数表达式提升他的变量名，不是函数名或函数体

  ```
  function example() {
    console.log(named); // => undefined
  
    named(); // => TypeError named is not a function
  
    superPower(); // => ReferenceError superPower is not defined
  
    var named = function superPower() {
      console.log('Flying');
    };
  }
  
  // 函数名和变量名一样是也如此
  function example() {
    console.log(named); // => undefined
  
    named(); // => TypeError named is not a function
  
    var named = function named() {
      console.log('named');
    };
  }
  ```



- [14.4](https://github.com/lin-123/javascript#hoisting--declarations) 函数声明则提升了函数名和函数体

  ```
  function example() {
    superPower(); // => Flying
  
    function superPower() {
      console.log('Flying');
    }
  }
  ```

- 详情请见[JavaScript Scoping & Hoisting](http://www.adequatelygood.com/2010/2/JavaScript-Scoping-and-Hoisting/) by [Ben Cherry](http://www.adequatelygood.com/).

**⬆ back to top**

## Comparison Operators & Equality



- [15.1](https://github.com/lin-123/javascript#comparison--eqeqeq) 用 `===` 和 `!==` 而不是 `==` 和 `!=`. eslint: [`eqeqeq`](http://eslint.org/docs/rules/eqeqeq.html)



- [15.2](https://github.com/lin-123/javascript#comparison--if) 条件语句如'if'语句使用强制`ToBoolean'抽象方法来评估它们的表达式，并且始终遵循以下简单规则：

  - **Objects** 计算成 **true**
  - **Undefined** 计算成 **false**
  - **Null** 计算成 **false**
  - **Booleans** 计算成 **the value of the boolean**
  - Numbers
    - **+0, -0, or NaN** 计算成 **false**
    - 其他 **true**
  - Strings
    - `''` 计算成 **false**
    - 其他 **true**

  ```
  if ([0] && []) {
    // true
    // 数组（即使是空数组）是对象，对象会计算成true
  }
  ```



- [15.3](https://github.com/lin-123/javascript#comparison--shortcuts) 布尔值用缩写，而字符串和数字要明确比较对象

  ```
  // bad
  if (isValid === true) {
    // ...
  }
  
  // good
  if (isValid) {
    // ...
  }
  
  // bad
  if (name) {
    // ...
  }
  
  // good
  if (name !== '') {
    // ...
  }
  
  // bad
  if (collection.length) {
    // ...
  }
  
  // good
  if (collection.length > 0) {
    // ...
  }
  ```



- [15.4](https://github.com/lin-123/javascript#comparison--moreinfo) 更多信息请见Angus Croll的[真理、平等和JavaScript —— Truth Equality and JavaScript](https://javascriptweblog.wordpress.com/2011/02/07/truth-equality-and-javascript/#more-2108)



- [15.5](https://github.com/lin-123/javascript#comparison--switch-blocks) 在`case`和`default`分句里用大括号创建一块包含语法声明的区域(e.g. `let`, `const`, `function`, and `class`). eslint rules: [`no-case-declarations`](http://eslint.org/docs/rules/no-case-declarations.html).

  > Why? 语法声明在整个`switch`的代码块里都可见，但是只有当其被分配后才会初始化，他的初始化时当这个`case`被执行时才产生。 当多个`case`分句试图定义同一个事情时就出问题了

  ```
  // bad
  switch (foo) {
    case 1:
      let x = 1;
      break;
    case 2:
      const y = 2;
      break;
    case 3:
      function f() {
        // ...
      }
      break;
    default:
      class C {}
  }
  
  // good
  switch (foo) {
    case 1: {
      let x = 1;
      break;
    }
    case 2: {
      const y = 2;
      break;
    }
    case 3: {
      function f() {
        // ...
      }
      break;
    }
    case 4:
      bar();
      break;
    default: {
      class C {}
    }
  }
  ```



- [15.6](https://github.com/lin-123/javascript#comparison--nested-ternaries) 三元表达式不应该嵌套，通常是单行表达式。

  eslint rules: [`no-nested-ternary`](http://eslint.org/docs/rules/no-nested-ternary.html).

  ```
  // bad
  const foo = maybe1 > maybe2
    ? "bar"
    : value1 > value2 ? "baz" : null;
  
  // better
  const maybeNull = value1 > value2 ? 'baz' : null;
  
  const foo = maybe1 > maybe2
    ? 'bar'
    : maybeNull;
  
  // best
  const maybeNull = value1 > value2 ? 'baz' : null;
  
  const foo = maybe1 > maybe2 ? 'bar' : maybeNull;
  ```



- [15.7](https://github.com/lin-123/javascript#comparison--unneeded-ternary) 避免不需要的三元表达式

  eslint rules: [`no-unneeded-ternary`](http://eslint.org/docs/rules/no-unneeded-ternary.html).

  ```
  // bad
  const foo = a ? a : b;
  const bar = c ? true : false;
  const baz = c ? false : true;
  
  // good
  const foo = a || b;
  const bar = !!c;
  const baz = !c;
  ```



- [15.8](https://github.com/lin-123/javascript#comparison--no-mixed-operators) 用圆括号来混合这些操作符。 只有当标准的算术运算符(`+`, `-`, `*`, & `/`)， 并且它们的优先级显而易见时，可以不用圆括号括起来。 eslint: [`no-mixed-operators`](https://eslint.org/docs/rules/no-mixed-operators.html)

  > Why? 这提高了可读性，并且明确了开发者的意图

  ```
  // bad
  const foo = a && b < 0 || c > 0 || d + 1 === 0;
  
  // bad
  const bar = a ** b - 5 % d;
  
  // bad
  // 别人会陷入(a || b) && c 的迷惑中
  if (a || b && c) {
    return d;
  }
  
  // good
  const foo = (a && b < 0) || c > 0 || (d + 1 === 0);
  
  // good
  const bar = (a ** b) - (5 % d);
  
  // good
  if (a || (b && c)) {
    return d;
  }
  
  // good
  const bar = a + b / c * d;
  ```

**⬆ back to top**

## Blocks



- [16.1](https://github.com/lin-123/javascript#blocks--braces) 用大括号包裹多行代码块。 eslint: [`nonblock-statement-body-position`](https://eslint.org/docs/rules/nonblock-statement-body-position)

  ```
  // bad
  if (test)
    return false;
  
  // good
  if (test) return false;
  
  // good
  if (test) {
    return false;
  }
  
  // bad
  function foo() { return false; }
  
  // good
  function bar() {
    return false;
  }
  ```



- [16.2](https://github.com/lin-123/javascript#blocks--cuddled-elses) `if`表达式的`else`和`if`的关闭大括号在一行。 eslint: [`brace-style`](http://eslint.org/docs/rules/brace-style.html)

  ```
  // bad
  if (test) {
    thing1();
    thing2();
  }
  else {
    thing3();
  }
  
  // good
  if (test) {
    thing1();
    thing2();
  } else {
    thing3();
  }
  ```



- [16.3](https://github.com/lin-123/javascript#blocks--no-else-return) 如果 `if` 语句中总是需要用 `return` 返回， 那后续的 `else` 就不需要写了。 `if` 块中包含 `return`， 它后面的 `else if` 块中也包含了 `return`， 这个时候就可以把 `return` 分到多个 `if` 语句块中。 eslint: [`no-else-return`](https://eslint.org/docs/rules/no-else-return)

  ```
  // bad
  function foo() {
    if (x) {
      return x;
    } else {
      return y;
    }
  }
  
  // bad
  function cats() {
    if (x) {
      return x;
    } else if (y) {
      return y;
    }
  }
  
  // bad
  function dogs() {
    if (x) {
      return x;
    } else {
      if (y) {
        return y;
      }
    }
  }
  
  // good
  function foo() {
    if (x) {
      return x;
    }
  
    return y;
  }
  
  // good
  function cats() {
    if (x) {
      return x;
    }
  
    if (y) {
      return y;
    }
  }
  
  // good
  function dogs(x) {
    if (x) {
      if (z) {
        return y;
      }
    } else {
      return z;
    }
  }
  ```

**⬆ back to top**

## Control Statements



- [17.1](https://github.com/lin-123/javascript#control-statements) 当你的控制语句(`if`, `while` 等)太长或者超过最大长度限制的时候， 把每一个(组)判断条件放在单独一行里。 逻辑操作符放在行首。

  > Why? 把逻辑操作符放在行首是让操作符的对齐方式和链式函数保持一致。这提高了可读性，也让复杂逻辑更容易看清楚。

  ```
  // bad
  if ((foo === 123 || bar === 'abc') && doesItLookGoodWhenItBecomesThatLong() && isThisReallyHappening()) {
    thing1();
  }
  
  // bad
  if (foo === 123 &&
    bar === 'abc') {
    thing1();
  }
  
  // bad
  if (foo === 123
    && bar === 'abc') {
    thing1();
  }
  
  // bad
  if (
    foo === 123 &&
    bar === 'abc'
  ) {
    thing1();
  }
  
  // good
  if (
    foo === 123
    && bar === 'abc'
  ) {
    thing1();
  }
  
  // good
  if (
    (foo === 123 || bar === 'abc')
    && doesItLookGoodWhenItBecomesThatLong()
    && isThisReallyHappening()
  ) {
    thing1();
  }
  
  // good
  if (foo === 123 && bar === 'abc') {
    thing1();
  }
  ```



- [17.2](https://github.com/lin-123/javascript#control-statements--value-selection) 不要用选择操作符代替控制语句。

  ```
  // bad
  !isRunning && startRunning();
  
  // good
  if (!isRunning) {
    startRunning();
  }
  ```

**⬆ back to top**

## Comments



- [18.1](https://github.com/lin-123/javascript#comments--multiline) 多行注释用 `/** ... */`

  ```
  // bad
  // make() returns a new element
  // based on the passed in tag name
  //
  // @param {String} tag
  // @return {Element} element
  function make(tag) {
  
    // ...
  
    return element;
  }
  
  // good
  /**
   * make() returns a new element
   * based on the passed-in tag name
   */
  function make(tag) {
  
    // ...
  
    return element;
  }
  ```



- [18.2](https://github.com/lin-123/javascript#comments--singleline) 单行注释用`//`，将单行注释放在被注释区域上面。如果注释不是在第一行，那么注释前面就空一行

  ```
  // bad
  const active = true;  // is current tab
  
  // good
  // is current tab
  const active = true;
  
  // bad
  function getType() {
    console.log('fetching type...');
    // set the default type to 'no type'
    const type = this._type || 'no type';
  
    return type;
  }
  
  // good
  function getType() {
    console.log('fetching type...');
  
    // set the default type to 'no type'
    const type = this._type || 'no type';
  
    return type;
  }
  
  // also good
  function getType() {
    // set the default type to 'no type'
    const type = this._type || 'no type';
  
    return type;
  }
  ```



- [18.3](https://github.com/lin-123/javascript#comments--spaces) 所有注释开头空一个，方便阅读。 eslint: [`spaced-comment`](http://eslint.org/docs/rules/spaced-comment)

  ```
  // bad
  //is current tab
  const active = true;
  
  // good
  // is current tab
  const active = true;
  
  // bad
  /**
   *make() returns a new element
   *based on the passed-in tag name
   */
  function make(tag) {
  
    // ...
  
    return element;
  }
  
  // good
  /**
   * make() returns a new element
   * based on the passed-in tag name
   */
  function make(tag) {
  
    // ...
  
    return element;
  }
  ```



- [18.4](https://github.com/lin-123/javascript#comments--actionitems) 在你的注释前使用`FIXME'或`TODO'前缀， 这有助于其他开发人员快速理解你指出的需要重新访问的问题， 或者您建议需要实现的问题的解决方案。 这些不同于常规注释，因为它们是可操作的。 动作是`FIXME： - 需要计算出来`或`TODO： - 需要实现`。



- [18.5](https://github.com/lin-123/javascript#comments--fixme) 用`// FIXME:`给问题做注释

  ```
  class Calculator extends Abacus {
    constructor() {
      super();
  
      // FIXME: shouldn't use a global here
      total = 0;
    }
  }
  ```



- [18.6](https://github.com/lin-123/javascript#comments--todo) 用`// TODO:`去注释问题的解决方案

  ```
  class Calculator extends Abacus {
    constructor() {
      super();
  
      // TODO: total should be configurable by an options param
      this.total = 0;
    }
  }
  ```

**⬆ back to top**

## Whitespace



- [19.1](https://github.com/lin-123/javascript#whitespace--spaces) tab用两个空格. eslint: [`indent`](http://eslint.org/docs/rules/indent.html)

  ```
  // bad
  function foo() {
  ∙∙∙∙const name;
  }
  
  // bad
  function bar() {
  ∙const name;
  }
  
  // good
  function baz() {
  ∙∙const name;
  }
  ```



- [19.2](https://github.com/lin-123/javascript#whitespace--before-blocks) 在大括号前空一格。 eslint: [`space-before-blocks`](http://eslint.org/docs/rules/space-before-blocks.html)

  ```
  // bad
  function test(){
    console.log('test');
  }
  
  // good
  function test() {
    console.log('test');
  }
  
  // bad
  dog.set('attr',{
    age: '1 year',
    breed: 'Bernese Mountain Dog',
  });
  
  // good
  dog.set('attr', {
    age: '1 year',
    breed: 'Bernese Mountain Dog',
  });
  ```



- [19.3](https://github.com/lin-123/javascript#whitespace--around-keywords) 在控制语句(`if`, `while` 等)的圆括号前空一格。在函数调用和定义时，参数列表和函数名之间不空格。 eslint: [`keyword-spacing`](http://eslint.org/docs/rules/keyword-spacing.html)

  ```
  // bad
  if(isJedi) {
    fight ();
  }
  
  // good
  if (isJedi) {
    fight();
  }
  
  // bad
  function fight () {
    console.log ('Swooosh!');
  }
  
  // good
  function fight() {
    console.log('Swooosh!');
  }
  ```



- [19.4](https://github.com/lin-123/javascript#whitespace--infix-ops) 用空格来隔开运算符。 eslint: [`space-infix-ops`](http://eslint.org/docs/rules/space-infix-ops.html)

  ```
  // bad
  const x=y+5;
  
  // good
  const x = y + 5;
  ```



- [19.5](https://github.com/lin-123/javascript#whitespace--newline-at-end) 文件结尾空一行. eslint: [`eol-last`](https://github.com/eslint/eslint/blob/master/docs/rules/eol-last.md)

  ```
  // bad
  import { es6 } from './AirbnbStyleGuide';
    // ...
  export default es6;
  ```

  ```
  // bad
  import { es6 } from './AirbnbStyleGuide';
    // ...
  export default es6;↵
  ↵
  ```

  ```
  // good
  import { es6 } from './AirbnbStyleGuide';
    // ...
  export default es6;↵
  ```



- [19.6](https://github.com/lin-123/javascript#whitespace--chains) 当出现长的方法链（>2个）时用缩进。用点开头强调该行是一个方法调用，而不是一个新的语句。eslint: [`newline-per-chained-call`](http://eslint.org/docs/rules/newline-per-chained-call) [`no-whitespace-before-property`](http://eslint.org/docs/rules/no-whitespace-before-property)

  ```
  // bad
  $('#items').find('.selected').highlight().end().find('.open').updateCount();
  
  // bad
  $('#items').
    find('.selected').
      highlight().
      end().
    find('.open').
      updateCount();
  
  // good
  $('#items')
    .find('.selected')
      .highlight()
      .end()
    .find('.open')
      .updateCount();
  
  // bad
  const leds = stage.selectAll('.led').data(data).enter().append('svg:svg').classed('led', true)
      .attr('width', (radius + margin) * 2).append('svg:g')
      .attr('transform', `translate(${radius + margin},${radius + margin})`)
      .call(tron.led);
  
  // good
  const leds = stage.selectAll('.led')
      .data(data)
    .enter().append('svg:svg')
      .classed('led', true)
      .attr('width', (radius + margin) * 2)
    .append('svg:g')
      .attr('transform', `translate(${radius + margin},${radius + margin})`)
      .call(tron.led);
  
  // good
  const leds = stage.selectAll('.led').data(data);
  ```



- [19.7](https://github.com/lin-123/javascript#whitespace--after-blocks) 在一个代码块后下一条语句前空一行。

  ```
  // bad
  if (foo) {
    return bar;
  }
  return baz;
  
  // good
  if (foo) {
    return bar;
  }
  
  return baz;
  
  // bad
  const obj = {
    foo() {
    },
    bar() {
    },
  };
  return obj;
  
  // good
  const obj = {
    foo() {
    },
  
    bar() {
    },
  };
  
  return obj;
  
  // bad
  const arr = [
    function foo() {
    },
    function bar() {
    },
  ];
  return arr;
  
  // good
  const arr = [
    function foo() {
    },
  
    function bar() {
    },
  ];
  
  return arr;
  ```



- [19.8](https://github.com/lin-123/javascript#whitespace--padded-blocks) 不要用空白行填充块。 eslint: [`padded-blocks`](http://eslint.org/docs/rules/padded-blocks.html)

  ```
  // bad
  function bar() {
  
    console.log(foo);
  
  }
  
  // also bad
  if (baz) {
  
    console.log(qux);
  } else {
    console.log(foo);
  
  }
  
  // good
  function bar() {
    console.log(foo);
  }
  
  // good
  if (baz) {
    console.log(qux);
  } else {
    console.log(foo);
  }
  ```



- [19.9](https://github.com/lin-123/javascript#whitespace--no-multiple-blanks)不要在代码之间使用多个空白行填充。 eslint: [`no-multiple-empty-lines`](https://eslint.org/docs/rules/no-multiple-empty-lines)

  ```
  // bad
  class Person {
    constructor(fullName, email, birthday) {
      this.fullName = fullName;
  
  
      this.email = email;
  
  
      this.setAge(birthday);
    }
  
  
    setAge(birthday) {
      const today = new Date();
  
  
      const age = this.getAge(today, birthday);
  
  
      this.age = age;
    }
  
  
    getAge(today, birthday) {
      // ..
    }
  }
  
  // good
  class Person {
    constructor(fullName, email, birthday) {
      this.fullName = fullName;
      this.email = email;
      this.setAge(birthday);
    }
  
    setAge(birthday) {
      const today = new Date();
      const age = getAge(today, birthday);
      this.age = age;
    }
  
    getAge(today, birthday) {
      // ..
    }
  }
  ```



- [19.10](https://github.com/lin-123/javascript#whitespace--in-parens) 圆括号里不要加空格。 eslint: [`space-in-parens`](http://eslint.org/docs/rules/space-in-parens.html)

  ```
  // bad
  function bar( foo ) {
    return foo;
  }
  
  // good
  function bar(foo) {
    return foo;
  }
  
  // bad
  if ( foo ) {
    console.log(foo);
  }
  
  // good
  if (foo) {
    console.log(foo);
  }
  ```



- [19.11](https://github.com/lin-123/javascript#whitespace--in-brackets) 方括号里不要加空格。看示例。 eslint: [`array-bracket-spacing`](http://eslint.org/docs/rules/array-bracket-spacing.html)

  ```
  // bad
  const foo = [ 1, 2, 3 ];
  console.log(foo[ 0 ]);
  
  // good， 逗号分隔符还是要空格的
  const foo = [1, 2, 3];
  console.log(foo[0]);
  ```



- [19.12](https://github.com/lin-123/javascript#whitespace--in-braces) 花括号里加空格。 eslint: [`object-curly-spacing`](http://eslint.org/docs/rules/object-curly-spacing.html)

  ```
  // bad
  const foo = {clark: 'kent'};
  
  // good
  const foo = { clark: 'kent' };
  ```



- [19.13](https://github.com/lin-123/javascript#whitespace--max-len) 避免一行代码超过100个字符（包含空格）。

- 注意： 对于[上面——strings--line-length](https://github.com/lin-123/javascript#strings--line-length)，长字符串不受此规则限制，不应分解。 eslint: [`max-len`](http://eslint.org/docs/rules/max-len.html)

  > Why? 这样确保可读性和可维护性

  ```
  // bad
  const foo = jsonData && jsonData.foo && jsonData.foo.bar && jsonData.foo.bar.baz && jsonData.foo.bar.baz.quux && jsonData.foo.bar.baz.quux.xyzzy;
  
  // bad
  $.ajax({ method: 'POST', url: 'https://airbnb.com/', data: { name: 'John' } }).done(() => console.log('Congratulations!')).fail(() => console.log('You have failed this city.'));
  
  // good
  const foo = jsonData
    && jsonData.foo
    && jsonData.foo.bar
    && jsonData.foo.bar.baz
    && jsonData.foo.bar.baz.quux
    && jsonData.foo.bar.baz.quux.xyzzy;
  
  // good
  $.ajax({
    method: 'POST',
    url: 'https://airbnb.com/',
    data: { name: 'John' },
  })
    .done(() => console.log('Congratulations!'))
    .fail(() => console.log('You have failed this city.'));
  ```



- [19.14](https://github.com/lin-123/javascript#whitespace--block-spacing) 作为语句的花括号内也要加空格 —— `{` 后和 `}` 前都需要空格。 eslint: [`block-spacing`](https://eslint.org/docs/rules/block-spacing)

  ```
  // bad
  function foo() {return true;}
  if (foo) { bar = 0;}
  
  // good
  function foo() { return true; }
  if (foo) { bar = 0; }
  ```



- [19.15](https://github.com/lin-123/javascript#whitespace--comma-spacing) `,` 前不要空格， `,` 后需要空格。 eslint: [`comma-spacing`](https://eslint.org/docs/rules/comma-spacing)

  ```
  // bad
  var foo = 1,bar = 2;
  var arr = [1 , 2];
  
  // good
  var foo = 1, bar = 2;
  var arr = [1, 2];
  ```



- [19.16](https://github.com/lin-123/javascript#whitespace--computed-property-spacing) 计算属性内要空格。参考上述花括号和中括号的规则。 eslint: [`computed-property-spacing`](https://eslint.org/docs/rules/computed-property-spacing)

  ```
  // bad
  obj[foo ]
  obj[ 'foo']
  var x = {[ b ]: a}
  obj[foo[ bar ]]
  
  // good
  obj[foo]
  obj['foo']
  var x = { [b]: a }
  obj[foo[bar]]
  ```



- [19.17](https://github.com/lin-123/javascript#whitespace--func-call-spacing) 调用函数时，函数名和小括号之间不要空格。 eslint: [`func-call-spacing`](https://eslint.org/docs/rules/func-call-spacing)

  ```
  // bad
  func ();
  
  func
  ();
  
  // good
  func();
  ```



- [19.18](https://github.com/lin-123/javascript#whitespace--key-spacing) 在对象的字面量属性中， `key` `value` 之间要有空格。 eslint: [`key-spacing`](https://eslint.org/docs/rules/key-spacing)

  ```
  // bad
  var obj = { "foo" : 42 };
  var obj2 = { "foo":42 };
  
  // good
  var obj = { "foo": 42 };
  ```



- [19.19](https://github.com/lin-123/javascript#whitespace--no-trailing-spaces) 行末不要空格。 eslint: [`no-trailing-spaces`](https://eslint.org/docs/rules/no-trailing-spaces)



- [19.20](https://github.com/lin-123/javascript#whitespace--no-multiple-empty-lines) 避免出现多个空行。 在文件末尾只允许空一行。 eslint: [`no-multiple-empty-lines`](https://eslint.org/docs/rules/no-multiple-empty-lines)

  ```
  // bad
  var x = 1;
  
  
  
  var y = 2;
  
  // good
  var x = 1;
  
  var y = 2;
  ```

**⬆ back to top**

## Commas



- [20.1](https://github.com/lin-123/javascript#commas--leading-trailing) 不要前置逗号。 eslint: [`comma-style`](http://eslint.org/docs/rules/comma-style.html)

  ```
  // bad
  const story = [
      once
    , upon
    , aTime
  ];
  
  // good
  const story = [
    once,
    upon,
    aTime,
  ];
  
  // bad
  const hero = {
      firstName: 'Ada'
    , lastName: 'Lovelace'
    , birthYear: 1815
    , superPower: 'computers'
  };
  
  // good
  const hero = {
    firstName: 'Ada',
    lastName: 'Lovelace',
    birthYear: 1815,
    superPower: 'computers',
  };
  ```



- [20.2](https://github.com/lin-123/javascript#commas--dangling) 额外结尾逗号: **要** eslint: [`comma-dangle`](http://eslint.org/docs/rules/comma-dangle.html)

  > Why? 这导致git diffs更清洁。 此外，像Babel这样的转换器会删除转换代码中的额外的逗号，这意味着你不必担心旧版浏览器中的[结尾逗号问题](https://github.com/airbnb/javascript/blob/es5-deprecated/es5/README.md#commas)。

  ```
  // bad - 没有结尾逗号的 git diff
  const hero = {
       firstName: 'Florence',
  -    lastName: 'Nightingale'
  +    lastName: 'Nightingale',
  +    inventorOf: ['coxcomb chart', 'modern nursing']
  };
  
  // good - 有结尾逗号的 git diff
  const hero = {
       firstName: 'Florence',
       lastName: 'Nightingale',
  +    inventorOf: ['coxcomb chart', 'modern nursing'],
  };
  ```

  ```
  // bad
  const hero = {
    firstName: 'Dana',
    lastName: 'Scully'
  };
  
  const heroes = [
    'Batman',
    'Superman'
  ];
  
  // good
  const hero = {
    firstName: 'Dana',
    lastName: 'Scully',
  };
  
  const heroes = [
    'Batman',
    'Superman',
  ];
  
  // bad
  function createHero(
    firstName,
    lastName,
    inventorOf
  ) {
    // does nothing
  }
  
  // good
  function createHero(
    firstName,
    lastName,
    inventorOf,
  ) {
    // does nothing
  }
  
  // good (note that a comma must not appear after a "rest" element)
  function createHero(
    firstName,
    lastName,
    inventorOf,
    ...heroArgs
  ) {
    // does nothing
  }
  
  // bad
  createHero(
    firstName,
    lastName,
    inventorOf
  );
  
  // good
  createHero(
    firstName,
    lastName,
    inventorOf,
  );
  
  // good (note that a comma must not appear after a "rest" element)
  createHero(
    firstName,
    lastName,
    inventorOf,
    ...heroArgs
  )
  ```

**⬆ back to top**

## Semicolons



- [21.1](https://github.com/lin-123/javascript#21.1) **Yup.** eslint: [`semi`](http://eslint.org/docs/rules/semi.html)

  > Why? 当 JavaScript 遇到没有分号结尾的一行，它会执行[自动插入分号 `Automatic Semicolon Insertion`](https://tc39.github.io/ecma262/#sec-automatic-semicolon-insertion)这一规则来决定行末是否加分号。如果JavaScript在你的断行里错误的插入了分号，就会出现一些古怪的行为。当新的功能加到JavaScript里后， 这些规则会变得更复杂难懂。显示的结束语句，并通过配置代码检查去捕获没有带分号的地方可以帮助你防止这种错误。

  ```
  // bad
  (function () {
    const name = 'Skywalker'
    return name
  })()
  
  // good
  (function () {
    const name = 'Skywalker';
    return name;
  }());
  
  // good, 行首加分号，避免文件被连接到一起时立即执行函数被当做变量来执行。
  ;(() => {
    const name = 'Skywalker';
    return name;
  }());
  ```

  [Read more](https://stackoverflow.com/questions/7365172/semicolon-before-self-invoking-function/7365214%237365214).

**⬆ back to top**

## Type Casting & Coercion



- [22.1](https://github.com/lin-123/javascript#coercion--explicit) 在语句开始执行强制类型转换。



- [22.2](https://github.com/lin-123/javascript#coercion--strings) Strings: eslint: [`no-new-wrappers`](https://eslint.org/docs/rules/no-new-wrappers)

  ```
  // => this.reviewScore = 9;
  
  // bad
  const totalScore = new String(this.reviewScore); // typeof totalScore is "object" not "string"
  
  // bad
  const totalScore = this.reviewScore + ''; // invokes this.reviewScore.valueOf()
  
  // bad
  const totalScore = this.reviewScore.toString(); // 不保证返回string
  
  // good
  const totalScore = String(this.reviewScore);
  ```



- [22.3](https://github.com/lin-123/javascript#coercion--numbers) Numbers: 用 `Number` 做类型转换，`parseInt`转换string常需要带上基数。 eslint: [`radix`](http://eslint.org/docs/rules/radix)

  ```
  const inputValue = '4';
  
  // bad
  const val = new Number(inputValue);
  
  // bad
  const val = +inputValue;
  
  // bad
  const val = inputValue >> 0;
  
  // bad
  const val = parseInt(inputValue);
  
  // good
  const val = Number(inputValue);
  
  // good
  const val = parseInt(inputValue, 10);
  ```



- [22.4](https://github.com/lin-123/javascript#coercion--comment-deviations) 请在注释中解释为什么要用移位运算和你在做什么。无论你做什么狂野的事，比如由于 `parseInt` 是你的性能瓶颈导致你一定要用移位运算。 请说明这个是因为[性能原因](https://jsperf.com/coercion-vs-casting/3),

  ```
  // good
  /**
   * parseInt是代码运行慢的原因
   * 用Bitshifting将字符串转成数字使代码运行效率大幅增长
   */
  const val = inputValue >> 0;
  ```



- [22.5](https://github.com/lin-123/javascript#coercion--bitwise) **注意:** 用移位运算要小心. 数字使用[64-位](https://es5.github.io/#x4.3.19)表示的，但移位运算常常返回的是32为整形[source](https://es5.github.io/#x11.7))。移位运算对大于32位的整数会导致意外行为。[Discussion](https://github.com/airbnb/javascript/issues/109). 最大的32位整数是 2,147,483,647:

  ```
  2147483647 >> 0 //=> 2147483647
  2147483648 >> 0 //=> -2147483648
  2147483649 >> 0 //=> -2147483647
  ```



- [22.6](https://github.com/lin-123/javascript#coercion--booleans) 布尔:

  ```
  const age = 0;
  
  // bad
  const hasAge = new Boolean(age);
  
  // good
  const hasAge = Boolean(age);
  
  // best
  const hasAge = !!age;
  ```

**⬆ back to top**

## Naming Conventions



- [23.1](https://github.com/lin-123/javascript#naming--descriptive) 避免用一个字母命名，让你的命名可描述。 eslint: [`id-length`](http://eslint.org/docs/rules/id-length)

  ```
  // bad
  function q() {
    // ...
  }
  
  // good
  function query() {
    // ...
  }
  ```



- [23.2](https://github.com/lin-123/javascript#naming--camelCase) 用小驼峰式命名你的对象、函数、实例。 eslint: [`camelcase`](http://eslint.org/docs/rules/camelcase.html)

  ```
  // bad
  const OBJEcttsssss = {};
  const this_is_my_object = {};
  function c() {}
  
  // good
  const thisIsMyObject = {};
  function thisIsMyFunction() {}
  ```



- [23.3](https://github.com/lin-123/javascript#naming--PascalCase) 用大驼峰式命名类。 eslint: [`new-cap`](http://eslint.org/docs/rules/new-cap.html)

  ```
  // bad
  function user(options) {
    this.name = options.name;
  }
  
  const bad = new user({
    name: 'nope',
  });
  
  // good
  class User {
    constructor(options) {
      this.name = options.name;
    }
  }
  
  const good = new User({
    name: 'yup',
  });
  ```



- [23.4](https://github.com/lin-123/javascript#naming--leading-underscore) 不要用前置或后置下划线。 eslint: [`no-underscore-dangle`](http://eslint.org/docs/rules/no-underscore-dangle.html)

  > Why? JavaScript 没有私有属性或私有方法的概念。尽管前置下划线通常的概念上意味着“private”，事实上，这些属性是完全公有的，因此这部分也是你的API的内容。这一概念可能会导致开发者误以为更改这个不会导致崩溃或者不需要测试。 如果你想要什么东西变成“private”，那就不要让它在这里出现。

  ```
  // bad
  this.__firstName__ = 'Panda';
  this.firstName_ = 'Panda';
  this._firstName = 'Panda';
  
  // good
  this.firstName = 'Panda';
  ```



- [23.5](https://github.com/lin-123/javascript#naming--self-this) 不要保存引用`this`， 用箭头函数或[函数绑定——Function#bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind).

  ```
  // bad
  function foo() {
    const self = this;
    return function () {
      console.log(self);
    };
  }
  
  // bad
  function foo() {
    const that = this;
    return function () {
      console.log(that);
    };
  }
  
  // good
  function foo() {
    return () => {
      console.log(this);
    };
  }
  ```



- [23.6](https://github.com/lin-123/javascript#naming--filename-matches-export) export default导出模块A，则这个文件名也叫A.*， import 时候的参数也叫A。 大小写完全一致。

  ```
  // file 1 contents
  class CheckBox {
    // ...
  }
  export default CheckBox;
  
  // file 2 contents
  export default function fortyTwo() { return 42; }
  
  // file 3 contents
  export default function insideDirectory() {}
  
  // in some other file
  // bad
  import CheckBox from './checkBox'; // PascalCase import/export, camelCase filename
  import FortyTwo from './FortyTwo'; // PascalCase import/filename, camelCase export
  import InsideDirectory from './InsideDirectory'; // PascalCase import/filename, camelCase export
  
  // bad
  import CheckBox from './check_box'; // PascalCase import/export, snake_case filename
  import forty_two from './forty_two'; // snake_case import/filename, camelCase export
  import inside_directory from './inside_directory'; // snake_case import, camelCase export
  import index from './inside_directory/index'; // requiring the index file explicitly
  import insideDirectory from './insideDirectory/index'; // requiring the index file explicitly
  
  // good
  import CheckBox from './CheckBox'; // PascalCase export/import/filename
  import fortyTwo from './fortyTwo'; // camelCase export/import/filename
  import insideDirectory from './insideDirectory'; // camelCase export/import/directory name/implicit "index"
  // ^ supports both insideDirectory.js and insideDirectory/index.js
  ```



- [23.7](https://github.com/lin-123/javascript#naming--camelCase-default-export) 当你export-default一个函数时，函数名用小驼峰，文件名需要和函数名一致。

  ```
  function makeStyleGuide() {
    // ...
  }
  
  export default makeStyleGuide;
  ```



- [23.8](https://github.com/lin-123/javascript#naming--PascalCase-singleton) 当你export一个结构体/类/单例/函数库/对象 时用大驼峰。

  ```
  const AirbnbStyleGuide = {
    es6: {
    }
  };
  
  export default AirbnbStyleGuide;
  ```



- [23.9](https://github.com/lin-123/javascript#naming--Acronyms-and-Initialisms) 简称和缩写应该全部大写或全部小写。

  > Why? 名字都是给人读的，不是为了适应电脑的算法的。

  ```
  // bad
  import SmsContainer from './containers/SmsContainer';
  
  // bad
  const HttpRequests = [
    // ...
  ];
  
  // good
  import SMSContainer from './containers/SMSContainer';
  
  // good
  const HTTPRequests = [
    // ...
  ];
  
  // also good
  const httpRequests = [
    // ...
  ];
  
  // best
  import TextMessageContainer from './containers/TextMessageContainer';
  
  // best
  const requests = [
    // ...
  ];
  ```



- [23.10](https://github.com/lin-123/javascript#naming--uppercase) 你可以用全大写字母设置静态变量，他需要满足三个条件。

  1. 导出变量
  2. 是 `const` 定义的， 保证不能被改变
  3. 这个变量是可信的，他的子属性都是不能被改变的

  > Why? 这是一个附加工具，帮助开发者去辨识一个变量是不是不可变的。

  - 对于所有的 `const` 变量呢？ —— 这个是不必要的。大写变量不应该在同一个文件里定义并使用， 它只能用来作为导出变量。 赞同！
  - 那导出的对象呢？ —— 大写变量处在export的最高级(e.g. `EXPORTED_OBJECT.key`) 并且他包含的所有子属性都是不可变的。

  ```
  // bad
  const PRIVATE_VARIABLE = 'should not be unnecessarily uppercased within a file';
  
  // bad
  export const THING_TO_BE_CHANGED = 'should obviously not be uppercased';
  
  // bad
  export let REASSIGNABLE_VARIABLE = 'do not use let with uppercase variables';
  
  // ---
  
  // 允许但不够语义化
  export const apiKey = 'SOMEKEY';
  
  // 在大多数情况下更好
  export const API_KEY = 'SOMEKEY';
  
  // ---
  
  // bad - 不必要的大写键，没有增加任何语言
  export const MAPPING = {
    KEY: 'value'
  };
  
  // good
  export const MAPPING = {
    key: 'value'
  };
  ```

**⬆ back to top**

## Accessors



- [24.1](https://github.com/lin-123/javascript#accessors--not-required) 不需要使用属性的访问器函数。



- [24.2](https://github.com/lin-123/javascript#accessors--no-getters-setters) 不要使用JavaScript的getters/setters，因为他们会产生副作用，并且难以测试、维护和理解。相反的，你可以用 getVal()和setVal('hello')去创造你自己的accessor函数

  ```
  // bad
  class Dragon {
    get age() {
      // ...
    }
  
    set age(value) {
      // ...
    }
  }
  
  // good
  class Dragon {
    getAge() {
      // ...
    }
  
    setAge(value) {
      // ...
    }
  }
  ```



- [24.3](https://github.com/lin-123/javascript#accessors--boolean-prefix) 如果属性/方法是`boolean`， 用 `isVal()` 或 `hasVal()`

  ```
  // bad
  if (!dragon.age()) {
    return false;
  }
  
  // good
  if (!dragon.hasAge()) {
    return false;
  }
  ```



- [24.4](https://github.com/lin-123/javascript#accessors--consistent) 用get()和set()函数是可以的，但是要一起用

  ```
  class Jedi {
    constructor(options = {}) {
      const lightsaber = options.lightsaber || 'blue';
      this.set('lightsaber', lightsaber);
    }
  
    set(key, val) {
      this[key] = val;
    }
  
    get(key) {
      return this[key];
    }
  }
  ```

**⬆ back to top**

## Events



- [25.1](https://github.com/lin-123/javascript#events--hash) 通过哈希而不是原始值向事件装载数据时(不论是DOM事件还是像Backbone事件的很多属性)。 这使得后续的贡献者（程序员）向这个事件装载更多的数据时不用去找或者更新每个处理器。例如：

  ```
  // bad
  $(this).trigger('listingUpdated', listing.id);
  
  // ...
  
  $(this).on('listingUpdated', (e, listingID) => {
    // do something with listingID
  });
  ```

  prefer:

  ```
  // good
  $(this).trigger('listingUpdated', { listingID: listing.id });
  
  // ...
  
  $(this).on('listingUpdated', (e, data) => {
    // do something with data.listingID
  });
  ```

**⬆ back to top**

## jQuery



- [26.1](https://github.com/lin-123/javascript#jquery--dollar-prefix) jQuery对象用`$`变量表示。

  ```
  // bad
  const sidebar = $('.sidebar');
  
  // good
  const $sidebar = $('.sidebar');
  
  // good
  const $sidebarBtn = $('.sidebar-btn');
  ```



- [26.2](https://github.com/lin-123/javascript#jquery--cache) 暂存jQuery查找

  ```
  // bad
  function setSidebar() {
    $('.sidebar').hide();
  
    // ...
  
    $('.sidebar').css({
      'background-color': 'pink'
    });
  }
  
  // good
  function setSidebar() {
    const $sidebar = $('.sidebar');
    $sidebar.hide();
  
    // ...
  
    $sidebar.css({
      'background-color': 'pink'
    });
  }
  ```



- [26.3](https://github.com/lin-123/javascript#jquery--queries) DOM查找用层叠式`$('.sidebar ul')` 或 父节点 > 子节点 `$('.sidebar > ul')`. [jsPerf](http://jsperf.com/jquery-find-vs-context-sel/16)



- [26.4](https://github.com/lin-123/javascript#jquery--find) 用jQuery对象查询作用域的`find`方法查询

  ```
  // bad
  $('ul', '.sidebar').hide();
  
  // bad
  $('.sidebar').find('ul').hide();
  
  // good
  $('.sidebar ul').hide();
  
  // good
  $('.sidebar > ul').hide();
  
  // good
  $sidebar.find('ul').hide();
  ```

**⬆ back to top**

## ES5 兼容性



- [27.1](https://github.com/lin-123/javascript#es5-compat--kangax) 参考[Kangax](https://twitter.com/kangax/)的ES5[兼容性列表](https://kangax.github.io/es5-compat-table/).

**⬆ back to top**

## ECMAScript 6+ (ES 2015+) Styles



- [28.1](https://github.com/lin-123/javascript#es6-styles) 这是收集到的各种ES6特性的链接

1. [箭头函数——Arrow Functions](https://github.com/lin-123/javascript#arrow-functions)
2. [类——Classes](https://github.com/lin-123/javascript#classes--constructors)
3. [对象缩写——Object Shorthand](https://github.com/lin-123/javascript#es6-object-shorthand)
4. [对象简写——Object Concise](https://github.com/lin-123/javascript#es6-object-concise)
5. [对象计算属性——Object Computed Properties](https://github.com/lin-123/javascript#es6-computed-properties)
6. [模板字符串——Template Strings](https://github.com/lin-123/javascript#es6-template-literals)
7. [解构赋值——Destructuring](https://github.com/lin-123/javascript#destructuring)
8. [默认参数——Default Parameters](https://github.com/lin-123/javascript#es6-default-parameters)
9. [Rest](https://github.com/lin-123/javascript#es6-rest)
10. [Array Spreads](https://github.com/lin-123/javascript#es6-array-spreads)
11. [Let and Const](https://github.com/lin-123/javascript#references)
12. [幂操作符——Exponentiation Operator](https://github.com/lin-123/javascript#es2016-properties--exponentiation-operator)
13. [迭代器和生成器——Iterators and Generators](https://github.com/lin-123/javascript#iterators-and-generators)
14. [模块——Modules](https://github.com/lin-123/javascript#modules)



- [28.2](https://github.com/lin-123/javascript#tc39-proposals) 不要用[TC39 proposals](https://github.com/tc39/proposals)， TC39还没有到 stage 3。

  > Why? [它还不是最终版](https://tc39.github.io/process-document/), 他可能还有很多变化，或者被撤销。 我们想要用的是 JavaScript， 提议还不是JavaScript。

**⬆ back to top**

## Standard Library

[标准库](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects)中包含一些功能受损但是由于历史原因遗留的工具类



- [29.1](https://github.com/lin-123/javascript#standard-library--isnan) 用 `Number.isNaN` 代替全局的 `isNaN`. eslint: [`no-restricted-globals`](https://eslint.org/docs/rules/no-restricted-globals)

  > Why? 全局 `isNaN` 强制把非数字转成数字， 然后对于任何强转后为 `NaN` 的变量都返回 `true` 如果你想用这个功能，就显式的用它。

  ```
  // bad
  isNaN('1.2'); // false
  isNaN('1.2.3'); // true
  
  // good
  Number.isNaN('1.2.3'); // false
  Number.isNaN(Number('1.2.3')); // true
  ```



- [29.2](https://github.com/lin-123/javascript#standard-library--isfinite) 用 `Number.isFinite` 代替 `isFinite`. eslint: [`no-restricted-globals`](https://eslint.org/docs/rules/no-restricted-globals)

  > Why? 理由同上，会把一个非数字变量强转成数字，然后做判断。

  ```
  // bad
  isFinite('2e3'); // true
  
  // good
  Number.isFinite('2e3'); // false
  Number.isFinite(parseInt('2e3', 10)); // true
  ```

## Testing



- [30.1](https://github.com/lin-123/javascript#testing--yup) **Yup.**

  ```
  function foo() {
    return true;
  }
  ```



- [30.2](https://github.com/lin-123/javascript#testing--for-real) **No, but seriously**:
- 无论用那个测试框架，你都需要写测试。
- 尽量去写很多小而美的纯函数，减少突变的发生
- 小心 stub 和 mock —— 这会让你的测试变得脆弱。
- 在 Airbnb 首选 [`mocha`](https://www.npmjs.com/package/mocha)。 [`tape`](https://www.npmjs.com/package/tape) 偶尔被用来测试一些小的，独立的模块。
- 100%测试覆盖率是我们努力的目标，即便实际上很少达到。
- 每当你修了一个bug， 都要写一个回归测试。 一个bug修复了，没有回归测试，很可能以后会再次出问题。