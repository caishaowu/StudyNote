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

- Lambda表达式为Java添加了缺失的函数式编程特性，使我们能将函数当做一等公民对待
- 在将函数作为一等公民的语言中，Lambda表达式的类型是函数（Python）。但是在Java中， Lambda表达式是对象，他们必须依附于一类特别的对象类型——函数式接口。

**遍历集合**

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

### **1.4、Function方法调用**

```java
public class FunctionTest {

    public static void main(String[] args) {
        FunctionTest test= new FunctionTest();
        System.out.println(test.compute(5, value -> value * value));

        System.out.println( test.compute(5,value -> value / 2));
        System.out.println( test.compute(3,value -> value - 3));



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
    public int composeTest(int a, Function<Integer,Integer> function1,Function<Integer,Integer> function2){
        return function1.compose(function2).apply(a);
    }
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



## 3、Stream

