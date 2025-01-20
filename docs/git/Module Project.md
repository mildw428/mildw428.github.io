---
layout: default
title: Module Project
parent: git
---

```groovy
//setting.gradle
includeFlat('core')

//root프로젝트 내부에 모듈이 있다면 include('core')

//build.gradle 
dependencies {
	implementation project(':core')
}
```

`core`는 모듈 프로젝트의 이름이다.
