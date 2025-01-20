---
layout: default
title: Entity Listener
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/entity-listener/src/test/java/com/example/entitylistener/MemberTest.java

Entity Listener를 사용하면 CRUD 이전과 이후에 메서드를 자동으로 실행할 수 있다.

```
------------------------- insert 이전
Hibernate: 
    insert 
    into
        `
        member` (
            `create_at`, `update_at`, `name`
        ) 
    values
        (?, ?, ?)
------------------------- insert 이후
Member(id=8, name=John)
Hibernate: 
    select
        member0_.`id` as id1_0_0_,
        member0_.`create_at` as create_a2_0_0_,
        member0_.`update_at` as update_a3_0_0_,
        member0_.`name` as name4_0_0_ 
    from
        `member` member0_ 
    where
        member0_.`id`=?
------------------------- Select 호출 이후
------------------------- Update 이전
Hibernate: 
    update
        `member` 
    set
        `update_at`=?,
        `name`=? 
    where
        `id`=?
------------------------- Update 이후
------------------------- Delete 이전
Hibernate: 
    delete 
    from
        `member` 
    where
        `id`=?
------------------------- Delete 이후
```


```java
	@PrePersist
    public void prePersist() {
        System.out.println("insert 이전");
    }

    @PostPersist
    public void postPersist() {
	    System.out.println("insert 이후");
    }

    @PreUpdate
    public void preUpdate() {
        System.out.println("Update 이전");
    }

    @PostUpdate
    public void postUpdate() {
        System.out.println("Update 이후");
    }

    @PreRemove
    public void preRemove() {
        System.out.println("Delete 이전");
    }

    @PostRemove
    public void postRemove() {
        System.out.println("Delete 이후");
    }

    @PostLoad
    public void postLoad() {
        System.out.println("Select 호출 이후");
    }
```

```java
@Getter  
@MappedSuperclass  
@EntityListeners(AuditingEntityListener.class)  
public abstract class BaseEntity {  
  
    @CreatedDate  
    @Column(updatable = false)  
    private LocalDateTime createDateTime;  
  
    @LastModifiedDate  
    private LocalDateTime updateDateTime;  
  
}
```

```java
@EnableJpaAuditing
@SpringBootApplication  
public class Application extends SpringBootServletInitializer {  
  
   @PostConstruct
   public void init() throws IOException {
	   // 초기화 메서드 수행
   }  
  
   public static void main(String[] args) {  
      SpringApplication.run(CmsApplication.class, args);  
   }  
  
}
```

