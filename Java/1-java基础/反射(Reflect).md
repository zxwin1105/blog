### 反射概念

>动态的获取指定的类以及动态的调用类中的内容

### 获取字节码文件

- 通过`Object`类中的`getClass()`方法

  > 弊端：使用这种方法获取字节码时，必须要有指定类，并对该类进行对象的创建，才可以调用`getClass()`方法，不能适用于reflect技术。

  - `Class clazz = Person.getClass();`

- 使用任意数据类型的静态成员`class`,所有的数据类型都具备该成员

  > 弊端：仍需要使用具体的类

  - `Class clazz = Person.class;`

- 使用`Class`类中的`forName(String className)`方法，通过给定类名来获取对应的字节码文件对象。

  > 反射技术中用来获取字节码文件的方式

  - `Class clazz = Class.forName(xx.xx.xx.Person);`

### Reflect常见异常

> - `ClassNotFoundException` : 类没有找到异常，类名错误
> - `InstantiationException` : 通过newInstance()来创建类的实例时，会调用空参构造器，若没有提供空参构造器，会抛出该异常
> - `IllegalAccessException` : 无效的访问异常，如果有提供方法权限不够时会抛出该异常

### Reflect获取构造器

> 在`Class`类中可以通过方法`Constructor<T> getConstructor<Class<T>... parameterTypes)`来返回对应参数的构造器对象
>
> `Constructor<T>` : 描写构造器的类
>
> `T newInstance(Object... initargs)` : 指定的构造器对象可以通过该方法实例化对象