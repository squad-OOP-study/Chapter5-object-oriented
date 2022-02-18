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

