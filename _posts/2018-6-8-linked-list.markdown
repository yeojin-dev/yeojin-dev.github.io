---
layout: post
title:  "Linked List"
date:   2018-6-8
categories: [algorithm]
---

<p class="intro"><span class="dropcap">L</span>inked List는 자료구조의 하나이다. 배열과 달리 항목의 추가, 삭제가 O(1)의 시간복잡도로 가능하다. 대신 항목을 참조할 때는 O(n)의 시간복잡도를 갖는다.</p>

## 배열의 참조

배열은 서로 연관성이 있는 데이터들을 메모리에 저장할 때 효과적이다. 변수 a, b, c를 각각 선언하는 것보다 number[3]이 훨씬 낫다는 뜻이다. 배열은 메모리 파편화를 예방할 수 있고 반복문에 사용하기도 용이하다. 배열의 첫 번째 요소의 주소값 포인터(number)와 index(3) 값을 이용한 메모리 주소 연산으로 배열의 요소를 쉽게 참조할 수 있기 때문이다. 예를 들어 number[3]이라면 아래와 같은 메모리 연산을 하게 된다. ()배열의 데이터 타입은 정수(int)라 가정하였다.)

```
참조할 메모리 주소 : number + index * sizeof(int)
```

이렇게 하면 O(1)의 시간복잡도로 참조가 가능하다.

## 배열의 문제점

하지만 배열은 치명적인 문제가 있는데 바로 배열 항목의 추가와 삭제이다. 배열은 정적으로 메모리를 할당받기 때문에 새로운 항목을 추가하려면 아예 배열을 새로 만들어야 한다. (삭제할 경우에도 삭제하는 항목 뒤의 배열 항목들을 옮겨주어야 한다.) 그리고 새로운 배열 이전의 항목을 모두 옮겨야 한다. C언어에서는 꽤 많은 소스코드가 필요할 것이고 연산 부하도 많이 걸린다. (이럴 때 JAVA에는  System.arraycopy()라는 메소드가 있다.) 확실히 매우 비효율적인 연산임에 틀림없다.

## Linked List

배열의 문제점에 대한 대안으로 Linked List가 있다. 리스트는 각 항목(Node라고 부른다.)별로 다음 항목의 주소값을 갖고 있는 포인터를 가지고 있다. 그리고 이 포인터를 순회하면서 각 항목에 접근한다. 아래 그림을 보자.

![Linked List](https://upload.wikimedia.org/wikipedia/commons/thumb/9/9c/Single_linked_list.png/400px-Single_linked_list.png)

위 그림에서 아래 파란색으로 표시된 부분이 포인터라고 할 수 있다. 이렇게 하면 참조는 상대적으로 어렵게 된다. 메모리 주소 연산이 아니라 직접 항목의 포인터를 참조해가야만 하기 때문이다. 반면에 항목의 추가와 삭제는 매우 쉬워진다. 새로운 항목의 포인터만 서로 바꿔주는 연산을 하면 되기 때문이다.

## Implementation

Node와 Linked List를 파이썬으로 구현해보았다. 가장 기본적인 메소드라고 할 수 있는 append(), insert(), delete(), pop()를 구현하였다.

```python
class Node:

    def __init__(self, data):
        self.data = data
        self.next = None

    def __str__(self):
        return self.data


class LinkedList:

    def __init__(self, data=None):
        self._head = Node(data)
        self._size = 1 if data else 0

    def __len__(self):
        return self._size

    def __iter__(self):
        self._index = 0
        return self

    def __next__(self):
        if self._index >= self._size:
            raise StopIteration
        curr_node = self[self._index]
        self._index += 1
        return curr_node

    def __getitem__(self, item):
        if item > self._size:
            raise ValueError
        result = self._head
        for i in range(item):
            result = result.next
        return result

    def append(self, data):
        new_node = Node(data)
        curr_node = self._head
        while curr_node.next:
            curr_node = curr_node.next
        curr_node.next = new_node
        self._size += 1

    def insert(self, index, data):
        if index > self._size:
            raise ValueError

        new_node = Node(data)
        curr_node = self._head

        if index == 0:
            new_node.next = self._head
            self._head = new_node

        else:
            for i in range(index - 1):
                curr_node = curr_node.next
            new_node.next = curr_node.next
            curr_node.next = new_node

        self._size += 1

    def delete(self, index):
        if index > self._size:
            raise ValueError

        curr_node = self[index - 1]
        curr_node.next = self[index].next
        self._size -= 1

    def pop(self):
        result = self[self._size - 1].data
        self.delete(self._size - 1)
        return result
```

## Linked List와 시간복잡도

delete() 메소드의 시간복잡도는 O(1)이 된다.
insert() 메소드에 바로 앞에 붙일 Node를 인자로 전달하면 훨씬 더 삽입이 쉬울 것이다. 이 경우 시간복잡도는 delete()와 같은 O(1)이 될 것이다.
반면에 \_\_getitem\_\_() 메소드와 같이 Node를 참조하려는 경우에는 O(n)의 시간복잡도가 된다.

## Magic Method

다른 언어로 구현된 Linked List와 다르게 파이썬은 \_\_iter\_\_() 등의 매직 메소드 구현이 매우 중요하다.
1. \_\_getitem\_\_()의 구현으로 linked_list[index]와 같은 표현식이 가능해진다.
2. \_\_iter\_\_(), \_\_next\_\_()의 구현은 enumerate() 메소드의 사용을 가능하게 해준다.
3. \_\_len\_\_() 메소드 구현으로 인해 len(linked_list) 코드로 Linked List의 Node 개수를 구할 수 있다.

매직 메소드는 파이썬 기본 자료형의 사용과 동일하게 개발자가 구현한 자료구조를 사용할 수 있게 해주기 때문에 유용하다. 자료구조를 설계할 때 매직 메소드 구현을 반드시 고려해야만 한다.
