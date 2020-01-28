# 仿真验证与Testbench编写

## 概述

仿真，也叫模拟，是通过使用EDA仿真工具，通过输入信号，比对输出信号（波形、文本或VCD（Value Change Dump）文件）和期望值，来确定是否得到与期望所一致的正确的设计结构，验证设计的正确性。

### 流程

1. 功能验证
2. 综合验证
3. 时序验证
4. 板级验证

## Testbench及其结构

在仿真时Testbench用来产生激励给待验证设计DUV（Design Under Verification），或者称为待测设计DUT（Design Under Test）

### T触发器测试程序示例

```verilog
module Tflipflop_tb;
    reg clk, rst_n, T;
    wire data_out;
    TFF U1(.data_out(data_out), .T(T), .clk(clk), .rst_n(rst_n))
    always
        #5 clk = ~clk;
    initial
        begin
            clk = 0;
            #3 rst_n = 0;
            #5 rst_n = 1;
            T = 1;
            #30 T = 0;
            #20 T = 1;
        end
    
    initial
        begin
            $monitor($time, "T=%b, clk=%b, rst_n=%b, data_out=%b", T, clk, rst_n, data_out);
        end
endmodule
```

### Testbench主要功能

1. 为DUT提供激励信号
2. 正确实例化DUT
3. 将仿真数据显示在终端或存为文件，也可以显示在波形窗口以供分析检查
4. 复杂设计可以使用EDA工具，或者通过用户接口自动比较仿真结果与理想值，实现结果的自动检查。

### 注意

1. Testbench代码不需要可综合，Testbench代码只是硬件描述行为不是硬件设计。
2. 行为级描述效率高。
3. 掌握结构化、程式化的描述方式。结构化的描述有利于设计维护，可通过initial、always和assign语句将不同的测试激励划分开来。一般不要将所有的测试都放在一个语句块中。

### 仿真结果确认

1. 直接观察波形
2. 打印文本法
   -  $display，直接输出到标准输出设备
   -  $monitor，监控参数的变化
   -  $fdisplay，输出到文件
3. 自动检查仿真结果
4. 使用VCD文件

### 仿真效率

Verilog HDL行为级仿真代码的执行时间比较长，主要原因是通过串行软件代码完成并行语义的转化。

1. 减小层次结构
2. 减少门级代码的使用
3. 仿真精度越高，效率越低
4. 进程越少，效率越高
5. 减少仿真器的输出显示

## 与仿真相关的系统任务

### \$display 和 \$write

`$display("<format_specifiers>", <signal, signal2, ..., signaln>);`

`$write("<format_specifiers>", <signal, signal2, ..., signaln>);`

"<format_specifiers>"：格式控制

<signal, signal2, ..., signaln>：信号输出列表

\$display 将待定信息输出到标准输出设备后换行

\$write 输出特定信息后不换行

输出格式说明由"%"和格式字符组成，其作用是将输出的数据转换成指定的格式输出

### \$monitor 和 \$strobe

1. \$monitor 的语法格式：

`$monitor("<format_specifiers>", <signal, signal2, ... , signaln>);`

\$monitor 提供了监控和输出参数列表中的表达式或变量值的功能，输出控制和输出列表规则与 \$display 相同。

\$monitor 建立了一个处理机制，每当参数列表中变量或表达式的值发生变化时，整个参数列表中变量或表达式的值都将输出显示。

\$monitoron 和 \$monitoroff 任务的作用是通过打开和关闭监控标志来控制任务 \$monitor 的启动和停止。

通常在通过调用 \$monitoron 启动 \$monitor 时，不管 \$monitor 参数列表中的值如何变化，总是立刻输出显示当前时刻参数列表中的值。

2. \$strobe 的语法格式：

`$strobe(<functions_or_signals>);`

`$strobe("<string_and/or_variables>", <functions_or_signals>);`

\$strobe 系统任务用于某时刻所有时间处理完后，在时间步的结尾输出一行格式化的文本。

\$strobe：十进制

\$strobeb：二进制

\$strobeo：八进制

\$strobeh：十六进制

\$strobe 任务在被调用时刻所有的赋值语句都完成后，才输出相应的文字信息。

### \$time 和 \$realtime

``timescale 1ns/1ns` 时钟、采样

系统函数 \$time 返回一个 64bit 整数表示当前的仿真时刻值，以模块开始仿真的时间为基准

\$realtime 返回一个实型数，很少用

### \$finish 和 \$stop

语法格式：

`$finish;`

`$finish(n);`

`$stop;`

`$stop(n);`

| n 的取值 |                         含义                         |
| :------: | :--------------------------------------------------: |
|    0     |                    不输出任何信息                    |
|    1     |                  给出仿真时间和位置                  |
|    2     | 给出仿真时间和位置，还有所用 memory 及 CPU时间的统计 |

\$finish 的作用是退出仿真器，结束仿真。默认参数值为 1。

\$stop 的作用是把EDA工具置于暂停模式，在仿真环境下给出一个交互式的命令提示符，将控制权交给用户。

### \$readmemh 和 \$readmemb

\$readmem 和 \$readmemb 用来从文件中读取数据到存储器中，可在仿真的任何时刻被执行调用

语法格式：

`$readmemb("<file_name>", <memory_name>);`

`$readmemb("<file_name>", <memory_name>, <start_addr>);`

`$readmemb("<file_name>", <memory_name>, <start_addr>, <finish_addr>);`

\$readmenh 同理

### \$random

\$random 是产生随机数的系统函数，每次调用将返回一个32位的带符号整型随机数。

语法格式：

`$random%<number>`

给出一个范围在`(-b+1, b-1)`之间的随机数

## 信号时间赋值语句

```mermaid
graph LR
	信号的时间延迟 --- 延时控制
	信号的时间延迟 --- 事件控制
	延时控制 --- 串行延迟控制
	延时控制 --- 并行延迟控制
	延时控制 --- 阻塞式延迟控制
	延时控制 --- 非阻塞式延迟控制
	事件控制 --- 边沿触发事件控制
	事件控制 --- 电平敏感事件控制
```

### 时间延迟的语法说明

语法格式：

`#<延迟时间> 行为语句;`

`#<延迟时间>`

#### 时间控制方式

1. 外部时间控制方式：时间控制出现在整个过程赋值语句的最左端

`#5 a = b;`

仿真时相当于：

```verilog
initial
    begin
        #5; 
        a = b;
    end
```

2. 内部时间控制方式：过程赋值语句中的时间控制部分出现在赋值操作符和赋值表达式之间的时间控制方式

`a = #5 b;`

执行时相当于：

```verilog
initial
    begin
        temp = b;
        #5;
        a = temp;
    end
```

### 时间延迟的描述形式

#### 串行延迟控制

begin-end 过程块加上延迟赋值语句

若在延迟时间后有相应的行为语句，仿真进行到这条语句后，会等到延迟时间量过去后再执行行为语句。

```verilog
`timescale 1ns/1ns
module serial_delay(q0_out, q1_out);
    output q0_out, q1_out;
    reg q0_out, q1_out;
    initial
        begin
            q0_out = 1'b0;
            #50 q0_out = 1'b1;
            #100 q0_out = 1'b0;
            ...
        end
    
    initial
        begin
            q1_out = 1'b0;
            #100 q1_out = 1'b1;
            #100 q1_out = 1'b0;
        end
endmodule
```

#### 并行延迟控制

fork-join 过程块加上延迟赋值语句

并行延迟控制方式中的多条延迟控制语句是并行执行的

```verilog
`timescale 1ns/1ns
module serial_delay(q0_out, q1_out);
    output q0_out, q1_out;
    reg q0_out, q1_out;
    initial
        fork
            q0_out = 1'b0;
            #50 q0_out = 1'b1;
            #100 q0_out = 1'b0;
            ...
        join
    
    initial
        fork
            q1_out = 1'b0;
            #100 q1_out = 1'b1;
            #100 q1_out = 1'b0;
        join
endmodule
```

#### 阻塞式延迟控制

在阻塞式过程赋值基础上带有延迟控制的情况

各条语句依次执行，上一条语句赋值操作没有完成之前下一条语句不会开始执行

```verilog
initial
    begin
        a = 0;
        a = #5 1;
        a = #10 0;
        ...
    end
```

#### 非阻塞式延迟控制

在非阻塞式过程赋值基础上带有延迟控制的情况

各条非阻塞式赋值语句均以并行方式执行

```verilog
initial
    begin
        a <= 0;
        a <= #5 1;
        a <= #10 0;
        ...
    end
```

### 边沿触发事件控制

边沿事件控制方式是在指定的信号变化时刻，即指定的信号跳变沿才触发语句的执行。

语法格式：

`@(<事件表达式>) 行为语句;`

`@(<事件表达式>);`

`@(<事件表达式1> or <事件表达式2> or ... or <事件表达式n>) 行为语句;`

`@(<事件表达式1> or <事件表达式2> or .. or <事件表达式n>) 行为语句;`

####  事件表达式

1. `<信号名>`
2. `posedge <信号名>`
3. `negedge <信号名>`

信号名可以是任何数据类型的标量或矢量

| 正跳变    | 负跳变 |
| --------- | ------ |
| $0\to x$ | $1\to x$ |
| $0\to z$ | $1\to z$ |
| $0\to1$ | $1\to0$ |
| $x\to1$ | $x\to0$ |
| $z\to1$ | $z\to0$ |

```verilog
//时钟脉冲计数器
module clk_counter(clk, count_out);
    input clk;
    output count_out;
    reg [3:0] count_out;
    initial
        count_out = 0;
    
    always@(posedge clk)
        count_out = count_out + 1;
endmodule
```

```verilog
//测定输入时钟高低电平持续时间
module clk_time_mea(clk);
    input clk;
    time posedge_time, negedge_time;
    time high_last_time, low_last_time, last_time;
    initial
        begin
            @(posedge clk);
            posedge_time = $time;
            @(negedge clk);
            negedge_time = $time;
            @(posedge clk);
            last_time = $time - posedge_time;
            high_last_time = negedge_time - posedge_time;
            low_last_time = last_time - high_last_time;
        end
endmodule
```

### 电平敏感事件控制

语法格式：

`wait(条件表达式) 行为语句;`

`wait(条件表达式);`

条件表达式为真时，执行语句

一般情况下会构成 latch， 但现在又不用这玩意……

```verilog
wait(enable == 1)
begin
    d = a & b;
    d = d | c;
end
```

第 2 种表达式没有包含行为语句，仿真进程执行到 wait 控制语句时，若条件表达式为假，进入等待状态，一直到条件表达式取值为真。

## 任务与函数

### 任务

如果子程序满足以下任一条件，则公共子程序的描述中必须使用任务而不能使用函数：

1. 子程序中包含有延迟、时序或者事件控制结构
2. 没有输出或者输入变量的数量大于1
3. 没有输入变量

#### 定义

```verilog
task <任务名>;
    端口和类型声明
    局部变量声明
    begin
        语句1;
        语句2;
    end
endtask
```

#### 注意

1. 任务名后不能出现输入输出端口列表。
2. 任务中可以有延时语句、敏感事件控制语句等事件控制语句。
3. 任务可以没有或可以有一个或多个输入、输出和双向端口。
4. 任务可以没有返回值，也可以通过输出端口或双向端口返回一个或多个值。
5. 任务可以调用其它的任务和函数，也可以调用该任务本身。
6. 任务定义结构内不允许出现过程块（ `initial` 或 `always` ）。
7. 任务定义结构内可以出现`disable`终止语句，这条语句的执行将中断正在执行的任务。任务被中断后，程序流程将返回到调用任务的地方继续向下执行。

#### 任务的调用

`<任务名>(端口1, 端口2,...)`

顺序与定义相同

使用任务可以使程序更加简洁易懂

#### 例

```verilog
module demo_task_invo_tb;
    reg [7:0] mem[127:0];
    reg [15:0] a;
    reg [31:0] b;
    initial
        begin
            a = 0;
            read_mem(a, b); //第一次调用
            #10;
            a = 64;
            read_mem(a, b); //第二次调用
        end
    task read_mem(); //任务定义部分
        input [15:0] address;
        output [31:0] data;
        reg [3:0] counter;
        reg [7:0] temp[1:4];
        begin
            for(counter = 1; counter <= 4; counter = counter + 1)
                temp[counter] = mem[address + counter - 1];
            data = {temp[1], temp[2], temp[3], temp[4]};
        end
    endtask
endmodule
```

### 函数

对于一个子程序，如果下面的所有条件成立，则可以使用函数来完成：

1. 子程序内不含有延迟、时序或者事件控制结构
2. 子程序只有一个返回值
3. 至少有一个输入变量
4. 没有输出或者双向变量
5. 不含有非阻塞赋值语句

#### 定义

```verilog
function <返回值类型或位宽> <函数名>;
    输入参量与类型声明
    局部变量声明
    begin
        语句1;
        语句2;
        ...
    end
endfunction
```

#### 注意

1. 与任务一样，函数定义结构只能出现在模块中，而不能出现在过程块内。
2. 函数至少必须有一个输入端口。
3. 函数不能有任何类型的输出端口和双向端口。
4. 在函数定义结构中的行为语句不能出现任何类型的时间控制描述，也不允许使用`disable`终止语句
5. 与任务定义一样，函数定义内部结构不能出现过程块。
6. 在一个函数内可以其他函数进行调用，但是函数不能调用其它任务。
7. 在第一行`function`语句中不能出现端口名列表。
8. 函数声明的时候，在 Verilog 内部隐含地声明了一个名为 function_identifier 的寄存器类型变量，函数的输出结果将通过这个寄存器变量被传递回来。

#### 函数的调用

`函数名(<输入表达式1>, <输入表达式2>, ... )`

#### 例

```verilog
module tryfact_tb;
    function [31:0] factorial;
        input [3:0] operand;
        reg [3:0] index;
        begin
            factorial = 1;
            for(index = 1; index <= oprand; index = index + 1)
                factorial = index * factorial;
        end
    endfunction
    
    reg [31:0] result;
    reg [3:0] n;
    initial
        begin
            result = 1;
            for(n = 1; n <= 9; n = n + 1)
                begin
                    result = factorial(n);
                    $display(...)
                end
        end
endmodule
```

### 任务与函数的区别

|                             函数                             |                           任务                           |
| :----------------------------------------------------------: | :------------------------------------------------------: |
|          函数能调用另一个函数，但不能调用另一个任务          |         任务能调用另一个任务，也能调用另一个函数         |
|                函数总是在仿真时刻0就开始执行                 |               任务可以在非 0 仿真时刻执行                |
|        函数一定不能包含任何延迟、时序或者事件控制语句        |        任务可以包含延迟、时序或者事件控制声明语句        |
|                  任务函数至少有一个输入变量                  |        任务可以没有或者有多个输入、输出和双向变量        |
|         函数只能返回一个值，函数不能有输出或双向变量         | 任务不返回任何值，任务可以通过输出或者双向变量传递多个值 |
| 函数不能单独作为一条语句出现，它只能以语句的一部分的形式出现 |       任务的调用是通过一条单独的任务调用语句实现的       |
|           函数调用可以出现在过程块或连续赋值语句中           |                任务调用只能出现在过程块中                |
|           函数的执行不允许由 disable 语句进行中断            |          任务的执行可以由 disable 语句进行中断           |

## 典型测试向量的设计



