# You Don't Know JS Yet: 作用域和闭包 - 第二版
# 第一章：什么是作用域？

当你刚开始编写程序时，你就会很自然地喜欢创建变量并把值存在变量上。使用变量是我们编程中最基础的事情之一！

但是你可能没有非常仔细地考虑过引擎组织和管理这些变量的底层机制。我不是指计算机是如果分配内存的，而是：JS如何知晓在给定的语句中有哪些变量可以访问，以及如何处理两个具有相同名称的变量。

此类问题的答案需要一组明确定义的规则叫做作用域。本书将深入探讨作用域的各个方面——工作原理、作用、应避免的陷阱——然后直指影响程序结构的通用作用域模式。

我们的第一步是揭示JS引擎在运行程序**之前**对它做了哪些处理。

## 关于这本书

Welcome to book 2 in the *You Don't Know JS Yet* series! If you already finished *Get Started* (the first book), you're in the right spot! If not, before you proceed I encourage you to *start there* for the best foundation.

我们的重点将是JS语言的三大支柱之首：作用域系统、其函数闭包、模块设计模式的强大功能。

JS通常被归类为一种解释型脚本语言，因此大多数人都认为JS程序是通过单次地、从上至下地处理的。但实际上，**在执行开始之前**，JS会在单独的阶段中进行解析/编译。在这个阶段，引擎将根据作用域规则，分析代码作者对变量、函数、块如何相互‘respect’地放置的设计。生成的作用域结构通常不受运行时条件的影响。

JS函数本身就是 ‘first-class’ 值；它们可以像数值或字符串一样被赋值、传递。但这些函数可以持有和接收变量，无论在程序的什么位置执行都保持原本的作用域。这称为闭包。

模块是一种代码组织模式，其特点是，使用公共方法（通过闭包）访问模块作用域内的变量和函数。

## 编译型 vs. 解释型

您可能以前听说过*代码编译*，但它可能看起来像一个神秘的黑盒子，源码从一端进入然后从另一端输出可执行程序。

其实它并不神秘或神奇。代码编译就是处理代码文本并转换为计算机可理解的指令列表的一组步骤。通常，整个源码会一起进行转换，然后将得到的指令另存为输出（通常存储在文件中），以便之后执行。

您可能还听说过代码可以*解释*，那么它和*编译*有何不同？

解释所执行的任务与编译类似，也就是将你们的程序转换为机器可理解的指令。但是它们的处理模型不同。不像上述的一次性编译完程序，解释是逐行转换源码的；在继续处理下一行源码之前，就会先执行已解释的行或语句。

<figure>
    <img src="images/fig1.png" width="650" alt="Code Compilation and Code Interpretation" align="center">
    <figcaption><em>Fig. 1: Compiled vs. Interpreted Code</em></figcaption>
    <br><br>
</figure>

Figure 1 illustrates compilation vs. interpretation of programs.

这两个处理模型是否互斥？一般来说，确实如此。但是，这个问题又很微妙，因为实际上解释可以采取其他形式，而不仅仅是对源码文本的逐行操作。实际上，现代JS引擎在处理JS程序时就会采用大量包括编译和解释的变体。

回想一下，在《入门》一书的第一章中，我们评述过这个话题。我们的结论是，将JS描述为一种**编译型语言**是最准确的。为了读者的‘benefit’，以下各节将重新讨论并扩展此主张。

## 编译代码

但是首先，为什么辩证JS是否为编译型会很重要呢？

作用域主要是在编译期间确定的，所以理解编译和执行之间的关系是掌握作用域的关键。

在经典的编译器理论中，编译器以以下三个基本阶段处理程序：

1. **分词/词法分析：** 将一连串字符打断成（对于语言来说）有意义的片段，称为 token（记号）。举例来说，考虑这段程序：`var a = 2;`。这段程序很可能会被打断成这些 token： `var`, `a`, `=`, `2` 和 `;`。空格可能会被保留为一个 token，也可能不会，这取决于它是否有意义。

    （分词和词法分析之间的区别是微妙和学术上的，其核心在于这些 token 是否以 *无状态* 或 *有状态* 的方式被识别。简而言之，如果分词器需要调用有状态的解析规则来确定`a`是否应当被考虑为一个独立的 token，还是只是其他 token 的一部分，那么*这*就是 **词法分析**。）

2. **解析：** 将一个 token 的流（数组）转换为一个嵌套元素的树，它综合地表示了程序的语法结构。这棵树称为“抽象语法树”（AST —— **A**bstract **S**yntax **T**ree）。

    例如，`var a = 2;` 的树也许开始于称为 `VariableDeclaration`（变量声明）顶层节点，带有一个称为 `Identifier`（标识符）的子节点（它的值为 `a`），和另一个称为 `AssignmentExpression`（赋值表达式）的子节点，而这个子节点本身带有一个称为 `NumericLiteral`（数字字面量）的子节点（它的值为`2`）。

3. **代码生成：** 将抽象语法树转换为可执行的代码。这部分内容会根据语言，目标平台和其他因素有很大差异。

    JS引擎将刚刚描述的 `var a = 2;` 的AST转换为机器指令，可实际 *创建* 一个称为 `a` 的变量（包括分配内存等等），然后在 `a` 中存入一个值。

| 注意： |
| :--- |
| JS引擎的实现细节（利用系统内存资源等）比我们在这里探讨的要深得多。我们将持续关注我们程序可观察的行为，由JS引擎来管理这些更深的系统层的抽象。 |

JS引擎要比这*区区*三步复杂太多了。例如，在解析和代码生成的处理中，存在优化执行效率的步骤（如压缩冗余元素）。实际上，代码甚至可以在执行过程中被重新编译和重新优化。

所以，我在此描绘的只是大框架。但是你很快就会明白为什么我们*要*提及的*这些*细节是重要的，虽然是在较高的层次上。

JS引擎没有大把的时间来执行它的工作和优化，因为JS的编译和其他语言不同，不是发生在一个提前的构建步骤中。它通常必须发生在执行代码前的仅仅几微秒之内（或更少！）。为了确保在这些约束条件下的最快性能，JS引擎使用了各种技巧（例如JIT，它可以懒编译甚至是热编译）；而这远超出了我们此处讨论的“作用域”。

### Required: 两个阶段

为了尽可能简单地说明，关于JS程序的处理，我们能做的最重要的观察是它发生在（至少）两个阶段：首先是解析/编译，然后是执行。

解析/编译阶段与后续执行阶段的分离是可以观察到的事实，而不是理论或观点。尽管JS规范没有明确要求“编译”，但它要求的行为实质上只能使用“编译-然后-执行”方法才能实现。

你可以观察这三个程序特性来证明这一点：语法错误、Early Errors和声明提升。

#### 从一开始就抛出语法错误

思考以下程序：

```js
var greeting = "Hello";

console.log(greeting);

greeting = ."Hi";
// SyntaxError: unexpected token .
```

这个程序不会有任何输出（不会打印`"Hello"`），但是会抛出一个关于`"Hi"`字符串前预料外的`.`token的`SyntaxError`异常。由于语法错误发生在格式良好的`console.log(..)`语句之后，因此，如果JS是从上至下逐行执行的，就可以期望在抛出语法错误之前先打印`"Hello"`消息。这并没有发生。

实际上，要JS引擎能在执行第一和第二行之前知道第三行有语法错误，唯一的方法就是JS引擎在执行任何程序之前先解析整个程序。

#### Early Errors

接下来，请思考：

```js
console.log("Howdy");

saySomething("Hello","Hi");
// Uncaught SyntaxError: Duplicate parameter name not
// allowed in this context

function saySomething(greeting,greeting) {
    "use strict";
    console.log(greeting);
}
```

尽管格式正确，但`"Howdy"`并未打印。

相反，就像上一节中的代码片段一样，在程序执行之前抛出来`SyntaxError`错误。在这个案例中，是因为严格模式（这里只在`saySomething(..)`函数中启用）禁止函数拥有重复的参数名；这在非严格模式下通常是允许的。

这里抛出的错误不是因为令牌的字符串格式不正确（像前面的`."Hi"`）的语法错误。但是规范仍然要求在任何执行开始之前抛出严格模式下的“early error”。

但是JS引擎又要如何得知`greeting`参数已经重复了呢？它又如何在处理参数列表时得知`saySomething(..)`函数出于严格模式（`"use strict"`出现在函数体内，参数之后）？

同样的，唯一可解释的理由就是代码一定是在执行发生之前被*完整地*解析了。

#### 声明提升

最后，请思考：

```js
function saySomething() {
    var greeting = "Hello";
    {
        greeting = "Howdy";  // error comes from here
        let greeting = "Hi";
        console.log(greeting);
    }
}

saySomething();
// ReferenceError: Cannot access 'greeting' before
// initialization
```

注释中的`ReferenceError`发生在有`greeting = "Howdy"`语句的那行。这里的情况是，这条语句中的`greeting`变量不属性前面的`var greeting = "Hello"`语句，而是属于下一行`let greeting = "Hi"`声明语句。

JS引擎可以在抛出错误的那一行知道，*下一条语句*将声明一个具有相同名称的块作用域变量（`greeting`），这唯一的途径不就是JS引擎已经在较早的过程中处理过此代码，并且已经建立了所有作用域及其变量的关联。这种作用域和声明的处理只能通过执行前的程序解析来准确地完成。

从技术上讲，这里的`ReferenceError`是因为`greeting = "Howdy"`**过早**地访问`greeting`变量，这是与称为临时死区（TDZ）的冲突。第5章将对此进行更详细的介绍。

| 警告： |
| :--- |
| 经常有人断言`let`和`const`声明没有被提升，作为对TDZ行为的一种解释。但这是不准确的。我们将在第5章中解释声明提升和`let`/`const`的TDZ。 |

希望现在你已经确信，JS程序在开始执行之前，已经解析过了。但这是否能证明它们已经被编译了呢？

这是一个有趣问题，值得思考。JS会在解析一个程序之后，**不**先编译程序，而是通过理解AST中的操作来执行代码吗？确实，有这种可能性。但这非常不可能，主要的因为是，这将会有极低的性能表现。

很难想象，一个生产品质的JS引擎克服所有解析程序的困难得到了AST，但是之后又不把AST转换（也就是“编译”）成效率最高的（二进制）表达形式，来让引擎做后续的执行。

很多人就大量的细微差别试图钻这个术语的牛角尖，然后到处是“well, actually...”的感叹词。但是本质上以及事实上，引擎处理JS的过程确实**更像是编译**。

Classifying JS as a compiled language is not concerned with the distribution model for its binary (or byte-code) executable representations, but rather in keeping a clear distinction in our minds about the phase where JS code is processed and analyzed; this phase observably and indisputedly happens *before* the code starts to be executed.

We need proper mental models of how the JS engine treats our code if we want to understand JS and scope effectively.

## Compiler Speak

With awareness of the two-phase processing of a JS program (compile, then execute), let's turn our attention to how the JS engine identifies variables and determines the scopes of a program as it is compiled.

First, let's examine a simple JS program to use for analysis over the next several chapters:

```js
var students = [
    { id: 14, name: "Kyle" },
    { id: 73, name: "Suzy" },
    { id: 112, name: "Frank" },
    { id: 6, name: "Sarah" }
];

function getStudentName(studentID) {
    for (let student of students) {
        if (student.id == studentID) {
            return student.name;
        }
    }
}

var nextStudent = getStudentName(73);

console.log(nextStudent);
// Suzy
```

Other than declarations, all occurrences of variables/identifiers in a program serve in one of two "roles": either they're the *target* of an assignment or they're the *source* of a value.

(When I first learned compiler theory while earning my computer science degree, we were taught the terms "LHS" (aka, *target*) and "RHS" (aka, *source*) for these roles, respectively. As you might guess from the "L" and the "R", the acronyms mean "Left-Hand Side" and "Right-Hand Side", as in left and right sides of an `=` assignment operator. However, assignment targets and sources don't always literally appear on the left or right of an `=`, so it's probably clearer to think in terms of *target* / *source* rather than *left* / *right*.)

How do you know if a variable is a *target*? Check if there is a value that is being assigned to it; if so, it's a *target*. If not, then the variable is a *source*.

For the JS engine to properly handle a program's variables, it must first label each occurrence of a variable as *target* or *source*. We'll dig in now to how each role is determined.

### Targets

What makes a variable a *target*? Consider:

```js
students = [ // ..
```

This statement is clearly an assignment operation; remember, the `var students` part is handled entirely as a declaration at compile time, and is thus irrelevant during execution; we left it out for clarity and focus. Same with the `nextStudent = getStudentName(73)` statement.

But there are three other *target* assignment operations in the code that are perhaps less obvious. One of them:

```js
for (let student of students) {
```

That statement assigns a value to `student` for each iteration of the loop. Another *target* reference:

```js
getStudentName(73)
```

But how is that an assignment to a *target*? Look closely: the argument `73` is assigned to the parameter `studentID`.

And there's one last (subtle) *target* reference in our program. Can you spot it?

..

..

..

Did you identify this one?

```js
function getStudentName(studentID) {
```

A `function` declaration is a special case of a *target* reference. You can think of it sort of like `var getStudentName = function(studentID)`, but that's not exactly accurate. An identifier `getStudentName` is declared (at compile time), but the `= function(studentID)` part is also handled at compilation; the association between `getStudentName` and the function is automatically set up at the beginning of the scope rather than waiting for an `=` assignment statement to be executed.

| NOTE: |
| :--- |
| This automatic association of function and variable is referred to as "function hoisting", and is covered in detail in Chapter 5. |

### Sources

So we've identified all five *target* references in the program. The other variable references must then be *source* references (because that's the only other option!).

In `for (let student of students)`, we said that `student` is a *target*, but `students` is a *source* reference. In the statement `if (student.id == studentID)`, both `student` and `studentID` are *source* references. `student` is also a *source* reference in `return student.name`.

In `getStudentName(73)`, `getStudentName` is a *source* reference (which we hope resolves to a function reference value). In `console.log(nextStudent)`, `console` is a *source* reference, as is `nextStudent`.

| NOTE: |
| :--- |
| In case you were wondering, `id`, `name`, and `log` are all properties, not variable references. |

What's the practical importance of understanding *targets* vs. *sources*? In Chapter 2, we'll revisit this topic and cover how a variable's role impacts its lookup (specifically, if the lookup fails).

## Cheating: Runtime Scope Modifications

It should be clear by now that scope is determined as the program is compiled, and should not generally be affected by runtime conditions. However, in non-strict-mode, there are technically still two ways to cheat this rule, modifying a program's scopes during runtime.

Neither of these techniques *should* be used—they're both dangerous and confusing, and you should be using strict-mode (where they're disallowed) anyway. But it's important to be aware of them in case you run across them in some programs.

The `eval(..)` function receives a string of code to compile and execute on the fly during the program runtime. If that string of code has a `var` or `function` declaration in it, those declarations will modify the current scope that the `eval(..)` is currently executing in:

```js
function badIdea() {
    eval("var oops = 'Ugh!';");
    console.log(oops);
}
badIdea();   // Ugh!
```

If the `eval(..)` had not been present, the `oops` variable in `console.log(oops)` would not exist, and would throw a `ReferenceError`. But `eval(..)` modifies the scope of the `badIdea()` function at runtime. This is bad for many reasons, including the performance hit of modifying the already compiled and optimized scope, every time `badIdea()` runs.

The second cheat is the `with` keyword, which essentially dynamically turns an object into a local scope—its properties are treated as identifiers in that new scope's block:

```js
var badIdea = { oops: "Ugh!" };

with (badIdea) {
    console.log(oops);   // Ugh!
}
```

The global scope was not modified here, but `badIdea` was turned into a scope at runtime rather than compile time, and its property `oops` becomes a variable in that scope. Again, this is a terrible idea, for performance and readability reasons.

At all costs, avoid `eval(..)` (at least, `eval(..)` creating declarations) and `with`. Again, neither of these cheats is available in strict-mode, so if you just use strict-mode (you should!) then the temptation goes away!

## Lexical Scope

We've demonstrated that JS's scope is determined at compile time; the term for this kind of scope is "lexical scope". "Lexical" is associated with the "lexing" stage of compilation, as discussed earlier in this chapter.

To narrow this chapter down to a useful conclusion, the key idea of "lexical scope" is that it's controlled entirely by the placement of functions, blocks, and variable declarations, in relation to one another.

If you place a variable declaration inside a function, the compiler handles this declaration as it's parsing the function, and associates that declaration with the function's scope. If a variable is block-scope declared (`let` / `const`), then it's associated with the nearest enclosing `{ .. }` block, rather than its enclosing function (as with `var`).

Furthermore, a reference (*target* or *source* role) for a variable must be resolved as coming from one of the scopes that are *lexically available* to it; otherwise the variable is said to be "undeclared" (which usually results in an error!). If the variable is not declared in the current scope, the next outer/enclosing scope will be consulted. This process of stepping out one level of scope nesting continues until either a matching variable declaration can be found, or the global scope is reached and there's nowhere else to go.

It's important to note that compilation doesn't actually *do anything* in terms of reserving memory for scopes and variables. None of the program has been executed yet.

Instead, compilation creates a map of all the lexical scopes that lays out what the program will need while it executes. You can think of this plan as inserted code for use at runtime, which defines all the scopes (aka, "lexical environments") and registers all the identifiers (variables) for each scope.

In other words, while scopes are identified during compilation, they're not actually created until runtime, each time a scope needs to run. In the next chapter, we'll sketch out the conceptual foundations for lexical scope.
