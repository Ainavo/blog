---
title: C 语言程序设计中一些值得注意的点
date: 2023-07-15 13:14:46
tags:
  - C
---

_Notes_:参考课程[<<浙大瓮恺 C 语言程序设计>>](https://space.bilibili.com/1355742754)

<!--more-->

- scanf() 中格式字符中的字符是必须输入的：

  ```C
      scanf("print %d", &a) //终端中输入必须是 "print xxx"
  ```

- 运算符和算子

  ```C
    a = b + c // a, b, c 算子； =, + 运算符；
  ```

- 运算符优先级
  | 优先级 | 运算符 | 运算 | 结合关系 | 示例 |
  | ---- | ---- | ---- | ---- | ---- |
  | 1 | + | 单目不变 | 自右向左 | a*+b |
  | 1 | - | 单目取负 | 自右向左 | a*-b |
  | 2 | \* | 乘 | 自左向右 | a\*b |
  | 2 | / | 除 | 自左向右 | a/b |
  | 2 | % | 取余 | 自左向右 | a%b |
  | 3 | + | 加 | 自左向右 | a+b |
  | 3 | - | 减 | 自左向右 | a-b |
  | 4 | = | 赋值 | 自右向左 | a=b|

- a++ 和 ++a

  ```C
    ++1; // 返回结果 2；并实现 +1 功能；
    1++; // 返回结果 1；并实现 +1 功能；
  ```

- 循环 tips：

  - 如果有固定次数，用 for 循环： for(; ; ;)
  - 如果必须执行一次：用 do_while 循环
  - 其他情况用 while 循环

- 跳出循环：

  - break：跳出循环
  - continue：跳出本轮循环，继续下一轮
  - goto：
    ```C
        ....
        goto out:
    out:
        return 0;
    ```

- 辗转相除法：

  ```C
    // 如果b等于0，计算结束，a就是最大公约数；
    // 否则，计算a除以b的余数，让a等于b，b等于那个余数；
    // 构建循环体。
    #include <stdio.h>

    int main(){
        int a, b;
        int temp;
        scanf("%d %d", &a, &b);
        while(b != 0){
            temp = a % b;
            a = b;
            b = temp;
            printf("a=%d, b=%d, temp=%d\n", a, b, temp);
        };
        printf("a与b的最大公约数为 %d", a);
        return 0;
    }
  ```

- 数字类型

  - 类型名称：int、long、double
  - 输入输出时的格式化：%d、%ld、%lf
  - 所表达数的范围：char < short < int < float < double
  - 内存中所占据的大小：1 个字节到 16 个字节
  - 内存中的表达形式：二进制数（补码）、编码

- sizeof：一个 int 表达一个寄存器的大小

  ```C
  #include<stdio.h>
  int main(){
    printf("sizeof(char) = %ld\n", sizeof(char)); // 1字节（8比特）
    printf("sizeof(short) = %ld\n", sizeof(short)); // 2字节
    printf("sizeof(int) = %ld\n", sizeof(int)); // 取决于编译器（CPU）
    printf("sizeof(long) = %ld\n", sizeof(long)); // 取决于编译器（CPU）
    printf("sizeof(long long) = %ld\n", sizeof(long long)); // 8字节
    return 0;
  }
  ```

- 补码的意义：补码和原码相加可以得到一个溢出的 0

- 整数的范围：对于一个字节（8 位）可以表达的是：

  - 00000000~11111111
  - 11111111~10000000-->-1~-128(作为补码来看)
  - 00000001~01111111--> 1~127

  ```C
    #include<stdio.h>

    int main(){
        char c = 255;
        int i = 255;
        printf("c=%d, i=%d\n", c, i); // c=-1, i=255
        // 11111111
        // 00000000 00000000 00000000 11111111
        return 0;
    }
  ```

  - char(1 字节):-128~127
  - short(2):-32768~32767
  - int:取决于编译器（CPU），通常意义上是"1 个字" $-2^{32}-1$ ~ $2^{32-1}-1$
  - long(4 字节)
  - long long(8 字节)

- unsigned:表示纯二进制(主要为了做移位)

  ```C
     #include<stdio.h>

      int main(){
          unsigned char a = 255;
          int i =255;
          printf("c=%d, i=%d\n", c, i);
          // 00000000 - 11111111 0~255 正常：-128~127
          // 00000000 00000000 00000000 11111111
          return 0;
      }
  ```

- 整数输入输出：int 和 long long

  - %d:int
  - %u:unsigned
  - %ld:long long
  - %lu: unsigned long long

- 浮点数类型：
  | 类型 | 字长 | 范围 | 有效数字 | scanf | printf |
  | ---- | ---- | ---- | ---- | ---- | ---- |
  | float | 32 | ±(1.2x $10^{-38}$~3.40x$10^{38}$), 0, ±inf, nan| 7 | %f |%f, %e(科学计数) |
  | double | 64 | ±(2.2x $10^{-308}$~1.79x$10^{308}$), 0, ±inf, nan | 15 | %lf | %f, %e(科学计数) |

- 超过范围的浮点数：

  - printf 输出 inf 表示无穷大
  - printf 输出 nan 表示不存在的浮点数

- 浮点运算的精度

  - 带小数点的字面量意思是 double 而非 float
  - float 需要用 f 或 F 后缀来表明身份
    ```C
        float a, b, c;
        a = 1.345f;
        b = 1.123f;
        c = a + b;
        if (c == 2.468)
            printf("相等\n");
        else
            printf("不相等！c=%.10f, 或%f\n", c, c)  // f1 == f2 可能失败 ； fabs(f1-f2) < 1e-12 验证;
    ```

- 浮点数的内部表达：

  - sign(1 bit)：符号位
  - exponent(11 bit)：指数位
  - fraction(52 bit)：小数位

- 字符类型

  - char(字符)：printf 和 scanf 用%c 输入输出
    ```C
        int main(){
            char a;
            a = '1';
            printf("c=%d", a);  // 1;
            printf("c=%c", a);  // 49 ASIIC 码
        }
    ```

- 逃逸字符：用来表达无法打印出来的控制字符或者特殊字符，他由一个反斜杠"\"开头，后面跟上另一个字符，这两个字符合起来，组成一个字符。
  | 字符 | 意义 | 字符 | 意义 |
  | ---- | ---- | ---- | ---- |
  | \b | 回退一格 | \" | 双引号 |
  | \t | 到下一个表格位 | \' | 单引号 |
  | \n | 换行 | \\ | 反斜杠本身 |
  | \r | 回车 | | |

- 自动类型转换：

  - 当运算符两边出现不一样类型时，会自动转换成较大（范围大）的类型；
  - char-->short-->int-->long-->long long
  - int-->float-->double
  - 注：
    - 对于 printf，任何小于 int 类型的都会转换成 int；float 会被转换成 double；
    - 对于 scanf 则不会，要输入 short，需要 %hd

- 强制转换：优先级高于四则运算

  - (类型)值
    ```C
    (int)32.3
    (short)32
    ```
  - 注意小的变量不总能表达大的变量

- 逻辑类型(bool)和逻辑运算
  | 运算符 | 描述 | 示例 | 结果 |
  | ---- | ---- | ---- | ---- |
  | ! | 逻辑非 | !a | 如果 a 是 true 结果就是 false；如果 a 是 false，结果就是 true； |
  | && | 逻辑与 | a && b | 如果 a 和 b 都是 true，结果就是 true；否则就是 false； |
  | \|\| | 逻辑或 | a || b | 如果 a 和 b 中有一个是 true，结果为 true；两个都为 false，结果为 false； |

- 运算符优先级
  | 优先级 | 运算符 | 结合性 |
  | ---- | ---- | ---- |
  | 1 | （） | 从左到右 |
  | 2 | ! + - ++ -- | 从右到左（单目的+和-） |
  | 3 | _ / % | 从左到右 |
  | 4 | + - | 从左到右 |
  | 5 | <<= >>= | 从左到右 |
  | 6 | == != | 从左到右 |
  | 7 | && | 从左到右 |
  | 8 | \|\| | 从左到右 |
  | 9 | = += -= _= /= %=| 从左到右 |

- 条件运算符：自右向左

  ```c
    m<n ?  a:a+5 // (条件) ? a : b
  ```

- 逗号运算符：所有运算符中优先级最低的

  ```C
    for(i=0, j=10; i<j; i++, j--) // 基本上只有 for 条件会用到
  ```

- 本地变量

  - 生存期：什么时候这个变量出现了，到什么时候它消亡了
  - 作用域：在代码什么范围内可以访问这个变量（这个变量起作用）
  - 对于本地变量，这两个问题答案是统一的：大括号内--块
    - 程序运行进入这个块之前，其中的变量不存在，离开这个块，其中的变量就消失了
    - 块外面定义的变量在里面仍然有效
    - 快里面定义了和外面同名的变量则掩盖了外面的
    - 不能在一 个块中定义同名变量

- 函数声明：

  ```C
      void swap(double a, double b); // 与函数名称相同
      void swap(void)  // 无参数
  ```

- 数组的定义

  - <类型> 变量名[元素数量]；
  - int grades[100];
  - double weight[20];

- 元素必须是整数；
  - 数组：是一种容器(存放东西的东西)
  - 其中所有元素都具有相同的数据类型；
  - 一旦创建，不能改变大小；
  - \*(数组中元素在内存中是连续依次排列的) -数组初始化：

```C
   int a[] = {1, 2, 3};
```

- C99 中支持：

  ```C
      int a[10] = {[0] = 2, [2] = 3, 6};
  ```

  - 用[n]在初始化数据中给出定位
  - 没有定位的数据接在前面的位置后面，其余位置自动补零
  - 也可以不给出数组大小，让编译器算
  - 适合初始数据稀疏的数组

- 数组的大小：sizeof(a)/sizeof(a[0])

- 数组的赋值：

  ```C
      int a[] = {1, 2, 3,};
      int b[] = a; // error
  ```

  - 数组不能直接赋值
  - 把一个数组元素交给另一个数组，必须采用遍历

- 数组作为函数的参数时：

  - 数组作为函数参数，往往必须再用另一个参数来传入数组的长度
  - 不能在 a[] 中 [] 给出数组大小
  - 不能利用 sizeof 计算元素个数

- 二维数组：

  - a[i][j]是一个 int：表示第 i 行第 j 列
  - a[i, j]错误表达：表示 a[j] ("**,**"运算符)
  - 列数必须有，行数可以省略

- 运算符 &

  - scanf("%d, &d)中&：获取变量的地址
  - &(a+b), &(a++), &(++a) 都会报错， &右边只能是固定数

- 指针：就是保存地址的变量

  ```C
    int i;
    int* p = &1;
    int* p, q;  // p是一个指针，指向int 同下；q不是一个指针；
    int *p, q; // 同上
  ```

- 指针作为参数：

  ```C
    void f(int *p);
    int main(){
        int i = 6;
        f(&i);
    }
  ```

- 访问地址变量\*：

  - \*是一个单目运算符，用来访问指针的值表示的地址上的变量
  - 可以作为左值或右值：
    - int k = \*p
    - \*p = k+1

- 指针应用场景：

  - 交换两个值
  - 函数返回运算状态，结果通过指针返回

- 指针最常见错误：指针未指向地址，就赋值

- 以下四种函数原型是等价的：

  - int sum(int \*ar, int n);
  - int sum(int \*, int);
  - int sum(int ar[], int n);
  - int sum(int[], int);

- 数组变量就是特殊的指针：
  - 数组变量本身表达地址：
    - int a[10]; int\*p = a; //无需用&取地址
    - 但是数组的单元表达的是变量，需要用&去取地址
    - a == &a[0]
  - [] 运算符可以对数组做，也可以对指针做：
    - p[0]<==>a[0]
  - \*运算符可以对指针做，也可以对数组做：
    - \*a = 25;
  - 数组变量是 const 指针：
    ```C
    int a[] <==> int * const a // 因此两个数组间不能直接赋值
    ```
- 指针与 const：

  - 指针是 const：表示一旦得到了某个变量的地址，不能再指向其他变量：
    ```C
    int *const q = &i; // q 是 const
    *q = 26； // OK
    q++; // error
    ```
  - 所指是 const：表示不能通过这个指针去修改那个变量（并不能使得那个变量成为 const）

  ```C
    const int*p = &i;
    *p = 26; //EEROR! (*p)是const
    i = 26; //OK
    p = &j; //OK
    // i 可以变；p 可以变；*p 不可以变；
  ```

  - 例子：判断 const 和*的位置关系--const 在\*前 *p 不能被修改；const 在\*后 p(地址不能被修改)

  ```C
    int i;
    const int* p1 = &i;
    int const* p2 = &i;
    int *const p3 = &i;
    // 技巧：主要看 const 修饰的是 *p，还是 p
  ```

- 指针的计算：

  - \*(p+1)、\*(p++)、\*(p--)
  - \*p++:
    - 取出 p 所指的那个数据，完事顺便把 p 移到下个位置
    - \*的优先级虽然高，但是没有++高
    - 常用于数组类的连续空间操作
    - 在某些 CPU 上，这可以直接被翻译成一条汇编指令
    ```C
        #include <stdio.h>
        int main(void){
            char ac[] = {0, 1, 2, 3, 4, -1};
            char *p = &ac[0]; // 等价 char *p = ac;
            int i;
            for (i=0; i<sizeof(ac)/sizeof(ac[0]); i++){
                printf("%d\n", ac[i]);
            }
            for(p=ac; *p=-1; p++){
                printf("%d\n", *p);
            }
            return 0;
        }
    ```
  - 指针比较：
    - <, <=, ==, >, >=, != 都可以对指针进行比较
    - 比较它们在内存中的地址
    - 数组中的 单元地址肯定是线性递增的
  - 0 地址：NULL 表示 0 地址
  - 指针的类型不能相互赋值
  - 指针类型转换
    - void\* 表示不知道指针指向什么东西的指针
      - 计算时与 char\*相同(但不相通)
    - 指针也可以转换类型
      - int \*p = &i;void\*q =(void\*) p;
      - 这并没有改变 p 所指的变量类型，而是让后人用不同的眼光通过 p 看它所指的变量--我不再当你是 int，我认为你就是个 void
  - 指针的用途：
    - 需要传入较大的数据时用作参数
    - 传入数组后对数组做操作
    - 函数返回不止一个结果
      - 需要用函数来修改不止一个变量
    - 动态申请内存

- 动态内存分配：malloc 返回的是 void\*，需要强制类型转换(申请的是字节)

  ```C
    #include<stdio.h>
    #include<stdlib.h>
    // ...
    int* a = (int*)malloc(n*sizeof(int));
  ```

  - free():释放内存

- 字符数组：

  ```C
    char word[] = {'H', 'e', 'l', 'l', 'o', '!'}; //不是C语言字符串，因为不能用字符串的方式做计算
    char word[] = {'H', 'e', 'l', 'l', 'o', '!', '\0'}; // 字符串
  ```

- 字符串：

  - 以 0（整数 0）结尾的一串字符
    - 0 或'\0'是一样的，但是和'0'不同
  - 0 标志字符串的结束，但它不是字符串的一部分
    - 计算字符串长度的时候不包括这个 0
  - 字符串以数组形式存在，以数组或指针的形式访问
    - 更多是以指针的形式
  - string.h 有很多处理字符串的函数

- 字符串变量

  - char \*str = "Hello"; // 只读的，不能写
  - char word[] = "Hello";
  - char line[10] = "Hello"; // 占位 6 个字符，结尾是 0
  - 如果要构造一个字符串-->数组
  - 如果要处理一个字符串-->指针

- 字符串输入和输出

  - char string[8];
  - scanf("%s", string); //scanf 读入一个单词（到空格、tab、回车为止）；scanf 不安全；
  - printf("%s", string);

- char \*\*: a 是一个指针，指向另一个指针，那个指针指向一个字符串
- char [][]

- 程序参数：

  - int main(int argc, char const\* argv[])
  - argv[0] 是命令本身

- 单字符的输入输出：

  - int putchar(int c); // 向标准写入一个字符；返回写了几个字符，EOF(-1)表示写入失败
  - int getchar(void); // 从标准输入读入一个字符

- string.h

  - strlen:返回字符串长度，不含 0；
  - strcmp：比较两个字符串：
    - 0：s1 == s2
    - 1：s1 > s2
    - -1: s1 < s2
  - strcpy(char *restrict dst, const char *restrict src)：拷贝 src-->dst;restrict 表明 src 和 dst 不重叠

    ```C
        #include <stdio.h>
        #include <string.h>

        char* mycpy(char* dst, const char *src){
            <!-- int idx = 0;
            while(src[idx]){
                dst[idx] = src[idx];
                idx++;
            }
            dst[idx] = '\0'; --> // 数组版本

            char *rest = *dst;
            while(*src){
                *dst = *src;
                src++;
                dst++;
            }
            return rest;
        }
        int main(){
            char s1[] = "abc";
            char s2[] = "abc";
            mycpy(s1, s2);
            return 0;
        }
    ```

  - strcat：
  - strchr：字符串找字符从左往右，返回指向字符的地址（如果找第二个则，继续在返回的地址中找）
  - strrchr：字符串找字符从右往左，返回指向字符的地址
  - strstr：字符串中找字符串

- 枚举：enum 枚举类型名字 {名字 0, 名字 1, 名字 2, ...};

  ```C
      enum COLOR {RED, YELLOW, GREEN};
  ```

- 结构类型

  ```C
    struct date {
        int month;
        int day;
        int year;
    };
    struct date today;

    // 另一种形式
    struct {
        int x,
        int y
    } p1, p2;

    // 另一种
    struct piont {
        int x,
        int y
    } p1, p2; // 既声明有定义

    // 初始化
    struct date today = {7， 31， 2023};
    struct data miao = {.month=7, .year=2023};

  ```

  - 结构跟数组有点像：
    - 要访问整个结构，直接用结构变量的名字
    - 对于整个结构，可以做赋值、取地址、也可以传递给函数参数
      - p1=(struct point){5, 10}; //相当于 p1.x=5, p1.y=10
      - p1 = p2; // 相当于 p1.x = p2.x; p1.y = p2.y;
  - 和数组不同的点是：结构变量的名字并不是结构变量的地址，必须用&运算符

    ```C
        struct date *pDate = &today;
    ```

  - 输入结构：

  ```C
    void getStruct(struct piont p){
        scanf("%d", &p.x);
        scanf("%d", &p.y);
        scanf("%d %d", &p.x, &p.y);

    }
  ```

  - 指向结构的指针
    ```C
        strcut date {
            int month;
            int day;
            int year;
        } myday;
        struct date *p = &myday;
        (*p).month = 12; // .优先级高于 *;
        p->month = 12;  // 用->表示指针所指的结构变量中的成员
    ```
  - 结构体和指针

  ```C
    #include <stdio.h>
    #include <string.h>

    struct point *getStruct(struct point *);
    void output(struct point);
    void print(const struct point *p);
    struct point
    {
        int x;
        int y;
    };
    int main(int argc, char const *argv[])
    {
        struct point y = {0, 0};
        getStruct(&y);
        output(y);
        output(*getStruct(&y));
        print(getStruct(&y));
        getStruct(&y)->x = 0;
        *getStruct(&y) = (struct point){1, 2};
    }

    struct point *getStruct(struct point *p)
    {
        scanf("%d", &p->x);
        scanf("%d", &p->y);
        return p;
    }

    void output(struct point p)
    {
        printf("%d, %d\n", p.x, p.y);
    }

    void print(const struct point *p)
    {
        printf("%d\n", p->x);
    }
  ```

- 结构数组

  ```C
    struct date dates[100];
    struct date dates[] = {
        {4, 5, 2005},
        {2, 4, 2005}
    };
  ```

- 类型定义：自定义数据类型(typedef)

  - typedef 用来声明一个已有数据类型的新名字比如：typedef int Length；
  - 声明之后，Length 就可以替代 int
  - 与结构体相关用法
    ```C
    typedef long int64_t;
    typedef struct ADate{
        int month;
        int day;
        int year;
    } Date;  // 简化了复杂的结构体，用 Date 代替 结构体
    int64_t i = 1000000000000;
    Date d = {9, 1, 2005}; // 简单又高效!!!
    ```

- 联合 union:共同使用一个空间
- 全局变量：

  - 定义在函数外面的变量是全局变量
  - 全局变量具有全局的生存期和作用域
    - 它们和任何函数都无关
    - 在任何函数内部都可以使用它们
  - 初始化：没做初始化的全局变量会得到 0 值；指针会得到 NULL 值
  - 只能用编译时刻已知的值来初始化全局变量
  - 它们的初始化发生在 main 函数前

- 静态本地变量：static [数据类型] name;

  - 实际上是特殊的全局变量
  - 位于相同的内存区域
  - 今天本地变量具有全局生存期，在函数内的局部作用域：
    - static 在这里的意思是局部作用域(本地可访问)

- 返回指针的函数：

  - 返回本地变量的地址是危险的
  - 返回全局变量或者静态本地变量的地址是安全的
  - 返回在函数内的 malloc 的内存是安全的，但是容易造成问题
  - 最好的做法是传入指针

  ```C
    #include<stdio.h>
    int* f(void);
    void g(void);

    int main(int argc, char const *argv[])
    {
        int *p = f();
        printf("*p=%d\n", *p);  // 12
        g();
        printf("*p=%d\n", *p);  // 24 p地址中的内容被覆盖

        return 0;
    }

    int* f(void)
    {
        int i=12;
        return &i;  // 不好，容易出错；正确做法：传入指针
    }

    void g(void)
    {
        int k = 24;
        printf("k=%d\n", k);
    }
  ```

- 预编译处理指令
  - #开头是预编译指令
  - 不是 C 语言的成分，但是 C 语言离不开他们
  - #define 用来定义一个宏： #define PI 3.14159 // 不能加 ;，因为不是 C 的语句
  - 预定义宏：
    - \_\_LINE\_\_
    - \_\_FILE\_\_
    - \_\_DATE\_\_
    - \_\_TIME\_\_
    - \_\_STDC\_\_
- 带参数的宏：所有用到参数的地方都要有括号

  ```C
    #define RADTODEG(x) ((x)*3.14)
  ```

- 多个.c 文件：

  - main()里的代码太长了适合分成几个函数
  - 一个源码文件太长了适合分成几个文件
  - 两个独立的源码文件不能编译成可执行文件

- 头文件：
  - 把函数原型放到一个头文件（以.h 结尾）中，在需要调用这个函数的原代码文件（.c 文件）中 #include 这个头文件，就能让编译器在编译的时候知道函数原型。
  - #include 有两种形式指出要插入的文件
    - ""要求编译器首先在当前目录（.c 所在目录）寻找这个文件，如果没有，到编译器指定的目录去找
    - <>让编译器只在指定的目录下找
  - 编译器自己知道自己的标准库的头文件在哪
  - 环境变量和编译器命令行参数也可以指定寻找头文件的目录
- #include 误区：

  - #include 不是用来引入库的
  - stdio.h 只有 printf 原型，printf 的代码再另外的地方，某个.lib(Windows)或.a(Unix)中。
  - 现在的 C 语言编译器默认会引入所有的标准库
  - #include <stdio.h> 只是为了让编译器知道 printf 函数的原型，保证调用时给出的参数值是正确的类型

- 声明 extern int i; // 变量的声明
  - 声明是不会产生代码的东西
    - 函数原型
    - 变量声明
    - 结构声明
    - 宏声明
    - 枚举声明
    - 类型声明
    - inline 函数
  - 定义是产生代码的东西
  - 只有声明可以被放在头文件中
    - 是规则不是法律
  - 否则会造成一个项目多个编译单元里有重名的实体
    - 某些编译器允许几个编译单元中存在着同名的函数或者用 weak 修饰符来强调这种存在
- 标准头文件结构(解决宏重复定义)

  ```C
    #ifndef _MAX_H
    #define _MAX_H
    ....

    #endif
  ```

- 格式化输入输出

  - printf

    - %[flags][width][.prec][hIL]type

    | Flag    | 含义         | width 或 prec | 含义                       | 类型修饰 | 含义        |
    | ------- | ------------ | ------------- | -------------------------- | -------- | ----------- |
    | +       | 左对齐       | number        | 最小字符数                 | hh       | 单个字节    |
    | -       | 在前面放+或- | \*            | 下一个参数是字符数         | h        | short       |
    | (space) | 正数留空     | .number       | 小数点后的位数             | I        | long        |
    | 0       | 0 填充       | .\*           | 下一个参数是小数点后的位数 | II       | long long   |
    |         |              |               |                            | L        | long double |

    | type   | 用于               | type   | 用于             |
    | ------ | ------------------ | ------ | ---------------- |
    | i 或 d | int                | g      | float            |
    | u      | unsigned int       | G      | float            |
    | o      | 八进制             | a 或 A | 十六进制的浮点数 |
    | x      | 十六进制           | C      | char             |
    | X      | 字母大写的十六进制 | s      | 字符串           |
    | f 或 F | float,6            | p      | 指针             |
    | e 或 E | 指数               | n      | 读入/写入的个数  |

  - scanf

    - %[flag]type

    | flag | 含义       | flag | 含义         |
    | ---- | ---------- | ---- | ------------ |
    | \*   | 跳过       | I    | long, double |
    | 数字 | 最大字符数 | II   | long long    |
    | hh   | char       | L    | long double  |
    | h    | short      |      |              |

    | type | 用于                           | type       | 用于           |
    | ---- | ------------------------------ | ---------- | -------------- |
    | d    | int                            | a, e, f, g | float          |
    | i    | 整数，可能为十六进制或者八进制 | c          | char           |
    | u    | unsigned int                   | s          | 字符串（单词） |
    | o    | 八进制                         | [...]      | 所允许的字符   |
    | x    | 十六进制                       | p          | 指针           |

- 文件的输入和输出

  - 用 > 和 < 做重定向：< 指定一个文件作为输入，> 指定一个文件写入输出
  - FILE* fopen(const char8 restrict path, const char* restrict mode);
  - int fclose(FILE \*stream);
  - fscanf(FILE\*, ...)
  - fprintf(FILE\*, ...)

  ```C
    FILE* fp = fopen("file", "r");
    if(fp){
        fscanf(fp, ...);
        fclose(fp);
    } else{
        ...
    }
  ```

  - fopen
    | 参数| 含义 |
    | ---- | ------------------------------ |
    | r | 打开只读 |
    | r+ | 打开读写，从文件头开始 |
    | w | 打开只写。如果不存在则新建，如果存在则清空 |
    | w+ | 打开读写。如果不存在则新建，如果存在则清空 |
    | a | 打开追加。如果不存在则新建，如果存在则从文件尾开写 |
    | ..x | 只新建，如果文件已存在，则不能打开 |

- 二进制文件

  - 其实所有文件最终都是二进制
  - 文本文件无非是最简单的方式可以读写的文件
    - more、tail
    - cat
    - vi
  - 而二进制文件是需要专门的程序来读写的问间
  - 文本文件的输入输出时格式化，可能经过转码
  - size_t fread(void *restrict ptr, size_t size, size_t nitems, FILE *restrict stream);
  - size_t fwrite(const void *restrict ptr, size_t size, size_t nitems, FILE *restrict stream);
  - 返回成功读写的字节数

- 按位运算

  - &：按位取与
    - 两个数都是 1，则为 1
    - 应用：
      - 让某一位或某些位置零：x & 0xFE
      - 取一个数中的一段：x & 0xFF
  - |：按位取或
    - 有 1 则 1
    - 应用：
      - 使一位或几位为 1：x | 0x01
      - 把两个数拼起来：0x00FF | 0xFF00
  - ~：按位取反
    - 把 1 变成 0, 0 变成 1
    - 应用：
      - 想要得到全部位为 1 的数：~0
      - 7 的二进制是 0111，x | 7 使得低 3 位为 1，而 x & ~7，就使得低 3 位为 0；
  - ^：按位的异或
    - 两位相同结果为 0，不同为 1
    - 应用：
      - 如果 x 和 y 相等，那么 x^y 的结果是 0；
      - 对于一个变量做两次异或相当于什么都没做：
        - x ^ y ^ x ==> y
  - <<：左移
    - i << j：i 中所有位想左移动 j 个位置，而右边填入 0；
    - 所有小于 int 的类型，移位以 int 方式来做结果还是 int：
      - x << 1 等价于 x \*= 2;
      - x << n 等价于 x \*= $2^n$;
  - \>>：右移
    - i 中所有的位向右移 j 位
    - 所有小于 int 的类型，移位以 int 的方式来做，结果是 int
    - 对于 unsigned 类型，左边填入 0；
    - 对于 signed 的类型，左边填入原来最高位(保持符号不变)
      - x >> 1 等价于 x /=2;
      - x >> n 等价于 x /= $2^n$

- 位段：把一个 int 的若干位组合成一个结构

  ```C
    struct {
        unsigned int leading : 3;
        unsigned int FLAG1 : 1;
        unsigned int FLAG2 : 1;
        int trailing : 11;
    };
  ```

  - 可以直接用位段的成员名称来访问
    - 比位、与、或方便
    - 编译器会安排其中的位的排列，不具有可移植性
    - 当所需的位超过一个 int 时会采用多个 int

- 可变数组

  - Array array_create(int init_size); // 创建
  - void array_free(Array \*a); // 回收
  - int array_size(const Array \*a); // 数组大小
  - int* array_at(Array *a, int index); // 访问某个单元
  - void array_inflate(Array \*a, int more_size); // 增长

  ```C
    typedef struct{
        int *array;
        int size;
    } Array;

    const BLOCK_SIZE = 20;

    Array array_create(init_size)
    {
        Array a;
        a.size = init_size;
        a.array = (int*)malloc(sizeof(int)*a.size);
        return a;
    }

    void array_size(Array *a)
    {
        free(a->array);
        a->array = NULL;
        a->size = 0;
    }

    // 封装（保护 a->size）
    int array_size(const Array *a)
    {
        return a->size;
    }

    int* array_at(Array *a, int index)
    {   if (index >= a->size){
        array_inflate(a, (index/BLOCK_SIZE+1)*BLOCK_SIZE - a->size);
        }
        return &(a->array[index]);
    }

    void array_inflate(Array *a, int more_size)
    {
        int *p = (int*)malloc(sizeof(int)(a->size+more_size));
        int i;
        for (i=0; i<a->size; i++)
        {
            p[i] = a->array[i];
        }
        free(a->array);
        a->array = p;
        a->size += more_size;
    }

    int main (int argc, char const *agrv[])
    {
        Array a = array_create(100);
        printf("%d\n", array_size(&a));
        *array_at(&a, 0) = 10;  // array_at 返回的是地址，因此直接*取地址中的内容
        printf("%d\n", *array_at(&a, 0));
        int number;
        int cnt;
        while (number != -1){
            scanf("%d", &number);
            if (number != -1)
                *array_at(&a, cnt++) = number;
        }
        array_free(&a);

        return 0;
    }
  ```

- 链表：数据+指针

  ```C
    typedef struct _node {
            int value;
            struct _node *next;
        } Node;

    int main(int argc, char const *argv[])
    {
        Node *head = NULL;
        int number;
        do {
            scanf("%d", &number);
            if (number != -1){
                // add to linked-list
                Node *p = (Node*)malloc(sizeof(Node));
                p->value = number;
                p->next = NULL;
                // find the last
                Node *last = head;
                if (last){
                    while(last->next){
                        last = last->next;
                    }
                    // attach
                    last->next = p;
                } else{ head = p ;}  // 初始 head = NULL，当有了第一个 p，应该指向 p
            }
        }

    }
  ```

  - 链表函数--(以下摘抄自[翁恺 程序设计进阶 C 语言笔记-链表(Linked List)](https://blog.csdn.net/qq_40570751/article/details/113513975?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-113513975-blog-111396626.235%5Ev38%5Epc_relevant_anti_vip&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-113513975-blog-111396626.235%5Ev38%5Epc_relevant_anti_vip&utm_relevant_index=3))：

  ```C
    //方案1，不行
    void add(Node* head,int number){
        ...
        else{
            head = p;       //形参在函数结束的时候就会释放，如果没有直接将内存里的数改掉，那么对指针的操作也带不到函数外面去
            //在这里head是形参指针，出了函数以后head就被释放了。
        }
    }

    //证明
    void test(int* head);

    int a = 1;
    int b = 2;
    int* p = &a;
    int* t = &b;
    int main()
    {
        test(p);
        //printf("%d\n",*p);
        printf("函数外：%d\n",*p);
        return 0;
    }

    void test(int* p){
        p = t;
        printf("函数内：%d\n",*p);
    }
    /*输出为
    函数内：2
    函数外：1*/

    //方案2，将指针的值返回
    Node* add(Node* head,int number){
        ...
        return head;
    }
    //调用时
    head = add(head,number);

    //方案3，穿入指针的指针
    //调用
    add(&head,number);   //对head取地址，就是head这个指针的指针传进去了

    add(Node**  phead,int number){     //两个*，表示指针的指针
        ...    //head等价于*phead，将head全改为*phead
        //return head;   不需要返回了
    }



    // 方案四
    #include <stdio.h>
    #include <stdbool.h>
    #include <node.h>
    #include <stdlib.h>

    typedef struct _list{
        Node* head;
        Node* tail;   //可以拓展功能，增加tail总是指向链表的末尾，然后不用遍历了，直接将tail指向新指针即可
    } List;

    int main()
    {
    //实现读入一个数字就存起来，知道读到-1为止
        int number;
        List list;
        list.head = NULL;    //定义链表的头部，这是一个指向Node类型结构体的指针
        do {
            //添加一个新的链表（数据块）,加在程序的最后面
            scanf("%d",&number);
            if (number != -1){
                add(&list,number);     //传入指向head的指针
                Node *p = (Node*)malloc(sizeof(Node));    //指针p指向新创建的链表，用malloc给他开辟内存
                p->next = NULL;   //新加入的链表放到最后面，所以是空指针
                p->value = number;
                //找到最后的那个链表
                Node *last = head;  //设用last指向最后那个链表，先设last为head
                //last和head一样，是指向头链表的指针 *head就是头链表那个结构体 last->next 等价于 (*head).next
                //判断last不为空，如果整个链表为空,此时head=NULL，last->就是NULL的next元素，这样是非法的，所以要判断
                if (last){
                    /*遍历，不能用if，这里是只要last指向的结构体的指针不为空，
                    即指向的链表的下一个链表不为空，则不停止循环，if只判断一次*/
                    while (last->next){
                                    last = last->next;   //指向下一个链表
                                }
                    last->next = p;    /*last->next等于NULL了，说明last已经指向原本的最后一个链表了，这里让last的next为我们新添加的链表
                    就是将老链表的最后一个中的next指针指向我们添加的新链表*/
                } else {
                    head = p;   //空链表，直接将head指向我们新创建的链表，此时新链表是头链表，也是唯一的链表。
                }

            }

        }while( number != -1);

        return 0;
    }

    void add(List* pList, int number)
    {
        // add to linked-list
        Node *p = (Node*)malloc(sizeof(Node));
        p->value = number;
        p->next = NULL;
        //find the last
        Node *last = pList->head;
        if (last) {
            while(last->next){
                last = last->next;
                }
            // attach
            last->next = p;
        } else {
            pList->head = p;  // 跟两数交换的原理一样，对指针指向的内容进行赋值(只不过赋的值是地址)
        }
    }
  ```

  - 链表的搜索：

    ```C
      void print(List *pList){
          Node *p;
          for(p=pList->head; p; p = p->next){
              printf("%d\t", p->value);
          }
          printf("\n");
      }

    ```

  - 链表的删除：

  ```C
    Node *q;
    for (q=NULL, p=head; p; q=p,p=p->next){
        if(p->value == number){
            if(q){                     // p-> 中的 p 必须保证是非 NULL
                q->next = p->next;
            } else{
                list.head -> p->next;
            }
            free(p);
            break;
        }
    }
  ```

  - 链表的清除：

  ```C
    for (p=head; p; p=q){
        q = p->next;
        free(p);
    }
  ```

  参考资料：

- [C 语言程序设计--翁恺](https://www.bilibili.com/video/BV1XZ4y1S7e1/?p=1&vd_source=3a9f66a8e4f96fbd39b999e86b2e0cf4)
- [翁恺 程序设计进阶 C 语言笔记-链表(Linked List)](https://blog.csdn.net/qq_40570751/article/details/113513975?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-113513975-blog-111396626.235%5Ev38%5Epc_relevant_anti_vip&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-113513975-blog-111396626.235%5Ev38%5Epc_relevant_anti_vip&utm_relevant_index=3) 强推!!总结的很好
