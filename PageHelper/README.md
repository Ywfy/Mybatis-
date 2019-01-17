# PageHelper
### 本教程只是PageHelper的简单使用，详情请查看PageHelper在github的[官方文档](https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md)

### 1、引入 Jar 包
```
jsqlparser-0.9.5.jar
log4j.jar
mybatis-3.4.1.jar
mysql-connector-java-5.1.37-bin.jar
pagehelper-5.0.0-rc.jar
```

### 2、在全局配置文件配置插件
```
<plugins>
  	<plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
</plugins>
```
OK，已经可以使用了。<br>
<br>

下面简单地介绍一些用法：<br>
```
@Test
	public void test01() throws IOException{
		
		SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
		SqlSession openSession = sqlSessionFactory.openSession();
		
		try {
			EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
			//(5,1)左边的5表示从第五页开始显示，1表示每页的记录数为1
			Page<Object> page = PageHelper.startPage(5,1);
			
			List<Employee> emps = mapper.getEmps();
			//传入要连续显示多少页
			PageInfo<Employee> info = new PageInfo<>(emps, 5);
			for(Employee emp:emps) {
				System.out.println(emp);
			}
		
			System.out.println("当前页码："+info.getPageNum());
			System.out.println("总记录数："+info.getTotal());
			System.out.println("每页的记录数："+info.getPageSize());
			System.out.println("总页码："+info.getPages());
			System.out.println("是否第一页："+info.isIsFirstPage());
			System.out.println("连续显示的页码：");
			int[] nums = info.getNavigatepageNums();
			for(int i=0; i < nums.length; i++) {
				System.out.println(nums[i]);
			}
		}finally {
			openSession.close();
		}
	}
```
