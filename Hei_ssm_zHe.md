![domain_account](img\domain_account.png)


配置spring和mybatis配置文件applicationContext.xml 位于resources   

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop.xsd
	http://www.springframework.org/schema/tx
	http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--开启注解的扫描，希望处理service和dao，controller不需要Spring框架去处理-->
    <context:component-scan base-package="cn.itcast" >
        <!--配置哪些注解不扫描-->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller" />
    </context:component-scan>

    <!--Spring整合MyBatis框架-->
    <!--配置连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql:///ssm"/>
        <property name="user" value="root"/>
        <property name="password" value="root"/>
    </bean>

    <!--配置SqlSessionFactory工厂 创建SqlSessionFactory放入容器-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!--扫描AccountDao接口所在包，以生成dao接口的代理实现-->
    <bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cn.itcast.dao"/>
    </bean>

    <!--配置Spring框架声明式事务管理-->
    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!--配置事务通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="*" isolation="DEFAULT"/>
        </tx:attributes>
    </tx:advice>

    <!--配置AOP增强-->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* cn.itcast.service.impl.*ServiceImpl.*(..))"/>
    </aop:config>

</beans>
```

配置mvc  springmvc.xml  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启注解扫描，只扫描Controller注解-->
    <context:component-scan base-package="cn.itcast">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller" />
    </context:component-scan>

    <!--配置的视图解析器对象-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--过滤静态资源-->
    <mvc:resources location="/css/" mapping="/css/**" />
    <mvc:resources location="/images/" mapping="/images/**" />
    <mvc:resources location="/js/" mapping="/js/**" />

    <!--开启SpringMVC注解的支持-->
    <mvc:annotation-driven/>

</beans>
```

web.xml

如何加载spring配置文件：

**web.xml配置DispatcherServlet加载mvc配置文件；配置监听器加载applicationContext.xml，mybatis配置也在这里**  

![image-20201215110400813](img\image-20201215110400813.png)  

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <!--配置Spring的监听器，默认只加载WEB-INF目录下的applicationContext.xml配置文件-->
  <listener>
    <!--监听servletContext域对象的创建和销毁，让监听器去加载spring配置文件-->
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <!--不使用默认路径；设置配置文件的路径-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>

  <!--<context-param>
    <param-name/>
    <param-value/>
  </context-param>-->

  <!--配置前端控制器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--加载springmvc.xml配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!--启动服务器，创建该servlet-->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

  <!--解决中文乱码的过滤器-->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  
</web-app>

```



domain  

```java
package cn.itcast.domain;
import java.io.Serializable;

public class Account implements Serializable{

    private Integer id;
    private String name;
    private Double money;
   get set
}

```

```java
package cn.itcast.dao;
import cn.itcast.domain.Account;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Repository;
import java.util.List;
/**
 * 帐户dao接口
 */
@Repository
public interface AccountDao {

    // 查询所有账户
    @Select("select * from account")
    public List<Account> findAll();

    // 保存帐户信息
    @Insert("insert into account (name,money) values (#{name},#{money})")
    public void saveAccount(Account account);
}

```

不写实现类 由mybatis代理实现  

业务层  

```java
package cn.itcast.service;
import cn.itcast.domain.Account;
import java.util.List;
(业务接口直接复制dao接口)
public interface AccountService {

    // 查询所有账户
    public List<Account> findAll();

    // 保存帐户信息
    public void saveAccount(Account account);

}

```

```java
业务层实现
package cn.itcast.service.impl;
import cn.itcast.dao.AccountDao;
import cn.itcast.domain.Account;
import cn.itcast.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service("accountService")
public class AccountServiceImpl implements AccountService{

    @Autowired
    private AccountDao accountDao;

    public List<Account> findAll() {
        System.out.println("业务层：查询所有账户...");
        return accountDao.findAll();
    }

    public void saveAccount(Account account) {
        System.out.println("业务层：保存帐户...");
        accountDao.saveAccount(account);
    }
}

```

控制器  

```java
package cn.itcast.controller;
import cn.itcast.domain.Account;
import cn.itcast.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;
/**
 * 帐户web
 */
@Controller
@RequestMapping("/account")
public class AccountController {

    @Autowired
    private AccountService accountService;

    @RequestMapping("/findAll")
    public String findAll(Model model){
        System.out.println("表现层：查询所有账户...");
        // 调用service的方法
        List<Account> list = accountService.findAll();
        model.addAttribute("list",list);
        return "list";
    }

    /**
     * 保存
     * @return
     */
    @RequestMapping("/save")
    public void save(Account account, HttpServletRequest request, HttpServletResponse response) throws IOException {
        accountService.saveAccount(account);
        //重定向到find 由find返回结果 
        response.sendRedirect(request.getContextPath()+"/account/findAll");
        return;
    }
}
```

index.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

    <a href="account/findAll">测试查询</a>

    <h3>测试包</h3>

    <form action="account/save" method="post">
        姓名：<input type="text" name="name" /><br/>
        金额：<input type="text" name="money" /><br/>
        <input type="submit" value="保存"/><br/>
    </form>

</body>
</html>

```

list.jsp  

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
    <!--引入jstl标签库用来遍历list-->
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

    <h3>查询所有的帐户</h3>

    <c:forEach items="${list}" var="account"><!--items遍历的集合，var每一项-->
        ${account.name}
    </c:forEach>

</body>
</html>
```



