# 도메인 매핑 시 json 필드 무시
```
import lombok.Builder;
import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@JsonIgnoreProperties(ignoreUnknown = true)  // 누락된 json 필드 무시 !
@Getter @Setter @NoArgsConstructor @ToString @EqualsAndHashCode
public class Actor {

	private long id;
	private String login;
	
	@Builder
	public Actor(long id, String login) {
		super();
		this.id = id;
		this.login = login;
	}
}
```

# json root 가 list 인 경우 역직렬화에서 type 누락에 대한 방지
```
		String source = "[{key1:val1, key2: val2,...}]";  // {} 배열에 대한 json source 문자열
		
		ObjectMapper mapper = new ObjectMapper();
		
		// List 만 지정해서, Model type 이 유실됨, LinkedHashMap 으로 매핑된다
//		@SuppressWarnings("unchecked")
//		List<Event> events = mapper.readValue(source, List.class);  

		// List, Model type 을 지정함, Object 로 매핑된다
		List<Event> events = mapper.readValue(source, 
				mapper.getTypeFactory()
				.constructCollectionType(List.class, Event.class)
				);
```
