# item 12. toString을 항상 재정의하라

## toString이란?
### Object 란?
- 모든 클래스의 가장 최상위 클래스
    - JAVA에서 제공하는 모든 클래스들은 계층 구조로 이루어져있고, 최상위로 올라가면 Object 클래스가 존재
    - 자바 컴파일러는 일반 클래스를 Object의 하위 클래스로 자동 설정
> 즉, 자바 라이브러리나 유저가 만든 모든 클래스는 Object 클래스를 부모클래스로 상속 받아 사용

### toString() 메소드란?
- 객체가 가지고 있는 정보나 값들을 문자열로 만들어 리턴하는 메소드
- toString()
    ```java
        public String toString() {
            return getClass().getName() + "@" + Integer.toHexString(hashCode());
        }
    ```
    - 예제
        ```java
        class test
        {
            int x;
            int y;
        }

        public class Main {
            public static void main(String[] args) {
                test t = new test();
                System.out.println(t.toString()); // test@123772c4
            }
        }
        ```

## toString은 항상 재정의하라
### toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉬움
- 작성한 객체를 참조하는 컴포넌트가 오류 메시지를 로깅할 때 자동으로 호출
    - 제대로 재정의하지 않는다면 쓸모없는 메시지만 로그에 남음
    - 재정의를 통해 어디서 오류가 발생했는지 추적할 수 있음
    - 예제
        ```java
            class test
            {
                int x;
                int y;

                @Override
                public String toString(){
                    return "test [x=" + x + ", y = " + y +
                            ", getClass()=" + getClass() + ", hashCode()=" + hashCode()
                            + ", toString()=" + super.toString()+ "]";
                }
            }

            public class Main {
                public static void main(String[] args) {
                    test t = new test();
                    System.out.println(t.toString()); // test [x=0, y = 0, getClass()=class test, hashCode()=305623748, toString()=test@123772c4]
                }
            }
        ```
        
### toString은 그 객체가 가진 주요 정보 모두를 반환하는게 좋음
- 이상적인 것은 스스로를 완벽히 설명하는 문자열이어야 함.
- **하지만** 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않으면 무리가 있음
    - 이런 상황일 경우 요약된 정보를 담는 것이 좋음
        - ex) `맨해튼 거주자 전화번호부(총 1487237개)` or `Thread[main,5,main]`

### toString() 문서화 포맷
- 반환값의 포맷을 문서화하여 명시
    - 값 클래스는 문서화를 권함
        - 전화번호, 행렬
    - 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공
        - ex) `Integer.toString();` / `Integer.parseInt(string);`

- 장점
    - 표준적이고, 명확하고, 사람이 읽을 수 있음
    - CSV 같이 데이터 객체 저장도 가능

- 단점
    - 포맷을 한번 입력하면 영구적으로 해당 퐷에 얽매이게 됨
    - 포맷을 명시하지 않는다면 향후 릴리스에서 정보를 더 넣거나 포맷을 개선할 수 있는 유연성을 얻을 수 있음

- 포맷을 명시할 경우
    ```java
        /**
        * 이 전화번호의 문자열 표현을 반환한다.
        * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
        * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
        * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
        *
        * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
        * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
        * 전화번호의 마지막 네 문자는 "0123"이 된다.
        */
        @Override public String toString() {
            return String.format("%03d-%03d-%04d",
                    areaCode, prefix, lineNum);
        }
    ```

- 포맷을 명시하지 않는 경우
    ```java
        /**
        * 이 약물에 관한 대략적인 설명을 반환한다.
        * 다음은 이 설명의 일반적인 형태이나,
        * 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
        *
        * "[약물 #9: 유형=사람, 냄새=테레빈유, 겉모습=먹물]"
        */
        @Override public String toString() { ... }
    ```

### toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자!
- 반환값에 대한 접근자를 제공해야함
    - 반환값의 접근자가 없으면...
        - 프로그래머는 toString의 반환값을 파싱할 수 밖에 없음
        - 향후 포맷을 바꾸면 시스템이 망가지는 결과를 초래


### String() 메서드를 재정의하지 않아도 되는 클래스
- 정적 유틸리티 클래스
- 열거타입
    - 자바가 이미 완벽한 toString을 제공
    - 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스라면 재정의 해야함

> 모든 구체 클래스에서 Object의 toString을 재정의하자. 상위 클래스에서 이미 알맞게 재정의한 경우는 예외다. toString을 재정의한 클래스ㅡㄴ 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다. toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.
