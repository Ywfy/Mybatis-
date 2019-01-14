# Mybatis-HelloWorld
### 注意：本文采用的是Eclipse开发。
### 步骤1：创建项目：
普通java项目或者javaWeb项目都行，此处以普通java项目为例。
以下为工程目录结构<br>
![无法加载图片](https://github.com/Ywfy/Mybatis-/blob/master/HelloWorld/img/1.png)
<br>
conf和lib文件夹是我自己新建的，conf文件夹用来放置各种配置文件，lib文件夹用来放置需要导入的jar包，Junit4不用管<br>
注意：关于文件夹有个小细节,若你的文件夹是普通的文件夹，那一定要手动将其中的文件(包括但不限于jar包、配置文件等)加入路径，或者最简单的 右键文件夹==>Build Path ==> Use as Source Folder,将文件夹变为资源文件夹，这样eclipse就会自动将文件夹下的内容引入<br>


### 步骤2：添加jar包到lib目录
1)mybatis-3.4.1.jar<br>
2)mysql-connector-java-5.1.37-bin.jar 【Mysql驱动包】<br>
3)log4j.jar 【日志包】：使Mybatis可以输出日志信息到控制台，需要配合在conf文件夹下新建 log4j.xml ,直接复制下面的代码即可<br>
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

### 步骤3：创建对应的数据库和表格
![无法加载图片](https://github.com/Ywfy/Mybatis-/blob/master/HelloWorld/img/%E6%95%B0%E6%8D%AE%E5%BA%93Employee%E6%A0%BC%E5%BC%8F.png)
<br>
此处，id为主键且自动递增。表名为tbl_employee<br>
可以使用以下SQL语句来创建<br>
```
CREATE DATABASE mybatis;
USE mybatis;
CREATE TABLE tbl_employee(
id INT(11) PRIMARY KEY AUTO_INCREMENT,
last_name VARCHAR(255),
email VARCHAR(255),
gender VARCHAR(255),
d_id INT(11)
);
```
<br>

### 步骤4：创建Mybatis的全局配置文件(mybatis-config.xml)
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <!-- 配置数据库连接信息 -->
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/Mybatis"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
      </dataSource>
    </environment>
  </environments>
</configuration>
```
<br>

### 步骤5：创建表对应的实体类(POJO)
在src目录下创建com.guigu.mybatis.bean包，再在com.guigu.mybatis.bean包下创建
跟tbl_employee对应的实体类(POJO),叫做Employee.java
```
package com.guigu.mybatis.bean;

public class Employee {

	private Integer id;
	private String lastName;
	private String email;
	private String gender;

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
<br>
注意：在此处我强烈建议，getter和setter方法、toString方法都用eclipse来自动生成，不要手动敲。<br>
方法是 shift+alt+S , 会有一系列的Generate方法,里面就有Getter and Setter 和 toString<br>

### 步骤6：创建实体类和表的映射文件
**采用“接口式编程”，所以要创建EmployeeMapper.java和EmployeeMapper.xml**<br>
**1)在src目录下创建com.guigu.mybatis.dao包，在包下创建EmployeeMapper.java**<br>
```
package com.guigu.mybatis.dao;

import com.guigu.mybatis.bean.Employee;

public interface EmployeeMapper {
	
	public Employee getEmpById(Integer id);
}
```
<br>

**2)在conf目录下创建EmployeeMapper.xml**
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.guigu.mybatis.dao.EmployeeMapper">
<!--
  namespace:名称空间    指定为接口的全类名,也就是EmployeeMapper.java的全类名 
-->
<!-- 
  id:唯一标识
  resultType:返回值类型 
  #{id} 从传递过来的参数中取出id值
-->
  <select id="getEmpById" resultType="com.guigu.mybatis.bean.Employee">
    select id,last_name lastName,email,gender  from tbl_employee where id = #{id}
  </select>
</mapper>
```
<br>

## 步骤7：在全局配置文件(mybatis-config.xml)中注册映射文件EmployeeMapper.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <!-- 配置数据库连接信息 -->
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
<br>

## 步骤8:编写测试代码
创建com.guigu.mybatis.test包，在包下创建MyBatisTest.java<br>
```
package com.guigu.mybatis.test;

import java.io.IOException;
import java.io.InputStream;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import com.guigu.mybatis.bean.Employee;
import com.guigu.mybatis.dao.EmployeeMapper;
public class MyBatisTest {
	
	public SqlSessionFactory getSqlSessionFactory() throws IOException{
		String resource = "mybatis-config.xml";
		InputStream inputStream = Resources.getResourceAsStream(resource);
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
		return sqlSessionFactory;
	}
  
        @Test
	public void test01() throws IOException{
		//1、获取sqlSessionfactory
		SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
		
		//2、获取sqlSession
		SqlSession openSession = sqlSessionFactory.openSession();
		
		try {
			//3、获取接口的实现类对象
			//会为接口自动的创建一个代理对象，代理对象去执行增删改查方法
			EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
		
			Employee employee = mapper.getEmpById(1);
			System.out.println(mapper.getClass());
			System.out.println(employee);
		}finally {
			openSession.close();
		}
	}
}
```
<br>

## 以下为我的总目录结构
![无法加载图片](https://github.com/Ywfy/Mybatis-/blob/master/HelloWorld/img/%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png)
<br>

接下来双击test01方法，右键Run as ==> JUnit Test，运行结果为<br>
![无法加载图片](https://github.com/Ywfy/Mybatis-/blob/master/HelloWorld/img/TIM%E6%88%AA%E5%9B%BE20190114120238.png)
<br>

至此，第一个Mybatis项目搞定。


