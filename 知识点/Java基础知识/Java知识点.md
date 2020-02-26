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
