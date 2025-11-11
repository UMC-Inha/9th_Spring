# Lombok

`Lombok`은 Java 개발에서 **반복되는 보일러플레이트 코드**를 줄여주는 라이브러리입니다.

getter/setter, 생성자, equals/hashCode, toString 같은 코드들을 자동으로 만들어줍니다.

즉, 개발자가 꼭 작성해야 했지만 의미는 명확해서
매번 타이핑하기 귀찮았던 코드들을 **어노테이션으로 처리**할 수 있게 해주는 도구라고 보면 됩니다.


## Lombok이 필요한 이유

Java는 기본적으로 문법이 장황하고, DTO·엔티티 같은 단순 데이터 객체만 만들어도 코드가 길어집니다.

```java
public class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
}
```

Lombok을 쓰면 이렇게 줄어듭니다!

```java
@Getter
@AllArgsConstructor
public class User {
    private String name;
    private int age;
}
```

## 자주 쓰는 Lombok 어노테이션

| 어노테이션 | 역할 |
| --- | --- |
| `@Getter` / `@Setter` | getter / setter 자동 생성 |
| `@NoArgsConstructor` | 기본 생성자 생성 |
| `@AllArgsConstructor` | 모든 필드 생성자 |
| `@RequiredArgsConstructor` | final 필드/`@NonNull` 필드 생성자 |
| `@Builder` | 빌더 패턴 자동 생성 |
| `@ToString` | toString() 자동 |
| `@EqualsAndHashCode` | equals(), hashCode() 자동 |
| `@Data` | getter + setter + toString + equals/hashCode + 생성자 |

## 주의점

Lombok을 사용할 때는 주의할 점이 있다. IDE 플러그인 의존성이 존재해 추가 설정이 필요하고, 코드가 애노테이션 뒤로 숨기 때문에 IDE의 도움 없이는 가독성이 떨어진다. 특히 `@Data`를 남용하면 객체가 과도하게 mutable해져 예상치 못한 동작이 발생할 수 있다. 또한 애노테이션 프로세싱 과정 때문에 프로젝트 초기 기동 속도가 미세하게 증가한다.

특히 **JPA Entity에 `@Data`를 사용하지 않는 것은 사실상 업계 표준이다.** 무분별하게 setter가 생성되면 엔티티 무결성이 깨질 수 있으며, equals/hashCode가 잘못 생성되면 영속성 컨텍스트에서 의도치 않은 문제를 유발한다. 게다가 lazy-loading 필드가 포함된 toString이 자동 생성되면 무한 루프나 성능 저하가 발생할 위험이 있다. 따라서 엔티티의 경우 필요한 메서드만 명시적으로 정의하며, 설계 의도를 코드에 드러내는 것이 바람직하다.