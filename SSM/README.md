# SSM整合
#### 本教程是以STS环境开发, JDK1.8, MySQL5.7

## 1、准备数据库
   首先，确保你的电脑已经安装有MySQL数据库，若是没有，以下是MySQL数据库的下载页面:<br>
   https://www.mysql.com/downloads/<br>
### 1）创建数据库
```
create database Mybatis;
```
顺便说一下MySQL数据库名、表名大小写的问题：<br>
在Linux环境下，是严格区分大小写的<br>
在Windows环境下，是不区分大小写的<br>

### 2) 创建表
```
use Mybatis;
CREATE TABLE tbl_employee(
	id int(11) PRIMARY KEY AUTO_INCREMENT,
	last_name VARCHAR(255),
	email VARCHAR(255),
	gender VARCHAR(1)
)
```

### 3)准备数据
```
USE Mybatis;

INSERT INTO tbl_employee VALUES
(NULL, "tom", "tom@123.com", "1"),
(NULL, "jerry", "jerry@234.com", "2"),
(NULL, "bade", "bade@345.com", "1"),
(NULL, "bulong", "bulong@456.com", "2");

select * from tbl_employee;
```
<br>


## 2、创建项目
在Eclipse新建Dynamic Web Project,命名为ssm，这里稍微注意一下：<br>
不要直接点Finish，要点击两次next，来到以下页面<br>
![无法加载图片](https://github.com/Ywfy/Mybatis-/blob/master/SSM/web.PNG)<br>
将 Generate web.xml deployment descriptor 打上勾，否则生成的项目没有web.xml文件<br>
<br>


## 3、导入jar包
在/WEB-INF/lib目录下放入以下jar包<br>
```
c3p0-0.9.1.2.jar
com.springsource.net.sf.cglib-2.2.0.jar
com.springsource.org.aopalliance-1.0.0.jar
com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar
commons-logging-1.1.3.jar
mybatis-3.4.1.jar
mybatis-spring-1.3.0.jar
mysql-connector-java-5.1.37-bin.jar
spring-aop-4.0.0.RELEASE.jar
spring-aspects-4.0.0.RELEASE.jar
spring-beans-4.0.0.RELEASE.jar
spring-context-4.0.0.RELEASE.jar
spring-core-4.0.0.RELEASE.jar
spring-expression-4.0.0.RELEASE.jar
spring-jdbc-4.0.0.RELEASE.jar
spring-orm-4.0.0.RELEASE.jar
spring-tx-4.0.0.RELEASE.jar
spring-web-4.0.0.RELEASE.jar
spring-webmvc-4.0.0.RELEASE.jar
taglibs-standard-impl-1.2.1.jar
taglibs-standard-spec-1.2.1.jar
```
* [jar包下载地址](https://github.com/Ywfy/Mybatis-/blob/master/SSM/lib.rar)<br>
<br>


## 4、创建POJO
```
package com.guigu.mybatis.bean;

import java.io.Serializable;
import org.apache.ibatis.type.Alias;

@Alias("emp")
public class Employee implements Serializable{
	
	/**
	 * 实现Serializable序列化接口是为了使用第三方缓存，当然本教程
         * 并不打算使用，有兴趣的同学可以配置一下。
	 */
	private static final long serialVersionUID = 1L;
	private Integer id;
	private String lastName;
	private String email;
	private String gender;
	
	public Employee() {
		super();
		// TODO Auto-generated constructor stub
	}
	public Employee(Integer id, String lastName, String email, String gender) {
		super();
		this.id = id;
		this.lastName = lastName;
		this.email = email;
		this.gender = gender;
	}
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getLastName() {
		return lastName;
	}
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public String getGender() {
		return gender;
	}
	public void setGender(String gender) {
		this.gender = gender;
	}
	@Override
	public String toString() {
		return "Employee [id=" + id + ", lastName=" + lastName + ", email=" + email + ", gender=" + gender + "]";
	}
	
}
```
<br><br>


## 5、创建EmployeeMapper.java
```
package com.guigu.mybatis.dao;

import java.util.List;
import com.guigu.mybatis.bean.Employee;

public interface EmployeeMapper {
	
	public Employee getEmpById(Integer id);
	
	public List<Employee> getEmps();

}
```
<br><br>


## 6、创建EmployeeMapper.xml
注意：必须将EmployeeMapper.xml和EmployeeMapper.java放在同一个包下，且namespace必须写EmployeeMapper的全类名
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.guigu.mybatis.dao.EmployeeMapper">
	
	<!-- public Employee getEmpById(Integer id); -->
	<select id="getEmpById" resultType="com.guigu.mybatis.bean.Employee">
		select * from tbl_employee where id = #{id}
	</select>
	
	<!-- public List<Employee> getEmps(); -->
	<select id="getEmps" resultType="com.guigu.mybatis.bean.Employee">
		select * from tbl_employee;
	</select>
</mapper>
```
<br><br>


## 7、创建EmployeeService 接口
```
package com.guigu.mybatis.service;

import java.util.List;
import com.guigu.mybatis.bean.Employee;

public interface EmployeeService {
	
	public List<Employee> getEmps();
}

```

## 8、创建EmployeeServiceImpl
@Service("EmployeeService")标识EmployeeServiceImpl为一个Service，且在IOC容器中的id改为EmployeeService<br>
@Autowired自动装配了EmployeeMapper<br>
```
package com.guigu.mybatis.service.impl;

import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.guigu.mybatis.bean.Employee;
import com.guigu.mybatis.dao.EmployeeMapper;
import com.guigu.mybatis.service.EmployeeService;

@Service("EmployeeService")
public class EmployeeServiceImpl implements EmployeeService{
	
	@Autowired
	private EmployeeMapper employeeMapper;
	
	public List<Employee> getEmps(){
		return employeeMapper.getEmps();
	}
}
```
<br><br>


## 9、创建EmployeeController
@Controller标识控制器
@Autowired自动装配EmployeeService(实际上是EmployeeServiceImpl）
```
package com.guigu.mybatis.controller;

import java.util.List;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import com.guigu.mybatis.bean.Employee;
import com.guigu.mybatis.service.EmployeeService;

@Controller
public class EmployeeController {
	
	@Autowired
	private EmployeeService employeeService;
	
	@RequestMapping("/emps")
	public String emps(Map<String,Object> map) {
		List<Employee> emps = employeeService.getEmps();
		map.put("allEmps", emps);
		return "list";
	}
}
```
<br><br>


## 10、编辑web.xml
在/WEB-INF目录下有web.xml文件，若是没有，也可以通过以下步骤生成<br>
右键项目名===>Java EE Tools===>Generate Deployment Descriptor Stub<br>
这样就自动生成了<br>

编辑web.xml主要有两个目的：<br>
(1)配置ContextLoaderListener，使得在Web app启动的时候，加载applicationContext.xml，进行Spring相关的初始化工作<br>
(2)配置DispatcherServlet，这是SpringMVC的范畴<br>
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
		 xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
		 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" 		                  id="WebApp_ID" version="3.1">
  <display-name>ssm</display-name>
  <!-- needed for ContextLoaderListener -->
	
	<!-- spring的配置文件 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
	</context-param>

	<!-- Bootstraps the root web application context before servlet initialization -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	
	<!-- spring mvc核心：分发servlet -->
	<!-- The front controller of this Spring Web application, responsible for handling all application requests -->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<!-- spring mvc的配置文件 -->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springMVC.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<!-- Map all requests to the DispatcherServlet for handling -->
	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
</web-app>
```
<br><br>


## 11、配置applicationContext.xml
在项目根目录下新建conf资源文件夹(Source Folder),专门存放配置文件<br>
在conf目录下新建Spring Bean Configuration File类型文件，命名为applicationContext.xml<br>
applicationContext.xml记得引入context、mybatis-spring、tx名称空间<br>
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
	xsi:schemaLocation="http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring-1.2.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd">

	<!-- Spring希望管理所有的业务逻辑组件,等... -->
	<context:component-scan base-package="com.guigu.mybatis">
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>
	
	<!-- 引入数据源配置文件 -->
	<context:property-placeholder location="classpath:dbconfig.properties"/>
	<!-- Spring用来控制业务逻辑、数据源、事务控制、aop -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="${jdbc.driver}"></property>
		<property name="user" value="${jdbc.username}"></property>
		<property name="password" value="${jdbc.password}"></property>
		<property name="jdbcUrl" value="${jdbc.url}"></property>
	</bean>
	
	<!-- 配置事务管理器 -->
	<bean id="dataSourceTransactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
			<property name="dataSource" ref="dataSource"></property>
	</bean>
	
	<!-- 开启基于注解的事务 -->
	<tx:annotation-driven transaction-manager="dataSourceTransactionManager"/>
	
	<!-- 
		整合mybatis
		目的:1、spring来管理所有组件，mapper的实现类
		 		service ===>  Dao	@Autowired:自动注入mapper
		 	2、spring来管理事务: spring声明式事务 
	-->
	<!-- 创建出SqlSessionFactory对象 -->
	<bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
		<!-- configLocation指定配置文件的位置 -->
		<property name="configLocation" value="classpath:mybatis-config.xml"></property>
		<!-- mapperLocations : 指定mapper文件的位置 -->
		<property name="mapperLocations" value="classpath:com/guigu/mybatis/dao/*.xml"></property>
	</bean>
	 
	<!-- 扫描所有的mapper接口的实现，让这些mapper能够自动注入
		base-package : 指定mapper接口的包名
	 -->
	<mybatis-spring:scan base-package="com.guigu.mybatis.dao"/>
	<!-- 老版项目会下面这么写 -->
	<!-- 
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.guigu.mybatis.dao"></property>
	</bean>
	 -->
	
</beans>
```
<br><br>


## 12、配置springMVC.xml
在conf目录下新建Spring Bean Configuration File类型文件，命名为springMVC.xml<br>
记得引入context、mvc名称空间<br>
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

	<!-- SpringMVC只是控制网站跳转逻辑 -->
	<!-- SpringMVC IOC容器只扫描控制器 -->
	<context:component-scan base-package="com.guigu.mybatis" use-default-filters="false">
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>

	<!-- 视图解析器 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/pages/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>

	<mvc:annotation-driven></mvc:annotation-driven>
	<mvc:default-servlet-handler/>

</beans>
```
<br><br>


## 13、创建jsp页面
在WebContent目录下创建index.jsp<br>
在/WEB-INF目录下新建pages文件夹,创建list.jsp<br>
#### index.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<a href="emps">查询所有员工</a>
</body>
</html>
```

#### list.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>返回结果</title>
</head>
<body>
	<table>
			<tr>
				<td>id</td>
				<td>lastName</td>
				<td>email</td>
				<td>gender</td>
			</tr>
		<c:forEach items="${allEmps }" var="emp">
			<tr>
				<td>${emp.id }</td>
				<td>${emp.lastName }</td>
				<td>${emp.email}</td>
				<td>${emp.gender }</td>
			</tr>
		</c:forEach>
	</table>
</body>
</html>
```
<br><br>


## 14、部署Tomcat运行
右键项目===>Run As===>Run on Server,查看运行效果<br>
若是尚未部署Tomcat，以下是在Eclipse中部署Tomcat的教程:<br>
https://jingyan.baidu.com/article/3065b3b6efa9d7becff8a4c6.html<br>
<br><br>


## 运行流程
```
1、当我们启动Tomcat运行项目时，会自动访问index.jsp
2、当我们点击“查询所有员工”后发送/emps请求
3、Tomcat根据web.xml的配置信息，拦截/emps请求，并交给springDispatcherServlet处理
4、springDispatcherServlet根据SpringMVC的配置，将请求交给EmployeeController处理，所以这时EmployeeController先要进行实例化
5、进行EmployeeController的实例化时要注入EmployeeService(其实是自动装配EmployeeServiceImpl，哪怕这里写接口类，若是接口的实现类只有一个，那就会自动地注入这个实现类；若是有多个实现类，则需要进行指定）
6、EmployeeService实例化时要注入EmployeeMapper
7、根据applicationContext.xml的配置信息，将EmployeeMapper.java和EmployeeMapper.xml相关联起来
8、成功实例化EmployeeController后，根据@RequestMapping确定相应的处理逻辑
9、调用employeeService的getEmps方法，获取到一系列Employee对象放到List中，最后封装到map中
10、返回逻辑视图list（注意：其实此时也进行了将map中的对象list提取放到request域的操作）
11、经过视图解析器，逻辑视图list变为物理视图/WEB-INF/pages/list.jsp,然后页面就跳转到物理视图
12、提取request域中的list，遍历显示信息
```

