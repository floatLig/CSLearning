
## 1. ajax

目前来，所有的请求的发送都是`通过浏览器自己直接进行发送`，响应是浏览器在接收到响应信息后自主的将响应数据覆盖当前页面显示。现在，要求在`保留原有页面内容的情况下显示新的响应内容`。

可以用Ajax解决这个问题。

### 1.1 介绍

**介绍：**

AJAX全程“Asynchronous JavaScript and XML”(`异步JavaScript和XML`)，是一种创建`交互式`网页应用的网页开发技术。

Ajax本质上是一个浏览器端的技术。

**为什么学习Ajax？**

Ajax通过`异步模式`，提升了用户体验。`比如网页的搜索栏`，搜索栏显示出来的内容一定是Ajax【根据用户的关键词，在搜索栏中显示结果；而绝不能为了显示结果而重新刷新整个页面，这样子太浪费时间和宽带资源了】

**原理：**

`请求由ajax引擎对象发送`，服务器收到请求后响应数据，浏览器不会直接进行处理，而是流转给发请求的ajax引擎对象。这样我们可以通过操作ajax引擎对象变相的实现在页面中显示新的响应资源。

js的DOM操作中的数据由程序员自己写死声明，变成可以在不刷新页面的情况下，从服务器动态的获取数据并在html上显示。

### 1.2 代码

**使用：**

```java
<!-- 声明js代码域 -->
<script type="text/javascript">
    function getData() {

        //创建ajax对象
        var ajax;
        if (window.XMLHttpRequest) {
            //现在的浏览器都使用这个，比如火狐、chrome
            ajax = new XMLHttpRequest();
        } else if (window.ActiveXObject) {
            //以前的浏览器是使用这个，比如ie
            ajax = new ActiveXObject("Msxm12.XMLHTTP");
        }

        //覆写方法，状态改变时会自动调用该函数
        ajax.onreadystatechange = function () {
        	//只有在状态为4的时候（收到response）才执行
			var responseResult;
			if (ajax.readyState === 4) {
				console.log("readyState为4");
                console.log("status:" + ajax.status);
                //获取HTML元素对象
                var showDiv = document.getElementsByName("showDiv");

				if (ajax.status == 200) {
					console.log("state为200");
					//获取服务器响应内容
					responseResult = ajax.responseText;
                    eval("var u=" + responseResult);
                    //json格式
					alert(responseResult);
					alert(u.uname);
					//写在HTML上
					showDiv.innerHTML = u.uname;
				} else if (ajax.status == 500) {
					console.log("state为500");
					//写在HTML上
					showDiv.innerHTML = "服务器繁忙";
				} else if (ajax.status == 404) {
					console.log("state为404");
					//写在HTML上
					showDiv.innerHTML = "服务器错误";
				}
			}else{
				showDiv = document.getElementsByName("showDiv");
				//状态为1、2、3的时候，说明响应未到，这个时候显示“加载ing的动态图”
				showDiv.innerHTML ="<img src='img/加载动态图.gif' width='200px' height='100px'>";
			}
        }
        //ajax发送请求
        // ajax.open("get", "AjaxServlet");
        // ajax.send(null);
        //post请求
		ajax.open("post","AjaxServlet");
		ajax.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
		ajax.send("name=张三&pwd=123");
    }
</script>
```

**服务器端代码：**

```java
@Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("utf-8");
        resp.setContentType("context/html;charset=utf-8");

        try {
            //模拟后台正在处理数据，让它先睡一会儿
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //从数据库获取数据，这里为了简便，自己创建一个对象
        User user = new User();
        user.setNum(10);
        user.setUname("userTest");
        user.setPasswd("123");

        //将对象用json的格式发到浏览器
        //传统格式这么写
        //resp.getWriter().write("{num:"+ user.getNum() + ",uname:" +
        //       user.getUname() +",pwd:"+ user.getPasswd()+"}");
        //使用封装的gson.jar包，调用函数写法
        System.out.println(new Gson().toJson(user));
        resp.getWriter().write(new Gson().toJson(user));
    }
```

**Ajax的readyState值的含义：**

| readyState值 |                             含义                             |
| :----------: | :----------------------------------------------------------: |
|      0       | 表示XMLHttpRequest已建立，但还未初始化，这时尚未调用open方法 |
|      1       |   表示open方法已经调用，但未调用send方法（已创建，未发送）   |
|      2       |              表示send方法已经调用，其他数据未知              |
|      3       |              表示请求已经成功发送，正在接收数据              |
|      4       |                       表示数据成功接收                       |

**HTTP状态的含义：**

| HTTP状态码 |      含义      |
| :--------: | :------------: |
|    200     |    请求成功    |
|    404     | 请求资源未找到 |
|    500     | 内部服务器错误 |

**json:**

json其实是数据的一种格式。【和XML性质差不多】

json格式：

```javaScript
var 对象名 = {
    属性值 : 属性值,
    属性值 : 属性值,
    属性值 : 属性值,
}
```

数据按照json格式拼接好的字符串，方便使用eval方法将接收的字符串数据直接转化成js对象。

## 2. filter

Servlet 的作用是针对浏览器发起的请求，进行请求的处理。通过Servlet 技术我们可以灵活的进行请求的处理，但是我们不但要对请求记性处理，我们还需`对服务器的资源进行统一的管理`，比如请求编码格式的统一设置，资源的统一分配,拦截关键字等等，这个时候该怎么办呢？

可以使用`过滤器`。

### 2.1 介绍

**作用：**

1. 对服务器资源进行管理
2. 保护servlet

**使用：**

1. 统一管理字符编码
2. session管理
3. 资源管理（统一水印，和谐词汇等等）

**原理：**

浏览器发起请求到服务器，服务器接收请求后，根据URI信息在web.xml中找到对应的过滤器执行doFilter方法，该方法对此次请求进行处理后如果符合条件则放行，放行后如果还有符合要求的过滤则继续进行过滤，找到执行对应的servlet进行请求处理。servlet对请求处理完毕后，也就service方法结束了。还需继续返回相应的doFilter方法继续执行。

### 2.2 代码

**xml配置：**

```java
    <filter>
        <filter-name>MyFilter</filter-name>
        <filter-class>filter.MyFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>MyFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

**url-pattern解析：**

/* 代表对所有的请求都进行过滤判断；  

除了"/*"之外，还有以下几种过滤的方式：

1. /*.do：以".do"结尾的所有请求都会进行过滤，一般用来进行模块拦截处理。
2. /my.do：发起my.do的请求会被过滤，是针对某个请求的拦截。

**filter Java类：**

```java
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("filter被创建了");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("filter被执行");
        //设置编码格式
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setContentType("text/html;charset=utf-8");
        //判断session
        HttpSession httpSession = ((HttpServletRequest)servletRequest).getSession();
        if (httpSession.getAttribute("user") == null) {
            ((HttpServletResponse)servletResponse).sendRedirect("index.jsp");
        }else {
            //放行
            filterChain.doFilter(servletRequest,servletResponse);
        }
    }

    @Override
    public void destroy() {
        System.out.println("filter被销毁");
    }
}
```

**覆写的接口：**

init方法：服务器启动时执行。资源初始化

doFilter方法：拦截请求的方法，在此方法中可以对资源进行管理。

注意：需要手动放行。放行的代码为：`chain.doFilter(request,response)`;

destory方法：服务器关闭执行。

## 3. listener

**概念：**

Servlet 监听器是Servlet 规范中定义的一种特殊类，用于监听ServletContext、HttpSession 和ServletRequest 等域`对象的创建与销毁事件【Initialized、Destroyed】`，以及监`听这些域对象中属性发生修改的事件【Added、Removed、Replaced】`。

**监听类代码：**

```java
public class MyListener implements ServletRequestListener, ServletRequestAttributeListener, HttpSessionListener, HttpSessionAttributeListener, ServletContextListener, ServletContextAttributeListener {

    /**
     * ServletContext
     */

    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {

    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {

    }

    @Override
    public void attributeAdded(ServletContextAttributeEvent servletContextAttributeEvent) {

    }

    @Override
    public void attributeRemoved(ServletContextAttributeEvent servletContextAttributeEvent) {

    }

    @Override
    public void attributeReplaced(ServletContextAttributeEvent servletContextAttributeEvent) {

    }

    /**
     * request
     */

    @Override
    public void requestDestroyed(ServletRequestEvent servletRequestEvent) {

    }

    @Override
    public void requestInitialized(ServletRequestEvent servletRequestEvent) {

    }

    @Override
    public void attributeAdded(ServletRequestAttributeEvent servletRequestAttributeEvent) {

    }

    @Override
    public void attributeRemoved(ServletRequestAttributeEvent servletRequestAttributeEvent) {

    }

    @Override
    public void attributeReplaced(ServletRequestAttributeEvent servletRequestAttributeEvent) {

    }

    /**
     * session
     */
    @Override
    public void sessionCreated(HttpSessionEvent httpSessionEvent) {

    }

    @Override
    public void sessionDestroyed(HttpSessionEvent httpSessionEvent) {

    }

    @Override
    public void attributeRemoved(HttpSessionBindingEvent httpSessionBindingEvent) {

    }

    @Override
    public void attributeAdded(HttpSessionBindingEvent httpSessionBindingEvent) {

    }

    @Override
    public void attributeReplaced(HttpSessionBindingEvent httpSessionBindingEvent) {

    }
}
```

**xml配置：**

```xml
    <listener>
        <listener-class>listenter.MyListener</listener-class>
    </listener>
```
