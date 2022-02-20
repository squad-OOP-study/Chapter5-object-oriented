# Chapter5-object-oriented

설계 원칙: SOLID feat(개발자가 반드시 정복해야할 객체 지향과 디자인 패턴)

1. 단일 책임 원칙(Single responsibility principle)

2. 개방 폐쇄 원칙(Open-closed principle)

3. 리스코프 치환 원칙(Liskov substitution principle)

4. 인터페이스 분리 원칙(Interface segregation principle)

4. 의존 역전 원칙(Dependency inversion principle)

## 1. 단일 책임 원칙

- 클래스는 단 하나만의 책임을 가져야한다.
- 클래스를 바꿀 상황이 생기면 바꾸려는 이유가 하나여야 한다.

### 단일 책임 원칙의 효과

1. 재사용성을 높인다
    - 단일 책임 원칙 미 준수시 타 클래스를 의존할때 필요없는 기능과 패키지 까지 가져오게 됨
    - 단일 책임 원칙 준수 시 클래스를 원하는 만큼만 가져다 씀으로써 재사용 성을 높인다.
  

2. 책임을 분리함으로써 변화에 대한 여파를 줄일 수 있다.
    - 단일 책임 원칙 미 준수시 한 클래스에서 책임들 끼리 의존할때 사용되는 책임의 변화시 수정할 부분이 많아진다.

```
단일 책임 원칙 미준수 상태에서
HTML 응답 문자열 에서 소켓으로 데이터를 로딩하는 방식이 바뀐다면
기존에 loadHtml()을 사용하는 모든 부분과 자료형을 바꾸어야한다.

그러나 기존 클래스에서 데이터 로드 클래스를 분리하고 DataDisplay 에서 데이터 로드 클래스의 loadData() 를 통해
구현한다면 로딩방식의 변화와 기존 클래스와는 무관할 것이다.
데이터 로드 클래스는 내부적으로 다양한 방법의 로딩 메서드를 갖고 외부와는 loadData() 를 통해서만 통신하면됨

// 미 준수 코드 
 class DataViewer {

    fun display() {
        val data = loadHtml()
        updataGui(data)
    }

    private fun loadHtml(): String {
        val client: HttpClient = HttpClient()
        client.connect(url)
        return client.getResponse()
    }
    
}
 
```

## 2. 개방 폐쇄 원칙
- 확장에는 열려 있어야하고 , 변경에는 닫혀 있어야한다.
- 기능에 변경과 확장은 가능하지만 그 기능을 사용하는 코드는 수정하지 않는다,

### 개방 폐쇄 원칙 구현 방법
1. 데이터 추상화
   - 추상화된 데이터 특히 인터페이스를 의존하고 있는 경우 구현체 클라스가 늘어나더라도(확장 O ) 의존 클래스는 변하지 않는다.(수정 X )
   
2. 상속
   - 상위 클래스의 기능을 사용 , 하위클래스에서 일부를 오버라이딩
   - 상위클래스의 기능으로 외부와 통신한다면 하위클래스에서 기능의 확장이 일어나더라도 수정 사항이 발생하지 않는다.
   
### 개방 폐쇄 원칙이 깨지는 증상
1. 다운 캐스팅 , instance of , is 연산자
   - 추상 타입이나 부모 타입에 의존할때 구현체 또는 자식 객체에 대해 다운 캐스팅
   
2. if - else 블록
```
- 코드와 같은 상황에서 결국 if-else 구문을 필요로한다.
- Enemy 클래스에 pathPatern 정보가 바뀌고 추가 될때 마다 코드가 변하는 것은 수정
- Pathpatern 클래스에 pathpatern 정보가 바뀌고 추가될 때마다 if-else 문에 추가되는 것은 확장이라고 볼 수 있음 

```