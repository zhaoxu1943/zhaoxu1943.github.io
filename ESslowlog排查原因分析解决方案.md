# 外部故障单根本原因分析及解决方案:ES-slowlog的定位问题

## 现象

由于现行产品线对ES使用的不规范,且无认证机制,导致现在ES难以区分slowlog到底是哪个应用产生,进而找不到实际负责人,以下给出了解决方案









## 解决方案

ESslowlog记录客户端

现在部门里应该都使用的统一编写的ES建立连接的方法

在其中设置请求头处添加`new BasicHeader("X-Opaque-ID", "应用标识符")`

```
 /**
     * <b>方法说明：</b>
     * <ul>
     *  设置公共的请求头
     * </ul>
     * @author jiaokz
     * @CreateDate 2018/11/29 22:23
     * @param clientBuilder
     * @return void
     */
    private void setDefaultHeaders(RestClientBuilder clientBuilder){
        // 设置请求头，每个请求都会带上这个请求头
        Header[] defaultHeaders = {new BasicHeader("header", "value")};
        clientBuilder.setDefaultHeaders(defaultHeaders);
    }

```

修改后(此处应用标识符应使用环境变量,举例中使用了长串的占位符)

```
/**
 * <b>方法说明：</b>
 * <ul>
 *  设置公共的请求头
 * </ul>
 * @author jiaokz
 * @CreateDate 2018/11/29 22:23
 * @param clientBuilder
 * @return void
 */
private void setDefaultHeaders(RestClientBuilder clientBuilder){
    // 设置请求头，每个请求都会带上这个请求头
    Header[] defaultHeaders = {new BasicHeader("header", "value")
            ,new BasicHeader("X-Opaque-ID", "assetttttttttttttt")};
    clientBuilder.setDefaultHeaders(defaultHeaders);
}
```

效果:

![img](C:\Users\zhaoxu\OneDrive\工作\评级\t63\原始文档\images\332-1.png)