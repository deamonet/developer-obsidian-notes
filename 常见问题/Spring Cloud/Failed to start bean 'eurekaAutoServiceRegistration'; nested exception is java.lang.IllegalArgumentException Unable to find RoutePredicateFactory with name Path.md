还以为是一个依赖冲突的问题，搞了半天原来是gateway配置文件中断言的部分，等号前后不能加空格。。。。。
spring:  
  main:  
    web-application-type: reactive  
  application:  
    name: gateway  
  cloud:  
    gateway:  
      routes:  
        - id: borrow-service  
          uri: lb://borrow-service  
          predicates:  
            - Path=/borrow/**
如果写成 Path = /borrow/** 就会报错。