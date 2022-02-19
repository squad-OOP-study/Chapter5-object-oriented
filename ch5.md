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


## :two: 개방 폐쇄 원칙

- 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다
<br>

- 기능을 변경하거나 확장할 수 있으면서
- 그 기능을 사용하는 코드는 수정하지 않는다
<br>

### < 추상화를 통한 개방 폐쇄 원칙 구현 >
![image](https://user-images.githubusercontent.com/69443895/154806315-8641ba40-85f4-4637-ad79-a423326ac354.png)

- 위 그림에서 FileByteSource, SocketByteSource의 기능 외에 추가로 메모리에서 byte를 읽어 오는 기능을 추가하기 위해 ByteSource 인터페이서를 상속받은 MemoryByteSource 클래스를 구현해볼 수 있다
- 새로운 기능을 위한 클래스가 추가되었지만, 이 기능을 사용하는 FlowController 클래스의 코드는 변경되지 않는다 
-> 사용되는 기능의 확장에는 열려있고, 기능을 사용하는 코드의 변경에는 닫혀있다
<br>

- 위 경우 확장되는 부분을 추상화했기에 개방 폐쇄 원칙을 구현할 수 있었다
- 인터페이스를 통해 구현했기에 FlowController 에서는 이 인터페이스 타입을 사용하면 되므로 새로운 기능이 추가되었어도 코드의 변경이 생기지 않는다

<br>

### < 상속을 통한 개방 폐쇄 원칙 구현 >
- 상속은 상위 클래스의 기능을 그대로 사용하면서 하위 클래스에서 일부 구현을 오버라이딩 할 수 있는 방법을 제공한다
- 하위 클래스에서 상속 받은 상위 클래스의 메서드를 오버라이딩 하여 기능을 추가해도, 상위 클래스의 메서드는 영향을 받지 않는다
-> 확장에는 열려 있으면서 변경에는 닫혀 있는 것이다

<br>

### < 개방 폐쇄 원칙이 깨질 때의 주요 증상 >

- 타입 다운 캐스팅을 하는 경우
  + 상위 클래스가 확장되면 다운 캐스팅이 일어난 곳에서 함께 수정된다
- 비슷한 if-else 블록이 존재하는 경우
  + 새로운 경우가 추가되는 경우 이에 해당하는 if 블록이 새로 추가될 것이다
  + 즉 변경에 닫혀 있지 않는 상태가 된다
  + 이런 경우 여러 if 블록의 형태에서 해당 기능을 수행하는 새로운 클래스를 구현하도록 한다 -> 기능이 추가될 때 마다 if블록을 추가할 필요가 없어진다
  
<br>

### < 개방 폐쇄 원칙 적용 예제 >

![image](https://user-images.githubusercontent.com/69443895/154807236-01b8da2a-6045-4d86-86f5-a5d93c97490f.png)

- Enemy 클래스는 PathPattern 클래스에 위임하여 이동 기능을 수행 
- 기본 PathPattern을 상속하여 여러 이동 기능을 추가할 수 있다 -> PathPattern 클래스는 변화x -> 개방 폐쇄 원칙 적용, PathPattern 클래스를 사용하는 Enemy도 변화가 없음!!
- PathPattern 도 이동 경로를 처리하는 책임? 만 담당하게 단일 책임 원칙을 준수하도록 구현하기
- Enemy와 같은 컨트롤러의 역할을 하는 클래스에서는 단일 책임 원칙을 준수하는 구체적인 기능을 수행하는 객체들을 조립하는 방식을 사용하는 구조를 가질 수 있다고 생각

<br>

### < 개방 폐쇄 원칙은 유연함에 대한 것 >
- 개방 폐쇄 원칙은 변화가 예상되는 것을 추상화해서 변경의 유연함을 얻도록 해준다
- 코드에 대한 변화 요구가 발생하면, 변화와 관련된 구현을 추상화해서 개방 폐쇄 원칙에 맞게 수정할 수 있는지 확인하는 습관을 가지는게 중요하다
***
<br>


## :three: 리스코프 치환 원칙
- 개방 폐쇄 원칙은 추상화와 다형성(상속)을 이용해서 구현했는데, 리스코프 치환 원칙은 개방 폐쇄 원칙을 받쳐 주는 다형성에 관한 원칙을 제공한다
- 리스코프 치환 원칙은 다음과 같다

#### 상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다
<br>

### < 리스코프 치환 원칙을 지키지 않을 때의 문제 >
:one:
```java
public class Rectangle {
  private int width;
  private int height;

  public void setWidth(int width) {
    this.width = width;

  }

  public void setHeight(int height) {
    this.height = height;

  }

  public void getWidth() {
    return width;

  }

  public void getHeight() {
    return height;

  }

}
```
- 정사각형을 직사각형의 특수한 경우로 보고, 정사각형을 표현하기 위한 Square 클래스가 Rectangle 클래스를 상속받도록 구현을 했다고 하자. 정사각형은 가로와 세로가 모두 동일한 값을 가져야 하므로, Square 클래스는 Rectangle 클래스의 setWidth() 메서드와 setHeight() 메서드를 재정의해서 가로와 세로 값이 일치하도록 구현하였다.

```java
public class Square extends Rectangle {
  @Override
  public void setWidth(int width) {
    super.setWidth(widht);
    super.setHeight(widht);

  }

  @Override
  public void setHeight(int height) {
    super.setWidth(height);
    super.setHeight(height);

  }

}
```

<br>

- Rectangle 클래스 타입을 인자로 받는 다음과 같은 메서드가 있다고 가정하자
- 높이와 폭을 비교해서 높이를 더 길게 만들어 주는 기능(여기서 메서드 실행 후 항상 height 값이 width 보다 커진다고 가정한다)
```java
public void increaseHeight(Rectangle rec) {
  if(rec.getHeight() <= rec.getWidth()) {
    rec.setHeight(rec.getWidth() + 10);
  }
}
```

- 여기서 위 메서드의 파라미터로 Square 객체가 전달되면, 오버라이딩 된 Square의 setHeight 메서드가 실행되고, 이 결과로 height 과 width의 값이 같아지게 된다
- Square 객체가 파라미터로 전달되는 경우에는 increaseHeight 메서드를 실행해도 높이 값이 폭 값보다 커지지 않는 것이다
<br>

- instanceof 연산자를 통해 Square 타입인 경우 걸러낼 수 있지만, instanceof 연산자를 사용한다는 것 자체가 리스코프 치환 원칙을 위반하는 것이다
- increaseHeight 메서드가 Rectangle 클래스의 확장에 열려 있지 않다는 것이다
<br>

:two:

상위 타입에서 지정한 리턴 값의 범위에 해당하지 않는 값을 리턴하는 경우 또한 리스코프 치환 원칙을 어기게 된다
<br>

```java
public class CopyUtil {
  public static void copy(InputStream is, OutputStream out) {
    byte[] data = new byte[512];
    int len = -1;

    // InputStream.read() 메서드는 스트림의 끝에 도달하면 -1을 리턴
    while((len = is.read(data)) != -1) {
      out.write(data, 0, len);

    }

  }

}
```
- InputStream의 read() 메서드는 스트림의 끝에 도달해서 더 이상 데이터를 읽어올 수 없는 경우 -1을 리턴한다고 정의되어 있고, CopyUtil.copy() 메서드는 이 규칙에 따라 is.read()의 리턴 값이 -1이 아닐 때까지 반복해서 데이터를 읽어와 out에 쓴다.
- 그러나 만약 InputStream을 상속한 하위 타입에서 read() 메서드를 아래와 같이 구현한다면 어떻게 될까?

```java
public class SatanInputStream implements InputStream {
  public int read(byte[] data) {
    ...
    return 0; // 데이터가 없을 때 0을 리턴하도록 구현

  }

}
```
- CopyUitl 객체 생성 시 위의 SatanInputStream 을 사용한다면, CopyUtil.copy() 메서드에서 is.read()의 리턴값이 -1이 되는 경우가 발생하지 않으므로 무한루프에 빠지게 된다
- 위와 같은 문제가 발생하는 이유는 SatanInputStream 타입의 객체가 상위 타입인 InputStream을 올바르게 대체하지 않기 때문이다. 
- 즉, 리스코프 치환 원칙을 지키지 않았기 때문에 발생한 것이다.

<br>

### < 리스코프 치환 원칙은 계약과 확장에 대한 것 >

기능 실행의 계약과 관련해서 흔히 발생하는 위반 사례로는 다음과 같은 것들이 있다.
- 명시된 명세에서 벗어난 값을 리턴한다.
- 명시된 명세에서 벗어난 익셉션을 발생한다.
- 명시된 명세에서 벗어난 기능을 수행한다.
하위 타입이 명세에서 벗어난 동작을 하게 되면, 이 명세에 기반해서 구현한 코드는 비정상적으로 동작할 수 있기 때문에, 하위 타입은 상위 타입에서 정의한 명세를 벗어나지 않는 범위에서 구현해야 한다.
<br>

#### 리스코프 치환 원칙을 어기면 개방 폐쇄 원칙을 어길 가능성이 높아진다.

```java
public class Coupon {
  public int calculateDiscountAmount(Item item) {
    return item.getPrice() * discountRate;
  }
}
```
- 상품에 쿠폰을 적용해서 할인되는 액수를 구해 주는 기능을 구현하기 위해 Coupon 클래스에서 Item 클래스의 값을 구한 뒤 할인되는 큼액을 계산할 수 있도록 구현을 했다
<br>

- 특수 Item은 무조건 할인을 해주지 않는 정책이 추가되어, 이를 위해 Item 클래스를 상속받는 SpecialItem 클래스를 추가했다고 하자.
- Coupon 클래스의 calculateDiscountAmount() 메서드는 item 객체의 실제 타입이 SpecialItem인 경우 할인 액수를 0으로 처리해 주어야 하는데, 이를 반영하기 위해 Coupon 클래스를 다음과 같이 수정할 수 있을 것이다.
```java
public class Coupon {
  public int calculateDiscountAmount(Item item) {
    if(item instanceof SpecialItem) { // LSP 위반 발생,  instanceof 사용
      return 0;
    }

    return item.getPrice() * discountRate;
  }
}
```
-  Item 타입을 사용하는 코드는 (이 예에서는 Coupon 클래스의 calculateDiscountAmount() 메서드는) SpecialItem 타입이 존재하는지 알 필요 없이 오직 Item 타입만 사용해야 한다.

- 타입을 확인하는 기능(예를 틀어, 자바의 instanceof 연산자)을 사용하는 것은 전형적인 리스코프 치환 원칙을 위반할 때 발생하는 증상이다. 클라이언트가(Coupon 클래스) instanceof 연산자를 사용한다는 것은 상위 타입(Item 클래스)만을 사용해서 프로그래밍 할 수 없다는 것을 의미한다.
- 이는 SpecialItem과 같은 새로운 종류의 하위 타입이 생길 때마다 상위 타입을 사용하는 코드를 수정해줘야 할 가능성이 높아지고, 결국 개방 폐쇄 원칙을 위반하게 된다.
- 리스코프 치환 원칙을 어기게 된 이유는 Item에 대한 추상화가 덜 되었기 때문이다. 할인되지 않는 상품 타입이 추가되었다는 것은 이후에 비슷한 요구가 발생할 수 있는 가능성이 높다는 것을 뜻한다. 

<br>

- 따라서, 상품의 가격 할인 가능 여부가 Item 및 하위 타입에서 변화되는 부분이 되며, 변화되는 부분을 Item 클래스에 추가함으로써 리스코프 치환 원칙을 지킬 수 있게 된다.
```java
public class Item {
  // 변화되는 기능을 상위 타입에 추가
  public boolean isDiscountAvailable() {
    return true;
  }

  ...

}

public class SpecialItem extends Item {
  // 하위 타입에서 알맞게 오버라이딩
  @Override
  public boolean isDiscountAvailable() {
    return false;
  }
}
```
<br>

- 변화되는 부분을 상위 타입에 추가함으로써, instanceof 연산자를 사용하던 코드를 Item 클래스으로만 구현할 수 있게 되었다.
```java
public class Coupon {
  public int calculateDiscountAmount(Item item) {
    if(! item.isDiscountAvailable()) {  // instanceof 연산자 사용 제거
      return 0;
    }

    return item.getPrice() * discountRate;
  }
}
```
- Coupon 클래스는 item이 구체적으로 어떤 타입의 아이템인지 알 필요가 없어졌다
- 위 예제를 통해 설계 단계에서 모든 추상화를 완벽히 할 수는 없고 개발 과정에서 객체 지향 원칙을 반복적으로 생각하며 추가적인 추상화를 해야 한다는 것을 느꼈다

***
<br>

## :four: 인터페이스 분리 원칙

#### - 인터페이스는 그 인터페이스를 사용하는 클라이언트를 기준으로 분리해야 한다

- 용도에 맞게 인터페이스를 분리하는 것은 단일 책임 원칙과도 연결된다
- 클라이언트 입장에서 사용하는 기능만 제공하도록 인터페이스를 분리함으로써 한 기능에 대한 변경의 여파를 최소화할 수 있게 된다
- 각 클라이언트가 사용하는 기능을 중심으로 인터페이스를 분리함으로써, 클라이언트로부터 발생하는 인터페이스 변경의 여파가 다른 클라이언트에 미치는 영향을 최소화할 수 있게 된다

***
<br>

## :five: 의존 역전 원칙

#### - 고수준 모듈은 저수준 모듈의 구현에 의존해서는 안 된다. 저수준 모듈이 고수준 모듈에서 정의한 추상 타입에 의존해야 한다

고수준 모듈: 어떤 의미 있는 단일 기능을 제공하는 모듈
저수준 모듈: 고수준 모듈의 기능을 구현하기 위해 필요한 하위 기능의 실제 구현

<br>

**의존 역전 원칙은 추상화를 이용해서 저수준 모듈이 고수준 모듈을 의존하게 만든다.**
앞서 파일 암호화 예에서 이미 봤듯이, 최초에 FlowController 객체는 FileDataReader 객체에 직접적으로 의존하고 있었으나, ByteSource 추상 타입을 도출함으로써 FlowControlelr와 FileDataReader가 모두 추상 타입인 ByteSource에 의존하도록 만들었다.

![image](https://user-images.githubusercontent.com/69443895/154813654-4d9e2ea2-82ad-4074-ba8c-007a08421afc.png)

고수준 모듈인 FlowController와 저수준 모듈인 FileDataReader가 모두 추상화 타입인 ByteSource에 의존함으로써, 고수준 모듈의 변경 없이 저수준 모듈을 변경할 수 있는 유연함을 얻게 되었다. 
즉, 의존 역전 원칙은 리스코프 치환 원칙과 개방 폐쇄 원칙의 기반이 되는 원칙이다.

***
<br>

## < SOLID 정리 >

- 단일 책임 원칙과 인터페이스 분리 원칙은 객체가 커지지 않도록 막아준다. 
- 리스코프 치환 원칙과 의존 역전 원칙은 개방 폐쇄 원칙을 지원한다.
  + 개방 폐쇄 원칙은 변화되는 부분을 추상화하고 다형성을 이용함으로써 기능 확장을 하면서도 기존 코드를 수정하지 않도록 만들어 준다.
여기서, 변화되는 부분을 추상화할 수 있도록 도와주는 원칙이 바로 의존 역전 원칙이고, 다형성을 도와주는 원칙이 리스코프 치환 원칙인 것이다.



