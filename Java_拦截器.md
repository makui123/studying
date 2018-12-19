##Java 拦截器

	package com.mk.coffee.test.interceptor;

	import lombok.extern.slf4j.Slf4j;
	import org.springframework.web.servlet.HandlerInterceptor;
	import org.springframework.web.servlet.ModelAndView;
	
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	
	/**
	 * @Auther: makui
	 * @Date: 2018/12/19
	 * @Description: 拦截器 实现HandlerInterceptor
	 */
	@Slf4j
	public class Interceptor_Demo implements HandlerInterceptor {
	
	    /**
	     * 预处理回调方法，实现处理器的预处理（如检查登陆），第三个参数为响应的处理器，自定义Controller
	     * 返回值：true表示继续流程（如调用下一个拦截器或处理器）；
	     * false表示流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器，此时我们需要通过response来产生响应；
	     */
	    @Override
	    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
	        log.info("---------pre-------");
	        return true;
	    }
	    /**
	     * 后处理回调方法，实现处理器的后处理（但在渲染视图之前），
	     * 此时我们可以通过modelAndView（模型和视图对象）对模型数据进行处理或对视图进行处理，modelAndView也可能为null。
	     */
	    @Override
	    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
	        log.info("---------post-------");
	    }
	
	    /**
	     * 整个请求处理完毕回调方法，即在视图渲染完毕时回调，
	     * 如性能监控中我们可以在此记录结束时间并输出消耗时间，
	     * 还可以进行一些资源清理，类似于try-catch-finally中的finally，但仅调用处理器执行链中
	     */
	    @Override
	    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
	        log.info("---------after-------");
	    }
	}
#HandlerInterceptorAdapter 拦截器适配器

继承HandlerInterceptorAdapter 拦截器适配器 根据自己的需要重写方法,不用重写每一个方法

	package com.mk.coffee.test.interceptor;
	
	
	import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
	
	/**
	 * @Auther: makui
	 * @Date: 2018/12/19
	 * @Description: HandlerInterceptorAdapter 可以根据自己的实际需要去重写方法
	 */
	public class HandlerInterceptorAdapter_Demo extends HandlerInterceptorAdapter {
	}


HandlerInterceptorAdapter源码如下：


	
	package org.springframework.web.servlet.handler;
	
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import org.springframework.web.servlet.AsyncHandlerInterceptor;
	import org.springframework.web.servlet.ModelAndView;
	
	public abstract class HandlerInterceptorAdapter implements AsyncHandlerInterceptor {
	    public HandlerInterceptorAdapter() {
	    }
	
	    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
	        return true;
	    }
	
	    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
	    }
	
	    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
	    }
	
	    public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
	    }
	}


#监听器 过滤器 拦截器的加载顺序： 监听器 -- 过滤器 -- 拦截器

##监听器 过滤器 拦截器 的区别

	