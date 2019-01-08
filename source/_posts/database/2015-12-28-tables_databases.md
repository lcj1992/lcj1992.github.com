---
layout: post
title: 数据库分库分表
categories: db
tags: mybatis
---




### 分表 

访问极为频繁且数量巨大的单表，可以考虑减少单表的记录条数，以便减少数据查询所需要的时间，提高数据库的吞吐。

两种方式:

#### 1.Param注解

dao:将表名传入

    public int save(@Param("table") String table, @Param("name") String name, @Param("gender") Integer gender);

mapper.xml  ***"$"动态传入表名 "#"传入列值,传入map***

    <insert id="save" parameterType="Map">
        insert into ${table} (name,gender)values(#{name},#{gender})
    </insert>

ps: $不对变量进行转义 ＃会对变量进行转义

#### 2.Model && Map

model:

    public class User {
        private Long id;
        private String table;
        private String name
        private Integer gender;
        ....getter setter
     }

dao:

    public int save(User user);

mapper.xml

    <insert id="save" parameterType="User">
        insert into ${table} (name,gender)values(#{name},#{gender})
    </insert>

ps:

1. 运价库的表是根据航班起飞日期来分的

### 分库 

分表能够解决单表数据量大带来的查询效率下降的问题，但是却无法给数据库的并发处理能力带来质的提升。高并发的读写访问，可以考虑分库。

#### db-context.xml

    <bean id="parentDatasource" class="xxxxx" destroy-method="close">
        ...
    </bean>

    <bean id="dataSource1" parent="parentDataSource">
        <property name="url" value="jdbc:mysql://localhost:3306/simple"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>

    <bean id="dataSource2" parent="parentPoolDataSource">
        <property name="url" value="jdbc:mysql://localhost:3306/simple2"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>

    <bean name="dynamicDataSource" class="com.xxx.web.util.DynamicDataSource">
        <property name="targetDataSources">
            <map key-type="java.lang.String">
                <entry value-ref="dataSource1" key="xxx1"/>
                <entry value-ref="dataSource2" key="xxx2"/>
            </map>
        </property>
    </bean>
        ...

#### DynamicDataSource.java

    public class DynamicDataSource extends AbstractRoutingDataSource {
        @Override
        protected Object determineCurrentLookupKey() {
            return AppContext.getContext();
        }
    }

#### AppContext.java

    public class AppContext(){
        private static InheritableThreadLocal<String> context = new InheritableThreadLocal<String>();

        public static void setContext(String context){        
            AppContext.context.set(context);    
        }

        public static String getContext(){        
            return AppContext.context.get();
        }
    }

ps:
1.  官网代购和国内旗舰店都是按照航司来分库滴

#### 分库分表 

有时数据库即可能面临高并发的压力，有需要面对海量数据的存储问题，可以考虑分库+分表。
