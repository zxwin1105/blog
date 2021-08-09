#                           Java抽象类

#### java中的抽象类

+ 当类中的有些功能不具体，这些不具体的功能需要在类中标识出来，java通过关键字abstract。定义了抽象方法的类也必须被abstract修饰。被abstract修饰的类是抽象类。

#### 抽象类的特点

	1. 抽象类和抽象方法都需要被abstract修饰，抽象方法一定要被定义在抽象类中。
 	2. 抽象类不可以创建实例。(调用抽象方法没有意义)
 	3. 只有覆盖了抽象类中的所有抽象方法后，其子类才可以实例化否则子类还是一个抽象类。

#### 细节问题

1. 抽象类一定是父类,因为抽象类是从子类共性中不断抽取而来的。
2. 抽象类中也有构造方法，虽然不能给自己的对象初始化，但是可以给自己子类对象初始化。
3. 抽象类中可以不定义抽象方法，仅仅是不让该类创建对象。
4. 抽象关键字abstract不可以和final，private，static三个关键字共存。

#### 案例

``` java
/**
 *需求:
 * 程序员
 *    属性： 姓名，工号，工资
 *    行为： 工作
 * 项目经理
 *    属性：姓名，工号，工资，奖金
 *    行为： 工作
 **************************************
 * 分析:
 *  1.程序员与项目经理之间不存在父子关系。
 *  2.程序员与项目经理之间有共性，可以抽取出来。
 *  3.程序员与项目经理的行为一致，但其工作的内容却不同，应定义为抽象方法。
 */
public class AbstractTest {
    public static void main(String[] args) {
        Employee e = new  Manager("zxw","001",20000,10000);
        e.work();
    }
}

//员工类
abstract class Employee {
    //抽取出员工的共有属性
    private String name;
    private String id;
    private double salary;

    public Employee(String name,String id,double salary){
        this.name = name;
        this.id = id;
        this.salary = salary;
    }
    //员工的共有行为
    public abstract void work();
}

//程序员类
class Programmer extends Employee{
    public Programmer(String name,String id,double salary){
        super(name,id,salary);
    }

    public void work(){
        System.out.println("写代码。。。");
    }
}

//项目经理类
class Manager extends Employee{
    private double bonus;
    public Manager(String name,String id,double salary,double bonus){
        super(name,id,salary);
        this.bonus = bonus;
    }

    public void work(){
        System.out.println("项目管理。。。");
    }
}

```

