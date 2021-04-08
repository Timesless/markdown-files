## Reactor 3 Doc 附录

- \1. 关于本文档
  - 1.1. 最新版本 & 版权说明
  - 1.2. 贡献本文档
  - 1.3. 获取帮助
  - 1.4. 如何开始阅读本文档
- \2. 快速上手
  - 2.1. 介绍 Reactor
  - 2.2. 前提
  - 2.3. 了解 BOM
  - 2.4. 获取 Reactor
- \3. 响应式编程
  - 3.1. 阻塞是对资源的浪费
  - 3.2. 异步可以解决问题吗？
  - 3.3. 从命令式编程到响应式编程
- \4. Reactor 核心特性
  - 4.1. `Flux`, 包含 0-N 个元素的异步序列
  - 4.2. `Mono`, 异步的 0-1 结果
  - 4.3. 简单的创建和订阅 Flux 或 Mono 的方法
  - 4.4. 可编程式地创建一个序列
  - 4.5. 调度器（Schedulers）
  - 4.6. 线程模型
  - 4.7. 处理错误
  - 4.8. Processors
- \5. 对 Kotlin 的支持
  - 5.1. 简介
  - 5.2. 前提
  - 5.3. 扩展
  - 5.4. Null 值安全
- \6. 测试
  - 6.1. 使用 `StepVerifier` 来测试
  - 6.2. 操控时间
  - 6.3. 使用 `StepVerifier` 进行“后校验”
  - 6.4. 测试 `Context`
  - 6.5. 用 `TestPublisher` 手动发出元素
  - 6.6. 用 `PublisherProbe` 检查执行路径
- \7. 调试 Reactor
  - 7.1. 典型的 Reactor Stack Trace
  - 7.2. 开启调试模式
  - 7.3. 阅读调试模式的 Stack Trace
  - 7.4. 记录流的日志
- \8. 高级特性与概念
  - 8.1. 打包重用操作符
  - 8.2. Hot vs Cold
  - 8.3. 使用 `ConnectableFlux` 对多个订阅者进行广播
  - 8.4. 三种分批处理方式
  - 8.5. 使用 `ParallelFlux` 进行并行处理
  - 8.6. 替换默认的 `Schedulers`
  - 8.7. 使用全局的 Hooks
  - 8.8. 增加一个 Context 到响应式序列
  - 8.9. 空值安全
- Appendix A: 我需要哪个操作符？
  - A.1. 创建一个新序列，它…
  - A.2. 对序列进行转化
  - A.3. “窥视”（只读）序列
  - A.4. 过滤序列
  - A.5. 错误处理
  - A.6. 基于时间的操作
  - A.7. 拆分 `Flux`
  - A.8. 回到同步的世界
- Appendix B: FAQ，最佳实践，以及“我如何…?”
  - B.1. 如何包装一个同步阻塞的调用？
  - B.2. 用在 `Flux` 上的操作符好像没起作用，为啥？
  - B.3. `Mono` `zipWith`/`zipWhen` 没有被调用
  - B.4. 如何用 `retryWhen` 来实现`retry(3)` 的效果？
  - B.5. 如何使用 `retryWhen` 进行 exponential backoff？
  - B.6. How do I ensure thread affinity using `publishOn()`?
- Appendix C: Reactor-Extra
  - C.1. `TupleUtils` 以及函数式接口
  - C.2. `MathFlux` 的数学操作符
  - C.3. 重复与重试工具
  - C.4. 调度器

> 名词：
>
> ==**Publisher（发布者）、Subscriber（订阅者）、Subscription（订阅 n.）、Subscrib（订阅 v.）**==
>
> event / signal（事件/信号，在该文档词义相同）
>
> sequence / stream（序列/流，”流序列“）
>
> element / item（序列中的元素，”元素“）
>
> emit / produce / generate（发出/产生/生成，emit多翻译为"触发，发出"）
>
> consume（消费）
>
> Processor（保留英文）
>
> **operator（操作符，声明式的可组装的相应方法，其组装的链译作”操作链“）**





### 1. Reactor

> Reactor 是一个用于 JVM 的完全非阻塞的响应式编程框架，具备高效的需求管理（对背压「backpressure」的控制）能力。与 Java8 函数式 API 直接集成，比如 CompletableFuture、Stream、Duration。它提供了异步序列 API Flux（用于[N]个元素）和 Mono（用于[0 | 1]个元素），并遵循与实现了”响应式扩展规范“「Reactive Extensions Specification」
>
> Reactor 的 `reactor-ipc` 组件还支持非阻塞的进程间通信「inter-process communicatin, IPC」。Reactor IPC 为 HTTP（包括Websockets）、TCP 和 UDP 提供了支持背压的网络引擎，从而适用于**微服务架构**。并且完整支持响应式编解码「reactive encoding and decoding」



#### 1.1 引入

> Reactor Core 运行在 Java 8 及以上版本
>
> 依赖`org.reactive-streams:reactive-streams:1.0.2`



#### 1.2 了解 BOM

> BOM「Bill of Materials，一种标准的 Maven artifact」
>
> 使用 BOM 可以管理一组集成的 maven artifacts，从而无需操心不同版本组件的依赖问题
>
> BOM 是一系列有版本信息的 artifacts，通过”列车发布“「release train」的发布方式管理，每趟发布列车由一个 **代号 + 修饰词**组成，例如：

``` 
Aluminium-RELEASE
Carbon-BUILD-SNAPSHOT
Aluminium-SR1
Bismuth-RELEASE
Carbon-SR32
```

这些代号主要来自 `Periodic Table of Elements`「元素周期表」

修饰词有：

+ BUILD-SNAPSHOT
+ M1 .. N：里程碑
+ RELEASE：第一次 GA「Generally Availability」发布
+ SR1 .. N：后续 GA发布（类似于 PATCH 号或 SR「Service Release」）



#### 1.3 获取 Reactor

> 最简单的方式是在你的项目中配置 BOM 以及相关依赖项
>
> 添加依赖时省略版本号，从而自动使用 BOM 中的版本

+ maven

``` xml
<dependencyManagement> 
  <dependencies>
    <dependency>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-bom</artifactId>
      <version>Bismuth-RELEASE</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

# 然后在项目中添加依赖
<dependencies>
  <dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId> 
  </dependency>
  <dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId> 
    <scope>test</scope>
  </dependency>
</dependencies>
```



+ gradle

``` groovy
plugins {
  id "io.spring.dependency-management" version "1.0.1.RELEASE" 
}

// 可在顶层项目中引入依赖管理
dependencyManagement {
  imports {
    mavenBom "io.projectreactor:reactor-bom:Bismuth-RELEASE"
  }
}
// 在具体子项目中引入
dependencies {
  compile 'io.projectreactor:reactor-core' 
}
```



##### Milestones 和 Snapshot

> 里程碑「Milestones」和开发预览版「developer previews」通过 **Spring Milestone Repository** 而非 Maven Central 来发布，所以需要添加到构建文件中

+ maven

``` xml
// milestones
<repositories>
  <repository>
    <id>spring-milestones</id>
    <name>Spring Milestones Repository</name>
    <url>https://repo.spring.io/milestone</url>
  </repository>
</repositories>

// snapshots
<repositories>
  <repository>
    <id>spring-snapshots</id>
    <name>Spring Snapshot Repository</name>
    <url>https://repo.spring.io/snapshot</url>
  </repository>
</repositories>
```

+ gradle

``` groovy
// milestones
repositories {
  maven { url 'http://repo.spring.io/milestone' }
  mavenCentral()
}

// snapshots
repositories {
  maven { url 'http://repo.spring.io/snapshot' }
  mavenCentral()
}
```



### 2. 响应式编程

> Reactor 是**响应式编程范式**的实现
>
> **==响应式编程是一种关注于数据流「data streams」与变化传递「propagation of changes」的异步编程方式==**

在响应式编程，微软跨出了第一步，它在`.NET`生态中创建了响应式扩展库「Reactive Extensions Library，Rx」

**接着，RxJava 在 JVM 上实现了响应式编程，JVM 定义了一套标准的==响应式编程规范==（它定义了一系列标准接口和交互规范，整合在 Java 9 中「Flow类」，但并未有具体实现）**



#### 2.1 观察者模式

> 响应式编程通常作为面向对象编程中的 ”观察者模式“「Observer design pattern」的扩展
>
> 响应式流「reactive streams」与”迭代器模式“「Iterator design patter」也有相通之处，其中也有`iterator-iterable`这样的对应关系，主要区别在于，**迭代器是基于"拉取"「pull」方式的，而响应式流是基于”推送“「push」方式的**。使用 iterator 是一种”命令式“「imperative」编程范式，访问元素是 iterator 的唯一职责。**关键在于执行 `next()` 获取元素取决于开发者**，而在响应式编程中，相应的角色是`Publisher-Subscriber`，当有新值到来的时候，**由发布者来通知订阅者**，这种推送模式是响应式编程的关键。
>
> 另外，对推送来的数据操作是通过”声明式“「declaratively」的方式（**开发者通过 描述”控制流程“来定义对数据流的处理逻辑**）
>
> 声明式：开发者通过：描述”控制流程“来定义对数据的操作（做什么）
>
> 命令时：怎么做



#### 2.2 数据推送

Publisher 可以推送新的值到它的 Subscriber（调用 onNext()）

除了数据推送，对错误处理「error handing」和完成信号「completion」的定义也很完善

也可以推送错误，完成信号到它的 Subscriber（onError()，onComplete()），错误和完成信号都可以终止响应式流

``` java
onNext x 0..N [onError | onComplete]
```



#### 2.3 异步

> 在非 I/O 上下文，异步的定义是调用直接返回，提高执行效率
>
> **任务发起异步调用后，执行过程会切换到另一个使用同样底层资源的活跃任务**，等待异步调用返回结果时再处理
>
> 但是，如何在 JVM 上编写异步代码呢？
>
> **Java 提供两种异步编程方式「其他语言也应该类似」**
>
> + **==回调「Callbacks」==**
>
> 异步调用无返回值，而是通过一个callback参数（lambda / 匿名类），当结果返回时调用这个`callback`
>
> + **==Future==**
>
> 异步调用立即返回一个`Future<T>`，该异步调用返回结果是 T 类型，这个结果并不是立即获取的，而是等实际处理结束后才能获取（future.get()）。 `比如， `ExecutorService` 执行 `Callable` 任务时会返回 `Future` 对象。 

这些技术够用吗？

这两种方式都有局限性，回调很难组合起来，很快就会变得难以理解和维护「**回调地狱**」

考虑以下场景：需要2个服务（第一个服务提供收藏内容的 ID 列表，第二个服务根据收藏 ID 获取收藏的内容）

##### 2.3.1 Callback

``` java
userService.getFavorites(userId, new Callback<List<String>>() {
  @Override
  public void onSuccess(List<String> list) {
    list.stream().limit(5).forEach(id -> {
      favoriteService.getDetail(id, new Callback<Favorite>() {
        public void onSuccess(Favorite details) {
          UiUtils.submitOnUiThread(() -> uiList.show(details));
        }
        public void onError(Throwable error) {
        	UiUtils.errorPopup(error);
        }
      });
    });
  }
  @Override
  public void onError(Throwale error) {
  	UiUtils.errorPopup(error);
  }
})
```



##### 2.3.2 Reactor 实现

``` java
// getFavorites 返回 Flux<String>
userService.getFavorites(userId) 
  .flatMap(favoriteService::getDetails) 
  .switchIfEmpty(suggestionService.getSuggestions()) 
  .take(5) 
  .publishOn(UiUtils.uiThreadScheduler()) 
  .subscribe(uiList::show, UiUtils::errorPopup);
```

|      | 获取到收藏ID的流                                             |
| ---- | ------------------------------------------------------------ |
|      | *异步地转换* 它们（ID） 为 `Favorite` 对象（使用 `flatMap`），现在我们有了 `Favorite`流。 |
|      | 一旦 `Favorite` 为空，切换到 `suggestionService`。           |
|      | 我们只关注流中的最多5个元素。                                |
|      | 最后，我们希望在 UI 线程中进行处理。                         |
|      | 通过描述对数据的最终处理（在 UI 中显示）和对错误的处理（显示在 popup 中）来触发（`subscribe`）。 |



##### 2.3.3 Reactor timeout

``` java
userService.getFavorites(userId)
  .timeout(Duration.ofMillis(1000))
  .onErrorResume(cacheService.cachedFavorites(userId))
  .flatMap(favoriteService::getDetails) 
  .switchIfEmpty(suggestionService.getSuggestions()) 
  .take(5) 
  .publishOn(UiUtils.uiThreadScheduler()) 
  .subscribe(uiList::show, UiUtils::errorPopup);
```

> 如果流在 1s 内没有发出「emit」任何值，则发出错误
>
> 一旦收到错误，交给cachaService去获取缓存



##### 2.3.4 CompetableFuture

> 令f(x) = x + 1，g(x) = x * 2
>
> fx.compose(gx).supply(1) = 4「g(f(x))」，fx执行完毕之后再调用gx
>
> fx.combine(gx).supply(1) = 3「f(g(x))」，fx未执行，先执行gx

Future 比回调要好一点，在 Java8 引入 `CompletableFuture`，它对多个异步任务的编排任然都不好用，此外 Future 还有另一个问题，当对 Future 对象的 `get()` 调用仍然是阻塞的，并且缺乏对错误更进一步的处理（CompletableFuture 提供错误机制）

``` java
CompletableFuture<List<String>> ids = ifhIds(); 

CompletableFuture<List<String>> result = ids.thenComposeAsync(l -> { 
  Stream<CompletableFuture<String>> zip =
    l.stream().map(i -> { 
    CompletableFuture<String> nameTask = ifhName(i); 
    CompletableFuture<Integer> statTask = ifhStat(i); 

    return nameTask.thenCombineAsync(statTask, (name, stat) -> "Name " + name + " has stats " + stat); 
  });
  List<CompletableFuture<String>> combinationList = zip.collect(Collectors.toList()); 
  CompletableFuture<String>[] combinationArray = combinationList.toArray(new CompletableFuture[combinationList.size()]);

  CompletableFuture<Void> allDone = CompletableFuture.allOf(combinationArray); 
  return allDone.thenApply(v -> combinationList.stream()
                           .map(CompletableFuture::join) 
                           .collect(Collectors.toList()));
});

List<String> results = result.join(); 
assertThat(results).contains(
  "Name NameJoe has stats 103",
  "Name NameBart has stats 104",
  "Name NameHenry has stats 105",
  "Name NameNicole has stats 106",
  "Name NameABSLAJNFOAJNFOANFANSF has stats 121");
```

|      | 以一个 Future 开始，其中封装了后续将获取和处理的 ID 的 list。 |
| ---- | ------------------------------------------------------------ |
|      | 获取到 list 后边进一步对其启动异步处理任务。                 |
|      | 对于 list 中的每一个元素：                                   |
|      | 异步地得到相应的 name。                                      |
|      | 异步地得到相应的 statistics。                                |
|      | 将两个结果一一组合。                                         |
|      | 我们现在有了一个 list，元素是 Future（表示组合的任务，类型是 `CompletableFuture`），为了执行这些任务， 我们需要将这个 list（元素构成的流） 转换为数组（`List`）。 |
|      | 将这个数组传递给 `CompletableFuture.allOf`，返回一个 `Future` ，当所以任务都完成了，那么这个 `Future` 也就完成了。 |
|      | 有点麻烦的地方在于 `allOf` 返回的是 `CompletableFuture`，所以我们遍历这个 Future 的`List`， ，然后使用 `join()` 来收集它们的结果（不会导致阻塞，因为 `AllOf` 确保这些 Future 全部完成） |
|      | 一旦整个异步流水线被触发，我们等它完成处理，然后返回结果列表。 |



##### 2.3.5 Reactor 实现

``` java
Flux<String> ids = ifhrIds();

Flux<String> combinations = ids.flatMap(id -> {
  Mono<String> nameTask = ifhrName(id);
  Mono<Integer> statTask = ifhrStat(id);
  return nameTask.zipWith(statTask, (name, stat) -> "Name: " + name + "has stats " + stat);
});
Mono<List<String>> monoResult = combinations.collectList();
List<String> result = monoResult.block();

assertThat(result).containsExactly(
  "Name NameJoe has stats 103",
  "Name NameBart has stats 104",
  "Name NameHenry has stats 105",
  "Name NameNicole has stats 106"
);
```

|      | 这一次，我们从一个异步方式提供的 `ids` 序列（`Flux`）开始。  |
| ---- | ------------------------------------------------------------ |
|      | 对于序列中的每一个元素，我们异步地处理它（`flatMap` 方法内）两次。 |
|      | 获取相应的 name。                                            |
|      | 获取相应的 statistic.                                        |
|      | 异步地组合两个值。                                           |
|      | 随着序列中的元素值“到位”，它们收集一个 `List` 中。           |
|      | 在生成流的环节，我们可以继续异步地操作 `Flux` 流，对其进行组合和订阅（subscribe）。 最终我们很可能得到一个 `Mono` 。由于是测试，我们阻塞住（`block()`），等待流处理过程结束， 然后直接返回集合。 |
|      | Assert 结果。                                                |



#### 2.4 从命令式到响应式编程

> **类似 Reactor 这样的响应式库是弥补语言本身提供的异步方式所带的不足**
>
> 此外，还会关注其它几个方面：
>
> + **可编排性「Composability」和可读性「Readability」**
> + 使用丰富的 **操作符** 来处理形如 **流** 的数据
> + 在 **订阅「subscribe」**之前什么都不发生
> + **背压「back pressure」**，具体来说就是： **==消费者能够反向告知生产者生产内容的速度的能力==**
> + **高层次的抽象**，从而对外表现出与 **并发无关** 的效果



##### 2.4.1 可编排与可读性

> 可编排：指对多个异步任务任意组合：将前一个任务的结果传递给后一个任务 或者 将多个任务分解再汇总「`fork-join`」

你可以想象，在响应式应用中数据处理就像流过一条装配流水线，Reactor 既是传送带，又是一个个的装配工 /  机器人。原材料从源「Publisher，source」流出，最终被加工为成品，等待被推送到消费者「Subscriber」

原材料会经过不同的中间处理过程，或者作为半成品与其它半成品组装，如果有齿轮卡住，或者包装花费了太多时间，相应的工位就向上游发出信号来限制 / 停止发出原材料



##### 2.4.2. 操作符（Operators）

在 Reactor 中，操作符（operator）就像装配线中的工位（操作员或装配机器人）。每一个操作符 对 `Publisher` 进行相应的处理，然后将 `Publisher` 包装为一个新的 `Publisher`。就像一个链条， 数据源自第一个 `Publisher`，然后顺链条而下，在每个环节进行相应的处理。最终，一个订阅者 (`Subscriber`）终结这个过程。请记住，**在订阅者（`Subscriber`）订阅（subscribe）到一个 发布者（`Publisher`）之前，什么都不会发生。**

|      | 理解了操作符会创建新的 `Publisher` 实例这一点，能够帮助你避免一个常见的问题， 这种问题会让你觉得处理链上的某个操作符没有起作用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

虽然响应式流规范（Reactive Streams specification）没有规定任何操作符， **类似 Reactor 这样的响应式库所带来的最大附加价值之一就是提供丰富的操作符。包括基础的转换操作， 到过滤操作，甚至复杂的编排和错误处理操作。**



##### 2.4.3 subscribe()之前什么都不会发生

> Reactor 中，当你创建一个 Publisher 处理链，数据还不会开始生成，事实上，你只是创建了 **一种抽象的对于异步处理流程的描述（从而方便重用和组装）**
>
> 当真正 订阅「subscribe」时，你需要将一个 Publisher 关联到一个 Subscriber，然后才会触发整个链条的流动，**subscribe() 实际上是向上游发送一个 request 信号，一直到达源头的 Publisher**



##### 2.4.4 背压

> **向上游传递信号可用于实现背压「back pressure」**
>
> 订阅者可以通过 request 机制来告知源头「Publisher」它一次最多能够处理 n 个元素
>
> `request(n)`
>
> 中间环节的操作也可以影响 request，想象一个能够提供10个元素分批打包的缓存，当订阅者请求1个元素（源头每次产生10个元素），这样能够将 **推送模式** 转换为 **推送 + 拉取混合的模式**，如果下游准备好了，可以从上游拉取 n 个元素，如果上游未准备好，下游还是需要依赖于上游的推送



##### 2.4.5 冷 / 热响应式流

Hot & Cold

> **在 `Rx`  响应式库中， 响应式流分”热” 和 “冷“两种类型**，区别主要在于响应式流如何对订阅者进行响应：
>
> + **冷序列**，指**对于每个 Subscriber**，都会收到从头开始所有的数据，如果源头生成一个 HTTP 请求，那么对于每个每个订阅都会创建一个新的 HTTP 请求
> + **热序列**，指**对于一个 Subscriber **，只能获取从它开始订阅之后源头发出的数据（注意：有些热响应式流可以缓存部分或全部的历史数据），一个”热“的响应式流，在即使没有任何订阅者接收数据的情况下也可以发出数据



### 3. Reactor 核心

> projectreactor Reactor 核心是 `reactor-core`，这是一个基于 Java 8 的实现了 **响应式规范「Reactive Streams Specification」** 的响应式库
>
> 实现 `Publisher` 的响应式类：`Flux Mono`，以及丰富的操作方式
>
> 一个 `Flux`对象代表：包含 0..N 个元素的响应式序列
>
> 一个 `Mono`对象代表：包含 0..1 个元素的响应式序列
>
> 有些操作可以改变基数，从而需要切换类型。比如：count 作用于 Flux，但返回结果是 Mono\<Long>



#### 3.1 Flux

包含 0..N 个元素的异步序列（响应式序列）

> `Flux<T>` 是一个能发出 0 到 N 个元素的标准的 `Publisher<T>`，**它会被一个”错误“ 或 ”完成”信号终止**
>
> 因此，一个 `Flux` 的结果可能是：value、completion、error
>
> 这三种类型的信号被翻译为面向下游的 `onNext(), onSuccess(), onError() `方法
>
> 所有信号事件都是可选的：如果没有 `onNext` 事件但是有一个 `onComplete` 事件，那么发出的就是 **空的有限序列**， 但是去掉 `onComplete` 那么得到的就是 **无限的空序列**
>
> `Flux.interval(Duration)` 生成的是一个 `Flux<Long>` 无限的周期性发出的时钟序列



#### 3.2 Mono

异步的 0 / 1 个结果

> `Mono<T>` 是一种特殊的 `Publisher<T>`，它最多发出一个元素，终止于一个 onComplete / onError 信号
>
> `Mono#concatWith(Publihser)` 返回一个 Flux，而 `Mono#then(Mono)` 返回另一个 Mono
>
> Mono 可用于表示 “空” 的只有完成概念的异步处理「即：不返回任何值，Runnable」，这种情况使用 `Mono<Void>` 创建



#### 3.3 创建和订阅 Flux / Mono

Flux 和 Mono 提供多种工厂方法

``` java
// 创建一个 String 序列
Flux<String> seq1 = Flux.just("foo", "bar", "foobar");
							seq2 = Flux.fromIterable(Arrays.asList("foo", "bar"));

Mono<String> emptyMono = Mono.empty();
						data = Mono.just("foo");
// 从5开始，生成3个元素
Flux<Integer> ns = Flux.range(5, 3);
```



+ 订阅

**在 subscribe() ，Flux 和 Mono 使用 Java 8 lambda 表达式，你可以传入各种 lambda 来定义回调**

``` java
subscribe();
subscribe(Consumer<? super T> consumer);
subscribe(Consumer<? super T> consumer, Consumer<? super Throwable> errorConsumer);
subscribe(consumer, errorConsumer, Runnable completeConsumer);
subscribe(consumer, errorConsumer, completeConsumer, Consumer<? super Subscription> subscriptionConsumer);
```

|      | 订阅并触发序列。                                             |
| ---- | ------------------------------------------------------------ |
|      | 对每一个生成的元素进行消费。                                 |
|      | 对正常元素进行消费，也对错误进行响应。                       |
|      | 对正常元素和错误均有响应，还定义了序列正常完成后的回调。     |
|      | 对正常元素、错误和完成信号均有响应， 同时也定义了对该 `subscribe` 方法返回的 `Subscription` 执行的回调。 |

>  以上方法会返回一个 `Subscription` 的引用，如果不再需要更多元素你可以通过它来取消订阅。 取消订阅时， 源头会停止生成新的数据，并清理相关资源。**取消和清理的操作在 Reactor 中是在 接口 `Disposable`中定义的。** 



+ 代码示例

``` java
Flux<Integer> ints = Flux.range(1, 5);

ints.subscribe();
ints.subscribe(System.out::println);

ints.map(i -> {
  if (i <= 3) {
    return i;
  }
  throw new RuntimeException("got to 4");
}).subscribe(System.out::print, error -> system.out.println(error));

ints.subscribe(System.out::println, System.out::println, () -> System.out.println("Done"));

```



##### 3.4.1 自定义 subscriber

>  扩展的时候通常至少要覆盖 `hookOnSubscribe(Subscription subscription)` 和 `hookOnNext(T value)` 这两个方法。
>
>  建议你同时重写 `hookOnError`、`hookOnCancel`，以及 `hookOnComplete` 方法。 

``` java
package io.projectreactor.samples;
import org.reactivestreams.Subscription;
import reactor.core.publisher.BaseSubscriber;

public class SampleSubscriber<T> extends BaseSubscriber<T> {
  public void hookOnSubscribe(Subscription subscription) {
    System.out.println("Subscribed");
    request(1);
  }
  
  // 覆写 hookOnNext 来配置 “背压”
  public void hookOnNext(T value) {
    System.out.print(value + “”);
    request(1);
  }
}

// 自定义 subscriber
SampleSubscriber<Integer> ss = new SampleSubscriber<Integer>();
Flux<Integer> ints = Flux.range(1, 4);
ints.subscribe(System.out::println, System.out::println, () -> System.out.println("Done"), scription -> scription.request(10));
ints.subscribe(ss);
```

|      | `BaseSubscriber` 是一个抽象类，所以我们可以创建一个匿名实现类。 |
| ---- | ------------------------------------------------------------ |
|      | `BaseSubscriber` 定义了多种用于处理不同信号的 hook。它还定义了一些捕获 `Subscription` 对象的现成方法，这些方法可以用在 hook 中。 |
|      | `request(n)` 就是这样一个方法。它能够在任何 hook 中，通过 subscription 向上游传递 背压请求。这里我们在开始这个流的时候请求1个元素值。 |
|      | 随着接收到新的值，我们继续以每次请求一个元素的节奏从源头请求值。 |
|      | 其他 hooks 有 `hookOnComplete`, `hookOnError`, `hookOnCancel`, and `hookFinally` （它会在流终止的时候被调用，传入一个 `SignalType` 作为参数）。 |

当你修改请求操作的时候，你必须注意让 subscriber 向上提出足够的需求， 否则上游的 Flux 可能会被“卡住”。所以 `BaseSubscriber` 在进行扩展的时候要覆盖 `hookOnSubscribe` 和 `onNext`，这样你至少会调用 `request` 一次。



#### 3.4 可编程式的创建序列

**==我们可以通过定义相对应的事件「onNext、onError、onComplete」创建一个 Flux / Mono ，所有这些方法都通过 API 来触发，我们叫做 sink（池）的事件==**



##### 3.4.1 Generate

使用 generate() 创建一个 Flux

这是一种 **同步的， 逐个的** 产生值的方法，意味着 **sink** 是一个 `SynchronousSink` 而且其 next() 在每次回调时最多只能被调用一次。也可以调用 error(Throwable) / complete()

``` java
Flux<String> flux = Flux.generate(
  () -> 0, 
  (state, sink) -> {
    sink.next("3 x " + state + " = " + 3*state); 
    if (state == 10) sink.complete(); 
    return state + 1; 
});
```

> 1. 初始化状态值（state）为0
> 2. 基于状态值 state 生成下一个值（state * 3）
> 3. 我们可以通过状态来决定什么时候终止序列
> 4. 返回一个新的 state 用于下一次调用

 **原生类型及其包装类，以及String等属于不可变类型**



+ 可变类型的状态变量

``` java
Flux<String> flux = Flux.generate(AtomicLong::new, (state, sink) -> {
  long s = state.getAndIncrement();
  sink.next("3 x " + s + " = " + 3 * s);
  if (s == 10) sink.complete();
  return state;
}, state -> System.out.println("state: " + state) );
```

state 会打印出11

**如果 state 使用了数据库或其他需要最终清理的资源，这个 Consumer 可用来关闭资源**



##### 3.4.2 Create

create 也可用于创建 Flux，create() 生成的序列既可以是同步的，也可以是异步的，并且还可以每次发出多个元素，该方法用到了 **FluxSink**，同样提供 next(), error(), complete() 等方法。与 generate() 不同的是， create() 不需要状态值， 另一方面，它可以在回调中触发多个事件

> ==create() 可以将现有的 API 转换为响应式，比如监听器的异步方法==

假设你有一个监听器 API， 它按 chunk「块」处理数据，有两种事件：

+ 一个 chunk 数据准备好的事件
+ 处理结束的事件

``` java
interface EventListener<T> {
  void onDataChunk(List<T> chunk);
  void onComplete();
}
```

你可以使用 create() 将 EventListener 转换为响应式 `Flux<T>`

``` java
Flux.create(sink -> {
  eventProcessor.register(new EventListener<String>() {
    @Override
    public void onDataChunk(List<String> chunk) {
      for (String s : chunk) {
        sink.next(s);
      }
    }
    @Overide
    public void onComplete() {
      sink.complete();
    }
	});
});
```

|      | 桥接 `EventListener`。                             |
| ---- | -------------------------------------------------- |
|      | 每一个 chunk 的数据转化为 `Flux` 中的一个元素。    |
|      | `processComplete` 事件转换为 `onComplete`。        |
|      | 所有这些都是在 `EventProcessor` 执行时异步执行的。 |

> create() 可以异步的，并且能够控制背压，你可以通过一个 `OverflowStrategy` 来定义背压行为

+ IGNORE：完全忽略下游背压请求，这可能会在下游队列积满时导致 `IllegalStateException`
+ ERROR：当下游跟不上节奏的时候发出一个 `IllegalStateException` 的错误信号
+ DROP：当下游没有准备好接收新的元素的时候抛弃这个元素
+ LATEST：让下游只得到上游最新的元素
+ **BUFFER：「默认值」**缓存所有下游没有来得及处理的元素「这个无限的缓存可能导致 `OutOfMemoryError`」

>  `Mono` 也有一个用于 `create` 的生成器（generator）—— `MonoSink`，它不能生成多个元素， 因此会抛弃第一个元素之后的所有元素。 

**create() 可用于 push / pull 模式**，因此适合桥接监听器的 API，因为事件消息会随时的到来，回调方法 onRequest 可以被注册到 FluxSink 以便跟踪请求。这个**回调可用于从源头请求更多数据，或者通过在下游请求到来的时候传递数据給 sink 以实现背压管理**，这是一种 **推送 / 拉取混合的模式**

==下游可以从上游拉取已就绪的数据，上游也可以在数据就绪后推送給下游==

``` java
Flux.create(sink -> {
  messageProcessor.register(new MessageListener<String>() {
  	@Override
    public void onMessage(List<String> messages) {
      for (String s : messages) {
        sink.next(s);
      }
    }
  });
  // 如果有就绪的 message，就发送到 sink
  sink.onRequest(n -> {
    List<String> messages = messageProcessor.request(n);
    for(String s : messages) {
      sink.next(s);
    }
  });
});
```

|      | 当有请求的时候取出一个 message。           |
| ---- | ------------------------------------------ |
|      | 如果有就绪的 message，就发送到 sink。      |
|      | 后续异步到达的 message 也会被发送给 sink。 |



+ 清理

  cleaning up

> onDispose 和 onCancel 这两个回调用于在被取消和终止后进行清理工作。
>
> onDispose 可用于在 Flux 完成有错误 / 被取消的时候执行清理
>
> onCancel 只针对 “取消信号” 执行相关操作，会先于 onDispose 执行

``` java
Flux.create(sink -> {
  sink.onRequest(n -> channel.poll(n))
    .onCancel(() -> channel.cancel())
    .onDispose(() -> channel.close())
});
```

| `onCancel` 在取消时被调用。 |                                            |
| --------------------------- | ------------------------------------------ |
|                             | `onDispose` 在有完成、错误和取消时被调用。 |



##### 3.4.3 push

推送模式

create() 的变体是 push，适合生成事件流，push 也可以是异步的，并且**==能够使用各种溢出策略「overflow strategies」来管理背压==**。



``` java
Flux.push(sink -> {
  eventProcessor.register(new SingleThreadEventListener<String>() {
    @Override
    public void onDataChunk(List<String> chunk) {
      for (String s : chunk) {
        sink.next(s);
      }
    }
    @Override
    public void onComplete() {
      sink.complete();
    }
    @Override
    public void onError(Throwable e) {
      sink.error(e);
    }
  });
});
```

|      | 桥接 `SingleThreadEventListener` API。                  |
| ---- | ------------------------------------------------------- |
|      | 在监听器所在线程中，事件通过调用 `next` 被推送到 sink。 |
|      | `complete` 事件也在同一个线程中。                       |
|      | `error` 事件也在同一个线程中。                          |



##### 3.4.4 handle

 `handle` 方法有些不同，它在 `Mono` 和 `Flux` 中都有。然而，它是一个实例方法 （instance method），意思就是它要链接在一个现有的源后使用（与其他操作符一样）。 

 它与 `generate` 比较类似，因为它也使用 `SynchronousSink`，并且只允许元素逐个发出。 然而，`handle` 可被用于基于现有数据源中的元素生成任意值，有可能还会跳过一些元素。 这样，可以把它当做 `map` 与 `filter` 的组合。`handle` 方法签名如下： 

` handle(BiConsumer<T, SynchronousSink<R>>) `

``` java
// 过滤掉负数
flux = Flux.just(-1, 20, 30, 40)
  .handle((i, sink) -> {
    if (i < 0) {
      sink.next(i);
    }
  });
flux.subscribe(System.out::println);
```



#### 3.5 调度器

Schedulers

**==Reactor 同 RxJava，都可以被认为是 并发无关「concurrenty agnostic」==**，意思是：它并不强制要求任何并发模型，它将选择权交给开发者，不过它也提供了一些方便并发执行的 API 库

> 在 Reactor 中，执行模式以及执行过程取决于所使用的 `Scheduler`，它是一个拥有广泛实现类的抽象接口。
>
> Schedulers 类提供的静态方法用于达成如下的执行环境：



##### 3.5.1 使用 / 创建调度器

+ 当前线程「Schedulers.immediate()」
+ 可重用线程「Schedulers.single()」，这个方法对所有调用者都提供同一个线程，直到该调度器「Scheduler」被抛弃。如果你想使用同一线程，请使用 `Schedulers.newSingle()`
+ 弹性线程池「Schedulers.elastic()」。它根据需要创建一个线程池，重用空闲线程。线程池如果空闲超时「默认 60s」就会被废弃，对于`I/O` 阻塞的场景比较适合。

+ 固定大小线程池「Schedulers.parallel()」，线程数量同 CPU 核数

> 此外，还可以通过 `Schedulers.formExecutorService(ExecurtorService)` 基于现有的线程池创建 Scheduler，也可以使用 Executor 来创建（虽然不太建议）
>
> **`Schedulers.newXxxx(schedulerName)` 创建调度器**

操作符基于非阻塞算法实现，从而可以利用到某些调度器的工作窃取（work stealing） 特性的好处。    

一些操作符默认会使用一个指定的调度器（通常也允许开发者调整为其他调度器）例如， 通过工厂方法 `Flux.interval(Duration.ofMillis(300))` 生成的每 300ms 打点一次的 `Flux`， 默认情况下使用的是 `Schedulers.parallel()`，下边的代码演示了如何将其装换为 `Schedulers.single()`：

```
Flux.interval(Duration.ofMillis(300), Schedulers.newSingle("test"))
```



##### 3.5.1 调整调度器

Reactor 体哦那个了两种在响应式链中调整调度器的方法

+ **==publishOn==**
+ **==subscribeOn==**

他们都接受一个 Scheduler 作为参数，从而可以改变调度器

**你首先要理解 nothing happens until you subscribe()**

在 Reactor 中，当你在操作链上添加操作符的时候，==你可以将 Flux 和 Mono 再包装为 Flux / Mono，一旦你 **订阅「subscribe」**了它，一个 **Subscriber** 的链就被创建了，一直到第一个 Publisher（最顶层的 Flux / Mono）。==这些对开发者是不可见的，开发者只能看到最外层 Flux / Mono 和 Subscription，但是具体的任务是在中间这些跟操作符相关的 subscriber 上处理的

基于此，我们仔细研究一下 `publishOn` 和 `subscribeOn` 这两个操作符

+ publishOn：用法与处于订阅链「subscriber chain」中其它操作符一样。**它将上游信号传给下游，它会改变后续的操作符的执行所在的线程，直到下一个 publishOn 出现在这个链上**

+ subscribeOn：用于订阅过程，作用于向上的订阅链（发布者在订阅时才会被激活，订阅的传递方向是向上游的），所以无论你把 subscribeOn 置于操作链的什么位置，**它都会影响到源头的线程执行环境「context」**，但是他不会影响到后续的 publishOn，后者仍然能够切换其后操作符的线程执行环境



#### 3.6 线程模型

==Flux 和 Mono 不会创建线程，一些操作符，比如 publishOn 会创建线程。==

同时，作为一种任务共享形式，这些操作符可能会从其它任务池「work pool 如果是空闲的话」"偷取"线程

无论是 Flux / Mono / Subscriber 都依赖这些操作符来管理线程和任务池。

> **publishOn 强制在它下面的操作符运行在 publishOn 指定的调度器的线程上**
>
> **类似的 subscribeOn 强制在它上面的操作符运行在 subscriberOn 指定的调度器的线程上**

==始终牢记：在 subscribe 之前，你只是定义了流程，而并未启动发布者，一旦你订阅了整个流程就开始工作==

下边栗子演示了支持任务共享的多线程模型：

``` java
Flux.range(1, 1000)
  .publishOn(Schedulers.parallel())
  .subscribe()
```

>  `Scheduler.parallel()` 创建一个基于 `ExecutorService` 的固定大小的任务线程池。 因为可能会有一个或两个线程导致问题，它总是至少创建 4 个线程。然后 publishOn 方法便共享了这些任务线程， 当 `publishOn` 请求元素的时候，会从任一个正在发出元素的线程那里获取元素。这样， 就是进行了任务共享（一种资源共享方式）。Reactor 还提供了好几种共享资源的方式，请参考 [Schedulers](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fprojectreactor.io%2Fdocs%2Fcore%2Frelease%2Fapi%2Freactor%2Fcore%2Fscheduler%2FSchedulers.html)。 
>
>  ==`Scheduler.elastic()` 也能创建线程，它能够很方便地创建专门的线程（以便跑一些可能会阻塞资源的任务， 比如一个同步服务）==，请见 如何包装一个同步阻塞的调用？。 

 内部机制保证了这些操作符能够借助自增计数器（incremental counters）和警戒条件（guard conditions） 以线程安全的方式工作。



#### 3.7 错误处理

 在响应式流中，错误（error）是终止（terminal）事件。当有错误发生时，它会导致流序列停止， 并且错误信号会沿着操作链条向下传递，直至遇到你定义的 `Subscriber` 及其 `onError` 方法。 



##### 3.7.1 常用的错误处理方法

命令式 try -catch 的错误处理：

1. 捕获并返回一个缺省值
2. 捕获并执行一个异常处理方法
3. 捕获并动态计算值来顶替
4. 捕获，再包装为 **业务相关异常抛出**
5. 捕获，记录日志，然后继续抛出
6. 使用 finally 清理资源，或 `try-with-resource`

以上传统处理方法在 Reactor 中都有相应的基于 `error-handing` 操作符处理方式



两种方式的对比：

+ **响应式链「reactive chain」方式**

+ **try - catch 方式**

``` java
// 响应式链错误处理方式「声明式」
flux = Flux.range(1, 10).map(v -> doDangerous(v)).map(v -> doTransform(v));

flux.subscribe(value -> System.out.println("value = " + value),
              error -> System.out.println("caught " + error));
```

> 1. 执行 map 转换，可能抛出异常
> 2. 如果没有异常则执行第二个 map 转换
> 3. 所有转换成功的值都打印
> 4. 一旦有错误，序列终止，并打印错误信息

``` java
// 传统错误处理「命令式」
try {
  for (int i = 0; i < 10; ++i) {
    String v1 = doDangerous(i);
    String v2 = doTransform(v1);
    System.out.println("value = " + value);
  }
} catch(Throwable t) {
  System.err.println("caught " + t);
}
```



##### 3.7.2 onErrorReturn 返回缺省值

``` java
Flux.just(10).map(this::doDangerous)
  .onErrorReturn("RECOVERD");

// 判断错误信息筛选，让某些错误仍然传递下去
Flux.just(10).map(this::doDangerous)
  .onErrorReturn(e -> e.getMessage().contains("NullPointerException"), "recoverd npe")
```



##### 3.7.2 onErrorResume 异常统一处理

发生异常时，调用统一的异常处理方法（**当服务器请求失败，返回一份本地缓存**）

``` java
Flux.just("k1", "k2").flatMap(k -> callRpc(k))
  .onErrorResume(e -> getFromCache(e));
```

> onErrorResume 也可以预先过滤错误内容，可以基于异常类 / Predicate 进行过滤，它实际是一个 Function
>
> 如果源超时使用本地缓存，如果无对应的key则创建一个新的实体，否则将问题重新抛出

``` java
Flux.just("timeout1", "unkonwn", "k")
  .flatMap(k -> callRpc(k))
  .onErrorResume(err -> {
    if (err instanceof TimeoutException) {
      return getFromCache(k);
    } else if (err instanceof UnkonwnKeyException) {
      return regiserNewEntry(k, "DEFAULT")
    } else {
      return Flux.error(error);
    }
  });
```



##### 3.7.3 onErrorResume 动态替补值

有时候想在接收到错误时计算一个候补值

例如：`Future.complete(T success) VS Future.completeExceptionally(Throwable err)`

``` java
errorFlux.onErrorResume(err -> Mono.just(wrapper.fromError(err)))
```

使用 Mono.just 创建一个 Mono，将异常包装



##### 3.7.4 onErrorMap

+ 

``` java
Flux.just("timeout").flatMap(k -> callRpc(k))
  .onErrorResume(origin ->
                 Flux.error(new BusinessException("oops", SLA exceeded), origin));
```

+ onErrorMap

``` java
Flux.just("timeout").flatMap(k -> callRpc(k))
  .onErrorMap(origin -> new BusinessException("oops, SLA exceeded", origin))
```



##### 3.7.5 doOnError 记录日志

**doOnError**：错误时执行 action，并继续抛出异常

``` java
LongAdder failureStat = new LongAdder();
Flux.just("unkown").flatMap(k -> callRpc(k))
  .doOnError(e -> {
    failureSta.increment();
    log.error("falling back, serivce failure for k" + k);
  }).onErrorResume(e -> getFromCache(k));
```



##### 3.7.6 doFinally 资源清理

**doFinally：在序列终止时执行，无论是 complete / error / cancel 信号**，并且能够判断是什么类型终止事件

``` java
LongAdder statsCancel = new LongAdder();
Flux.just("foo", "bar", "foobar")
  .doFinally(type -> {
    if (type == SignalType.CANCEL) {
      statsCancel.increment();
    }
  }).take(1);
```

> 1. 我们想要统计取消的次数，所以用到 `LongAdder`
> 2. doFinally 检查终止信号的类型
> 3. 如果是 `Signal.CANCEL`，那么统计数据自增
> 4. take(1) 能够在发出 1 个元素后取消流



##### 3.7.7 onError

> interval 基于一个 `timer scheduler` 来执行，需要挡住 main 线程防止 JVM 立即退出
>
> 即便多给 1s 也没有更多的 ticket 信号产生了， 序列确实被错误信息终止了

``` java
flux = Flux.interval(Duration.ofMillis(250))
  .map(i -> {
    if (i < 3) {
      return "tick " + i;
    }
    throw new RuntimeException("boom")
  }).onErrorReturn("Uh oh");

flux.subscribe(sout);
// 挡住主线程
Thread.sleep(2100);
```

> 执行结果：tick 0，tick 1，tick 2，Uh oh



##### 3.7.8 retry 重试

**retry 用于对出现错误的序列进行重试**

==问题是，retry 对上游 Flux 是基于重订阅「re-subscribing」的方式，这实际已经是一个不同序列了，产生错误信号的序列仍然是终止了的==



``` java
Flux.interval(Duraion.ofMillis(250))
  .map(i -> {
    if (i < 3) {
      return "tick " + i;
    }
    throw new RuntimeException("boom")
  })
  .elapsed()
  .retry(1)
  // onNext, onError
  .subscribe(sout, sout);

THread.sleep(2100)
```

>  `elapsed` 会关联从当前值与上个值发出的时间间隔（译者加：如下边输出的内容中的 259/249/251…）。 
>
> 我们也打印了 onError 时 的内容
>
> sleep() 确保我们有足够时间进行 4 * 2 次 tick

```
# 执行结果
259,tick 0
249,tick 1
251,tick 2
506,tick 0 
248,tick 1
253,tick 2
java.lang.RuntimeException: boom
```

> **可见 retry(1) 不过是再次订阅了原始的 interval，tick 从 0 开始，第二次，由于异常再次出现，所以将异常传递到下游了** 



##### 3.7.9 retryWhen



#### 3.8 Processors

> **==Processor 既是一种特殊的发布者（Publisher）又是一种订阅者（Subscriber）==**
>
> 这意味着你可以订阅一个 Processor（通常他们会实现 Flux），也可以调用相关方法来插入数据到序列，或终止序列
>
> Processor 有多种类型，它们都有特别的语义规则



##### 3.8.1 是否需要使用 Processor ？

多数情况下，应该避免使用 Processor，可以选择以下两种替代方式：

1. 是否有一个或多个操作符的组合能够满足需求
2. "generator" 操作符能否解决问题？（**通常这些操作符可以用来桥接非响应式 API，他们提供了 sink**）



##### 3.8.2 sink 门面对象来线程安全的生成流

比起直接使用 Proccesors，更好的方式通过调用 sink()  来得到 Processor 的 Sink，大概意思是：**Processor 中组合了 Sink，很多方法可以直接委托給 Sink 去处理**

`FluxProcessor` 的 sink 是线程安全的生产者「producer」，能在多线程并发的生成数据

例如，一个序列化（serialized）的 sink 能够通过 `UnicastProcessor` 创建：

``` java
UnicastProcessor processor = UnicastProcessor.create();
FluxSink<Integer> sink = processor.sink(overflowStrategy);
sink.next(10);
```

**next()  产生溢出有两种可能的处理方式**

+ 无限的 processor 通过丢弃或缓存自行处理溢出
+ 有限的 processor 阻塞在 IGNORE 策略， 或将 overflowStrategy 应用于 sink



##### 3.8.3 现有的 Porcessors

Reactor Core 内置多种 Processor，大概分三类：

+ **直接的「direct」**：DirectProcessor 和 UnicastProcessor，只能通过直接调用 Sink 的方法来推送数据
+ **同步的「synchronous」**：EmitterProcessor 和 ReplayProcessor，既可以直接调用 Sink 的方法来推送数据，也可以通过订阅到一个上游的发布者来同步的产生数据。
+ **异步的「asynchronous」**：WrokQueueProcessor 和 TopicProcessor，它们可以将多个上游发布者得到的数据推送下去，由于使用了 RingBuffer 的数据结构来缓存多个来自上游的数据，更具有健壮性

> **异步的 processor 暴露出的是 Builder 接口，而简单的 processor 提供了静态的工厂方法**

+ DirectProcessor

`DirectProcessor` 可以将信号分发给零到多个订阅者（`Subscriber`）。它是最容易实例化的，使用静态方法 `create()` 即可。另一方面，**它的不足是无法处理背压**。所以，当 `DirectProcessor` 推送的是 N 个元素，而至少有一个订阅者的请求个数少于 N 的时候，就会发出一个 `IllegalStateException`。

 一旦 `Processor` 终止（通常通过调用它的 `Sink` 的 `error(Throwable)` 或 `complete()` 方法）， 虽然它允许更多的订阅者订阅它，但是会立即向它们重新发送终止信号。 

+ UnicastProcessor

`UnicastProcessor` 可以使用一个内置的缓存来处理背压。代价就是它最多只能有一个订阅者。

`UnicastProcessor` 有多种选项，因此提供多种不同的 `create` 静态方法。例如，它默认是 *无限的（unbounded）* ：如果你在在订阅者还没有请求数据的情况下让它推送数据，它会缓存所有数据。

可以通过提供一个自定义的 `Queue` 的具体实现传递给 `create` 工厂方法来改变默认行为。如果给出的队列是 有限的（bounded）， 并且缓存已满，而且未收到下游的请求，processor 会拒绝推送数据。

在上边 *有限的* 例子中，还可以在构造 processor 的时候提供一个回调方法，这个回调方法可以在每一个 被拒绝推送的元素上调用，从而让开发者有机会清理这些元素。



+ EmitterProcessor

`EmitterProcessor` 能够向多个订阅者发送数据，并且可以对每一个订阅者进行背压处理。它本身也可以订阅一个`Publisher` 并同步获得数据。

最初如果没有订阅者，它仍然允许推送一些数据到缓存，缓存大小由 `bufferSize` 定义。 之后如果仍然没有订阅者订阅它并消费数据，对 `onNext` 的调用会阻塞，直到有订阅者接入 （这时只能并发地订阅了）。

因此第一个订阅者会收到最多 `bufferSize` 个元素。然而之后， processor 不会重新发送（replay） 数据给后续的订阅者。这些后续接入的订阅者只能获取到它们开始订阅 **之后** 推送的数据。这个内部的 缓存会继续用于背压的目的。

默认情况下，如果所有的订阅者都取消了（基本意味着它们都不再订阅（un-subscribed）了）， 它会清空内部缓存，并且不再接受更多的订阅者。这一点可以通过 `create` 静态工厂方法的 `autoCancel` 参数来配置。

+ ReplayProcessor

`ReplayProcessor` 会缓存直接通过自身的 `Sink` 推送的元素，以及来自上游发布者的元素， 并且后来的订阅者也会收到重发（replay）的这些元素。

可以通过多种配置方式创建它：

- 缓存一个元素（`cacheLast`）。
- 缓存一定个数的历史元素（`create(int)`），所有的历史元素（`create()`）。
- 缓存基于时间窗期间内的元素（`createTimeout(Duration)`）。
- 缓存基于历史个数和时间窗的元素（`createSizeOrTimeout(int, Duration)`）。

+ TopicProcessor

`TopicProcessor` 是一个异步的 processor，它能够重发来自多个上游发布者的元素， 这需要在创建它的时候配置 `shared`（见 `build()` 的 `share(boolean)` 配置）。

注意，如果你企图在并发环境下通过并发的上游 Publisher 调用 `TopicProcessor` 的 `onNext`、 `onComplete`，或 `onError`方法，就必须配置 shared。

否则，并发调用就是非法的，从而 processor 是完全兼容响应式流规范的。

`TopicProcessor` 能够对多个订阅者发送数据。它通过对每一个订阅者关联一个线程来实现这一点， 这个线程会一直执行直到 processor 发出 `onError` 或 `onComplete` 信号，或关联的订阅者被取消。 最多可以接受的订阅者个数由构造者方法 `executor` 指定，通过提供一个有限线程数的 `ExecutorService` 来限制这一个数。

这个 processor 基于一个 `RingBuffer` 数据结构来存储已发送的数据。每一个订阅者线程 自行管理其相关的数据在 `RingBuffer` 中的索引。

这个 processor 也有一个 `autoCancel` 构造器方法：如果设置为 `true` （默认的），那么当 所有的订阅者取消之后，源 `Publisher`(s) 也就被取消了。

+ WorkQueueProcessor

`WorkQueueProcessor` 也是一个异步的 processor，也能够重发来自多个上游发布者的元素， 同样在创建时需要配置 `shared` （它多数构造器配置与 `TopicProcessor` 相同）。

它放松了对响应式流规范的兼容，但是好处就在于相对于 `TopicProcessor` 来说需要更少的资源。 它仍然基于 `RingBuffer`，但是不再要求每一个订阅者都关联一个线程，因此相对于 `TopicProcessor` 来说更具扩展性。

代价在于分发模式有些区别：来自订阅者的请求会汇总在一起，并且这个 processor 每次只对一个 订阅者发送数据，因此需要循环（round-robin）对订阅者发送数据，而不是一次全部发出的模式。

|      | 无法保证完全公平的循环分发。 |
| ---- | ---------------------------- |
|      |                              |

`WorkQueueProcessor` 多数构造器方法与 `TopicProcessor` 相同，比如 `autoCancel`、`share`， 以及 `waitStrategy`。下游订阅者的最大数目同样由构造器 `executor` 配置的 `ExecutorService` 决定。

|      | 你最好注意不要有太多订阅者订阅 `WorkQueueProcessor`，因为这 **会锁住 processor**。 如果你需要限制订阅者数量，最好使用一个 `ThreadPoolExecutor` 或 `ForkJoinPool`。这个 processor 能够检测到（线程池）容量并在订阅者过多时抛出异常。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |



### 4 测试

> `reactor-test` 的两个主要用途：
>
> - 使用 `StepVerifier` 一步一步地测试一个给定场景的序列。
> - 使用 `TestPublisher` 生成数据来测试下游的操作符（包括你自己的operator）。

``` xml
<dependency>
  <groupId>io.projectreactor</groupId>
  <artifactId>reactor-test</artifactId>
  <scope>test</scope>
</dependency>
```

``` groovy
dependencies {
   testCompile 'io.projectreactor:reactor-test'
}
```



#### 4.1 StepVerifier

``` java
flux = Flux.just("foo", "bar")
  .concatWith(Mono.error(new IllegalArguemntException("boom")));

// 测试构建
StepVerifier.create(flux)
  .exceptNext("foo")
  .exceptNext("bar")
  .exceptErrorMessage("boom")
  // 触发测试
  .verify();
```



API

+ exceptNext(T)
+ exceptNextCount(long)
+ consumeNextWith(Consumer<? super T>)
+ thenAwait(Duration)
+ then(Runnable)
+ exceptComplete()
+ exceptError()
+ verifyComplete()
+ verifyError()
+ verifyErrorMessage(String)

> `StepVerifier.setDefaultTimeout(Duration)`设置一个全局超时时间





### 5. 调试







### 6. 高级特性

+ 打包重用操作符
+ Hot vs Cold
+ ConnectableFlux 对多个订阅者进行广播
+ 三种分批处理方式
+ ParallelFlux 进行并行处理
+ 替换默认的 Scheduler
+ 使用全局 Hooks
+ 增加一个 Context 到响应式序列
+ 空置安全



#### 6.1 打包重用操作符

代码重用是一个好办法，如果你认为一段操作链很常用，你可以将该操作链打包封装后备用



##### 6.1.1 transform

**transform 操作符将一段操作链封装为一个函数「function」**

这个函数式能在操作期（assembly time）将被封装的操作符还原并接入到调用 transform 的位置。例如：

``` java
Function<Flux<String>, Flux<String>> f&m = f -> 
  f.filter(color -> !color.equals("orange")).map(String::toUpperCase);

Flux.fromIterable("blue", "green", "orange").doOnNext(sout)
  .transform(f&m)
  .subscribe(d -> sout("Subscriber to transformed MapAndFilter: " + d));

//执行结果
blue
Subscriber to Transformed MapAndFilter: BLUE
green
Subscriber to Transformed MapAndFilter: GREEN
orange
```



##### 6.1.2 compose

 `compose` 操作符与 `transform` 类似，也能够将几个操作符封装到一个函数式中。 主要的区别就是，这个函数式作用到原始序列上的话，是 **基于每一个订阅者的（on a per-subscriber basis）** 。这意味着它对每一个 subscription 可以生成不同的操作链（通过维护一些状态值）。 如下例所示： 

``` java
AtomicInteger ai = new AtomicInteger();
Function<Flux<String>, Flux<String>> filterAndMap = f -> {
  if (ai.incrementAndGet() == 1) {
    return f.filter(color -> !color.equals("orange"))
      .map(String::toUpperCase);
  }
  return f.filter(color -> !color.equals("purple"))
    .map(String::toUpperCase);
};

Flux<String> composedFlux =
  Flux.fromIterable(Arrays.asList("blue", "green", "orange"))
  .doOnNext(System.out::println)
  .compose(filterAndMap);

composedFlux.subscribe(d -> sout("Subscriber 1 to Composed MapAndFilter :"+d));
composedFlux.subscribe(d -> sout("Subscriber 2 to Composed MapAndFilter: "+d));

// 运行结果
blue
Subscriber 1 to Composed MapAndFilter :BLUE
green
Subscriber 1 to Composed MapAndFilter :GREEN
orange
blue
Subscriber 2 to Composed MapAndFilter: BLUE
green
Subscriber 2 to Composed MapAndFilter: GREEN
orange
```



#### 6.2 Hot & Cold

到目前为止，我们一直认为 `Flux`（和 `Mono`）都是这样的：它们都代表了一种异步的数据序列， 在订阅（subscribe）之前什么都不会发生。

但是实际上，广义上有两种发布者：“热”与“冷”（**hot** and **cold**）。

 （本文档）到目前介绍的其实都是 **cold** 家族的发布者。它们为每一个订阅（subscription） 都生成数据。如果没有创建任何订阅（subscription），那么就不会生成数据。 

 **热** 发布者，不依赖于订阅者的数量。即使没有订阅者它们也会发出数据， 如果有一个订阅者接入进来，那么它就会收到订阅之后发出的元素 



+ hot

``` java
UnicastProcessor<String> hotSource = UnicastProcessor.create();
Flux<String> hotFlux = hotSource.publish()
  .autoConnect()
  .map(String::toUpperCase);

hotFlux.subscribe(d -> System.out.println("Subscriber 1 to Hot Source: "+d));

hotSource.onNext("blue");
hotSource.onNext("green");

hotFlux.subscribe(d -> System.out.println("Subscriber 2 to Hot Source: "+d));

hotSource.onNext("orange");
hotSource.onComplete();

// 结果如下
Subscriber 1 to Hot Source: BLUE
Subscriber 1 to Hot Source: GREEN
Subscriber 1 to Hot Source: ORANGE
Subscriber 2 to Hot Source: ORANGE
```

> 第一个订阅者收到了所有的三个颜色，第二个订阅者只收到了订阅之后 publisher 发出的元素



#### 6.3 ConnectableFlux 对多个订阅者进行广播

某些时候，你可能需要所有订阅者到齐之后才开始发布元素

**Flux API 有两种返回 ConnectableFlux 的方式：**

+ ==publish==：会尝试满足各个不同订阅者的需求「背压」，并综合这些请求反馈給源，尤其是如果有某个订阅者的需求为0，publish 会暂停它对源的请求。
+ ==replay==：将对第一个订阅后产生的数据进行缓存，最多缓存数量取决于配置「时间 / 缓存大小」，它会对后续接入的订阅者重新发送数据

`ConnectableFlux` 提供了多种对下游订阅的管理：

+ connect()：对 Flux 手动执行一次，它会触发对上游源的订阅
+ autoConnect(n)：在有 n 个订阅时自动触发对上游源的订阅
+ refCount(n)：不仅能偶在帝国约占接入时自动触发，还会检测订阅者的取消动作，如果订阅者数量 < n，会将源“断开连接”，再添加订阅者时才会继续连上“源”

+ refCount(int, Duration)，在 refCount(n) 的基础上添加超时控制



connect()

``` java
Flux<Integer> source = Flux.range(1, 2)
  .doOnSubscribe(s -> System.out.println("subscribed to source"));

ConnectableFlux<Integer> co = source.publish();

co.subscribe(System.out::println, e -> {}, () -> {});
co.subscribe(System.out::println, e -> {}, () -> {});

System.out.println("done subscribing");
Thread.sleep(500);
System.out.println("will now connect");

// 手动触发一次对上游源的订阅
co.connect();

// 执行结果
done subscribing
will now connect
subscribed to source
1
1
2
2
```



autoConnect(int)

``` java
Flux<Integer> source = Flux.range(1, 3)
  .doOnSubscribe(s -> System.out.println("subscribed to source"));
// 订阅者达到2个时，自动触发对上游源的订阅
Flux<Integer> autoCo = source.publish().autoConnect(2);

autoCo.subscribe(System.out::println, e -> {}, () -> {});
System.out.println("subscribed first");
Thread.sleep(500);
System.out.println("subscribing second");
autoCo.subscribe(System.out::println, e -> {}, () -> {});

// 执行结果
subscribed first
subscribing second
subscribed to source
1
1
2
2
```



#### 6.4 三种分批处理方式

当你有许多元素并且想分批处理，Reactor 提供三种方案：

+ **分组「grouping」**

+ **窗口「windowing」**

+ **缓存「buffer」**

> 分组和分段操作都会创建一个 `Flux<Flux<T>>`，而缓存操作得到的是一个 `Flux<Collection<T>>`



##### 6.4.1 分组

用 `Flux<GroupedFlux<T>>` 进行分组

分组能够根据 key 将源 `Flux<T>` 拆分为多个批次，对应的操作符是 groupBy

每一组用 `GroupedFlux<T>` 类型表示，使用 key() 可以得到改组的 key

在组内，元素并不需要连续，当源发出一个新的元素，该元素会被分发到与之匹配的 key 所对应的组中（如果还没有该 key 对应的组，则创建一个）

+ 组互相没有交集的
+ 组会包含原始序列中任意位置的元素
+ 不会为空

``` java
StepVerifier.create(
	Flux.just(1, 2, 3, 4, 5, 6)
  .groupBy(i -> (i & 1) == 0 ? "even" : "odd")
  .concatMap(g -> g.defaultIfEmpty(-1)
            .map(String::valueOf)
            .startWith(g.key()))
).exceptNext("odd", "1", "3", "5")
  .exceptNext("even", "2", "4", "6")
  .verifyComplete();
```



##### 6.4.2 window

`Flux<Flux<T>>` 进行 window 操作

window 根据 个数，时间条件，或能够定义边界的发布者「boundary-defining Publisher」，把源 `Flux<T>` 拆分为 windows，返回类型 `Flux<Flux<T>>`

对应的 API 有：

**window、windowTimeout、windowUntil、windowWhile、windowWhen**

与 groupBy 主要区别在于，窗口操作能保持序列稳定性，并且同一时刻最多能有2个 windows 是开启的

window 可以重叠，操作符参数有 maxSize 和 skip

maxSize：指定收集多少个元素后就关闭 window

skip：指定收集多少个元素后就打开下一个 window，如果 maxSize > skip，两个 window 会重叠

``` java
StepVerifier.create(
  Flux.range(1, 10)
  .window(5, 3) //overlapping windows
  .concatMap(g -> g.defaultIfEmpty(-1)) //将 windows 显示为 -1
)
  .expectNext(1, 2, 3, 4, 5)
  .expectNext(4, 5, 6, 7, 8)
  .expectNext(7, 8, 9, 10)
  .expectNext(10)
  .verifyComplete();
```

>  如果将两个参数的配置反过来（`maxSize` < `skip`），序列中的一些元素就会被丢弃掉， 而不属于任何 window。 

对于条件判断的 windowUntil 和 windowWhile，如果序列中元素不匹配判断条件，那么可能导致空 windows



##### 6.4.3 缓存

使用 `Flux<List<T>>` 进行缓存

缓存与窗口类似，缓存操作之后会发出 buffers（类型为 `Collection<T>`，默认值 `List<T>`），而不是 windows（类型为 `Flux<T>`）

缓存的 API 如下：

**buffer、bufferTimeout、bufferUntil、bufferWhile、bufferWhen**

如果对于窗口操作符是开启一个窗口，那么对于缓存操作符就是创建一个新的集合，然后对其添加元素

``` java
StepVerifier.create(
	Flux.range(1, 8).buffer(5, 3)
)
  .exceptNext(Arrays.asList(1, 2, 3, 4, 5))
  .exceptNext(Arrays.asList(4, 5, 6, 7, 8))
  .exceptNext(Collections.singletonList(7))
  .verifyComplete();
```



#### 6.5 ParallerFlux

使用 `ParallelFulx<T>` 进行并行处理

`flux.parallel()` 将得到一个 `ParallelFulx<T>`，将负载划分到多个“轨道（rails）”，默认：轨道个数与 CPU 核数相等。

给定并行数：

``` java
Flux.range(1, 2)
    .parallel(2) 
    .subscribe(i -> System.out.println(Thread.currentThread().getName() + " -> " + i));

// 结果
main 1
main 2

Flux.range(1, 2)
    .parallel(2)
  	// 指定调度器
    .runOn(Schedulers.parallel())
    .subscribe(i -> System.out.println(Thread.currentThread().getName() + " -> " + i));

// 执行结果
parallel-1 -> 1
parallel-2 -> 2
```

> 如果想退回到单线程的序列，使用 `sequential()` 即可
>
>  注意 `subscribe(Subscriber)` 会合并所有的执行轨道，而 `subscribe(Consumer)` 会在所有轨道上运行。 
>
> 意思是：subscribe(Subscriber) API 会自动调用 sequential()，从而串行



#### 6.6 替换 Schedulers

 Reactor Core 内置许多 `Scheduler` 的具体实现。 你可以用形如 `new*` 的工厂方法来创建调度器，每一种调度器都有一个单例对象，**你可以使用单例工厂方法 （比如 `Schedulers.elastic()` 而不是 `Schedulers.newElastic()`）来获取它。** 



``` java
Schedulers.elastic();
Schedulers.paralle();
Schedulers.boundedElastic();
Schedulers.single();
```

> 当你不明确指定调度器的时候，那些需要调度器的操作符会使用这些默认的单例调度器
>
> 例如：
>
> `Flux#delayElements(Duration)` 使用的是 `Schedulers.parallel()` 调度器对象
>
> 你也可以选择对已有的调度器进行简单的包装，比如：统计每个被调度任务执行时常

+ 统一的更换所有添加功能后的调度器

**`Schedulers.Factory`** 类来改变默认的调度器

`Schedulers.setFactory(factory)`





#### 6.7 全局 Hooks

Reactor 的可配置的应用于多种场合的回调，他们在 Hooks 类中定义，总体分三类：

+ 丢弃事件的 Hooks
+ 内部错误 Hooks
+ 组装 Hooks



##### 6.7.1 丢弃事件的 Hooks

当生成源的操作符不遵循响应式规范的时候，Dropping hooks（处理丢弃事件的 hooks）会被调用，这种类型的错误是出于正常的执行机制之外的（也就是说不能通过 onError 传播）

典型的栗子是：

一个发布者在被调用 onComplete 之后仍然可以调用 onNext 操作符，这种情况 onNext 的值会被丢弃，如果有其它多余的 onError 信号亦是如此。

相应的 hook：**onNextDropped，onErrorDropped** 可提供一个全局的 Consumer，以便能够在被丢弃的时候进行处理，例如：你可以用它对丢弃事件记录日志，或资源清理



##### 6.7.2 内部错误 hook

如果操作符在执行其 onNext / onError / onComplete 方法的时候抛出异常，那么 **onOperatorError** 这个钩子将被调用。

这个 hook 处于正常路径，一个典型的栗子是：map 操作产生 RuntimeException，这时候还会执行到 onError

首先，它将被传递给 onOperatorError，利用这个 hook 可以检查错误，或者可以改变异常，或者记录日志...

 默认的 hook 可以使用 `Hooks.resetOnOperatorError()` 方法重置 



##### 6.7.3 组装 Hooks

组装「assembly」hooks 关联了操作符的生命周期，它们会在一个操作链被组装起来的时候（实例化时）被调用。每一个新的操作符组装到操作链时，**onEachOperator，onLastOperator** 





##### 6.7.4 预置 hooks

> **Hooks 工具类还提供了一些内建「built-in」的 hooks**

+ onNextDroppedFail()： `onNextDropped` 通常会抛出 `Exceptions.failWithCancel()` 异常。 现在它默认还会以 DEBUG 级别对被丢弃的值记录日志。如果想回到原来的只是抛出异常的方式，使用 `onNextDroppedFail()`。 

+ onOperatorDebug()：  这个方法会激活 debug mode。 



#### 6.8 Context

**==当从命令式编程风格切换到响应式编程风格的时候，一个最大的挑战就是线程处理==**

**在响应式编程中，一个线程「Thread」可被用于处理多个同时运行的异步序列（实际上是非阻塞），执行过程也经常从一个线程切换到另一个线程**

这样的情况，对开发者来说，如果依赖线程模型中相对“稳定”的特性——比如 `ThreadLocal` ，它会让你将数据绑定到一个线程上，这在响应式编程环境中使用就会很困难（因为执行过程中可能会切换线程）

通常对 ThreadLocal 的替代方案是：

+ **Tuple**

**==将环境相关的数据 C，同业务数据 T 一起置于序列中，例如：使用`Tuple2<T, C>` 。但这种方案不是很好，它会在方法和反省中暴露环境数据信息==**

+ **Context**

Reactor 引入一个类似于 ThreadLocal 的高级功能：Context，它作用于一个 Flux / Mono 上，而不是线程

``` java
// 读写 Context 的栗子
String key = "message";
Mono<String> mono = Mono.just("Hello")
  .flatMap(s -> Mono.subscriberContext().map(ctx -> s + " " + ctx.get(key)))
  .subscriberContext(ctx -> ctx.put(key, "World"));

StepVerifier.create(r)
  .exceptNext("hello world")
  .verifyComplete();

```

>  这是一个主要面向库开发人员的高级功能。这需要开发者对 `Subscription` 的生命周期 充分理解，并且明白它主要用于 subscription 相关的库。 



##### 6.8.1 Context API

> Context 是类似于 Map 的接口：它存储 k - v 对
>
> + k v 都是 Object 类型
> + Context **不可变「immutable」**
> + put(k, v) 来存储一个键值对，返回一个新的 Context 对象，也可以使用 putAll(Context) 合并
> + hasKey(k) 检查一个 key 是否存在
> + getOrDefault(k, T defaultVal) 
> + getOrEmpty(k) 来得到一个 `Optional<T>`，Context 会藏尸将值转换为 T
> + delete(k) 来删除 key 关联的值，并返回一个新的 Context

创建 Context 也可以使用静态方法 Context.of 预先存储最多 5 个键值对

也可以使用 Context.empty() 创建一个空 Context



##### 6.8.2 绑定 Context 到 Flux and Writing

为了使用 Context，它必须绑定到一个序列，并且链上的每个操作符都可以访问它

**实际上，一个 Context 是绑定到每一个链中的 Subscriber 上的**，它使用 Subscription 传播机制来让自己对每一个操作符都可见（从最后一个 subscribe 沿操作链向上）

**为了填充 Context（只能在订阅时填充），你需要使用 `subscriberContext` 操作符**

`subscriberContext(Context)` 会将你提供的 Context 与来自下游的（Context 是从下游向上游传播）的 Context 合并，这是通过 `Context putAll(Context)` 实现的，最后生成一个新的 Context 給上游

>  你也可以用更高级的 `subscriberContext(Function)`。它接受来自下游的`Context`，然后你可以根据需要添加或删除值，然后返回新的 `Context`。 



##### 6.8.3 读取 Context

读取 Context 数据同样重要，多数时候添加到 Context 是用户的责任，但利用数据是库的责任，因为库通常是客户代码的上游。

使用 `Mono.subscriberContext()` 读取数据



##### 6.8.4 栗子

本例是为了让你对如何使用 Context 有个更好的理解

``` java
String key = "message";
Mono<String> r = Mono.just("Hello")
  .subscriberContext(ctx -> ctx.put(key, "World")) 
  .flatMap( s -> Mono.subscriberContext()
           .map( ctx -> s + " " + ctx.getOrDefault(key, "Stranger")));  
StepVerifier.create(r)
  .expectNext("Hello Stranger") 
  .verifyComplete();

Mono<String> r = Mono.subscriberContext() 
  .map( ctx -> ctx.put(key, "Hello")) 
  .flatMap( ctx -> Mono.subscriberContext()) 
  .map( ctx -> ctx.getOrDefault(key,"Default")); 
StepVerifier.create(r)
  .expectNext("Default") 
  .verifyComplete();

Mono<String> r = Mono.just("Hello")
  .flatMap( s -> Mono.subscriberContext().map( ctx -> s + " " + ctx.get(key)))
  .subscriberContext(ctx -> ctx.put(key, "Reactor")) 
  .subscriberContext(ctx -> ctx.put(key, "World")); 
StepVerifier.create(r)
  .expectNext("Hello Reactor") 
  .verifyComplete();

Mono<String> r = Mono.just("Hello")
  .flatMap( s -> Mono.subscriberContext().map( ctx -> s + " " + ctx.get(key))) 
  .subscriberContext(ctx -> ctx.put(key, "Reactor")) 
  .flatMap( s -> Mono.subscriberContext().map( ctx -> s + " " + ctx.get(key))) 
  .subscriberContext(ctx -> ctx.put(key, "World")); 
StepVerifier.create(r)
  .expectNext("Hello Reactor World") 
  .verifyComplete();
```



##### 6.8.5 完整的栗子



``` java
// 用户调用
doPut("www.example.com", Mono.just("Walter"));

// 为了传播一个关联ID，应该这样调用
doPut("www.example.com", Mono.just("Walter"))
        .subscriberContext(Context.of(HTTP_CORRELATION_ID, "2-j3r9afaf92j-afkaf"));


// 以下演示了从库的角度由 context 读取值的模拟代码
static final String HTTP_CORRELATION_ID = "reactive.http.library.correlationId";

Mono<Tuple2<Integer, String>> doPut(String url, Mono<String> data) {
  Mono<Tuple2<String, Optional<Object>>> dataAndContext =
    data.zipWith(Mono.subscriberContext() 
                 .map(c -> c.getOrEmpty(HTTP_CORRELATION_ID))); 

  return dataAndContext.<String>handle((dac, sink) -> {
      if (dac.getT2().isPresent()) { 
        sink.next("PUT <" + dac.getT1() + "> sent to " + url + " with header X-Correlation-ID = " + dac.getT2().get());
      }
      else {
        sink.next("PUT <" + dac.getT1() + "> sent to " + url);
      }
      sink.complete();
    })
    .map(msg -> Tuples.of(200, msg));
}
```



### 7. Appendix A

>  TIP：在这一节，如果一个操作符是专属于 `Flux` 或 `Mono` 的，那么会给它注明前缀。 公共的操作符没有前缀。如果一个具体的用例涉及多个操作符的组合，这里以方法调用的方式展现， 会以一个点（.）开头，并将参数置于圆括号内，比如： `.methodCall(parameter)`。 

+ 创建一个新序列
+ 对序列进行转换
+ 过滤序列
+ “窥视（只读）序列”
+ 错误处理
+ 基于时间的操作
+ 拆分 Flux
+ 回到同步的世界



#### A.1 创建序列

- 发出一个 `T`，我已经有了：`just`
  - …基于一个 `Optional`：`Mono#justOrEmpty(Optional)`
  - …基于一个可能为 `null` 的 T：`Mono#justOrEmpty(T)`
- 发出一个 `T`，且还是由 `just` 方法返回
  - …但是“懒”创建的：使用 `Mono#fromSupplier` 或用 `defer` 包装 `just`
- 发出许多 `T`，这些元素我可以明确列举出来：`Flux#just(T...)`
- 基于迭代数据结构:
  - 一个数组：`Flux#fromArray`
  - 一个集合或 iterable：`Flux#fromIterable`
  - 一个 Integer 的 range：`Flux#range`
  - 一个 `Stream` 提供给每一个订阅：`Flux#fromStream(Supplier)`
- 基于一个参数值给出的源：
  - 一个 `Supplier`：`Mono#fromSupplier`
  - 一个任务：`Mono#fromCallable`，`Mono#fromRunnable`
  - 一个 `CompletableFuture`：`Mono#fromFuture`
- 直接完成：`empty`
- 立即生成错误：`error`
  - …但是“懒”的方式生成 `Throwable`：`error(Supplier)`
- 什么都不做：`never`
- 订阅时才决定：`defer`
- 依赖一个可回收的资源：`using`
- 可编程地生成事件（可以使用状态）:
  - 同步且逐个的：`Flux#generate`
  - 异步（也可同步）的，每次尽可能多发出元素：`Flux#create` （`Mono#create` 也是异步的，只不过只能发一个）



#### 		A.2 序列转化

- 我想转化一个序列：
  - 1对1地转化（比如字符串转化为它的长度）：`map`
    - …类型转化：`cast`
    - …为了获得每个元素的序号：`Flux#index`
  - 1对n地转化（如字符串转化为一串字符）：`flatMap` + 使用一个工厂方法
  - 1对n地转化可自定义转化方法和/或状态：`handle`
  - 对每一个元素执行一个异步操作（如对 url 执行 http 请求）：`flatMap` + 一个异步的返回类型为 `Publisher` 的方法
    - …忽略一些数据：在 flatMap lambda 中根据条件返回一个 `Mono.empty()`
    - …保留原来的序列顺序：`Flux#flatMapSequential`（对每个元素的异步任务会立即执行，但会将结果按照原序列顺序排序）
    - …当 Mono 元素的异步任务会返回多个元素的序列时：`Mono#flatMapMany`
- 我想添加一些数据元素到一个现有的序列：
  - 在开头添加：`Flux#startWith(T...)`
  - 在最后添加：`Flux#concatWith(T...)`
- 我想将 `Flux` 转化为集合（一下都是针对 `Flux` 的）
  - 转化为 List：`collectList`，`collectSortedList`
  - 转化为 Map：`collectMap`，`collectMultiMap`
  - 转化为自定义集合：`collect`
  - 计数：`count`
  - reduce 算法（将上个元素的reduce结果与当前元素值作为输入执行reduce方法，如sum） `reduce`
    - …将每次 reduce 的结果立即发出：`scan`
  - 转化为一个 boolean 值：
    - 对所有元素判断都为true：`all`
    - 对至少一个元素判断为true：`any`
    - 判断序列是否有元素（不为空）：`hasElements`
    - 判断序列中是否有匹配的元素：`hasElement`
- 我想合并 publishers…
  - 按序连接：`Flux#concat` 或 `.concatWith(other)`
    - …即使有错误，也会等所有的 publishers 连接完成：`Flux#concatDelayError`
    - …按订阅顺序连接（这里的合并仍然可以理解成序列的连接）：`Flux#mergeSequential`
  - 按元素发出的顺序合并（无论哪个序列的，元素先到先合并）：`Flux#merge` / `.mergeWith(other)`
    - …元素类型会发生变化：`Flux#zip` / `Flux#zipWith`
  - 将元素组合：
    - 2个 Monos 组成1个 `Tuple2`：`Mono#zipWith`
    - n个 Monos 的元素都发出来后组成一个 Tuple：`Mono#zip`
  - 在终止信号出现时“采取行动”：
    - 在 Mono 终止时转换为一个 `Mono`：`Mono#and`
    - 当 n 个 Mono 都终止时返回 `Mono`：`Mono#when`
    - 返回一个存放组合数据的类型，对于被合并的多个序列：
      - 每个序列都发出一个元素时：`Flux#zip`
      - 任何一个序列发出元素时：`Flux#combineLatest`
  - 只取各个序列的第一个元素：`Flux#first`，`Mono#first`，`mono.or (otherMono).or(thirdMono)`，`flux.or(otherFlux).or(thirdFlux)
  - 由一个序列触发（类似于 `flatMap`，不过“喜新厌旧”）：`switchMap`
  - 由每个新序列开始时触发（也是“喜新厌旧”风格）：`switchOnNext`
- 我想重复一个序列：`repeat`
  - …但是以一定的间隔重复：`Flux.interval(duration).flatMap(tick -> myExistingPublisher)`
- 我有一个空序列，但是…
  - 我想要一个缺省值来代替：`defaultIfEmpty`
  - 我想要一个缺省的序列来代替：`switchIfEmpty`
- 我有一个序列，但是我对序列的元素值不感兴趣：`ignoreElements`
  - …并且我希望用 `Mono` 来表示序列已经结束：`then`
  - …并且我想在序列结束后等待另一个任务完成：`thenEmpty`
  - …并且我想在序列结束之后返回一个 `Mono`：`Mono#then(mono)`
  - …并且我想在序列结束之后返回一个值：`Mono#thenReturn(T)`
  - …并且我想在序列结束之后返回一个 `Flux`：`thenMany`
- 我有一个 Mono 但我想延迟完成…
  - …当有1个或N个其他 publishers 都发出（或结束）时才完成：`Mono#delayUntilOther`
    - …使用一个函数式来定义如何获取“其他 publisher”：`Mono#delayUntil(Function)`
- 我想基于一个递归的生成序列的规则扩展每一个元素，然后合并为一个序列发出：
  - …广度优先：`expand(Function)`
  - …深度优先：`expandDeep(Function)`



#### A.3. 只读序列

- 在不对序列造成改变的情况下，我想：
  - 得到通知或执行一些操作：
    - 发出元素：`doOnNext`
    - 序列完成：`Flux#doOnComplete`，`Mono#doOnSuccess`
    - 因错误终止：`doOnError`
    - 取消：`doOnCancel`
    - 订阅时：`doOnSubscribe`
    - 请求时：`doOnRequest`
    - 完成或错误终止：`doOnTerminate`（Mono的方法可能包含有结果）
      - 但是在终止信号向下游传递 **之后** ：`doAfterTerminate`
    - 所有类型的信号（`Signal`）：`Flux#doOnEach`
    - 所有结束的情况（完成complete、错误error、取消cancel）：`doFinally`
  - 记录日志：`log`
- 我想知道所有的事件:
  - 每一个事件都体现为一个 `single` 对象：
    - 执行 callback：`doOnEach`
    - 每个元素转化为 `single` 对象：`materialize`
      - …在转化回元素：`dematerialize`
  - 转化为一行日志：`log`



#### A.4. 过滤序列

- 我想过滤一个序列
  - 基于给定的判断条件：`filter`
    - …异步地进行判断：`filterWhen`
  - 仅限于指定类型的对象：`ofType`
  - 忽略所有元素：`ignoreElements`
  - 去重:
    - 对于整个序列：`Flux#distinct`
    - 去掉连续重复的元素：`Flux#distinctUntilChanged`
- 我只想要一部分序列：
  - 只要 N 个元素：
    - 从序列的第一个元素开始算：`Flux#take(long)`
      - …取一段时间内发出的元素：`Flux#take(Duration)`
      - …只取第一个元素放到 `Mono` 中返回：`Flux#next()`
      - …使用 `request(N)` 而不是取消：`Flux#limitRequest(long)`
    - 从序列的最后一个元素倒数：`Flux#takeLast`
    - 直到满足某个条件（包含）：`Flux#takeUntil`（基于判断条件），`Flux#takeUntilOther`（基于对 publisher 的比较）
    - 直到满足某个条件（不包含）：`Flux#takeWhile`
  - 最多只取 1 个元素：
    - 给定序号：`Flux#elementAt`
    - 最后一个：`.takeLast(1)`
      - …如果为序列空则发出错误信号：`Flux#last()`
      - …如果序列为空则返回默认值：`Flux#last(T)`
  - 跳过一些元素：
    - 从序列的第一个元素开始跳过：`Flux#skip(long)`
      - …跳过一段时间内发出的元素：`Flux#skip(Duration)`
    - 跳过最后的 n 个元素：`Flux#skipLast`
    - 直到满足某个条件（包含）：`Flux#skipUntil`（基于判断条件），`Flux#skipUntilOther` （基于对 publisher 的比较）
    - 直到满足某个条件（不包含）：`Flux#skipWhile`
  - 采样：
    - 给定采样周期：`Flux#sample(Duration)`
      - 取采样周期里的第一个元素而不是最后一个：`sampleFirst`
    - 基于另一个 publisher：`Flux#sample(Publisher)`
    - 基于 publisher“超时”：`Flux#sampleTimeout` （每一个元素会触发一个 publisher，如果这个 publisher 不被下一个元素触发的 publisher 覆盖就发出这个元素）
- 我只想要一个元素（如果多于一个就返回错误）…
  - 如果序列为空，发出错误信号：`Flux#single()`
  - 如果序列为空，发出一个缺省值：`Flux#single(T)`
  - 如果序列为空就返回一个空序列：`Flux#singleOrEmpty`



#### A.5. 错误处理

- 我想创建一个错误序列：`error`…
  - …替换一个完成的 `Flux`：`.concat(Flux.error(e))`
  - …替换一个完成的 `Mono`：`.then(Mono.error(e))`
  - …如果元素超时未发出：`timeout`
  - …“懒”创建：`error(Supplier)`
- 我想要类似 try/catch 的表达方式：
  - 抛出异常：`error`
  - 捕获异常：
    - 然后返回缺省值：`onErrorReturn`
    - 然后返回一个 `Flux` 或 `Mono`：`onErrorResume`
    - 包装异常后再抛出：`.onErrorMap(t -> new RuntimeException(t))`
  - finally 代码块：`doFinally`
  - Java 7 之后的 try-with-resources 写法：`using` 工厂方法
- 我想从错误中恢复…
  - 返回一个缺省的：
    - 的值：`onErrorReturn`
    - `Publisher`：`Flux#onErrorResume` 和 `Mono#onErrorResume`
  - 重试：`retry`
    - …由一个用于伴随 Flux 触发：`retryWhen`
- 我想处理回压错误（向上游发出“MAX”的 request，如果下游的 request 比较少，则应用策略）…
  - 抛出 `IllegalStateException`：`Flux#onBackpressureError`
  - 丢弃策略：`Flux#onBackpressureDrop`
    - …但是不丢弃最后一个元素：`Flux#onBackpressureLatest`
  - 缓存策略（有限或无限）：`Flux#onBackpressureBuffer`
    - …当有限的缓存空间用满则应用给定策略：`Flux#onBackpressureBuffer` 带有策略 `BufferOverflowStrategy`



#### A.6. 基于时间的操作

- 我想将元素转换为带有时间信息的 `Tuple2`…
  - 从订阅时开始：`elapsed`
  - 记录时间戳：`timestamp`
- 如果元素间延迟过长则中止序列：`timeout`
- 以固定的周期发出元素：`Flux#interval`
- 在一个给定的延迟后发出 `0`：static `Mono.delay`.
- 我想引入延迟：
  - 对每一个元素：`Mono#delayElement`，`Flux#delayElements`
  - 延迟订阅：`delaySubscription`



#### A.7. 拆分 `Flux`

- 我想将一个 `Flux` 拆分为一个 `Flux>`：
  - 以个数为界：`window(int)`
    - …会出现重叠或丢弃的情况：`window(int, int)`
  - 以时间为界：`window(Duration)`
    - …会出现重叠或丢弃的情况：`window(Duration, Duration)`
  - 以个数或时间为界：`windowTimeout(int, Duration)`
  - 基于对元素的判断条件：`windowUntil`
    - …触发判断条件的元素会分到下一波（`cutBefore` 变量）：`.windowUntil(predicate, true)`
    - …满足条件的元素在一波，直到不满足条件的元素发出开始下一波：`windowWhile` （不满足条件的元素会被丢弃）
  - 通过另一个 Publisher 的每一个 onNext 信号来拆分序列：`window(Publisher)`，`windowWhen`
- 我想将一个 `Flux` 的元素拆分到集合…
  - 拆分为一个一个的 `List`:
    - 以个数为界：`buffer(int)`
      - …会出现重叠或丢弃的情况：`buffer(int, int)`
    - 以时间为界：`buffer(Duration)`
      - …会出现重叠或丢弃的情况：`buffer(Duration, Duration)`
    - 以个数或时间为界：`bufferTimeout(int, Duration)`
    - 基于对元素的判断条件：`bufferUntil(Predicate)`
      - …触发判断条件的元素会分到下一个buffer：`.bufferUntil(predicate, true)`
      - …满足条件的元素在一个buffer，直到不满足条件的元素发出开始下一buffer：`bufferWhile(Predicate)`
    - 通过另一个 Publisher 的每一个 onNext 信号来拆分序列：`buffer(Publisher)`，`bufferWhen`
  - 拆分到指定类型的 "collection"：`buffer(int, Supplier)`
- 我想将 `Flux` 中具有共同特征的元素分组到子 Flux：`groupBy(Function)` TIP：注意返回值是 `Flux>`，每一个 `GroupedFlux` 具有相同的 key 值 `K`，可以通过 `key()` 方法获取。



#### A.8. 回到同步的世界

- 我有一个 `Flux`，我想：
  - 在拿到第一个元素前阻塞：`Flux#blockFirst`
    - …并给出超时时限：`Flux#blockFirst(Duration)`
  - 在拿到最后一个元素前阻塞（如果序列为空则返回 null）：`Flux#blockLast`
    - …并给出超时时限：`Flux#blockLast(Duration)`
  - 同步地转换为 `Iterable`：`Flux#toIterable`
  - 同步地转换为 Java 8 `Stream`：`Flux#toStream`
- 我有一个 `Mono`，我想：
  - 在拿到元素前阻塞：`Mono#block`
    - …并给出超时时限：`Mono#block(Duration)`
  - 转换为 `CompletableFuture`：`Mono#toFuture`



### 8. Appendix B



#### B.1 如何包装一个同步阻塞的调用？

很多时候，信息源是同步阻塞的，在 Reactor 中我们可以这样处理：

``` JAVA
Mono blockingWrapper = Mono.fromCallable(() -> return synchroRpcCall());
blockingWrapper.subscribeOn(Schedulers.elastic());
```

> 1. 使用 fromCallable 生成一个 Mono
>
> 2. 返回同步、阻塞的资源
>
> 3. 使用 Schedulers.elastic() 确保每一个订阅都运行在一个专门的线程上
>
>    **Schedulers.elastic() 会创建一个线程来等待阻塞的调用返回，subscribeOn 并不会订阅这个 Mono，而是指定了订阅操作使用哪个调度器「Scheduler」**



#### B.2. 用在 `Flux` 上的操作符好像没起作用，为啥？

请确认你确实对调用 `.subscribe()` 的发布者应用了这个操作符。

Reactor 的操作符是装饰器（decorators）。它们会返回一个不同的（发布者）实例， 这个实例对上游序列进行了包装并增加了一些的处理行为。所以，最推荐的方式是将操作符“串”起来。

对比下边的两个例子：

没有串起来（**不正确的**）

```
Flux<String> flux = Flux.just("foo", "chain");
// 返回的是匿名的 Flux
flux.map(secret -> secret.replaceAll(".", "*")); 
flux.subscribe(next -> System.out.println("Received: " + next));
```

**正确的**

``` java
Flux<String> secrets = Flux
  .just("foo", "chain")
  .map(secret -> secret.replaceAll(".", "*"))
  .subscribe(next -> System.out.println("Received: " + next));
```



#### B.3 Mono# zipWith / zipWhen 没有被调用

``` java
myMethod.process("a") // 这个方法返回 Mono<Void>
  .zipWith(myMethod.process("b"), combinator) //没有被调用
  .subscribe();
```

 如果源 `Mono` 为空或是一个 `Mono`（`Mono` 通常用于“空”的场景）， 下边的组合操作就不会被调用。 

在 zipWhen 前使用 defaultIfEmpty

``` java
myMethod.emptySequenceForKey("a") // 这个方法返回一个空的 Mono<String>
  .defaultIfEmpty("") // 将空序列转换为包含字符串 "" 的序列
  .zipWhen(aString -> myMethod.process("b")) // 当 "" 发出时被调用
  .subscribe();
```



#### B.4 如何使用 retryWhen 实现 retry(3)

``` java
Flux<String> flux =
  Flux.<String>error(new IllegalArgumentException())
  .retryWhen(companion -> companion.zipWith(Flux.range(1, 4), 
                      (error, index) -> { 
                        if (index < 4) return index; 
                        else throw Exceptions.propagate(error); 
                      })
            );
```

|      | 技巧一：使用 `zip` 和一个“重试个数 + 1”的 `range`。          |
| ---- | ------------------------------------------------------------ |
|      | `zip` 方法让你可以在对重试次数计数的同时，仍掌握着原始的错误（error）。 |
|      | 允许三次重试，小于 4 的时候发出一个值。                      |
|      | 为了使序列以错误结束。我们将原始异常在三次重试之后抛出。     |



#### B.5. 如何使用 `retryWhen` 进行 exponential backoff？

Exponential backoff 的意思是进行的多次重试之间的间隔越来越长， 从而避免对源系统造成过载，甚至宕机。基本原理是，如果源产生了一个错误， 那么已经是处于不稳定状态，可能不会立刻复原。所以，如果立刻就重试可能会产生另一个错误， 导致源更加不稳定。

下面是一段实现 exponential backoff 效果的例子，每次重试的间隔都会递增 （伪代码： delay = attempt number * 100 milliseconds）：

```
Flux<String> flux =
Flux.<String>error(new IllegalArgumentException())
    .retryWhen(companion -> companion
        .doOnNext(s -> System.out.println(s + " at " + LocalTime.now())) 
        .zipWith(Flux.range(1, 4), (error, index) -> { 
          if (index < 4) return index;
          else throw Exceptions.propagate(error);
        })
        .flatMap(index -> Mono.delay(Duration.ofMillis(index * 100))) 
        .doOnNext(s -> System.out.println("retried at " + LocalTime.now())) 
    );
```

|      | 记录错误出现的时间；                                   |
| ---- | ------------------------------------------------------ |
|      | 使用 `retryWhen` + `zipWith` 的技巧实现重试3次的效果； |
|      | 通过 `flatMap` 来实现延迟时间递增的效果；              |
|      | 同样记录重试的时间。                                   |

订阅它，输出如下：

```
java.lang.IllegalArgumentException at 18:02:29.338
retried at 18:02:29.459 
java.lang.IllegalArgumentException at 18:02:29.460
retried at 18:02:29.663 
java.lang.IllegalArgumentException at 18:02:29.663
retried at 18:02:29.964 
java.lang.IllegalArgumentException at 18:02:29.964
```

|      | 第一次重试延迟大约 100ms |
| ---- | ------------------------ |
|      | 第二次重试延迟大约 200ms |
|      | 第三次重试延迟大约 300ms |



#### B.6. How do I ensure thread affinity using `publishOn()`?

如 Schedulers 所述，`publishOn()` 可以用来切换执行线程。 `publishOn` 能够影响到其之后的操作符的执行线程，直到有新的 `publishOn` 出现。 所以 `publishOn` 的位置很重要。

比如下边的例子， `map()` 中的 `transform` 方法是在 `scheduler1` 的一个工作线程上执行的， 而 `doOnNext()` 中的 `processNext` 方法是在 `scheduler2` 的一个工作线程上执行的。 单线程的调度器可能用于对不同阶段的任务或不同的订阅者确保线程关联性。

```java
EmitterProcessor<Integer> processor = EmitterProcessor.create();
processor.publishOn(scheduler1)
  .map(i -> transform(i))
  .publishOn(scheduler2)
  .doOnNext(i -> processNext(i))
  .subscribe();
```



### 9. Appendix C: Reactor-Extra

`reactor-extra` 为满足 `reactor-core` 用户的更高级需求，提供了一些额外的操作符和工具。

由于这是一个单独的包，使用时需要明确它的依赖：

```
dependencies {
     compile 'io.projectreactor:reactor-core'
     compile 'io.projectreactor.addons:reactor-extra' 
}
```

|      | 添加 reactor-extra 的依赖。参考 获取 Reactor 了解为什么使用BOM的情况下不需要指定 version。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |



在 Java 8 提供的函数式接口基础上，`reactor.function` 包又提供了一些支持 3 到 8 个值的 `Function`、`Predicate` 和 `Consumer`。

`TupleUtils` 提供的静态方法可以方便地用于将相应的 `Tuple` 函数式接口的 lambda 转换为更简单的接口。

这使得我们在使用 `Tuple` 中各成员的时候更加容易，比如：

```
.map(tuple -> {
  String firstName = tuple.getT1();
  String lastName = tuple.getT2();
  String address = tuple.getT3();

  return new Customer(firstName, lastName, address);
});
```

可以用下面的方式代替：

```
.map(TupleUtils.function(Customer::new)); 
```

|      | （因为 `Customer` 的构造方法符合 `Consumer3` 的函数式接口标签） |
| ---- | ------------------------------------------------------------ |
|      |                                                              |



T`reactor.math` 包的 `MathFlux` 提供了一些用于数学计算的操作符，如 `max`、`min`、`sumInt`、`averageDouble`…



`reactor.retry` 包中有一些能够帮助实现 `Flux#repeatWhen` 和 `Flux#retryWhen` 的工具。入口点（entry points）就是 `Repeat` 和 `Retry` 接口的工厂方法。

两个接口都可用作可变的构建器（mutative builder），并且相应的实现（implementing） 都可作为 `Function` 用于对应的操作符。



Reactor-extra 提供了若干专用的调度器： - `ForkJoinPoolScheduler`，位于 `reactor.scheduler.forkjoin` 包； - `SwingScheduler`，位于 `reactor.swing` 包； - `SwtScheduler`，位于 `reactor.swing` 包。