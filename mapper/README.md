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

  
