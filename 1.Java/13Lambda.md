# Lambda

简介 : Lambda(底层是ASM技术实现) 和 匿名内部类 类似, 底层都是创建一个 匿名类 实现 interface;

Lambda表达式(匿名子类的匿名对象) --> 函数; 使用Lambda表达式,有且仅有一个抽象方法的接口(@FunctionalInterface注解可以实现 接口为函数式接口 );

*Lambda语法格式 :*

<img src="https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20230702103202817.png" alt="image-20230702103202817" style="zoom:80%;" /> 

*常见的函数式接口 :*

- Runnable
- Suppiler
- Consumer
- Comparator
- Predicate
- Function

`Test`

```java
public class LambdaTest {
    public static void main(String[] args) {
        // 匿名内部类,implement IA interface
        IA ia = new IA() {
            @Override
            public void show() {
                System.out.println("匿名内部类实现接口");
            }
        };
        ia.show();

        // Lambda表达式,implement IA interface (interface just only one abstract method)
        IA ia1 = ()->{
            System.out.println("Lambda表达式实现接口");
        };

        ia1.show();
    }
}

```

```java
// 添加 @FunctionalInterface 声明为 functional interface, 只能有一个 abstract method
@FunctionalInterface
public interface IA {
    public void show();
}
```

`Pass param`

```java
    public static void main(String[] args) {

        // Lambda表达式,function pass param
        IA ia1 = (int num1,int num2)->{
            System.out.println(num1 + num2);
        };
        ia1.show(10,20);

        // Simplify
        IA ia2 = (n1, n2) -> {
            System.out.println(n1 + n2);
        };
        ia2.show(20,30);
    }

```

```java
@FunctionalInterface
public interface IA {
    public void show(int a ,int b);
}
```

`Lambda as param`

```java
public class LambdaTest {
    public static void main(String[] args) {
        test(() -> {
            System.out.println("hello world");
        });
    }

    public static void test(IA ia) {
        ia.show();
    }
}
@FunctionalInterface
interface IA {
    void show();
}
```

`Return Lambda`

```java
public class Main {
    public static void main(String[] args) {
        IA ia = getIA();
    }

    public static IA getIA() {
        return () -> {
            System.out.println("hello world");
        };
    }
}
@FunctionalInterface
interface IA {
    void show();
}
```

# Runnable

```java
public class RunnableTest {
    public static void main(String[] args) {
        // implement Runnable interface
        Runnable runnable = ()->{
            System.out.println("Lambda implement runnable ! !");
        };

        // implement runnable, run() 可以开启线程
        new Thread(runnable).start();
    }
}
```

```java
@FunctionalInterface
public interface Runnable {

    public abstract void run();
}
```

# Supplier

```java
public class SupplierTest {
    public static void main(String[] args) {
        // implement Supplier interface , return data
        Supplier<String> supplier = () -> {
          return "Hello Banne!";
        };

        String str = supplier.get();
        System.out.println(str);
    }
}

```

```java
@FunctionalInterface
public interface Supplier<T> {
    //Gets a result.
    //Returns:a result
    T get();
}
```

# Consumer

```java
public static void main(String[] args) {
    test1((msg) -> {
        System.out.println(msg);
    });
    
    test2((msg) -> {
        System.out.println(msg.toUpperCase()); // HELLO WORLD
    }, (msg) -> {
        System.out.println(msg.toLowerCase()); // hello world
    });
}

public static void test1(Consumer<String> consumer) {
    // implement consumer interface, pass param
    consumer.accept("hello world");
}

public static void test2(Consumer<String> consumer1, Consumer<String> consumer2) {
    // consumer1 调用 toUpperCase("hello WORLD"), consumer2 调用 toLowerCase("hello WORLD"), 不是 consumer1 的 ret val 作为 consumer2 的 param
    consumer1.andThen(consumer2).accept("hello WORLD");
}
```

```java
@FunctionalInterface
public interface Consumer<T> {
    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
}
```

# Comparator

```java
public class ComparatorTest {
    public static void main(String[] args) {
        // implement Comparator interface, can pass param and return data
        Comparator<String> comparator = (o1, o2) -> {
            return o1.length() - o2.length();
        };
        
        String[] strs = {"Ban", "Bann", "Banne"};
        
        Arrays.sort(strs, comparator);
    }
}
```

# Predicate

```java
public class PredicateTest {
    public static void main(String[] args) {
        andMethod(str -> str.contains("B"),str1 -> str1.contains("!"));

        orMethod(str ->str.contains("!"),str1->str1.contains("Z"));

        negateMethod(str -> str.length() < 20);
    }
    static void andMethod(Predicate<String> one,Predicate<String> two){
        //  &&  default and()方法, 判断两个Predicate 是否都满足 test()定义的条件
        boolean isValid = one.and(two).test("Banne!!");
        System.out.println("字符串符合要求吗 "+ isValid);
    }

    static void orMethod(Predicate<String> one,Predicate<String> two){
        //  || default or()方法, 存在一个Predicate 满足 test() 定义条件即可
        boolean isValid = one.or(two).test("Banne!!");
        System.out.println("字符串符合要求吗 "+ isValid);
    }

    static void negateMethod(Predicate<String> predicate){
        //  !  default negate()方法 , 取反
        boolean isValid = predicate.negate().test("Banne!!");
        System.out.println("字符串符长度大于20吗 "+ isValid);
    }
}
```

# Function

```java
public static void main(String[] args) {
    test1((s) -> {
        return Integer.parseInt(s);
    });

    test2((s) -> {
        return Integer.parseInt(s); // 10
    }, (i) -> {
        return i * 100; // 1000
    });
}

public static void test1(Function<String, Integer> function) {
    // 接收 String, 返回 Integer, 接收 "10", 返回 10
    int res = function.apply("10");
}

public static void test2(Function<String, Integer> function1, Function<Integer, Integer> function2) {
    // function1 调用 return Integer.parseInt("10"), function2 调用 return 10 * 100, function1 的 ret val 作为 function2 的 param
    int res = function1.andThen(function2).apply("10"); // 1000
}
```

```java
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
}
```

# Method Reference

简介 : 引用已存在的方法,避免重复逻辑;简化Lambda表达式;

```java
public class MethodReferenceTest {
    public static void main(String[] args) {
        // 普通 Lambda 表达式的使用
        PrintString(str -> System.out.println(str));

        /* 方法引用:
        printString()函数(方法),实现的功能为打印 String;
        而且,System.out的 println()函数,实现的功能与该方法相同;可进行简化.
        */
        PrintString(System.out::println);// 普通方法引用语法 --> 对象名 :: 方法名
    }
    public static void PrintString(IA p){
        p.printString("Hello Banne ! ! ");
    }
}
```

```java
@FunctionalInterface
public interface IA {
    void printString (String str);
}
```

*方法引用的兼容格式 :*

Param 兼容

- param type 向下兼容
  - e.g: test1(Object obj) 兼容 test2(String str) 
- param number 兼容
  - e.g: test1(String str, Integer... nums) 兼容 test2(String str) 

Return valuie 兼容

- void 兼容
  - e.g: void test1() 兼容 String test2()

## Static Method Reference

```java
public class StaticMethodReference {
    public static void main(String[] args) {
        // Lambda implement interface method
        IB ib = (str)->{
            return Integer.parseInt(str);
        };
        ib.test("10");

        // static method reference   对象名 :: 静态方法名
        IB ib1 = Integer::parseInt;
        ib1.test("10");
    }
}
```

```java
 @FunctionalInterface
public interface IB {
    int test(String str);
}
```

## Constructor Reference

```java
public class ConstructorReference {
    public static void main(String[] args) {
        // Lambda implement interface
        IC ic = (str) ->{
            return  new Person(str);
        };
        Person person = ic.constructor("Banne");
        System.out.println(person);

        // constructor method reference   类名 :: new
        IC ic1  = Person::new;
        Person person1 = ic1.constructor("Harvey");
        System.out.println(person1);
    }
}

class Person implements IC{
    public String name;

    public Person(){}

    public Person(String name){
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                '}';
    }

    @Override
    public Person constructor(String name) {
        return null;
    }
}
```

```java
 @FunctionalInterface
public interface IC {
    Person constructor(String name);
}
```

## Common Method Reference

```java
public class MethodReference  {
    public static void main(String[] args) {
        // lambda implement interface method
        ID id = (str) -> {
            CommonMethod commonMethod = new CommonMethod();
            commonMethod.printUpperCaseString(str);
        };
        id.printString("Banne");

        // common method implement  对象名 :: 方法名
        CommonMethod commonMethod = new CommonMethod();
        ID id1 = commonMethod::printUpperCaseString;
        id1.printString("Harvey");
    }
}
class CommonMethod{
   public void printUpperCaseString(String str){
       System.out.println(str.toUpperCase());
   }
}
```

```java

@FunctionalInterface
public interface ID {
    void printString(String str);
}
```

## This Method Reference

```java
class A {
    void aShow(String str) {
        System.out.println(str);
    }
    
    void test() {
        //  lambda implement interface method
        IA ia1 = (str) -> {
            this.aShow(str);
        };
        ia1.iaShow("hello world");

        // this method implement     this :: 方法名
        IA ia2 = this::aShow;
        ia2.iaShow("hello world");
    }
}
```

```java
@FunctionalInterface
interface IA {
    void iaShow(String str);
}
```

## Array Method Reference

```java
public class ArrayMethodReference {
    public static void main(String[] args) {

        // Lambda implement interface
        int[] arr = createArray(10,length -> {
           return new int[length];
        });

        System.out.println(arr.length);

        // array method implement   数组的方法引用: 有且只有构造器的引用(int[] :: new),没有普通方法的引用
        int [] arr1 = createArray(20,int[]::new);
        System.out.println(arr1.length);
    }

    public static int[] createArray(int length,IE ie){
        return ie.arrayLength(length);
    }
}
```

```java
@FunctionalInterface
public interface IE {
    int [] arrayLength(int length);
}
```

# Stream流

简介 : Stream流,专门对于容器对象(Collection,Map)的`聚合操作`(一系列的基本操作),提供`串行/并行`两种模式,使用`Fork / Join框架`(分合操作)拆分人物,提高编程效率 可读性;

```java
public class StreamTest {
    public static void main(String[] args) {
        // 通过 Stream 流式操作,1.找出集合中名字为张,2.长度为3的人,3.打印;
        ArrayList<String> arrayList = new ArrayList();
        arrayList.add("小白");
        arrayList.add("张三四");
        arrayList.add("张雪峰");
        arrayList.add("李小白");

        // Stream 流式操作
        arrayList.stream()
                 .filter(name ->name.contains("张")) // intermediate   返回新 Stream 流
                 .filter(name -> name.length() == 3) // intermediate
                 .forEach(name -> System.out.println(name)); // terminal
    }
}
```

*Stream流使用步骤 :*

1. 获取流
2. 中间操作(延迟处理)  intermediate(Lazy)
3. 终结操作 terminal

![image-20230704145554445](https://banne.oss-cn-shanghai.aliyuncs.com/Java/image-20230704145554445.png)

## Get Stream

```java
public class GetStream {
    public static void main(String[] args) {
        // List --> Stream
        List<String> list = new ArrayList<String>();
        Stream stream1 = list.stream();

        // Set --> Stream
        Set<String> set = new HashSet<String>();
        Stream<String> stream2 = set.stream();

        // Map keySet --> Stream
        Map<String,String> map = new HashMap<String,String>();
        Set<String> keySet = map.keySet();
        Stream<String> stream3 = keySet.stream();

        // Map valueSet -> Stream
        Stream<Integer> stream4 = new HashMap<String, Integer>().values().stream();

        // Map entrySet -> Stream
        Stream<Map.Entry<String, Integer>> stream5 = new HashMap<String, Integer>().entrySet().stream();

        // array --> Stream
        Integer[] arr = {10,20,30,40};
        // Stream Stream.of(array)
        Stream<Integer> stream6 = Stream.of(arr);
        
    }
}
```

## Intermediate

Intermediate: 中间操作(存在延迟)

*常见的中间操作 :*

- filter()   map()   concat() distinct() 
- sorted() peek() limit()   skip()

### filter()

```java
public class IntermediateTest {
    public static void main(String[] args) {
        // Stream filter(Lambda表达式) 过虑操作
        ArrayList<String> arrayList = new ArrayList<>();
        arrayList.add("Banne");
        arrayList.add("Harvey");
        arrayList.add("Hermione");
            
        arrayList.stream()
                 .filter(name -> name.contains("H")) // 通过 filter() 方法,过滤出包含 H 的名字;return Stream流
                 .forEach(name -> System.out.println(name)); 
    }
}
```

### map()

```java
public class IntermediateTest {
    public static void main(String[] args) {
        // Stream filter(Lambda表达式) 过虑操作
        ArrayList<String> arrayList = new ArrayList<>();
        arrayList.add("Banne");
        arrayList.add("Harvey");
        arrayList.add("Hermione");
        
        // Stream map(Lambda表达式)
        arrayList.stream()
                .map(String::toUpperCase) // 通过 map()方法,可改变原始流数据的形式或者类型
                .forEach(System.out::println);
    }
}
```

### skip()

```java
    public static void main(String[] args) {
        // Stream skip(long n)
        Stream<String> stream = Stream.of("Banne", "Hermione", "Ron");
        // 跳过前两个 2 位 
        stream.skip(2).forEach(System.out::println);
    }
```

### concat()

```java
    public static void main(String[] args) {
        // static Stream  concat(Stream1,Stream2)
        Stream<String> stream1 = Stream.of("你", "好");
        Stream<String> stream2 = Stream.of("呀", "!");
        // concat()方法 实现拼接操作
        Stream.concat(stream1,stream2).forEach(System.out::print);// 你好呀!
    }
```

## Terminate

Terminate: 最终操作 (执行完最终操作后,数据流的数据才会被全面处理)

*常见的最终操作 :*

- forEach(), collect(), count(), toArray(), reduce(),
-  min(), max(), iterator(), anyMatch(), findAny()

### forEach()

```java
// 遍历数据流处理过后的数据
stream.forEach(System.out::println);
```

### count()

```java
 // long count() 统计数据流里面的数据个数
long count = stream.count();
```

### collect()

Stream -> List

```java
List<String> list = stream.collect(Collectors.toList());
```

Stream -> Set

```java
Set<String> set = stream.collect(Collectors.toSet());
```

Stream -> Map

```java
Stream<String> stream = Stream.of(new String[]{"Banne:18", "Harvey:20", "Ron:22"});

// (str) -> str.split(":")[0] 作 key, (str) -> Integer.parseInt(str.split(":")[1] 作 val
Map<String, Integer> map = stream.collect(Collectors.toMap((str) -> {
    return str.split(":")[0];
}, (str) -> {
    return Integer.parseInt(str.split(":")[1]);
}));
```

