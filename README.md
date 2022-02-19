# Chapter5-object-oriented
## 설계 원칙: SOLID feat(개발자가 반드시 정복해야할 객체 지향과 디자인 패턴)
- 컴퓨터 프로그래밍에서 SOLID란 로버트 마틴이 2000년대 초반에 명명한 객체 지향 프로그래밍 및 설계의 다섯 가지 기본 원칙을 마이클 페더스가 두문자어 기억술로 소개한 것이다.
- 간단히 말해, 변화에 유연하게 대처할 수 있는 설계 원칙

### 1. 단일 책임 원칙(Single responsibility principle)
- 클래스는 단 한개의 책임을 가져야 한다는 규칙
  - 다른 말로, 클래스를 변경하려는 이유는 단 한가지여야 한다.
#### 🔖 단일 책임 원칙을 지키지 않았을 때 일어나는 일
1. 한 메서드의 변경이 다른 메서드에 영향을 끼치게 된다.
![1](https://user-images.githubusercontent.com/95393311/154807964-afaa4f11-a05f-40cd-841a-e8f0ac5dcc76.JPG)
- 위의 예제에서 `Data를 읽는 책임과 이를 바탕으로 읽어진 Data를 GUI를 통해 보여주는 책임`이 한 클래스에 존재한다면?
  - 추가 요구 사항: `Data 타입`이 바꿔라 -> `Data를 읽는 메서드를 수정해야 한다`
  - 추가 요구 사항: `Data를 보여주는 방식을 GUI가 아닌 다른 방식` 으로 바꿔라 -> `Data를 보여주는 GUI 메서드를 수정해야 한다`
- 따라서 2개 이상의 이유로 위의 클래스는 변경될 수 있으며, 각 메서드의 변경이 다른 메서드의 변경을 야기시킬 수 있다.
2. 재사용성이 어려워짐
- 위의 예제에서 사용자는 데이터를 읽어오는 `loadData()` 만 이용하고 싶을 때, 불필요하게 `displayDataByGui()` 메서드를 구현하는 패키지(jar파일)도 같이 필요하게 된다.

#### 🔖 단일 책임 원칙을 지킬 수 있는 방법
![2](https://user-images.githubusercontent.com/95393311/154807966-ae3f4ca7-2be3-4498-89e3-ea2f9665bd3e.JPG)
- 메서드를 기준으로, 클래스를 사용하는 사용자가 각각 다른 메서드가 필요하면 해당 클래스의 책임을 분리해야 하는 가능성이 높다.

### 2. 개방 폐쇄 원칙(Open-closed principle)
- 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.
  - 기능을 변경하거나 확장할 수 있으면서
  - 그 기능을 사용하는 코드는 수정하지 않는다
- 개방 폐쇄 원칙은 유연함에 대한 것

#### 🔖 개방 페쇄 원칙을 지키는 방법
![3](https://user-images.githubusercontent.com/95393311/154809017-2c05b7a8-e54c-45d8-ae29-dbdea8139633.JPG)
1. 추상 타입을 사용하자!(챕터 3장에서 배운 내용)
   - 위의 예제에서 원래 파일과 소켓으로부터만 Byte 리소스를 얻었지만 추가 요구 사항으로 메모리로부터도 Byte 리소스를 얻어야 한다면 `MemoryByteSource`와 같이 콘크리트 클래스만 추가하면 FlowController는 변경하지 않아도 된다
     - 즉, 확장에는 열려 있어야 하고 (`MemoryByteSource 추가`) 변경에는 닫혀 있어야 한다. (`FlowController 변경없음`)
2. 상속을 쓰자
   - 상속은 재사용성의 관점보다는 기능 확장의 관점에서 사용해야 한다는 점을 잊지 말자

#### 🔖 개방 페쇄 원칙이 깨지는 원인
1. 다운 캐스팅을 할 때
업캐스팅이란 자식 클래스의 객체 타입을 부모 클래스 타입으로 형변환하는 것
다운캐스팅은 반대로 업캐스팅된 객체를 다시 자식 클래스의 타입으로 형변환하는 것
- 자바에서는 `instanceof`를 사용하지 말자
- 코틀린에서는 `if (is 하위클래스타입)`를 사용하지 말자
  - 추가 구현 클래스가 생기면 `if ( is 추가 구현 클래스 타입)` 가 추가로 수정돼야 한다

2. 비슷한 if-else 블록이 존재할 때

> 해결하는 방안은?
>> 변화가 발생하는 부분을 추상화하기(챕터 3장)

### 3. 리스코프 치환 원칙(Liskov substitution principle)
- LSP는 개방 폐쇄 원칙을 받쳐 주는 다형성에 관한 원칙을 제공
- 상위 타입의 객체를 하위 타입의 겍체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.
```
fun someMethod(class: SuperClass) {}
someMethod(superClass())
someMethod(subClass())
```
- 리스코프 치환 원칙을 위반하면 결국 개방 폐쇄 원칙도 위반하게 된다.
- 리스코프 치환 원칙은 계약과 확장에 대한 내용이다.

#### 🔖 리스코프 치환 원칙을 위반하는 이유?
1.실제로 상속 관계가 아니다 
  - 직사각형-정사각형 문제: 정사각형은 직사각형과 상속 관계가 아니다.
2. 계약 - 상위 타입에서 정의된 명세를 벗어날 때
  - 명시된 명세에서 벗어난 값을 리턴
    - 상위 타입에서는 0 이상 값을 리턴하는데 하위 타입에서는 -1과 같이 0 미만의 값을 리턴함
  - 명시된 명세에서 벗어난 예외상황을 발생
    - 상위 타입에서 IOException만 발생시킨다고 정의했지만 하위 타입에서 IllegalArgumentException이 발생됨
  - 명시된 명세에서 벗어난 기능을 수행
3. 확장 - 다운 캐스팅을 이용한다

#### 🔖 리스코프 치환 원칙을 지키는 방법?
1. 정의된 명세를 지키자
2. 변화가 발생하는 부분을 추상화하자


### 4. 인터페이스 분리 원칙(Interface segregation principle)
- 인터페이스는 그 인터페이스를 사용하는 클라이언트를 기준으로 분리해야 한다.
  - 클라이언트는 자신이 사용하는 메서드에만 의존해야 한다

#### 🔖 인터페이스를 분리하는 이유?
![4](https://user-images.githubusercontent.com/95393311/154811428-e7238a08-b781-4be4-b28e-727eba605687.jpg)
1. 한 클라이언트의 영향이 다른 클라이언트에게 영향을 미치지 않는다
2. 용도에 맞는 인터페이스 분리는 곧 단일 객체의 원칙과 연결된다.
   - 위에서도 말한 변경의 여파를 최소화할 수 있다.
   - 인터페이스와 콘크리트 클래스의 재사용성이 높아진다.

<복합기 예시>  
![6](https://user-images.githubusercontent.com/95393311/154814114-6f3853a8-7b59-4867-a5cc-0abfcd0037b0.JPG)  
[출처](https://defacto-standard.tistory.com/114?category=703460)

#### 🔖 인터페이스를 분리 기준
- 인터페이스를 분리하는 기준은 클라이언트
  - 각 클라이언트가 사용하는 기능을 중심으로 분리

### 5. 의존 역전 원칙(Dependency inversion principle)
- 고수준 모듈은 저수준 모듈의 구현에 의존해서는 안된다. 저수준 모듈이 고수준 모듈에서 정의한 추상 타입에 의존해야 한다.
- 고수준 모듈
  - 어떤 의미 있는 단일 기능을 제공하는 모듈
- 저수준 모듈
  - 고수준 모듈의 기능을 구현하기 위해 필요한 하위 기능의 실제 구현
  
#### 🔖 고수준 모듈이 저수준 모듈에 의존할 때의 문제
- 저수준 모듈의 변경이 고수준 모듈의 변경을 야기시킬 수 있다

#### 🔖 의존 역전 원칙을 지키는 방법
![5](https://user-images.githubusercontent.com/95393311/154812631-00a0032d-b1a4-4e6f-82b4-81daa68ac05d.JPG)
- 추상화를 통해 고수준 모듈과 저수준 모듈 모두를 추상 타입에 의존하도록 함
- 의존 역전 원칙은 런타임의 의존이 아닌 소스 코드의 의존을 역전시킴으로써 변경의 유연함을 확보하는 원칙
  - 런타임의 의존을 역전시키는 것이 아니다!
- 의존 역전 원칙은 타입의 소유도 역전시킨다
  - 즉, 기능에 대한 타입의 소유가 저수준 모듈에서 고수준 모듈로 이동하게 된다.
  - 저수준 모듈을 독립적으로 배포할 수 있게 됨
  - 이로 인해 개방 폐쇄 원칙을 클래스 수준뿐만이 아니라 패키지 수준까지 확장시킴
> 의존 역전 원칙은 리스코프 치환 원칙과 개방 폐쇄 원칙을 만들어주는 기반


### 6. SOLID 정리
- 각 원칙들은 서로 밀접하게 연결되어 있음
- 단일 책임 + 인터페이스 분리 원칙 -> 객체가 많은 책임(기능)을 가지지 않도록 함
  - 변경의 여파를 최소화함
  - 유연성과 재사용성 높임
- 리스코프 치환 + 의존 역전 원칙 -> 개방 폐쇄 원칙을 지원
  - 다형성(리스코프 치환 원칙이 도와줌) + 추상화(리스코프 치환 원칙이 도와줌)를 통해 유연성과 재사용성 높임
- SOLID 원칙은 사용자 관점에서의 설계/기능 사용을 지향
  - 인터페이스 분리 원칙
    - `클라이언트 입장` 에서 인터페이스 분리
  - 의존 역전 원칙
    - 저수준 모듈을 `사용하는 고수준 모듈 입장`에서 추상화 타입을 도출
  - 리스코프 치환 원칙
    - `사용자`에게 기능 명세를 제공하고 이에 대한 기능 구현을 약속