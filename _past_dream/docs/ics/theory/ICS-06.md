## 《计算机系统基础》第6章 层次结构存储系统 习题选解

---
### 第3题 
计算机主存的最大寻址空间为4G，按字节编址。假定用64M×8位的具有8个位平面的DRAM芯片组成容量为512MB，传输宽度为64位的内存条（主存模块）。
（1）单个DRAM芯片的存储容量为
\[
    64M \times 8 bites = 64MB   
\]每个内存条需要DRAM芯片数量为
\[
    \frac{512MB}{64MB} = 8
\]
（2）构建容量为2GB的主存时，需要的内存条数量为
\[
    \frac{2GB}{512MB} = 4
\]
（3）根据上述内容，由于主存的最大寻址空间为4GB，注意到
\[
    4GB = 2^{32}B
\]按字节编址，主存地址共有32位，记为$A_{31}A_{30}A_{29}...A_{2}A_{1}A_{0}$。
根据《计算机系统基础》第258页内容，采用双译码结构的DRAM芯片地址译码器分为X和Y方向两个译码器。对于本题中的DRAM芯片，存储阵列有64M = $2^{26}$ 个单元，需要26根地址线，其中高13位送X译码器作为行地址选择一行单元；低13位送Y译码器作为列地址控制一列单元的位线控制门。
于是DRAM芯片内地址为$A_{28}A_{27}...A_{4}A_{3}$，其中行地址$A_{28}A_{27}...A_{17}A_{16}$，列地址$A_{15}A_{14}...A_{4}A_{3}$，用于选择芯片的位为$A_{2}A_{1}A_{0}$。
而每次内存条内的8个DRAM芯片同时读写，每个芯片有8个位平面，因此可以同时读出64位数据。

### 第4题
计算机按字节编址，已配有0000H~7FFFH的ROM区域，现再用$16K \times 4bite$的RAM芯片形成$32K \times 8bite$的存储区域，中央处理器地址线16位。
（1）RAM存储区域大小
\[
    32K \times 8bite = 32KB = 2^{15}B
\]地址起始为8000H，则RAM区地址范围为8000H~FFFFH。
需要RAM芯片数量为
\[
    \frac{32K \times 8bite}{16K \times 4bite} = 4
\]CPU地址线用来区分RAM区和ROM区的位为$A_{15}$。
（2）需要的RAM芯片数为
\[
    \frac{2^{24}B - 2^{15}B}{16K \times 4bite} = 2^{11} - 2^{2} = 2044
\]

### 第5题
程序重复完成将磁盘上的一个4KB数据块读出，在时钟频率为500MHz的处理器进行20000个时钟周期的处理后写入另一个数据区，数据块内信息在磁盘上连续存放，数据块随机位于磁盘的一个磁道上，磁盘转速7200转/分，平均寻道时间10ms，最大内部数据传输率40MB/s，磁盘控制器开销2ms。
本过程需要计两次控制器时间；计两次磁盘寻道时间；计两次旋转等待时间，其中平均等待时间取磁盘旋转一周所需时间的一半，为$\frac{1}{2} \times \frac{60s}{7200} = \frac{1}{240}s$；两次数据传输时间，每次数据传输时间为$\frac{4KB}{40MB/s} = \frac{8}{78125}s$；一次数据处理时间，为$\frac{20000}{500MHz} = 4 \times 10^{-5}s$；过程总时间为
\[
    2 \times 2ms + 2 \times 10ms + 2 \times \frac{1}{240}s + 2 \times \frac{8}{78125}s + 4 \times 10^{-5}s \approx 0.032578s
\]程序每秒可以完成数据块操作次数约为
\[
    \frac{1s}{0.032578s} \approx 30.6954
\]

### 第8题
计算机主存地址空间大小1GB，按字节编址，cache数据区64KB，块大小128B，采用直接映射和write-through方式。
根据《计算机系统基础》6.4节高速缓冲存储器的内容，主存块和cache行之间采用直接映射方式表明每个主存块映射到cache的固定行中，也称为模映射，映射关系为
\[
   \textbf{cache 行号} = \textbf{主存块号 } mod \textbf{  cache 行数}
\]直接映射方式下，主存被分成标记、cache行号、块内地址三个字段。假定cache共有$2^{c}$行，主存共有$2^{m}$块，主存块大小占$2^{b}$字节，则从高到低依次有(m-c)位标记字段，c位cache行号，b位块内地址。
Write-through方式则表明当CPU执行写操作时，若写命中，则同时写cache和主存。
（1）注意到主存地址空间大小$1GB = 2^{30}B$，因此主存地址有30位。块大小$128B = 2^{7}B$，因此低7位为块内地址。cache行数为$\frac{64KB}{128B} = 512 = 2^{9}$，因此中间9位为cache行号，高14位为标记字段。
（2）已知cache行数为$512$，而cache数据区64KB。除了数据区，cache还有标记区，每行cache有14位标记位，1位有效位。于是cache总容量应为
\[
    512 \times (1 + 14 + 128 \times 8) = 531968\textbf{位}
\]

### 第12题
根据《计算机系统基础》第6.4.1节内容，对大型典型程序运行情况分析结果表明，在较短时间间隔内，程序产生的地址往往集中在存储空间一个很小的范围，这种现象称为**程序访问的局部性**。这种局部性可以细分为被访问的某个存储单元在一个较短的时间间隔内很可能又被访问的**时间局部性**和被访问的某个存储单元的邻近单元在一个较短的时间间隔内很可能也被访问的**空间局部性**。
Cache的**组相联映射**组合直接映射与全相联映射，取长补短。其将cache分成大小相等的组，每个主存块被映射到cache中的任意一行，采用“组间模映射，组内全映射”的方式，映射关系为\[
   \textbf{cache 行号} = \textbf{主存块号 } mod \textbf{  cache 组数}
\]
对于如题的向量点乘程序：
（1）访问数组x和y时均是按顺序依次访问，因此空间局部性较好；由于数组中每个元素都只被访问一次,因此时间局部性很差。由于数组规模较小且空间局部性较好，推测命中率相对较高，但是也不排除本题问（2）中命中率很低的情形。
（2）主存块大小为16B，cache数据区大小为32B，则cache有$\frac{32B}{16B} = 2$行。对于浮点数组x,y，根据《计算机系统基础》第2.6.1节的内容，C语言中float数据类型的宽度为4。数组x存放在0x8049040（第0x804904块的起始地址）开始的区域，连续存储2个块；数组y存放在0x8049060开始的区域，连续存储2个块。数据cache采用的是直接映射方式，则注意到存储x[0]-x[3]与存储y[0]-y[3]会被映射至cache的同一行，存储x[4]-x[7]与存储y[4]-y[7]会被映射至cache的同一行；而程序中数组x和y被交替访问，则cache在此过程中被不断重写，而命中率为0。
（3）将（2）中的cache改为2-路组相联映射，块大小改为8字节，则cache数据区有4行，被分为2组。同上，我们注意到存储x[0]-x[1]与存储y[0]-y[1]会被映射至cache的同一组，存储x[2]-x[3]与存储y[2]-y[3]会被映射至cache的同一组......但是每组有两行，组内全映射可以使x与y均保存在cache内。因此当i为偶数时，cache不命中，同时将存储x[i]-x[i+1]/y[i]-y[i+1]的主存块复制到cache中；当i为奇数时，cache命中。因此，该程序数据访问的命中率为50%。
（4）在（2）中条件不变的情况下，将数组x定义为```float x[12]```，很有意思，数组x存放在0x8049040（第0x804904块的起始地址）开始的区域，连续存储3个块；数组y存放在0x8049070开始的区域，连续存储2个块。数据cache采用的是直接映射方式，则此时存储x[0]-x[3]与存储y[0]-y[3]会被映射至cache的不同行（分别映射至第0行、第1行），存储x[4]-x[7]与存储y[4]-y[7]也会被映射至cache的不同行（分别映射至第1行、第0行）。程序中数组x和y被交替访问，当 i = 0 或 4 时，cache不命中，同时将存储x[i]-x[i+3]/y[i]-y[i+3]的主存块复制到cache中；当i为其他数时，cache命中。因此，该程序数据访问的命中率为75%。

### 第13题
解决cache一致性的 **回写法（write-back）** 的基本做法是：当CPU执行写操作时，若写命中，信息只被写入cache而不被写入主存；若写不命中则在cache中分配一行，将主存块调入cache行中并更新cache中相应单元的内容（因此该方式下在写不命中时，通常采用写分配法进行写操作）。也即：在CPU执行写操作时，回写法不会更新主存单元，只有当cache行中的主存块被替换且修改位（dirty bit)时才将该主存块内容一次性写会主存，减少了写主存的次数，降低了主存的带宽要求。
本计算机只有一级cache，其中L1数据cache的数据区大小32B，主存块大小16B，则cache有2行。数组dst从主存地址0x804c000开始存放，数组src从主存地址0x804c040开始存放，则（表中斜杠前的数字表示映射到的cache行数，斜杠后为是否命中，下同）：
<table>
  <tr>
    <th colspan="1"></th>
    <th colspan="4">src数组</th>
    <th colspan="4">dst数组</th>
  </tr>
  <tr>
    <td></td>
    <td>col=0</td>
    <td>col=1</td>
    <td>col=2</td>
    <td>col=3</td>
    <td>col=0</td>
    <td>col=1</td>
    <td>col=2</td>
    <td>col=3</td>
  </tr>
  <tr>
    <td>row=0</td>
    <td>0/miss</td>
    <td>0/miss</td>
    <td>0/hit</td>
    <td>0/miss</td>
    <td>0/miss</td>
    <td>0/miss</td>
    <td>0/miss</td>
    <td>0/miss</td>
  </tr>
  <tr>
    <td>row=1</td>
    <td>1/miss</td>
    <td>1/hit</td>
    <td>1/miss</td>
    <td>1/hit</td>
    <td>1/miss</td>
    <td>1/miss</td>
    <td>1/miss</td>
    <td>1/miss</td>
  </tr>
  <tr>
    <td>row=2</td>
    <td>0/miss</td>
    <td>0/miss</td>
    <td>0/hit</td>
    <td>0/miss</td>
    <td>0/miss</td>
    <td>0/miss</td>
    <td>0/miss</td>
    <td>0/miss</td>
  </tr>
  <tr>
    <td>row=3</td>
    <td>1/miss</td>
    <td>1/hit</td>
    <td>1/miss</td>
    <td>1/hit</td>
    <td>1/miss</td>
    <td>1/miss</td>
    <td>1/miss</td>
    <td>1/miss</td>
  </tr>
</table>

若L1的数据区容量改为128B，则一级高速缓存有8行，有：
<table>
  <tr>
    <th colspan="1"></th>
    <th colspan="4">src数组</th>
    <th colspan="4">dst数组</th>
  </tr>
  <tr>
    <td></td>
    <td>col=0</td>
    <td>col=1</td>
    <td>col=2</td>
    <td>col=3</td>
    <td>col=0</td>
    <td>col=1</td>
    <td>col=2</td>
    <td>col=3</td>
  </tr>
  <tr>
    <td>row=0</td>
    <td>4/miss</td>
    <td>4/hit</td>
    <td>4/hit</td>
    <td>4/hit</td>
    <td>0/miss</td>
    <td>0/hit</td>
    <td>0/hit</td>
    <td>0/hit</td>
  </tr>
  <tr>
    <td>row=1</td>
    <td>5/miss</td>
    <td>5/hit</td>
    <td>5/hit</td>
    <td>5/hit</td>
    <td>1/miss</td>
    <td>1/hit</td>
    <td>1/hit</td>
    <td>1/hit</td>
  </tr>
  <tr>
    <td>row=2</td>
    <td>6/miss</td>
    <td>6/hit</td>
    <td>6/hit</td>
    <td>6/hit</td>
    <td>2/miss</td>
    <td>2/hit</td>
    <td>2/hit</td>
    <td>2/hit</td>
  </tr>
  <tr>
    <td>row=3</td>
    <td>7/miss</td>
    <td>7/hit</td>
    <td>7/hit</td>
    <td>7/hit</td>
    <td>3/miss</td>
    <td>3/hit</td>
    <td>3/hit</td>
    <td>3/hit</td>
  </tr>
</table>

### 第19题
处理器带有一个数据区容量为256B的cache，主存块大小为32B（cache有8行）有```sizeof(int) = 4```，每块存放整数数量为8。若cache采用直接映射方式，我们注意到计算缺失率只需要考虑如下代码：
```C
for(i = 0; i < 10000; i ++)
    for(j = 0; j < 128; j += s)
        c = a[j];
```
（1）当 s = 64 且cache采用直接映射方式时，我们注意到a[0]与a[64]的块号一定相差为8，且本程序访存表现为依次循环访问a[0]与a[64]10000次，而存储a[0]的块与存储a[64]的块在cache采用直接映射方式时会被映射至cache的同一行，则没有一次可以命中，命中率为0，缺失率为
\[
    1 - 0 = 100\%
\]
（2）当 s = 63 且cache采用直接映射方式时，本程序访存表现为依次循环访问a[0]，a[63]，a[126] 10000次。依题设，数组a从一个主存块开始处存放，则存储a[63]的块与存储a[126]的块在cache采用直接映射方式时会被映射至cache的同一行，存储a[0]的块与存储a[63]（a[126]）的块在cache采用直接映射方式时会被映射至cache的不同行。在此过程中，a[0]除了第一次循环（i = 0）都将命中，而a[63]与a[126]不会命中，于是命中率为
\[
    \frac{10000-1}{3 \times 10000} \times 100\% = 33.33\%
\]缺失率为
\[
    1 - 33.33\% = 66.67\%
\]
（3）当 s = 64 且cache采用2-路组相连映射方式时，我们注意到a[0]与a[64]的块号一定相差为8，且本程序访存表现为依次循环访问a[0]与a[64]10000次，而存储a[0]的块与存储a[64]的块在cache采用组相连映射方式时会被映射至cache的同一组，但是幸运的是每组中有2行，可以分别存放存储a[0]的块与存储a[64]的块，则在此过程中，a[0]和a[64]除了第一次循环（i = 0）都将命中，于是命中率为
\[
    \frac{2\times (10000-1)}{2 \times 10000} \times 100\% = 99.99\%
\]缺失率为
\[
    1 - 99.99\% = 0.01\%
\]
（4）当 s = 63 且cache采用2-路组相连映射方式时，本程序访存表现为依次循环访问a[0]，a[63]，a[126] 10000次。依题设，数组a从一个主存块开始处存放，则存储a[63]的块与存储a[126]的块在cache采用直接映射方式时会被映射至cache的同一组，存储a[0]的块与存储a[63]（a[126]）的块在cache采用直接映射方式时会被映射至cache的不同组。幸运的是每组中有2行，可以分别存放存储a[63]的块与存储a[126]的块，则在此过程中，a[0]，a[63]，a[126]除了第一次循环（i = 0）都将命中，于是命中率为
\[
    \frac{3\times (10000-1)}{3 \times 10000} \times 100\% = 99.99\%
\]缺失率为
\[
    1 - 99.99\% = 0.01\%
\]

### 第20题
目前各类通用计算机中广泛采用了虚拟存储技术，虚拟存储器主要有三种类型：分页式、分段式、段页式。在**分页式存储系统**中，虚拟地址空间被划分为大小相等的页面，硬盘与主存间按页面交换信息。在虚拟存储机制中采用全相联映射，每个虚拟页可以存放到主存任何一个空闲页框中，页表则用于描述各个虚拟页与所存放的主存页框号或磁盘上存储位置的关系。进程中每个虚拟页在页表中都有一个对应的表项称为页表项，包括虚拟页的存放位置、装入位（valid）、修改位（dirty）、使用位、访问权限位、禁止缓存位等。
本题中，每页大小为$16KB = 2^{14}B$，因此页内地址数为14位；虚拟地址为40位，因此虚页号位数为 $40-14 = 26$；物理地址为36位，因此虚页号位数为 $36-14 = 22$；每个进程的页表项数为 $2^{26}$，每个页表项位数至少为$22 + 4 = 26$。不考虑对齐，每个页表项26位构建页表，每个进程的页表大小为
\[
  26b \times 2^{26} = 1.625 \times 2^{27} B = 208MB
\]考虑对齐简化对页表项的访问，每个页表项32位构建页表，每个进程的页表大小为
\[
  32b \times 2^{26} = 1 \times 2^{28} B = 256MB
\]
正如《计算机系统基础》6.5.3节内容所指出的：页表属于进程控制信息，位于虚拟地址空间的内核空间，页表在主存的首地址记录在页表基址寄存器中。页表的项数由虚拟地址空间大小决定，而虚拟地址空间是一个用户编程不受其限制的足够大的地址空间，因此页表项会很多，带来页表过大的问题。本题中如果按计算出的实际大小构建页表就会遇到页表过大的问题。

### 第21题
在分页式虚拟存储器中，后备转换缓冲器（TLB）通常称为**快表**，比页表小得多，通常具有较高的关联度，大多采用全相联映射或组相联映射，每个表项内容由页表加上一个**TLB标记字段**（表示表项取自哪个虚拟页对应的页表项）构成。在全相联方式下TLB标记字段就是该页表项对应的虚拟页号；在组相联方式下对应虚拟页号的高位部分，虚拟页号的低位部分作为TLB索引用于选择TLB组。
本题中，
（1）虚拟地址有16位，页大小为$128B = 2^{7}B$，因此页内地址数为7位。则虚拟地址中的高9位表示虚拟页号，低7位表示页内偏移量。
TLB采用了4路组相联映射方式，共有16个页表项，因此有$4 = 2^{2}$组。因此虚拟页号的高7位表示TLB标记，低2位表示TLB组索引。
（2）物理地址有12位，同样的低7位表示页内偏移量，而高5位表示物理页号。
（3）本题中cache采用直接映射方式，块大小4B，共16行，则cache数据区大小为$64B = 2^{6}B$，于是物理地址的低2位作为块内地址字段，高6位作为标记字段，中间4位作为行索引字段。
（4）从虚拟地址067AH（0b0000011001111010）中读取一个short变量，依据（1）问中得出的虚拟地址划分，虚拟页号的高7位（0b0000011 = 0x03）表示TLB标记，低2位（0b00 = 0x0）表示TLB组索引。查TLB第0组得到标记为03的项有效位为0；查页表得到虚页号（0b000001100 = 0x0C）对应的页框号为0x19（0b11001），得到物理地址为0b110011111010，再依据（3）问中给出的物理地址划分，得到标记字段为0b110011 = 0x33，行索引字段为$0b1110 = 0xE$，块内地址字段为0b10 = 0x2，cache命中。依据《计算机系统基础》第2.6.1节的内容，C语言中short数据类型的宽度为2。以块内地址为0x2处开始读出两个字节内容为0x4A2D，因此变量的值为0x4A2D。

### 第23题

（1）手工写出循坏语句S对应的指令序列可能如下
```Am
  mov $0, %ecx
.LOOP
  cmp %ebx, %ecx
  je .EXIT
  add (%edx,%ecx,4), %eax
  add $1, %ecx
  jmp .LOOP
.EXIT  
```
使用gcc将下面的C语言源文件编译生成汇编文件
```C
int main(){
  int N=10, i,sum = 0;
  int a[N];
  for(i = 0; i < N; i ++) sum+=a[i];
    return 0;
}
```
由于使用了AMD64架构，gcc version 13.2.0 (Ubuntu 13.2.0-23ubuntu4)，可能与试题要求有一些差异，因此也不再做详细比较。
```Am
main:
.LFB0:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$48, %rsp
	movq	%fs:40, %rax
	movq	%rax, -8(%rbp)
	xorl	%eax, %eax
	movq	%rsp, %rax
	movq	%rax, %rsi
	movl	$10, -28(%rbp)
	movl	$0, -32(%rbp)
	movl	-28(%rbp), %eax
	movslq	%eax, %rdx
	subq	$1, %rdx
	movq	%rdx, -24(%rbp)
	cltq
	leaq	0(,%rax,4), %rdx
	movl	$16, %eax
	subq	$1, %rax
	addq	%rdx, %rax
	movl	$16, %edi
	movl	$0, %edx
	divq	%rdi
	imulq	$16, %rax, %rax
	movq	%rax, %rcx
	andq	$-4096, %rcx
	movq	%rsp, %rdx
	subq	%rcx, %rdx
.L2:
	cmpq	%rdx, %rsp
	je	.L3
	subq	$4096, %rsp
	orq	$0, 4088(%rsp)
	jmp	.L2
.L3:
	movq	%rax, %rdx
	andl	$4095, %edx
	subq	%rdx, %rsp
	movq	%rax, %rdx
	andl	$4095, %edx
	testq	%rdx, %rdx
	je	.L4
	andl	$4095, %eax
	subq	$8, %rax
	addq	%rsp, %rax
	orq	$0, (%rax)
.L4:
	movq	%rsp, %rax
	addq	$3, %rax
	shrq	$2, %rax
	salq	$2, %rax
	movq	%rax, -16(%rbp)
	movl	$0, -36(%rbp)
	jmp	.L5
.L6:
	movq	-16(%rbp), %rax
	movl	-36(%rbp), %edx
	movslq	%edx, %rdx
	movl	(%rax,%rdx,4), %eax
	addl	%eax, -32(%rbp)
	addl	$1, -36(%rbp)
.L5:
	movl	-36(%rbp), %eax
	cmpl	-28(%rbp), %eax
	jl	.L6
	movl	$0, %eax
	movq	%rsi, %rsp
	movq	-8(%rbp), %rdx
	subq	%fs:40, %rdx
	je	.L8
	call	__stack_chk_fail@PLT
.L8:
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
```

（2）IA-32+Linux平台采用两级页表分页虚拟存储管理方式，则控制寄存器CR0中的控制位PE = 1（保护模式）， PG = 1（启用分页）。

（3）指令I实现了赋值语句```sum+=a[i];```，对应的汇编形式（AT&T格式）应该为
```Am
movl	(%eax,%edx,4), %eax
```
依据《计算机系统基础》3.2.2节的内容，存储器操作数寻址方式为比例变址加位移。

（4）当i = 50时，取指令操作过程中MMU得到的线性地址是指令I的线性地址，为0x8048c08，页大小$4KB = 2^{12}B$，虚页号为高20位，为0x8048。页目录索引为高10位，为0b0000100000 = 0x20；页表索引为中10位，为0x0001001000 = 0x48；页内偏移量为低12位，为0xc08。
IA-32+Linux平台采用**两级页表分页虚拟存储管理方式**。指令I的页目录项存储在主存$0x3d000 + 0x080 = 0x3d080$处，大小为4B；指令I的页表项存储在主存$0x5c8000 + 0x120 = 0x5c8120$处；指令I所在页在主存中分配的页框号为1020（我们将其视为十六进制处理），则指令I存放的主存地址为0x1020c08。
第一次执行指令I时，指令I对应页表项中P = 1（说明段已经存在于主存中，Linux总是把P置为1，因为其以页为单位交换），R/W = 0（表示页只能读不能写，因为指令I对应的页为指令页），U/S = 1（表示允许用户进程访问，这是一个C语言源程序对应的用户进程，不是操作系统所使用的页），A = 1（段已经被访问过），D = 1（地址和数据为32位宽）。

（5）指令I在第一次执行时，有可能发生缺页异常，因为取操作数a[0]不能保证不发生缺页异常，可能虚页0x8049此前未被访问。如果发生缺页异常，页故障线性地址为0x8049300，该地址保存在寄存器CR2（页故障线性地址寄存器）中。

（6）根据《计算机系统基础》6.5.3节的内容，TLB命中则页一定命中，可知若缺页则TLB一定缺失。在（5）问中我们指出指令I在第一次执行时，可能发生缺页异常，于是由相同理由可知可能发生TLB缺失。若指令TLB共有16个表项，采用4路组相联方式，则TLB共有4组，虚页号的低2位作为TLB组索引，高18位作为TLB标记。据此可知，对于虚拟地址为0x8048c08的指令I，TLB组索引为0b00 = 0，TLB标记为0x2012.查询TLB的第0组，没有匹配的标记。原来TLB是个烟雾弹。
注意到题目中给出了指令I所在页在主存中分配的页框号为1020（我们将其视为十六进制处理），则指令I存放的主存地址为0x1020c08。

（7）指令cache的数据区容量为8KB，主存块大小32B，采用2-路组相联映射，则cache有$\frac{8KB}{32B} = 2^{8} = 256$行，128组。指令I的地址为0x8048c08，后5位为块内地址，中7位为cache索引组号，为$0b1100000 = 0x60$，因此将映射至cache的第0x60=96组中。考虑到I的地址为0x8048c08，其不在主存块起始位置，大概率不会发生cache缺失（执行此前指令时很可能已经装载）。

（8）当N = 2000时，链接后确定a的首地址为0x8049300，页大小为$4KB = 2^{12}B$，起始虚页号为$0x8049$；共有2000个long型数据元素，每个数据元素分配4字节空间，对于按字节编址，数组a连续存放，末尾为$0x8049300 + 2000 \times 4 = 0x8049300 + 0x1F40 = 0x8051240$。因此数组a占用3个页面，每个页的虚页号依次为0x8049、0x8050、0x8051。数组元素a[1200]应该位于虚页0x8050中。


---