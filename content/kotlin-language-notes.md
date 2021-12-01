+++
title = "Kotlin 学习笔记"
date = "2021-12-01"

[taxonomies]
categories = ["blog"]
tags = ["Kotlin", "Programming", "Language"]
+++

学习 Kotlin，简要记录一些语言细节以供备忘。写测试代码用的 Kotlin 语言版本为 1.5。

### 基本语法

#### 字面量

为了方便理解，下面这个精简后的语法清单里混用了 EBNF 和正则来表达 Kotlin 的字面量规则。想看正式的定义请参考 [Kotlin 语言规范](https://kotlinlang.org/spec/syntax-and-grammar.html)。

```ebnf
# null 字面量
NullLiteral      := 'null'

# 布尔值字面量
BooleanLiteral   := 'true'
                  | 'false'

# 连续的数字之间允许插入下划线来增加可读性，允许使用十进制、十六进制、二进制
# 注意，Kotlin 不允许写八进制整数字面量（比如 0755）
IntegerLiteral   := /([1-9][_0-9]*)?[0-9]/
                  | /0[xX]([0-9a-fA-F][_0-9a-fA-F]*)?[0-9a-fA-F]/
                  | /0[bB]([01][_01]*)?[01]/

# 不加后缀 'f' 或 'F' 默认双精度浮点数，浮点数字面量同样允许插入下划线来增加可读性
DoubleLiteral    := _DecimalDigits? '.' _DecimalDigits ( /[eE][-+]?/ _DecimalDigits )?
                  | _DecimalDigits ( /[eE][-+]?/ _DecimalDigits )?
_DecimalDigits   := /([0-9][_0-9]*)?[0-9]/

# Kotlin 的长整型字面量后缀只能用大写 'L'，可以避免小写 'l' 和 数字 '1' 引起视觉上的歧义
LongLiteral      := IntegerLiteral 'L'

# Kotlin 1.5 以后支持无符号整型，Java 并不直接支持无符号整型
UnsignedLiteral  := IntegerLiteral /[uU]L?/

# 单精度浮点数字面量，后缀是小写 'f' 或大写 'F'
FloatLiteral     := DoubleLiteral /[fF]/

# 字符字面量用单引号，注意 Kotlin 的转义不支持八进制 \ooo 和单个字节 \xXX
CharacterLiteral := /'(\\[rntb$'"\\]|\\u[0-9a-fA-F]{4}|[^'\n\r\\])*'/

# 单行字符串字面量使用双引号，多行字符串字面量使用三个双引号，注意 Kotlin 的多行字符串并不会自动去除缩进空白
# 字符串允许 $ref 或 ${expr} 形式插值，本质依然是 StringBuilder
StringLiteral    := '"' ( /\\[rntb$'"\\]|\\u[0-9a-fA-F]{4}|[^"$\n\r\\]/ | '$' SimpleIdentifier | '${' Expression '}' )* '"'
                  | '"""' ( /[^"$]|"(?!"")/ | '$' SimpleIdentifier | '${' Expression '}' )* '"""'

# 标识符以 Unicode 字母或下划线开头，后接 Unicode 字母、Unicode 数字或下划线，反引号括起来时可以用关键字或特殊字符作标识符
SimpleIdentifier := /[_\p{Ll}\p{Lm}\p{Lo}\p{Lt}\p{Lu}\p{Nl}][_\p{Nd}\p{Ll}\p{Lm}\p{Lo}\p{Lt}\p{Lu}\p{Nl}]*/
                  | /`[^`\n\r]+`/
```

#### 内建类型

1. `Any` 类型对应除了 null 所有取值的集合，可为 null 时则使用 `Any?`。
2. `Unit` 类型只有唯一一个取值，可以看作 Java 的 void。
3. `Nothing` 类型没有任何有效的取值，用于标记表达式永远不会正常返回一个值，比如死循环或者异常。
4. `Boolean` 布尔类型。
5. `Char` 字符类型，UTF-16 码点。
6. `String` 字符串类型，UTF-16 字符串。
7. `Byte`, `Short`, `Int`, `Long` 有符号整数类型。对应的 `UByte`, `UShort`, `UInt`, `ULong` 是内联类，JVM 并不直接支持无符号整数。
8. `Float`, `Double` 浮点数类型。
9. `Array<T>`, `BooleanArray`, `CharArray`, `ByteArray`, `ShortArray`, `IntArray`, `LongArray` 数组类型。
10. `Iterator<out T>`, `Comparable<in T>`, `Function<out R>` 内置接口。
11. `Enum<E>` 枚举类基类。
12. `Throwable` 可抛出的异常或错误。

#### 操作符

Kotlin 支持以下这些操作符：

1. `||`, `&&` 逻辑运算符，支持短路。此外支持 智能类型推断，比如 `x != null && ...` 后面会自动将 `T?` 的 `x` 推断为 `T`。
2. `==`, `!=` 值等价运算符，根据操作数是否为基本类型，决定使用基本类型比较还是自动调用 `equals`。注意，浮点数的基本类型比较遵循 IEEE-754 规定的 `NaN != NaN && 0.0 == -0.0`，而 `equals` 比较则可能出现 `NaN == NaN && 0.0 != -0.0`，因此在编译器无法静态确定类型时会出现预期外的行为差异。
3. `===`，`!==` 地址等价运算符，和 Java 的 `==` 比较运算符行为一致。
4. `<`, `<=`, `>`, `>=` 比较运算符，根据需要自动调用 `compareTo` 做比较运算。
5. `in`, `!in` 属于运算符，判断元素是否存在于集合中。
6. `is`, `!is` 运行时类型判断运算符。
7. `as`, `as?` 运行时强制类型转换运算符，前者无法转换时抛异常，后者无法转换时返回 null 值。
8. `?:` elvis 运算符，当前一个操作数为 null 值时，取后一个操作数的值，支持短路。
9. `!!` 非空断言一元运算符，作为后缀使用。
10. `..` 区间运算符，用于构造闭区间比如 `0..9`。区间可以用在 `for` 循环以及 `in` 判断上，支持重载。
11. `+`, `-`, `*`, `/`, `%` 算术运算符，支持重载。对应
12. `++`, `--` 自增、自减运算符，即可以作为前缀使用，也可以作为后缀使用。
13. `!`, `+`, `-` 逻辑非、取正、取负运算符，作为前缀使用。
14. `@` annotation, label `@` 注解或标签一元运算符，作为前缀使用，用于给修饰的表达式添加注解或标签。
15. `[` index [ `,` ... ] `]`, `.`, `.?`, `::` 下标运算符、成员访问操作符、安全成员访问操作符、方法引用操作符。
16. `(` `)`, `(` `)` `{` `}` , `{` `}` 调用操作符、带匿名函数的调用操作符、单个匿名函数参数的调用操作符，支持重载。
17. infix 自定义中缀操作符，比如 `0 until 10` 中 until 是一个自定义的中缀操作符

```kotlin
fun main() {
    // is 运算符，在运行时判断子类型
    println(castAny("") is String)

    // in 运算符判断属于关系，.. 运算符构造区间
    println(5 in 0..9)

    // ==、!=、>、>=、<、<= 运算符对于对象语义相当于 equals/compareTo，对于基本类型语义相当于 Java 的运算符
    // ===、!== 运算符对于对象语义相当于 Java 的 ==
    println(0.0 == -0.0)
    println(castAny(0.0) == castAny(-0.0))
    println(Vec2(1.0, 1.0) == Vec2(1.0, 1.0))
    println(Vec2(1.0, 1.0) === Vec2(1.0, 1.0))
    println(java.math.BigInteger("1") > java.math.BigInteger("0"))

    // as、as? 运算符运行时强制转换类型，!! 运算符断言非 null，?. 运算符安全访问成员，?: 运算符处理转换 null 值
    val p = castAny(Vec2(0.0, 0.0))
    println(p as Vec2)
    println((p as? Vec2)!!)
    println((p as? Vec2)?.x)
    println((p as? Vec2) ?: throw NullPointerException())

    // ++、-- 自增自减运算符可用于表达式，=、+=、-=、/=、*=、%= 赋值操作则只能作为语句使用
    var x = 0
    println(Vec2((x++).toDouble(), (++x).toDouble()))
    x = 0
    x += 1
    println(x)

    // 下标运算可以传多个参数
    val m = Mat2(0, 1, 2, 3)
    println(m[1, 0])

    // 方法调用的不同形式
    println(m.map({ it + 1 }))
    println(m.map { it + 1 })
    println(m map { it + 1 })
    println(m.mapOffset(1, { it - 1 }))
    println(m.mapOffset(1) { it - 1 })
}

// 数据实体类，会自动实现 equals
data class Vec2(var x: Double, var y: Double)

data class Mat2(val a00: Int, val a01: Int, val a10: Int, val a11: Int) {
    // 重载下标运算
    operator fun get(i: Int, j: Int) = when (i to j) {
        0 to 0 -> a00
        0 to 1 -> a01
        1 to 0 -> a10
        1 to 1 -> a11
        else -> throw IndexOutOfBoundsException()
    }

    // 定义中缀运算符
    infix fun map(fn: (Int) -> Int) = Mat2(fn(a00), fn(a01), fn(a10), fn(a11))

    fun mapOffset(offset: Int, fn: (Int) -> Int = { it }) =
        Mat2(offset + fn(a00), offset + fn(a01), offset + fn(a10), offset + fn(a11))
}

// 任意类型转 Object
fun castAny(x: Any): Any {
    return x
}
```

总结：Kotlin 不提供与二进制位运算相关的运算符；默认的等价运算符和比较运算符分别对应 Java 的 `equals` 和 `compareTo` 语义；赋值类的操作不再是表达式而是语句；新增了一系列 null 值处理的运算符比如 `?:`、`!!`、`.?`；新增了集合属于运算 `in`、区间构造运算 `..` 以及自定义中缀运算符；支持对一部分操作符和语言构造进行重载。

#### 空指针

为了避免重蹈 Java 空指针的覆辙，Kotlin 进行了以下设计：

1. 类名作为类型使用代表此类型非 null，加上 `?` 后缀时代表此类型取值可以为 null，这样可以在编译期避免一部分空指针问题。
2. 泛型参数不指定约束时，默认的约束是 `Any?` 而不是 `Any`，默认并不限制参数类型可以取 null 值。
3. 处理可为 null 值的对象时，可以使用 `!!` 断言，也可以使用 `?:` 处理位 null 的情况。取可为 null 值的对象的成员时，可以使用 `.?` 防止空指针。

#### 控制流

Kotlin 的控制流有 `if else`、`when`、`for`、`while`、`do while`、`break`、`continue`、`try catch finally`、`throw`、`return`，囊括了条件判断、循环迭代、异常处理、强制跳转在内的常见控制流。

Kotlin 的一些控制流属于表达式，而不仅仅是语句。比如，涉及条件判断和异常处理的 `if else`、`when`、`try catch finally` 这几种控制流都属于表达式，它们可以返回一个值。此外，`break`、`continue`、`throw`、`return` 在 Kotlin 里同样也可以用在表达式的位置，不过它们的返回类型是底类型 [Nothing](https://kotlinlang.org/spec/built-in-types-and-their-semantics.html#kotlin.nothing-builtins)，代表执行到此处并不会正常返回。

与更激进的语言比如 Rust 不同，Kotlin 并不是所有控制流都是表达式。循环迭代的 `for`、`while`、`do while` 都属于语句，并不能当作表达式来使用。

下面举例说明 Kotlin 的控制流写法，着重介绍和 Java 不同的地方：

1. 条件判断

```kotlin
// if 和 Java 一致，也支持省略大括号
var line = readLine()!!.trim()
if (line == "") {
    println("empty line")
} else if (line.matches(kotlin.text.Regex("^\\S+$"))) {
    println("single word")
} else {
    println("multiple words")
}

// if 作表达式使用，替代 Java 的三目运算符
var varchar2 = if (line == "") null else line

// when 用于判断多个分支条件，可以选择携带一个待判断的值
// 一般用来判断枚举值、不确定具体类型的字段等等
//
// when 携带待判断的值时，可以选择临时把待判断的值赋值给一个常量，之后在分支里面使用这个常量
// 各个分支可以使用 <value>[, <value>[, ...]]、in <range>、!in <range>、is <type>、!is <type>
// 最后一个分支条件使用 else 时，代表该分支为默认情况
// 
// when 和 if else 一样可以当做表达来使用
when (val num = kotlin.random.Random.nextInt(21) - 10) {
    !is Int -> {
        println("$num is not even an integer")
        throw kotlin.IllegalArgumentException()
    }
    in Int.MIN_VALUE..-1 ->
        println("$num is negative")
    in 0..4 ->
        println("$num is less than five")
    5 ->
        println("$num is exactly five")
    6, 7, 8, 9 ->
        println("$num is six to nine")
    else ->
    	println("$num is more than nine")
}

// when 也可以不携带任何待判断的值，这种情况下每一个分支上需要写的是完整的判断条件
// 可以考虑用 when 替代过多分支的 if else
var num = kotlin.random.Random.nextInt(21) - 10
when {
    num !is Int -> {
        println("$num is not even an integer")
        throw kotlin.IllegalArgumentException()
    }
    num < 0 ->
        println("$num is negative")
    num in 0..4 ->
        println("$num is less than five")
    num == 5 ->
        println("$num is exactly five")
    num < 10 ->
        println("$num is six to nine")
    else ->
    	println("$num is more than nine")
}
```

2. 循环迭代

```kotlin
// 经典 while/do while 循环，没啥好说的和 Java 一模一样
while (this.receive()) {
    println("received ${this.len}")
    this.len = 0
}
do {
    txn.writeSomething()
} while (!txn.commit())

// Kotlin 有且只有 foreach 循环，in 前面是循环变量的声明部分，in 后面是待遍历的 Iterable<*> 表达式部分
// 声明循环变量时可以用 : <type> 来指定变量类型，也可以使用多变量声明语法，但不能在最前面写 var 关键字
for (item in listOf(1, 1, 4, 5, 1, 4)) {
    println(item)
}
for ((i, x: Long) in (10L downTo 0 step 2).withIndex()) {
    println("[$i] $x")
}

// break 和 continue 默认只控制内层循环
for (i in 0..9) {
    for (j in 0..9) {
        if (j > i) {
            break
        }
        print("($i, $j) ")
    }
    println()
}

// 通过语句标签可以控制外层循环跳转
// Java 在语句前加 label: 定义标签，Kotlin 在语句前加 label@ 定义标签，使用 break@label、continue@lable 来跳转
// 注意，定义时 @ 前面不能有空白字符，跳转时 @ 前后都不能有空白字符
outer@
for (i in 0..9) {
    for (j in 0..9) {
        if (i == 5 && j == 5) {
            break@outer
        }
        print("($i, $j) ")
    }
    println()
}
```

3. 异常处理

```kotlin
// try 不支持 try-with-resource 语法
// catch 语句使用冒号 ':' 指定捕捉的异常类型，和 Java 不同 Kotlin 不支持写 Union Type
val host = "example.com"
val socket = java.net.Socket(host, 80)
try {
    socket.getOutputStream().write("HEAD / HTTP/1.0\r\nHost: $host\r\n\r\n".toByteArray())
    print(socket.getInputStream().bufferedReader().readText())
} catch (e: java.io.IOException) {
    println("error: $e")
} finally {
    socket.close()
}

// 使用 Closeable.use 来代替 try-with-resource
val host = "example.com"
java.net.Socket(host, 80).use {
    it.getOutputStream().write("HEAD / HTTP/1.0\r\nHost: $host\r\n\r\n".toByteArray())
    print(it.getInputStream().bufferedReader().readText())
}

// try catch finally 可以用作表达式
val (x, y) = Pair(1, 0)
val division: Int? = try { x / y } catch (e: ArithmeticException) { null }

// Kotlin 去除了 Java 强制在方法签名上标记检查型异常的设计
// 因此如果需要从 Java 调用 Kotlin，建议在函数前加上 @Throws 注解
@Throws(java.io.IOException::class)
fun requestHead(host: String, port: Int = 80, uri: String = "/"): String {
    val hostAndPort = if (port == 80) host else "$host:$port"
    java.net.Socket(host, port).use {
        it.getOutputStream().write("HEAD $uri HTTP/1.0\r\nHost: $hostAndPort\r\n\r\n".toByteArray())
        return it.getInputStream().bufferedReader().readText()
    }
}
```

#### 变量声明

Kotlin 使用  `var` 关键字声明变量，`val`  关键字声明不可变量，后者相当于 Java 的 `final`。

```kotlin
// 声明变量或不可变量，并初始化，自动推导类型
var var1 = 1
val const1 = 1

// 声明变量或不可变量，并初始化，指定类型
// Kotlin 所有可以声明变量或常量的地方都使用冒号 <id>: <type> 的形式
var var2: Int = 1
val const2: Int = 1

// 声明变量或不可变量，如果不立刻初始化，则必须指定类型
var var3: Int
val const3: Int
if (kotlin.random.Random.Default.nextBoolean()) {
    var3 = 1
    const3 = 1
} else {
    var3 = 0
    const3 = 0
}

// Kotlin 声明变量或不可变量时支持基本的元组解构，右值需要重载 componentN 系列操作符来支持此行为
// 比如 Collection<*> 目前最多重载到了 component5 操作符，因此集合只支持五元组以下的情况
val (x0, x1, x2, x3, x4) = listOf(0, 1, 2, 3, 4)

// 元组解构内每个变量或不可变量都可以选择性写明类型
val (x: String, y: Int) = Pair("0", 0)
```

#### 函数声明

和 Java 不同，Kotlin 的函数并不是必须要定义在某个类之中的。Kotlin 的函数可以直接定义在文件级，甚至定义在另一个函数或方法的内部。定义在文件顶级的方法最终会被编译到文件级的类字节码中，成为一个静态方法。比如，定义在 `Demo.kt` 文件顶级的 `function1` 函数最终会被编译为 `DemoKt.class` 类的静态方法 `function1`，定义在 `Demo.kt` 文件的 `SomeDemo` 类的方法 `method1` 中的内部函数`privateFunction1` 最终会被编译为 `SomeDemo.class` 类的 `method1$privateFunction1` 静态方法。

```kotlin
// 文件顶级可以定义函数，编译为静态方法
// 顶级函数或类方法的默认修饰符为 public，函数或方法内部定义的局部函数修饰符一定是 private
fun function1() {
    println("hello world")
}

class SomeDemo {
    // 直接定义在类中的函数，都不是静态的
    fun method1() {
        // 方法内部也可以定义函数
        // 使用 fun 关键字定义函数或方法时每个参数都必须写明类型
        // 函数的返回值也必须写清楚类型，如果返回类型省略不写，则代表返回 Unit（相当于 Java 的 void）
        fun privateFunction1(x: Int, y: Long): Long {
            return x + y
        }
        println(privateFunction1(1, 2L))

        // Kotlin 允许把一行表达式能写明白的函数用等号 '=' 直接定义
        // 此时返回值类型 Kotlin 能够自动推导出来，可以省略不写
        fun privateFunction2(x: Int, y: Long) = x + y
        println(privateFunction2(1, 2L))
    }
}

// JVM 的 main 方法必须是静态方法，而 Kotlin 类里不允许直接定义静态方法，因此在 main 方法只能定义在文件顶级
// 此外，Kotlin 1.3 后允许 main 方法不带 Array<String> 参数，这样做的话编译器会自动生成一个 synthetic 的 main 方法
fun main(args: Array<String>) {
    function1()
    SomeDemo().method1()
}

// 函数可以写泛型，子类型约束可以加在泛型参数或者 where 子句位置
fun <F, T> genericFunction1(fn: F): Pair<T, T> where F: () -> T, T: Number {
    return Pair(fn(), fn())
}
```

Kotlin 的函数或方法允许为参数提供默认值，调用时也可以像 Python 一样指定传参的名字。此外还可以使用 `vararg` 定义可变参数列表。参数包含默认值的函数或方法，在生成字节码时会产生两个方法：一个 Java 可以直接调用的方法，不支持默认值；另一个是有额外参数来记录各个是否传值的 synthetic 方法，供 Kotlin 调用。

```kotlin
// 函数形参定义时可以用 '=' 提供一个默认值表达式，这个表达式会在每次调用函数且缺少传参时被求值，注意默认值不必是常量
// vararg 声明的是可变参数列表，它是一个 Array<*> 类型，比如下面这个可变参数 rest 的实际类型是 Array<Any>
// 一个函数最多只能有一个可变参数，不过 Kotlin 并不要求 vararg 必须是最后一个形参
fun fancyFunction(foo: String, bar: Int = 0, baz: Boolean = randomDecide(), vararg rest: Any) {
    println("foo=$foo, $bar=$bar, baz=$baz, rest=${rest.contentToString()}")
}

// 一个有副作用的函数，但可以用在函数形参默认值的表达式里
fun randomDecide(): Boolean {
    val decision = kotlin.random.Random.Default.nextBoolean()
    println("invoke randomDecide() -> $decision")
    return decision
}

fun main() {
    // 没有默认值的参数必须传参
    fancyFunction("wow")
    // 没有默认值的参数一样可以按名字传参
    fancyFunction(foo = "wow")
    // 按位置传参和按名字传参可以混用，按名字传参的参数可以调换位置
    // 从测试现象看，传参时实参和形参的对应关系只和位置有关，而与类型无关
    // 形参与实参从左到右挨个匹配，可变长参数则尽可能多地匹配实参
    fancyFunction("wow", bar = 1)
    fancyFunction("wow", baz = true)
    fancyFunction("wow", baz = true, bar = 1)
    fancyFunction("wow", 1, true)
    fancyFunction("wow", bar = 1, true)
    fancyFunction(baz = true, bar = 1, foo = "wow")
    fancyFunction("wow", 1, baz = true, "you", "can", "really", "dance")
    // 可变长参数可以使用传播操作符 '*' 直接传递一个数组类型，语法和 Python 很像
    fancyFunction("wow", 1, baz = true, *arrayOf("you", "can", "really", "dance"))
}
```

Kotlin 允许在类外为某个类拓展定义新的方法，这个语法糖通过将 `this` 转化为第一个形参，并变换所有被调用处的传参方式来实现。编译生成的实际方法属于定义的位置，而不是被拓展的类的位置，可以理解为一个工具函数或方法，因此并不会改变被拓展类的字节码。

```kotlin
data class Point<N : Number>(val x: N, val y: N)

// 这个拓展方法会在文件级的类上生成 static double distanceTo(Point, Point)
// 这个方法是静态分发的，this 作为第一个形参被传递进来，并不是虚表分发的 this
infix fun <N : Number> Point<N>.distanceTo(that: Point<N>): Double {
    val dx = this.x.toDouble() - that.x.toDouble()
    val dy = this.y.toDouble() - that.y.toDouble()
    return kotlin.math.sqrt(dx * dx + dy * dy)
}

// 在文件级的类上生成 static main(String[])，因此这是一个合法的程序入口
fun Array<String>.main() {
    // 调用相当于 XxxKt.distanceTo(new Point(1, 2), new Point(5, 5))
    println(Point(1, 2) distanceTo Point(5, 5))
    println(Incrementer(1.0, 1.0).apply(Point(2.0, 3.0)))
}

class Incrementer(dx: Double, dy: Double) {
    private val delta = Point(dx, dy)

    fun apply(point: Point<Double>): Point<Double> {
        // 调用相当于 this.increment(point)
        return point.increment()
    }

    // 将拓展方法定义在其他类的内部，可能需要引用类的实例，因此生成的方法不能是 static 需要走虚表分发，
    // 即便如此，方法体内的 this 和实际上虚表传递进进来的 this 也不是一回事，比如以下方法实际相当于：
    // private Point<Double> increment(Point<Double> $this) {
    //     return new Point($this.getX() + this.delta.getX(), $this.getY() + this.delta.getY())
    // }
    private fun Point<Double>.increment(): Point<Double> {
        return Point(this.x + delta.x, this.y + delta.y)
    }
}
```

Kotlin 的函数或方法定义可以被注解，或是被修饰符修饰，常见的修饰符有：

1. `tailrec` 尾递归修饰符，编译器会将符合条件的尾递归函数转化为等价的循环实现。
2. `inline` 允许内联优化，主要用于减少匿名函数开销。此外，当开启了内联函数时，Kotlin 允许使用 "非局部 return"。
3. `suspend` 声明为可挂起的函数或方法，编译器通过 CPS 变换实现异步，suspend 只能被 suspend 调用。
4. `operator` 修饰类的方法，用于操作符重载。
5. `infix` 修饰类的方法，可作为中缀操作符来调用。
6. `external` 声明原生方法，相当于 Java 的 `native`。

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.runBlocking
import java.util.concurrent.atomic.AtomicBoolean
import java.util.concurrent.atomic.AtomicLong

fun main() {
    println(factorial(5))

    println(nest(Int::toString, String::toInt)(1))

    // 操作符重载、中缀运算符方法调用
    val (x: Int) = Option.of("123".toIntOrNull()) map { -it }
    println(x)

    // 在协程内执行可挂起方法
    val debounced = debounce({ println("called @${System.currentTimeMillis()}") })
    runBlocking {
        debounced()
        debounced()
        delay(1000L)
        debounced()
        debounced()
    }
}

fun factorial(n: Int): Int {
    // 声明为尾递归的函数或方法，内部实现会被优化成等价的循环
    tailrec fun impl(n: Int, acc: Int): Int =
        if (n > 0) impl(n - 1, acc * n) else acc
    return impl(n, 1)
}

// 内联函数或方法在调用时会被内联到调用方，因此首先要保证不访问任何私有成员
//
// Kotlin 的 inline 修饰符不仅影响内联函数本身，也会默认导致入参中的匿名函数自动被内联
// 这样有一个效果是，被内联的匿名函数中允许直接写 return 来返回最靠近调用位置的方法或函数
// 即内联函数允许入参的匿名函数使用 "非局部 return"
//
// 为了控制对入参匿名函数的内联，可以使用 noinline 形参修饰符完全禁止内联入参的匿名函数
// 使用 crossinline 形参修饰符则标记该参数虽然会被内联，但里面禁止使用 "非局部 return"
// 下面这种返回匿名函数的内联函数，就是典型的需要入参写 croosinline 修饰符的情况
inline fun <A, B, C> nest(crossinline f: (A) -> B, crossinline g: (B) -> C): (A) -> C {
    return { a: A -> g(f(a)) }
}

fun debounce(action: () -> Unit, timeout: Long = 1000L): suspend () -> Unit {
    val running = AtomicBoolean(false)
    val lastRun = AtomicLong(0)

    // 定义一个可挂起函数，类型签名是 suspend () -> Unit
    // suspend 只能够在另一个 suspend 中被调用，最终在最外层的协程上下文内被执行
    // 可挂起函数是通过 CPS 变换和状态机类来实现的异步函数，和其他语言的 Promise/Future 并不是一个套路
    suspend fun debounced() {
        if (!running.get() && System.currentTimeMillis() > lastRun.get() + timeout) {
            delay(timeout)
            if (running.compareAndSet(false, true)) {
                lastRun.set(System.currentTimeMillis())
                action()
                running.compareAndSet(true, false)
            }
        }
    }
    return ::debounced
}

class Option<out T : Any> {
    private val value: T?

    private constructor() {
        this.value = null
    }

    private constructor(value: T) {
        this.value = value
    }

    companion object {
        fun <T : Any> of(value: T?): Option<T> {
            return if (value != null) Option(value) else Option()
        }
    }

    // 重载一系列元组析构操作符 componentN 中的 component1
    // 此外，加减乘除、自增自减、比较、下标、调用、属性 getter/setter 委托等等操作也可以被重载
    operator fun component1(): T {
        return this.value ?: throw ArrayIndexOutOfBoundsException()
    }

    // 定义中缀操作符方法，可以通过中缀操作符的语法形式进行方法调用
    infix fun <U : Any> map(f: (T) -> U?): Option<U> {
        return of(f(this.value ?: return Option()))
    }
}
```

#### 面向对象

列举一些在面向对象的语法和语义上 Kotlin 比较有特点的地方：

2. 实例化对象时，调用构造函数不允许加 `new` 前缀。
3. 定义类时允许在类名后面直接写一个主要构造函数，也支持在类内部使用 `constructor` 关键字定义次要构造函数。和 Java 一样，不写任何构造函数则默认给一个空构造函数。
4. Kotlin 支持定义不同类型的类或接口：普通类 `class`、接口 `interface`、枚举类 `enum class`、注解 `annotation class`、数据实体类 `data class` 自动实现 getter/setter/toString/copy/equals/hashCode/componentsN 等常见方法、内联类 `value class` 用于告知编译器允许直接内联优化掉、密封类 `sealed class` 只允许本包中的类继承、密封接口 `sealed interface` 只允许本包中的类实现、函数式接口 `fun interface` 只包含唯一一个抽象方法可作为匿名函数类型使用的接口。
5. 可以使用 `object` 关键字来定义单例或者匿名内部类对象。
6. 可以通过 `typealias` 定义类型别名，只允许在 Kotlin 内使用别名，在 Java 下无法使用。
7. 泛型参数允许使用 `out`/`in` 修饰符来指定参数类型的协变/逆变关系。
8. 不允许定义静态成员，要通过 `companion` 伴生对象来模拟。
9. 类的内部定义的类，默认是静态内部类，如果需要定义引用上级对象 `this` 的内部类应当显式地通过 `inner class` 来定义。
10. 类以及类成员的访问修饰符支持 `private` 私有、`protected` 子类可见、`internal` 包可见、`public` 共有，和 Java 的四个级别一一对应，但 Kotlin 默认的修饰符是 `public` 而不是 `internal`。
11. 类以及类成员如果不加 `open` 修饰符则默认都不允许重载，即默认自带 `final`。如果加了 `abstract`/`sealed` 修饰符则默认 `open`，接口成员默认 `open`。
12. 类实现接口允许通过 `by` 关键字使用委托模式语法糖，会自动嵌入一个私有的 synthetic 字段在初始化时存放委托对象。
13. 非私有的属性会自动生成对应的 getter/setter，对应的类字段本身是 `private` 修饰。Kotlin 默认使用 getter/setter 进行成员访问。
14. 由于自动生成 getter/setter 方法，属性就可以和方法一样被 `override` 修饰并重写。
15. 除非被 `abstract` 修饰，否则属性或者私有的类字段必须被初始化。这和 Java 自动放零值，不要求必须初始化的行为不一样。
16. Kotlin 支持用语法糖写自定义 getter/setter，更进一步也支持使用 'by' 关键字写属性委托。被 `private` 修饰的类字段可以自定义其私有的 getter/setter，但不能使用属性委托。
17. Kotlin 的接口里中不仅可以写方法，还可以写属性，本质上依然是调用 getter/setter。
18. 支持操作符重载 `operator`、中缀操作符 `infix` 等一系列语法糖。
```kotlin
// 定义密封接口，只允许同包下实现该接口
// 泛型参数可以额外修饰 out/in，out 代表协变，in 代表逆变
sealed interface BasePoint<out N : Number> {
    // 接口里可以定义属性，var 对应 getter + setter，val 对应只有 getter
    var x: Double
    var y: Double
}

// 定义类型别名，只在编译器生效，并不产生对应的字节码
typealias DoublePoint = BasePoint<Double>

// 定义抽象类 AbstractDoublePoint 实现 BasePoint<Double> 接口，只允许包内访问
// 冒号 ':' 前面允许写 Kotlin 的一个重要的语法糖 "主要构造函数"，形参可以加 var/val 代表直接定义为属性，这样定义的属性不能直接自定义 getter/setter
// 冒号 ':' 后面允许写父类和实现接口列表，用逗号 ',' 分隔，可以调用父类构造方法或者为接口使用委托
internal abstract class AbstractDoublePoint(override var x: Double, override var y: Double) : DoublePoint {
    abstract fun absolute(): Double
}

// 定义实体类 DataPoint 实现 BasePoint<Double> 接口，会自动实现一系列数据实体类应该有的方法
// 注意 data class 一定是 final 的，所以某些框架生成子类来实现惰性加载在 Kotlin 的 data class 上行不通
data class DataPoint(override var x: Double, override var y: Double) : DoublePoint

// 定义类 MyPoint 通过构造委托对象实现 BasePoint<Double> 接口
// 类定义的同时就可以写主要构造函数，后面的继承和实现列表
// 注意这里主要构造函数的实参仅仅是传参而不是定义属性，因此不写 var/val 更不用修饰 override
// 构造函数也是函数，因此可以写默认值
open class MyPoint(x: Double = 0.0, y: Double = 0.0) : DoublePoint by delegatePoint(x, y) {
    // 属性或字段初始化，修饰为 private 则只生成一个不带 getter/setter 的类字段
    // 不是 private 则是生成带 getter/setter 的属性，并对应生成一个同名的类字段
    private val dummyField1: Dummy = Dummy("dummyField1")

    // 属性或字段可以紧接着直接定义 getter/setter
    // 使用关键字 field 指代属性对应的字段，可以防止递归
    var dummyField2: String = "dummyField2"
        get() = "($field)"
        // setter 可以不省略形参的类型
        set(value) {
            if (Regex("^\\(.+\\)$") matches value) {
                field = value.substring(1, value.length - 1)
            } else {
                throw IllegalArgumentException()
            }
        }

    // 属性或字段也可以使用委托
    val dummyField3: String by lazy { "dummyField3" }

    // Kotlin 使用伴生对象来模拟类的静态属性、静态方法、静态初始化，而不是直接生成 static
    companion object {
        init {
            println("MyPoint.<clinit>() calls MyPoint.Companion.<init>()")
        }

        fun originPoint(): DoublePoint {
            return OriginPoint
        }
    }

    // 可以定义多个初始化块，里面的代码会在实例初始化时被调用
    // 初始化块、属性初始化表达式的执行先后顺序和书写顺序有关，在构造函数以及委托之后执行
    init {
        println("execute init block")
    }

    // 可以定义多个次要构造函数，在冒号 ':' 后可以写 this/super 调用其他构造函数
    constructor() : this(0.0, 0.0) {
        println("run secondary constructor")
    }

    // 定义一个内部类，不加 inner 默认是静态内部类
    private inner class Dummy(name: String) {
        init {
            // 内部类可以通过 this@ 来明确引用的是外层哪一个 this
            println("initialize ${this@MyPoint}.$name")
        }
    }
}

// 定义单例对象
// 最终生成一个同名的子类字节码，继父类或实现接口，包含一个在类加载时初始化的 INSTANCE 静态字段
internal object OriginPoint : AbstractDoublePoint(0.0, 0.0) {
    override fun absolute(): Double {
        return 0.0
    }
}

fun delegatePoint(x: Double, y: Double): DoublePoint {
    println("construct delegate")
    // 对象表达式，类似 Java 的匿名内部类
    return object : AbstractDoublePoint(x, y) {
        override fun absolute(): Double {
            return kotlin.math.sqrt(x * x + y * y)
        }
    }
}

fun main() {
    // 直接使用单例对象
    println(OriginPoint)

    // 调构造函数不加 new，看起来和函数调用没有区别
    println(MyPoint())
    println(MyPoint(x = 0.0, y = 0.0))
    println(DataPoint(0.0, 0.0))
}
```

#### 匿名函数

// TODO

### 类型

// TODO

### 协程

// TODO
