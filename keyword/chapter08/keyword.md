## Java Exception

- 애플리케이션 실행 중에 발생 수 있는 개발자가 처리할 수 있는 문제를 의미한다.
- Exception을 상속받는 Runtime Exception은 **`Unchecked Exception`**이다.
    - Unchecked Exception: 런타임 시점에 발생하는 예외로 컴파일러가 예외 처리를 강제하지 않는다.
- Runtime Exception을 제외한 예외는 **`Checked Exception`**이다.
    - Check Exception: 컴파일 시점에 반드시 처리해야 하는 예외. 개발자가 반드시 try-catch문으로 처리해야한다.

### Checked Exception

- **Exception**
    - 모든 Checked Exception의 부모 클래스
- **IOException**
    - 읽기/쓰기 하는 도중 오류가 발생했을 때 발생하는 예외 클래스
    - 네트워크를 통해서 다른 컴퓨터와 데이터 교환 중 오류가 발생했을 때 발생하는 예외 클래스
- **FileNotFoundException**
    - 파일을 찾을 수 없을 때 발생하는 예외 클래스
- **SQLException**
    - 데이터베이스 액세스 작업 중 오류가 발생했을 때 발생하는 예외 클래스

### Unchecked Exception

- **RuntimeException**
    - 모든 Unchecked Exception의 부모 클래스
- **NullPointerException**
    - 참조 변수의 값이 NULL인 상태에서 필드나 메소드를 사용할 때 발생하는 예외 클래스
- **ClassCaseException**
    - 클래스의 형변환이 가능하지 않을 때 발생하는 예외 클래스
- **ArithmeticException**
    - 나눗셈에서 어떤 값을 0으로 나눌 때 발생하는 예외 클래스
- **IndexOutOfBoundsException**
    - 배열, 리스트, 문자열에서 인덱스 범위를 벗어난 위치를 조회했을 때 발생하는 예외 클래스
- **NumberFormatException**
    - Integer.parseInt(s), Double.parseDouble(s) 등을 실행할 때 발생하는 예외 클래스
    - IllegalArgumentException을 상속 받는다.
- **IllegalArgumentException**
    - 매개 변수에 전달된 인자가 유효하지 않을 때 발생하는 예외 클래스

