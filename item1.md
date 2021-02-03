
> https://johngrib.github.io/wiki/static-factory-method-pattern/

# 생성자 대신 정적 팩터리 메서드를 고려하라
## 정적 팩터리 메서드의 장점은?
- 이름이 있다. (of, valueOf, getType 등)
    - 이름이 있기 때문에 하나의 시그니처로 다양한 인스턴스 생성이 가능하다.
    ```java
    public static StaticClass of(String a){
        return new StaticClass(a, 10);
    }

    public static StaticClass valueOf(String a){
        return new StaticClass(a, 20);
    }
    ```

- 호출될때마다 인스턴스를 새로 생성하지 않아도 된다.
    - 불변클래스는 인스턴스를 미리 만들거나 새로 생성한 인스턴스를 캐싱/재활용
    - 불변클래스란 그 인스턴스의 내부 값을 수정할 수 없는 클래스. (String, BigInteger 등등)
    - 플라이웨이트 패턴
        - 객체의 내부에서 참조하는 객체를 직접 만드는 것이 아니라 없다면 만들고 있다면 공유.
        - 대부분 팩토리 메서드 패턴을 사용해 객체 생성
        - 자바의 String pool은 대표적인 flyweight pattern
        - 아래 두 클래스의 getStringA()와 getStringB()는 모두 같은 위치에 있는 "Hello" String을 리턴.(Permanent 영역)
        ```java
        public class A {
            public String getStringA() {
                return "Hello";
            }
        }
        public class B {
            public String getStringB() {
                return "Hello";
            }
        }
        ```
    - 반복되는 요청에서 같은 객체를 반환하기 때문에 인스턴스 통제가 가능하다.
        - 싱글턴으로 만들수도, 인스턴스 불가로도 만들 수 있다.
- 반환 타입의 하위 타입 객체를 반환할 수 있다. (ITEM20)
    ```java
    class OrderUtil {
        public static Discount createDiscountItem(String discountCode) throws Exception {
            if(!isValidCode(discountCode)) {
                throw new Exception("잘못된 할인 코드");
            }
            // 쿠폰 코드인가? 포인트 코드인가?
            if(isUsableCoupon(discountCode)) {
                return new Coupon(1000);
            } else if(isUsablePoint(discountCode)) {
                return new Point(500);
            }
            throw new Exception("이미 사용한 코드");
            }
    }
    class Coupon extends Discount { }
    class Point extends Discount { }
    ```
    - ITEM64, 객체는 인터페이스를 사용해 참조.
    ```java
    // 좋은 예
    Set<Son> sonset = new LinkedHashSet<>();
    // 나쁜 예
    LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
    ```
        - 인터페이스를 타입으로 사용하는 습관을 길러야 프로그램이 더 유연해진 수 있기 때문.

- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    ```java
    public abstract class EnumSet { //EnumSet은 abstract하여 객체 생성이 불가능하다.
    //noneOf 메소드에서 상황에 따라 다른 구현체 객체들을 만들어서 반환해주고 있다.
        public static noneOf(...) {
            if (enumElementSize <= 64)
                return new RegularEnumSet<>(...);  
            else
                return new JumboEnumSet<>(...);
        }
    }
    ```
    - 여기서 클라이언트는 팩터리가 건네주는 객체가 RegularEnumSet인지, JumboEnumSet인지 알 필요가 없다.
- 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    ```java
    String driverName = "com.mysql.jdbc.Driver";
    String url = "jdbc:mysql://localhost:3306/test";
    String user = "root";
    String password = "root";

    try {
        Class.forName(driverName);
        Connection connection = DriverManager.getConnection(url, user, password);
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    }
    ```
    - 책에서 설명하는 제공자 등록 API 역할을 맡은, DriverManager.registerDriver()도 없이 Driver 설정작업이 일어났고, getConnection으로 Connection 객체를 호출하게 된다. 
    - Class.forName(...)은 파라미터로 받은 driverName에 해당하는 클래스를 로딩하고, 클래스가 로드 될 때, static 필드의 내용이 실행되는 것을 이용해 자기자신을 DriverManager 클래스에 등록하게 된다.
    - 다시 말해 class.forName에 의해 Driver를 로드하고, DriverManager.registerDriver()를 호출하여 인스턴스를 생성한다. (Driver클래스 안에 registerDriver가 있다.)
    ```java
    // com.mysql.jdbc.Driver
    public class Driver extends NonRegisteringDriver implements java.sql.Driver {
        static {
            try {
            java.sql.DriverManager.registerDriver(new Driver());
            } catch (SQLException E) {
                throw new RuntimeException("Can't register driver!");
            }
        }
    }
## 그렇다면 단점은? 
- 상속을 하려면 public이나 protected 생성자가 필요한데, 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
    - 상속보다는 컴포지션과 불변타입을 이용하자. 자세한건 나중에 공부하자.
- 프로그래머가 찾기 어렵다.
    - 갓-바독을 이용하자.