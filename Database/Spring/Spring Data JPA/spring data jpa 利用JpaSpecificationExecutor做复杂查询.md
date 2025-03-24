[node2017](https://blog.csdn.net/yingxiake "node2017")![](media/newCurrentTime2.png)于 2016-03-30 16:01:59 发布![](media/articleReadEyes2.png)51667 ![](media/tobarCollect2.png) 收藏 7

分类专栏： [spring data jpa](https://blog.csdn.net/yingxiake/category_6177741.html) 文章标签： [JpaSpecifi](https://so.csdn.net/so/search/s.do?q=JpaSpecifi&t=all&o=vip&s=&l=&f=&viparticle=)

版权

[![](media/20201014180756928.png)spring data jpa专栏收录该内容](https://blog.csdn.net/yingxiake/category_6177741.html "spring data jpa")

7 篇文章5 订阅

订阅专栏

spring data jpa 通过创建方法名来做查询，只能做简单的查询，那如果我们要做复杂一些的查询呢，多条件分页怎么办，这里，spring data jpa为我们提供了JpaSpecificationExecutor接口，只要简单实现toPredicate方法就可以实现复杂的查询

1.首先让我们的接口继承于JpaSpecificationExecutor

```
public interface TaskDao extends JpaSpecificationExecutor<Task>{

}
```

2.JpaSpecificationExecutor提供了以下接口

```
public interface JpaSpecificationExecutor<T> {

    T findOne(Specification<T> spec);

    List<T> findAll(Specification<T> spec);

    Page<T> findAll(Specification<T> spec, Pageable pageable);

    List<T> findAll(Specification<T> spec, Sort sort);

    long count(Specification<T> spec);
}
```

其中Specification就是需要我们传进去的参数，它是一个接口

```

public interface Specification<T> {

    Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);
}
```

提供唯一的一个方法toPredicate，我们只要按照JPA 2.0 criteria api写好查询条件就可以了，关于JPA 2.0 criteria api的介绍和使用，欢迎参考  
[http://blog.csdn.net/dracotianlong/article/details/28445725](http://blog.csdn.net/dracotianlong/article/details/28445725)  
[http://developer.51cto.com/art/200911/162722.htm](http://developer.51cto.com/art/200911/162722.htm)

2.接下来我们在service bean

```
@Service
public class TaskService {

    @Autowired TaskDao taskDao ;


    /**
     * 复杂查询测试
     * @param page
     * @param size
     * @return
     */
    public Page<Task> findBySepc(int page, int size){

        PageRequest pageReq = this.buildPageRequest(page, size);
        Page<Task> tasks = this.taskDao.findAll(new MySpec(), pageReq);

        return tasks;

    }

     /**
      * 建立分页排序请求 
      * @param page
      * @param size
      * @return
      */
     private PageRequest buildPageRequest(int page, int size) {
           Sort sort = new Sort(Direction.DESC,"createTime");
           return new PageRequest(page,size, sort);
     }

    /**
     * 建立查询条件
     * @author liuxg
     * @date 2016年3月30日 下午2:04:39
     */
    private class MySpec implements Specification<Task>{

        @Override
        public Predicate toPredicate(Root<Task> root, CriteriaQuery<?> query, CriteriaBuilder cb) {

     //1.混合条件查询
          /*Path<String> exp1 = root.get("taskName");
            Path<Date>  exp2 = root.get("createTime");
            Path<String> exp3 = root.get("taskDetail");
            Predicate predicate = cb.and(cb.like(exp1, "%taskName%"),cb.lessThan(exp2, new Date()));
            return cb.or(predicate,cb.equal(exp3, "kkk"));

            类似的sql语句为:
            Hibernate: 
                select
                    count(task0_.id) as col_0_0_ 
                from
                    tb_task task0_ 
                where
                    (
                        task0_.task_name like ?
                    ) 
                    and task0_.create_time<? 
                    or task0_.task_detail=?
            */

    //2.多表查询
        /*Join<Task,Project> join = root.join("project", JoinType.INNER);
            Path<String> exp4 = join.get("projectName");
            return cb.like(exp4, "%projectName%");

            Hibernate: 
            select
                count(task0_.id) as col_0_0_ 
            from
                tb_task task0_ 
            inner join
                tb_project project1_ 
                    on task0_.project_id=project1_.id 
            where
                project1_.project_name like ?*/ 
           return null ;  
        }
    }
}
```

3.实体类task代码如下

```
@Entity
@Table(name = "tb_task")
public class Task {

    private Long id ;
    private String taskName ;
    private Date createTime ;
    private Project project;
    private String taskDetail ;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    public Long getId() {
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }

    @Column(name = "task_name")
    public String getTaskName() {
        return taskName;
    }
    public void setTaskName(String taskName) {
        this.taskName = taskName;
    }

    @Column(name = "create_time")
    @DateTimeFormat(pattern = "yyyy-MM-dd hh:mm:ss")
    public Date getCreateTime() {
        return createTime;
    }
    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }


    @Column(name = "task_detail")
    public String getTaskDetail() {
        return taskDetail;
    }
    public void setTaskDetail(String taskDetail) {
        this.taskDetail = taskDetail;
    }

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "project_id")
    public Project getProject() {
        return project;
    }
    public void setProject(Project project) {
        this.project = project;
    }

}
```

通过重写toPredicate方法，返回一个查询 [Predicate](https://so.csdn.net/so/search?q=Predicate&spm=1001.2101.3001.7020)，spring data jpa会帮我们进行查询。

也许你觉得，每次都要写一个类来实现Specification很麻烦，那或许你可以这么写

```
public class TaskSpec {

    public static Specification<Task> method1(){

        return new Specification<Task>(){
            @Override
            public Predicate toPredicate(Root<Task> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                return null;
            }

        };
    }

    public static Specification<Task> method2(){

        return new Specification<Task>(){
            @Override
            public Predicate toPredicate(Root<Task> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                return null;
            }

        };
    }

}
```

那么用的时候，我们就这么用

```
Page<Task> tasks = this.taskDao.findAll(TaskSpec.method1(), pageReq);
```

JpaSpecificationExecutor的介绍就到这里，下次再看看怎么通过写hql或者sql语句来进行查询