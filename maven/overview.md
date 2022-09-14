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





