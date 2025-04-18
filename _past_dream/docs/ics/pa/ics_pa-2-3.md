# PA 2-3 完善调试器——字符串求值

## 代码实现
!!! info "完善调试器"
    ##### * 实现表达式的词法分析

    你需要完成以下的内容:

    1. 为算术表达式中的各种`token`类型添加规则, 你需要注意C语言字符串中转义字符的存在和正则表达式中元字符的功能.

    2. 在成功识别出`token`后, 将`token`的信息依次记录到`tokens`数组中.

    ##### * 实现表达式的递归求值 

    你需要实现上文`BNF`中列出的功能. 一个要注意的地方是词法分析中编写规则的顺序, 不正确的顺序会导致一个运算符被识别成两部分, 例如 `!=` 被识别成 `!` 和 `=`. 关于变量的功能, 它需要涉及符号表和字符串表的查找, 因此你会在下一阶段中实现它.

    上面的`BNF`并没有列出C语言中所有的运算符, 例如各种位运算, `<=` 等等. `==` , `!=` 和逻辑运算符很可能在使用监视点的时候用到, 因此要求你实现它们. 如果你在将来的使用中发现由于缺少某一个运算符而感到使用不方便, 到时候你再考虑实现它.

    在完成上述两个任务的时候，你可以尝试分步走的办法：首先针对简单的算术表达式实现其词法和求值功能，再扩展到更为复杂的表达式。

框架代码给出了良好的词法分析框架, 只需要实现`token`的识别和`tokens`数组的记录即可。而且本阶段没有具体需要通过的测试样例，且作为选做阶段是用于补分的，因此本阶段实现丰俭由人，可以良好实现，也可以实现基础功能后直接提交。我们将在编译原理中更深入地讨论词法分析和语法分析。

!!! success "PA2-3阶段结束"
    保持初心，继续前行。
