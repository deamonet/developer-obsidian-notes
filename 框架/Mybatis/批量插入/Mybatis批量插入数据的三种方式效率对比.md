

![头像](media/头像.png)

**WillLiaowh**

6518



](https://segmentfault.com/u/willliaowh)

[

发布于

2019-12-15

](https://segmentfault.com/a/1190000021290975/revision)

数据库为MySql。

1.for循环insert

        long start = System.currentTimeMillis();
        for(int i = 0 ;i < 100000; i++) {
            User user = new User();
            user.setId("id" + i);
            user.setName("name" + i);
            user.setPassword("password" + i);
            userMapper.insert(user);
        }
        long end = System.currentTimeMillis();
        System.out.println("---------------" + (start - end) + "---------------");

    <insert id="insert">
      INSERT INTO t_user (id, name, password)
          VALUES(#{id}, #{name}, #{password})
    </insert>

时间为380826ms

2.Mybatis batch模式

    SqlSession sqlSession = sqlSessionTemplate.getSqlSessionFactory().openSession(ExecutorType.BATCH, false);//跟上述sql区别
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

        long start = System.currentTimeMillis();
        for (int i = 0; i < 100000; i++) {
            User user = new User();
            user.setId("id" + i);
            user.setName("name" + i);
            user.setPassword("password" + i);
            userMapper.insert(user);
        }
        sqlSession.commit();
        long end = System.currentTimeMillis();
        System.out.println("---------------" + (start - end) + "---------------");

    <insert id="insert">
      INSERT INTO t_user (id, name, password)
          VALUES(#{id}, #{name}, #{password})
    </insert>

时间为203660ms

3.批量foreach插入

      long start = System.currentTimeMillis();
        List<User> userList = new ArrayList<>();
        for (int i = 0; i < 100000; i++) {
            User user = new User();
            user.setId("id" + i);
            user.setName("name" + i);
            user.setPassword("password" + i);
            userMapper.insert(user);
        }
        userMapper.insertBatch(userList);
        long end = System.currentTimeMillis();
        System.out.println("---------------" + (start - end) + "---------------");

<insert id="insertBatch">
        INSERT INTO t_user
        (id, name, password)
        VALUES
        <foreach collection ="userList" item="user" separator =",">
            (#{id}, #{name}, #{password})
        </foreach >
    </insert>

时间为8706ms

结论:foreach批量插入 > mybatis batch模式插入 > for循环insert


https://segmentfault.com/a/1190000021290975