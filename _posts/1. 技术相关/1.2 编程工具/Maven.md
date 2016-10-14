# Maven

## 1. Maven插件

- [assembly](http://www.8qiu.cn/archives/315)
- [maven常用插件介绍](http://www.cnblogs.com/crazy-fox/archive/2012/02/09/2343722.html)


## 2. Maven常用命令

`强制更新依赖`

```
mvn clean install -e -U -Dmaven.test.skip=true
-e详细异常，-U强制更新
```

`查看jar包依赖`

```
mvn dependency:tree
```

`打包上传`

```
mvn clean compile install deploy -Dmaven.test.skip=true
```

`打包部署包`

```
mvn -U clean package -Dtest -DfailIfNoTests=false -Pbuild
```

`打tgz包`
	
```
mvn assembly:assembly -DfinalName=<>  -Dmaven.test.skip=true
```

`只打依赖包到tgz包`

```
mvn assembly:single
```

`打包源码`

```
<build>
    <plugins>
        <!-- deploy时带上source和doc -->
        <plugin>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.2</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

`把依赖包打到一个大jar包中`

```
<build>
    <plugins>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </plugin>
    </plugins>
</build>
```
然后执行`mvn assembly:assembly -Dmaven.test.skip=true`

`不同环境配置参数过滤`

```
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <deploy.env>dev</deploy.env>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault><!--默认启用的是dev环境配置-->
        </activation>
    </profile>

    <profile>
        <id>production</id>
        <properties>
            <deploy.env>production</deploy.env>
        </properties>
    </profile>

    <profile>
        <id>test</id>
        <properties>
            <deploy.env>test</deploy.env>
        </properties>
    </profile>
</profiles>

<build>
    <filters>
        <filter>${deploy.env}.properties</filter>
    </filters>

    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

然后生产环境打包命令`mvn clean compile war:war -Pproduction`

`mvn package –P${profileId}` profileId表示环境参数
