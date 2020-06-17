# Java基础

## 代码优化建议

### 1.尽量少的创建标志位，多标志位不好管控
### 2.空指针问题要找到源头，不能直接做判空操作
### 3.某些判断中进入return后，要考虑不进该判断条件时后续有无释放操作#### 4.解耦 精简模块对外接口，减少模块间的耦合

## 一、容器
### 1.**for循环中为什么list、map不能作remove操作**
执行以下代码，会抛出异常java.util.ConcurrentModificationException
```java
List<String> a = new ArrayList<>();
a.add("1");
a.add("2");
for (String temp : a) {
    System.out.println(temp);
    if("2".equals(temp)){
        a.remove(temp);
    }
}
```
&emsp;&emsp;根据反编译的代码，在执行for循环时，其实是先调用了ArrayList中的iterator方法得到一个Iterator，然后每次循环时，判断Iterator.hasNext，如果hasNext(根据cursor != size判断)，则调用Iterator.next，此时就会判断List是否被意外修改(checkForComodification)，if (modCount != expectedModCount)则抛出ConcurrentModificationException。如果remove的cursor刚好为size-2，即倒数第二个对象，下次判断hasNext时，cursor == size了，就返回false，认为没有后面的元素了，不会再调用Iterator.next，因此也不会抛出异常，但是也不会把最后一个没有遍历过的元素拿出来进行业务处理。

### 2.**set如何单行初始化**
```java 
//此方式效率低，构建数组->转换为列表->使用列表来创建集合
Set<String> h = new HashSet<>(Arrays.asList("a", "b"));
//此方式相对高效
public static final String[] SET_VALUES = new String[] { "a", "b" };
public static final Set<String> MY_SET = new HashSet<>(Arrays.asList(SET_VALUES));
```
### 3.HashMap
来自公众号 【安琪拉的博客】
#### 3.1 HashMap插入元素
1. 判断数组是否为空，若数组为空，则进行初始化
```java
   if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
```
2. 若数组不为空，则计算key的hash值，通过```(n-1)&hash```计算应该存放到数组中的下标index

3. 查看table[index]是否存在数据，如果没有数据，则新建一个Node节点存放到table[index]
```java
  if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
```
4. 存在数据，说明发生了hash冲突，继续判断key是否相等，若相等，则用新value覆盖旧value
```java
 if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))
    e = p;
```
5. 如果不相等，判断当前节点是不是树型节点，如果是树形节点，创建树型节点插入红黑树中
```java
  else if (p instanceof TreeNode)
    e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
```
6. 如果不是树型节点，创建普通Node加入链表中；判断链表长度是否大于8，大于8的话链表转换为红黑树
```java
 for (int binCount = 0; ; ++binCount) {
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            treeifyBin(tab, hash);
        break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        break;
    p = e;
}
```
7. 插入完成判断当前节点数是否大于阈值，如果大于阈值，则扩容为初始容量的2倍
```java
 if (++size > threshold)
    resize();
```

#### 3.2 如何确定HashMap的初始化大小
- 如果new HashMap()不传值，默认大小为16，负载因子是0.75
```java
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```
- 如果传入初始值为k，初始化大小为大于k的2的整数次方，比如传入值为10，则大小为16
```java
  //初始二进制分别右移1，2，4，8，16位，与自己异或，把高位第一个为1的数通过不断右移，把高位为1右边全部变为1，最后返再+1，使得最高位为1的高一位为1，如看为50，右移异或后值为111111，111111 + 1 = 2^6 = 64即为初始容量大小
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
#### 3.3 HashMap的hash函数如何设计的
>哈希函数通过先拿到key的hashcode（32位的int值），然后让hashcode的高16位于低16位进行异或操作
##### 3.3.1 为何如此设计hash函数
这个也叫做扰动函数，如此设计的原因有两点：
1. 尽可能的降低hash碰撞，越分散越好
2. 算法一定要尽可能的高效，因为是高频操作，所以采用位运算
##### 3.3.2 为什么使用hashcode的高16位和低16位异或能降低hash碰撞，hash函数能不能直接用key的hashcode
&emsp;&emsp;key的hashcode()调用的是key键值类型自带的哈希函数，返回int的散列值，int值的范围为-20多亿~20多亿，一共40多亿个地址映射空间，只要哈希函数映射的比较松散，很难出现hash碰撞。  
&emsp;&emsp;但40亿长度的数组，内存放不下，数组index值需要对数组长度取模，得到余数才能访问数组下标，源码中的模运算是把散列值和数组长度-1做了"与"操作，位运算比%速度快。
```java
bucketIndex = indexFor(hash,table.length);
static int indexFor(int h,int length) {
    return h & (length - 1)
}
```
&emsp;&emsp;这也解释了HashMap长度要取2的整数次幂，因为这样（数组长度 - 1）正好相当于一个 “低位掩码”。“与”操作的结果就是散列值的高位全部归零，只保留低位值来做数组下标访问。  **(n-1)&hash**  
&emsp;&emsp;以初始长度16为例，16 - 1 = 15，二进制表示为00000000 00000000 00001111，与散列值做与操作:
> &nbsp;&nbsp;&nbsp;10100101 11000100 00100101  
&00000000 00000000 00001111   
=00000000 00000000 00000101

&emsp;&emsp;但是这样的方式有些问题，就算散列值分布的再松散，如果只取最后几位，也难免会发生很多hash碰撞，更重要的是，如果散列本身做的不好，分布成等差数列漏洞，如果正好让最后几个低位呈现规律性重复，那就非常难受了。  
&emsp;&emsp;这时候“扰动函数”的作用就体现出来了，高半区右移16位，高半区与低半区做异或操作，混合原始哈希码的高位和低位，由此来加大低位的随机性，而且混合后低位也掺杂了部分高位特性，变相保留了高位的信息。
##### 3.3.3 JDK1.8相比JDK1.7还做了哪些优化
1. 数组+链表 -> 数组+链表+红黑树  
防止在发生hash冲突后，链表长度过长，将时间复杂度由O(n)优化为O(logn)
2. 链表的插入方式从头插法 -> 尾插法  
多线程环境下，头插法可能会产生环
3. 扩容时，JDK1.7需要懂数组中元素重新定位在新数组中位置，JDK1.8采用更简单的判断逻辑，位置不变或索引+旧容量大小  
**为何1.8不重新hash就能得到扩容后原节点在新数据中的位置？**  
&emsp;&emsp;扩容时会扩容为原数组大小的两倍，用于计算数组位置的掩码只是高位多了一个1，举例来说，扩容前长度为16，用于计算```(n-1)&hash```的二进制```n-1```为0000 1111，扩容为32后，用于进行hash计算的```n-1```为0001 1111。  
&emsp;&emsp;因为是&运算，1和任何数&都是它本身，分两种情况，元数据hashcode高位第4位为0和高位为1的情况:第4位高位为0，重新hash数值不变；第4位高位为1，重新hash数值比原来大16（旧数组的容量）
4. 插入时。JDK1.7先判断是否需要扩容，再进行插入；JDK1.8先进行插入，插入完成再判断是否需要扩容

##### 3.3.4 HashMap不是线程安全的，怎么解决这个问题？
&emsp;&emsp;JDK1.7中，会产生死循环、数据丢失、数据覆盖问题；JDK1.8中，会有数据覆盖问题  
&emsp;&emsp;HashTable、Collections.synchronizedMap、ConcurrentHashMap可以实现线程安全的Map 
1. HashTable直接在操作方法上加上synchronized关键字，锁住整个数组，粒度较大；
2. Collections.synchronizedMap是使用Collections集合工具的内部类，通过传入Map封装出一个SynchronizedMap对象，内部定义了一个对象锁，方法内通过对象锁实现
3. ConcurrentHashMap使用分段锁，降低了锁粒度，大大提高了并发度  
##### 3.3.5 ConcurrentHashMap分段锁实现原理
成员变量使用volatiole修饰，免除了指令重排序，同时保证了内存可见性，另外使用CAS操作和synchronezed结合实现赋值操作，多线程操作只会锁住当前操作索引的节点
## 二、反射
>&emsp;&emsp;反射机制运行时，能够获得一个类的所有属性和方法；对于任意一个对象，能够调用它的任意一个属性或方法。这种能够动态获取信息以及动态调用对象的方法，叫做java的反射机制。  
&emsp;&emsp;反射机制的重点在**运行时**，该机制使得我们可以在程序运行时加载、探索、以及使用编译期间生成的.class文件。换句话说，java程序可以加载一个运行时才得知名称的.class文件，获悉其完整构造，并生成对象实体，可对其变量(fields)赋值，也可调用其中的方法(methods)。  

https://juejin.im/post/598ea9116fb9a03c335a99a4
### 2.1 获取变量
```getDeclaredFields()```获取所有public的变量（包括从父类继承的）  
```getDeclaredFields()```获取所有本类声明的所有权限的变量

```java
    //1.获取并输出类的名称
    Class mClass = SonClass.class;
    //2.1 获取所有 public 访问权限的变量 包括本类声明的和从父类继承的
    Field[] fields = mClass.getFields();
    //2.2 获取所有本类声明的变量（不问访问权限）
    Field[] fields = mClass.getDeclaredFields();
```
#### 2.1.1 获取私有变量
```getDeclaredField``` 获取私有变量  
```setAccessible(true)``` 获取私有变量的访问权  
调用 ```set(object , value)``` 修改变量的值
# 2.2 获取方法
```getMethods()```获取所有public的方法（包括从父类继承的）  
```getDeclaredMethods()```获取所有本类声明的所有权限的方法

```java
    //1.获取并输出类的名称
    Class mClass = SonClass.class;
    //2.1 获取所有 public 访问权限的方法 包括自己声明和从父类继承的
    Method[] mMethods = mClass.getMethods();
    //2.2 获取所有本类的的方法（不问访问权限）
    Method[] mMethods = mClass.getDeclaredMethods();
```
#### 2.2.1 获取私有方法
```setAccessible(true)```  获取私有方法的访问权  
```invoke()``` 使用 invoke 反射调用私有方法 

```java
public class TestClass {
    private String MSG = "Original";
    private void privateMethod(String head , int tail){
        System.out.print(head + tail);
    }
    public String getMsg(){
        return MSG;
    }
}

    //1. 获取 Class 类实例
    TestClass testClass = new TestClass();
    //2. 获取私有方法
    //第一个参数为要获取的私有方法的名称
    //第二个为要获取方法的参数的类型，参数为 Class...，没有参数就是null
    //方法参数也可这么写 ：new Class[]{String.class , int.class}
    Method privateMethod = mClass.getDeclaredMethod("privateMethod", String.class, int.class);
    //获取私有方法的访问权 只是获取访问权，并不是修改实际权限
    privateMethod.setAccessible(true);
    //使用 invoke 反射调用私有方法 privateMethod 获取到的私有方法;testClass 要操作的对象;后面两个参数传实参
    privateMethod.invoke(testClass, "Java Reflect ", 666);

```
### 2.3 修改私有常量
>&emsp;&emsp;常量使用final修饰  
&emsp;&emsp;Java虚拟机在编译.java文件得到.class文件时，会优化代码提升效率。其中一个优化就是JVM在编译阶段会把引用常量的代码换成具体的常量值。  
&emsp;&emsp;**对于基本类型的静态常量，JVM在编一阶段会引用此常量代码替换为具体常量值**  
&emsp;&emsp;**基本类型**  字符类型char；布尔类型boolean；数值类型byte、short、int、long、float、double

在构造函数中初始化常量值或者使用三目表达式来对常量赋值，都可以**避免在编译时被优化**，从而使得修改常量生




# JVM
## 一、JVM在编译期所做的优化
&emsp;&emsp;Java编译器是指一段操作过程，具体可以分为三类：  
- Javac 前端静态编译器，将*.java编译为*.class文件
- JIT 后端运行编译器，将*.class转变成机器码的过程
- AOT 静态提前编译器，把*.java文件转换为本地机器码的过程

>源文件、字节码、机器码、本地代码都是啥？  
- 源文件是*.java文件
- 字节码是*.class文件
- 机器码和本地代码是计算机能够直接识别的代码，即机器指令

### 1. Javac编译器 
学习链接：https://juejin.im/post/5b9fa2e5f265da0ad2217f84  
&emsp;&emsp;一般情况下，*.java文件会被编译为 *.class字节码文件，再由JVM翻译为计算机可执行的文件。javac的作用是将java源代码转换为class字节码文件。

### 2. JIT编译器
https://juejin.im/post/5c8112d66fb9a049e4137cf9
### 3.  AOT编译器
