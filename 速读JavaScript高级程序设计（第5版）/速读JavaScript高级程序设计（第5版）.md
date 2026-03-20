# 速读《JavaScript高级程序设计（第5版）》

<img src="./assets/image-20260320004855637.png" alt="image-20260320004855637" style="zoom:50%;" />

接下来，我们以Nodejs为运行环境，将注意力集中到JS语法本身，排除掉所有浏览器相关内容。

## 第3章 语言基础

### 1. 变量

**（1）var，let区别**

|            | var                                                          | let                                                          |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 作用域     | 函数作用域                                                   | 块级作用域                                                   |
| 变量提升   | 变量声明会自动提升到函数作用域顶部，因此可以重复声明，也可以先使用再声明。 | 不会自动提升，因此不能重复声明，不能先使用再声明，会抛出ReferenceError，也被称为暂时性死区。 |
| window属性 | 在全局作用域中使用var声明的变量会自动成为window对象的属性    | 不会成为window属性                                           |

**（2）let，const区别**

const的行为和let基本一致，区别在于初始化时必须赋值，并且后续不能修改。

因此，为了避免各种奇怪的错误，不再使用var，只使用let，const，优先使用const。



### 2. 数据类型

**（1）8种数据类型**

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



### 3. 操作符

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



### 4. 语句

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

js中的switch语句可以用于所有数据类型，不仅限于数字类型。条件值不需要是常量，可以是变量或表达式。

switch语句在比较每个条件值时会实用全等操作符。因此不会强制转换数据类型。

另外漏掉break会造成穿透，后续case不会再判断直接执行直到遇到break为止，这会造成不可预测的结果，因此都建议带break。

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







