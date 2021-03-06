---
layout: post
title: java8
date: 2015-05-20
categories: java
tags:
    - java8
    - Predicate
    - Function
    - Supplier
    - Consumer
    - Comparator
    - Optional
    - Stream
---

java8函数式编程里一句话，觉得说的特好：函数式编程是对行为进行抽象。

### 接口可以有实现了, default和static

#### 为什么要有default方法?

java8中Collection接口中增加了stream方法，这就意味着所有Collection的子类都要实现该接口方法，对于核心类库，实现该方法即可，但是对于第三方的类库，eg：MyArrayList都要实现么，这太坑爹了，于是引入了default方法，你没有我给你提供个默认的，保证二进制兼容性（你用jdk1的javac上编译过能运行的代码，用jdk8也能编译过能运行）。

#### 这样接口就可以有实现了。

eg:

	interface Man {
		// adds a java 8 default method
		default void sayHello() {
	        System.out.println("Hello");
	    }
	    static void sayBye(){
	        System.out.println("byebye");
	    }
	}

这样就可以实现多继承了。

但是问题又来了

1.	如果实现了两个接口，两个接口包含了同名的方法。解决办法：实现类覆盖该方法

2.	如果继承了父类，实现了接口，父类和接口包含了同名的方法。父类方法优先

那么带有default方法的接口与抽象类有什么区别呢？

1.	抽象类可以有成员变量,接口不能

2.	接口可以用于lamda表达式，抽象类不能

注：default方法不能覆盖toString，hashcode和equals方法

eg：

	interface Man {
	    default void sayHello() {
	        System.out.println("Hello");
	    }
	    default void sayBye() {
	        System.out.println("Bye");
	    }
	//  default void toString(){
	//      System.out.println("hehda");
	// }
	}

	interface Woman {
	    default void sayHello() {
	        System.out.println("Hi");
	    }
	}

	class Human{
	    public void sayHello(){
	        System.out.println("haha");
	    }
	}

	class Fool extends Human implements Man, Woman {
	    public void sayHello() {
	        Man.super.sayHello();
	    }
	}
	interface FunctionInterface{
	    String hehe(Integer a);
	//    String haha(Integer b);
	}
	class Fool2 extends Human implements Man{}

	public class MultiExtend {
	    public static void main(String[] args) {
	        new Fool().sayHello();
	        new Fool().sayBye();
	        new Fool2().sayHello();
		FunctionInterface test = String::valueOf;

	    }
	}

#### java8中增加的一些default和static方法

1. Collection中增加了stream default方法
2. Iterator中增加了forEach方法
3. ...

### 增加@FunctionalInterface注解

1.	接口可以有实现
2.	如果接口只有一个非static，非default的方法，那么这个接口就是函数式的接口
3.	可以有这几种声明方式：小括号、类型、参数名、花括号都可能是没有的！
	*   正常是这样的:  `函数式接口 = (类型 参数名) -> { 函数实现};`
	*	如果接口声明中指定了类型，实现中可以不指定类型,会自行推断`Function<Integer,Integer> = (i) -> i + 1;`
	*   当只有一个参数时,且可以不指定类型时，可以不要小括号`Function<Integer,Integer> = i -> i + 1;`
    *   如果方法体只有一行,可以不要大括号
	*   `方法引用`: 如果是某个类的某个方法和接口 与函数式接口的入参,返回值一样,可以使用::来指定

eg:

	//1.param 可以不用指定类型，会自行推断
	aInterface = (param) ->{
	   函数实现;
	   return  result;
	}
	//2.方法体只有一行代码，可以省略大括号的
	aInterface = (param) -> 方法体;
	//3.允许你用::关键字来传递方法或者构造函数的引用
	converter = Integer::valueOf;
	@FunctionalInterface
	interface Converter<F, T> {
	    T convert(F from);

	    default int hehe(F from) {
	        System.out.println("jianzhiheheda");
	        return 0;
	    }

	    // 静态方法中不能有范型,你能在编译期就确定类型么？
	    // static String haha(F f){
	    //
	    // }
	    static int haha(int from, int to) {
	        return (from + to);
	    }
	}

	@FunctionalInterface
	interface Converter2<F, T, R> {
	    R convert(F from1, T from2);
	}

	public class FunctionInterfaceTest {

	    public static void main(String[] args) {
	        Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
	        converter = (from) -> {
	            return Integer.valueOf(from);
	        };
	        converter = Integer::valueOf;
	        Integer converted = converter.convert("123");

	        Converter2<String, String, Integer> converter2 = (from1, from2) -> from1.compareTo(from2);
	        int i = converter2.convert("a", "b");
	    }
	}

### 部分函数式接口

#### Predicate

1.	抽象方法: `test`
2.	default方法: `and`,`or`,`negate`
3.	static方法: `isEqual`（是否看到了guava的影子）

eg:

	// Predicate
	Predicate<String> predicate = (s) -> s.length() > 0;
	predicate.test("foo");
	predicate.negate().test("foo");
	Predicate<Boolean> nonNull = Objects::nonNull;
	Predicate<Boolean> isNull = Objects::isNull;
	Predicate<String> isEmpty = String::isEmpty;
	Predicate<String> isNotEmpty = isEmpty.negate();
	nonNull.and(isNull).test(false);
	nonNull.or(isNull).test(true);

#### Function

1.	抽象方法:`apply`
2.	default方法:`compose`和`andThen`
3.	static方法:`identity`

eg:

	// Function
	Function<String, Integer> toInteger = Integer::valueOf;
	toInteger.apply("123");
	Function<String, String> backToString = toInteger.andThen(String::valueOf);
	backToString.apply("123");

#### Supplier

抽象方法`get`

eg:

	Supplier personSupplier = Person::new;
	personSupplier.get();
	Person person = personSupplier.get();

#### Consumer

1.	抽象方法: `accept`
2.	default方法: `andThen`

eg:

	// consumer
	Consumer<Person> greeter = (p) -> System.out.println("hello " + p.firstName);
	greeter.accept(new Person("Luke", "Skywalker"));


#### Comparator

1.	抽象方法:`compare`
2.	default方法: ...
3.	static方法: ...

eg:

	// comparator
	Comparator comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);
	Person p1 = new Person("John", "Doe");
	Person p2 = new Person("Alice", "WonderLand");
	comparator.compare(p1, p2);
	comparator.reversed().compare(p1, p2);

### 除此之外还有

#### Optional

替换null值。

eg:

	// Optional
	Optional<String> optional = Optional.of("bam");
	optional.isPresent();
	optional.get();
	optional.orElse("fallBack");
	optional.ifPresent((s) -> System.out.println(s.charAt(0)));

#### Stream

1. `高阶函数`是指接受另外一个函数作为参数或者返回一个函数的函数。高阶函数不难辨认：看函数签名就够了，如果函数的参数列表里包含函数接口，或该函数返回一个函数接口，那么该函数就是高阶函数。
stream接口中的大部分函数都是高阶函数。

2. 像filter这样的只描述stream，最终不产生新集合的方法叫做`惰性求值`方法，而像count这样最终会从stream产生值的方法叫做`及早求值`方法。判断一个操作是惰性求值还是及早求值很简单：只需看它的返回值。
   惰性： map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered
   及早： forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator

3. 除了Stream还有ParallelStream并行流


常用的流操作：

1. collect(toList()) : 由stream里的值生成一个列表，是一个及早求值操作`Stream.of("a","b","c").collect(Collectors.toList())`
2. map : 将一种类型的值转换成另外一种类型，将一个流转换成一个新的流
3. filter : 遍历数据并检查其中的元素
4. flatMap : 可用Stream替换值，然后将多个Stream连接成一个Stream, 打平。
5. max、min、count : 最大值，最小值，统计
6. reduce : 实现从一组值中生成一个值，参数是一个BinaryOperator, max、min、count都是reduce操作，只是因为太常用了，而被纳入了标准库。
7. 数据分块: Collectors.partitioningBy 传入一个Predicate，操作的是stream最基本的元素，返回key包含true和false的两个kv的map。
8. 数据分组：Collectors.groupingBy传入一个Function，操作的是stream最基本的元素，返回以Function计算得出的结果为key的map。
9. 字符串： Collectors.joining, 三种形式的。
eg:

	// Stream
	List<String> stringCollection = new ArrayList<>();
	stringCollection.add("ddd2");
	stringCollection.add("aaa2");
	stringCollection.add("bbb1");
	stringCollection.add("aaa1");
	stringCollection.add("bbb3");
	stringCollection.add("ccc");
	stringCollection.add("bbb2");
	stringCollection.add("ddd1");

    // Stream的工厂方法
    Stream.of("ddd2","aaa2","bbb1","aaa1","bbb3","ccc","bbb2","ddd1").collect(Collectors.toList());

	// Filter
	stringCollection.stream().filter((s) -> s.startsWith("a")).forEach(System.out::println);

	// sort
	stringCollection.stream().sorted().filter((s) -> s.startsWith("a")).forEach(System.out::println);

	// map
	stringCollection.stream().map(String::toUpperCase).sorted((a, b) -> b.compareTo(a))
	        .forEach(System.out::println);

	// match
	boolean anyStartsWithA = stringCollection.stream().anyMatch((s) -> s.startsWith("a"));
	boolean allStartsWithA = stringCollection.stream().allMatch((a) -> a.startsWith("a"));
	boolean noneStartWithZ = stringCollection.stream().noneMatch((s) -> s.startsWith("z"));

	// count
	long startWithB = stringCollection.stream().filter((s) -> s.startsWith("b")).count();

	// reduce
	Optional<String> reduced = stringCollection.stream().sorted().reduce((s1, s2) -> s1 + "#" + s2);
	reduced.ifPresent(System.out::println);

ps:
java8中ThreadLocal中增加了一个工厂方法，接受一个lambda表达式，并产生一个新的ThreadLocal对象

    ThreadLocal<String> source = ThreadLocal.withInitial(() -> "mtp");


### 参考

[java8函数式编程](https://book.douban.com/subject/26346017/)

[Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/)
