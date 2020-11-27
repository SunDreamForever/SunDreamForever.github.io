# shopping 项目 

临时补充:

```
谷歌浏览器 80之后的版本都会存才一个SameSite属性 但是会影响cookie使用
chrome://flags/地址栏输入此字符串
然后在选项框中选中相应的功能,关闭SameSite的功能 修改示例如下图

还有就是白名单功能先不使用 , 在最后再添加 免得影响500错误的提示 ,导致编程效率的低下.
```

![image-20201123171610297](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20201123171610297.png)

### 整体

```
1. 数据库的整理 
2. 准备前端 页面 
3. 准备服务端 
4. Result格式 User实体类 BeanFactory工厂 JDBCUtil工具 ObjectUtil工具 UUIDUtil工具 db数据库连接配置文件
```

### 一 建立通信与数据传输

```
第一招: 欺骗域名 --> hosts 的编写 ; HBuilder的使用 以及 对 已有前端页面资源的 导入使用(这里要十分注意路径问题)
第二招: 前后端通信 --> 把filter写了,给通关文牒一样的响应头 不然浏览器 截胡
第三招: 前后端数据的传送 --> 简单的和难的  简单的->脾气好的前端 一行jquery ;难的-->和你有仇的前端 后端 一页的代码
总结: 没事多给前端小姐姐送奶茶 有火花,少干活 
```

#### 前端

```HTML
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<script type="text/javascript" src="resources/js/axios-0.18.0.js"></script>
		<script type="text/javascript" src="resources/js/jquery-1.11.3.min.js"></script>
		<script >
			function sendReq(){
				//api应用程序接口
				let user={
					username:"小明",
					password:"123456"
				}
				
				let params = $.param(user);
				
				axios.post("http://www.itheima370.com:8080/data04",params)
				.then(function(response){
					let result = response.data;
					
					console.log(result);
					if(result.code==1){
						
					}					
				})
			}
		</script>
	</head>
	<body>
		<input type="button" value="给 服务器互发数据" onclick="sendReq()" />	
	</body>
</html>

```

#### 后端

1.创建过滤器CORSFilter,将请求地址加入到白名单中,在响应头里添加"Access-Control-Allow-Origin",解决前后端跨域问题.

```java
@WebFilter("/*")
public class CORSFilter implements Filter {
    public void destroy() {
    }

    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;
        request.setCharacterEncoding("utf-8");

        //先看看你是谁?
        String origin = request.getHeader("origin");

        //实际开发过程中,有白名单
        List<String> list = new ArrayList<>();
        //添加白名单
        list.add("http://www.itheima370.com:8020");

        if (origin != null) {
            if (list.contains(origin)) {
                //设置相应头
                //服务器端同意数据的使用
                response.setHeader("Access-Control-Allow-Origin", origin);
            } else {
                return;
            }
        }
        chain.doFilter(req, resp);
    }

    public void init(FilterConfig config) throws ServletException {

    }

}
```

2.TestServlet 用于测试接收前端转来的json格式的 数据 并且解析 json

```java
@WebServlet("/test")
public class TestServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("utf-8");

        String username = req.getParameter("username");
        System.out.println("接收到的用户名:" + username);

        //通常定义基本返回的数据格式
        Result result = new Result(1, "操作成功", null);

        ObjectMapper objectMapper = new ObjectMapper();
        String s = objectMapper.writeValueAsString(result);

        resp.setContentType("application/json;charset=utf-8");
        resp.getWriter().print(s);
    }
}

```

![image-20201121155235938](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20201121155235938.png)

浏览器控制台打印如上所示;

### 二 编写注册功能

#### 前端:

```
第一步: 选中要操作的div 进行id编写
第二步: 将创建vue;创建 data:{user&error}以及methods:{register方法}
第三步: 细节编写div中的每一条要输入数据相应的 div 加入v-model属性 并给 user对象 赋值
第四步: 将提交按钮 改变属性为button 并绑定onclick="register"的注册方法
```

代码如下

```html
<style>
		.err{
			color:red;
		}
	</style>	

	<div id="app" class="row">
        
        <div  v-model="" ></div>
        
       </div> 
									
<input @click="register" type="button" width="100" value="注册" name="submit" border="0" style="background-color:#CD062D;height:35px;width:100px;color:white;">
<script>
		new Vue({
			el:"#app",
			data:{
				user:{},
				errors:{
					username:"",
					password:""
				}
			},
			methods:{
				register(){
					//作为一个 令牌 ,有true才能继续执行
					let flag=true;
					//校验格式!!!!
					//校验用户名,必须是4-8之间 必须是字母数字
					let username=this.user.username;
					let req_username=/^[a-zA-Z_0-9]{4,8}$/
					if(!req_username.test(username)){
						//不正确在方法体内执行
						this.errors.username="用户名格式不正确!!!!";
						flag=false;						
					}else{
						this.errors.username="";					
					}
					//校验密码.必须是6-18之间 必须是字母数字
					let password=this.user.password;
					let req_password=/^[a-zA-Z_0-9]{6,18}$/
					if(!req_password.test(password)){
						//不正确执行此方法体
						this.errors.password="密码格式不正确!!!"
						flag=false;
					}else{
						this.errors.password=""
					}
					console.log(this.user);
					
					if(flag){
						//提交数据
						
						//使用axios发送请求的时候
						let param=$.param(this.user);
						axios.post("/user/register",param)
						.then(function(response){//服务端发送过来响应之后 跳转到路径"登录页面上去"
							let result=response.data;
							if(result.code==1){
								location.href="login.html";
							}
						})
					}
				}
			}
			
		})
		
	</script>
```

#### 后端:

编写baseServlet(抽象方法父类 -->用于被普通的servlet继承)

```
1.考虑到 很多功能 以及需求会在统一条 线程进行操作 ,倒不如直接把请求和响应对象直接放在 线程助手ThreadLocal当中 
2.在主流的service方法中  通过请求路径的分割 拿到相应的 请求方法名,在使用反射,拿到对应方法名的方法,并调用此方法对请求和响应进行处理
3.编写一个方法 对从前端传过来的数据格式进行 解析json数据的操作.
4.封装ok和no方法 然后简化后面的操作 (失败和成功的信息)
```

代码如下

```java
package com.itheima.web.servlet.biz;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.itheima.web.vo.Result;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.lang.reflect.Method;

//创建抽象父类 这样就可以默认修饰符为
public abstract class BaseServlet extends HttpServlet {
    ThreadLocal<HttpServletRequest> requestThreadLocal = new ThreadLocal<>();
    ThreadLocal<HttpServletResponse> responseThreadLocal = new ThreadLocal<>();
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws  IOException {
        //获取当前请求路径
        String uri = req.getRequestURI();
        //{/user/login}
        //找到最后一个 / 的在路径上的索引位置
        int i = uri.lastIndexOf("/");
        //截取 uri 在 i 这个索引之后的字符串
        String func = uri.substring(i + 1);
        System.out.println("获取当前正在 访问的逻辑方法名字: " + func);

        //找到这个方法-->有名字 通过反射就行
        // getDeclaredMethod(方法名, 方法的形参列表..)
        try {
            Method method = this.getClass().getDeclaredMethod(func, HttpServletRequest.class, HttpServletResponse.class);
            //将request 和 response 对象放到 thread对象中 使用线程助理 能够在程序运行的线程中 说取走就取走,不受到其他线程的影响
            // 拿到线程助理

            //线程助理将对象放到对应的线程助理对象中保存
            requestThreadLocal.set(req);
            responseThreadLocal.set(resp);

            //反射调用这个方法
            method.invoke(this, req, resp);
        } catch (NoSuchMethodException e) {
            System.out.println("老兄,你的请求路径跟你的方法名 不一致~~");
        } catch (Exception e) {
            e.printStackTrace();
            //这里的异常 是兜底的 !!!
            //给人家一些提示 ,免得把自己服务器的一些500错误反馈给客户 ,让别人给你查看异常
            Result result = new Result(0, "服务器开了小差,马上回来,别着急", null);
            ObjectMapper objectMapper = new ObjectMapper();
            String s = objectMapper.writeValueAsString(result);
            resp.setContentType("application/json;charset=utf-8");
            resp.getWriter().print(s);

        }
    }

    public void ok(Object bizData) throws IOException {
        //拿到前端传过来的对象,并创建以对象作为data数值的result对象
        Result result = new Result(1, "操作成功", bizData);
        //调用writeResult方法 将前端传过来的 数据打印出来
        writeResult(result);
    }

    public void ok() throws IOException {
        //有一些不需要值的操作
        Result result = new Result(1, "操作成功", null);
        //调用writeResult方法 将前端传过来的 数据打印出来
        writeResult(result);
    }

    public void no(Object bizData) throws IOException {
        //一个不正确的数值 创建操作失败的result对象
        Result result = new Result(0, "操作失败", bizData);
        writeResult(result);
    }

    public void no() throws IOException {
        //有一些不需要值的操作
        Result result = new Result(0, "操作失败", null);
        writeResult(result);
    }

    public void writeResult(Result result) throws IOException {
        //解析接收的 json 格式的数据
        ObjectMapper objectMapper = new ObjectMapper();
        String s = objectMapper.writeValueAsString(result);

        //线程助理里面提出需要的对象
        //响应对象拿到手,再将解析的字符通过响应对象的打印流 打印出来
        HttpServletResponse response = responseThreadLocal.get();
        response.setContentType("application/json;charset=utf-8");
        response.getWriter().print(s);

    }
}


```

创建UserServlet-->分配登录路径

```
1.拿到请求的数据,在后端进行简单的数据格式的校验-->应用正则
2.创建对象,为了便于后端对数据的管理,我们后端服务器给用户分配id
3.由于用户注册要输入相对比较多的信息,用map集合来进行封装,在接下来就是简单的无脑操作->service->dao那种
4.调用ok方法,给前端一个相应成功的反馈.

```

代码如下

```java
@WebServlet({
        "/user/register"
})
public class UserServlet extends BaseServlet {
    private UserService userService = BeanFactory.getBean(UserService.class);

    public void register(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //提交给我们的数据 用户名 密码 邮箱 啊...
        //提交的数据格式 不对怎么
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        //一次性将 需要校验的 数据全部校验玩  把错误信息收集起来
        //错误的信息 有键值对,那就用hashmap来接收
        Map<String, String> errors = new HashMap<>();
        if (username != null && !username.matches("^[a-zA-Z_0-9]{4,8}$")) {
            errors.put("username", "用户名格式不正确");
        }
        if (password != null && !password.matches("^[a-zA-Z_0-9]{6,18}$")) {
            errors.put("password", "密码格式不正确");
        }
        //判断一下map集合大小
        if (errors.size() > 0) {
            //有一个数据不对,那都不能注册成功
            // 给人家返回错误的信息对象,让人家知道那儿错了
            no(errors);
        } else {
            //走到这一步就说明 ,传过来的数据是对的 能注册
            //调用service 保存用户 用工厂模式那种方法
            User user = new User();
            //id 创建一个id 我们自己写的  -->使用工具类UUID
            user.setUid(UUIDUtil.randID());

            //创建一个注册信息的集合
            Map<String, String[]> parameterMap = req.getParameterMap();
            //BeanUtils位于org.apache.commons.beanutils.BeanUtils下面，其方法populate的作用解释如下：
            //BeanUtils.populate( Object bean, Map properties )，
            //这个方法会遍历map<key, value>中的key，如果bean中有这个属性，就把这个key对应的value值赋给bean的属性。
            try {
                BeanUtils.populate(user, parameterMap);
                System.out.println(user);
            } catch (Exception e) {
                System.out.println("除了问题");
                e.printStackTrace();
            }
            userService.save(user);
            //返回成功信息
            ok();
        }
    }
}

```

### 三 用户登陆登出功能

#### 前端:

```
1.对登录页面login.html进行编写
2.vue 绑定 对应的 div 自己选,并创建 user对象
3.输入的信息div使用v-model双向绑定赋予user对象相应的 姓名 密码值
4.修改验证码图片的 src路径 指向 后端服务器提供的code.png路径 -->servlet中使用java拼写了验证码图片
5.验证码div 绑定方法changImg ; 登录div绑定lougin方法

6.修改header.html
7.编写<ol><li>列表->展示功能
8.使用jquery-->里面new Vue(created方法在vue创建的时候就启动了-->绑定回显功能,回显登陆的人姓名[使用cookie从 session中获取])
9.logout方法绑定退出的<li><a>
```

login代码如下

```html
<div id="loginForm"   class="col-md-5">
<div class="col-sm-6">
	<input type="text" class="form-control" id="username" v-model="user.username" placeholder="请输入用户名" name="username">
</div>
	
<div class="col-sm-6">
	<input type="password" class="form-control" id="password" v-model="user.password" placeholder="请输入密码" name="password">
</div>

<div class="col-sm-3">
	<input type="text" class="form-control" id="code" v-model="user.code" placeholder="请输入验证码">
</div>

<div class="col-sm-3">
	<img id="codeImg" @click="changeImg()"  src="http://www.itheima370.com:8080/code.png" />
</div>


<script>
		new Vue({
			el:"#loginForm",
			data:{
				user:{},
				error:""
			},
			methods:{
				changeImg(){
					let t =new Date().getTime();
					//attr() 拿到第一个索引位置的key对应元素的值
					$("#codeImg").attr("src","http://www.itheima370.com:8080/code.png?_t="+t)
				},
				login(){
					//提交当前user数据
					let params=$.param(this.user)
					
					let f=(response)=>{
						//箭头函数体内 是没有调用者 这个说法的
						//捕获声明上下文中的this
						console.log(this);
						let result=response.data;
					
						if(result.code==0){
							this.error=result.bizData;
						}else{
							//成功跳转首页
							location.href="index.html";													
						}					
					}
					
					axios.post("/user/login",params)
					.then(f);
				}
			}
			
		})
		
</script>
```

表头代码如下:包含logout 以及 回显 登陆用户名

```html
<script>
	$(function(){
		new Vue({
			el:"#login-menu",
			data:{
				nickname:""
			},
			created(){//回显姓名
				axios.post("/user/myname")
				.then((response)=>{
					let result=response.data;
					if(result.code==1){
						this.nickname=result.bizData;
					}
				})
			},
			methods:{
				logout(){
					//发送请求 销毁自己的session
					axios.post("/user/logout")
					.then((response)=>{
						this.nickname="";
					})
				}
			}
		});		
	})
</script>    

<ol  v-if="nickname==''" class="list-inline">
	<li><a href="http://www.itheima370.com:8020/web/login.html">登录</a></li>
	<li><a href="http://www.itheima370.com:8020/web/register.html">注册</a>		</li>
	<li><a href="http://www.itheima370.com:8020/web/view/cart/list.html">购物车	</a></li>
	<li><a href="http://www.itheima370.com:8020/web/view/cart/list.html">我的订	单</a></li>
</ol>
<ol  v-else class="list-inline">
	<li>{{nickname}}欢迎您</li>
	<li><a href="javascript:;" @click="logout">退出登陆</a></li>
	<li><a href="http://www.itheima370.com:8020/web/view/cart/list.html">购物车	</a></li>
	<li><a href="http://www.itheima370.com:8020/web/view/cart/list.html">我的订	单</a></li>
</ol>
```

#### 后端:

```
1.引入一个servlet编验证码图片
2.logout代码编写,校验前端发送来的user信息是否存在
3.myname方法编写,从session中取出登陆用户的信息中的名字回显在表头中
4.logout代码编写,简单的销毁用户对应的session,就取不到user了
5.CONSTANTS 编码java类的编写, 便于整个项目开发的整齐
```

验证码servlet

```java

@WebServlet("/code.png")
public class CodeServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("utf-8");
        resp.setContentType("text/html;charset=utf-8");

        //使用java图形界面技术绘制一张图
        int charNum = 4;
        int width = 20 * 4;
        int height = 28;

        //1.创建一张内存图片
        BufferedImage bufferedImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);

        // 2.获得绘图对象
        Graphics graphics = bufferedImage.getGraphics();

        // 3、绘制背景颜色
        graphics.setColor(Color.YELLOW);
        graphics.fillRect(0, 0, width, height);

        // 4、绘制图片边框
        graphics.setColor(Color.GRAY);
        graphics.drawRect(0, 0, width - 1, height - 1);

        // 5、输出验证码内容
        graphics.setColor(Color.RED);
        graphics.setFont(new Font("宋体", Font.BOLD, 22));

        // 随机输出4个字符
        String s = "ABCDEFGHGKLMNPQRSTUVWXYZ23456789";
        Random random = new Random();


        String msg = "";

        int x = 5;
        for (int i = 0; i < charNum; i++) {
            int index = random.nextInt(32);
            String content = String.valueOf(s.charAt(index));

            msg += content;
            graphics.setColor(new Color(random.nextInt(255), random.nextInt(255), random.nextInt(255)));
            graphics.drawString(content, x, 22);
            x += 20;
        }


        req.getSession().setAttribute("code",msg);



        // 6、绘制干扰线
        graphics.setColor(Color.GRAY);
        for (int i = 0; i < 5; i++) {
            int x1 = random.nextInt(width);
            int x2 = random.nextInt(width);

            int y1 = random.nextInt(height);
            int y2 = random.nextInt(height);
            graphics.drawLine(x1, y1, x2, y2);
        }

        // 释放资源
        graphics.dispose();

        ImageIO.write(bufferedImage, "png", resp.getOutputStream());
    }
}
```

userServlet编写

```java
@WebServlet({
        "/user/login",
        "/user/register",
        "/user/myname",
        "/user/logout"
})
public class UserServlet extends BaseServlet {
    private UserService userService = BeanFactory.getBean(UserService.class);

    //注册
    public void register(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //提交给我们的数据 用户名 密码 邮箱 啊...
        //提交的数据格式 不对怎么
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        //一次性将 需要校验的 数据全部校验玩  把错误信息收集起来
        //错误的信息 有键值对,那就用hashmap来接收
        Map<String, String> errors = new HashMap<>();
        if (username != null && !username.matches("^[a-zA-Z_0-9]{4,8}$")) {
            errors.put("username", "用户名格式不正确");
        }
        if (password != null && !password.matches("^[a-zA-Z_0-9]{6,18}$")) {
            errors.put("password", "密码格式不正确");
        }
        //判断一下map集合大小
        if (errors.size() > 0) {
            //有一个数据不对,那都不能注册成功
            // 给人家返回错误的信息对象,让人家知道那儿错了
            no(errors);
        } else {
            //走到这一步就说明 ,传过来的数据是对的 能注册
            //调用service 保存用户 用工厂模式那种方法
            User user = new User();
            //id 创建一个id 我们自己写的  -->使用工具类UUID
            user.setUid(UUIDUtil.randID());

            //创建一个注册信息的集合
            Map<String, String[]> parameterMap = req.getParameterMap();
            //BeanUtils位于org.apache.commons.beanutils.BeanUtils下面，其方法populate的作用解释如下：
            //BeanUtils.populate( Object bean, Map properties )，
            //这个方法会遍历map<key, value>中的key，如果bean中有这个属性，就把这个key对应的value值赋给bean的属性。
            try {
                BeanUtils.populate(user, parameterMap);
                System.out.println(user);
            } catch (Exception e) {
                System.out.println("除了问题");
                e.printStackTrace();
            }
            userService.save(user);

            //返回成功信息
            ok();

        }

    }

    //登陆
    public void login(HttpServletRequest req,HttpServletResponse resp) throws IOException {
        //验证码 为了防止机器人
        //取出验证码
        String code_request = req.getParameter("code");
        String code_session = (String) req.getSession().getAttribute("code");

        if (code_request==null//请求是否输入了验证码
                ||code_request.trim().length()==0//请求里的验证码是否为空字符串
                ||!code_request.equalsIgnoreCase(code_session)//请求里输入的验证码不正确
        ) {
            //以上符合一样
            no("您输入的验证码不正确");
            return;
        }

        //接收用户名和密码
        String username = req.getParameter("username");
        String password = req.getParameter("password");

        //检查用户名是否是正确的
        User user = userService.findUserByUsernameAndPassword(username, password);

        if (user != null) {
            //这个输入的密码和名字是数据库里的
            //登陆成功 保存登陆状态, 将自己的user对象保存到Session中
            req.getSession().setAttribute(CONSTANTS.SESSION_USER_NAME, user);

            ok();
            //移除session code值
            req.getSession().removeAttribute("code");
        } else {
            no("账户密码错误");
        }

    }

    //在页头显示 已经登陆的用户的名字 和 登出 的关键字
    public void myname(HttpServletRequest req,HttpServletResponse resp) throws IOException {
        User user = (User) req.getSession().getAttribute(CONSTANTS.SESSION_USER_NAME);

        if (user != null) {
            ok(user.getNickname());
        } else {
            no();
        }
    }

    //登出
    public void logout(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        //invalidate session自废武功
        req.getSession().invalidate();
        ok();
    }
}
```

CONSTANTS.java类的编写

```java
public interface CONSTANTS {
    String SESSION_USER_NAME = "user";
}
```

### 四 商品分类信息的展示

#### 前端

```
1.对head.html进行编写,将id为cate_list的div绑定一个Vue对象,用来回显从数据库查询来的分类信息
2.查询回来的分类信息为一个集合,用v-for来进行遍历,对应的<ul><li>的标签项 将遍历的每一项 信息 展示出来-->根据cid展示cname
3.<ul><li>查询cid的操作,绑定的路径需要跳转/product/list.html
10.axios从
```

header代码如下:

```js
new Vue({
			el:"#cate_list",
			data:{
				//商品分类集合
				categoryList:[]
			},
			created(){
				axios.post("/category/findAll")
				.then((response)=>{
					//访问成功,将响应数据中的result提取出来
					let result = response.data;
					//拿到result里面的bizData 放入商品分类
					this.categoryList=result.bizData;
				});
			}
		})

```

```html
<ul  id="cate_list" class="nav navbar-nav">
	<li v-for="cate in categoryList"><!--遍历categoryList 取出cate  -->
	<a :href="'http://www.itheima370.com:8020/web/view/product/list.html?cid='+cate.cid">{{cate.cname}}	</a>
	</li>
</ul>
				
```

#### 后端

```
1.创建CategoryServlet,调用categoryService来进行下一步操作
2.导入redis.properties文件 和RedisUtil工具类 ,模板中的ObjectUtil工具类比较旧了,导入含有新功能的ObjectUtil
3.在categoryService的findAll方法中 ,
①因为商品分类信息的展示属于高频功能,且不常修改,所以选择使用redis来作为缓存,在展示的时候直接使用redis,减少数据库的压力
②获取redis的连接,从redis中根据key查询所需要的的value,如果没有,再从数据库中查询,具体见代码
4.继续编写dao以便能从数据库中查询需要的数据,保存在redis中,并且返还给前端
5.记得配置bean.xml
```

代码如下

```java

@WebServlet("/category/findAll")
public class CategoryServlet extends BaseServlet {
    private CategoryService categoryService = BeanFactory.getBean(CategoryService.class);
    
    public void findAll(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        //查询商品分类列表信息的集合
        List<Category> categories = categoryService.findAll();
        //返回给前端
        ok(categories);

    }
}


public class CategoryServiceImpl implements CategoryService {
    private CategoryDao categoryDao = BeanFactory.getBean(CategoryDao.class);

    /*
     * //常用高频数据 放到缓存中 减少对于(mysql) 数据库的压力
     * 什么数据适合放到缓存中?
     * 高频访问 且不容易改变的数据
     * 主要指标是 高频!!!
     * */
    @Override
    public List<Category> findAll() {

        //使用redis
        //获取连接
        Jedis connection = null;

        try {
            connection = RedisUtil.getConnection();
            //从缓存数据库中查询
            String categoryListCache = connection.get("categoryList_370");

            //如果缓存数据库中没有
            if (categoryListCache == null) {
                //在从物理数据库中查询
                List<Category> categoriesDB = physicsFromDatabase();

                //因为要转给前端 所以先转为json格式的 字符串!!!
                String s = ObjectUtil.writeJson(categoriesDB);
                connection.set("categoryList_370", s);

                return categoriesDB;
            } else {
                //缓存数据库存在 得到一个字符串
                //转换list集合
                return (List<Category>) ObjectUtil.readJson(categoryListCache, List.class);
            }
        } finally {
            //释放连接
            if (connection != null) {
                connection.close();
            }
        }

    }

    private List<Category> physicsFromDatabase() {
        return categoryDao.findAll();
    }
}

```

```xml
    <bean id="CategoryDao" class="com.itheima.dao.impl.CategoryDaoImpl"></bean>
    <bean id="CategoryService" class="com.itheima.service.impl.CategoryServiceImpl"></bean>

```

### 五 主页显示商品信息

#### 前端

```
1.主页index.html页面 中 在商品显示 添加一个大的id为products的div标签 ,来控制住整个商品显示
2.创建Vue对象,绑定上面创建的整个标签,创建news和hots 新产品对象集合和 热门产品对象集合
3.因为是主页信息,需要及时回显 create方法发送axios查询商品信息
4.v-for遍历对象集合中的商品,并在路径中带上相应的pid用作查询的根据,具体见代码

5.每一个商品的点击都绑定了一个地址,这个地址跳转的是info.html页面,这个页面展示的是当前被点击的商品的详细内容
6.在info.html页面中的给控制整个页面的div赋予一个id-->products
7.js代码中 使用HM工具获取主页跳转此页面时地址栏上的pid参数
8.创建Vue,创建product对象,因为页面直接显示,回显功能使用create 发送ajax请求,根据pid查询数据库中product信息

```

代码如下

index.html代码如下

```html
<div id="products">

<div v-for="p in hots" class="col-md-2" style="text-align:center;height:200px;padding:10px 0px;">
	<a :href="'http://www.itheima370.com:8020/web/view/product/info.html?pid='+p.pid">
		<img :src="'http://www.itheima370.com:8020/web/'+p.pimage" width="130" height="130" style="display: inline-block;">
	</a>
	<p>
	<a :href="'http://www.itheima370.com:8020/web/view/product/info.html?pid='+p.pid" style='color:#666'>{{p.pname}}</a>
	</p>
	<p>
	<font color="#E4393C" style="font-size:16px">&yen;{{p.shop_price}}</font>
	</p>
</div>

    
    
<div v-for="p in news" class="col-md-2" style="text-align:center;height:200px;padding:10px 0px;">
	<a :href="'http://www.itheima370.com:8020/web/view/product/info.html?pid='+p.pid">
	<img :src="'http://www.itheima370.com:8020/web/'+p.pimage" width="130" height="130" style="display: inline-block;">
	</a>
	<p>
	<a :href="'http://www.itheima370.com:8020/web/view/product/info.html?pid='+p.pid" style='color:#666'>{{p.pname}}</a>
	</p>
	<p>
	<font color="#E4393C" style="font-size:16px">&yen;{{p.shop_price}}</font>
	</p>
</div>
    
    
<script>
		new Vue({
			el: "#products",
			data: {
				news: [],
				hots: []
			},
			created() {
				axios.post("/product/hotsandnews")
					.then((response) => {
						let result = response.data;
						this.news = result.bizData.news;
						this.hots = result.bizData.hots;
					})
			}

		})
</script>

    
```

info.html

```html
<div id="product" class="container">
    里面还有很多替代属性{{product.属性名}}
<script>
		//发送请求,知道查询那个商品
		//从地址栏获取到 商品参数
		let pid=HM.getParameter("pid");
		
		if(pid==null){
			alert("兄嘚 你确定要直接打开吗?")
		}
		
		new Vue({
			el:"#product",
			data:{
				product:{}
			},
			created(){
				
				axios.post("/product/findById","pid="+pid)
				.then((response)=>{
					let result = response.Data;
					this.product=result.bizData;
				})
				
			}
			
		})
		
	</script>                
               
```

#### 后端

简单的功能书写,直接上代码,cv吧

### 六 分页展示各商品分类的商品

#### 前端

```
1.在list.html中进行操作,展示对应商品分类的商品,所以应该从主页上跳转到list.html路径中获取对应的商品分类的cid参数
2.给较大的div赋予id=product,然后创建Vue来控制整个页面
3.js代码中,先使用HM.getParamer获取请求路径中的cid参数,在Vue对象中创建pb对象,来保存商品信息
4.因为跳转过来之后立即回显,使用create(){} 
5.分页查询,需要发送两个参数,一个是要查询的商品类别cid,一个是当前页码pageNumber,将对象在前端封装转换,let params =$.param(paramsObj)  
6.axios的响应处理中,调用HM.page方法,传递查询到的那些商品pb,以及需要跳转的页面.HM.page中封装好了拼接分页的方法
7.板顶在对应的html中
```

前端代码展示

```html
<div id="products" class="container">

<div v-for="p in pb.data" class="col-md-2" style="height: 250px;">
	<a :href="'http://www.itheima370.com:8020/web/view/product/info.html?pid='+p.pid'">
		<img :src="'http://www.itheima370.com:8020/web/'+p.pimage" width="170" height="170" style="display: inline-block;">
	</a>
	<p><a :href="'http://www.itheima370.com:8020/web/view/product/info.html?pid='+p.pid'" style='color:green'>{{p.pname}}</a></p>
	<p><font color="#FF0000">商城价：&yen;{{p.shop_price}}</font></p>
</div>

</div>
	<!--分页 -->
	<div style="width:380px;margin:0 auto;margin-top:50px;">
		<ul class="pagination" style="text-align:center; margin-top:10px;"></ul>
</div>
	<!-- 分页结束=======================        -->

<script>
		//发送请求 根据cid查询-->HM.getParameter从请求路径中获取 参数
		let cid=HM.getParameter("cid");
		if(cid==null){
			alert("兄嘚,你确定要直接打开该页面吗?")
		}
		
		new Vue({
			el:"#products",
			data:{
				pb:{}
			},
			created(){
				//发送两个参数
				//一个cid 一个pageNumber
				let pageNumber=HM.getParameter("pageNumber")==null?1:HM.getParameter("pageNumber");
				
				let paramsObj={
					cid:cid,
					pageNumber:pageNumber
				}
				
				let params =$.param(paramsObj)
				
				axios.post("/product/page",params)
				.then((response)=>{
					let result=response.data;
					
					this.pb=result.bizData;
					//拼接下方的工具条
					
					let html=HM.page(this.pb,"http://www.itheima370.com:8020/web/view/product/list.html?cid="+cid)
					
					$(".pagination").html(html)
				})
				
			}
			
		})
		
</script>

```

#### 后端

```
准备:引入PageBean类,封装 data 数据集合,总商品数,每页展示个数,计算总共需要多少页,当前页,分页栏的第一个页码start,分页栏的最后一个页码和一个showNum的方法展示分页栏
1.在ProductServlet中 创建page方法 来进行分页查询操作
2.servlet中, 从请求中获取传递的cid 和 pageNumber, 自定义每页展示多少 pageSize,传递给业务
3.service中,findPage方法中拿到cid,pageNumber,pageSize,创建一个pageBean对象 将pageNumber,pageSize存入其中
4.dao中,page方法中,sql语句中,根据传来的cid分页查询limit?,?;
```

代码如下

```java
  public void page(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        //前端带来 cid 和 pageNumber;
        String cid = req.getParameter("cid");
        int pageNumber = ObjectUtil.mustInt(req.getParameter("pageNumber"));
        int pageSize = 3;

        //查询数据即可
        //一次性返回pageBean
        PageBean pb = productService.findPage(cid, pageNumber, pageSize);

        ok(pb);
    }

public PageBean findPage(String cid, int pageNumber, int pageSize) {
        //包含了 当前页的数据部分 和当前页码 总页数
        // 创建pageBean对象,保存Product对象
        PageBean<Product> pageBean = new PageBean<>();
        //pageBean中设置相应的pageNumber,pageSize
        pageBean.setPageNumber(pageNumber);
        pageBean.setPageSize(pageSize);

        //需要查询数据库
        List<Product> products = productDao.page(cid, pageNumber, pageSize);
        pageBean.setData(products);

        return pageBean;
}


public List<Product> page(String cid, int pageNumber, int pageSize) {
        QueryRunner queryRunner = new QueryRunner(JDBCUtil.getDataSource());

        String sql = "select * from product where cid=? limit ?,?";

        try {
            return queryRunner.query(
                    sql,
                    new BeanListHandler<>(Product.class),
                    cid,
                    (pageNumber - 1) * pageSize,
                    pageSize
            );
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
}
```

### 七 购物车

#### 准备购物车[在后端]:

```
1.首先明确的是 ,这个购物车 我们并不打算 保存到数据库中,而是保存到内存session中,当服务器关闭,也就找不到这个购物车了.
2.创建Cart购物车 充血式javaBean之前,我们需要明确,购物车里会放那些东西
①{"1":购物项,"2":购物项}-->前端发送来的购物车肯定就是json格式的购物车,所以还需要一个购物项javaBean
②在购物项CartItem中,有产品 以及 产品对应的 数量
③在Cart中,序列化购物车,保存数据
④用HashMap来保存 购物项,方便取出;用total[double类型]来计量合集多少钱;创建Date对象,记录创建购物车的时间[没啥用]
3.getItems方法 使用Collection集合来存储下来,查询到的购物项的内容
4.getTotal来拿到总共花了多少钱
5.添加购物项方法-->代码中逻辑很清楚
6.删除购物项方法-->map集合调用remove方法,更具产品pid来删
7.清空购物车方法-->map集合调用cleat方法
8.以上几个方法都需要在方法结束时,重新对total总金额进行计算
9.[多余]setget 购物车创建Date方法
```

#### 前端

```
1.购物车这部分只需要编写cart文件夹下的list.html页面
2.常规操作,绑定一个div,创建一个Vue,回显方法发送/cart/my请求,在该绑定的地方,就是用双向绑定的{{}},并且,修改发送请求的路径.具体看代码
3.删除方法,传递pid,发送删除请求后,再发送查询自己购物车的请求,及时反馈给页面回显删除后的结果
4.清空购物车方法,发送清空请求后,再发送查询自己购物车的请求,及时反馈给页面清空后的结果
```

```html
			<div id="cart" class="container" style="min-height: 441px;">
				<div class="row">
					<div style="margin:0 auto; margin-top:10px;width:950px;">
						<strong style="font-size:16px;margin:5px 0;">购物车详情</strong>
						<table class="table table-bordered">
							<tbody>
								<tr class="warning">
									<th>图片</th>
									<th>商品</th>
									<th>价格</th>
									<th>数量</th>
									<th>小计</th>
									<th>操作</th>
								</tr>
							
								<tr  v-for="i in cart.items" class="active">
									<td width="60" width="40%">
										<img :src="'http://www.itheima370.com:8020/web/'+i.product.pimage" width="70" height="60">
									</td>
									<td width="30%">
										<a target="_blank">{{i.product.pname}}</a>
									</td>
									<td width="20%">
										￥{{i.product.shop_price}}
									</td>
									<td width="10%">
										{{i.count}}
									</td>
									<td width="15%">
										<span class="subtotal">￥{{i.subtotal}}</span>
									</td>
									<td>
										<a href="javascript:;" @click="del(i.product.pid)" class="delete">删除</a>
									</td>
								</tr>
							</tbody>
						</table>
					</div>
				</div>
	
				<div style="margin-right:130px;">
					<div style="text-align:right;">
						商品金额: <strong style="color:#ff6600;">￥{{cart.total}}元</strong>
					</div>
					<div style="text-align:right;margin-top:10px;margin-bottom:10px;">
						<a href="javascript:;" @click="clear">清空购物车</a>
						<a href="javascript:;" @click="generate">
							<input type="button" width="100" value="提交订单" id="submit" border="0" style="background-color:#CD062D;height:35px;width:100px;color:white;">
						</a>
					</div>
				</div>
			</div>



	<script>
		new Vue({
			el:"#cart",
			data:{
				cart:{}
			},
			created(){
				//发请求, 请取自己的购物车对象
				axios.post("/cart/my")
				.then((response)=>{
					let result=response.data;
					
					this.cart=resutl.bizData;					
				})				
			},
			methods:{
				del(pid){
					//弹出确认框
					if(confirm("您确认要删除该购物项?")){
						//发送请求 删除该购物项
						axios.post("/cart/del","pid="+pid)
						.then((response)=>{
							//location.reload();
							axios.post("/cart/my")
							.then((response)=>{
								let result=response.data;
								
								this.cart=result.bizData;
								
							})
							
						})						
					}
				},
				clear(){
					if(confirm("您确认要清空 该购物车吗?")){
						axios.post("/cart/clear")
						.then((response)=>{
							
							axios.post("/cart/my")
							.then((response)=>{
								let result = response.data;
								this.cart = result.bizData;
							})
						})
					}
				}
				//订单
				generate(){
					//发送请求即可
					axios.post("/order/generate")
					.then((response)=>{
						let result=response.data;
						if( result.code==CODE_NOT_LOGIN ){
							
							//跳转 我得获取当前页面地址
							//作为参数 传递给登录页面地址栏 
							let currentUrl = location.href
							currentUrl=encodeURIComponent(currentUrl);
							
							location.href="../../login.html?returnUrl=" + currentUrl;
							
						}else{
							alert("生成订单成功")
						}
					})
					
				}
			}
		})
		
	</script>
```

#### 后端代码

```java
public class CartItem {
    private Product product;
    private int count;

    public Product getProduct() {
        return product;
    }

    public void setProduct(Product product) {
        this.product = product;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }

    public double getSubtotal() {
        return this.product.getShop_price()*count;
    }


}




public class Cart implements Serializable{
    /**
     *
     * {
     *     "1":购物项,
     *     "2":购物项
     * }
     *
     *
     */
    private Map<String,CartItem> items=new HashMap();
    private double total;
    private Date createdDate=new Date();


    public Collection<CartItem> getItems() {
        return items.values();
    }

    public double getTotal() {
        return total;
    }

    //添加购物项
    public void add(CartItem item){
        //判断该种类商品是否已经在购物车存在了
        String pid = item.getProduct().getPid();

        //存在则 数量变化
        if (items.containsKey(pid)){
            //将原来的数量改变就行
            CartItem _cartItem = items.get(pid);
            _cartItem.setCount(_cartItem.getCount()+item.getCount());
        }else{
            //不存在 则直接放入购物车中
            items.put(pid,item);
        }

        //总金额?
        total+=item.getSubtotal();

    }
    //删除购物项
    public void remove(String pid){
        CartItem remove = items.remove(pid);

        total-=remove.getSubtotal();
    }
    //清空购物车
    public void clear(){
        items.clear();
        total=0.0;
    }

    public Date getCreatedDate() {
        return createdDate;
    }

    public void setCreatedDate(Date createdDate) {
        this.createdDate = createdDate;
    }
}



@WebServlet({
        "/cart/my",
        "/cart/del",
        "/cart/clear"
})
public class CartServlet extends BaseServlet {
    private ProductService productService = BeanFactory.getBean(ProductService.class);

    public void my(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        ok(getMyCart());
    }

    public void del(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        //请求传来pid
        String pid = req.getParameter("pid");
        //拿车,扔出去
        Cart myCart = getMyCart();
        myCart.remove(pid);

        ok();
    }

    public void clear(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        //获取自己的车
        getMyCart().clear();
        ok();
    }


    private Cart getMyCart() {
        //只有一个购物车
        HttpServletRequest request = requestThreadLocal.get();
        HttpSession session = request.getSession();

        //检查一下session中有没有存在购物车
        Cart myCart = (Cart) session.getAttribute(CONSTANTS.CART_NAME);

        if (myCart == null) {
            myCart = new Cart();
            //放入到自己的session中
            session.setAttribute(CONSTANTS.CART_NAME, myCart);
        }
        return myCart;
    }


}

```

### 八 向购物车中添加商品

#### 前端

```
1.在into.html页面中,有具体的商品信息,和添加购物车按钮,
我们在Vue的data中添加count对象,用来统计购买的数量,
2.在相应的位置修改count的双向绑定
3.添加methods @click添加购物车按钮
方法中传递当前商品的pid ,用来查找该商品;
传递count购买商品的数量
4.axios发送/cart/add给后端,直接跳转页面到购物车页面
```

前端代码

```html
							<div style="padding:10px;border:1px solid #e7dbb1;width:330px;margin:15px 0 10px 0;;background-color: #fffee6;">
								<div style="border-bottom: 1px solid #faeac7;margin-top:20px;padding-left: 10px;">购买数量:
									<input v-model="count"  value="1" maxlength="4" size="10" type="text"> </div>
	
								<div style="margin:20px 0px 10px 0px;padding-left: 70px;">
									<a @click="addItem" href="javascript:;">
										<input style="background: url('http://www.itheima370.com:8020/web/resources/img/product.gif') no-repeat scroll 0 -600px rgba(0, 0, 0, 0);height:36px;width:127px;" value="加入购物车" type="button">
									</a> 
								</div>
							</div>



	new Vue({
			el:"#product",
			data:{
				product:{},
				count:1
			},
			created(){
				
				axios.post("/product/findById","pid="+pid)
				.then((response)=>{
					let result = response.data;
					this.product=result.bizData
				})				
			},
			methods:{
				addItem(){
					
					//一个商品id  一个购买数量
					let params={
						pid:pid,
						cont:this.count
					}
					
				let param=$.param(params);
				axios.post("/cart/add",param)
				.then((response)=>{
					let result=response.data;
					if(result.code==1){
						location.href="../cart/list.html";
					}
				})
					
				}
			}
		})
```

#### 后端

```
1.在servlet中的add方法里,把pid和count拿到手
2.根据pid从数据库中查询product对象  
3.传给购物项[购物项需要 count 和 product]
4.购物车有个add方法,把购物项加入购物车,返回ok
```

后端代码

```java
public void add(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        //获取提交的pid 和count;
        String pid = req.getParameter("pid");
        int count= ObjectUtil.mustInt(req.getParameter("count"),1);

        //构成一个购物项
        CartItem cartItem = new CartItem();
        cartItem.setCount(count);
        //查询一个商品对象
        Product product = productService.findById(pid);
        cartItem.setProduct(product);


        //购物项添加到自己的购物车中
        Cart cart = getMyCart();

        cart.add(cartItem);


        //返回成功!
        ok();

    }
```

### 九 购物车发送提交订单功能

#### 前端

```
1.在购物车页面中,有提交订单按钮, 绑定方法 generate,用来将提交订单
2.generate这个方法在使用的时候需要先确认用户是否登录,如果没有登录,那么按下按钮就直接跳转页面至登录页面,登录成功后再直接跳回来
3.具体操作如下,
①先判断有没有登录,登陆了就直接弹出"订单生成弹框"
②如果没有登录,就获取当前购物车的地址栏中的地址location.href,然后使用encodeURIComponent方法对获取的地址进行解码,
③然后跳转到首页再在首页的地址后面拼接字符串示例如下
							location.href="../../login.html?returnUrl=" + currentUrl;
4.但是我们login页面的登录功能把跳转页面写死了,所以需要去login.html页面进行js的修改
①在login功能下,将写死的地址 改为判断 从地址栏中获取参数,如果参数为null,那么登陆成功后就直接跳转首页,如果不为null,那么首先将获取到的参数进行解码操作,然后在跳转到响应的地址页面
```

cart.html购物车页面

```html
	<a href="javascript:;" @click="generate">
							<input type="button" width="100" value="提交订单" id="submit" border="0" style="background-color:#CD062D;height:35px;width:100px;color:white;">
						</a>

//订单
generate(){
//发送请求即可
axios.post("/order/generate")
.then((response)=>{
let result=response.data;
if( result.code==CODE_NOT_LOGIN ){

//跳转 我得获取当前页面地址
//作为参数 传递给登录页面地址栏 
let currentUrl = location.href
currentUrl=encodeURIComponent(currentUrl);

location.href="../../login.html?returnUrl=" + currentUrl;

}else{
alert("生成订单成功")
}
});
}


```

login.html页面

```html
login(){
//提交当前user数据
let params=$.param(this.user)
					
let f=(response)=>{
	//箭头函数体内 是没有调用者 这个说法的
	//捕获声明上下文中的this
	console.log(this);
	let result=response.data;
					
	if(result.code==0){
		this.error=result.bizData;
	}else{
		//成功跳转首页
		//location.href="index.html";													
		
		//如果需要直接跳转,那么久改为一下内容
		let destUrl = HM.getParameter("returnUrl");
		if(destUrl==null){
			location.href="index.html";
		}else{
			destUrl=decodeURIComponent(destUrl);
			
			location.href=destUrl;
		}
		
	}					
}
					
axios.post("/user/login",params)
.then(f);
				}
			}
```

#### 后端

```
1.加入两个javaBean类对象 一个订单类 Order 一个订单项类OrderItem
2.servlet中, generate方法里面,因为无论购物车还是登陆信息都保存在了session里面,所以就从session中先拿到登陆信息,因为订单应该有一个对应的user来操作
3.如果未登录,就返回一个未登录状态信息给前端,好让前端页面进行判断和跳转
4.有人登陆,就获取该人自己的购物车-->在这一页中,也设置一个getCart方法,以便拿到session中的购物车
5.把购物车中的商品选中,保存到集合中
6.将购物车里的购物项转为订单中的订单项 
7.创建一个ArrayList集合保存这些订单项
8.后面的操作重复劳动,注意的是,需要在业务层分开订单和订单项的保存,因为两个保存的内容其实不太一样
```

代码如下

```java
@WebServlet("/order/generate")
public class OrderServlet extends BaseServlet {
    private OrderService orderService = BeanFactory.getBean(OrderService.class);

    public void generate(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        //先保证 有人处于登录状态
        User user = (User) req.getSession().getAttribute(CONSTANTS.SESSION_USER_NAME);

        if (user == null) {
            writeResult(new Result(CONSTANTS.CODE_NOT_LOGIN, "未登录", null));
            return;
        }
        //保存订单信息
        // 获取自己的购物车
        Cart myCart = getMyCart();
        //把购物车里的商品选中,保存到一个集合中
        Collection<CartItem> cartItems = myCart.getItems();
        if (cartItems.size() == 0) {
            no("购物车空空如也!!");
            return;
        }

        //创建订单 购物车-->订单
        Order order = new Order();
        order.setOrdertime(new Date());
        order.setUid(user.getUid());
        order.setTotal(myCart.getTotal());
        order.setState(CONSTANTS.ORDER_STATE_WEIFUKUAN);
        String oid = UUIDUtil.randID();
        //将购物项集合转换成订单项集合
        //为什么使用ArrayLIst-->因为好遍历保存
        List<OrderItem> orderItems = new ArrayList<>();

        for (CartItem cartItem : cartItems) {
            //遍历购物车项,保存到订单集合里面
            OrderItem orderItem = new OrderItem();
            orderItem.setOid(oid);
            orderItem.setPid(cartItem.getProduct().getPid());

            orderItem.setCount(cartItem.getCount());
            orderItem.setSubtotal(cartItem.getSubtotal());

            orderItems.add(orderItem);
        }

        //调用service 完成保存
        orderService.save(order,orderItems);

        ok();
    }

    public Cart getMyCart() {
        //只有一个车
        HttpServletRequest request = requestThreadLocal.get();
        HttpSession session = request.getSession();
        //看看有没有车
        Cart mycart = (Cart) session.getAttribute(CONSTANTS.CART_NAME);
        if (mycart == null) {
            mycart = new Cart();
            session.setAttribute(CONSTANTS.CART_NAME, mycart);
        }
        return mycart;
    }
}



public class OrderServiceImpl implements OrderService {
    private OrderDao orderDao = BeanFactory.getBean(OrderDao.class);

    @Override
    public void save(Order order, List<OrderItem> orderItems) {
        //保存一个订单
        orderDao.saveOrder(order);

        //将订单项保存到订单集合里面去
        for (OrderItem orderItem : orderItems) {
            orderDao.saveItem(orderItem);
        }
    }
}


public class OrderDaoImpl implements OrderDao {
    @Override
    public void saveOrder(Order o) {
        QueryRunner queryRunner = new QueryRunner();

        String sql = "insert into orders values(?,?,?,?,?,?,?,?)";

        try {
            queryRunner.update(                    
                    sql,
                    o.getOid(),o.getOrdertime(),o.getTotal(),o.getState(),
                    o.getAddress(),o.getName(),o.getTelephone(),o.getUid()
            );
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public void saveItem(OrderItem oi) {
        QueryRunner queryRunner = new QueryRunner();

        String sql = "insert into orderitem values(?,?,?,?)";

        try {
            queryRunner.update(
	                DatasourceUtil.getConnection(),
                    sql,
                    oi.getCount(),oi.getSubtotal(),oi.getPid(),oi.getOid()

            );
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}


```

### 十 订单页面的功能完善

#### ①完善generate方法

````
1. cart.html中的提交订单功能中,改进,提交完订单,就跳转到订单页面去[order文件夹下的info.html].
2.与此同时,需要改进generate方法-->是后端servlet路径下的generate方法 所以直接前后端一起改动
3. 首先对应 提交订单来说,提交的订单会 直接扣除库存,那么业务逻辑中,就需要进行判断库存的情况,但是这是业务的逻辑,在Orderservlet
中不需要展示,只需要抛出库存不出的异常,将考虑库存的情况放在业务逻辑层OrderService当中
4.当库存充足时,就保存订单和订单项 -->因为保存的时候会进行两个以上的操作,所以需要事务来同步管理操作,保持一致性
5.当库存不足时,就回滚操作,抛出库存不足异常-->这个自定义的异常
````

```html
//订单
generate(){
//发送请求即可
axios.post("/order/generate")
.then((response)=>{
let result=response.data;
if( result.code==CODE_NOT_LOGIN ){

//跳转 我得获取当前页面地址
//作为参数 传递给登录页面地址栏 
let currentUrl = location.href;
currentUrl=encodeURIComponent(currentUrl);

location.href="../../login.html?returnUrl=" + currentUrl;

}else{
//跳转订单页面
let oid=result.bizData;

location.href="../order/info.html?oid="+oid;
}
});

}
```

```java
OrderServlet当中  generate方法
//调用service 完成保存

        try {
            orderService.save(order,orderItems);
            ok(oid);
        } catch (KuCunException e) {
            no("库存不足");
        }


OrderService中
@Override
    public void save(Order order, List<OrderItem> orderItems) throws KuCunException {

        try {
            //检查一下 库存是否充足
            //开启事务-->因为保存的时候会进行两个以上的操作,所以需要事务来同步管理操作,保持一致性
            //保存一个订单
            orderDao.saveOrder(order);

            //将订单项保存到订单集合里面去
            for (OrderItem orderItem : orderItems) {
                orderDao.saveItem(orderItem);
            }
        } catch (Exception e) {
            //犯错误就回滚
            DatasourceUtil.rollback();
            //抛出异常
            throw new RuntimeException(e);
        }
    }

```

#### ②查询订单方法的编写

```
1.在order文件夹下的info.html页面中, 绑定id为order的div ,new VUe 创建order对象来保存订单
2.到此页面,直接回显,所以需要回显订单的相应的订单号-->从地址栏中获取订单号参数oid
3.axios.post("/order/findById","oid="+oid)路径中请求到后端,才从后端返回来的订单中拿到相应的订单信息
4.来到后端的orderServlet中,从请求中拿到oid的值,创建一个findByIdWithItems方法,来获得相应的订单中的具体的订单项信息
5.沿着这个findByIdWithItems,走到service当中,业务当中查询相应的订单项信息,需要订单项信息,也需要将订单项信息封装在一个OrderItemVo对象中,这个对象可以很好的用来展示在前端页面中.[查询订单项信息,只在查询表中,而查询信息封装在OrderItemVo中,需要订单表和产品表联查]
6.记住要导入一个OrderItemVo的javaBean对象,还有在Order的javaBean对象中,添加List<OrderItemVo> orderItemVos 成员变量,以及相应的get set方法

```

具体的html中的内容就不展示了

```html
<script >
		let oid=HM.getParameter("oid");
		if(oid==null){
			alert("兄弟,你是不是直接打开的这个页面")
		}
		
		new Vue({
			el:"#order",
			data:{
				order:{}
			},
			created(){
				
				axios.post("/order/findById","oid="+oid)
				.then((response)=>{
					
					let result=response.data;
					this.order=result.bizData;
					
				});				
            }
			
		});
		
	</script>
```

后端代码servlet中

```java
 public void findById(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        String oid = req.getParameter("oid");

        Order order = orderService.findByIdWithItems(oid);
        ok(order);
    }
```

在service中

```java
  @Override
    public Order findByIdWithItems(String oid) {
        Order order = orderDao.findById(oid);

        //继续查询 订单项!!!
        List<OrderItemVo> vos = orderDao.findByIdItemVos(oid);

        order.setOrderItemVos(vos);

        return order;
    }

```

在dao中

```java
@Override
    public Order findById(String oid) {
        QueryRunner queryRunner = new QueryRunner(DatasourceUtil.getDataSource());

        String sql = "select * from orders where oid=?";

        try {
            return queryRunner.query(	               
                    sql,
                    new BeanHandler<>(Order.class),
                    oid
            );
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

@Override
    public List<OrderItemVo> findByIdItemVos(String oid) {
        QueryRunner queryRunner = new QueryRunner(DatasourceUtil.getDataSource());

        String sql = "SELECT oi.*,p.pname,p.shop_price AS price,p.pimage AS img FROM orderitem oi,product p WHERE oi.pid=p.pid AND oid=?";

        try {
           return queryRunner.query(
                    sql,
                    new BeanListHandler<>(OrderItemVo.class),
                    oid
            );
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
```

#### ③收货人信息的提交

```
1.前端继续在方法里面添加confirmOrder方法,将用户输入的信息保存好,把收货人信息封装之后发给后端,进行保存操作
2.进入OrderServlet当中,前端发过来的收货人信息比较碎,用getParameterMap来保存,建立一个订单对象order来保存这些信息,
使用BeanUtils.populate将收货人信息封住好
3.创建updateReceiver方法来向数据库中的order表中存储收货人信息.
4.创建findById方法,来查询对应订单号的订单信息,然后调用阿里支付工具类,把需要的订单编号,备注信息,以及订单当中的总计金额进行提交
5.导入阿里支付工具的同时,记得在pom文件中添加阿里支付的依赖
```

html页面中的编写省略

```html
<script>
    confirmOrder(){
					//订单信息 提交给服务器
					let paramsObj={
						oid:this.order.oid,
						name:this.order.name,
						telephone:this.order.telephone,
						address:this.order.address	
					}
					
					let params=$.param(paramsObj);
					axios.post("/order/confirm",params)
					.then((response)=>{
						
						let result=response.data;
						//这段代码实际上是一个form表单
						let html=result.bizData;
						
						$("#topay").html(html);
						
						
					})
					
				}
</script>
```

后端servlet

```java
 public void confirm(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        //接收某订单, 收货人信息
        Map<String, String[]> parameterMap = req.getParameterMap();
        Order order = new Order();
        try {
            /*
            * 首先，它是在org.apache.commons.beanutils.BeanUtils包中的一个方法。
方法的作用：用来将一些 key-value 的值（例如 hashmap）映射到 bean 中的属性。

servlet中有这样的使用：
先定义form表单内容的Info对象(当然你要先写一个bean,这个bean中包含form表单中各个对象的属性)
    InsuranceInfo info = newInsuranceInfo();   （这是一个javabean）
    BeanUtilities.populateBean(info, request);
——> populateBean(info,request.getParameterMap());（先将request内容转为Map类型）
——>BeanUtils.populate(info,propertyMap);（调用包中方法映射）

映射的过程就是将页面中的内容先用request获得，然后再将之转换为Map(这里用request.getParameterMap()）
最后使用BeanUtils.populate(info，map)方法将页面各个属性映射到bean中。之后我们就可以这样使用bean.getXxxx()来取值了。

            * */
            BeanUtils.populate(order, parameterMap);
        } catch (Exception e) {
            e.printStackTrace();
        }

        //调用service 把这个收货人信息,保存了再说
        orderService.updateReceiver(order);

        Order order1 = orderService.findById(order.getOid());

        try {
            String s = AlipayUtil.generateAlipayTradePagePayRequestForm(order.getOid(), "随便", order1.getTotal());
            ok(s);
        } catch (AlipayApiException e) {
            e.printStackTrace();
            no("出现一些异常 稍后再试");
        }

    }
```

 因为之前已经在dao当中定义过查询订单的方法,所以dao中的方法相对来说就添加一个

```java
 @Override
    public void updateReceiver(Order order) {
        //保存收货人信息
        QueryRunner queryRunner = new QueryRunner(DatasourceUtil.getDataSource());


        String sql="update orders set name=? ,address=?,telephone=? where oid=?";


        try {
            queryRunner.update(
				
                    sql,
                    order.getName(),order.getAddress(),order.getTelephone(),order.getOid()
            );

        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }


```

#### ④用阿里支付

```
1.登录支付宝开发者,->开发服务->研发服务-.沙箱应用->信息配置部分
2.使用阿里私钥工具,生成自己的私钥0和公钥1,再在信息配置部分给阿里自己的公钥1,拿到阿里给的公钥2
3.向resources文件夹中添加alipay_config.properties配置文件,添加相应的信息
4.后端Orderservlet中"/order/callback"请求路径,具体方法如下
```

```java
public void callback(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        //第一反应是校验 这个请求是不是 支付宝发给你的!!!!

        boolean check = AlipayUtil.check(req.getParameterMap());

        if (check){
            //现在的要做的事情 获取订单编号
            //将其状态改为已支付
            String oid = req.getParameter("out_trade_no");

            orderService.updateState(oid,Constants.ORDER_STATE_YIFUKUAN);

            //完事了?

            resp.sendRedirect("http://www.itheima370.com:8020/heima370/view/order/info.html?oid="+oid);

        }else{
            resp.getWriter().print("<div style='font-size:88px;text-align:center'>滚!</div>");
        }

    }
    
```

配置文件如下

```properties
#appid 就是咱们申请应用的id
APPID=
# 就是跳转支付宝的网关 网址
# 沙箱环境地址跟真实环境是有区别的
serverUrl=https://openapi.alipaydev.com/gateway.do
#自己的私钥
privateKey=
#自己的公钥
publicKey=
#阿里给你公钥
aliPublicKey=
#数据格式
format=json
#字符集设置

charset=utf-8
# 有一个加密算法叫做rsa rsa升级了 rsa2
signType=RSA2
# 浏览器回调地址
returnUrl=http://www.itheima370.com:8080/order/callback
# 异步回调地址
notifyUrl=http://192.168.32.100:80/order/notify
```

