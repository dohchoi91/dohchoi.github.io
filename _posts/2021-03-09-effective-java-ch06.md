---
title: "06.열거타입과 애너테이션"
subtitle: "아이템34 ~ 아이템41"
permalink: /notes/effective-java/ch06
author_profile: true
date: 2021-03-09 12:55
categories:
  - 이펙티브 자바
tags:
  - java
last_modified_at: 2021-03-09 12:55
---

자바에는 특수한 목적의 참조 타입이 두가지 있다. 

1. 열거타입 (클래스 종류 중 하나)
2. 애너테이션(인터페이스 종류 중 하나)

# 아이템34 : int 상수 대신 열거 타입을 사용하라

> 열거 타입은 확실히 정수 상수보다 뛰어나다. 더 읽기 쉽고 안전하고 강력하다. 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다.

## 정수 열거 패턴의 단점

- 타입 안전을 보장 할 수 없고, 표현력이 좋지 않다.
- 프로그램이 깨지기 쉽다. 즉 상수값의 변경 시 반드시 다시 컴파일 해야 한다.
- 문자열로 출력하기가 까다롭다.

## 열거 타입(enum type)

- 열거타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다.
- 컴파일 타입의 안정성을 제공한다.
- 열거타입은 각자의 이름 공간이 있어서 이름이 같은 상수도 공존한다.
- 임의의 메서드나 필드를 추가 할 수 있다.
- 임의의 인터페이스를 구현 할 수 있다.
- **열거 타입 상수 각각을 특정 데이터와 연결지으려면 데이터를 받아 인스턴스 필드에 저장하면 된다.**
- 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 `values`메서드 제공

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6);
    // ...

    private final double mass; // 질량
    private final double radius; // 반지름
    private final double surfaceGravity; // 표면중력

    private static final double G = 6.67300E-11; // 중력 상수

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
```

## 상수별 메서드 구현

열거 타입에 추상 메서드를 선언하고, 각 상수별 클라이언트 몸체에 각 상수에 맞게 재정의 하는 방법

```java
public enum Operation {
	PLUS("+"){public double apply(double x, double y){return x+y;}},
	MINUS("-"){public double apply(double x, double y){return x-y;}},
	TIMES("*"){public double apply(double x, double y){return x*y;}},
	DIVIDE("/"){public double apply(double x, double y){return x/y;}};
	
	private final String symbol;
	Operation(String symbol) {this.symbol = symbol;}  
	public abstract double apply(double x, double y);
} 
```

## 전략 열거 타입 패턴

```java
enum PayrollDay {
	MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), 
	THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
	SATURDAY(WEEKEND), SUNDAY(PayType.WEEKEND);

	private final PayType payType;

	PayrollDay(PayType payType) {this.payType = payType;}
	int pay(int minutesWorked, int payRate) {
		return payType.pay(minutesWorked, payRate);
	}
	// 전략 열거 타입
	enum PayType {
		WEEKDAY {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked <= MINS_PER_SHIFT ? 0 : 
					(minsWorked  - MINS_PER_SHIFT) * payRate / 2;
			}
		},
		WEEKEND {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked * payRate / 2;
			}
		};

		abstract int overtimePay(int mins, int payRate);
		private static final int MINS_PER_SHIFT = 8 * 60;

		int pay(int minutesWorked, int payRate) {
			int basePay = minutesWorked * payRate;
			return basePay + overtimePay(minutesWorked, payRate);
		}
	}
}
```

## 열거 타입을 사용할 때

필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합일 경우
ex) 태양계 행성, 한 주의 요일, 체스 말 등등

## 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.

열거 타입은 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되어 있다. 

# 아이템35 : ordinal 메서드 대신 인스턴스 필드를 사용하라

> 열거타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고 인스턴스 필드에 저장 하자

## ordinal 메서드

- 열거타입에서 해당 상수가 열거 타입에서 몇 번째 위치인지를 반환 메서드
- 상수 선언 순서를 바꾸는 순간 오작동한다.
- Enum API 문서에서는 해당 메서드는 `EnumSet`과 `EnumMap` 같이 열거 타입 기반의 범용 자료구조에 사용할 목적으로 설계 되었다.

# 아이템36 : 비트 필드 대신 EnumSet을 사용하라

> 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다 하더라도 비트필드를 사용할 이유는 없다. EnumSet 클래스가 비트 필드 수준의 명령함과 성능을 제공하고 있다. 유일한 단점은 불변 EnumSet을 만들 수 없다.

## 비트필드

아래와 같은 식으로 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있고, 이렇게 만들어진 집합을 비트필드라 한다.

```java
public class Text {
	public static final int STYLE_BOLD          = 1 << 0 // 1
	public static final int STYLE_ITALIC        = 1 << 1 // 2
	public static final int STYLE_UNDERLINE     = 1 << 2 // 4
	public static final int STYLE_STRIKETHROUGH = 1 << 3 // 8
	
	// 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
	public void applyStyles(int styles) {...}
}
```

- 사용 시 비트별 연산을 사용해 합집합, 교집합 같은 집합 연산 기능을 효율적으로 수행 할 수 있다.
- 단순한 열거 상수를 출력할 때 보다 해석하기가 훨씬 어렵다.
- 비트필드 하나하나 녹아 있는 모든 원소를 순회하기가 까다롭다
- 최대 몇 비트가 필요한지 API 작성시 미리 예측하여 적절한 타입(int, long)을 선택해야 한다.

## EnumSet : 비트 필드를 대체하는 현대적 기법

- 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.
- Set 인터페이스를 완벽히 구현하며, 타입 안전하다
- EnumSet 내부는 비트벡터로 구현되어 있다.

```java
public class Text {
	public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}
	// 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
	public void applyStyles(Set<Style> styles) {...}
}
```

# 아이템37 : ordinal 인덱싱 대신 EnumMap을 사용하라

> 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라.

## ordinal메서드로 배열 인덱스로 사용하면?

- 배열은 제네릭과 호환이 되지 않으니 비검사 형변환을 수행해야하고, 컴파일이 깔끔하게 되지 않는다.
- 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
- 정확한 정수값을 사용한다는 것을 보증해야한다.
⇒ 잘못된 값을 사용시 오작동하거나 `ArrayIndexOutOfBoundException` 발생

# 아이템38 : 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

> 열거 타입 자체는 확장 할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다. 이렇게 하면 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입을 만들 수 있다. 그리고 API가 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.

# 아이템39 : 명명패턴보다 애너테이션을 사용하라

> 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.
자바 프로그래머라면 예외없이 자바가 제공하는 애너테이션 타입들을 사용해야한다.

## 명명패턴의 단점

1. 오타가 나면 안된다.
2. 올바른 프로그램 요소에서만 사용되리라 보증 할 방법이 없다.
3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.

## 메타애너테이션

애너테이션 선언에 다는 애너테이션

```java
import java.lang.annotation.*

@Retention(RetentionPolicy.RUNTIME) //메타애너테이션
@Target(ElementType.METHOD) //메타애너테이션
public @interface Test {
}
```

# 아이템40 : @Override 애너테이션을 일관되게 사용하라

> 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면, 실수 했을 경우 컴파일러에서 바로 알려줄 것이다.

## @Override

상위 타입의 메서드를 재정의했음을 의미

# 아이템41 : 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

> 마커 인터페이스와 마커 애너테이션은 각자의 쓰임이 있다.
새로 추가하는 메서드 없이 단지 **타입 정의가 목적이라면 마커 인터페이스**를 선택하자.
**클래스나 인터페이스 외의 프로그램 요소에 마킹**해야 하거나, 애너테이션을 적극 활용하는 프레임워크 일부로 **그 마커를 편입시키고자 한다면 마커 애너테이션**이 올바른 선택이다.

## 마커 인터페이스

- 아무 메서드도 담지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스 ex) `Serializable` 인터페이스
- 컴파일 타임에 오류 검출을 할 수 있다.

## 마커인터페이스가 마커 애너테이션 보다 나은 점

1. 마커인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있다.
2. 적용대상을 정밀하게 지정이 가능하다.

## 마커 애너테이션이 마커인터페이스 보다 나은 점

거대한 애너테이션 시스템의 지원을 받는다.
⇒ 애너테이션을 적극 활용하는 프레임워크(ex. Spring)에서는 마커 애너테이션을 쓰는 쪽이 일관성을 지키는데 유리할 것이다.