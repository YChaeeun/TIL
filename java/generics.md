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



## 제한된 제네릭 클래스

* extends 를 사용하면 해당 타입의 자식들만 대입할 수 있도록 타입을 제한할 수 있다

  ```kotlin
  class FruitBox<T extends Fruit> { // Fruit 의 자식 타입만 대입 가능
      ArrayList<T> list = new ArrayList<T>();
  }

  // 인터페이스로 제약할 때도 extends (implements 안씀)
  interface Edible { }
  class FruitBox<T extends Edible> { }

  // 동시에 제약을 해야 한다면 & 사용
  class FruitBox<T extends Fruit & Edible> { }
  ```



## 와일드 카드 ?

* static 메서드에서는 제네릭 타입을 쓸 수 없으므로, 만약 Fruit 타입으로 대입한 FruitBox 를 인자로 받게 한 경우, 해당 메서드는 무조건 Fruit 타입한테만 쓸 수 있다 \(하위 타입인 Apple, Grape에서는 못써\)

  * 그런데, 그렇다고 Fruit 타입, Apple 타입, Grape 타입 이렇게 타입만 다른 메소드를 오버로딩 하게 할 수도 없다ㅠ --&gt; 제네릭 타입이 다른 것 만으로는 오버로딩이 성립하지 않음

  ```java
  class Juicer {
      static Juice makeJuice(FruitBox<Fruit> box) { // Fruit 로 타입을 대입
      }
    
      // 그렇다고 오버로딩 하는 건 안됨
      // 다른 메서드로 인정이 안됨,,,, 다형성 성립 X
      static Juice makeJuice(FruitBox<Apple> box) { }
      static Juice makeJuice(FruitBox<Grape> box) { }
  }

  Juicer.makeJuice(fruitBox); // Fruit 타입일 때만 사용 가능
  Juicer.makeJuice(Apple); // (X) Apple 이 Fruit 의 자식이어도 사용 불가
  ```

* 이럴 경우 와일드 카드 ? 를 쓴다~!

  * 와일드 카드는 어떠한 타입도 될 수 있음
  * extends, super 로 상하한을 제한할 수 있음
    * `<? extends T>` : 상한 제한, T와 그 자식들만 가능
    * `<? super T>` : 하한 제한, T 와 그 조상들만 가능
    * `<?>` : 제한 없음, 모든 타입 가능 &lt;? extends Object&gt; 랑 같음

  ```java
  class Juicer {
      static Juice makeJuice(FruitBox<? extends Fruit> box) {
                                  // Fruit와 그 자식들 가능
      }    
  }
  ```

* 와일드 카드 super 의 예
  * Comparator&lt;? super Apple&gt; 가능한 타입
    * Comparator&lt;Apple&gt;, Comparator&lt;Fruit&gt;, Comparator&lt;Object&gt;
  * Comparator&lt;? super Grape&gt;
    * Comparator&lt;Grape&gt;, Comparator&lt;Fruit&gt;, Comparator&lt;Object&gt;



## 제네릭 메서드

* 메서드 선언부에 제네릭 타입이 선언된 메서드를 제네릭 메서드라고 함
  * ```java
    class FruitBox<T> { // 얘랑 
        static <T> void sort(List<T> list, Comparator<? super T> c { // 얘 T는 다른거
        }
    }

    // 앞에 makeJuice 를 바꾸면 
    static <T extends Fruit> Juice makeJuice(FruitBox<T> box) {
    }  

    Juicer.<Fruit>makeJuice(fruitBox);
    Juicer.makeJuice(fruitBox); // 타입 추론이 가능할 경우 타입 생략 가능
    ```

    * 이때 제네릭 클래스에 정의된 타입 매개변수랑\(FruitBox의 T\), 제네릭 메서드에 정의된 타입 매개변수 &lt;T&gt; void sort\(T\)는 전혀 다른 것!!
      * 같은 타입 문자 T 를 썼지만 다른거
  * static 멤버에는 타입 매개변수를 사용할 수 없지만, 메서드에 제네릭 타입을 선언하고 사용하는 것은 가능하다!
    * 왜냐면 이건 지역 변수를 선언한 것과 같기 때문
  * 코드를 간략하게 만들 수도 있음

    ```java
    public static void printAll(
            ArrayList<? extends Product> list, 
            ArrayList<? extends Product list2) {}

    // 간략하게 하면
    public static <T extends Product> void printAll( // 타입 중복된 부분을 뽑아줌
            ArrayList<T> list, 
            ArrayList<T> list2){}
        
    /* ----------------------------- */
    public static <T extends Comparable<? super T>> void sort(List<T> list)
            // T extends Comparable : T는 Comparable를 구현한 클래스여야 하고
            // Comparable<? super T> : T 또는 그 조상을 비교하는 Comparable

    // 간략하게 하면
    public static <T extends Comparable<T>> void sort(List<T> list)
    ```

## 제네릭 타입의 형변환

* 불가능
  * 대입 타입이 다른 제네릭 타입 \(Object 여도 불가능\)
* 가능
  * 제네릭 타입 --&gt; 원시 타입 / 원시타입 --&gt; 제네릭 타입 \(경고 발생\)
  * 와일드 카드 ?
    * &lt;String&gt; --&gt; &lt;? extends Object&gt;
    * &lt;? extends Object&gt; --&gt; &lt;String&gt; \(미확인 타입으로 형변환 경고 발생\)
    * &lt;? extends Object&gt; --&gt; &lt;? extends String&gt; / &lt;? extends String&gt; --&gt; &lt;? extends Object&gt; \(미확정 타입으로 형변환 경고\)

```java
// 불가능
Box<Object> objBox = null;
Box<String> strBox = (Box<String>)objBox; // (X) 안됨!

Box<Object> objBox = new Box<String>(); // (X) 안됨!!!


// 가능 - 제네릭 -> 원시 / 원시 -> 제네릭
Box box            = null;
Box<Object> objBox = null;

box    = (Box)objBox;        // 경고 발생
objBox = (Box<Object>)box;   // 경고 발생

// 가능 - 와일드 카드
Box<? extends Object> wBox = new Box<String>();

FruitBox<? extends Fruit> box = null;
FruitBox<Apple> appleBox = (FruitBox<Apple>)box; // 미확인 타입으로 형변환 경고

FruitBox<? extends Object> objBox = null;
FruitBox<? extends String> strBox = (FruitBox<? extends String>)objBox; // 미확정 타입으로 형변환 경고


```

## 제네릭 타입의 제거

* 컴파일러는 제네릭 타입을 이용해서 소스파일을 체크, 필요한 곳에 형변환을 넣어준 뒤 제네릭 타입을 제거함
  * 즉, 컴파일 된 파일\(\*.class\) 에는 제네릭 타입에 대한 정보가 없음
  * 제네릭 도입 이전의 소스코드와의 호환성을 유지하기 위해 도입되었음
    * 앞으로 가능하면 원시 타입을 쓰지 말자\(????\)
* 기본적인 제거 과정
  1. 제네릭 타입의 경계 bound 를 제거
     * &lt;T extends Fruit&gt; 라면 T는 Fruit 로 치환됨
     * &lt;T&gt; 인 경우 T는 Object 로 치환됨
     * class Box&lt;T extends Fruit&gt; { } 의 경우 클래스 옆 선언 제거 --&gt; class Box { }
  2. 제네릭 타입 제거 후 타입이 일치하지 않으면 형변환 추가
     * **T** get\(int i\) { } --&gt; **Fruit** get\(int i\) { }
     * 와일드 카드가 포함되어 있는 경우에도 적절한 타입으로의 형변환이 추가됨

