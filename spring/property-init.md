# 配置文件加载

* https://docs.spring.io/spring-boot/docs/1.2.0.M1/reference/html/boot-features-external-config.html

* 如果同时存在properties和yml文件，将合并配置文件，如果有相同的配置，按properties为准（即使properties文件在jar包中）
* 若工程存在application.yml ，引入的包也有application.yml ，将忽略引入包的application.yml 文件


* 无法使用公共包配置文件作为默认配置文件
  * 不能完全确保日常配置和公共包配置不重复
  * 工程是用properties文件，jar里面使用yml文件 -- 不现实，yml文件更友好
  * SpringApplication.setDefaultProperties 这个貌似无法再jar中使用
