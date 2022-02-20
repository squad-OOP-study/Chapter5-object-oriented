# chap5 설계원칙 : SOLID

- 2022.2.19

> 목차

- Single Responsibility priciple
- Open-closed principle
- Liskov substitution principle 리스코드 치환 원칙 (LSP)
- Interface segregation principle(ISP)
- Dependency inversion principle (DI)



___

## Single Responsibility priciple

> 정의
- 클래스는 단 한 개의 책임을 가져야 한다.
- 클래스를 변경하는 이유는 단 한개여야 한다.

> 특징
- 다른 원칙들에게 영향을 주기 때문에 최대한 잘 지켜야 하는 원칙
- 한 개의 책임에 대한 정의가 명확하지 않고, 책임 도출은 다양한 경험이 필요하기 때문에 어렵다.

> 단일 원칙 위반으로 인한 문제점
- 문제점을 통해 단일 책임에 대한 감을 잡고 단일 책임의 의미를 생각해보기.

1. 하나의 기능 변화로 클래스 내부에 **변화가 연쇄적**으로 발생
    - 한 책임의 변화가 다른 책임의 코드에 영향을 준다.
    - 예) DataView Class : "데이터 읽는 책임" 변화가 "화면에 보여주는 책임"까지 영향
    - loadHtml() 의 return Value가 char[]로 변경되면 display()와 관련 메서드까지 영향을 받는다.
    ```java
    public class DataViewer {
        //화면에 보여주는 책임
        public void display() {
            String data = loadHtml();
            updateGui(data);
        }

        //데이터 읽는 책임
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
            //파싱 처리 코드
        }
    }
    ```

    - [해결 방법]   
        아래 방법으로 데이터 읽어 오는 부분의 변경 때문에 화면을 보여주는 부분의 코드가 변경되는 상황을 막을 수 있다.
      - 두 개의 책임을 각각 클래스로 분리 
      - 두 클래스 간 주고 받을 데이터를 저수준의 String이 아닌 **알맞게 추상화된 타입을 사용**
        ```java
        public class DataViewer {
            //화면에 보여주는 책임
            public void display(DataLoader dataLoader) {
                Data data = dataLoader.load();
                updateGui(data);
            }

            private void updateGui(Data data) {
                GuiData guiModel = parseDataToGuiData(data);
                tableUI.changeData(guiModel);
            }

            private GuiData parseDataToGuiData(Data data) {
                //파싱 처리 코드
            }
        }

        public class DataLoader {
            //데이터 읽는 책임
            public Data load() {
                HttpClient client = new HttpClient();
                client.connect(url);
                return client.getResponse();
            }                
        }

        // [Q] 클래스 간 주고 받을 데이터 타입 : best practice 궁금하다.
        public class Data {
            private String data;
            public Data(char[] data){
                this.data = String.valueOf(data);
            }

            public Data(String data){
                this.data = data;
            }
            public String getData(){ }
        }
        ```

2. 재사용을 어렵게 한다.
   - 책임이 분리되어 있지 않으면 필요하지 않는 패키지까지 필요하다. 
   - 예시 클래스DataViewer에서 "데이터를 읽어오는" DataRequiredClient 클래스를 만들어야 한다면    
     필요한 것은 DataViewer 클래스와 HttpClient.jar 파일이다.
   - 그러나 **실제 사용하지 않는 기능이 의존하는** GuiComp.jar까지 필요하게 된다.
   
   <img src = "https://user-images.githubusercontent.com/55780251/154810761-2f6612f0-e412-4e9f-9ad2-2ee5090cbde7.jpg" width="60%">
   
    <br>

   - [해결방법]     
     - 데이터를 읽어 오는 데 필요한 dataloader 패키지와 HttpClient 패키지만 필요하며
     - 데이터를 읽어 오는 것과 상관없는 GuiComp 패키지나 datadisplay패키지는 포함시킬 필요가 없어진다. 
     <img src = "https://user-images.githubusercontent.com/55780251/154810792-5465af6e-f6ad-49fb-886d-3a79aa73fa30.jpg" width="60%">


> 책임이란 변화에 대한 것
1. 기능 변경 요구가 없을 때 수정에 대한 문제가 없다는 것의 의미는,   
   **책임의 단위는 변화되는 부분과 관련**된다는 의미 
   - 예시) DataViewer클래스에서 데이터를 읽어 오는 기능에 변화 발생하면, 데이터 읽어 오는 기능이 별도로 분리되어야 할 책임이라는 것을 알 수 있는 것이다.
  
2. 책임은 서로 다른 이유로 변경되고, 서로 다른 비율로 변경된다.
   - 서로 다른 이유로 바뀌는 책임들이 한 클래스에 포함되어 있다면 해당 클래스는 SRP를 어기고 있다고 볼 수 있다.
   - 예) 데이터를 읽어 오는 책임의 기능이 변경될 때 데이터를 보여주는 책임은 변경되지 않는다.
   - 서로 다른 이유로 변경되는 것을 알아채려면 경험이 필요.

> 단일 책임 원칙 잘 지키기 위한 방법
- 메서드를 실행하는 것이 누구인지 확인해 보는 것!
- 클래스의 사용자들이 서로 다른 메서드를 사용한다면, 각각 다른 책임에 속할 가능성이 높고 책임 분리 후보가 될 수 있다.
<img src ="https://user-images.githubusercontent.com/55780251/154810809-6f25fd29-c86a-4f8d-a7d6-6ed3d9ccfbc7.jpg" width="60%">

___

## Open-closed principle (OCP)
- 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.
- 기능을 변경하거나 확장할 수 있으면서, 그 기능을 사용하는 코드는 수정하지 않는다.


> OCP 구현 방법
1. 확장되는 부분(변화되는 부분)을 interface로 추상화해서 표현
   - 사용자 입장에서 변화 고정시킨다.
 
   - <img src="https://user-images.githubusercontent.com/55780251/154810815-97567c6b-3a55-474b-ae45-0120ed5e408d.jpg" width="60%">
 
2. 변화되는 부분을 상속의 protected 제어자로 고정
   - 상위 클래스의 기능 그대로 사용하면서 하위 클래스에서 일부 구현을 오버라이딩 할 수 있는 방법을 제공한다.
   - [Q] public으로 되어 있는 부분도 overriding가능한데.. 왜 고정되었다고 했을까?
   - protected 공개범위를 갖고 있기 때문에 하위 클래스에서 해당 메서드를 overriding할 수 있다.
     - protected 멤버는 같은 패키지에 속하는 클래스와 다른 패키지에 속하는 자식 클래스에서만 접근할 수 있다.
     - 접근제어자 : http://www.tcpschool.com/java/java_modifier_accessModifier

    ```java
    public class ResponseSender {
        private Data data;
        public ResponseSender(Data data) {
            this.data = data;
        }

        public Data getData() {
            return data;
        }

        public void send() {
            sendHeader();
            sendBody();
        }

        protected void sendHeader() {
            //헤더 데이터 전송
        }

        protected void sendBody() {
            //바디 데이터 전송
        }
    }
    ```
    [GoF디자인 패턴의 템플릿 패턴] 
    - 템플릿 메서드 패턴 : 상위 클래스에서 실행할 기본 코드를 만들고 하위 클래스에서 필요에 따라 확장해 나가는 패턴. 

<br>

- 예시) 압축해서 데이터를 전송하는 기능을 추가하기 위해 상속받은 클래스에서 setHeader(), setBody()를 overriding!
- 압축 기능을 확장하면서도 기존 ResponseSender클래스의 코드는 바뀌지 않았다.
- ReponseSender 클래스는 확장에는 열려 있으면서 변경에는 닫혀 있는 것.
    ```java
    public class ZippedResponseSender extends ResponseSender{

        public ZippedResponseSender(Data data) {
            super(data);
        }

        @Override
        protected void sendHeader() {
            super.sendHeader();
        }

        @Override
        protected void sendBody() {
            super.sendBody();
        }
    }
    ```

> 개방 폐쇄 원칙이 깨질 때 증상

1. 다운 캐스팅 한다.
    - instanceof 와 같은 타입 확인 연산자가 사용된다면 해당 코드는 OCP 원칙을 지키지 않을 가능성이 높다.
    - 타입 캐스팅 후 실행하는 **메서드가 변화 대상인지 확인**해 봐야 한다.
    - 예) drawSpecific() 메서드가 객체마다 다르게 동작할 수 있는 변화 대상인지 확인해 보는 것. 
    - 객체마다 다르게 동작한다면 추상화해서 Character 타입에 추가해주어야 한다.
2. 비슷한 if-else 블럭이 존재한다.   
   - 예) Enemy 클래스: **변경에 닫혀 있지 않은 클래스 예시**
   - Enemy 캐릭터의 움직이는 경로를 몇 가지 패턴으로 정한다고 하자.      
   - 정해진 패턴에 따라 경로를 이동하는 코드 작성.
   - 새로운 경로 패턴을 추가할 경우 if 블록이 추가된다.  
    
    <br>

    [해결 방법]
    - 경로 패턴을 추상화하고 Enemy에서 추상화 타입을 사용하는 구조로 바꾼다.
    - 새로운 이동 패턴이 생기면 새로운 타입의 PathPettern 구현 클래스를 추가해 주면 된다.
    - Enemy의 draw() 메서드는 변경되지 않는다.
    - <img src="./chap5/OCP_패턴추상화.jpg" width="60%">

> 개방 폐쇄 원칙은 유연함에 대한 것

- 변경의 유연함과 관련된 것. 
- 기존 기능 확장하려고 기존 코드를 수정해 주어야 한다면 새로운 기능 추가하는 것은 점점 힘들어진다. 
- 변화되는 부분을 추상화 함으로 사용자 입장에서 변화를 고정시킨다.
- 코드에 대한 변화 요구가 발생하면 변화와 관련된 구현을 추상화해서 OCP에 맞게 수정할 수 있는지 확인하는 습관 갖자.
  - 스텝을 진행하면서 느껴보자!


## Liskov substitution principle 리스코드 치환 원칙 (LSP)
- 상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.

> LSP 지키지 않을 때 문제

1. 개념적으로 상속 관계에 있는 것처럼 보여도 실제 구현에서는 상속 관계가 아닐 수도 있다.
   - LSP의 대표적 예 : 직사각형-정사각형 
   - 실제 프로그램에서는 사격형과 정사각형을 상속으로 묶을 수 없는 것이다. 
   - 별개의 타입으로 구현해 주어야 한다.


    ```java
    public class Square extends Rectangle {

        @Override
        public void setWidth(int width) {
            super.setWidth(width);
            super.setHeight(width);
        }

        @Override
        public void setHeight(int height) {
            super.setWidth(height);
            super.setHeight(height);
        }
    }

    public class RectangleService {
        public void increseHeight(Rectangle rec) {
            if( rec instanceof Square){
            throw new CantSupportSquareException();
            }
            if(rec.getHeight() <= rec.getWidth()) {
            rec.setHeight(rec.getWidth() +10);
            }
        }
    }

    ```
   - Square 클래스가 Rectangle 클래스를 상속받도록 구현 했을 때 setWidth(), setHeight 재정의해서 가로=세로 일치하게 구현.
   - 그러나 Rectangle 클래스를 사용하는 코드에서 높이와 폭을 비교해서 높이를 더 길게 만들어주는 기능을 제공한다고 했을 때, 
   - Square 객체가 전달되면 이 가정은 깨진다. 
   - 이걸 방지하기 위해 instanceof를 사용하면 리스코프원칙 위반과 함께 Rectangle이 확장에 열려 있지 않음을 의미.

2. 상위 타입에서 지정한 리턴 값의 범위에 해당되지 않는 값을 리턴
   - 하위 타입이 상위 타입을 올바르게 대체하지 않을 때. 

> LSP는 계약과 확장에 대한 것

1. 계약에 관한 것 
    - 기능 실행 계약 위반 사례
      - 명시된 명세에서 벗어난 값 리턴
      - 명시된 명세에서 벗어난 익셉션 발생
      - 명시된 명세에서 벗어난 기능 수행
2. 확장에 관한 것 
    - 예) Coupon 클래스 : Item 클래스의 값을 구한 뒤 할인 되는 금액 계산
    - SpecialItem 클래스 :  할인이 안되는 아이템. 
    - instanceof를 사용해서 Item 구현 클래스별 구분해 구현 하면 LSP위반 발생. 
    - 즉, **하위타입 SpecialItem가 상위타입 Item을 완벽하게 대체하지 못하는 상황**
    - [문제 원인]
      - Item에 대한 추상화가 덜 되었기 때문에 
    - [해결 방안]
      - 상품의 가격 할인 여부 : 변화되는 부분
      - 변화되는 부분을 Item클래스에 추가
    ```java
    public class Item {
        //변화되는 기능을 상위 타입에 추가
        public boolean isDiscountAvailable() {
            return true;
        }
        ...
    }

    public class SpecialItem extends Item {
        //하위 타입에서 알맞게 오버라이딩
        @Override
        public boolean isDiscountAvailable() {
            return false;
        }
    }

    public class Coupon {
        public int calculateDiscountAmount(Item item) {
            if(!item.isDiscountAvailable()) { //instanceof 연산자 제거
            return 0;
            }
            return itm.getPrice() * discountRate;
        }
    }
    ```
___

## Interface segregation principle(ISP)

- 인터페이스는 그 인터페이스를 사용하는 클라이언트를 기준으로 분리해야 한다.
- 클라이언트는 자신이 사용하는 메서드에만 의존해야 한다.

- 자신이 사용하는 메서드에만 의존해야 한다는 원칙 
- 각 클라이언트가 필요로 하는 인터페이스들로 분리함으로써, 사용하지 않은 인터페이스에 변경이 발생하더라도 영향 받지 않아야 한다.
- 기능 변경의 여파를 최소화한다.
- 그럼으로써 인터페이스와 콘크리트 클래스의 재사용성을 높여 주는 효과

> 인터페이스 분리 원칙은 클라이언트에 대한 것 

- 클라이언트 입장에서 인터페이스를 분리하라! 
- 그럼으로서 다른 클라이언트에 미치는 영향 최소화한다.
- 예) 게시글 목록 UI요구로 인해 ArticleService 인터페이스가 변경될 수 있다. 


## Dependency inversion principle (DI)

- 고수준 모듈은 저수준 모듈의 구현에 의존해서는 안된다.   
  저수준 모듈이 고수준 모듈에서 정의한 추상 타입에 의존해야 한다.

- 고수준 모듈 : 상대적으로 큰 틀에서 프로그램 다룬다.
- 저수준 모듈 : 각 개별 요소(상세)가 어떻게 구현될지에 대해서 다룬다.

> 고수준 모듈이 저수준 모듈에 의존할 때의 문제 

- 예) 쿠폰, 아이템
- 쿠폰을 이용한 가격 계산 모듈이 개별적인 쿠폰 구현에 의존하게 되면, 새로운 쿠폰이 추가되거나 변경되면 가격 계산 모듈이 변경된다. 
- 쿠폰을 이용한 가격 계산 모듈 : 고수준 모듈
- 개별적인 쿠폰 구현 : 저수준 모듈

> 해결책 : 추상화

- 저수준 모듈이 고수준 모듈을 의존하게 만든다 : 추상화를 통해 
- 예) FlowController, ByteSource 인터페이스 , FileDataReader
- 소스코드의 의존을 역전시킴으로써 변경의 유연함 확보. 
- 런타임에서의 의존을 역전시키는 것은 아니다.

