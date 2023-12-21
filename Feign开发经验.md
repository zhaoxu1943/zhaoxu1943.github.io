

## 依赖

```xml
<!-- feign client -->  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-openfeign</artifactId>  
</dependency>  
  
<dependency>  
    <groupId>io.github.openfeign</groupId>  
    <artifactId>feign-httpclient</artifactId>  
</dependency>

```

openfeign请使用feign-httpclient作为http客户端,打印日志更全面

HOST等都会打印
[Open: Pasted image 20231110160700.png](img/60f4908018d60612bc49f7b10fdc9e92_MD5.jpeg)
![](img/60f4908018d60612bc49f7b10fdc9e92_MD5.jpeg)


## 配置

配置日志打印,配置超时时间,feign日志请打满

```java

@Configuration  
public class FeignConfig {  
  
    @Bean  
    Logger.Level feignLoggerLevel() {  
        return Logger.Level.FULL;  
    }  
  
  
    @Bean  
    Request.Options feignRequestOptions() {  
        return new Request.Options(3, TimeUnit.SECONDS, 30, TimeUnit.SECONDS, true);  
    }  
  
}

```


## FeignClient类中

1. 开发使用URL方式暂且不表
2. 当使用服务名方式时
```java
@FeignClient(value = "tib")
```

注意这个tib是不包含在request url 中的


```java
//查询IOC情报库中的数据  
@GetMapping("/tib/openApi/cti/type/distribution")  
Map<String, Object> getStatistics();

```

下面的地址一定要写满~

