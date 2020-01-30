
## 1. web.xml和server.xml

### 1.1 web.xml

**作用：**

1. 存储项目相关的配置信息，保护Servlet。
2. 解耦一些数据对程序的依赖。

**使用位置：**

1. 私有web.xml: 每个Web 项目中都有一个属于这个项目的web.xml
2. 公共的web.xml: Tomcat 服务器中(在服务器目录conf 目录中)

**区别：**

1. Web 项目下的web.xml 文件为局部配置，针对本项目的位置。
2. Tomcat 下的web.xml 文件为全局配置，配置公共信息。

**内容（核心组件）：**

1. 全局上下文配置（全局参数配置）
2. servlet配置
3. 过滤器配置
4. 监听器配置

**服务器启动时，内容（核心组件）加载顺序：**

1. ServletContext
2. context-param
3. listener
4. filter
5. servlet

web容器会按照上面这个顺序加载。

但是这些元素可以配置在web.xml的任意位置，即书写位置随机，加载顺序固定。【在服务器启动时加载】

### 1.2 server.xml

浏览器发起请求后，服务器根据请求在webapps 目下调用对应的Servlet 进行请求处理。那么为什么是webapps 目录难道不能是其他的目录吗？

可以的。可以在server.xml里面配置。

server.xml文件核心组件如下：

```xml
<Server>
    <Service>

        <Connector />
        <Connector />

        <Engine>
            <Host>
                <Context />
            </Host>
        </Engine>

    </Service>
</Server>
```

其中，Connector用于确定tomcat使用的端口号，Context可以配置热部署，Host配置服务器启动时localhost找哪个文件夹(webapps)
