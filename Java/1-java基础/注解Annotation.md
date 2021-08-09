### 什么时注解

> - Annotation是从jdk5.0开始引入的
> - Annotation的作用
>   - 可以对程序做出解释
>   - ***可以被其他程序读取***(如编译器)
> - Annotation的格式
>   - 注解是以"@注释名"的形式存在，还可以添加一些参数如@SuppressWarnings(value="unchecked")
> - Annotation的使用
>   - 可以在package,class,method,field等上使用	

### 内置注解

> `@Override`:定义在java.lang.Override中，该注释只用于修饰方法，表示一个方法声明打算重写超类中的另一个方法
>
> `@Deprecated`:定义在java.lang.Deprecated中，可用来修饰方法，属性，类。表示已过时的元素
>
> `@SuppressWarnings`: 定义在java.lang.SuppressWarnings中，用于抑制编译时的警告信息

### 元注解

> - 元注解的作用时负责注解其他注解的。Java定义了4个标准的meta-annotation类型， 他们被用来提供对其他annotation类型作说明
> - 这些类型和它们所支持的类在.java.lang.annotation包中可以找到(@Target,@Retention,@Documented,@Inherited)
>   - `@Target` : 用于描述注解的使用范围为
>   - `@Retention`：表示需要在什么级别保存该注解信息，描述注解的声明周期
>     - (SOURCE < CLASS < RUNTIME)
>   - `@Documented`:  说明该注解将被包含在Javadoc中
>   - `@Inherited`: 说明子类可以继承父类中的该注解