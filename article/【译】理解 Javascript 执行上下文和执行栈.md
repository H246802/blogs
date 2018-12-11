# 【译】理解JavaScript执行上下文和执行栈

> - 原文地址：[Understanding Execution Context and Execution Stack in Javascript](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0)
> - 译文地址：[理解JavaScript执行上下文和执行栈](https://github.com/h246802/blogs/issues/1)


如果你是一名 JavaScript 开发者，或者想要成为一名 JavaScript 开发者，那么你必须知道 JavaScript 程序内部的执行机制。理解执行上下文和执行栈同样有助于理解其他的 JavaScript 概念如变量提升机制、作用域和闭包等。

正确理解执行上下文和执行栈的概念将有助于你成为一名更好的 JavaScript 开发人员。

所以不用多说了，让我们开始吧！

## 1 什么是执行上下文（ Execution Context）

简单来说，执行上下文就是当前JavaScript代码被解析和执行时所在环境的抽象概念，JavaScript中运行的所有的代码都是在执行上下文中运行。

### 1.1 执行上下文的类型

执行上下文总共有三种类型

- **全局执行上下文：** 这是默认的、最基础的执行上下文。对于不在任何函数中的代码都位于全局执行上下文中。它做了两件事
   1. 创建一个全局对象，在浏览器中这个全局对象就是window对象
   2. 将`this`指针指向这个全局对象。
    
   一个对象只能存在一个全局执行上下文
   
- **函数执行上下文：** 每次调用函数时，都会为该函数创建一个新的执行上下文。每个函数都拥有自己的执行上下文，但是只有在函数被调用的时候才会被创建。一个程序可以存在任意数量的函数执行上下文。每当一个新的执行上下文被创建，它都会按照特定的顺序执行一系列步骤
  
- **Eval函数执行上下文：** 运行在 `eval` 函数中的代码也获得了自己的执行上下文，但由于JavaScript开发人员通常不使用`eval`，因此我不在文中讨论

## 2 执行栈（Execution Stack）

执行栈，在其他语言中也会称之为调用栈，具有LIFO（后进先出）结构，用于存储代码执行期间创建的所有执行上下文

每当 **JavaScript** 引擎首次读取你的JavaScript代码时，它会创建一个全局执行上下文并将其推入当前的执行栈。每当JavaScript引擎找到一个函数调用，引擎都会为该函数创建一个新的执行上下文并将其推到当前执行栈的顶部

引擎会先运行其执行上下文在执行栈顶端的函数，当此函数运行完成后，其对应的执行上下文将会从执行栈中弹出，上下文控制权将移到当前执行栈的下一个执行上下文

## 3 执行上下文是如何被创建的
到目前为止，我们已经看到了 JavaScript 引擎如何管理执行上下文，现在就让我们来理解 JavaScript 引擎是如何创建执行上下文的。

执行上下文分成两个阶段：1）创建阶段； 2）执行阶段

让我们通过下面的代码示例来理解这一点：

```js
let a = 'Hello World!';
function first() {
  console.log('Inside first function');
  second();
  console.log('Again inside first function');
}
function second() {
  console.log('Inside second function');
}
first();
console.log('Inside Global Execution Context');
```
![image](https://user-images.githubusercontent.com/22425699/49794269-2691f980-fd72-11e8-874b-d5d7cbc0cb17.png)
 <p align="center"  style="color: #666;">上述代码的执行上下文堆栈。</p>

### 3.1 执行上下文创建阶段

在任意的 JavaScript 代码被执行前，执行上下文处于创建阶段。在创建阶段总共发生了三件事

1. 确定 **this** 的值，也称之为 **This Binding**。
2. **LexicalEnvironment（词法环境）** 组件被创建。
3. **VariableEnvironment（变量环境）** 组件被创建。

因此执行上下文可以在概念上表示为：

```
ExecutionContext = {
    ThisBinding = <this value>,
    LexicalEnvironment = {...},
    VariableEnvironment = {...},
}
```

#### 3.1.1 This Binding

在全局执行上下文中，`this` 的值指向全局对象，在浏览器中，`this` 的值指向 window对象。

在函数执行上下文中，`this` 的值取决于函数的调用方式。如果它被一个对象引用调用，那么 `this` 的值被设置为该对象（其实根本还是看 call）

#### 3.1.2 Lexical Environment（词法环境）

`官方ES6` 文档将词法环境定义为：

> 词法环境是一种规范类型,基于 ECMAScript 代码的词法嵌套结构来定义标识符与特定变量和函数的关联关系。词法环境由环境记录（environment record）和可能给为空引用（null）的外部词法环境组成

简而言之，词法环境是一个包含**标识符变量映射**的结构。（这里的**标识符**表示变量/函数的名称，变量是对实际对象【包括函数类型对象】或原始值的引用）

#### 3.1.2.1 词法环境组成部分 

在词法环境中，有两个组成部分：**（1）环境记录（environment record） （2）对外部词法环境的引用**

1. **环境记录**是存储变量和函数声明的实际位置。
2. **对外部环境的引用**意味着它可以访问外部词法环境。

**词法环境**有两种类型：

- **全局环境**（在全局执行上下文中）是一个外部环境的词法环境。全局环境的外部环境引用为 **null**。它使用一个全局对象（window对象）及其关联的方法和属性（例如数组方法）以及任何用户自定义的全局变量，`this` 的值指向这个全局对象。
- **函数环境**，用户在函数中定义的变量被存储在`环境记录`中。对外部词法环境的引用可以是全局，也可以是包含内部函数的外部函数环境。

**注意：** 对于**函数环境**而言，**环境记录** 还包含了一个 `arguments` 对象，该对象包含了索引和传递给函数的参数之间的映射以及传递给函数的参数的**长度（数量）**。

**环境记录（environment record）**同样有两种类型（如下所示）

- **声明性环境记录** 存储变量、函数和参数。一个函数词法环境包含声明性环境记录。
- **对象环境记录** 用于定义在全局执行上下文中出现的变量和函数的关联。全局词法环境包含对象环境记录。

抽象来讲，我们可以写出下面的伪代码来表示不同类型下的词法环境

```
// 全局执行上下文
GlobalExecutionContext = { 
    LexicalEnvironment = {  // 词法环境
        EnvironmentRecord:{  // 环境记录
            Type:'Object',   // 对象型环境记录
            // 包括环境记录的变量、函数
        }
        outer:<null>  // 对外部词法环境的引用
    }
}

// 函数执行上下文
FunctionExecutionContext = {
    LexicalEnvironment = {  // 词法环境
        EnvironmentRecord:{  // 环境记录
            Type:'Declarative'  // 声明性环境记录
            // 包括环境记录的变量、函数、参数
        },
        // 对外部词法环境的引用
        outer:<Global or outer function lexical environment reference>
    }
}
```

### 3.1.3 对象环境(variable environment)

它也是一个词法环境，其`EnvironmentRecord`包含了由 `VariableStatements（变量声明）`在此执行上下文创建的绑定。

如上所述，变量环境也是一个词法环境，因此它具有上面定义的词法环境的所有属性。

在ES6中，**LexicalEnvironment** 和 **VariableEnvironment** 组件的区别在于前者用于存储函数声明和变量（`let`和`const`）的绑定，后者仅仅用于存储变量（`var`）绑定

结合实际代码可以更清晰理解概念：

```js
let a = 20;  
const b = 30;  
var c;

function multiply(e, f) {  
 var g = 20;  
 return e * f * g;  
}

c = multiply(20, 30);
```

伪代码的执行上下文如下所示：

```

// 全局执行上下文 
GlobalExecutionContext = {
    ThisBinding:<Global Object>,
    LexicalEnvironment:{
        EnvironmentRecord:{
            Type:'Object',
            a:<uninitialized>,  // a变量uninitialized(未初始化),
            b:<uninitialized>,  // b变量uninitialized(未初始化)
            multiply:<func...>  // 
        },
        outer:<null>  // 引用外部词法环境
    },
    VariableEnvironment:{
        EnvironmentRecord:{
            Type:'Object',
            c:<undefined>  // c变量在执行上下文期间先定义为undefined
        },
        outer:<null>  // 引用外部词法环境
    }
}

// 函数执行上下文
GlobalExecutionContext = {
    ThisBinding:<Global Object>,  // 根据调用确定
    LexicalEnvironment:{
        EnvironmentRecord:{
            Type:'Declarative',  // 声明型词法环境
            Arguments:{0:20,1:30,length:2} 
        },
        outer:<GlobalLexicalEnvironment>  // 引用全局词法环境
    },
    VariableEnvironment:{
        EnvironmentRecord:{
            Type:'Declarative',
            g:<undefined>  // g变量在执行上下文期间先定义为undefined
        },
        outer:<GlobalLexicalEnvironment>  // 引用外部词法环境
    }
}
```

**注意：**函数的执行上下文的创建是在函数调用的时候

你可能已经注意到了 `let` 和 `const` 定义的变量没有任何与之关联的值，此时为未初始化状态，但 `var` 定义的变量设置为`undefined`。

这是因为在创建阶段，代码会被扫描并解析变量和函数声明，其中函数声明存储在环境中，而变量会被设置为`undefined`（在 `var` 的情况下）或保持未初始化（在`let` 和 `const` 的情况下）。

这就是为什么你可以在声明之前访问`var` 定义的变量（尽管是 `undefined` ），但如果在声明之前访问 `let` 和 `const` 定义的变量就会提示引用错误的原因。

这就是我们所谓的变量提升。

## 4.执行阶段

这是整篇文章中最简单的部分。在此阶段，完成对所有变量的分配，最后执行代码。

**注：** 在执行阶段，如果 Javascript 引擎在源代码中声明的实际位置找不到 let 变量的值，那么将为其分配 undefined 值。

## 5.总结
 
我们已经讨论了 JavaScript 内部是如何执行的。虽然你没有必要学习这些所有的概念从而成为一名出色的 JavaScript 开发人员，但对上述概念的理解将有助于你更轻松、更深入地理解其他概念，如提升、域和闭包等。
