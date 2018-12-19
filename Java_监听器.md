##监听器

 监听器模型涉及以下三个对象，模型图如下：

（1）事件：用户对组件的一个操作，称之为一个事件

（2）事件源：发生事件的组件就是事件源

（3）事件监听器（处理器）：监听并负责处理事件的方法
	![](https://img-blog.csdn.net/20160612004040293?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#执行顺序 
 1.给事件源注册监听器

 2.组件接受外部作用，也就是事件被触发

 3.组件产生相应的事件对象，并把此时间对象传给事件监听器。

 4.事件处理器启动，并处理相应的操作。

	 
#监听器模式：

	事件源注册监听器之后，当事件源触发事件，监听器就可以回调事件对象的方法；
	更形象地说，监听者模式是基于：注册-回调的事件/消息通知处理模式，就是被监控者将消息通知给所有监控者。 
	
	1、注册监听器：事件源.setListener；
	2、回调：事件源实现onListener。


##案例：
	


	package com.mk.coffee.test.listener;

	import com.sun.xml.internal.ws.api.pipe.Fiber;
	import lombok.extern.slf4j.Slf4j;
	import org.springframework.stereotype.Component;
	
	import javax.servlet.ServletContextEvent;
	import javax.servlet.ServletContextListener;
	import javax.servlet.ServletRequestEvent;
	import javax.servlet.ServletRequestListener;
	import javax.servlet.http.HttpSession;
	import javax.servlet.http.HttpSessionEvent;
	import javax.servlet.http.HttpSessionListener;
	import javax.xml.bind.Unmarshaller;
	
	/**
	 * @Auther: makui
	 * @Date: 2018/12/19
	 * @Description: 事件源
	 * ServletContextListener  它能够监听 ServletContext 对象的生命周期，实际上就是监听 Web 应用的生命周期。
	 * ServletRequestListener  用户响应监听器，用于对Request请求进行监听（创建、销毁）
	 * HttpSessionListener   可以用来收集用户session信息
	 */
	@Slf4j
	@Component
	public class Person implements ServletContextListener,
	        HttpSessionListener, ServletRequestListener {
	
	    private PersonEvent personEvent;
	
	    public void eat(){
	      log.info("【人在吃饭】");
	      personEvent.eatEvent();
	    }
	
	    //注册监听器
	    public void setEventListener(PersonEvent p){
	        this.personEvent = p;
	    }
	
	    @Override
	    public void sessionCreated(HttpSessionEvent httpSessionEvent) {
	        HttpSession session = httpSessionEvent.getSession();
	        session.setAttribute("username","xiaomage");
	        log.info("【用户session生成】username = {}","xiaomage" );
	    }
	
	    @Override
	    public void sessionDestroyed(HttpSessionEvent httpSessionEvent) {
	        HttpSession session = httpSessionEvent.getSession();
	        String name = (String)session.getAttribute("username");
	        log.info("【用户session销毁】username = {}",name );
	    }
	
	    @Override
	    public void contextInitialized(ServletContextEvent servletContextEvent) {
	        log.info("【Web容器正在生成】");
	    }
	
	    @Override
	    public void contextDestroyed(ServletContextEvent servletContextEvent) {
	        log.info("【Web容器正在销毁】");
	    }
	
	    @Override
	    public void requestDestroyed(ServletRequestEvent servletRequestEvent) {
	        log.info("【正在请求】"+ servletRequestEvent.getServletRequest().getServerName());
	
	    }
	
	    @Override
	    public void requestInitialized(ServletRequestEvent servletRequestEvent) {
	        log.info("【请求结束】"+ servletRequestEvent.getServletRequest().getServerName());
	    }
	}
	

	
事件源：Person类 注册事件监听器

	import lombok.extern.slf4j.Slf4j;

	/**
	 * @Auther: makui
	 * @Date: 2018/12/19
	 * @Description: 事件源
	 */
	@Slf4j
	public class Person {

	    private PersonEvent personEvent;
		
		//触发事件
	    public void eat(){
	      log.info("【人在吃饭】");
	      personEvent.eatEvent();
	    }
	
	    //注册监听器
	    public void setEventListener(PersonEvent p){
	        this.personEvent = p;
	    }
	}

事件监听器 ： 

	package com.mk.coffee.test.listener;

	/**
	 * @Auther: makui
	 * @Date: 2018/12/19
	 * @Description:  事件对象
	 */
	public interface PersonEvent {
	
	    void eatEvent();
	}


