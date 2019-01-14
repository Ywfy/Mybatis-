# 全局配置文件详解
## 引入dtd约束
只有引入了dtd约束，在编写配置文件时，按alt+'/'才会有提示<br>
![无法加载图片](https://github.com/Ywfy/Mybatis-/blob/master/mybatis-config/dtd.png)
<br>
通过Eclipse，我们发现了两个dtd文件，<br>
(1)mybatis-3-config.dtd<br>
(2)mybatis-3-mapper.dtd<br>
非常明显，第一个是全局配置文件的dtd，第二个是mapper映射文件的dtd<br>
接下来我们需要用压缩软件打开mybatis-3.4.1.jar，按照图上的路径（即org/apache/ibatis/builder/xml），解压出两个dtd文件<br>


在Eclipse导航栏===>Window===>Preferences===>XML===>XML Catalog===>Add,<br>
![无法加载图片](https://github.com/Ywfy/Mybatis-/blob/master/mybatis-config/TIM%E6%88%AA%E5%9B%BE20190114151845.png)<br>
点击File System...选择解压出来的mybatis-3-config.dtd文件<br>
Key type选择URI<br>
Key 复制配置文件开头的http网址<br>
![无法加载图片](https://github.com/Ywfy/Mybatis-/blob/master/mybatis-config/dtd2.png)<br>
然后确定、OK。<br>
**最后最后：请记住先关闭一次配置文件，再重新打开，这样才会生效**<br>
**mapper.xml的dtd约束也是一样的操作，只是File System...选的是mybatis-3-mapper.dtd而已**<br>

<br><br>
## 全局配置文件标签
在全局配置文件中，将鼠标移到<configuration>标签上，会有提示，里面有个Content Model，标明了全局配置文件拥有的标签，而且请注意他们的顺序,<br>
  在编写全局配置文件时一定要按照这个提示的顺序来写，不然编译器会报错.<br>
  以下为全局配置文件拥有的标签:<br>
 (properties, settings, typeAliases, typeHandlers, objectFactory, 
 objectWrapperFactory, reflectorFactory, plugins, environments, databaseIdProvider, mappers)<br>
  下面来细说：<br>
  * properties
  ```
   <!-- 
  		  1、mybatis可以使用properties来引入外部properties配置文件的内容
  		  resource:引入类路径下的资源
  		  url：引入网络路径或者磁盘路径下的资源
        后面取值用${}
   -->
  <properties resource="dbconfig.properties"></properties>
  ```
  这里顺便说一下外部properties配置文件的编写:我是在根目录下新建一个conf资源文件夹，里面新建File,命名为db.properties
  ```
  jdbc.driver=com.mysql.jdbc.Driver
  jdbc.url=jdbc:mysql://localhost/Mybatis
  jdbc.username=root
  jdbc.password=123456
  ```
  
  * settings
  ```
   <!--
  		2、settings包含很多重要的设置项
  		  setting:用来设置每一个设置项
  		  	name:设置项名	value:设置项取值
    -->
  <settings>
  	<setting name="mapUnderscoreToCamelCase" value="true"/>
  </settings>
  ```
  
  * typeAliases
  ```
  <!--
      3、typeAliases：别名处理器，可以为我们的java类型起别名
  		别名不区分大小写
  		总的来说，推荐使用全类名，不推荐使用别名
  -->
  <typeAliases>
  	<!-- 
  		1、typeAlias:为某个java类型起别名 ;
  		默认别名就是类名小写：employee
  		alias：指定新的别名
  	-->
  	<!-- <typeAlias type="com.guigu.mybatis.bean.Employee" alias="emp"></typeAlias> -->
  	
  	
  	<!-- 
  		2、package:为某个包下的所有类批量起别名 
  		name：指定包名（为当前包以及下面所有的后代包的每一个类都起一个默认别名（类名小写）。）
  	-->
  	<package name="com.guigu.mybatis.bean" />
  	
  	<!-- 
  	  3、批量起别名的情况下，使用@Alias注解为某个类型指定新的别名 
  		@Alias("emp")
		  public class Employee {}
  	-->
  </typeAliases>
 
  ```
  
 * typeHandlers, objectFactory, objectWrapperFactory, reflectorFactory, plugins略过，以后遇到再细说
 
 * environments
 ```
 <!-- 
  	4、environments:环境s ， mybatis可以配置多种环境 ，default指定使用某种环境，可以达到快速切换环境的目的
  		environment:配置一个具体的环境信息:必须有两个标签,id代表环境的唯一标识
  			transactionManger：事务管理器
  				type：事务管理器的类型；JDBC(JdbcTransactionFactory)|MANAGED(ManagedTransactionFactory)
  					自定义事务管理器：实现TransactionFactory接口，type指定为全类名
  			dataSource：数据源；
  				type：数据源类型；UNPOOLED(UnpooledDataSourceFactory)
  				|POOLED(PooledDataSourceFactory)
  				|JNDI(JndiDataSourceFactory)
  				自定义数据源：实现DataSourceFactory接口，type是全类名
  	-->
  <environments default="development">
    <environment id="test">
    	<transactionManager type="JDBC"></transactionManager>
      <dataSource type="UNPOOLED">
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
    
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
  </environments>
 ```
 
 * databaseIdProvider
 ```
 <!-- 5、databaseIdProvider:支持多数据库厂商  
  		type="DB_VENDOR",VendorDatabaseIdProvider
  		作用就是得到数据库厂商的标识（驱动getDatabaseProductName()），
  			mybatis就能根据数据库厂商标识来执行不同的sql
  		MySQL  Oracle  SQL Server, xxx
  -->
  <databaseIdProvider type="DB_VENDOR">
        <!-- 为不同的数据库厂商起别名 -->
  		<property name="MySQL" value="mysql"/>
  		<property name="Oracle" value="oracle"/>
  		<property name="SQL Server" value="sql server"/>
  </databaseIdProvider>
 ```
 
 * mappers
 ```
  <!--我们写好的sql映射文件 一定要注册到全局配置文件中 -->
  <!-- 6、mappers:将sql映射文件注册到全局配置中 -->
  <mappers>
  <!-- 
  	mapper：注册一个sql映射 
  		注册配置文件
  		resource：引用类路径下的sql映射文件
  		mybatis/mapper/EmployeeMapper.xml
  		url：引用网络路径或者磁盘路径下的sql映射文件
  		
  		注册接口
  		class:引用（注册）接口，
  			1、有sql映射文件，映射文件名必须和接口同名，并且放在与接口同一目录下
  			2、没有sql映射文件，所有的sql都是利用注解写在接口上
  			推荐，比较重要的、复杂的Dao接口写sql映射文件
  				不重要，简单的Dao接口为了开发快速可以使用注解
          
        @Select("select * from tb1_employee where id=#{id}")
	      public Employee getEmpById(Integer id);
  -->
  	
  <!-- 
  	<mapper url=""/> 	
    <mapper resource="mybatis/mapper/EmployeeMapper.xml"/>
  	<mapper class="com.guigu.mybatis.dao.EmployeeMapperAnnotation"/> 
  -->
  
  <!-- 
    批量注册:跟class注册接口是一样的情况，只是name可以填包名，一次性注册多个接口
  -->
  	<package name="com.guigu.mybatis.dao"/>
  </mappers>
 ```
  
  
