# MyBatis使用association实现一对一级联查询的几种案例

我们平日经常会遇到需要级联查询的场景，这里通过案例给大家展示实现过程。我们要查询的用户信息里面有个角色信息，一个用户对应一个角色，我们现在要求查出用户信息的同时，关联查出用户的角色信息，那么这个时候我们可以通过级联属性的方式，将角色中的数据查出来封装到用户User的role属性里面，我们还有另外一种方式来实现数据的封装。接下来我们将介绍一下association标签的相关用法，包括**嵌套查询**和**分段查询**两种方式。

准备工作：



#### **在Oracle数据库新建两张数据库表t_user和t_role，并插入若干条数据**

```html
CREATE TABLE t_user (
    id NUMBER(2) NOT NULL,
    userName varchar2(100) DEFAULT NULL,
    roleId NUMBER(2),
    note varchar2(255) DEFAULT NULL,
    PRIMARY KEY (id)
  ) 
INSERT INTO t_user(id,userName,roleId,note) VALUES (1, '张三', 1, '负责编码');
INSERT INTO t_user(id,userName,roleId,note) VALUES (2, '李四', 2, '负责测试');
INSERT INTO t_user(id,userName,roleId,note) VALUES (3, '王五', 3, '产品设计');
CREATE TABLE t_role (
  id NUMBER(2) NOT NULL,
  roleName varchar(20) DEFAULT NULL,
  PRIMARY KEY (id)
)
INSERT INTO t_role(id,roleName) VALUES (1,'JAVA工程师');
INSERT INTO t_role(id,roleName) VALUES (2,'测试工程师');
INSERT INTO t_role(id,roleName) VALUES (3,'产品经理');
```

#### **新建JavaBean类 User**

```html
/**
 * 博客地址  https://blog.csdn.net/guobinhui
 */
package com.hoomsun.wealth.crm.model;

/**
 * 作者：guo bin hui <br>
 * 创建时间：2018年7月3日 <br>
 * 描述：
 */
public class User {

	// ID，唯一性
	private int id;
	// 用户名
	private String userName;
	// 角色
	private Role role;
	// 备注
	private String note;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getUserName() {
		return userName;
	}
	public void setUserName(String userName) {
		this.userName = userName;
	}
	public Role getRole() {
		return role;



	}



	public void setRole(Role role) {
		this.role = role;
	}
	public String getNote() {
		return note;
	}
	public void setNote(String note) {
		this.note = note;
	}
}
```

#### **新建JavaBean类 Role**

```html
/**
 * 博客地址  https://blog.csdn.net/guobinhui
 */
package com.hoomsun.wealth.crm.model;

/**
 * 作者：guo bin hui <br>
 * 创建时间：2018年7月3日 <br>



 * 描述：



 */



public class Role {



 



	private int id;



	private String roleName;



	public int getId() {



		return id;



	}



	public void setId(int id) {



		this.id = id;



	}



	public String getRoleName() {



		return roleName;



	}



	public void setRoleName(String roleName) {



		this.roleName = roleName;



	}




```

#### **新建接口类UserMapper.java，增加接口方法getUserById**

```html
/**



 * 博客地址  https://blog.csdn.net/guobinhui



 */



package com.hoomsun.wealth.crm.dao;



 



import com.hoomsun.wealth.crm.model.User;



 



/**



 * 作者：Administrator <br>



 * 创建时间：2018年7月3日 <br>



 * 描述：



 */



public interface UserMapper {



 



	public User getUserById(Integer id);



}
```

接下来我们将分步介绍**嵌套查询**实现方式

**方式一**

#### **UserMapper.xml文件如下：**

```html
MyBatis使用association实现一对一级联查询的几种案例

guobinhui 2018-07-03 17:44:20  1997  收藏
分类专栏： Java基础 编程笔记
版权
我们平日经常会遇到需要级联查询的场景，这里通过案例给大家展示实现过程。我们要查询的用户信息里面有个角色信息，一个用户对应一个角色，我们现在要求查出用户信息的同时，关联查出用户的角色信息，那么这个时候我们可以通过级联属性的方式，将角色中的数据查出来封装到用户User的role属性里面，我们还有另外一种方式来实现数据的封装。接下来我们将介绍一下association标签的相关用法，包括嵌套查询和分段查询两种方式。

准备工作：


在Oracle数据库新建两张数据库表t_user和t_role，并插入若干条数据
CREATE TABLE t_user (
    id NUMBER(2) NOT NULL,
    userName varchar2(100) DEFAULT NULL,
    roleId NUMBER(2),
    note varchar2(255) DEFAULT NULL,
    PRIMARY KEY (id)
  ) 
INSERT INTO t_user(id,userName,roleId,note) VALUES (1, '张三', 1, '负责编码');
INSERT INTO t_user(id,userName,roleId,note) VALUES (2, '李四', 2, '负责测试');
INSERT INTO t_user(id,userName,roleId,note) VALUES (3, '王五', 3, '产品设计');
CREATE TABLE t_role (
  id NUMBER(2) NOT NULL,
  roleName varchar(20) DEFAULT NULL,
  PRIMARY KEY (id)
)
INSERT INTO t_role(id,roleName) VALUES (1,'JAVA工程师');
INSERT INTO t_role(id,roleName) VALUES (2,'测试工程师');
INSERT INTO t_role(id,roleName) VALUES (3,'产品经理');
新建JavaBean类 User
/**
 * 博客地址  https://blog.csdn.net/guobinhui
 */
package com.hoomsun.wealth.crm.model;
 
/**
 * 作者：guo bin hui <br>
 * 创建时间：2018年7月3日 <br>
 * 描述：
 */
public class User {
 
	// ID，唯一性
	private int id;
	// 用户名
	private String userName;
	// 角色
	private Role role;
	// 备注
	private String note;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getUserName() {
		return userName;
	}
	public void setUserName(String userName) {
		this.userName = userName;
	}
	public Role getRole() {
		return role;
	}
	public void setRole(Role role) {
		this.role = role;
	}
	public String getNote() {
		return note;
	}
	public void setNote(String note) {
		this.note = note;
	}
}
新建JavaBean类 Role
/**
 * 博客地址  https://blog.csdn.net/guobinhui
 */
package com.hoomsun.wealth.crm.model;
 
/**
 * 作者：guo bin hui <br>
 * 创建时间：2018年7月3日 <br>
 * 描述：
 */
public class Role {
 
	private int id;
	private String roleName;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getRoleName() {
		return roleName;
	}
	public void setRoleName(String roleName) {
		this.roleName = roleName;
	}
}
新建接口类UserMapper.java，增加接口方法getUserById
/**
 * 博客地址  https://blog.csdn.net/guobinhui
 */
package com.hoomsun.wealth.crm.dao;
 
import com.hoomsun.wealth.crm.model.User;
 
/**
 * 作者：Administrator <br>
 * 创建时间：2018年7月3日 <br>
 * 描述：
 */
public interface UserMapper {
 
	public User getUserById(Integer id);
}
接下来我们将分步介绍嵌套查询实现方式

方式一

UserMapper.xml文件如下：
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hoomsun.wealth.crm.dao.UserMapper">
	<resultMap id="userResultMap" type="com.hoomsun.wealth.crm.model.User">
		<id column="id" jdbcType="INTEGER" property="id" />
		<result column="userName" jdbcType="VARCHAR" property="userName" />
		<result column="note" jdbcType="DECIMAL" property="note" />		
	   <!--assocication可以指定联合的JavaBean对象 
               property="role"指定关联查询的Role 类型的role 属性
               javaType:指定这个role属性对象的类型
            -->	
	    <association property="role" javaType="com.hoomsun.wealth.crm.model.Role">
	    	<id column="role_id" property="id"/>
	    	<result column="roleName" property="roleName"/>
	    </association>
	</resultMap>
 
	<select id="getUserById" resultMap="userResultMap" parameterType="java.lang.Integer">
		select u.id id,
		u.userName userName,u.note note,r.id role_id, r.roleName roleName
		from t_user u left join t_role r on u.roleId=r.id
		where u.id=#{id}
	</select>
</mapper>
方式二

<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hoomsun.wealth.crm.dao.UserMapper">
	<resultMap id="userResultMap" type="com.hoomsun.wealth.crm.model.User">
		<id column="id" jdbcType="INTEGER" property="id" />
		<result column="userName" jdbcType="VARCHAR" property="userName" />
		<result column="note" jdbcType="DECIMAL" property="note" />	
		<result property="role.id" column="role_id" />
		<result property="role.roleName" column="roleName" />			
	</resultMap>
 
	<select id="getUserById" resultMap="userResultMap" parameterType="java.lang.Integer">
		select u.id id,
		u.userName userName,u.note note,r.id role_id, r.roleName roleName
		from t_user u left join t_role r on u.roleId=r.id
		where u.id=#{id}
	</select>
</mapper>

接着在PostMan上测试一下，断点查看返回数据格式如下:




这样我们就通过association实现了关联属性的查询，使用association定义关联的单个对象的封装结果。
至此，我们关于MyBatis使用association解决一对一关联查询介绍完毕。
```

**方式二**

```html
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">



<mapper namespace="com.hoomsun.wealth.crm.dao.UserMapper">



	<resultMap id="userResultMap" type="com.hoomsun.wealth.crm.model.User">



		<id column="id" jdbcType="INTEGER" property="id" />



		<result column="userName" jdbcType="VARCHAR" property="userName" />



		<result column="note" jdbcType="DECIMAL" property="note" />	



		<result property="role.id" column="role_id" />



		<result property="role.roleName" column="roleName" />			



	</resultMap>



 



	<select id="getUserById" resultMap="userResultMap" parameterType="java.lang.Integer">



		select u.id id,



		u.userName userName,u.note note,r.id role_id, r.roleName roleName



		from t_user u left join t_role r on u.roleId=r.id



		where u.id=#{id}



	</select>



</mapper>
```



接着在PostMan上测试一下，断点查看返回数据格式如下:

![img](https://img-blog.csdn.net/20180703173919319?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b2Jpbmh1aQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/2018070317402568?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b2Jpbmh1aQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
这样我们就通过association实现了关联属性的查询，使用association定义关联的单个对象的封装结果。
至此，我们关于MyBatis使用association解决一对一关联查询介绍完毕。