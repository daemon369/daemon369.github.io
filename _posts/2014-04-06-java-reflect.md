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
#Java 反射相关的API简介：(java.lang.reflect)

    Class类：代表一个类
    Filed类：代表类的成员变量
    Method类：代表类的方法
    Constructor类：代表类的构造方法
    Array类：提供了动态创建数组，以及访问数组的元素的静态方法。该类中的所有方法都是静态方法

<!--excerpt-->

***
#Class类

在 java 的Object类中的申明了数个应该在所有的java类中被改写的methods：hashCode(), equals(),clone(),toString(),getClass()等，其中的getClass()返回一个Class 类型的对象。

Class类十分的特殊，它和其它的类一样继承自Object，其实体用以表达java程序运行时的 class和 interface，也用来表达 enum，array，primitive，Java Types 以及关键字void，当加载一个类，或者当加载器(class loader)的defineClass()被JVM调用，便产生一个Class对象，Class是Reflection起源，针对任何你想探勘的class（类），唯有先为它产生一个Class的对象，接下来才能经由该对象唤起相应的反射API。

Java允许我们从多种途径为一个类class生成对应的Class对象。

1.运用 getClass（）：Object类中的方法，每个类都拥有此方法
 
    String str="abc";
    Class cl=str.getClass();

2.运用 Class.getSuperclass（）：Class类中的方法，返回该Class的父类的Class

3.运用 Class.forName（）静态方法：

4.运用 类名.class

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

    Class c=Class.forName("DynTest");
    Class[] ptype=new Class[]{double.class,int.class}；
    Constructor ctor=c.getConstructor(ptypr);
    Object[] obj=new Object[]{new Double(3.1415),new Integer(123)};
    Object object=ctor.newInstance(obj);
    System.out.println(object);

***
#运行时调用Method

首先准备一个Class[]{}作为getMethod(String name，Class[])方法的参数类型，接下来准备一个Obeject[]放置自变量，然后调用Method对象的invoke(Object obj，Object[])方法。

***
#运行时调用Field内容

变更Field不需要参数和自变量，首先调用Class的getField()并指定field名称，获得特定的Field对象后便可以直接调用Field的 get(Object obj)和set(Object obj,Object value)方法

***
#示例

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