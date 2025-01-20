---
layout: default
title: Mock
parent: memo
---
# Mock
---

## mock이란?
 mock은 실제 객체를 만들어 사용하기에 시간, 비용이 높거나 서로간의 의존성이 강해 구현하기 힘들 경우 가짜 객체를 만들어서 사용하는 방법이다.


>Test Double은 xUnit Test Patterns의 저자인 제라드 메스자로스가 만든 용어로 테스트를 진행하기 어려운 경우 이를 대신해 테스트를 진행할 수 있도록 만들어주는 객체를 말합니다.  
>
>단어 자체의 뜻은 '대역'에 가깝고 스턴트맨과 유사한 의미를 가집니다.



mock을 사용하는 이유 - https://cobbybb.tistory.com/16

> mock 을 안쓴다면?
> 

## mock의 장점

```java
public interface UserService {
    User getUserById(int id);
    void saveUser(User user);
}

public class UserServiceImpl implements UserService {
    private UserDao userDao;
    
    public UserServiceImpl(UserDao userDao) {
        this.userDao = userDao;
    }
    
    public User getUserById(int id) {
        return userDao.getUserById(id);
    }
    
    public void saveUser(User user) {
        userDao.saveUser(user);
    }
}
```

UserService 인터페이스를 구현한 UserServiceImpl 클래스를 테스트하는 경우, UserDao 의존성 객체를 생성해야 합니다. 하지만 실제 데이터베이스를 사용하면 테스트의 복잡성이 증가하며, 예상치 못한 결과가 발생할 수 있습니다.

이를 해결하기 위해, UserDao 인터페이스의 구현체 대신 Mockito를 사용하여 UserDao 의존성을 가짜(mock) 객체로 대체할 수 있습니다. 이를 위해서는 UserDao 인터페이스의 mock 객체를 만들고, UserServiceImpl 생성자에서 mock 객체를 전달하면 됩니다.

```java
public class UserServiceImplTest {
    @Mock
    private UserDao userDao;
    
    @InjectMocks
    private UserServiceImpl userServiceImpl;
    
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
    }
    
    @Test
    public void testGetUserById() {
        // mock UserDao 객체에 getUserById 메서드 호출 시, User 객체 반환하도록 지정
        when(userDao.getUserById(1)).thenReturn(new User(1, "Alice"));
        
        // UserServiceImpl의 getUserById 메서드 호출 시, mock UserDao 객체의 getUserById 메서드가 실행됨
        User user = userServiceImpl.getUserById(1);
        
        // mock UserDao 객체의 getUserById 메서드가 호출되었는지 검증
        verify(userDao).getUserById(1);
        
        // 결과 검증
        assertEquals(user.getId(), 1);
        assertEquals(user.getName(), "Alice");
    }
}

```

## mock의 단점

모의(Mock)는 실제 객체를 대체하는 가짜 객체로, 테스트를 위해 사용됩니다. 하지만 모의(Mock)는 몇 가지 단점이 있습니다.

1.  코드와 모의(Mock)의 동기화: 코드에서 변경이 생기면 모의(Mock)도 함께 변경해주어야 합니다. 그렇지 않으면 모의(Mock)는 예상한 대로 작동하지 않을 수 있습니다.
2.  모의(Mock)의 제한: 모의(Mock)는 실제 객체의 일부 동작을 모방하므로, 실제 객체의 모든 기능을 대체하지는 못합니다. 때로는 모의(Mock)로 대체할 수 없는 복잡한 객체들도 있습니다.
3.  테스트 오용: 모의(Mock)를 사용하면 간단하고 빠른 테스트를 작성할 수 있습니다. 그러나 이로 인해 프로그래머가 코드에 대해 더 이상 충분한 검증을 수행하지 않는 경우가 종종 있습니다.
4.  의존성 문제: 모의(Mock)는 실제 객체에 대한 의존성을 줄이지만, 모의(Mock) 자체에 대한 의존성을 도입할 수도 있습니다. 모의(Mock)를 작성하려면 몇 가지 라이브러리를 사용해야 할 수도 있습니다.
5.  시간의 흐름에 따른 문제: 모의(Mock)는 상태를 저장하지 않습니다. 이것은 가끔 문제가 될 수 있습니다. 예를 들어, 일부 시스템은 시간의 경과에 따라 상태를 변경합니다. 이러한 상황에서는 모의(Mock) 대신 다른 방법을 사용해야 합니다.


## 사용법

| Mock 종류 | 의존성 주입 Target |
|-----------|------------------|
| @Mock | @InjectMocks |
| @MockBean | @SpringBootTest |

Mock - 어노테이션 단어 그대로 Mock 객체, 즉 가짜 객체로 쓰겠다는 뜻이다.
MockBean - 어노테이션 단어 뜻 그대로 Mock Bean, 즉 가짜 Bean을 만들겠다는 뜻이다.

- @Mock은 @InjectMocks에 대해서만 해당 클래스안에서 정의된 객체를 찾아서 의존성을 해결합니다.

- @MockBean은 mock 객체를 스프링 컨텍스트에 등록하는 것이기 때문에 @SpringBootTest를 통해서 Autowired에 의존성이 주입되게 됩니다.

#### Spring Context란?
- Bean의 확장 버전으로 Spring이 Bean을 다루기 좀 더 쉽도록 기능들이 추가된 공간이다.
bean은 모두 context안에서 이루어진다.

```java
@WebMvcTest(controllers = Controller.class)  
@AutoConfigureMockMvc  
class ContollerTest {  
  
    private MockMvc mockMvc;  
  
    @MockBean  
    private Service service;  
  
    @BeforeEach  
    void setUp(WebApplicationContext webApplicationContext) {  
        this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)  
                .addFilter(new CharacterEncodingFilter("UTF-8", true))  
                .build();  
    }  
  
    @Test  
    void findAll() {  
        given(service.findAll()).willReturn(...);  
        mockMvc.perform(get("/")  
                        .accept(MediaType.APPLICATION_JSON))  
                .andExpect(status().isOk()))  
                ...  
  
    }  
}
```

@WebMvcTest 이기 때문에 Controller까지는 로드 된다. 하지만 Controller의 협력 객체인 Service는 로드되지 않는다.

그래서 contoller에 실제로 요청을 보낼 때 controller의 협력 객체인 service가 Bean Container에 생성되어 있지 않다면 NPE가 터진다.

이 처럼 @MockBean은 Bean Container에 생성돼야만 하는 가짜 객체일 때 사용하면 된다.


#### Mock Stub 차이

- `Stub`은 테스트 중에 만들어진 호출에 미리 준비된 답변을 제공하고 일반적으로 테스트를 위해 프로그래밍된 것 외에는 전혀 응답하지 않습니다.
- `Mock`은 호출될 것으로 예상되는 사양을 형성하는 기대값으로 미리 프로그래밍 된 객체입니다.

`Mock`만 행동 검증을 수행합니다. 다른 것들은 상태 검증에 사용될 수 있습니다.


---

## TestContainers

TestContainers는 Docker 컨테이너를 사용하여 테스트 환경을 제공합니다. 이를 통해 테스트 중에 일시적인 인스턴스의 데이터베이스, 웹 브라우저 또는 다른 서비스를 시작하고 중지할 수 있습니다. 이러한 컨테이너는 테스트가 시작될 때 생성되고 테스트가 완료되면 삭제됩니다.

예를 들어, MySQL 데이터베이스의 컨테이너화된 인스턴스를 사용하여 데이터 액세스 계층 코드의 완전한 호환성을 테스트할 수 있습니다. 이 경우 개발자의 컴퓨터에 복잡한 설정이 필요하지 않으며 테스트가 항상 알려진 DB 상태로 시작됩니다.

TestContainers는 JUnit과 함께 사용하기 위해 설계되었습니다. 따라서 JUnit의 @Rule 어노테이션을 사용하여 컨테이너를 설정하고 관리할 수 있습니다. 예를 들어, 다음과 같이 MySQL 데이터베이스의 컨테이너화된 인스턴스를 만들 수 있습니다:

```java
@Rule
public MySQLContainer mysql = new MySQLContainer();
```

위의 코드는 테스트가 시작될 때 MySQL 컨테이너를 시작하고 테스트가 완료되면 컨테이너를 중지합니다.
