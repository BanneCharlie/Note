# Info

简介 : apache common 提供了许多处理各种数据 文件等的依赖,其中依赖中提供了许多封装好的工具类用于简化代码程序的开发;

官方网址: https://commons.apache.org/

# Commons Lang

简介 : Commons Lang库提供了许多常用的工具类来简化java编程,常用的工具类为 `StringUitls` `NumberUtils` `ObjectUtils` `ArrayUtils`等;

`导入相关依赖`

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>
```

`Test.java` StringUtils 常用的为 isBlank()   isEmpty()

```java
    public void test() {
        String str = "   ";
        
        // isBlank() 判断字符串是否为null / "" / "  "
        boolean result = StringUtils.isBlank(str);
        System.out.println(result); //true
        // isEmpty()  判断字符串是否为 null / ""
        boolean empty = StringUtils.isEmpty(str);
        System.out.println(empty); //false
        // join() 字符串的拼接
        String banne = StringUtils.join(str, "banne");
        System.out.println(banne); //   banne
        String str1 =  "你好,banne,";
        // split() 字符串的分割
        String[] split = StringUtils.split(str1, ",");
        for (String object :split) {
            System.out.println(object); // 你好 banne
        }
    }
```

`Test.java` NumberUtils常用的为 toInt() toDouble() compare() max() min()

```java
public void test1(){
        // toInt()  toDouble() 将字符串转换为数字
        String str = "12.0";
        int anInt = NumberUtils.toInt(str);
        double aDouble = NumberUtils.toDouble(str);
        System.out.println(
                "anInt = " + anInt + ", aDouble = " + aDouble
        );
        // compare() 比较两个数字的大小 num1>num2返回值为1 num1 = num2返回值为0 num1<num2返回值为-1
        System.out.println(NumberUtils.compare(1, 2));
        // 取最大最小值
        int[] array = {1,100,200,2};
        System.out.println(NumberUtils.max(array));
        System.out.println(NumberUtils.min(array));
    }
```

`Test.java`ObjectUtils常用的为 equals()  hashCdoe()

```java
public void test2(){
        String str1 = new String("10");
        String str2 = new String();

        // equals() null安全的对象比较
        System.out.println(ObjectUtils.equals(str1, str2));// true

        // hashCode() 计算对象的哈希码
        System.out.println(ObjectUtils.hashCode(str1));// 1567
    }
```

`Test.java`ArrayUtils常用的 addAll() indexOf() reverse()

```java
    public void test3(){
        int[] arr1 = {1,2,3,4,};
        int[] arr2 = {1,2,3,4,};

        // indexOf() 查找元素在数组中的索引
        System.out.println(ArrayUtils.indexOf(arr1, 1));// 0
        
        // reverse() 将数组进行翻转
        ArrayUtils.reverse(arr1);
    }
```

