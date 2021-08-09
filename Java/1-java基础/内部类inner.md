### 内部类的使用场景

- 当描述事物时，事物的内部还有事物，这个内部的事物还在访问外部事物中的内容,这时就将这个事物通过内部类来描述。

### 内部类与外部类之间的访问方式

- 内部类可以直接访问外部类的所有成员，包括私有。
- 外部类要想访问内部类中的成员，必须要创建内部类的对象。

### 局部内部类

- 局部内部类不能被成员修饰符修饰
- 内部类定义在局部时，只能访问被final修饰的局部变量

### 匿名内部类

**匿名内部类其实就是一个带有内容的子类对象**

- 匿名内部类的前提：内部类必须要继承父类或者实现接口

```java
abstract class AbsDemo{
    abstract void show();
}

interface Inter{
    void show1();
    void show2();
}

class Outer{
    int num = 3;
   	public void mothed(){
         //匿名内部类实现1
        new AbsDemo(){
            //覆盖父类方法
            void show (){
                System.out.println("匿名内部类");
            }
        }.show();//调用方法
        
        //匿名内部类实现2
        Inter in = new Inter(){
            //覆盖方法
            void show1(){
                System.out.println("Inter1");
            }
            void show1(){
                Sustem.out.ptintln("inter2");
            }
        }
        in.show1();
        in.show2();
    }
}
```



