# 仿真验证与 Testbench 编写

## 概述

仿真，也叫模拟，是通过使用 EDA 仿真工具，通过输入信号，比对输出信号（波形、文本或 VCD 文件）和期望值，来确定是否得到与期望所一致的正确的设计结构，验证设计的正确性

验证流程：

1. 功能验证
2. 综合验证
3. 时序验证
4. 板级验证



## Testbench 及其结构

在仿真时 Testbench 用来产生激励给待验证设计 DUV (Design Under Verification) ，或者称为待测设计 DUT (Design Under Test)

### T触发器测试示例

```verilog
module Tflipflop_tb;
    reg clk, rst_n, T;
    wire data_out;
    //实例化模块
    TFF U1(.data_out(data_out), .T(T), .clk(clk), .rst_n(rst_n));
    
    always			//产生测试激励
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
    
    initial			//对输出响应进行收集
        begin
            $monitor
            ($time, "T=%b, clk=%b, rst_n=%b, data_out=%b", T, clk, rst_n, data_out);
        end
endmodule
```



### 1. Testbench 主要功能

(1) 为DUT提供激励信号

(2) 正确实例化DUT

(3) 将仿真数据显示在终端或存为文件，也可以显示在波形窗口以供分析检查

(4) 复杂设计可以使用EDA工具，或者通过用户接口自动比较仿真结果与理想值，实现结果的自动检查。



### 2. 注意事项

(1) Testbench代码不需要可综合，Testbench代码只是硬件描述行为不是硬件设计

(2) 行为级描述效率高

(3) 掌握结构化、程式化的描述方式。结构化的描述有利于设计维护，可通过 `initial`、`always` 和 `assign` 语句将不同的测试激励划分开来。一般不要将所有的测试都放在一个语句块中



### 3. 仿真环境搭建

#### (1) 组合电路

```verilog
//eg.全加器测试
module adder1(a, b, ci, so, co);
    input a, b, ci;
    output so, co;
    assign {co, so} = a + b + ci;
endmodule

module adder1_tb;
    wire so, co;
    reg a, b, ci;
    adder1 U1(a, b, ci, so, co);
    
    initial
        begin
            	a = 0; b = 0; ci = 0;
            #20 a = 0; b = 0; ci = 1;
            #20 a = 0; b = 1; ci = 0;
            #20 a = 0; b = 1; ci = 1;
            #20 a = 1; b = 0; ci = 0;
            #20 a = 1; b = 0; ci = 1;
            #20 a = 1; b = 1; ci = 0;
            #20 a = 1; b = 1; ci = 1;
            #200 $finish;
        end
endmodule
```



#### (2) 时序电路

```verilog
//eg.十进制加法计数器测试
module cnt10_tb;
    reg clk, rst, ena;
    wire [3:0] q;
    wire cout;
    cnt10 U1(clk, rst, ena, q, cout);
    
    always
        #50 clk = ~clk;
    initial begin
        clk = 0; rst = 0; ena = 1;
        #1200	rst = 1;
        #120  	rst = 0;
        #2000 	ena = 0;
        #200  	ena = 1;
        #20000 	$finish;
    end
endmodule
```



### 4. 仿真结果确认

#### (1) 直接观察波形

+ 通过直接观察信号波形的输出，比较测试值和期望值的大小，来确定仿真结果的正确性

+ 该确认方式是最简单清晰的

#### (2) 打印文本输出

+ 系统任务打印任务：
  + `$display`，直接输出到标准输出设备
  + `$monitor`，监控参数的变化

+ 一般性打印一些关键错误信号

#### (3) 自动检测

+ 通过设计代码中的关键节点，添加断言监控器，形成对电路逻辑综合的注释或对设计特点的说明，以提高设计模块的观察性
+ 这种方法的效率是最高的，但不够直观

#### (4) 使用 VCD 文件

+ VCD 文件是一种标准格式的波形记录文件，只记录发生变化的波形
+ 适合观测超大规模的设计电路



### 5. 仿真效率

+ 减小层次结构

+ 减少门级代码的使用

+ 仿真精度越高，效率越低

+ 进程越少，效率越高

+ 减少仿真器的输出显示