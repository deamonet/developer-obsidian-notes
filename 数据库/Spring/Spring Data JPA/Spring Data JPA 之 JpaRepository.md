[西瓜游侠](https://blog.csdn.net/hbtj_1216 "西瓜游侠")![](media/newCurrentTime2.png)于 2018-04-01 23:54:20 发布![](media/articleReadEyes2.png)51083 ![](media/tobarCollect2.png) 收藏 150

分类专栏： [微服务合集](https://blog.csdn.net/hbtj_1216/category_7400816.html) [数据库](https://blog.csdn.net/hbtj_1216/category_7095033.html) 文章标签： [Spring Data JPA](https://so.csdn.net/so/search/s.do?q=Spring+Data+JPA&t=all&o=vip&s=&l=&f=&viparticle=) [JpaRepository](https://so.csdn.net/so/search/s.do?q=JpaRepository&t=all&o=vip&s=&l=&f=&viparticle=) [SpringBoot](https://so.csdn.net/so/search/s.do?q=SpringBoot&t=all&o=vip&s=&l=&f=&viparticle=) [hibernate](https://so.csdn.net/so/search/s.do?q=hibernate&t=all&o=vip&s=&l=&f=&viparticle=)

版权

[![](media/20201014180756757.png)微服务合集同时被 2 个专栏收录![](media/newArrowDown1White.png)](https://blog.csdn.net/hbtj_1216/category_7400816.html "微服务合集")

43 篇文章5 订阅

订阅专栏

[![](media/20201014180756916.png)数据库](https://blog.csdn.net/hbtj_1216/category_7095033.html "数据库")

20 篇文章1 订阅

订阅专栏

> `SpringBoot`版本：`2.3.2.RELEASE`  
> `SpringBoot Data JPA`版本：`2.3.2.RELEASE`

`JpaRepository`是`SpringBoot Data JPA`提供的非常强大的基础接口。

## 1 [JpaRepository](https://so.csdn.net/so/search?q=JpaRepository&spm=1001.2101.3001.7020)

### 1.1 JpaRepository接口定义

`JpaRepository`接口的官方定义如下：

```java
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T>
```

可以看出`JpaRepository`继承了接口`PagingAndSortingRepository`和`QueryByExampleExecutor`。而`PagingAndSortingRepository`又继承`CrudRepository`。  
因此，`JpaRepository`接口同时拥有了基本`CRUD`功能以及分页功能。

当我们需要定义自己的`Repository`接口的时候，我们可以直接继承`JpaRepository`，从而获得`SpringBoot Data JPA`为我们内置的多种基本数据操作方法，例如：

```java
public interface UserRepository extends JpaRepository<User, Integer> {

}
```

### 1.2 内置方法

#### 1.2.1 CrudRepository<T, ID>提供的方法

```java
    /**
	 * 保存一个实体。
	 */
	<S extends T> S save(S entity);

	/**
	 * 保存提供的所有实体。
	 */
	<S extends T> Iterable<S> saveAll(Iterable<S> entities);

	/**
	 * 根据id查询对应的实体。
	 */
	Optional<T> findById(ID id);

	/**
	 * 根据id查询对应的实体是否存在。
	 */
	boolean existsById(ID id);

	/**
	 * 查询所有的实体。
	 */
	Iterable<T> findAll();

	/**
	 * 根据给定的id集合查询所有对应的实体，返回实体集合。
	 */
	Iterable<T> findAllById(Iterable<ID> ids);

	/**
	 * 统计现存实体的个数。
	 */
	long count();

	/**
	 * 根据id删除对应的实体。
	 */
	void deleteById(ID id);

	/**
	 * 删除给定的实体。
	 */
	void delete(T entity);

	/**
	 * 删除给定的实体集合。
	 */
	void deleteAll(Iterable<? extends T> entities);

	/**
	 * 删除所有的实体。
	 */
	void deleteAll();
```

#### 1.2.2 [PagingAndSortingRepository](https://so.csdn.net/so/search?q=PagingAndSortingRepository&spm=1001.2101.3001.7020)<T, ID>提供的方法

```java
    /**
	 * 返回所有的实体，根据Sort参数提供的规则排序。
	 */
	Iterable<T> findAll(Sort sort);

	/**
	 * 返回一页实体，根据Pageable参数提供的规则进行过滤。
	 */
	Page<T> findAll(Pageable pageable);
```

#### 1.2.3 JpaRepository<T, ID>提供的方法

```java
    /**
	 * 将所有未决的更改刷新到数据库。
	 */
	void flush();

	/**
	 * 保存一个实体并立即将更改刷新到数据库。
	 */
	<S extends T> S saveAndFlush(S entity);

	/**
	 * 在一个批次中删除给定的实体集合，这意味着将产生一条单独的Query。
	 */
	void deleteInBatch(Iterable<T> entities);

	/**
	 * 在一个批次中删除所有的实体。
	 */
	void deleteAllInBatch();

	/**
	 * 根据给定的id标识符，返回对应实体的引用。
	 */
	T getOne(ID id);
```

`JpaRepository<T, ID>`还继承了一个`QueryByExampleExecutor<T>`，提供按“实例”查询的功能。

## 2 方法测试

下面对以上提供的所有内置方法进行测试，给出各方法的用法。

首先定义实体类Customer：

```java
package com.tao.springboot.hibernate.entity;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
@Table(name = "tb_customer")
@Data
@NoArgsConstructor
@RequiredArgsConstructor
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(nullable = false)
    private Long id;

    @Column(nullable = false)
    private String name;
    
    @Column(nullable = false)
    private Integer age;
}
```

然后定义接口`CustomerRepository`：

```java
package com.tao.springboot.hibernate.repository;

import com.tao.springboot.hibernate.entity.Customer;
import org.springframework.data.jpa.repository.JpaRepository;

public interface CustomerRepository extends JpaRepository<Customer, Long> {
    
}
```

接下来对各个方法进行测试~

### 2.1 save

```java
    /**
	 * 保存一个实体。
	 */
	<S extends T> S save(S entity);
```

测试代码：

```java
    @GetMapping("/customer/save")
    public Customer crudRepository_save() {

        // 保存一个用户michael
        Customer michael = new Customer("Michael", 26);
        Customer res = customerRepository.save(michael);
        return res;
    }
```

测试结果：  
![这里写图片描述](media/这里写图片描述.jpg)  
![这里写图片描述](media/这里写图片描述.jpg)

### 2.2 saveAll

```java
    /**
	 * 保存提供的所有实体。
	 */
	<S extends T> Iterable<S> saveAll(Iterable<S> entities);
```

测试代码：

```java
    @GetMapping("/customer/saveAll")
    public List<Customer> crudRepository_saveAll() {

        // 保存指定集合的实体
        List<Customer> customerList = new ArrayList<>();
        customerList.add(new Customer("Tom", 21));
        customerList.add(new Customer("Jack", 21));
        List<Customer> res = customerRepository.saveAll(customerList);
        return res;
    }
```

测试结果：  
![这里写图片描述](media/这里写图片描述-1.jpg)  
![这里写图片描述](media/这里写图片描述.jpg)

### 2.3 findById

```java
    /**
	 * 根据id查询对应的实体。
	 */
	Optional<T> findById(ID id);
```

测试代码：

```java
    @GetMapping("/customer/findById")
    public Customer crudRepository_findById() {

        // 根据id查询对应实体
        Optional<Customer> customer = customerRepository.findById(1L);
        if(customer.isPresent()) {
            return customer.get();
        }
        return null;
    }
```

测试结果：  
![这里写图片描述](media/这里写图片描述-2.jpg)

### 2.4 existsById

```java
    /**
	 * 根据id查询对应的实体是否存在。
	 */
	boolean existsById(ID id);
```

测试代码：

```java
    @GetMapping("/customer/existsById")
    public boolean crudRepository_existsById() {

        // 根据id查询对应的实体是否存在
        return customerRepository.existsById(1L);
    }
```

测试结果：  
![这里写图片描述](media/这里写图片描述-1.jpg)

### 2.5 findAll

```java
    /**
	 * 查询所有的实体。
	 */
	Iterable<T> findAll();
```

测试代码：

```java
    @GetMapping("/customer/findAll")
    public List<Customer> crudRepository_findAll() {

        // 查询所有的实体
        List<Customer> customerList = customerRepository.findAll();
        return customerList;
    }
```

测试结果：  
![这里写图片描述](media/这里写图片描述-2.jpg)

### 2.6 findAllById

```java
    /**
	 * 根据给定的id集合查询所有对应的实体，返回实体集合。
	 */
	Iterable<T> findAllById(Iterable<ID> ids);
```

测试代码：

```java
    @GetMapping("/customer/findAllById")
    public List<Customer> crudRepository_findAllById() {

        // 根据给定的id集合查询所有对应的实体，返回实体集合
        List<Long> ids = new ArrayList<>();
        ids.add(2L);
        ids.add(1L);
        List<Customer> customerList = customerRepository.findAllById(ids);
        return customerList;
    }
```

测试结果：  
![这里写图片描述](media/这里写图片描述-2.jpg)

### 2.7 count

```java
    /**
	 * 统计现存实体的个数。
	 */
	long count();
```

测试代码：

```java
    @GetMapping("/customer/count")
    public Long crudRepository_count() {

        // 统计现存实体的个数
        return customerRepository.count();
    }
```

测试结果：  
![这里写图片描述](media/这里写图片描述.jpg)

### 2.8 deleteById

```java
    /**
	 * 根据id删除对应的实体。
	 */
	void deleteById(ID id);
```

测试代码：

```java
    @GetMapping("/customer/deleteById")
    public void crudRepository_deleteById() {

        // 根据id删除对应的实体
         customerRepository.deleteById(1L);
    }
```

测试结果：  
删除前~~  
![这里写图片描述](media/这里写图片描述-1.jpg)  
删除后~~  
![这里写图片描述](media/这里写图片描述.jpg)  
![这里写图片描述](media/这里写图片描述-2.jpg)

### 2.9 delete(T entity)

```java
	/**
	 * 删除给定的实体。
	 */
	void delete(T entity);
```

测试代码：

```java
    @GetMapping("/customer/delete")
    public void crudRepository_delete() {

        // 删除给定的实体
        Customer customer = new Customer(2L, "Tom", 21);
        customerRepository.delete(customer);
    }
```

测试结果：  
删除前~~  
![这里写图片描述](media/这里写图片描述-2.jpg)  
删除后~~  
![这里写图片描述](media/这里写图片描述-2.jpg)  
![这里写图片描述](media/这里写图片描述-2.jpg)

### 2.10 deleteAll(Iterable<? extends T> entities)

```java
	/**
	 * 删除给定的实体集合。
	 */
	void deleteAll(Iterable<? extends T> entities);
```

测试代码：

```java
    @GetMapping("/customer/deleteAll(entities)")
    public void crudRepository_deleteAll_entities() {

        // 删除给定的实体集合
        Customer tom = new Customer(2L,"Tom", 21);
        Customer jack = new Customer(3L,"Jack", 21);
        List<Customer> entities = new ArrayList<>();
        entities.add(tom);
        entities.add(jack);
        customerRepository.deleteAll(entities);
    }
```

测试结果：  
删除前~~  
![这里写图片描述](media/这里写图片描述-2.jpg)  
删除后~~  
![这里写图片描述](media/这里写图片描述-2.jpg)  
![这里写图片描述](media/这里写图片描述-2.jpg)

### 2.11 deleteAll

```java
	/**
	 * 删除所有的实体。
	 */
	void deleteAll();
```

测试代码：

```java
    @GetMapping("/customer/deleteAll")
    public void crudRepository_deleteAll() {

        // 删除所有的实体
        customerRepository.deleteAll();
    }
```

测试结果：  
删除前~~  
![这里写图片描述](media/这里写图片描述-2.jpg)  
删除后~~  
![这里写图片描述](media/这里写图片描述-2.jpg)  
![这里写图片描述](media/这里写图片描述-2.jpg)

### 2.12 findAll(Sort sort)

```java
    /**
	 * 返回所有的实体，根据Sort参数提供的规则排序。
	 */
	Iterable<T> findAll(Sort sort);
```

测试代码：

```java
    @GetMapping("/customer/findAll(sort)")
    public List<Customer> pagingAndSortingRepository_findAll_sort() {

        // 返回所有的实体，根据Sort参数提供的规则排序
        // 按age值降序排序
        Sort sort = new Sort(Sort.Direction.DESC, "age");
        List<Customer> res = customerRepository.findAll(sort);
        return res;
    }
```

测试结果：  
![这里写图片描述](media/这里写图片描述-1.jpg)  
格式化之后发现，确实是按照age的值降序输出的！！！

### 2.13 findAll(Pageable pageable)

```java
    /**
	 * 返回一页实体，根据Pageable参数提供的规则进行过滤。
	 */
	Page<T> findAll(Pageable pageable);
```

测试代码：

```java
    @GetMapping("/customer/findAll(pageable)")
    public void pagingAndSortingRepository_findAll_pageable() {

        // 分页查询
        // PageRequest.of 的第一个参数表示第几页（注意：第一页从序号0开始），第二个参数表示每页的大小
        Pageable pageable = PageRequest.of(1, 5); //查第二页
        Page<Customer> page = customerRepository.findAll(pageable);

        System.out.println("查询总页数:" + page.getTotalPages());
        System.out.println("查询总记录数:" + page.getTotalElements());
        System.out.println("查询当前第几页:" + (page.getNumber() + 1));
        System.out.println("查询当前页面的集合:" + page.getContent());
        System.out.println("查询当前页面的记录数:" + page.getNumberOfElements());
    }
```

测试结果：  
![这里写图片描述](media/这里写图片描述-1.jpg)