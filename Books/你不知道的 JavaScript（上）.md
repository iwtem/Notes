## 你不知道的 JavaScript（上）

### 1. 作用域

#### 1.1 编译原理

在传统编译语言的流程中，程序中的一段源代码在执行之前会经历三个步骤，统称为“编 译”。

- **分词/词法分析(Tokenizing/Lexing)**

这个过程会将由字符组成的字符串分解成有意义的代码块，这些代码块被称为词法单元(token)。例如，考虑程序 `var a = 2;`。这段程序通常会被分解成为下面这些词法单元: `var`、`a`、`=`、`2`、`;`。空格是否会被当作词法单元，取决于空格在这门语言中是否具有意义。

- **解析/语法分析(Parsing)**

这个过程是将词法单元流(数组)转换成一个由元素逐级嵌套所组成的代表了程序语法结构的树。这个树被称为“抽象语法树”(Abstract Syntax Tree，AST)。

`var a = 2;` 的抽象语法树中可能会有一个叫作 VariableDeclaration 的顶级节点，接下来是一个叫作 Identifier(它的值是 a)的子节点，以及一个叫作 AssignmentExpression 的子节点。AssignmentExpression 节点有一个叫作 NumericLiteral(它的值是 2)的子 节点。

```json
{
  "type": "VariableDeclaration",
  "kind": "var",
  "declarations": [
    {
      "type": "VariableDeclarator",
      "id": {
        "type": "Identifier",
        "name": "a"
      },
      "init": {
        "type": "Literal",
        "value": 2
      }
    }
  ]
}
```

- **代码生成**

将 AST 转换为可执行代码的过程称被称为代码生成。这个过程与语言、目标平台等息息相关。抛开具体细节，简单来说就是有某种方法可以将 `var a = 2;` 的 AST 转化为一组机器指令，用来创建一个叫作 a 的变量(包括分配内存等)，并将一个值储存在 a 中。



> 对于 JavaScript 引擎，不会有大量的(像其他语言编译器那么多的)时间用来进行优化，因为与其他语言不同，JavaScript 的编译过程不是发生在构建之前的。对于 JavaScript 来说，大部分情况下编译发生在代码执行前的几微秒(甚至更短!)的时间内。在我们所要讨论的作用域背后，JavaScript 引擎用尽了各种办法(比如 JIT，可以延 迟编译甚至实施重编译)来保证性能最佳。
>
> 简单地说，任何 JavaScript 代码片段在**执行前都要进行编译**(通常就在执行前)。因此， JavaScript 编译器首先会对 `var a = 2;` 这段程序进行编译，然后做好执行它的准备，并且通常马上就会执行它。

