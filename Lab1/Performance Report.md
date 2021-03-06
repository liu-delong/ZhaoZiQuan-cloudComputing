# Performance Report For Lab 1 

## 一、实验内容

### 1.问题描述

实验1针对数独求解问题比较多线程与单线程的性能差异、同一功能不同代码实现的性能差异以及多线程在不同硬件环境下的性能差异。求解数独的算法分为BASIC,DANCE,MINA和MINAC。其中，DANCE算法速度最快，BASIC算法速度最慢。

### 2.工作概述

#### 2.1 Basic version 基础任务

全部实现

#### 2.2 Advanced version 高级任务

- [x] 实现了任意文件输入并且文件的大小可以是任意的，并且输入的时机任意
- [x] 程序可以边处理程序边进行外部输入
- [x] 专门开相应的线程来负责文件输入
- [x] 将文件的不同部分逐个读入内容并逐一解决

#### 2.3Algorithm 算法部分

使用了最优算法Dancing-Link，并且与基础版本BASIC比较性能。

#### 2.4Input/Output 输入输出

##### a.输入

1. 在程序运行时输入测试文件名
2. 可以输入多个文件名，输入时机随意
3. 可以输入任意大小的文件
4. 使用 ctrl-d 结束文件输入，当结果计算完成之后会自动退出程序

##### b.输出

默认情况下标准输出到屏幕（命令行）。

## 二、测试部分

### 1.测试方法

可以使用脚本 `Lab1/test.sh ` 来进行一键式测试，但最终不能完全满足我们的请求，故舍弃，所以为了方便测试，可以通过命令行运行程序即可。即在Lab 1的工程下面先运行`make`,然后运行程序`./sudoku_solve`,最后输入测试数据`testXXX`即可，为了最后统计时间方便将求解结果输出部分进行了注释，所以最后只输出时间，如需要验证结果可调整`main.cpp`中的输出语句。

### 2.实验环境

| 用户 |      OS      |                            CPU                            | RAM（Used/Total） |
| :--: | :----------: | :-------------------------------------------------------: | :---------------: |
| LYH  | Ubuntu 20.04 |   11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz（8核）   |   981Mb/3908Mb    |
| LHZ  | Ubuntu 20.04 | 10th Gen Intel(R) Core(TM) i5-10210U @ 1.60GHz（2核） |   1633Mb/3906Mb   |
| ZZQ  | Ubuntu 20.04 |   11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz（4核）   |   804Mb/3908Mb    |
| ZJJ  | Ubuntu 20.04 |   11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz（2核）   |   852Mb/3908Mb    |

### 3.正确性测试

对于数独问题的求解结果，将我们的程序得到的问题的解和老师给的基本版本BASIC得到的问题的解用 `Lab1/thread_pool/test.cpp`结果比较，二者完全一致，结果正确。

### 4.性能分析

#### 4.1 线程数

![image](https://github.com/RemHero/ReUp/blob/main/Lab1/Images/%E7%BA%BF%E7%A8%8B.png)

***图一 对于输入10000且不分块的数据在不同的环境下测试时间***

##### 分析

从上面不同环境的测试结果中可以看出：

1. 在多核的情况下**多线程确实要比单线程要快**，并且效率至少提高了两倍以上，但是由于CPU核心数量的不同，达到峰值速度的时间也不同，以LYH的运行环境为例，在线程数量为7的时候已经接近峰值570ms左右，后面随着线程的增加效果并没有明显变化，这是由于我们分配了一个线程专门进行文件读入，CPU总共有8个核所以7个线程的时候避免了多个线程之间的调度从而避免了资源的浪费，而随着线程数的不断增加，效果不明显是因为线程的切换占用了大量的时间。
2. 另外还有一个**有趣的发现**，相比于8核的CPU，4核的CPU更快的到达了峰值，并且二者峰值速度相当，理论上8和CPU的峰值应该比4核CPU的峰值快，可能是由于不同的机器环境导致。

#### 4.2 读入块

![image](https://github.com/RemHero/ReUp/blob/main/Lab1/Images/%E5%9D%97%E5%A4%A7%E5%B0%8F.png)

  ***图二 对于输入10000、线程数为6的数据在不同的环境下测试时间***

##### 分析

从上面不同环境的测试结果中可以看出：

1. 随着块大小的不断增加，用时越来越少，这是由于分块读取待处理数据的时候有更好的并行性能；
2. 从上面的结果可以看出（LHZ）机器用时普遍比较长，是由于他是在虚拟机上运行且相应的运行核数为2，并且CPU的频率没有其他人的高导致；
3. 从每个人的数据来看，分块达到相应的10000左右达到峰值，并且比将数据一个一个分块（不分块）效率多出4倍，大大提高了性能；
4. 最终的峰值达到了将近500ms。

#### 4.3 加速比

![image](https://github.com/RemHero/ReUp/blob/main/Lab1/Images/%E5%8A%A0%E9%80%9F%E6%AF%94.png)

***图四：在不同机器环境下，运行多线程的解数独程序与实验提供的默认BASIC版本相比，在不同问题规模下的加速比***

##### 分析

从上面的结果可以看出：

**在问题规模小的时候问题数越多加速比越大，在问题规模增大到一定程度的时候发现加速比会下降。**

从实验数据可以看到(ZZQ,ZJJ)结果比较相近，(LYH)结果最优，由于环境采用了四人中最优的版本，但是(LHZ)代码与老师的几乎一致，但是(LHZ)环境下在测试的时候使用了多线程以及分布读入等全部优化，所以应该是配置的问题，优化的效果并不明显，需要说明的是在最后的1M数据的测试上由于等待时间过长所以几乎与老师的相当。而且(LHZ)在运行的时候开了6个线程，但系统只分配了2个核，所以线程之间切换导致的性能开销并不可忽略。

