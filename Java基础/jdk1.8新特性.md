

## 1、Lambda表达式

### 1.1、定义

> Lambda：Lambda is an operator used to denote **anonymous functions** or **closures**[^闭包],following the usage of lambda calculus

Why

> 在Java中，我们无法将函数作为参数传递给一个方法，也无法声明返回一个函数方法；

```java
public class MyRunnable {
    public static void main(String[] args) {
		//JDK1.8之前，匿名内部类
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("before jdk1.8");
            }
        });
        thread.start();
		//JDK1.8，Lambda表达式
        new Thread( ()-> System.out.println("after jdk1.8")).start();
    }
}

```

### 1.2、表达式基本语法

> (type param1,type param2) -> {body}
>
> 其中，type可以省略，若参数只有一个，则（）也可以省略，主体只有一条语句，则{ }可以省略例如
>
> (param1,param2) - >{body} 
>
> param1 -> body

### 1.3、表达式的作用

- Lambda表达式为 Java 添加了缺失的函数式编程特性，使我们能将函数当做一等公民对待
- 在将函数作为一等公民的语言中，Lambda表达式的类型是函数（Python）。但是在Java中， Lambda表达式是对象，他们必须依附于一类特别的对象类型——函数式接口。
- Lambda 表达式是一种匿名函数，它是没有声明的方法，即没有访问修饰符、返回值声明和名字。

#### **1.3.1、遍历集合**

```java
public class MyList {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3);
        //普通for循环
        for(int i =0; i < list.size(); i++){
            System.out.println(list.get(i));
        }
        //外部迭代
        for (Integer i : list){
            System.out.println(i);
        }
        // 内部迭代（Lambda） 类型推断，编译时编译器可以根据上下文推断出该变量的类型
        list.forEach((Integer i -> System.out.println(i)));
        list.forEach(i-> System.out.println(i));

        //方法引用
        list.forEach(System.out::println);
    }
}
```

#### 1.3.2、集合排序

```java
public class MyListSort {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,-5,0,30);
        //JDK1.7写法
        Collections.sort(list,new Comparator<Integer>() {
    		@Override
   			public int compare(Integer o1, Integer o2) {
        		return o1 - o2;
    		}
		});
		System.out.println(list);
        
        //JDK1.8写法
        list.sort((i1,i2) -> i1-i2);
        System.out.println(list);
    }
}
```

### 1.4、注意事项

Lambda表达式的实质其实还是匿名内部类，而匿名内部类在访问外部局部变量时，要求变量必须声明为`final`！不过我们在使用Lambda表达式时无需声明`final`，这并不是说违反了匿名内部类的规则，因为Lambda底层会隐式的把变量设置为`final`，在后续的操作中，一定不能修改该变量：

**正确示范：**

```java
// 定义一个局部变量
int num = -1;
Runnable r = () -> {
    // 在Lambda表达式中使用局部变量num，num会被隐式声明为final
    System.out.println(num);
};
new Thread(r).start();// -1
```

错误示范：

```java
// 定义一个局部变量
int num = -1;
Runnable r = () -> {
    // 在Lambda表达式中使用局部变量num，num会被隐式声明为final，不能进行任何修改操作
    System.out.println(num++);
};
new Thread(r).start();//报错
```





---



## 2、函数式接口（@FunctionalInterface）

### 2.1、函数式接口定义

> 如果**接口**只有**一个抽象**方法，那么称该接口为函数式接口,函数式接口可以通过Lambda表达式、方法引用、构造器引用创建。

- 如果我们在一个接口上声明了@FunctionalInterface 注解，那么该编译器就会按照函数式接口的定义来要求改接口，若不符合定义，则会生成一个错误信息
- 如果某个接口只有一个抽象方法，但我们没有声明 @FunctionalInterface 注解，那么编译器依然会将该接口视为函数式接口
- 如果一个函数式接口重写了 Object 中的方法， 仍认为该接口只有一个抽象方法，即该接口仍是函数式接口

```java
@FunctionalInterface
public interface MyInterface {
    void test();
    @Override
    boolean equals(Object obj);
    @Override
    String toString();
}
```

### 2.2、Function及BiFunction函数式接口

#### 2.2.1、Function接口

```java
@FunctionalInterface
public interface Function<T, R> {
	// 接收一个参数T，返回一个结果R
    R apply(T t);
}
```

Function 代表的是有参数，有返回值的函数。

#### 2.2.2、BiFunction接口

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {

    /**
     * Applies this function to the given arguments.
     *
     * @param t the first function argument
     * @param u the second function argument
     * @return the function result
     */
    R apply(T t, U u);
}
```

类似的 Function 接口：



| 接口名                 | 描述                                        |
| :--------------------- | ------------------------------------------- |
| `DoubleFunction<R>`    | 接收double类型参数，并且返回R类型结果的函数 |
| `IntFunction<R>`       | 接收int类型参数，并且返回R类型结果的函数    |
| `LongFunction<R>`      | 接收long类型参数，并且返回R类型结果的函数   |
| `ToDoubleFunction<T>`  | 接收T类型参数，并且返回double类型结果       |
| `ToIntFunction<T>`     | 接收T类型参数，并且返回int类型结果          |
| `ToLongFunction<T>`    | 接收T类型参数，并且返回long类型结果         |
| `DoubleToIntFunction`  | 接收double类型参数，返回int类型结果         |
| `DoubleToLongFunction` | 接收double类型参数，返回long类型结果        |

这些都是一类函数接口，在 Function 基础上衍生出的，要么明确了参数不确定返回结果，要么明确结果不知道参数类型，要么两者都知道。

#### 2.2.3、用法示例

```java
public class FunctionTest {

    public static void main(String[] args) {
        FunctionTest test= new FunctionTest();
        System.out.println(test.compute(5, value -> value * value)); //25

        System.out.println( test.compute(5,value -> value / 2));   //2
        System.out.println( test.compute(3,value -> value - 3));   //0

        test.div(5);
    }
	//JDK1.8，Lambda表达式是一种行为，用的时候才将行为传递过来
    //高阶函数：如果一个函数接受一个函数作为参数，或者返回一个函数，那么该函数叫做高阶函数
    public int compute(int a, Function<Integer,Integer> function){
        return function.apply(a);
    }
    //JDK1.8之前，需要提前先将行为定义好
    public int multi(int a){
        return a * a;
    }
    public int div(int a){
        return a /2;
    }
    public int reduce(int a){
        return a - 3;
    }
}
```

```java
public class FunctionTest2 {
    public static void main(String[] args) {
        FunctionTest2 test2 = new FunctionTest2();
        System.out.println(test2.composeTest(2,value -> value * 3, value -> value * value));  //12
        System.out.println(test2.andThenTest(2,value -> value * 3, value -> value * value));  //36
        System.out.println(test2.compute(1,2,(value1,value2) -> value1 + value2));//3
		System.out.println(test2.biAndThenTest(1,2,(value1,value2) -> value1 + value2,value -> value * value));  //9
    }
    
    //先计算function2，再计算function1
    public int composeTest(int a, Function<Integer,Integer> function1,Function<Integer,Integer> function2){
        return function1.compose(function2).apply(a);
    }
    //先计算function1，再计算function2
    public int andThenTest(int a, Function<Integer,Integer> function1,Function<Integer,Integer> function2){
        return function1.andThen(function2).apply(a);
    }

    /**
     *  实现两个变量的数值操作，具体行为由用户定义
     */
    public int compute(int a, int b, BiFunction<Integer,Integer,Integer> function){
        return function.apply(a,b);
    }
    public int biAndThenTest(int a, int b, BiFunction<Integer,Integer,Integer> biFunction, Function<Integer,Integer> function){
        return biFunction.andThen(function).apply(a,b);
    }
}

//BiFunction没有compose方法，因为BiFunction需要接收两个值，语法原因
```

### 2.3、Predicate函数式接口

#### 2.3.1、Predicate接口

```java
@FunctionalInterface
public interface Predicate<T> {
	// 接收T类型参数，返回boolean类型结果
    boolean test(T t);
}
```

#### 2.3.2、用法示例

```java
public class PredicateTest {
    public static void main(String[] args) {
        Predicate<String> predicate = p -> p.length() < 5;
        //boolean test(T t);
        System.out.println(predicate.test("hello"));
    }
}
```

```java
public class PredicateTest2 {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
        PredicateTest2 test2 = new PredicateTest2();
        test2.conditionFilter(list,i ->i % 2 == 0);
        test2.conditionFilter(list,i->true);

        test2.conditionFilter(list, i->i>5, i->i%2==0);
    }
    //满足一个条件的过滤器
    private void conditionFilter(List<Integer> list, Predicate<Integer> predicate){
        for(Integer i:list){
            if (predicate.test(i)){
                System.out.println(i);
            }
        }
    }
    //同时满足两个条件的过滤器
    private void conditionFilter(List<Integer> list,Predicate<Integer> predicate1,Predicate<Integer> predicate2){
        for(Integer i:list){
            if(predicate1.and(predicate2).test(i)){   
                System.out.println(i);
            }
        }
    }
}
```



### 2.4、Supplier函数式接口

#### 2.4.1、Supplier接口

```java
@FunctionalInterface
public interface Supplier<T> {
	// 无需参数，返回一个T类型结果
    T get();
}
```

#### 2.4.2、用法示例

```java
public class SupplierTest {
    public static void main(String[] args) {
        Supplier<Person> supplier = () -> new Person();
        Supplier<Person> supplier1 = Person::new;
        System.out.println(supplier.get().getUsername());
        System.out.println(supplier1.get().getUsername());
    }
}
```

### 2.5、BinaryOperator函数式接口

#### 2.5.1、BinaryOperator接口

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    
    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }
     public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```

#### 2.5.2、用法示例

```java
public class BinaryOperatorTest {
    public static void main(String[] args) {
        BinaryOperatorTest binaryOperator = new BinaryOperatorTest();
        System.out.println(binaryOperator.compute(1, 2, (a, b) -> a + b));
		//返回长度比较短的字符串
        System.out.println(binaryOperator.minStr("hello1","world",(a,b)-> a.length() - b.length()));
        //返回首字母比较小的字符串
        System.out.println(binaryOperator.minStr("hello1","world",(a,b)->a.charAt(0) - b.charAt(0)));

    }
    private int compute(int a, int b, BinaryOperator<Integer> binaryOperator){
        return binaryOperator.apply(a,b);
    }
    private String minStr(String a, String b, Comparator<String> comparator){
        return BinaryOperator.minBy(comparator).apply(a,b);
    }
}
```



## 3、Stream

- 和迭代器不同的是，Stream可以并行化操作，迭代器只能命令式地、串行化操作
- 当使用串行方式去遍历时，每个item读完后再读下一个item
- 使用并行去遍历时，数据会被**分成多个段**，其中每一个都在**不同的线程**中处理，然后将结果一起输出
- Stream的并行操作依赖于 Java7 中引入的 Fork/Join 框架



## 4、Optional

### 4.1、定义

>Optional 是一个可能包含或不包含 non-null 值的容器对象，也是一个 value-basd 类
>
>关于value-based 类的定义如下：
>
>- 由 final 修饰，是不可变的（可能包含指向可变对象的引用）
>- 实现equals、hashCode、toString方法，基于实例的状态计算得出，而不是通过其他的对象或变量进行计算
>- 没有可访问的构造方法，由工程方法创建实例
>- 当两个实例相等时，可以进行任意的交换
>- 不要使用 == 等操作

`Optional一般用来避免NPE（NULL POINT EXCEPTION),不建议用来当做成员变量或方法参数使用`

### 4.2、简单使用

```java
public class OptionalTest {
    public static void main(String[] args) {
        //value-based class
        Optional<String> optional = Optional.ofNullable(null);

        //面向对象式，不推荐
//        if(optional.isPresent()){
//            System.out.println(optional.get());
//        }

        //函数式编程，推荐
        optional.ifPresent(str-> System.out.println(str));
        //若optional为空，则输出world
        System.out.println(optional.orElse("world"));
        System.out.println(optional.orElseGet(() -> "nihao"));
    }
}
```

```java
public class OptionalTest2 {
    public static void main(String[] args) {
        Employee employee = new Employee("zhangsan",24);
        Employee employee2 = new Employee("lisi",32);
        List<Employee> list = Arrays.asList(employee,employee2);

        Company myCompany = new Company("myCompany",list);
        Optional<Company> optional = Optional.ofNullable(myCompany);
        System.out.println(optional.map( company -> company.getEmployees()).
                orElse(Collections.emptyList()));
    }
}
```

## 5、方法引用（method reference）

### 5.1、定义

方法引用实际上是 Lambda 表达式的一种语法糖，操作符是双冒号：：，可以用来直接访问类或实例的方法或构造方法。

### 5.2、方法引用的几种类型

- 类名：：静态方法名
- 引用名：：实例方法名
- 类名：：实例方法名
- 类名：：new

### 5.3、简单使用

首先定义一个 Student 类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    private String username;
    private int age;

    //声明的这两个静态方法实际上不符合类的设计原则，因为这两个方法与Person类没有任何关系，这里为了方便演示，所以这么设计
    public static int comparePersonByAge(Person p1,Person p2){
        return p1.getAge() - p2.getAge();
    }
  
    public int comparePersonByAge1(Person p1,Person p2){
        return p1.getAge() - p2.getAge();
    }

    public int comparePersonByAge2(Person p1){
        return this.age - p1.getAge();
    }

}

```

再定义一个测试类

```java
public class MethodReferenceTest {
    public String getString(Supplier<String> supplier){
        return supplier.get();
    }
    public String getString(String str, Function<String,String> function){
        return function.apply(str);
    }
    public static void main(String[] args) {
        Person p1 = new Person("zhangsan",15);
        Person p2 = new Person("lisi",10);
        Person p3 = new Person("wangwu",32);
        Person p4 = new Person("zhaoliu",30);

        List<Person> list = Arrays.asList(p1,p2,p3,p4);
        //类名：：静态方法名
//        list.sort(Person::comparePersonByAge);
//        list.forEach(System.out::println);

        //引用名：：实例方法
//        list.sort(p1::comparePersonByAge1);
//        list.forEach(System.out::println);

        //类名：：实例方法
//        list.sort((item1,item2) -> item1.comparePersonByAge2(item2));
//        list.sort(Person::comparePersonByAge2);
//        list.forEach(System.out::println);

        //类名：：new
        MethodReferenceTest methodReferenceTest = new MethodReferenceTest();
//      methodReferenceTest.getString(()->new String());
        methodReferenceTest.getString(String::new);
        methodReferenceTest.getString("hi",String::new);
    }
}
```

## 6、接口默认方法

从JDK1.8开始，接口可以有默认实现方法，使用 default 关键字，若接口定义了默认方法，则实现该接口的类可以调用该方法。

一个简单接口的默认方法如下

```java
public interface myInterface1{
    default void defaultMethod(){
        System.out.println("myInterface");
    }
}
```

```java
public interface myInterface2{
    default void defaultMethod(){
        System.out.println("myInterface");
    }
}
```

若一个类实现了两个接口，且这两个接口有同样的默认方法，那么这个类需要重写该默认方法或指定使用哪个接口的默认方法，否则编译器会报错。

```java
public class DefaultMethodTest implements MyInterface1,MyInterface2{
    @Override
    public void defaultMethod() {
        MyInterface1.super.defaultMethod();
    }

    public static void main(String[] args) {
        DefaultMethodTest test = new DefaultMethodTest();
        test.defaultMethod();
    }
}
```

```java
public class MyInterface1Impl implements MyInterface1{
    @Override
    public void defaultMethod(){
        System.out.println("MyInterface1Impl");
    }
}
```

若一个类继承了一个有默认方法的接口实现，同时实现了另一个有相同默认方法的接口，编译器会调用父类的默认方法。**注意：父类必须重写了默认方法，否则这里依然会报错。**

```java
public class DefaultMethodTest extends MyInterface1Impl implements  MyInterface2{

    public static void main(String[] args) {
        DefaultMethodTest test = new DefaultMethodTest();
        test.defaultMethod();  //MyInterface1Impl
    }
}

```









