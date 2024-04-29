# JDK

## OOP

POP 更注重任务执行的顺序和步骤, OOP 更注重任务的参与者, 面对复杂问题时, 先去拆分任务, 将任务对应到每个对象上, 最终由多个对象合作完成任务

POP 更高效, OOP 更易于复用, 易于扩展

OOP 的封装隐藏了内部实现细节, 外部调用封装好的方法, 不需要考虑内部实现细节, 如 ORM 框架, 引入 MyBatis 后, 直接帮我们省去了连接的创建和管理, 如果使用 POP, 就需要去考虑这些细节

OOP 的继承保证了代码的复用问题

OOP 的多态有助于扩充方法的实现效果, 同一个父类引用, 只需要更改子类实现, 即可实现不同的效果, 并且引入接口, 设计模式, 更易于项目的扩展

## HashMap

HashMap 采用 Array + LinkedList + RedBlackTree 的结构, 无法保证线程安全.

HashMap 是懒惰加载, 在创建对象时并没有初始化数组, 在无参的构造函数中, 设置了默认的 Load Factor 为 0.75.

HashMap Infinite Loop, [Explain](https://www.bilibili.com/video/BV1yT411H7YK?p=85&spm_id_from=pageDriver&vd_source=2b0f5d4521fd544614edfc30d4ab38e1)

- JDK7 的 HashMap 采用头插法, 扩容后链表顺序会颠倒, 在并发场景下, 数组进行扩容数据迁移时, 有可能导致死循环.
- JDK8 的 HashMap 采用尾插法, 扩容后链表顺序不\会颠倒, 避免了死循环.

JDK8 的 Hash Algo 通过 hash ^ (hash >>> 16) 是在 hash 的基础上增加了高位对低位的影响, 这种混淆和干扰能够更均匀地分布 Hash, 降低碰撞的概率.

HashMap 采用 RedBlackTree 不仅可以提高查询效率, 还可以防止 DDos. 

- RedBlackTree 是一种自平衡的二叉搜索树, 与普通的平衡二叉树相比, 主要区别在于其引入了颜色标记和一些额外的规则, 以保持树的平衡性.
- 颜色标记提供了额外信息, 规则简化了平衡维护, 通过颜色变化和旋转等操作可以相对简单地调整树的结构, 而无需进行复杂的平衡计算, 使得平衡调整可以在常数时间内完成, 而不会导致整个树的结构变化.

当 LinkedList 的元素个数 > 8 && Array 的容量 > 64 时, 就会将 LinkedList 进化成 RedBlackTree. 当 LinkedList 的元素个数 < 6 是, 就会将 RedBlackTree 退化成 LinkedList.

## ConcurrentHashMap

JDK7 采用 Array + LinkedList, 通过 Segment + ReentrantLock 保证线程安全.

- Segment 本质上就是一个独立的 ﻿HashTable, 也就是说 ﻿ConcurrentHashMap 是由多个 ﻿Segment 组成的.
- ﻿Segment 可以独立锁定, 这允许多个线程可以同时访问 ﻿ConcurrentHashMap 的不同部分, 从而实现了比完全锁定整个 ﻿Map 更高级别的并发性.
- Segment 包含一个 ReentrantLock, 当一个线程需要访问某个 Segment 中的元素时, 需要先获得这个 Segment 对应的 ReentrantLock, 实现了对数据的分段加锁, 这种策略是 ﻿ConcurrentHashMap 能提供比 ﻿HashTable 更好的并发性能的关键.

JDK8 采用 Array + LinkedList + RedBlackTree, 通过 CAS + synchronized 保证线程安全, 放弃了 Segment 的臃肿设计, 通过 synchronized 锁定每个索引上的第一个结点, 如果当前索引有结点了, 就需要通过 CAS 去争抢锁添加结点, 如果当前索引没有结点, 就不受 synchronized 影响, 效率还是很高的.

JDK8 引入了多线程并发扩容, 对原始数组进行分片, 每个线程负责一个分片的数据迁移, 从而提升了扩容的效率

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202403022009974.png)

ConcurrentHashMap 可以通过 size() 获取元素格式, 这要求在保证 Atomicity 的前提下, 去维护一个整形的递增, 这个效率是非常低的, ConcurrentHashMap 针对此做了特殊优化.

ConcurrentHashMap 通过维护 CounterCell[] 来实现 size(), CounterCell 的 volatile long value 记录了 ConcurrentHashMap 的元素个数, 不同的线程操作不同的 ConcurrentCell, 不存在并发问题. 最终调用 size() 时, 就遍历 CounterCell[] 进行求和, 类似于 LongAdder.

## IO

Java NIO (New IO) 提供了 Channel 和 Selector, 通过他们配置可以实现 Multiplexing IO 的效果.

- Channel 相当于 IO Stream, 但支持非阻塞数据流的读写
- Selector 相当于 OS Level 的 Multiplexer, 可以注册多个 Channel, 并通过轮询方式检查哪个 Channel 的 IO 事件已经就绪, 从而实现高效处理多个连接

## Serialization

Serialization 是将 object 转成 binary byte stream, 存储 data 和 data type

Deserialization 是将 binary byte stream 转成 object, 恢复 data 和 data type

Serilization 会对 class 的所有 member 进行 Serialization, 除了 static member 和 transient member

Serializable 可以被继承, Integer 继承 Number, Number 继承 Serializable, 则 Integer 也可以进行 Serializatioin

Serialization file

```
aced 0005 7715 6400 0000 c801 004d 000b
6865 6c6c 6f20 776f 726c 6473 7200 0141
6090 9788 d5e1 af30 0200 0078 70
```

class 实现 Serializable 表示该 class 的 object 需要进行序列化, 这仅仅是一个标识作用, 明确指定哪些 object 需要进行序列化, 防止不需要进行序列化的 object 也被序列化了

```java
public class User implements Serializable {}
```

serialVersionUID 标识了 class 的 version, 每次修改完 class, 就会自动生成不同的 serialVersionUID, 进行 Deserialization 时, 会去比较 serialVersionUID 是否相同, 如果我们在 Deserialization 之前就修改了 class, 则会报错

手动固定 serialVersionUID 就不会有这样的问题了

```java
public class User implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;
}
```

# JVM

## Class Lifecycle

Class Lifecycle 是指类从被加载到虚拟机中开始, 直到卸载出虚拟机为止的整个过程

- Class Loading: 使用 new 关键字实例化对象, 访问静态字段, 调用静态方法等操作都会触发 Class Loading, 在这个阶段, JVM 会将 class 文件读入内存, 并为之创建一个 java.lang.Class 对象, 不仅是应用程序直接引用到的类需要加载, 被这些类引用的类也会被递归加载
- Class Using: 类的正常使用阶段, 可以创建实例, 调用方法, 在此阶段, 类完全加载完成, 可以自由地被应用程序使用
- Class Unloading: 类卸载的情况比较少见, 只有当该类的 ClassLoader 实例被垃圾回收时, 这个类才会被卸载, 这通常意味着没有任何活跃的引用指向该 ClassLoader 实例和它加载的类

## Class Loading

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241741801.png)

Loading

- 由专门的 Class Loader 通过 Full Class Name 找到 Class File, 获取 Byte Stream, 加载数据到 Memory 中
- 在 Heap 中生成 Class Obj (通过 Reflect 获取的 Class Obj)
- 在 Method Area 中生成 Klass 对象, 本质上是二进制数据, 表示 Class Info

Linking, 分为 Verify, Prepare 和 Resolve 三个步骤

- Verify, 对 File Format, Metadata, Byte Code, Symbolic Ref 进行 Verification
- Prepare, 为 Static Member 和 Constant 分配 Memory, 进行 Default Initialization
    - Basic Type 的 Final Static Field 在 Prepare 进行 Specified Initialization
    - Ref Type 的 Final Static Field 在 Initialization 进行 Specified Initialization
- Resolve, 将 Class Constant Pool 中 Class, Interface, Field, Method 的 Symbolic Ref 转换成 Direct Ref

Initialization

- 执行 clinit(), 对 Static Member 进行 Specified Initialization, 优先初始化 Par Class
  - Compiler 收集 Static Field 的 Assignment Statement 和 Static Code Block 到 clinit()
  - Compiler 会同步 clinit(), 保证 Memory 中只有一个 Class Object
  - JVM 会对 clinit() 加锁, 保证只被一个 Thread 执行一次
  - Active Loading 会 Initialization, Passive Loading 不会出发 Initialization
- 执行 Static Code Block, 优先执行 Par Class 的 Static Code Block

## Program Counter Register

Program Counter Register 用于存储下一条 instruction 在 JVM 中的 adress, 本质上就是 Byte Code 的行号, CPU 会在不同 thread 之间切换, Execute Engine 就需要在 Program Counter Register 中找到下一条 instruction, 如果执行的是 native method, 则找不着

每一个 thread 都有一个 Program Counter Register 记录该 thread 需要执行的 instruction, 防止被其他 thread 覆盖影响

Program Counter Register 占用非常小, 可以忽略不计, 是运行速度最快的存储区域

Program Control (eg: loop, branch) 都是由 Program Counter Register 完成的

Program Counter Register 没有 GC, 也是 JVM 中唯一没有 OutOfMemoryError 的部分

## Virtual Machine Stack

Virtual Machine Stack 结构简单, 没有 GC, 可以快速执行, 速度仅次于 Program Counter Register

Virtual Machine Stack 存储了多个 Stack Frame, 1 个 Stack Frame 对应 1 个 method, 执行 method 对应 Stack Frame 进入 stack, 结束 method 对应 Stack Frame 离开 stack

Virtual Machine Stack 最上层的 Stack Frame 就是 Current Stack Frame 对应 Current Method 和 Current Class

一个 thread 中, 只有 Current Stack Frame 在执行, Current Stack Frame 执行结束, 返回 result 给下一个 Stack Frame, 离开 stack, 由下一个 Stack Frame 成为 Current Stack Frame

Stack Frame 正常执行完和遇到 exception 都会被弹出 stack

other thread 无法访问 current thread 的 Stack Frame, 只有 Stack Frame 执行完毕后, 会通知结果给 other thread

一个 Stack Frame 中存储了 Local Variable Table, Operand Stack, Dynamic Linking, Return Address 和 Additional Info

## Static Linking

Static Linking (Early Binding) 在 Compile Stage 可以确定 Symbolic Reference 对应的 Direct Reference, 不具备 Polymorphism 的 class 就可以进行 Static Linking, 不能被 override 的 method 也可以进行 Static Linking (eg: Static Method, Private Method, Final Method, Constructor) 

Dynamic Linking (Late Binding) 在 Compile Stage 无法确定 Symbolic Reference 对应的 Direct Reference, 需要在 Runtime Stage 的 Class Loading 的 Linking 的 Resolve 中确定

Static Linking 相比 Dynamic Linking 具有更快的启动时间和执行速度, 生成的可执行文件独立于外部环境, 不再依赖于外部的库文件, 但是可重用性较差, 资源占用也较多

## Dynamic Linking

Dynamic Linking 会发生在两个阶段. 如果能在 Class Loading 期间确定的 Reference, 则会在 Resolve 这一步将 Symbolic Reference 转成 Direct Reference. 由于 Dynamic Binding 的存在, 很多引用需要在 Runtime 时确定, 将 Symbolic Reference 转成 Direct Reference

Symbolic Reference 是一个标识, 用于描述所引用的目标的各种符号信息, 包括类和接口的全限定名, 字段的名称和描述符, 方法的名称和描述符等, 存储在 class file 的 Class Constant Pool

Direct Reference 指向 Heap 中的 instance 的地址, 存储在 JVM 的 Runtime Constant Pool

## Class Constant Pool

Class Constant Pool 存储在 Class File 中, 包含 Literal 和 Symbolic Reference, 类似于一个 Table, 每个 Literal 和 Symbolic Reference 都标识了 Index, 可以通过 Index 访问

Literal 包含 Basic Field, String Field 和 Final Field

Symbolic Reference 本质上还是 Literal, 用作标识, Class File 占用小, 就是因为采用了 Symbolic Reference

## Runtime Constant Pool

Class File 经过 Class Loader, 会将 Class Constant Pool 中必要的信息加载到 Runtime Constant Pool 中, 包括 Literal, Dynamic Ref, Symbolic Ref (有部分 Symbolic Ref 在 Class Loading 阶段无法转成 Dynamic Ref)

Runtime Constant Pool 具有 Dynamic, 可以在 Code 中修改 Runtime Constant Pool (eg: String 的 Intern())

## String Constant Pool

SCP (String Constant Pool) 维护了一个 StringTable 通过 HashTable 实现, 存储的 String Object 都是唯一的

- JDK7 时, SCP 存储在 Method Area 中, 只有触发 FGC 才会进行清理, JDK8 后, SCP 移动到 Heap 中, GC 效率稍高了一些
- JDK8 中 StringTable 默认长度为 65536, 可以通过 `-XX:StringTableSize` 设置 HashTable 的长度
- 存储 String Object 过多, 就会导致 Hash Complict, 导致 Linked List 过长, 性能下降

通过 intern() 也可以保证存储唯一的 String Object. 调用 intern() 后会先去 SCP 中寻找该 String Object

- 如果找得到, 就会返回该 String Object 的 Reference
- 如果找不到, 就会创建该 String Object, 然后返回 Reference
- 通过 intern() 创建 String Object, 可以节省 Memory, 并且 GC 也会容易很多

new String("ab") 可以在 Compile Stage 确定, 不仅仅会在 Heap 中创建 "ab", 还会在 SCP 中创建 "ab"

```java
// s1 指向 SCP 的 "ab"
String s1 = "ab";
// s2 指向 Heap 的 "ab"
String s2 = new String("ab");
// SCP 中已经有 "ab" 了, intern() 就直接返回 "ab" 的 Ref, 所以 s3 指向 SCP 的 "ab"
String s3 = s2.intern();
System.out.println(s1 == s2); // false
System.out.println(s1 == s3); // true

// s1 指向 Heap 的 "ab"
String s1 = new String("ab");
// s1.intern() 会去返回 SCP 的 "ab", 发现没有 "ab", 则去创建了一个 "ab"
// s2 指向 SCP 的 "ab"
String s2 = s1.intern();
// s3 指向 SCP 的 "ab"
String s3 = "ab";
System.out.println(s1 == s2); // false
System.out.println(s1 == s3); // false
System.out.println(s2 == s3); // true
```

JVM 为了会进行编译优化, String s = "a" + "b" 经过编译优化, ByteCode 中会是 String s = "ab"

```java
String s1 = "a" + "b";
String s2 = "ab";
System.out.println(s1 == s2); // true
```

String 的拼接, 底层依赖的是 StringBuilder 调用 append() 追加 String Object

```java
String s = new String("a") + new String("b");

// Similar to this
String s = new StringBuilder().append("a").append("b").toString();
```

StringBuilder 拼接的字符串 "ab" 无法在 Compile Stage 确定, 所以就不会在 SCP 中创建

通过 intern() 访问, 则会在 SCP 中存储一个引用指向 Heap 中的 "ab" 中, 所以 SCP 不仅存储 String Object, 还会存储 String Object Ref

```java
// s1 指向 Heap 的 "ab"
String s1 = new StringBuilder().append("a").append("b").toString();
// s1.intern() 处理 StringBuilder 拼接的字符串, 创建一个引用指向 "ab", 非常特别
// s2 指向 Heap 的 "ab"
String s2 = s1.intern();
// s3 指向 Heap 的 "ab"
String s3 = "ab";
System.out.println(s1 == s2); // true
System.out.println(s1 == s3); // true
System.out.println(s2 == s3); // true

// new String("a") + new String("b") 本质上就是通过 StringBuilder 的 append() 拼接的, 所以下面的效果一样
String s1 = new String("a") + new String("b");
// s2 指向 Heap 的 "ab"
String s2 = s1.intern();
// s3 指向 Heap 的 "ab"
String s3 = "ab";
System.out.println(s1 == s2); // true
System.out.println(s1 == s3); // true
System.out.println(s2 == s3); // true
```

## Object Instantiation

Object Instantiation 包括 Class Loading, Memory Allocation 和 Object Initialization

1. Class Loading

- JVM 执行 new 时, 会去 Constant Pool 中寻找对应的 Symbolic Reference 标识的 Class, 如果该 Class 不存在, 则会去通过 Parent Delegation 找到对应的 ClassLoader 进行 Class Loading
- 根据 ClassLoader + Package Name + Class Name 查找到对应的 Class File 进行 Class Loading, 生成 Class, 如果没有找到 Class File, 则抛出 ClassNotFoundException

2. Memory Allocation

- 采用 Bump the Pointer 或 Free List 给 Object 分配内存
- 如果 Memory 规整 (eg: 从左往右, 依次存储使用), JVM 会通过 Bump Pointer 指向了 Memory 中空闲的区域, 插入一个 Object 到 Pointer 指向的地址后, Pointer 再移动到 Object 的后头, 指向下一个空闲的区域
- 如果 Memory 不规整, JVM 会维护一个 Free List, 记录哪些区域为空闲, 后续查询该 List 即可知道 Object 可以分配在哪了

3. Object Initialization

- 调用对象的构造函数进行初始化, 构造函数的执行过程包括初始化字段, 执行构造方法体等, 子类构造函数会调用父类构造函数, 确保所有父类和子类的构造函数都得到执行
- 对象的字段会根据其类型进行默认初始化, 保证不赋值时, 也基本可用
- 构造方法体中的代码将会执行, 完成对象的特定初始化操作
- 构造函数执行完毕后, 将对象的引用返回给调用者, 这样对象就可以在程序中被引用和操作了

Object Instantiation 需要保证线程安全, 防止多个线程并发访问和修改同一个对象的状态

- Class Loading 过程, Class Loader 通常在单线程中执行, 可以保证线程安全
- Memory Allocation 过程, 通常是线程安全的, 每个线程分配的内存地址都是独立的, JVM 通过 CAS 保证了多线程环境下的原子性分配
- Object Initialization 过程, JVM 通过 Initialization Lock 保证多个线程中只有一个线程执行类的初始化过程

## Object Structure

Object Structure 包括 Header, Instance Data 和 Padding

1. Header: 包括 Mark Word, Class Pointer 和 Array Length

- Mark Word: 存储 HashCode, GC Age (4b, 所以 GC Age 最大 15), Lock, Lock State, Thread Id, Thread Stamp
- Class Pointer: 指向 Method Area 中对应 Class 的 Klass Class Info
- Array Length: 如果 Object 是 Array, 则会记录 Array Length

2. Instance Data: 存储了对象的实际数据, 

- 如果是 Normal Obj, 则 Instance Data 包括 Cur Class 和 Par Class 的 Field
  - Par Class 的 Field 会在 Sub Class 的 Field 之前, 如果宽度相同, 则会分配在一块, 如果 CompactFields 为 true, 则 Sub Class 的 Narrow Field 则可能插入 Par 的 Field 的缝隙
- 如果是 Array Obj, 则 Instance Data 包括 Array Length 和 Array Element

3. Padding: JVM 要求 Object 的 Start Address 为 8B 的整数倍, 通过 Padding 向上对齐 (eg: 14B 向上对齐为 16B)

Heap 中是不存储 Class Info 的, 只存储 Instance Data, 通过 ClassPointer 指向当前对象对应的 Klass 查询 Class Info

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202402292302358.png)

## Object Memory Allcation

64 bit OS 中, Mark Word 占 8B, Class Pointer 占 8B, 共占 16B, Pointer Compression 会将 Class Pointer 压缩成 4B, 可以通过 `-XX:-UseCompressedOops` 禁用 Pointer Compression

直接通过 `new Object()` 创建一个空对象, 就是 MarkWord 占 8B, Class Pointer 占 4B, Padding 占 4B, 共 16B

通过 JOL 查看 Memory Allcation

```xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.17</version>
</dependency>
```

```java
public class Main {
    public static void main(String[] args) throws IOException {
        System.out.println(ClassLayout.parseInstance(new MyObject()).toPrintable());
    }
}

class MyObject { // MarkWord 8B, ClassPointer 4B
    int i; // 4B
    long l; // 8B
    boolean b; // 1B
    Object o; // 4B
} // 8 + 4 + 4 + 8 + 1 + 4 + 3 = 32B
```

```
OFF  SZ    TYPE DESCRIPTION
  0   8    (object header: mark)
  8   4    (object header: class)
 12   4    int MyObject.i
 16   8    long MyObject.l
 24   1    boolean MyObject.b
 25   3    (alignment/padding gap)   
 28   4    java.lang.Object MyObject.o
Instance size: 32 bytes
Space losses: 3 bytes internal + 0 bytes external = 3 bytes total
```

## Execution Engine

Class File 包含 Class Instruction, 只有 JVM 能识别 Class Instruction, 需要通过 Eexecution Engine 将 Class Instruction 编译成对应 CPU 的 Machine Instruction.

Execution Engine 既可以从 PC Register 中获取下一条 Instruction 的 Address, 也可以通过 Local Variable Table 的 Reference 找到 Heap 中的 Object.

Execution Engine 包含 Interpreter 和 JIT 两种 Compiler, 负责将 Class Instruction 转成 Machine Instruction.

Interpreter 是逐行编译 Class Instruction. 现在主流的 Template Interpreter 是一条 Class Instruction 关联一个 Template Function, Template Function 可以直接产出对应的 Machine Instruction, 提供性能. 

HotSpotVM 的 Template Interpreter 包含 Interpreter Template 和 Code Template. Interpreter Template 负责主要核心功能. Code Template 负责管理生成的 Machine Instruction.

JIT (Just In Time Compiler) 是 Dynamic Compile, 会将整个 Function 编译成 Machine Instruction, 保存在 Cache 中, 后续再次执行 Function 只要调用对应的一系列 Machine Instruction, 不需要像 Interpreter 一样去重新编译.

JIT 包含 C1 Compiler (Client Compiler) 和 C2 Compiler (Server Compiler, def).

- C1 适合 Client Program, 编译时的优化较浅, 编译耗时较短, 响应快, 资源占用少. 可以设置 `-client` 开启 C1. C1 主要采用方法内联, 栈上替代, 去虚拟化, 冗余消除进行优化.
- C2 适合 Server Program, 编译时的优化较深, 编译耗时较长, 执行效率更高. 可以设置 `-server` 开启 C2. C2 主要采用方法内联, 标量替换, 栈上分配, 同步消除进行优化.

HotSpotVM 为了实现 Java 的跨平台, 避免采用 Static Compilation, 通过 Interpreter 保留 JVM 的动态性. 可以设置 `-Xint` 只采用 Interpreter, 设置 `-Xcomp` 只采用 JIT, 设置 `-Xmixed` 采用 Interpreter + JIT.

HotSpotVM 刚启动时, 会由 Interpreter 先进行解释, 不需要等待全部编译完. 执行过程中, HotSpot Detection 会统计 Method 的调用次数, 达到一定阈值, 就会触发 OSR (On Stack Replacement), 调用 JIT 对一些常用的 Class Instruction 进行优化, 编译成 Machine Instruction 存储到 Cache 中. Cache 存储在 Method Area 中, 即 Native Memory.

C1 触发 OSR 的阈值默认为 1500 次, C2 触发 OSR 的阈值默认为 10000 次. 可以通过 `-XX:CompileThreashold` 设置阈值.

HotSpot Detection 统计的是一段时间内代码的热度, 减少长时间不活跃的代码的计数值, 有助于保持对热点代码的准确性, 如果超过一定的时间限度还不达不到阈值, 就会触发 Counter Decay, 减半统计的次数, 这个时间限度就是 Counter Half Life Time. 可以通过 `-XX:-UseCounterDecay` 关闭 Counter Decay. 可以通过 `-XX:CounterHalfLifeTime` 设置 Counter Half Life Time.

JDK9 引入了 AOT (Ahead Of Time Compiler), 借助 Graal Compiler, 牺牲了动态性, 在程序执行前, 将 Class Instruction 全部转成 Machine Instruction. 

## Object Reference

JVM 进行对象定位时, 有句柄引用 和 直接引用 (def) 两种方式

直接引用 是 引用 -> 对象

句柄引用 是 引用 -> 句柄 -> 对象, 在堆中维护一个句柄池, 想要定位对象, 就得先定位到句柄池里的某一个句柄, 再通过该句柄定位到具体的对象. 在 GC SweepCompact 时, 需要频繁改变对象的位置, 就只需要调整句柄的引用即可, 不需要去调整栈帧的引用

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202402292305667.png)

## Method Area

Metaspace 是 HotSpot 对 Method Area 的实现, 不存储在 JVM Memory 中, 存储在 Native Memory 中

- JDK7, Method Area 存储在 JVM Memory 的 Heap 中, 非常容易导致 OOM
  - java.lang.OutOfMemoryError: Permspace
- JDK8, Method Area 存储在 Native Memory 中, 不容易导致 OOM
  - java.lang.OutOfMemoryError: Metaspace

Method Area 的初始化大小 MetaspaceSize 为 21MB, 当占用达到 MetaspaceSize 后, 会触发 FGC, 如果清理后还是不够用, 会向上扩容, 直到 MaxMetaspaceSize, 如果清理了很多后, 会降低 MetaspaceSize

逻辑上 Runtime Constant Pool, String Constant Pool, Static Member, Class Info 存储在 Method Area 中

实际上 Hotspot 将 String Constant Pool, Static Member 存储在 Heap 中, Class Info, Runtime Constant Pool 存储在 Method Area 中

- JDK7, Static Obj 存储在 Heap 中, Static Obj Ref 存储在 Method Area 中
- JDK8, Static Obj 和 Static Obj Ref 都存储在 Heap 中
- Local Obj 存储在 Heap 中, Basic Type 和 Local Obj Ref 存储在 Stack 中
- Member Obj 和 Member Obj Ref 都存储在 Heap 中

Method Area 触发 GC 主要清理 Runtime Constant Pool 和 Class Info, 要求非常苛刻, 包括三个条件

- Class 和 Sub Class 都不存在 Instance
- 加载该 Class 的 Class Loader 已经被回收
- 该 Class 的 Class Object 没有被引用

Method Area 通过 Klass 这种数据结构表示 Class Info (Klass 和 Class 发音相同)

配置 Method Area

- `-XX:MetaspaceSize=21m`
- `-XX:MaxMetaspaceSize=100m`

## Class Constant Pool

Class Constant Pool 存储在 Class File 中, 包含 Literal 和 Symbolic Reference, 类似于一个 Table, 每个 Literal 和 Symbolic Reference 都标识了 Index, 可以通过 Index 访问

Literal 包含 Basic Field, String Field 和 Final Field

Symbolic Reference 本质上还是 Literal, 用作标识, Class File 占用小, 就是因为采用了 Symbolic Reference

## Runtime Constant Pool

Class File 经过 Class Loader, 会将 Class Constant Pool 中必要的信息加载到 Runtime Constant Pool 中, 包括 Literal, Dynamic Ref, Symbolic Ref (有部分 Symbolic Ref 在 Class Loading 阶段无法转成 Dynamic Ref)

Runtime Constant Pool 具有 Dynamic, 可以在 Code 中修改 Runtime Constant Pool (eg: String 的 Intern())

## String Constant Pool

SCP (String Constant Pool) 维护了一个 StringTable 通过 HashTable 实现, 存储的 String Object 都是唯一的

- JDK7 时, SCP 存储在 Method Area 中, 只有触发 FGC 才会进行清理, JDK8 后, SCP 移动到 Heap 中, GC 效率稍高了一些
- JDK8 中 StringTable 默认长度为 65536, 可以通过 `-XX:StringTableSize` 设置 HashTable 的长度
- 存储 String Object 过多, 就会导致 Hash Complict, 导致 Linked List 过长, 性能下降

通过 intern() 也可以保证存储唯一的 String Object. 调用 intern() 后会先去 SCP 中寻找该 String Object

- 如果找得到, 就会返回该 String Object 的 Reference
- 如果找不到, 就会创建该 String Object, 然后返回 Reference
- 通过 intern() 创建 String Object, 可以节省 Memory, 并且 GC 也会容易很多

new String("ab") 可以在 Compile Stage 确定, 不仅仅会在 Heap 中创建 "ab", 还会在 SCP 中创建 "ab"

## Direct Memory

Direct Memory 不存储在 JVM Memory 中, 存储在 Native Memory 中, 通过 NIO 直接操作 Native Memory

BIO 时, 需要先访问 User Mode 的 Buffer, 复制数据到 Kernel Mode 的 Buffer 中, 再访问 Kernel Mode 的 Buffer, 才能再操作 Disk, 重复存储了两份数据, 效率低

NIO 时, OS 会直接划分一块 Buffer 供 Java Program 访问, OS 会同步 Buffer 中的数据到 Disk 中, 只存储一份数据, 少了一次复制操作, 适合大文件存储

# JUC

## Lock

尽量使用 object lock, 而不是 class lock

添加了 synchronized 的 method 会添加一个 ACC_SYNCHRONIZED flag

Pessimistic Locking 先锁定资源, 再操作资源, 适合写操作多的场景

Optimistic Locking 先判断资源是否被更新过, 再操作资源, 如果没有被更新, 就直接操作资源, 如果被更新过, 就采取相应策略 (eg: 放弃修改, 重试强锁), 适合读操作的场景, 一般通过 OCC 或 CAS 判断资源是否被操作过

OCC (Optimistic Concurrency Control) 是一种乐观锁的并发控制策略, 它假设事务冲突的概率较低, 因此允许多个事务同时进行, 并在提交时检查版本是否有冲突 (eg: AtomicInteger)

OCC 和 CAS 都是基于乐观锁, 都是比较版本号解决并发冲突, OCC 更侧重软件实现, CAS 更侧重硬件实现

## Dead Lock

一个线程需要持有多把锁时, 并且锁之间存在嵌套关系, 就非常容易导致 Dead Lock, 尽量在设计程序时, 避免一个线程去争抢多把 Lock, 实在没办法, 也要让多把锁的逻辑放在一个层级, 不要去嵌套

死锁的四个必要条件

- 互斥 (Mutual Exclusion): 共享资源 X 和 Y 只能同时被一个线程所持有
- 占有等待 (Hold and Wait): A 持有 X, A 在等待 Y 的过程中不释放 X
- 不可抢占 (No Preemption): A 持有 X, B 持有 Y, A 无法强行抢占 B 的 Y
- 循环等待 (Circular Wait): A 等待 B 释放 Y, B 等待 A 释放 X

只要打破四个必要条件的任意一个即可解决死锁问题

- 互斥: 互斥是实现同步的根本原理, 无法被打破
- 占有等待: A 一次性获取所需的 X 和 Y, 避免等待, 或者引入锁超时机制避免永远等待
- 不可抢占: 使用可中断的锁, A 想要的 Y 被 B 抢走了, A 就主动在程序中释放 B 的 Y, 然后自己再获取 Y
- 循环等待: 指定共享资源的获取顺序, 想要获取 Y 一定要先获取 X

检测程序中是否存在死锁是一个相对复杂的任务, 因为死锁通常发生在运行时, 并且需要在系统中的某个时刻进行检测

- jstack 是 JVM 提供的一个 Command Tool, 可以打印出给定 Java 进程中所有线程的堆栈跟踪, 在一个运行中的 Java 程序中, 可以使用 jstack 来获取线程堆栈信息, 以分析是否存在死锁
- VisualVM 是 JVM 提供的一个管理和分析 Java 应用程序的图形化工具, 可以通过 VisualGC, Thread Dump Analyzer 来分析线程和内存信息, 进而检测死锁
- Deadlock Detector 是一些第三方的死锁检测工具

## Monitor

Monitor 是 JVM 的 Lock, 是实现 synchronized 线程同步的基础, 每个 Object 都会关联一个 Monitor Obj, 所有的 Thread 都是去争抢 Monitor Obj.

Monitor 的 Field

- `_owner` 记录持有当有当前 Mointor Obj 的 Thread Id
- `_count` 标识了 Monitor object 是否被锁, 每次添加 Lock, 就 +1, 每次释放 Lock 就 -1
- `_recursions` 记录 Reentrant 的次数, 每次进入一层, 就 +1, 每次出来一层, 就 -1
- `_EntryList` 存储了 Blocked Thread, 当一个线程尝试去获取一个已经被占有的 Monitor Obj 时, 就会进入 `_EntryList`
- `_WaitSet` 存储了 Waited Thread, 当线程调用了 wait(), 当前线程就会释放锁, 并进入 `_WaitSet`

每个 Mointor 都有 Entry Lock 和 WaitSet Lock

- 当一个线程想要获取 Mointor 时, 就会去尝试获取 Entry Lock
- 当一个线程进入 WaitSet 时, 就会尝试去获取 WaitSet Lock

## ThreadLocal

ThreadLocal 提供了一系列用于访问和操作线程局部变量的方法, 使得每个线程都可以拥有自己的变量副本, 而不需要考虑线程安全性, 不同的方法之间就不需要通过全局变量实现通信了

```java
ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);;

for (int i = 0; i < 10; i++) {
    new Thread(() -> {
        try {
            for (int j = 0; j < 10; j++) {
                threadLocal.set(threadLocal.get() + 1);
            }
            System.out.println(Thread.currentThread().getName() + " " + threadLocal.get());
        } finally {
            threadLocal.remove();
        }
    }).start();
}
```

## ThreadLocalMap

每一个 Thread 都有一个关联的 ThreadLocal.ThreadLocalMap 用于存储线程局部变量, 每次第一次通过 get(), set() 去操作一个 ThreadLocal Obj 时, 都会去创建一个 ThreadLocal Obj 的副本 Entry 存储到当前 Thread 的 ThreadLocalMap Obj 中, 后续通过 get(), set() 或 remove() 去操作 ThreadLocal Obj 都是在操作 Entry 副本, 不影响其他线程, 保证了线程安全

```java
public class Thread implements Runnable {
    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

```java
public class ThreadLocal<T> {
    static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {}
    }
}
```

JDK7 中, ThreadLocal 维护 ThreadLocal.ThreadLocalMap Obj, 如果 Thread Obj 销毁后, ThreadLocal Obj 没有销毁, 内部的 ThreadLocalMap Obj 就不会销毁, 导致 Memory Leak

JDK8 中, Thread 维护 ThreadLocal.ThreadLocalMap Obj, 当 Thread Obj 销毁后, 就会断开对 ThreadLocalMap Obj 的引用, 就会自然的回收 ThreadLocalMap Obj, 避免了 Memory Leak

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202403141417049.png)

## ThreadLocal Memory Leak

ThreadLocalMap 的 Entry 继承了 WeakReference, 所有的 Key 都是通过 Weak Ref 指向 ThreadLocal Obj, 如果发生 GC, Key 就会断开对 Weak Ref, 此时 Key 为 Null, Val 为 ThreadLocal Obj 的副本

ThreadPool 中的线程不会销毁, 所以就不会断开对 ThreadLocalMap Obj 的 Strong Ref, 就无法销毁 ThreadLoacalMap Obj, 所以存储的 Entry 就不会销毁, 这会导致 Entry 的 Val 一直存储着之前 ThreadLocal Obj 的副本, 导致 Memory Leak

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202403141345757.png)

ThreadLocal 的 set(), get(), remove() 底层都会调用 expungeStaleEntry() 销毁 key 为 null 的 Stale Entry

```java
private int expungeStaleEntry(int staleSlot) {
    // ...
    if (k == null) {
        e.value = null;
        tab[i] = null;
        size--;
    }
    // ...
}
```

为了避免这个 Memory Leak, 使用 ThreadLocal 时, 一定要调用 remove() 销毁 Stale Entry

```java
try {
    System.out.println(threadLocal.get());
} finally {
    threadLocal.remove();
}
```

ThreadLocal 能实现 Data Isolation, 重点在于 ThreadLocalMap, 所以 ThreadLocal 可以通过 statci 修饰, 只分配一块空间即可

```java
static ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);;
```

## ThreadLocal Dirty Read

这里 T1 和 T2 是两个不同的线程, 第一次调用 set(), get() 或 remove() 时, 都会去创建一个 ThreadLocalMap Obj, 然后存储 userId 的副本, 所以相互之间不会有任何影响

```java
private static final ThreadLocal<Integer> userId = ThreadLocal.withInitial(() -> null);

public static void main(String[] args) {
    new Thread(() -> {
        userId.set(1);
        System.out.println(Thread.currentThread().getName() + " get " + userId.get()); // t1 get 1
    }, "t1").start();
    try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + " get " + userId.get()); // t2 get null
    }, "t2").start();
}
```

因为 ThreadPool 中的线程不会销毁, 所以同一个线程就会一直反复使用同一个 ThreadLocalMap Obj, 导致了第二次执行任务时的脏读

```java
private static final ThreadLocal<Integer> userId = ThreadLocal.withInitial(() -> null);

public static void main(String[] args) {
    ExecutorService threadPool = Executors.newFixedThreadPool(1);
    threadPool.submit(() -> {
        userId.set(1);
        System.out.println(Thread.currentThread().getName() + " get " + userId.get()); // pool-1-thread-1 get 1
    });
    try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
    threadPool.submit(() -> {
        System.out.println(Thread.currentThread().getName() + " get " + userId.get()); // pool-1-thread-1 get 1
    });
    threadPool.shutdown();
}
```

这里通过 remove() 去清除了 ThreadLocalMap 中该 ThreadLocal Obj 的副本, 第二次执行任务时, 在 ThreadLocalMap 中找不到该 ThreadLocal Obj 的副本, 就会去创建一个 ThreadLocal Obj 的副本, 从而解决了 Dirty Read 问题

```java
private static final ThreadLocal<Integer> userId = ThreadLocal.withInitial(() -> null);

public static void main(String[] args) {
    ExecutorService threadPool = Executors.newFixedThreadPool(1);
    threadPool.submit(() -> {
        userId.set(1);
        System.out.println(Thread.currentThread().getName() + " get " + userId.get()); // pool-1-thread-1 get 1
        userId.remove();
    });
    try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
    threadPool.submit(() -> {
        System.out.println(Thread.currentThread().getName() + " get " + userId.get()); // pool-1-thread-1 get 1
    });
    threadPool.shutdown();
}
```

## InheritableThreadLocal

InheritableThreadLocal 是 ThreadLocal 的一个扩展, 它不仅提供了线程局部变量, 而且还能将父线程的局部变量值传递给子线程, 这意味着当一个线程创建一个新的线程时, InheritableThreadLocal 可以将父线程中的局部变量的值传递给子线程的局部变量

ThreadLocal 仅限于当前线程, 而 InheritableThreadLocal 允许父线程向子线程传递变量

```java
ThreadLocal<Integer> userId = new ThreadLocal<>();
userId.set(1);
new Thread(() -> {
    System.out.println(userId.get()); // null
}).start();
```

```java
InheritableThreadLocal<Integer> userId = new InheritableThreadLocal<>();
userId.set(1);
new Thread(() -> {
    System.out.println(userId.get()); // 1
}).start();
```

使用 InheritableThreadLocal 在父线程和子线程之间共享用户会话信息

```java
private static final InheritableThreadLocal<String> sessionInfo = new InheritableThreadLocal<>();

public static void main(String[] args) {
    sessionInfo.set("UserSessionID: 123456");

    System.out.println("Par Thread: " + sessionInfo.get());

    new Thread(() -> {
        System.out.println("Sub Thread: " + sessionInfo.get());
    }).start();

    sessionInfo.remove();
}
```

Thread 底层维护了一个 `ThreadLocal.ThreadLocalMap inheritableThreadLocals`, Thread 的 init() 中进行线程的初始化时, 会根据 `boolean inheritThreadLocals` 判断是否需要处理 InheritableThreadLocal, 然后根据父线程的 InheritableThreadLocal 进行配置

```java
public class Thread implements Runnable {
    ThreadLocal.ThreadLocalMap threadLocals = null;
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
}
```

```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    // 这里的 currentThread() 是父线程 (当前执行 init() 的线程), this 是子线程 (正在被创建的线程)
    Thread parent = currentThread();
    
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);

    // ...
}
```

通过 get(), set(), remove() 操作的 ThreadLocal 时都会去调用 getMap() 获取 ThreadLocalMap, 从而来操作 Entry 副本

InheritableThreadLocal 继承 ThreadLocal, 也重写了 ThreadLocal 的 getMap(), 所以调用的其实是 InheritableThreadLocal 的 getMap(), 返回的是 inheritableThreadLocals 而不是 threadLocals 了

```java
public class ThreadLocal<T> {
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        // Do something with the map
    }
}
```

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }
}
```

## ThreadPool

当一个新任务提交给线程池时, 线程池会判断其中的工作线程数量, 如果当前的工作线程数量小于核心线程数, 线程池会创建一个新的工作线程来执行这个任务, 如果大于或等于核心线程数, 线程池则不会立即创建新的线程

如果线程池中的工作线程数目达到了核心线程数, 新的任务就会被存入到任务队列, 等待被执行, 当任务队列已满, 且当前线程数小于最大线程数时, 线程池会创建新的工作线程来执行任务

当任务队列已满, 并且当前线程数达到了最大线程数, 那么线程池会根据其 RejectedExecutionHandler 策略来处理这个任务

工作线程从任务队列中取出任务执行, 执行完毕后, 继续从队列中取出下一个任务执行, 直到队列为空, 如果设置了 allowCoreThreadTimeOut, 那么核心线程在等待时间超过 keepAliveTime 之后也会被回收

## Lock Escalation

Lock Escalation 是指 synchronized 实现的 Lock, 随着竞争的情况根据 Non Lock -> Biased Lock -> Light Lock -> Heavy Lock 的顺序逐渐升级, 并且是不可逆的过程

Non Lock -> Biased Lock, 当一个线程第一次访问同步块时, 会在对象头和栈帧中记录偏向的锁的 Thread ID, 之后该线程再次进入和退出同步块时, 不需要进行 CAS 操作来加锁和解锁

Biased Lock -> Light Lock, 当第二个线程尝试访问同步块时, 偏向锁就需要撤销, 撤销过程需要等待全局安全点 (在这个点上没有正在执行的字节码), 撤销后, 偏向锁就升级为轻量级锁

Light Lock -> Heavy Lock, 如果轻量级锁的竞争激烈, 即有多个线程同时申请轻量级锁, 那么轻量级锁就会膨胀为重量级锁

## Biased Lock

Current Thread 持有 Biased Lock 后, 如果后续没有 Other Thread 访问, 则 Current Thread 不会触发 Sync, 不需要再获取 Lock 了, 即一个 Thread 吃到撑, 适合经常单个 Thread 操作的场景, 不需要添加 Lock, 性能非常强

Thread 获取 Lock 后通过 CAS 修改 Lock 的 MarkWord 的 Lock Identify 为 101, 即标识为 Biased Lock, 同时在 Monitor 的 Owner Field 中存储 Cur Thread 的 Id. 下一次有 Thread 访问时, 会先判断 Thread 的 Id 和记录的 Id 相同

如果 Biased Lock 记录了 T1 的 Id, 此时 T1 访问, 就可以不触发 Sync, 也不需要进行 CAS. 如果 T2 访问, 就会发生 Conflict. T2 会通过 CAS 尝试抢 Biased Lock. 如果 T2 抢到 Biased Lock, 则会替换 Owner 为 T2 的 Id. 如果 T2 没有抢到 Biased Lock, 则会撤销 T1 的 Biased Lock, 等待 T1 到 Safe Region 后, 发动 Stop The World !!! 升级为 Light Lock !!! 此时 Thread A, 原先持有的 Biased Lock 会替换为 Light Lock, 继续执行完后, 释放 Light Lock, 两个 Thread 公平竞争 Light Lock

Biased Lock 的 Pointer 和 Epoch 会覆盖 Non Lock 的 HashCode, 所以 Biased Lock 无法与 HashCode 共存

- 如果获取 HashCode 后再开启 Sync, 就会直接忽略 Biased Lock, 直接升级为 Light Lock
- 如果使用 Biased Lock 的过程中, 试图获取 HashCode, 就会撤销 Biased Lock, 直接膨胀为 Heavy Lock

由于 Biased Lock 维护成本太高, JDK15 后逐渐废弃了, 默认禁用了 Biased Lock, 可以通过 `-XX:+UseBiasedLocking` 手动开启

Biased Lock 默认在程序启动后 4s 开启, 可以通过 `-XX:+BiasedLockingStartupDelay=0` 手动调整

Test Biased Lock

```java
Object o = new Object();
new Thread(() -> {
    synchronized (o) {
        System.out.println("hello world");
    }
}).start();
System.out.println(ClassLayout.parseInstance(o).toPrintable());
```

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241746693.png)

## Light Lock

Light Lock 本质就是 CAS. 多个 Thread 交替抢 Lock, 执行 Sync. 不需要从 User Mode 切换到 Kernel Mode, 也不需要发生 Blocking. Conflict 的处理速度非常快, 适合经常多个 Thread 操作的场景, 不会造成 Thread Blocking, 执行速度也很高

JVM 为每一个 Thread 的 Stack Frame, 都开辟了 LockRecord 的空间, 称为 Displaced MarkWord, 存储 LockRecord Object

T1 通过 CAS 修改 Lock 的 MarkWord 的 Lock Identify 为 00, 修改 Lock 的 Markword 的 ptr_to_lock_record 指向 T1 的 LocalRecord Object, 复制 Lock 的 MarkWord 到 LockRecord Object 中, 并且 T1 的 LockRecord Object 也会存储了一个 Pointer 指向 Lock

T2 通过 CAS 尝试修改 ptr_to_lock_record 指向 T2 的 LockRecord Object. 如果修改成功, 则表示 T2 抢到了 Lock. 如果 T2 尝试了多次 Spining 后, 还是没修改成功, 则会升级 Light Lock 为 Heavy Lock

T1 释放 Lock 时, 会将 Displaced MarkWord 复制到 MarkWord 中. 如果复制成功, 则本次 Sync 结束. 如果复制失败, 则说明 Light Lock 升级为 Heavy Lock 了, 此时 T1 会释放 Lock, 唤醒其他 Blocing Thread, 一起竞争 Heavy Lock

如果是 Reentrant Lock 在进行重入时, 每次重入一个锁, 就需要通过 CAS 创建一个 LockRecord Object, 后续创建的 LockRecord Object 不需要再去修改 ptr_to_lock_record 了

JDK6 前, 默认 Spining 10 次, 就会进行 Lock Escalation, 可以通过 `-XX:PreBlockSpin=10` 手动设置

JDK6 后, 采用 Adaptive Spin Lock, 如果 Spinning 成功, 则会提高 PreBlockSpin, 如果 Sprinning 失败, 则会降低 PreBlockSpin

Biased Lock 是一个 Thread 抢 Lock. Light Lock 是多个 Thread 抢 Lock. Biased Lock 是每次执行完, 不会释放 Lock. Light Lock 是每次执行完, 都会释放 Lock

## Heavy Lock

Heavy Lock 通过 Monitor 实现 Sync, 但是 Monitor 依赖 OS 的 Mutex Lock, 需要从 User Mode 切换为 Kernel Mode, 频繁切换, CPU 消耗非常多, 适合 Conflict 激烈的场景, 可以保证绝对的 Thread Safty, 但性能太差, 尽量避免用于同步时间较长的地方, 当 CAS 带来的消耗太多时, 也可以考虑切换为 Heavy Lock

Heavy Lock 的 MarkWord 存储的 Pointer 指向 Heap 的 Monitor object. Monitor 存储了 Non Lock 的 HashCode, GC Age 等信息, 释放 Heavy Lock 时, 也会将这些信息写回到 MarkWord 中

JVM Instruction

- `monitorenter` 表示进入 Monitor object, 开启 Sync
- `monitorleave` 表示离开 Monitor object, 关闭 Sync

Test Heavy Lock

```java
Object o = new Object();
new Thread(() -> {
    synchronized (o) {
        System.out.println("hello world");
    }
}).start();
new Thread(() -> {
    synchronized (o) {
        System.out.println("hello world");
    }
}).start();
```

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241746694.png)

## Lock Elimination

JIT 进行 Compoile 时, 会对 lock 进行 Escape Analysis, 这里的 obj 未发生 Escape, 所以这个 lock 毫无意义 JIT 会自动帮我们去除 lock, 提升性能, 但是这种编译优化手段依旧会影响一定性能, 应当避免这样的失误

```java
public void test() {
    Object obj = new Object();
    synchronized(obj) {
        System.out.println("hello world");
    }
}
```

## Lock Coarsening

一个 Method 内, 多处将同一个 Object 作为 Lock 没有意义, JIT 会帮我们合并内容, 扩大范围

```java
// Before JIT optimization
Object o = new Object();
new Thread(() -> {
    synchronized (o) {
        System.out.println("hello world");
    }
    synchronized (o) {
        System.out.println("hello world");
    }
    synchronized (o) {
        System.out.println("hello world");
    }
}).start();

// After JIT optimization
Object o = new Object();
new Thread(() -> {
    synchronized (o) {
        System.out.println("hello world");
        System.out.println("hello world");
        System.out.println("hello world");
    }
}).start();
```

## CAS

CAS 由 JDK 提供, 通过 hardware 保证了 NoBlocking, Atomicity, 本质是 cmpxchg instruction, 执行 cmpxchg 时, 由 thread 先给 bus 加 lock, 再去执行 CAS 操作, 相比 synchronized 更高效

JUC 提供了的 AtomicInteger, AtomicLong, AtomicReference, AtomicStampReference 都是基于 CAS 实现的, 通过 compareAndSet() 去调用操作系统级别的 cmpxchg 指令

- AtomicReference 存在 ABA 问题, AtomicStampReference 不存在 ABA 问题

## LongAdder

LongAdder 专用于高并发场景下, 原子累加一个长整型变量, 与 AtomicLong 相比, AtomicLong 操作 CAS, 每次只有一个 thread 修改成功, 其他的 thread 一直在 Spining, 导致 CPU 消耗过多

在 Low Concurrency 下, LongAdder 只操作 base, 效果和 AtomicLong 没有区别

在 High Concurrency 下, LongAdder 通过 add() 判断是否需要调用 longAcumulate(), 将单个的变量分解成多个独立的单元 Cell, 每个单元都独自维护一个独立的计数值, 首次会新建 2 个 Cell, 效果和 base 相同, 帮助 base 分散压力, 通过 Hash Algo 保证分布均匀, 当 Cell 不够用时, 会每扩容 2 个 Cell, 全部计算完后调用 sum() 叠加 base 和 Cell 得到结果

LongAdder 不保证 Strong Consistency, 有可能在得到 sum 后, 又有 thread 修改了 Cell 导致 Inconsistency

LongAdder, LongAccumulator, DoubleAdder, DoubleAccumulator 低层原理一致, 使用了一种类似于分段锁的机制

## Spin Lock

CAS 通过 Spin Lock 实现 Atomicity 不需要 wait, thread 获取 memoryValue, 记录为 exptecedValue, 当 thread 修改完, 需要同步 newValue 到内存时, 先比较 expectedValue 和 memoryValue 是否相同, 如果不同, 则说明在此期间有其他线程修改过了, 本次运算作废, 重新获取 value 进行重试

这种方式避免了线程的阻塞, 减少竞争, 减少了发生死锁的可能性, 减少了上下文切换的开销, 提高了系统的吞吐量

Spin Lock 是通过 infinite loop 实现, 资源耗费较多

## AQS

AQS (AbstractQueuedSynchronizer) 是构建 Lock 和 Synchronizer 的底层框架, 它提供了一种可重入的, 分离式的, 并发控制的机制, 基于 FIFO 队列的的资源获取方式

AQS 是一个 Abstract Class, 本身没有实现, 通过 Template Pattern 规定好模版后, 让 Sub Class 实现 (eg: CountDownLatch, CyclicBarrier, Semaphore, ReentrantLock, BlockingQueue 都是基于 AQS 实现的)

AQS 的 Exclusive Mode 是每个 Thread 独占一个 Lock (eg: ReentrantLock)

AQS 的 Shared Mode 允许多个 Thread 共享一个 Lock (eg: ReentrantReadWriteLock)

AQS 通过 CAS 实现对其内部状态的原子性修改, 以实现锁的获取和释放等操作

## CLH Queue

AQS 通过 State 表示同步状态, State 通过 Volatile 保证 Visibility. 不同的 Sub Class 有不同的实现, 如 ReentrantLock 的 State, 0 表示未占用, 1 表示被占用, 并且被同一个 Thread 重入了 1 次, 3 就表示被重入了 3 次

Thread A 试图获取 Lock 时, 会先检查 State

- 如果 State == 0 表示未占用, Thread A 就会尝试通过 CAS 将 State 改为 1, 表示 Thread A 获取了 Lock
- 如果 State != 0 表示被占用, 那么 AQS 就会将 Thread A 封装成一个 Node 存储进 CLH Queue, 通过 LockSupport 让 Thread A 进入等待状态, 当 Lock 释放后, 先进入 Queue 的 Node 就会被唤醒, 试图去争抢 Lock

CLH Queue 使用 Double LinkedList 的原因

- Double LinkedList 相比 Single LinkedList 可以访问前驱节点, 加入到 CLH Queue 的节点都需要去判断前面的节点是否存在异常, 如果存在异常会无法唤醒后续等待的节点, 如果使用 Single LinkedList 就需要从头开始遍历, 非常低效
- CLH Queue 中堵塞的节点, 下次唤醒时, 应该只有头节点去参与锁竞争, 避免同时唤醒所有的节点去竞争导致的惊群线程, 从而消耗大量资源, CLH Queue 中的节点只需要判断自己的前一个节点是否为头节点即可解决这个问题
- 有些 AQS 的实现类实现了 lockInterruptibly() 表示是可以中断的 Lock (eg: ReentrantLock), 被中断的线程在后续就不应该参与到锁的竞争中了, 通过 Double LinkedList 可以很方便的删除这个节点, 如果使用 Single LinkedList 会导致删除操作和遍历操作的竞争

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241746778.png)

## ReentrantLock

Reentrant Lock 允许 Outer Lock 和 Inner Lock 相同时, 获取 Outer Lock 后, 就可以直接进入 Inner Lock, 不需要释放后在重新获取

synchronized 和 ReentrantLock 都是 Reentrant Lock 的实现

```java
Object o = new Object();

new Thread(() -> {
    synchronized (o) {
        synchronized (o) {
            synchronized (o) {
                System.out.println(Thread.currentThread().getName());
            }
        }
    }
}).start();
```

ReentrantLock 维护了 FairLock 和 UnfairLock, 都采用是 Exclusive Mode.

Fair Lock 是多个 Thread 按照顺序获取 Lock, 分配均匀, 不存在锁饥饿, 但是性能稍差.

Unfair Fock 是多个 Thread 随意争抢 Lock, 容易造成同一个 Thread 抢走所有 Lock 的情况, 造成锁饥饿, 分配不均匀, 但是性能更强.

```java
// true is Fair Lock, false is Unfair Lock (def: false)
private ReentrantLock lock = new ReentrantLock(true);

private void sellTicket() {
    lock.lock();
    try {
        if (poll > 0) {
            System.out.println(Thread.currentThread().getName() + " " + poll--);
            Thread.sleep(100);
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }
}
```

ReentrantLock 提供了 Fair Lock 和 Unfair Lock 两种模式, 这两种模式主要的区别在于获取锁的策略不同

Fair Lock 的获取采取先来先得的原则, 也就是说, 先请求锁的线程将先获取到锁, 具体实现上, 系统维护了一个有序队列, 每当有新的线程请求锁时, 都会加入队列的末尾, 只有队列头部的线程能够获得锁, 当线程释放锁时, 它在队列中的节点将被移除, 队列的第二个节点将成为头部节点并获得锁

Unfair Lock 的获取不再遵循先来先得的原则, 当一个线程请求锁时, 如果当前无线程持有锁, 那么这个线程可以直接获取到锁, 无论是否有其他线程正在等待, 这样可能导致一些线程等待时间过长

Unfair Lock 的处理效率比 Fair Lock 要高, 线程的上下文切换和唤醒是需要耗费一定系统资源的, 不如直接让一个处于就绪的线程获取锁来的实在

```java
﻿ReentrantLock unfairLock = new ReentrantLock();
﻿ReentrantLock fairLock = new ReentrantLock(true);
```

## ReentrantReadWriteLock

ReenrantReadWriteLock 相比 ReentrantLock 不仅仅维护了 FairLock 和 UnfariLock, 还维护了一个 ReadLock 和 WriteLock, 这两个 Lock 都采用了 Shared Mode

ReentrantLock 采用 Exclusive Mode, 不适合多线程中, 读多写少的场景, 通过 ReenrantReadWriteLock 可以让多线程之间, 读读不互斥, 读写互斥, 写读互斥, 写写互斥, 可以保证读时的高性能, 和写时的安全

## JMM

JMM (Java Memory Model) 是一个 standard, 规定了 variable 在 memory 中的访问方式, 保证了 multithreaded program 在不同 CPU, 不同 OS 下, 访问 memory 时达到一致的访问效果, 实现 Atomicity, Visibility, Orderliness

thread 从 global memory 中拷贝 shared variable 到 local memory 中操作, 修改完再推送到 global memory 中, 实现修改, 不同 thread 之间, 无法访问对方的 local memory, 必须依靠 global memory 实现交互

Visibility, 一个 thread 修改了一个 variable, 通知其他 thread 有了变化, 如果其他 thread 正在操作这个 vairable, 就会去 global memory 中获取新的 variable, 解决 Dirty Read

Atomicity, 一个 thread 的操作不会被其他 thread 打断

Orderliness, 不同 CPU 和 OS 下, instruction 可以调整执行顺序来提高性能, 但是有些情况下, 不允许,调整顺序, 需要遵守 Happends-Before

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241746185.png)

JMM 为了提高性能, 减少对主内存的频繁访问, 允许修改 Local Memory 中的变量后可以等一会, 不需要立即写回到 Global Memory, 这就可能导致无法保证 Visibility

- T1 在 Local Memory 中修改完变量, 不立即写会到 Global Memory, 此时 T2 读取到的是 Local Memory 中的旧值

Local Memory 中的变量满足下面的条件, 才会写回 Global Memory

- 线程正常结束
- 线程获得锁或释放锁

## Happens-Before

Happens-Before 规范了两个操作的先后关系, 第一个操作的结果对第二个操作可见, 保证 Visibility, 交换两个操作的顺序, 如果不影响结果, 则允许, 如果影响结果, 则不运行, 保证 Orderliness

## Memory Fence

Memory Fence 实现了 Happens-Before, 是 CPU 和 Compiler 对 memory 随机访问中的一个同步点, 该同步点前的所有操作执行完毕后, 才可以执行后续操作, 保证 Visibility, 不允许将 Memory Fence 之后的操作重新排序到 Memory Fence 之前, 保证 Orderliness

LoadLoad Fence 确保在 Load 操作之前的所有 Load 操作都已经完, 确保一个线程读取共享变量的值之前, 它之前的所有读操作都已经完成

StoreStore Fence 确保在 Store 操作之前的所有 Store 操作都已经完成, 确保一个线程修改共享变量的值之前, 它之前的所有写操作都已经完成

LoadStore Fence 确保在 Load 操作之前的所有 Store 操作都已经完成, 确保一个线程读取共享变量的值之前, 其他线程对该变量的写操作已经完成

StoreLoad Fence 确保在 Store 操作之前的所有 Load 操作都已经完成, 确保一个线程修改共享变量的值之前，其他线程对该变量的读操作已经完成

Java 对 Memory Fence 的实现有 volatile, synchronized, final, concurrent ...

## volatile

volatile 是对 JMM 的一种实现, volatile 修饰的变量可以保证 Visibility 和 Orderliness, 但是无法保证 Atomicity

通过 volatile 修饰 variable, 每次修改都会立即写回到 Global Memory 中, 再通过 JMM 清空其他线程中变量的值, 让他们重新来获取最新的值, 保证了 Visibility

通过 volatile 修饰 variable, 执行 write variable 前会先执行 lock, lock 会清空其他 thread 的 local memory 中的 variable, 其他 thread, 就需要重新执行 read variable, 实现 multithreaded interaction, 所以 volatile 适合表示 flag, 适合读的场景

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241746186.png)

```java
public static volatile boolean isLoop = true;

public static void main(String[] args) throws InterruptedException, ExecutionException, TimeoutException {
    new Thread(() -> {
        while (isLoop) {}
    }).start();
    
    try { TimeUnit.MICROSECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
    
    isLoop = false;
}
```

这里不通过 volatile 修饰变量, 修改 Local Memory 中的数据后不会立即进行同步, 这里的 isLoop 就会一直是 Local Memory 中的 false

这里不仅仅是因为无法读取到更改后的数据, 更重要的是 JIT 会自动将 while(isLoop) 优化成 while(true) 避免了一次数据读取, 这也导致了 Infinte Loop, 通过 volatile 修饰变量后, 就会禁用这个 JIT Optimization

```java
public static boolean isLoop = true;

public static void main(String[] args) throws InterruptedException, ExecutionException, TimeoutException {
    new Thread(() -> {
        while (isLoop) {}
    }).start();
    
    try { TimeUnit.MICROSECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
    
    isLoop = false;
}
```

volatile 通过 Memory Fence 保证了 variable 的 Visibility, Orderliness

```java
private volatile int sharedValue;

public void writeSharedValue(int value) {
    // StoreStore Fence
    sharedValue = value;
    // StoreLoad Fence
}

public int readSharedValue() {
    return sharedValue;
    // LoadLoad Fence
    // LoadStore Fence
}
```

volatile read 后面会插入一个 LoadLoad Fence, 防止和后面的 normal read 进行 reorder, 还会再插入一个 LoadStore Fence, 防止和后面的 normal write 进行 reorder, 最终保证了 volatile read 后的 instruction 不会重排到 volatile read 前面

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202402031244938.png)

volatile write 前面插入一个 StoreStore Fence, 防止和前面的 normal read 进行 reorder, 后面插入一个 StoreLoad Fence, 防止和后面的 volatile read, volatile write 进行 reorder, 最终保证了 volatile write 前的 instruction 不会重排到 volatile write 后面

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202402031244590.png)

这里, 如果通过 volatile 修饰 x, 无法避免 x 和 y 重排序, 如果通过 volatile 修饰 y, 可以避免 x 和 y 的重排序

为了避免重排序, 要么全部加上 volatile, 要么就把 volatile read 放在最后, 把 volatile write 放在最前面, 巧妙利用 Memory Fence 避免重排序

```txt
volatile init x # Add volatile to x (not recommanded)
init y
--- Fence for x write --- # This can not prohibit reordering x below and y below
write x = 10
write y = 10
read y
read x
--- Fence for x read --- # This can not prohibit reordering x above and y above

init x
volatile init y # Add volatile to y (recommanded)
write x = 10
--- Fence for y write --- # This can prohibit reordering x above and y below
write y = 10
read y
--- Fence for y read --- # This can prohibit reordering x below and y above
read x
```

volatile 无法保证 Atomicity, 不适合参与需要依赖当前 variable 的运算

这里 count++ 包含 Data Loading, Data Calculation, Data Assignment 三个步骤, 无法保证 Atomicity

一个 thread 修改完 count 后, 将 latest data 写入 global memory 后, 清空其他 thread 的 count, 有些 thread 可能已经执行完 count++, 还没来得及写入 global memory, 就被清空了, 重新读取了 latest data, 导致刚刚的 count++ 丢失

```java
public static volatile int count = 0;

public static void main(String[] args) throws Exception {
    for (int i = 0; i < 10000; i++) {
        new Thread(() -> {
            count++;
        }).start();
    }
    
    try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
    
    System.out.println(count);
}
```

通过 volatile 保证 Visibility, 最终 Read Consistency

通过 synchronized 保证 Atomicity, 最终 Write Consistency

```java
private volatile int value;

// volatile ensure Visibility
public int getValue() {
    return value;
}

// synchronized ensure Atomicity
public synchronized void setValue(int value) {
    this.value = value;
}
```

## DCL NullPointException

DCL 是一种在单例模式中使用的延迟加载策略, 它尝试通过检查对象是否已经实例化来避免每次获取单例时都需要加锁的开销, 如果不通过 volatile 修饰 instance 会导致 NullPointException

Object Creation 包含 Memory Allocation, Object Initialization, Reference Points to Memory 三个步骤, 在多线程环境下, 由于指令重排序的存在导致了 NullPointException

- T1 进行 Object Creation 时, JVM 将 Reference Points to Memory 重排序到了 Object Initialization 之前
- T2 又来访问, 发现 Reference 不为 null, 就会直接拿走, 但是此时 T1 还没有执行 Object Initialization, T2 直接访问就会导致 NullPointException

通过 volatile 修饰 singleton, 禁止重排序, 避免 NullPointException

```java
class DoubleCheckSingleton {
    private static volatile DoubleCheckSingleton instance;
    
    private DoubleCheckSingleton() {}
    
    public static DoubleCheckSingleton getInstance() {
        if (instance == null) {
            synchronized (DoubleCheckSingleton.class) {
                if (instance == null) {
                    instance = new DoubleCheckSingleton();
                }
            }
        }
        return instance;
    }
}
```

# MySQL

## Transaction ACID

Atomicity 是指一个 TRX 是一个不可分割的工作单元, 要么全成功提交, 要么全失败回滚, 成王败寇, 没有妥协之说, 通过 Undo Log 保证.

Consistency 是指数据需要从一个合法性状态变化到另一个合法性状态, 这个合法是业务层面的合法 (eg: A 扣钱, 扣成了负数, 则不符合业务层面的要求, 即不合法). 

Isolation 是指一个 TRX 内部使用到的数据对其他 TRX 隔离, 不会受到其他 TRX 的影响, 通过 MVCC 保证.

Durability 是指一个 TRX 一旦被提交, 它对数据库中数据的改变就是永久性的, 通过 Redo Log 保障的, 先将数据库的变化信息记录到 Redo Log 中, 再对数据进行修改, 这样做, 即使数据库崩掉了, 也可以根据 Redo Log 进行恢复.

## Transaction Status

Active 是指 TRX 正在执行.

Partially Committed 是指 TRX 已经执行了最后一个提交, 操作的都是内存中的数据, 还没有进行刷盘.

Committed 是指 TRX 在 Partially Committed 后, 成功进行了刷盘.

Failed 是指 TRX 在 Active 和 Partially Committed 阶段遇到了某些错误, 而无法继续执行.

Aborted 是指 TRX 在 Failed 阶段后, 进行回滚, 恢复了到 TRX 最初的状态.

## Isolation Level

从严重程度上来讲, Dirty Write > Dirty Read > Non-Repeatable Read > Phantom Read. Dirty Write 是一定不能接受的, 而 Non-Repeatable Read 和 Phantom Read 在一定场景下, 是可以接受的.

MySQL 提供的 Isolation Level 包含 `read-uncommitted`, `read-committed`, `repeatable-read` 和 `serializable` 用于解决一定的并发问题.

- `read-uncommitted` 可以解决 Dirty Write.
- `read-committed` 可以解决 Dirty Write, Dirty Read.
- `repeatable-read` 可以解决 Dirty Write, Dirty Read, Non-Repeatable Read. MySQL 的 `repeatable-read` 可以解决一定的 Phantom Read, 并通过 Row Lock + Gap Lock + Next-Key Lock 解决 Phantom Read.
- `serializable` 可以解决 Dirty Write, Dirty Read, Non-Repeatable Read, Phantom Read.

查看当前的 Isolation Level.

```sql
select @@transaction_isolation;
```

设置 Session 的 Isolation Level 会在当前 Session 中立即生效. 设置 Global 的 Isolation Level 会在下一次 Session 中生效.

```sql
set session transaction_isolation = 'read-uncommitted';
```

## Redo Log

Redo Log 和 Undo Log 都是一种恢复操作, 他们回滚数据是逻辑层面的回滚, 而不是物理层面的回滚. 插入一条记录后, 就会记录一条对应的删除操作. 开辟了一个数据页后回滚, 是无法回滚到开辟数据页之前的, 只是通过操作相反的命令达到数据上的统一. 

```txt
start transaction;

select col; -- ''

-- Record col = '' to Undo Log
update col = 'a';
-- Record col = 'a' to Redo Log

-- Record col = 'a' to Undo Log
update col = 'b';
-- Record col = 'b' to Redo Log

-- Flush Disk
commit;
```

InnoDB 采用 WAL (Write-Ahead Logging), 先写日志, 再写硬盘, 只有日志写成功了, 才算事务提交成功. 发生宕机且数据未刷到磁盘时, 就可以根据 Redo Log 恢复数据, 保证了 ACID 的 D.

如果不采用 Redo Log, 为了保证数据安全性, 每次执行 SQL, 就需要进行 Random IO, 将硬盘的数据读取到内存中, 修改完再进行刷盘, 不仅效率低, 丢失数据的风险更高, 而且为了修改一点数据, 就将 Page 来回折腾, 非常不划算.

Redo Log 可以保障一定的安全性, 所以就没有必要实时进行内存到硬盘的刷盘操作, 可以稍微间隔长一点 (eg: 1s 刷盘一次).

Redo Log 占用非常小, 而且是通过 Sequential IO 存储到硬盘上的, 可以说成本非常低.

Redo Log 是在 Storage Engine 层面产生的, Bin Log 是 DB 层面产生的, 两者有着很大的区别 (eg: 插入 100 的过程中, Redo Log 是不断更新的, 等全部加载完, 再一次性写入到 Bin Log 中).

MySQL Server 启动后, 会立即申请一块 Redo Log Buffer, 用来存储 Redo Log, 这块空间被分成若干个连续的 Redo Log Block, 1 个 Block 占 512B.

执行一个修改操作后, 会生成一条 Redo Log 写入到 Redo Log Buffer 中, 记录的是修改后的数值, 当提交后, 就会将 Redo Log Buffer 中的数据追加写入到 OS 的 Page Cache 中, 再进行刷盘, 追加写入到硬盘的 Log File 中.

通过 `innodb_log_buffer_size` 设置 Redo Log Buffer 的大小 (def: 16M).

通过 `innodb_flush_log_at_trx_commit` 设置不同的刷新策略 (def: 1).

- `0`: 提交后, 不会进行任何操作, 等待 Server 自动进行一秒一次自动同步. 将数据存储在 Buffer 中, 依靠自动同步, 风险最高, 但是性能最强.
- `1`: 提交后, 将数据写入到 Page Cache, 再从 Page Cache 写入到硬盘. 直接写会到了硬盘中, 非常安全, 但是性能最差, 默认就是如此.
- `2`: 提交后, 将数据写入到 Page Cache. 将数据写入到 OS 到 Page Cache 中, 一般 OS 宕机的几率是非常低的, 还是蛮安全的, 性能也比较好.

## Undo Log

Undo Log 可用于回滚数据, 可用于 MVCC. 

InnoDB 默认有 2 个提供给 Undo Log 的 Table Space, 共包含 128 个 Rollback Segment, 每个 Rollback Segment 中包含 1024 个 Undo Log Segment.

1 个 Rollback Segment 可能同时服务于 n 个 TRX, 开启 1 个 TRX 后, 就会去制定 1 个 Rollback Segment, 如果 TRX 中的数据被修改了, 原始的数据就会记录到 Rollback Segment 中.

通过 `innodb_undo_directory` 设置 Rollback Segment 的存储位置 (def: ./).

通过 `innodb_undo_tablespaces` 设置 Rollback Segment 的文件数量 (def: 2).

通过 `innodb_rollback_segments` 设置 Rollback Segment 的数量 (def: 128).

InnoDB 的 Record Header 还会有一些隐藏列.

- `DB_TRX_ID`: 每个 TRX 都会自动分配一个 TRX ID.
- `DB_ROLL_PTR`: 指向 Undo Log 的 Pointer.

执行 `insert` 后, 会产生一个 Undo Log, 提交后, 立即删除.

执行 `delete` 和 `update` 后, 会产生一个 Undo Log, 提交后, 放入 Linked List 中, 提供 MVCC 使用, 等待 Purge Thread 删除. Purge Thread 在删除数据时, 只是进行逻辑删除, 将 deletemark 标记为 1, 后续采用覆盖的方式插入数据实现删除.

Undo Log 的存储是离散的, 要回收非常麻烦, 所以 TRX 提交后, 不会立即删除 Undo Log, 而会放入到一个 Linked List 中, 然后判断 Undo Log 所属 Page 的使用空间是否小于 3/4, 如果小于 3/4, 那么它就不会被回收, 其他 TRX 的 Undo Log 会继续使用当前 Page.

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241733175.png)

更新 Priamry Key 时, 会将 Old Record 的 deletemark 标识为 1, 再新建一个 New Record, 递增 Undo Log 的 no, 保证回滚时向前可以找到 Undo log.

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241733176.png)

## Bin Log

在 TRX 提交之前, 会记录 DDL 和 DML 到 Bin Log 中, 以达到重放 SQL 语句的目的, 主要用于数据库回滚、复制、数据恢复.

Bin Log 可以用于数据恢复, 如果 MySQL Server 挂掉了, 可以通过 Bin Log 查询到用户执行了哪些修改操作, 可以根据 Bin Log 来恢复数据.

Bin Log 可以用于主从复制, Log 具有延续性和时效性, 可以根据 Bin Log 同步 Master 和 Slave 之间的数据.

查看 Bin Log 状态.

```sql
show variables like '%log_bin%';

show binary logs;
```

配置 Bin Log. (file: my.cnf)

```
[mysqld]
log-bin=mysql-bin
binlog_expire_logs_seconds=600
max_binlog_size=100M
```

Bin Log 是一对二进制文件, 所以无法直接查看, 这里通过 `mysqlbinlog` 查看 Bin Log

```shell
mysqlbinlog '/var/lib/mysql/mysql-bin.000004'
```

查看 Bin Log Events.

```sql
show binlog events \G

show binlog events in 'msyql-bin.000004';

-- 从 236 行开始向后查 5 条
show binlog events in 'msyql-bin.000004' from 236 limit 5;
```

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241732538.png)

## MVCC

MVCC (Multi Version Concurrency Control) 可以在不加锁的情况下, 采用非堵塞的方式解决读写冲突.

MVCC 主要有 Undo Log 和 Page View 实现的, 通过 Undo Log 构成一个 Version Linked List, 通过 Page View 选择一个 Version 供访问.

MVCC 在 RC 和 RR 中生效, 不在 RU 中生效, 因为 RU 可以读到未提交的事务, 所以直接读取最新版本即可.

当 TRX 来读取数据时, 会生成一个 Page View, 该 Page View 包含了几个属性.

- `creator_trx_id` 记录了 Cur TRX 的 ID, TRX ID 是会自增的, 所以后面的 TRX ID 一定要大于前面的 TRX ID, 这里 `creator_trx_id = 50`.
- `trx_ids` 记录了所有正在活跃的 TRX, 这里 `trx_ids = 20, 30`.
- `min_trx_id` 记录了 `trx_ids` 中最小的 `trx_id`, 这里 `min_trx_id = 20`.
- `max_trx_id` 记录了 `trx_ids` 中最大的 `trx_Id` + 1, 这里 `max_trx_Id = 31`.

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241733177.png)

依次遍历 Version Linked List, 根据下面这四步查找规则, 找到一个合适的 Version.

- `trx_id == creator_trx_id`, 表示当前的 Version 就是由 Cur TRX 创建的, 可以访问.
- `trx_id < min_trx_id`, 表示当前 Version 已经提交, 可以访问.
- `trx_id > max_trx_id`, 表示当前 Version 是在当前 Page View 生成之后才开启的, 不能访问, 直接退出, 不需要进行后续的遍历判断了.
- `trx_id not in trx_ids`, 表示当前 Version 已经提交了, 可以访问.

InnoDB 的 RC, 每次执行 `select`, 就会创建一个 Page View, 可以解决 Dirty Read.

InnoDB 的 RR, 只有第一次执行 `select`, 才会创建 Page View, 后续执行插入后, 不影响第一次查询的 Page View, 所以不仅可以解决 Dirty Rad, 还可以解决 Non-Repatable 和 Phantom Read.

## MySQL Replication

Replication 可以让读写分离, Master 负责写, Slave 负责读, 分摊了压力. Slave 就是一个天然的数据备份, 实现了高可用.

Master 更新数据后, 将信息写入 Bin Log. Log Dump Thread 会去读取 Bin Log, 将内容发送给 Slave, 这个过程需要加锁, 因为 Master 也在进行写操作. Slave 的 IO Thread 会去进行解读内容, 并写入 Relay Log. Slave 的 SQL Thread 会去读取 Relay Log 执行 SQL 进行数据同步.

Canal 阿里巴巴开源的一种基于数据库增量日志解析的数据同步工具, 主要用于 MySQL 数据库的 Binlog 增量订阅和消费

Canal 会模拟 MySQL Slave 的交互协议, 伪装自己为 MySQL Slave, 向 MySQL Master 发送 dump 协议

## Architecture

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312241726446.png)

MySQL 基于 CS Arch, Client (eg: JDBC, ODBC) 发送一段 SQL 给 Server (eg: mysqld), Server 返回处理结果给 Client

Connection Layer 负责管理 Connection, Validation, Authorization. Client 和 Connection Layer 建立 TCP Connection 后, 会对 User 进行 Validation. 如果不通过, 就会返回 Error (Access denied for users). 如果通过, 就会进行 Authorization, 并创建一个 Thread 监听处理 Client Request.

Connection Layer 通过 Connection Pool 实现 Connection, 避免频繁的 Creation 和 Destroy

Service Layer 包含 MySQL 的大多数核心功能 (eg: Query Caching, SQL Parsing, SQL Analysis, SQL Optimizing, Built-in Function), 所有的 Cross Storage Engine 的 Function 都是在这实现的 (eg: Stored Procedure, Trigger, View).

Storage Engine Layer 负责 Data Storage 和 Data Extraction, 支持多种 Storage Engine (eg: InnoDB,MyISAM, Memory).

MySQL 执行 SQL, 需要 Client 和 Connector 建立连接, 接着依次执行 Query Caching, SQL Parsing, SQL Preprocessing, SQL Optimization, SQL Execution.

## Query Caching

Query Caching 可以将一条 SQL 作为 key, 将处理结果作为 value, 缓存起来. 下次查询, 会先查询 Cache 中是否有匹配 key 的, 如果有直接返回 value, 就不需要做接下来的操作了

MySQL 8 删除了 Query Caching. Cache Hit Ratio 太低, 要求 key 完全相同比较困难. 如果修改数据后, 缓存更行不及时, 会导致脏数据

## SQL Parsing

SQL Parsing 是对 SQL 进行 Preprocessing 和 Parsing, 提取 Keyword, 先进行 Lexical Analysis, 再进行 Syntax Analysis, 生成一个 AST, 接着优化 AST, 生成 Physical Planning.

Lexical Analysis 是将 SQL 分成一系列的词元和标记, 识别出每个字符串代表什么.

Syntax Analysis 是将对输入的 SQL 进行语法校验.

## SQL Preprocessing

进行一些基本的检查 (eg: 检查表和字段是否存在, 用户是否有权限访问目标数据), 此阶段可能会对 AST 进行树的改写以进行一些优化

## SQL Optimization

接收到 Physical Planning 进行 LQO 和 PQO, 找到最好的执行计划后.

LQO (Logical Query Optimization) 更改查询方式, 以提高效率, 就是这里更改了 SQL 的执行顺序.

PQO (Physical Query Optimization) 通过 Index 和 Table Join 进行优化.

## SQL Execution

接收到 Physical Planning 后, 进行权限检查, 执行查询, 处理结果集, 如果遇到错误, 需要进行错误处理.

## AST

AST (Abstract Syntax Tree) 是对 SQL 语句的结构化解析表示, AST 将 SQL 语句进行抽象化解析, 将其转换为一个树形结构, 这种结构能够反映出 SQL 语句中各元素 (eg: 关键字、表达式、函数、操作符) 之间的逻辑关系.

MySQL 的源代码中有对 SQL 语句进行解析并生成 AST 的部分, 但对 AST 的直接操作和查看并不像某些语言 (eg: JavaScript) 的 AST 工具那样普遍和容易. 目前可以使用 [sql-ast](https://github.com/aleclarson/sql-ast), 可以将来自 `mysqldump` 的输出解析为 AST.

## Slow Query

一条 Query 执行耗时太久, 就是一条 Slow Query, 一般可以通过 SkyWalking, Promotheus, Arthas 这样的监测工具去监测耗时较久的 API. MySQL 也提供了 Slow Query Log 记录 Slow Query.

一般优化一些高并发场景的 Slow Query, 从 Disk IO, CPU, Network Bandwidth 三个角度来分析问题.

- Disk IO: 是否正确使用了 Index
- CPU: Order By, Distinct, Group By 是否占用太多, 针对性的添加 Index 进行优化
- Network Bandwidth: 提升 Network Bandwidth

Slow Query Log 开启后, 会影响一定性能, 一般需要调优时, 就可以打开辅助调优.

## Split

MySQL 数据库的分库分表主要是为了应对大数据量或者高并发的场景, 分库分表存在分布式事务一致性的问题, 跨结点关联查询, 分页查询, 排序查询的问题, 主键避重的问题

单库无法承受当前的并发量, 请求量极高, 需要大量的计算和 IO 时, 就需要分库

单表数据量过大时 (eg: 单表的数据量达到 1000W 或 20G), 就需要拆分表

分库分表虽然可以拆分业务, 可以降低一定系统间的影响, 但是导致的问题太多, 非常难处理, 数据量没这么大时, 不要分库分表 !!!

## Split Database

水平分库, 将一个库的数据拆分到多个库中, 通过 Id 取模的方式路由到不同的库中

- eg: db_user 拆分成 db_user_0, db_user_1, db_user_2 三个库, 通过 id % 3 的方式选择不同的库进行访问

水平分表, 将一个表的数据拆分到多个表中, 通过 Id 取模的方式路由到不同的表中

- eg: tb_user 拆分成 tb_user_0, tb_user_1, tb_user_2 三个表, 通过 id % 3 的方式选择不同的表进行访问

## Split Table

垂直分库, 将一个库的表拆分到多个库中

- eg: db_service 拆分成 db_user, db_order, db_sku, 不同的业务访问不同的库

垂直分表, 将一个表的字段拆分到多个表中, 根据字段不同的访问频率进行冷热隔离

- eg: user(id, name, desc) 拆分成 user(id, name) 和 user_detail(id, desc)

## SQL Injection

查看 Log 和 DB 确认是否存在 SQL Injection, 立即隔离受影响的系统, 降低风险

PrepareStatement 会对 SQL 进行预处理, 不需要 "+" 拼接字符串, 不存在 SQL Injection, 尽量不要使用字符串拼接

```java
Scanner scanner = new Scanner(System.in);
// name 输入 1' or
String name = scanner.nextLine();
// password 输入 or '1' = '1
String password = scanner.nextLine();
// select * from accounts where name = '1' or' and password = 'or '1' = '1';
String sql = "select * from accounts where name = '" + name + "' and password = '" + password + "';";
```

```sql
select * from accounts where name = '1' or ' and password = ' or '1' = '1';
```

对入参进行校验, 过滤

用户的权限应该控制在最小粒度, 避免权限过高导致的大问题

定期进行安全审计, 设置监控和告警机制

# Redis

## Redis Replication

Replication 可以实现读写分离, 可以用于数据备份, 可以水平扩展, 可以实现高可用, Master 负责写, Slave 负责读, Master 更新数据后, 异步同步数据

Master 可读可写, Slave 只可以读. Slave 默认每 10s 发送一次心跳给 Master. Master 关机后, Slave 不会上位, 等待 Master 开机后, 一切照旧.

Slave 进行同步时, 会携带 Replication Id 和 Offset 请求 Master, Master 会去比较该 Replication Id 是否和自身的相同. 如果不同, 则认为是第一次同步, 会通过 RDB 进行全量同步, 如果相同, 则会进行增量同步

Master 进行全量同步时, 一边通过子线程创建 Snapshotting, 一边累积命令到缓存中. RDB Persistence 结束后, 会发送 Snapshotting 和缓存中的命令给 Slave, 覆盖 Slave 的数据, 实现第一次的全量同步

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312261306562.png)

Offset 会去记录上一次同步的位置, 再次请求时, Master 一看 Replication Id 相同, 就会认为这次是增量同步, 就会将 Replication BackLog 中 Offset 后面的数据同步给 Slave 实现增量同步, 在 Master 宕机恢复后, 这又相当于 Breakpoint Resume

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312261306398.png)

Replication BackLog 本质是一个数组, 也是有空间上限的, 超出上限后, 会去直接覆盖先前的内容. 如果 Slave 断开太久, 导致未备份的数据被覆盖了, 就无法基于 BackLog 进行增量同步, 只能被迫进行低效的全量同步

优化全量同步, 减少一个 Master 连接的 Slave 数量, 可以让 Slave 再去连接多个 Slave, 分摊 Master 压力, 减少单节点的内存占用, 减少 RDB 的 IO 次数

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202312261457996.png)

## Redis Sentinel

Sentinel 通过 PING PANG 不断监控 Master 和 Slave 是否按预期工作. 如果 Master 故障了, Sentinel 会根据投票数将一个 Slave 转成 Master, 并通知 Client 最新的 Master IP 和 Slave IP 进行转移故障

Subjective Down, 单个 Sentinel 发送心跳给 Master, 一定时间内没有收到合法回复 (def. 30s), 就主观认定 Master 不可用

Objective Down, 超过指定数量的 Sentinel 都主观认为下线时, 就客观认定 master 不可用

Sentinel 认定 Master 为 Objective Down, 所有 Sentinel 根据 Raft Algo 选举一个 Sentinel 成为 Leader, 执行 Failover. Leader 根据 down-after-milliseconds, replica-priority, replication offset, run id 选择一个 Slave 成为 New Master. 等 Old Master 和 Slave 重启后, 会添加配置到 redis.conf 中, 让他们服从 New Master

- down-after-milliseconds 表示该 Slave 和 Master 失联的时长, 超过一定数值后, 直接排除
- replica-priority 越小, 优先级越高
- replication offset 表示复制的完整度, 越大优先级越高
- run id 是一串字符, 根据 ASCII 比较, 越小, 优先级越高

Cleft Brain 是指 Master, Slave 和 Sentinel 失联了, 处于不同的网络分区, Sentinel 无法感知 Master, 就会去从 Slave 中选举一个 New Master, 但是 Old Master 那边还在跟 Client 进行通信, 等网络恢复后, Old Master 会被降级为 Slave, 导致失联那会的数据丢失

解决 Cleft Brain, 设置 Master 最少连接的 Slave 数量, 如果小于该数值, 就认为该 Master 失联了, 就停止和 Client 进行通信

```shell
min-replicas-to-write 1
```

解决 Cleft Brain, 设置 Master 和 Slave 之间 Replication 的延迟, 如果大于该数值, 就认为该 Master 失联了, 就停止和 Client 进行通信

```shell
min-replicas-max-lag 5
```

## Redis Shard

Shard Cluster, 多个 Master, 每个 Master 可以挂载多个 Slave, 每个 Master 负责管理部分 Slot 的数据, Master 之间会不断的通过 PING 去检测彼此健康状态, Client 连接任意一个结点, 请求就会被转发到正确的位置

Redis 共有 16384 个 Slot, 通过 Slot 存储数据, 添加结点, 删除结点, 只需要移动 Slot, 而且移动 Slot 是不需要停止服务的. 存储数据时, 会通过 CRC16 Algo 将 Key 转换成一个 Hash, 再对 16384 取模确定具体存储到哪个 Slot 中. 数据跟 Slot 绑定, 而不跟 Node 绑定, 这样即使 Node 宕机了, 也可以转移 Slot 来恢复数据

如果 Key 有 {} 包裹的部分, 就会根据 {} 里的内容计算 Hash. 如果 Key 没有 {}, 就会直接根据 Key 计算 Hash. 所以想要同一批数据存储在同一个 Slot 中, 就可以通过 {} 赋予共同的前缀

Slot 越少, 压缩比率就越高, 请求头就越小, 占用的带宽就越少, 一般不建议 Redis Cluster 超过 1000 个结点, 所以 16384 个 Slot 绝对够用

Redis Cluster 中, 节点之间需要同步心跳包, 这个心跳包的大小就取决于 Slot 的数量, 如果数据包太大, 数据之间的心跳同步就会占用很多带宽

Client 发送写请求后, 如果 Master 停机, Slave 还来不及进行 Replication, 就会造成数据丢失

CRC16 Algo 的 source code 在 cluster.c 的 keyHashSlot()

Hashing Algo, hash 按照结点的数量取余, 根据结果存储到对应的结点中, 如果结点数量发生变化, 影响数据的存储

Consistent Hashing Alog, 通过 hash() 控制 Hash 范围, 头尾相连, 构成 Hash Circle, 通过 hash() 计算结点和数据的 Hash, 分布在 Hash Circle 上, 数据沿着顺时针向前寻找到最近的结点, 存储在该结点上. 结点数量发生变化, 只影响一段数据的存储, 但是如果分布不均匀, 数据倾斜, 部分结点的压力会很大

## TTL

Redis 通过 redisDb 这个结构体表示 DB, 通过 Dict* dict 存储所有的 key-val, 通过 Dict* expires 存储所有 key-ttl

Redis 只需要查询 expires 就可以得到 key-ttl, 就可以判断该 key 是否过期

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031140695.png)

如果一个 key 到期后要立即删除, 就得给每一个 key 添加一个监视器, 到期后进行删除, 当 key 的数量特别多时, 非常耗费资源

Redis 采用 Lazy Deletion + Cycle Deletion 的方式删除一个 key, 不会在一个 key 过期后, 就立即进行删除. Lazy Deletion 是下一次访问 key 时, 判断该 key 是否已经过期, 如果过期了就进行删除. 如果一个 key 过期后, 一直没有被访问, 就会导致无法删除该 key, 所以还需要结合 Cycle Deletion 周期性的抽样部分过期的 key 进行删除, 这个抽样会逐步遍历所有过期的 key, 不会存在遗漏的情况. Cycle Deletin 有 Slow 和 Fast 两种模式, Slow Mode 是每 100ms 执行一次, 每次执行不超过 25ms, Fast Mode 是执行频率不固定, 但是两次间隔不低于 2ms, 每次执行不超过 1ms

## Eviction Policy

Redis 在执行 processCommand() 时, 就会跟据 Eviction Policy 去挑选部分 key 进行删除. 如果没有明显的冷热数据区分, 优先使用 allkeys-lru. 如果数据的访问频率差别不大, 优先使用 allkeys-random. 如果有需要置顶的数据, 这些置顶数据不会设置 TTL, 这时就优先使用 volatile-lru. 如果业务中有短时间内高频访问的数据, 再考虑 allkyes-lfu 和 volatile-lfu

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031140696.png)

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031140697.png)

配置 Eviction Policy

```shell
maxmemory-policy noeviction
```

## RDB

RDB 可以用于 Full Persistence, 定时存储 Snapshotting 到硬盘, 在 Redis 启动时, 加载 RDB 文件, 恢复数据. 还可以用于 Master-Slave Replication, Master 将 RDB 文件发送给 Slave, 用于初次全量复制和快速同步

RDB 会在 Redis Server 服务结束前自动执行, 会在达到了保存条件时自动执行, 执行 `FLUSHALL`, `FLUSHDB`, `SHUTDOWN` 时也会触发 RDB, 还可以通过 `SAVE` 和 `BGSAVE` 手动触发

- `SAVE` 是由主线程执行, `BGSAVE` 是由子线程进行, `SAVE` 一定会造成线程堵塞, 生产环境中不要用

Redis 进行 RDB Persistence 时, 会调用 fork() 创建一个子进程, 这个子进程不需要执行 exec(), 而是会直接复制一份父进程的 Page Directory 和 Page Table, 主线程执行完 fork() 就可以继续去处理请求了, 两者相不干扰. 如果主线程想要修改数据, 就会采用 Copy-On-Write 的方式, 给内存中的原始数据加上 ReadOnly Lock, 然后复制一份出来进行修改, 修改完再去修改 Page Table 的指向

- Page Table 中记录了虚拟地址和物理地址的映射, 子进程就可以通过这个 Page Table 去读取数据进行持久化操作了

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401021948769.png)

RDB 文件占用小, 加载速度快, 主线程不需要进行 IO, 性能极强

RDB 执行间隔耗时较长, 两次 RDB 直接的数据有风险, 并且复制, 压缩, IO 都是比较耗时的

RDB 适合做备份, 适合做灾难恢复, 适合数据完整性和一致性要求不高的场景

开启 RDB 的 Auto Snapshotting

```
# After 3600 seconds (an hour) if at least 1 change was performed
# save 3600 1

# After 300 seconds (5 minutes) if at least 100 changes were performed
# save 300 100

# After 60 seconds if at least 10000 changes were performed
# save 60 10000

# Snapshotting can be completely disabled with a single empty string argumen
save "" 
```

配置 RDB

```
# filename
dbfilename dump.rdb

# save path
dir /data/redis/

stop-writes-on-bgsave-error yes

# Compress string objects using LZF when dump .rdb databases
# rdbcompression yes

rdbchecksum yes 
```

手动触发 RDB

```
# save with blocking
SAVE

# save without blocking (recommand)
BGSAVE

# get the timestamp of the last snappshotting
LASTSAVE
```

通过 RDB 文件修复数据

```shell
redis-check-rdb /home/harvey/data/redis/dump.rdb
```

建议, 使用 RDB 做数据备份, RDB 占用非常小, 可以定期在一台压力比较小的 Slave 上手动执行 RDB 进行备份

## AOF

AOF (Append Only File) 会去追加写入所有的修改命令到 AOF 文件中, 类似于 MySQL 的 Redo Log, 在 Redis 重启时, 可以使用这些命令来重构数据

AOF 丢失数据的风险会小很多, 并且通过追加的方式写入, 不存在 Path-Seeking 问题

AOF 记录的命令多, 占用更大, 恢复也需要一条一条的执行, 恢复很慢, 占用的 CPU 资源也相当多

开启 AOF

```shell
appendonly yes
```

配置 AOF

```shell
appendfilename "appendonly.aof"
appenddirname "appendonlydir"
```

Redis 将写入操作追加到 AOF Buffer 中, 再自动将数据写入到 OS 的 Page Cache 中, 接着执行 fsync() 将 Page Cache 中的数据立刻刷入 (flush) 到 Disk

配置 AOF 的写入策略

- `everysec` 的策略, 每隔 1s 将 Page Cache 中的数据刷盘到 Disk 中, 最多丢失 1s 内的数据, 性能也适中, 而且 OS 也不太容易崩溃, 所以一般建议使用这个
- `always` 是每次写入操作都会立即刷盘到 Disk 中, 性能差, 安全性强
- `no` 是每次写入操作, 只会将数据写入到 Page Cache 中, 后续 Redis 就不管了, 由 OS 决定何时进行刷盘, 性能强, 安全性差.

```shell
# appendfsync always
# appendfsync no
appendfsync everysec
```

修复 AOF 文件

```shell
redis-check-aof --fix /home/harvey/data/redis/appendonlydir/appendonly.aof.1.incr.aof
```

建议, 关闭 RDB Persitence, 开启 AOF Persistence, 因为 RDB 的刷盘频率太低, 不适合做 Persitence

## Mixed

如果 AOF 存在, 加载 AOF file, 如果 AOF 不存在, 加载 RDB file

```shell
aof-use-rdb-preamble yes
```

## Log Rewriting

追加写入修改命令, 会有很多无用的操作 (eg: `set k1 v1`, `set k1 v2`, `set k1 v3 ` 这几条命令就等价于 `set k1 v3`), 所以很有必要定期对 AOF 文件进行重写, 这就是 Log Rewriting

开启 Auto Rewriting 后, 子线程会去读取 Old AOF File, 然后分析命令, 压缩命令, 写入到一个临时文件中. 主线程一直累积命令在缓存中, 正常写入命令到 Old AOF File 中, 保证 Old AOF File 的可用性. 当子线程完成 Rewriting 后, 会发送一个信号给主线程, 主线程再将缓存中的累积的命令追加写入到 New AOF File 中, 再通过 New AOF File 代替 Old AOF File

- 子线程读取 Old AOF File 后, 会将文件内容加载到内存中进行处理, 所以主线程后续修改 Old AOF File 不会对子线程的读取造成影响
- 如果替换过程中如果发生了故障, Redis 依然可以通过 Old AOF File 来恢复数据, 这就是为什么在重写过程中 Old AOF File 一直要保持可用状态

开启 Auto Log Rewriting

```shell
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

手动执行 Log Rewriting

```shell
BGREWRITEAOF
```

主线程执行完命令, 将命令写入 AOF Buffer 中, 每隔 1s 就从 AOF Buffer 中读取命令, 进行刷盘, 即 fsync. 主线程写入完后, 会去判断上一次 fsync 耗时, 如果超过 2s, 主线程就会进入堵塞, 等待 fsync 结束, 因为刷盘出了问题, 必须要保证数据的安全

在 Log Rewriting 期间, 进行 AOF, 就有可能因为 AOF 导致主线程堵塞, 可以禁止在 Log Rewriting 期间进行 AOF

```shell
no-appendfsync-on-rewrite yes
```

建议, 设置一个合理的 Log Rewriting 阈值, 避免频繁的 Log Rewriting, 太占用资源了

建议, 预留足够的空间处理 Fork 和 Log Rewring

## Redisson

Redisson 底层通过 Redis 的 SETNX 进行加锁的, A 想要获取锁, 就会尝试通过 SETNX 去修改一个 Key, 如果修改成功, 就认为是成功获取了锁. B 这个时候想要获取锁, 也去尝试修改, 修改不成功就认为是没有获取到锁, B 就会进入自旋, 自旋到一定时间, 就会放弃

Redisson 底层为了防止 A 执行的业务耗时太久, 导致锁的 TTL 到期失效的问题, 就让 Watch Dog 去监听这个锁, 每隔 releaseTime / 3 的时间就去重置 Lock 的过期时间为 releaseTime

Redisson 底层通过 Redis 的 Hash 实现 Reentrant Lock, 存储 Key 为锁名, Field 为线程名, Value 为重入的次数. 重入获取锁时, 就去判断当前线程和锁的拥有者是否为相同, 如果相同, 就让 Value + 1, 释放锁后, 就让 Value - 1, 当 Value 为 0 时, 就认为该线程释放了锁

Redisson 底层所有的操作中, 需要保证原子性的地方, 就会采用 Lua 脚本 (eg: 判断当前线程是否为锁的持有者和释放锁, 这两个操作需要保证原子性, 就会采用 Lua 脚本)

Redisson 底层通过 RedLock 解决 Master-Slave 和 Cluster 环境下, Lock 的一致性问题, 创建分布式锁时, 直接在 n / 2 + 1 个 Redis 实例上创建锁, 即使当前 Redis 实例挂掉了, 也能保证数量的领先, 只会认定数量多的那把锁. 一般不建议采用这种方案, 性能太差, 而且 Redis 遵循的是 AP, 更注重性能, 后续可以通过 MQ 来保证最终一致性, 不在乎这点一致性. 如果非要保证 High Consistency, 就需要结合 Zookeeper 实现 Distributed Lock

## DataStructure

- String 的 INCR 实现分布式环境下的自增 ID
- Hash 实现购物车 `hset cart:<user_id> <goods_id> <goods_number>`
- List 的 LPUSH, LRANGE 获取最新的微博消息
- Set 实现抽奖功能 
  - SADD 参加抽奖活动
  - SMEMBERS 查看抽奖人员信息
  - SRANDMEMBER 随机抽奖
  - SPOP 抽一二三等奖, 将抽到奖的用户移除 SET, 防止重复抽奖
- Set 实现朋友圈功能
  - `SADD like:<msg_id> <user_id>` 实现点赞
  - `SREM like:<msg_id> <user_id>` 实现取消点赞
  - `SMEMBERS like<msg_id>` 查看点赞列表
  - `SCARD like<msg_id>` 获取点赞用户数量
- Set 实现微博关系功能
  - `SINTER following:harvey following:bruce` 实现共同关注
  - `SDIFF following:harvey following:brude` 实现可能认识的人
- ZSet 实现微信点赞功能, 以时间戳作为 Priority, 实现热搜排行, 实现新闻排行- 

# MyBatis

## MyBatis Lazy Loading

MyBatis 通过 CGLIB 创建 Proxy Obj, 在需要时去加载关联数据, 提高查询性能, 当调用 getOrderList() 时, 会被 Proxy Obj 的 invoke() 拦截, 判断 orderList 是否为空, 如果为空才去执行 SQL 查询数据, 填充到 orderList, 再去调用 getOrderList() 执行后续逻辑, 实现 Lazy Loading

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401221740212.png)
## MyBatis Cache

Local Cache 基于 PerpetualCache, 本质是一个 HashMap

Lv1 Cache 基于 PerpetualCache, 作用于 Session, 执行 close() 和 flush() 后, 就会清空 Cache (def: enable)

Lv2 Cache 作用于 Namespace 和 Mapper, 不依赖 Session (def: disable)

这里 userMapper1 和 userMapper2 的操作在同一个 Session 下, 所以 userMapper1 执行了 SQL 后, 缓存数据到 Cache 中, userMapper2 再次执行 selectById(1) 就是查询的 Cache 中的数据

```java
SqlSession sqlSession = sqlSessionFactory.openSession();

UserMapper userMapper1 = sqlSession.getMapper(UserMapper.class);
User user = userMapper1.selectById(1);

UserMapper userMapper2 = sqlSession.getMapper(UserMapper.class);
User user = userMapper2.selectById(1);
```

这里 userMapper1 和 userMapper2 的操作在不同的 Session 下, 所以无法共享 Cache

```java
SqlSession sqlSession1 = sqlSessionFactory.openSession();
UserMapper userMapper1 = sqlSession1.getMapper(UserMapper.class);
User user = userMapper1.selectById(1);

SqlSession sqlSession2 = sqlSessionFactory.openSession();
UserMapper userMapper2 = sqlSession2.getMapper(UserMapper.class);
User user = userMapper2.selectById(1);
```

# Spring

## Spring IOC

IOC (Inversion of Control) 一种设计原则, 用于减小计算机程序中各模块之间的依赖关系. 我们只需要定义一个 Bean 的创建过程, 而真正的创建, 初始化, 装配, 生命周期都由 Container (eg: ApplicationContext, BeanFactory) 管理. 通过 DI 注入对象, 只需要关注自己的核心逻辑, 而不需要关注如何获取其他对象.

IOC 最佳实践了 Singleton 和 Fast Fail, 不仅可以节省大量不必要的对象创建, 防止 GC, 还在项目启动时, 就实例化所有的 Bean, 可以将 Bean 的创建由运行期提前至启动期, 在启动时期就可以检测出问题, 而不是在运行时遇到问题停机. Singleton 是不可变状态, 可以保证线程安全.

IOC 最佳实践了 DIP (Dependence Inversion Principle), 高层模块不直接依赖低层模块, 而是依赖低层模块的抽象, 低层模块去实现抽象 (eg: Controller 通过 Service 访问 ServcieImpl), 实现 Decoupling, 同时接口的引入便于后续扩展, 便于引入 Design Pattern (JDK's Dynamic Proxy).

## Spring IOC Process

IOC 的核心思想就将对象的管理交给容器, 应用需要使用某个对象的实例, 就去容器中获取即可, 降低了程序中对象和对象之间的耦合性

IOC 初始化: 加载声明的 Bean, 保存到 IOC 中, IOC 的初始化就是保存这些 Bean Instance 到 singletonObjects 中功能

IOC 的 Bean 的初始化: 通过反射去初始化那些没有设置 Layz Init 的 Bean

IOC 的 Bean 的使用: 通过 DI 获取的 Bean 实例, 对于设置了 Lazy Init 的 Bean 则是在通过 DI 获取时, 才会进行初始化

## Spring Lifecycle

Starting: 通过 BootstrapContext 启动 Application, 发布 ﻿ApplicationStartingEvent

Environment Prepared: 准备 ﻿Environment Obj, 会发布 ﻿ApplicationEnvironmentPreparedEvent

Context Prepared: 准备 SpringApplication Obj, 加载 Source, 创建 ApplicationContext Obj, 发布 ApplicationContextInitializedEvent

Context Loaded: 刷新 ApplicationContext, 初始化 Bean 的依赖关系, 发布 ApplicationPreparedEvent

Started: 在启动 CommandLineRunner 和 ApplicationRunner 之前, 此时 ApplicationContext 已经被刷新并且所有的 ﻿Spring Bean 都已经被创建, 发布 ApplicationStartedEvent

Ready: 在启动 CommandLineRunner 和 ApplicationRunner 之后, 已经完全启动并准备好接受 HTTP 请求, 发布 ApplicationReadyEvent

Shutdown: 关闭 Application, 完成一些清理工作或者关闭应用所用的资源, 发布 ﻿ContextClosedEvent

Failed: 启动过程中出现错误或异常, 就会发布 ApplicationFailedEvent 表示启动失败

## Bean Lifecycle

Spring 创建 Bean 的过程

- 调用 loadBeanDefinitions() 扫描 XML 或 Annotation 声明的 Bean, 封装成 BeanDefinition Obj 放入 beanDefinitionMap 中, 再遍历 beanDefinitionMap, 通过 createBean() 创建 Bean
- 调用 createBeanInstance() 构建 Bean Instance, 去获取 Constructor, 先准备 Constructor 需要的 Parameter, 再执行 Constructor
- 调用 populateBean() 填充 Bean, 通过 Three-Level Cache 去注入当前 Bean 所依赖的 Bean (通过 @Autowired 注入的 Bean)

Spring 初始化 Bean 的过程

- 调用 initializeBean() 初始化 Bean
- 调用 invokeAwareMethods() 去填充 Bean 实现的 Aware 信息, Bean 有可能实现了 BeanNameAware, BeanFactoryAware 或 ApplicationContextAware 去扩展 Bean (类似于 Neddle, 可以感知到 Bean Lifecycle 中的信息)
- 调用 applyBeanProcessorsBeforeInitialization() 去处理 Bean 实现的 BeanPostProcessor 的 postProcessBeforeInitialization()
- 调用 Bean 中添加了 @PostConstruct 的 Init Method
- 调用 Bean 实现的 InitializingBean 的 afterPropertiesSet()
- 调用 Bean 中添加了 @Bean(initMethod = "initMethod") 的 Init Method
- 调用 applyBeanProcessorsAfterInitialization() 去处理 Bean 实现的 BeanPostProcessor 的 postProcessAfterInitialization(), AOP 动态代理就是由该 Processor 实现的
- 调用 registerDisposableBean() 注册实现了 Disposable 的 Bean, 这样销毁时, 就会自动执行 destroy()
- 调用 addSingleton() 将 Bean 放入 singletonObjects 中, 后续使用 Bean 都是从 singletonObjects 中获取

jSpring 销毁 Bean 的过程

- 调用 Bean 中添加了 @PreDestroy 的 Destroy Method
- 调用 destroyBeans() 遍历 singletonObjects, 逐一销毁所有的 Bean, 这个过程会依次执行 Bean 的 destroy()
- 调用 Bean 中添加了 @Bean(destroyMethod = "destroyMethod") 的 Destroy Method

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401081756800.png)

## SpringMVC Process

SpringMVC 处理 JSP 的流程

- Client 请求 Servlet, Servlet 转发请求给 SpringMVC 的 DispatcherServlet
- DispatcherServlet 请求 HandlerMapping 解析请求, 查询 Handler, HandlerMapping 返回 HandlerExecutionChain
    - HandlerMapping 就是一个 Map, Key 为 uri (eg: /user/1), Val 为 Class#Method (eg: com.harvey.controller.UserController#getById())
    - HandlerExecutionChain 包含了 Controller Method 和 Controller 的 Filter
- DispatcherServlet 携带 HandlerExecutionChain 请求 HandlerAdapter
- HandlerAdapter 根据 HandlerExecutionChain 找到对应的 Handler
    - HandlerAdapter 会处理请求参数 (eg: Query Param, Body Param, Path Param), 将处理好的参数交给 Handler
- Handler 执行 Method 后, 返回结果给 HandlerAdapter
    - HandlerAdapter 接受到响应结果后, 封装成 ModelAndView
- HandlerAdapter 返回 ModelAndView 给 DispatcherServlet
- DispatcherServlet 携带 ModelAndView 请求 ViewResolver
- ViewResolver 处理 ModelAndView, 返回 View Obj 给 DispatcherServlet
    - ViewResolver 会将 ModelAndVie 这个逻辑视图转成 JSP 视图或者 Thymeleaf视图
- DispatcherServlet 响应 View 给 Client

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401181550236.png)

SpringMVC 处理 JSON 的流程

- Client 请求 DispatcherServlet
- DispatcherServlet 请求 HandlerMapping 查询 Handler
- HandlerMapping 返回 HandlerExecutionChain
- DispatcherServlet 请求 HandlerAdapter
- HandlerAdapter 请求 Handler
- Handler 执行 Method, 通过 HttpMessageConverter 转换响应数据为 JSON 格式, 返回结果给 HandlerAdapter
- HandlerAdpater 返回结果给 DispatcherServlet
- DispatcherServlet 再返回给 Client

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401221740209.png)

## Spring AOP

AOP (Aspect-Oriented Programming) 是面向切面编程, 可以不修改源代码的情况下, 抽取并封装一个可重用的模块, 可以同时作用于多个方法, 减少模块耦合的同时, 扩展业务功能. 可用于记录操作日志, 处理缓存, 事务处理

OOP 可以解决 Class 级别的代码冗余问题, AOP 可以解决 Method 级别的代码冗余问题.

Bean Lifecycle 的 postProcessAfterInitialization 阶段, 会调用 BeanPostProcessor 的实现类 AbstractAutoProxyCreator 的 postProcessAfterInitialization(), 先判断 Bean 是否需要实现 Dynamic Proxy, 如果需要则会去根据当前 Bean 是否有 Interface 选择是采用 JDK 还是 CGLib 实现 Dynamic Proxy.

Spring 底层的 TRX 就是通过 AOP 实现的, 通过 Surround 的方式扩展, 在方法开启前开启事务, 在方法结束后提交事务, 无侵入, 碉堡了 !!!

## AspectJ AOP

Aspecjt AOP 通过 Weaver (AspectJ Aop 自己的 Compiler), 将 @Before, @After, @Around 的代码编译成字节码织入到目标方法的字节码文件中, 即 AspectJ AOP 在编译器期间就完成了增强, 而 Spring AOP 是通过 Dynamic Proxy 实现了目标方法的增强.

AspectJ AOP 支持在方法调用, 方法内调, 构造器调用, 字段设置, 获取等级别的织入, 更加灵活强大.

AspejcJ AOP 不需要借助 Dynamic Proxy, 而是直接编译成字节码, 所以性能也要好很多.

## Auto Configuration

导入 Dependency 的 starter pkg 后, 会自动导入该 Dependency 的 autoconfigure pkg (eg. 导入 spring-boot-stater 后, 会自动导入 spring-boot-autoconfigure)

App.java 的 @EnableAutoConfiguration 底层包含 @Import({AutoConfigurationImportSelector.class}), 根据 SPI 访问 META-INF/spring/%s.imports (eg. META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports), 动态加载 Component

%s.imports 包含该 Dependency 提供的所有 Configuration (eg. xxx.AutoConfiguration.imports 包含 xxx.aop.AopAutoConfiguration, xxx.amqp.RabbitAutoConfiguration), Configuration 包含 @Import({xxxSelector.class}) 通过 @Condition 按条件导入 Bean, 实现 Auto Configuration

通过 Auto Configuration 导入的 Configuration 用到的 Properties Obj 可以在 application.properties 中配置

## Three-Level Cache

Spring 的 DefaultSingletonBeanRegistry Cls 中声明了 singletonObjects (ConcurrentHashMap), earlySingletonObjects (ConcurrentHashMap) 和 singletonFactories (HashMap) 用于实现 Three-Level Cache

- singletonObjects 是 Lv1 Cache, 存放经历了完整 Life Cycle 的 Bean Obj
  - singletonObjects 的 Key 为 Bean Name, Val 为 Bean Obj
  - 通过 applicationContext.getBean() 获取 Bean 就是访问 singletonObjects 这个 Map
- earlySingletonObjects 是 Lv2 Cache, 存放未经历完整 Life Cycle 的 Bean Obj, 解决 Circurlar Reference 的关键
- singletonFactories 是 Lv3 Cache, 存放各种 Bean 的 ObjectFactory, 可以用来创建 Normal Obj 或 Proxy Obj
  - singletonFactories 是 HashMap, 而不是 ConcurrentHashMap, 因为 singletonFactories 通常只在 Bean 的创建过程中使用, 一旦 Bean 创建完成, 即使有多线程对创建好的 Bean 进行访问, 访问的是 singletonObjects, 而不是 singletonFactories, 不存在线程安全问题.

```java
// Lv1 Cache
private final Map<String, Object> singletonObjects = new ConcurrentHashMap(256);
// Lv2 Cache
private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap(16);
// Lv3 Cache
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap(16);
```

Spring 的 DefaultSingletonBeanRegistry Cls 中声明了 singletonsCurrentlyInCreation (Collections.newSetFromMap(new ConcurrentHashMap<>(16))) 存储正在创建过程中的 Bean, 用来判断是否存在 Circular Reference.

```java
private final Set<String> singletonsCurrentlyInCreation = 
    Collections.newSetFromMap(new ConcurrentHashMap<>(16));
```

Spring 通过 Three-Level Cache 解决了大部分的 Circular Reference, 需要使用 A 时, 会执行下面的步骤

- 调用 getBean() 获取 A, 依次查询 singletonObjects, earlySingletonObjects 和 singletonFactories, 未查询到 A Cache, 调用 getSingleton() 创建 A

  - 调用 beforeSingletonCreation() 添加 A 到 singletonsCurrentlyInCreation 中, 表示 A 正在创建过程中
  - 调用 singleFactory.getObject() 通过 Reflect 创建 A Obj, 此时 A Obj 的成员都是空的, 即 A 引用的 B 也是空的
  - 生成 A 的 ObjectFactory 存入 singletonFactories, ObjectFactory 本质是一个 Lambda, 可用于动态创建 A 的 Normal Obj 或 Proxy Obj
  - 通过 BeanPostProcessor 发现 A 依赖 B, 需要去创建 B

- 调用 getBean() 获取 B, 依次查询 singletonObjects, earlySingletonObjects 和 singletonFactories, 未查询到 B Cache, 调用 getSingleton() 创建 B

  - 调用 beforeSingletonCreation() 添加 B 到 singletonsCurrentlyInCreation 中
  - 调用 singleFactory.getObject() 通过 Reflect 创建 B Obj, 此时 B Obj 的成员都是空的, 即 B 引用的 A 也是空的
  - 生成 B 的 ObjectFactory 存入 singletonFactories
  - 通过 BeanPostProcessor 发现 B 依赖 A, 并发现 A 也在 singletonsCurrentlyInCreation 中, 说明 A 和 B 存在 Circular Reference, 需要去处理 Circular Reference

- 调用 getBean() 获取 A, 依次查询 singletonObjects, earlySingletonObjects 和 singletonFactories, 从 singletonFactories 中获取到 A 的 OpenFactory, 执行 Lambda 创建 A Obj 放入 earlySingletonObjects, 并移除 singletonFactories 中 A 的 OpenFactory

  - 如果 C 引用了 A, 直接从 earlySingletonObjects 获取 A 即可, 不需要再通过 A 的 OpenFactory 获取 A Obj 了

- 调用 populateBean() 填充 B 依赖的 A, 此时 B 创建完成, 向 singleObjects 添加 B, 从 singletonsCurrentlyInCreation 和 singletonFactories 移除 B

  - 如果再使用 B, 就可以直接从 singleObjects 中获取

- 调用 populateBean() 填充 A 依赖的 B, 此时 A 创建完成, 向 singleObjects 添加 A, 从 singletonsCurrentlyInCreation 和 earlySingletonObjects 移除 A


![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401152302912.png)

Transaction ACID

Atomicity 是指一个 TRX 是一个不可分割的工作单元, 要么全成功提交, 要么全失败回滚, 成王败寇, 没有妥协之说, 通过 Undo Log 保证.

Consistency 是指数据需要从一个合法性状态变化到另一个合法性状态, 这个合法是业务层面的合法 (eg: A 扣钱, 扣成了负数, 则不符合业务层面的要求, 即不合法). 

Isolation 是指一个 TRX 内部使用到的数据对其他 TRX 隔离, 不会受到其他 TRX 的影响, 通过 MVCC 保证.

Durability 是指一个 TRX 一旦被提交, 它对数据库中数据的改变就是永久性的, 通过 Redo Log 保障的, 先将数据库的变化信息记录到 Redo Log 中, 再对数据进行修改, 这样做, 即使数据库崩掉了, 也可以根据 Redo Log 进行恢复.

## Design Pattern

Singleton Pattern: Bean 默认采用 Singleton Pattern, 通过 Container 管理 Bean Lifecycle, 保证每个 Bean 只被创建一次.

Factory Pattern: BeanFactory 和 ApplicationContext 采用 Factory Pattern 创建并管理 Bean.

Proxy Pattern: AOP 采用 Dynamic Proxy Pattern 创建 Aspect, 这里的 Aspect 就是 Proxy Obj.

Observer Pattern: ApplicationEventPublisher 采用 Observer Pattern, 异步处理 Event, 实现 Decoupling.

Template Pattern: JdbcTemplate, RedisTemplate 和 RestTemplate 都是采用的 Template Pattern. 抽象出了公共的操作流程, 将差异化的部分交给子类或者回调函数实现, 极大地提高了代码的复用性并且降低了出错可能性.

Strategy Pattern: MultipartResolver 采用了 Strategy Pattern, 可以选择 StandardServletMultipartResolver 或 CommonsMultipartResolver 对其进行实现, 在不同的环境或配置中选择不同的策略, 可以使得代码更灵活, 扩展性更好.

Chain of Responsibility Pattern: Filter 和 Interceptor 采用了 Chain of Responsibility Pattern, 多个 Filter 和 Interceptor 按照一定顺序执行, 每个 Filter 和 Interceptor 可以拦截请求或者响应并做出相应的处理

# Distributed

## CAP

CAP: Consistency, Availability, Partition 同时只能满足两个, 结点之间形成分区后, 要么拒绝请求, 保证 Consistency, 放弃 Availability, 要么依旧提供服务, 保证 Availability, 放弃 Consistency

BASE: 对 CAP 的一种解决方案, 结点之间形成分区后, 允许 Partial Availability, 要求 Core Availability, 允许 Temporary Incosistency, 要求 Eventual Consistency

- AP Mode: Sub Transaction 分别执行 Operation 和 Commit, 允许 Temporary Incosistency, 后续采用 Remedy, 保证 Eventual Consistency (eg: Redis)
- CP mode: Sub Transaction 分别执行 Operation, 相互等待, 放弃 Partial Availability, 保证 Core Availability, 共同执行 Commit (eg: ElasticSearch)

## Two Phase Commit

Two Phase Commit (2PC) 是一种原子提交协议, 是 MySQL 对分布式事务的 XA Mode 的实现, 用于协调参与分布式原子事务的所有进程, 决定提交或回滚, 该协议在许多临时系统失败的情况下依然能实现其目标, 因此得到了广泛的使用. 然而, 两阶段提交协议并不能抵御所有可能的失败配置, 在极少数情况下, 需要人工干预来纠正结果, 为了从失败中恢复 (大部分情况下是自动的), 协议的参与者使用日志记录协议的状态

Two Phase Commit 由两个阶段组成, 如果这两个阶段的数据不一致, 则会进行回滚, 保证数据一致性

- Prepare Phase: TC 通知 RM 去执行修改操作, RM 先记录 Undo Log, 接着执行修改操作, 再记录 Redo Log, RM 通知 TC 是否执行完毕
- Commit Phase: TC 根据 RM 的响应结果进行处理, 如果都是 Ready, 则 TC 通知 RM 进行 Commit, 如果有 Fail, 则 TC 通知 RM 进行 Rollback

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202403091222615.png)

Two Phase Commit 缺点

- 同步阻塞问题: 在二阶段提交协议中, 所有参与者在等待协调者的决定阶段都处于阻塞状态, 无法进行其它操作
- 单点故障: 如果在二阶段提交过程中, 协调者出现故障, 会导致所有参与者一直等待, 不能进行其它操作
- 数据不一致: 如果 RM 接收到 Prepare 请求后, 未发送 ACK 确认就宕机, 而在此之后其它 RM 都发送了 ACK 确认, 则此时 TC 将发起 Commit 请求, 导致数据状态不一致

## Three Phase Commit

Three Phase Commit (3PC) 对 Two Phase Commit 进行了优化, 引入了 Timeout 和 CanCommit Phase, 以避免阻塞和单点故障问题

- CanCommit Phase: TC 向所有 RM 发送 CanCommit 请求询问是否可以提交事务, RM 返回 Yes / No
- PreCommit Phase: 如果所有 RM 都返回 Yes, TC 向 RM 发送 PreCommit 请求, 接收到请求的 RM 表示接受 TC 的决定, 并回复 ACK
- doCommit Phase: TC 收到 RM 的 ACK 后, 向所有 RM 发送 doCommit 请求, RM 接受到请求后进行真正的事务提交并回复 ACK

Three Phase Commit 的任一阶段, 只要有 RM 回复 No 或者超时未回复, TC 都会向所有 RM 发送 Abort 请求, 所有 RM 在执行完事务的回滚操作后回复 ACK

Three Phase Commit 中的 TC 出现单点故障, 无法通知 RM 进行提交, 则 RM 会在等待一段时间后自动提交事务, 但是这也会导致一些问题. 如果某一个 RM 出现异常, 返回了 No, 刚好此时 TC 单点故障, 则会导致其他 RM 超时后自动提交, 造成不一致

Three Phase Commit 虽然解决了 Two Phase 的一些问题, 但是增加了一次网络往返, 而且在处理网络分区和多数故障的情况下, Three Phase Commit 也无法保证一致性, 当前实际使用中更常见的是使用具有超时和故障恢复机制的 Two Phase Commit (eg: Paxos, Raft)

## Quorum

Quorum (法定人数机制) 是分布式系统中的重要共识技术, 被设计用来提高分布式系统的可用性和容错能力, 尤其在面对节点故障或网络问题时

Quorum 的核心思想是一个请求在一个多节点分布式系统中被认为有效, 并在多数节点 (法定数量) 上一致同意后才会被执行, 所以在一个包含 N 个节点的系统中, 至少需要 N / 2 + 1 个节点同意, 这个请求才会被看做是已经达成共识

Quorum 中的节点包括 R (Read) 和 W (Write), 只要 R + W > N 就可以保证系统的一致性, 通常设置 R = N / 2 + 1, W = N / 2 + 1 (eg: N = 5, R = 3, W = 3)

- C1 写入数据 X, 会在 3 个节点上写入数据 (W)
- C2 读取数据 X, 会从 3 个节点上读取数据 (R), 所以总能保证至少读取到一条最新的数据, 再通过 Version Id 去区分哪一条是最新数据

Quorum 优点

- 容错能力: 即使系统中一部分节点发生故障或无法达成共识, 只要有超过半数的节点可以正常运行并且能够彼此通信, 系统就能够继续提供服务, 这使得系统能够在网络分区或节点宕机时保持可用性
- 数据一致性: 由于一个请求需要在大多数节点上被接受, 所以可以保证只要有一个最新数据的副本在正常工作的节点集合中, 那么所有的读请求都可以返回最新的数据

Quorum 在许多分布式系统中都有应用 (eg: Zookeeper 的 ZAB, Google's Spanner 的 Paxos), 通过 Quorum 来保证分布式数据一致性和高容错

## WARO

WARO (Write All Read One) 是一种在分布式数据存储系统中常用的数据一致性策略

- Write All: 所有的写操作都必须在所有的副本节点上完成, 在所有的参与节点中同步更新信息, 这保证了在完成写操作后, 所有的副本都持有最新的数据
- Read One: 读操作只需要在任何一个副本上进行就可以了, 由于写操作保证了所有副本的数据一致性, 因此无论从哪个副本读取数据, 读取的数据都是最新的

WARO 同时考虑了系统性能和数据一致性, 尽管需要在所有副本上完成写操作可能会导致性能开销, 但它可以保证数据的强一致性, 另一方面, 只需要在一个节点上完成读操作可以显著提高读操作的性能

WARO 模型的性能会受到系统负载, 网络延时和副本数量等因素的影响 (eg: 如果有大量的写操作, 就需要维护大量的副本, 调整网络延时和副本数量也可能影响系统的性能)

## Paxos

Paxos 是解决分布式系统中的一致性问题的一种算法, 能在一个可靠的系统中, 在任何时候提供一致性的保证, 在面对网络延迟, 分区和节点故障的时候依然可以正确地工作

Paxos 角色

- Proposer: 提案发起者, 可以理解为客户端或者是服务请求者, 负责发起一个提议, Paxos 将程序中的操作抽象成提议 Value (eg: 修改某个变量的值)
- Acceptor: 提案接收者, 主要负责接收 Proposer 的提议, 并对提议给予反馈, 至少 N / 2 + 1的 Acceptor 批准后, 才可以通过提议
- Learner: 观察者, 不参与提议过程, 仅在提议决议确定后, 通过监听 Proposer 和 Acceptor 的交互, 得知决议结果, 这个角色可以实际观察到系统状态的改变, 进行相应的操作, 在实际应用中, Learner 可以是 Acceptor 自身, 也可以是独立的角色
- Leader: Proposer 的一种特殊形式, 主要负责对外部请求的初步处理, 并发起提议, Leader 的选举是通过 Paxos 协议进行的, 通过不断抛出提案, 最终形成一个决议来达成 Leader

Paxos 选举过程

- Proposer 生成全局唯一且递增的 Proposal Id, 向 Paxos 集群的所有机器发送 Prepare 请求, 这里不携带 Value, 只携带 Proposal Id (N)
- Acceptor 接受到 Prepare 后, 判断 Proposal Id 是否比 Max_N 大, Max_N 记录了之前响应的 Proposal Id
  - 如果大, 就记录 Proposal Id 到 Max_N 中, 并返回之前的 Max_N
  - 如果小, 就不回复或者回复 Error
- 进行 P2a, P2b, P2c 三个步骤 (省略)

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202403091509474.png)

## Raft

Raft 主要用于分布式集群环境下, 从众多从节点中选举一个主节点, 如 RocketMQ, Nacos, Redis Sentinel 都采用了 Raft 来选举主节点

Raft 角色

- Leader: 领导者负责处理所有的客户端请求, 并将日志条目复制到其他的服务器, 在任何时候, 只有一个 Leader, 在更换 Leader 的过程中, 可能会短暂的出现没有 Leader 的情况, Leader 只会在它的日志里增加新的条目, 不会去删除或修改已存在的条目
- Follower: 响应 Leader 的请求, 如果 Leader 挂了, 会参与投票选举新的 Leader
- Candidate: 候选者通过 RPC 请求其他节点给自己投票

Log Entry 日志条目, 是一个基本的数据结构, 通常用于记录系统中发生的操作或事件, 每一次的客户端请求, 或者系统状态的改变, 都会生成一条日志条目, 添加到日志中

Log Entry 组成

- Term: 任期号, 也可以理解为该日志条目创建的时间点, 在 Raft 协议中, 每当有新一轮的领导者选举开始, Term 就会增加
- Command: 命令, 这通常是客户端请求的一部分 (eg: 一个键值对的 PUT 操作)
- Index: 索引, 这是日志条目在日志中的位置
- Command Parameter: 一些命令可能需要附带其它参数 (eg: 一个键值对的 PUT 操作, 需要知道要 PUT 的值)

Raft 选举流程

1. 初始状态

- A, B, C 在初始状态都是 Follower, 此时还没有 Leader, 所以不会接受到 Leader 的 PING, 都会进入超时等待的状态
- 每个节点的超时时间都是随机的, 防止同时有多个 Candicate 去拉票导致平票的情况, 假如这里 A 150ms, B 300ms, C 200ms, 其中 A 最先完成超时等待
- A 从 Follower 切换为 Candidate, 将自身的 Term 从 0 更新为 1
- A 先投自己一票, 然后通过 RPC 向其他节点请求投自己一票, 这里 B, C 最先收到 A 的拉票请求后, 就会立即给他 A 投一票, 并且也跟着修改自己的 Term 为 A 的 Term, 后续如果还有节点想要来拉票, 这里的 B, C 都会直接拒绝
- A 的票数为 3 >= 2 (N / 2 + 1 = 3 / 2 + 1 = 2) 完成选举, 成为 Leader, 开始处理客户端请求

2. Leader 宕机, 选举新的 Leader

- A 宕机或者网络堵塞了, B 和 C 都接受不到 A 的 PING, 进入超时等待的状态, 假如这里 B 150ms, C 300ms, 其中 B 最先完成超时等待
- B 从 Follower 切换为 Candidate, 将自身的 Term 从 1 更新为 2
- B 先投自己一票, 然后通过 RPC 向其他节点请求投自己一票, 这里 C 最先收到 B  的拉票请求后, 就会立即给他 B 投一票, 并且也跟着修改自己的 Term 为 B 的 Term
- B 的票数为 2 >= 2 完成选举, 成为 Leader, 开始处理客户端请求

3. Old Leader 恢复

- A 恢复健康后, 发现自己的 Term 比其他节点的 Term 小, 则自动成为 Follower

## Micro Service

Micro Service 是 Distributed 下的一种架构风格, 将一个大型服务拆分成多个小型服务, 实现服务之间的解耦, 每个服务都运行在自己的进程中, 通过一些轻量的通信协议进行通信 (eg: RPC), 每个服务都可以独立部署, 独立扩展, 独立更新, 可维护性, 可伸缩性大大提升

## Nacos

Nacos Server 是一个 Registration Center 和 Configuration Center

Nacos Client 请求 Nacos Server 实现注册, Nacos Server 会将各种信息 (eg: IP, Port) 存储到一个 Map 中

Nacos Client 会定时 (def: 20s) 向 Nacos Server 拉取最新的 Service List, 从而获取其他 Service 的信息实现通信, 当遇到多个相同服务时, 就通过 Load Balancing 选取一个 Service 进行通信

Nacos Server Cluster 中, Nacos Server 之间需要同步 Service List 保证数据一致性

Nacos Server 会定时检查 Service Health, 发现 Service 死亡后, 会推送变更消息给 Nacos Client

- Nacos Client 定时 (def: 5s) 发送 Heatbeat 给 Nacos Server, 表示健康
- Ephemeral Instance 在一定时间内 (def: 15s) 未发送 Heartbeat, 会被标记为 Unhealthy, 再过一段时间 (def: 30s) 未发送 Heartbeat, 会被剔除 Service List, Service 恢复后, 可以再次进行 Service Registry
- Persistent Instance 在一定时间内未发送 Heartbeat, 会被标记为 Unhealthy, 但是不会被剔除 Service List

Nacos Server 会在 Unhealthy Instance (eg: 5) / All Instance (eg: 10) < Protect Threshold (eg: 0.4) 时, 启动 Unhealthy 的 Persistent Instance 进行 Avalanche Protection

Server Load Balancing: Server 通过 Hardware (eg: F5) 或 Software (eg: Nginx) 拦截请求, 转发请求, 实现 Load Balancing

Client Load Balancing: Client 通过 Software (eg: Ribbon, LoadBalancer) 拦截请求, 转发请求, 实现 Load Balancing

## Load Balancing

Micro Service 同一个模块可能有多个实例, 就需要通过 Load Balancing 选取一个模块进行通信, 最新的 RestTemplate 和 OpenFeign 都可以通过 LoadBalancer 实现 Load Balancing

常见的 Load Balancing Policy

- Random Load Balancing: 随机算法, 适用于服务器性能相近的情况
- Weighted Random Load Balancing: 权重随机算法, 权重可以根据服务器的性能, 负载, 网络带宽等因素来设置
- Round Robin Load Balancing: 轮询算法, 适用于服务器性能相近的情况
- Weighted Round Robin Load Balancing: 权重轮询算法, 在轮询算法的基础上加入了权重的考虑
- Least Connections Load Balancing: 最少连接算法, 将新的请求分配给当前连接数最少的服务器, 适用于处理请求所需时间差异较大的情况
- Source IP Hash Load Balancing: 源地址哈希算法, 根据请求的源地址进行哈希计算, 将请求分配给服务器, 可以保证来自同一源地址的请求总是被分配到同一台服务器

## RPC

RPC 和 HTTP 是两种常用的网络通信协议, 它们在设计理念, 使用场景, 优势和限制等方面有所不同

RPC 允许一台计算机上的程序调用另一台计算机上的程序, 就像调用本地程序一样, 目的是为了使分布式系统中的跨网络的程序调用变得透明, RPC 可以基于多种底层传输协议实现 (eg: TCP, UDP, HTTP)

- RPC 更适用于构建分布式系统和微服务架构中服务间的通信, 特别是在需要高效, 紧密耦合的系统组件间进行远程过程调用时
- RPC 协议通常需要服务提供者和消费者之间有更紧密的协作和协议一致性, RPC 客户端和服务器可能需要使用相同的技术栈或遵循特定的协议规范
- RPC 的安全性取决于所使用的底层传输协议和额外的安全措施 (eg: TLS/SSL 加密, 认证机制)

HTTP 是一种应用层协议, 设计用于 Web 浏览器和服务器之间的通信, 但也被广泛用于不同系统之间的数据交换, HTTP 遵循请求响应模式, 主要用于传输 HTML 文档, 图片, 视频等超媒体信息

- HTTP 相对简单, 易于理解和调试, 虽然 HTTP2 提供了性能改进 (eg: 头部压缩, 多路复用), 但通常来说，HTTP 在性能上可能不及专为远程调用优化的 RPC
- HTTP 作为 Web 的基石, 具有极高的互操作性和灵活性, 它支持多种数据格式和编码标准, 使得不同技术栈的系统能够轻松集成和通信
- HTTP 本身支持通过 HTTPS 进行加密通信, 提供了数据传输的安全性

Dubbo 是基于 RPC 的分布式框架, 支持服务注册发现, 远程通信, 负载均衡, 超时处理, 熔断降级, 适用于构建高性能且复杂的微服务架构

OpenFeign 是一个声明式的 Http 客户端, 简化了 Http 的远程通信过程, 可以像调用本地接口一样实现 Http 请求, 适用于构建简单的微服务

# Network

## Multiplexing IO

Blocking IO 和 Non Blocing IO 都是第一时间调用 recvfrom() 获取数据, 数据不存在时, 要么等待, 要么空转, 都无法很好的利用 CPU, 还会导致其他 Socket 的等待, 非常糟糕

Client 连接 Server 后, 会建立一个关联的 Socket, 这个 Socket 有一个对应的 File Descriptor (FD), FD 是一个从 0 开始递增的 Unsigned Int, 用来关联一个文件, 在 Linux 中一切皆文件, 所以 FD 可以关联一些, 当然就可以用来关联 Socket

Multiplexing IO 中, User 想要读取数据, 会先调用 epoll(), 将所有 Socket 的 FD 传递给 Kernel, Kernel 只需要一个单线程时刻监听这些 FD, 哪个数据就绪了, 就告知 User 去获取数据, 这个时候 User 再调用 recvfrom() 去获取数据, 就不会有堵塞和空转的情况了, 非常好的利用了 CPU

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139651.png)

---

select() 和 poll() 是早期用于 Multiplexing IO 的函数, 他们只会通知 User 有 FD 就绪了, 但是不确定是哪个 FD, 需要 User 遍历所有的 FD 来确定. epoll() 则会告知是哪些 FD 就绪了, 不需要 User 查找了, 非常高效

select() 将所有的 FD 存储到 fds_bits 中, 这是一个 1024b 的数组, 调用 select() 后, 需要先将 fds_bits 从 User Space 复制到 Kernel Space, 等 select() 执行完, 还需要将 fds_bits 从 Kernel Space 复制到 User Space. User 无法直接确定是哪个 FD 就绪了, 还需要再遍历 fds_bits 寻找就绪的 FD. fds_bits 的大小固定为了 1024b, 这个数量放在如今已经完全不够用了

poll() 通过一个 LinkedList 存储 FD, 所以可以存储的 FD 就没有上限了, 但是依旧无法避免两次复制和遍历寻找就绪的 FD. 如果存储的 FD 太多, 遍历的时间会变长, 性能就会下降

epoll() 通过一个 RedBlackTree 存储所有的 FD (rbr), 通过一个 LinkedList 存储就绪的 FD (rdlist). Server 启动时, 会调用 epoll_create() 创建一个 epoll 实例, 包含 rbr 和 rdlist. User 想要添加一个 FD 时, 会调用 epoll_ctl() 添加一个 FD 到 rbr 上, 并且绑定一个 ep_poll_callback, 一旦该 FD 就绪, 就会触发该 Callback, 将 FD 添加到 rdlist 中. 后续只需要循环调用 epoll_wait() 检查 rdlist 中是否有 FD, 如果没有, 就进入等待, 如果有, 就复制到 User Space 的 events 中, 实现事件通知, User 就知道哪些 FD 就绪了, 就可以针对性的发送请求进行读写操作了. epoll() 不需要来回的两次复制, 也不需要遍历寻找就绪的 FD, 性能极强, 而且通过 RedBlackTree 存储 FD, 既能存储大量的 FD, 也能保证性能的稳定

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139652.png)

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202401031139653.png)

---

epoll_wait() 的事件通知有 LevelTriggered (LT, def) 和 EdgeTriggered (ET) 两种模式

LT 进行事件通知后, 会判断该 FD 是否已经全部处理完毕, 如果没有处理完, 就会反复通知, 直到处理完毕, 才会将该 FD 从 rdlist 中移除 (eg: A 需要处理 3KB 的数据, 调用 epoll_wait() 得到数据后, 只处理了 1KB 数据, 此时 LT 模式下, 不会移除该 FD, 而是再次通知 A 去处理数据)

ET 进行事件通知后, 会直接移除该 FD (eg: A 还剩 2KB 数据没有处理, 此时 ET 模式下, 不会管 A 是否处理完的, 直接移除 FD)

通过 LT 处理数据, 需要反复调用 epoll_wait() 进行处理, 这个过程是非常消耗资源的. 如果多个线程都在等待一个 FD, 通知一次给一个线程后, 还会再去通知其他线程, 这就造成了惊群

通过 ET 处理数据, 可以开启一个异步线程, 将 FD 中的所有数据全部循环处理完即可, 不需要反复调用 epoll_wait(), 性能极强, 而且一个线程被通知处理完数据后, 其他线程直接从 User Space 中读取数据即可, 不存在惊群

## TCP

TCP (Transmission Control Protocol) 是互联网协议套件的主要协议之一, 提供了在通过 IP 网络通信的主机上运行的应用程序之间的可靠, 有序和错误检查的字节流传输. TCP 是 TCP/IP 套件的传输层的一部分, SSL/TLS 经常在 TCP 上运行. 诸如万维网, 电子邮件, 远程管理和文件传输等主要的互联网应用都依赖于 TCP.

## 3-Way Handshake

TCP 的三次握手, 是建立 TCP 连接的重要过程, 以保证连接的双向通信

- Client 发送 SYN 给 Server, 表示要建立连接, Client 切换状态为 SYN_SEND
- Server 接受到 SYN 后, 返回 SYN 和 ACK 给 Client, Server 切换状态为 SYN_RCVD
- Client 接受到 SYN 和 ACK 后, 返回 ACK 给 Server, Client 切换状态为 ESTABLISHED. Server 接受到 ACK 后, Server 切换状态为 ESTABLISHED, 完成 TCP 的三次握手, 可以开始传输数据了

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202402151439103.png)

TCP 的三次握手以最少的通信次数确认了双方的接收和发送能力是否正常, 确认了双方的初始序列号以便后续的数据传输

如果只进行两次握手, 就会因为网络延迟导致的 Client 和 Server 的连接状态不一致

- Client 发送 SYN1 给 Server, 但因为网络延迟没有到达 Server, 于是 Client 又发送一个 SYN2 给 Server
- Server 接受到 SYN2 后, 返回 SYN2 和 ACK2 给 Client 建立 Connection2
- 等 SYN1 到达 Server 后, Server 会以为是 Client 想要建立 Connection1, 于是返回一个 ACK1 给 Client, 在 Server 视角会认为已经建立了两个连接
- Client 接受到 SYN1 和 ACK1 后, 会判断该连接失败, 在 Client 视角会认为只建立一个连接

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202402151451792.png)

## 4-Way Wavehand

TCP 的四次挥手是指在 TCP 连接中, Client 和 Server 中任意一方在数据传输结束后, 想要结束连接, 就务器必须要做的四个动作

- Client 发送 FIN 给 Server, 表示要关闭连接, Client 切换状态为 FIN_WAIT_1
- Server 接受到 FIN 后, 返回 ACK 给 Client, Server 切换状态为 CLOSE_WAIT, 在这个过程中, Server 还可以发送数据给 Client, Server 发送完数据后, 发送 FIN 给 Client, Server 切换状态为 LAST_ACK
- Client 接受到 FIN 后, 返回 ACK 给 Server, Client 切换状态为 TIME_WAIT, 在这过程中, Client 会进入等待, 防止 Server 没有接受到刚才的 ACK 重新发送 FIN 要求结束, Client 等待两倍的 MSL 的时间后没有重新接受到 FIN, 就会切换状态为 CLOSED
- Server 接受到 ACK 后, 直接切换状态为 CLOSED

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202402151515308.png)

## SYN, FIN, ACK

SYN (Synchronize), FIN (Finish) 和 ACK (Acknowledgment) 是 TCP 用于建立连接和确认数据包成功到达的标志位, 本质上 TCP 的数据包 Segment 在二进制位上做一些标识来标识的

SYN 和 FIN 会在 Sequence Number 中存储一个随机的数字 x, Node1 发送给 Node2 后, Node2 需要返回一个 ACK, ACK 会在 Acknowledgment Number 中存储 x+1, Node1 接受到 Segment 后, 去判断是否返回正确, 从而实现一次握手

- Sequence Number 可以处理乱序问题, Sequence Number 是顺序递增的, 可以确认 Segment 的顺序
- Sequence Number 可以处理丢包问题, 一个数据是被切分成多个数据包的, 如果因为网络问题丢包了, 就可以通过 Sequence Number 确认少的是哪一个包, 重新向对方申请这个包即可

## SYN Flood

SYN Flood 是一种常见的 DoS 攻击, 它通过 Client 大量伪造 IP 发送 SYN 给 Server，Server 需要返回 SYN 和 ACK 给 Client, 而 Client 又不回复 Server, 就会导致 Server 有大量的连接处于 SYN_RCVD 状态, 这些 Half-Open Connection 会一直占用 Half-Open Connection Queue, 当 Queue 满后, 就无法处理正常的请求连接了

## TFO

TFO (TCP Fast Open) 是对 TCP 的一种简化握手流程的拓展, 用于提高两端点间连接的打开速度

- Client 发送 SYN 给 Server, 通过一个标识符表示要开启 Fast Open
- Server 接受到 SYN 后, 不仅返回 SYN 和 ACK, 还会返回一个 Cookie 给 Client
- Client 接受到 SYN 和 ACK 后, 返回 ACK 给 Server 完成三次握手后, 存储 Cookie 到本地, 后续请求都携带这个 Cookie, Server 只需要校验 Cookie 是否正确, 既可以直接进入连接状态, 不需要三次握手了

TFO 在后续的每次请求中都会少一个 RTT, 效率提升很多

TFO 能有效解决 SYN Flood, Server 接受到 SYN 后, 会去检查是否携带 Cookie, 如果没有 Cookie 就不会再进行后续的三次握手了

TFO 存在一定安全风险, TFO 默认不会对结果的 TCP 连接提供任何形式的加密保护, 也不对端点进行身份保证, 如果需要这类防护, 可以与 TLS 或 IPsec 这样的加密协议组合使用

## MSL

MSL (Maximum Segment Life) 指的是网络数据包 (Segment) 在网络中可能存在的最长寿命, MSL 定义了一个数据包从发送到不能再被使用也就是丢弃所需的时间, 这是为了防止网络中的数据包过多导致网络拥塞, 数据包超过 MSL 时限后, 将被自动丢弃

TCP 中, MSL 常常关联到 TIME_WAIT 状态, 当一个 TCP 连接关闭后, 为了保证确保最后的 ACK 确认包能够成功送达, 或是为了避免已经关闭的连接的延迟数据包在新的连接中被错误处理, TCP 会进入一种叫做 TIME_WAIT 的状态, 并等待 2 倍的 MSL 时长, 这样设定的原因是要让网络中该连接两端可能存在的任何数据包都在网络中消失

## OSI

OSI (开放式系统互联参考模型) 的七层模型是一个把网络通信协议分为七个层次的标准模型, 其目的是为了让计算机网络的设计和管理更加灵活和模块化

OSI 的每层都有自己的独立功能和责任, 这种分层的方式使得每个层次都可以独立工作, 同时还能够很好地协调上下

- 应用层: 主要提供各种服务和应用程序 (eg: 电子邮件, 文件传输, 远程登录, Web 浏览), 应用层服务可以 使用不同的协议实现 (eg: HTTP, SMTP, FTP, TELNET)
- 表示层: 主要负责数据格式转换, 加密解密, 压缩解压等服务, 表示层使得应用程序可以使用不同的数据格式和编码, 同时还提供了数据的安全性和完整性保护等服务
- 会话层: 主要负责建立, 管理和终止会话, 提供会话控制和同步等服务, 会话层还负责处理多个应用程序之间的数据交换
- 传输层: 主要负责数据传输的可靠性和流量控制等, 同时还包括分段, 组装, 连接建立和断开等功能
  - 应用: 路由器
  - TCP, UDP
- 网络层: 主要负责数据在网络中的传输, 包括路由选择, 分组转发, 数据报文的封装等, 网络层还处理数据包的寻址和控制流量等
  - 传输的数据单位: IP 数据包
- 数据链路层: 主要负责把数据分成数据帧进行传输, 并对错误进行检测和纠正, 数据链路层还负责物理地址的分配、数据流量控制、错误校验等
  - 应用: 交换机, PPP, HDLC
  - 传输的数据单位: MAC 数据帧
- 物理层: 主要负责通过物理媒介传输比特流，如电缆、光纤、无线电波等。物理层规定了物理连接的规范，包括电缆的类型, 接口的规范
  - 应用: 网卡

![](https://note-sun.oss-cn-shanghai.aliyuncs.com/image/202403041351618.png)

## TCP

OSI 七层模型是一种理论模型, 在实际应用中较少使用, 现在比较常用的是 TCP/IP 四层模型

- 应用层
  - 应用: WWW, HTTP, SMTP, POP3, FTP, Telnet, DHCP
- 传输层
  - 应用: TCP, UDP
- 网络互联层
  - 应用: 路由器, IP, ARP, RARP, ICMP
  - 传输的数据单位: IP 数据包
- 网络接口和硬件层
  - 应用: 交换机, 中继器, 集线器, 网卡
  - 传输的数据单位: MAC 数据帧

OSI 和 TCP/IP 的对应关系

- OSI 的 应用层, 表示层, 会话层 对应 TCP/IP 的 应用层
- OSI 的 传输层 对应 TCP/IP 的 传输层
- OSI 的 网络层 对应 TCP/IP 的 网络互联层
- OSI 的 数据链路层, 物理层 对应 TCP/IP 的 网络接口和硬件层

## Symmetric Key Cryptography

Client 通过 Secret Key 对 Plaintext 进行 Encode 得到 Ciphertext, 发送给 Server

Server 通过 Secret Key 对 Ciphertext 进行 Decode 得到 Plaintext

Algo 必须内置在 Client 和 Server 中, 而 Nginx 又是 Open Source, 所以 Algo 也是公开的, 无法保障安全, Secret Key 需要通过 Network 传输, 也不安全

## Asymmetric Key Cryptography

Client 访问 Server, Server 生成 Public Key 返回给 Client

Client 通过 Publich Key 进行 Encode, 发送给 Server, Server 通过 Private Key 进行 Decode 得到 Plaintext

Server 通过 Private Key 进行 Encode, 发送给 Client, Client 通过 Private Key 进行 Decode 得到 Plaintext

Public Key 通过 Network 传输, Private Key 保存在 Server 不丢失

Hacker 无法伪造 Client 通过 Publich Key 进行 Decode

Hacker 伪造 Server, 发送 Fake Public Key 给 Client, 让 Client 和 Hacker 交互, Hacker 再去跟 Server 交互, 所以只靠 Asymmetric Key Cryptography 无法保障安全

## Certification

Server 提交 Public Key 给 Ca 认证, Ca 通过 Publich Key 进行 Encode得到 Certification 返回给 Server, Server 返回 Certification 给 Client

Client 通过 Ca 的 Public Key 对 Certification 进行 Decode 得到 Server 的 Public Key, 再通过 Asymmetric Encryption 和 Server 进行交互

Hacker 拦截到 Server 返回的 Certification 后, 可以通过 Public Key 进行 Decode, 但是没有 Private Key 就无法进行 Encode, 再次发送给 Client 后, Client 无法 Decode, 就会校验失败, 解决了 Hacker 伪造 Server 的问题

# Computer

## Thread, Process

进程是正在运行程序的实例, 进程包含多个线程, 每个线程执行不同的任务, 不同的进程使用不同的内存空间, 在当前进程下的所有线程可以共享内存空间

线程是进程内一个相对独立的, 最小执行单元, 线程更轻量, 线程上下文切换成本一般上要比进程上下文切换低, 上下文切换指的是从一个线程切换到另一个线程

# Project

## 断点续传

- 前端调用浏览器的 API 对大文件进行分块, 每次上传分块文件前请求后端查询该分块是否已经存在后端, 如果不存在, 则不上传这一块了
- 后端接受到分块后, 保存在 service 本地, 如果出现异常, 就返回该分块的序号给前端, 通知他重新上传
- 前端全部上传完之后, 再发送一个请求给后端, 后端将保存在本地的分块文件进行合并, 再上传完整文件到 MINIO 中
  - 也可以直接将分块上传到 MINIO 中, 合并时就直接调用一个 API 通知 MINIO 进行合并
- 完整文件存储形式 `"/" + fileMd5.charAt(0) + "/" + fileMd5.charAt(1) + "/" + fileMd5 + "/" + fileName`
- 分块文件存储形式 `"/" + fileMd5.charAt(0) + "/" + fileMd5.charAt(1) + "/" + fileMd5 + "/" + chunkNo;`
- 服务端记录一个 tmp 文件, 记录当前已经上传的分片序号, 后续合并失败, 就去查看缺少哪个分片序号, 通知前端补发就可以了

## 视频处理

- 上传视频的时候可以指定修改文件编码 (eg: avi -> mp4), 上传之后, 判断是否需要转换编码, 如果要转换, 就存储一个编码消息到 DB 中, 类似于 MQ, 通过 XXL-JOB 代替 MQ 实现相同的效果
- 通过 shardIndex 和 shardTotal 给 XXL-JOB 的服务分配任务 `select * from media_process where id % #{shardTotal} = #{shardIndex} and (status = 1 or status = 3) and fail_count < #{maxFailCount} limit #{maxRecordCount};`
- 通过 ThreadPool + CountDownLatch 处理获取到的 MediaProcessList
- 如果要节省一部分 CPU, 就可以通过增量式的扫表方式, 通过 limit 获取 MediaProcessList

## 短信登录

短信登录：通过 Redis + Aliyun Web Service 实现手机验证码登录功能, 通过 Redis 存储验证码信息.

- Key 2m 过期, 验证的时候如何没有查询到 Key, 就说明验证码过期
- 创建验证码时, 先查询 Redis 中的验证码 Key 是否存在, 如果存在，就说明是频繁申请了
- 校验成功后, 登录成功后, 就删除这个 验证码 Key, 清空验证码统计 Key
- `code:<phone>` 统计一个账户在一段时间内校验的次数, 通过 INCR, DECR 统计验证码, 1h 过期, 超过 3 次, 就拒绝访问

短信登录: 通过 Redis + Web Service + Hash 实现手机验证码登录功能

- Hash 的 key 为 `code:<phone>`, field 为 code, val 为校验次数

## 分布式登录状态同步

分布式登录状态同步：通过 Redis + JWT + Hash 解决分布式环境下登录状态同步的问题.

- 用户登入后, 会存储到一个 Hash 中, Key 为 `token:login:<userid>`, Field 为 Create Time, Val 为 Token, 每次存储 New Token 前会先判断 Hash 存储的 Token 数量是否达到了存储上限, 如果达到了, 就移除 Oldest Token, 并将该 Oldest Token 加入 BlackList
- 用户登出后, 需要删除 Hash 中的 Token, 并且将 Token 加入 BlackList
- 拦截器拒绝该 BlackList 中 Token 的访问

分布式登录状态同步：通过 Redis + JWT + List 解决分布式环境下登录状态同步的问题

- lpush 添加 token, 执行 ltrim 保留 0 ~ 2 的 token, 删除超出的 token
- 每次请求都查询一次 Redis 都 Token list, 对比 token 是否相同
- 也不需要通过 XXL-JOB 来实现周期性删除 Redis 中过期的 Token

分布式登录状态同步: JWT + 黑名单 + 定时调度清理黑名单

- JWT 无法修改过期时间, 所以就无法在用户退出后, 就将移除其登录状态, 所以可以搭配黑名单实现登出

## SSO

SSO: 共享 Cookie 的方式

- 存储 Cookie 时指定存储到相同域名后缀中, 访问多个相同域名后缀的网站时, 都可以携带上这个 Token

SSO: OAuth2 实现

- User 请求 Client A, Client A 发现 User 没有 Token, 就引导 User 携带 Client Id 和 Redirect URL 去请求 Authorization Server, Authorizationi Server 会返回 code 给 User, User 携带 code ...
  - User 登录 Authorization Server 后, 会存储一个 Authorization Server 的 Token 到本地, 方便后续请求 Authorization Server 时, 判断该 User 是否已登陆, 通过 `token:login:auth:<user_agent>:<user_id>` list 进行存储
  - 由于 Authorization Server 的实现很多, 最好是前端请求后端, 后端返回对应 Authorization Server URL
- User 请求 Client B, Client A 发现 User 没有 Token, 就引导 User 携带 Client Id 和 Redirect URL 去请求 Authorization Server, 此时 Authorizationi Server 发现 User 已经登录过了, 就直接返回 code 给 User, 不需要 User 登录, User 携带 ...
- 不同 Client 的权限策略是不同的, 共用同一个 Token, 明显不合适, 所以每次的 code 申请的 Token 都应该不同, 如果一个站点退出了, 就应该清除其他所有的 Token, Token 都使用 Redis 进行存储
- 不同 User Agent 之间的 Token 管理应该相互隔离, 如果一个站点退出了, 也应该退出其他相同 User Agent 的 Token, 而不应该退出其他 User Agent 的 Token, 可以根据 `token:login:<client_id>:<user_agent>:<user_id>` list 进行存储
- 用户修改密码后, 立即清空其他 Token, 或者删除所有 Token, 然后给当前用户再颁发一个新的 Token

## 日活跃统计

日活跃统计: 通过 MQ + Interceptor + Redis 的 HyperLogLog 统计日活跃用户数量

- Interceptor 拦截到请求后, 将 IP 封装成 Msg, 由 MQ 异步处理, 以 `statistic:user:daily:2022:01:01` 的 Key, IP 为 Val 操作 HyperLogLog 统计日活跃用户
- 讲述由 List => Set => BitMap => HyperLogLog 的选择过程

## 缓存击穿处理

缓存击穿处理：通过 Bitmap + Bloom Filter 解决缓存击穿

- Guava -> Hutool -> Redis -> Redisson + Interceptor

## 超时订单

超时订单: 通过 Redis + MQ + Delayed Queue 实现超时订单处理, 通过 XXL-JOB 进行兜底处理.

- 通过 RabbitMQ 的 Delayed Queue 实现超时订单, 分为 10s, 20s, 30s, 40s, 50s, 1m, 5m, 10m 去检查订单状态, 如果支付, 直接退出, 如果超过 10m 也未支付, 则直接恢复库存, 允许其他用户下单
- 通过 XXL-JOB 每隔 10s 进行一次分页查询, 处理一部分超时订单, 总共在 10m 内完成一次全表扫描, 不仅可以兜底, 也可以防止 MQ 消息丢失和 MQ 宕机的问题
- 成本高, 需要额外存储消息, 就需要搭建集群, 如果存储的延迟任务过多, 会导致峰值压力, 适合延时较短的任务

超时订单: 通过 XXL-JOB 实现超时订单

- 单独维护一个超时库, 将所有需要处理超时的任务, 塞入这个库, 通过 XXL-JOB 一次性或者增量式全部捞出来, 然后通过批量修改的方式操作数据库
- 稳定, 成本低, 延时高, 不存在峰值压力, 适合存储延时较长的任务

超时订单: 时间轮算法

- MQ 的延迟消息也有用到时间轮算法

## 在线人数统计

在线人数统计: MySQL 实现

- 登录设置 user tbl 的 is_login 为 1, 登出设置为 0, 统计时直接查询 is_login 为 1 的数量

在线人数统计: Redis 实现

- `sadd uid_login <uid>` 和 `srem uid_login <uid>` 维护一个登录统计缓存, 通过 scard 获取总数量

在线人数统计: BitMap 实现

- `setbit user_login <uid> 1` 和 `setbit user_login <uid> 0` 维护一个位图登录统计, 通过 `bitcount` 统计在线人数
- 通过 `setbit user_ios <uid> 1` 维护一个 ios 用户位图, 通过 `user_ios & user_login` 统计 ios 用户在线人数

在线人数统计: HyperLogLog 实现

## 秒杀商品

秒杀商品: Redis + Lua + Redissson + MQ 实现

[Explain](https://www.bilibili.com/video/BV1Uz4y137UA/?spm_id_from=333.337.search-card.all.click&vd_source=2b0f5d4521fd544614edfc30d4ab38e1)

- 单机锁 -> 分布式锁 (Redis SETNX / Zookeeper)
- 业务执行耗时太久, 就可以适当增加锁的过期时间, 或者开启一个子线程定时检查主线程是否依旧在线处理任务, 如果在就重设过期时间, 续命嘛, watch dog
  - Watch Dog 就是通过时间轮算法实现的, 一般使用 Netty 的 HashedWheelTimer
  - 一般续期都需要先判断是否过期, 然后再去修改过期时间, 是多个操作, 需要保证原子性, 就通过 Lua, 通过递归的方式继续进行续期操作
- Key 设定为当前线程对应的 UUID, 每次删除 key, 只会释放属于自己的锁

秒杀商品: Redis + Lua + Nginx + MQ 实现

- Nginx + Lua + Redis decr 实现订单的预检和订单的扣减, 由于 Redis 执行命令是原子性的, 并且 lua 脚本也是原子性的, 所以非常安全, 在网关层就将无效的请求阻挡了
- 订单扣减完, 通过 lua 发送一条 mq 去异步创建订单, 并且设置 MQ 的 QOS 限制一次处理的请求数量, 这里创建订单是异步创建的, 前端是无法感知的, 所以前端可以每隔一段时间发送一个请求, 检查订单状态, 前端就在那转圈圈就好了, 一般很快就能查询到订单结果
- Redis 扛不住, 就上架构, 主从, 哨兵, 集群来干!!
- 订单取消或者订单超时了, 需要通过 lua 去执行 incr

## Idempotent

Idempotent 要求重复多次调用接口不会改变业务状态, 保证单次调用结果和多次调用结果一致 (eg: 用户重复点击下单, 要求只能有一个订单生效)

- GET 和 DELETE 符合 Idempotent, PUT 和 POST 不符合 Idempotent

通过 Token + Redis 实现 Idempotent, 适合高并发场景, 前端压力大, 性能最强

- 前端生成 Token 连着表单一块传递给后端, 后端通过 SETNX + TTL 处理 Token. 如果操作失败, 就直接报错或者返回空结果. 如果操作成功, 就执行后续业务逻辑
- 这种方法只能防止 RPC 或 MQ 的重试导致的非幂等问题, 而用户重复点击提交, 造成的 Repeated Request 就无法解决了, 一般需要搭配前端的按钮置灰进行处理

```js
import axios from 'axios'; 
import { nanoid } from 'nanoid';

const token = nanoid();
const formData = {};

axios({
    method: 'post',
    url: '/your-endpoint',
    data: formData,
    headers: {
        'X-Token': token,
    },
}).then((response) => {
    console.log(response.data);
}).catch((error) => {
    console.log(error);
});
```

```java
@Aspect
@Component
public class IdempotentAspect {
    @Autowired
    private RedisTemplate redisTemplate;

    @Around("@annotation(Idempotent)")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        String token = request.getHeader("X-Token");
        if (StringUtils.isBlank(token)) {
            throw new RuntimeException("Token does not exists");
        }
        if (!redisTemplate.setIfAbsent(token, "", 60, TimeUnit.SECONDS)) {
            throw new RuntimeException("Repetitive operation");
        }
        Object result = point.proceed();
        return result;
    }
}
```

通过 Token + Redis 实现 Idempotent, 适合高并发场景, 前端压力小, 但是资源耗费多

- 进入表单提交页面时, 就先向后端申请一个 Token, 前端提交表单时携带上 Token, 后端通过 SETNX + TTL 或者 DEL 来处理 Token. 如果操作失败, 就直接报错或者返回空结果. 如果操作成功, 就执行后续业务逻辑
- 这种方法可以解决所有幂等问题, 也不需要担心用户重复点击提交导致的非幂等问题, 因为在进入表单提交页面时, 就确定了 Token

通过 Business 实现 Idempotent, 不适合高并发场景

- 后端执行业务前先查询 Status, 判断是否已经操作过

通过 MessageId + Redis 解决 MQ 的 Idempotent, 专用于保证 MQ 的 Idempotent, 效率高, 性能强

- RabbitMQ 发送的消息默认不会生成 Message Id, 需要配置 MessageConverter 生成 Message Id
- RabbitMQ 的每条 Message 都可以设置 Message Id, 将 Message Id 作为 Token, 通过 SETNX + TTL 处理 Token 实现 Idempotent.

```java
@Bean
public MessageConverter messageConverter() {
    Jackson2JsonMessageConverter jackson2JsonMessageConverter = new Jackson2JsonMessageConverter();
    jackson2JsonMessageConverter.setCreateMessageIds(true);
    return jackson2JsonMessageConverter;
}
```

```java
@Autowired
RedisTemplate redisTemplate;

@RabbitListener(queues = {"direct.queue"})
public void listener(Message message) {
    // Store message into Redis, key is message id, val is message
    Boolean isAbsent = redisTemplate.opsForValue().setIfAbsent(
        message.getMessageProperties().getMessageId(), 
        new String(message.getBody()), 
        60, TimeUnit.SECONDS);

    if (!isAbsent) {
        return;
    }
    
    System.out.println("deal non-idempotent business");
}
```

## Log

使用统计的日志框架记录日志, 而不是使用自定义的接口, 系统日志切换时, 可以很迅速

通过异步的方式记录日志, 基于 Disruptor 高性能队列记录日志

使用占位符记录日志, 性能和使用便捷性上都要优于使用字符串拼接记录日志

- 当使用占位符记录日志时, 只会在实际需要记录该日志级别的消息时发生, 如果日志级别配置得足够高, 使得当前的日志消息不会被记录, 那么消息字符串的构建过程就会被完全跳过, 这种延迟构建的机制可以节省不必要的字符串创建和拼接开销, 尤其是当日志内容包含了复杂的表达式或方法调用时

所有的日志保存 15 天, 敏感数据保存 6 个月 (eg: 金钱的更改, 金钱的提现)

少使用 error, 多使用 warn

日志记录 Class#Method, 方便定位

灵活切换日志级别, 避免无用日志

方法调用前打印日志, 确定入参, 方法调用后打印日志, 确定返回值