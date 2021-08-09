### 目录

- [泛型引入](#泛型引入)
- [泛型的好处](#泛型的好处)
- [泛型类](#泛型类)
- [泛型方法](#泛型方法)
- [泛型接口](#泛型接口)
- [泛型通配符](#泛型通配符)
- [泛型限定](#泛型限定)

###  泛型引入

**集合中可以存储任意类型的元素，导致在取出时可能在强转是会抛出运行时异常ClassCastException，在JDK1.5之后出现了解决方案，使用容器时，必须明确容器中元素的类型，将这种机制称为“泛型”**

### 泛型的好处

- 是一种安全机制
- 将运行时期的ClassCastException转移到了编译时期的问题
- 泛型技术，是给编译器使用的技术
- 避免了强转的麻烦

### 泛型类

**在定义类时，如果不确定对象的类型，可以定义成参数，有使用该类的调用者来确定类型**

```java
class Tool<T>{
    private T obj;
    
    public T getObj(){
        return this.obj;
    }
    
    public void setObj(T obj){
        this.obj = obj;
    } 
}

public class Demo{
    public static void main(String[] args){
        Tool<String> tool = new Tool<String>();
        tool.setObj("set");
        System.out.println(tool.getObj);
    }
}
```

>set

### 泛型方法

**当方法要操作的类型不确定和类的泛型不一定一致，这时可以将泛型定义在方法上**

```java
class Tool<T>{
    //该方法参数类型与类泛型类型一致
    public void print(T args){
        System.out.Println(args);
    }
    
    //该方法参数类型与类泛型类型不一致
  	//这种方法叫做泛型方法
    public <Q> void show(Q args){
        System.out.println(args);
    }
}

public class Demo{
	public static void main(String[] args){
        //创建String类型的Tool对象
        Tool<String> tool = new Tool<String>();
        tool.print("TEST");//该方法的参数类型与类类型一致是String
        tool.show(4); //该方法参数类型与类类型无关
        tool.show("SHOW");  
    }
}
```

>TEST
>
>4
>
>SHOW

### 泛型接口

```java
//泛型接口的定义
interface Inter<T>{
    void show(E e);
}

//泛型接口的使用,使用的时候应该明确类型
class InterImpl implements Inter<String>{
    public void show(String e){}
}

//当子类类型也不确定时
class InterImpl1<E> implements Inter<E>{
    public void show(E e){}
}
```

### 泛型通配符

```java
public class Test {
    public static void main(String[] args) {
        Set<String> set = new HashSet<String>();
        set.add("java");
        set.add("python");
        set.add("scala");
        printCollection(set);
        List<Integer> list = new ArrayList<Integer>();
        list.add(1);
        list.add(3);
        list.add(5);
        printCollection(list);
    }
    /*
     泛型通配符 ？:当使用泛型类或者泛型接口时。要传递的具体类型不确定可以使用通配符表示。
     注意此时的 ？ 是实际参数
     */

    /**
     *  将集合的遍历抽取成方法printCollection()
     *  由于集合不确定，所以方法形参类型为Collection
     */
    public static void printCollection(Collection<?> col) {
        for(Iterator<?> it = col.iterator(); it.hasNext();)
            System.out.println(it.next().toString());
    }
} 
```

### 泛型限定

```java
public class Test {
    public static void main(String[] args) {
        Set<Student> set = new HashSet<Student>();
        set.add(new Student("zs",21));
        set.add(new Student("lis",22));
        set.add(new Student("wangw",20));
        printCollection(set);
        List<Worker> list = new ArrayList<Worker>();
       	list.add(new Worker("wmz",32));
        list.add(new Worker("zxs",32));
        list.add(new Worker("dfds",42));
        printCollection(list);
    }
    /*
     泛型通配符 ？:当使用泛型类或者泛型接口时。要传递的具体类型不确定可以使用通配符表示。
     注意此时的 ？ 是实际参数
     --------------------
     该printCollection()参数只接收Person类或子类时要使用泛型的限定
     上限：？ extends E 只允许接收E类型或其子类型
     下限：？ super E 只允许接收E类型或其父类型
     */
    public static void printCollection(Collection<?> col) {
        for(Iterator<?> it = col.iterator(); it.hasNext();)
            System.out.println(it.next().toString());
    }
} 
```