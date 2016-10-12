# CIL基础指令
---

## 定义
* CIL（Common Intermediate Language）：公共中间语言，是.NET平台使用的，在CLR上运行的中间语言，由支持.NET的各种语言编译生成的exe、dll均为CIL语言。CIL指令集是图灵完备的。

## 用途
* 了解CIL可以很好的帮助了解.NET语言的各种实现方式，以及可以通过CIL实现高层语言之间的无缝互转。

## 基本指令（使用内置类型）

## add
* 将栈顶两个值相加并重新压栈
* 整型变量操作不检查溢出，除非使用add.ovf
* 浮点数溢出将返回+inf或-inf

## add.ovf & add.ovf.un
* 作用与add相同，add.ovf为有符号整型加法，add.ovf.un为无符号整型加法
* 对结果进行溢出检查，如果溢出则抛出OverflowException

## and
* 将栈顶两个值进行位与并重新压栈

## arglist
* 将当前方法的参数列表引用压栈，类型为System.RuntimeArgumentHandle

## beq target & beq.s target
* 弹出栈顶两个元素并比较，相等则跳转至target
* 效果与ceq + brtrue target相同
* beq比较int32，beq.s比较int8
* 若跳转至target且target包含有前缀，则控制被交给target的第一个前缀

## bge target & bge.s target
* 弹出栈顶两个元素并比较，若第二个弹出的元素大于等于第一个则跳转至target
* 效果与clt(clt.un) + brfalse target相同

## bge.un & bge.un.s
* bge的无符号（无序）版本

## bgt target & bgt.s target
* 弹出栈顶两个元素并比较，若第二个弹出的元素大于第一个则跳转至target

## bgt.un & bgt.un.s
* bgt的无符号（无序）版本

## ble target & ble.s target
* 弹出栈顶两个元素并比较，若第二个弹出的元素小于等于第一个则跳转至target
* 效果与cgt(cgt.un) + brfalse相同

## ble.un & ble.un.s
* ble的无符号（无序）版本

## blt & blt.s & blt.un & blt.un.s
* ble的不判等版本

## bne.un & bne.un.s
* beq相反的版本，判不等

## br target
* 无条件跳转至target

## break
* 调试用，break告诉调试器到达一个断点，除此之外无其他对程序的副作用
* break会导致当前程序“陷入”调试器，具体的行为由调试器来实现

## brfalse target & brfalse.s & brnull & brnull.s & brzero & brzero.s
* 其中brfalse、brnull、brzero是完全相同的指令，brfalse.s、brnull.s、brzero.s是完全相同的指令
* 弹出栈顶元素并判断栈顶元素是否为0（或false、null），若为0（或false、null）则跳转至target，否则继续执行之后的指令

## brtrue target & brtrue.s & brinst & brinst.s
* 其中brtrue、brinst是完全相同的指令，brtrue.s、brinst.s是完全相同的指令
* 与brfalse正好相反，跳转条件是弹出的栈顶元素的并判断是否为非0（非null）

## call method
* 调用method指定的方法，method为指定某方法的元数据
* 从栈中弹出N个变量做为方法的实参传入，出栈顺序与参数顺序相反，若方法有返回值则方法结束后将返回值压栈
* method元数据为methodref、methoddef、methodspec中的一种，其包含的信息可以确定所调用函数类型（静态、实例成员、虚函数或全局函数）。与callvirt指令不同，这里直接使用method元数据中的函数入口地址，callvirt函数则由callvirt之前压栈的变量类型决定其调用函数的入口位置。故call指令调用的直接是基类的函数。
* 所有的方法在编译器中都会添加一个参数，该参数在method参数列表的第一个，指示调用该method的实例对象。其中对于值类型调用的方法，this指针不指向某一实例，而是一个托管指针。
* 可以直接使用call指令调用虚函数，此时不再动态根据调用调用方法的类型进行推断，而是直接使用method包含的元数据来指定，常用于调用基类函数时使用。
* 委托的Invoke方法被call或callvirt指令调用均可。
* 函数调用的参数由starg指令隐式传入。
* 在method运行前，call从运算栈中弹出参数和this指针，其中最后弹出的arg0是this指针，将作为method实际的第一个参数被传入方法。方法结束时若有返回值，则返回值被压入运算栈中。

## calli callsitedescr
* 间接函数调用
* calli指令调用栈顶的所指向的方法，并将栈内其他值做为实参传入

## ceq
* 比较栈顶的两个值，若相等则在运算栈中压入1，否则压入0
* 无限大、小的两个值是相等的

## cgt & cgt.un
* 比较栈顶的两个值，若第二个弹出的大于第一个弹出的，则在运算栈中压入1，否则压入0
* cgt.un为无符号整型或不可比较类型使用的指令

## Ckfinite
* 检查栈顶元素是否为有限值，不对栈做操作，当栈顶元素为NaN或+/-infinity的时候抛出ArithmeticException。

## clt & clt.un
* 逻辑与cgt刚好相反

## conv.<to type> & conv.ovf.<to type> & conv.ovf.<to type>.un
* 类型转换指令，支持的type有i1, i2, i4, i8, r4, r8, u1, u2, u4, u8, i, u, r.un
* i系列转换为有符号整型，分别为1、2、4、8字节大小整型
* r系列转换为浮点数，分别为4、8字节大小浮点数
* u系列转换为无符号整型，分别为1、2、4、8字节大小整型
* i转换为本地int类型，u转换为本地无符号int类型，r.un将无符号整数转换为浮点数（涉及位扩展）
* 方式为转换栈顶，并将结果重新压栈
* 浮点数向整型的转换均向零取整，即正数向下取整，负数向上取整
* conv.<to type>转换在溢出时不会抛出异常，conv.ovf.<to type>会抛出OverflowException

## cpblk
* 内存赋值指令，运算栈中依次弹出复制大小（unsigned int）、源地址（native int或&）、目的地址（native int或&）三个参数
* 该指令不会检查内存是否越界，故当src和dst内存重叠时，无法预测运行结果
* 该指令默认假设src和dst都已经内存对齐，但可以被volatile或unaligned前缀指令修饰而改变其特征
* 当地址无效时会抛出NullReferenceException异常

## Div & div.un
* 除法指令，栈顶第二个弹出的变量除以第一个弹出的变量，并将结果压栈
* 特别地：0/0=NaN，infinity/infinity=NaN，X/infinity=0。
* 若相除后无法正确表示，则会抛出ArithmeticException
* 若被除数不为0而除数为0则抛出DivideByZeroException
* 浮点数除法不会抛出异常
* div.un会将变量当做无符号整数处理，无论其类型如何，且所得结果也为无符号整数

## dup
* 复制栈顶元素并重新压栈

## endfilter
* 异常处理的异常过滤结束指令
* 从栈顶弹出value，value可以表示是否匹配异常，匹配则不再搜索，否则继续搜索catch中的异常处理

## endfinally & endfault
* finally块代码的或fault块代码段结束指令

## initblk
* 初始化内存块的指令
* 从运算栈中分别弹出size（unsigned int32）、value（unsigned int8）、addr（native int），从addr开始初始化size大小的内存，使其初值为value。

## jmp method
* 跳出当前函数，并跳转到method函数
* 其中method为一个methodref或methoddef的元数据
* 运行当前指令时运算栈应该为空

## ldarg.<length>
* 将值类型实参压栈
* ldarg.N表示将第N的参数压栈
* 对于变长参数函数，ldarg仅会压栈指定了的参数，可变部分不会压栈

## ldarga.<length>
* 将引用类型实参压栈
* 该方法对应ldarg，a表示address，即将该参数的托管指针压入运算栈

## ldc.<type>
* 将值类型常量压栈
* 包含ldc.i4 num, ldc.i8 num, ldc.r4 num, ldc.r8 num, ldc.i4.0...8, ldc.r4.m1, ldc.r4.M1, ldc.i4.s num
* 带num的将num转换成相应格式压栈，i表示整型，r表示浮点型，-1...8可以直接使用特定指令压栈为int32类型的常量。
* 对于32位以上的常量，直接使用ldc.i8压栈。对于大于8，可以在32位以内表示的常量使用conv.i8转换后使用ldc.i4压栈

## ldftn
* 将函数指针压栈
* 将托管指针压入运算栈顶，可用于calli进行调用

## ldind.<type>
* 间接加载变量到运算栈顶
* 从栈顶获取变量内存地址，并读取该值后压回栈顶
* 包含ldind.i1, ldind.i2, ldind.i4, ldind.i8, ldind.u1, ldind.u2, ldind.u4, ldind.u8, ldind.r4, ldind.r8, ldind.i, ldind.ref
* 其中，除了ldind.ref是加载引用外，其他均为加载值类型。值类型的指令都是ldobj指令的内置简化版本。

## ldloc & ldloc.s
* 加载局部变量到栈顶
* ldloc indx 加载第indx个局部变量到栈顶

## ldloca & ldloca.s
* ldloc的引用加载版本，将局部变量indx的地址压入栈顶

## ldnull
* 在栈顶压入一个空指针

## leave target & leave.s target
* 离开当前代码段，跳转到target所指示的代码段，此指令会清空运算栈
* leave和br均不能跳出finally代码段，且leave可以跳出多层嵌套代码

## localloc
* 在本地动态内存池中分配空间
* 从运算栈中取出size（native unsigned int or U4），并在本地动态内存池中分配内存，然后将分配好的内存起始地址压回栈顶
* 空间不足时会抛出StackOverflowException。

## mul & mul.ovf
* 乘法的不检查溢出和检查溢出指令
* 从栈顶取两个数，将结果压回栈顶

## neg & neg.ovf
* 取负的不检查溢出和检查溢出指令，取负溢出发生在对最小负数取负的操作中
* 从栈顶取数，取负后压回栈顶

## nop
* 空指令，什么都不执行，用于填充代码中不对齐的空缺

## not
* 按位取反，从栈顶取一个值，按位取反后压回栈顶

## or
* 按位或，从栈顶取两个值，按位或后压回栈顶

## pop
* 从栈顶弹出一个值

## rem & rem.un
* 求模指令，remainder为求得的余数，会压回栈顶

## ret
* return指令，将被调用函数的运算栈中的返回值取出，压入调用函数的运算栈顶。
* 执行该指令时，被调用函数除了返回值外运算栈内应该没有其他变量

## shl
* 左移指令，从栈中分别弹出shiftAmount（int32或native int）和value（int32，int64或native int），将value左移shiftAmount位，用零补齐并将结果压回栈顶

## shr & shr.un
* 与shl对应的右移指令，shr和shr.un分别对应有无符号，无符号始终用0补高位，有符号则补充位与符号位一致。

## starg num & starg.s num
* 将变量存入参数槽num中
* 从栈顶弹出一个变量，并将其存入参数槽num中

## stind
* 将栈顶变量间接存入内存某地址中，弹出栈顶变量，弹出内存地址

## stloc indx
* 将栈顶元素弹出，并存储到局部变量indx中

## sub & sub.vof & sub.vof.un
* 减法的不检查溢出和检查溢出指令

## switch
* switch实现的是一个跳转表，swtich(N,t1,t2...tN)中N表示参数个数，tN是一系列标识跳转目标的数字，switch将逐个比较tM和N，小于N的将会执行，大于等于N的将会跳过。

## xor
* 取栈顶两元素，按位异或后将结果压回栈顶

## 对象模型指令（不特定对象）

## box typeTok
* 装箱指令，从栈顶弹出值类型，装箱后将其引用压回栈顶
* 对于不可空值类型，直接装箱压栈。对于可空类型，则检查HasValue字段，若假则直接压入一个空指针，若真则装箱压栈。
* 对于引用类型，装箱不做任何操作
* 对于泛型类型，装箱操作会根据运行时的具体类型执行上述操作中的一种
* typeTok是typedef，typeref，typespec中的一种
* 可能抛出OutOfMemoryException、TypeLoadException（typeTok找不到）

## callvirt
* 调用对象的晚绑定方法，格式与call相同，具体方法与运行时具体类型相关
* 第一个参数不再是this，而是obj
* method地址的运算，根据obj，该指令会向上寻找第一个具有该虚函数实现的基类并调用其函数实现
* 可能抛出MethodAccessException、MissingMethodException、NullReferenceException、SecurityException异常

## castclass typeTok
* 强制类型转换，值类型转引用类型相当于装箱
* 可能抛出InvalidCastException、TypeLoadException

## cpobj typeTok
* 对象拷贝指令
* 从栈中依次弹出src和dest，类型为typeTok，若src和dest位置的类型不匹配，结果是不可知的
* 可能抛出NullReferenceException、TypeLoadException

## initobj typeTok
* 对象初始化指令
* 从栈中弹出dest并对其地址所指向的对象进行初始化
* 值类型初始化为0或null，引用类型初始化为null，相当于运行stind.ref+ldnull指令。
* 与newobj不同的是，initobj不调用构造函数，但initobj后对象实例已经准备好进行构造函数调用。

## isinst typeTok
* 判断某对象是否为typeTok类型
* 从栈顶弹出obj，并检查其是否为typeTok类型，并将结果压回栈中

## ldelem & ldelem.<type> & ldelem.i1 & ldelem.i2 & ldelem.i4 & ldelem.i8 & ldelem.u1 & ldelem.u2 & ldelem.u4 & ldelem.u8 & ldelem.r4 & ldelem.r8 & ldelem.i & ldelem.ref
* 从数组中载入一个元素
* 从栈顶弹出index和array，并将array[index]压回栈顶
* 不带类型的载入使用array类型加载value，带类型的载入则具有强制转换
* 可能抛出IndexOutOfRangeException、NullReferenceException

## ldelema
* ldelem的地址载入版本，载入实例对象的指针地址

## ldfld field & ldflda field
* 载入对象的某字段，从栈顶弹出obj，并载入field字段，将其值压栈
* 可能抛出FieldException、MissingFieldException、NullReferenceException

## ldlen
* 载入数组的长度，从栈顶取出数组地址，并将其长度压回栈顶

## ldobj typeTok
* 将src所指向地址内的值拷贝到栈顶
* 从栈中弹出src，并将其值val拷贝并压入栈顶

## ldsfld field & ldsflda field
* 加载静态字段到栈顶，直接加载field指示的静态字段并压入栈顶

## ldstr
* 加载一个字段串常量引用到栈顶
* CLI中，引用同样字符串常量的变量，其引用地址是相同的，即==判断将返回true
* 字符串常量机制由“string interning”进行保障

## ldtoken token
* 载入句柄到栈顶
* 直接加载token指示的句柄到栈顶，其类型为RuntimeHandle

## ldvirtftn method
* 加载虚函数指针到栈顶
* 较复杂，先不了解

## mkrefany class
* 从栈顶加载ptr指针，并将其指向的对象使用class进行token解析，转换为typeref并压回栈顶

## newarr etype
* 新建一个从0开始的一维数组
* 从栈顶弹出numElems，即元素数量，分配内存并将array的地址压回栈顶
* 可能抛出OutOfMemoryException、OverflowException

## newobj ctor
* 新建一个对象实例
* 从栈顶弹出N个构造函数所需的参数，调用ctor构造函数并将obj引用指针压回栈顶

## refanytype
* 从栈顶弹出TypedRef，并将其type压回栈顶

## refanyval type
* 从栈顶弹出TypedRef，并将其地址压回栈顶

## rethrow
* 抛出当前的异常

## sizeof typeTok
* 将typeTok的大小u4压入栈顶

## stelem typeTok & stelem.i1 & stelem.i2 & stelem.i4 & stelem.i8 & stelem.r4 & stelem.r8 & stelem.i stelem.ref
* 依次从栈顶弹出value、index、array
* 在array的index位置存入value

## stfld field
* 将值存入对象的某字段中
* 从栈顶弹出value和obj，并将obj的field字段存为value

## stobj typeTok
* 将对象存入某地址
* 从栈顶弹出src和dest，并将src中的值存入dest中

## stsfld field
* 将值存入静态字段
* 从栈顶弹出val，并将其存入field指示的静态字段中

## throw
* 抛出异常
* 从栈顶取出obj，并将其抛出

## unbox valuetype
* 拆箱操作
* 拆箱并不会导致数据拷贝，而是将值指针地址存入栈中，所以拆箱比装箱速度快很多
* 可能抛出InvalidCastException、NullReferenceException、TypeLoadException

## unbox.any typeTok
* 对于值类型，相当于unbox + ldobj
* 对于引用类型，相当于unbox + castclass

