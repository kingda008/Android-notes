# 搭建服务器

**1.下载Tomcat,解压**

**2.菜单栏 window 的 Prefernces 选项**

**3.Prefernces -> Server -> RuntimeEnvironments -> 点击右边的 add**

![image-20221021153019892](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021153019892-16663374215761.png)

**4.加载tomcat**

![image-20221021153059932](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021153059932.png)





**5.创建工程。右击 -> new -> Dynamic Web Project -> 请看下图**

 

**6，显示server** 

window - show view - other - 选择server

![image-20221021153313833](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021153313833-16663375964542.png)

 使用tomcat 

![image-20221021153402369](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021153402369-16663376442223.png)



**7.测试发布 -> 请看下图 -> 右击tomcat -> 点击star**



如果端口被占用，可以通过netstat -ano|findstr 8081 查看哪个进程占用的

浏览器输入 ip：8080 如果能看到tomcat 说明配置正确



![image-20221021154554074](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021154554074-16663383562494.png)





tomcat默认8080端口



**8.新建web工程**  



new  一个 Dynamic Web Project 工程。

![image-20221021154917876](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021154917876.png)



在WebContent 根目录下新建一个html文件



**9，将新建的web工程发布到tomcat**

![image-20221021155139039](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021155139039-16663386999065.png)

右键 Add and Remove。添加我们刚刚新建的工程 

右键restart 进行发布









用浏览器打开网页

http://本机地址:8080/HttpServer/test.html

就能看到结果





# **测试servlet接口**



![image-20221021161258583](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021161258583-16663399807646.png)





```java
package com.baoge_server;

import java.io.File;
import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/TestServlet")
public class TestServlet extends HttpServlet{

	private static final long serialVersionUID = 1L;
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		doPost(req, resp);
	}
	
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		PrintWriter printWriter = resp.getWriter();
		System.out.println("Request");
		printWriter.write("test post");
		printWriter.flush();
		printWriter.close();
	}
	
	@Override
	protected long getLastModified(HttpServletRequest req) {
		// 这个文本有没有更新过
		File file  = new File(req.getContextPath(),"index.html");
		return file.lastModified();
	}
}

```

然后在浏览器中输入 http://192.168.1.59:8080/HttpServer/TestServlet 

可以看到浏览器结果和tocat打印

![image-20221021162610356](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021162610356.png)



谷歌浏览器 网页上右键-检查 会弹出调试界面

![image-20221021163133551](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021163133551-16663410946367.png)

再回车，可以看到报文

![image-20221021163319439](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221021163319439-16663412020438.png)





## cookie

```java
package com.baoge_server;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.alibaba.fastjson.JSON;
import com.baoge_server.response.entity.ResponseEntity;
import com.baoge_server.response.entity.UserInfoEntitiy;
@WebServlet("/LoginServlet")
public class LoginServlet extends BaseJsonServlet{
	
	private static final long serialVersionUID = 1L;

	@Override
	protected ResponseEntity onHandler(HttpServletRequest req, HttpServletResponse resp) throws Exception {
		
		
		ResponseEntity responseEntity = new ResponseEntity();
		responseEntity.code = "0022";
		responseEntity.msg = "用户名或密码错误";
		
		String userName = req.getParameter("userName");
		if("Darren".equals(userName)){
			responseEntity.code = "0000";
			responseEntity.msg = "登录成功";
			
			UserInfoEntitiy infoEntitiy = new UserInfoEntitiy();
			infoEntitiy.userName = userName;
			infoEntitiy.userSex = "男";
			responseEntity.data = JSON.toJSONString(infoEntitiy);
			// 设置登录 Cookie 登录有效时间
			Cookie cookie = new Cookie("userName", "Darren");
			cookie.setMaxAge(10); // 设置Cookie的过期时间
			resp.addCookie(cookie);
		}
		
		
		return responseEntity;
	}

}

```

浏览器输入：http://192.168.1.59:8080/HttpServer/LoginServlet?userName=Darren

![image-20221022110026098](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221022110026098-16664076272389.png)
服务端应答报文中会放回cookie，客户端在**有效时间内**将返回的cookie放到访问报文中，不在有效范围内则不带上cookie

![image-20221022110313218](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221022110313218-166640779504310.png)



![image-20221022110359521](02_%E6%90%AD%E5%BB%BAhttp%E6%9C%8D%E5%8A%A1.assets/image-20221022110359521-166640784059211.png)

使用效果可以看下面评论的例子，用户评论的时候会判断cookie是否在有效期内，如果在则评论成功，否则失败

```java
@WebServlet("/UserCommentServlet")
public class UserCommentServlet extends BaseJsonServlet{
	
	private static final long serialVersionUID = 1L;

	@Override
	protected ResponseEntity onHandler(HttpServletRequest req, HttpServletResponse resp) throws Exception {
		// 判断该用户有没有登录，没有登录或者登录过期都不让评论
		
		ResponseEntity responseEntity = new ResponseEntity();
		responseEntity.code = "0033";
		responseEntity.msg = "评论失败，用户未登录";
		
		Cookie[] cookies = req.getCookies();
		if(cookies!=null){
		for(Cookie cookie : cookies){
			String userName = cookie.getName();
			if(userName != null){
				responseEntity.code = "0000";
				responseEntity.msg = "评论成功";
				return responseEntity;
			}
		}
		}
		
		return responseEntity;
	}

}

```

用户先登录 http://192.168.1.59:8080/HttpServer/LoginServlet?userName=Darren ，10秒内访问

http://192.168.1.59:8080/HttpServer/UserCommentServlet 则评论成功，否则失败







## Http缓存

Cache-Control（缓存策略）：Public、private、no-cache、max-age  、no-store（不缓存）

Expires（缓存的过期策略）：指名了缓存数据有效的绝对时间，告诉客户端到了这个时间点（比照客户端时间点）后本地缓存就作废了，

在这个时间点内客户端可以认为缓存数据有效，可直接从缓存中加载展示。

https://mp.weixin.qq.com/s/qOMO0LIdA47j3RjhbCWUEQ

如果有缓存并且过期了那么发起请求，那么服务端会给我们数据？（不一定会给）服务器的数据没有变动就不会给，状态码会变为 304  ，自己拿之前过期的缓存