# Mybatis批量执行
其实在Mybatis的全局配置文件下，settings可以设置一个属性叫defaultExecutorType<br>
```
<setting name="defaultExecutorType" value="">
作用是配置默认的执行器。
value可以取以下值：
    SIMPLE 就是普通的执行器
    REUSE 执行器会重用预处理语句（prepared statements）
    BATCH 执行器将重用语句并执行批量更新
```
<br>
但这个配置是全局配置，并不是很实用，我们还是希望在需要批量执行的时候提出来单独执行<br>
这时可以采用下面的方法:<br>

```
SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
SqlSession openSession  = sqlSessionFactory.openSession(ExecutorType.BATCH);

以往sqlSessionFactory.openSession();是生成一个默认全局配置的executor
而现在我们指定了ExecutorType.BATCH，新生成的SqlSession里的executor的类型就会是BATCH，
这时我们使用这个SqlSession去执行的时候就会是批量操作
```
<br>

### BATCH与SIMPLE的区别
我们写一个测试代码：<br>
```
@Test
	public void testBatch() throws IOException {	
		SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
                //记录起始时间
		long start = System.currentTimeMillis();
		//可以执行批量操作的sqlSession
		SqlSession openSession  = sqlSessionFactory.openSession(ExecutorType.BATCH);
		try {
			EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
			for(int i=0; i < 1000; i++) {
				mapper.addEmp(new Employee(UUID.randomUUID().toString().substring(0, 5),"b","1"));
			}
			openSession.commit();
                        //记录结束时间
			long end = System.currentTimeMillis();
			
			System.out.println("Used-Time: " + (end-start));
		}finally {
			openSession.close();
		}
	}
```

以下是部分执行结果(BATCH)：<br>
```
DEBUG 01-17 11:22:50,322 ==> Parameters: cf268(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:22:50,322 ==> Parameters: 99206(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:22:50,322 ==> Parameters: 72e9a(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:22:50,323 ==> Parameters: cd1fd(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:22:50,323 ==> Parameters: 57788(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:22:50,323 ==> Parameters: 075ab(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:22:50,323 ==> Parameters: 06e1b(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:22:50,323 ==> Parameters: 3d6a2(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
Used-Time: 2443
```
<br>

去掉Executor.BATCH,使用默认全局配置生成SIMPLE类型executor执行，执行结果(SIMPLE):
```
DEBUG 01-17 11:27:12,846 ==>  Preparing: insert into tbl_employee(last_name,email,gender) values(?,?,?)   (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:27:12,846 ==> Parameters: 72bd2(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:27:12,847 <==    Updates: 1  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:27:12,847 ==>  Preparing: insert into tbl_employee(last_name,email,gender) values(?,?,?)   (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:27:12,847 ==> Parameters: 90c6c(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:27:12,848 <==    Updates: 1  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:27:12,849 ==>  Preparing: insert into tbl_employee(last_name,email,gender) values(?,?,?)   (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:27:12,849 ==> Parameters: 91739(String), b(String), 1(String)  (BaseJdbcLogger.java:145) 
DEBUG 01-17 11:27:12,850 <==    Updates: 1  (BaseJdbcLogger.java:145) 
Used-Time: 5425
```
<br>

区别非常明显:<br>
BATCH是 预编译sql一次==>设置参数10000次==>执行一次  ，耗时2443ms<br>
SIMPLE是 预编译sql(1000次)==>设置参数(1000次)==>执行(1000次) , 耗时5425ms
