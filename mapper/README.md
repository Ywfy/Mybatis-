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
  	insert into tbl_employee(last_name,email,gender)
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
  	select * from tbl_employee where id=#{id} and last_name=#{lastName}
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
select * from tbl_employee where id=${id} and last_name=#{lastName}
Preparing: select * from tbl_employee where id=1 and last_name=?
```
冲着这一点，#{}就应该成为我们的首要选择，而尽量少用${}<br>
不过在原生JDBC不支持占位符的地方就可以使用${}进行取值<br>
比如分表：排序....按照年份分表拆分<br>
```
select * from ${year}_salary where xxx;
select * from tbl_employee order by ${f_name} ${order}
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



<br><br><br>
## mapper结果返回
### 1、简单类型返回值
Mybatis允许增删改直接定义以下类型的返回值：
	int、long、boolean及他们的包装类 和 void
直接在接口处声明返回值的类型，而不需要在XML里面描写,mybatis会自动处理
```
 <!-- public void deleteEmpById(Integer id);  -->
  <delete id="deleteEmpById">
  	delete from tbl_employee where id=#{id} 
  </delete>
```
<br>

select的返回类型才是重中之重，其有 resultType 和 resultMap 两种形式。
### 2、resultType
resultType很简单，只需要看一下接口的返回值，对应一下就OK了。
```
(1)返回POJO，就填POJO的全类名
  <!-- public Employee getEmpByIdAndLastName(Integer id,String name); -->
  <select id="getEmpByIdAndLastName" resultType="com.guigu.mybatis.bean.Employee">
  	select * from tbl_employee where id=#{id} and last_name=#{lastName}
  </select>
  
(2)如果返回的是一个集合，要写集合中元素的类型
  <!-- public List<Employee> getEmpsByLastNameLike(String lastName); -->
  <select id="getEmpsByLastNameLike" resultType="com.guigu.mybatis.bean.Employee">
  	select * from tbl_employee where last_name like #{lastName}
  </select>
  
(3)多条记录封装一个map：返回Map<Integer, POJO>,写POJO的全类名
  <!-- 
  //多条记录封装一个map：Map<Integer,Employee>:key是这条记录的主键，值是记录封装后的javaBean
  //告诉mybatis封装这个map的时候使用哪个属性作为map的key
  @MapKey("lastName")
  public Map<Integer,Employee> getEmpByLastNameLikeReturnMap(String lastName); 
  -->
  <select id="getEmpByLastNameLikeReturnMap" resultType="com.guigu.mybatis.bean.Employee">
  	select * from tbl_employee where last_name like #{lastName}
  </select>
  这个要详细说下，通过传入一个OGNL表达式字符串来模糊匹配获取一系列对象，例如"%e%"，名字中有e的，而显然返回结果一般来说会有很多个
  对象，这时采用Map来封装的话，就存在一个问题，谁作为map的key，所以通过在方法接口上声明 @MapKey("lastName")来确定POJO的哪个属性作为key
  
(4)返回一条记录的map:返回Map<String, Object>,写map(Map类的默认别名)
  //返回一条记录的map，key就是列名，值就是对应的值。
  <!-- public Map<String,Object> getEmpByIdReturnMap(Integer id); -->
  <select id="getEmpByIdReturnMap" resultType="map">
  	select * from tbl_employee where id=#{id}
  </select>
```

### 3、resultMap
resultMap，就是提供给我们自定义某个javaBean的封装规则用的<br>
```
        <!-- 
		自定义某个javaBean的封装规则 
		type：自定义规则的java类型
		id：唯一id方便引用
	-->
	<resultMap type="com.guigu.mybatis.bean.Employee" id="MySimpleEmp">
		<!-- 指定主键列的封装规则 
		id定义主键会底层有优化
		column:指定哪一列
		property：指定对应的javaBean属性
		-->
		<id column="id" property="id"/>
		<!-- 定义普通列封装规则 -->
		<result column="last_name" property="lastName"/>
		<!-- 其他不指定的列会自动封装 ：推荐要写resultMap就把全部的映射规则都写上-->
		<result column="email" property="email"/>
		<result column="gender" property="gender"/>
	</resultMap>
	
	<!-- resultMap:自定义结果集映射规则 -->
	<!-- public Employee getEmpById(Integer id); -->
	<select id="getEmpById"  resultMap="MySimpleEmp">
		select * from tbl_employee where id=#{id}
	</select>
```
<br>

接下来讨论两个特殊情况:<br>
#### 1)对象内嵌着一个普通对象(非集合类、Map)
场景：
	查询Employee的同时查询员工对应的部门
	Employee===Department
	tbl_employee(id last_name gender d_id) || tbl_dept(did dept_name)
##### 解决方式1===>联合查询：级联属性封装结果集
```
 <resultMap type="com.guigu.mybatis.bean.Employee" id="MyDifEmp">
 	<id column="id" property="id"/>
	<result column="last_name" property="lastName"/>
	<result column="genger" property="gender" />
	<result column="did" property="dept.id"/>
	<result column="dept_name" property="dept.departmentName"/>
</resultMap>
<!-- public Employee getEmpAndDept(Integer id); -->
<select id="getEmpAndDept" resultMap="MyDifEmp">
	SELECT e.id id,e.last_name last_name,e.gender gender,e.d_id d_id,
	d.id did,d.dept_name dept_name FROM tbl_employee e LEFT JOIN tbl_dept d 
	ON e.d_id=d.id WHERE e.id=#{id}
</select>
```
<br>

##### 解决方式2===>使用association定义关联的单个对象的封装规则
```
 <resultMap type="com.guigu.mybatis.bean.Employee" id="MyDifEmp2">
	 	<id column="id" property="id"/>
	 	<result column="last_name" property="lastName"/>
	 	<result column="genger" property="gender" />
		
		<!-- association可以指定联合的javaBean对象  
		property="dept",指定哪个属性是联合的对象
		javaType:指定这个属性对象的类型[不能省略]
		-->
		<association property="dept" javaType="com.guigu.mybatis.bean.Department">
			<id column="did" property="id"/>
			<result column="dept_name" property="departmentName"/>
		</association>
	 </resultMap>
	 <!-- public Employee getEmpAndDept(Integer id); -->
	 <select id="getEmpAndDept" resultMap="MyDifEmp2">
	 	SELECT e.id id,e.last_name last_name,e.gender gender,e.d_id d_id,
		d.id did,d.dept_name dept_name FROM tbl_employee e LEFT JOIN tbl_dept d 
		ON e.d_id=d.id WHERE e.id=#{id}
	 </select>
```

##### 解决方式2扩展===>分步查询、延迟加载
使用association进行分步查询<br>
	1、先按照员工ID查询员工信息<br>
	2、根据查询员工信息中的d_id值去部门表查出部门信息<br>
	3、部门设置到员工中；<br>
```
	 <!-- id last_name gender d_id did dept_name -->
	 <resultMap type="com.guigu.mybatis.bean.Employee" id="MyEmpByStep">
	 	<id column="id" property="id"/>
	 	<result column="last_name" property="lastName"/>
	 	<result column="email" property="email"/>
	 	<result column="gender" property="gender"/>
	 	<!-- association定义关联对象的封装规则 
	 		select:表明当前属性是调用select指定的方法查出的结果
	 		column:指定将哪一列的值传给这个方法
	 		
	 		流程：使用select指定的方法（传入column指定的列参数的值）查出对象，并封装给property
	 	-->
	 	<association property="dept" 
	 		select="com.guigu.mybatis.dao.DepartmentMapper.getDeptById"
	 		column="d_id">
	 	</association>
	 </resultMap>
	 <!-- public Employee getEmpByIdStep(Integer id); -->
	 <select id="getEmpByIdStep" resultMap="MyEmpDis">
	 	select * from tbl_employee where id=#{id}
	 </select>
```
<br>

延迟加载(懒加载)的配置：<br>
  之前每次查询Employee对象的时候都是一起将Department查询出来，这对于大型数据库来说无疑是性能的浪费，
  此时可以通过配置延迟加载的方式来实现，当我们查询员工的部门信息的时候再去查询department
  延迟加载的配置非常简单，只需要在分布查询的基础上添加两个配置即可
  来到全局配置文件中，在<settings>标签中添加两个<setting>
```
<settings>
  <setting name="lazyLoadingEnabled" value="true"/> 
  <setting name="aggressiveLazyLoading" value="false"/> 
</settings>
```
这样就OK了<br>

 	
	 

	
#### 2)对象内嵌着一个集合类对象
 场景:<br>
	查询部门的时候将部门对应的所有员工信息也查询出来。<br>
```
<!-- 
  public class Department {
	private Integer id;
	private String departmentName;
	private List<Employee> emps;
did dept_name || eid last_name email gender
-->
```
##### 解决方法1===>collection嵌套结果集的方式：定义关联的集合类型元素的封装规则
```
<!-- 嵌套结果集的方式：使用collection标签定义关联的集合类型的属性封装规则 -->
  	 <resultMap type="com.guigu.mybatis.bean.Department" id="MyDept">
  	 	<id column="did" property="id"/>
  	 	<result column="dept_name" property="departmentName"/>
  	 	<!-- 
  	 		collection定义关联集合类型的属性的封装规则 
  	 		ofType指定集合里面元素的类型
  	 	-->
  	 	<collection property="emps" ofType="com.guigu.mybatis.bean.Employee">
  	 		<!-- 定义这个集合中元素的封装规则 -->
  	 		<id column="eid" property="id"/>
  	 		<result column="last_name" property="lastName"/>
  	 		<result column="email" property="email"/>
  	 		<result column="gender" property="gender"/>
  	 	</collection>
  	 </resultMap>
  	<!-- public Department getDeptByIdPlus(Integer id); -->
  	<select id="getDeptByIdPlus" resultMap="MyDept">
  		select d.id did,d.dept_name dept_name,
			e.id eid,e.last_name last_name,e.email email,
			e.gender gender 
		From tbl_dept d LEFT JOIN tbl_employee e
		ON d.id=e.d_id
		Where d.id=#{id}
  	</select>
```
<br>

##### 解决方法2===>collection分段查询
```
<resultMap type="com.guigu.mybatis.bean.Department" id="MyDeptStep">
  	<id column="id" property="id"/>
  	<result column="dept_name" property="departmentName"/>
  	<collection property="emps" 
  		select="com.guigu.mybatis.dao.EmployeeMapperPlus.getEmpsByDeptId"
  		column="{deptId=id}" fetchType="lazy">
  	</collection>
  </resultMap>
  <!-- public Department getDeptByIdStep(Integer ud); -->
  <select id="getDeptByIdStep" resultMap="MyDeptStep">
  	select id,dept_name departmentName from tbl_dept where id=#{id}
  </select>
```
扩展：
传递多列的值的方法===>将多列的值封装给map传递
```
column="{key1=column1,key2=column2}"
```
fetchType===>非全局定义延迟加载策略
```
在内嵌的对象或集合标签上，添加
fetchType="lazy"：表示使用延迟加载：
  	- lazy 延迟
  	- eager 立即
如果配置它，它将覆盖掉原有在MyBatis设置的全局策略。
```



<br><br><br>
## 动态SQL
### 1、if判断标签
```
<if test=""></if> 判断，test内部使用OGNL表达式，从参数中取值进行判断，注意遇见特殊符号应该去写转义字符:
			&& ==> &amp;&amp;   "" ==> &quot;&quot;<br>
 <select id="getEmpsByConditionIf" resultType="com.guigu.mybatis.bean.Employee">
  	select * from tbl_employee where
 	<if test="id!=null">
  		id=#{id}
 	</if>
 <select>
```

### 2、Trim字符串截取标签
```
<!-- public List<Employee> getEmpsByConditionTrim(Employee employee); -->
  	 <select id="getEmpsByConditionTrim" resultType="com.guigu.mybatis.bean.Employee">
  	 	select * from tbl_employee
  	 	<!-- 后面多出的and或者or where标签不能解决 
  	 	trim标签：trim标签针对的处理对象是标签体中所有SQL语句拼串后的结果
  	 		prefix="" 加前缀   
  	 		prefixOverrides="" 前缀覆盖：去掉前面多余的字符
  	 		suffix="" 加后缀
  	 		suffixOverrides="" 后缀覆盖，去掉后面多余的字符
  	 	
  	 	-->
  	 	<!-- 自定义SQL语句字符串截取规则 -->
  	 	<trim prefix="where" suffixOverrides="and">
  	 		<if test="id!=null">
  	 			id=#{id} and
  	 		</if>
  	 		<if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
  	 			last_name like #{lastName} and
  	 		</if>
  	 		<if test="email!=null and email.trim()!=&quot;&quot;">
  	 			email=#{email} and 
  	 		</if>
  	 		<!-- OGNL会进行字符串与数字的转换判断"0"==0 -->
  	 		<if test="gender==0 or gender==1">
  	 			gender=#{gender}
  	 		</if>
  	 	</trim>
  	 </select>
```

### 3、where标签
where其实就是Trim标签条件限定版，可以帮助我们去除SQL语句 where后面多余的and<br>
注意：and是放在前面！前面的！
```
<!-- 查询员工：要求：携带了哪个字段查询条件就带上这个字段的值 -->
  	 <!-- public List<Employee> getEmpsByConditionIf(Employee employee); -->
  	 <select id="getEmpsByConditionIf" resultType="com.guigu.mybatis.bean.Employee">
  	 	select * from tbl_employee 
  	 	<!-- where -->
  	 	<where>
  	 	<if test="id!=null">
  	 		id=#{id}
  	 	</if>
  	 	<if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
  	 		and last_name like #{lastName}
  	 	</if>
  	 	<if test="email!=null and email.trim()!=&quot;&quot;">
  	 		and email=#{email} 
  	 	</if>
  	 	<!-- OGNL会进行字符串与数字的转换判断"0"==0 -->
  	 	<if test="gender==0 or gender==1">
  	 		and gender=#{gender}
  	 	</if>
  	 	</where>
  	 </select>
```

### 4、set标签
set其实也是Trim标签针对Update语句的限定版，可以帮助我们去除SQL语句 set后面多余的逗号<br>
注意：逗号(,)是放在后面！后面的！
```
 <!-- public void updateEmp(Employee employee); -->
  	 <update id="updateEmp">
  	 	<!-- set标签的使用 -->
  	 	update tbl_employee 
  		<set>
  		<if test="lastName!=null">
  			last_name=#{lastName},
  		</if>
  		<if test="email!=null">
  			email=#{email},
  		</if>
  		<if test="gender==0 or gender==1">
  			gender=#{gender}
  		</if>
  		</set> 
  		<where>
  			<if test="id!=null">
  				id=#{id}
  			</if>
  		</where> 
```

### 5、choose when otherwise分支选择标签
```
<!-- public List<Employee> getEmpsByConditionChoose(Employee employee); -->
  	 <select id="getEmpsByConditionChoose" resultType="com.guigu.mybatis.bean.Employee">
  	 	select * from tbl_employee
  	 	<where>
  	 		<!-- 如果带了id就用id查，如果带了lastName就用lastName查；只会进入其中一个 -->
  	 		<choose>
  	 			<when test="id!=null">
  	 				id=#{id}
  	 			</when>
  	 			<when test="lastName!=null">
  	 				last_name like #{lastName}
  	 			</when>
  	 			<when test="email!=null">
  	 				email=#{email}
  	 			</when>
  	 			<otherwise>
  	 				gender=0
  	 			</otherwise>
  	 		</choose>
  	 	</where>
  	 </select>
```

### 6、foreach循环遍历标签
```
<!-- public List<Employee> getEmpsByConditionForeach(List<Integer> ids); -->
  	 <select id="getEmpsByConditionForeach" resultType="com.guigu.mybatis.bean.Employee">
  	 	select * from tbl_employee where id in
  	 	<!-- 
  	 		collection:指定要遍历的集合，
  	 			list类型的参数会特殊处理封装在map中，map的key就叫list
  	 		item:将遍历出的元素赋值给指定的变量
  	 		separator:每个元素之间的分隔符
  	 		open:遍历出所有结果拼接一个开始的字符
  	 		close:遍历出所有结果拼接一个结束的字符
  	 		index:索引，遍历list的时候是索引，item是值
  	 				      遍历map的时候表示map的key，item就是map的值
  	 		#{变量名}就能取出变量的值也就是当前遍历出的元素
  	 	 -->
  	 	<foreach collection="ids" item="item_id" separator=","
  	 		open="(" close=")" >
  	 		#{item_id}
  	 	</foreach>
  	 </select>
	 

 <!-- 批量保存 -->
  	 <!-- public void addEmps(@Param("emps")List<Employee> emps); -->
  	 <!-- MySQL下批量保存：可以foreach遍历 Mysql支持values(),(),()语法 -->
  	 <insert id="addEmps">
  	 	INSERT INTO tbl_employee(
  	 		<!-- 引用外部定义的sql  -->
  	 		<include refid="insertColumn">
  	 			<property name="testColumn" value="abc"/>
  	 		</include>
  	 	)
		VALUES
		<foreach collection="emps" item="emp" separator=",">
			(#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
		</foreach>
  	 </insert> 
	 

        <!--以下
	这种方式需要数据库连接属性allowMultiQueries=true ,
  	这种分号分隔多个sql可以用于其他的批量操作（删除、修改）
  	 -->
  	 <!-- <insert id="addEmps">
  	 	<foreach collection="emps" item="emp" separator=";">
  	 		INSERT INTO tbl_employee(last_name,email,gender,d_id)
  	 		values(#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
  	 	</foreach>
  	 </insert> -->
  	 
```

### 7、两个内置参数 
不只是方法传递过来的参数可以被用来判断，取值。
```
mybatis默认还有两个内置参数：
_parameter:代表整个参数
	 单个参数：_parameter就是这个参数
	 多个参数：参数会被封装为一个map，_parameter就是代表这个map
	 
_databaseId：如果配置了dataBaseIdProvider标签,
	 		_databaseId就是代表当前数据库的别名

<!-- 	public List<Employee> getEmpsTestInnerParameter(Employee employee); -->
	 <select id="getEmpsTestInnerParameter" resultType="com.guigu.mybatis.bean.Employee">
	 	<!-- bind:可以将OGNL表达式的值绑定到一个变量中，方便以后引用这个变量的值 -->
	    <bind name="_lastName" value="'_'+lastName+'%'"/> 
	 	<if test="_databaseId=='mysql'">
	 		select * from tbl_employee
	 		<if test="_parameter!=null">
	 			where last_name like #{lastName}
	 		</if>
	 	</if>
	 	<if test="_databaseId=='oracle'">
	 		
	 	</if>
	 </select>
```

### 8、SQL代码重用
```
<insert id="addEmps">
  	 INSERT INTO tbl_employee(
  	 	<!-- 引用外部定义的sql  -->
  	 	<include refid="insertColumn">
  	 		<property name="testColumn" value="abc"/>
  	 	</include>
  	)
	VALUES
	<foreach collection="emps" item="emp" separator=",">
		(#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
	</foreach>
</insert> 

 <sql id="insertColumn">
  	<if test="_databaseId=='mysql'">
  	 last_name,email,gender,d_id
  	</if>
  	 <if test="_databaseId=='oracle'">
  	 		
  	 </if>
 </sql>
```

### 9、discriminator 鉴别器
```
<!-- <discriminator javaType=""></discriminator> 
	   		鉴别器：mybatis可以使用discriminator判断某列的值，然后根据某列的值改变封装行为
	   		封装Eployee：
	   			如果查出的是女生，就把部门信息查询出来；
	   			如果是男生，把last_name这一列的值赋值给email
	   -->
	   <resultMap type="com.guigu.mybatis.bean.Employee" id="MyEmpDis">
	   		<id column="id" property="id"/>
	 		<result column="last_name" property="lastName"/>
	 		<result column="email" property="email"/>
	 		<result column="gender" property="gender"/>
	   		
	   		<!-- 
	   			column:指定要判断的列 
	   			javaType:列值对应的java类型
	   		-->
	   		<discriminator javaType="string" column="gender">
	   			<!-- 女生   resultType：指定封装的结果类型,不能省略或者resultMap-->
	   			<case value="0" resultType="com.guigu.mybatis.bean.Employee">
	   				<association property="dept" 
	 					select="com.guigu.mybatis.dao.DepartmentMapper.getDeptById"
	 					column="d_id">
	 				</association>
	   			</case>
	   			<!-- 男生 -->
	   			<case value="1" resultType="com.guigu.mybatis.bean.Employee">
	   				<id column="id" property="id"/>
	 				<result column="last_name" property="lastName"/>
	 				<result column="last_name" property="email"/>
	 				<result column="gender" property="gender"/>
	   			</case>
	   		</discriminator>
	   	
	   </resultMap>
```





<br><br><br>
## Cache
Mybatis的缓存为两级缓存策略。
![无法加载图片](https://github.com/Ywfy/Mybatis-/blob/master/Cache/%E6%8D%95%E8%8E%B733434.PNG)<br>


```
	/**
	 * 两级缓存：
	 * 一级缓存：（本地缓存）sqlSession级别的缓存.一级缓存是一直开启的。SqlSession级别的一个Map
	 * 		与数据库同一次会话期间查询到的数据会放在本地缓存中，
	 * 		以后如果需要获取相同的数据，直接从缓存中拿，没必要再去查询数据库。
	 * 		
	 * 		一级缓存失效情况（没有使用到当前一级缓存的情况，就是还需要向数据库发出查询）：
	 * 		1、sqlSession不同
	 * 		2、sqlSession相同，查询条件不同.(当前一级缓存中还没有这个数据)
	 * 		3、sqlSession相同，两次查询期间执行了增删改操作(这次增删改可能对当前数据有影响）
	 * 		4、sqlSession相同，手动清除了一级缓存
	 * 二级缓存：（全局缓存）基于namespace级别的缓存，一个namespace对应一个二级缓存
	 *		工作机制：
	 *		1、一个会话，查询一条数据，这个数据就会被放在当前会话的一级缓存中
	 *		2、如果当前会话关闭，一级缓存中的数据会被保存到二级缓存中，新的会话查询信息，就可以参照二级缓存中的内容
	 *		3、sqlSession===EmployeeMapper==>Employee
	 *					    DepartmentMapper===>Department
	 *			不同namespace查出的数据会放在自己对应的缓存（map）中
	 *			效果：不同的sqlSession可以从二级缓存中获取数据
	 *				查出的数据会默认先被放在一级缓存中，
	 *				只有会话提交或者关闭以后，一级缓存中的数据才会转移到二级缓存中
	 *		使用：
	 *		1、开启全局二级缓存配置<setting name="cacheEnabled" value="true"/>
	 *		2、去mapper.xml中配置使用二级缓存<cache></cache>
	 *		3、我们的POJO需要实现序列化接口
	 *		additonal:在一个mapper.xml中配置使用二级缓存<cache></cache>后，其他mapper.xml就只需要引用就行了
	 			  <!-- 引用缓存：namespace：指定和哪个名称空间下的缓存一样 -->
  				  <cache-ref namespace="com.guigu.mybatis.dao.EmployeeMapper"/>
	 
	 *	和缓存有关的设置/属性：
	 *			1、cacheEnabled=true   false:关闭缓存(二级缓存关闭）
	 *			2、每个select标签都有useCache="true"，
	 *					false：不适用二级缓存
	 *			3、每个增删改标签的：flushCache="true"
	 *					即执行后清除缓存(一级缓存、二级缓存都清空)
	 *					查询标签也有，默认为flushCache="false";
	 *						若改为flushCache="true"，则每次查询之后都会清空缓存
	 *			4、sqlSession.clearCache();只是清除当前SqlSession的一级缓存
	 *			5、localCacheScope:本地缓存作用域（一级缓存SESSION）：当前会话的所有数据保存在会话缓存中
	 *						STATEMENT：可以禁用一级缓存；
	 *
	 *第三方缓存整合：
	 *		1、导入第三方缓存包
	 *		2、导入与第三方缓存整合的适配包，官方有
	 *      3、mapper.xml中使用自定义缓存
	 *      	<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
	 *      
	 *
	 */
```
```
	mapper.xml中<cache>标签详解
	<!-- <cache eviction="LRU" flushInterval="60000" readOnly="false" size="1024"></cache> -->
	<!-- 
	eviction 缓存回收策略
		LRU – 最近最少使用的:移除最长时间不被使用的对象。
		FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
		SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
		WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。
	 	默认的是 LRU。
	 flushInterval:缓存刷新间隔
	 	缓存多长时间清空一次，默认是不清空，设置一个毫秒值
	 readOnly:是否只读，默认false
	 	true 只读 :mybatis认为所有从缓存中获取数据的操作都是只读操作，不会修改数据
	 			mybatis为了加快获取速度，直接就会将数据在缓存中的引用交给用户。不安全，速度快
	 	false 非只读:mybatis认为获取的数据可能会被修改，
	 			mybatis会利用序列化&反序列化的技术克隆一份新的数据给你。安全，速度慢
	 size:缓存中存放多少元素
	 type:指定自定义缓存的全类名
	 		实现Cache接口，放在type上
	 -->
```
