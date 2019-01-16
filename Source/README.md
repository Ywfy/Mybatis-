# Mybatis源码简单总结
#### 这是我看的视频里老师作的总结，感觉仍然不是太理解，似懂非懂
```
/**
	 * 1、获取sqlSessionfactory
	 * 		解析文件的每一个信息保存在configuration中，返回包含configuration的DefaultSqlSessionFactory
	 * 		注意:【MappedStatement】，代表一个增删改查标签的详细信息
	 * 2、获取sqlSession
	 * 		返回一个DefaultSqlSession对象，包含Executor和Configuration;
	 * 		这一步会创建Executor对象
	 * 3、获取接口的代理对象(MapperProxy)
	 * 		getMapper使用MapperProxyFactory创建一个MapperProxy的代理对象，
	 * 		代理对象里面包含了DefaultSqlSession(Executor)
	 * 4、执行增删改查方法
	 * 
	 * 
	 * 总结:
	 * 1、根据配置文件(全局，SQL映射)初始化出Configuration对象
	 * 2、创建一个DefaultSqlSession对象，他里面包括Configuration以及
	 * 		Executor(根据全局配置文件中的defaultExecutorType创建出对应的Executor)
	 * 3、DefaultSqlSession.getMapper(), 拿到Mapper接口对应的MapperProxy
	 * 4、MapperProxy里面有(DefaultSqlSession)
	 * 5、执行增删改查方法:
	 * 		1)调用DefaultSqlSession的增删改查(Executor)，
	 * 		2)会创建一个StatementHandler对象,同时也会创建出ParameterHandler和ResultSetHandler
	 * 		3)调用statementHandler预编译参数以及设置参数值。
	 * 			使用ParameterHandler来给Sql设置参数
	 * 		4)调用statementHandler的增删改查
	 * 		5)ResultSetHandler封装结果
	 * 注意：	
	 * 		四大对象每个创建的时候都有一个
	 * 		interceptorChain.pluginAll(parameterHandler);
	 * 		
	 * @throws IOException
	 */
```
![图片无法加载](https://github.com/Ywfy/Mybatis-/blob/master/Source/img/%E5%9B%BE%E7%89%871.png)
<br><br>
![图片无法加载](https://github.com/Ywfy/Mybatis-/blob/master/Source/img/3.png)
<br><br><br>
![图片无法加载](https://github.com/Ywfy/Mybatis-/blob/master/Source/img/4.png)
<br><br><br>
![图片无法加载](https://github.com/Ywfy/Mybatis-/blob/master/Source/img/9.png)
<br><br><br>
![图片无法加载](https://github.com/Ywfy/Mybatis-/blob/master/Source/img/10.png)
<br><br><br>
![图片无法加载](https://github.com/Ywfy/Mybatis-/blob/master/Source/img/11.png)
<br><br><br>
**查询流程图**
![图片无法加载](https://github.com/Ywfy/Mybatis-/blob/master/Source/img/12.png)
<br><br><br>
![图片无法加载](https://github.com/Ywfy/Mybatis-/blob/master/Source/img/13.png)
