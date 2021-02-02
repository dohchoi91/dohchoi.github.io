---
title: "1. 객체 생성과 파괴"
subtitle: "아이템1 ~ 아이템9"
permalink: /notes/effective-java/ch01
author_profile: true
date: 2021-02-03 00:15
categories:
  - 이펙티브 자바
tags:
  - java
last_modified_at: 2021-02-02
---
## 아이템1. 생성자 대신 팩터리 메서드를 고려하라.

### 정적 팩터리 메서드가 생성자보다 좋은 장점

1. 이름을 가질 수 있다.
2. 호출 될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
3. 반환 타입의 하위 타입 객체를 **반환**할 수 있는 능력이 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환 할 수 있다.
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.  
ex) jdbc

### 정적 팩터리 메서드가 생성자보다 좋지 않은 단점

1. 상속을 하려면 public, protected 생성자가 필요한데, 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
⇒ 상속보다 컴포지션을 사용을 유도하게 됨으로써 장점이 될  수 있다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
⇒ 생성자 처럼 API 설명에 드러나지 않기 때문에 이름을 잘 지어서 이러한 문제를 완화 시킨다.
    - 정적팩토리 메서드에 흔한 명명 방식
        - `from` : 매개 변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형 변환 메서드
        ex) Date d = Date.from(instance)
        - `of` : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
        ex) Set<Rank> faceCard = EnumSet.of(JACK, QUEEN, KING);
        - `valueOf` : from과 of의 더 자세한 버전
        ex) BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
        - `instance` 혹은 `getInstance` : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환 하지만, 같은 인스턴스임을 보장하지 않는다.
        ex) StackWalker luke = StackWalker.getInstance(options);
        - `create` 혹은 `newInstance` : `instance` 혹은 `getInstance` 와 같지만, 매번 새로운인스턴스를 생성해 반환함을 보장한다.
        ex) Object new Array = Array.newInstance(classObject, arrayLen);
        - `getType` :  `getInstance`와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.  ("Type"은 팩터리 메서드가 반환할 객체의 타입이다.)
        ex) FileStore fs = Files.getFileStore(path)
        - `newType` : `newInstance`와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의 할 때 쓴다. ("Type"은 팩터리 메서드가 반환할 객체의 타입이다.)
        ex) BufferdREader br = Files.newBufferedReader(path)
        - `type` : `getType` , `newType` 의 간결한 버전
        ex) List<Complaint> litany = Collections.list(legacyLitany);

정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 그렇다고 하더라도 **정적팩터리를 사용하는게 유리한 경우 더 많으므로 무작정 pulbic 생성자를 제공하던 습관이 있다면 고치자.**

## 아이템2. 생성자에 매개변수가 많다면 빌더를 고려하라

정적 팩터리와 생성자에는 선택적 매개변수가 많을 경우 대응하기가 어렵다는 제약이 하나 있다.

### 대안1. 생성자 패턴

생성자를 여러개 만들어 사용.

⇒ 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.

### 대안2. 자바빈즈 패턴

매개변수가 없는 생성자를 만두 후, 세터를 호출하여 값을 설정한다.

⇒ 객체 하나를 만들려면 메서드를 여러 개 호출해야하고, 객체가 완전히 완성되기 전까지 일관성이 무너진 상태가 된다.

⇒ 클래스를 불변으로 만들 수 없다.

⇒ 스레드에 대한 안정성으로 인하여 작성해야하는 코드가 어렵다.

### 대안3. 빌더 패턴

필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다. 그리고 나서 빌더 객체가 제공하는 세터 메서드들로 선택적으로 매개변수들을 설정한다. 그리고 매개변수가 없는 build 메서드를 호출하여 객체를 얻는다.

```java
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	
	public static classs Builder {
		// 필수 매개변수
		private final int servingSize;
		private final int servings;
		// 선택 매개변수
		private final int calories = 0;
		private final int fat = 0;

		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings= servings;
		}
		public Builder calories(int val) {
			calories = val; return this;
		}
		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}
	private NutritionFacts(Builder buider) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
	}
	// 다음과 같이 사용하면 된다.
	// NutritionFacts cola = new NutritionFacts.Builder(240, 8).calories(100).build();
}
```

생성자나 정적팩터리가 **처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게  더 낫다.**
빌더는 점층적 생성자 보다 클라이언트 코드를 읽고 쓰기가 간결하고, 자바빈즈보다 안전하다.

## 아이템3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워 질 수 있다.

### 싱글턴을 만드는 방식

공통적으로 생성자를 private으로 감춰두고, 유일하게 인스턴스를 접근 할 수 있는 수단으로 public static 멤버를 하나 마련해 둔다.

##### public static final 필드 방식

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis(); // 초기화 할때 딱 한번 호출 된다.
	private Elvis() {...}
	public void leaveTheBuilding() {...}
	
	// 싱글턴임을 보장해주는 readResolve 메서드
	private Object readResolve() {
		// 진짜 Elvis를 반환하고, 가짜 Elvis는 GC에 맡긴다.
		return INSTANCE;
	}
}
```

##### 정적 팩터리 방식

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis(); // 초기화 할때 딱 한번 호출 된다
	private Elvis() {...}
	public static Elvis getInstance() {return INSTNCE;} 
	public void leaveTheBuilding() {...}

	// 싱글턴임을 보장해주는 readResolve 메서드
	private Object readResolve() {
		// 진짜 Elvis를 반환하고, 가짜 Elvis는 GC에 맡긴다.
		return INSTANCE;
	}
}
```

- API의 수정 없이 싱글턴을 풀 수 있다.
- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
- 정적팩터리 메서드의 참조를 공급자로 사용 할 수 있다.(`Supplier<Elvis>`)

이 두가지 방법으로 만든 싱클턴 클래스를 직렬화하려면 단순히 Serializable 선언 만으로는 부족하다. 모든 인스턴스 필드를 일시적이라고 선언하고 readResolve 메서드를 제공해야만한다. ⇒ 그렇지 않을 경우 역정렬화 할 때 새로운 인스턴스가 만들어진다.

##### 열거 타입 방식의 싱글턴

```java
public enum Elvis {
	INSTANCE;
	public void leaveTheBuilding() {...}
}
```

- public static final 필드 방식과 비슷하지만, 보더 간결하다.
- 직렬화 상황이나 리플렉션 공격에도 새로운 인스턴스가 생긱는 일을 완벽히 막아준다.
- 대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

만들려는 싱글턴이 Enum 외에 클래스를 상속해야 한다면 열거타입 방식은 사용 할 수 없다.

## 아이템4. 인스턴스화를 막으려거든 private 생성자를 사용하라

정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 게 아니다. 하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다. 또한 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다. (하위 클래스를 만들어서 인스턴스화 하면 되기 때문)

**⇒ private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.**

```java
public class UtilityClass {
	private UtilityClass() {
		throw new AsserionError();
	}
}
```

## 아이템5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.

의존 객체 주입을 사용하게 되면 인스턴스를 생성할 때  생성자에 필요한 자원을 넘겨줄 수 있다.

```java
public class SpellChecker {
	private final Lexicon dictionary;
	public SpellChecker(Lexicon dictionary) {
		this.dictionary = dictionary;
	}
}
```

의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 의존성이 수천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 한다.
⇒ Spring 같은 프레임워크를 사용하면 이러한 부분을 해소 할 수 있다.

클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 뿐만 아니라 직접 만들게 해도 안된다. 대신 필요한 주언을 생성자(혹은 정적 팩터리나 빌더)에 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연상, 재사용성, 테스트 용이성을 기막히게 개선해준다.

## 아이템6. 불필요한 객체 생성을 피하라.

생성자 대신 정적 팩터리 메서드를 제공하는 불편 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다.

생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 전혀 그렇지 않다.

특히나 생성 비용이 아주 비싼 객체도 있다. 이러한 객체를 반복해서 생성하는 것은 비효율적이다.

예를 들어 String.matches는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않다.

```java
static boolean isRomanNumberal(String s) {
	return s.matches("...생략...");
}
```

값비싼 객체를 재사용 한다면 성능을 개선 시킬 수 있다.

```java
public class RomanNumberal {
	private static final Pattern ROMAN = Pattern.compile("...생략...");
	
	static boolean isRomanNumberal(String s) {
		return ROMAN.matcher(s).matches();
	}
}
```

**객체 생성 자체가 비싸므로 피해야하는 의미가 아니다.** 
실제 JVM에서는 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일은 크게 부담되지 않는다.  프로그램의 명확성, 간셜성, 기능을 위해서라면 좋은일 이다. 

## 아이템7. 다 쓴 객체는 참조를 해제하라.

- GC로 인해서 메모리 관리에 신경을 쓰지 않아도 된다는 오해를 할 수 있지만, 이는 절대 사실이 아니다.
- 객체 참조 하나를 살려두면 GC는 그 객체 뿐만 아니라 참조하는 모든 객체를 회수해 가지 못한다.

### 해법

1. 참조를 다 썼을 때 null처리 시킨다.

    null처리는 객체를 다 쓰자마자 일일히 null 처리하라는 의미는 아니다.
    (그럴 필요도 없다.)

2. 가장 좋은 방법은 그 참조를 담은 변수를 Scope 밖으로 밀어내는 것이다.

### null 처리를 해야하는 경우
1. 일반적으로 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야한다.
2. 캐시
⇒ 백그라운드 스레드를 활용하거나, 새 엔트리를 추가 할 때 추가 작업으로 수행 시킨다.
3. 리스너, 콜백
⇒ 약한 참조에 저장

## 아이템8. finalizer와 cleaner 사용을 피하라

자바에서는 두 가지 소멸자를 제공한다.

- **`finializer` :** 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
- **`cleaner` :** finalizer보다는 덜 위험하지만, 예측할 수 없고, 느리고 일반적으로 불필요하다.

다음 두 가지 용도로 사용되어 진다.

1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할이다.
ex) FileInputStream, FileOutputStream, ThreadPoolExecutor
2. 네이티브 피어와 연결된 객체에서 사용

### 단점

- 공통적으로 제 때 실행되어야 하는 작업은 절대 할 수 없다.
- 상태를 영구적으로 수정하는 작업에서는 두 가지 소멸자에 의존해서는 안된다.
- 심각한 성능문제도 동반한다.
- **`finializer`** 을 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
- 객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, finalizer가 있다면 그렇지 않다.

### 대안

finalizer, cleaner를 대신해줄 방법은 **AutoCloasable**을 구현하고 인스턴스를 다 쓰고 나면 close를 호출 하는 것이다.

## 아이템9. try-finally 보다는 try-with-resource를 사용하라

자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야하는 자원이 많다.

ex) `InputStream`, `OutputStream`, `java.sql.Connection`

### try-finally

전통적으로 많이 쓰여 왔다.

```java
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine()
	} finally {
		br.close()
	};
}
```

이 방식은 자원이 둘 이상 사용 될 경우 코드가 너무 지저분해 진다.
뿐만 아니라  try와 finally 블록에서 모두 예외가 발생할 수 있는데, 두 군데 모두 예외가 발생하게 되고, 마지막에 발생된 예외에 의해서 첫번째 예외를 집어삼키게 된다. 즉 디버깅이 매우 어려워진다.

### try-with-resources

간결해 졌을 뿐만 아니라 문제를 진단하기도 훨씬 좋다.

```java
static String firstLineOfFile(String path) throws IOException {
	try (BufferedReader br = new BufferedReader(new FileReader(path))) {
		return br.readLine()
	}
}
```

```java
static String copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
	 	   OutputStream out = new FileOutputStream(dst))) {
		byte[] buf = new byte(BUFFERED_SIZE];
		int n;
		while((n = in.read(buf)) >= 0)
			out.write(buf, 0, n);
	}
}
```

**꼭 회수해야하는 자원을 다룰 때는 반드시 try-with-resources를 사용하자**

## 관련 용어

- **불변(immutable, immutability)**  
    어떠한 변경도 허용하지 않는다는 뜻으로, 주로 변경을 허용하는 가변(mutable) 객체와 구분하는 용도로 쓰인다. (대표적으로 String)

- **불변식(invariant)**  
    프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야하는 조건을 말한다.

- **싱글턴(Sigleton)**  
    인스턴스를 오직 하나만 생성할 수 있는 클래스

- **어댑터**  
    실제 작업은 뒷단 객체 위임하고, 자신은 제 2의 인터페이스를 해주는 객체이다. 어댑터는 뒷단 객체만 관리하면 된다. 즉 뒷단 객체 외에는 관리할 상태가 없으므로 뒷단 객체 하나당 어댑터 하나씩만 만들어지면 충분하다.

- **오토박싱**  
    프로그래머가 기본 타입과 박싱도니 기본타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다.
    (int와 Integer와 같은 기본타입과 Wrapper 클래스)
    ⇒ 기본타입과 대응되는 박싱타입의 구분을 흐려주지만 없애는 주는 것은 아니다.

- **네이티브 피어**  
    일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체 
    (자바 객체가 아니므로 GC대상이 아니다.)
	
-	**참조유형**  
		[(Java) 참조 유형 (Strong Reference/ Soft Reference/ Weak Reference/ Phantom References)](https://lion-king.tistory.com/entry/Java-%EC%B0%B8%EC%A1%B0-%EC%9C%A0%ED%98%95-Strong-Reference-Soft-Reference-Weak-Reference-Phantom-References)