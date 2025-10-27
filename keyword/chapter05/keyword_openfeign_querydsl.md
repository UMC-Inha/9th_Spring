
### 1. 보안 취약점 발생 (CVE-2024-49203)

2024년 QueryDSL(5.x 이하)에서 `orderBy()` 관련 HQL/SQL 인젝션 취약점 **CVE-2024-49203**이 보고되었다.
해당 취약점은 사용자 입력을 적절히 검증하지 않은 채 동적으로 쿼리를 생성하는 과정에서 악의적인 문자열이 주입될 수 있는 문제이다. 이로 인해 서비스 지연(DoS)이나 데이터 노출이 발생할 수 있다.
더불어 원본 QueryDSL 프로젝트의 릴리스 및 유지보수 활동이 사실상 정체되어 있었기 때문에 취약점에 대한 대응이 지연되는 문제가 발생하였다.

---

### 2. QueryDSL 개발 정체와 배경

기존 QueryDSL은 자바의 어노테이션 프로세서를 이용해 `QMember`, `QUser` 등 Q 클래스를 생성하는 구조이다.
자바 프로젝트에서는 `annotationProcessor` 설정만으로 코드 생성이 원활하지만, Kotlin 프로젝트에서는 이를 수행하기 위해 `kapt`(Kotlin Annotation Processing Tool)를 사용해야 한다.
그러나 `kapt`는 2024년 이후 유지보수 모드로 전환되어 더 이상의 구조적 개선이나 기능 개선이 이루어지지 않고 있으며, 그 결과 빌드 속도 저하와 Kotlin 최신 기능과의 비효율 등 한계가 존재한다. 이러한 `kapt`의 한계는 QueryDSL을 Kotlin 환경에서 점차 취약하게 만들었다.

---

### 3. OpenFeign 포크와 해결책

이 문제를 해결하기 위해 커뮤니티와 Netflix의 OpenFeign 팀은 QueryDSL을 포크하여 `OpenFeign/querydsl`로 개발을 재개하였다.
OpenFeign 포크는 `kapt` 의존을 제거하고 **KSP(Kotlin Symbol Processing)** 기반의 코드 생성으로 전환하였다. 또한 CVE-2024-49203에 대한 패치를 적용하였다.
그 결과 OpenFeign QueryDSL은 빌드 성능과 Kotlin 최신 버전과의 호환성을 개선하였고, 보안 문제를 해결한 실무 사용 가능한 현대화된 대안이 되었다.

---

### 4. 권장 사항

스프링 환경에서 `kapt` 의존이 모든 프로젝트에 해당하는 것은 아니지만, CVE-2024-49203 취약점은 스프링 부트 사용 여부와 무관하게 원본 QueryDSL 아티팩트를 사용할 경우 영향을 미칠 수 있다.
따라서 가능한 한 **OpenFeign 포크**(`io.github.openfeign.querydsl`, 6.x 계열)로 전환하는 것을 권장한다. OpenFeign 포크는 보안 패치 적용, KSP 기반 코드 생성 도입, Kotlin 최신 버전 호환성 향상이라는 장점을 제공한다.

