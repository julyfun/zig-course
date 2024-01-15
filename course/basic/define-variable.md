---
outline: deep
---

# 基本类型

> 变量的声明和定义是编程语言中最基础且最常见的操作之一。

## 变量声明

> 变量是在内存中存储值的单元。

在 zig 中，我们使用 `var` 来进行变量的声明，格式是 `var variable:type = value;`，以下是一个示例：

```zig
const std = @import("std");

pub fn main() void {
    // 声明变量 variable 类型为u16, 并指定值为 666
    var variable: u16 = 666;

    std.debug.print("变量 variable 是{}\n", .{variable});
}
```

::: info 🅿️ 提示

当前 zig 的 `nightly` 版本不允许使用不会被修改的变量，即尽可能使用常量。

同时，zig 还要求所有的非顶层定义的变量（常量）均被使用，如果未被使用编译器会报告错误，但可通过将其分配给 `_` 来解决此问题。

:::

### 标识符命名

在 zig 中，**_禁止变量覆盖外部作用域_**！

命名须以 **_字母_** 或者 **_下划线_** 开头，后跟任意字母数字或下划线，并且不得与关键字重叠。

如果一定要使用不符合这些规定的名称（例如与外部库的链接），那么请使用 `@""` 语法。

```zig
const @"identifier with spaces in it" = 0xff;
const @"1SmallStep4Man" = 112358;

const c = @import("std").c;
pub extern "c" fn @"error"() void;
pub extern "c" fn @"fstat$INODE64"(fd: c.fd_t, buf: *c.Stat) c_int;

const Color = enum {
  red,
  @"really red",
};
const color: Color = .@"really red";
```

### 常量

zig 使用 `const` 作为关键字来声明常量，它无法再被更改，只有初次声明时可以赋值。

```zig
const std = @import("std");

pub fn main() void {
    const constant: u16 = 666;

    std.debug.print("常量 constant 是{}\n", .{constant});
}
```

注意： 所有常量都是 [_编译期_](/advanced/comptime) 已知的！

### `undefined`

我们可以使用 `undefined` 使变量保持未初始化状态。

```zig
const std = @import("std");

pub fn main() void {
    var variable: u16 = undefined;

    variable = 666;

    std.debug.print("变量 variable 是{}\n", .{variable});
}
```

::: warning ⚠️ 警告

慎重使用 `undefined`，如果一个变量是未定义的，使用它出现无法预知的情况。

当一个变量未定义时，那么它将不会执行默认的初始化操作，此时变量的值将会是无意义的值（我们绝对不能在赋值前使用它）。

在 `Debug` 模式下，Zig 将 `0xaa` 字节写入未定义的内存。这是为了尽早发现错误，并帮助检测调试器中未定义内存的使用。但是，此行为只是一种实现功能，而不是语言语义，因此不能保证代码可以观察到它。
:::

## 块

块（block）用于限制变量声明的范围，例如以下代码是非法的：

```zig
{
    var x: i32 = 1;
    _ = &x;
}
x += 1;
```

块也可以是一个表达式，当它有标签时，`break` 会从块中返回一个值出来。

```zig
var y: i32 = 123;

const x = blk: {
    y += 1;
    break :blk y;
};
```

上方的 `blk` 是标签名字，它可以是你设置的任何名字。

## 注释

先来看一下在 zig 中如何正确的书写注释，zig 本身支持三种注释方式，分别是普通注释、文档注释、定义文档注释。

`//` 就是普通的注释，就只是和其他编程语言中 `//` 起到的注释效果相同。

::: details 小细节
值得一提的是，zig 本身并未提供类似`/* */` 这种多行注释，这意味着多行注释的最佳实践形式就是多行的`//`了。

PS:说实话，我认为这个设计并不太好。
:::

`///` 就是文档注释，用于给函数、类型、变量等这些提供注释，文档注释记录了紧随其后的内容。

```zig
/// 存储时间戳的结构体，精度为纳秒
/// (像这里就是多行文档注释)
const Timestamp = struct {

    /// 自纪元开始后的秒数 (此处也是一个文档注释).
    seconds: i64,  // 我们可以以此代表1970年前 (此处是普通注释)

    /// 纳秒数 (文档注释).
    nanos: u32,

    /// 返回一个 Timestamp 结构体代表 unix 纪元;
    /// 1970年 1月1日 00:00:00 UTC (文档注释).
    pub fn unixEpoch() Timestamp {
        return Timestamp{
            .seconds = 0,
            .nanos = 0,
        };
    }
};
```

`//!` 是顶层文档注释，通常用于记录一个文件的作用，**必须放在作用域的顶层，否则会编译错误**

```zig
//! 顶层文档注释
//! 顶层文档注释

const S = struct {
    //! 顶层文档注释
};
```

::: details 小细节
为什么是作用域顶层呢？实际上，zig 将一个源码文件看作是一个容器。
:::