---
layout: post
title: spring事务处理和事务配置
category: Meta
author: 王哲

excerpt: spring事务处理和事务配置

---

前段时间对spring的事务配置做了比较深入的研究，在此之间对Spring的事务配置虽说也配置过，但是一直没有一个清楚的认识。通过这次的学习发觉Spring的事务配置只要把思路理清，还是比较好掌握的。

    总结如下：

    Spring配置文件中关于事务配置总是由三个组成部分，分别是DataSource、TransactionManager和代理机制这三部分，无论哪种配置方式，一般变化的只是代理机制这部分。

    DataSource、TransactionManager这两部分只是会根据数据访问方式有所变化，比如使用hibernate进行数据访问时，DataSource实际为SessionFactory，TransactionManager的实现为HibernateTransactionManager。

    具体如下图：
![image](/resources/img/transaction.jpeg)





 根据代理机制的不同，总结了五种Spring事务的配置方式，配置文件如下：

    第一种方式：每个Bean都有一个代理


[html] view plain copy 在CODE上查看代码片派生到我的代码片
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:aop="http://www.springframework.org/schema/aop"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
           http://www.springframework.org/schema/context  
           http://www.springframework.org/schema/context/spring-context-2.5.xsd  
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">  
  
    <bean id="sessionFactory"    
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">    
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />    
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />  
    </bean>    
  
    <!-- 定义事务管理器（声明式的事务） -->    
    <bean id="transactionManager"  
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">  
        <property name="sessionFactory" ref="sessionFactory" />  
    </bean>  
      
    <!-- 配置DAO -->  
    <bean id="userDaoTarget" class="com.bluesky.spring.dao.UserDaoImpl">  
        <property name="sessionFactory" ref="sessionFactory" />  
    </bean>  
      
    <bean id="userDao"    
        class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">    
           <!-- 配置事务管理器 -->    
           <property name="transactionManager" ref="transactionManager" />       
        <property name="target" ref="userDaoTarget" />    
         <property name="proxyInterfaces" value="com.bluesky.spring.dao.GeneratorDao" />  
        <!-- 配置事务属性 -->    
        <property name="transactionAttributes">    
            <props>    
                <prop key="*">PROPAGATION_REQUIRED</prop>  
            </props>    
        </property>    
    </bean>    
</beans>  

第二种方式：所有Bean共享一个代理基类



[html] view plain copy 在CODE上查看代码片派生到我的代码片
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:aop="http://www.springframework.org/schema/aop"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
           http://www.springframework.org/schema/context  
           http://www.springframework.org/schema/context/spring-context-2.5.xsd  
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">  
  
    <bean id="sessionFactory"    
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">    
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />    
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />  
    </bean>    
  
    <!-- 定义事务管理器（声明式的事务） -->    
    <bean id="transactionManager"  
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">  
        <property name="sessionFactory" ref="sessionFactory" />  
    </bean>  
      
    <bean id="transactionBase"    
            class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean"    
            lazy-init="true" abstract="true">    
        <!-- 配置事务管理器 -->    
        <property name="transactionManager" ref="transactionManager" />    
        <!-- 配置事务属性 -->    
        <property name="transactionAttributes">    
            <props>    
                <prop key="*">PROPAGATION_REQUIRED</prop>    
            </props>    
        </property>    
    </bean>      
     
    <!-- 配置DAO -->  
    <bean id="userDaoTarget" class="com.bluesky.spring.dao.UserDaoImpl">  
        <property name="sessionFactory" ref="sessionFactory" />  
    </bean>  
      
    <bean id="userDao" parent="transactionBase" >    
        <property name="target" ref="userDaoTarget" />     
    </bean>  
</beans>  

第三种方式：使用拦截器



[html] view plain copy 在CODE上查看代码片派生到我的代码片
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:aop="http://www.springframework.org/schema/aop"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
           http://www.springframework.org/schema/context  
           http://www.springframework.org/schema/context/spring-context-2.5.xsd  
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">  
  
    <bean id="sessionFactory"    
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">    
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />    
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />  
    </bean>    
  
    <!-- 定义事务管理器（声明式的事务） -->    
    <bean id="transactionManager"  
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">  
        <property name="sessionFactory" ref="sessionFactory" />  
    </bean>   
     
    <bean id="transactionInterceptor"    
        class="org.springframework.transaction.interceptor.TransactionInterceptor">    
        <property name="transactionManager" ref="transactionManager" />    
        <!-- 配置事务属性 -->    
        <property name="transactionAttributes">    
            <props>    
                <prop key="*">PROPAGATION_REQUIRED</prop>    
            </props>    
        </property>    
    </bean>  
        
    <bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">    
        <property name="beanNames">    
            <list>    
                <value>*Dao</value>  
            </list>    
        </property>    
        <property name="interceptorNames">    
            <list>    
                <value>transactionInterceptor</value>    
            </list>    
        </property>    
    </bean>    
    
    <!-- 配置DAO -->  
    <bean id="userDao" class="com.bluesky.spring.dao.UserDaoImpl">  
        <property name="sessionFactory" ref="sessionFactory" />  
    </bean>  
</beans>  



第四种方式：使用tx标签配置的拦截器




[html] view plain copy 在CODE上查看代码片派生到我的代码片
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:aop="http://www.springframework.org/schema/aop"  
    xmlns:tx="http://www.springframework.org/schema/tx"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
           http://www.springframework.org/schema/context  
           http://www.springframework.org/schema/context/spring-context-2.5.xsd  
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd  
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">  
  
    <context:annotation-config />  
    <context:component-scan base-package="com.bluesky" />  
  
    <bean id="sessionFactory"    
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">    
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />    
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />  
    </bean>    
  
    <!-- 定义事务管理器（声明式的事务） -->    
    <bean id="transactionManager"  
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">  
        <property name="sessionFactory" ref="sessionFactory" />  
    </bean>  
  
    <tx:advice id="txAdvice" transaction-manager="transactionManager">  
        <tx:attributes>  
            <tx:method name="*" propagation="REQUIRED" />  
        </tx:attributes>  
    </tx:advice>  
      
    <aop:config>  
        <aop:pointcut id="interceptorPointCuts"  
            expression="execution(* com.bluesky.spring.dao.*.*(..))" />  
        <aop:advisor advice-ref="txAdvice"  
            pointcut-ref="interceptorPointCuts" />          
    </aop:config>        
</beans>  



第五种方式：全注解





[html] view plain copy 在CODE上查看代码片派生到我的代码片
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:aop="http://www.springframework.org/schema/aop"  
    xmlns:tx="http://www.springframework.org/schema/tx"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
           http://www.springframework.org/schema/context  
           http://www.springframework.org/schema/context/spring-context-2.5.xsd  
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd  
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">  
  
    <context:annotation-config />  
    <context:component-scan base-package="com.bluesky" />  
  
    <tx:annotation-driven transaction-manager="transactionManager"/>  
  
    <bean id="sessionFactory"    
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">    
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />    
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />  
    </bean>    
  
    <!-- 定义事务管理器（声明式的事务） -->    
    <bean id="transactionManager"  
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">  
        <property name="sessionFactory" ref="sessionFactory" />  
    </bean>  
      
</beans>  

此时在DAO上需加上@Transactional注解，如下：



[java] view plain copy 在CODE上查看代码片派生到我的代码片
package com.bluesky.spring.dao;  
  
import java.util.List;  
  
import org.hibernate.SessionFactory;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.orm.hibernate3.support.HibernateDaoSupport;  
import org.springframework.stereotype.Component;  
  
import com.bluesky.spring.domain.User;  
  
@Transactional  
@Component("userDao")  
public class UserDaoImpl extends HibernateDaoSupport implements UserDao {  
  
    public List<User> listUsers() {  
        return this.getSession().createQuery("from User").list();  
    }  
      
       
}  

Spring声明式事务让我们从复杂的事务处理中得到解脱。使得我们再也无需要去处理获得连接、关闭连接、事务提交和回滚等这些操作。再也无需要我们在与事务相关的方法中处理大量的try…catch…finally代码。 
我们在使用Spring声明式事务时，有一个非常重要的概念就是事务属性。事务属性通常由事务的传播行为，事务的隔离级别，事务的超时值和事务只读标志组成。我们在进行事务划分时，需要进行事务定义，也就是配置事务的属性。 
Spring在TransactionDefinition接口中定义这些属性,以供PlatfromTransactionManager使用, PlatfromTransactionManager是spring事务管理的核心接口。 



[java] view plain copy 在CODE上查看代码片派生到我的代码片
1.  TransactionDefinition    
2.  public interface TransactionDefinition {    
3.      int getPropagationBehavior();    
4.      int getIsolationLevel();    
5.      int getTimeout();    
6.      boolean isReadOnly();    
7.  }    

getTimeout()方法，它返回事务必须在多少秒内完成。 
isReadOnly(),事务是否只读，事务管理器能够根据这个返回值进行优化，确保事务是只读的。 
getIsolationLevel()方法返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据。 

在TransactionDefinition接口中定义了五个不同的事务隔离级别 
ISOLATION_DEFAULT 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别.另外四个与JDBC的隔离级别相对应 
ISOLATION_READ_UNCOMMITTED 这是事务最低的隔离级别，它充许别外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。 
  例如: 
  Mary的原工资为1000,财务人员将Mary的工资改为了8000，但未提交事务 

Java代码  

1. Connection con1 = getConnection();  

2. con.setAutoCommit(false);  

3. update employee set salary = 8000 where empId ="Mary";  


与此同时，Mary正在读取自己的工资 

Java代码  

1. Connection con2 = getConnection();  

2. select  salary from employee where empId ="Mary";  

3. con2.commit();  



Mary发现自己的工资变为了8000，欢天喜地！ 
而财务发现操作有误，而回滚了事务,Mary的工资又变为了1000 

Java代码  

1. //con1  

2.   con1.rollback();  


像这样,Mary记取的工资数8000是一个脏数据。 

ISOLATION_READ_COMMITTED  保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。 

ISOLATION_REPEATABLE_READ  这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生(不可重复读)。 

在事务1中，Mary 读取了自己的工资为1000,操作并没有完成 

Java代码  

1. con1 = getConnection();  

2. select salary from employee empId ="Mary";  



在事务2中，这时财务人员修改了Mary的工资为2000,并提交了事务. 

Java代码  

1. con2 = getConnection();  

2. update employee set salary = 2000;  

3. con2.commit();  



在事务1中，Mary 再次读取自己的工资时，工资变为了2000 

Java代码  

1. //con1  

2. select salary from employee empId ="Mary";  



在一个事务中前后两次读取的结果并不致，导致了不可重复读。 
使用ISOLATION_REPEATABLE_READ可以避免这种情况发生。 

ISOLATION_SERIALIZABLE 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻像读。 

目前工资为1000的员工有10人。 
事务1,读取所有工资为1000的员工。 

Java代码  

1. con1 = getConnection();  

2. Select * from employee where salary =1000;  

共读取10条记录 

这时另一个事务向employee表插入了一条员工记录，工资也为1000 

Java代码  

1. con2 = getConnection();  

2. Insert into employee(empId,salary) values("Lili",1000);  

3. con2.commit();  



事务1再次读取所有工资为1000的员工 

Java代码  

1. //con1  

2. select * from employee where salary =1000;  



共读取到了11条记录，这就产生了幻像读。 
ISOLATION_SERIALIZABLE能避免这样的情况发生。但是这样也耗费了最大的资源。 

getPropagationBehavior()返回事务的传播行为，由是否有一个活动的事务来决定一个事务调用。 

在TransactionDefinition接口中定义了七个事务传播行为。 

PROPAGATION_REQUIRED 如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。 

Java代码  

1. //事务属性 PROPAGATION_REQUIRED  

2. methodA{  

3. ……  

4. methodB();  

5. ……  

6. }  

7.   

8. //事务属性 PROPAGATION_REQUIRED  

9. methodB{  

10.   ……  

11.}  


使用spring声明式事务，spring使用AOP来支持声明式事务，会根据事务属性，自动在方法调用之前决定是否开启一个事务，并在方法执行之后决定事务提交或回滚事务。 

单独调用methodB方法 

Java代码  

1. main{  

2.   metodB();  

3. }  


相当于 

Java代码  

1. Main{  

2. Connection con=null;  

3.   

4.    rry{  

5.       con = getConnection();  

6.       con.setAutoCommit(false);  

7. //方法调用  

8. methodB();  

9. //提交事务  

10.con.commit();  

11.}  

12.Catch(RuntimeException ex){  

13.  //回滚事务  

14.  con.rollback();    

15.}  

16.finally{  

17.  //释放资源  

18.  closeCon();  

19.}  

20.}  


Spring保证在methodB方法中所有的调用都获得到一个相同的连接。在调用methodB时，没有一个存在的事务，所以获得一个新的连接，开启了一个新的事务。 

单独调用MethodA时，在MethodA内又会调用MethodB. 

执行效果相当于 

Java代码  

1. main{  

2.    Connection con = null;  

3.    try{  

4.       con = getConnection();  

5.       methodA();  

6.       con.commit();  

7. }  

8. cathc(RuntimeException ex){  

9.  con.rollback();  

10.}  

11.finally{  

12.  closeCon();  

13.}   

14.}  


调用MethodA时，环境中没有事务，所以开启一个新的事务. 
当在MethodA中调用MethodB时，环境中已经有了一个事务，所以methodB就加入当前事务。 

PROPAGATION_SUPPORTS 如果存在一个事务，支持当前事务。如果没有事务，则非事务的执行。但是对于事务同步的事务管理器，PROPAGATION_SUPPORTS与不使用事务有少许不同。 

Java代码  

1. //事务属性 PROPAGATION_REQUIRED   

2. methodA(){  

3.   methodB();  

4. }  

5.   

6. //事务属性 PROPAGATION_SUPPORTS   

7. methodB(){  

8.   ……  

9. }  


单纯的调用methodB时，methodB方法是非事务的执行的。 
当调用methdA时,methodB则加入了methodA的事务中,事务地执行。 

PROPAGATION_MANDATORY 如果已经存在一个事务，支持当前事务。如果没有一个活动的事务，则抛出异常。 

Java代码  

1. //事务属性 PROPAGATION_REQUIRED   

2. methodA(){  

3.   methodB();  

4. }  

5.   

6. //事务属性 PROPAGATION_MANDATORY   

7. methodB(){  

8.   ……  

9. }  


当单独调用methodB时，因为当前没有一个活动的事务，则会抛出异常 
throw new IllegalTransactionStateException("Transactionpropagation 'mandatory' but no existing transaction found"); 

当调用methodA时，methodB则加入到methodA的事务中，事务地执行。 

PROPAGATION_REQUIRES_NEW 总是开启一个新的事务。如果一个事务已经存在，则将这个存在的事务挂起。 

Java代码  

1. //事务属性 PROPAGATION_REQUIRED   

2. methodA(){  

3.   doSomeThingA();  

4. methodB();  

5. doSomeThingB();  

6. }  

7.   

8. //事务属性 PROPAGATION_REQUIRES_NEW   

9. methodB(){  

10.  ……  

11.}  


当单独调用methodB时，相当于把methodb声明为REQUIRED。开启一个新的事务，事务地执行。 

当调用methodA时 

Java代码  

1. main(){  

2.   methodA();  

3. }  

情况有些大不一样.相当于下面的效果。 

Java代码  

1. main(){  

2.  TransactionManager tm = null;  

3. try{  

4.   //获得一个JTA事务管理器  

5.    tm = getTransactionManager();  

6.    tm.begin();//开启一个新的事务  

7.    Transaction ts1 = tm.getTransaction();  

8.    doSomeThing();  

9.    tm.suspend();//挂起当前事务  

10.   try{  

11.     tm.begin();//重新开启第二个事务  

12.     Transaction ts2 = tm.getTransaction();  

13.     methodB();  

14.     ts2.commit();//提交第二个事务  

15.       

16.   }  

17.  Catch(RunTimeException ex){  

18.     ts2.rollback();//回滚第二个事务  

19.  }  

20.  finally{  

21.    //释放资源  

22.  }  

23.   //methodB执行完后，复恢第一个事务  

24.   tm.resume(ts1);  

25.doSomeThingB();  

26.   ts1.commit();//提交第一个事务  

27.}  

28.catch(RunTimeException ex){  

29.  ts1.rollback();//回滚第一个事务  

30.}  

31.finally{  

32.  //释放资源  

33.}  

34.}  


在这里，我把ts1称为外层事务，ts2称为内层事务。从上面的代码可以看出，ts2与ts1是两个独立的事务，互不相干。Ts2是否成功并不依赖于ts1。如果methodA方法在调用methodB方法后的doSomeThingB方法失败了，而methodB方法所做的结果依然被提交。而除了methodB之外的其它代码导致的结果却被回滚了。 
使用PROPAGATION_REQUIRES_NEW,需要使用JtaTransactionManager作为事务管理器。 

PROPAGATION_NOT_SUPPORTED  总是非事务地执行，并挂起任何存在的事务。 

Java代码  

1. //事务属性 PROPAGATION_REQUIRED   

2. methodA(){  

3.   doSomeThingA();  

4. methodB();  

5. doSomeThingB();  

6. }  

7.   

8. //事务属性 PROPAGATION_NOT_SUPPORTED   

9. methodB(){  

10.  ……  

11.}  


当单独调用methodB时，不启用任何事务机制，非事务地执行。 
当调用methodA时，相当于下面的效果 

Java代码  

1. main(){  

2.  TransactionManager tm = null;  

3. try{  

4.   //获得一个JTA事务管理器  

5.    tm = getTransactionManager();  

6.    tm.begin();//开启一个新的事务  

7.    Transaction ts1 = tm.getTransaction();  

8.    doSomeThing();  

9.    tm.suspend();//挂起当前事务  

10.     methodB();  

11.   //methodB执行完后，复恢第一个事务  

12.   tm.resume(ts1);  

13.doSomeThingB();  

14.   ts1.commit();//提交第一个事务  

15.}  

16.catch(RunTimeException ex){  

17.  ts1.rollback();//回滚第一个事务  

18.}  

19.finally{  

20.  //释放资源  

21.}  

22.}  

使用PROPAGATION_NOT_SUPPORTED,也需要使用JtaTransactionManager作为事务管理器。 

PROPAGATION_NEVER 总是非事务地执行，如果存在一个活动事务，则抛出异常 

Java代码  

1. //事务属性 PROPAGATION_REQUIRED   

2. methodA(){  

3.   doSomeThingA();  

4. methodB();  

5. doSomeThingB();  

6. }  

7.   

8. //事务属性 PROPAGATION_NEVER   

9. methodB(){  

10.  ……  

11.}  

单独调用methodB，则非事务的执行。 
调用methodA则会抛出异常 
throw new IllegalTransactionStateException( 
"Transaction propagation 'never' butexisting transaction found"); 


PROPAGATION_NESTED如果一个活动的事务存在，则运行在一个嵌套的事务中. 如果没有活动事务, 则按TransactionDefinition.PROPAGATION_REQUIRED 属性执行 

这是一个嵌套事务,使用JDBC 3.0驱动时,仅仅支持DataSourceTransactionManager作为事务管理器。需要JDBC 驱动的java.sql.Savepoint类。有一些JTA的事务管理器实现可能也提供了同样的功能。 

使用PROPAGATION_NESTED，还需要把PlatformTransactionManager的nestedTransactionAllowed属性设为true; 
而nestedTransactionAllowed属性值默认为false; 

Java代码  

1. //事务属性 PROPAGATION_REQUIRED   

2. methodA(){  

3.   doSomeThingA();  

4. methodB();  

5. doSomeThingB();  

6. }  

7.   

8. //事务属性 PROPAGATION_NESTED  

9. methodB(){  

10.  ……  

11.}  


如果单独调用methodB方法，则按REQUIRED属性执行。 

如果调用methodA方法，相当于下面的效果 

Java代码  

1. main(){  

2. Connection con = null;  

3. Savepoint savepoint = null;  

4. try{  

5.   con = getConnection();  

6.   con.setAutoCommit(false);  

7.   doSomeThingA();  

8.   savepoint = con2.setSavepoint();  

9.   try  

10.      methodB();  

11.  }catch(RuntimeException ex){  

12.     con.rollback(savepoint);  

13.  }  

14.  finally{  

15.    //释放资源  

16.  }  

17.  

18.  doSomeThingB();  

19.  con.commit();  

20.}  

21.catch(RuntimeException ex){  

22.  con.rollback();  

23.}  

24.finally{  

25.  //释放资源  

26.}  

27.}  

当methodB方法调用之前，调用setSavepoint方法，保存当前的状态到savepoint。如果methodB方法调用失败，则恢复到之前保存的状态。但是需要注意的是，这时的事务并没有进行提交，如果后续的代码(doSomeThingB()方法)调用失败，则回滚包括methodB方法的所有操作。 

嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚。 

PROPAGATION_NESTED与PROPAGATION_REQUIRES_NEW的区别:它们非常类似,都像一个嵌套事务，如果不存在一个活动的事务，都会开启一个新的事务。使用PROPAGATION_REQUIRES_NEW时，内层事务与外层事务就像两个独立的事务一样，一旦内层事务进行了提交后，外层事务不能对其进行回滚。两个事务互不影响。两个事务不是一个真正的嵌套事务。同时它需要JTA事务管理器的支持。 
使用PROPAGATION_NESTED时，外层事务的回滚可以引起内层事务的回滚。而内层事务的异常并不会导致外层事务的回滚，它是一个真正的嵌套事务。DataSourceTransactionManager使用savepoint支持PROPAGATION_NESTED时，需要JDBC 3.0以上驱动及1.4以上的JDK版本支持。其它的JTA TrasactionManager实现可能有不同的支持方式。 

PROPAGATION_REQUIRED应该是我们首先的事务传播行为。它能够满足我们大多数的事务需求。 


[java] view plain copy 在CODE上查看代码片派生到我的代码片
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = { AppBizExeA.class } , noRollbackFor = { AppBizExeB.class })    
    public void method1() throws Exception {    
        System.out.println("method1 start");    
        TPerson per = new TPerson();    
        per.setAge("24");    
        per.setId(123);    
        per.setName("xj");    
        personDao.add(per);    
        throw new NullPointerException();    
    }    
    
        
    @Transactional(propagation = Propagation.NESTED, rollbackFor = { AppBizExeA.class })    
    public void method2() throws Exception {    
        System.out.println("method2 start");    
        TPerson per = new TPerson();    
        per.setAge("24");    
        per.setId(1234);    
        per.setName("xj");    
        personDao.add(per);    
        System.out.println("method2 end");    
    
    }    

Java代码  

1. public static void main(String[] args) throws Exception {  

2.   

3.         ApplicationContext context = new ClassPathXmlApplicationContext(new String[] { "classpath:com/benx/benx.xml" });  

4.         PersonService service = (PersonService) context.getBean("personService");  

5.         service.method1();  

6.   

7.     }  

 

 

 

今天只谈传播行为为REQUIRED的，其他先不谈，因为使用比较少，而且支持也不够好

 

1、执行service.method1()，不管method里面是否嵌套了method2或其他，事务都是以method1开始和结尾，且事务配置同样以method1为准，忽略其他REQUIRED行为的配置，比如异常机制，以method1为准，忽略method2的异常配置

 

2、Transactional的异常控制，默认是Check Exception 不回滚，unCheck Exception回滚

 

3、如果配置了rollbackFor 和 noRollbackFor 且两个都是用同样的异常，那么遇到该异常，还是回滚

 

4、rollbackFor 和noRollbackFor 配置也许不会含盖所有异常，对于遗漏的按照Check Exception 不回滚，unCheck Exception回滚

 


 @Transactional只能被应用到public方法上, 对于其它非public的方法,如果标记了@Transactional也不会报错,但方法没有事务功能.

Spring使用声明式事务处理，默认情况下，如果被注解的数据库操作方法中发生了unchecked异常，所有的数据库操作将rollback；如果发生的异常是checked异常，默认情况下数据库操作还是会提交的。
这种默认的行为是可以改变的。


使用@Transactional注解的noRollbackFor和rollbackFor属性

如：@Transactional(rollbackFor=Exception.class)可以使checked异常发生时，数据库操作也rollback、@Transactional(noRollbackFor=RuntimeException.class)可以使unchecked异常发生时也提交数据库操作。

也可以使用noRollbackForClassName、rollbackForClassName属性来指定一个异常类名的String数组来改变默认的行为。


 
读取数据库中的数据时是不需要事务管理的，这种情况下可以使用事务的传播行为来告诉Spring不需要开启事务，
如：@Transactional(propagation =Propagation.NOT_SUPPORTED)。

事务的传播行为有：

1.           REQUIRED：表示业务方法需要在一个事务中处理，如果业务方法执行时已经在一个事务中，则加入该事务，否则重新开启一个事务。这也是默认的事务传播行为；

2.           NOT_SUPPORTED：声明业务方法不需要事务，如果业务方法执行时已经在一个事务中，则事务被挂起，等方法执行完毕后，事务恢复进行；

3.           REQUIRES_NEW：表明业务方法需要在一个单独的事务中进行，如果业务方法进行时已经在一个事务中，则这个事务被挂起，并重新开启一个事务来执行这个业务方法，业务方法执行完毕后，原来的事务恢复进行；

4.           MANDATORY：该属性指定业务方法只能在一个已经存在的事务中进行，业务方法不能发起自己的事务；如果业务方法没有在一个既有的事务中进行，容器将抛出异常；

5.           SUPPORTS：该属性指定，如果业务方法在一个既有的事务中进行，则加入该事务；否则，业务方法将在一个没有事务的环境下进行；

6.           NEVER：指定业务方法不可以在事务中进行，如果业务方法执行时已经在一个事务中，容器将抛出异常；

7.           NESTED：该属性指定，如果业务方法在一个既有的事务中执行，则该业务方法将在一个嵌套的事务中进行；否则，按照REQUEIRED来对待。它使用一个单独的事务，这个事务可以有多个rollback点，内部事务的rollback对外部事务没有影响，但外部事务的rollback会导致内部事务的rollback。这个行为只对DataSourceTransactionManager有效。

 

事务的隔离级别

  使用@Transactional的Isolation属性可以指定事务的隔离级别。但事务的隔离级别是由底层的数据库实现的，并不是由Spring来实现。

1.       READ_UNCOMMITTED：会出现脏读、不可重复读和幻读问题；

2.       READ_COMMITTED：会出现不可重复读和幻读问题；

3.       REPEATABLE_READ：会出现幻读问题；

4.       SERIALIZABLE：串行化，不会出现上面的问题。

  一般的数据库默认提供的是READ_COMMITTED隔离级别，如sqlserver2000；MySQL默认提供的是REPEATABLE_READ。

@Transactional  的所有可选属性如下:
属性     类型     默认值     说明
propagation                            Propagation枚举    REQUIRED     事务传播属性 
isolation                                   isolation枚举            DEFAULT     事务隔离级别
readOnly                                  boolean                    false     是否只读
timeout                                      int                            -1     超时(秒)
rollbackFor                              Class[]  {}                 需要回滚的异常类
rollbackForClassName          String[]  {}                 需要回滚的异常类名
noRollbackFor                        Class[]  {}                 不需要回滚的异常类
noRollbackForClassName     String[]  {}                 不需要回滚的异常类名


//事务传播属性
    @Transactional(propagation=Propagation.REQUIRED) //如果有事务,那么加入事务,没有的话新建一个(不写的情况下)
    @Transactional(propagation=Propagation.NOT_SUPPORTED) //容器不为这个方法开启事务
    @Transactional(propagation=Propagation.REQUIRES_NEW) //不管是否存在事务,都创建一个新的事务,原来的挂起,新的执行完毕,继续执行老的事务
    @Transactional(propagation=Propagation.MANDATORY) //必须在一个已有的事务中执行,否则抛出异常
    @Transactional(propagation=Propagation.NEVER) //必须在一个没有的事务中执行,否则抛出异常(与Propagation.MANDATORY相反)
    @Transactional(propagation=Propagation.SUPPORTS) //如果其他bean调用这个方法,在其他bean中声明事务,那就用事务.如果其他bean没有声明事务,那就不用事务.
    
 
    @Transactional(propagation=Propagation.NESTED) 
    @Transactional (propagation =Propagation.REQUIRED,readOnly=true) //readOnly=true只读,不能更新,删除 
    @Transactional (propagation =Propagation.REQUIRED,timeout=30)//设置超时时间 
    @Transactional (propagation =Propagation.REQUIRED,isolation=Isolation.DEFAULT)//设置数据库隔离级别


用 spring 事务管理器,由spring来负责数据库的打开,提交,回滚.
默认遇到运行期例外(throw new RuntimeException("注释");)会回滚，即遇到不受检查（unchecked）的例外时回滚；
而遇到需要捕获的例外(throw new Exception("注释");)不会回滚,即遇到受检查的例外（就是非运行时抛出的异常，编译器会检查到的异常叫受检查例外或说受检查异常）时，需我们指定方式来让事务回滚 ,如下:
@Transactional(rollbackFor=Exception.class) //指定回滚,遇到异常Exception时回滚
    public void methodName() {
       throw new Exception("注释");
       
    }
@Transactional(noRollbackFor=Exception.class)//指定不回滚,遇到运行期例外(throw newRuntimeException("注释");)会回滚
    public ItimDaoImpl getItemDaoImpl() {
        throw new RuntimeException("注释");
    } 