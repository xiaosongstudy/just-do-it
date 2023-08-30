## 一、简介

1. 什么是函数式接口呢？
```txt
有且仅有一个未实现的抽象方法的接口叫函数式接口
```
具体代码实例如下：
```java
/**  
 * @author shiping.song  
 * @date 2023/8/31 00:30  
 * @email 2453332538@qq.com  
 */
 public interface Demo02 {  
  
    void doIt();  
}
```
这样的也算
```java
public interface Demo02 {  
  
    void doIt();  
  
    default void doOther() {  
  
    }  
}
```
一般情况，类似于我们重写方法时加==@Override==标识注解一样，函数式接口也有一个辅助注解，开发者可以使用这个注解标识当前接口是否为一个函数式接口，同时也可以起到校验的作用。
![](images/Pasted%20image%2020230831003603.png)
类似如下这种便不符合要求，在编译器便会报错
![](images/Pasted%20image%2020230831003659.png)
2. 函数式接口从什么开始有的，为什么需要使用函数式接口？
```txt
java从jdk8开始有函数式接口。至于为什么需要函数式接口呢？jdk8中还引入了一个特别重要的一个东西就是lambda表达式，函数式接口是lambada表达式的基石，它能够很方便的让我们能够使用java进行函数式编程。另外还有一点就是使用函数式接口能够起到一定的解耦作用。
备注：lambada其中很重要的一点就是【可推导】，例如：可推导出参数类型，可推导出是否有返回值等等以此来进行简化。而这些有一个很重要的一个前提就是它必须是函数式接口，否则无法进行推导。
```
## 二、JDK中常用的函数式接口

1. Function
```txt
接收一个T类型的入参，一个R类型的响应参数，可以理解为转化器。
```
2. Consumer
```txt
接收一个T类型的入参，可以理解为消费者。
```
3. Supplier
```txt
返回一个T类型的响应参数，可以理解为生产者。
```
4. Predicate
```txt
接收一个T类型的入参，返回一个布尔值，可以理解为断言。
```
还有一些变式
1. BiFunction
```txt
接受两个不同类型的入参，返回一个指定类型的响应参数，本质上还是转化器。
```
2. BinaryOperator
```txt
是bifunction的变式，两个入参以及响应的参数都是同一个类型，本质上式转化器。
```
3. BiConsumer
```txt
接收两个不同类型的入参，本质上还是消费者。
```
4. BiPredicate
```txt
接收两个不同类型的入参，返回一个布尔值，本质上还是可以理解为断言。
```
其它的还有，但是日常有这些基本上就够用了。
## 三、实际应用(函数式编程)
1. stream流
```txt
要说函数式接口应用最为广泛且最为经典的当属stream了，如果我们留心观察便会发现内部使用了很多的函数式接口。
这里举几个例子，例如：
- 我们使用forEach遍历list集合时使用的是consumer接口
- 遍历map时会用到biconsumer接口
- 使用map进行元素转化时会使用到function以及bifunction接口
- 使用过滤器过滤时会涉及到predicate断言
- 使用groupingBy时会涉及到supplier接口
...
所以，熟练掌握常用的几个函数式接口会让我们在日常使用stream流的过程中更加的得心应手
```
以下展示部分示例：
```java
import lombok.AllArgsConstructor;  
import lombok.Data;  
import lombok.NoArgsConstructor;  
  
import java.io.Serializable;  
import java.time.LocalDate;  
import java.util.List;  
  
/**  
 * @author shiping.song  
 * @date 2023/8/28 23:44  
 * @email 2453332538@qq.com  
 */@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class Demo implements Serializable {  
    private static final long serialVersionUID = 5385289373863206429L;  
  
    private Integer id;  
  
    private String username;  
  
    private LocalDate birthday;  
  
    private List<Demo> childDemo;  
}
```

```java
// 1.分组条件，以默认收纳规则进行收纳，有key值相同的收纳进一个list组中 以某一个字段属性作为key此时可借助方法引用简写  
Map<Integer, List<Demo>> collect = demos.stream().collect(Collectors.groupingBy(Demo::getId));  
for (Map.Entry<Integer, List<Demo>> integerListEntry : collect.entrySet()) {  
    System.out.printf("key=%s, value=%s\n", integerListEntry.getKey(), integerListEntry.getValue());  
}  
System.out.println("=============================================================================");  
// 1.2 多个属性构成分组key，可自行实现Function<? super T, ? extends K> classifier  
Map<String, List<Demo>> dayIdToDemos = demos.stream().collect(Collectors.groupingBy(demo -> demo.getId() + "--" + DateTimeFormatter.ofPattern("yyyyMMdd").format(demo.getBirthday())));  
for (Map.Entry<String, List<Demo>> dataMap : dayIdToDemos.entrySet()) {  
    System.out.printf("key=%s, value=%s\n", dataMap.getKey(), dataMap.getValue());  
}  
System.out.println("=============================================================================");
// 先根据 id分组，然后根据生日分组  
Map<Integer, Map<String, List<Demo>>> idAfterBirthdayToDemo = demos.stream().collect(Collectors.groupingBy(Demo::getId,  
        Collectors.groupingBy(demo -> DateTimeFormatter.ofPattern("yyyyMMdd").format(demo.getBirthday()))));  
idAfterBirthdayToDemo.forEach((id, birthdayToDemoMap) -> {  
    birthdayToDemoMap.forEach((birthday, demoList) -> {  
        System.out.printf("id编号为：%s, 生日为：%s，符合两个分组条件的收纳的信息数据为：%s\n", id, birthday, demoList);  
    });  
});  
System.out.println("=============================================================================");

System.out.println(demos.stream().filter(demo -> demo.getId() > 1).count());  
System.out.println("=============================================================================");  
// 只需要一部分字段  
Set<Integer> collect = demos.stream().collect(Collectors.mapping(Demo::getId, Collectors.toSet()));  
System.out.println(collect);  
System.out.println("=============================================================================");  
System.out.println(demos.stream().map(Demo::getId).collect(Collectors.toList()));  
System.out.println("=============================================================================");  
// 只收纳部分属性  
Map<Integer, List<LocalDate>> collect1 = demos.stream().collect(  
        Collectors.groupingBy(Demo::getId,  
                Collectors.mapping(Demo::getBirthday, Collectors.toList())));  
collect1.forEach((id, birthdayList) -> {  
    System.out.printf("id=%s, birthdayList=%s\n", id, birthdayList);  
});  
System.out.println("=============================================================================");  
Map<Integer, LocalDate> idToBirthday = demos.stream().collect(Collectors.toMap(Demo::getId, Demo::getBirthday));  
idToBirthday.forEach((id, birthday) -> {  
    System.out.printf("id=%s, birthday=%s\n", id, birthday);  
});  
System.out.println("=============================================================================");  
boolean allMatch = demos.stream().allMatch(demo -> 14 == demo.getId());  
System.out.println(allMatch);  
System.out.println("=============================================================================");  
List<LocalDate> collect2 = demos.stream().flatMap(demo -> Stream.of(demo.getBirthday())).collect(Collectors.toList());  
System.out.println(collect2);  
System.out.println("=============================================================================");  
List<Demo> collect3 = demos.stream().flatMap(demo -> demo.getChildDemo().stream()).collect(Collectors.toList());  
collect3.forEach(System.out::println);  
System.out.println("=============================================================================");  
int max = demos.stream().flatMapToInt(demo -> demo.getChildDemo().stream()  
        .mapToInt(Demo::getId)).max().orElse(-1);  
System.out.println(max);  
System.out.println("=============================================================================");
```
2.  lambda表达式
> 这一点就不扩充了，日常开发应该都用的挺多的。

3. 松耦合、灵活性高
假如说：现在我们需要抽取一个公共的处理逻辑，目前只知道有一个入参，然后具体如何操作处理这个入参我们并不知道，同时要确保操作不会影响到程序的后续执行。那么应该如何编写这段处理逻辑呢？
```txt
根据前面介绍的函数式接口，我们不难想到这个地方可以使用consumer接口实现，于是乎便可以得到如下的程序代码
```

```java
public <T> void doBusiness(T businessParams, Consumer<T> businessHandler) {  
    try {  
        businessHandler.accept(businessParams);  
    } catch (Exception e) {  
        log.error("业务处理异常", e);  
    }  
}
```

```txt
如上，我们将如何处理业务参数的主动权交由调用者去控制，并向调用者“保证”不管你前面业务处理中发生任何的异常，我都不会影响后续流程的进行。这样封装使得方法的通用性及灵活性更高了。
类似的应用有很多很多，可以根据实际需要结合常用的函数式接口来设计我们的程序。
```

