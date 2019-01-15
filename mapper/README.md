# Mapper文件详解

## 自增主键值的获取
首先，Mybatis获取自增主键的值必须要底层数据库支持自增主键，自增主键值的获取，<br>
MySQL是没问题的，其可以通过方法statement.getGeneratedKeys() 来获取自增主键的值。<br>
但是Oracle是不行的，Oracle是使用序列生成主键的方式。


其实，Mybatis获取自增主键的值非常简单，只需要在<insert>标签中添加两个属性就行了,即<br>
  useGeneratedKeys="true",使用自增主键获取主键值策略<br>
   keyProperty:指定对应的主键属性，也就是Mybatis获取到主键值以后，将这个值封装给javaBean的哪个属性<br>
示例如下:<br>
  ```
  <insert id="addEmp" parameterType="com.guigu.mybatis.bean.Employee"
  	useGeneratedKeys="true" keyProperty="id">
  	insert into tb1_employee(last_name,email,gender)
  	values(#{lastName},#{email},#{gender})
  </insert>
  ```
  <br>
  非常明显，这个实例最后是将自增主键的值赋给了Employee对象的id属性。
  
  
  
<br><br><br>

## mapper参数传递
### 1、单个参数
单个参数的值传递非常简单，Mybatis不会做特殊处理，只需要用#{参数名}就能够取出参数值。

### 2、多个参数
多个参数的值传递首先要明确决不能像单个参数那样直接进行#{参数名}取值,如下<br>
方法：public Employee getEmpByIdAndLastName(Integer id,String name);<br>
取值：#{id},#{lastName}<br>
上述的方法会抛出以下异常：<br>
```
org.apache.ibatis.exceptions.PersistenceException: 
	### Error querying database.  Cause: org.apache.ibatis.binding.BindingException: Parameter 'id' not found. Available parameters are [0, 1, param1, param2]
	### Cause: org.apache.ibatis.binding.BindingException: Parameter 'id' not found. Available parameters are [0, 1, param1, param2]
```
多个参数的获取有以下几种方式：<br>
#### 1）通用的方式
实际上，在传递多个参数时，Mybatis会做特殊处理，即<br>
将多个参数封装成一个Map<br>
Map的key为 param1， param2， param3... paramN<br>
Map的value 就是传入的参数值<br>
此时通过#{paramN}的方式就可以取出对应位置参数的值，这也是最通用的方式。<br>

#### 2）注解指定@Param
命名参数：明确指定封装参数时map的key：@Param("id")<br>
	多个参数会被封装成一个map<br>
		key：使用@Param注解指定的值<br>
		value：参数值<br>
	#{指定的key}取出对应的参数值<br>
示例：<br>
```
<!-- public Employee getEmpByIdAndLastName(@Param("id")Integer id,@Param("lastName")String lastName); -->
 <select id="getEmpByIdAndLastName" resultType="com.guigu.mybatis.bean.Employee">
  	select * from tb1_employee where id=#{id} and last_name=#{lastName}
 </select>

```
<br>

#### 3）POJO
如果多个参数正好是我们业务逻辑的数据模型，我们就可以直接传入pojo<br>
	#{属性名}：取出传入的pojo的属性值<br>
示例：<br>
```
<!-- public void updateEmp(Employee employee); -->
  <update id="updateEmp">
  	update tb1_employee 
  		set last_name=#{lastName},email=#{email},gender=#{gender}
  		where id=#{id}
  </update>	
```
<br>

#### 4）自制Map
如果参数真的很多，并且这多个参数不是业务模型中的数据，没有对应的POJO，也不经常使用，我们也可以考虑自己传入一个Map<br>
示例：<br>
```
<!-- public Employee getEmpByMap(Map<String, Object> map); -->
  <select id="getEmpByMap" resultType="com.guigu.mybatis.bean.Employee">
  	select * from ${tableName} where id=${id} and last_name=#{lastName}
  </select>
```
<br>

#### 5）To(transfer object)数据传输对象
如果多个参数不是业务模型中的数据，但是经常要使用，推荐编写一个To
示例:<br>
```
<!--
Page{
	int index;
	int size;
}
-->
<!-- public xxxx getxxx(Page page); -->
<select id="getxxx" resultType="xxxx.xxxx.xxxx">
  select * from xxx where index=#{index}
</select>
```
<br>


### 3、Collection(List、Set)，数组
如果是Collection（List、Set）类型或者是数组，也会特殊处理，也是把传入的list或者数组封装在map中。<br>
			key Collection(collection),<br>
			如果是List还可以使用key(list),<br>
			数组(array)<br>
示例:<br>
```
public Employee getEmpById(List<Integer> ids);
	取值：取出第一个id的值：#{list[0]}
```
<br>

### 总结：多使用@Param
参数多的时候会自动封装map，为了程序简单易懂，可以多使用@Param来指定封装时使用的key，<br>
然后#{key}就可以取出map中的值<br>


<br><br>
## 4、#{}和${}
#{}和${}都可以获取map中的值或者pojo对象属性的值<br>
它们的首要区别是:#{}是线程安全的，是以预编译的形式，将参数设置到sql语句中，PreParedStatement:防止sql注入<br>
               ${}是非线程安全的,取出的值直接拼装在sql语句中，会有安全问题<br>
```
select * from tb1_employee where id=${id} and last_name=#{lastName}
Preparing: select * from tb1_employee where id=1 and last_name=?
```
冲着这一点，#{}就应该成为我们的首要选择，而尽量少用${}<br>
不过在原生JDBC不支持占位符的地方就可以使用${}进行取值<br>
比如分表：排序....按照年份分表拆分<br>
```
select * from ${year}_salary where xxx;
select * from tb1_employee order by ${f_name} ${order}
```
<br><br>

## 5、#{}的扩展用法
规定参数的一些规则：
	javaType、jdbcType、mode（存储过程）、numericScale、<br>
	resultMap、typeHandler、jdbcTypeName、expression（未来准备支持的功能）<br>
	
	jdbcType通常需要在某种特定的条件下被设置：<br>
		在数据为null的时候，有些数据库可能不能识别mybatis对null的默认处理，比如Oracle（报错）<br>
		jdbcType OTHER：无效的类型：因为mybatis对所有的null都映射的是原生jdbc的JDBC OTHER类型，而Oracle不能正确处理；<br>
		<br>
		由于全局配置中，jdbcTypeForNull=OTHER，oracle不支持，两种解决方法：<br>
		1、#{email，jdbcType=NULL}<br>
		2、jdbcTypeForNull=NULL<br>
			<setting name="jdbcTypeForNull" value="NULL"/><br>

