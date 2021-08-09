### 获取Stream流

- 所有的`Collection`集合都可以通过`stream()`方法获取流
  - `default Stream<E> stream()`
- `Stream`接口的静态方法`of()`可以获取数组对应的流
  - `static <T> Stream<T> of(T... values) `参数是一个可变参数，可以传递一个数组

### Stream流的特点

**Stream流属于管道流，只能被使用一次，第一个Stream流调用完方法，数据就会传到下一个Stream流上，第一个Stream流就会关闭不能在调用方法**

### 常用方法

###### 	方法类型

- **延迟方法**：返回值类型仍是`Stream`接口自身类型的方法，支持链式调用。
- **终结方法**：返回值不再是`Stream`接口自身类型的方法，不支持链式调用 `Stream`接口终结方法有`count()` `forEach()`

######     常用方法

- `void forEach(Consumer<? super T> action);`  用来遍历流中的数据

- `long count();` 用于统计Stream流中元素的个数额

- `Stream<T> filter(Predicate<? super T> predicate);` 过滤方法将一个流通过predicate过滤后转换成另外一个子集流

- `<R> Stream<R> map(Function<? super T,? extends R> mapper);可以将流中的元素映射到另外一个流中`

- `Stream<T> limit(long maxSize);` 对流进行截取前maxSize

- `Stream<T> skip(long n);` 跳过前n个元素，如果n大于当前流的长度则返回一个空的流。

- `static <T> Stream<T> concat(Stream<? extends T> a,Stream<? extends  T> b);`该静态方法可以将两个流合并成一个流。

  

  