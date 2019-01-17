# Plugin插件开发
## 原理
```
/**
	 * 在四大对象创建的时候:
	 * 1、每个创建出来的对象不是直接返回的，而是interceptorChain.pluginAll(parameterHandler);
	 * 2、获取到所有的interceptor(拦截器)(插件需要实现的接口),调用interceptor.plugin(target);返回target包装后的对象
	 * 3、插件机制,我们可以使用插件为目标对象创建一个代理对象:AOP(面向切面),插件可以为四大对象创建出代理对象,
	 * 		代理对象就可以拦截到四大对象的每一个执行方法
	 *
	 *public Object pluginAll(Object target) {
		    for (Interceptor interceptor : interceptors) {
		      target = interceptor.plugin(target);
		    }
		    return target;
		  }
	 */
```
<br><br>

## 插件编写流程
### 1、编写Interceptor的实现类
#### 1)MyFirstPlugin
```
package com.guigu.mybatis.dao;

import java.util.Properties;

import org.apache.ibatis.executor.statement.StatementHandler;
import org.apache.ibatis.plugin.Interceptor;
import org.apache.ibatis.plugin.Intercepts;
import org.apache.ibatis.plugin.Invocation;
import org.apache.ibatis.plugin.Plugin;
import org.apache.ibatis.plugin.Signature;
import org.apache.ibatis.reflection.MetaObject;
import org.apache.ibatis.reflection.SystemMetaObject;


public class MyFirstPlugin implements Interceptor{

	/**
	 * intercept:拦截
	 * 		拦截目标对象的目标方法的执行
	 */
	@Override
	public Object intercept(Invocation invocation) throws Throwable {
		// TODO Auto-generated method stub
		System.out.println("MyFirstPlugin...intercept: " + invocation.getMethod());
		//动态的改变一下sql运行的参数:以前是1号员工，实际从数据库查询11号员工
		Object target = invocation.getTarget();
		System.out.println("当前拦截到的对象: " + target);
		//拿到StatementHandler===>parameterHandler===>parameterObject
		//拿到target的元数据
		MetaObject metaObject = SystemMetaObject.forObject(target);
		Object value = metaObject.getValue("parameterHandler.parameterObject");
		System.out.println("sql语句用的参数是: " + value);
		//修改完Sql语句要用的参数
		metaObject.setValue("parameterHandler.parameterObject", 11);
		//执行目标方法
		Object proceed = invocation.proceed();
		
		//返回执行后的返回值
		return proceed;
	}
	
	/**
	 * plugin:
	 * 		包装目标对象:包装，其实就是为目标对象创建一个代理对象
	 */
	@Override
	public Object plugin(Object target) {
		// TODO Auto-generated method stub
		System.out.println("MyFirstPlugin...plugin:mybatis将要包装的对象 " + target);
		
		//我们可以借助Plugin的wrap方法来使用当前Interceptor包装我们的目标对象
		Object wrap = Plugin.wrap(target, this);
		
		//返回为当前target创建的动态代理
		return wrap;
	}

	/**
	 * setProperties:
	 * 		将插件注册时的property属性设置进来
	 */
	@Override
	public void setProperties(Properties properties) {
		// TODO Auto-generated method stub
		System.out.println("插件配置的信息: " + properties);
	}

}
```
<br>

##### 2)MySecondPlugin
```
package com.guigu.mybatis.dao;

import java.util.Properties;

import org.apache.ibatis.executor.statement.StatementHandler;
import org.apache.ibatis.plugin.Interceptor;
import org.apache.ibatis.plugin.Intercepts;
import org.apache.ibatis.plugin.Invocation;
import org.apache.ibatis.plugin.Plugin;
import org.apache.ibatis.plugin.Signature;


public class MySecondPlugin implements Interceptor{

	@Override
	public Object intercept(Invocation invocation) throws Throwable {
		// TODO Auto-generated method stub
		System.out.println("MySecondPlugin...intercept: " + invocation.getMethod());
		return invocation.proceed();
	}

	@Override
	public Object plugin(Object target) {
		// TODO Auto-generated method stub
		System.out.println("MySecondPlugin...plugin: " + target);
		return Plugin.wrap(target, this);
	}

	@Override
	public void setProperties(Properties properties) {
		// TODO Auto-generated method stub
		
	}
	
}
```
<br><br>


### 2、使用@Intercepts注解完成插件签名
```
/**
 * 完成插件签名:告诉Mybatis当前插件用来拦截哪个对象的哪个方法
 * type 四大对象的哪一个
 * method 哪个方法
 * args 解决存在方法重载时的鉴别问题
 */
@Intercepts(
		{
			@Signature(type=StatementHandler.class, method="parameterize", args=java.sql.Statement.class)
		})
public class MyFirstPlugin implements Interceptor{...


在这里，我对MySecondPlugin类做一样的处理，拦截同一对象的同一个方法
@Intercepts(
		{
			@Signature(type=StatementHandler.class, method="parameterize", args=java.sql.Statement.class)
		})
public class MySecondPlugin implements Interceptor{...
```
<br><br>


### 3、将写好的插件注册到全局配置文件中
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

  <!-- plugins:注册插件 -->
  <plugins>
  	<plugin interceptor="com.guigu.mybatis.dao.MyFirstPlugin">
  		<property name="username" value="root"/>
  		<property name="password" value="123456"/>
  	</plugin>
  	<plugin interceptor="com.guigu.mybatis.dao.MySecondPlugin">
  	</plugin>
  </plugins>

  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/Mybatis"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
      </dataSource>
    </environment>
  </environments>
  
  
  <!--我们写好的sql映射文件 一定要注册到全局配置文件中 -->
  <mappers>
    <mapper resource="EmployeeMapper.xml"/>
  </mappers>
</configuration>
```
<br><br>
至此插件开发完成，进行测试运行<br>

### 运行结果
```
插件配置的信息: {password=123456, username=root}
MyFirstPlugin...plugin:mybatis将要包装的对象 org.apache.ibatis.executor.CachingExecutor@56ac3a89
MySecondPlugin...plugin: org.apache.ibatis.executor.CachingExecutor@56ac3a89
MyFirstPlugin...plugin:mybatis将要包装的对象 org.apache.ibatis.scripting.defaults.DefaultParameterHandler@3632be31
MySecondPlugin...plugin: org.apache.ibatis.scripting.defaults.DefaultParameterHandler@3632be31
MyFirstPlugin...plugin:mybatis将要包装的对象 org.apache.ibatis.executor.resultset.DefaultResultSetHandler@13a5fe33
MySecondPlugin...plugin: org.apache.ibatis.executor.resultset.DefaultResultSetHandler@13a5fe33
MyFirstPlugin...plugin:mybatis将要包装的对象 org.apache.ibatis.executor.statement.RoutingStatementHandler@370736d9
MySecondPlugin...plugin: org.apache.ibatis.executor.statement.RoutingStatementHandler@370736d9
DEBUG 01-17 09:27:48,276 ==>  Preparing: select id,last_name lastName,email,gender from tbl_employee where id = ?   (BaseJdbcLogger.java:145) 
MySecondPlugin...intercept: public abstract void org.apache.ibatis.executor.statement.StatementHandler.parameterize(java.sql.Statement) throws java.sql.SQLException
MyFirstPlugin...intercept: public abstract void org.apache.ibatis.executor.statement.StatementHandler.parameterize(java.sql.Statement) throws java.sql.SQLException
当前拦截到的对象: org.apache.ibatis.executor.statement.RoutingStatementHandler@370736d9
sql语句用的参数是: 1
DEBUG 01-17 09:27:48,327 ==> Parameters: 11(Integer)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 09:27:48,354 <==      Total: 1  (BaseJdbcLogger.java:145) 
class com.sun.proxy.$Proxy6
Employee [id=11, lastName=Ellen, email=Ellen@atSG.com, gender=1]
```
<br>
这个运行的结果非常明显的体现了多个插件执行时的先后顺序:<br>
包装四大对象时，一直是先MyFirstPlugin,然后是MySecondPlugin<br>
拦截方法时，先执行MySecondPlugin的intercept方法，然后执行MyFirstPlugin的intercept方法<br>
<br>
实际上，非常好理解，代理对象是层层包装的，最初的对象先被一个拦截器包装，包装的结果再被另一个拦截器包装<br>
至于包装的顺序则是看全局配置文件中插件的配置顺序<br>
```
<!-- plugins:注册插件 -->
  <plugins>
  	<plugin interceptor="com.guigu.mybatis.dao.MyFirstPlugin">
  		<property name="username" value="root"/>
  		<property name="password" value="123456"/>
  	</plugin>
  	<plugin interceptor="com.guigu.mybatis.dao.MySecondPlugin">
  	</plugin>
  </plugins>
```
<br>
用图片来进行说明：<br>
![图片无法加载](https://github.com/Ywfy/Mybatis-/blob/master/Plugin/%E6%8F%92%E4%BB%B6%E6%B5%81%E7%A8%8B.PNG)
执行的顺序看图也能清楚的明白了，从外到内。
