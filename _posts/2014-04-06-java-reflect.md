---
layout: post
keywords: java, reflect
description: java reflect
title: "Java的反射机制"
categories: [Java]
tags: [java, reflect]
group: Java
icon: file-alt
---
{% include JB/setup %}

在Java 运行时 环境中，对于任意一个类，能否知道这个类有哪些属性和方法？对于任意一个对象，能否调用他的方法？这些答案是肯定的，这种动态获取类的信息，以及动态调用类的方法的功能来源于JAVA的反射。从而使java具有动态语言的特性。

***
#JAVA反射机制主要提供了以下功能：

    1.在运行时判断任意一个对象所属的类
    2.在运行时构造任意一个类的对象
    3.在运行时访问任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）
    4.在运行时调用任意一个对象的方法（注意：前提都是在运行时，而不是在编译时）

***
#Java 反射相关的API简介：

与反射有关的所有接口以及类都在java.lang.reflect包里。

###接口
<table>
    <tr>接口摘要</tr>
    <tr>
        <td>AnnotatedElement</td>
        <td>表示目前正在此 VM 中运行的程序的一个已注释元素。</td>
    </tr>
    <tr>
        <td>GenericArrayType</td>
        <td>GenericArrayType 表示一种数组类型，其组件类型为参数化类型或类型变量。</td>
    </tr>
    <tr>
        <td>GenericDeclaration</td>
        <td>声明类型变量的所有实体的公共接口。</td>
    </tr>
    <tr>
        <td>InvocationHandler</td>
        <td>InvocationHandler 是代理实例的调用处理程序 实现的接口。</td>
    </tr>
    <tr>
        <td>Member</td>
        <td>成员是一种接口，反映有关单个成员（字段或方法）或构造方法的标识信息。</td>
    </tr>
    <tr>
        <td>ParameterizedType</td>
        <td>ParameterizedType 表示参数化类型，如 Collection&lt;String&gt;。</td>
    </tr>
    <tr>
        <td>Type</td>
        <td>Type 是 Java 编程语言中所有类型的公共高级接口。</td>
    </tr>
    <tr>
        <td>TypeVariable&lt;D extends GenericDeclaration&gt;</td>
        <td>TypeVariable 是各种类型变量的公共高级接口。</td>
    </tr>
    <tr>
        <td>WildcardType</td>
        <td>WildcardType 表示一个通配符类型表达式，如 ?、? extends Number 或 ? super Integer。</td>
    </tr>
</table>

###类
<table>
    <tr>类摘要</tr>
    <tr>
        <td>AccessibleObject</td>
        <td>AccessibleObject 类是 Field、Method 和 Constructor 对象的基类。</td>
    </tr>
    <tr>
        <td>Array</td>
        <td>Array 类提供了动态创建和访问 Java 数组的方法。</td>
    </tr>
    <tr>
        <td>Constructor&lt;T&gt;</td>
        <td>Constructor 提供关于类的单个构造方法的信息以及对它的访问权限。</td>
    </tr>
    <tr>
        <td>Field</td>
        <td>Field 提供有关类或接口的单个字段的信息，以及对它的动态访问权限。</td>
    </tr>
    <tr>
        <td>Method</td>
        <td>Method 提供关于类或接口上单独某个方法（以及如何访问该方法）的信息。</td>
    </tr>
    <tr>
        <td>Modifier</td>
        <td>Modifier 类提供了 static 方法和常量，对类和成员访问修饰符进行解码。</td>
    </tr>
    <tr>
        <td>Proxy</td>
        <td>Proxy 提供用于创建动态代理类和实例的静态方法，它还是由这些方法创建的所有动态代理类的超类。</td>
    </tr>
    <tr>
        <td>ReflectPermission</td>
        <td>反射操作的 Permission 类。</td>
    </tr>
</table>

<!--excerpt-->

***
#Class类

在 java 的Object类中的申明了数个应该在所有的java类中被改写的methods：hashCode(), equals(),clone(),toString(),getClass()等，其中的getClass()返回一个Class 类型的对象。

Class类十分的特殊，它和其它的类一样继承自Object，其实体用以表达java程序运行时的 class和 interface，也用来表达 enum，array，primitive，Java Types 以及关键字void，当加载一个类，或者当加载器(class loader)的defineClass()被JVM调用，便产生一个Class对象，Class是Reflection起源，针对任何你想探勘的class（类），唯有先为它产生一个Class的对象，接下来才能经由该对象唤起相应的反射API。

Java允许我们从多种途径为一个类class生成对应的Class对象。

    class TestClass {
        private int a;
        public String b;
        
        public TestClass() {
            a = 1;
            b = "b";
        }
        
        public TestClass(int a, String b) {
            this.a = a;
            this.b = b;
        }
        
        private void fun1() {
            System.out.println("fun1");
        }
        
        public void fun2() {
            System.out.println("fun2");
        }
        
        @Override
        public String toString() {
            return "a:[" + a + "] b:[" + b + "]";
        }
    }

1.运用 getClass（）：Object类中的方法，每个类都拥有此方法
 
    String str="abc";
    Class cl=str.getClass();

2.运用 Class.getSuperclass（）：Class类中的方法，返回该Class的父类的Class

3.运用 Class.forName（）静态方法：

    Class.forName("java.lang.String");
    Class.forName("com.javatest.TestClass");

4.运用 类名.class

    int.class
    String.class
    TestClass.class

5.运用primitive wrapper classes的TYPE语法： 基本类型包装类的TYPE，如：Integer.TYPE
注意：TYPE的使用，只适合原生（基本）数据类型

***
#运行时生成Class的实例

运行时通过反射机制生成Class的实例，有两种方法，

    Class.newInstance()//只能使用无参数的构造函数
    Constructor.newInstance()//可以传入参数，调用有参构造函数
    
只有这两个类拥有newInstance()方法，分别是Class类和Constructor类。其中Class类中的newInstance()方法是不带参数的，而Constructro类中的newInstance()方法是带参数的，需要提供必要的参数。

如果想调用带参数的构造方法，不能直接调用Class类中的newInstance()，而是调用Constructor类中newInstance()方法：

首先准备一个Class[]作为Constructor的参数类型。然后调用该Class对象的getConstructor()方法获得一个与参数匹配的Constructor的对象，最后再准备一个Object[]作为Constructor对象的newInstance()方法的实参。

例:

    Class c = Class.forName("com.javatest.TestClass");
    //无参构造函数
    Object object1 = c.newInstance();
    System.out.println(object1);//a:[1] b:[b]
    //有参构造函数
    Class[] ptype=new Class[]{int.class, String.class};
    Constructor ctor=c.getConstructor(ptype);
    Object[] obj=new Object[]{new Integer(3),new String("abc")};
    Object object2=ctor.newInstance(obj);
    System.out.println(object2);//a:[3] b:[abc]

***
#运行时调用Method

首先准备一个Class[]{}作为getMethod(String name，Class[])方法的参数类型，接下来准备一个Obeject[]放置自变量，然后调用Method对象的invoke(Object obj，Object[])方法。

###获取Method

1. public Method getMethod(String name, Class&lt;?&gt;... parameterTypes);

name是方法名称；parameterTypes是Class对象的数组，标识了参数的类型，顺序与方法中参数的声明顺序一致，为null代表无参方法。方法的搜索顺序：class -> super class -> super interface。Java语言不允许声明参数相同、返回值类型不同的多个方法，但是JVM却没有这个限制，因此搜索结果可能有多个。如果name为&lt;init&gt;或者&lt;clinit&gt;，或者方法没有找到，则抛出*NoSuchMethodException*。
搜索不到，

2. public Method[] getMethods();

返回所有public的方法的数组，包括类本身的方法以及其实现的接口的方法。不排序。

3. public Method getDeclaredMethod(String name, Class&lt;?&gt;... parameterTypes);

用法同getMethod，但是可以访问包括private、default(package)和protected的方法。注意，如果需要调用private的方法，需要先调用Method.setAccessible(true)，使得私有方法可访问，否则会抛出IllegalAccessException异常。

4. public Method[] getDeclaredMethods();

用法同getMethods，返回包括public、protected、package和private的所有方法。

###示例

    Class c = Class.forName("com.yc.javatest.TestClass");
    Object object1 = c.newInstance();
    //访问private方法
    Method m1 = c.getDeclaredMethod("fun1", null);
    m1.setAccessible(true);
    m1.invoke(object1, null);
    //访问public方法
    Method m2 = c.getDeclaredMethod("fun1", null);
    m2.invoke(object1, null);

***
#运行时调用Field内容

变更Field不需要参数和自变量，首先调用Class的getField()并指定field名称，获得特定的Field对象后便可以直接调用Field的 get(Object obj)和set(Object obj,Object value)方法

###获取Field

1. public Field getField(String name);

根据给定的field的名称来反射该类或给类实现的接口的public的成员变量。查找的顺序是：class -> interface -> super class。查找不到则抛出NoSuchFieldException异常。

    Class c = TestClass.newInstance();
    Field f = c.getField("member1");

2. public Field[] getFields();

返回Class或Class实现的接口的所有可访问成员的数组，数组不排序。没有则返回长度为0的数组。

3. public Field getDeclaredField(String name);

用法同getField，可以访问private修饰符修饰的field。同getDeclaredMethod一样，访问private的field，调用Field.setAccessible(true)方法来修改field的访问权限，否则会抛出IllegalAccessException异常。

4. public Field[] getDeclaredFields();

用法同getFields()，可以获取所有类型的field。

###访问Field

对field的访问可用getxxx(Object)方法和setxxx(Object, Object)方法。其中，引用类型使用:

    public Object get(Object obj);
    public void set(Object obj, Object value);

八个基本类型使用：

    public boolean getBoolean(Object obj);
    public byte getByte(Object obj);
    public char getChar(Object obj);
    public short getShort(Object obj);
    public int getInt(Object obj);
    public long getLong(Object obj);
    public float getFloat(Object obj);
    public double getDouble(Object obj);
    
    public void setBoolean(Object obj, boolean z);
    public void setByte(Object obj, byte b);
    public void setChar(Object obj, char c);
    public void setShort(Object obj, short s);
    public void setInt(Object obj, int i);
    public void setLong(Object obj, long l);
    public void setFloat(Object obj, float f);
    public void setDouble(Object obj, double d);

###示例

    Field f1 = c.getDeclaredField("a");
    f1.setAccessible(true);
    Field f2 = c.getField("b");
    System.out.println(object1);
    f1.set(object1, 2);
    f2.set(object1, "abc");
    System.out.println(object1);

***
#DEMO

    package cn.com.reflection;   

    import java.lang.reflect.Field;   
    import java.lang.reflect.InvocationTargetException;   
    import java.lang.reflect.Method;   

    public class ReflectTester {   
        /**  
         * 在这个类里面存在有copy（）方法，根据指定的方法的参数去 构造一个新的对象的拷贝  
         * 并将他返回
         */  
        public Object copy(Object obj) throws IllegalArgumentException, SecurityException, InstantiationException, IllegalAccessException, InvocationTargetException, NoSuchMethodException{   

            //获得对象的类型   
            Class classType=obj.getClass();   
            System.out.println("该对象的类型是："+classType.toString());   

            //通过默认构造方法去创建一个新的对象，getConstructor的视其参数决定调用哪个构造方法   
            Object objectCopy=classType.getConstructor(new Class[]{}).newInstance(new Object[]{});   

            //获得对象的所有属性,包括private的
            Field[] fields=classType.getDeclaredFields();   

            for(int i=0;i<fields.length;i++)
                //获取数组中对应的属性   
                Field field=fields[i];   

                String fieldName=field.getName();   
                String stringLetter=fieldName.substring(0, 1).toUpperCase();   

                //获得相应属性的getXXX和setXXX方法名称   
                String getName="get"+stringLetter+fieldName.substring(1);   
                String setName="set"+stringLetter+fieldName.substring(1);   

                //获取相应的方法   
                Method getMethod=classType.getMethod(getName, new Class[]{});   
                Method setMethod=classType.getMethod(setName, new Class[]{field.getType()});   

                //调用源对象的getXXX（）方法   
                Object value=getMethod.invoke(obj, new Object[]{});   
                System.out.println(fieldName+" :"+value);   

                //调用拷贝对象的setXXX（）方法   
                setMethod.invoke(objectCopy,new Object[]{value});    
            }
            return objectCopy;
        }   

        public static void main(String[] args) throws IllegalArgumentException, SecurityException, InstantiationException, IllegalAccessException, InvocationTargetException, NoSuchMethodException {   
            Customer customer=new Customer();   
            customer.setName("hejianjie");   
            customer.setId(new Long(1234));   
            customer.setAge(19);   

            Customer customer2=null;   
            customer2=(Customer)new ReflectTester().copy(customer);   
            System.out.println(customer.getName()+" "+customer2.getAge()+" "+customer2.getId());   

            System.out.println(customer);   
            System.out.println(customer2);
        }
    }   
  
  
    class Customer{   

        private Long id;
        private String name;
        private int age;   

        public Customer(){   
               
        }   

        public int getAge() {   
            return age;   
        }   

        public void setAge(int age) {   
            this.age = age;   
        }   

        public Long getId() {   
            return id;   
        }   

        public void setId(Long id) {   
            this.id = id;   
        }   

        public String getName() {   
            return name;   
        }   

        public void setName(String name) {   
            this.name = name;   
        }   
    }  


    package cn.com.reflection;   

    import java.lang.reflect.Array;   

    public class ArrayTester1 {   

        /**  
         * 此类根据反射来创建  
         * 一个动态的数组   
         */  
        public static void main(String[] args) throws ClassNotFoundException {   

            Class classType=Class.forName("java.lang.String");   

            Object array= Array.newInstance(classType,10);  //指定数组的类型和大小   

             //对索引为5的位置进行赋值   
            Array.set(array, 5, "hello");   

            String s=(String)Array.get(array, 5);   
            System.out.println(s); 

            //循环遍历这个动态数组   
            for(int i=0;i<((String[])array).length;i++){
                String str=(String)Array.get(array, i);
                System.out.println(str);   
            }
        }
    }

***
#参考:

[JAVA反射机制的学习](http://hejianjie.iteye.com/blog/136205)

[菜鸟学Java（十四）——Java反射机制（一）](http://blog.csdn.net/liushuijinger/article/details/14223459)

[菜鸟学Java（十五）——Java反射机制（二）](http://blog.csdn.net/liushuijinger/article/details/15378475)