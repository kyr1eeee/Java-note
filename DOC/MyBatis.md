### MyBatis

#{} 和${}的区别

前者会在sql预编译时将参数设置进去，相当于prepareStatement，能够预防sql注入；后者则直接将参数拼接sql语句，不能预防sql注入;前者还支持定制参数的一些规则，比如真的参数为null时，可以指定jdbcType=NULL(默认的是OTHER,而orcle识别不了这个)。不过对于Mysql来说，这两种JdbcType都能识别。

resultType和resultMap

resultType根据指定的类，安装java bean属性名和数据库记录字段名对应映射（数据库的_命名方式，对应bean的驼峰命名方式）。

resultMap则可以自定义java bean 的属性与数据库记录字段的映射关系

```xml
<!--自定义某个javaBean的封装规则
	type：自定义规则的Java类型
	id:唯一id方便引用
	  -->
	<resultMap type="com.atguigu.mybatis.bean.Employee" id="MySimpleEmp">
		<!--指定主键列的封装规则
		id定义主键会底层有优化；
		column：指定哪一列
		property：指定对应的javaBean属性
		  -->
		<id column="id" property="id"/>
		<!-- 定义普通列封装规则 -->
		<result column="last_name" property="lastName"/>
		<!-- 其他不指定的列会自动封装：我们只要写resultMap就把全部的映射规则都写上。 -->
		<result column="email" property="email"/>
		<result column="gender" property="gender"/>
	</resultMap>
<!-- resultMap:自定义结果集映射规则；  -->
	<!-- public Employee getEmpById(Integer id); -->
	<select id="getEmpById"  resultMap="MySimpleEmp">
		select * from tbl_employee where id=#{id}
	</select>
<!-- 
	场景一：
		查询Employee的同时查询员工对应的部门
		Employee===Department
		一个员工有与之对应的部门信息；
		id  last_name  gender    d_id     did  dept_name (private Department dept;)
	 -->
	 
	 
	<!--
		联合查询：级联属性封装结果集
	  -->
	<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyDifEmp">
		<id column="id" property="id"/>
		<result column="last_name" property="lastName"/>
		<result column="gender" property="gender"/>
		<result column="did" property="dept.id"/>
		<result column="dept_name" property="dept.departmentName"/>
	</resultMap>


	<!-- 
		使用association定义关联的单个对象的封装规则；
	 -->
	<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyDifEmp2">
		<id column="id" property="id"/>
		<result column="last_name" property="lastName"/>
		<result column="gender" property="gender"/>
		
		<!--  association可以指定联合的javaBean对象
		property="dept"：指定哪个属性是联合的对象
		javaType:指定这个属性对象的类型[不能省略]
		-->
		<association property="dept" javaType="com.atguigu.mybatis.bean.Department">
			<id column="did" property="id"/>
			<result column="dept_name" property="departmentName"/>
		</association>
	</resultMap>
	<!--  public Employee getEmpAndDept(Integer id);-->
	<select id="getEmpAndDept" resultMap="MyDifEmp">
		SELECT e.id id,e.last_name last_name,e.gender gender,e.d_id d_id,
		d.id did,d.dept_name dept_name FROM tbl_employee e,tbl_dept d
		WHERE e.d_id=d.id AND e.id=#{id}
	</select>
```

