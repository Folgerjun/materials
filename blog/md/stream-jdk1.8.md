> 别处看到的文章，对其再次进行了整理。收获很多。

## Stream简介
Stream 作为 Java 8 的一大亮点，它与 java.io 包里的 InputStream 和 OutputStream 是完全不同的概念。它也不同于 StAX 对 XML 解析的 Stream，也不是 Amazon Kinesis 对大数据实时处理的 Stream。Java 8 中的 Stream 是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)。Stream API 借助于同样新出现的 Lambda 表达式，极大的提高编程效率和程序可读性。同时它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用 fork/join 并行方式来拆分任务和加速处理过程。通常编写并行代码很难而且容易出错, 但使用 Stream API 无需编写一行多线程的代码，就可以很方便地写出高性能的并发程序。所以说，Java 8 中首次出现的 java.util.stream 是一个函数式语言+多核时代综合影响的产物。

## 什么是聚合操作
在传统的J2EE应用中，Java代码经常不得不依赖于关系型数据库的聚合操作来完成某些操作，诸如：

- 客户每月平均消费金额
- 最昂贵的在售商品
- 本周完成的有效订单（排除了无效的）
- 取十个数据样本作为首页推荐

但在当今这个数据大爆炸的时代，在数据来源多样化、数据海量化的今天，很多时候不得不脱离RDBMS,或者以底层返回的数据为基础进行更上层的数据统计。而Java的集合API中，仅仅有极少量的辅助性方法，更多的时候是程序员需要用Iterator来遍历集合，完成相关的聚合应用逻辑。这是一种远不够高效、笨拙的方法。在Java7中，如果要发现type为grocery的所有交易，然后返回以交易值降序排序好的交易ID集合，我们需要这样写：

### Java7的排序、取值实现
```
List<Transaction> groceryTransactions = new Arraylist<>();

for(Transaction t: transactions){

 if(t.getType() == Transaction.GROCERY){

 groceryTransactions.add(t);

 }

}

Collections.sort(groceryTransactions, new Comparator(){

 public int compare(Transaction t1, Transaction t2){

 return t2.getValue().compareTo(t1.getValue());

 }

});

List<Integer> transactionIds = new ArrayList<>();

for(Transaction t: groceryTransactions){

 transactionsIds.add(t.getId());

}
```

### Java8的排序、取值实现
Java8中使用Stream，代码更加简洁易读，而且使用并发模式，程序执行速度更快。
```
List<Integer> transactionsIds = transactions.parallelStream().

 filter(t -> t.getType() == Transaction.GROCERY).

 sorted(comparing(Transaction::getValue).reversed()).

 map(Transaction::getId).

 collect(toList());
```

## Stream总览
### 什么是流
Stream不是集合元素，它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的Iterator。原始版本的Iterator，用户只能显式地一个一个遍历元素并对其执行某些操作；高级版本的Stream，用户只要给出需要对其包含的元素执行什么操作，比如“过滤掉长度大于10的字符串”、“获取每个字符串的首字母”等，Stream会隐式地在内部进行遍历，做出相应的数据转换。

Stream就如同一个迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。

而和迭代器又不同的是，Stream可以并行化操作，迭代器只能命令式地、串行化操作。顾名思义，当使用串行方法去遍历时，每个item读完后再读下一个item。而使用并行去遍历时，数据会被分成多段，其中每一个都在不同的线程中处理，然后将结果一起输出。Stream的并行操作依赖于Java7中引入的Fork/Join框架（JSR166y）来拆分任务和加速处理过程。Java的并行API演变历程基本如下：

- 1.0-1.4中的java.lang.Thread
- 5.0中的java.util.concurrent
- 6.0中的Phasers等
- 7.0中的Fork/Join框架
- 8.0中的Lambda

**Stream的另外一大特点是，数据源本身可以是无限的。***

### 流的构成
当我们使用一个流的时候，通常包括三个基本步骤：

获取一个数据源（source） → 数据转换 → 执行操作获取想要的结果，每次转换原有Stream对象不改变，返回一个新的Stream对象（可以有多次转换），这就允许对其操作可以像链条一样排列，变成一个管道，如下图所示：

**流管道（Stream Pipeline）的构成**

![](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/img001.png)

有多种方式生成 Stream Source：

- 从Collection和数组
    + Collection.stream()
    + Collection.parallelStream()
    + Arrays.stream(T array) or Stream.of()

- 从BufferedReader
    + java.io.BufferedReader.lines()\

- 静态工厂
    + java.util.stream.IntStream.range()
    + java.nio.file.Files.walk()

- 自己构建
    + java.util.Spliterator

- 其它
    + Random.ints()
    + BitSet.stream()
    + Pattern.splitAsStream(java.lang.CharSequence)
    + JarFile.stream()

流的操作类型分为两种：

- Intermediate：一个流可以后面跟随零个或多个intermediate操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。
- Terminal：一个流只能有一个terminal操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个side effect。

在对于一个Stream进行多次转换操作（Intermediate操作），每次都对Stream的每个元素进行转换，而且是执行多次，这样时间复杂度就是N（转换次数）个for循环里把所有操作都做掉的总和么？其实不是这样的，转换操作都是lazy的，多个转换操作只会在Terminal操作的时候融合起来，一次循环完成。我们可以这样简单的理解，Stream里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在Terminal操作的时候循环Stream对应的集合，然后对每个元素执行所有的函数。

还有一种操作被称为short-circuiting。用以指：

- 对于一个intermediate操作，如果它接收的是一个无限大（infinite/unbounded）的Stream，但返回一个有限的新Stream。
- 对于一个terminal操作，如果它接受的是一个无限大的Stream，但能在有限的时间计算出结果。

当操作一个无限大的Stream，而又希望在有限时间内完成操作，则在管道内拥有一个short-circuiting操作是必要非充分条件。

**一个流操作的示例**
```
int sum = widgets.stream()

.filter(w -> w.getColor() == RED)

.mapToInt(w -> w.getWeight())

.sum();
```
stream()获取当前小物件的source,filter和mapToInt为intermediate操作，进行数据筛选和转换，最后一个sum()为terminal操作，对符合条件的全部小物件作重量求和。

## 流的使用详解
简单说，对Stream的使用就是实现一个filter-map-reduce过程，产生一个最终结果，或者导致一个副作用（side effect）。
### 流的构造与转换
下面提供最常见的几种构造Stream的样例。

**构造流的几种常见方法**
```
// 1. Individual values

Stream stream = Stream.of("a", "b", "c");

// 2. Arrays

String [] strArray = new String[] {"a", "b", "c"};

stream = Stream.of(strArray);

stream = Arrays.stream(strArray);

// 3. Collections

List<String> list = Arrays.asList(strArray);

stream = list.stream();
```
需要注意的是，对于基本数值型，目前有三种对应的包装类型Stream：

IntStream、LongStream、DoubleStream。当然我们也可以用Stream<Integer>、Stream<Long>、Stream<Double>，但是boxing和unboxing会很耗时，所以特别为这三种基本数值型提供了对应的Stream。

Java8中还没有提供其它数值型Stream，因为这将导致扩增的内容较多。而常规的数值型聚合运算可以通过上面三种Stream进行。

**数据流的构造**
```
IntStream.of(new int[]{1, 2, 3}).forEach(System.out::println);

IntStream.range(1, 3).forEach(System.out::println);

IntStream.rangeClosed(1, 3).forEach(System.out::println);
```

**流转换为其它数据结构**
```
// 1. Array

String[] strArray1 = stream.toArray(String[]::new);

// 2. Collection

List<String> list1 = stream.collect(Collectors.toList());

List<String> list2 = stream.collect(Collectors.toCollection(ArrayList::new));

Set set1 = stream.collect(Collectors.toSet());

Stack stack1 = stream.collect(Collectors.toCollection(Stack::new));

// 3. String

String str = stream.collect(Collectors.joining()).toString();
```
***注：一个Stream只可以使用一个，以上示例只是为了代码简洁而重复使用数次。***

### 流的操作
接下来，当把一个数据结构包装成Stream后，就要开始对里面的元素进行各类操作了。常见的操作可以归类如下：

- Intermediate：map(mapToint，flatMap等)、filter、distinct、sorted、peek、limit、skip、parallel、sequential、unordered
- Terminal：forEach、forEachOrdered、toArray、reduce、collect、min、max、count、anyMatch、allMatch、noneMatch、findFirst、findAny、iterator
- Short-circuiting：anyMatch、allMatch、noneMatch、findFirst、findAny、limit

接下来下面看一下Stream的比较典型用法。

- map/flatMap

先来看map。如果熟悉scala这类函数式语言，对这个方法应该很了解，它的作用就是把input Stream的每一个元素，映射成output Stream的另外一个元素。

**转换大写**
```
List<String> output = wordList.stream().

map(String::toUpperCase).

collect(Collectors.toList());
```

**平方数**
```
List<Integer> nums = Arrays.asList(1, 2, 3, 4);

List<Integer> squareNums = nums.stream().

map(n -> n * n).

collect(Collectors.toList());
```

从上面例子可以看出，map生成的是个1:1映射，每个输入元素，都按照规则转换成为另外一个元素。还有一些场景，是一对多映射关系的，这时需要flatMap。

**一对多**
```
Stream<List<Integer>> inputStream = Stream.of(

 Arrays.asList(1),

 Arrays.asList(2, 3),

 Arrays.asList(4, 5, 6)

 );

Stream<Integer> outputStream = inputStream.

flatMap((childList) -> childList.stream());
```
flatMap把input Stream中的层级结构扁平化，就是将最底层元素抽出来放到一起，最终output的新Stream里面已经没有List了，都是直接的数字。

- filter

filter对原始Stream进行某项测试，通过测试的元素被留下来生成一个新Stream。

**留下偶数**
```
Integer[] sixNums = {1, 2, 3, 4, 5, 6};

Integer[] evens =

Stream.of(sixNums).filter(n -> n%2 == 0).toArray(Integer[]::new);
```
经过条件“被2整除”的filter，剩下的数字为{2,4,6}。

**把单词挑出来**
```
List<String> output = reader.lines().

flatMap(line -> Stream.of(line.split(REGEXP))).

filter(word -> word.length() > 0).

collect(Collectors.toList());
```
这段代码首先把每行的单词用flatMap整理到新的Stream，然后保留长度不为0的，就是整篇文章中的所有单词了。

- forEach

forEach方法接收一个Lambda表达式，然后在Stream的每一个元素上执行该表达式。

**打印姓名（forEach和pre-java8的对比）**
```
// Java 8

roster.stream()

 .filter(p -> p.getGender() == Person.Sex.MALE)

 .forEach(p -> System.out.println(p.getName()));

// Pre-Java 8

for (Person p : roster) {

 if (p.getGender() == Person.Sex.MALE) {

 System.out.println(p.getName());

 }

}
```
对一个人员集合遍历，找出男性并打印姓名。可以看出来，forEach是为Lambda而设计的，保持了最紧凑的风格。而且lambda表达式本身是可以重用的，非常方便。当需要为多核系统优化时，可以parallelStream().forEach()，只是此时原有元素的次序没法保证，并行的情况下将改变串行时操作的行为，此时forEach本身的实现不需要调整，而Java8以前的for循环code可能需要加入额外的多线程逻辑。

但一般认为，forEach和常规for循环的差异不涉及到性能，它们仅仅是函数式风格与传统Java风格的差别。

另外一点需要注意，forEach是terminal操作，因此它执行后，Stream的元素就被“消费”掉了，你无法对一个Stream进行两次terminal运算。下面的代码是错误的：
```
stream.forEach(element -> doOneThing(element));

stream.forEach(element -> doAnotherThing(element));
```
相反，具有相似功能的intermediate操作peek可以达到上述目的。如下是出现在该api javadoc上的一个示例。

**peek 对每个元素执行操作并返回一个新的 Stream**
```
Stream.of("one", "two", "three", "four")

.filter(e -> e.length() > 3)

.peek(e -> System.out.println("Filtered value: " + e))

.map(String::toUpperCase)

.peek(e -> System.out.println("Mapped value: " + e))

.collect(Collectors.toList());
```
forEach不能修改自己包含的本地变量值，也不能用break/return之类的关键字提前结束循环。

- findFirst

这是一个terminal兼short-circuiting操作，它总是返回Stream的第一个元素，或者空。

这里比较重点的是它的返回值类型：Optional。这也是一个模仿Scala语言中的概念，作为一个容器，它可能含有某值，或者不包含。使用它的目的是尽可能避免NullPointerException。

**Optional的两个用例**
```
String strA = " abcd ", strB = null;

print(strA);

print("");

print(strB);

getLength(strA);

getLength("");

getLength(strB);

public static void print(String text) {

 // Java 8

 Optional.ofNullable(text).ifPresent(System.out::println);

 // Pre-Java 8

 if (text != null) {

 System.out.println(text);

 }

 }

public static int getLength(String text) {

 // Java 8

return Optional.ofNullable(text).map(String::length).orElse(-1);

 // Pre-Java 8

// return if (text != null) ? text.length() : -1;

 };
```
在更复杂的if(xx != null)的情况中，使用Optional代码的可读性更好，而且它提供的是编译时检查，能极大的降低NPE这种Runtime Exception对程序的影响，或者迫使程序员更早的在编码阶段处理空值问题，而不是留到运行时再发现和调试。

Stream中的findAny、max/min、reduce等方法返回Optional值。还有例如IntStream.average()返回Optional Double等等。

- reduce

这个方法的主要作用是把Stream元素组合起来。它提供一个起始值（种子），然后依照运算规则（BinaryOperator），和前面Stream的第一个、第二个、第n个元素组合。从这个意义上说，字符串拼接、数值的sum、min、max、average都是特殊的reduce。例如Stream的sum就相当于
```
Integer sum = integers.reduce(0,(a,b) -> a+b);
```
或
```
Integer sum = integers.reduce(0,Integer::sum);
```
也有没有起始值的情况，这时会把Stream的前面两个元素组合起来，返回的是Optional。

**reduce的用例**
```
// 字符串连接，concat = "ABCD"

String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat);

// 求最小值，minValue = -3.0

double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min);

// 求和，sumValue = 10, 有起始值

int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);

// 求和，sumValue = 10, 无起始值

sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();

// 过滤，字符串连接，concat = "ace"

concat = Stream.of("a", "B", "c", "D", "e", "F").

 filter(x -> x.compareTo("Z") > 0).

 reduce("", String::concat);
```
上面代码例如第一个示例的reduce(),第一个参数（空白字符）即为起始值，第二个参数（String::concat）为BinaryOperator。这类有起始值的reduce()都返回具体的对象。而对于第四个示例没有起始值的reduce(),由于可能没有足够的元素，返回的是Optional，请留意这个区间。

- limit/skip

limit返回Stream的前面n个元素；skip则是扔掉前n个元素（它是由一个叫subStream的方法改名而来）。

**limit和skip对运行次数的影响**
```
public void testLimitAndSkip() {

 List<Person> persons = new ArrayList();

 for (int i = 1; i <= 10000; i++) {

 Person person = new Person(i, "name" + i);

 persons.add(person);

 }

List<String> personList2 = persons.stream().

map(Person::getName).limit(10).skip(3).collect(Collectors.toList());

System.out.println(personList2);

}

private class Person {

 public int no;

 private String name;

 public Person (int no, String name) {

 this.no = no;

 this.name = name;

 }

 public String getName() {

 System.out.println(name);

 return name;

 }

}
```
**结果为：**
```
name1

name2

name3

name4

name5

name6

name7

name8

name9

name10

[name4, name5, name6, name7, name8, name9, name10]
```

这是一个有10000个元素的Stream，但在short-circuiting操作limit和skip的作用下，管道中map操作指定的getName()方法的执行次数为limit所限定的10次，而最终返回结果在跳过前3个元素后只有后面7个返回。

还有一种情况是limit/skip无法达到short-circuiting目的地，就是把它们放在Stream的排序操作后，原因跟sorted这个intermediate操作有关：此时系统并不知道Stream排序后的次序如何，所以sorted中的操作看上去就像完全没有被limit或者skip一样。

**limit和skip对sorted后的运行次数无影响**
```
List<Person> persons = new ArrayList();

 for (int i = 1; i <= 5; i++) {

 Person person = new Person(i, "name" + i);

 persons.add(person);

 }

List<Person> personList2 = persons.stream().sorted((p1, p2) ->

p1.getName().compareTo(p2.getName())).limit(2).collect(Collectors.toList());

System.out.println(personList2);
```
首先对5个元素的Stream排序，然后进行limit操作。输出结果为：
```
name2

name1

name3

name2

name4

name3

name5

name4

[stream.StreamDW$Person@816f27d, stream.StreamDW$Person@87aac27]
```
即虽然最后的返回元素数量是2，但整个管道中的sorted表达式执行次数没有像之前示例一样相应减少。

最后有一种需要注意的是，对一个parallel的Stream管道来说，如果其元素是有序的，那么limit操作的成本会比较大，因为它的返回对象必须是前n个也有一样次序的元素。取而代之的策略是取消元素间的次序，或者不要用parallel Stream。

- sorted

对Stream的排序通过sorted进行，它比数组的排序更强之处在于你可以首先对Stream进行各类map、filter、limit、skip甚至distinct来减少元素数量后，再排序，这能帮助程序明显缩短执行时间。对之前示例可进行优化：

**排序前进行 limit 和 skip**
```
List<Person> persons = new ArrayList();

 for (int i = 1; i <= 5; i++) {

 Person person = new Person(i, "name" + i);

 persons.add(person);

 }

List<Person> personList2 = persons.stream().limit(2).sorted((p1, p2) -> p1.getName().compareTo(p2.getName())).collect(Collectors.toList());

System.out.println(personList2);
```
**结果为：**
```
name2

name1

[stream.StreamDW$Person@6ce253f1, stream.StreamDW$Person@53d8d10a]
```
当然这种优化是有business logic上的局限性的：即不要求排序后再取值

- min/max/distinct

min和max的功能也可以通过对Stream元素先排序，再findFirst来实现，但前者的性能会更好，为O(n)，而sorted的成本是O(n log n)。同时它们作为特殊的reduce方法被独立出来也是因为求最大最小值是很常见的操作。

**找出最长一行的长度**
```
BufferedReader br = new BufferedReader(new FileReader("c:\\SUService.log"));

int longest = br.lines().

 mapToInt(String::length).

 max().

 getAsInt();

br.close();

System.out.println(longest);
```

**找出全文的单词，转小写，并排序**
```
List<String> words = br.lines().

 flatMap(line -> Stream.of(line.split(" "))).

 filter(word -> word.length() > 0).

 map(String::toLowerCase).

 distinct().

 sorted().

 collect(Collectors.toList());

br.close();

System.out.println(words);
```

- Match

Stream有三个match方法，从语义上说：

    allMatch：Stream中全部元素符合传入的predicate，返回true
    anyMatch：Stream中只要有一个元素符合传入的predicate，返回true
    noneMatch：Stream中没有一个元素符合传入的predicate，返回true

它们都不是要遍历全部元素才能返回结果。例如allMatch只要一个元素不满足条件，就skip剩下的所有元素，返回false。

**使用Match**
```
List<Person> persons = new ArrayList();

persons.add(new Person(1, "name" + 1, 10));

persons.add(new Person(2, "name" + 2, 21));

persons.add(new Person(3, "name" + 3, 34));

persons.add(new Person(4, "name" + 4, 6));

persons.add(new Person(5, "name" + 5, 55));

boolean isAllAdult = persons.stream().

 allMatch(p -> p.getAge() > 18);

System.out.println("All are adult? " + isAllAdult);

boolean isThereAnyChild = persons.stream().

 anyMatch(p -> p.getAge() < 12);

System.out.println("Any child? " + isThereAnyChild);
```

**输出结果：**
```
All are adult? false

Any child? true
```

## 进阶：自己生成流

- Stream.generate

通过实现Supplier接口，你可以自己来控制流的生成。这种情形通常用于随机数、常量的Stream，或者需要前后元素间维持着某种状态信息的Stream。把Supplier实例传递给Stream.generate()生成的Stream，默认是串行（相对parallel而言）但无序的（相对ordered而言）。由于它是无限的，在管道中，必须利用limit之类的操作限制Stream大小。

**生成10个随机整数**
```
Random seed = new Random();

Supplier<Integer> random = seed::nextInt;

Stream.generate(random).limit(10).forEach(System.out::println);

//Another way

IntStream.generate(() -> (int) (System.nanoTime() % 100)).

limit(10).forEach(System.out::println);
```
Stream.generate()还接受自己实现的Supplier。例如在构造海量测试数据的时候，用某种自动的规则给每一个变量赋值；或者依据公式计算Stream的每个元素值。这些都是维持状态信息的情形。

**自实现Supplier**
```
Stream.generate(new PersonSupplier()).

limit(10).

forEach(p -> System.out.println(p.getName() + ", " + p.getAge()));

private class PersonSupplier implements Supplier<Person> {

 private int index = 0;

 private Random random = new Random();

 @Override

 public Person get() {

 return new Person(index++, "StormTestUser" + index, random.nextInt(100));

 }

}
```

**输出结果：**
```
StormTestUser1, 9

StormTestUser2, 12

StormTestUser3, 88

StormTestUser4, 51

StormTestUser5, 22

StormTestUser6, 28

StormTestUser7, 81

StormTestUser8, 51

StormTestUser9, 4

StormTestUser10, 76
```

- Stream.iterate

iterate跟reduce操作很像，接受一个种子值，和一个UnaryOperator（例如f）。然后种子值成为Stream的第一个元素，f(seed)为第二个，f(f(seed))第三个，以此类推。

**生成一个等差数列**
```
Stream.iterate(0, n -> n + 3).limit(10). forEach(x -> System.out.print(x + " "));
```

**输出结果：**
```
0 3 6 9 12 15 18 21 24 27
```
与Stream.generate相仿，在iterate时候管道必须有limit这样的操作来限制Stream大小。


## 进阶：用Collectors来进行reduction操作

java.util.stream.Collectors类的主要作用就是辅助进行各类有用的reduction操作，例如转变输出为Collection，把Stream元素进行归组，

- groupingBy/partitioningBy

**按照年龄归组**
```
Map<Integer, List<Person>> personGroups = Stream.generate(new PersonSupplier()).

 limit(100).

 collect(Collectors.groupingBy(Person::getAge));

Iterator it = personGroups.entrySet().iterator();

while (it.hasNext()) {

 Map.Entry<Integer, List<Person>> persons = (Map.Entry) it.next();

 System.out.println("Age " + persons.getKey() + " = " + persons.getValue().size());

}
```
上面的示例，首先生成100人的信息，然后按照年龄归组，相同年龄的人放到同一个list中，可以看到如下的输出：
```
Age 0 = 2

Age 1 = 2

Age 5 = 2

Age 8 = 1

Age 9 = 1

Age 11 = 2

……
```

**按照未成年人和成年人归组**
```
Map<Boolean, List<Person>> children = Stream.generate(new PersonSupplier()).

 limit(100).

 collect(Collectors.partitioningBy(p -> p.getAge() < 18));

System.out.println("Children number: " + children.get(true).size());

System.out.println("Adult number: " + children.get(false).size());
```

**输出结果：**
```
Children number: 23

Adult number: 77
```
在使用条件“年龄小于18”进行分组后可以看到，不到18岁的未成年人是一组，成年人是另外一组。partitioningBy其实是一种特殊的groupingBy，它依照条件测试的是否两种结果来构造返回的数据结构，get(true)和get(false)能即为全部的元素对象。

## 结束语
总之，Stream的特性可以归纳为：

- 不是数据结构
- 它没有内部存储，它只是用操作管道从source（数据结构、数组、generator function、IO channel）抓取数据
- 它也绝不修改自己所封装的底层数据结构的数据。例如Stream的filter操作会产生一个不包含被过滤元素的新Stream，而不是从source删除那些元素
- 所有Stream的操作必须以lambda表达式为参数
- 不支持索引访问
- 你可以请求第一个元素，但无法请求第二个，第三个或者最后一个。不过请参阅下一项
- 很容易生成数组或者List
- 惰性化
- 很多Stream操作是向后延迟的，一直到它弄清楚了最后需要多少数据才会开始
- Intermediate操作永远是惰性化的
- 并行能力
- 当一个Stream是并行化的，就不需要再写多线程代码，所有对它的操作会自动并行进行的
- 可以是无限的
- 集合有固定大小，Stream则不必。limit(n)和findFirst()这类的short-circuiting操作可以对无限的Stream进行运算并很快完成