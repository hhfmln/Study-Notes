### java三大组件之过滤器Filter

#### 1.Filter简介

Filter也称之为过滤器，WEB开发人员通过Filter技术，对web服务器管理的所有web资源：例如Jsp,Servlet, 静态图片文件或静态html文件等进行拦截，从而实现一些特殊的功能。例如实现URL级别的权限访问控制、过滤敏感词汇、压缩响应信息等一些高级功能。
　　Servlet API中提供了一个Filter接口，开发web应用时，如果编写的Java类实现了这个接口，则把这个java类称之为过滤器Filter。通过Filter
技术，开发人员可以实现用户在访问某个目标资源之前，对访问的请求和响应进行拦截，Filter接口源代码：

```
public abstract interface Filter{
    public abstract void init(FilterConfig paramFilterConfig) throws ServletException;
    public abstract void doFilter(ServletRequest paramServletRequest, ServletResponse paramServletResponse, FilterChain 
        paramFilterChain) throws IOException, ServletException;
    public abstract void destroy();
}
```

#### 2.Filter是如何实现拦截的？

Filter接口中有一个doFilter方法，当我们编写好Filter，并配置对哪个web资源进行拦截后，WEB服务器每次在调用web资源的service方法之前，都会先调用一下filter的doFilter方法，因此，在该方法内编写代码可达到如下目的：
​    调用目标资源之前，让一段代码执行。
​    是否调用目标资源（即是否让用户访问web资源）。
​    调用目标资源之后，让一段代码执行。
　　web服务器在调用doFilter方法时，会传递一个filterChain对象进来，filterChain对象是filter接口中最重要的一个对象，它也提供了一个
doFilter方法，开发人员可以根据需求决定是否调用此方法，调用该方法，则web服务器就会调用web资源的service方法，即web资源就会被访问，否则web资源不会被访问。

#### 3.Filter开发流程

Filter开发分为2步：

编写java类实现Filter接口，并实现其doFilter方法。

在web.xml 文件中使用<filter>和<filter-mapping>元素对编写的filter类进行注册，并设置它所能拦截的资源。

过滤器范例：

```
import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
/**
 * @author hhf
 * @description 过滤器Filter的工作原理
 */
public class FilterTest implements Filter{
 
    public void destroy() {
        System.out.println("----Filter销毁----");
    }
 
    public void doFilter(ServletRequest request, ServletResponse response,FilterChain filterChain) throws IOException, ServletException {
        // 对request、response进行一些预处理
        request.setCharacterEncoding("UTF-8");
        response.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");
        System.out.println("----调用service之前执行一段代码----");
        filterChain.doFilter(request, response); // 执行目标资源，放行
        System.out.println("----调用service之后执行一段代码----");
    }
 
    public void init(FilterConfig arg0) throws ServletException {
        System.out.println("----Filter初始化----");
    }
}
```

##### 在web. xml中配置过滤器：

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.0" 
    xmlns="http://java.sun.com/xml/ns/javaee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
  <display-name></display-name>    
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
  <!--配置过滤器-->
  <filter>
      <filter-name>FilterTest</filter-name>
      <filter-class>com.yangcq.filter.FilterTest</filter-class>
  </filter>
  <!--映射过滤器-->
  <filter-mapping>
      <filter-name>FilterTest</filter-name>
      <!--“/*”表示拦截所有的请求 -->
      <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
```

##### 过滤非法字符

```
import java.io.IOException;
import java.util.Enumeration;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import org.apache.commons.lang.StringUtils;
/**
 * SelfDefineInvalidCharacterFilter：过滤request请求中的非法字符，防止脚本攻击
 */
public class SelfDefineInvalidCharacterFilter implements Filter{
 
    public void destroy() {
        
    }
 
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        String parameterName = null;
        String parameterValue = null;
        // 获取请求的参数
        @SuppressWarnings("unchecked")
        Enumeration<String> allParameter = request.getParameterNames();
        while(allParameter.hasMoreElements()){
            parameterName = allParameter.nextElement();
            parameterValue = request.getParameter(parameterName);
            if(null != parameterValue){
                for(String str : invalidCharacter){
                    if (StringUtils.containsIgnoreCase(parameterValue, str)){
                        request.setAttribute("errorMessage", "非法字符：" + str);
                        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/error.jsp");
                        requestDispatcher.forward(request, response);
                        return;
                    }
                }
            }
        }
        filterChain.doFilter(request, response); // 执行目标资源，放行
    }
 
    public void init(FilterConfig filterConfig) throws ServletException {
        
    }
    // 需要过滤的非法字符
    private static String[] invalidCharacter = new String[]{
        "script","select","insert","document","window","function",
        "delete","update","prompt","alert","create","alter",
        "drop","iframe","link","where","replace","function","onabort",
        "onactivate","onafterprint","onafterupdate","onbeforeactivate",
        "onbeforecopy","onbeforecut","onbeforedeactivateonfocus",
        "onkeydown","onkeypress","onkeyup","onload",
        "expression","applet","layer","ilayeditfocus","onbeforepaste",
        "onbeforeprint","onbeforeunload","onbeforeupdate",
        "onblur","onbounce","oncellchange","oncontextmenu",
        "oncontrolselect","oncopy","oncut","ondataavailable",
        "ondatasetchanged","ondatasetcomplete","ondeactivate",
        "ondrag","ondrop","onerror","onfilterchange","onfinish","onhelp",
        "onlayoutcomplete","onlosecapture","onmouse","ote",
        "onpropertychange","onreadystatechange","onreset","onresize",
        "onresizeend","onresizestart","onrow","onscroll",
        "onselect","onstaronsubmit","onunload","IMgsrc","infarction"
    };
}
```

##### web.xml

```
<filter>
        <filter-name>SelfDefineInvalidCharacterFilter</filter-name>
        <filter-class>com.yangcq.filter.SelfDefineInvalidCharacterFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>SelfDefineInvalidCharacterFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

#### 4.Filter链

在一个web应用中，可以开发编写多个Filter，这些Filter组合起来称之为一个Filter链。web服务器根据Filter在web.xml文件中的注册顺序，
决定先调用哪个Filter，当第一个Filter的doFilter方法被调用时，web服务器会创建一个代表Filter链的FilterChain对象传递给该方法。在doFilter方法中，开发人员如果调用了FilterChain对象的doFilter方法，则web服务器会检查FilterChain对象中是否还有filter，如果有，则调用第2个filter，如果没有，则调用目标资源。

#### 5.Filter的创建

Filter的创建和销毁由web服务器负责。 web应用程序启动时，web服务器将创建Filter的实例对象，并调用其init方法，完成对象的初始化
功能，从而为后续的用户请求作好拦截的准备工作，filter对象只会创建一次，init方法也只会执行一次。通过init方法的参数，可获得代表当前
filter配置信息的FilterConfig对象。

##### Filter的销毁

web容器调用destroy方法销毁Filter。destroy方法在Filter的生命周期中仅执行一次。在destroy方法中，可以释放过滤器使用的资源。

##### FilterConfig

用户在配置filter时，可以使用<init-param>为filter配置一些初始化参数，当web容器实例化Filter对象，调用其init方法时，会把封装了
filter初始化参数的filterConfig对象传递进来。因此开发人员在编写filter时，通过filterConfig对象的方法，就可获得：
　　String getFilterName()：得到filter的名称。
　　String getInitParameter(String name)： 返回在部署描述中指定名称的初始化参数的值。如果不存在返回null.
　　Enumeration getInitParameterNames()：返回过滤器的所有初始化参数的名字的枚举集合。
　　public ServletContext getServletContext()：返回Servlet上下文对象的引用。

#### 6.Filter的部署时的一些参数的含义

##### Filter的部署分为两个步骤：

##### 　　1、注册Filter

```
<filter>
    <description>过滤器名称</description>
    <filter-name>自定义的名字</filter-name>
    <filter-class>com.yangcq.filter.FilterTest</filter-class>
    <!--配置FilterTest过滤器的初始化参数-->
    <init-param>
        <description>配置过滤器的初始化参数</description>
        <param-name>name</param-name>
        <param-value>gacl</param-value>
    </init-param>
    <init-param>
        <description>配置FilterTest过滤器的初始化参数</description>
        <param-name>like</param-name>
        <param-value>java</param-value>
    </init-param>
</filter>

```

##### 　　2、映射Filter

```
<!--映射过滤器-->
  <filter-mapping>
      <filter-name>FilterTest</filter-name>
      <!--“/*”表示拦截所有的请求 -->
      <url-pattern>/*</url-pattern>
  </filter-mapping>

```

REQUEST：当用户直接访问页面时，Web容器将会调用过滤器。如果目标资源是通过RequestDispatcher的include()或forward()方法访问
时，那么该过滤器就不会被调用。
​    INCLUDE：如果目标资源是通过RequestDispatcher的include()方法访问时，那么该过滤器将被调用。除此之外，该过滤器不会被调用。
​    FORWARD：如果目标资源是通过RequestDispatcher的forward()方法访问时，那么该过滤器将被调用，除此之外，该过滤器不会被调用。
​    ERROR：如果目标资源是通过声明式异常处理机制调用时，那么该过滤器将被调用。除此之外，过滤器不会被调用。

