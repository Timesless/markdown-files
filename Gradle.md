## Gradle深入理解



### 1. Gradle基础



#### 1.1 Distribution



#### 1.2 Wrapper

``` shell
./gradlew help

./gradlew build.gradle

# 1. 如果没有对应版本gradle，那么首先下载gradlew版本
# 2. 启动client jvm 查找daemon
# 3. 如果没有daemon运行，那么会先启动daemon jvm
# 4. client发送参数，daemon执行构建，返回日志 
```



#### 1.3 GradleUserHome

``` shell
# 默认路径
~/.gradle/

# 保存jar包和其它信息，gradle会定期清理
~/.gradle/caches

# 可以配置全局替换仓库等
~/.gradle/init.d/
~/.gradle/init.gradle

# 所有下载的wrapper
~/.gradle/wrapper
```



#### 1.4 Daemon

> gradle 首先启动一个 client jvm，与后台daemon jvm通讯
>
> client只负责转发请求，传递参数，接受日志
>
> daemon默认3个小时后退出
>
> --no-daemon 不启用gradle daemon



### 2. Groovy基础

`https://github.com/Timesless/groovy-all`



#### 2.1 动态调用与MOP



#### 2.2 闭包

``` groovy
// 闭包可以引用外部类的变量，这与lambda不同
def str = "hello"
def c4 = {
  println("$str $it")
/*
    闭包会修改引用的外部变量
    我认为是单线程的缘故，C++ lambda引用外部变量修改的是作为匿名类实例的字段，但不能改外部变量本身，Java则是final捕获的
    Golang闭包修改的也是引用的外部变量作为匿名函数实例的字段的值，外部变量本身的值是不能修改的
    js 和 groovy的闭包则可以修改外部变量本身的值，这应该是单线程的原因
    闭包大约在2003年
*/
  str = "zzz"
}
c4.call("param")
println("==== outer str = " + str)
```



### 3 Gradle构建

Gradle生命周期

> 我们通常只关心2个阶段
>
> + configure
> + execution

1. Initialization

>  读取项目信息（settings.gradle），决定哪些项目参与构建，为每个需要构建的项目创建Project instance
>
> Project Instance是gradle构建的核心

2. Configuration（不执行构建，只配置）

> 对Project实例，运行build.gradle（从头到尾解释执行，本质是grovvy脚本）

3. Execution

> 执行配置阶段生成的task

``` groovy
// 定义一个task
task('hello gradle', {
  println('configure')
  
  doLast({
    println('executing task')
  })
})

// 执行结果
configure
executing task

/*
 * 解释：
 *	2. configuration阶段『解释执行build.gradle文件（groovy脚本）』
 *	3. excution阶段 
 *
 *  创建了一个hello gradle的任务，使用闭包「闭包是函数实例」去configure这个task
 */
```



#### 3.1 Gradle核心模型

``` groovy
task("hello gradle") {
    println("configureing ")
    // Configure时，只是将该闭包添加到任务的动作列表的最前面，并不实际执行
    doFirst {
        println('Executing first')
    }
    doLast {
        println('Executing last')
    }
}
```



#### 3.2 Project

``` groovy
project.parent.childProjects

```



#### 3.3 Task

> help任务是所有gradle项目存在的一个任务
>
> task是gradle最小单元，maven中最小单元是每一个lifecycle

``` java
// 定义4个任务
// gradle task2
// gradlew task2
(0..<5).each { i ->
    task('task' + i) {
        // 偶数任务依赖 hello gradle 任务
        if ((i & 1) == 0) {
            dependsOn('hello gradle')
        }
        def captureI = i
        doLast {
            println("Excuting task" + captureI)
        }
    }
}
```



#### 3.4 Lifecycle与Hook

``` groovy
/*
    钩子函数
    任何无主的函数，gradle都在Project api中查找

    解释执行完build.gradle触发afterEvaluate hook
 */
afterEvaluate {
    println('----- after evaluate -----')
}
```



### 4. 插件编写



#### 4.1 构建逻辑的复用



#### 4.2 简单插件

> build.gradle

``` groovy
// 使用已定义好的插件
apply plugin: 'java'
apply plugin: 'groovy'
/*
 * apply(Map<String, ?>)
 * apply([plugin: 'java'])
 * groovy 语法糖可省略map的[] -> apply(plugin: 'java')
 * 不产生歧义的情况下可省略() -> apply plugin: 'java'
 */
```



#### 4.3 script插件

> build.gradle

``` groovy
/*
    gradle插件编写

    1. build.gradle class MyPlugin implements Plugin<Project> {}
    2. user.dir/buildSrc/src/main/java/MyPlugin2 implements <Project> {}

    apply plugin: MyPlugin
    apply plugin: MyPlugin2
 */
class MyPlugin implements Plugin<Project> {
    @Override
    void apply(Project project) {
        (5..<10).each { i ->
            project.task('task' + i) {
                println('....plugin task' + i +' configure....')
                // 闭包的延迟执行需捕获外部变量
                def captureI = i
                doLast {
                    println("Excuting task" + captureI)
                }
            }
        }
    }
}

/*
    语法糖 apply plugin: MyPlugin
    desugar，本质等价于
    apply([plugin: MyPlugin])

    当参数是map时，可以省略[]
    当不产生歧义时，方法调用的()可以省略
 */
apply plugin: MyPlugin
apply plugin: MyPlugin2
```



#### 4.4 buildSrc插件

> user.dir/buildSrc/src/main/java/MyPlugin.java

``` java
import org.gradle.api.Plugin;
import org.gradle.api.Project;
/**
 * @author yangzl
 * @date 2021/2/8
 * @desc
 *
 * 		gradle 插件抽取到 Java类
 * 		该类不需要声明packeage
 *
 * 	gradle整个过程是如何实现的呢？
 * 	该过程类似于maven的install 本地仓库
 * 		1. 通过约定在buildSrc
 * 		2. 在外层build.gradle运行之前先编译buildSrc
 * 		3. 将buildSrc打包的结果libs下，放入buildScript 下 dependencies { classpath 中 }，见build.gradle
 *
 */
public class MyPlugin2 implements Plugin<Project> {

	@Override
	public void apply(Project project) {
		for (int i = 10; i < 15; i++) {
			System.out.println("**** java plugin" + i + " configure ****");
			project.task("task" + i);
		}
	}
}
```



#### 4.5 发布的插件

> binary插件

将项目独立，发布到仓库，在buildScript {} 中引入

==apply plugin: 'java'是将插件引入到buildScript{}中，供configure阶段使用==



### 5. 实际插件分析







### 6. 构建聚合项目

``` groovy
// parent -> build.gradle
allProjects {
  group 'com.yangzl'
  version: '1.0-SNAPSHOT'

  apply plugin: 'java'

  sourceCompatibility = 1.8

  repositories {
    mavenLocal()
    mavenCentral()
  }

  dependencies {
    compile 'org.spring.framework:spring-context:5.0.2.RELEASE'

    testCompile group: 'junit', name: 'junit', version: '4.12'
  }
}

// service -> build.gradle
dependencies {
  complie project(":dao")
}

// dao -> build.gradle

// web -> build.gradle
apply plugin: 'war'

```

