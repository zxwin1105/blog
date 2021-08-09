### 异常和错误

>异常：异常可以有针对性的处理方式
>
>错误：错误没有针对性的处理方式；Error的发生往往都是系统级别的问题。只能通过修正代码处理

### 异常发生过程

![exceptionProcess](\image\exceptionProcess.png)

### Exception和RuntimeException的区别

> Exception:如果程序内抛出Exception，说明该程序出现了问题，Java会认为这个程序本身存在隐患，需要捕获或者声明出来。
>
> RuntimeException:不是功能本身的异常，而是因为如调用者传递参数错误而导致运行失败，这时也是问题需要用异常体现，但是这个异常不用生命。
>
> **生命的目的是为了让调用者处理异常；不声明的目的是为了不让调用者处理，就是为了让程序停止。**

### 异常在继承或实现中的使用细节

> 1. 子类覆盖父类方法时，如果父类声明了异常，子类只能声明父类异常或者其子类异常，也可以不声明异常
> 2. 当父类方法声明了多个异常时，子类覆盖时只能声明多个异常的子集
> 3. 当被覆盖的方法没有声明异常时，子类方法时无法声明异常的

- 原因

```java
class AException extends Exception{}
class AAException extends AException{}
class BException extends Exception{}
class Fu{
    void show()throws AException{
        
    }
}
class Zi extends Fu{
    void show(){
      
}
public calss Demo{
    /*
    当一个功能调用父类的功能时，因为父类方法对外声明了异常，所以method要的父类方法的异常进行捕获，
    fu.show()声明的是AException所以捕获AException;此时如果method方法所传参数对象是Zi,如果Zi.show()方法声明的异常不是AEception或其子类method无法对其异常进行捕获处理，会编译失败。
    */
    public void method(Fu fu){
        try{
            fu.show();
        }catch(AException e){
            
        }
    }
    public static void main(String[] args){
        
    }
}

```

## 权限访问

|          | public | protected | default | private |
| :------: | :----: | :-------: | :-----: | :-----: |
| 一个类中 |   √    |     √     |    √    |    √    |
| 一个包中 |   √    |     √     |    √    |         |
|  子类中  |   √    |     √     |         |         |
| 不同包中 |   √    |           |         |         |

> 包与包中只有两种权限可以使用，而且protected只能给不同包的子类使用。