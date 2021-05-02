#### 加载本地jar包到Local repository 命令

```shell
mvn install:install-file 
-Dfile=jar包本地路径(D:\xxx\xxx.jar) 
-DgroupId=jar包对应的groupId(com.aliyun) 
-DartifactId=jar包对应的artifactId(aliyun-java-sdk-core) 
-Dversion=jar包的版本号(4.5.3) -Dpackaging=jar
```
