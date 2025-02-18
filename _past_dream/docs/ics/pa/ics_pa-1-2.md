# PA 1-2 整数的表示、存储和运算——不停运算的机器

我们首先考察整数的表示、存储和运算问题。在计算机中，一切都是离散的。用于表示各类信息的数据，尽管其含义可能千差万别，但最终在计算机内的表示总可以在形式上归于整数（机器数）形式。因此，从整数开始讨论计算机内数据的表示、存储和运算，是一个较为理想的切入点。
![pa-1](pa_pic/pa-1.png)

## 代码实现
!!! info "ALU的模拟"
    1. 实现`nemu/src/cpu/alu.c`中的各个整数运算函数；

    2. 使用
    > make clean
    > make

    命令编译项目；

    3. 使用

    > make test_pa-1

    命令执行NEMU并通过各个整数运算测试用例。

在ALU中，我们实现了整数运算的各个函数，包括加法、减法、乘法、除法、逻辑与、逻辑或、逻辑异或、逻辑非、移位等。这些函数在`nemu/src/cpu/alu.c`中实现。对于整数运算，标志位`ZF`、`SF`、`OF`、`CF`、`PF`等标志位在运算过程中会被设置。其中，`ZF`、`SF`、`PF`的逻辑在各个运算间保持了较好的一致性，我们实现如下（如你所见，`PF`实现的有些笨重，你可以采用更加精致的方法来实现这一点）。

```C
void set_ZF(uint32_t result, size_t data_size) {
	result = result & (0xFFFFFFFF >> (32 - data_size));
	cpu.eflags.ZF = (result == 0);
	return;
}
void set_SF(uint32_t result, size_t data_size) {
	result = sign_ext(result & (0xFFFFFFFF >> (32 - data_size)), data_size);
	cpu.eflags.SF = sign(result);
	return;
}
void set_PF(uint32_t result) {
    result = result & 0xFF;
    if(result == 0||result == 3||result == 5||result == 6||result == 9||result == 10||result == 12||result == 15||result == 17||result == 18||result == 20||result == 23||result == 24||result == 27||result == 29||result == 30||result == 33||result == 34||result == 36||result == 39||result == 40||result == 43||result == 45||result == 46||result == 48||result == 51||result == 53||result == 54||result == 57||result == 58||result == 60||result == 63||result == 65||result == 66||result == 68||result == 71||result == 72||result == 75||result == 77||result == 78||result == 80||result == 83||result == 85||result == 86||result == 89||result == 90||result == 92||result == 95||result == 96||result == 99||result == 101||result == 102||result == 105||result == 106||result == 108||result == 111||result == 113||result == 114||result == 116||result == 119||result == 120||result == 123||result == 125||result == 126||result == 129||result == 130||result == 132||result == 135||result == 136||result == 139||result == 141||result == 142||result == 144||result == 147||result == 149||result == 150||result == 153||result == 154||result == 156||result == 159||result == 160||result == 163||result == 165||result == 166||result == 169||result == 170||result == 172||result == 175||result == 177||result == 178||result == 180||result == 183||result == 184||result == 187||result == 189||result == 190||result == 192||result == 195||result == 197||result == 198||result == 201||result == 202||result == 204||result == 207||result == 209||result == 210||result == 212||result == 215||result == 216||result == 219||result == 221||result == 222||result == 225||result == 226||result == 228||result == 231||result == 232||result == 235||result == 237||result == 238||result == 240||result == 243||result == 245||result == 246||result == 249||result == 250||result == 252||result == 255)  cpu.eflags.PF = 1;
    else cpu.eflags.PF = 0;
    return;
}
```
此后，我们参考《计算机系统基础》或i386手册，实现整数运算的各个函数。
加法运算的实现代码可以参考PA1-2的PPT，并在其他运算中以此为参考，其中对于代码健壮性的考量是值得学习的。作为刚开篇的PA实验，我们需要注意两点——尽管这些可能的注意事项显得有些幼稚。

!!! tip "PA1-2的注意事项"
    1.在```alu_```运算的实现中，记得```return ans```（返回计算结果）而不是```return 0```。
    2.在```alu_```运算的实现中，注意带符号整数与无符号整数的区别，注意```src```和```dest```的符号位。

```C
void set_CF_add(uint32_t res, uint32_t src, size_t data_size){
    res = sign_ext(res & (0xFFFFFFFF >> (32 - data_size)), data_size);
    src = sign_ext(src & (0xFFFFFFFF >> (32 - data_size)), data_size);
    cpu.eflags.CF = (res < src);
    return;
}
void set_OF_add(uint32_t result, uint32_t src, uint32_t dest, size_t data_size) {
	switch(data_size) {
		case 8: 
			result = sign_ext(result & 0xFF, 8); 
			src = sign_ext(src & 0xFF, 8); 
			dest = sign_ext(dest & 0xFF, 8); 
			break;
		case 16: 
			result = sign_ext(result & 0xFFFF, 16); 
			src = sign_ext(src & 0xFFFF, 16); 
			dest = sign_ext(dest & 0xFFFF, 16); 
			break;
		default: break;// do nothing
	}
	if(sign(src) == sign(dest)) {
		if(sign(src) != sign(result))
			cpu.eflags.OF = 1;
		else
			cpu.eflags.OF = 0;
	} else {
		cpu.eflags.OF = 0;
	}
}

uint32_t alu_add(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_add(src, dest, data_size);
#else
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
    src = src & (0xFFFFFFFF >> (32 - data_size));
	uint32_t res = 0;
	res = dest + src;
	set_CF_add(res, src, data_size);
	set_PF(res);
	//set_AF();
	set_ZF(res, data_size);
	set_SF(res, data_size);
	set_OF_add(res, src, dest, data_size);
	return  res & (0xFFFFFFFF >> (32 - data_size));
#endif
}

void set_CF_adc(uint32_t res, uint32_t src, size_t data_size){
    res = sign_ext(res & (0xFFFFFFFF >> (32 - data_size)), data_size);
    src = sign_ext(src & (0xFFFFFFFF >> (32 - data_size)), data_size);
    if(cpu.eflags.CF == 1) cpu.eflags.CF = (res <= src);
    else cpu.eflags.CF = (res < src);
    return;
}
void set_OF_adc(uint32_t result, uint32_t src, uint32_t dest, size_t data_size) {
	switch(data_size) {
		case 8: 
			result = sign_ext(result & 0xFF, 8); 
			src = sign_ext(src & 0xFF, 8); 
			dest = sign_ext(dest & 0xFF, 8); 
			break;
		case 16: 
			result = sign_ext(result & 0xFFFF, 16); 
			src = sign_ext(src & 0xFFFF, 16); 
			dest = sign_ext(dest & 0xFFFF, 16); 
			break;
		default: break;// do nothing
	}
	if(sign(src) == sign(dest)) {
		if(sign(src) != sign(result))
			cpu.eflags.OF = 1;
		else
			cpu.eflags.OF = 0;
	} else {
		cpu.eflags.OF = 0;
	}
}
uint32_t alu_adc(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_adc(src, dest, data_size);
#else
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
    src = src & (0xFFFFFFFF >> (32 - data_size));
	uint32_t res = 0;
	res = dest + src + cpu.eflags.CF;
	set_CF_adc(res, src, data_size);
	set_PF(res);
	//set_AF();
	set_ZF(res, data_size);
	set_SF(res, data_size);
	set_OF_adc(res, src, dest, data_size);
	return  res & (0xFFFFFFFF >> (32 - data_size));
#endif
}
```

为简便起见，我们下面将直接依次给出所有运算的参考实现代码。
```C
void set_CF_sub(uint32_t src, uint32_t dest, size_t data_size){
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
    src = src & (0xFFFFFFFF >> (32 - data_size));
    cpu.eflags.CF = (dest < src);
    return;
}
void set_OF_sub(uint32_t result, uint32_t src, uint32_t dest, size_t data_size) {
	switch(data_size) {
		case 8: 
			result = sign_ext(result & 0xFF, 8); 
			src = sign_ext(src & 0xFF, 8); 
			dest = sign_ext(dest & 0xFF, 8); 
			break;
		case 16: 
			result = sign_ext(result & 0xFFFF, 16); 
			src = sign_ext(src & 0xFFFF, 16); 
			dest = sign_ext(dest & 0xFFFF, 16); 
			break;
		default: break;// do nothing
	}
	if(sign(src) != sign(dest)) {
		if(sign(dest) != sign(result))
			cpu.eflags.OF = 1;
		else
			cpu.eflags.OF = 0;
	} else {
		cpu.eflags.OF = 0;
	}
}
uint32_t alu_sub(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_sub(src, dest, data_size);
#else
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
    src = src & (0xFFFFFFFF >> (32 - data_size));
	uint32_t res = 0;
	res = dest - src;
	set_CF_sub(src, dest, data_size);
	set_PF(res);
	//set_AF();
	set_ZF(res, data_size);
	set_SF(res, data_size);
	set_OF_sub(res, src, dest, data_size);
	return  res & (0xFFFFFFFF >> (32 - data_size));
#endif
}

void set_CF_sbb(uint32_t src, uint32_t dest, size_t data_size){
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
    src = src & (0xFFFFFFFF >> (32 - data_size));
    if(!cpu.eflags.CF)  cpu.eflags.CF = (dest < src);
    else cpu.eflags.CF = (dest <= src);
    return;
}
void set_OF_sbb(uint32_t result, uint32_t src, uint32_t dest, size_t data_size) {
	switch(data_size) {
		case 8: 
			result = sign_ext(result & 0xFF, 8); 
			src = sign_ext(src & 0xFF, 8); 
			dest = sign_ext(dest & 0xFF, 8); 
			break;
		case 16: 
			result = sign_ext(result & 0xFFFF, 16); 
			src = sign_ext(src & 0xFFFF, 16); 
			dest = sign_ext(dest & 0xFFFF, 16); 
			break;
		default: break;// do nothing
	}
	if(sign(src) != sign(dest)) {
		if(sign(dest) != sign(result))
			cpu.eflags.OF = 1;
		else
			cpu.eflags.OF = 0;
	} else {
		cpu.eflags.OF = 0;
	}
}
uint32_t alu_sbb(uint32_t src, uint32_t dest, size_t data_size){
#ifdef NEMU_REF_ALU
	return __ref_alu_sbb(src, dest, data_size);
#else
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
    src = src & (0xFFFFFFFF >> (32 - data_size));
	uint32_t res = 0;
	res = dest - (src + cpu.eflags.CF);
	set_CF_sbb(src, dest, data_size);
	set_PF(res);
	//set_AF();
	set_ZF(res, data_size);
	set_SF(res, data_size);
	set_OF_sbb(res, src, dest, data_size);
	return  res & (0xFFFFFFFF >> (32 - data_size));
#endif
}


void set_CF_OF_mul(uint64_t res , size_t data_size){
    res = res >> data_size;
    cpu.eflags.CF = (res != 0);
    cpu.eflags.OF = (res != 0);
    return;
}
void set_ZF_mul(uint64_t result, size_t data_size) {
	result = result & (0xFFFFFFFFFFFFFFFF >> (64 - data_size));
	cpu.eflags.ZF = (result == 0);
	return;
}
void set_SF_mul(uint64_t result, size_t data_size) {
	result = result & (0xFFFFFFFFFFFFFFFF >> (64 - data_size));
	cpu.eflags.SF = ((result >> (data_size - 1)) & 0x1);
	return;
}
void set_PF_mul(uint64_t result) {
    result = result & 0xFF;
    if(result == 0||result == 3||result == 5||result == 6||result == 9||result == 10||result == 12||result == 15||result == 17||result == 18||result == 20||result == 23||result == 24||result == 27||result == 29||result == 30||result == 33||result == 34||result == 36||result == 39||result == 40||result == 43||result == 45||result == 46||result == 48||result == 51||result == 53||result == 54||result == 57||result == 58||result == 60||result == 63||result == 65||result == 66||result == 68||result == 71||result == 72||result == 75||result == 77||result == 78||result == 80||result == 83||result == 85||result == 86||result == 89||result == 90||result == 92||result == 95||result == 96||result == 99||result == 101||result == 102||result == 105||result == 106||result == 108||result == 111||result == 113||result == 114||result == 116||result == 119||result == 120||result == 123||result == 125||result == 126||result == 129||result == 130||result == 132||result == 135||result == 136||result == 139||result == 141||result == 142||result == 144||result == 147||result == 149||result == 150||result == 153||result == 154||result == 156||result == 159||result == 160||result == 163||result == 165||result == 166||result == 169||result == 170||result == 172||result == 175||result == 177||result == 178||result == 180||result == 183||result == 184||result == 187||result == 189||result == 190||result == 192||result == 195||result == 197||result == 198||result == 201||result == 202||result == 204||result == 207||result == 209||result == 210||result == 212||result == 215||result == 216||result == 219||result == 221||result == 222||result == 225||result == 226||result == 228||result == 231||result == 232||result == 235||result == 237||result == 238||result == 240||result == 243||result == 245||result == 246||result == 249||result == 250||result == 252||result == 255)  cpu.eflags.PF = 1;
    else cpu.eflags.PF = 0;
    return;
}
uint64_t alu_mul(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_mul(src, dest, data_size);
#else
	uint64_t res = 1;
	src = src  & (0xFFFFFFFF >> (32 - data_size));
	dest = dest  & (0xFFFFFFFF >> (32 - data_size));
	res = res * src * dest;
	res = res & (0xFFFFFFFFFFFFFFFF >> (64 - 2*data_size));
	set_PF_mul(res);
	//set_AF();
	set_ZF_mul(res, 2*data_size);
	set_SF_mul(res, 2*data_size);
	set_CF_OF_mul(res , data_size);
	return res;
#endif
}

int64_t alu_imul(int32_t src, int32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_imul(src, dest, data_size);
#else
	int64_t res = 1;
	res = res * src * dest;
	set_PF_mul(res);
	//set_AF();
	set_ZF_mul(res, 2*data_size);
	set_SF_mul(res, 2*data_size);
	set_CF_OF_mul(res , data_size);
	return res;
#endif
}

uint32_t alu_div(uint64_t src, uint64_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_div(src, dest, data_size);
#else
	uint32_t res = 0;
	res = dest / src;
	return res & (0xFFFFFFFF >> (32 - data_size));
#endif
}

// need to implement alu_imod before testing
int32_t alu_idiv(int64_t src, int64_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_idiv(src, dest, data_size);
#else
	int32_t res = 0;
	res = dest / src;
	return res & (0xFFFFFFFF >> (32 - data_size));
#endif
}

uint32_t alu_mod(uint64_t src, uint64_t dest)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_mod(src, dest);
#else
	uint32_t res = 0;
	res = dest % src ;
	return res & 0xFFFFFFFF;
#endif
}

int32_t alu_imod(int64_t src, int64_t dest)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_imod(src, dest);
#else
	int32_t res = 0;
	res = dest % src;
	return res;
#endif
}

void set_CF_and(){
    cpu.eflags.CF = 0;
    return;
}
void set_OF_and(){
    cpu.eflags.OF = 0;
    return;
}
uint32_t alu_and(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_and(src, dest, data_size);
#else
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
    src = src & (0xFFFFFFFF >> (32 - data_size));
	uint32_t res = 0;
	res = src & dest;
	set_CF_and();
	set_PF(res);
	//set_AF();
	set_ZF(res, data_size);
	set_SF(res, data_size);
	set_OF_and();
	return  res & (0xFFFFFFFF >> (32 - data_size));
#endif
}


void set_CF_xor(){
    cpu.eflags.CF = 0;
    return;
}
void set_OF_xor(){
    cpu.eflags.OF = 0;
    return;
}
uint32_t alu_xor(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_xor(src, dest, data_size);
#else
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
    src = src & (0xFFFFFFFF >> (32 - data_size));
	uint32_t res = 0;
	res = src ^ dest;
	set_CF_xor();
	set_PF(res);
	//set_AF();
	set_ZF(res, data_size);
	set_SF(res, data_size);
	set_OF_xor();
	return  res & (0xFFFFFFFF >> (32 - data_size));
#endif
}


void set_CF_or(){
    cpu.eflags.CF = 0;
    return;
}
void set_OF_or(){
    cpu.eflags.OF = 0;
    return;
}
uint32_t alu_or(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_or(src, dest, data_size);
#else
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
    src = src & (0xFFFFFFFF >> (32 - data_size));
	uint32_t res = 0;
	res = src | dest;
	set_CF_and();
	set_PF(res);
	//set_AF();
	set_ZF(res, data_size);
	set_SF(res, data_size);
	set_OF_and();
	return  res & (0xFFFFFFFF >> (32 - data_size));
#endif
}

void set_CF_shl(uint32_t dest, uint32_t ShiftAmt, size_t data_size){
    cpu.eflags.CF = ((dest >> (data_size - ShiftAmt)) & 1);
    return;
}
void set_OF_shl(uint32_t res, size_t data_size){
    uint32_t res_sign_temp = res >> (data_size - 1);
    cpu.eflags.OF = (res_sign_temp != cpu.eflags.CF);
    return;
}
uint32_t alu_shl(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_shl(src, dest, data_size);
#else
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
	uint32_t res = 0 , ShiftAmt = 0;
	ShiftAmt = src % 32;
	res = dest << ShiftAmt;
	set_CF_shl(dest , ShiftAmt , data_size);
	set_PF(res);
	//set_AF();
	set_ZF(res, data_size);
	set_SF(res, data_size);
	set_OF_shl(res , data_size);
	return  res & (0xFFFFFFFF >> (32 - data_size));
#endif
}


void set_CF_shr(uint32_t dest, uint32_t ShiftAmt, size_t data_size){
    cpu.eflags.CF = ((dest >> (ShiftAmt - 1)) & 1);
    return;
}
void set_OF_shr(uint32_t dest ,size_t data_size){
    cpu.eflags.OF = ((dest >> (data_size - 1)) & 1);
    return;
}
uint32_t alu_shr(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_shr(src, dest, data_size);
#else
    dest = dest & (0xFFFFFFFF >> (32 - data_size));
	uint32_t res = 0 , ShiftAmt = 0;
	ShiftAmt = src % 32;
	res = dest >> ShiftAmt;
	set_CF_shr(dest , ShiftAmt , data_size);
	set_PF(res);
	//set_AF();
	set_ZF(res, data_size);
	set_SF(res, data_size);
	set_OF_shr(dest , data_size);
	return res & (0xFFFFFFFF >> (32 - data_size));
#endif
}

void set_CF_sar(uint32_t dest, uint32_t ShiftAmt, size_t data_size){
    cpu.eflags.CF = ((dest >> (ShiftAmt - 1)) & 1);
    return;
}
void set_OF_sar(){
    cpu.eflags.OF = 0;
    return;
}
uint32_t alu_sar(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_sar(src, dest, data_size);
#else
	dest = dest & (0xFFFFFFFF >> (32 - data_size));
	uint32_t res = 0 , ShiftAmt = 0 , temp_res_s = 1;
	ShiftAmt = src % 32;
	if(ShiftAmt >= data_size){
	    ShiftAmt = data_size;
	}
	uint32_t dest_sign_temp = dest >> (data_size - 1);
	res = dest >> ShiftAmt;
	if(dest_sign_temp){
	for(int j = 0 ; j <= data_size - 1; j++){
	   if(j >= data_size - ShiftAmt) res += temp_res_s;
	   temp_res_s *= 2;
	}
	}
	set_CF_sar(dest , ShiftAmt , data_size);
	set_PF(res);
	//set_AF();
	set_ZF(res, data_size);
	set_SF(res, data_size);
	set_OF_sar();
	return res & (0xFFFFFFFF >> (32 - data_size));
#endif
}

uint32_t alu_sal(uint32_t src, uint32_t dest, size_t data_size)
{
#ifdef NEMU_REF_ALU
	return __ref_alu_sal(src, dest, data_size);
#else
	return alu_shl(src, dest, data_size);
#endif
}
```

此后使用

> make test_pa-1

命令执行NEMU并通过各个整数运算测试用例，若测试用例均通过，则说明PA-1的整数运算部分实现正确。

## 框架导读
!!! tip "我们ALU的模拟实现是如何被测试的？"

在PA1阶段，当我们执行测试时，事实上对应了`Makefile`中的如下一系列
```
test_pa-1: nemu 
	$(call git_commit, "test_pa-1", $(TIME_MAKE))
	./nemu/nemu --test-reg
	./nemu/nemu --test-alu add
	./nemu/nemu --test-alu adc
	./nemu/nemu --test-alu sub
	./nemu/nemu --test-alu sbb
	./nemu/nemu --test-alu and
	./nemu/nemu --test-alu or
	./nemu/nemu --test-alu xor
	./nemu/nemu --test-alu shl
	./nemu/nemu --test-alu shr
	./nemu/nemu --test-alu sal
	./nemu/nemu --test-alu sar
	./nemu/nemu --test-alu mul
	./nemu/nemu --test-alu div
	./nemu/nemu --test-alu imul
	./nemu/nemu --test-alu idiv
	./nemu/nemu --test-fpu add
	./nemu/nemu --test-fpu sub
	./nemu/nemu --test-fpu mul
	./nemu/nemu --test-fpu div
```
本阶段，我们主要关注`test-alu`的测试用例，有关的测试文件位于```pa_nju/nemu/src/cpu/test/```目录下。我们来看一下怎么个事。

我们这里以`alu_test_add()`为例，来看一下测试用例的实现。
```C
#define assert_res_CPSZO(dataSize) \
		"pushf;" \
		"popl %%edx;" \
		: "=a" (res_asm), "=d" (res_eflags) \
		: "a" (a), "c" (b)); \
	test_eflags.val = res_eflags; \
	res_asm = res_asm & (0xFFFFFFFF >> (32 - dataSize)); \
	fflush(stdout); \
	assert(res == res_asm); \
	assert(cpu.eflags.CF == test_eflags.CF); \
	assert(cpu.eflags.PF == test_eflags.PF); \
	assert(cpu.eflags.SF == test_eflags.SF); \
	assert(cpu.eflags.ZF == test_eflags.ZF); \
	assert(cpu.eflags.OF == test_eflags.OF); \

void alu_test_add() {
	uint32_t a, b;
	int input[] = {0x10000000, -3, -2, -1, 0, 1, 2};
	int n = sizeof(input) / sizeof(int);
	int i, j;
	for(i = 0 ; i < n ; i++) {
		for(j = 0 ; j < n ; j++) {
			a = input[i];
			b = input[j];
			{internel_alu_test_CPSZO(alu_add, 32, "addl %%ecx, %%eax;")}
			{internel_alu_test_CPSZO(alu_add, 16, "addw %%cx, %%ax;")}
			{internel_alu_test_CPSZO(alu_add, 8 , "addb %%cl, %%al;")}
		}
	}

	srand(time(0));
	for(i = 0 ; i < 1000000 ; i++) {
		a = rand();
		b = rand();
		{internel_alu_test_CPSZO(alu_add, 32, "addl %%ecx, %%eax;")}
		{internel_alu_test_CPSZO(alu_add, 16, "addw %%cx, %%ax;")}
		{internel_alu_test_CPSZO(alu_add, 8 , "addb %%cl, %%al;")}
	}
	printf("alu_test_add()  \e[0;32mpass\e[0m\n");
	if( get_ref() ) printf("\e[0;31mYou have used reference implementations, DO NOT submit this version!\e[0m\n");
}
```
首先测试样例先测试了一些边缘条件下的情况，接着随机测试了1000000次。其中每一次又分别对32位、16位、8位的数值、标志位进行了测试，将实现的结果与内联汇编的结果相比较，若不相等，则打印出错误信息。

于是，我们如果遇到测试样例与内联汇编结果不一致的情况，也可以仿照上述测试样例，对`#define assert_res_CPSZO(dataSize) `或其他内容进行修改，输出错误的测试样例与结果。

在下一阶段对于浮点数运算的模拟中，测试样例更加清晰，也给出了用于输出调试的代码（在测试样例中被注释了，尽管其中一些随着版本的迭代不能直接使用），我们会在下一阶段的框架导读中介绍。

!!! success "PA-1-2阶段结束"
    本讲内容从另一个视角对ALU进行了理解与模拟（在前置数字逻辑电路与计算机组成原理中，我们是从数字电路的角度来理解并模拟实现ALU的）。事实上，不得不承认我那一部分学的很差，不过，那种勃勃生机、万物竞发的境界犹在眼前。（下附本人数电课程ALU的实现图）

	![PA-1-2](pa_pic/pa-1-2-success.png)

    > 境自远尘皆入咏，物含妙理总堪寻
