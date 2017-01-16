# 这是 MyBatis Generator 插件的拓展插件包
应该说使用Mybatis就一定离不开[MyBatis Generator](https://github.com/mybatis/generator)这款代码生成插件，而这款插件自身还提供了插件拓展功能用于强化插件本身，官方已经提供了一些[拓展插件](http://www.mybatis.org/generator/reference/plugins.html)，本项目的目的也是通过该插件机制来强化Mybatis Generator本身，方便和减少我们平时的代码开发量。  
>因为插件是本人兴之所至所临时发布的项目（本人已近三年未做JAVA开发，代码水平请大家见谅），但基本插件都是在实际项目中经过检验的请大家放心使用，但因为项目目前主要数据库为MySQL，Mybatis实现使用Mapper.xml方式，所以代码生成时对于其他数据库和注解方式的支持未予考虑，请大家见谅。  
  
---------------------------------------
插件列表：  
* 查询单条数据插件（SelectOneByExamplePlugin）
* MySQL分页插件（LimitPlugin）
* 数据Model链式构建插件（ModelBuilderPlugin）
* Example Criteria 增强插件（ExampleEnhancedPlugin（CriteriaBuilderPlugin这个是老版本叫法，已经弃用））
* Example 目标包修改插件（ExampleTargetPlugin）
* 批量插入插件（BatchInsertPlugin）
* 逻辑删除插件（LogicalDeletePlugin）
  
---------------------------------------
Maven引用：  
```xml
<dependency>
  <groupId>com.itfsw</groupId>
  <artifactId>mybatis-generator-plugin</artifactId>
  <version>1.0.4</version>
</dependency>
```
---------------------------------------
### 1. 查询单条数据插件
对应表Mapper接口增加了方法  
插件：
```xml
<!-- 查询单条数据插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.SelectOneByExamplePlugin"/>
```
使用：  
```java
public interface TbMapper {    
    /**
     * 这是Mybatis Generator拓展插件生成的方法(请勿删除).
     * This method corresponds to the database table tb
     *
     * @mbg.generated
     * @author hewei
     */
    Tb selectOneByExample(TbExample example);
}
```
### 2. MySQL分页插件
对应表Example类增加了Mysql分页方法，limit(Integer rows)、limit(Integer offset, Integer rows)和page(Integer page, Integer pageSize)  
插件：
```xml
<!-- MySQL分页插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.LimitPlugin"/>
```
使用：  
```java
public class TbExample {
    /**
     * 这是Mybatis Generator拓展插件生成的属性(请勿删除).
     * This field corresponds to the database table tb
     *
     * @mbg.generated
     * @author hewei
     */
    protected Integer offset;

    /**
     * 这是Mybatis Generator拓展插件生成的属性(请勿删除).
     * This field corresponds to the database table tb
     *
     * @mbg.generated
     * @author hewei
     */
    protected Integer rows;
    /**
     * 这是Mybatis Generator拓展插件生成的方法(请勿删除).
     * This method corresponds to the database table rc_user_token
     *
     * @mbg.generated
     * @author hewei
     */
    public TbExample limit(Integer rows) {
        this.rows = rows;
        return this;
    }

    /**
     * 这是Mybatis Generator拓展插件生成的方法(请勿删除).
     * This method corresponds to the database table rc_user_token
     *
     * @mbg.generated
     * @author hewei
     */
    public TbExample limit(Integer offset, Integer rows) {
        this.offset = offset;
        this.rows = rows;
        return this;
    }

    /**
     * 这是Mybatis Generator拓展插件生成的方法(请勿删除).
     * This method corresponds to the database table tb
     *
     * @mbg.generated
     * @author hewei
     */
    public TbExample page(Integer page, Integer pageSize) {
        this.offset = page * pageSize;
        this.rows = pageSize;
        return this;
    }
    
    // offset 和 rows 的getter&setter
    
    // 修正了clear方法
    /**
     * This method was generated by MyBatis Generator.
     * This method corresponds to the database table tb
     *
     * @mbg.generated
     */
    public void clear() {
        oredCriteria.clear();
        orderByClause = null;
        distinct = false;
        rows = null;
        offset = null;
    }
}
public class Test {
    public static void main(String[] args) {
        this.tbMapper.selectByExample(
                new TbExample()
                .createCriteria()
                .andField1GreaterThan(1)
                .example()
                .limit(10)  // 查询前10条
                .limit(10, 10)  // 查询10~20条
                .page(1, 10)    // 查询第2页数据（每页10条）
        );
    }
}
```
### 3. 数据Model链式构建插件
这个是仿jquery的链式调用强化了表的Model的赋值操作  
插件：
```xml
<!-- 数据Model链式构建插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.ModelBuilderPlugin"/>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // 直接new表Model的内部Builder类，赋值后调用build()方法返回对象
        Tb table = new Tb.Builder()
               .field1("xx")
               .field2("xx")
               .field3("xx")
               .field4("xx")
               .build();
    }
}
```
### 4. Example 增强插件(example,andIf)
* Criteria的快速返回example()方法。  
* Criteria链式调用增强，以前如果有按条件增加的查询语句会打乱链式查询构建，现在有了andIf(boolean ifAdd, CriteriaAdd add)方法可一直使用链式调用下去。
* Example增强了setOrderByClause方法，新增orderBy(String orderByClause)方法直接返回example，增强链式调用，可以一路.下去了。
插件：
```xml
<!-- Example Criteria 增强插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.ExampleEnhancedPlugin"/>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // -----------------------------------example-----------------------------------
        // 表Example.Criteria增加了工厂方法example()支持，使用后可链式构建查询条件使用example()返回Example对象
        TbExample ex = new TbExample()
                    .createCriteria()
                    .andField1EqualTo(1)
                    .andField2EqualTo("xxx")
                    .example();
        this.tbMapper.selectByExample(ex);
        
        // -----------------------------------andIf-----------------------------------
        // Criteria增强了链式调用，现在一些按条件增加的查询条件不会打乱链式调用了
        // old
        TbExample oldEx = new TbExample();
        TbExample.Criteria criteria = oldEx
                .createCriteria()
                .andField1EqualTo(1)
                .andField2EqualTo("xxx");
        // 如果随机数大于0.5，附加Field3查询条件
        if (Math.random() > 0.5){
            criteria.andField3EqualTo(2)
                    .andField4EqualTo(new Date());
        }
        this.tbMapper.selectByExample(oldEx);

        // new
        this.tbMapper.selectByExample(
                new TbExample()
                        .createCriteria()
                        .andField1EqualTo(1)
                        .andField2EqualTo("xxx")
                        // 如果随机数大于0.5，附加Field3查询条件
                        .andIf(Math.random() > 0.5, new TbExample.Criteria.ICriteriaAdd() {
                            @Override
                            public TbExample.Criteria add(TbExample.Criteria add) {
                                return add.andField3EqualTo(2)
                                        .andField4EqualTo(new Date());
                            }
                        })
                        // 当然最简洁的写法是采用java8的Lambda表达式，当然你的项目是Java8+
                        .andIf(Math.random() > 0.5, add -> add
                                .andField3EqualTo(2)
                                .andField4EqualTo(new Date())
                        )
                        .example()
        );
        
        // -----------------------------------orderBy-----------------------------------
        // old
        TbExample ex = new TbExample();
        ex.createCriteria().andField1GreaterThan(1);
        ex.setOrderByClause("field1 DESC");
        this.tbMapper.selectByExample(ex);

        // new
        this.tbMapper.selectByExample(
                new TbExample()
                .createCriteria()
                .andField1GreaterThan(1)
                .example()
                .orderBy("field1 DESC")
        );
    }
}
```
### 5. Example 目标包修改插件
Mybatis Generator 插件默认把Model类和Example类都生成到一个包下，这样该包下类就会很多不方便区分，该插件目的就是把Example类独立到一个新包下，方便查看。  
插件：
```xml
<!-- Example 目标包修改插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.ExampleTargetPlugin">
    <!-- 修改Example类生成到目标包下 -->
    <property name="targetPackage" value="com.itfsw.mybatis.generator.dao.example"/>
</plugin>
```
### 6. 批量插入插件
提供了一个批量插入batchInsert方法，因为方法使用了使用了JDBC的getGenereatedKeys方法返回插入主键，所以只能在MYSQL和SQLServer下使用。  
插件：
```xml
<!-- 批量插入插件 -->
<plugin type="com.itfsw.mybatis.generator.plugins.BatchInsertPlugin"/>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // 构建插入数据
        List<Tb> list = new ArrayList<>();
        list.add(new Tb.Builder()
                .field1(0)
                .field2("xx0")
                .field3(0)
                .field4(new Date())
                .build()
        );
        list.add(new Tb.Builder()
                .field1(1)
                .field2("xx1")
                .field3(1)
                .field4(new Date())
                .build()
        );
        int inserted = this.tbMapper.batchInsert(list);
    }
}
```
### 7. 逻辑删除插件
因为很多实际项目数据都不允许物理删除，多采用逻辑删除，所以单独为逻辑删除做了一个插件，方便使用。  
插件：
```xml
<xml>
    <!-- 逻辑删除插件 -->
    <plugin type="com.itfsw.mybatis.generator.plugins.LogicalDeletePlugin">
        <!-- 这里配置的是全局逻辑删除列和逻辑删除值，当然在table中配置的值会覆盖该全局配置 -->
        <!-- 逻辑删除列类型只能为数字、字符串或者布尔类型 -->
        <property name="logicalDeleteColumn" value="del_flag"/>
        <!-- 未设置该属性或者该属性的值为null或者NULL,逻辑删除时会把该字段置为NULL。 -->
        <property name="logicalDeleteValue" value="9"/>
    </plugin>
    
    <table tableName="tb">
        <!-- 这里可以单独表配置逻辑删除列和删除值，覆盖全局配置 -->
        <property name="logicalDeleteColumn" value="del_flag"/>
        <property name="logicalDeleteValue" value="1"/>
    </table>
</xml>
```
使用：  
```java
public class Test {
    public static void main(String[] args) {
        // 1. 逻辑删除ByExample
        TbExample ex = new TbExample()
                .createCriteria()
                .andField1EqualTo(1)
                .example();
        
        this.tbMapper.logicalDeleteByExample(ex);
        
        // 2. 逻辑删除ByPrimaryKey
        this.tbMapper.logicalDeleteByPrimaryKey(1L);
        
        // 3. 同时Example中提供了一个快捷方法来过滤逻辑删除数据
        TbExample selEx = new TbExample()
                .createCriteria()
                .andField1EqualTo(1)
                // 新增了一个andDeleted方法过滤逻辑删除数据
                .andDeleted(true)
                // 当然也可直接使用逻辑删除列的查询方法，我们数据Model中定义了一个逻辑删除常量DEL_FLAG
                .andDelFlagEqualTo(Tb.DEL_FLAG)
                .example();
        
        List<Tb> list = this.tbMapper.selectByExample(selEx);
    }
}
```