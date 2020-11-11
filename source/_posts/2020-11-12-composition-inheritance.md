---
layout: post
title:  "The concept of composition and inheritance"
date:   2020-11-12 00:0054 +0900
categories: script
---

It’s too hard to build the structure of codes when i was beginner about the react library. because React don’t support the inheritance concept. I am really familar about the java language. so I couldn’t think the some structure using the composition. I would record about the difference between them.

### IS-A, HAS-A relationship.

We usually express the inheritance is for expressing a ‘HAS-A’ relationship and compostition is for expressing a ‘IS-A’ relationship.

### React express ‘IS-A’ as a composition

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