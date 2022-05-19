# idea jdk 已经配置成了1.8，还提示Diamond types are not supported at language level '1.5'

需要配置pom.xml文件配置java编译环境
```
<build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
参考：

[set-compiler-source-and-target](https://maven.apache.org/plugins/maven-compiler-plugin/examples/set-compiler-source-and-target.html)
