##### 本质

​	当处理一些事务的时候有可能会连续执行几个步骤，其中任何一个步骤出错都导致整个步骤执行失败。

​	比如说我想在数据库中插入一条消息并删除一条消息，如果删除消息失败了，那么也将导致插入消息失败，而不是插入消息成功，删除消息失败。

​	**相当于保证原子性**



##### 步骤

 1.  配置声明式事务

     ```xml
     <!--    配置声明式事务-->
         <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
             <property name="dataSource" ref="dataSource"></property>
         </bean>
     ```

	2. ```xml
    <!--    结合AOP实现事务织入-->
    <!--    配置事务通知-->
        <tx:advice id="txAdvice" transaction-manager="transactionManager">
    <!--        给哪些方法配置事务-->
    <!--        配置事务的传播特性 new propagation=&#45;&#45;-->
            <tx:attributes>
                <tx:method name="add" propagation="REQUIRED"></tx:method>
                <tx:method name="query" read-only="true"></tx:method>
                <tx:method name="delete" propagation="REQUIRED"/>
                <tx:method name="update" propagation="REQUIRED"/>
                <tx:method name="*" propagation="REQUIRED"/>
            </tx:attributes>
        </tx:advice>
       ```

	3. 配置事务切入

    ```xml
    <!--    配置事务切入-->
        <aop:config>
            <aop:pointcut id="txPointCut" expression="execution(* mapper.*.*(..))"/>
            <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"></aop:advisor>
        </aop:config>
    ```