# MyBatis Generator（MBG）
#### 本教程使用由XML配置文件驱动的方法，只是对官方的MBG配置文件示例进行简单修改使用,
详细使用方式请查询[官方文档](http://www.mybatis.org/generator/)<br>

以下是官方的MBG配置文件示例:<br>
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
  <classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />

  <context id="DB2Tables" targetRuntime="MyBatis3">
    <jdbcConnection driverClass="COM.ibm.db2.jdbc.app.DB2Driver"
        connectionURL="jdbc:db2:TEST"
        userId="db2admin"
        password="db2admin">
    </jdbcConnection>

    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>

    <javaModelGenerator targetPackage="test.model" targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>

    <sqlMapGenerator targetPackage="test.xml"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>

    <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>

    <table schema="DB2ADMIN" tableName="ALLTYPES" domainObjectName="Customer" >
      <property name="useActualColumnNames" value="true"/>
      <generatedKey column="ID" sqlStatement="DB2" identity="true" />
      <columnOverride column="DATE_FIELD" property="startDate" />
      <ignoreColumn column="FRED" />
      <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
    </table>

  </context>
</generatorConfiguration>
```
<br>

我们根据官方配置文件来讲解<br>
### 1、classPathEntry
```
<classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />
MBG在生成dao文件时需要连接数据库去读取表信息，所以至少需要指定数据库驱动jar或zip包的位置,
也可以用于其他需要加载的包路径

不过若是已经将jar包手动放入了类路径中，那这句就可以省略了
```
<br>

### 2、context
```
<context id="DB2Tables" targetRuntime="MyBatis3">
 targetRuntime拥有两个值:
 1、MyBatis3：默认的值，生成基于MyBatis3.x以上版本的内容，包括XXXBySample；
 2、MyBatis3Simple：类似MyBatis3，只是不生成XXXBySample，和拥有selectAll()方法
```
<br>

### 3、jdbcConnection
```
<!-- jdbcConnection：指定如何连接到目标数据库 -->
<jdbcConnection driverClass="COM.ibm.db2.jdbc.app.DB2Driver"
        connectionURL="jdbc:db2:TEST"
        userId="db2admin"
        password="db2admin">
</jdbcConnection>  
```
<br>

### 4、javaTypeResolver
```
 <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
 </javaTypeResolver>
 Java类型解析器不应强制使用BigDecimal字段 - 这意味着如果可能，
		将替换整数类型（Short，Integer，Long等）。
		此功能旨在使数据库DECIMAL和NUMERIC列更易于处理。
```
<br>

### 5、javaModelGenerator
```
  javaModelGenerator：指定javaBean的生成策略 
	targetPackage="test.model"：目标包名,受enableSubPackages属性控制；
	targetProject="\MBGTestProject\src"：目标工程目录,如果目录不存在，MBG不会自动建目录
  
  enableSubPackages:
  在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false
  因为table的标签中schema =“DB2ADMIN”，
  如果为true，则生成的包为test.model.db2admin
  如果为false，则生成的包为test.model
  
  trimStrings:设置是否在getter方法中，对String类型字段调用trim()方法，
              这对于表中字符串数据后面存在空格的情况非常有用
  
<javaModelGenerator targetPackage="test.model" targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
</javaModelGenerator>
```
<br>

### 6、sqlMapGenerator
```
<!-- sqlMapGenerator：sql映射生成策略： -->
 <sqlMapGenerator targetPackage="test.xml"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
 </sqlMapGenerator>
```
<br>

### 7、javaClientGenerator
```
<!-- javaClientGenerator:指定mapper接口所在的位置 -->
 <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
 </javaClientGenerator>
```
<br>

### 8、table
```
<!-- 指定要逆向分析哪些表：根据表来创建javaBean -->
<table schema="DB2ADMIN" tableName="ALLTYPES" domainObjectName="Customer" >
      <property name="useActualColumnNames" value="true"/>
      <generatedKey column="ID" sqlStatement="DB2" identity="true" />
      <columnOverride column="DATE_FIELD" property="startDate" />
      <ignoreColumn column="FRED" />
      <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
</table>
```
<br>

以下是经过修改的简单实例：<br>
### 1、新建一个普通Java项目，在项目根目录下新建一个mbg.xml<br>
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>

  <context id="DB2Tables" targetRuntime="MyBatis3">
  	<!-- jdbcConnection：指定如何连接到目标数据库 -->
    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
        connectionURL="jdbc:mysql://192.168.88.131:3306/Mybatis?allowMultiQueries=true"
        userId="root"
        password="aA4405821985%">
    </jdbcConnection>

	<!-- 
		Java类型解析器不应强制使用BigDecimal字段 - 这意味着如果可能，
		将替换整数类型（Short，Integer，Long等）。
		此功能旨在使数据库DECIMAL和NUMERIC列更易于处理。
	 -->
    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>

	<!-- javaModelGenerator：指定javaBean的生成策略 
	targetPackage="test.model"：目标包名
	targetProject="\MBGTestProject\src"：目标工程
	-->
    <javaModelGenerator targetPackage="com.atguigu.mybatis.bean" 
    		targetProject=".\src">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>

	<!-- sqlMapGenerator：sql映射生成策略： -->
    <sqlMapGenerator targetPackage="com.atguigu.mybatis.dao"  
    	targetProject=".\conf">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>

	<!-- javaClientGenerator:指定mapper接口所在的位置 -->
    <javaClientGenerator type="XMLMAPPER" targetPackage="com.atguigu.mybatis.dao"  
    	targetProject=".\src">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>

	<!-- 指定要逆向分析哪些表：根据表要创建javaBean -->
    <table tableName="tbl_dept" domainObjectName="Department"></table>
    <table tableName="tbl_employee" domainObjectName="Employee"></table>
  </context>
</generatorConfiguration>
```
<br>

### 2、在项目根目录下新建一个资源文件夹，叫lib,放入所需的jar包
```
log4j.jar
mybatis-3.4.1.jar
mybatis-generator-core-1.3.2.jar
mysql-connector-java-5.1.37-bin.jar
ojdbc6.jar
slf4j-api-1.6.1.jar
slf4j-log4j12-1.6.2.jar
```
<br>

### 3、创建包
跟MBG里面所写的相对应。
```
在src目录下
com.atguigu.mybatis.bean
com.atguigu.mybatis.dao
com.atguigu.mybatis.test 这个是用来放生成与测试代码的

在conf目录下
com.atguigu.mybatis.dao
```
<br>

### 4、在根目录下创建conf资源文件夹，放入log4j.xml(为了日志输出)
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
 
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
 
 <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
   <param name="Encoding" value="UTF-8" />
   <layout class="org.apache.log4j.PatternLayout">
    <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS} %m  (%F:%L) \n" />
   </layout>
 </appender>
 <logger name="java.sql">
   <level value="debug" />
 </logger>
 <logger name="org.apache.ibatis">
   <level value="info" />
 </logger>
 <root>
   <level value="debug" />
   <appender-ref ref="STDOUT" />
 </root>
</log4j:configuration>
```
<br>

### 5、编写运行代码
```
package com.atguigu.mybatis.test;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;
import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.internal.DefaultShellCallback;

import com.atguigu.mybatis.bean.Employee;
import com.atguigu.mybatis.bean.EmployeeExample;
import com.atguigu.mybatis.bean.EmployeeExample.Criteria;
import com.atguigu.mybatis.dao.EmployeeMapper;


public class MyBatisTest {

	public SqlSessionFactory getSqlSessionFactory() throws IOException {
		String resource = "mybatis-config.xml";
		InputStream inputStream = Resources.getResourceAsStream(resource);
		return new SqlSessionFactoryBuilder().build(inputStream);
	}
  
  @Test
	public void testMbg() throws Exception {
		List<String> warnings = new ArrayList<String>();
		   boolean overwrite = true;
		   File configFile = new File("mbg.xml");
		   ConfigurationParser cp = new ConfigurationParser(warnings);
		   Configuration config = cp.parseConfiguration(configFile);
		   DefaultShellCallback callback = new DefaultShellCallback(overwrite);
		   MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
		   myBatisGenerator.generate(null);
	}
```
<br>

### 6、双击testMbg()方法，右键Run As ===> Junit Test,刷新项目
![无法加载图片](https://github.com/Ywfy/Mybatis-/blob/master/MBG/mbg.png)<br>
代码出来了
<br>

### 7、编写全局配置文件mybatis-config.xml
MBG已经帮我们完成了大部分的工作，但是全局配置文件还是需要我们自己写的<br>
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	
	<properties resource="dbconfig.properties"></properties>
	
	<settings>
		<setting name="mapUnderscoreToCamelCase" value="true"/>
		<setting name="jdbcTypeForNull" value="NULL"/>
		
		<!--显式的指定每个我们需要更改的配置的值，即使他是默认的。防止版本更新带来的问题  -->
		<setting name="cacheEnabled" value="true"/>
		<setting name="lazyLoadingEnabled" value="true"/>
		<setting name="aggressiveLazyLoading" value="false"/>
	</settings>
	
	<typeAliases>
		<package name="com.atguigu.mybatis.bean"/>
	</typeAliases>
		
	<environments default="dev_mysql">
		<environment id="dev_mysql">
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
	</environments>
	
	<databaseIdProvider type="DB_VENDOR">
		<!-- 为不同的数据库厂商起别名 -->
		<property name="MySQL" value="mysql"/>
		<property name="Oracle" value="oracle"/>
		<property name="SQL Server" value="sqlserver"/>
	</databaseIdProvider>
	
	
	<mappers>	
		<package name="com.atguigu.mybatis.dao"/>
	</mappers>
</configuration>
```
<br>

### 8、因为我引用了外部属性文件，所以要编写外部属性文件dbconfig.properties
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/Mybatis?allowMultiQueries=true
jdbc.username=root
jdbc.password=123456
```
<br>

### 9、测试代码
```
        @Test
	public void testMyBatis3() throws IOException {
		SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
		SqlSession openSession = sqlSessionFactory.openSession();
		
		try {
			EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
			//1、xxxExample就是封装查询条件的
			List<Employee> emps = mapper.selectByExample(null);
			//2、查询员工名字中有e字母的，和员工性别是1的
			//封装员工查询条件的example
			EmployeeExample example = new EmployeeExample();
			//创建一个Criteria,这个Criteria就是拼装查询条件
			//select id, last_name, email, gender, d_id from tb1_employee 
			//WHERE ( last_name like ? and gender = ? ) or email like "%e%"
			Criteria criteria = example.createCriteria();
			criteria.andLastNameLike("%e%");
			criteria.andGenderEqualTo("1");
			
			Criteria criteria2 = example.createCriteria();
			criteria2.andEmailLike("%e%");
			example.or(criteria2);
			
			List<Employee> emps2 = mapper.selectByExample(example);
			for(Employee emp:emps2) {
				System.out.println(emp.getId());
			}
		}finally {
			openSession.close();
		}
	}
```






