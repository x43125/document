# maven overview

## 一、maven项目结构

```ascii
a-maven-project
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
│   └── test
│       ├── java
│       └── resources
└── target
```

## 二、maven pom.xml示例

```xml
<project ...>
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.itranswarp.learnjava</groupId>
	<artifactId>hello</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>
	<properties>
        ...
	</properties>
	<dependencies>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
	</dependencies>
</project>
```

其中：

- groupId: 类似Java包名，通常是公司或组织名称
- artifactId: 类似Java类名，通常是项目名
- version: 版本号

这三个字段，在我们自己的项目里需要来指定，以便其他项目导入我们的项目

在引入别的依赖时也需要通过指定这三个字段来唯一确定一个依赖。

- scope: 这个字段是用来指示引入的依赖在何时起到作用，包括四种：
    - compile: 编译时需要（默认）
    - test: 编译Test是哦需要
    - runtime: 编译时不需要，但运行时需要
    - provided: 编译时需要用到，但运行时由JDK或某个服务器提供了

## 三、生命周期

生命周期全过程的阶段：其中加粗的为几个比较重要的阶段

validate -> initialize -> generate-sources -> process-sources -> generate-resources -> 

process-resources -> **compile** -> process-classes -> generate-test-sources -> process-test-sources -> 

generate-test-resources -> process-test-resources -> test-compile -> process-test-classes -> **test** -> 

prepare-package -> **package**  -> pre-integration-test -> integration-test -> post-integration-test -> 

verify -> **install** -> **deploy**

## 四、命令

### 4.1 命令

- mvn package: 打包，到package阶段
- mvn compile: 编译，到compile阶段
- mvn clena: 清除已打包的文件
- mvn test: 测试，执行到test阶段

### 4.2 经常用到的

- mvn clean
- mvn clean compile
- mvn clean package
- mvn clean test

## 五、插件

在pom.xml中声明插件

```xml
<project>
    ...
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-shade-plugin</artifactId>
                <version>3.2.1</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>shade</goal>
						</goals>
						<configuration>
                            ...
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
```

常用插件：

- maven-shade-plugin：打包所有依赖包并生成可执行jar；
- cobertura-maven-plugin：生成单元测试覆盖率报告；
- findbugs-maven-plugin：对Java源码进行静态分析以找出潜在问题。

## 六、模块管理

拆分项目为多个模块，然后通过maven进行依赖

父模块：项目和创建普通Java项目相同，区别是：将maven的打包方式改为 **pom**

```xml
<packaging>pom</packaging>
```

然后在子模块中添加父模块的坐标

```xml
<parent>
    <!-- 父工程坐标 -->
    <groupId>...</groupId>
    <artifactId>Parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <!-- 指定从当前子工程的pom.xml文件出发，查找父工程的pom.xml的路径 -->
    <relativePath>../Parent/pom.xml</relativePath>
</parent>
```

> 如果子工程的 groupId、version 和父工程一样则可以删除。

将父模块的依赖内容 使用 `dependencyManagement`来括起来

如下：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

然后就可以将子模块的相应的依赖，删除版本和范围

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
</dependencies>
```





















































































