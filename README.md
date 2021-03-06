### 项目介绍
数据库读写分离是再基础不过的需求了,读写分离通常有三种方案实现:
1. 多数据源,通过代码硬编码实现.
2. 修改ORM框架实现.
3. 实现数据库协议来实现.

方案一最简单,但是开发人员工作量最大,并且容易犯错;虽然方案三开发人员来说是透明的且不限制编程语言,但是开发难度最大且数据库的支持范围
较窄.本项目基于方案二,选择了Java中最流行的Mybatis和Spring来实现,所以只适用于基于Mybatis+Spring实现的Java项目.


### 功能
1. 事务,非readonly到主库,readonly到从库.
2. select到读库,insert/update/delete到主库.
3. 支持select强制路由到主库(尽量避免,通过业务逻辑优化来绕过).


### 稳定度
该项目在笔者公司,上百个项目中广泛应用,已经很成熟,测试代码中有详细的配置和测试代码.


### 核心配置

```
    <bean id="dataSource" class="info.yangguo.perseus.DynamicDataSource">
        <property name="master" ref="master"/>
        <property name="slaves">
            <list>
                <ref bean="slave1"/>
                <ref bean="slave2"/>
            </list>
        </property>
    </bean>

    <!-- ibatis3 工厂类 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="typeAliasesPackage" value="info.yangguo.perseus.test.domain"/>
        <property name="configLocation" value="classpath:sqlMapConfig.xml"/>
        <property name="mapperLocations" value="classpath:info/yangguo/perseus/test/dao/*.xml"/>
    </bean>

    <bean class="info.yangguo.perseus.MapperScannerConfigurer">
        <property name="basePackage" value="info.yangguo.perseus.test.dao"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>

    <!-- 定义单个jdbc数据源的事务管理器 -->
    <bean id="transactionManager"
          class="info.yangguo.perseus.DynamicDataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- 以 @Transactional 标注来定义事务  -->
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
```

