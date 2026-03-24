# 速读《JavaScript高级程序设计（第5版）》

<img src="./assets/image-20260320004855637.png" alt="image-20260320004855637" style="zoom:50%;" />

接下来，我们以Nodejs为运行环境，将注意力集中到JS语法本身，排除掉所有浏览器相关内容。

## 第3章 语言基础

### 变量

|          | var                                                          | let                                                          |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 作用域   | 函数作用域                                                   | 块级作用域                                                   |
| 变量提升 | var变量声明会自动提升到函数作用域顶部，因此可以先使用再声明。 | let也存在变量提升，**但存在暂时性死区（TDZ）**，在声明前访问会抛 `ReferenceError` |
| 重复声明 | 可以重复声明                                                 | 不能重复声明，报SyntaxError                                  |

- Node.js 顶层 `name = "xxx"` → 隐式全局变量，挂载到 `global`，会污染全局作用域（不推荐）。
- Node.js 顶层 `var name = "xxx"或 let name = "xxx"` → 模块私有变量，不绑定任何对象。
- const的行为和let基本一致，区别在于初始化时必须赋值，并且后续不能修改。
- 为了避免各种自动提升的错误，不再使用var，只使用let，const，优先使用const。



### 数据类型

**（1）数据类型**

| 类型 | Undefined | Null | Boolean    | Number   | String | Object                              | BigInt            | Symbol                   |
| ---- | --------- | ---- | ---------- | -------- | ------ | ----------------------------------- | ----------------- | ------------------------ |
| 值   | undefined | null | true/false | 123，NaN | "abc"  | {name: "张三", age: 25}，new Date() | 123n，BitInt(123) | Symbol()，Symbol('name') |

**（2）Undefined和Null不是一个东西**

undefined表示声明了变量，但没有初始化，Null表示一个空对象指针。他们概念不同却相等（==判断）。

他们在使用场景上不同，undefined不需要显示设置，但是如果一个对象不再使用，需要手动设置null。

**（3）Object属性键是变量时需要使用中括号**

``` js
const keyName = "age";
const user = {
    userName: 'flyzing', 
    [keyName]: 20 //变量属性
};
//测试输出：flyzing:20
console.log(user.userName + ':' + user[keyName]);
```

**（4）BigInt用于处理超过Number.MAX_SAFE_INTEGER的大整数**

BigInt用于处理2的53次方以上的数，大概九千万级别，平时很难用到。

**（5）Symbol用于创建唯一且不可重复的标识符，解决属性名冲突**

字面量只是描述这个符号的用途，即使字面量相同值也不同，Symbol('name')不等于Symbol('name')，因此需要放在常量类里引用。

``` js
//第一步：创建constants.js 统一管理Symbol
export const USER_ID = Symbol('user_id'); //创建我的用户id
export const USER_NAME = Symbol('user_name'); //创建我的用户名

//第二步：业务中导入Symbol使用
import {USER_ID, USER_NAME} from './constants.js'

//第三步：在其他方法中访问Symbol属性
function getUserId(data) {
    return data[USER_ID];
}

const user = {
    [USER_ID]: '1001', //Symbol唯一，不会和任何字符串键冲突
    [USER_NAME]: 'flyzing',
    age: 40
};
//测试输出：1001
console.log(getUserId(user)); 
```



### 操作符

**（1）相等**

先进行强制类型转换再判断值，这种隐式转换容易出错，因此不建议再使用。

``` js
console.log(1 == true); //true先转换为1再比较，返回true
```

**（2）全等**

先判断类型再判断值，类型不同直接返回false，判断更加准确，代替相等。

``` js
console.log(1 === true); //类型不同，返回false
console.log('123' !== 123); //类型不同，返回true
console.log(null === undefined); //类型不同，返回false
```

**（3）空集合并操作**

为处理空值，只给null或undefined提供默认值。使用||操作符提供默认值时，容易被假性值干扰（可以转为false的值，如0，''），因此不建议再使用。

``` js
const values = [null, undefined, 0, ''];
console.log(values.map(x => x || 'default')); 
//输出：[ 'default', 'default', 'default', 'default' ]
console.log(values.map(x => x ?? 'default'));
//输出：[ 'default', 'default', 0, '' ]
```



### 语句

**（1）if语句**

这里的条件表达式可以是任意表达式，js会自动将其转为boolean值，如果为true则执行后续语句。

注意Js中的假性值有false，0，NaN，'', null, undefined，其余值转boolean都返回true。

**（2）for-of语句和for-in语句区别**

for-of用于遍历可迭代对象的元素，for-in用于枚举对象中的非符号键属性，容易出现不可预见的结果，因此不建议使用for-in。

```js
let arr = [1, 2, 3];
for(const el of arr) {
    console.log(el); //依次输出1,2,3
}
for(const propName in arr) {
    //依次输出0,1,2，JS中数组本质是对象，索引就是属性名，这种不可预见的结果导致不建议使用for-in
    console.log(propName); 
}
```

**（3）switch语句**

- js中的switch语句可以用于所有数据类型，不仅限于数字类型。
- case条件值不必须是常量，可以是变量或表达式。
- 在比较每个条件值时会实用全等操作符。因此不会强制转换数据类型。

另外漏掉break会造成穿透，后续case不会再判断直接执行直到遇到break为止，这会造成不可预测的结果，因此建议都带break。

```js
let dynamicVal = "dynamicVal";
function checkValue(val) {
  switch (val) {
    case dynamicVal:
      console.log("这是变量" + val);
      break;
    case 5:
      console.log("这是数字5");
    // break;
    case "5":
      console.log("这是字符串5");
      break;
    default:
      console.log("无匹配");
  }
}
checkValue(5);
//依次输出：这是数字5，这是字符串5
```

### 函数

函数要么有函数值，要么不返回值，只在某个条件下返回值的函数会带来麻烦，尤其是调试时。这在其他语言中编译阶段就会报错，JS不报错但不建议这么做。

```js
//默认返回值兜底
function calculateScore1(isPass) {
  if (isPass) {
    return 100;
  }
  return null; // 显式返回 null，而非隐式 undefined
}

//没有返回值的异常场景
function calculateScore2(isPass) {
  if (!isPass) {
    throw new Error("未通过，无分数"); // 异常场景抛错，终止执行
  }
  return 100; // 正常场景必返回数值
}

console.log(calculateScore1(false)); //null
console.log(calculateScore2(false)); //Error: 未通过，无分数
```



## 第4章 变量、作用域与内存

### 引用类型和值类型

|                   | 值类型                                                       | 引用类型                                                 |
| ----------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| 存储 / 访问方式   | 直接存储 / 访问“值”本身                                      | 存储 / 访问“值的内存地址”（引用）                        |
| 动态属性          | 不可变，不能动态添加属性                                     | 可以动态添加 / 修改 / 删除属性                           |
| 复制值 / 传递参数 | 创建值的副本，修改互不影响                                   | 创建内存地址的副本，指向同一个对象                       |
| 确定类型          | 使用typeof，返回undefined、object(Null返回)、boolean、number、bigInt、string和symbol | 使用typeof都会返回object，需要使用instanceof判断具体类型 |

### 上下文与作用域

每段代码执行时都有一个执行环境，这就是**执行上下文**（简称上下文）。上下文采用**栈式存储（先进后出）**，执行时**动态变化**。

```
执行上下文 = {
  1. 变量对象/活动对象（VO/AO）：存储变量、函数、arguments → 变量的实际存储位置
  2. 作用域链（Scope Chain）：变量查找的规则 → 按顺序导航到VO/AO中查找变量
  3. this绑定（This Binding）：上下文的核心 → 函数执行时的“归属对象”
}
```

上下文中包含的**作用域链**，采用**链式结构**，核心作用是逐级查找变量，在函数**定义时就已固定（静态）**。

```js
name = "张三"; //绑定在global对象上，全局上下文的VO里
function fn() { // 函数fn的执行上下文：
    let innerVar = '内部变量';
    console.log(this.name); //上下文的核心：this的指向
    console.log(innerVar);  //上下文AO里的变量，通过作用域找到
}
// 1. 全局上下文初始化
// VO: { name: "张三", fn: 函数对象 }
// 作用域链: [全局VO]
// this: global

fn(); // 输出 "张三"，内部变量
// 2. 调用fn，创建fn的上下文：
// AO: { innerVar: "内部变量" }
// 作用域链: [fn的AO → 全局VO]
// this: global

let obj = { name: "李四", fn: fn };
// 3. 全局上下文：新增 obj 变量
// VO: { name: "张三", fn: 函数对象, obj: { name: "李四", fn: fn }}
// 作用域链: [全局VO]
// this: global

obj.fn(); // 输出 "李四"（this变了），内部变量
// 4. 作为对象方法调用，fn的上下文变了（核心是this变，变量规则不变）：
// AO: { innerVar: "内部变量" }
// 作用域链: [fn的AO → 全局VO]（找变量的规则没变）
// this: obj（核心变了）
```

另外，不要使用with改变作用域链，会导致不可预期的结果。

### 垃圾回收

- 离开作用域的值会自动标记为可回收，然后在垃圾回收期间被删除。
- 为促进内存回收，全局对象，全局对象的属性和循环引用都应该在不需要时解除引用。

```js
function setName() {
    // 1. 未用var/let/const声明的name，绑定在全局对象global上，不会被回收，避免使用。
    name = 'flyzing';
}
setName();

let userName = 'flyzing';
const timer = setInterval(() => {
    // 2. 定时器回调函数使用了外部变量userName，定时器一直运行就不会被回收。
    console.log(userName);
}, 1000)
// 因此当不需要时，必须清除定时器
// clearInterval(timer);  // 这样 userName 就可以被回收了

let outer = function() {
    let name = 'flyzing';
    return function() {
        // 3. 在创建闭包时，就保存了外部变量name。不会被回收。
        return name;
    }
}
let fn = outer();  
// 不再使用时需要置为空，fn 被回收，name 才会被回收
fn = null;  // 现在 name 可以被垃圾回收了
```



## 第5章 基本引用类型

### 原始值包装类型

前文提到，JS中变量分为两大类：

- 原始类型：值不可变，存储在栈内存，赋值时是”值拷贝“。包括：String、Number、Boolean、Undefined、Null、Symbol、BigInt。
- 引用类型：值可变，存储在堆内存，赋值时是“引用拷贝”。包括：Object。

原始类型本身是没有属性和方法的，那为什么我们如下使用？

```js
console.log('abc'.length); // 3
console.log(123.456.toFixed(2)); //123.46
console.log(true.toString()); //true
```

这是因为String、Number、Boolean这三个原始类型非常特殊，他们的字面量是原始类型，但可以通过new来创建对应的包装对象是引用类型。当我们使用他们的方法时，JS自动创建临时的包装对象，调用方法，然后立即销毁包装对象（不是等到作用域结束）。

另外，不建议手动创建包装类型，会造成不可预期的错误，如Boolean包装对象即使值是false，在真值判断（如 if/while 条件）中也会被判定为真值。

### 单例内置对象

顾名思义，js开始执行时就存在的，只有一个实例的对象，如下：

- Math 数学工具集，提供计算相关方法（min/max，round，random等）， 无构造器能力。

- JSON 处理JSON序列化/反序列化（parse，stringify），全局唯一的JSON工具对象。
- globalThis ES规范中统一的全局对象（浏览器中等于window，Nodejs中等于global），承载所有全局变量和全局方法，如encodeURIComponent，eval等。

## 第6章 高级引用类型

### Array

```js

//1、扩展操作符
//可以将数组元素逐个展开，从而更容易的完成数组复制，合并。
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5, 6];
console.log(arr2); //[ 1, 2, 3, 4, 5, 6 ]
//还可以用于对象展开
const user = {name: '张三', age: 18};
const newUser = {...user, city: '武汉'};
console.log(newUser); //{ name: '张三', age: 18, city: '武汉' }

//2、剩余操作符
//用于将数组的剩余元素收集到一个新数组中，通常用于函数参数
function sum(...numbers) {
  //reduce((累加结果, 当前项) => 累加结果 + 当前项, 初始值)
  return numbers.reduce((total, num) => total + num, 0);
}
console.log(sum(1, 2, 3, 4, 5)); //15
//或解构赋值
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(rest); //[ 3, 4, 5 ]

//3、栈方法，提供后进先出（LIFO）的方法，即插入和删除都在栈顶。
let stack = [1, 2];
//push推入栈顶
let count = stack.push(3, 4); 
console.log(`stack length: ${count}`); //stack length: 4
//pop从栈顶取出
console.log(stack.pop()); //4

//4、队列方法，提供先进先出（FIFO）的方法，即在队尾插入，在队首删除。
let queue = [1, 2];
//push加入队尾
count = queue.push(3, 4); 
console.log(`queue length: ${count}`); //queue length: 4
//shift从队首取出
console.log(stack.shift()); //1

//5、排序
let values = [0, 10, 5, 1, 15];
//默认情况下，先调用String()转换函数，再按照字符串升序排序
values.sort(); 
console.log(values); //[ 0, 1, 10, 15, 5 ]
//想要按照数值排序，可以使用比较函数做参数。
//如果第一个参数应该排在第二个参数前面，则返回-1，反之为1，相等为0
values.sort((a, b) => { 
  if (a < b) { //升序
    return -1; 
  } else if (a > b) {
    return 1;
  } else {
    return 0;
  }
});
console.log(values); //[ 0, 1, 5, 10, 15 ]
//简化：
values.sort((a, b) => b - a) //倒序
console.log(values);  //[ 15, 10, 5, 1, 0 ]

//6、截取，不修改原数据
let colors = ['red', 'green', 'blue', 'yellow', 'pink'];
let colors2 = colors.slice(1, 4);
console.log(colors2); //[ 'green', 'blue', 'yellow' ]

//7、万能增删改
//从第0个位置，删除1个元素。
colors = ['red', 'green', 'blue', 'yellow', 'pink'];
let removed = colors.splice(0, 1); 
console.log(removed); //[ 'red' ]
console.log(colors); //[ 'green', 'blue', 'yellow', 'pink' ]
//从第1个位置，删除2个元素，然后在该位置在插入2元素
colors = ['red', 'green', 'blue', 'yellow', 'pink'];
removed = colors.splice(1, 2, 'black', 'gray');
console.log(removed); //[ 'green', 'blue' ]
console.log(colors); //[ 'red', 'black', 'gray', 'yellow', 'pink' ]
```

