---
title: Spark 项目 jar 包依赖冲突问题总结
date: 2022-09-20 16:15:56
categories:
- 大数据
tags: 
- Spark
- Maven
toc: true
---
## Spark 项目 jar 包加载顺序
当我们编写的 Spark 项目的依赖较多时，提交运行任务时便很容易出现因为包冲突导致的 `java.lang.NoSuchMethodError` 报错。原因是当用户提供 Spark 任务运行时，Spark 需要首先加载自身的依赖库（jars），一般位于 `$SPARK_HOME/jars` 目录下，然后再加载用户提交的 jar 包，当两者存在同样的 jar 但是版本不同时，如果高低版本不能互相兼容，则会报错。

Spark jar 包加载顺序：
1. `SystemClassPath: $SPARK_HOME/jars` 即 Spark 安装时候提供的依赖包
2. `UserClassPath: Spark-submit --jars` 用户提交的依赖包
3. `UserClassPath: Spark-submit app.jar` 用户的 Spark 任务 jar 包
<!--more-->
## spark-submit 提交指定参数解决包冲突
既然 Spark 是顺序加载 jar 包，我们可以尝试通过改变其加载顺序解决依赖冲突。以 `jackson-core` 为例，其在我 Spark 项目中的 Maven 依赖如下：
```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.13.2</version>
</dependency>
```
但是提交任务的集群安装的是 Spark 2.2.0 版本，该版本的 Spark 依赖 `jackson-core` 2.6.5 版本，与我项目中的依赖存在版本冲突，在 2.13.2 版本中，存在 `com.fasterxml.jackson.core.JsonParser.currentName()` 方法，而在 2.6.5 版本中则没有该方法。当采用 Spark 默认加载 jar 顺序的方式，会加载 `jackson-core` 2.6.5 版本，并出现以下报错信息：
```
 java.lang.NoSuchMethodError: com.fasterxml.jackson.core.JsonParser.currentName()Ljava/lang/String
```
此时可以通过指定以下参数，优先加载用户提交的依赖 jar 包：
```bash
spark-submit \
--master yarn \
--jars /data/jar/jackson-core-2.13.2.jar \
--conf "spark.driver.userClassPathFirst=true" \
--conf "spark.executor.userClassPathFirst=true" \
--class com.xxx.SparkApp \
spark_app.jar
```
其中相关参数解释如下：
- `--jars` 用于提交用户的依赖包，若有多个依赖包，之间用逗号分开
- `--conf "spark.driver.userClassPathFirst=true"` 指定 driver 优先加载用户提交的 jar 包
- `--conf "spark.executor.userClassPathFirst=true"` 指定 executor 优先加载用户提交的 jar 包

但是当项目有很多依赖都与 Spark 本身的依赖存在冲突时，这种方式显然就非常不灵活了，需要指定所有冲突的 jar 包，相当麻烦。并且为了保持集群上 Class Path 的纯净，不影响 Spark 本身的运行，我们一般会将开发完成的 Spark 项目打包成 `uber-jar`，即包含所有依赖的 jar，直接提交到 Spark 集群运行，不需要依赖外部 jar，此时就可以利用 `maven-plugin-shade` 制作 `shade jar` 来解决冲突了。
## 利用 maven-plugin-shade 制作 shade jar 解决冲突
### reloaction 重定位 class 文件
使用 shade 提供的重定位功能，可以把指定的类移动到一个全新的包中，实现隔离多个项目依赖同一类的不同版本，以解决版本冲突问题。示例如下：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.4</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <relocations>
                    <!-- 重定位 com.fasterxml.jackson 至 com.shade.jackson -->
                    <relocation>
                        <pattern>com.fasterxml.jackson</pattern>
                        <shadedPattern>com.shade.jackson</shadedPattern>
                    </relocation>
                </relocations>
            </configuration>
        </execution>
    </executions>
</plugin>
```
- `<pattern>`：原始包名
- `<shadedPattern>`：重命名后的包名

在上述示例中，我们把 `com.fasterxml.jackson` 包内的所有子包及 class 文件重定位到了 `com.shade.jackson` 包内。
### 拆分模块
受提交 Spark 项目时发生的依赖冲突问题启发，如果开发的项目本身也存在依赖冲突问题时，显然通过上述的方法就无法解决了，因为没有办法分隔开对同一依赖的调用过程，当类在调用不同版本的依赖时，都会引用重定位后的依赖，此时只存在一个版本，所以依旧会发生依赖冲突问题。

此时就应该将项目拆分为子模块，将依赖不同版本的代码拆分成独立的子模块，各自重定位有冲突的依赖。

如果项目本身就是一个多模块项目，各模块之间有依赖关系，当模块内部存在较多的依赖冲突时，可以为该模块制作一个纯净的子模块，用于重定位所有有冲突的包，然后给该模块引用，如下图所示：
![shade 子模块](/img/shade-model.png)
### 依赖传递
当一个项目的某个模块依赖另一个模块时，如果这两个模块同时依赖同一个依赖包的不同版本时，则打包当前模块时，最终打进 jar 的依赖包是当前模块的依赖版本，而不是被依赖的另一个模块的依赖版本。所以，如果需要打包特定版本的依赖包，则需要在 `pom.xml` 中手动引入指定版本。举例如下：

当前模块的部分依赖：
```xml
<!-- 模块依赖 -->
<dependency>
    <groupId>com.xxx</groupId>
    <artifactId>model-a</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.11</artifactId>
    <version>2.2.0</version>
    <scope>provided</scope>
</dependency>
```
模块 a （model-a）的部分依赖：
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.2</version>
</dependency>
```
spark-core_2.11 的部分依赖：
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.6.5</version>
    <scope>compile</scope>
</dependency>
```
则打包当前模块时，打入最终 jar 包的是 2.6.5 版本的 `jackson-databind` 依赖，此时如果想打包 2.13.2 版本，则需要在 `pom.xml` 中手动引入该版本：
```xml
<!-- 模块依赖 -->
<dependency>
    <groupId>com.xxx</groupId>
    <artifactId>model-a</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.11</artifactId>
    <version>2.2.0</version>
    <scope>provided</scope>
</dependency>

<!-- spark 2.2.0 版本中使用的 jackson 版本是2.6.5 -->
<!--但是依赖的 model a 中依赖的 jackson 是 2.13.2 版本，所以需要手动引入，解决包冲突-->
<!--不然会报错：com.fasterxml.jackson.core.JsonParser.currentName()Ljava/lang/String -->
<!--因为 jackson 2.6.5 版本中不包含此方法-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.2</version>
</dependency>
```
> 参考链接：
https://www.playpi.org/2019112901.html
https://www.lynsite.cn/20210713/1X0CvPcgvpOzkHSF/
