---
title: "05.제네릭"
subtitle: "아이템26 ~ 아이템33"
permalink: /notes/effective-java/ch05
author_profile: true
date: 2021-03-05 13:35
categories:
  - 이펙티브 자바
tags:
  - java
last_modified_at: 2021-03-05 13:35
---
제네릭은 자바5부터 사용 할 수 있다. 컴파일러는 알아서 형변환 코드를 추가 할 수 있고, 엉뚱한 타입의 객체를 넣으려는 시도를 컴파일 과정에서 차단하여 더 안전하고 명확한 프로그램을 만들어 준다.

## 아이템26 : 로 타입은 사용하지 말라.

> 로 타입을 사용하면 런타임에 예외가 일어날 수 있으니 사용하면 안 된다.

### 제네릭

**클래스와 인터페이스 선언에 따라 매개변수가 쓰이면 이를 제네릭 클래스 혹은 제네릭 인터페이스라 한다. 제네릭 클래스, 제네릭 인터페이스를 통틀어 제네릭이라고 한다.**

- 각각의 제네릭 타입은 **일련의 매개변수화 타입을 정의**한다.
- 클래스(혹은 인터페이스) 이름이 나오고, 이어서 꺽쇠괄호 안에 실제 타입 매개변수들을 나열한다. ex) List<String>
- 제네릭 타입을 하나 정의하면 그에 딸린 로타입(raw type)도 함께 정의된다.

### 로 타입

제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 애기한다. ex) List list; 

- 로 타입은 컴파일 에러는 발생하지 않지만 절대 써서는 안된다.
- 로 타입을 쓰면 제네릭이 안겨주는 안정성과 표현력을 모두 잃게 된다.

### 와일드 카드 타입

비한정적 와일드카드 타입을 대신 사용하는 게 좋다. 

제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않다면 물음표(?)를 사용하자. 

```java
// 비한정적인 와일드 카드 타입을 사용하라 - 타입 안전하며 유연하다
static int numElementInCommon(Set<?> s1, Set<?> s2) {...}
```

### 로 타입 사용 예외

1. class 리터럴
ex) `List.class`,  `int.class` 
2. instanceof  사용시

    ```java
    if ( o instanceof Set) {
    	Set<?> s = (Set<?>) o;
    	...
    }
    ```

## 아이템27 : 비검사 경고를 제거하라

> 비검사 경고는 중요하니 무시하지 말자. 경고는 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거하라. 방법을 찾지 못했다면 그 코드의 아전함을 증명하고 가능한 범위를 좁히고  @SuppressWarnings("unchecked")애너테이션으로 경고를 숨겨라

할 수 있는 한 모든 비검사 경고를 제거하라. 그러면 해당 코드의 타입의 안정성이 보장된다.

- 경고를 제거 할 수는 없지만 타입 안전하다고 확신할 수 있다면`@SuppressWarnings("unchecked")` 애너테이션을 달아 경고를 숨기자.
- `@SuppressWarnings` 애너테이션은 항상 가능한 한 좁은 범위 적용하자
- `@SuppressWarnings("unchecked")` 애너테이션을 사용할 때면 그경고를 무시해도 안전한 이유를 항상주석으로 남겨라.

## 아이템28 : 배열보다는 리스트를 사용하라

> 배열과 제네릭에는 매우 다른 규칙이 적용된다. 배열은 공변이고 실체화가 되는 방면 제네릭은 불공변이고 타입 정보가 소거된다. 그 결과 배열은 런타임에는 타입 안전하지 않지만, 컴파일타임에는 그렇지 않다. 그래서 둘을 썩어 쓰기란 쉽지 않다. 둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용해 보자.

### 배열과 제네릭 타입 차이

1. 배열은 공변이다. (공변 : 함께 변한다.)
Sub가 Super의 하위타입이라면 배열 Sub[]는 배열 Super[]의 하위 타입이 된다.
2. **제네릭은 불공변이다.**
**List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아니다.**

```java
// 런타임에 실패한다.
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다.";

// 컴파일 되지 않는다.
List<Object> ol = new ArrayList<Long>(); // 호환이 되지않는 타입이다.
ol.add("타입이 달라 넣을 수 없다.");
```

### 제네릭 배열을 만들지 못하게 막는 이유는?

- 타입이 안전하지 않기 때문이다.
- 만약 허용 시 `ClassCastException`이 발생 할 수 있다.

*배열로 형변환 할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 E[] 대신 컬렉션인 List<E>를 사용하면 해결된다.*

## 아이템29 : 이왕이면 제네릭 타입으로 만들라

> 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다. 그러니 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라. 그렇게 하려면 제네릭 타입으로 만들어야 할 경우가 많다. 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하자.

### 제네릭으로 만들기

기존 Object를 타입매개변수로 변환한다. 

```java
// 제네릭 스택으로 가는 첫단계 - 컴파일 되지 않는다.
public class Stack<E> {
	private E[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack() {
		elements = new E[DEFAULT_INITIAL_CAPACITY];
	}
	public void push(E e) {
		ensureCapacity();
		elements[size++] = e;
	}
	public E pop() {
		if (size == 0)
			throw new EmptyStackException();
		E result = elements[--size];
		elements[size] = null;
		return result;
	}
} 
```

**방법1**

```java
// 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
// 따라서 타입 안정성은 보장하지만, 이 배열이 런타임 타입은 E[]가 아닌 Object[]다.
@SuppressWarnings("unchecked")
public Stack() {
		elements = (E[]) new E[DEFAULT_INITIAL_CAPACITY];
}
```

**방법2**

```java
// 비검사 경고를 적절히 숨긴다.
public E pop() {
	if (size == 0)
		throw new EmptyStackException();
	// push에서 E타입만 허영하므로 이 형변환은 안전하다.
	@SuppressWarnings("unchecked") E result = (E) elements[--size];
	elements[size] = null;
	return result;
}
```

## 아이템30 : 이왕이면 제네릭 메서드로 만들라

> 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다. 타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 그렇게 하려면 제네릭 메서드가 되어야 한다.

### 제네릭 메서드

(타입 매개변수들을 선언하는) **타입 매개변수 목록**은 메서드의 제한자와 반환 타입사이에 온다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<>(s1);
	result.addAll(s2);
	return result;
}
```

### 재귀적 타입 한정

- 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용범위를 한정 할 수 있다.
- 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰인다.

## 아이템31 : 한정적 와일드카드를 사용해 API 유연성을 높여라

> 와일드카드 타입을 적용하면 API가 유연해진다.
PECS 공식을 기억하자.

### 유연성을 극대화화려면 원소의 생산자나 소비자용입력 매개변수에 와일드카드 타입을 사용하라.

```java
// E 생산자(producer) 매개변수에 와일드카드 타입 적용
// E의 하위 타입의 Iterable임을 의미
public void pushAll(Iterable<? extends E> src) {
	for(E e : src)
		push(e);
}
// E 소비자(consumer) 매개변수에 와일드카드 타입 적용
// E의 상위 타입의 Collection임을 의미
public void popAll(Collection<? super E> dst) {
	while(!isEmpty()) {
		dst.add(pop())
	}
}
```

※  입력매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을게 없다.

※  반환타입에는 한정적 와일드 카드를 사용하면 안된다. 유연성을 높여주기는 커녕 클라이언트 코드에서도 와일드카드 타입을 써야 하기 때문이다.

### 펙스(PECS) : producer-extends, consumer-super

- 매개변수화 타입 T가 생산자라면 `<? extends T>`를 사용한다
ex) 위에 예시로 Stack이 사용할 E 인스턴스를 생산
- 매개변수화 타입 T가 소비자라면 `<? super T>`를 사용한다
ex) 위에 예시로 Stack으로 부터 E 인스턴스를 소비

## 아이템32 : 제네릭과 가변인수를 함께 쓸 때는 신중하라

> 가변인수와 제네릭은 궁합이 좋지 않다. 가변인수 기능은 배열을 노출하여 추상화가 완변하지 못하고, 배열과 제네릭의 타입 규칙이 서로 다르기 때문이다. 제네릭 varargs 매개변수는 타입이 안전하지는 않지만, 허용은 된다. 

사용을 하고자 한다면, 먼저 그 메서드가 타입 안전한지 확인한 다음@SafeVarargs를 추가한다.

### 가변인수(varargs)

- 메서드에 넘기는 인수의 개수를 클라이언트가 조절하게 해준다.
- 가변인수 메서드를 호출하면 가변인수를  담기 위한 배열이 자동으로 하나 만들어진다.

### 제네릭과 varargs 혼용

- 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.
- 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.
- 제네릭 varargs 배열 매개변수에 다른 메서드가 접근하도록 허용하면 안전하지 않다.

```java
static void dangerous(List<String> ... stringL ists) {
	List<Integer> intList = List.of(42);
	Object[] objects = stringLists;
	objects[0] = intList; // 힙 오염 발생
	String s = stringLists[0].get(0); // ClassCastException
}
```

### @SafeVarargs

- 메서드 작성자가 그 메서드가 안전함을 보장하는 장치.
- 재정의 할 수 없는 메서드에만 달아야한다. 재정의한 메서드도 안전할지는 보장할 수 없기 때문이다.

### 제네릭과 varargs 혼용이 가능한 경우

1. varargs 매개변수 배열에 아무것도 저장하지 않는다.
2. 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출시키지 않는다.

## 아이템33 : 타입 안전 이종 컨테이너를 고려하라.

> 컬력션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어있다. 하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 **타입 안전 이종 컨테이너**를 만들 수 있다. 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라고 한다. 또한 직접 구현 키 타입도 쓸 수 있다. 예컨데 데이터베이스의 행(컨테이너)를 표현한 DatabaseRow 타입에는 제네릭 타입인 Column<T>를 키로 사용할 수 있다.

### 타입 안전 이종 컨테이너 패턴

컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해주게 된다.

- API 예제 코드

    ```java
    public class Favorites {
    	private Map<Class<?>, Object> favorites = new HashMap<>();

    	public <T> void putFavorite(Class<T> type, T instance) {
    		favorites.put(Objects.requireNonNull(type), instance);
    	}
    	public <T> T getFavorite(Class<T> type) {
    		return type.cast(favorites.get(type));
    	}
    } 
    ```

- 클라이언트 예제 코드

    ```java
    public static void main(String[] args) {
    	Favorites f = new Favorites();
    	f.putFavorite(String.class, "java");
    	f.putFavorite(Integer.class, 0xcafebabe);
    	f.putFavorite(Class.class, Favorites.class);
    	
    	String favoritesString = f.getFavorite(String.class);
    	int favoritesInteger = f.getFavorite(Integer.class);
    	Class<?> favoritesClass = f.getFavorite(Class.class);
    }
    ```