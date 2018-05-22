# 京客云redis集成文档

## 集成文件 

> jar包

## 集成步骤

1. 接入方接入jar包方式：
    >
    - jar包以maven方式引入。
    >

    > 执行以下命令：

    ```xml
    mvn install:install-file -Dfile=\*your jar位置*\ -DgroupId=\*your groupid*\ -DartifactId=\*your artifactId*\ -Dversion=\*your jar version*\ -Dpackaging=jar
    ```

    > 安装成功后，需要在pom文件里引入：

    ```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>3.1.0.RELEASE</version>
    </dependency>
    ```

    - 本地方式引入：

    >

    > 暂无。后期补上

2. 接入方工程配置。

* 在web.xml中配置应用的上下文xml，如下：

    ```xml
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:application-context.xml
        </param-value>
    </context-param>
    ```
    在application-context.xml配置jar包的扫描路径。

    ```xml
    <context:component-scan base-package="com.redis.*">
    ```

* 由于现在并无远程redis数据库，所以产品方需要本地安装redis。安装步骤可以参考以下地址：

    [windows]("http://www.runoob.com/redis/redis-install.html")    
    [mac]("https://www.jianshu.com/p/af33284aa57a")

* 设置redis启动的地址，端口，以及redis的密码。密码可以通过redis-cli客户端配置

    如果redis连接报一下错误，需要设置密码：

    > ERR Client sent AUTH, but no password is set

    参考地址：[参考]("https://blog.csdn.net/j_mani/article/details/76459176")

*  web工程的pom.xml中配置，redis依赖包：

    ```xml
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.6.2</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-redis</artifactId>
        <version>1.4.1.RELEASE</version>
    </dependency>
    ```

*  在application-context.xml中配置redis相关数据映射，及redis本身配置

    ```xml
    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
		<property name="maxTotal" value="300" />
		<property name="maxIdle" value="100" />
		<property name="maxWaitMillis" value="1000" />
		<property name="testOnBorrow" value="true" />
	</bean>
	<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
		<property name="usePool" value="true"></property>
		<property name="hostName">
			<value>${jedis.hostName}</value>
		</property><!-- -->
		<property name="port">
			<value>${jedis.port}</value>
		</property>
		<property name="timeout">
			<value>${jedis.timeout}</value>
		</property>
		<property name="password">
			<value>${jedis.password}</value>
		</property>
		<constructor-arg index="0" ref="jedisPoolConfig" />
	</bean>
    ```
    其中里面的value属性，可以已hardcode的方式写在xml中，不过建议已properties文件的方式配置在工程中。如下图：

    ![](http://ohwrspy13.bkt.clouddn.com/18-5-22/5355918.jpg)

    redis.properties中文件如下：

    ![](http://ohwrspy13.bkt.clouddn.com/18-5-22/88913489.jpg)

    如果工程中存在多个配置文件，需要在application-context.xml中配置：

    ```xml
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>classpath:database.properties</value>
				<value>classpath:/config/redis.properties</value>
			</list>
		</property>
	</bean>
    ```

* 在applicaion-context.xml中配置数据映射：

    ```xml

    <bean id="myredisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">
		<property name="connectionFactory" ref="jedisConnectionFactory" />
		<property name="keySerializer">
			<bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
		</property>
		<property name="valueSerializer">
			<bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
		</property>
		<property name="hashValueSerializer">
			<bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
		</property>
	</bean>

    ```    
    __这个myredisTemplate是jar包注解映射的名称，不能修改__

* 做完如上配置以后，需要在工程spring-mvc的配置文件中配置拦截器：

    ```xml
    <mvc:interceptors>
		<!-- 使用bean定义一个Interceptor，直接定义在mvc:interceptors根下面的Interceptor将拦截所有的请求 -->
		<mvc:interceptor>
			<mvc:mapping path="/auth/testPromote.action"/>
			<!-- 定义在mvc:interceptor下面的表示是对特定的请求才进行拦截的 -->
			<bean class="com.redis.interceptor.PromoteInterceptor"/>
		</mvc:interceptor>
	</mvc:interceptors>

	<mvc:interceptors>
		<!-- 使用bean定义一个Interceptor，直接定义在mvc:interceptors根下面的Interceptor将拦截所有的请求 -->
		<mvc:interceptor>
			<mvc:mapping path="/auth/testAudit.action"/>
			<!-- 定义在mvc:interceptor下面的表示是对特定的请求才进行拦截的 -->
			<bean class="com.redis.interceptor.AuditInterceptor"/>
		</mvc:interceptor>
	</mvc:interceptors>
    ```    
    __注意：PromoteInterceptor是推广系统的拦截器，AuditInterceptor是审核系统的拦截器,映射的路径接入方按照业务配置__

* 登录系统必须要有以下两个字段才能完成审核工程，mac地址以及session。相应的key如下：

    mac地址key：**mac**

    session地址key： 

    推广系统：**key_promote_session_id**

    审核系统：**key_audit_session_id**
      
    如下：
    > 推广系统
    
    ![](http://ohwrspy13.bkt.clouddn.com/18-5-22/77170859.jpg)

    > 审核系统

    ![](http://ohwrspy13.bkt.clouddn.com/18-5-22/72961292.jpg)