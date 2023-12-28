---
title: "Java Stream.flatMap() 메서드 알아보기"
last_modified_at: 2023-12-28
categories:
  - Java
tags:
  - Java
  - Stream
  - FlatMap
---

> 다른 분이 개발한 코드를 수정하면서 Stream.flatMap() 메서드를 처음 보았는데요.  
> 접해보지 못했던 함수라 어떤 식으로 사용하는지 학습하고 이를 정리해봤어요.  

##### 1. 먼저 map() 메서드를 알아보자

map() 메서드는 스트림 내의 요소들을 특정 함수에 매핑하여 새로운 스트림을 반환해요.

예제를 통해서 map() 함수 동작에 대해 알아볼게요.

```java
public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        map();
    }

    public static void map() {
        List<Person> people = List.of(
                new Person("John", 20),
                new Person("Sara", 21),
                new Person("Jane", 21),
                new Person("Greg", 35)
        );
        List<String> names = people.stream().map(person -> person.getName()).collect(Collectors.toList());
        System.out.println(names);
    }
}
```

```json
[John, Sara, Jane, Greg]
```

위의 코드에서 map() 메서드는 Person 객체를 String 으로 변환하여 새로운 스트림을 반환해요.

##### 2. flatMap() 메서드는 무엇인가

그러면 flatMap() 함수는 어떤 역할을 할까요.

영어 flat 이라는 단어의 뜻은 '평평한' 인데요.

단어의 뜻대로 flatMap()은 고차원 배열 또는 리스트를 1차원 배열 또는 리스트로 평평하게 만드는 역할을 해요.

예제를 통해 flatMap() 함수 동작에 대해 알아볼게요.

```java
public class Main {
    public static void main(String[] args) {
        flatMap();
    }

    public static void flatMap() {
        List<List<Person>> peopleGroup = List.of(
                List.of(
                        new Person("John", 20),
                        new Person("Sara", 21),
                        new Person("Jane", 21),
                        new Person("Greg", 35)
                ),
                List.of(
                        new Person("Mick", 80),
                        new Person("Tom", 81),
                        new Person("Simon", 81),
                        new Person("Kane", 95)
                )
        );

        List<Person> flatMapPeople = peopleGroup.stream()
                .flatMap(Collection::stream)
                .collect(Collectors.toList());

        List<String> flatMapNames1 = peopleGroup.stream()
                .flatMap(Collection::stream)
                .map(Person::getName)
                .collect(Collectors.toList());

        List<String> flatMapNames2 = peopleGroup.stream()
                .flatMap(list -> list.stream().map(Person::getName))
                .collect(Collectors.toList());

        System.out.println(flatMapPeople);
        System.out.println(flatMapNames1);
        System.out.println(flatMapNames2);
    }
}
```

```json
[Person{name='John', age=20}, Person{name='Sara', age=21}, Person{name='Jane', age=21}, Person{name='Greg', age=35}, Person{name='Mick', age=80}, Person{name='Tom', age=81}, Person{name='Simon', age=81}, Person{name='Kane', age=95}]
[John, Sara, Jane, Greg, Mick, Tom, Simon, Kane]
[John, Sara, Jane, Greg, Mick, Tom, Simon, Kane]
```

위 코드를 보면 flatMap() 함수를 통해 2차원 리스트를 1차원 리스트로 바꾸는 것을 알 수 있어요.

그리고 flatMap() 으로 얻은 Person 리스트에 대하여 map() 함수를 통해 String 리스트로 변환하는 것도 확인할 수 있어요.

지금까지 flatMap() 함수에 대해 알아보았는데요.

flatMap() 함수의 역할을 기억해뒀다가 필요할 때 사용하면 개발이 더 편해질 것 같아요.

오늘도 미션 클리어! 👍