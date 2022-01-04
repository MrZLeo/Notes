# CPU设计

> CPU结构；CPU指令设计；指令执行过程；CPU部件设计

## CPU的基本组成

1. 部件：寄存器、ALU、控制器、其他
2. 数据通路：总线BUS、专用通路
3. 外部交互：外部总线

![image-20210615205010074](/Users/mrzleo/Library/Application Support/typora-user-images/image-20210615205010074.png)

- 数据总线和CPU内部总线是**双向**的。
- 总线的条数称为数据总线宽度。比如，**16**位总线，指其数据总线为**16**根。
- 数据总线是**三态的**，是指**0**，**1**和**高阻抗**三个状态。由于总线是公共通道，在某一时刻，只允许接收某一设备的信号，其他一切设备都应和它断开。

- 控制总线上传送一个部件对另一个部件的控制信号。 
- 主设备与从设备：在总线上所连接的各类设备，按其对总线**有无控制功能**可分为主设备和从设备。 主设备对总线有控制权，从设备只能响应。

- 地址总线上传送地址信号，用来指定需要访问的地址/部件
- 总线上的所有部件接收到该地址信号，但只有经过译码电路选中的部件才接收控制信号。
- 地址总线是单向的
- 地址总线也是三态的  

## CPU指令执行

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617152655350.png" alt="image-20210617152655350" style="zoom:50%;" />

> 取指 —> 译码 —> 执行 —> 写回

```assembly
# add [2000H], ax
# 取add指令
pc -> AR
AR -> AB
AB -> MAR
Read(MM->MDR)
MDR -> DB
DB -> DR
DR -> IR
pc++
# 取操作数
pc -> AR
AR -> AB
AB -> MAR
Read(MM->MDR)
MDR -> DB
DB -> DR
DR -> S
pc++
# 执行
ax -> ALU
add, ALU -> T
# 存储
T -> DR
DR -> DB
IR(2000h) -> AR
AR -> AB
DB -> MDR
AB -> MAR
Write
```

> #### Insights:
>
> 1. 一条机器指令大约由10～30个更基础的器件级操作构成——**微操作**
> 2. 微操作需要**按顺序执行**（需要时序控制）
> 3. CPU内的控制单元CU控制微操作

## 时序和控制

- 时序系统的作用：将各种控制信号严格定时，在时间上相互配合完成某一功能
- 分级：
  1. 指令周期：执行一条指令所用的时间
  2. 机器周期（CPU周期）：完成一个子周期(一组微操作)所用的时间
  3. 时钟周期（节拍周期）：完成一个微操作所用的时间
  4. [时钟脉冲、节拍脉冲]：微操作过程中的最小动作单位

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617154123424.png" alt="image-20210617154123424" style="zoom:50%;" />

- 时序控制的方式：
  - 同步：每个控制信号都由事先确定的统一的时序信号进行统一控制
  - 异步：当控制器发出某一操作控制信号后，等待执行部件完成操作后发回“回答(ACK)”信号，再开始新的操作。
  - 联合：同步控制和异步控制相结合的方式

> **CU**的主要作用: 发出满足一定时序关系的信号，实现指令系统所规定的各条指令的功能，并保证计算机系统正常运行。

## 控制器的设计

### 组合逻辑控制器（硬编码）

利用组合逻辑电路，将对应的输入转化成对应的输出（枚举发生条件，将其描述为与或门逻辑）

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617160412085.png" alt="image-20210617160412085" style="zoom:50%;" />

> ( 时间信息、指令信息、状态信息 ) —> 控制信号

### 微程序控制器

> 指导思想：将程序控制的思想引入控制信号的形成和控制
>
> 基本思想：按照微程序顺序，产生所需的控制信号。相当于把控制信号存储起来，因此又称存储控制逻辑方法
>
> 微程序控制器：根据外部输入的信号，调用内部由微指令组成的微程序，输出微命令，完成一系列微操作。

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617165639877.png" alt="image-20210617165639877" style="zoom:50%;" />

- 一条(机器)指令对应一个微程序，该微程序包含从取指令到执行指令一个完整微操作序列对应的全部微指令，它被存入一个称为控制存储器(**control memory**)的**ROM**中。

- **CM**中存放着指令系统中定义的所有指令的微程序。
- 微指令周期：一条微指令执行的时间(包括从控制存储器中取得微指令和执行微指令所用时间)
- <img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617170708743.png" alt="image-20210617170708743" style="zoom:50%;" />

#### 微指令寻址

1. 两地址格式

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617171919571.png" alt="image-20210617171919571" style="zoom:50%;" />

2. 单地址格式

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617171951823.png" alt="image-20210617171951823" style="zoom:50%;" />

3. 可变格式

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617172045998.png" alt="image-20210617172045998" style="zoom:50%;" />

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617172117254.png" alt="image-20210617172117254" style="zoom:50%;" />

#### 控制信号编码

- 水平型微指令(**horizontal microinstruction**) ：多个控制信号同时有效 → 多个微操作同时发生。
- 垂直型微指令(**vertical microinstruction**) ：类似于机器指令，利用微操作码的不同编码来表示不同的 微操作功能。

#### 水平型

##### 直接表示法

- 可以在同一个时间有效的控制信号称为相容信号，具有相容性
- 不能在同一个时间有效的控制信号称为互斥信号，具有互斥性

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617172354247.png" alt="image-20210617172354247" style="zoom:50%;" />

##### 字段译码法(字段编码) 

- 将控制域分为若干字段，字段内垂直编码，字段间水平编码。

> - 若各字段的编码相互独立，则通过各字段独立译码就可以获得计算机系统的全
>   部控制信号，这被称作**直接译码**方式。
>
> - 若某些字段的编码相互关联，则关联字段要通过两级译码才能获得相关的控制
>   信号，这被称作**间接译码**方式。

- 每个字段中要设计一个**无效控制信号**的编码

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617172915970.png" alt="image-20210617172915970" style="zoom: 33%;" />

<img src="/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617172934546.png" alt="image-20210617172934546" style="zoom: 40%;" />

![image-20210617173202291](/Users/mrzleo/Library/Application Support/typora-user-images/image-20210617173202291.png)

## CPU内部指令系统设计

- 两大步骤：
  1. 给指令编码
  2. 给指令解码

