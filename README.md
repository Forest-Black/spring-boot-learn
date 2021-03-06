# spring-boot-learn



是基于maven多模块工程来记录学习springboot的知识的一个过程



目录：

- ## spring-boot-learn-word

  针对word的操作：

  - spire.doc.free 简单示例
  - itextpdf实现对生成后的PDF添加页码示例

  

- ## spring-boot-learn-excel

  针对excel的操作：

  - hutool工具实现对excel导入导出的示例



- ## spring-boot-learn-crawler-webmagic

  简单学习WebMagic，是一个简单灵活的Java爬虫框架

  - 使用webmagic爬取了笔趣阁小说类别
  - 爬取了笔趣阁小说《剑来》目录



- ## spring-boot-learn-mybatis

  spring boot整合mybatis持久层框架


第一步：添加spring boot和mybatis的依赖，可以根据实际情况进行修改版本

~~~xml

<!-- Spring Boot 整合的 mybatis 的依赖jar包，
其中包含mybatis相关的依赖 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>

<!-- MySql驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

~~~

第二步：application.yml文件添加配置
分为数据源配置 + mybatis的配置

~~~yaml

## 数据源的配置
spring:
  datasource:
    username: root
    password: mysql
    ## 新版本MySQL驱动链接URL必须添加时区配置 serverTimezone=UTC
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
    driver-class-name: com.mysql.jdbc.Driver
    
   
## mybatis的配置
mybatis:
  ## 配置mybatis的总配置文件路径
  config-location: classpath:mybatis/mybatis.xml
  ## 配置文件各个mapper xml文件的路径
  mapper-locations: classpath:mybatis/mapper/*.xml
  ## 配置取别名的实体类包全路径
  type-aliases-package: com.tianya.springboot.mybatis.entity
  
  
~~~

第三步：在启动类上添加mapper接口扫描路径

~~~java

@SpringBootApplication
// 自动扫描mapper接口全路径
@MapperScan("com.tianya.springboot.mybatis.mapper")
public class SpringMybatisApplication {
	
	public static void main(String[] args) {
		SpringApplication.run(SpringMybatisApplication.class, args);
	}

}
~~~

第四步：在接口上添加mapper注解

~~~java
// mapper是mybatis的注解，定义一个mapper接口
// 也会被添加到spring bean容器动
// 若是只用@Repository注解，不会有mybatis的自动扫描配置
// 若是配置了SqlSessionFactory，也是可以的
@Mapper
public interface IUserMapper {
	public List<UserBean> getUserList() ;
}

~~~

第五步：在mapper xml文件具体实现的SQL的方法

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace必须是mapper接口的全路径 -->
<mapper namespace="com.tianya.springboot.mybatis.mapper.IUserMapper">
    <!-- id必须是mapper接口中定义的方法名，不可重载 -->
    <select id="getUserList" resultType="UserBean">
        select * from users
    </select>
</mapper>

~~~



- ## spring-boot-learn-capture-screen
  使用服务端推送技术SSE+屏幕截屏，实现一个简单的屏幕共享功能
  - SseEmitter 实现服务端推送功能
  - java.awt.Toolkit 获取屏幕截屏

**获取屏幕截屏**

~~~java

/**
* 捕获屏幕，生成图片
*/
public static String capture() {

    // 获取屏幕大小
    Dimension screenSize = Toolkit.getDefaultToolkit().getScreenSize();
    String fileName = null ;
    try {

        Robot robot = new Robot();
        Rectangle screenRect = new Rectangle(screenSize);
        // 捕获屏幕
        BufferedImage screenCapture = robot.createScreenCapture(screenRect);
        //存放位置
        String path = ClassUtils.getDefaultClassLoader().getResource("static").getPath();
        fileName = System.currentTimeMillis() + ".png" ;
        LOGGER.info("屏幕截屏路径：{}", path);
        // 把捕获到的屏幕 输出为 图片
        ImageIO.write(screenCapture, "png", new File(path +File.separator + fileName));

    } catch (Exception e) {
        LOGGER.error("获取屏幕截屏发生异常", e);
    }

    return fileName ;
}


~~~



**服务端推送**

方式一：自己拼接返回参数值

~~~java
// 服务端推送 返回结果必须以 data: 开始
public static final String SSE_RETURN_START = "data:" ;

// // 服务端推送 返回结果必须以 \n\n 结束
public static final String SSE_RETURN_END = "\n\n" ;

// 服务端推送SSE技术 内容格式必须是  text/event-stream
@GetMapping(value = {"","/"}, produces = "text/event-stream;charset=UTF-8")
public String index() {
    String capture = CaptureScreenUtils.capture();
    // 返回的内容 必须以 data: 开头，以 \n\n 结尾，不然前端JS处始终不会进入到 onmessage 方法内
    return SSE_RETURN_START + capture + SSE_RETURN_END ;
}
~~~

方式二：使用Spring MVC已经封装的对象SseEmitter 来实现

~~~java

@GetMapping("/index")
public SseEmitter sse() {

    String capture = CaptureScreenUtils.capture();
    SseEmitter emitter = new SseEmitter(100L) ;
    try {
        emitter.send(capture);
    } catch (IOException e) {
        e.printStackTrace();
    }

    return emitter;
}
	
~~~

实际是封装的内部拼接的头尾

~~~java
// org.springframework.web.servlet.mvc.method.annotation.SseEmitter.SseEventBuilderImpl

@Override
public SseEventBuilder data(Object object, @Nullable MediaType mediaType) {
    append("data:");
    saveAppendedText();
    this.dataToSend.add(new DataWithMediaType(object, mediaType));
    append("\n");
    return this;
}
~~~

**前端页面JS**

~~~javascript
var source = new EventSource("/sse/index");
source.onmessage = function(event){
    console.log('onmessage');
    console.log(event.data);
    document.getElementById('captureImgId').src = event.data;
};
~~~







## spring-boot-learn-websocket



采用全注解方式实现 web socket聊天

第一步：首先引入websocket的依赖

~~~xml

<!-- spring boot WebSocket相关包 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
~~~



第二步：采用注解方式实现websocket服务端

~~~java
package com.tianya.springboot.websocket.server;

import java.io.IOException;
import java.util.LinkedHashSet;
import java.util.Objects;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.tianya.springboot.websocket.handle.AppIdMsgHandle;

import lombok.extern.slf4j.Slf4j;

/**
 * @description
 *	WebSocket 服务端
 * 每种应用下，对应的会话连接
 * <pre>key: appId 不同应用，
 * 如：
 * 	msg对应消息推送，
 * 	log对应日志推送等，
 * 	chat对应聊天信息推送</pre>
 * @author TianwYam
 * @date 2020年9月17日下午1:47:22
 */
@Slf4j
@Service
@ServerEndpoint(value = "/ws/{appId}")
public class WebSocketServer {
	
	// 自动注入消息处理的实现
	// @Autowired 在这儿自动注入会报空指针异常，取不到
	private static Set<AppIdMsgHandle> appIdMsgHandles ;

	/*** 每种应用下，对应的会话连接 */
	private static final ConcurrentHashMap<String, LinkedHashSet<Session>> APPID_CONNECT_SESSION_MAP = new ConcurrentHashMap<>();
	
	
	/**
	 * @description
	 *	session 连接 开始
	 * @author TianwYam
	 * @date 2020年9月17日下午2:27:12
	 * @param session 会话连接
	 * @param appId 应用ID
	 */
	@OnOpen
	public void open(Session session, @PathParam(value = "appId") String appId) {
		
		// 获取此应用下所有连接
		LinkedHashSet<Session> sessions = getSessions(appId);
		if (sessions == null) {
			sessions = new LinkedHashSet<>();
			APPID_CONNECT_SESSION_MAP.put(appId, sessions);
		}
		
		// 新会话session 连接成功
		sessions.add(session);
		log.info("新连接进入，目前[{}]总人数：{}", appId, sessions.size());
	}
	
	
	/**
	 * @description
	 *	session 连接关闭
	 * @author TianwYam
	 * @date 2020年9月17日下午3:32:45
	 * @param session
	 * @param appId
	 */
	@OnClose
	public void close(Session session, @PathParam(value = "appId") String appId) {
		
		// 获取此应用下所有连接
		LinkedHashSet<Session> sessions = getSessions(appId);
		if (sessions != null) {
			// 会话session 连接断开
			sessions.remove(session);
			log.info("连接断开，目前[{}]总人数：{}", appId, sessions.size());
		}
		
	}
	
	
	/**
	 * @description
	 *	接受消息 
	 * @author TianwYam
	 * @date 2020年9月17日下午3:39:22
	 * @param message 消息内容
	 * @param session 会话
	 * @param appId 应用
	 */
	@OnMessage
	public void onMessage(String message, Session session, @PathParam(value = "appId") String appId) {
		
		// 消息处理
		for (AppIdMsgHandle appIdMsgHandle : appIdMsgHandles) {
			if (Objects.equals(appIdMsgHandle.getAppId(), appId)) {
				appIdMsgHandle.handleMsg(message, session);
			}
		}
	}
	
	

	/**
	 * @description
	 *	连接 报错
	 * @author TianwYam
	 * @date 2020年9月17日下午3:50:12
	 * @param session
	 * @param error
	 */
    @OnError
    public void onError(Session session, Throwable error) {
        log.error("websocket发生错误", error);
    }

	
	
	
	/**
	 * @description
	 *	根据应用获取所有在线人员
	 * @author TianwYam
	 * @date 2020年9月17日下午3:17:04
	 * @param appId 应用ID
	 * @return
	 */
	public static LinkedHashSet<Session> getSessions(String appId){
		return APPID_CONNECT_SESSION_MAP.get(appId);
	}
	
	
	
	/**
	 * @description
	 *	发送文本消息
	 * @author TianwYam
	 * @date 2020年9月17日下午3:52:52
	 * @param session
	 * @param message
	 */
	public static void sendTxtMsg(Session session , String message) {
		try {
			session.getBasicRemote().sendText(message);
		} catch (IOException e) {
			log.error("websocket发生消息失败", e);
		}
	}



	/**
	 * @description
	 * <pre>
	 *	WebSocket是线程安全的，有用户连接时就会创建一个新的端点实例，一个端点只能保证一个线程调用。总结就是，WebSocket是多例的。
	 *	Autowired 是程序启动时就注入进去
	 *	用静态变量来保存消息处理实现类，避免出现空指针异常
	 *	另一种方式就是 使用 ApplicationContext.getBean()的方式也可以获取
	 *	</pre>
	 * @author TianwYam
	 * @date 2020年9月17日下午7:43:13
	 * @param appIdMsgHandles
	 */
	@Autowired
	public void setAppIdMsgHandles(Set<AppIdMsgHandle> appIdMsgHandles) {
		WebSocketServer.appIdMsgHandles = appIdMsgHandles;
	}	
}

~~~



第三步：界面websocket客户端

~~~javascript
$(function() {

    var websocket;

    // 首先判断是否 支持 WebSocket
    if('WebSocket' in window) {
        websocket = new WebSocket("ws://localhost:8080/ws/chat");
    } else if('MozWebSocket' in window) {
        websocket = new MozWebSocket("ws://localhost:8080/ws/chat");
    } else {
        websocket = new SockJS("http://localhost:8080/ws/chat");
    }

    // 打开时
    websocket.onopen = function(event) {
        console.log("  websocket.onopen  ");
        console.log(event);
        $("#msg").append("<p>(<font color='blue'>欢迎加入聊天群</font>)</p>");
    };

    // 处理消息时
    websocket.onmessage = function(event) {
    	console.log(event);
        $("#msg").append("<p>(<font color='red'>" + event.data + "</font>)</p>");
        console.log("  websocket.onmessage   ");
    };

    websocket.onerror = function(event) {
        console.log("  websocket.onerror  ");
    };

    websocket.onclose = function(event) {
        console.log("  websocket.onclose  ");
    };

    // 点击了发送消息按钮的响应事件
    $("#TXBTN").click(function(){
        // 获取消息内容
        var text = $("#tx").val();
        // 判断
        if(text == null || text == ""){
            alert(" content  can not empty!!");
            return false;
        }
        // 发送消息
        websocket.send(text);
        $("#tx").val('');
    });
});

~~~

html界面

~~~html
<!DOCTYPE HTML>
<html>
    <head>
        <title>WebSocket聊天</title>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta name="renderer" content="webkit">
        <!-- 引入 JQuery  -->
        <script type="text/javascript" src="js/jquery.min.js"></script>
        <!-- 引入 sockJS  -->
        <script type="text/javascript" src="js/sockjs.min.js" ></script>
        <!-- 自定义JS文件 -->
        <script type="text/javascript" src="js/index.js"></script>
    </head>
    <body>
        <!-- 最外边框 -->
        <div style="margin: 20px auto; border: 1px solid blue; width: 300px; height: 500px;">
            <!-- 消息展示框 -->
            <div id="msg" style="width: 100%; height: 70%; border: 1px solid yellow;overflow: auto;"></div>
            <!-- 消息编辑框 -->
            <textarea id="tx" style="width: 98%; height: 20%;"></textarea>
            <!-- 消息发送按钮 -->
            <button id="TXBTN" style="width: 100%; height: 8%;">发送数据</button>
        </div>
    </body>
</html>

~~~

