### SpringBoot拦截器实现简单的登录认证模块

#### 1.前端界面

```
<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>登录注册界面</title>
<link rel="stylesheet" type="text/css" href="../css/login.css">
</head>
<body>
<div id="wrapper" class="login-page">
<div id="login_form" class="form">
<form class="login-form" action="../admin/login" method="get">
 <h2>管理登录</h2>
<input type="text" placeholder="用户名" value="admin" id="user_name" name="username"/>
<input type="password" placeholder="密码" id="password" name="password"/>
<button type="submit">登　录</button>
    <p class="message">${msg}</p>
</form>
</div>
</div>

<script src="..js/jquery.min.js"></script>
<script type="text/javascript">
	function check_login()
	{
	 var name=$("#user_name").val();
	 var pass=$("#password").val();
	 if(name=="Admin" && pass=="Admin")
	 {
	  alert("登录成功！");
	  $("#user_name").val("");
	  $("#password").val("");
       $(location).attr('href', 'index.html');
	 }
	 else
	 {
	  $("#login_form").removeClass('shake_effect');  
	  setTimeout(function()
	  {
	   $("#login_form").addClass('shake_effect')
	  },1);  
	 }
	}
	function check_register(){
		var name = $("#r_user_name").val();
		var pass = $("#r_password").val();
		var email = $("r_email").val();
		if(name!="" && pass=="" && email != "")
		 {
		  alert("注册成功！");
		  $("#user_name").val("");
		  $("#password").val("");
		 }
		 else
		 {
		  $("#login_form").removeClass('shake_effect');  
		  setTimeout(function()
		  {
		   $("#login_form").addClass('shake_effect')
		  },1);  
		 }
	}
	$(function(){
		$("#create").click(function(){
			check_register();
			return false;
		})
		$("#login").click(function(){
			check_login();
			return false;
		})
		$('.message a').click(function () {
		    $('form').animate({
		        height: 'toggle',
		        opacity: 'toggle'
		    }, 'slow');
		});
	})
	</script>
</body>
</html>
```

#### 2.跳转后跳转到index界面

#### 3.后端代码

##### 程序入口

```
@EnableAutoConfiguration
@MapperScan(basePackages = "com.gyf.dao")

//@ComponentScan(basePackages = {"com.gyf.web","com.gyf.service"})
//@ComponentScan(basePackages = {"com.gyf.dbconfig","com.gyf.datasource","com.gyf.web","com.gyf.test1.service","com.gyf.test2.service"})
@ComponentScan(basePackages = "com.gyf")
//@MapperScan(basePackages = "com.gyf.dao")
public class App 
{
    public static void main( String[] args )
    {
        SpringApplication.run(App.class,args);
    }
}
```

##### Controller

```
@Controller
@RequestMapping("admin")
public class AdminController {

    @Autowired
    private AdminService adminService;

    @RequestMapping("login")
    public String login(@Param("username") String username, @Param("password") String password, Model model, HttpServletRequest req){
        Admin admin = adminService.login(username,password);
        if (admin != null){
            req.getSession().setAttribute("admin",admin.getAdmin_name());
            System.out.println(admin.getAdmin_name());
            model.addAttribute("admin",admin);
            return "index";
        }else {
            model.addAttribute("msg","用户名或密码错误");
            return "login";
        }
    }
}
```

##### Interceptor

LoginInterceptor.java是整个登录认证模块中的核心类之一，它实现了`HandlerInterceptor`类，由它来拦截并过滤到来的每一个请求；它的三个方法能分别作用于每个请求的不同生命周期，你可以根据自己的需要来加入相应的处理逻辑；

```
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        Object admin_name = httpServletRequest.getSession().getAttribute("admin");
        if (admin_name == null || !(admin_name instanceof String)){
            httpServletResponse.sendRedirect("/admin/login");
            return false;
        }
        String admin_name1 = (String) admin_name;
        System.out.println("当前用户已登录用户名为"+admin_name1);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}
```

##### Configuration

```
LoginConfiguration.java是另一个核心类之一，它实现了WebMvcConfigurer类，负责注册并生效我们自己定义的拦截器配置；
在这里要注意定义好拦截路径和排除拦截的路径；
```

```
@Configuration
public class LoginConfiguration implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 注册拦截器
        LoginInterceptor loginInterceptor = new LoginInterceptor();
        InterceptorRegistration loginRegistry = registry.addInterceptor(loginInterceptor);
        // 拦截路径
        loginRegistry.addPathPatterns("/**");
        // 排除路径
        loginRegistry.excludePathPatterns("/");
        loginRegistry.excludePathPatterns("/view/login");
        loginRegistry.excludePathPatterns("/admin/login");
        loginRegistry.excludePathPatterns("/loginout");
        // 排除资源请求
        loginRegistry.excludePathPatterns("/css/**");
        loginRegistry.excludePathPatterns("/fonts/**");
        loginRegistry.excludePathPatterns("/images/**");
        loginRegistry.excludePathPatterns("/js/**");
        loginRegistry.excludePathPatterns("/lib/**");
    }
}
```

