---
layout: post
title:  "O.O.P 해결방법과 F.P 해결방법에 대한 고찰"
date:   2020-11-12 00:0054 +0900
categories: script
---

범용적으로 한가지 문제에 대해서 해결할 수 있는 방법은 보통 다양하게 존재하며 이 방법들은 상황에 따라 어떤 것이 좋은지가 달라진다. 이 글에서는 한가지 문제 상황을 제시할 것이며, 이 상황을 O.O.P 방식의 해결방법과 F.P 방식의 해결방법은 어떻게 다를지에 대해서 다루도록 한다. 이 글은 저자의 주관적인 생각이 담겨있음으로 맹목적인 신뢰는 지양하기 바란다.

#### 문제 제시

```
라면을 잘 만드는 두 요리사가 있다. 이 두 요리사에게는 자신들 만의 라면 제조법이 있는데, 이 방법은 서로 간에 상이하며 다만 결과적으로 라면을 만든다라는 것만 동일하다.
```

위와 같은 상황을 코드에 녹여내기 위해서 O.O.P 방법과, F.P 방법 두가지를 활용해 제시할 것이다. 둘 간에 어떤 방법이 좋은지는 상황에 따라 달라지며, 개발자에 선택에 따라 맞추서 사용해야 한다.

#### O.O.P 관점에서의 해결방법

O.O.P 방식으로 이 문제를 해결하기 위해서는 우선 요리사라는 인터페이스, 혹은 추상 클래스를 만들고, 

React 라이브러리를 활용해서 처음으로 프론트 사이트 개발을 했을 때 나에게 가장 어려웠던 것은 Component의 구조를 잡는 것이였다. Java 개발을 주로 했었어서 React 라이브러리가 추구하는 방향성을 잘 이해하지 못했기 때문이였다. 나는 React 라이브러리를 활용해 Component를 만들면서 상속의 개념을 넣고 싶었다. 그러나 실제로 많은 자료들을 찾아보면 React는 상속패턴을 지원하지 않는다고 나와있고 또 상속패턴을 지원할 이유도 찾지 못했다고 나와있다.

이 글은 React 라이브러리는 Component를 어떤식으로 구성하는지, 왜 상속패턴이 필요하지 않고 이를 합성패턴으로 모두 대체가 가능한지에 대해서 저자의 생각을 담아놓은 글이다. 주관적인 생각이 많이 들어가기 때문에 맹목적인 지지를 지양하기를 바란다.

### 'IS-A', 'HAS-A' 관계

객체지향 프로그래밍에 대해서 공부하면 위와 같은 용어를 접하게 되는데. 이 용어하는 바는 상속관계인가 혹은 합성관계인가로 바라볼 수 있다.

예를 들어서 노트북이 있다고 해보자. 노트북은 컴퓨터의 하위 개념으로 노트북은 컴퓨터다 라는 말이 적절하다. 혹은 기타라는 물건은 악기라는 개념의 하위 개념이다. 따라서 기타는 악기이다 라는 말도 적절하다. 객체지향 프로그래밍에서는 이런 관계를 상속관계라 말하고 보통 'IS-A' 관계로 표현한다.

```
Notebook is a Computer
Guitar is an instrument
```

반대로 HAS-A 관계는 소유의 개념이 있는지 없는지를 확인한다. 일반적으로 노트북은 모니터라는 개념이 필요하고 이를 가지고 있다. 또 기타의 경우 기타줄이라는 개념을 가지고 있다. 이러한 관계를 합성관계로 바라볼 수 있다. HAS-A 관계를 활용해 여러 객체들을 합칠 수 있기 때문이다.

```
Notebook has a Monitor
Guitar has a string
```

즉 다시 말하자면 상속개념은 어떤 특정 개념이 다른 개념과 상하위 관계가 있는가를 이야기 하고, 합성개념은 특정 개념이 다른 개념을 소유하는가를 이야기 한다.

### React 라이브러리는 왜 상속의 개념이 없을까?

React 라이브러리는 정확히는 상속의 개념이 없으며, 사실 오직 합성의 개념만 존재한다. VanilaJS 에는 ES6 부터 Class 문법이 생겨나고 기존의 함수기반에 코딩에서 객체지향의 코드도 작성할 수 있도록 발전해가지만 왜 React 라이브러리는 상속관계를 지원하지 않을까? 그 이유는 왜 상속의 개념을 사용해야하는가를 먼저 이해하면 답을 찾을 수 있다.

#### O.O.P 에서 상속 패턴을 사용하는 이유

실무적으로 상속 패턴을 사용하는 가장 큰 이유는 다형성을 활용하기 위함이다. 비슷한 객체이나 특정 동작에 대해서 다르게 처리해야할 필요성이 있을 때 보통 O.O.P 방법으로는 이 비슷한 객체를 하나로 묶도록 부모 객체를 만들고 이를 상속 받아서 다르게 처리해야하는 부분을 오버라이딩한다. 이는 O.O.P 방식의 해결 방법이다. 

그러나 React 라이브러리는 JS 기반을 동작하고 있고 JS는 F.P 방식을 지원한다. 그렇기 때문에 위와 같이 특정 동작에 대해서 다르게 처리해야할 필요성이 있다면 상속의 방법을 활용하지 않고 F.P 방법으로 해결이 가능하다.

html 문법은 태그들을 계층형 구조로 만들어 낸다. React 라이브러리에서 객체에 해당하는 Component 들을 만들어도 사실 이 객체는 결국 html 태그들의 집합이다. 결과물 관점에서 봤을 때 즉 ht

It’s really hard to understand about this. Why does React not support inheritance technique?. I think the answer is the purpose of React library is create the html tag. As you knew, the html tags are tree structures. This tags can’t express the inheritance relationship. so React don’t need to create ‘IS-A’ relationship because It eventually should present the html tags, which means tree structure.

### Abstract technique in Composition

My hardest part is the problem how we show the abstrat thing without inheritance. But it’s possible. abstract thing means ‘It is something’ in the ‘IS-A’ relationship and ‘It has something’ in the ‘HAS-A’ relationship.

```java

interface Computer { }

class Laptop extends Computer { }

class Desktop extends Computer { }

class Main {
	public static void main(String[] args) {
		Computer computer = ...;
	}
}

```

The above code is expressing the abstract things using Inheritance technique.

```typescript

interface ComputerProps { }

const Computer: React.FC<ComputerProps> = ({ children }) => {
	return <>{children}</>;	
}



interface LaptopProps { }

const Laptop: React.FC<LaptopProps> = ({}) => {
	return <Computer>It’s Laptop</Computer>
}

```

And it’s expressing the abstract things using composition technique.

Consequently, React support the more limit envrionment than the host languages like Java. but it’s not the limitation of the javascript. It’s just a principle of React. React don’t need to have a Inheritance techniques. because It eventually will return html tag tree.  

#### Atomic Design