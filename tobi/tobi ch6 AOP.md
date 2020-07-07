# AOP

# TODO
- 이 문서 전체 다시 검토
	- 요약 부분, 인용 부분 추가하여 다시 작성
- 예제 코드 기반 위주의 문서로 변경
	- 책에서는 동떨어져 있는 before 와 after 를 한군데 모아서 작성

## 403p
- 선언적 트랜잭션 기능, 트랜잭션 경계설정 기능을 AOP를 이용
	
## 406p
- 직접 사용하는 것이 문제가 된다면 간접적으로 사용하면 된다. DI의 기본 아이디어는 실제 사용할 오브젝트의 클래스 정체는 감춘 채 인터페이스를 통해 간접으로 접근하는 것이다. 그 덕분에 구현 클래스는 얼마든지 외부에서 변경할 수 있다.
	
## 409p
- UserServiceTx는 사용자 관리라는 비즈니스 로직을 전혀 갖지 않고 고스란히 다른 UserService 구현 오브젝트에 기능을 위임한다. 이를 위해 UserService 오브젝트를 DI 받을 수 있도록 만든다.
	
## 412p
- 수정한 스프링의 설정파일에는 UserService라는 인터페이스 타입을 가진 두 개의 빈이 존재하기 때문이다. 같은 타입의 빈이 두개라면 @Autowired를 적용한 경우 어떤 빈을 가져올까? @Autowired는 기본적으로 타입을 이용해 빈을 찾지만 만약 타입으로 하나의 빈을 결정할 수 없는 경우에는 필드 이름을 이용해 빈을 찾는다. 따라서 UserSeviceTest에서 다음과 같은 userService 변수를 설정해두면 아이디가 userService인 빈이 주입될 것이다.

```
@Autowired UserService userService;
```
	
## 417p
- 테스트를 의존 대상으로부터 분리해서 고립시키는 방법은 MailSender에 적용해봤던 대로 테스트를 위한 대역을 사용하는 것이다. MailSender에는 이미 DummyMailSender라는 테스트 스텁을 적용했다. 또 테스트 대역이 테스트 검증에도 참여할 수 있도록, 특별히 만든 MockMailSender라는 목 오브젝트도 사용해봤다.

## 418p
- UserServiceImpl의 upgradeLevels() 메소드는 리턴 값이 없는 void형이다. 따라서 메소드를 실행하고 그 결과를 받아서 검증하는 것은 아예 불가능하다. upgradeLevels()는 DAO를 통해 필요한 정보를 가져와, 일정한 작업을 수행하고 그 결과를 다시 DAO를 통해 DB에 반영한다. 따라서 그 코드의 동작이 바르게 됐는지 확인하려면 결과가 남아 있는 DB를 직접 확인할 수 밖에 없다. 따라서 기존 테스트 코드에서는 UserService의 메소드를 실행시킨 후에 UserDao를 이용해 DB에 들어간 결과를 가져와 검증하는 방법을 사용했다.
- 그런데 의존 오브젝트나 외부 서비스에 의존하지 않는 고립된 테스트 방식으로 만든 UserServiceImpl은 아무리 그 기능이 수행돼도 그 결과가 DB 등을 통해서 남지 않으니, 기존의 방법으로는 작업 결과를 검증하기 힘들다. upgradeLevels()처럼 결과가 리턴되지 않는 경우는 더더욱 그렇다.
- 그래서 이럴 땐 테스트 대상인 UserServiceImpl과 그 협력 오브젝트인 UserDao에게 어떤 요청을 했는지를 확인하는 작업이 필요하다. 테스트 중에 DB에 결과가 반영되지는 않았지만, UserDao의 update() 메소드를 호출하는 것을 확인할 수 있다면, 결국 DB에 그 결과가 반영될 것이라고 결론을 내릴 수 있기 때문이다.
	
## 421p
- 그래서 getAll()에 대해서는 스텁으로서, update()에 대해서는 목 오브젝트로서 동작하는 UserDao 타입의 테스트 대역이 필요하다. 이 클래스의 이름을 MockUserDao라고 하자
- UserServiceTest 전용일테니 역시 스태틱 내부 클래스로 만들면 편리하다.

```
static class MockUserDao implements UserDao {
	private List<User> users;
	private List<User> updated = new ArrayList<>();

	private MockUserDao(List<User> users) {
	this.users = users;
}

	public List<User> getUpdated() {
		return this.updated;
	}
	
	// 스텁 기능 제공
	public List<User> getAll() {
		return this.users;
	}
	
	// 목 오브젝트 기능 제공
	public void update(User user) {
		updated.add(user);
	}
```

## 428p

```
// 이렇게 만들어진 목 오브젝트는 아직 아무런 기능이 없다.
UserDao mockuserDao = mock(UserDao.class)

// 여기에 먼저 getAll() 메소드가 불려올 때 사용자 목록을 리턴하도록 스텁 기능을 추가해줘야 한다. 
// 다음의 코드면 충분하다.
when(mockUserDao.getAll()).thenReturn(this.users);

// 다음은 update()호출이 있었는지를 검증하는 부분이다. 
// 테스트를 진행하는 동안 mockUserDao의 update() 메소드가 두 번 호출됐는지 확인 하고 싶다면, 
// 다음과 같이 검증 코드를 넣어주면 된다.
// User타입의 오브젝트를 파라미터로 받으며 update() 메소드가 두 번 호출됐는지 확인하라는 것이다.
verify(mockUserDao, times(2)).update(any(User.class));
```
	
## 432p
- 프록시 패턴, 프록시, 타겟
- 부가기능 코드에서는 핵심기능으로 요청을 위임해주는 과정에서 자신이 가진 부가적인 기능을 적용해 줄수 있다. 비즈니스 로직 코드에 트랜잭션 기능을 부여해주는 것이 바로 그런 대표적인 경우다.
- 이렇게 마치 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것을 대리자, 대리인과 같은 역할을 한다고 해서 프록시라고 부른다. 그리고 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 타겟 또는 실체라고 부른다.
- 프록시의 특징은 타깃과 같은 인터페이스를 구현했다는 것과 프록시가 타깃을 제어할 수 있는 위치에 있다는 것이다.

```
클라이언트 --> 프록시 --> 타겟
```
	
## 433p
- 데코레이터 패턴
- 따라서 데코레이터 패턴에서는 프록시가 꼭 한 개로 제한되지 않는다.
- 프록시가 여러 개인 만큼 순서를 정해서 단계적으로 위임하는 구조를 만들면 된다.
	
## 434p
- 프록시로서 동작하는 각 데코레이터는 위임하는 대상에도 인터페이스로 접근하기 때문에 자신이 최종 타깃으로 위임하는지, 아니면 다음 단계의 데코레이터 프록시로 위임하는지 알지 못한다. 그래서 데코레이터의 다음 위임 대상은 인터페이스로 선언하고 생성자나 수정자 메소드를 통해 우임 대상을 외부에서 런타임 시에 주입받을 수 있도록 만들어야 한다.

```
클라이언트 --> 라인넘버 데코레이터 --> 컬러 데코레이터 --> 소스코드 출력 기능 타깃(핵심기능)
```

```
// InputStream 인터페이스
// 타깃인 FileInputStream
// 버퍼 읽기 기능을 제공하는 BufferedInputStream 테코레이터
InputStream is = new BufferedInputStream(new FileInputStream("a.txt"));
```
	
## 436p
- Collections의 unmodifiableCollection()을 통해 만들어지는 오브젝트가 전형적인 접근권한 제어용 프록시라고 볼 수 있다.
	
## 437p
- 그렇다면 목 오브젝트를 만드는 불편함을 목 프레임워크를 사용해 편리하게 바꿨던 것처럼 프록시도 일일이 모든 인터페이스를 구현해서 클래스를 새로 정의하지 않고도 편리하게 만들어서 사용할 방법은 없을까?
- 물론 있다. 자바에는 java.lang.reflect 패키지 안에 프록시를 손쉽게 만들 수 있도록 지원해주는 클래스들이 있다.
- 프록시는 다음의 두 가지 기능으로 구성된다.
	- 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.
	- 지정된 요청에 대해서는 부가기능을 수행한다.

## 447p
- 다이나믹 프록시

```
// 프록시 생성
Hello proxiedHello = (Hello) Proxy.newProxyInstance(
	getClass.getClassLoader(), 
	new Class[] { Hello.class }, 
	new UppercaseHandler(new HelloTarget())
);
```

- InvocationHandler 방식의 또 한가지 장점은 타깃의 종류에 상관없이도 적용기 가능하다는 점이다.
- 어떤 종류의 인터페이스를 구현한 타깃이든 상관없이 재사용할 수 있고, 메소드의 리턴 타입이 스트링인 경우만 대문자로 결과를 바꿔주도록 UppercaseHandler를 만들 수 있다.

```
public class UppercaseHandler implements InvocationHandler {
	Object target;
	
	// FIXME 책에서 왜 private 지 ?
	private UppercaseHandler(Object target) {
		this.target = target;
	}
	
	public Object invoke(Object proxy, Method method, Object[] args) {
	
		Object ret = method.invoke(target, args);
		if(ret instance of String) {
			return ((String) ret).toUpperCase();
		}
		else {
			return ret;
		}
	}
}
```
	
## 451p
- 스프링의 빈은 기본적으로 클래스 이름과 프로퍼티로 정의된다.
- Class.newInstance() 메소드는 해당 클래스의 파라미터가 없는 생성자를 호출하고, 생성되는 오브젝트를 돌려주는 리플렉션 API다.
- 스프링은 내부적으로 리플렉션 API를 이용해서 빈 정의에 나오는 클래스 이름을 가지고 빈 오브젝트를 생성한다.

```
Date now = (Date) Class.forName("java.util.Date").newInstance();
```

## 452p
- 다이나믹 프록시는 Proxy클래스의 newProxyInstance()라는 스태틱 팩토리 메소드를 통해서만 만들 수 있다.
- 팩토리 빈
- 팩토리 빈이란 스프링을 대신해서 오브젝트의 생성로직을 담당하도록 만들어진 특별한 빈을 말한다.
- 팩토리 빈을 만드는 방법에는 여러 가지가 있는데, 가장 간단한 방법은 스프링의 FactoryBean이라는 인터페이스를 구현하는 것이다.
- FactoryBean 인터페이스를 구현한 클래스를 스프링의 빈으로 등록하면 팩토리 빈으로 동작한다.

```
// 생성자를 제공하지 않는 Message 클래스
// 
public class Message {

	String text;
	
	private Message(String text) {
		this.text = text;
	}
	
	public String getText() {
		return text;
	}
	
	public static Message newMessage(String text) {
		return new Message(text);
	}
}

// Message 의 팩토리 빈 클래스
// 
public class MessageFactoryBean implements FactoryBean<Message> {

	String text;
	
	public void setText(String text) {
		this.text = text;
	}
	
	public Message getObject() throws Exception {
		return Message.newMessage(this.text);
	}
	
	public Class<? extends Message> getObjectType() {
		return Message.class;
	}
	
	public boolean isSingleton() {
		return false;
	}

}
```	
	
## 454p
- 스프링은 FactoryBean 인터페이스를 구현한 클래스가 빈의 클래스로 지정되면, 팩토리 빈 클래스의 오브젝트의 getObject() 메소드를 이용해 오브젝트를 가져오고, 이를 빈 오브젝트로 사용한다.
- 빈의 클래스로 등록된 팩토리 빈은 빈 오브젝트를 생성하는 과정에서만 사용될 뿐이다.

## 454p
- FactoryBeanTest-context.xml 이름으로 저장해두고 테스트를 작성한다.

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration  // 설정 파일 이름을 지정하지 않으면 클래스이름 + "-context.xml 이 디폴트로 사용된다.
public class FactoryBeanTest {
	...
}
```
	
## 464p
- ProxyFactoryBean
- 스프링은 프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈을 제공해준다.

## 467p
- MethodInterceptor 는 Advice 인터페이스를 상속하고 있는 서브인터페이스이기 때문이다.
- MethodInterceptor 처럼 타깃 오브젝트에 적용하는 부가기능을 담은 오브젝트를 스프링에서는 Advice 라고 부른다.

## 470p
- 부가기능을 제공하는 오브젝트를 어드바이스
- 메소드 선정 알고리즘을 담은 오브젝트를 포인트컷
- 어드바이스와 포인트컷은 모두 프록시에 DI로 주입돼서 사용된다.
- 두 가지 모두 여러 프록시에서 공유가 가능하도록 만들어지기 때문에 스프링의 싱글톤 빈으로 등록이 가능하다.
- 포인트컷은 Pointcut 인터페이스를 구현해서 만들면 된다.

## 471p
- 프록시로부터 어드바이스와 포인트컷을 독립시키고 DI를 사용하게 한 것은 전형적인 전략 패턴 구조다.
- 어드바이스와 포인트컷을 묶은 오브젝트를 인터페이스 이름을 따서 어드바이저라고 부른다.

```
어드바이저 = 포인트컷(메소드 선정 알고리즘) + 어드바이스 (부가기능)
```

## 473p

```
// 트랜젝션 어드바이스
//

package springbook.user.service;

public class TransactionAdvice implements MethodInterceptor {

	@Setter
	PlatformTransactionManager transactionManager;
	
	// 타깃을 호출하는 기능을 가진, 콜백 오브젝트를, 프록시로부터 받는다.
	// 덕분에 어드바이스는, 특정 타깃에 의존하지 않고, 재사용 가능하다.
	public Object invoke(MethodInvocation invocation) throws Trhowable {
		
		// 트랜젝션 시작
		TranscationStatus status = transactionManager.getTranscation(
				new DefaultTransactionDefinition());
		
		try {
		
			Object ret = invocation.proceed();  // 콜백으로 타깃의 메소드 실행
			transactionManager.commit(status);  // 커밋
			return ret;  // 콜백 리턴
			
		} catch (RuntimeException e) {
			transactionManager.rollback(status);  // 롤백
			throw e;
		}
	}
}

// xml 스프링 빈 설정
//

// 어드바이스
<bean id="transactionAdvice" class="springbook.user.service.TransactionAdvice">
	<property name="transactionManager" ref="transactionManager" />
</bean>

// 포인트컷
<bean id="transcationPointcut" class="springbook.user.service.TranscationPointcut">
	<property name="mappedName" value="upgrade*" />
</bean>

// 어드바이저
<bean id="transcationAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
	<property name="advice" ref="transactionAdvice" />
	<property name="pointcut" ref="transcationPointcut" />
</bean>

// 프록시 팩토리 빈 = userService
<bean id="userService" class="org.springframework.aop.framework.ProxyFactoryBean">
	<property name="target" ref="userServiceImpl" />
	<property name="interceptorNames" >
		<list>
			<value>transcationAdvisor</value>
		</list>
	</property>
</bean>

// 테스트 코드
//

@Test
@DirtiesContext // 컨텍스트 설정을 변경하기 때문에 여전히 필요하다
public void upgradeAllOrNothing() {
	
	TestUserService testUserService = new TestUserService(users.get(3).getId());
	testUserService.setUserDao(userDao);
	testUserService.setMailSender(mailSender);
	
	ProxyFactoryBean txProxyFactoryBean = 
		context.getBean("&userService", ProxyFactoryBean.class);  // &userService 이므로 FactoryBean 자체를 가져옴
	txProxyFactoryBean.setTarget(testUserService);  // 컨텍스트 설정 변경
	UserService txUserService = (UserService) txProxyFactoryBean.getObject();
	
	...
}
```

## 479p
- 빈후처리기
- 스프링은 컨테이너로서 제공하는 기능 중에서 변ㄷ하지 않는 핵심적인 부분외에는 대부분 확장할 수 있도록 확장 포인트를 제공해준다.
	- BeanPostProcessor 인터페이스를 구현해서 만드는 빈 후처리기다.
	- 빈 오브젝트로 만들어지고 난 후에, 빈 오브젝트를 다시 가공할 수 있게 해준다.
	- 빈 후처리기의 구조와 자세한 사용법은 2부에서 다시 다룰 것이다．
	- DefaultAdvisorAutoProxyCreator

## 481p

```
public interface Pointcut {
	ClassFilter getClassFilter();  // 프록시를 적용할 클래스인지 확인
	MethodMatcher getMethodMatcher();  // 어드바이스를 적용할 메소드인지 확인
}
```

## 482p

```
@Test
public void classNamePointcutAdvisor() {

	// 포인트컷 준비
	// NameMatchMethodPointcut 은 ClassFilter 에 기능이 없으므로, 오버라이딩으로 기능 추가
	NameMatchMethodPointcut classMethodPointcut = new NameMatchMethodPointcut() {
		public ClassFilter getClassFilter() {
			return new ClassFilter() {
				public boolean matches(Class<?> clazz) {
					return clazz.getSimpleName().startsWith("HelloT");
				}
			}
		}
	};
	classMethodPointcut.setMappedName("sayH*");
	
	// 테스트, 적용 클래스가 맞다
	checkAdviced(new HelloTarget(), classMethodPointcut, true);
	
	// 메소드 이너 클래스, 적용 클래스가 아니다
	// 단순히 클래스 이름을 변경하기 위해서 상속을 사용
	class HelloWord extends HelloTarget {};
	checkAdviced(new HelloWord(), classMethodPointcut, false);
	
	// 테스트, 적용 클래스가 맞다
	class HelloToby extends HelloTarget {};
	checkAdviced(new HelloToby(), classMethodPointcut, true);
}

private void checkAdviced(Object target, Pointcut pointcut, boolean adviced) {

	ProxyFactoryBean pfBean = new ProxyFactoryBean();
	pfBean.setTarget(target);
	pfBean.addAdvisor(new DefaultAdvisorAutoProxyCreator(pointcut, new UppercaseAdvice()));
	Hello proxiedHello = (Hello) pfBean.getObject();
	
	// 메소드 선정 방식을 통해 어드바이스 적용
	if(adviced) {
		assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));  // 어드바이스 적용
		assertThat(proxiedHello.sayHi"Toby"), is("HI TOBY"));  // 어드바이스 적용
		assertThat(proxiedHello.sayThankYou("Toby"), is("Thank You Toby"));  // 안됨
	}
	else {  // 아예 어드바이스 적용 대상에서 제외
		assertThat(proxiedHello.sayHello("Toby"), is("Hello Toby"));
		assertThat(proxiedHello.sayHi"Toby"), is("Hi Toby"));
		assertThat(proxiedHello.sayThankYou("Toby"), is("Thank You Toby"));
	}

}
```

## 484p

```
// 빈 id 가 없음
// 다른 빈에서 참조되거나 코드에서 빈 이름으로 조회될 필요가 없는 빈이라면
// 아이디를 등록하지 않아도 된다
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" />
```

## 485p
```
// 클래스 필터가 포함된 포인트컷
//
package springbook.service;

public class NameMatchClassMethodPointcut extends NameMatchMethodPointcut {

	public void setMappedClassName(String mappedClassName) {
		this.setClassFilter(new SimpleClassFilter(mappedClassName));
	}
	
	static class SimpleClassFilter implements ClassFilter {
	
		String mappedName;
		
		private SimpleClassFilter(String mappedName) {
			this.mappedName = mappedName;
		}
		
		public boolean matchs(Class<?> clazz) {
			// PatternMatchUtils: 스프링 유틸
			return PatternMatchUtils.simpleMatch(mappedName, clazz.getSimpleName());
		}
	}
}

// 변경전, 포인트컷
// 
<bean id="transcationPointcut" class="springbook.user.service.TranscationPointcut">
	<property name="mappedName" value="upgrade*" />
</bean>

// 변경후, 포인트컷
<bean id="transcationPointcut" class="springbook.service.NameMatchClassMethodPointcut">
	<property name="mappedClassName" value="*ServiceImpl" />
	<property name="mappedName" value="upgrade*" />
</bean>
```

## 487p
```
// 수정한 테스트용 UserService 구현 클래스
// 트랜젝션 실패가 목적
//
static class TestUserServiceImpl extends UserServieImpl {
	private String id = "madnite1";
	
	protected void upgradeLevel(User user) {
		if(user.getId().equals(this.id) {
			throw new TestUserServiceException();
		}
		super.upgradeLevel(user);
	}
}

// 테스트용 UserService 등록
// parent: 프로퍼티 정의를 포함해서 userService 빈의 설정을 상속받는다
// testUserService 는 userService 빈의 설정을 상속받고, 클래스만 변경된다.
// 따라서 userDao 나 mailSender 프로퍼티를 지정해줄 필요가 없다.
//
<bean id="testUserService"
	class="springbook.user.service.UserServiceTest$TestUserServiceImpl"
	parent="userService" />
```

## 490p
```
// DefaultAdvisorAutoProxyCreator에 의해 userService 빈이 프록시로 바꿔치기 됐다면
// getBean("userService")로 가져온 오브젝트는 TestUserService 타입이 안디라 JDK의 Proxy 타입일 것이다.
// 모든 JDK 다이내믹 프록시 방식으로 만들어지는 프록시는 Proxy 클래스의 서버클래스이기 때문이다
//
@Test
public void advisorAutoProxyCreator() {
	assertThat(testUserService, is(java.lang.reflet.Proxy.class));
}
```

## 491p - 495p
- 포인트컷 표현식
- 포인트컷 표현식을 지원하는 포인트컷을 적용하려면 AspectJExpressionPointcut 클래스를 사용하면 된다.

```
// 포인트컷 테스트용 클래스들
// 

public interface TargetInterface {
	public void hell() ;
	public void hello(String a) ;
	public int minus(int a, int b) throws RuntimeException ;
	public void int plus(int a, intb) ;
}

public class Target implements TargetInterface {
	public void hell() {}
	public void hello(String a) {}
	public int minus(int a, int b) throws RuntimeException { return 0; }
	public void int plus(int a, intb) { return 0; }
	public void method() {}  // new
}

public class Bean {
	public void method() throws RuntimeException { }
}

// 메소드 시그니처를 이용한 포인트컷 표현식 테스트
//
@Test
public void methodSignaturePointcut throws SecurityException, NoSuchMethodException {

	// Target 클래스, minus() 메소드 시그니처
	AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
	pointcut.setExpression("execution("
		+ "public int "
		+ "springbook.spring.pointcut.Target."
		+ "minus(int, int) "
		+ "throws java.lang.RuntimeException"
		+ ")"
		);
		
	// Target.minus()
	// 클래스 필터와 메소드 매처를 가져와 각각 비교한다
	assertThat(
				pointcut.getClassFilter()
					.matches(Target.class)
				&&
				pointcut.getMethodMatcher()
					.matches(Target.class.getMethod("minus", int.class, int.class), null)
			, is(true)
			);
	
	// Target.plus()
	// 메소드 매처에서 실패
	assertThat(
		pointcut.getClassFilter().matches(Target.class)
		&&
		pointcut.getMethodMatcher().matches(
			Target.class.getMethod(
				"plus", int.class, int.class)
			, null)
		), is(false)
		);
	
	// Bean.method()
	// 클래스 필터에서 실패
	assertThat(
		pointcut.getClassFilter().matches(Bean.class)
		&&
		pointcut.getMethodMatcher().matches(
			Target.class.getMethod("method"), null)
		, is(false)
		);
}
```

## 500p
- AspectJ 포인트컷 표현식은 메소드를 선정하는 데 편리하게 쓸 수 있는 강력한 표현식 언어다.
- ``` bean(*Service) ``` : 아이디가 Service로 끝나는 모든 빈
- ``` @annotation(org.springframework.transcation.annotation.Transcational) ``` : @Transcational 적용된 모든 메소드를 선정하는 포인트컷

## 501p
```
// 변경전
//
<bean id="transcationPointcut" class="springbook.service.NameMatchClassMethodPointcut">
	<property name="mappedClassName" value="*ServiceImpl" />
	<property name="mappedName" value="upgrade*" />
</bean>

// 변경후
//
<bean id="transcationPointcut"
	class="org.springframework.aop.aspectj.AspectJExpressionPointcut">
	<property name="expression"
		value="execution(* *..*ServiceImpl.upgrade*(..))" 
		/>
</bean>
```

## 503p
- UserSerivce에 트랜잭션을 적용해온 과정 정리
	- 트랜잭션 서비스 추상화
	- 프록시, 데코레이터 패턴
	- 다이나믹 프록시, 프록시 팩토리 빈
	- 자동 프록시 생성 방법, 포인트컷
		- 스프링 컨테이너의 빈 생성 후처리 기법
			- DefaultAdvisorAutoProxyCreator

## 506p
- aspect 애스팩트
	- 그 자체로 어플리케이션의 핵심기능을 담지는 않지만
	- 핵심기능에 부가되어 의미를 갖는 특별한 모듈
	- 애스펙트 = 어드바이스(부가기능 정의) + 포인트컷(적용 조건)
		- 어드바이저는 아주 단순한 형태의 애스팩트
- AOP: Aspect Oriented Programming
	- 어플리케이션이 핵심적인 기능에서
	- 부가적인 기능을 분리해서
	- 애스팩트라는 독특한 모듈로 만들어서
	- 설계하고 개발하는 방법론
	- 어플리케이션을 다양한 관점에서 바라보며 개발할 수 있게 도와준다
		- 어플리케이션을 사용자 관리라는 핵심 로직이 아니라
		- 대신 트랜잭션 경계설정이라는 관점에서 바라보고
			- 그 부분에 집중해서 설계하고 개발할 수 있게 된다
		- 트랜잭션 기술의 적용에만 주목하고 싶다면
			- TransactionAdvice 에만 집중하면 된다































