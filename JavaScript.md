# JavaScript
[TOC]
## 简介
**JavaScript组成：**

* 核心（ECMAScript）
* 文档对象模型（DOM）
* 浏览器对象模型（BOM）

## 在HTML中使用
### &lt;script&gt;元素
```
<script type="text/javascript">
    function sayHi() {
        alert("Hi!");
    }
</script>
<script type="text/javascript" src="example.js"></script>
```
`<script>`元素内部的JavaScript代码将被从上至下依次解释。就前面例子来说，解释器会解释一个函数的定义，然后将该定义保存在自己的环境当中。在解释器对`<script>`元素内部的所有代码求值完毕以前，页面中的其余内容都不会被浏览器加载或显示。

与解析嵌入式JavaScript代码一样，在解析外部JavaScript文件（包括下载该文件）时，页面的处理也会暂时停止。

**标签的设置**
传统的做法时将`<script>`元素放在页面的`<head>`元素中，意味着必须等到全部JavaScript代码都被下载、解析和执行完成后才开始呈现页面内容。为避免这个问题，现代Web应用程序一般把全部JavaScript引用放在`<body>`元素页面内容后面。
## 基本概念
### 语法
ECMA中的一切（变量、函数名和操作符）都区分大小写。
ECMAScript中的语句以一个分号结尾`;`如果省略分号，则由解析器确定语句的结尾。
### 变量
ECMAScript的变量是松散类型的，可以用来保存任何类型的数据。每个变量仅仅是一个用于保存值的占位符而已。
用`var`操作符定义的变量将成为定义该变量的作用域中的局部变量。
``` javascript
var message = "hi";
// 变量message中保存了一个字符串值"hi"。像这样初始化变量并不会把它标记为字符串类型；
// 始化过程就是给变量赋一个值那么简单。因此，可以在修改变量值的同时修改值的类型
message = 100;
```
### 数据类型

ECMAScript中有5种简单数据类型（基本数据类型）：Undefined、Null、Boolean、Number和String。还有1种复杂数据类型Object，Object本质上是由一组无序的名值对组成的。
#### typeof操作符
用来检测给定变量的数据类型。对一个值使用`typeof`操作符可能返回下列某个字符串：

* `"undefined"`——未定义
* `"boolean"`——布尔值
* `"string"`——字符串
* `"number"`——数值
* `"object"`——对象或`null`，特殊值`null`被认为是一个空的对象引用
* `"function"`——函数

对正则表达式调用会返回`"function"`或`"object"`。
> 从技术角度讲，函数在ECMAScript中是对象，不是数据类型。然而，函数也确实有一些特殊的属性，因此通过typeof操作符来区分函数和其他对象是有必要的。

#### Undefined类型
Undefined类型只有一个值：`undefined`。在使用`var`声明变量但未对其初始化时，这个变量的值就是`undefined`。
对于尚未声明过的变量，只能执行一项操作，即使用`typeof`操作符检测其数据类型。

#### Null类型
Null类型也只有一个值：`null`。从逻辑角度来看，`null`值表示一个空对象指针。
如果定义的变量准备在将来保存对象，最好将该变量初始化为`null`。这样一来只要直接检查`null`值就可以知道相应的变量是否已经保存了一个对象的引用。

实际上，`undefined`值是派生自`null`值的，因此ECMA-262规定对它们的相等性测试要返回`true`

#### Boolean类型
Boolean类型只有两个字面值：`true`和`false`。

虽然Boolean类型的字面值只有两个，但ECMAScript中所有类型的值都有与这两个Boolean值等价的值。要将一个值转换为其对应的Boolean值，可以调用转型函数Boolean()：
```javascript
var message = "Hello world!";
var messageAsBoolean = Boolean(message);
```

数据类型|true|false
-|-|-
Boolean|`true`|`false`
String|非空字符串|空字符串
Number|非零数字（包括无穷大）|`0`和`NaN`
Object|任何对象|`null`
Undefined|n/a(not applicable)，不适用|`undefined`

#### Number类型
Number类型使用IEEE754格式来表示整数和浮点数（双精度数值）。
八进制的第一位必须是0，然后是八进制数字序列（0~7），如果字面值中的数值超出范围，那么前导零将被忽略，后面的数值被当作十进制数值解析。
十六进制前两位必须是0x，后跟任何十六进制数字（0~9及A~F）。字母可以大写或小写。
在进行算术计算时，所有以八进制和十六进制表示的数值最终都将被转换成十进制数值。

**浮点数值**
默认情况下，ECMASctipt会将小数点后面带有6个零以上的浮点数值以e表示法表示。（例如，`0.0000003`会被转换成`3e-7`）

**数值范围**
ECMAScript能够表示的最小数值保存在`Number.MIN_VALUE`中：`5e-324`；最大数值保存在`Number.MAX_VALUE`中，如果某次计算的结果超出数值范围，那么这个数值将被自动转换成特殊的`Infinity`值。

**NaN**
非数值（Not a Number）是一个特殊的数值，用于表示一个本来要返回数值的操作数未返回数值的情况（这样就不会抛出错误了）。例如，在其他编程语言中，任何数值除以`0`都会导致错误，从而停止代码执行。但在ECMAScript中，任何数值除以`0`会返回`NaN`，因此不会影响其他代码的执行。
`NaN`本身有两个非同寻常的特点。首先，任何涉及`NaN`的操作（例如`NaN/10`）都会返回`NaN`，这个特点在多步计算中有可能导致问题。其次，`NaN`与任何值都不相等，包括`NaN`本身。

**数值转换**
转型函数`Number()`可以用于任何数据类型，`ParseInt()`和`parseFloat()`用于把字符串转换成数值。
`Number()`函数的转换规则：

* Boolean值，`true`和`false`分别转换为`1`和`0`
* 数字值，简单的传入和返回
* `null`值，返回`0`
* `undefined`，返回`NaN`
* 字符串，遵循下列规则：
    - 只包含数字，转换为十进制数值，`"123"`变成`123`，`"011"`变成`11`（忽略前导零）
    - 包含有效浮点格式，转换为对应的浮点数值（忽略前导零）
    - 包含有效十六进制格式，转换为相同大小的十进制整数值
    - 空字符串，转换为`0`
    - 包含除上述格式之外的字符，转换为`NaN`
* 对象，调用`valueOf()`方法，依照前面的规则转换。如果转换结果是NaN，则调用`toString()`方法，再次依照前面的规则转换。

> 由于`Number()`函数在转换字符串时比较复杂而且不够合理，因此在处理整数的时候更常用的是`parseInt()`函数。

`parseInt()`函数在转换字符串时，更多的是看其是否符合数值模式。它会忽略字符串前面的空格，直至找到第一个非空格字符。如果第一个字符不是数字字符或者负号，`parseInt()`就会返回`NaN`；如果是数字字符，会继续解析第二个字符，直到解析完所有后续字符或者遇到了一个非数字字符。

`parseInt()`能够识别出各种整数格式（十进制、八进制和十六进制数）。如果字符串以`"0x"`开头且后跟数字字符，就会将其当作十六进制整数；以`"0"`开头且后跟数字字符，则会当作八进制数来解析。
``` javascript
var num1 = parseInt("");         // NaN
var num2 = parseInt("22.5");     // 22
var num3 = parseInt("1234blue"); // 1234
var num4 = parseInt("70");       // 70 （十进制数）
var num5 = parseInt("070");      // 56 （八进制数）
var num6 = parseInt("0xA");      // 10 （十六进制数）
// ECMAScript 5 JavaScript引擎中，parseInt()不具有解析八进制值的能力，因此前导零会无效。
// 为消除这种困惑，可以提供第二个参数：转换时使用的参数。
var num1 = parseInt("0xAF", 16); // 175
var num2 = parseInt("AF", 16);   // 175
```

`parseFloat()`也是从第一个字符开始解析每个字符。而且也是一直解析到字符串末尾，或者解析到遇见一个无效的浮点数字字符为止。也就是说，字符串中的第一个小数点是有效的，而第二个小数点就是无效的了，因此它后面的字符串将被忽略。

第二个区别在于它始终都会忽略前导零。`parseFloat()`可以识别所有浮点数值格式。只解析十进制值，所以没有第二个参数。还要注意一点：如果字符串包含的是一个可解析为整数的数（没有小数点，或者小数点后都是零），`parseFloat()`会返回整数。
``` javascript
var num1 = parseFloat("1234blue"); // 1234
var num2 = parseFloat("0xA");      // 0
var num3 = parseFloat("22.5");     // 22.5
var num4 = parseFloat("22.34.5");  // 22.34
var num5 = parseFloat("3.125e7");  // 31250000
```

#### String类型
String类型用于表示由零或多个16位Unicode字符组成的字符序列，即字符串。字符串可以由双引号（"）或单引号（'）表示。

**字符字面量（转义序列）**

字面量|含义
-|-
\n|换行
\t|制表
\b|空格
\r|回车
\f|进纸
\ \|斜杠
\ '|单引号，在用单引号表示的字符串中使用
\ "|双引号，在用双引号表示的字符串中使用
\xnn|以十六进制代码nn表示一个字符。\x41表示"A"
\unnnn|以十六进制代码nnnn表示一个Unicode字符。\u03a3表示希腊字符Σ

**转换为字符串**
一种方式是使用`toString()`方法。数值、布尔值、对象和字符串值都有`toString()`方法，`null`和`undefined`值没有。
不知道要转换的值是不是`null`或`undefined`的情况下，可以使用转型函数`String()`；

* 如果值有`toString()`方法则调用
* 如果值是`null`，则返回`"null"`
* 如果值是`undefined`,则返回`"undefined"`

#### Object类型
ECMAScript中的对象就是一组**数据**和**功能**的集合。对象可以通过执行`new`操作符后跟要创建的对象类型的名称来创建。
创建`Object`类型的实例并为其添加属性和方法，就可以创建自定义对象。
``` javascript
var o = new Object();
var o = new Object; // 不推荐
```
Object类型是所有它的实例的基础。其所具有的任何属性和方法存在于更具体的对象中。
Object的每个实例都具有下列属性和方法：

* `constructor`（构造函数）：保存着用于创建当前对象的函数。例：`Object()`
* `hasOwnProperty(propertyName)`：检查给定的属性是否存在当前对象实例中（而不是在实例的原型中）
* `isPrototypeOf(object)`：检查传入的对象是否是传入对象的原型
* `propertyIsEnumerable(propertyName)`：检查属性是否能够使用`for-in`语句来枚举
* `toLocaleString()`、`toString()`：返回对象的字符串表示
* `valueOf()`：返回对象的字符串、数值或布尔值表示。通常与`toString()`方法的返回值相同

### 函数
``` javascript
// ECMAScript中的函数使用function关键字来声明，后跟一组参数以及函数体。
// 函数的基本语法：
function functionName(arg0, arg1, ..., argN) {
    statements
}
// 函数实例：
function sayHi(name, message) {
    alert("Hello " + name + ", " + message);
}
// 调用
sayHi("Nicholas", "how are you today?");

// 函数在定义时不必指定是否返回值。
// 任何函数在任何时候都可以通过return语句后跟要返回的值来实现返回值
function sum(num1, num2) {
    return num1 + num2;
}

var result = sum(5, 10);

// return语句可以不带有任何返回值，函数将在停止执行后将返回undefined值。
```

**理解参数**
ECMAScript函数不介意传递进来多少个参数，也不在乎传递进来参数是什么数据类型。即便定义的函数只接受两个参数，调用这个函数时也未必一定要传递两个参数。

在函数体内可以通过`arguments`对象来访问参数数组。`arguments`对象只是与数组类似（并不是Array的实例），可以使用方括号语法访问元素`arguments[0]`，使用`length`属性来确定参数个数。

这个事实说明ECMAScript函数的一个重要特点：命名的参数只提供便利，但不是必须的。

`arguments`的值永远与对应命名参数的值保持同步，但是它们的内存空间是独立的。`arguments`对象的长度是由传入的参数个数决定的，不是由定义函数时的命名参数的个数决定的。

## 变量、作用域
### 基本类型和引用类型的值
ECMAScript变量可能包含两种不同数据类型的值：基本类型值和引用类型值。基本类型值指的是简单的数据段，而引用类型值指那些可能由多个值构成的对象。

**动态的属性**
定义基本类型值和引用类型值的方式是类似的：创建一个变量并为该变量赋值。但是，当这个值保存到变量中以后，对不同类型值可以执行的操作则大相径庭。对于引用类型的值，我们可以为其添加属性和方法，也可以改变和删除其属性和方法。
``` javascript
var person = new Object();
person.name = "Nicholas";
```

**检测类型**
通常，我们并不是想知道某个值是对象，而是想知道它是什么类型的对象。为此，ECMAScript提供了`instanceof`操作符，语法如下：
``` javascript
result = variable instanceof constructor

alert(person instanceof Object);
alert(colors instanceof Array);
alert(pattern instanceof RegExp);
```
所有引用类型的值都是`Object`的实例。因此，在检测一个引用类型值和Object构造函数时，`instanceof`操作符始终会返回`true`。

##执行环境及作用域
**执行环境**（`execution context`）是JavaScript中最为重要的一个概念。执行环境定义了变量或函数有权访问的其他数据，决定了它们各自的行为。每个执行环境都有一个与之关联的**变量对象**（`variable object`），环境中定义的所有变量和函数都保存在这个对象中。虽然我们编写的代码无法访问这个对象，但解析器在处理数据时会在后台使用它。

**全局执行环境**是最外围的一个执行环境。在Web浏览器中，全局执行环境是`window`对象，因此所有全局变量和函数都是作为`window`对象的属性和方法创建的。某个执行环境中的所有代码执行完毕后，该环境被销毁，保存在其中的所有变量和函数定义也随之销毁（全局执行环境直到应用程序退出——例如关闭网页或浏览器——时才会被销毁）。

**每个函数都有自己的执行环境**。当执行流进入一个函数时，函数的环境就会被推入一个环境栈中。而在函数执行之后，栈将其环境弹出，把控制权返回给之前的执行环境。ECMAScript程序中的执行流正是由这个方便的机制控制着。

当代码在一个环境中执行时，会创建变量对象的一个**作用域链**（`scope chain`）。作用域链的用途，是保证对执行环境有权访问的所有变量和函数的有序访问。作用域链的前端，始终都是当前执行的代码所在环境的变量对象。如果这个环境是函数，则将其**活动对象**（`activation object`）作为变量对象。活动对象在最开始时只包含一个变量，即`arguments`对象（这个对象在全局环境中是不存在的）。作用域链中的下一个变量对象来自包含（外部）环境，而再下一个变量对象则来自下一个包含环境。这样，一直延续到全局执行环境；全局执行环境的变量对象始终都是作用域链中的最后一个对象。

标识符解析是沿着作用域链一级一级地搜索标识符的过程。搜索过程始终从作用域链的前端开始，然后逐级地向后回溯，直至找到标识符为止（如果找不到标识符，通常会导致错误发生）。
``` javascript
var color = "blue";

function changeColor() {
    var anotherColor = "red";

    function swapColors() {
        var tempColor = anotherColor;
        anotherColor = color;
        color = tempColor;
        // 这里可以访问color、anotherColor、tempColor
    }
    // 这里可以访问color、anotherColor
}
// 这里只能访问color
changeColor();
```
> 函数参数也被当作变量来对待，因此其访问规则与执行环境中的其他变量相同。

###没有块级作用域
在JavaScript中，`if`、`for`语句中的变量声明会将变量添加到当前的执行环境中。
使用`var`声明的变量会自动被添加到最接近的环境中。在函数内部，最接近的环境就是函数的局部环境；在`with`语句中，最接近的环境是函数环境。如果初始化变量时没用使用`var`声明，该变量会自动被添加到全局环境。
``` javascript
if (true) {
    var color = "blue";
}
alert(color); // "blue"

for (var i=0; i < 10; i++) {
    doSomething(i);
}
alert(i); // 10
```

## 引用类型
引用类型的值（对象）是**引用类型**的一个实例。在ECMAScript中，引用类型是一种数据结构，用于将数据和功能组织在一起。它也常被称为**类**。引用类型有时候也被称为**对象定义**，因为它们描述的是一类对象所具有的属性和方法。
新对象是使用`new`操作符后跟一个构造函数来创建的。构造函数本身就是一个函数，只不过该函数是出于穿件新对象的目的而定义的。
### Object类型
创建Object实例的方式有两种：
``` javascript
// 使用new操作符后跟Object构造函数
var person = new Object();
person.name = "Nicholas";
person.age = 29;
// 使用对象字面量表示法，对象定义的一种简写形式，目的在于简化创建包含大量属性的对象的过程。
var person = {};
var person = {
    name : "Nicholas",
    "age" : 29 // 属性名可以使用字符串
}
```
一般来说，访问对象属性时使用的都是点表示法，这也是很多面向对象语言中通用的语法。在JavaScript也可以使用方括号来访问对象的属性。
``` javascript
alert(person["name"]);
alert(person.name);
```

### Array类型
ECMAScript数组的每一项可以保存任何类型的数据。大小时可以动态调整的。
``` javascript
var colors = new Array();
var colors = new Array(20);
var colors = new Array("red", "blue", "green");
// 可以省略new操作符
var colors = Array(3);
var names = Array("Greg");
// 使用数组字面量表示法
var colors = ["red", "blue", "green"];
var name = [];
// 读取和设置数组的值
alert(colors[0]);
colors[2] = "black"; // 修改第三项
colors[3] = "brown"; // 新增第四项
// length属性不是只读的。通过设置这个属性可以从数组的末尾移除或添加项
colors.length = 2;
alert(colors[2]);  // undefined
```

#### 检测数组
``` javascript
if (value instanceof Array) {}
if (Array.isArray(value)) {}
```

#### 转换方法
``` javascript
var colors = ["red", "blue", "green"];
alert(colors.toString()); //red,blue,green
alert(colors.valueOf());  //red,blue,green
alert(colors);            //red,blue,green 调用toString()
// join()方法可以使用不同的分隔符来构建这个字符串
alert(colors.join("||")); //red||green||blue
```

#### 重排序方法
``` javascript
// reverse()方法会反转数组项的顺序
var values = [0, 1, 5, 10, 15];
values.reverse();
alert(values); //15,10,5,1,0
// 默认情况下，sort()按升序排列，会调用每个数组项的toString()转型方法进行比较。即使数组中的每一项都是数值
var values = [0, 1, 5, 10, 15];
values.sort();
alert(values); //0,1,10,15,5
// sort()方法可以接收一个比较函数作为参数
function compare(value1, value2) {
    if (value1 < value2) {
        return -1;
    } else if (value1 > value2) {
        return 1;
    } else {
        return 0;
    }
}
values.sort(compare);
alert(values); //0,1,5,10,15
```

#### 操作方法
``` javascript
var colors = ["red", "green", "blue"];
var colors2 = colors.concat("yellow", ["black", "brown"]);
alert(colors);  //red,green,blue
alert(colors2); //red,green,blue,yellow,black,brown

var colors = ["red", "green", "blue", "yellow", "purple"];
var colors2 = colors.slice(1);
var colors3 = colors.slice(1, 4); // [1, 4) 包括1，不包括4
alert(colors2); //green,blue,yellow,purple
alert(colors3); //green,blue,yellow

var colors = ["red", "green", "blue"];
var removed = colors.splice(0, 1); // 删除第一项
alert(colors);  //green,blue
aloert(removed);//red, 返回的数组中只包含一项
removed = colors.splice(1, 0, "yellow", "orange"); // 从位置1开始插入两项
alert(colors);  //green,yellow,orange,blue
alert(removed); //返回的是一个空数组
removed = colors.splice(1, 1, "red", "purple"); // 插入两项，删除一项
alert(colors);  //green,red,purple,orange,blue
alert(removed); //yellow，返回的数组中只包含一项

```

####  位置方法
``` javascript
var numbers = [1, 2, 3, 4, 5, 4 ,3, 2, 1];
alert(numbers.indexOf(4));        //3
alert(numbers.lastIndexOf(4));    //5
alert(numbers.indexOf(4, 4));     //5
alert(numbers.lastIndexOf(4, 4)); //3
// 会使用全等操作符
var person = { name: "Nicholas" };
var people = [{ name: "Nicholas" }];
var morePeople = [person];
alert(people.indexOf(person));     //-1
alert(morePeople.indexOf(person)); //0 
```

#### 迭代方法
ECMAScript5为数组定义了5个迭代方法。每个方法都接收两个参数：**要在每一项上运行的函数**和（可选的）**运行该函数的作用域对象**。传入这些方法中的函数接收三个参数：**数组项的值**、**该项在数组中的位置**、**数组对象本身**。

* `forEach()`：对数组每一项运行给定函数，没有返回值
* `every()`：对数组每一项运行给定函数，如果该函数对每项都返回`true`，则返回`true`
* `some()`：对数组每一项运行给定函数，如果该函数对其中一项返回`true`，则返回`true`
* `filter()`：对数组每一项运行给定函数，返回该函数会返回`true`的项组成的数组
* `map()`：对数组每一项运行给定函数，返回每次函数调用的结果组成的数组

``` javascript
var numbers = [1,2,3,4,5,4,3,2,1];
var everyResult = numbers.every(function(item, index, array) {
    return (item > 2);
});
alert(everyResult); //false

var mapResult = numbers.map(function(item, index, array) {
    return item * 2;
});
alertr(mapResult); //[2,4,6,8,10,8,6,4,2]
```

#### 归并方法
`reduce()`和`reduceRight()`。这两个方法都会迭代数组的所有项，然后构建一个最终返回的值。
`reduce()`从数组的第一项开始往后遍历。`reduceRight()`从最后一项开始，往前遍历。
两个方法都要接收两个参数：**在每一项上调用的函数**和（可选的）**作为归并基础的初始值**。
里面的函数接收4个参数：**前一个值**、**当前值**、**项的索引**和**数组对象**。这个函数返回的任何值都会作为第一个参数自动传给下一项。第一次迭代发送在数组的第二项上，因此第一个参数是数组的第一项，第二个参数是数组的第二项。
``` javascript
var values = [1, 2, 3, 4, 5];
var sum = values.reduce(function(prev, cur, index, array) {
    return prev + cur;
});
alert(sum); // 15
```

### Date类型
Date类型使用自UTC（Coordinated Universal Time，国际协调时间）1970年1月1日零时开始经过的毫秒数来保存日期。
想根据时间创建日期对象，必须传入毫秒数，为简化这一计算过程，ECMAScript提供两个方法：`Date.parse()`和`Date.UTC()`
`Date.parse()`根据接收的字符串返回相应的毫秒数，如果字符串不能表示日期，会返回`NaN`。可以将字符串直接传递给Date构造函数，会在后台调用`Date.parse()`。
`Date.UTC()`的参数：年份、基于0的月份、天数、小时数（0到23）、分钟、秒、毫秒数
`valueOf()`返回日期的毫秒表示。
``` javascript
// 构造函数不传递参数，自动获得当前日期和时间
var now = new Date();
// 将地区设置为美国的浏览器通常接受下列日期格式：
// "月/日/年"          6/13/2004
// "英文月名 日,年"    January 12,2004
// "英文星期几 英文月名 日 年 时:分:秒 时区"  Tue May 25 2004 00:00:00 GMT-0700
// "ISO 8601扩展格式 YYYY-MM-DDTHH:mm:ss:ssssZ" 2004-05-25T00:00:00
var someDate = new Date(Date.parse("May 25, 2004"));
var someDate = new Date("May 25, 2004");         // parse
var allFives = new Date(2005, 4, 5, 17, 55, 55); // UTC
```

#### 日期格式化方法
Date类型有一些专门用于将日期格式化为字符串的方法
``` javascript
var date = new Date("Sat Aug 11 00:00:00 CST 2018");
alert(date.toDateString());       // Sat Aug 11 2018
alert(date.toTimeString());       // 14:00:00 GMT+0800(中国标准时间)
alert(date.toLocaleDateString()); // 2018/8/11
alert(date.toLocaleTimeString()); // 下午2:00:00
alert(date.toUTCString());        // Sat, 11 Aug 2018 06:00:00 GMT
```

#### 日期/时间组件方法

方法（部分）|说明
-|-
`getTime()`|返回表示日期的毫秒数；与`valueOf()`方法返回的值相同
`getFullYear()`|取得4位数的年份（如2007而非仅07）
`getMonth()`|返回日期中的月份，其中0表示一月，11表示十二月
`getDate()`|返回日期月份中的天数（1到31）
`getDay()`|返回日期中星期的星期几（其中0表示星期日，6表示星期六）
`getHours()`|返回日期中的小时数（0到23）
`getMinutes()`|返回日期中的分钟数（0到59）
`getSeconds()`|返回日期中的秒数（0到59）
`getMilliseconds()`|返回日期中的毫秒数

### RegExp类型
ECMAScript通过RegExp类型来支持正则表达式。
模式（pattern）部分可以是任何简单或复杂的正则表达式，可以包含字符类、限定符、分组、向前查找以及反向引用。每个正则表达式都可带有一或多个标志（flags），用以标明正则表达式的行为。三个标志：

* g：表示全局（global）模式，即模式将被应用于所有字符串，而非再发现第一个匹配项时立即停止
* i：表示不区分大小写（case-insensitive）模式，即在确定匹配项时忽略模式与字符串的大小写
* m：表示多行（multiline）模式，即在到达一行文本末尾时还会继续查找下一行中是否存在与模式匹配的项

``` javascript
// 字面量形式
var expression = / pattern / flags;

var pattern1 = /at/g;       // 匹配字符串中所有"at"的实例
var pattern2 = /[bc]at/i    // 匹配第一个"bat"或"cat"，不区分大小写
var pattern3 = /.at/gi      // 匹配所有以"at"结尾的3个字符的组合，不区分大小写

// 模式中使用的所有元字符都必须转义。元字符包括：( [ { \ ^ $ | ) ? * + .]}
var pattern4 = /\[bc\]at/i; // 匹配第一个"[bc]at"，不区分大小写
var pattern5 = /\.at/gi;    // 匹配所有".at"，不区分大小写

// RegExp构造函数
// 由于RegExp构造函数的模式参数时字符串，所以在某些情况下要对字符进行双重转义。
// 所有元字符都必须双重转义，那些已经转义过的字符也是如此。
var pattern6 = new RegExp("[bc]at", "i"); 
var pattern7 = new RegExp("\\[bc\\]at", "i");
```

### Function类型
函数实际上是对象。每个函数都是Function类型的实例。
``` javascript
// 函数声明
function sum(num1, num2) {
    return num1 + num2;
}
//函数表达式（这种情况下创建的函数叫做匿名函数）
var sum = function(num1, num2) {
    return num1 + num2;
};
// Function构造函数
var sum = new Function("num1", "num2", "return num1 + num2");

// 使用不带圆括号的函数名是访问函数指针，而非调用函数
var anotherSum = sum;
alert(anotherSum(10, 10));
```
解析器会率先读取**函数声明**，并使其在执行任何代码之前可用（可以访问）；至于**函数表达式**，则必须等到解析器执行到它所在的代码行才会真正被解释执行。
``` javascript
// 正常运行
alert(sum(10, 10));
function sum(num1, num2) {
    return num1 + num2;
}
// 错误
alert(sum(10, 10));
var sum = function(num1, num2) {
    return num1 + num2;
}
```

#### 作为值的函数
函数名本身就是变量，所以函数也可以作为值来使用。
``` javascript
function createComparisonFunction(propertyName) {
    return function(object1, object2) {
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];

        if (value1 < value2) {
            return -1;
        } else if (value1 > value2) {
            return 1;
        } else {
            return 0;
        }
    };
}

var data = [{name: "Zachary", age: 28}, {name: "Nicholas", age; 29}];
data.sort(createComparisonFunction("name"));
alert(data[0].name); // Nicholas
alert(date[0].name); // Zachary
```

#### 函数内部属性
在函数内部有两个特殊的对象：`arguments`和`this`。
`arguments`是一个**类数组对象**，包含着传入函数中的所有参数。该对象还有一个`callee`属性，该属性是一个指针，指向拥有这个`arguments`对象的函数。
``` javascript
function factorial(num) {
    if (num <= 1) {
        return 1;
    } else {
        return num * factorial(num - 1);
    }
}
// 等价，重写后的函数体内，没有再引用函数名。无论引用函数时使用什么名字，都可以保证正常完成
function factorial(num) {
    if (num <=1) {
        return 1;
    } else {
        return num * arguments.callee(num - 1);
    }
}
```
`this`代表函数运行时，自动生成的一个内部对象，只能在函数内部使用（当在网页的全局作用域中调用函数时，`this`对象引用的就是`window`）
``` javascript
window.color = "red";
var o = {
    color: "blue"
};
function sayColor() {
    alert(this.color);
}
sayColor();   //"red"
o.sayColor = sayColor;
o.sayColor(); //"blue" 
// 当在全局作用域中调用sayColor()时，this引用的时全局对象window
// 当把这个函数赋给对象o并调用o.sayColor()时，this引用的是对象o
```
`caller`属性保存着调用当前函数的函数的引用，如果在全局作用域中调用当前函数，它的值为`null`

#### 函数属性和方法
`length`属性表示函数希望接受的命名参数的个数。
每个函数继承的`toLocaleString()`、`toString()`和`valueOf()`方法始终都返回函数的代码。
`prototype`属性保存着引用类型所有实例方法的真正所在。`toString()`和`valueOf()`等方法实际上都保存在`prototype`名下,只不过是通过各自对象的实例访问罢了。
`apply()`、`call()`是两个非继承而来的方法。他们的用途是在特定的作用域中调用函数。
`apply()`方法接收两个参数：**函数的作用域**、**参数数组**。参数可以是`Array`实例，也可以是`arguments`对象。
`call()`方法第一个参数没有变化，其余参数是直接传递给函数的。
``` javascript
function sum(num1, num2) {
    return num1 + num2;
}
function callSum1(num1, num2) {
    return sum.apply(this, arguments);
}
function callSum2(num1, num2) {
    return sum.apply(this, [num1, num2]);
}
alert(callSum1(10, 10)); // 20
alert(callSum2(10, 10)); // 20
// 因为是在全局作用域中调用的，所以传入的就是window对象
function callSum(num1, num2) {
    return sum.call(this, num1, num2);
}
alert(callSum(10, 10));  // 20
```
`apply()`和`call()`真正强大的地方是能够扩充函数赖以运行的作用域。
``` javascript
window.color = "red";
var o = { color: "blue"};
function sayColor() {
    alert(this.color);
}
sayColor();           //red
sayColor.call(this);  //red
sayColor.call(window);//red
sayColor.call(o);     //blue
```
ECMAScript 5还定义了一个方法：`bind()`。这个方法会创建一个函数的实例，其`this`值会被绑定到传给`bind()`函数的值。
``` javascript
window.color = "red";
var o = { color: "blue"};
function sayColor() {
    alert(this.color);
}
var objectSayColor = sayColor.bind(o);
objectSayColor();     //blue
```

### 基本包装类型
每当读取一个基本类型值的时候，后台就会创建一个对应的基本包装类型的对象，从而让我们能够调用一些方法来操作这些数据。
``` javascript
var s1 = "some text";
var s2 = s1.substring(2);
```
引用类型与基本包装类型的主要区别就是对象的生存期。使用`new`操作符创建的引用类型的实例，在执行流离开当前作用域之前都一直保存在内存中。而自动创建的基本包装类型的对象，则只存在于一行代码的执行瞬间，然后立即被销毁。

`Object`构造函数也会像工厂方法一样，根据传入值的类型返回相应基本包装类型的实例
``` javascript
var obj = new Object("some text");
alert(obj instanceof String); //true
```

####  Number类型
Number对象的`valueOf()`方法返回对象表示的基本类型的数值，`toLocaleString()`和`toString()`返回字符串形式的数值。
`toFixed()`方法会按照指定的小数位返回数值的字符串表示，能够自动舍入的特性，使得它很适合处理货币值。
``` javascript
var numberObject = new Number(10);

var num = 10;
alert(num.toFixed(2)); //"10.00"
var num = 10.005;
alert(num.toFixed(2)); //"10.01"
```

#### String类型
继承的`valueOf()`、`toLocaleString()`和`toString()`方法都返回对象所表示的基本字符串值。
`length`属性表示字符串中包含多个字符，即使字符串中包含双字节字符（不是占一个字节的ASCII字符），每个字符也仍然算一个字符。
``` javascript
// 字符方法
var stringValue = "hello world";
alert(stringValue.charAt(1));     //"e"
alert(stringValue.charCodeAt(1)); //"101"字符编码
alert(stringValue[1]);            //"e"
// 字符串操作方法
var stringValue = "hello ";
var result = stringValue.concat("world"); //拼接
alert(stringValue); //"hello"
alert(result);      //"hello world"
var result = stringValue.concat("world", "!"); //拼接多个参数
alert(result);      //"hello world!"

var stringValue = "hello world";
alert(stringValue.slice(3));       //"lo world"
alert(stringValue.substring(3));   //"lo world"
alert(stringValue.substr(3));      //"lo world"
alert(stringValue.slice(3, 7));    //"lo w"(开始位置，结束位置)[3, 7)
alert(stringValue.substring(3, 7));//"lo w"(开始位置，结束位置)
alert(stringValue.substr(3, 7));   //"lo worl"(开始位置，返回字符个数)
// 字符串位置方法
var stringValue = "hello world";
alert(stringValue.indexOf("o"));       //4
alert(stringValue.lastIndexOf("o"));   //7
alert(stringValue.indexOf("o", 6));    //7（从哪个位置开始搜索）
alert(stringValue.lastIndexOf("o", 6));//4
// trim()
var stringValue = "  hello world  ";
var trimmedStringValue = stringValue.trim();
alert(stringValue);       //"  hello world  "
alert(trimmedStringValue);//"hello world"
// 字符串大小写转换方法
var stringValue = "hello world";
alert(stringValue.toUperCase());       //"HELLO WORLD"
alert(stringValue.toLowerCase());      //"hello world"
```

### 单体内置对象
由ECMAScript实现提供的、不依赖于宿主环境的对象，这些对象在ECMAScript程序执行之前就已经存在了。
#### Global对象
不属于任何其他对象的属性和方法，最终都是它的属性和方法。所有在全局作用域中定义的属性和函数，都是Global对象的属性。诸如`isNaN()`、`isFinite()`、`parseInt()`
大多数ECMAScript实现中都不能直接访问Global对象；不过Web浏览器实现了承担该角色的window对象。

**URI编码方法**
``` javascript
var uri = "http://www.wrox.com/illegal value.htm#start";
encodeURI(uri);         //http://www.wrox.com/illegal&20value.htm#start
encodeURIComponent(uri);//http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.htm%23start

var uri = "http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.htm%23start";
decodeURI(uri);         //http%3A%2F%2Fwww.wrox.com%2Fillegal value.htm%23start
decodeURIComponent(uri);//http://www.wrox.com/illegal value.htm#start
```
**eval()方法**
`eval()`方法就像是一个完整的ECMAScript解析器，它只接受一个参数，即要执行的ECMAScript（或JavaScript）字符串。
``` javascript
eval("alert('hi')");
// 等价
alert("hi");
```
**window对象**
Web浏览器都是将`Global`这个全局对象作为`window`对象的一部分加以实现的。在全局作用域中声明的所有变量和函数，就都成为了`window`对象的属性。

#### Math对象

## 面向对象的程序设计
ECMA-262把对象定义为：“**无序属性的集合，其属性可以包含基本值、对象或者函数**。”严格来讲，这就相当于说对象是一组没有特定顺序的值。对象的每个属性或方法都有一个名字，而每个名字都映射都一个值。正因为这样，我们可以把ECMAScript的对象想象成散列表：无非就是一组**名值对**，其中值可以是数据或函数。
每个对象都是基于一个**引用类型**创建的，这个引用类型可以是**原生类型**，也可以是**开发人员定义的类型**。

### 理解对象
``` javascript
// 早期的开发人员经常使用这个模式创建新对象
var person = new Object();
person.name = "Nicholas";
person.age = 29;
person.job = "Software Engineer";
person.sayName = function() {
    alert(this.name);
}
// 目前，对象字面量成为创建这种对象的首选模式
var person = {
    name: "Nicholas",
    age: 29,
    job: "Software Engineer",
    sayName: function() {
        alert(this.name);
    }
}
```

#### 属性的类型和特性
ECMAScript中有两种属性：**数据属性**和**访问器属性**
对象的属性在创建时带有一些**特性**（`characteristic`），JavaScript通过这些特性来定义它们的行为。
> 为了表示特性是内部值，规范（ECMA-262）就把它们放到了两对儿方括号里了，例如\[[Enumerable]]。

**数据属性**
一般用于存储数据数值，有4个描述其行为的特性

数据描述符特性|说明|默认
-|-|-
\[[Value]]|属性的当前值。读取/写入属性值都在这个位置|`undefined`
\[[Writable]]|`true`或`false`。能否修改属性值|`false`
\[[Enumerable]]|`true`或`false`。能否由`for-in`语句枚举属性|`false`
\[[Configurable]]|`true`或`false`。能否更改属性的特性、删除属性|`false`

`Object.defineProperty()`方法用于修改属性默认的特性。这个方法接收三个参数：

1. 属性所在的对象
2. 需定义或修改的属性名
3. 包含设置特性的对象

``` javascript
var person = {};
Object.defineProperty(person, "name", {
    writable: false,
    value: "Nicholas"
});
alert(person.name); //"Nicholas"
person.name = "Greg";
alert(person.name); //"Nicholas"

var person = {};
Object.defineProperty(person, "name", {
    configurable: false, //一旦把属性定义为不可设置的，就不能再把它变回可设置
    value: "Nicholas"
});
alert(person.name); //"Nicholas"
delete person.name;
alert(person.name); //"Nicholas"
// 可以多次调用Object.defineProperty()方法修改同一个属性，但在把configurable特性设置为false之后就会有限制了
```
对于直接在对象上定义的属性，它们的`Configurable`、`Enumerable`和`Writable`特性都被设置为`true`
在调用`Object.defineProperty()`时，如果不指定，`configurable`、`enumerable`和`writable`特性的默认值都是`false`

**访问器属性**
访问器属性包含一对`getter`和`setter`函数（非必需）。读取访问器属性会调用`getter`函数；写入访问器属性会调用`setter`函数。只指定`getter`意味着属性是不能写，只指定`setter`意味着属性不能读。访问器属性有如下4个特性

访问器描述符特性|说明|默认
-|-|-
\[[Get]]|返回属性值的函数|`undefined`
\[[Set]]|设置属性值的函数|`undefined`
\[[Enumerable]]|`true`或`false`。能否由`for-in`语句枚举属性|`false`
\[[Configurable]]|`true`或`false`。能否更改属性的特性、删除属性|`false`

``` javascript
// 访问器属性必须使用Object.defineProperty()来定义
// _year前面的下划线是一种常用的记号，用于表示只能通过对象方法访问的属性(理论上可以直接访问，但是这样访问器就没有意义了)
var book = {
    _year: 2004,
    edition: 1
};
Object.defineProperty(book, "year", {
    get: function() {
        reutrn this._year;
    },
    set: function(newValue) {
        if (newValue > 2004) {
            this._year = newValue;
            this.edition += newValue - 2004;
        }
    }
});
book.year = 2005;
alert(book.edition); //2
```

#### 读取属性的特性
`Object.defineProperties()`可以批量为属性设置特性
`Object.getOwnPropertyDescriptor()`可以取得属性的特性
``` javascript
var book = {};

Object.defineProperties(book, {
    _year: {
        value: 2004
    },
    edition: {
        value: 1
    },
    year: {
        get: function() {
            return this._year;
        },
        set: function(newValue) {
            if (newValue > 2004) {
                this._year = newValue;
                this.edition += newValue - 2004;
            }
        }
    }
});
var descriptor = Object.getOwnPropertyDescriptor(book, "_year");
alert(descriptor.value);       //2004
alert(descriptor.configurable);//false
alert(typeof descriptor.get);  //"undefined"

var descriptor = Object.getOwnPropertyDescriptor(book, "year");
alert(descriptor.value);       //2004
alert(descriptor.enumerable);  //false
alert(typeof descriptor.get);  //"function"

```













4. 如果在调用普通函数语句前加上new，那么就变成了构造函数，一般函数首字母大写。
6. 根据 W3C 的 HTML DOM 标准，HTML 文档中的所有内容都是节点（JavaScript中作为Node类型实现）：
    * 整个文档是一个文档节点
    * 每个 HTML 元素是元素节点
    * HTML 元素内的文本是文本节点
    * 每个 HTML 属性是属性节点
    * 注释是注释节点
7. Node属性：
    * 基本：*nodeType、nodeName、nodeValue*
    * 关系型：*childNodes、parentNode、previousSibling、nextSibling、firstChild、lastChild、ownerDocument*
8. Node方法：
    * *appendChild()、insertBefore()、replaceChild()、removeChild()、cloneNode()、normalize()*
9. JavaScript中的所有节点类型都继承自Node类型。
    * Document类型：
属性：*documentElement、document.body、document.doctype*
方法：*getElementById()、getElementByTagName()、getElementByName()、write()*
    * Element类型：
属性：*id、title、lang、dir、className、attributes*
方法：*getAttribute()、setAttribute()、removeAttribute()*
    * Attr类型：
属性：*name、value、specified*