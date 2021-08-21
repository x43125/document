# JNI ERROR

1. 执行Java和编译Java不同
2. 这是因为在使用Maven打包的时候导致某些包的重复引用，以至于打包之后的META-INF的目录下多出了一些*.SF,*.DSA,*.RSA文件所致，我们可以在pom文件里面加入以下配置：

```xml
  <build>
  <plugins>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>2.2</version>
    <configuration>
      <filters>
        <filter>
          <artifact>*:*</artifact>
          <excludes>
            <exclude>META-INF/*.SF</exclude>
            <exclude>META-INF/*.DSA</exclude>
            <exclude>META-INF/*.RSA</exclude>
          </excludes>
        </filter>
      </filters>
    </configuration>
  </plugin>
  </plugins>
  </build>
```

