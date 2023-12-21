## 现象

高版本Spring带的Zk版本较高,无法注册到公司的低版本zk上

## 解决

- search spring doc
- search version

[Open: Pasted image 20231123164850.png](img/0918f28c2d8e79919c1f194c1a0740f3_MD5.jpeg)
![](img/0918f28c2d8e79919c1f194c1a0740f3_MD5.jpeg)


可以看见, 高版本的spring注册到低版本zk,需要通过maven依赖版本来解决这个问题

[Open: Pasted image 20231123164956.png](img/001f57ae02f8216f26986c460e30336c_MD5.jpeg)
![](img/001f57ae02f8216f26986c460e30336c_MD5.jpeg)



## 实例


```xml


<!--curator-->  
<curator.version>4.2.0</curator.version>


<!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->  
<dependency>  
    <groupId>org.apache.curator</groupId>  
    <artifactId>curator-framework</artifactId>  
    <version>${curator.version}</version>  
</dependency>  
  
<dependency>  
    <groupId>org.apache.curator</groupId>  
    <artifactId>curator-recipes</artifactId>  
    <version>${curator.version}</version>  
    <scope>compile</scope>  
    <exclusions>        <exclusion>  
            <artifactId>log4j</artifactId>  
            <groupId>log4j</groupId>  
        </exclusion>  
        <exclusion>            <artifactId>netty</artifactId>  
            <groupId>io.netty</groupId>  
        </exclusion>  
        <exclusion>            <groupId>org.apache.zookeeper</groupId>  
            <artifactId>zookeeper</artifactId>  
        </exclusion>  
    </exclusions>  
    <optional>true</optional>  
</dependency>


```