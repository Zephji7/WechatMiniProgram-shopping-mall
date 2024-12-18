# 前言
This is a fork from the original repo. It was meant to supply setup proccedure, and modified certain config to run both the backend and the miniprogram.

Environment:

- mac mini m4
- macos 15
- java version

```bash
openjdk version "21.0.5" 2024-10-15 LTS
OpenJDK Runtime Environment Temurin-21.0.5+11 (build 21.0.5+11-LTS)
OpenJDK 64-Bit Server VM Temurin-21.0.5+11 (build 21.0.5+11-LTS, mixed mode, sharing)
```

- homebrew installed mvn, tomcat(tomcat needs to be tomcat9), mysql

## 1. Sql setup

- install mysql using brew
    - Need to setup the root user password after install, brew should show the prompt
- create a database name generate_test

```sql
create database general_test;
```

- create a test_user with the previledge to access to the generate_test

```bash
mysql -u root -p
```

```sql
CREATE USER 'test_user'@'localhost' IDENTIFIED BY 'test_password';
GRANT ALL PRIVILEGES ON general_test.* TO 'newuser'@'localhost';
```

```bash
mysql -u test_user -p general_test
```

- port all .sql files under /sqls to the table

```bash
mysql -u root -p general_test < general_test_human.sql
mysql -u root -p general_test < general_test_cart.sql
mysql -u root -p general_test < general_test_good.sql
mysql -u root -p general_test < general_test_category.sql
mysql -u root -p general_test < general_test_order.sql
```

- start the sql service

```bash
brew services start mysql
```

## 2. Mybatis setup

- pom.xml
    - Add org.glassfish.corba dependency for java environment support
    - Add mysql dependency for sql connection

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.yi</groupId>
	<artifactId>yMybatis</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>yMybatis Maven Webapp</name>
	<url>http://maven.apache.org</url>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-spring</artifactId>
			<version>1.3.1</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.springframework/spring-beans -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-beans</artifactId>
			<version>4.3.12.RELEASE</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.4.2</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.fusesource.jansi/jansi -->
		<dependency>
			<groupId>org.fusesource.jansi</groupId>
			<artifactId>jansi</artifactId>
			<version>1.16</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>3.2.8.RELEASE</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-core</artifactId>
			<version>2.9.2</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.9.2</version>
		</dependency>
		<dependency>
			<groupId>org.glassfish.corba</groupId>
			<artifactId>glassfish-corba-omgapi</artifactId>
			<version>4.1.0</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>8.0.33</version>
		</dependency>

		
	</dependencies>
	<build>
		<finalName>yMybatis</finalName>
	</build>
</project>

```

- `src/main/webapp/WEB-INF/web.xml` has duplicated mapping that might conflict with each other, modify it as below

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
	<display-name>Archetype Created Web Application</display-name>

	<servlet>
		<servlet-name>springmvc</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<!-- DispatcherServlet的初始化方法会启动spring容器， contextConfigLocation用于指定spring配置文件的位置 -->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring_*.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>springmvc</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	<!-- <servlet-mapping>
		<servlet-name>springmvc</servlet-name>
		<url-pattern>/*</url-pattern>
	</servlet-mapping> -->
	<servlet-mapping>
		<servlet-name>default</servlet-name>
		<url-pattern>*.js</url-pattern>
	</servlet-mapping>

	

</web-app>
```

- setup the correct sql connection identity in `/src/main/resources/configuration.xml`
    
    ```xml
    <property name="username" value="test_user" />
    <property name="password" value="test_password" />
    ```
    

- `test/java/yMybatis` should be copy and paste to `src/main/java/`

- All the .xml files under `src/main/java/org/yi/dao` needs to move to `src/main/resources/org/yi/dao` for myBatis to find it correctly.

- to compile and build the project

```bash
mvn clean package
```

  This will generate a yMybatis.war file under `/target`

## 3. Tomcat setup

 For this project, please install Tomcat 9 for compatibility issue.

- Use homebrew to install tomcat 9

```xml
brew install tomcat@9
```

- Add user & row for the manage gui via config

```bash
/opt/homebrew/opt/tomcat/libexec/conf/tomcat-users.xml
```

  commenting out these two lines

```xml
  <user username="admin" password="admin" roles="manager-gui"/>
  <user username="robot" password="robot" roles="manager-script"/>
```

- Run tomcat

```xml
brew services start tomcat@9
```

- Open `localhost:8080/manager`, and enter the username&password which has manager-gui role above

- Choose the .war file generated from the previous section, and deploy it.

- Check if [`localhost:8080/yMybatis`](http://localhost:8080/yMybatis) serves the index.jsp page
    - If it shows 404, it is highly likely due to the servlet-mapping issue mentioned above

- Check if [localhost:8080/yMybatis/test/get_all](http://localhost:8080/yMybatis/test/get_all) returns anything
    - If error occur, might be your tomcat version issue, please check the root cause section on the return content to diagnose

- Check if [localhost:8080/yMybatis/good/get_all](http://localhost:8080/yMybatis/good/get_all) returns anything

## 4.  Run the mini program