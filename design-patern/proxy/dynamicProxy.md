# 动态代理

## 引入

和静态代理相同, 都是在原有类的行为基础上，加入一些多出的行为, 不过动态代理更进一步, 可以做到完全替换原有的行为
动态代理有一个强制性要求，就是被代理的类必须实现了某一个接口，或者本身就是接口，就像我们的Connection。


## 定义

同静态代理

## 结构



## 代码分析

在静态代理的代码分析基础上, 动态代理如下: 

```java
public class Client {

    public static void main(String[] args) {
        Car car = new Car();
        // 这里必须使用接口接收, 不然会转型异常
        Moveable proxy = (Moveable) Proxy.newProxyInstance(
                car.getClass().getClassLoader(), // 被代理对象的classloader
                new Class[] {Moveable.class},// 手动列出接口
                new InvocationHandler() {// 请求处理类, 匿名类
                    
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        Object rst = null;
                        
                        String methodName = method.getName();
                        if ("move".equals(methodName)) {
                            System.out.println("方法被代理前...");
                            rst = method.invoke(car, args);
                            System.out.println("方法被代理后...");
                        }
                        else {
                            rst = method.invoke(car, args);
                        }
                        return rst;
                    }
                });
        proxy.move();
    }
}
```

现在看这样一个问题, 现在需要手动实现一个数据库连接池, 没有使用代理模式的情况下是这样:

```java
public class MyPool {
 
	private int init_count = 3;        // 初始化连接数目
	private int max_count = 6;        // 最大连接数
	private int current_count = 0;  // 记录当前使用连接数
	// 连接池 （存放所有的初始化连接）
	private LinkedList<Connection> pool = new LinkedList<Connection>();
	//1.  构造函数中，初始化连接放入连接池
	public MyPool() {
		// 初始化连接
		for (int i=0; i<init_count; i++){
			// 记录当前连接数目
			current_count++;
			// 创建原始的连接对象
			Connection con = createConnection();
			// 把连接加入连接池
			pool.addLast(con);
		}
	}
	//2. 创建一个新的连接的方法
	private Connection createConnection(){
		try {
			Class.forName("com.mysql.jdbc.Driver");
			// 原始的目标对象
			 Connection con = DriverManager.getConnection("jdbc:mysql:///jdbc_demo", "root", "root");
			return con;
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
	//3. 获取连接
	public Connection getConnection(){
		// 3.1 判断连接池中是否有连接, 如果有连接，就直接从连接池取出
		if (pool.size() > 0){
			return pool.removeFirst();
		}
		// 3.2 连接池中没有连接： 判断，如果没有达到最大连接数，创建；
		if (current_count < max_count) {
			// 记录当前使用的连接数
			current_count++;
			// 创建连接
			return createConnection();
		}
		// 3.3 如果当前已经达到最大连接数，抛出异常
		throw new RuntimeException("当前连接已经达到最大连接数目 ！");
	}
	//4. 释放连接
	public void realeaseConnection(Connection con) {
		// 4.1 判断： 池的数目如果小于初始化连接，就放入池中
		if (pool.size() < init_count){
			pool.addLast(con);
		} else {
			try {
				// 4.2 关闭 
				current_count--;
				con.close();
			} catch (SQLException e) {
				throw new RuntimeException(e);
			}
		}
	}

	// 测试：
	public static void main(String[] args) throws SQLException {
		MyPool pool = new MyPool();
		System.out.println("当前连接: " + pool.current_count);  // 3
		// 使用连接
		pool.getConnection();
		pool.getConnection();
		Connection con4 = pool.getConnection();
		Connection con3 = pool.getConnection();
		Connection con2 = pool.getConnection();
		Connection con1 = pool.getConnection();
		// 释放连接, 连接放回连接池
//        pool.realeaseConnection(con1);
	/*
	 * 希望：当关闭连接的时候，要把连接放入连接池！【当调用Connection接口的close方法时候，希望触发pool.addLast(con);操作】把连接放入连接池
	 * 解决1：实现Connection接口，重写close方法   connection接口方法太多，都实现太麻烦，放弃
	 * 解决2：动态代理
	 */
	con1.close();
	// 再获取
	pool.getConnection();
	System.out.println("连接池：" + pool.pool.size());      // 0
	System.out.println("当前连接: " + pool.current_count);  // 3
	}
}

```

采用动态代理模式, jdk中有现成api, 改进如下: 

```java
/**
 *  JDK 动态代理  Object obj = Proxy.newProxyInstance(....)；
 *  1.参数1：ClassLoader loader ,确定类加载器。程序运行时动态创建类，需要类加载加载到内存。类加载器作用：class文件 --> Class对象
 *      * 一般情况使用都是当前类的类加载器
 *      * 类加载器获得方式：MyFactory.class.getClassLoader();
 *  2.参数2：Class[] interfaces  代理需要实现的接口们（可能有多个）
 *      * 方式1：userService.getClass().getInterfaces()【此方式只能在代理对象和接口是父子关系时使用】
 *      * 方式2：new Class[]{UserService.class}【当被代理对象和其实现接口之间是隔代关系时（即祖孙关系）必须使用方式2获取接口(即:一个一个列出接口)
 *  3.参数3：InvocationHandler h 请求处理类，代理类方法执行时，需要请求处理类来处理。
 *      * 一般采用匿名内部类：new InvocationHandler(){}
 *      * 实现方法 invoke ，代理类每一个方法执行一次，将调用一次invoke
 *          参数1.1：Object proxy ,代理成功对象（即 proxyService，不是“代理之前对象”），一般不用。
 *          参数2.2：Method method ，当前执行的方法
 *              * 当前调用方法名：method.getName();
 *              * 执行目标类方法：Object obj = method.invoke(代理之前对象 , args)
 *          参数3.3：Object[] args
 *              * 当前方法实际参数
 * @return
     */
public class MyPool {
 
	private int init_count = 3;        // 初始化连接数目
	private int max_count = 6;        // 最大连接数
	private int current_count = 0;  // 记录当前使用连接数
	// 连接池 （存放所有的初始化连接）
	private LinkedList<Connection> pool = new LinkedList<Connection>();
	//1.  构造函数中，初始化连接放入连接池
	public MyPool() {
		// 初始化连接
		for (int i=0; i<init_count; i++){
			// 记录当前连接数目
			current_count++;
			// 创建原始的连接对象
			Connection con = createConnection();
			// 把连接加入连接池
			pool.addLast(con);
		}
	}
	//2. 创建一个新的连接的方法
	private Connection createConnection(){
		try {
			Class.forName("com.mysql.jdbc.Driver");
			// 原始的目标对象
			final Connection con = DriverManager.getConnection("jdbc:mysql:///jdbc_demo", "root", "root");
			/**********对con对象代理**************/
			// 对con创建其代理对象
			Connection proxy = (Connection) Proxy.newProxyInstance(
					con.getClass().getClassLoader(),    // 类加载器
					//con.getClass().getInterfaces(),   // 当目标对象是一个具体的类的时候 
					new Class[]{Connection.class},      // 目标对象实现的接口
					new InvocationHandler() {            // 当调用con对象方法的时候， 自动触发事务处理器
						@Override
						public Object invoke(Object proxy, Method method, Object[] args)
								throws Throwable {
							// 方法返回值
							Object result = null;
							// 当前执行的方法的方法名
							String methodName = method.getName();
							// 判断当执行了close方法的时候，把连接放入连接池
							if ("close".equals(methodName)) {
								System.out.println("begin:当前执行close方法开始！");
								// 连接放入连接池
								pool.addLast(con);
								System.out.println("end: 当前连接已经放入连接池了！");
							} else {
								// 调用目标对象方法，注意这里不是代理对象
								result = method.invoke(con, args);
							}
							return result;
						}
					}
			);
			return proxy;
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
	//3. 获取连接
	public Connection getConnection(){
		// 3.1 判断连接池中是否有连接, 如果有连接，就直接从连接池取出
		if (pool.size() > 0){
			return pool.removeFirst();
		}
		// 3.2 连接池中没有连接： 判断，如果没有达到最大连接数，创建；
		if (current_count < max_count) {
			// 记录当前使用的连接数
			current_count++;
			// 创建连接
			return createConnection();
		}
		// 3.3 如果当前已经达到最大连接数，抛出异常
		throw new RuntimeException("当前连接已经达到最大连接数目 ！");
	}
	//4. 释放连接
	public void realeaseConnection(Connection con) {
		// 4.1 判断： 池的数目如果小于初始化连接，就放入池中
		if (pool.size() < init_count){
			pool.addLast(con);
		} else {
			try {
				// 4.2 关闭 
				current_count--;
				con.close();
			} catch (SQLException e) {
				throw new RuntimeException(e);
			}
		}
	}
	// 测试：
	public static void main(String[] args) throws SQLException {
		MyPool pool = new MyPool();
		System.out.println("当前连接: " + pool.current_count);  // 3
		// 使用连接
		pool.getConnection();
		pool.getConnection();
		Connection con4 = pool.getConnection();
		Connection con3 = pool.getConnection();
		Connection con2 = pool.getConnection();
		Connection con1 = pool.getConnection();
		// 释放连接, 连接放回连接池
//        pool.realeaseConnection(con1);
		/*
		 * 希望：当关闭连接的时候，要把连接放入连接池！【当调用Connection接口的close方法时候，希望触发pool.addLast(con);操作】把连接放入连接池
		 * 解决1：实现Connection接口，重写close方法
		 * 解决2：动态代理
		 */
		con1.close();
		// 再获取
		pool.getConnection();
		System.out.println("连接池：" + pool.pool.size());      // 0
		System.out.println("当前连接: " + pool.current_count);  // 3
	}
}

```

## 模式分析

*   动态代理有个条件: 代理对象和目标对象都实现统一接口, 或者目标就是一个接口

优点: 

*   解耦
*   远程代理, 虚拟代理, 保护代理

缺点: 

*   请求变慢
*   系统变得复杂

## 适用场景

见静态代理

## 实例

见静态代理

## 总结

见静态代理
