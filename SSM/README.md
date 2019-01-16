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
