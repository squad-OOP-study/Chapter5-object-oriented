# chap5 설계원칙 : SOLID

- 2022.2.19

> 목차

- Single Responsibility priciple
- Open-closed principle


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
   <img src = ".https://user-images.githubusercontent.com/55780251/154810761-2f6612f0-e412-4e9f-9ad2-2ee5090cbde7.jpg" width="60%">
   
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
  <img src="https://user-images.githubusercontent.com/55780251/154810815-97567c6b-3a55-474b-ae45-0120ed5e408d.jpg" width="60%">
 
2. 상속 
   - 상위 클래스의 기능 그대로 사용하면서 하위 클래스에서 일부 구현을 오버라이딩 할 수 있는 방법을 제공한다.
   - protected 공개범위를 갖고 있기 때문에 하위 클래스에서 해당 메서드를 overriding할 수 있다.
   - 접근제어자 
     - 참고자료: http://www.tcpschool.com/java/java_modifier_accessModifier



