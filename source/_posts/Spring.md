---
title: SpringFrameWork
date: 2019-12-11 21:11:20
tags: 
    - Spring
---
# SpringFrameWork

## IoC container
* BeanFactory
  * ApplicationContext

> 初始化IoC Container

  ```java
    ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
  ```


> 获取Beans

  ```java
      // create and configure beans
    ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

    // retrieve configured instance
    PetStoreService service = context.getBean("petStore", PetStoreService.class);

    // use configured instance
    List<String> userList = service.getUsernameList();
  ```
  > 如果是Groovy定义的

  ```java
    ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
  ```
### Way of DI (Dependencies Injection)
* XML configuration
>  属性name元素引用JavaBean属性的名称，`ref`元素引用另一个bean定义的名称。
> `id`属性是一个字符串，用于标识单个bean定义。 `class`属性定义bean的类型并使用完全限定的classname

  ```XML
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans/http://www.springframework.org/schema/beans/spring-beans.xsd">
          <!-- id属性是一个字符串，用于标识单个bean定义的变量名。 class属性定义bean的类型并使用完全限定的classname-->
    <bean id="petStore"
     class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
  </beans>

  ```

 > 指定工厂方法来生成实例

  ```xml
  <bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
  ```

   ```java

  public class ClientService {
     private static ClientService clientService = new ClientService();
     private ClientService() {}

     public static ClientService createInstance() {
         return clientService;
     }
  }
   ```

* Groovy bean definition DSL

  ```java
  beans {
      dataSource(BasicDataSource) {
          driverClassName = "org.hsqldb.jdbcDriver"
          url = "jdbc:hsqldb:mem:grailsDB"
          username = "sa"
          password = ""
          settings = [mynew:"setting"]
      }
      sessionFactory(SessionFactory) {
          dataSource = dataSource
      }
      myService(MyService) {
          nestedBean = { AnotherBean bean ->
              dataSource = dataSource
          }
      }
  }
  ```
* Annotation-based configuration
* Java-based configuration
  ```Java
  @Configuration
  public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
      return new TransferServiceImpl();
    }
  }
  ```
  > 效果等同于

  ```xml
  <beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
  </beans>
  ```

## Bean属性
| Property  | Explained in…​  |
| :------------ |:---------------:|
| class      | Instantiating beans |
| name     | Naming beans        |
| scope | Bean scopes      |
| constructor arguments|  Dependency Injection    |
| properties | Dependency Injection     |
| autowiring mode | Autowiring collaborators     |
| lazy-initialization mode | Lazy-initialized beans     |
| initialization method | Initialization callbacks     |
| destruction method | Destruction callbacks     |
