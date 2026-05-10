### 컬렉션
- JPA는 기본적으로 Collection, List, Set, Map 과 같은 컬렉션을 지원하고 아래와 같은 경우 사용할 수 있다.
  - @OneToMany, @ManyToMany를 사용해 일대다, 다대다 엔티티 관계 매핑할 때
  - @ElementCollection 을 사용해서 값 타입을 하나 이상 보관할 때

#### JPA와 컬렉션
```java
@Entity
public class Team {
  @Id private Long id;
  @OneToMany(mappedBy="team")
  private Collection<Member> memebers = new ArrayList<>();
}
@Entity
public class Member {
  @Id private Long id;
  @ManyToOne
  @JoinColumn(name="team_id")
  private Team team;
}
```
```java
Team team = new Team();
System.out.println("before = " team.getMembers().getClass()); //java.util.ArrayList
entityManager.persist(team);
System.out.println("after = " team.getMembers().getClass()); // org.hibernate.collection.internal.PersistentBag
```
- 하이버네이트가 제공하는 내장 컬렉션은 원본 컬렉션을 감싸고 있어 레퍼 컬렉션이라고 한다.
  - Collection: org.hibernate.collection.internal.PersistentBag (중복 허용, 순서 보관X) 
  - List: org.hibernate.collection.internal.PersistentBag (중복 허용, 순서 보관X)
  - Set: org.hibernate.collection.internal.PersistentSet (중복 허용X, 순서 보관X)
  - List + @OrderColumn: org.hibernate.collection.internal.PersistentList (중복 허용, 순서 보관)
- Collection과 List
  - Collection과 List는 엔티티를 추가할 때 중복된 엔티티가 있는지 비교하지 않고 단순히 저장만 하면 된다.
  - 엔티티를 추가해도 지연 로딩된 컬렉션을 초기화하지 않는다.
- Set
  - 엔티티를  추가할 때 중복된 엔티티가 있는지 비교해야 한다.
  - 따라서 엔티티를 추가할 때 지연 로딩된 컬렉션응ㄹ 초기화해야 한다.
- List+ @OrderColumn
  - 데이터베이스에 순서 값을 저장해서 조회할 때 사용한다.
  - 순서가 있는 컬렉션은 데이터베이스에 순서 값도 함께 관린된다.
 
#### OrderColumn의 단점
- 예시
  ```java
  @Entity
  public class Board {
    @Id private Long id;
    @ManyToOne(mappedBy="board")
    @OrderColumn("POSITION")
    private List<Comment> comments = new ArrayList<>();
  }
  @Entity
  public class Comment {
    @Id private Long id;

    private Long position;
    @OneToMany
    @JoinColumn("board_id")
    private Board board;
  }
  ```
- OrderColumn은 Board 엔티티에서 매핑하므로 Comment는 POSITION 값을 알 수 없다. Comment를 insert할 때, POSITION이 저장되지 않고 update 쿼리가 추가 발생
- List 변경 시에 연관된 많은 위치 값을 변경해야 한다. update 계속 발생

#### @OrderBy
- DB order by 절을 사용해 컬렉션을 정렬하며 모든 컬렉션에 사용할 수 있다.

### 컨버터
- Converter를 사용하면 엔티티의 데이터를 변환해 데이터베이스에 저장할 수 있다.
- 예시
  ```java
  @Entity
  public class Member {
    @Id private Long id;
    @Converter(converter=BooleanToYNConverter.class)
    private Boolean vip;
  }
  ```
  ```java
  @Converter
  public class BooleanToYNConverter implements AttributeConverter<Boolean, String> {
    @Override
    public String convertToDatabaseColumn(Boolean attribute) {
      return (attribute != null && attribute) ? "Y" : "N";
    }

    @Override
    public Boolean convertToEntityAttribute(String dbData) {
      return "Y".equals(dbData);
    }
  }
  ```
  - Boolean 타입을 Y,N으로 Converting

### 리스너
- 이벤트 종류
  - New -> (persist()) -> Managed: PrePersist
  - Managed -> (remove()) -> Removed: PreRemove
  - Managed -> (flush()) -> DB : PreUpdate
  - DB -> (find()) -> Managed : PostLoad
- 엔티티의 적용
  ```java
  @Entity
  public class Member {
    @Id private Long id;
    @PrePersist
    public void prePersist() {
      System.out.println("Pre Persist");
    }
    // ...
  }
  ```
- 별도의 리스너 등록
  ```java
  @Entity
  @EntityListeners(DuckListeners.class)
  public class Duck {}

  public class DuckListener {
    @PrePersist private void prePersist(Object obj) { ... }

    @PostLoad private void postUpdate(Object obj) { ... }
  }
  ```
