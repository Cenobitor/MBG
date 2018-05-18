# 1 如何集成通用的Mapper
注意：新版本已经不再使用拦截器的形式，只有两种：java配置的方式和spring配置的方式

## 1.1 添加依赖
```
<dependency>   
    <groupId>tk.mybatis</groupId>    
    <artifactId>mapper</artifactId>    
    <version>3.4.6</version>
 </dependency>
```
 
## 1.2 配置
### 1.2.2 spring配置方式xml的配置
```
<bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">     
    <property name="basePackage" value="com.isea533.mybatis.mapper"/>     
    <property name="properties">         
    <value>mappers=tk.mybatis.mapper.common.Mapper</value>     
    </property> 
</bean>
```
 
### 1.2.2 java的配置方式

1.通用的mapper依赖的添加

2.配置通用的mapper---->官方提供的是通过xml的方式配置 ，我们这里通过java类来配置，主要是通过@bean实现实例化一个bean到spring容器中

3.使用：

编写接口 需要extends Mapper<T>   而且必须有@mapper注解来修饰表示spring容器将来要扫描到这个类 生成代理对象

编写T 需要使用到jpa的注解
```
@SpringBootApplication//自动配置包扫描 在这个类所在的包下 或者子包下都起作用
@MapperScan(basePackages = "com.cenobitor.springboot.mapper")
public class SpringbootdemoApplication {

	//作用：就相当于初始化spring容器 启动容器并且发布
	public static void main(String[] args) {
		SpringApplication.run(SpringbootdemoApplication.class, args);
	}

	@Bean//创建一个类交给spring容器管理 <bean class="MapperScannerConfigurer"
	public MapperScannerConfigurer createConfigure(){
		MapperScannerConfigurer configurer = new MapperScannerConfigurer();
		configurer.setBasePackage("com.cenobitor.springboot.mapper");
		Properties properites = new Properties();
		properites.setProperty("mappers","tk.mybatis.mapper.common.Mapper");
		configurer.setProperties(properites);
		return configurer;
	}
}
```

**你没看错，就是这么配置的，注意这里是** `**tk.mybatis.xxx**`**,****和****MyBatis****的唯一区别就是** `**org.**` **改成了** `**tk.**`**，方便修改和记忆。**

**通用****Mapper****的各项属性通过** `**properties**` **属性进行配置，如果默认配置就是一行**`**mappers=tk.mybatis.mapper.common.Mapper**`**时，可以不写，就会变成：**

```
<bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">     
    <property name="basePackage" value="com.isea533.mybatis.mapper"/>        
 </bean>
```

没有特殊的使用要求，可以使用这样的配置项即可。

# 2 如何使用通用的Mapper

## 2.1 继承通用的Mapper<T>,必须指定泛型<T>

例如下面的例子:

```
public interface UserInfoMapper extends Mapper<UserInfo> {
  //其他必须手写的接口...

}
```

一旦继承了`Mapper<T>`,继承的`Mapper`就拥有了`Mapper<T>`所有的通用方法。

## 2.2 泛型(实体类)<T>的类型必须符合要求

实体类按照如下规则和数据库表进行转换,注解全部是JPA中的注解:

1. 表名默认使用类名,驼峰转下划线(只对大写字母进行处理),如`UserInfo`默认对应的表名为`user_info`。

2. 表名可以使用`@Table(name      = "tableName")`进行指定,对不符合第一条默认规则的可以通过这种方式指定表名.

3. 字段默认和`@Column`一样,都会作为表字段,表字段默认为Java对象的`Field`名字驼峰转下划线形式.

4. 可以使用`@Column(name      = "fieldName")`指定不符合第3条规则的字段名

5. 使用`@Transient`注解可以忽略字段,添加该注解的字段不会作为表字段使用.

6. **建议一定是有一个**`**@Id**`**注解作为主键的字段****,****可以有多个**`**@Id**`**注解的字段作为联合主键****.**

7. **默认情况下****,****实体类中如果不存在包含**`**@Id**`**注解的字段****,****所有的字段都会作为主键字段进行使用****(****这种效率极低****).**

8. 实体类可以继承使用,可以参考测试代码中的`tk.mybatis.mapper.model.UserLogin2`类.

9. 由于基本类型,如int作为实体类字段时会有默认值0,而且无法消除,所以实体类中建议不要使用基本类型.

10. `@NameStyle`注解，用来配置对象名/字段和表名/字段之间的转换方式，该注解优先于全局配置`style`，可选值：

11. - `normal`:使用实体类名/属性名作为表名/字段名
    - `camelhump`:**这是默认值**，驼峰转换为下划线形式
    - `uppercase`:转换为大写
    - `lowercase`:转换为小写

通过[使用Mapper专用的MyBatis生成器插件](http://git.oschina.net/free/Mapper/blob/master/wiki/mapper3/7.UseMBG.md)可以直接生成符合要求带注解的实体类。

## 2.3 重点强调 @Transient 注解

许多人由于不仔细看文档，频繁在这个问题上出错。

如果你的实体类中包含了不是数据库表中的字段，你需要给这个字段加上`@Transient`注解，这样通用Mapper在处理单表操作时就不会将标注的属性当成表字段处理！ 
 

# 3 MBG逆向工程生成插件

介绍逆向工程相关的配置项如下：
https://gitee.com/free/Mybatis_Utils/blob/master/MybatisGeneator/MybatisGeneator.md

## 3.1 使用逆向工程：

参考逆向工程：
|-MBG

## 3.2 配置项解释及搭建步骤

### 3.2.1 创建一个maven工程
|-MBG
	|--src
		|--- main
		|--- test
	|--pom.xml

### 3.2.2 加入依赖及相关的属性配置
```
<project xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"          
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0  http://maven.apache.org/xsd/maven-4.0.0.xsd">     
    <modelVersion>4.0.0</modelVersion>      
    <groupId>com.cenobitor</groupId>     
    <artifactId>MBG</artifactId>     
    <version>1.0-SNAPSHOT</version>     
    <packaging>jar</packaging>      
    <name>MBG</name>     
    <url>http://maven.apache.org</url>     
    <properties>         
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
        <targetJavaProject>${basedir}/src/main/java</targetJavaProject>
        <!-- mapper接口生成路径 -->  
        <targetMapperPackage>com.pinyougou.mapper</targetMapperPackage>  
        <!-- 实体生成路径 -->  
        <targetModelPackage>com.pinyougou.model</targetModelPackage>  
        <!--  XML映射文件生成路径  -->         
        <targetResourcesProject>${basedir}/src/main/resources</targetResourcesProject>         
        <targetXMLPackage>mapper</targetXMLPackage>         
        <!--  依赖版本  -->         
        <mapper.version>3.4.6</mapper.version>         
        <mysql.version>5.1.29</mysql.version>     
    </properties>    
       
    <dependencies>         
        <dependency>             
            <groupId>junit</groupId>             
            <artifactId>junit</artifactId>             
            <version>3.8.1</version>             
            <scope>test</scope>         
        </dependency>         
        <dependency>             
            <groupId>tk.mybatis</groupId>             
            <artifactId>mapper</artifactId>             
            <version>${mapper.version}</version>         
        </dependency>         
        <!-- https://mvnrepository.com/artifact/org.mybatis.generator/mybatis-generator-core -->         
        <dependency>             
            <groupId>org.mybatis.generator</groupId>             
            <artifactId>mybatis-generator-core</artifactId>             
            <version>1.3.2</version>         
        </dependency>      
    </dependencies>      
    <build>         
        <plugins>             
            <plugin>                
                <groupId>org.mybatis.generator</groupId>                 
                <artifactId>mybatis-generator-maven-plugin</artifactId>                 
                <version>1.3.2</version>                 
                <configuration>                     
                        <configurationFile>${basedir}/src/main/resources/generator/generatorConfig.xml</configurationFile>                     
                        <overwrite>true</overwrite>                     
                        <verbose>true</verbose>                 
                </configuration>                 
                <dependencies>                     
                        <dependency>                         
                            <groupId>mysql</groupId>                         
                            <artifactId>mysql-connector-java</artifactId>                         
                            <version>${mysql.version}</version>                     
                        </dependency>                     
                        <dependency>                        
                            <groupId>tk.mybatis</groupId>                        
                            <artifactId>mapper</artifactId>                         
                            <version>${mapper.version}</version>                     
                        </dependency>                 
                </dependencies>             
            </plugin>         
        </plugins>     
    </build> 
</project>
```

### 3.2.3 在resources下创建目录及配置文件
|- resources
|-- generator
|--- generatorConfig.xml
 
### 3.2.4 配置generatorConfig.xml
```
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE generatorConfiguration         
    PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"         
    "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">  
    
    <generatorConfiguration>     
        <properties resource="config.properties"/>      
        <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat"> 
               <property name="beginningDelimiter" value="`"/>         
               <property name="endingDelimiter" value="`"/>           
               
               <!--支持序列化-->         
               <plugin type="org.mybatis.generator.plugins.SerializablePlugin"></plugin>    
                     
               <plugin type="${mapper.plugin}">             
                   <property name="mappers" value="${mapper.Mapper}"/>             
                   <property name="forceAnnotation" value="${mapper.forceAnnotation}" />         
               </plugin>          
               <jdbcConnection driverClass="${jdbc.driverClass}"                         
                               connectionURL="${jdbc.url}"                        
                               userId="${jdbc.user}"                         
                               password="${jdbc.password}">         
               </jdbcConnection>          
           
               <javaModelGenerator targetPackage="${targetModelPackage}" 
                                   targetProject="${targetJavaProject}"/>          
               <sqlMapGenerator targetPackage="${targetXMLPackage}" 
                                   targetProject="${targetResourcesProject}"/>          
               <javaClientGenerator targetPackage="${targetMapperPackage}" 
                                    targetProject="${targetJavaProject}"                              
                                    type="XMLMAPPER"/>          
               
               <table tableName="%">             
                       <generatedKey column="id" sqlStatement="Mysql" identity="true"/>         
               </table>     
        </context> 
     </generatorConfiguration>
```
 
### 3.2.5 创建config.properties
```
# 数据库配置 
jdbc.driverClass=com.mysql.jdbc.Driver jdbc.url=jdbc:mysql://localhost:3306/springboot 
jdbc.user=root 
jdbc.password=itcast  

# 通用Mapper配置 
mapper.plugin=tk.mybatis.mapper.generator.MapperPlugin  
mapper.Mapper=tk.mybatis.mapper.common.Mapper
  mapper.forceAnnotation=true
```

### 3.2.6 运行maven命令

 mybatis-generator:generate
 
# 4 使用
使用接口进行增删改查,类似于JPA的用法,基本的增删改查,无需自己写;特殊的方法自己写于接口中,sql语句写在相应的方法的注解上;
 

