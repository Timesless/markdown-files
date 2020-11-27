### 1 Gradle

#### Wapper

``` tex
gradle/wapper/gradle-wrapper.jar
gradle/wapper/gradle-wrapper.propertites
```

``` bash
# 执行./gradlew 会启动一个轻量级JVM，下载或查询gradlew配置文件中gradle版本
./gradlew

# 创建daemon
./gradle help
```



#### GradleUserHome

``` tex
./gradle

./gradle/caches	保存jar包和其它信息，gradle会定期清理

./gradle/init.gradle	配置全局替换仓库等

./gradle/wrapper	所有下载的wrapper
```



#### Daemon

> 第一次编译，gradle生成daemon JVM，每次执行编译会传递参数到兼容的daemon JVM进行编译，3小时未连接会自动销毁
>
> 如果daemon JVM不兼容，则创建新的daemon



### 2 Grovvy

通用的DSL语言 [grovvy dsl](http://docs.groovy-lang.org/docs/latest/html/documentation/core-domain-specific-languages.html)



#### closure

``` gro
/**
 * 定义闭包
 * 默认最后一行语句为返回值
 * 若不传递参数，那么默认参数名为 it
 */
def closure = {
	it * 2;
	it + 1;
}

```

#### method

``` groovy
/**
 * 方法最后一个参数是闭包，把闭包放在括号外面（和Ruby定义相同）
 * 不产生歧义，可以省略调用时的括号
 */

def method(int i, Closure c) {
    c(i)
}

println method(2, { it * 2 })
println method(2) { it * 2 }

// 例如，gradle配置文件
plugins {
    id("com.diffplug.gradle.spotless").version("3.13.0")
}

// 本质上等于plugins方法调用
plugins(
    { id("com.diffplug.gradle.spotless").version("3.13.0") }
)

```



### 3 Gradle构建

idea打开==build.gradle==，提供gradle home，使用gradle wrapper

> 生命周期
>
> 1. Initialization
>
> 读取项目信息，决定哪些项目参与构建，创建Project instance
>
> 2. Configuration（不构建，只配置）
>
> 对Project实例，运行build.gradle（从头到尾运行，本质是grovvy脚本）
>
> 3. Execution
>
> 执行配置阶段生成的task



#### build.gradle使用第三方jar

``` groovy

// build.gradle
buildScript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'org.apache.commons', name: 'commons-lang3', version: '3.9'
      
    }
}

apply plugin: 'java'
...
```



#### build.gradle

``` groovy
/**
 * task方法调用
 * 创建name = 'hello world'的空白的task
 * 然后使用闭包（函数），去configure该task
 * 对task，调用闭包（函数）
 */
task("hello world", {
    println('configure')
    
    // doLast()调用，只是把闭包添加到任务的执行列表，但不执行它
    doLast({
        println('Executing task')
    })
})

// 对于grovvy dsl的支持，省略括号
task('hello world') {
    println('configure')
    doLast {
        println('Executing')
    }
}
```



#### Project

``` groovy
// api
buildScript()
afterEvaluate() 钩子函数
```



#### Task

``` groovy
// api
dependsOn();
doLast();	// 只添加到任务列表不执行，直到任务被触发才执行
doFirst();	// 只添加到任务列表不执行，直到任务被触发才执行

task('first') {
    // configure时执行
    println "configuring"
    // 只有在task被调用，才会调用
    doLast {
        println "I'm first task"
    }
}

(0..<5).each {
    i -> task('task' + i) {
        
        // 所有偶数任务依赖于第一个任务
        if (i & 1 == 0) { dependsOn('first') }
        
        def captureI = i;
        doLast {
            println "task ${captureI}"
        }
    }
}

// 执行
./gradlew task4

```



#### Plugin

插件：被复用的逻辑

+ ==build.gradle中编写==
+ ==buildSrc/src/main/java==
+ ==提取到仓库，在buildScript中引入==

``` groovy

// build.gradle中编写， 提取到仓库，放到buildSrc/src/main/java
class MyPlugin implements Plugin<Project> {
    
    @override
    void apply(Project project) {
        (0..<5).each {
            i -> project.task('task' + i) {

                // 所有偶数任务依赖于第一个任务
                if (i & 1 == 0) { dependsOn('first') }

                def captureI = i;
                doLast {
                    println "task ${captureI}"
                }
            }
        }
    }
}

apply plugin: MyPlugin

/**
 * desugar
 * 参数是一个map，可以去掉方括号
 * 查找MyPlugin，执行apply()
 */
apply([plugin: MyPlugin])


// 将Myplugin定义为本地脚本 | http url脚本
apply plugin: 'http://...'
```



##### Plugin方式2

``` grovvy
// build.gradle

buildScript {

}
```



##### Plugin方式3

> 通过约定在
>
> buildSrc/src/main/java/MyPlugin.java
>
> 然后在 build.gradle中使用
>
> apply plugin: Myplugin
>
> 



### 构建聚合项目

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
        compile group: 'org.spring.framework', name: 'spring-context', version: '5.0.2.RELEASE'
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

