---
layout: default
title: Template Method Pattern
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/template-method-pattern/src/main/java/com/example/templatemethodpattern/application/TemplateService.java

타입에 따라 로직을 다르게 수행해야할 때 사용할 수 있는 디자인 패턴입니다.
Case별로 Enum을 지정하여 타입별로 서비스레이어를 다양하게 호출하여 사용합니다.

```java
@Service
@RequiredArgsConstructor
public class TemplateService {

    private final List<TemplateAbstract> caseServices;
    private final EnumMap<CaseType, TemplateAbstract> caseMap = new EnumMap<>(CaseType.class);

    @PostConstruct
    public void init() {
        for (TemplateAbstract template : caseServices) {
            caseMap.put(template.getCase(), template);
        }
    }

    public Object step1(CaseType caseType) {

        TemplateAbstract caseService = caseMap.get(caseType);

        return caseService.step1();
    }

    public Object step2(CaseType caseType) {

        TemplateAbstract caseService = caseMap.get(caseType);

        return caseService.step2();
    }

    public Object step3(CaseType caseType) {

        TemplateAbstract caseService = caseMap.get(caseType);

        return caseService.step3();
    }
```

```java
public abstract class TemplateAbstract {

    abstract CaseType getCase();
    abstract Object step1();
    abstract Object step2();
    abstract Object step3();

}
```

```java
@Service
public class Case1Service extends TemplateAbstract {

    @Override
    CaseType getCase() {
        return CaseType.CASE_1;
    }

    @Override
    Object step1() {
        return "case 1 - step 1";
    }

    @Override
    Object step2() {
        return "case 1 - step 2";
    }

    @Override
    Object step3() {
        return "case 1 - step 3";
    }
}
```

```java
@Service
public class Case2Service extends TemplateAbstract {

    @Override
    CaseType getCase() {
        return CaseType.CASE_2;
    }

    @Override
    Object step1() {
        return "case 2 - step 1";
    }

    @Override
    Object step2() {
        return "case 2 - step 2";
    }

    @Override
    Object step3() {
        return "case 2 - step 3";
    }
}
```
