# 自定义类型处理器
#### 本教程以自定义枚举类型处理器为例


## 情景设定:<br>
1、Employee有个enum属性，叫empStatus,表员工状态
```
package com.guigu.mybatis.bean;

public class Employee {
	
	private Integer id;
	private String lastName;
	private String email;
	private String gender;
	//员工状态
	private EmpStatus empStatus = EmpStatus.LOGOUT;
	
	public Employee() {
		super();
		// TODO Auto-generated constructor stub
	}
	
	public Employee(String lastName, String email, String gender) {
		super();
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
	
	public EmpStatus getEmpStatus() {
		return empStatus;
	}

	public void setEmpStatus(EmpStatus empStatus) {
		this.empStatus = empStatus;
	}

	@Override
	public String toString() {
		return "Employee [id=" + id + ", lastName=" + lastName + ", email=" + email + ", gender=" + gender + "]";
	}
	
}
```
<br>

2、empStatus的定义如下：
```
package com.guigu.mybatis.bean;

public enum EmpStatus {
	LOGIN(100, "用户登录"), LOGOUT(200, "用户登出"), REMOVE(300, "用户不存在");
	
	private Integer code;
	private String msg;
	private EmpStatus(Integer code, String msg) {
		this.code = code;
		this.msg = msg;
	}
	public Integer getCode() {
		return code;
	}
	public void setCode(Integer code) {
		this.code = code;
	}
	public String getMsg() {
		return msg;
	}
	public void setMsg(String msg) {
		this.msg = msg;
	}
	
	//按照状态码返回枚举对象
	public static EmpStatus getEmpStatusByCode(Integer code) {
		switch(code) {
		case 100:
			return EmpStatus.LOGIN;
		case 200:
			return EmpStatus.LOGOUT;
		default:
			return EmpStatus.REMOVE;
		}
	}
}
```
<br>

```
3、
   我们想要平时使用的时候，Employee的empStatus保存到数据库的值是我们自定义的状态码，即100、200、300
   而返回给前端的值也是使用状态码
   此时，问题就出现了，我们查阅Enum类型的TypeHandler，发现只有
   EnumTypeHandler:保存枚举的名字到数据库
   EnumOrdinalTypeHandler：保存枚举的索引到数据库
   并没有原生的TypeHandler符合我们的要求，所以我们只能自定义一个枚举类型的TypeHandler
```
<br>

## 自定义枚举类型TypeHandler
### 1、实现TypeHandler<EmpStatus>接口

```
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.TypeHandler;

import com.guigu.mybatis.bean.EmpStatus;

/**
 * 1、实现TypeHandler接口，或者继承BaseTypeHandler
 */
public class MyEnumEmpStatusTypeHandler implements TypeHandler<EmpStatus>{

	/**
	 * 定义当前数据如何保存到数据库中
	 */
	@Override
	public void setParameter(PreparedStatement ps, int i, EmpStatus parameter, JdbcType jdbcType) throws SQLException {
		// TODO Auto-generated method stub
		System.out.println("要保存的状态码: " + parameter.getCode());
		ps.setString(i, parameter.getCode().toString());
	}

	@Override
	public EmpStatus getResult(ResultSet rs, String columnName) throws SQLException {
		// TODO Auto-generated method stub
		//需要根据从数据库中拿到的枚举的状态码返回一个枚举对象
		int code = 	rs.getInt(columnName);
		System.out.println("从数据库中获取的状态码：" + code);
		EmpStatus status = EmpStatus.getEmpStatusByCode(code);
		return status;
	}

	@Override
	public EmpStatus getResult(ResultSet rs, int columnIndex) throws SQLException {
		// TODO Auto-generated method stub
		int code = 	rs.getInt(columnIndex);
		System.out.println("从数据库中获取的状态码：" + code);
		EmpStatus status = EmpStatus.getEmpStatusByCode(code);
		return status;
	}

	@Override
	public EmpStatus getResult(CallableStatement cs, int columnIndex) throws SQLException {
		// TODO Auto-generated method stub
		int code = 	cs.getInt(columnIndex);
		System.out.println("从数据库中获取的状态码：" + code);
		EmpStatus status = EmpStatus.getEmpStatusByCode(code);
		return status;
	}
	
}

```
<br>

### 2、配置typeHandler
配置typeHandler有两种方式:<br>
#### 1)、全局配置<typeHandlers>内配置

```
<typeHandlers>
		<!-- 配置我们自定义的TypeHandler -->
		<typeHandler handler="com.guigu.mybatis.typehandler.MyEnumEmpStatusTypeHandler" javaType="com.guigu.mybatis.bean.EmpStatus"/>
</typeHandlers>
注：因为TypeHandler是支持泛型的，所以要通过javaType指定处理类型。
```

<br>
#### 2)、在处理某个字段的时候告诉mybatis用什么类型处理器

```
保存:#{empStatus, typeHandler=xxxx}
查询:
<resultMap type="com.guigu.mybatis.bean.Employee" id="MyEmp">
  		<id column="id" property="id"/>
  		<result column="empStatus" property="empStatus" typeHandler=""/>
</resultMap>
注意：如果在参数位置修改TypeHandler，应该保证保存数据和查询数据用的TypeHandler是一样的。
```


