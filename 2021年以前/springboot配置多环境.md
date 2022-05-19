# springboot配置多环境

application.yml 配置
```
spring:
  profiles:
    active: @profileActive@
```
在pom文件配置
```
<profiles>
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <profileActive>dev</profileActive>
            </properties>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <profileActive>test</profileActive>
            </properties>
        </profile>
        <profile>
            <id>pro</id>
            <properties>
                <profileActive>pro</profileActive>
            </properties>
        </profile>
    </profiles>
```
每个环境使用 ---分开，公共部分放在最后，每个环境使用spring.profiles区分不同环境
```
spring:
  profiles: dev
```
