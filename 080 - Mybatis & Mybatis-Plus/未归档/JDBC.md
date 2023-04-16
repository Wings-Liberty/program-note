```java
@Test  
public void test() throws SQLException {

	Connection conn = null;
	
	PreparedStatement pstmt = null;
	
	try {  
		// 1.加载驱动  
		Class.forName("com.mysql.jdbc.Driver");  
		 
		// 2.创建连接  
		conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/my_db", "root", "root");
		  
		// SQL语句 
		String sql="select * from t_user where id=?";  
		 
		// 获得sql执行者  
		pstmt = conn.prepareStatement(sql);
		
		pstmt.setInt(1,1);  
		 
		// 以下两句相当于这一句 ResultSet rs= pstmt.executeQuery();  
		pstmt.execute();  
		ResultSet rs= pstmt.getResultSet();  
		 
		rs.next();
		User user = new User();  
		user.setId(rs.getLong("id"));  
		user.setUserName(rs.getString("user_name"));  
		user.setCreateTime(rs.getDate("create_time"));  
		System.out.println(user.toString());
		
	} catch (Exception e) {  
		e.printStackTrace();  
	}
}
```



> [!NOTE] DriverManager 的 SPI 机制使得无需手动加载 Driver 实现类
> DriverManager 的静态代码块会执行 SPI 加载所有被注册的 Driver

```java
ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
```

![Pasted image 20220712120846](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220712120846.png)

![Pasted image 20220712120904](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220712120904.png)

