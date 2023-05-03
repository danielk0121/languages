```java
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.util.StringUtils;
import org.springframework.web.client.RestTemplate;

public class Sample {
    public void sampleGetRequest() {
        HttpComponentsClientHttpRequestFactory httpRequestFactory = new HttpComponentsClientHttpRequestFactory();
        httpRequestFactory.setConnectTimeout(5000);
        RestTemplate rt = new RestTemplate(httpRequestFactory);
        HttpHeaders headers  = new HttpHeaders();
        HttpEntity<?> entity = new HttpEntity<>(headers);

        ResponseEntity<String> responseEntity = rt.exchange(url, HttpMethod.GET, entity, String.class);
        String resBody = responseEntity.getBody();
        resBody = StringUtils.hasText(resBody) ? resBody.trim() : "";
        log.info("response: {}", resBody);        
    }
}
```