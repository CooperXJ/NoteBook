#### Mybatis Generator的作用

自动生成dao层，mapper以及对应的xml文件



#### 使用步骤

1. 添加maven

   ```xml
   <!-- MyBatis 生成器 -->
   <dependency>
       <groupId>org.mybatis.generator</groupId>
       <artifactId>mybatis-generator-core</artifactId>
       <version>1.3.7</version>
   </dependency>
   ```

2. 编写GeneratorConfig配置文件（其中可以使用外部的properties文件进行配置，比如数据源的配置）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <properties resource="generator.properties"/>
    <context id="MySqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        <!--beginningDelimiter和endingDelimiter：指明数据库的用于标记数据库对象名的符号，比如ORACLE就是双引号，MYSQL默认是`反引号； -->
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
        <property name="javaFileEncoding" value="UTF-8"/>
        <!-- 为模型生成序列化方法-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <!-- 为生成的Java模型创建一个toString方法 -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>
        <!--可以自定义生成model的代码注释-->
        <commentGenerator type="com.cooper.mbg.CommentGenerator">
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="true"/>
            <property name="addRemarkComments" value="true"/>
        </commentGenerator>
        <!--配置数据库连接-->
        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.connectionURL}"
                        userId="${jdbc.userId}"
                        password="${jdbc.password}">
            <!--解决mysql驱动升级到8.0后不生成指定数据库代码的问题-->
            <property name="nullCatalogMeansCurrent" value="true" />
        </jdbcConnection>
        <!--指定生成model的路径-->
        <javaModelGenerator targetPackage="com.cooper.mbg.model" targetProject="./src/main/java"/>
        <!--指定生成mapper.xml的路径-->
        <sqlMapGenerator targetPackage="com.cooper.mall.tiny.mbg.mapper" targetProject="./src/main/resources"/>
        <!--指定生成mapper接口的的路径-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.cooper.mbg.mapper"
                             targetProject="./src/main/java"/>
        <!--生成全部表tableName设为%-->
        <table tableName="pms_brand">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
    </context>
</generatorConfiguration>
```

需要注意的是在指定生成的接口的地址

	- mac   需要是 ’/‘ 而不是 ’\‘ 
	- win    ’\‘

3. 编写生成类（固定的写法）

```java
public class Generator {
    public static void main(String[] args) throws Exception {
        //MBG 执行过程中的警告信息
        List<String> warnings = new ArrayList<String>();
        //当生成的代码重复时，覆盖原代码
        boolean overwrite = true;
        //读取我们的 MBG 配置文件
        InputStream is = Generator.class.getResourceAsStream("/generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(is);
        is.close();

        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        //创建 MBG
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        //执行生成代码
        myBatisGenerator.generate(null);
        //输出警告信息
        for (String warning : warnings) {
            System.out.println(warning);
        }
    }
}
```

	1. 如果需要自定义自动生成类的实体类的注释则可以重新DefaultCommentGenerator的方法

    ```java
    /**
     * 自定义注释生成器
     */
    public class CommentGenerator extends DefaultCommentGenerator {
        private boolean addRemarkComments = false;
        private static final String EXAMPLE_SUFFIX="Example";
        private static final String API_MODEL_PROPERTY_FULL_CLASS_NAME="io.swagger.annotations.ApiModelProperty";
    
        /**
         * 设置用户配置的参数
         */
        @Override
        public void addConfigurationProperties(Properties properties) {
            super.addConfigurationProperties(properties);
            this.addRemarkComments = StringUtility.isTrue(properties.getProperty("addRemarkComments"));
        }
    
        /**
         * 给字段添加注释
         */
        @Override
        public void addFieldComment(Field field, IntrospectedTable introspectedTable,
                                    IntrospectedColumn introspectedColumn) {
            String remarks = introspectedColumn.getRemarks();
            //根据参数和备注信息判断是否添加备注信息
            if(addRemarkComments&&StringUtility.stringHasValue(remarks)){
    //            addFieldJavaDoc(field, remarks);
                //数据库中特殊字符需要转义
                if(remarks.contains("\"")){
                    remarks = remarks.replace("\"","'");
                }
                //给model的字段添加swagger注解
                field.addJavaDocLine("@ApiModelProperty(value = \""+remarks+"\")");
            }
        }
    
        /**
         * 给model的字段添加注释
         */
        private void addFieldJavaDoc(Field field, String remarks) {
            //文档注释开始
            field.addJavaDocLine("/**");
            //获取数据库字段的备注信息
            String[] remarkLines = remarks.split(System.getProperty("line.separator"));
            for(String remarkLine:remarkLines){
                field.addJavaDocLine(" * "+remarkLine);
            }
            addJavadocTag(field, false);
            field.addJavaDocLine(" */");
        }
    
        @Override
        public void addJavaFileComment(CompilationUnit compilationUnit) {
            super.addJavaFileComment(compilationUnit);
            //只在model中添加swagger注解类的导入
         if(!compilationUnit.isJavaInterface()&&!compilationUnit.getType().getFullyQualifiedName().contains(EXAMPLE_SUFFIX)){
                compilationUnit.addImportedType(new FullyQualifiedJavaType(API_MODEL_PROPERTY_FULL_CLASS_NAME));
            }
        }
    }
    ```



#### 生成的***Example类的作用

作用：mybatis-generator会为每个字段产生Criterion，为底层的mapper.xml创建动态sql。如果表的字段比较多,产生的example类会十分庞大。理论上通过example类可以构造你想到的任何筛选条件。

Example类的成员:

```java
    //升序还是降序:字段+空格+asc(desc)
    protected String orderByClause;
    //去除重复:true是选择不重复记录,false,反之
    protected boolean distinct;
    //自定义查询条件
    protected List<Criteria> oredCriteria;
```

*需求*:根据用户名查询查询user
*sql*:select id, username, birthday, sex, address from user WHERE ( username = ‘张三’ ) order by username asc

```java
@Test
    public void testFindUserByName(){

        //通过criteria构造查询条件
        UserExample userExample = new UserExample();
        userExample.setOrderByClause("username asc"); //asc升序,desc降序排列
        userExample.setDistinct(false); //去除重复,true是选择不重复记录,false反之
        UserExample.Criteria criteria = userExample.createCriteria(); //构造自定义查询条件
        criteria.andUsernameEqualTo("张三");

        //自定义查询条件可能返回多条记录,使用List接收
        List<User> users = userMapper.selectByExample(userExample);

        System.out.println(users);
    }
```

