### 输入流处理基本单元
| 特性        | 字节定向 (Byte-Oriented) | 宽定向 (Wide-Oriented)     |
|-------------|-----------------------|--------------------------|
| 基本单元    | 字节 (byte)             | 宽字符 (wide character)    |
| 主要用途    | 二进制数据，单字节编码文本 | 多字节编码文本，国际化应用 |
| 处理多字节字符 | 需要手动组合字节         | 直接处理完整字符         |
| 效率        | 通常较高                | 可能略低                 |
| 平台依赖性   | 较低                   | 较高（`wchar_t` 大小）     |
| 复杂性      | 处理多字节字符较复杂    | 处理多字节字符更简单     |



### ISO C 缓冲类型
+ 当且仅当`stdin`和`stdout`不指向终端时，缓冲类型为全缓冲
+ `stderr`不是全缓冲，默认无缓冲
+ `stdin`如果指向终端，默认行缓冲，否则是全缓冲
+ 可以调用`setvbuf`函数设置缓冲类型
+ `fflush(FILE *fp)`可以强制刷新缓冲区，如果`fp`为空，则刷新所有缓冲区
+ `setbuf(FILE *fp, char *buf)`可以设置缓冲区，`buf`为空时，取消缓冲


**`gets()`是一个不推荐使用的函数，因为`gets()`函数不检查缓冲区大小，容易导致缓冲区溢出**



| 模式 | 描述 |
| ---- | ---- |
| r 或 rb | 为读而打开 |
| w 或 wb | 为写而打开 |
| a 或 ab | 追加：为在第一个 null 字节处写而打开 |
| r+ 或 r+b 或 rb+ | 为读和写而打开 |
| w+ 或 w+b 或 wb+ | 把文件截断至 0 长，为读和写而打开 |
| a+ 或 a+b 或 ab+ | 追加：为在第一个 null 字节处读和写而打开 |
    
```c
#include <stdio.h>

int main(void) {
    int c; // 将 c 声明为 int

    while ((c = getchar()) != EOF) {
        putchar(c);
    }

    return 0;
}
```
这段代码的问题在于用于存储 getchar() 返回值的变量 c 的数据类型不正确。

**问题分析：**

getchar() 的返回值类型： getchar() 函数从标准输入读取一个字符，并以 int 类型返回该字符的 ASCII 值。如果遇到文件结束符 (EOF)，则返回 EOF（通常定义为 -1）。

char 类型可能无法表示 EOF： 变量 c 被声明为 char 类型。 char 类型的大小和符号性（signed 或 unsigned）在不同的编译器和平台上可能有所不同。

如果 char 是有符号的 (signed char)： 它通常可以表示 -1，因此在遇到 EOF 时，c 可以存储 EOF 的值，循环条件 c != EOF 能够正确判断，程序正常结束。

如果 char 是无符号的 (unsigned char)： 它无法表示负数。当 getchar() 返回 EOF (-1) 时，将 -1 赋值给一个 unsigned char 变量会发生类型转换，通常会将 -1 转换为该无符号类型的最大值（例如 255）。这样，循环条件 c != EOF 永远为真，导致程序进入无限循环。

为什么在一些机器上运行正确，而在另一些机器上运行出错：

编译器和平台的差异： 不同的编译器或操作系统平台可能将 char 类型默认实现为 signed char 或 unsigned char。这就是导致行为不一致的原因。

**正确的做法：**

为了确保代码的跨平台兼容性和正确性，应该将用于存储 getchar() 返回值的变量声明为 int 类型。