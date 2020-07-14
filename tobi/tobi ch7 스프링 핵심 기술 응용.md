# 7장 스프링 핵심 기술의 응용

## 565 페이지
- SQL 서비스 인터페이스
```
// 주어진 키를 가지고 SQL string 을 가져오기만 하면 된다
public interface SqlService {
	String getSql(String key);
}
```

- UserDaoJdbc
```
// UserDaoJdbc 는 SqlService 인터페이스를 통해 필요한 SQl 을 가져와 사용할 수 있도록 구성
public class UserDaoJdbc implements UserDao {

	private SqlService sqlService;
	
	public void setSqlService(SqlService sqlService) {
		this.sqlService = sqlService;
	}
	
	// SqlService 는 모든 DAO에서 서비스 빈을 사용하게 만들것이다
	// 따라서 SQL 키 이름이 DAO 별로 중복되지 않게 해야 한다
	// 따라서 add 대신 userAdd 와 같은 식으로 도메인 이름을 함계 사용한다
	public void add(User user) {
		this.jdbcTemplate.update(
			this.sqlService.getSql("userAdd"),
			user.getId(), 
			user.getName(), user.getPassword(), 
			user.getEmail(), user.getLevel().intValue(), user.getLogin()
			);
	}
	
	public User get(String id) {
		return this.jdbcTemplate.queryForObject(
			this.sqlService.getSql("userGet"), 
			new Object[] { id }, 
			this.userMapper
			);
	}
	
	public List<User> getAll() {
		return this.jdbcTemplate.query(
			this.sqlService.getSql("userGetAll"), 
			this.userMapper
			);
	}
	
	public void deleteAll() {
		this.jdbcTemplate.update(
			this.sqlService.getSql("userDeleteAll")
			);
	}
	
	public int getCount() {
		return this.jdbcTemplate.queryForInt(
			this.sqlService.getSql("userGetCount")
			);
	}
	
	public void update(User user) {
		this.jdbcTemplate.update(
			this.sqlService.getSql("userUpdate"), 
			user.getName(), user.getPassword(), 
			user.getEmail(), user.getLevel().intValue(), user.getLogin(), 
			user.getId()
			);
	}
}
```

- SimpleSqlService
```
public class SimpleSqlService implements SqlService {

	private Map<String, String> sqlMap;
	
	// 스프링 빈 xml 설정파일에서 <map> 으로 정의된 정보를 가져오도록
	// 프로퍼티로 등록해준다
	public void setSqlMap(Map<String, String> sqlMap) {
		this.sqlMap = sqlMap;
	}
	
	public String getSql(String key) {
		String sql = sqlMap.get(key);
		if(sql == null) {
			throw new RuntimException(key + " 에 대한 sql 을 찾을 수 없음");
		}
		return sql;
	}
}
```

- sqlMap 프로퍼티에 대한 xml 설정
```
<bean id="userDao" class="springbook.dao.UserDaoJdbc">
	<property name="dataSource" ref="dataSource" />
	<property name="sqlService" ref="sqlService" />
</bean>

<bean id="sqlService" class="springbook.user.sqlservice.SimpleSqlService">
	<property name="sqlMap">
		<map>
			<entry key="userAdd" 				value="insert into ..." />
			<entry key="userGet" 				value="select * from users where id = ?" />
			<entry key="userGetAll" 		value="select * from users order by id" />
			<entry key="userDeleteAll" 	value="delete from users" />
			<entry key="userGetCount" 	value="select count(1) from users" />
			<entry key="userUpdate" 		value="update users set name = ?, ..." />
		</map>
	</property>
</bean>
```

## 570 페이지
- JAXB: Java Architecture for XML Binding
	- java 6부터
	- java.xml.bind 패키지

```
<sqlmap>
	<sql key="userAdd">insert into users(...) ... </sql>
	<sql key="userGet">select * from users ... </sql>
</sqlmap>

// 위 sqlmap 문서의 구조를 정의하는 스키마
// sqlmap.xsd 이름으로 프로젝트 루트에 저장
// JAXB 컴파일러로 컴파일한다
<?xml version="1.0" encoding="UTF-8""?>
<schema xmlns="http://www.w3.org/2001/XMLSchema"
	targetNamespace="http://www.eprill.com/sqlmap"
	xmlns:tns="http://www.eprill.com/sqlmap"
	elementFormDefault="qualified" >
	
	// <sqlmap> 에 대한 정의
	<element name="sqlmap">
		<complexType>
			<sequence>
				<element name="sql" maxOccurs="unbounded" type="tns:sqlType" />
			</sequence>
		</complexType>
	</element>
	
	// <sql> 에 대한 정의
	<complexType name="sqlType">
		<simpleContent>
			<extension base="string">
				<attribute name="key" use="required" type="string" />
			</extension>
		</simpleContent>
	</complexType>

</schema>
```

```
// sqlmap.xsd 파일이 있는 루트 폴더에서 실행함
// 실행하면 자바 클래스와 팩토리 클래스가 만들어진다
xjc -p springbook.user.sqlservice.jaxb sqlmap.xsd -d src

parsing a schema..
compiling a schema ..
springbook\user\sqlservice\jaxb\ObjectFactory.java
springbook\user\sqlservice\jaxb\SqlType.java  <-- <sqlmap> 바인딩 클래스
springbook\user\sqlservice\jaxb\sqlmap.java   <-- <sql> 바인딩 클래스
springbook\user\sqlservice\jaxb\package-info.java
```

- 컴파일러가 만들어준 XML 문서 바인딩용 클래스
- 어노테이션에 정보를 포함한다
```
// <sqlmap> 이 바인딩될 클래스
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name="sqlmapType", propOrder= { "sql" })
@XmlRootElement(name = "sqlmap")
public class Sqlmap {

	@XmlElement(required = true)
	protected List<SqlType> sql;  // <sql> 태그 정보를 담은 SqlType 객체 리스트
	
	public List<SqlType> getSql() {
	
		if(sql == null) {
			sql = new ArrayList<SqlType>();
		}
		return this.sql;
	}

}

// <sql> 태그 정보를 담을 클래스
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "sqlType" propOrder = { "value" })
public class SqlType {

	@XmlValue
	protected String value;
	
	@XmlAttribute(required = true)
	protected String key;
	
	// getter, setter
}
```

- 언마샬링 학습 테스트 코드
```
<?xml version="1.0" encoding="UTF-8"?>
<sqlmap xmlns="http://www.eprill.com/sqlmap"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.eprill.com/sqlmap ../../../../../sqlmap.xsd" >

	<sql key="add">insert</sql>
	<sql key="get">select</sql>
	<sql key="delete">delete</sql>

</sqlmap>

public class JaxbTest {

	@Test
	public void readSqlmap() throws JAXBException, IOException {
	
		// 바인딩용 클래스들의 위치를 가지고 JAXB 컨텍스트를 생성
		String contextPath = Sqlmap.class.getPackage().getName();
		JAXBContext context = JAXBContext.newInstance(contextPath);
		
		// 언마샬러 생성
		Unmarshaller unmarshaller = context.createUnmarshaller();
		
		// 언마살링을 하면 매핑된 오브젝트 트리의 루트인 Sqlmap 을 리턴함
		// 테스트 클래스와 같은 폴더에 있는 sqlmap.xml 파일을 사용함
		Sqlmap sqlmap = (Sqlmap) unmarshaller.unmarshal(
			getClass().getResourceAsStream("sqlmap.xml"));
		
		List<SqlType> sqlList = sqlmap.getSql();
		
		// 테스트 검증
		assertThat(sqlList.size(), is(3));
		assertThat(sqlList.get(0).getKey(), is("add"));
		assertThat(sqlList.get(0).getValue(), is("insert"));
		... 생략 ...
	}
}
```

## 577 페이지
- 생성자 초기화 방법을 사용하는 XmlSqlService
```
// 스프링 빈 등록
<bean id="sqlService" class="springbook.user.sqlservice.XmlSqlService" />

// 빈 정의 클래스
public class XmlSqlService implements SqlService {

	private Map<String, String> sqlMap = new HashMap<>();
	
	// 외부에서 sqlmapFile 경로를 string 으로 주입 받는다
	private String sqlmapFile;
	public void setSqlmapFile(String sqlmapFile) {
		this.sqlmapFile = sqlmapFile;
	}
	
	// 생성자에서 sqlMap 맵 객체 초기화
	// 생성자에서 예외가 발생할 수도 있는 복잡한 초기화 작업	
	/*
	public XmlSqlService() {
	
		String contextPath = Sqlmap.class.getPackage.getName();
		try {
		
			JAXBContext context = JAXBContext.newInstance(contextPath);
			Unmarshaller unmarshaller = context.createUnmarshaller();
			
			// 읽어들일 파일의 위치와 이름이 코드에 고정되어 있다
			InputStream is = UserDao.class.getResourceAsStream("sqlmap.xml");

			Sqlmap sqlmap = (Sqlmap) unmarshaller.unmarshal(is);
			
		} catch (JaXBException e) {
		
			throw new RuntimException(e);
		}
	}
	*/
	
	// load 로직을 생성자에서 메소드로 옮긴다
	public void loadSql() {
	
		String contextPath = Sqlmap.class.getPackage.getName();
		try {
		
			JAXBContext context = JAXBContext.newInstance(contextPath);
			Unmarshaller unmarshaller = context.createUnmarshaller();
			
			// 읽어들일 파일의 위치와 이름이 코드에 고정되어 있다
//		InputStream is = UserDao.class.getResourceAsStream("sqlmap.xml");

			// DI 로 파일 위치를 주입 받는다
			InputStream is = UserDao.class.getResourceAsStream(this.sqlmapFile);
			
			Sqlmap sqlmap = (Sqlmap) unmarshaller.unmarshal(is);
			
		} catch (JaXBException e) {
		
			throw new RuntimException(e);
		}
	}
	
	public String getSql(String key) {
	
		String sql = sqlMap.get(key);
		if(sql == null) {
			throw new RuntimException(key + " sql not found");
		}
		return sql;
	}
}

// XmlSqlService 객체 초기화 방법
XmlSqlService sqlProvider = new XmlSqlService();
sqlProvider.setSqlmapFile("sqlmap.xml");
sqlProvider.loadSql();
```

## 580 페이지
- XmlSqlService 객체는 빈이므로 생성은 물론 초기화도 제어권이 스프링에 있다
- 빈 후처기리를 사용해서
	- 빈 객체를 생성하고, DI 작업을 수행해서, 프로퍼티를 모두 주입 한 후에
	- 미리 지정한 초기화 메소드를 호출 하도록 한다
```
// context xml 파일에 추가
<context:annotation-config />

// JDK6 포함, java.lang.annotation 패키지
@PostConstruct
public void loadSql() {
	...
}
```

## 582 페이지
- XmlSqlService 고찰
	- 책임에 따른 인터페이스 정의
	- 분리 가능한 관심사를 구분해 본다
		- 첫번째, SQL 정보를 외부 리소스로부터 읽어오는 것
		- 두번째, 읽오온 SQL 정보를 보관해두고 있다가 필요할 때 제공하는 것
```
// SqlService 서비스 객체의 구조
DAO -> SqlService 
					-> 읽기 요청      -> SqlReader    ---> Sql 리소스
          -> Sql 등록,조회  -> SqlRegistry  <--- SqlUpdater
```

## 584 페이지부터...






























