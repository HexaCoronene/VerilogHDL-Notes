

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

## 仿真结果确认

1. 直接观察波形
2. 打印文本法
   -  $display，直接输出到标准输出设备
   -  $monitor，监控参数的变化
   -  $fdisplay，输出到文件
3. 自动检查仿真结果
4. 使用VCD文件

## 仿真效率

pass

1. pass
2. pass
3. 仿真精度越高，效率越低
4. 进程越少，效率越高
5. 减少仿真器的输出显示