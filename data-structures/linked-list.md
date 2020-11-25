# Linked List

현재 노드의 값, 다음 노드를 가리키는 레퍼런스를 갖는 노드들로 이루어진 자료구조. 다음과 같이 간소화한 형태로 나타낼 수 있다.

```plaintext
┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ╷
│ 5 │╶┼─>│ 1 │╶┼─>│ 4 │╶┼─>│ 5 │╶┼─>│ 6 │╶┼─>│
└───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  ╵
```

Linked list는 다음과 같은 동작을 수행한다.

- 초기화
- 노드 탐색
- 노드 삽입
- 노드 값 갱신
- 노드 삭제

-----

## 설계

위의 그림을 보고 시작 노드를 리스트 자체로 사용하는 형태로 구현하려면, 리스트 초기화나 노드 삽입·삭제를 구현할 때 코너 케이스를 처리하는 데 신경이 쓰일 것이다. 무슨 말인지 감이 잘 안 온다면 다음 질문에 답해 보자.

1. 빈 리스트를 표현하려면 어떻게 해야 하는가?
2. 리스트의 맨 앞에 노드를 추가하려면 어떻게 해야 하는가?
3. 리스트의 맨 앞의 노드를 삭제하려면 어떻게 해야 하는가?

빈 리스트를 `null`로 표현할 수는 있겠다. 그렇다면 ‘빈 리스트’와 ‘아무것도 할당하지 않은 상태’는 어떻게 구분할 것인가? 당장 문자열에서도 빈 문자열(`""`)과 `null`은 분명 다르다.

또, 리스트의 맨 앞에서 추가·삭제를 수행할 때마다 리스트의 시작 노드를 가리키는 변수의 값을 업데이트해야 한다. 시작 노드를 리스트 자체로서 사용하면서 이런 동작을 수행하는 명료한 방법이 떠오르지 않는다. 노드 추가·삭제 함수가 새로운 시작 노드를 리턴하게 하고 노드를 추가·삭제할 때마다 이 리턴 값을 리스트에 재할당한다든지 하는 장황한 방법 정도가 떠오른다.

이런 문제들을 해결하는 방법은 두 가지가 있다.

1. 래퍼(wrapper)로 노드를 감싸기
2. 데이터를 갖지 않는 sentinel node를 맨 앞에 추가

먼저, 래퍼로 감싸고 시작 노드로 `null`을 주면 빈 리스트를 표현할 수 있다. 아무것도 할당하지 않은 상태는 래퍼 자체가 `null`인 것으로 표현할 수 있다. 리스트의 맨 앞에서 추가·삭제를 수행하는 경우도 (코너 케이스에 대한 분기가 필요하기는 하지만) 처리할 수 있다.

맨 앞에 sentinel node를 두면 좀 더 간결하게 해결할 수 있다. 빈 리스트는 sentinel node 하나만 있는 상태로 표현할 수 있다. 맨 앞에 sentinel node가 있으므로 리스트 맨 앞에서의 노드 추가·삭제도 (코너 케이스에 대한 분기 없이) 리스트 중간에서 하는 것과 동일하게 처리할 수 있다. 다만, 이 방법은 리스트 자체와 노드의 역할을 분리하지 않으므로 깔끔하지는 않다.

위의 두 가지 방법을 모두 적용하면 한 가지만 적용했을 때의 문제까지 보완할 수 있다. 그러면 다음과 같은 형태가 될 것이다.

```plaintext
┌─────────────────────────────────────────────────────────┐
│ head                                                    │
│ ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ╷ │
│ │   │╶┼─>│ 5 │╶┼─>│ 1 │╶┼─>│ 4 │╶┼─>│ 5 │╶┼─>│ 6 │╶┼─>│ │
│ └───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  ╵ │
└─────────────────────────────────────────────────────────┘
```

이 구조는 TypeScript로 다음과 같이 표현할 수 있다.

```typescript
interface LinkedListNode<T> {
  value: T | null,
  next: LinkedListNode<T> | null,
}

class LinkedList<T> {
  private head: LinkedListNode<T>;
  // Methods go here...
}
```

-----

## 초기화 구현

Linked list의 초기화는 ‘다음 노드가 `null`인 sentinel node’를 생성하여 `head`에 할당하도록 구현한다.

```typescript
class LinkedList<T> {
  private head: LinkedListNode<T>;

  constructor() {
    this.head = {
      value: null,
      next: null,
    };
  }
}
```

시간 복잡도는 `O(1)`이다.

-----

## 노드 탐색 구현

노드 탐색은 두 가지 방법을 고려해 볼 수 있다.

1. 인덱스(위치)로 해당 노드의 값 찾기
2. 값으로 특정 노드 또는 그 노드의 위치 찾기

여기서는 가장 기본적이고 리스트의 모든 노드에 접근할 수 있는 첫 번째 방법으로 구현한다.

배열에서는 첨자를 이용하여 원하는 인덱스의 값을 (거의) 바로 가져올 수 있다. 하지만 linked list에서는 앞에서부터 노드 개수를 세어 가며 찾아야 한다.

```typescript
class LinkedList<T> {
  // ...

  getValue(index: number): T {
    if (index < 0) {
      throw RangeError();
    }

    let currentNode: LinkedListNode<T> = this.head;
    for (let i: number = 0; i <= index; i += 1) {
      currentNode = currentNode.next;
      if (currentNode === null) {
        throw RangeError();
      }
    }
    return currentNode.value;
  }
}
```

최악의 상황은 `index`가 길이보다 크거나 같은 경우이며, 이때 리스트의 끝까지 탐색해야 한다. 시간 복잡도는 `O(n)`이다. `LinkedList<T>`에 길이 필드를 별도로 두어 관리한다면 최악의 상황은 `index`가 (길이 - 1)인 경우이며, 시간 복잡도는 동일하게 `O(n)`이다.

-----

## 노드 삽입 구현

리스트의 특정 위치에 노드를 삽입하는 과정은 다음과 같다.

```typescript
// LinkedList<T>.insert: (index: number, value: T) => void
list.insert(2, 'E');
```

1. 삽입하려는 위치의 직전 노드를 찾는다.

```plaintext
head      0       [1]       2        3
┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ╷
│   │╶┼─>│ A │╶┼─>│ B │╶┼─>│ C │╶┼─>│ D │╶┼─>│
└───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  ╵
```

2. 삽입하려는 값을 가지면서 1에서 찾은 노드의 다음 노드를 가리키는 새 노드를 생성한다.

```plaintext
head              prev
┌───┬─┐  ┌───┬─┐  ┌───┬─┐           ┌───┬─┐  ┌───┬─┐  ╷
│   │╶┼─>│ A │╶┼─>│ B │╶┼──────────>│ C │╶┼─>│ D │╶┼─>│
└───┴─┘  └───┴─┘  └───┴─┘  ┌───┬─┐┌>└───┴─┘  └───┴─┘  ╵
                           │ E │╶┼┘
                           └───┴─┘
```

3. 1에서 찾은 노드가 2에서 생성한 노드를 가리키도록 변경한다.

```plaintext
head              prev
┌───┬─┐  ┌───┬─┐  ┌───┬─┐           ┌───┬─┐  ┌───┬─┐  ╷
│   │╶┼─>│ A │╶┼─>│ B │╶┼┐          │ C │╶┼─>│ D │╶┼─>│
└───┴─┘  └───┴─┘  └───┴─┘│ ┌───┬─┐┌>└───┴─┘  └───┴─┘  ╵
                         └>│ E │╶┼┘
                           └───┴─┘
```

그런데 위의 1번 순서가 앞서 구현한 노드 탐색과 중복된다. 먼저 노드를 찾는 부분을 메서드로 추출한다.

```typescript
class LinkedList<T> {
  // ...

  private getNode(index: number): LinkedListNode<T> {
    if (index < 0) {
      throw RangeError();
    }

    let currentNode: LinkedListNode<T> = this.head;
    for (let i: number = 0; i <= index; i += 1) {
      currentNode = currentNode.next;
      if (currentNode === null) {
        throw RangeError();
      }
    }
    return currentNode;
  }

  getValue(index: number): T {
    return this.getNode(index).value;
  }
}
```

이제 추출한 메서드 `getNode`를 활용하여 노드 삽입을 구현한다.

```typescript
class LinkedList<T> {
  // ...

  insert(index: number, value: T): void {
    const prevNode: LinkedListNode<T> = this.getNode(index - 1);
    const newNode: LinkedListNode<T> = {
      value,
      next: prevNode.next,
    };
    prevNode.next = newNode;
  }
}
```

삽입 위치 직전의 노드를 찾는 과정의 시간 복잡도는 `O(n)`, 노드를 생성하고 연결하는 과정의 시간 복잡도는 `O(1)`이다. 따라서 전체 시간 복잡도는 `O(n)`이다.

-----

## 노드 값 갱신 구현

리스트의 특정 위치의 노드의 값을 갱신하는 과정은 다음과 같다.

```typescript
// LinkedList<T>.update: (index: number, value: T) => void
list.update(2, 'X');
```

1. 값을 갱신하려는 노드를 찾는다.

```plaintext
head      0        1       [2]       3
┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ╷
│   │╶┼─>│ A │╶┼─>│ B │╶┼─>│ C │╶┼─>│ D │╶┼─>│
└───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  ╵
```

2. 찾은 노드의 값을 변경한다.

```plaintext
head                       target
┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ╷
│   │╶┼─>│ A │╶┼─>│ B │╶┼─>│ X │╶┼─>│ D │╶┼─>│
└───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  ╵
```

```typescript
class LinkedList<T> {
  // ...

  update(index: number, value: T): void {
    this.getNode(index).value = value;
  }
}
```

노드를 찾는 과정의 시간 복잡도는 `O(n)`, 노드의 값을 변경하는 과정의 시간 복잡도는 `O(1)`이다. 따라서 전체 시간 복잡도는 `O(n)`이다.

-----

## 노드 삭제 구현

리스트의 특정 위치의 노드를 삭제하는 과정은 다음과 같다.

```typescript
// LinkedList<T>.remove: (index: number) => void
list.remove(2);
```

1. 삭제하려는 위치의 노드와 그 직전 노드를 찾는다.

```plaintext
head      0       [1]      [2]       3
┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ╷
│   │╶┼─>│ A │╶┼─>│ B │╶┼─>│ C │╶┼─>│ D │╶┼─>│
└───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  ╵
```

2. 삭제 위치 직전의 노드가 삭제 위치 다음 노드를 가리키도록 변경한다.

```plaintext
                           target
                           ┌───┬─┐
head              prev     │ C │╶┼┐
┌───┬─┐  ┌───┬─┐  ┌───┬─┐  └───┴─┘└>┌───┬─┐  ╷
│   │╶┼─>│ A │╶┼─>│ B │╶┼──────────>│ D │╶┼─>│
└───┴─┘  └───┴─┘  └───┴─┘           └───┴─┘  ╵
```

3. (삭제 대상 노드를 메모리에서 해제한다.)

```plaintext
head              prev
┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ┌───┬─┐  ╷
│   │╶┼─>│ A │╶┼─>│ B │╶┼─>│ D │╶┼─>│
└───┴─┘  └───┴─┘  └───┴─┘  └───┴─┘  ╵
```

```typescript
class LinkedList<T> {
  // ...

  remove(index: number): void {
    const prevNode: LinkedListNode<T> = this.getNode(index - 1);
    const targetNode: LinkedListNode<T> = prevNode.next;
    prevNode.next = targetNode.next;
  }
}
```

삭제할 노드와 그 직전 노드를 찾는 과정의 시간 복잡도는 `O(n)`, 대상 직전 노드와 대상 다음 노드를 연결하(고 대상 노드를 메모리에서 해제하)는 과정의 시간 복잡도는 `O(1)`이다. 따라서 전체 시간 복잡도는 `O(n)`이다.

-----

## 구현한 전체 코드

```typescript
interface LinkedListNode<T> {
  value: T | null,
  next: LinkedListNode<T> | null,
}

class LinkedList<T> {
  private head: LinkedListNode<T>;

  constructor() {
    this.head = {
      value: null,
      next: null,
    };
  }

  private getNode(index: number): LinkedListNode<T> {
    if (index < 0) {
      throw RangeError();
    }

    let currentNode: LinkedListNode<T> = this.head;
    for (let i: number = 0; i <= index; i += 1) {
      currentNode = currentNode.next;
      if (currentNode === null) {
        throw RangeError();
      }
    }
    return currentNode;
  }

  getValue(index: number): T {
    return this.getNode(index).value;
  }

  insert(index: number, value: T): void {
    const prevNode: LinkedListNode<T> = this.getNode(index - 1);
    const newNode: LinkedListNode<T> = {
      value,
      next: prevNode.next,
    };
    prevNode.next = newNode;
  }

  update(index: number, value: T): void {
    this.getNode(index).value = value;
  }

  remove(index: number): void {
    const prevNode: LinkedListNode<T> = this.getNode(index - 1);
    const targetNode: LinkedListNode<T> = prevNode.next;
    prevNode.next = targetNode.next;
  }
}
```
