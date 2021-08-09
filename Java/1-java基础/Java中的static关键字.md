  # Java中的static关键字

- 思考1：创建对象的作用就是为了产生实例，并进行数据的封装。而调用方法时，却没有用到这些对象中封装的数据，该对象还有创建的意义吗？虽然可以编译通过、运行。但是对堆内存中的空间比较浪费。
- 思考2：是不是所有的方法都可以定义为static？什么时候才可以使用？
  - 如果功能不需要访问类中的成员变量时，该功能就可以使用修饰。

```java
class PersonDemo{
    public static void main (String[] args){
        Person p = new Person("xiaoming",23);
        p.printPerson();
        Person.sleep(); //被static修饰，可以直接使用类名进行访问
    }
}

class Person{
    private String name;
    private int age;
    
    Public Person(String name, int age){
        this.name = name;
        this.age = age;
    }
    
    //此方法中，访问了成员变量，则不可以使用static修饰
    public void printPerson(){
        System.out.println(this.name + ";" + this.age);
    }
    //此方法中，没有访问成员变量，则可以使用static修饰
    public void sleep(){
        System.out.println("sleep...");
    }
}
```

### 结论

- 静态方法不能访问非静态成员，但非静态可以访问静态成员。
- 静态方法中不允许出现this,super关键字。

### 原理 

+ 静态是随着类的加载就加载了，随着类的消失而消失。
+ 静态优先于对象的存在，被对象共享。
+ 因为静态先存在于内存中无法访问后来的对象中的数据，所以静态无法访问非静态。

### 静态变量

######  当成员变量的值，每一个对象都一致时，就对该成变量进行静态修饰。

1. 静态变量与成员变量的区别。

   + 内存存储区域不同
     + 静态变量存储在方法区中；
     + 成员变量存储在堆内存中；

   + 加载时期不同
     + 静态变量随着类加载而加载；
     + 成员变量随着对象的加载而加载；

   + 范围不同:

     + 静态变量所属类；
     + 成员变量所属对象。

   + 调用不同

     + 静态变量可以被类和对象调用；

     + 成员变量只能被对象调用。

        

### 静态代码块

###### 类一加载就需要做的一些动作，不需用创建对象。

+ 静态代码块的特点:
  + 随着类的加载而执行，仅执行一次。

```java
class Demo{
    static int x = 9;
    static{
		//静态代码块。在静态变量显示初始化以后才执行。
        System.out.println("静态代码块" + x);
    }
    
    static void show(){
        System.out.println("show");
    }
}
```



### 静态加载内存图解

![static](../image/static加载内存图解.png)