##Java 过滤器
	
	package com.mk.coffee.test.filter;


	import lombok.extern.slf4j.Slf4j;
	import org.apache.catalina.filters.AddDefaultCharsetFilter;
	import org.apache.commons.lang3.StringUtils;
	
	import javax.servlet.*;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import javax.sound.midi.Soundbank;
	import java.io.IOException;
	
	/**
	 * @Auther: makui
	 * @Date: 2018/12/19
	 * @Description: 过滤器 首先实现Filter接口
	 */
	@Slf4j
	public class Filter_Demo implements Filter {
	    @Override
	    public void init(FilterConfig filterConfig) throws ServletException {
	      log.info("Filter_DEMO 的过滤器正在初始化");
	    }
	
	    @Override
	    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
	        log.info("Filter_DEMO 的过滤器正在工作");
	        HttpServletRequest request = (HttpServletRequest)servletRequest;
	        HttpServletResponse response = (HttpServletResponse)servletResponse;
	        request.setCharacterEncoding("utf-8");
	        response.setContentType("text/html;charset=utf-8");
	        String uri = request.getRequestURI();
	        log.info("当前请求的地址是{}",uri);
	        if(! StringUtils.isEmpty(uri)){
	            filterChain.doFilter(servletRequest,servletResponse); 
	        }
	    }
	
	    @Override
	    public void destroy() {
	        log.info("Filter_DEMO 的过滤器正在销毁");
	    }
	}
