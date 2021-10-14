---
layout: post
title: Spring Cache
summary: Spring Cache
author: devhtak
date: '2021-10-12 21:41:00 +0900'
category: Spring
---

#### Spring Cache

- Spring Cache 는 cache 기능을 추상화를 지원
- EhCache, Couchbase, Redis 등의 추가적인 캐시 저장소와 빠르게 연동하여 bean으로 설정할 수 있도록 도와준다.
- 추가적인 캐시 저장소와 연결하지 않는다면, ConcurrentHashMap 기반의 Map 저장소가 자동으로 추가된다.
- 간단하게 토큰 정도만 캐시처리가 필요 한 경우 빠르게 사용할 수 있다.
- application간의 공유가 불가능하기 때문에, clustering isntance간에 공유해야 하는 경우 Redis, MemCached를 이용하는게 맞다.

#### Spring Cache 구성

- dependency 추가

- bean 설정
  ```java
  @EnableCaching
  @Configuration
  public class LocalCacheConfig {

      @Bean
      public CacheManager cacheManager() {
          SimpleCacheManager simpleCacheManager = new SimpleCacheManager();
          simpleCacheManager.setCaches(List.of(
                  new ConcurrentMapCache("example.store1"), new ConcurrentMapCache("example.store2")));

          return simpleCacheManager;
      }
  }
  ```
  - @EnableCaching 을 붙여야 한다.
  - 여러개의 저장소를 설정하는 경우 List로 만들어 반환하면 된다.

#### ConcurrentMapCache

- ConcurrentMapCache.java
  ```java
  public class ConcurrentMapCache extends AbstractValueAdaptingCache {
      private final String name;
      private final ConcurrentMap<Object, Object> store;
      @Nullable
      private final SerializationDelegate serialization;

      public ConcurrentMapCache(String name) {
          this(name, new ConcurrentHashMap(256), true);
      }

      public ConcurrentMapCache(String name, boolean allowNullValues) {
          this(name, new ConcurrentHashMap(256), allowNullValues);
      }

      public ConcurrentMapCache(String name, ConcurrentMap<Object, Object> store, boolean allowNullValues) {
          this(name, store, allowNullValues, (SerializationDelegate)null);
      }

      protected ConcurrentMapCache(String name, ConcurrentMap<Object, Object> store, boolean allowNullValues, @Nullable SerializationDelegate serialization) {
          super(allowNullValues);
          Assert.notNull(name, "Name must not be null");
          Assert.notNull(store, "Store must not be null");
          this.name = name;
          this.store = store;
          this.serialization = serialization;
      }
      // ...
  }
  ```
  - JDK java.util.concurrent package 코어를 기반으로 구현한 단순 cache
  - ConcurrentHashMap 을 사용한다.
    - ConcurrentHashMap에 경우, null을 저장할 때 내부에 미리 정의된 default object로 대체된다.
    - 하지만, allowNullValues에 따라 다르게 설정될 수 있다.

- ConcurrentHashMap 
  - Hashtable 클래스의 단점을 보완하면서 MultiThread 환경에서 사용할 수 있도록 나온 클래스
    - Hashtable 은 get, put 메소드에 synchronized로 구현되어 있어 MultiThread 환경에 적합한 듯 보이나 동시에 작업하려면 Lock을 갖기 때문에 병목현상이 발생한다.
    - 동시성을 높이기 위해 나왔다
    - get 메서드에는 synchronized가 존재하지 않고 put 중간에 synchronized가 존재한다.
    - ConcurrentHashMap은 읽기 작업에는 여러 쓰레드가 동시에 읽을 수 있지만, 쓰기 작업에는 특정 세그먼트, 버킷에 대한 Lock을 사용

#### Service

- caching 대상 데이터
  ```java
  public class CacheData {
      private String value;

      private LocalDateTime expirationDate;

      public CacheData() {}

      public CacheData(String value, LocalDateTime expirationDate) {
          this.value = value;
          this.expirationDate = expirationDate;
      }

      public String getValue() {return value;}
      public LocalDateTime getExpirationDate() {return expirationDate;}
      public void setValue(String value) {this.value = value;}
      public void setExpirationDate(LocalDateTime expirationDate) {this.expirationDate = expirationDate;}
  }
  ```
  
- caching service
  ```java
  @Service
  public class CacheService {

      private static  final CacheData EMPTY_DATA = new CacheData();

      @Cacheable(cacheNames = "example.store1", key="#key")
      public CacheData getCacheData(final String key) {
          System.out.println(key + "에 한 값이 존재하지 않는 경우 실행");
          return EMPTY_DATA;
      }

      @CachePut(cacheNames = "example.store1", key="#key")
      public CacheData updateCacheData(final String key, final String value) {
          System.out.println(key + " 에 대한 값을 " + value + "로 업데이트하는 경우 실행");
          CacheData cacheData = new CacheData();
          cacheData.setValue(value);
          cacheData.setExpirationDate(LocalDateTime.now().plusDays(1));

          return cacheData;
      }

      @CacheEvict(cacheNames = "example.store1", key="#key")
      public boolean expireCacheData(final String key) {
          System.out.println(key + " 를 삭제하는 경우 실행");
          return true;
      }
      
      public boolean isValidation(final CacheData cacheData) {
          return !ObjectUtils.isEmpty(cacheData)
                && !ObjectUtils.isEmpty(cacheData.getExpirationDate())
                && StringUtils.hasText(cacheData.getValue())
                && cacheData.getExpirationDate().isAfter(LocalDateTime.now());
      }
  }
  ```
  
- @Cacheable
  - cache를 가져오는 메서드에 붙이는 애노테이션
  - cacheNames는 cache 저장소 이름(ConcurrentMapCache), value도 같은 역할을 한다.
  - key에 경우 캐시 데이터가 들어있는 key 값이며, ConcurrentHashMap의 key이다
  - key에 대한 value가 존재하면 기존 value가 리턴되고, getCacheData 는 수행되지 않는다.

- @CachePut
  - cache 저장소(example.store1) 의 key에 대한 value 를 업데이트 할 때 사용

- @CacheEvict
  - cache 저장소(example.store1) 의 key에 대한 value 를 삭제 할 때 사용

- key
  - #key: SpEL(Spring Expression Language) 문법을 사용
  - String, Integer, Long 등의 값은 #변수명 형태로 사용 가능
  - 객체안의 멤버변수를 비교하는 경우 #객체명.멤버명 형태로 사용
  - 파라미터를 보고 KeyGenerator에 의해 default key를 생성
    - 파라미터가 없는 경우: 빈값
    - 하나인 경우: 해당 값
    - 여러 개인 경우: 모든 파라미터를 포함한 키 생성(모든 파라미터 및 해시코드 조합)

- 조건 부여
  ```java
  @CachePut(cacheNames="example.store1", key="#cacheData.value", condition="#cacheData.value.length() > 5")
  public CacheData updateCacheData(CacheData cacheData) { return cacheData; }
  ```
  -  cacheData에 대한 조건을 넣을 수 있다.
  

#### Test

```java
@SpringBootTest
class CacheServiceTest {

    @Autowired CacheService cacheService;

    @Test
    @DisplayName("cache data test")
    void getCacheTest() {
        String key = "testKey";
        String value = "testValue";
        CacheData cacheData = cacheService.getCacheData(key);

        if(!cacheService.isValidation(cacheData)) {
            cacheData = cacheService.updateCacheData(key, value);
        }
        System.out.println(cacheData.toString());

        CacheData returnData = cacheService.getCacheData(key);
        System.out.println(returnData.toString());

        assertThat(cacheData.getValue()).isEqualTo(returnData.getValue());
        assertThat(cacheData.getExpirationDate()).isEqualTo(returnData.getExpirationDate());

        cacheService.expireCacheData(key);
        CacheData emptyData = cacheService.getCacheData(key);
        assertThat(emptyData.getValue()).isNull();
    }
}
```

#### 출처

- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/concurrent/ConcurrentMapCache.html
- https://devlog-wjdrbs96.tistory.com/269
- https://sunghs.tistory.com/132
