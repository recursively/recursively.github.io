---
title: >-
  Interesting Thoughts on Data Flow Analysis and Taint Tracking Mechanism of
  SAST Tool(Coverity)
date: 2022-01-13 08:16:19
categories: SAST
tags: [Data flow, Taint tracking, Coverity, Code quality]
keywords: [Data flow, Taint tracking, Coverity, Code quality]
description: Some SAST tools introduce data flow and taint tracking technology to improve the accuracy of scanning results. Coverity is one of the greatest SAST tools that are able to significantly reduce code defects. This post will illustrate some interesting points while performing code scanning with Coverity.
---
## Example 1 - Predetermined Value Operation In Loop Block

For the common integer overflow problems, Coverity can definitely find those defects. But it will be a little tricky in some scenarios, such as the loop condition. 

```C
#include <limits.h>
#include <stdio.h>

int func() {
  int a = 1;
  int b = 0;
  int c = 5;
  while (a < 3) {
    b = INT_MAX + c;
    a++;
    printf("%d", b);
  }
  return -1;
}

int main()
{
  int result = func();
}
```

For this code, Coverity cannot find this kind of integer overflow defect with common checkers even like the data flow analysis.

```shell
cov-analyze --dir idir/ --all --enable-audit-mode
```

The scanning result will be like below:

```shell
Analysis summary report:
------------------------
Files analyzed                 : 1 Total
    C                          : 1
Total LoC input to cov-analyze : 2459
Functions analyzed             : 2
Paths analyzed                 : 10
Time taken by analysis         : 00:00:03
Defect occurrences found       : 0
```

But the interesting thing is that Coverity is able to identify this defect when we use the CERT-C coding standard.

```shell
cov-analyze --dir idir/ --coding-standard-config "/opt/cov-analysis-linux64/config/coding-standards/cert-c/cert-c-all.config"
```

The defect was identified for sure.

```shell
Analysis summary report:
------------------------
Files analyzed                       : 1 Total
    C                                : 1
Total LoC input to cov-analyze       : 2459
Functions analyzed                   : 2
Paths analyzed                       : 10
Time taken by analysis               : 00:00:03
Defects/Coding rule violations found : 1 CERT INT32-C
```

<img src="pic_1.jpeg" width="100%" height="100%">

I don't know why coverity doesn't idendify this kind of defect in the common checkers. Maybe it will cause some extra performance burden like memory usage. My understanding is that Coverity will introduce another placeholder to keep the value which may cause integer overflow, in our sample here is the macro *INT_MAX*. The situation will be getting more complicated within the while loop. Some changes of variables in the loop block rely on the runtime status. The scenario in this sample code is not that complicated cuz the value of variable *c* will not change as the number of cycles increases. For Coverity's analyzer, it's just enough for only calculating the sum result of macro *INT_MAX* and variable *c*, if we change the value of *c* to 0 like below:

```C
#include <limits.h>
#include <stdio.h>

int func() {
  int a = 1;
  int b = 0;
  int c = 0;
  while (a < 3) {
    b = INT_MAX + c;
    a++;
    printf("%d", b);
  }
  return -1;
}

int main()
{
  int result = func();
}
```

```shell
cov-analyze --dir idir/ --coding-standard-config "/opt/cov-analysis-linux64/config/coding-standards/cert-c/cert-c-all.config"
```

This integer overflow defect is eliminated and of course the finding will not be triggered. It makes sense that Coverity will introduce extra memory to keep the result, which is not enabled for common checkers.

```shell
Analysis summary report:
------------------------
Files analyzed                       : 1 Total
    C                                : 1
Total LoC input to cov-analyze       : 2459
Functions analyzed                   : 2
Paths analyzed                       : 10
Time taken by analysis               : 00:00:03
Defects/Coding rule violations found : 0
```

In fact, the value of operand *INT_MAX* and *c* can be grabbed in the semantic analysis stage. The sum value which cause integer overflow can be easily calculated through the AST.

```shell
clang -cc1 -ast-dump overflow-test.cpp
```

```shell
|-FunctionDecl 0x7fc719855918 <test.cpp:4:1, line:16:1> line:4:5 used func 'int ()'
| `-CompoundStmt 0x7fc719856048 <col:12, line:16:1>
|   |-DeclStmt 0x7fc719855a58 <line:5:3, col:12>
|   | `-VarDecl 0x7fc7198559d0 <col:3, col:11> col:7 used a 'int' cinit
|   |   `-IntegerLiteral 0x7fc719855a38 <col:11> 'int' 1
|   |-DeclStmt 0x7fc719855b10 <line:6:3, col:12>
|   | `-VarDecl 0x7fc719855a88 <col:3, col:11> col:7 used b 'int' cinit
|   |   `-IntegerLiteral 0x7fc719855af0 <col:11> 'int' 0
|   |-DeclStmt 0x7fc719855bc8 <line:7:3, col:12>
|   | `-VarDecl 0x7fc719855b40 <col:3, col:11> col:7 used c 'int' cinit
|   |   `-IntegerLiteral 0x7fc719855ba8 <col:11> 'int' 5
|   |-DeclStmt 0x7fc719855c80 <line:8:3, col:18>
|   | `-VarDecl 0x7fc719855bf8 <col:3, /Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/usr/include/i386/limits.h:71:25> test.cpp:8:7 used d 'int' cinit
|   |   `-IntegerLiteral 0x7fc719855c60 </Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/usr/include/i386/limits.h:71:25> 'int' 2147483647
|   |-WhileStmt 0x7fc719855e20 <test.cpp:9:3, line:13:3>
|   | |-BinaryOperator 0x7fc719855cf0 <line:9:10, col:14> 'bool' '<'
|   | | |-ImplicitCastExpr 0x7fc719855cd8 <col:10> 'int' <LValueToRValue>
|   | | | `-DeclRefExpr 0x7fc719855c98 <col:10> 'int' lvalue Var 0x7fc7198559d0 'a' 'int'
|   | | `-IntegerLiteral 0x7fc719855cb8 <col:14> 'int' 3
|   | `-CompoundStmt 0x7fc719855e00 <col:17, line:13:3>
|   |   |-BinaryOperator 0x7fc719855da8 <line:10:5, col:19> 'int' lvalue '='
|   |   | |-DeclRefExpr 0x7fc719855d10 <col:5> 'int' lvalue Var 0x7fc719855a88 'b' 'int'
|   |   | `-BinaryOperator 0x7fc719855d88 </Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/usr/include/i386/limits.h:71:25, test.cpp:10:19> 'int' '+'
|   |   |   |-IntegerLiteral 0x7fc719855d30 </Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/usr/include/i386/limits.h:71:25> 'int' 2147483647
|   |   |   `-ImplicitCastExpr 0x7fc719855d70 <test.cpp:10:19> 'int' <LValueToRValue>
|   |   |     `-DeclRefExpr 0x7fc719855d50 <col:19> 'int' lvalue Var 0x7fc719855b40 'c' 'int'
|   |   `-UnaryOperator 0x7fc719855de8 <line:11:5, col:6> 'int' postfix '++'
|   |     `-DeclRefExpr 0x7fc719855dc8 <col:5> 'int' lvalue Var 0x7fc7198559d0 'a' 'int'
|   |-CallExpr 0x7fc719855fa0 <line:14:3, col:17> 'int'
|   | |-ImplicitCastExpr 0x7fc719855f88 <col:3> 'int (*)(const char *, ...)' <FunctionToPointerDecay>
|   | | `-DeclRefExpr 0x7fc719855f38 <col:3> 'int (const char *, ...)' lvalue Function 0x7fc7198919f8 'printf' 'int (const char *, ...)'
|   | |-ImplicitCastExpr 0x7fc719855fd0 <col:10> 'const char *' <ArrayToPointerDecay>
|   | | `-StringLiteral 0x7fc719855ef8 <col:10> 'const char [3]' lvalue "%d"
|   | `-ImplicitCastExpr 0x7fc719855fe8 <col:16> 'int' <LValueToRValue>
|   |   `-DeclRefExpr 0x7fc719855f18 <col:16> 'int' lvalue Var 0x7fc719855bf8 'd' 'int'
|   `-ReturnStmt 0x7fc719856038 <line:15:3, col:11>
|     `-UnaryOperator 0x7fc719856020 <col:10, col:11> 'int' prefix '-'
|       `-IntegerLiteral 0x7fc719856000 <col:11> 'int' 1
`-FunctionDecl 0x7fc7198560b8 <line:18:1, line:21:1> line:18:5 main 'int ()'
  `-CompoundStmt 0x7fc7198562b8 <line:19:1, line:21:1>
    `-DeclStmt 0x7fc7198562a0 <line:20:3, col:22>
      `-VarDecl 0x7fc719856170 <col:3, col:21> col:7 result 'int' cinit
        `-CallExpr 0x7fc719856280 <col:16, col:21> 'int'
          `-ImplicitCastExpr 0x7fc719856268 <col:16> 'int (*)()' <FunctionToPointerDecay>
            `-DeclRefExpr 0x7fc719856220 <col:16> 'int ()' lvalue Function 0x7fc719855918 'func' 'int ()'
```

## Example 2 - Iteration Value Operation In Loop Block

This sample will be a little different, the variable within the loop block will keep changing as the number of cycles increases. The program will be keeping running infinitely cuz the value of variable *m* will overflow and the while loop will never come to an end.

```C
#include <limits.h>
#include <stdio.h>

int binary_search(int n, long v) {
  int l = 0, u = n-1;
  while (l <= u ) {
    int m = (l + u) / 2;
    if (m < v) l = m + 1;
    else if (m > v) u = m - 1;
    else return m;
  }
  return -1;
}

int main()
{
  int result = binary_search(INT_MAX/2+3, INT_MAX/2 + 1);
  printf("%d\n", result);
}
```

If we run the Coverity analyzer it will not identify the defect no matter what checkers or standards enabled.

```shell
cov-analyze --dir idir/ --all --enable-audit-mode
```

```shell
cov-analyze --dir idir/ --coding-standard-config "/opt/cov-analysis-linux64/config/coding-standards/cert-c/cert-c-all.config"
```

```shell
Analysis summary report:
------------------------
Files analyzed                 : 1 Total
    C                          : 1
Total LoC input to cov-analyze : 2459
Functions analyzed             : 2
Paths analyzed                 : 22
Time taken by analysis         : 00:00:02
Defect occurrences found       : 0
```

It's obvious that Coverity will not handle this kind of problem if the values of variables will be different as the runtime status changes.

## Example 3 - Command Execution Function Call In Loop Block

Now that Coverity will not check the defects if the iteration variable is calculated as an operand. I'm curious if Coverity will check the code behavior if the iteration variable is used in other kind of code block such as judge clause. In the code below, it read data from external file and passes the data to command execution function *system()*.

```C
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

#define BUZZ_SIZE 20

void foo1(char* a);
void foo2(char* a);
void foo3(char* a);

void foo1(char* a)
{
  foo2(a);
}

void foo2(char* a)
{

  char buff[BUZZ_SIZE] = {};
  FILE *f = fopen("f.txt", "r");
  for (int i = 0; i < 100; i += 2) {
  if (i == 90) fgets(buff, BUZZ_SIZE, f);
  }
  fclose(f);
  memcpy(&buff[strlen(buff) -1], a, strlen(a) + 1);
  foo3(buff);
}

void foo3(char* a)
{
  char* r;
  r = a;
  system(r);
  return;
}

int main(int argc, char** argv)
{
  char *l = " -l";
  foo1(l);
  return 0;
}
```

```shell
cov-analyze --dir idir/ --all --enable-audit-mode
```

```shell
Analysis summary report:
------------------------
Files analyzed                 : 1 Total
    C                          : 1
Total LoC input to cov-analyze : 4387
Functions analyzed             : 4
Paths analyzed                 : 51
Time taken by analysis         : 00:00:03
Defect occurrences found       : 1 OS_CMD_INJECTION
```

We can find that Coverity has truly detected the defect through it's data flow analysis and taint tracking process.

<img src="pic_2.jpeg" width="100%" height="100%">

But I'm still curious what kind of method coverity use to find the problem. Let's try to modify the value from 90 to 91 in the judge clause.

```C
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

#define BUZZ_SIZE 20

void foo1(char* a);
void foo2(char* a);
void foo3(char* a);

void foo1(char* a)
{
  foo2(a);
}

void foo2(char* a)
{

  char buff[BUZZ_SIZE] = {};
  FILE *f = fopen("f.txt", "r");
  for (int i = 0; i < 100; i += 2) {
  if (i == 91) fgets(buff, BUZZ_SIZE, f);
  }
  fclose(f);
  memcpy(&buff[strlen(buff) -1], a, strlen(a) + 1);
  foo3(buff);
}

void foo3(char* a)
{
  char* r;
  r = a;
  system(r);
  return;
}

int main(int argc, char** argv)
{
  char *l = " -l";
  foo1(l);
  return 0;
}
```

Re-analyze the code and the problem doesn't exist any more. 

```shell
Analysis summary report:
------------------------
Files analyzed                 : 1 Total
    C                          : 1
Total LoC input to cov-analyze : 4387
Functions analyzed             : 4
Paths analyzed                 : 44
Time taken by analysis         : 00:00:03
Defect occurrences found       : 0
```

Interesting, at least we can assure Coverity actually maintained a placeholder to keep the value of variables in the loop block while performing data flow analysis or taint tracking.