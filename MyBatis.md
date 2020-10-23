# MyBatis——mapper.xml文件

## 一、标签分类

- 定义SQL语句
  - insert
  - delete
  - update
  - select
- 配置关联关系
  - collection
  - association
- 配置java对象属性与查询结果集中列名的对应关系
  - resultMap
- 控制动态SQL拼接
  - foreach
  - if
  - choose
- 格式化输出
  - where
  - set
  - trim
- 定义常量
  - sql
- 其他
  - include

## 二、标签总结

### 1. 基础SQL标签

#### 1.1 查询select

- 标签属性
  - **id** 唯一的名称，对应dao中mapper的接口名称
  
  - **paramterType** 定义传入的参数类型
  
  - **resultType** 返回数据类型对应实体类
  
  - **resultMap** 外部 resultMap 的命名引用。结果集的映射是 MyBatis 最强大的特性，使用 resultMap 或 resultType，但不能同时使用
  
  - **flushCache** 将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false
  
  - **useCache** 将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true
  
  - **timeout** 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）
  
  - **fetchSize** 这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动）。
  
  - **statementType** STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement使用$符号，PreparedStatement使用#符号 或 CallableStatement，默认值：PREPARED。
  
  - **resultSetType** FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）。
  
  - **databaseId** 如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。
  
  - **resultOrdered** 这个设置仅针对嵌套结果 select 语句适用：如果为true，就是假设包含了嵌套结果集或是分组了，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。这就使得在获取嵌套的结果集的时候不至导致内存不够用。默认值：false。
  
  - **resultSets** 这个设置仅对多结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称是逗号分隔的。
  
  - @Param 注解说明：1.多个参数时，修饰参数本省
  
    ```xml
    public List<User> test(@Param("name") String name,@Param("pwd") String password);
    
    <select id = "test" resultType="com.mbm.User">
    		select * from user where name=#{name} and password=#{pwd};
    </select>
    ```
  
    2.采用@Param修饰对象
  
    ```xml
    public User test(@Param("user") User user);
    
    <select id = "test" resultType="com.mbm.User">
    		select * from user where name=#{user.name}
    </select>
    
    当参数单一时，可以不用@Param：public User test(User user);
    
    <select id = "test" resultType="com.mbm.User">
    		select * from user where name=#{name};//参数是对象时，可以直接使用其中成员变量
    </select>
    ```
  
    

```xml
    /**
     * 根据条件查询用户集合
     */
    List<User> selectUsers(@Param("user")Map<String, Object> map);

    <!-- 返回的是List，resultType给定的值是List里面的实体类而不是list，mybatis会自动把结果变成List -->
    <select id="selectUsers" parameterType="map" resultType="con.mbm.User">
        select id, username, password, sex, birthday, address from user u
        <where>
            <trim suffixOverrides=",">//suffixOverrides：去除语句后缀
                <if test="user.username != null and user.username != ''">
                    u.username = #{user.username},
                </if>
                <if test="user.sex != null">
                    and u.sex = #{user.sex},
                </if>
                 <if test="user.beginTime != null">
                    <![CDATA[  and DATE_FORMAT(u.birthday, '%Y-%m-%d %H:%T:%s') >= DATE_FORMAT(#{beginTime}, '%Y-%m-%d %H:%T:%s')，   ]]>
                </if>
                <if test="user.endTime != null">
                    <![CDATA[  and DATE_FORMAT(u.birthday, '%Y-%m-%d %H:%T:%s') <= DATE_FORMAT(#{endTime}, '%Y-%m-%d %H:%T:%s')，   ]]>
                </if>
                <if test="user.address != null and user.address != ''">
                    and u.addrerss like '%' || #{user.address} || '%',//由于suffixOverrides=","存在，则，被去除
                </if>
            </trim>
        </where>
    </select>
  //like '%' || #{user.address} || '%' 
      
      //模糊查询
      like '%${user.address}%' 或者 LIKE CONCAT('%',#{user.address},'%')
```

#### 1.2 增删改

- 标签属性
  - **id** 唯一的名称，对应dao中mapper的接口名称
  - **parameterType** 将要传入语句的参数的完全限定类名或别名。这个属性是可选的，因为 MyBatis 可以通过 TypeHandler 推断出具体传入语句的参数，默认值为 unset。
  - **flushCache** 将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：true（对应插入、更新和删除语句）。
  - **timeout** 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。
  - **statementType** STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。
  - **useGeneratedKeys**（仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server这样的关系数据库管理系统的自动递增字段, oracle使用序列是不支持的，通过selectKey可以返回主键），默认值：false。
  - **keyProperty** （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey子元素设置它的键值，默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。
  - **keyColumn**（仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。
  - **databaseId** 如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。

```xml
    <insert id="insert" parameterType="com.mbm.User">
    
        <!-- 使用序列插入oracle数据库返回主键 -->
        <selectKey resultType="long" order="BEFORE" keyProperty="id">//oracle先取出主键再插入数据库
            SELECT user_seq.NEXTVAL as id from DUAL
        </selectKey>
        
        insert into User (ID, USERNAME, PASSWORD, SEX, ADRESS, CREATED_BY, CREADTED_DATE)
        values (#{id}, #{username}, #{password}, #{sex}, #{adress}, #{createdBy}, SYSDATE)
    </insert>

<!-- 使用序列插入MySQL数据库返回主键 -->
<insert id = "insert" parameterType="com.mbm.User" useGeneratedKeys = "true" keyProperty = "id">
 insert into User (ID, USERNAME, PASSWORD, SEX, ADRESS, CREATED_BY, CREADTED_DATE)
        values (#{id}, #{username}, #{password}, #{sex}, #{adress}, #{createdBy}, SYSDATE)
</insert>

或者
    <insert id="insert" parameterType="com.mbm.User">
        <selectKey resultType="long" order="AFTER" keyProperty="id">//MySQL先插入数据库后获取主键
            SELECT LAST_INSERT_ID()
        </selectKey>
        insert into User (ID, USERNAME, PASSWORD, SEX, ADRESS, CREATED_BY, CREADTED_DATE)
        values (#{id}, #{username}, #{password}, #{sex}, #{adress}, #{createdBy}, SYSDATE)
    </insert>
```

#### 1.3 其他基础标签

###### 1.3.1 sql 标签

定义一些常用的sql语句片段

```xml
<sql id="selectParam">
    id, username, password, sex, birthday, address
</sql>
```

###### 1.3.2 include 标签

引用其他的常量，通常和sql一起使用

```xml
<select>
    select <include refid="selectParam"></include>
    from user
</select>

这里等同于
select id, username, password, sex, birthday, address from user
```

###### 1.3.3 if 标签

*动态语句，如果判断不成立，则字段不被写入数据库，默认为null，在不用动态判断时，则写入“ ”

基本都是用来判断值是否为空，**注意Integer的判断，mybatis会默认把0变成** ‘ ’

```xml
<if test="item != null and item != ''"></if>

<!-- 如果是Integer类型的需要把and后面去掉或是加上or-->
<if test="item != null"></if>
<if test="item != null and item != '' or item == 0"></if>
```

###### 1.3.4 别名

经常使用的类型可以定义别名，方便使用，mybatis也注册了很多别名方便我们使用，详情见底部

```xml
<typeAliases>
     <typeAlias type="com.mbm.User" alias="User"/>
</typeAliases>
```

### 2. collection与association标签

collection与association的属性一样，都是用于resultMap返回关联映射使用，collection关联的是集合，而association是关联单个对象

- 标签属性
  - **property** resultMap返回实体类中字段和result标签中的property一样
  - **column** 数据库的列名或者列标签别名,是关联查询往下一个语句传送值。注意： 在处理组合键时，您可以使用column=“{prop1=col1,prop2=col2}”这样的语法，设置多个列名传入到嵌套查询语句。这就会把prop1和prop2设置到目标嵌套选择语句的参数对象中。
  - **javaType** 一般为ArrayList或是java.util.List
  - **ofType** java的实体类，对应数据库表的列名称，即关联查询select对应返回的类
  - **select** 执行一个其他映射的sql语句返回一个java实体类型

```java
/**
  *问题表
  */
public class Question {
    
    private Long id; //问题id

    private String question; //问题
    
    private Integer questionType; //问题类型
    
    private List<QuestionAnswer> answerList; //问题选项集合
    
    //Getter和Setter省略
}

/**
  *问题选项表
  */
public class QuestionAnswer {
    
    private Long id; //选项id
    
    private Long questionId;  //问题id

    private String answer; //选项
    
    //Getter和Setter省略
}

```

### 3. resultMap标签

- resultMap属性
  - **id** 唯一标识
  - **type** 返回类型
  - **extends** 继承别的resultMap，可选
- 关联其他标签
  - **id** 设置主键使用，使用此标签配置映射关系（可能不止一个）
  - **result** 一般属性的配置映射关系，一般不止一个
  - **property** 表示实体类属性名
  - **association** 关联一个对象使用
  - **collection** 关联一个集合使用

```xml
<!-- 返回关联查询的问题 -->
<resultMap id="detail_result" type="com.Question">
    <id column="id" property="id" />
    <result column="question" property="question" />
    <result column="question_type" property="questionType" />
    <collection property="answerList" javaType="java.util.List"
                ofType="com.it.bean.QuestionAnswer" column="id" 
                select="setlectQuestionAnswerByQuestionId"/>
</resultMap>

<!-- 查询问题集 -->
<select id="selectQuestions" parameterType="map" resultMap="detail_result">
    select q.id, q.question, q.question_type 
    from question q 
    <where>
        <if test="cond.id != null">
            q.id = #{question.id}
        </if>
        <if test="question.idList != null and question.idList.size() != 0">
            q.id in 
            <foreach collection="question.idList" item="id" open="(" separator="," close=")">
                #{id}
            </foreach>
        </if>
    </where>
</select>

<!-- 查询对应问题的答案集 -->
<select id="setlectQuestionAnswerByQuestionId" parameterType="long" resultType="com.it.bean.QuestionAnswer">
    select a.id, a.answer from question_answer a where a.question_id = #{id}
</select>
```

### 4. foreach标签

- foreach属性
  - **collection** 循环的集合。传的是集合为list，数组为array, 如果是map为java.util.HashMap
  - **item** 循环的key
  - **index** 循环的下表顺序
  - **open** 循环的开头
  - **close** 循环结束
  - **separator** 循环的分隔符

```xml
<sql id="base_column">id, question_id, answer</sql>
<!-- Mysql的批量插入，主键自增 -->
<insert id="insertBatchMysql" parameterType="list">
    insert into question_answer ( <include refid="base_column" /> ) 
    values 
        <foreach collection="list" item="item" open="(" separator="union all" close=")">
            #{item.id}, #{item.questionId}, #{item.answer}
        </foreach>
</insert>
```

### 5. where标签

where用来去掉多条件查询时，开头多余的and

```xml
    <select id="selectUserList" parameterType="com.it.bean.User" resultType="com.mbm.User">
        <!-- 引用Sql片段 -->
        select <include refid="selectParam"> from user u
        <where>
            <!--where 可以自动去掉条件中的第一个and-->
            <if test="id != null">
                and u.id = #{id}
            </if>
            <if test="name != null and name != ''">
                and u.name = #{name}
            </if>
        </where>
    </select>
```

### 6. set标签

set是mybatis提供的一个智能标记，当在update语句中使用if标签时，如果前面的if没有执行，则或导致逗号多余错误。使用set标签可以将动态的配置SET 关键字，和剔除追加到条件末尾的任何不相关的逗号。
没有使用if标签时，如果有一个参数为null，都会导致错误，如下示例：

```xml
    <update id="updateUser" parameterType="com.user.user">
        update user
        <set>
            <if test="username != null and username != ''">
                username = #{username},
            </if>
            <if test="sex != null and sex == 0 or sex == 1">
                sex = #{sex},
            </if>
            <if test="birthday != null ">  
                birthday = #{birthday},
            </if >  
            <if test="address != null and address != ''">
                address = #{address},
            </if>
            <if test="lastModifiedBy != null and lastModifiedBy != ''">
                last_modified_by = #{lastModifiedBy},
                last_modified_date = SYSDATE,
            </if>
        </set>
        <where>
            id = #{id}
        </where>
    </update>
```

### 7. trim标签

trim标记是一个格式化的标记，可以完成set或者是where标记的功能

- 标签属性
  - **prefix、suffix** 表示trim标签部分的前面或后面添加内容（注意：是没有prefixOverrides，suffixOverrides的情况下）
  - **prefixOverrides，suffixOverrides** 表示覆盖内容，如果只有这两个属性表示删除内容

```xml
<update id="test" parameterType="com.it.bean.User">
    update user
        <!-- 开头加上set，结尾去除最后一个逗号 -->
        <trim prefix="set" suffixOverrides=",">
            <if test="username!=null and username != ''">
                name= #{username},
            </if>

            <if test="password!=null and password != ''">
                password= #{password},
            </if>

        </trim>
        <where>
            id = #{id}
        </where>
    </update>
```

### 8. choose、when、otherwise标签

有时候我们并不想应用所有的条件，而只是想从多个选项中选择一个。MyBatis提供了choose 元素，按顺序判断when中的条件出否成立，如果有一个成立，则choose结束。当choose中所有when的条件都不满则时，则执行 otherwise中的sql。类似于Java 的switch 语句，choose为switch，when为case，otherwise则为default。if是与(and)的关系，而choose是或（or）的关系

```xml
<select id="getUserList" resultType="com.mbm.User" parameterType="com.mbm.User">  
    SELECT <include refid="resultParam"></include> FROM User u   
    <where>  
        <choose>  
            <when test="username !=null and username != ''">  
                u.username LIKE CONCAT(CONCAT('%', #{username}),'%')  
            </when >  
            <when test="sex != null">  
                AND u.sex = #{sex}  
            </when >  
            <when test="birthday != null ">  
                AND u.birthday = #{birthday}  
            </when >  
            <otherwise>  
            </otherwise>  
        </choose>  
    </where>    
</select>  
```

> 附Mybatis已经注册好的别名表

| 别名       | 映射类型   |
| ---------- | ---------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| map        | Map        |
| hashmap    | HashMap    |
| list       | list       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |