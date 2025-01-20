---
layout: default
title: Utils - Page Rq Rs
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/tree/main/core/src/main/java/com/example/core/paging

Pageable 그대로의 객체는 가지고있는 필드가 많아 필요한 부분만 사용하도록 객체를 구성했습니다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class PageRq {

    private Integer page;

    private Integer size;

    private Sort.Direction direction;

    private String property;

    public Pageable toPageable() {
        if (direction != null && property != null) {
            Sort sort = Sort.by(new Sort.Order(this.direction, this.property));
            return PageRequest.of(this.page, this.size, sort);
        }

        return PageRequest.of(this.page, this.size);
    }

    public PageRq(Integer page, Integer size, Sort.Direction direction, String property) {
        this.page = page;
        this.size = size;
        this.direction = direction;
        this.property = property;
    }

}
```

```java
@Getter
@NoArgsConstructor
public class PageRs {
    private Integer page;
    private Integer size;
    private Integer totalPages;
    private long totalElements;

    public PageRs(Pageable pageable, int totalPages, long totalElements) {
        this.page = pageable.getPageNumber();
        this.size = pageable.getPageSize();
        this.totalPages = totalPages;
        this.totalElements = totalElements;
    }

    public PageRs(int page, int size, int totalPages, int totalElements) {
        this.page = page;
        this.size = size;
        this.totalPages = totalPages;
        this.totalElements = totalElements;
    }

    public PageRs(Pageable pageable, long totalElements) {
        this.page = pageable.getPageNumber();
        this.size = pageable.getPageSize();
        this.totalPages = (int) Math.ceil((double) totalElements / pageable.getPageSize());
        this.totalElements = totalElements;
    }

    public PageRs(Page<?> page) {
        this.page = page.getPageable().getPageNumber();
        this.size = page.getPageable().getPageSize();
        this.totalPages = page.getTotalPages();
        this.totalElements = page.getTotalElements();
    }
}
```
