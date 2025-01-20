---
layout: default
title: Generics
parent: git
---

```java
@Entity  
@Getter  
@Inheritance(strategy = InheritanceType.JOINED)  
@DiscriminatorColumn(name = "product_type")  
@NoArgsConstructor(access = AccessLevel.PROTECTED)  
public abstract class Product {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    @Column(name = "product_type", insertable = false, updatable = false)  
    @Enumerated(EnumType.STRING)  
    protected ProductType productType;  
  
    @OneToOne(fetch = FetchType.LAZY)  
    @JoinColumn(name = "solver_content_version_id")  
    protected SolverContentVersion solverContentVersion;  
  
}

  
@Getter  
@Entity  
@Table(name = "product_leadership")  
@DiscriminatorValue("LEADERSHIP")  
@PrimaryKeyJoinColumn(name = "product_id")  
@NoArgsConstructor  
public class LeadershipProduct extends Product {  
  
    @OneToOne(fetch = FetchType.LAZY)  
    @JoinColumn(name = "solver_grade_range_setting_id")  
    private SolverGradeRangeSetting gradeRangeSetting;  
  
    public List<SolverGradeRange> getDefaultGradeRanges() {  
        return this.gradeRangeSetting.getSolverGradeRanges();  
    }  
  
    @Builder(builderMethodName = "leadershipTestBuilder")  
    public LeadershipProduct(Long id, ProductType productType, DefaultAnalysisSettingInfo defaultAnalysisSettingInfo, SolverGradeRangeSetting gradeRangeSetting) {  
        super(id, productType, defaultAnalysisSettingInfo);  
        this.gradeRangeSetting = gradeRangeSetting;  
    }  
}

```


```java
  
@Getter  
@MappedSuperclass  
@NoArgsConstructor(access = lombok.AccessLevel.PROTECTED)  
@AllArgsConstructor(access = lombok.AccessLevel.PROTECTED)  
public abstract class BaseManagement<  
        Evaluator extends BaseEvaluator,  
        Evaluation extends BaseEvaluation,  
        Target extends BaseTarget,  
        AnalysisStatus extends BaseAnalysisStatus,  
        Notification extends BaseNotification  
        > implements Management {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    protected Long id;  
  
    protected String title;  
  
    protected Long productId;  
  
    protected Integer evaluatorLimit;  
  
    @Embedded  
    protected Period period;  
  
    @Enumerated(EnumType.STRING)  
    protected BasicResultMethodType methodType;  
  
    protected Boolean isOpen = false;  
  
    protected Boolean isDeleted = false;  
  
    @Embedded  
    protected Description description = new Description();  
  
    protected LocalDateTime postDateTime;  
  
    @Embedded  
    protected Attention attention = new Attention("진단 전 체크리스트", List.of(  
            "본 진단이 중요한 진단임을 알고 있다.",  
            "누구보다 객관적으로 진단에 임할 것이다.",  
            "다른 일을 미루고 진단에 집중할 것이다."  
    ));  
  
    @Getter(AccessLevel.PRIVATE)  
    @OneToMany(mappedBy = "management", cascade = CascadeType.ALL, orphanRemoval = true)  
    protected List<Evaluator> evaluators = new ArrayList<>();  
  
    @Getter(AccessLevel.PRIVATE)  
    @OneToMany(mappedBy = "management", cascade = CascadeType.ALL, orphanRemoval = true)  
    protected List<Target> targets = new ArrayList<>();  
  
    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL, orphanRemoval = true)  
    @JoinColumn(name = "analysis_status_id")  
    protected AnalysisStatus analysisStatus;  
  
    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL, orphanRemoval = true)  
    @JoinColumn(name = "result_setting_id")  
    protected ResultSetting resultSetting;  
  
    @OneToMany(mappedBy = "management", cascade = CascadeType.ALL, orphanRemoval = true)  
    protected List<Notification> notifications = new ArrayList<>();  
  
    @Enumerated(EnumType.STRING)  
    protected ResultReadScopeLeader leaderReadScope = ResultReadScopeLeader.NOT_ALLOWED;  
  
    @Enumerated(EnumType.STRING)  
    protected ResultReadScopeMember memberReadScope = ResultReadScopeMember.NOT_ALLOWED;
}

  
@Entity  
@Getter  
@AssociationOverrides({  
        @AssociationOverride(  
                name = "analysisStatus",  
                joinColumns = @JoinColumn(name = "leadership_analysis_status_id")  
        )  
})  
@NoArgsConstructor(access = AccessLevel.PROTECTED)  
@AllArgsConstructor(access = AccessLevel.PRIVATE)  
public class LeadershipManagement extends BaseManagement<  
        LeadershipEvaluator,  
        LeadershipEvaluation,  
        LeadershipTarget,  
        LeadershipAnalysisStatus,  
        LeadershipNotification  
        > {  
  
    @Enumerated(EnumType.STRING)  
    private LeadershipManagementStep stepInfo = LeadershipManagementStep.TWO;
}
```


```java
private OrderSpecifier[] getOrderSpecifiers(Pageable pageable) {  
    List<OrderSpecifier> orders = new ArrayList<>();  
  
    if (!pageable.getSort().isEmpty()) {  
        for (Sort.Order order : pageable.getSort()) {  
            Order direction = order.getDirection().isAscending() ? Order.ASC : Order.DESC;  
            switch (order.getProperty()) {  
                case ProbationResultTargetDto.Fields.id:  
                    orders.add(new OrderSpecifier(direction, probationTarget.id));  
                    return orders.stream().toArray(OrderSpecifier[]::new);  
                case ProbationResultTargetDto.Fields.name:  
                    orders.add(new OrderSpecifier(direction, member.name));  
                    break;                case ProbationResultTargetDto.Fields.organization:  
                    orders.add(new OrderSpecifier(direction, memberRole.organization.name));  
                    break;                case ProbationResultTargetDto.Fields.employeeId:  
                    orders.add(new OrderSpecifier(direction, member.employeeId));  
                    break;                case ProbationResultTargetDto.Fields.responsibility:  
                    orders.add(new OrderSpecifier(direction, memberRole.responsibility.name));  
                    break;                case ProbationResultTargetDto.Fields.isAnalyzed:  
                    orders.add(new OrderSpecifier(direction, solverProbationFactScore.isNull()));  
                    break;                case ProbationResultTargetDto.Fields.totalScore:  
                    orders.add(new OrderSpecifier(direction, solverProbationFactScore.score));  
                    break;                case ProbationResultTargetDto.Fields.totalResult:  
                    orders.add(new OrderSpecifier(direction, solverProbationContent.totalResultType));  
                    break;                default:  
                    orders.add(new OrderSpecifier(Order.DESC, probationTarget.id));  
                    break;            }  
        }  
    }  
    orders.add(new OrderSpecifier(Order.DESC, member.id));  
  
    return orders.stream().toArray(OrderSpecifier[]::new);  
}

@Getter  
@Setter  
@NoArgsConstructor  
@AllArgsConstructor  
@FieldNameConstants  <-
public class ProbationResultTargetDto {  
    private Long id;  
   //...
}
```

