# 제네릭스 Generics

## 제네릭스

* 다양한 타입의 객체들을 다루는 메서드나 컬렉션 클래스를 컴파일 할 때 타입 체크\(compile-time type check\) 를 해주는 기능
  * 객체의 타입을 컴파일 시에 체크하므로 객체 타입 안정성을 높이고 타입 형변환의 번거로움을 줄일 수 있음
* 타입 안정성이 높다는 말은
  * 의도하지 않은 타입의 객체가 저장되는 것을 막음
  * 저장된 객체를 꺼내올 때, 원래 타입과 다른 타입으로 잘못 형변환 되지 않도록 해줌

### 제네릭 클래스 선언

* 타입 변수 type variable
  * 임의의 참조형 타입을 의미
    * T 나 E, &lt;K,V&gt; 처럼 의미있는 문자를 주로 사용
  * 기존에는 다양한 타입을 다뤄야 하는 경우, 최상위 타입인 Object 를 반환하고 형변환해서 사용했지만, 제네릭을 사용하면 원하는 타입으로 사용 가능

```java
class Box<T> { // 타입 변수 T
    T item;
    void setItem(T item) { this.item = item; }
    T getItem() { return item; }
}

Box<String> box = new Box<String>(); 
    // 객체를 생성할 때 타입 변수 자리에 원하는 타입을 넣어줌
box.setItem(123) // 불가 String 타입만 넣을 수 있음
box.setItem("ABC") 

String item = box.getItem(); // 이때 별도의 형변환 필요 없음
            // 만약 제네릭 대신, Object 를 반환하는 거였다면 (String) 형변환 해줘야 함
```

### 용어

* `class Box<T> { }`
  * Box&lt;T&gt;  : 제네릭 클래스 "T의 Box" 혹은 "T Box" 라고 읽음
  * T : 타입 변수 또는 타입 매개변수 \(T 는 타입 문자\)
  * Box : 원시 타입 \(raw type\)
    * 컴파일 한 후에는 원시 타입만 남는다
* Box&lt;String&gt; box = new Box&lt;String&gt;\(\)
  * Box&lt;String&gt; : 제네릭 타입 호출
  * String : 지정된 타입, 매개변수화된 타입 parameterized type, 대입된 타입

### 제한

* static 멤버에 타입 변수 T를 사용할 수 없다
  * static 멤버는 동일한 것이어야 하므로, 타입 변수에 대입된 타입에 따라 다른 타입의 객체가 될 수 있다는 건 말도 안됨~!
  * static 멤버는 인스턴스 변수를 참조할 수 없는데, 타입변수 T는 인스턴스 변수로 간주 됨
* 제네릭 타입의 배열을 생성할 수 없다 new \(X\)
  * 제네릭 배열 타입의 참조 변수를 선언하는 것은 가능하지만 `T[] itemArr;`
  * new 연산자로 생성하는 것은 불가능 `T[] temp = new T[5];` \(X\)
    * new 연산자는 컴파일 시점에 타입 T 가 뭔지를 정확하게 알아야 하는데, 제네릭 타입의 경우 컴파일 하는 시점에 T 가 어떤 타입이 될 지 전혀 알수가 없기 때문
      * 같은 이유로 instanceof 도 T 를 피연산자로 사용 불가
  * 꼭 배열을 선언해야 한다면 
    * new 연산자 대신 ReflectionsAPI 의 newInstance\(\) 와 같이 동적으로 객체를 생성하는 메서드로 배열을 생성하거나,
    * 혹은 Object 배열을 생성해서 복사한 다음 T\[\] 로 형변환

## 제네릭 클래스의 객체 생성과 사용

```java
class Box<T> {
    ArrayList<T> list = new ArrayList<T>();
    
    void add(T item) { list.add(item); };
    T get(int i) { return list.get(i); };
    ArrayList<T> getList() { return list; };
    int size() { return list.size(); };
    public String toString() { return list.toString(); };
}


// 객체 생성
Box<Apple> appleBox = new Box<Apple>(); // 참조변수와 생성자에 대입된 타입이 같아야 함

Box<Fruit> fruitBox = new Box<Apple>(); // (X) Fruit 가 Apple 상위 클래스라도 안돼
Box<Apple> appleBox = new FruitBox<Apple>(); // (O) 제네릭 클래스 간의 상속관계가 있으면 이건 돼

Box<Fruit> fruitBox = new FruitBox();
fruitBox.add(new Fruit()); // 가능
fruitBox.add(new Apple()); // 요것도 가능~! Fruit item 을 넣을 수 있는데, Apple 은 Fruit의 자식
```





