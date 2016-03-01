# PHP 7

PHP 7 在 [December 3rd, 2015]
(http://php.net/archive/2015.php#id2015-12-03-1) 发布了. 它带来了大量的新特性，改变和向后兼容破损，这些改变概述如下：

**[性能](#性能)**

**[特性](#特性)**
* [合并的比较操作符](#combined-comparison-operator)
* [空指针合并操作符](#null-coalesce-operator)
* [标量类型声明](#scalar-type-declarations)
* [返回值类型声明](#return-type-declarations)
* [匿名类](#anonymous-classes)
* [Unicode编码点转移语法](#unicode-codepoint-escape-syntax)
* [闭包 `call()` 方法](#closure-call-method)
* [带过滤的 `unserialize()`](#filtered-unserialize)
* [`IntlChar` 类](#intlchar-class)
* [期望](#expectations)
* [组和 `use` 声明](#group-use-declarations)
* [生成器返回表达式](#generator-return-expressions)
* [生成器委托](#generator-delegation)
* [整形除使用 `intdiv()`](#integer-division-with-intdiv)
* [`session_start()` 参数](#session_start-options)
* [`preg_replace_callback_array()` 函数](#preg_replace_callback_array-function)
* [CSPRNG 函数](#csprng-functions)
* [支持 `define()` 定义数组常量](#support-for-array-constants-in-define)
* [新增的反射类](#reflection-additions)

**[改动](#changes)**
* [放松保留字的限制](#loosening-reserved-word-restrictions)
* [同一变量格式](#uniform-variable-syntax)
* [引擎中的异常](#exceptions-in-the-engine)
* [可抛出的接口](#throwable-interface)
* [整形语义](#integer-semantics)
* [JSON 扩展替代 JSOND](#json-extension-replaced-with-jsond)
* [ZPP Failure on Overflow](#zpp-failure-on-overflow)
* [修复 `foreach()`的行为](#fixes-to-foreachs-behaviour)
* [修改 `list()`的行为](#changes-to-lists-behaviour)
* [改变被0整除的语义](#changes-to-division-by-zero-semantics)
* [修复自定义Session Handler返回值](#fixes-to-custom-session-handler-return-values)
* [标记PHP 4风格的构造函数过期](#deprecation-of-php-4-style-constructors)
* [移除 date.timezone 警告](#removal-of-datetimezone-warning)
* [移除可替换PHP标签](#removal-of-alternative-php-tags)
* [移除Switch语句中的多默认块](#removal-of-multiple-default-blocks-in-switch-statements)
* [移除重复名字参数的重定义](#removal-of-redefinition-of-parameters-with-duplicate-names)
* [移除Dead Server APIs](#removal-of-dead-server-apis)
* [移除数字字符串中的十六进制支持](#removal-of-hex-support-in-numerical-strings)
* [移除已标记过期的函数](#removal-of-deprecated-functionality)
* [再分级与移除 E_STRICT 通知](#reclassification-and-removal-of-e_strict-notices)
* [反对使用`password_hash()`的盐选项](#deprecation-of-salt-option-for-password_hash)
* [无效的八进制字面量错误](#error-on-invalid-octal-literals)
* [`substr()` 返回值改动](#substr-return-value-change)

**[FAQ](#faq)**
 * [PHP 6发生了什么?](#what-happened-to-php-6)

## 性能
毫无争议，PHP 7带来的最大的变化就是极大的性能提升，这得益于对Zend引擎的重构，新的引擎使用了更加紧凑的数据结构和更少的堆分配/回收操作。

在实际项目使用中，性能的提升效果不是固定的，虽然很多应用在使用PHP 7之后活的了超过100%的性能加速，同时还只使用更少的内存消耗！

重构后的代码基础也为将来的优化（比如JIT编译）提供了更多的可能，所以，可以预计在接下来的PHP版本仍然会看到不错的性能提升。

PHP 7性能对比图标：


 - [Turbocharging the Web with PHP 7](https://www.zend.com/en/resources/php7_infographic)
 - [Benchmarks from Rasmus's Sydney Talk](http://talks.php.net/oz15#/drupalbench)

## 特性

### 组合比较操作符
组合比较符（也称太空船操作符）用于对两个操作数进行三路比较的简化符号，它返回一个整数，其值为以下之一：

* 一个正整数（当左操作数大于右操作数时）
* 0（当两个操作数相等时）
* 一个负整数（当右操作数大于左操作数时）

该操作符和等式运算符（`==`，`！=`，`===`，`！==`）等有相同的优先级，并且和其它松比较操作符（`<`, `>=`等）有相同的行为，同样组合比较付也是非组合的，不允许对操作数进行链式调用（像 `1 <=> 2 <=> 3`）。

```PHP
// compares strings lexically
var_dump('PHP' <=> 'Node'); // int(1)

// compares numbers by size
var_dump(123 <=> 456); // int(-1)

// compares corresponding array elements with one-another
var_dump(['a', 'b'] <=> ['a', 'b']); // int(0)
```

对象是不可比较的，所以对对象使用组合比较符会导致未定义的行为。

RFC: [Combined Comparison Operator](https://wiki.php.net/rfc/combined-comparison-operator)

### 空引用合并操作符
空引用合并操作符（或者称isset三元操作符）是`isset()`检测的三元操作符的简化符号。空引用检测在应用中是经常需要的操作，为了这个目的引入了这个新的语法

```PHP
// PHP 7 之前
$route = isset($_GET['route']) ? $_GET['route'] : 'index';

// PHP 7+ 
$route = $_GET['route'] ?? 'index';
```

> 这有点类似于其他语言的 `?:` 合并操作符


RFC: [Null Coalesce Operator](https://wiki.php.net/rfc/isset_ternary)

### 标量类型声明
标量类型声明有两种模式： **强制**（默认）和**严格**。下面类型的参数现在能够强制执行（无论是强制模式还是严格模式）：字符串（`string`），整数(`int`），浮点数（`float`），以及布尔值（`bool`）。它们扩充了PHP 5中引入的其他类型：类名（class names），接口(interfaces)，数组(`array`)和回调(`callable`)类型。

```PHP
// 强制模式 
function sumOfInts(int ...$ints)
{
    return array_sum($ints);
}

var_dump(sumOfInts(2, '3', 4.1)); // int(9)
```

要使用严格模式，一个 `declare()`声明指令必须放在文件的顶部，这意味着声明严格标量是基于文件可配置的。这个指令不仅影响参数的类型声明，也影响到函数的返回值声明（参考[返回值类型声明](返回值类型声明)，内置的PHP函数以及扩展中加载的函数）。

如果类型检查失败了，一个 `TypeError`异常会（见[引擎中的异常](#引擎中的异常)）抛出，在严格模式下，唯一的例外是，当整数当作一个浮点数提供到上下文时，整数到浮点数的自动类型转换（反过来不允许）。

```PHP
declare(strict_types=1);

function multiply(float $x, float $y)
{
    return $x * $y;
}

function add(int $x, int $y)
{
    return $x + $y;
}

var_dump(multiply(2, 3.5)); // float(7)
var_dump(add('2', 3)); // Fatal error: Uncaught TypeError: Argument 1 passed to add() must be of the type integer, string given...
```

注意，**只有**在*调用上下文*执行类型检查时才会适用这些规则，这意味着严格类型检查只在函数/方法调用时使用，而在它们的定义处不使用。在上面的例子中，两个函数可以定义在严格或者强制模式的文件中，但是，只要是它们在声明了严格模式的文件中被调用，就会使用严格类型规则。

**BC Breaks**
 - 现在 `int`, `string`, `float`, and `bool` 不再允许作为类名出现。


RFC: [Scalar Type Declarations](https://wiki.php.net/rfc/scalar_type_hints_v5)

### 返回值类型声明
返回类型声明允许为函数，方法或者闭包指定返回值的类型了，支持下列作为返回值类型： `string`,
`int`, `float`, `bool`, `array`, `callable`, `self` (仅方法), `parent`
(仅方法), `Closure`, 类名和接口名。

```PHP
function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}

print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));
/* Output
Array
(
    [0] => 6
    [1] => 15
    [2] => 24
)
*/
```

在子类化的情况下，返回值类型必须是不变的。这意味着一个方法被子类重写或者被实现时，它的返回值类型必须和父类中的类型完全匹配。

```PHP
class A {}
class B extends A {}

class C
{
    public function test() : A
    {
        return new A;
    }
}

class D extends C
{
    // overriding method C::test() : A
    public function test() : B // Fatal error due to variance mismatch
    {
        return new B;
    }
}
```

覆盖的方法 `D::test() : B`导致了一个 `E_COMPILE_ERROR`，因为返回值类型不允许修改，`D::test()`方法必须使用返回值类型为 `A`。

```PHP
class A {}

interface SomeInterface
{
    public function test() : A;
}

class B implements SomeInterface
{
    public function test() : A // all good!
    {
        return null; // Fatal error: Uncaught TypeError: Return value of B::test() must be an instance of A, null returned...
    }
}
```

这一次，实现的方法在执行时抛出了 `TypeError` 异常（参见 [引擎中的异常](#引擎中的异常)），这是因为 `null` 不是一个正确的返回类型的值，只有类A的实例作为返回值才是允许的。

RFC: [Return Type Declarations](https://wiki.php.net/rfc/return_types)

### 匿名类

在简单的，没有对象实例要创建的时候，匿名类是非常有用的。

```PHP
// PHP 7 之前
class Logger
{
    public function log($msg)
    {
        echo $msg;
    }
}

$util->setLogger(new Logger());

// PHP 7+ 
$util->setLogger(new class {
    public function log($msg)
    {
        echo $msg;
    }
});
```

同样可以传递参数到匿名类的构造函数，继承其它类或者实现某些接口，还可以像普通类一样使用 traits：

```PHP
class SomeClass {}
interface SomeInterface {}
trait SomeTrait {}

var_dump(new class(10) extends SomeClass implements SomeInterface {
    private $num;

    public function __construct($num)
    {
        $this->num = $num;
    }

    use SomeTrait;
});

/** Output:
object(class@anonymous)#1 (1) {
  ["Command line code0x104c5b612":"class@anonymous":private]=>
  int(10)
}
*/
```

在一个类中嵌套的匿名类没有对外部类的私有或者受保护的字段和方法的访问权限。为了使用外部类的 protected属性或者方法，匿名类可以通过继承外部类实现。为了使用外部类的私有的或者受保护的属性或方法，需要将外部类作为匿名类的构造函数的参数：

```PHP
<?php

class Outer
{
    private $prop = 1;
    protected $prop2 = 2;

    protected function func1()
    {
        return 3;
    }

    public function func2()
    {
        return new class($this->prop) extends Outer {
            private $prop3;

            public function __construct($prop)
            {
                $this->prop3 = $prop;
            }

            public function func3()
            {
                return $this->prop2 + $this->prop3 + $this->func1();
            }
        };
    }
}

echo (new Outer)->func2()->func3(); // 6
```

RFC: [Anonymous Classes](https://wiki.php.net/rfc/anonymous_classes)

### Unicode编码点转义语法

这可以打印一个使用双引号或者heredoc包围的UTF-8编码的Unicode编码点（codepoint）。可以接受任何有效的codepoint，并且先导的 `0`是可以省略的。


```PHP
echo "\u{aa}"; // ª
echo "\u{0000aa}"; // ª (same as before but with optional leading 0's)
echo "\u{9999}"; // 香
```

RFC: [Unicode Codepoint Escape Syntax](https://wiki.php.net/rfc/unicode_escape)

### Closure::call()

闭包的新的 `call()`方法是用于简单干练地绑定一个方法到对象上闭包并调用它。新的方法有更好的性能并且省去了在调用前创建一个中间闭包的代码，使代码更加紧凑。


```PHP
class A {private $x = 1;}

// PHP 7 之前
$getXCB = function() {return $this->x;};
$getX = $getXCB->bindTo(new A, 'A'); // intermediate closure
echo $getX(); // 1

// PHP 7+ 
$getX = function() {return $this->x;};
echo $getX->call(new A); // 1
```

RFC: [Closure::call](https://wiki.php.net/rfc/closure_apply)

### 带过滤的 `unserialize()`

这个特性旨在为反序列化不可信数据时提供更好的安全性。它通过允许开发者以白名单的形式来规定可以反序列化的类，以此防止潜在的代码注入。

```PHP
// converts all objects into __PHP_Incomplete_Class object
$data = unserialize($foo, ["allowed_classes" => false]);

// converts all objects into __PHP_Incomplete_Class object except those of MyClass and MyClass2
$data = unserialize($foo, ["allowed_classes" => ["MyClass", "MyClass2"]);

// default behaviour (same as omitting the second argument) that accepts all classes
$data = unserialize($foo, ["allowed_classes" => true]);
```

RFC: [Filtered unserialize()](https://wiki.php.net/rfc/secure_unserialize)

### `IntlChar` 类

新的 `IntChar` 类型旨在暴露更多的ICU功能，这个类自身定义了许多静态方法用于操作多字符集的unicode字符。

```PHP
printf('%x', IntlChar::CODEPOINT_MAX); // 10ffff
echo IntlChar::charName('@'); // COMMERCIAL AT
var_dump(IntlChar::ispunct('!')); // bool(true)
```

为了使用这个类，必须安装 `Intl`扩展。

**BC Breaks**
 - Classes in the global namespace must not be called `IntlChar`.

RFC: [IntlChar class](https://wiki.php.net/rfc/intl.char)

### 预期

预期是向后兼容并增强之前的 `assert()`函数。它使得在生产环境中启用断言的成本为零，并且提供当断言失败时抛出特定异常的能力。

`assert()`函数的原型如下

```
void assert (mixed $expression [, mixed $message]);
```
在旧的API中，如果 `$expression`是一个字符串，那么它会被计算，如果第一个参数falsy，那么断言失败。第二个参数可以是一个字符串（导致触发一个 AssertionError）或者是一个包含错误消息的自定义异常对象。

```PHP
ini_set('assert.exception', 1);

class CustomError extends AssertionError {}

assert(false, new CustomError('Some error message'));
```

这个特性与 PHP.ini 中的两个设置有关

 - zend.assertions = 1
 - assert.exception = 0

**zend.assertions** 有三个值:
 - **1** = 生产并执行代码 (开发模式)
 - **0** = 生成代码并在执行时跳过
 - **-1** = 不生产代码 (零开销, 生产模式)

**assert.exception** 意味着断言失败时抛出一个异常，为了与旧的 `assert()`函数保持兼容，此选项默认是关闭的。

RFC: [Expectations](https://wiki.php.net/rfc/expectations)

### Group `use` Declarations

This gives the ability to group multiple `use` declarations according to the
parent namespace. This seeks to remove code verbosity when importing multiple
classes, functions, or constants that come under the same namespace.

```PHP
// Pre PHP 7 code
use some\namespace\ClassA;
use some\namespace\ClassB;
use some\namespace\ClassC as C;

use function some\namespace\fn_a;
use function some\namespace\fn_b;
use function some\namespace\fn_c;

use const some\namespace\ConstA;
use const some\namespace\ConstB;
use const some\namespace\ConstC;

// PHP 7+ code
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
```

RFC: [Group use Declarations](https://wiki.php.net/rfc/group_use_declarations)

### Generator Return Expressions

This feature builds upon the generator functionality introduced into PHP 5.5.
It enables for a `return` statement to be used within a generator to enable for
a final *expression* to be returned (return by reference is not allowed). This
value can be fetched using the new `Generator::getReturn()` method, which may
only be used once the generator has finishing yielding values.

```PHP
// IIFE syntax now possible - see the Uniform Variable Syntax subsection in the Changes section
$gen = (function() {
    yield 1;
    yield 2;

    return 3;
})();

foreach ($gen as $val) {
    echo $val, PHP_EOL;
}

echo $gen->getReturn(), PHP_EOL;

// output:
// 1
// 2
// 3
```

Being able to explicitly return a final value from a generator is a handy
ability to have. This is because it enables for a final value to be returned by
a generator (from perhaps some form of coroutine computation) that can be
specifically handled by the client code executing the generator. This is far
simpler than forcing the client code to firstly check whether the final value
has been yielded, and then if so, to handle that value specifically.

RFC: [Generator Return Expressions](https://wiki.php.net/rfc/generator-return-expressions)

### Generator Delegation

Generator delegation builds upon the ability of being able to return
expressions from generators. It does this by using an new syntax of `yield from
<expr>`, where <expr> can be any `Traversable` object or array. This <expr>
will be advanced until no longer valid, and then execution will continue in the
calling generator. This feature enables `yield` statements to be broken down
into smaller operations, thereby promoting cleaner code that has greater
reusability.

```PHP
function gen()
{
    yield 1;
    yield 2;

    return yield from gen2();
}

function gen2()
{
    yield 3;

    return 4;
}

$gen = gen();

foreach ($gen as $val)
{
    echo $val, PHP_EOL;
}

echo $gen->getReturn();

// output
// 1
// 2
// 3
// 4
```

RFC: [Generator Delegation](https://wiki.php.net/rfc/generator-delegation)

### Integer Division with `intdiv()`

The `intdiv()` function has been introduced to handle division where an integer is to be returned.

```PHP
var_dump(intdiv(10, 3)); // int(3)
```

**BC Breaks**
 - Functions in the global namespace must not be called `intdiv`.

RFC: [intdiv()](https://wiki.php.net/rfc/intdiv)

### `session_start()` Options

This feature gives the ability to pass in an array of options to the
`session_start()` function. This is used to set session-based php.ini options:

```PHP
session_start(['cache_limiter' => 'private']); // sets the session.cache_limiter option to private
```

This feature also introduces a new php.ini setting (`session.lazy_write`) that
is, by default, set to true and means that session data is only rewritten if it
changes.

RFC: [Introduce session_start() Options](https://wiki.php.net/rfc/session-lock-ini)

### `preg_replace_callback_array()` Function

This new function enables code to be written more cleanly when using the
`preg_replace_callback()` function. Prior to PHP 7, callbacks that needed to be
executed per regular expression required the callback function (second
parameter of `preg_replace_callback()`) to be polluted with lots of branching
(a hacky method at best).

Now, callbacks can be registered to each regular expression using an associative
array, where the key is a regular expression and the value is a callback.

Function Signature:
```
string preg_replace_callback_array(array $regexesAndCallbacks, string $input);
```

```PHP
$tokenStream = []; // [tokenName, lexeme] pairs

$input = <<<'end'
$a = 3; // variable initialisation
end;

// Pre PHP 7 code
preg_replace_callback(
    [
        '~\$[a-z_][a-z\d_]*~i',
        '~=~',
        '~[\d]+~',
        '~;~',
        '~//.*~'
    ],
    function ($match) use (&$tokenStream) {
        if (strpos($match[0], '$') === 0) {
            $tokenStream[] = ['T_VARIABLE', $match[0]];
        } elseif (strpos($match[0], '=') === 0) {
            $tokenStream[] = ['T_ASSIGN', $match[0]];
        } elseif (ctype_digit($match[0])) {
            $tokenStream[] = ['T_NUM', $match[0]];
        } elseif (strpos($match[0], ';') === 0) {
            $tokenStream[] = ['T_TERMINATE_STMT', $match[0]];
        } elseif (strpos($match[0], '//') === 0) {
            $tokenStream[] = ['T_COMMENT', $match[0]];
        }
    },
    $input
);

// PHP 7+ code
preg_replace_callback_array(
    [
        '~\$[a-z_][a-z\d_]*~i' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_VARIABLE', $match[0]];
        },
        '~=~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_ASSIGN', $match[0]];
        },
        '~[\d]+~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_NUM', $match[0]];
        },
        '~;~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_TERMINATE_STMT', $match[0]];
        },
        '~//.*~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_COMMENT', $match[0]];
        }
    ],
    $input
);
```

**BC Breaks**
 - Functions in the global namespace must not be called `preg_replace_callback_array`.

RFC: [Add preg_replace_callback_array Function](https://wiki.php.net/rfc/preg_replace_callback_array)

### CSPRNG Functions

This feature introduces two new functions for generating cryptographically
secure integers and strings. They expose simple APIs and are
platform-independent.

Function signatures:
```
string random_bytes(int length);
int random_int(int min, int max);
```

Both functions will emit an `Error` exception if a source of sufficient
randomness cannot be found.

**BC Breaks**
 - Functions in the global namespace must not be called `random_int` or `random_bytes`.

RFC: [Easy User-land CSPRNG](https://wiki.php.net/rfc/easy_userland_csprng)

### Support for Array Constants in `define()`

The ability to define array constants was introduced in PHP 5.6 using the
`const` keyword. This ability has now been applied to the `define()` function
too:

```PHP
define('ALLOWED_IMAGE_EXTENSIONS', ['jpg', 'jpeg', 'gif', 'png']);
```

RFC: no RFC available

### Reflection Additions

Two new reflection classes have been introduced in PHP 7. The first is
`ReflectionGenerator`, which is used for introspection on generators:

```PHP
class ReflectionGenerator
{
    public __construct(Generator $gen)
    public array getTrace($options = DEBUG_BACKTRACE_PROVIDE_OBJECT)
    public int getExecutingLine(void)
    public string getExecutingFile(void)
    public ReflectionFunctionAbstract getFunction(void)
    public Object getThis(void)
    public Generator getExecutingGenerator(void)
}
```

The second is `ReflectionType` to better support the scalar and return type
declaration features:

```PHP
class ReflectionType
{
    public bool allowsNull(void)
    public bool isBuiltin(void)
    public string __toString(void)
}
```

Also, two new methods have been introduced into `ReflectionParameter`:
```PHP
class ReflectionParameter
{
    // ...
    public bool hasType(void)
    public ReflectionType getType(void)
}
```

As well as two new methods in `ReflectionFunctionAbstract`:
```PHP
class ReflectionFunctionAbstract
{
    // ...
    public bool hasReturnType(void)
    public ReflectionType getReturnType(void)
}
```

**BC Breaks**
 - Classes in the global namespace must not be called `ReflectionGenerator` or
   `ReflectionType`.

RFC: no RFC available

## Changes

### Loosening Reserved Word Restrictions

Globally reserved words as property, constant, and method names within classes,
interfaces, and traits are now allowed. This reduces the surface of BC breaks
when new keywords are introduced and avoids naming restrictions on APIs.

This is particularly useful when creating internal DSLs with fluent interfaces:
```PHP
// 'new', 'private', and 'for' were previously unusable
Project::new('Project Name')->private()->for('purpose here')->with('username here');
```

The only limitation is that the `class` keyword still cannot be used as a
constant name, otherwise it would conflict with the class name resolution
syntax (`ClassName::class`).

RFC: [Context Sensitive Lexer](https://wiki.php.net/rfc/context_sensitive_lexer)

### Uniform Variable Syntax

This change brings far greater orthogonality to the variable operators in PHP.
It enables for a number of new combinations of operators that were previously
disallowed, and so introduces new ways to achieve old operations in terser
code.

```PHP
// nesting ::
$foo::$bar::$baz // access the property $baz of the $foo::$bar property

// nesting ()
foo()() // invoke the return of foo()

// operators on expressions enclosed in ()
(function () {})() // IIFE syntax from JS
```

The ability to arbitrarily combine variable operators came from reversing the
evaluation semantics of indirect variable, property, and method references. The
new behaviour is more intuitive and always follows a left-to-right evaluation
order:

```PHP
                        // old meaning            // new meaning
$$foo['bar']['baz']     ${$foo['bar']['baz']}     ($$foo)['bar']['baz']
$foo->$bar['baz']       $foo->{$bar['baz']}       ($foo->$bar)['baz']
$foo->$bar['baz']()     $foo->{$bar['baz']}()     ($foo->$bar)['baz']()
Foo::$bar['baz']()      Foo::{$bar['baz']}()      (Foo::$bar)['baz']()
```

**BC Breaks**
 - Code that relied upon the old evaluation order must be rewritten to
   explicitly use that evaluation order with curly braces (see middle column of
the above). This will make the code both forwards compatible with PHP 7.x and
backwards compatible with PHP 5.x

RFC: [Uniform Variable Syntax](https://wiki.php.net/rfc/uniform_variable_syntax)

### Exceptions in the Engine

Exceptions in the engine converts many fatal and recoverable fatal errors into
exceptions. This enables for graceful degradation of applications through
custom error handling procedures. It also means that cleanup-driven features
such as the `finally` clause and object destructors will now be executed.
Furthermore, by using exceptions for application errors, stack traces will be
produced for additional debugging information.

```PHP
function sum(float ...$numbers) : float
{
    return array_sum($numbers);
}

try {
    $total = sum(3, 4, null);
} catch (TypeError $typeErr) {
    // handle type error here
}
```

The new exception hierarchy is as follows:
```
interface Throwable
    |- Exception implements Throwable
        |- ...
    |- Error implements Throwable
        |- TypeError extends Error
        |- ParseError extends Error
        |- AssertionError extends Error
        |- ArithmeticError extends Error
            |- DivisionByZeroError extends ArithmeticError
```

See the [Throwable Interface](#throwable-interface) subsection in the Changes
section for more information on this new exception hierarchy.

**BC Breaks**
 - Custom error handlers used for handling (and typically ignoring) recoverable
   fatal errors will not longer work since exceptions will now be thrown
 - Parse errors occurring in `eval()`ed code will now become exceptions,
   requiring them to be wrapped in a `try...catch` block

RFC: [Exceptions in the Engine](https://wiki.php.net/rfc/engine_exceptions_for_php7)

### Throwable Interface

This change affects PHP's exception hierarchy due to the introduction of
[exceptions in the engine](#exceptions-in-the-engine). Rather than placing
fatal and recoverable fatal errors under the pre-existing `Exception` class
hierarchy, [it was
decided](https://wiki.php.net/rfc/engine_exceptions_for_php7#doodle__form__introduce_and_use_baseexception)
to implement a new hierarchy of exceptions to prevent PHP 5.x code from
catching these new exceptions with catch-all (`catch (Exception $e)`) clauses.

The new exception hierarchy is as follows:
```
interface Throwable
    |- Exception implements Throwable
        |- ...
    |- Error implements Throwable
        |- TypeError extends Error
        |- ParseError extends Error
        |- AssertionError extends Error
        |- ArithmeticError extends Error
            |- DivisionByZeroError extends ArithmeticError
```

The `Throwable` interface is implemented by both `Exception` and `Error` base
class hierarchies and defines the following contract:
```
interface Throwable
{
    final public string getMessage ( void )
    final public mixed getCode ( void )
    final public string getFile ( void )
    final public int getLine ( void )
    final public array getTrace ( void )
    final public string getTraceAsString ( void )
    public string __toString ( void )
}
```

`Throwable` cannot be implemented by user-defined classes - instead, a custom
exception class should extend one of the pre-existing exceptions classes in
PHP.

RFC: [Throwable Interface](https://wiki.php.net/rfc/throwable-interface)

### Integer Semantics

The semantics for some integer-based behaviour has changed in an effort to make
them more intuitive and platform-independent. Here is a list of those changes:
 - Casting `NAN` and `INF` to an integer will always result in 0
 - Bitwise shifting by a negative number of bits is now disallowed (causes a
   bool(false) return and emits an E_WARNING)
 - Left bitwise shifts by a number of bits beyond the bit width of an integer will always result in 0
 - Right bitwise shifts by a number of bits beyond the bit width of an integer
   will always result in 0 or -1 (sign dependent)

**BC Breaks**
 - Any reliance on the old semantics for the above will no longer work

RFC: [Integer Semantics](https://wiki.php.net/rfc/integer_semantics)

### JSON Extension Replaced with JSOND

The licensing of the old JSON extension was regarded as non-free, causing
issues for many Linux-based distributions. The extension has since been
replaced with JSOND and comes with some [performance
gains](https://github.com/bukka/php-jsond-bench/blob/master/reports/0001/summary.md)
and backward compatibility breakages.

**BC Breaks**
 - A number *must not* end in a decimal point (i.e. `34.` must be changed to either `34.0` or just `34`)
 - The `e` exponent *must not* immediately follow the decimal point (i.e.
   `3.e3` must be changed to either `3.0e3` or just `3e3`)

RFC: [Replace current json extension with jsond](https://wiki.php.net/rfc/jsond)

### ZPP Failure on Overflow

Coercion between floats to integers can occur when a float is passed to an
internal function expecting an integer. If the float is too large to represent
as an integer, then the value will be silently truncated (which may result in a
loss of magnitude and sign). This can introduce hard-to-find bugs. This change
therefore seeks to notify the developer when an implicit conversion from a
float to an integer has occurred and failed by returning `null` and emitting an
E_WARNING.

**BC Breaks**
 - Code that once silently worked will now emit an E_WARNING and may fail if
   the result of the function invocation is directly passed to another function
(since `null` will now be passed in).

RFC: [ZPP Failure on Overflow](https://wiki.php.net/rfc/zpp_fail_on_overflow)

### Fixes to `foreach()`'s Behaviour

PHP's `foreach()` loop had a number of strange edge-cases to it. These were all
implementation-driven and caused a lot of undefined and inconsistent behaviour
when iterating between copies and references of an array, when using iterator
manipulators like `current()` and `reset()`, when modifying the array currently
being iterated, and so on.

This change eliminates the undefined behaviour of these edge-cases and makes
the semantics more predictable and intuitive.

`foreach()` by value on arrays
```PHP
$array = [1,2,3];
$array2 = &$array;

foreach($array as $val) {
    unset($array[1]); // modify array being iterated over
    echo "{$val} - ", current($array), PHP_EOL;
}

// Pre PHP 7 result
1 - 3
3 -

// PHP 7+ result
1 - 1
2 - 1
3 - 1
```

When by-value semantics are used, the array being iterated over is now not
modified in-place. `current()` also now has defined behaviour, where it will
always begin at the start of the array.

`foreach()` by reference on arrays and objects and by value on objects
```PHP
$array = [1,2,3];

foreach($array as &$val) {
    echo "{$val} - ", current($array), PHP_EOL;
}

// Pre PHP 7 result
1 - 2
2 - 3
3 -

// PHP 7+ result
1 - 1
2 - 1
3 - 1
```

The `current()` function is no longer affected by `foreach()`'s iteration on
the array. Also, nested `foreach()`'s using by-reference semantics work
independently from each other now:
```PHP
$array = [1,2,3];

foreach($array as &$val) {
    echo $val, PHP_EOL;

    foreach ($array as &$val2) {
        unset($array[1]);
        echo $val, PHP_EOL;
    }
}

// Pre PHP 7 result
1
1
1

// PHP 7+ result
1
1
1
3
3
3
```

**BC Breaks**
 - Any reliance on the old (quirky and undocumented) semantics will no longer work.

RFC: [Fix "foreach" behavior](https://wiki.php.net/rfc/php7_foreach)

### Changes to `list()`'s Behaviour

The `list()` function was documented as not supporting strings, however in few cases strings could have been used:
```PHP
// array dereferencing
$str[0] = 'ab';
list($a, $b) = $str[0];
echo $a; // a
echo $b; // b

// object dereferencing
$obj = new StdClass();
$obj->prop = 'ab';
list($a, $b) = $obj->prop;
echo $a; // a
echo $b; // b

// function return
function func()
{
    return 'ab';
}

list($a, $b) = func();
var_dump($a, $b);
echo $a; // a
echo $b; // b
```

This has now been changed making string usage with `list()` forbidden in all cases.

Also, empty `list()`'s are now a fatal error, and the order of assigning variables has been changed to left-to-right:
```PHP
$a = [1, 2];
list($a, $b) = $a;

// OLD: $a = 1, $b = 2
// NEW: $a = 1, $b = null + "Undefined index 1"

$b = [1, 2];
list($a, $b) = $b;

// OLD: $a = null + "Undefined index 0", $b = 2
// NEW: $a = 1, $b = 2
```

**BC Breaks**
 - Making `list()` equal to any non-direct string value is no longer possible.
   `null` will now be the value for the variable `$a` and `$b` in the above
examples
 - Invoking `list()` without any variables will cause a fatal error
 - Reliance upon the old right-to-left assignment order will no longer work

RFC: [Fix list() behavior inconsistency](https://wiki.php.net/rfc/fix_list_behavior_inconsistency)

RFC: [Abstract syntax tree](https://wiki.php.net/rfc/abstract_syntax_tree)

### Changes to Division by Zero Semantics

Prior to PHP 7, when a divisor was 0 for either the divide (/) or modulus (%) operators,
an E_WARNING would be emitted and `false` would be returned. This was nonsensical for
an arithmetic operation to return a boolean in some cases, and so the behaviour has been
rectified in PHP 7.

The new behaviour causes the divide operator to return a float as either +INF, -INF, or
NAN. The modulus operator E_WARNING has been removed and (alongside the new `intdiv()`
function) will throw a `DivisionByZeroError` exception. In addition, the `intdiv()`
function may also throw an `ArithmeticError` when valid integer arguments are supplied
that cause an incorrect result (due to integer overflow).

```PHP
var_dump(3/0); // float(INF) + E_WARNING
var_dump(0/0); // float(NAN) + E_WARNING

var_dump(0%0); // DivisionByZeroError

intdiv(PHP_INT_MIN, -1); // ArithmeticError
```

**BC Breaks**
 - The divide operator will no longer return `false` (which could have been silently coerced
 to 0 in an arithmetic operation)
 - The modulus operator will now throw an exception with a 0 divisor instead of returning `false`

RFC: No RFC available

### Fixes to Custom Session Handler Return Values

When implementing custom session handlers, predicate functions from the
`SessionHandlerInterface` that expect a `true` or `false` return value did not
behave as expected. Due to an error in the previous implementation, only a `-1`
return value was considered false - meaning that even if the boolean
`false` was used to denote a failure, it was taken as a success:
```PHP
<?php

class FileSessionHandler implements SessionHandlerInterface
{
    private $savePath;

    function open($savePath, $sessionName)
    {
        return false; // always fail
    }

    function close(){return true;}

    function read($id){}

    function write($id, $data){}

    function destroy($id){}

    function gc($maxlifetime){}
}

session_set_save_handler(new FileSessionHandler());

session_start(); // doesn't cause an error in pre PHP 7 code
```

Now, the above will fail with a fatal error. Having a `-1` return value will
also continue to fail, whilst `0` and `true` will continue to mean success. Any
other value returned will now cause a failure and emit an E_WARNING.

**BC Breaks**
 - If boolean `false` is returned, it will actually fail now
 - If anything other than a boolean, `0`, or `-1` is returned, it will fail and cause a warning to be emitted

RFC: [Fix handling of custom session handler return values](https://wiki.php.net/rfc/session.user.return-value)

### Deprecation of PHP 4-Style Constructors

PHP 4 constructors were preserved in PHP 5 alongside the new `__construct()`.
Now, PHP 4-style constructors are being deprecated in favour of having only a
single method (`__construct()`) to be invoked on object creation. This is
because the conditions upon whether the PHP 4-style constructor was invoked
caused additional cognitive overhead to developers that could also be confusing
to the inexperienced.

For example, if the class is defined within a namespace or if an
`__construct()` method existed, then a PHP 4-style constructor was recognised
as a plain method. If it was defined above an `__construct()` method, then an
E_STRICT notice would be emitted, but still recognised as a plain method.

Now in PHP 7, if the class is not in a namespace and there is no
`__construct()` method present, the PHP 4-style constructor will be used as a
constructor but an E_DEPRECATED will be emitted. In PHP 8, the PHP 4-style
constructor will always be recognised as a plain method and the E_DEPRECATED
notice will disappear.

**BC Breaks**
 - Custom error handlers may be affected by the raising of E_DEPRECATED
   warnings. To fix this, simply update the class constructor name to
`__construct`.

RFC: [Remove PHP 4 Constructors](https://wiki.php.net/rfc/remove_php4_constructors)

### Removal of date.timezone Warning

When any date- or time-based functions were invoked and a default timezone had
not been set, a warning was emitted. The fix was to simply set the
`date.timezone` INI setting to a valid timezone, but this forced users to have
a php.ini file and to configure it beforehand. Since this was the only setting
that had a warning attached to it, and it defaulted to UTC anyway, the warning
has now been removed.

RFC: [Remove the date.timezone warning](https://wiki.php.net/rfc/date.timezone_warning_removal)

### Removal of Alternative PHP Tags

The alternative PHP tags `<%` (and `<%=`), `%>`, `<script language="php">`, and
`</script>` have now been removed.

**BC Breaks**
 - Code that relied upon these alternative tags needs to be updated to either
   the normal or short opening and closing tags. This can either be done
   manually or automated with [this porting script](https://gist.github.com/nikic/74769d74dad8b9ef221b).

RFC: [Remove alternative PHP tags](https://wiki.php.net/rfc/remove_alternative_php_tags)

### Removal of Multiple Default Blocks in Switch Statements

Previously, it was possible to specify multiple `default` block statements
within a switch statement (where the last `default` block was only executed).
This (useless) ability has now been removed and causes a fatal error.

**BC Breaks**
 - Any code written (or more likely generated) that created switch statements
   with multiple `default` blocks will now become a fatal error.

RFC: [Make defining multiple default cases in a switch a syntax error](https://wiki.php.net/rfc/switch.default.multiple)

### Removal of Redefinition of Parameters with Duplicate Names

Previously, it was possible to specify parameters with duplicate names within a function definition.
This ability has now been removed and causes a fatal error.

```PHP
function foo($version, $version)
{
    return $version;
}

echo foo(5, 7);
  
// Pre PHP 7 result
7

// PHP 7+ result
Fatal error: Redefinition of parameter $version in /redefinition-of-parameters.php
```

**BC Breaks**
 - Function parameters with duplicate name will now become a fatal error.

### Removal of Dead Server APIs

The following SAPIs have been removed from the core (most of which have been moved to PECL):
- sapi/aolserver
- sapi/apache
- sapi/apache_hooks
- sapi/apache2filter
- sapi/caudium
- sapi/continuity
- sapi/isapi
- sapi/milter
- sapi/nsapi
- sapi/phttpd
- sapi/pi3web
- sapi/roxen
- sapi/thttpd
- sapi/tux
- sapi/webjames
- ext/mssql
- ext/mysql
- ext/sybase_ct
- ext/ereg

RFC: [Removal of dead or not yet PHP7 ported SAPIs and extensions](https://wiki.php.net/rfc/removal_of_dead_sapis_and_exts)

### Removal of Hex Support in Numerical Strings

A Stringy hexadecimal number is no longer recognised as numerical.
```PHP
var_dump(is_numeric('0x123'));
var_dump('0x123' == '291');
echo '0x123' + '0x123';

// Pre PHP 7 result
bool(true)
bool(true)
582

// PHP 7+ result
bool(false)
bool(false)
0
```

The reason for this change is to promote better consistency between the
handling of stringy hex numbers across the language. For example, explicit
casts do not recognise stringy hex numbers:
```PHP
var_dump((int) '0x123'); // int(0)
```

Instead, stringy hex numbers should be validated and converted using the `filter_var()` function:
```PHP
var_dump(filter_var('0x123', FILTER_VALIDATE_INT, FILTER_FLAG_ALLOW_HEX)); // int(291)
```

**BC Breaks**
 - This change affects the `is_numeric()` function and various operators, including `==`, `+`, `-`, `*`, `/`, `%`, `**`, `++`, and `--`

RFC: [Remove hex support in numeric strings](https://wiki.php.net/rfc/remove_hex_support_in_numeric_strings)

### Removal of Deprecated Functionality

All Deprecated functionality has been removed, most notably:
 - The original mysql extension (ext/mysql)
 - The ereg extension (ext/ereg)
 - Assigning `new` by reference
 - Scoped calls of non-static methods from an incompatible `$this` context
   (such as `Foo::bar()` from outside a class, where `bar()` is not a static
method)

**BC Breaks**
 - Any code that ran with deprecation warnings in PHP 5 will no longer work (you were warned!)

RFC: [Remove deprecated functionality in PHP 7](https://wiki.php.net/rfc/remove_deprecated_functionality_in_php7)

### Reclassification and Removal of E_STRICT Notices

E_STRICT notices have always been a bit of a grey area in their meaning. This
changes removes this error category altogether and either: removes the E_STRICT
notice, changes it to an E_DEPRECATED if the functionality will be removed in
future, changes it to an E_NOTICE, or promotes it to an E_WARNING.

**BC Breaks**
 - Because E_STRICT is in the lowest severity error category, any error
   promotions to an E_WARNING may break custom error handlers

RFC: [Reclassify E_STRICT notices](https://wiki.php.net/rfc/reclassify_e_strict)

### Deprecation of Salt Option for `password_hash()`

With the introduction of the new password hashing API in PHP 5.5, many began
implementing it and generating their own salts. Unfortunately, many of these
salts were generated from cryptographically insecure functions like mt_rand(),
making the salt far weaker than what would have been generated by default.
(Yes, a salt is always used when hashing passwords with this new API!) The option to
generate salts have therefore been deprecated to prevent developers from
creating insecure salts.

RFC: no RFC available

### Error on Invalid Octal Literals

Invalid octal literals will now cause a parse error rather than being
truncated and silently ignored.

```PHP
echo 0678; // Parse error:  Invalid numeric literal in...
```

**BC Breaks**
 - Any invalid octal literals in code will now cause parse errors

RFC: no RFC available

### `substr()` Return Value Change

`substr()` will now return an empty string instead of `false` when the start
position of the truncation is equal to the string length:
```PHP
var_dump(substr('a', 1));

// Pre PHP 7 result
bool(false)

// PHP 7+ result
string(0) ""
```

`substr()` may still return `false` in other cases, however.

**BC Breaks**
 - Code that strictly checked for a `bool(false)` return value may now be
 semantically invalid

RFC: no RFC available

## FAQ

### What happened to PHP 6?

PHP 6 was the major PHP version that never came to light. It was supposed to
feature full support for Unicode in the core, but this effort was too ambitious
with too many complications arising. The predominant reasons why version 6 was
skipped for this new major version are as follows:
 - **To prevent confusion**. Many resources were written about PHP 6 and much
   of the community knew what was featured in it. PHP 7 is a completely
different beast with entirely different focuses (specifically on performance)
and entirely different feature sets. Thus, a version has been skipped to
prevent any confusion or misconceptions surrounding what PHP 7 is.
 - **To let sleeping dogs lie**. PHP 6 was seen as a failure and a large amount
   of PHP 6 code still remains in the PHP repository. It was therefore seen as
best to move past version 6 and start afresh on the next major version, version
7.

RFC: [Name of Next Release of PHP](https://wiki.php.net/rfc/php6)
