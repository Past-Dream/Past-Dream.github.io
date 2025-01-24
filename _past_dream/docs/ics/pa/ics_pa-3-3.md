# PA 3-3 分页机制——超越容量的界限

在上一节中，我们实现从逻辑地址到线性地址的转换。在实现分页机制之前，线性地址就当做物理地址使用。而自从80386开始，计算机又提供了一种全新的存储管理方式，那就是分页机制。在分页机制下，每一个进程都拥有独立的存储空间。同时，每一个进程独立的存储空间又具有相同的地址划分方式。此时，线性地址就需要通过进一步的转换，才能获得最终要访问的物理地址。

![pa-3-3](pa_pic/pa-3-3.png)

!!! info "分页机制的模拟"
    1. 修改Kernel和testcase中`Makefile`的链接选项

    2. 在`include/config.h`头文件中定义宏`IA32_PAGE`并`make clean`

    3. 在`CPU_STATE`中添加`CR3`寄存器

    4. 修改`laddr_read()`和`laddr_write()`，适时调用`page_translate()`函数进行地址翻译

    5. 修改Kernel的`loader()`，使用`mm_malloc`来完成对用户进程空间的分配

首先需要处理的是在`CPU_STATE`中添加`CR3`寄存器，这需要我们查阅i386手册，与前述CR0一致的，我们可以找到这样的描述
![CR3](pa_pic/3-3-CR3.png)
由此也可以写出`CR3`的结构模拟
```c
typedef union {
	struct {
		uint32_t reserve :12;
		uint32_t pdbr :20;
	};
	uint32_t val; 	
} CR3;
```
!!! tip "PA3-3的一个小提示"
    在本节的```page_translate()```函数中，我们发现当```#define TLB_ENABLED```时，框架代码给出了TLB查询的实现。因此在继续进行PA3-3时，我们可以先设置```#define TLB_ENABLED```，然后进行laddr_read和laddr_write的实现，最后再取消```#define TLB_ENABLED```并实现```page_translate()```函数。

!!! danger "一个需要注意的地方"
    laddr_read()和laddr_write()的行为也要发生改变：当CR0的PG位被置为1时，需要按照上一小节中所述的方法，查询两级页表，将线性地址转换成物理地址再加以访问。注意在laddr_read()和laddr_write()中要处理地址访问跨越页边界的情形，若发生跨越页边界，则应当将一次读写拆分成两次物理地址读写来进行。


!!! example "一个投机取巧的实现"
    你不难注意到，这样实现也可以通过所有的测试样例，但是，这并不是一个好的实现。
    ```C
    uint32_t laddr_read(laddr_t laddr, size_t len){
        assert(len == 1 || len == 2 || len == 4);
    #ifdef IA32_PAGE
        if( cpu.cr0.pg == 1) {
            paddr_t paddr = page_translate(laddr);
	        return paddr_read(paddr, len);
        } else return paddr_read(laddr, len);
    #else
        return paddr_read(laddr, len);
    #endif
    }
    void laddr_write(laddr_t laddr, size_t len, uint32_t data){
	    assert(len == 1 || len == 2 || len == 4);	
    #ifdef IA32_PAGE
        if( cpu.cr0.pg == 1) {
            paddr_t paddr = page_translate(laddr);
	        paddr_write(paddr, len, data);
        } else  paddr_write(laddr, len, data);
    #else
        paddr_write(laddr, len, data);
    #endif
    }
    ```

!!! bug "一个错误的实现"
    或许在PA的漫漫长途上，一些错误再所难免，在PA3-3中，我犯了一个错误，并在后面造成了灾难性的后果。
    ```
    uint32_t laddr_read(laddr_t laddr, size_t len){
    assert(len == 1 || len == 2 || len == 4);
	    if( cpu.cr0.pg == 1) {
		    if (laddr + len - 1 > (laddr |0xFFF)) {
			    paddr_t paddr = page_translate(laddr);
			    uint32_t temp_l = paddr_read(paddr, (((laddr >> 12) << 12)|0xFFF) - laddr + 1);
			    paddr = page_translate((laddr|0xFFF) + 1);
			    uint32_t temp_h = paddr_read(paddr, len - ((laddr |0xFFF) - laddr + 1));
			    uint32_t res = (temp_h << (8 * (laddr|0xFFF) - laddr + 1)) + temp_l;
			    return res;
		    } else {
			    paddr_t paddr = page_translate(laddr);
			    return paddr_read(paddr, len);	
		    }
	    } else {
	        return paddr_read(laddr, len);
	    }
    }
    void laddr_write(laddr_t laddr, size_t len, uint32_t data){
	    assert(len == 1 || len == 2 || len == 4);
	    if( cpu.cr0.pg == 1) {
		    if (laddr + len - 1 > (laddr|0xFFF)) {
			    paddr_t paddr = page_translate(laddr);
			    paddr_write(paddr, (laddr|0xFFF) - laddr + 1, data & (0xFFFFFFFF >> (8 * (laddr|0xFFF) - laddr + 1)));
			    paddr = page_translate((laddr|0xFFF) + 1);
			    paddr_write(paddr, len - (((laddr|0xFFF) - laddr + 1)), data >> (8 * ((laddr|0xFFF) - laddr + 1)));
		}   else{
			    paddr_t paddr = page_translate(laddr);
			    paddr_write(paddr, len, data);	
		    }
	    }else{
	        paddr_write(laddr, len, data);
	    }
    }
    ```

!!! question "为什么我在这里给出了我曾经错误的实现"
    我曾经想，作为一份看起来很正式的学习记录，不应该出现错误或误导。但是，我后来发现，这并不是一个好的学习记录。本记录的写作之初心是记录学习过程，而不是记录学习成果。因此，我决定在错误的地方加上注释，以提醒自己，也提醒后来者。正如我的一个同学说的
    > 那不是黑历史 那是来时路