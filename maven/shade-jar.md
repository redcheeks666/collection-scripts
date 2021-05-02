#### shade 打包方式

避免包命名冲突

pom.xml中指定通过shade打包的相关依赖

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.1.1</version>
                <configuration>
                    <!-- put your configurations here -->
                    <relocations>
                        <!-- 此处通过shaded的打包方式,将第三方依赖驱动org.postgresql
                                 重命名为org.shaded.postgresql -->
                        <relocation>
                            <pattern>org.postgresql</pattern>
                            <shadedPattern>org.shaded.postgresql</shadedPattern>
                        </relocation>
                    </relocations>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

#### shade 打包命令

```shell
mvn clean package org.apache.maven.plugins:maven-shade-plugin:3.1.1:shade -DskipTests
```

