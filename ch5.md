# ch5 객체지향 설계 원칙 : SOLID

- 객체 지향적으로 설계하는데 기본이 되는 설계 원칙인 SOLID는 다음의 다섯 가지 원칙으로 구성된다
- 원칙들은 서로 밀접하게 연결되어 있으므로 종합적으로 연결지어 이해할 수 있도록 하자
<br>

:one: 단일 책임 원칙 (Single responsibility principle; SRP)

:two: 개방-폐쇄 원칙 (Open-closed principle; OCP)

:three: 리스코프 치환 원칙 (Liskov substitution principle: LSP)

:four: 인터페이스 분리 원칙 (Interface segregation principle: ISP)

:five: 의존 역전 원칙 (Dependency inversion principle; DIP)
***
<br>

## :one: 단일 책임 원칙

객체 지향의 기본은 책임을 객체에게 할당하는 데 있다. 객체를 객체로 존재하게 하는 이유가 책임인데, 단일 책임 원칙은 이 책임과 관련된 원칙이다.

#### 단일 책임 원칙 ->  클래스는 단 한 개의 책임을 가져야 한다 -> 클래스를 변경하는 이유는 단 한 개여야 한다

- 단일 책임 원칙은 가장 중요한 원칙 중의 하나이다
- 단일 책임 원칙이 잘 지켜지지 않으면 다른 원칙들도 그 효과가 반감된다
- 중요하지만 가장 어려운 원칙이다
- 한 개의 책임에 대한 정의가 명확하지 않고, 책임을 도출하기 위해서는 다양한 경험이 필요하기 때문이다

### < 단일 책임 원칙 위반이 불러오는 문제점 >

#### :one: 한 책임의 변화가 다른 책임에 영향을 준다
```java
public class DataViewer {

  public void display() {
    String data = loadHtml();
    updateGui(data);

  }

  public String loadHtml() {
    HttpClient client = new HttpClient();
    client.connect(url);
    return client.getResponse();

  }

  private void updateGui(String data) {
    GuiData guiModel = parseDataToGuiData(data);
    tableUI.changeData(guiModel);

  }

  private GuiData parseDataToGuiData(String data) {
    ... // 파싱 처리 코드

  }

  ... // 기타 필드 등 다른 코드

}
```
- 위 코드는 HTTP 프로토콜을 이용해서 데이터를 읽어온 후 화면에 보여주는 기능을 구현한 예시이다
- display 메서드는 loadHtml 메서드에서 읽어 온 HTML 응답 문자열을 updateGui 메서드에 보낸다
- updateGui 메서드는 parseDataToGuiData 메서드를 이용해서 HTML 응답 메시지를 GUI에 보옂기 위한 GuiData 객체로 변환한 뒤에 실제 tableUI를 이용해서 데이터를 보여주고 있다
<br>

- 만약 데이터를 제공하는 서버가 HTTP 프로토콜에서 소켓 기반의 프로토콜로 변경되면 이 프로토콜은 응답 데이터로 byte 배열을 제공한다
- 즉 display 메서드에서 data 타입이 String 에서 byte[]로 변경될 것이다
<br>

![image](https://user-images.githubusercontent.com/69443895/154730506-4757a727-170f-4152-acf3-16dab0e6cf34.png)

- 데이터 타입이 변경되면서 위 그림과 같이 연쇄적으로 변화가 발생했다
- 위 문제는 한 클래스에 데이터를 읽는 책임과 화면에 보여주는 책임이 밀접하게 결합되어 있어 발생한 문제다
- 책임의 개수가 많아질수록 한 책임의 기능 변화가 다른 책임에 주는 영향은 비례해서 증가하게 된다
- 이는 결국 코드를 절차 지향적으로 만들어 변경을 어렵게 만든다
<br>

![image](https://user-images.githubusercontent.com/69443895/154731061-41b7ae33-4a92-4581-81ec-7835e5cc9544.png)

- 위 그림과 같이 두 책임을 분리하고, 데이터의 타입을 저수준의 String이 아닌 알맞게 추상화된 타입을 사용하면, 데이터를 읽어 오는 부분의 변경 때문에 화면을 보여주는 부분의 코드가 변경되는 상황을 막을 수 있다
- DataLoader 클래스에서 내부적인 변화가 생겨도 DataDisplayer는 영향을 받지 않는다
<br>

#### :two: 재사용이 어렵다
- 앞서 DataViewer 예제로 다시 돌아가 보자. HttpClient 패키지와 GuiComp 패키지가 각각 별도의 jar 파일로 제공된다고 가정한다.
- 이 상태에서 데이터를 읽어 오는 기능이 필요한 DataRequiredClient 클래스를 만들어야 한다면, 구현하기 위해 필요한 것은 DataViewer와 HttpClient jar 파일이다. 
- 하지만, 실제로는 DataViewer가 GuiComp를 필요로 하므로 GuiComp jar 파일까지 필요하다. 
- 즉, 실제로 사용하지 않는 기능이 의존하는 jar 파일까지 필요한 것이다.

<br>

### < 책임이란 변화에 대한 것 >
- 예를 들어, DataViewer 객체에서 데이터를 읽어 오는 기능에 변화가 발생했는데, 이런 변화를 통해 데이터를 읽어 오는 기능이 별도로 분리되어야 할 책임이라는 것을 알 수 있을 것이다.
- 그러면 어떻게 단일 책임 원칙을 지킬 수 있을까? 
  + 방법은 바로 메서드를 실행하는 것이 누구인지 확인해 보는 것이다
  + 객체의 사용자들이 서로 다른 메서드들을 사용한다면 그들 메서드는 각각 다른 책임에 속할 가능성이 높으므로 책임 분리 후보가 된다.(위 예제처럼 한 클래스에서 데이터 읽기/보여주기 메서드가 있는 경우)
***
<br>


