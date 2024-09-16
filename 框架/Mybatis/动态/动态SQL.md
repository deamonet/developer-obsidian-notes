# [Mybatis 动态insert动态插入的坑](https://www.cnblogs.com/h-c-g/p/10097810.html)

在写insert子句的时候，由于不知道需要插入多少字段，mybatis通过prefix,suffix,suffixOverrides很好的解决了该问题，实现了动态insert语句。

用这种动态插入时<if test=""></test>这里test的字段一定不要写错，本来直接写字段名就可以了，写错为#字段名 就不起作用了

<insert id="insertSelective" parameterType="com.bootdo.system.domain.LogDO">  
insert into sys_log  
<trim prefix="(" suffix=")" suffixOverrides=",">  
<if test="id != null">   id,  </if>  
<if test="userId != null">  user_id,  </if>  
<if test="username != null">  username,  </if>  
<if test="operation != null">  operation,  </if>  
<if test="time != null">  time,  </if>  
<if test="method != null">  method,  
</if>  
<if test="params != null">  
params,  
</if>  
<if test="ip != null">  
ip,  
</if>  
<if test="gmtCreate != null">  
gmt_create,  
</if>  
</trim>  
<trim prefix="values (" suffix=")" suffixOverrides=",">  
<if test="id != null">  
#{id,jdbcType=BIGINT},  
</if>  
<if test="userId != null">  
#{userId,jdbcType=BIGINT},  
</if>  
<if test="username != null">  
#{username,jdbcType=VARCHAR},  
</if>  
<if test="operation != null">  
#{operation,jdbcType=VARCHAR},  
</if>  
<if test="time != null">  
#{time,jdbcType=INTEGER},  
</if>  
<if test="method != null">  
#{method,jdbcType=VARCHAR},  
</if>  
<if test="params != null">  
#{params,jdbcType=VARCHAR},  
</if>  
<if test="ip != null">  
#{ip,jdbcType=VARCHAR},  
</if>  
<if test="gmtCreate != null">  
#{gmtCreate,jdbcType=TIMESTAMP},  
</if>  
</trim>  
</insert>

顺便记一下批量插入

Mapper接口中提供  
public void batchSave(List<Emp> empList);

Mapper.xml提供

<insert id="batchSave">  
into t_emp(emp_name,emp_email,dept_id) VALUES  
<foreach collection="list" item="emp" separator=",">  
(#{emp.empName}, #{emp.empEmail}, #{emp.deptId})  
</foreach>  
</insert>