---
title: JDK1.8的新特性
date: 2020-08-18 09:43:28
tags: Java基础
excerpt: Lambda表达式、函数式接口、Optional类的用法
---

## 一、Lambda表达式

### 1.1Lambda简化代码例子
下面就以几个例子来看看Lambda表达式是怎么简化我们代码的编写的。
    
首先我们来看看创建线程：

```
public static void main(String[] args) {
    // 用匿名内部类的方式来创建线程
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("公众号：Java3y---回复1进群交流");
        }
    });

    // 使用Lambda来创建线程
    new Thread(() -> System.out.println("公众号：Java3y---回复1进群交流"));
}
```

再来看看遍历Map集合：

```
public static void main(String[] args) {
    Map<String, String> hashMap = new HashMap<>();
    hashMap.put("公众号", "Java3y");
    hashMap.put("交流群", "回复1");

    // 使用增强for的方式来遍历hashMap
    for (Map.Entry<String, String> entry : hashMap.entrySet()) {
        System.out.println(entry.getKey()+":"+entry.getValue());
    }

    // 使用Lambda表达式的方式来遍历hashMap
    hashMap.forEach((s, s2) -> System.out.println(s + ":" + s2));
}
```

在List中删除某个元素

```
public static void main(String[] args) {

    List<String> list = new ArrayList<>();
    list.add("Java3y");
    list.add("3y");
    list.add("光头");
    list.add("帅哥");

    // 传统的方式删除"光头"的元素
    ListIterator<String> iterator = list.listIterator();
    while (iterator.hasNext()) {
        if ("光头".equals(iterator.next())) {
            iterator.remove();
        }
    }

    // Lambda方式删除"光头"的元素
    list.removeIf(s -> "光头".equals(s));

    // 使用Lambda遍历List集合
    list.forEach(s -> System.out.println(s));
}
```

从上面的例子我们可以看出，Lambda表达式的确是可以帮我们简化代码的。

### 1.2Lambda简单讲解

语法形式为 () -> {}，其中 () 用来描述参数列表，{} 用来描述方法体，-> 为 lambda运算符 ，读作(goes to)。

## 二、接口

接口新增默认方法与静态方法
```
public interface Interface1 {
 
	void method1(String str);
	//a default method
	default void log(String str){
		System.out.println("I1 logging::"+str);
	}
}

```

类继承多个Interface接口同名方法(如show())时，必须在子类中@Override重写父类show()方法。

```
public interface MyData {
           static boolean isNull(String str) {
   	 System.out.println("Interface Null Check");
   	 return str == null ? true : "".equals(str) ? true : false;
           }
    }
```

### 2.1 函数式接口

* 函数式接口的特点：由@FunctionalInterface注解标识，接口有且仅有一个抽象方法,但是可以有多个非抽象方法的接口。

java.lang.Runnable接口 java.util.concurrent.Callable接口是两个最典型的函数式接口。

函数式接口可以被隐式转换为 lambda 表达式。

使用Lambda表达式，其实都是建立在函数式接口上的。我们看看上面的代码的接口：

创建多线程的Runnable接口：

```
@FunctionalInterface
   public interface Runnable {
       public abstract void run();
   }
```

遍历HashMap的BiConsumer接口：

```
@FunctionalInterface
public interface BiConsumer<T, U> {
    void accept(T t, U u);
    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
        Objects.requireNonNull(after);
        return (l, r) -> {
            accept(l, r);
            after.accept(l, r);
        };
    }
}
```

在List中删除元素的Predicate接口：

```
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```
Java 内置四大核心函数式接口

 |<font color=gray>函数式接口</font>|<font color=gray>参数类型</font>|<font color=gray>返回类型</font>|<font color=gray>用途</font>|
 |-|-|-|-|
 |Consumer<T>消费型接口|T|void|对类型为T的对象应用操作，包含方法：void accept(T t)|
 |Supplier<T>供给型接口|无|T|返回类型为T的对象，包含方法：T get();|
 |Function<T,R>函数型接口|T|R|对类型为T的对象应用操作，并返回结果。 结果是R类型的对象。包含方法：R apply(T t);|
 |Predicate<T>断定型接口|T|boolean|确定类型为T的对象是否满足某约束，并返回boolean 值。包含方法boolean test(T t);|


## 三、注解

 Java 5引入了注解机制，这一特性就变得非常流行并且广为使用。然而，使用注解的一个限制是相同的注解在同一位置只能声明一次，不能声明多次。Java 8打破了这条规则，引入了重复注解机制，这样相同的注解可以在同一地方声明多次。重复注解机制本身必须用@Repeatable注解.
 
 jdk 8扩展了注解的上下文。现在几乎可以为任何东西添加注解：局部变量、泛型类、父类与接口的实现，就连方法的异常也能添加注解。
 
## 四、底层数据结构改变

jdk1.8 中对集合的底层结构做了调整。

如HashMap从1.7的数据+链表的形式调整为数据+链表+红黑树。

ConcurrentHashMap从分段机制+数组+链表+红黑树到CAS+数组+链表+红黑树。

## 五、 Optional类

它是一个容器，装载的对象不能是null，，可以是非null，提供了一系列的方法供我们判断该容器里的对象是否存在(以及后续的操作)。

### 5.1创建Optional容器

我们先来看看Optional的属性以及创建Optional容器的方法：

```

 // 1、创建出一个Optional容器，容器里边并没有装载着对象
    private static final Optional<?> EMPTY = new Optional<>();

    // 2、代表着容器中的对象
    private final T value;

    // 3、私有构造方法
    private Optional() {
        this.value = null;
    }

    // 4、得到一个Optional容器，Optional没有装载着对象
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    // 5、私有构造方法(带参数)，参数就是具体的要装载的对象，如果传进来的对象为null，抛出异常
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }

    // 5.1、如果传进来的对象为null，抛出异常
    public static <T> T requireNonNull(T obj) {
        if (obj == null)
            throw new NullPointerException();
        return obj;
    }


    // 6、创建出Optional容器，并将对象(value)装载到Optional容器中。
    // 传入的value如果为null，抛出异常(调用的是Optional(T value)方法)
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

    // 创建出Optional容器，并将对象(value)装载到Optional容器中。
    // 传入的value可以为null，如果为null，返回一个没有装载对象的Optional对象
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
```
所以可以得出创建Optional容器有两种方式：

* 调用ofNullable()方法，传入的对象可以为null

* 调用of()方法，传入的对象不可以为null，否则抛出NullPointerException

```
import lombok.Data;
@Data
public class User {

    private Integer id;
    private String name;
    private Short age;
}
```

```
public static void main(String[] args) {

    User user = new User();
    User user1 = null;

    // 传递进去的对象不可以为null，如果为null则抛出异常
    Optional<User> op1 = Optional.of(user1);

    // 传递进去的对象可以为null，如果为null则返回一个没有装载对象的Optional容器
    Optional<User> op2 = Optional.ofNullable(user);
}
```

### 5.2Optional容器简单的方法

```
// 得到容器中的对象，如果为null就抛出异常
public T get() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}

// 判断容器中的对象是否为null
public boolean isPresent() {
    return value != null;
}

// 如果容器中的对象存在，则返回。否则返回传递进来的参数
public T orElse(T other) {
    return value != null ? value : other;
}
```

这三个方法是Optional类比较常用的方法，并且是最简单的。(因为参数不是函数式接口)

下面我们继续看看用法：

```
public static void main(String[] args) {

        User user = new User();
        User user1 = null;

        Optional<User> op1 = Optional.ofNullable(user);
        System.out.println(op1.isPresent());   //teue
        System.out.println(op1.get());          // user(id=null, name=null, age=null)
        System.out.println(op1.orElse(user1));   // user(id=null, name=null, age=null)

    }
```

我们调换一下顺序看看：

```
public static void main(String[] args) {

    User user = new User();
    User user1 = null;

    Optional<User> op1 = Optional.ofNullable(user1);
    System.out.println(op1.isPresent());  //false
    System.out.println(op1.orElse(user));  // user(id=null, name=null, age=null)
    System.out.println(op1.get());    //NoSuchElementException

}
```

### 5.3Optional容器进阶用法

#### 5.3.1 ifPresent方法

```
public static void main(String[] args) {

    User user = new User();
    user.setName("Java3y");
    test(user);
}

public static void test(User user) {

    Optional<User> optional = Optional.ofNullable(user);

    // 如果存在user，则打印user的name
    optional.ifPresent((value) -> System.out.println(value.getName()));

    // 旧写法
    if (user != null) {
        System.out.println(user.getName());
    }
}
```

#### 5.3.2 orElseGet和orElseThrow方法

```
public static void main(String[] args) {

    User user = new User();
    user.setName("Java3y");
    test(user);
}

public static void test(User user) {

    Optional<User> optional = Optional.ofNullable(user);

    // 如果存在user，则直接返回，否则创建出一个新的User对象
    User user1 = optional.orElseGet(() -> new User());

    // 旧写法
    if (user != null) {
        user = new User();
    }
}
```

#### 5.3.3filter方法

```
public static void test(User user) {

    Optional<User> optional = Optional.ofNullable(user);

    // 如果容器中的对象存在，并且符合过滤条件，返回装载对象的Optional容器，否则返回一个空的Optional容器
    optional.filter((value) -> "Java3y".equals(value.getName()));
}
```

#### 5.3.4map方法

```
public static void test(User user) {

    Optional<User> optional = Optional.ofNullable(user);

    // 如果容器的对象存在，则对其执行调用mapping函数得到返回值。然后创建包含mapping返回值的Optional，否则返回空Optional。
    optional.map(user1 -> user1.getName()).orElse("Unknown");
}

// 上面一句代码对应着最开始的老写法：

public String tradition(User user) {
    if (user != null) {
        return user.getName();
    }else{
        return "Unknown";
    }
}
```

#### 5.3.5flatMap方法

```
// flatMap方法与map方法类似，区别在于apply函数的返回值不同。map方法的apply函数返回值是? extends U，而flatMap方法的apply函数返回值必须是Optional
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Objects.requireNonNull(mapper.apply(value));
    }
}
```

### 5.3.6总结

```
public static void main(String[] args) {
    User user = new User();
    user.setName("Java3y");
    System.out.println(test(user));
}

// 以前的代码v1
public static String test2(User user) {
    if (user != null) {
        String name = user.getName();
        if (name != null) {
            return name.toUpperCase();
        } else {
            return null;
        }
    } else {
        return null;
    }
}

// 以前的代码v2
public static String test3(User user) {
    if (user != null && user.getName() != null) {
        return user.getName().toUpperCase();
    } else {
        return null;
    }
}

// 现在的代码
public static String test(User user) {
    return Optional.ofNullable(user)
            .map(user1 -> user1.getName())
            .map(s -> s.toUpperCase()).orElse(null);
}
```

filter，map或flatMap一个函数，函数的参数拿到的值一定不是null。所以我们通过filter，map 和 flatMap之类的函数可以将其安全的进行变换，最后通过orElse系列，get，isPresent 和 ifPresent将其中的值提取出来。

## 最后

本文转载自Java3y,部分有本人自己整理

