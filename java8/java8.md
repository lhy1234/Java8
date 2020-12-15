### 行为参数化

行为参数化，就是一个方法接受多个不同的行为作为参数，并在内部使用它们，完成不同行为的能力。

行为参数化可让代码更好地适应不断变化的要求，减轻未来的工作量

1.用 Comparator 来排序**

初始化数据：

Apple类：

```java
public class Apple {
    private String color;
    private Integer weight;

    public Apple(){}
    public Apple(String color, Integer weight) {
        this.color = color;
        this.weight = weight;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public Integer getWeight() {
        return weight;
    }

    public void setWeight(Integer weight) {
        this.weight = weight;
    }

    @Override
    public String toString() {
        return "Apple{" +
                "color='" + color + '\'' +
                ", weight=" + weight +
                '}';
    }
}

//初始化数据
List<Apple> list = new ArrayList<>();
list.add(new Apple("红色",150));
list.add(new Apple("绿色",100));
list.add(new Apple("灰色",250));
list.add(new Apple("黑色",200));
```



在Java 8中， List 自带了一个 sort 方法，sort 的行为可以用 java.util.Comparator 对象来参数化

```java
//List 的sort方法
default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }

//Comparator接口里的compare方法
public interface Comparator<T> {
	int compare(T o1, T o2);
}
```

排序：

```java
//1.传进入一个java.util.Comparator接口的实现即可
list.sort(new Comparator<Apple>() {
    @Override
    public int compare(Apple o1, Apple o2) {
        return o1.getWeight()-o2.getWeight();
    }
});

//2：使用lambda表达式
list.sort((Apple a1,Apple a2)->a1.getWeight()-a2.getWeight());


//打印结果
[Apple{color='绿色', weight=100}, Apple{color='红色', weight=150}, Apple{color='黑色', weight=200}, Apple{color='灰色', weight=250}]
```

![1606900708820](D:\Z_lhy\STUDY\java8\img\1606900708820.png)

* 参数列表——这里它采用了 Comparator 中 compare 方法的参数，两个 Apple 
* 箭头——箭头 -> 把参数列表与Lambda主体分隔开。
* Lambda主体——比较两个 Apple 的重量。表达式就是Lambda的返回值



### Lambda表达式

```java
Comparator<Apple> byWeight =
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

* 参数列表——这里它采用了 Comparator 中 compare 方法的参数，两个 Apple 

* 箭头——箭头 -> 把参数列表与Lambda主体分隔开。

* Lambda主体——比较两个 Apple 的重量。表达式就是Lambda的返回值

  **Lambda的基本语法：**

  (parameters) -> expression   或者 (parameters) -> { statements; }



![1606901380161](D:\Z_lhy\STUDY\java8\img\1606901380161.png)

#### 在哪里以及如何使用 Lambda

可以在函数式接口上使用Lambda表达式

**函数式接口是什么？**

<font color='red'>函数式接口就是只定义一个抽象方法的接口</font>

比如：

```java
//java.util.Comparator
public interface Comparator<T> {
	int compare(T o1, T o2);
}

public interface Runnable{
	void run();
}
```





Lambda表达式启动线程：

```java
Thread thread = new Thread(()-> System.err.println("x"));
```

这种写法没毛病：

```java
Runnable r = ()-> System.err.println("oooo");
```





### Stream流

 filter 、 map 、 reduce 、 find 、 match 、 sort、limit

很多流操作本身会返回一个流，这样多个操作就可以链接起来

流只能遍历一次，遍历完之后，这个流已经被消费掉了。

```java
Stream<Apple> stream = list.stream();
stream.forEach(item->item.getName());
stream.forEach(item->item.getName());
//java.lang.IllegalStateException: stream has already been operated upon or closed
```

**1，筛选并排序**

```java
public class Apple {

    private String chanDi;//产地
    private String name;
    private Integer weight;
    private Integer tangFen; //糖分
}
//init data
List<Apple> list = new ArrayList<>();
        list.add(new Apple("山东","山东红富士",50,98));
        list.add(new Apple("甘肃","甘肃天水",60,97));
        list.add(new Apple("山东","花牛苹果",70,96));
        list.add(new Apple("山东","山东红富士",80,93));


//选出重量小于80的苹果按照糖分排序
        List<Apple> result = list.stream().filter(a -> a.getWeight() < 80).sorted(Comparator.comparingInt(Apple::getTangFen)).collect(Collectors.toList());

//多核处理
        List<Apple> result2 = list.parallelStream().filter(a -> a.getWeight() < 80).sorted(Comparator.comparingInt(Apple::getTangFen)).collect(Collectors.toList());


//打印
[Apple{chanDi='山东', name='花牛苹果', weight=70, tangFen=96}, Apple{chanDi='甘肃', name='甘肃天水', weight=60, tangFen=97}, Apple{chanDi='山东', name='山东红富士', weight=50, tangFen=98}]

```

**2.分组**

```java
//按产地分组成map，key是产地
Map<String, List<Apple>> map = list.stream().collect(Collectors.groupingBy(Apple::getChanDi));
        

{山东=[Apple{chanDi='山东', name='山东红富士', weight=50, tangFen=98}, Apple{chanDi='山东', name='花牛苹果', weight=70, tangFen=96}, Apple{chanDi='山东', name='山东红富士', weight=80, tangFen=93}], 甘肃=[Apple{chanDi='甘肃', name='甘肃天水', weight=60, tangFen=97}]}
```

**3.筛选+排序+map转换+limit**

```java
//选出重量小于80的苹果按照糖分排序,并返回苹果品种名称,并只选择2个
        List<String> result3 = list
                .stream()//获得流
                .filter(a -> a.getWeight() < 80)//筛选重量<80的
                .sorted(Comparator.comparingInt(Apple::getTangFen))//按糖分排序
                .map(Apple::getName)//只获取名称
                .limit(2)//只选择2个
                .collect(Collectors.toList());//将结果保存在另一个 List 中
//打印
[花牛苹果, 甘肃天水]
```

**4.去重**

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
        numbers.stream()
                .filter(i -> i % 2 == 0)
                .distinct()//去重，根据流所生成元素的hashCode和equals方法实现
                .forEach(System.out::println);
System.err.println(numbers);
//打印
2
4
[1, 2, 1, 3, 3, 2, 4]
```

**5.截短流&跳过元素**

 limit(n)：如果流是有序的，则最多会返回前 n 个元素，无序流上，随机。

 skip(n) 方法，返回一个扔掉了前 n 个元素的流。如果流中元素不足 n 个，则返回一个空流。

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
        List<Integer> limit = numbers.stream()
                .skip(1) //跳过第一个
                .limit(2) //查前俩
                .collect(Collectors.toList());
        System.err.println("-->"+limit.toString());
//-->[2, 1]

```

**映射**

流支持map方法，它会接受一个函数作为参数。这个函数会被应用到每个元素上，并将其映射成一个新的元素

```java
List<String> appleNames = list.stream()
                .map(Apple::getName)
                .collect(Collectors.toList());
        System.err.println(appleNames);
        //[山东红富士, 甘肃天水, 花牛苹果, 山东红富士]

//给定一个单词列表，你想要返回另一个列表，显示每个单词中有几个字母
        List<String> words = Arrays.asList("Java 8", "Lambdas", "In", "Action");
        List<Integer> wordLengths = words.stream()
                .map(String::length)
                .collect(Collectors.toList());
        System.err.println(wordLengths);
        //[6, 7, 2, 6]
```





**数组转化为流 Arrays.stream()**

```java
int[] arr = {1,32,34,6,8};
IntStream stream = Arrays.stream(arr);
String[] strs = {"xx","oo"};
Stream<String> stream2 = Arrays.stream(strs);
```

**使用 flatMap将各个生成流扁平化为单个流**

```java
String[] strArr = {"你好","幸阳","嵩山少年"};
        List<String> collect = Arrays.stream(strArr)
                                        .map(s -> s.split(""))//拆分成List<String[]>
                                        .flatMap(Arrays::stream) //“扁平化”，合并为一个流
                                        .collect(Collectors.toList());
        System.err.println(collect.toString());
        //[你, 好, 幸, 阳, 嵩, 山, 少, 年]
```

**查找和匹配**

数据集中的某些元素是否匹配一个给定的属性, allMatch 、 anyMatch 、 noneMatch 、 findFirst 和 findAny 

**anyMatch有一个匹配**

```java
//anyMatch 有一个匹配
//产地是否有山东的
if(list.stream().anyMatch(apple -> apple.getChanDi().equals("山东"))){
    System.err.println("有产地是山东的水果");
}
```

**allMatch匹配所有**

```java
//allMatch，匹配所有
//产地是否都是山东
if(list.stream().allMatch(apple -> apple.getChanDi().equals("山东"))){
    System.err.println("所有苹果都是山东产的");
}
```

**findAny 短路,找到一个就返回**

```java
//找到第一个产地是山东的，就返回
Apple app = list.stream()
                .filter(apple -> apple.getChanDi().equals("山东"))
                .findAny()//findAny 短路,找到一个就返回
                .orElse(null);//orElse(T other) 会在值存在时返回值，否则返回一个默认值。
        //app.isPresent() 将在 Optional 包含值的时候返回 true , 否则返回 false 。
        //app.get() 会在值存在时返回值
```



**Map-Reduce**

list.stream().reduce() 接受两个参数：

```java
//Map-Reduce
List<Integer> numbers = Arrays.asList(1,3,4,6);
//reduce 接受两个参数：
//1，求和:一个初始值，这里是0；一个 BinaryOperator<T>来将两个元素结合起来产生一个新值，这里我们用的是lambda (a, b) -> a + b 。

int sum = numbers.stream().reduce(0, (a, b) -> a + b);
//也可以写成
int sum2 = numbers.stream().reduce(0, Integer::sum);

//reduce 变体，没有初始值，但是会返回一个 Optional对象
Optional<Integer> sum3 = numbers.stream().reduce((a, b) -> (a + b));

//2.最大值和最小值
Optional<Integer> max = numbers.stream().reduce(Integer::max);//min

//用 map 和 reduce 方法统计流中个数
int reduce = list.stream()
    .map(d -> 1) //把流中每个元素都映射成数字 1 ，
    .reduce(0, (a, b) -> a + b); //然后用 reduce 求和
long count = list.stream().count();//可以用内置的count

```

**实践：**

```java
/**
 * 交易员
 * @author lihaoyang
 * @date 2020/12/14
 */
public class Trader {

    private final String name;
    private final String city;
    public Trader(String n, String c){
        this.name = n;
        this.city = c;
    }
    public String getName(){
        return this.name;
    }
    public String getCity(){
        return this.city;
    }

    @Override
    public String toString(){
        return "Trader:"+this.name + " in " + this.city;
    }
}

/**
 * 交易
 * @author lihaoyang
 * @date 2020/12/14
 */
public class Transaction {
    private final Trader trader;
    private final int year;
    private final int value;
    public Transaction(Trader trader, int year, int value){
        this.trader = trader;
        this.year = year;
        this.value = value;
    }
    public Trader getTrader(){
        return this.trader;
    }
    public int getYear(){
        return this.year;
    }
    public int getValue(){
        return this.value;
    }
    @Override
    public String toString(){
        return "{" + this.trader + ", " +
                "year: "+this.year+", " +
                "value:" + this.value +"}";
    }
}
//
Trader raoul = new Trader("Raoul", "剑桥");//劳尔
        Trader mario = new Trader("Mario","米兰");//马里奥
        Trader alan = new Trader("Alan","剑桥");//艾伦
        Trader brian = new Trader("Brian","剑桥");//布瑞恩
        List<Transaction> transactions = Arrays.asList(
                new Transaction(brian, 2011, 300),
                new Transaction(raoul, 2012, 1000),
                new Transaction(raoul, 2011, 400),
                new Transaction(mario, 2012, 710),
                new Transaction(mario, 2012, 700),
                new Transaction(alan, 2012, 950));

        //(1) 找出2011年发生的所有交易，并按交易额排序（从低到高）
        List<Transaction> transSort = transactions.stream()
                .filter(t -> t.getYear() == 2011)//给filter 传递一个谓词来选择2011年的交易
                .sorted(Comparator.comparingInt(Transaction::getValue))//按照交易额进行排序
                .collect(Collectors.toList());//将生成的 Stream中的所有元素收集到一个List 中
        //System.err.println(transSort);
        //[{Trader:布瑞恩 in Cambridge, year: 2011, value:300}, {Trader:劳尔 in Cambridge, year: 2011, value:400}]


        //(2) 交易员都在哪些不同的城市工作过？
        List<String> cities = transactions.stream()
                .map(p -> p.getTrader().getCity())//提取与交易相关的每位交易员的所在城市
                .distinct() //只选择互不相同的城市
                .collect(Collectors.toList());
        //你可以去掉 distinct() ，改用 toSet() ，这样就会把流转换为集合。
        Set<String> cities2 =
                transactions.stream()
                        .map(transaction -> transaction.getTrader().getCity())
                        .collect(Collectors.toSet());
        System.err.println(cities);

        //(3) 查找所有来自于剑桥的交易员，并按姓名排序。
        List<Trader> traders = transactions.stream()
                .map(Transaction::getTrader)//提取所有交易员
                .filter(t -> t.getCity().equals("剑桥"))//过滤
                .distinct()//去重
                .sorted(Comparator.comparing(Trader::getName))//排序
                .collect(Collectors.toList());
        System.err.println(traders);

        //(4)返回所有交易员的姓名字符串，按字母顺序排序。
        //效率不高，new很多String
        String names = transactions.stream()
                .map(t -> t.getTrader().getName())//提取所有交易员姓名，生成一 Strings 构成的 Stream
                .distinct()//去重
                .sorted()
                .reduce("",(n1,n2)->n1+n2);//逐个拼接每个名字，得到一个将所有名字连接 起来的 String
        //效率高
        String traderStr =
                transactions.stream()
                        .map(transaction -> transaction.getTrader().getName())
                        .distinct()
                        .sorted()
                        .collect(Collectors.joining());

        System.err.println("names:"+traderStr);

        //(5) 有没有交易员是在米兰工作的？
        boolean milan = transactions
                .stream()
                .anyMatch(t -> t.getTrader().getCity().equals("米兰"));
        System.err.println("是否有在米兰:"+milan);

        //(6) 打印生活在剑桥的交易员的所有交易额。
        transactions
                .stream()
                .filter(t -> t.getTrader().getCity().equals("剑桥"))
                .map(Transaction::getValue).forEach(System.out::println);//打印每个值
        //System.err.println("剑桥的交易员的所有交易额:"+jianqiaoValues);

        //(7) 所有交易中，最高的交易额是多少？
        Optional<Integer> max = transactions.stream()
                .map(Transaction::getValue)//提 取 每 项 交 易的交易额
                .reduce(Integer::max);//计算生成的流中的最大值
        System.err.println("最大交易金额:"+max.get());

        //(8)找到交易额最小的交易。
        Optional<Transaction> min = transactions.stream().min(Comparator.comparingInt(Transaction::getValue));
        System.err.println("min:"+min.get());
```











流操作有两类：中间操作和终端操作

```java
//在下列流水线中，你能找出中间操作和终端操作吗？
long count = menu.stream()
                .filter(d -> d.getCalories() > 300)
                .distinct()
                .limit(3)
                .count();
//答案：流水线中最后一个操作 count 返回一个 long ，这是一个非 Stream 的值。因此它是一个终端操作。所有前面的操作， filter 、 distinct 、 limit ，都是连接起来的，并返回一个 Stream ，因此它们是中间操作。
```

**总结：**

总而言之，流的使用一般包括三件事：

  一个数据源（如集合）来执行一个查询；
  一个中间操作链，形成一条流的流水线；（中间操作并不开始计算，直到调用了终端操作）
  一个终端操作，执行流水线，并能生成结果。

![1607926163798](D:\Z_lhy\STUDY\java8\img\1607926163798.png)

### 数值流

Java 8引入了三个原始类型特化流接口来解决这个问题： IntStream 、 DoubleStream 和//LongStream ，分别将流中的元素特化为 int 、 long 和 double ，从而避免了暗含的装箱成本

```java
//这么求和有装箱、拆箱成本，Integer需要转成int再计算，
Integer reduce = list.stream().map(Apple::getWeight).reduce(0, Integer::sum);
```

例子：

提取苹果的重量并求和，mapToInt会返回IntStream,而不是 Stream<Integer>避免装箱拆箱，

如果流是空的， sum 默认返回 0

```java
int sum = list.stream().mapToInt(Apple::getWeight).sum();
System.err.println(sum);
//还有max 、 min 、 average
```



**OptionalInt**

对于求最大值最小值，如果流是空的没有元素，返回默认值0可能不合适，要找到 IntStream 中的最大元素，可以调用 max 方法，它会返回一个 OptionalInt ：

```java
OptionalInt max = list.stream().mapToInt(Apple::getWeight).max();
int maxWeight = max.orElse(1);//没有最大值，默认弄成1
```

