

### 行为参数化

传递一个代码块，被调用时才延迟执行。



#### 筛选苹果

``` java
enum Color { GREEN, RED; }

/**
 * 1 按颜色筛选苹果
 */
static List<Apple> filterAppleByColor(List<Apple> apples, Color c) {
    List<Apple> result = ...;
    for (Apple apple : apples)  {
        if (Green.equals(c))
            result.add(apple);
    }
    return result;
}

/**
 * 2 可能改变需求，按重量筛选
 */
static List<Apple> filterApplesByWeight(List<Apple> apples, int weight) {
    // 代码同上
    ...;
    if (apple.getWeight() > 150)
        ...;
}

/**
 * 3 对所有可能需要的属性作为参数
 * 但对于调用就很笨拙
 */
static List<Apple> filterApples(List<Apple> apples, Color c, int weight) {
    // 代码同上
    ...;
}
List<Apple> heavyApps = filterApples(apple, null, 150);

/**
 * 4 考虑行为参数化，我们需要更高层次的抽象，一种可能的解决是对标准建模
 * 根据Apple的某些属性，返回一个boolean值。我们称之为 “谓词”
 * 此时我们对于接口采用不同实现类，谓之“策略模式”，代码如下
 * greenApple策略：GreenApplePredicate
 * 其它策略以此类推
 */
interface ApplePredicate {
    boolean test(Apple apple);
}
class GreenApplePredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return Green.equals(apple.getColor())
    }
}
class HeavyApplePredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

/**
 * 5 我们再提供一次抽象，封装为一个函数（类方法）
 */
static List<Apple> filterApples(List<Apple> apples, ApplePredicate p) {
    List<Apple> rs = ...;
    for (Apple apple : apples) {
        // 谓词封装了测试条件
        if (p.test(apple)) {
            rs.add(apple);
        }
    }
    return rs;
}

/**
 * 6 我们可以考虑传递代码（匿名内部类）
 * 调用5提供的函数，以匿名内部类的方式提供策略
 * tips：对于匿名内部类的this，指自己
 */
List<Apple> heavyApples = filterApples(apples, new HeavyApplePredicate() {
    @Override
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
});


/**
 * 7 对于Java8，我们可以使用λ表达式
 */
List<Apple> heavyApples = filterApples(apples, apple -> apple.getWeight() > 150);

/**
 * 8 最后，我们将类型抽象化
 * 这样我们可以把filter()用在橘子，香蕉 | 其它水果上
 */
static <T> List<T> filter(List<T> list, Prediate<T> p) {
    List<T> result = ...;
    for (T t : list) {
        if (p.test(t)) {
            result.add(t);
        }
    }
    return result;
}
```



#### 标准库中真实例子

```
// Comparator
apples.sort((x, y) -> Integer.compare(x.getWeight(), y.getWeight()));

// Runnable
new Thread(() -> sout(Thread.currentThread().getName()));

/**
 * Callable<V>
 * tips: submit()有返回值， execute()无返回值
 */
ExecutorService service = Excutors.newCachedThreadPool();
Future<String> tName = service.submit(() -> Thread.currentThread().getName());
```

