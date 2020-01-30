## 1 cookie

使用重定向，但是又要求不同请求数据共享，怎么办呢？HTTP 协议是没有记忆功能的，一次请求结束后，相关数据会
被销毁。如果第二次的请求需要使用相同的请求数据怎么办呢？难道是让用户再次请求书写吗?

可以使用cookie。

Cookie 技术其实是`浏览器端的数据存储技术`，解决了`不同请求需要使用相同的请求数据的问题`。我们把请求需要共享的请求数据，存储在浏览器端，避免用户进行重复的书写请求数据。但是哪些数据需要使用Cookie 技术存储起来是一个主观问题，`需要在后台进行响应的时候来告诉浏览器`，有些数据其他请求还会使用，需要存储起来。

`cookie是浏览器端的存储数据【当然，cookie是在服务器后端创建的，并resp到浏览器的】。`

特点：

1. 浏览器端的数据存储技术  
1. 适合少量数据  
1. 键值对  
1. 不安全  

```java
使用：
	//创建cookie对象
	Cookie c = new Cookie(String name,String value);
	//【可选】设置有效期
	c.setMaxAge(int seconds);
	//【可选】设置有效路径
	c.setPath(String uri);
	//响应Cookie信息给客户端
	resp.addCookie(c);

	cookie的获取
	//获取cookie信息数据
	Cookie[] cookies = req.getCookies（）；
	if(cookies != null){
		for(cookie : cookies){
			String name = cookie.getName();
			String value = cookie.getValue（）；
			System.out.println(name + ":" + value);
		}
	}

注意：
	一个cookie对象存储一条数据。要存储多条数据，可以创建多个Cookie
特点：
	浏览器端的数据存储技术
	存储的数据声明在服务器端。
	临时存储：存储在浏览器的运行内存中，浏览器关闭即失效
	定时存储：设置了cookie的有效期，存储在客户端的硬盘中，在有效期内符合路径要求的都会附带该信息
	默认cookie信息存储之后，每次请求都会附带，除非设置有效路径
```

```java
    //保存cookie到浏览器端
    Cookie temporaryCookie = new Cookie("temporaryCookie","temporaryCookie");
    resp.addCookie(temporaryCookie);

    Cookie fixedTimeCookie = new Cookie("fixedTimeCookie","fixedTimeCookie");
    fixedTimeCookie.setMaxAge(3*24*3600);
    resp.addCookie(fixedTimeCookie);

    Cookie pathCookie = new Cookie("pathCookie","pathCookie");
    pathCookie.setPath("/path");
    resp.addCookie(pathCookie);

    //服务器后台获取cookie
    Cookie[] cookies = req.getCookies();
    if (cookies != null) {
        for (Cookie cookie : cookies) {
            System.out.println(cookie.getName() + ":" + cookie.getValue());
        }
    }
```

**代码举例：**

```java
@Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=utf-8");
        //只能对实体主体的数据进行编码格式转化，不能对URL的编码进行格式转换。因为post不会乱码,get需要额外设置其他东西。
        req.setCharacterEncoding("utf-8");

        //获取jsp的uname和pwd的值
        String uname = req.getParameter("uname");
        String pwd = req.getParameter("pwd");
        System.out.println(uname + ":" + pwd);

        //保存cookie到浏览器端
        Cookie temporaryCookie = new Cookie("temporaryCookie","temporaryCookie");
        resp.addCookie(temporaryCookie);

        Cookie fixedTimeCookie = new Cookie("fixedTimeCookie","fixedTimeCookie");
        fixedTimeCookie.setMaxAge(3*24*3600);
        resp.addCookie(fixedTimeCookie);

        Cookie pathCookie = new Cookie("pathCookie","pathCookie");
        pathCookie.setPath("/path");
        resp.addCookie(pathCookie);

        //服务器后台获取cookie
        Cookie[] cookies = req.getCookies();
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                System.out.println(cookie.getName() + ":" + cookie.getValue());
            }
        }


        //验证是否登录成功
        LoginService ls = new LoginServiceImpl();
        User user = ls.checkLogin(uname, pwd);
        if (user != null) {
            System.out.println(user + "\n");
        }

        //根据验证的结果，判断页面怎么跳转
        if (user != null) {
            //重定向
            resp.sendRedirect("main");
        } else {
            //请求转发
            req.getRequestDispatcher("index.jsp").forward(req,resp);
        }
    }
```

**补充：**

[Chrome浏览器修改cookie的值](https://segmentfault.com/q/1010000000678627)

## 2. session

请求转发可以解决`一次请求内`的不同Servlet 的数据共享问题，那么一个用户的`不同请求`的处理需要`使用相同的数据`怎么办呢?

可以使用session。

session是`存储在服务器`的数据存储技术，一般会存放“`用户身份的信息`”，便于不同的请求是为这个用户服务的。比如查成绩，知道查的成绩是哪个用户的。

**原理：**

用户使用浏览器第一次向服务器发送请求，服务器在接受到请求后，调用对应的Servlet 进行处理。在处理过程中会给用户创建一个session 对象，用来存储用户请求处理相关的公共数据，`并将此session 对象的JSESSIONID 以Cookie 的形式存储在浏览器中(临时存储，浏览器关闭即失效)`。用户在发起第二次请求及后续请求时，请求信息中会`附带JSESSIONID`，服务器在接收到请求后，调用对应的Servlet 进行请求处理，`同时根据JSESSIONID 返回其对应的session 对象【如果没有JSESSIONID，就创建一个新的session】`。

session的原理是在服务器创建一个session对象，代表一个用户。JSESSIONID是该用户/session的唯一标识符。

**特点：**

1. session技术依赖cookie
2. session是服务器端的数据存储技术
3. 由服务器创建
4. 每个用户都拥有一个session
5. 默认存储时间30分钟

**作用：**

解决了一个用户的不同请求的数据共享问题

**使用：**

```java
//创建session对象 | 获取session对象
HttpSession httpSession = req.getSession();
//设置存储时间，默认为30分钟
httpSession.setMaxInactionInterval(int seconds);
//存储数据到session 一般会存储用户的个人信息到session中
httpSession.setAttribute(String key, Object obj);
//获取session对象的数据  如果获取的数据不存在就会返回null
Obj obj = httpSession.getAttribute(String key);
```

**注意：**

只要不关闭浏览器，并且session不失效的情况下，`同一个用户【浏览器的JSESSIONID】`的任意请求在项目的任意Servlet 中获取到的都是`同一个session对象`。

**作用域：**

一次会话，关闭浏览器后失效

**拓展：**

- **session默认的时间是30分钟。那怎么修改这个默认值呢？**

在tomcat目录下--->conf--->web.xml

修改session-timeout的时间就可以修改默认session的存活时间。

```xml
  <!-- ==================== Default Session Configuration ================= -->
  <!-- You can set the default session timeout (in minutes) for all newly   -->
  <!-- created sessions by modifying the value below.                       -->

    <session-config>
        <session-timeout>30</session-timeout>
    </session-config>
```

设置这个存活时间是：为了销毁闲置过长的对象，释放内存资源

而且这里值得注意的是，设置时间30分钟，代表这个session如果30分钟`没有被使用`，那么就会销毁这个session。但是，如果这个session有被使用过，那么30分钟会重新计时。举个例子：

一个session在第29分钟的时候，有一个请求需要获取session的数据，那么这个session就还可以再存活30分钟（重新计时30分钟）

- **getSession()这个函数做了什么事呢？**

getSession()这个函数其实为我们做了很多的事。如果用户第一次登录，此时后台服务器没有session对象，那么

1. getSession()就会创建一个session对象，并有一个唯一标识符JSESSIONID
2. 创建JSESSIONID的cookie，并resp到浏览器

如果后台已经有了session，那么getSession() :

1. 根据浏览器的JSESSIONID找到后台的session对象，并且返回这个session对象

- **session失效的情况：**

1. 服务器session失效，时间到
2. 浏览器失效，关闭浏览器

**代码举例：**

利用session显示欢迎界面

```java
public class MainServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=utf-8");

        String userName = "";
        HttpSession userSession = req.getSession();
        if (userSession.getAttribute("userSession") != null) {
            User user = (User) userSession.getAttribute("userSession");
            userName = user.getUname();
        }

        resp.getWriter().write("<H1>欢迎" + userName + "来到后台管理系统</H1>");
    }
}
```

但是这么写要注意一个问题，你要确保session存在！

用户登录有两种途径，一种是账号密码登录，这种肯定会有session，因为即使没有session也创建一个session出来。

另一种是免密登录。因为关闭浏览器的cookie JSESSIONID会消失，所以如果是免密登录，要确保session必须存在,免密登录的代码

```java
//通过判断cookie uid，进行免密登录，看看能不能跳转到主页面
        if (uid != null ) {
            int uidInt = Integer.parseInt(uid);
            LoginService loginService = new LoginServiceImpl();
            User user = loginService.checkUidLogin(uidInt);
            if (user != null) {
                //如果浏览器关闭了，执行三天免登录，这一步确保一定会有session
                req.getSession().setAttribute("userSession",user);

                resp.sendRedirect("main");
                System.out.println("\ncookie登陆成功！");
            }else {
                resp.sendRedirect("index.jsp");
            }
        }else {
            resp.sendRedirect("index.jsp");
        }
```

另一方面，要警惕服务器后台session的过期，如果session过期了，一定要让用户重新登录，否则，服务器后台就不是用户是谁了【联想我们的教务系统】。

## 3. ServletContext

Request 解决了一次请求内的数据共享问题，session 解决了用户不同请求的数据共享问题，那么不同的用户的数据共享该怎么办呢？

使用ServletContext 对象.

### 3.1 原理介绍

**原理：**

ServletContext 对象`由服务器进行创建`，`一个项目只有一个对象`。不管在项目的任意位置进行`获取得到的都是同一个对象`，那么不同用户发起的请求获取到的也就是同一个对象了，`该对象由用户共同拥有`。

**特点：**

1. 服务器进行创建
2. 用户共享
3. 一个项目只有一个

**生命周期：**

服务器启动到服务器关闭

**作用域：**

项目内

### 3.2 使用-代码

```java
    //多种方式获取servletContext
    ServletContext servletContext = this.getServletContext();
    //ServletContext servletContext1 = req.getSession().getServletContext();
    //ServletContext servletContext2 = this.getServletConfig().getServletContext();

    //数据存储
    String str = "servletContext String";
    servletContext.setAttribute("servletContext", str);

    //数据获取
    req.getServletContext().getAttribute("servletContext");

    //获取xml文件的全局信息
    Enumeration<String> initParameterNames = servletContext.getInitParameterNames();
    while (initParameterNames.hasMoreElements()) {
        String param = initParameterNames.nextElement();
        System.out.println(param + ":" + servletContext.getInitParameter(param));
    }

    //获取绝对路径
    System.out.println(servletContext.getRealPath("/iWEB-INF/web.xml"));
    //获取的路径为：D:\Workspace\sxt\04ServletAndTomcat\02Login\out\artifacts\02Login_war_exploded\iWEB-INF\web.xml
    //由此可以看到，idea的路径不是在tomcat下，而是在out目录下

    //获取流对象
    InputStream resourceAsStream = servletContext.getResourceAsStream("/index.jsp");
```

使用ServletContext实现网页访问次数统计

```java
    //判断该网站是不是刚创建，如果刚创建，初始化servletContext的visits的值
    if (this.getServletContext().getAttribute("visits") == null) {
        this.getServletContext().setAttribute("visits",0);
    }
    //显示网站访问的次数获取visits并让visits自增1
    int visits = (int) this.getServletContext().getAttribute("visits");
    visits ++;
    this.getServletContext().setAttribute("visits",visits);
    resp.getWriter().write("<b>本站访问次数 " + visits + " 次数</b>");
```

这里还需要注意一点，如果你停掉你的tomcat服务器，ServletContext的生命周期也就结束了，“访问次数”也随之服务器的停止而消失。

因此，为了使“访问次数”保留，需要用IO存在外存的文件上。

```java
@WebServlet(name = "VisitsServlet")
public class VisitsServlet extends HttpServlet {

    @Override
    public void init() throws ServletException {
        FileReader fileReader = null;
        BufferedReader bufferedReader = null;
        String visitsStr = null;
        String path = null;
        try {
            path = this.getServletContext().getRealPath("/WEB-INF/log/visits.txt");
            fileReader = new FileReader(path);
            bufferedReader = new BufferedReader(fileReader);
            visitsStr = bufferedReader.readLine();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

        if (visitsStr == null) {
            visitsStr = "0";
        }

        int visits = Integer.parseInt(visitsStr);
        System.out.println("\n" + path + "提取出的visits：" + visits);
        this.getServletContext().setAttribute("visits", visits);
    }

    @Override
    public void destroy() {
        int visits = (int) this.getServletContext().getAttribute("visits");
        System.out.println("\n将页面访问的" + visits + "次保存到visits.txt文件中");

        FileWriter fileWriter = null;
        BufferedWriter bufferedWriter = null;
        try {
            String path = this.getServletContext().getRealPath("/WEB-INF/log/visits.txt");
            System.out.println("visits.txt 保存的路径为:" + path);
            fileWriter = new FileWriter(path);
            bufferedWriter = new BufferedWriter(fileWriter);
            bufferedWriter.write(visits + "");
            bufferedWriter.flush();
        } catch (IOException e) {
            System.out.println("保存visits出错");
            e.printStackTrace();
        }
    }
}
```
