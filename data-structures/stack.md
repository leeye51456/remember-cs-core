# 스택

나중에 추가된 데이터를 먼저 내보내는(LIFO: Last In First Out) 자료구조. Stack은 다음과 같은 동작을 수행한다.

- 초기화
- Push: 값 추가
- Pop: 값 추출

스택 자료구조는 여러 가지 방법으로 구현할 수 있는데, 여기서는 배열을 이용한 구현 방법과 linked list를 이용한 구현 방법을 다룬다.

-----

## 배열과 스택 포인터(`top`)를 이용한 스택

### 배열을 이용한 스택의 설계

먼저 들어온 값이 작은 인덱스를 갖게 하는 방식으로 구현할 것이다. 여기서 `top`의 위치를 명확하게 정의해 두어야 하는데, 두 가지 선택지가 있다. 각 선택지에서 스택이 비거나 가득 찼을 때 `top`의 값은 다음과 같다.

- `top`을 ‘마지막으로 들어온 항목의 인덱스’로 정의한 경우
  - Empty: `top`이 `-1`
  - Full: `top`이 (배열의 용량 - 1)
- `top`을 ‘다음에 들어올 항목의 인덱스’로 정의한 경우
  - Empty: `top`이 `0`
  - Full: `top`이 (배열의 용량)

그리고 각 선택지마다 `push`, `pop` 동작을 수행할 때 `top`의 값을 더하고 빼는 순서가 달라진다. 이런 디테일은 직접 구현하다 보면 금방 알고 적용할 수 있을 것이다.

기본적인 구조는 TypeScript로 다음과 같이 표현할 수 있다. TypeScript에서는 배열 자체를 그대로 스택처럼 활용할 수 있지만, 여기서는 레거시 형태로 직접 구현하기로 한다. (C나 Java의 배열로 구현하듯이 구현한다.)

```typescript
class Stack<T> {
  private contents: T[];
  private capacity: number;
  private top: number;
  // Methods go here...
}
```

### 배열을 이용한 스택의 초기화

초기화할 스택의 크기를 받아서 내용을 담을 배열, 그 배열의 용량, `top`을 초기화한다. 여기서는 `top`을 ‘다음에 들어올 항목의 인덱스’로 정의했고 `0`으로 초기화했다.

```typescript
class Stack<T> {
  private contents: T[];
  private capacity: number;
  private top: number;

  constructor(capacity: number) {
    this.contents = [];
    this.capacity = capacity;
    this.top = 0;
  }
}
```

### 배열을 이용한 스택의 `isEmpty`, `isFull` 메서드

`push`와 `pop` 메서드를 구현하기 전에, 먼저 스택이 비었는지 또는 가득 찼는지 확인해야 한다. 이 부분을 별도의 메서드로 구현한다.

```typescript
class Stack<T> {
  // ...

  isEmpty(): boolean {
    return top === 0;
  }

  isFull(): boolean {
    return top === capacity;
  }
}
```

### 배열을 이용한 스택의 `push` 메서드

`push`는 주어진 값을 스택에 추가하는 메서드이다. 스택이 가득 차서 값을 추가할 수 없다면 에러를 던진다.

```typescript
class Stack<T> {
  // ...

  push(value: T): void {
    if (this.isFull()) {
      throw new RangeError();
    }

    this.contents[this.top] = value;
    this.top += 1;
  }
}
```

### 배열을 이용한 스택의 `pop` 메서드

`pop`은 스택에서 값을 꺼내 반환하는 메서드이다. 스택이 비어서 값을 꺼낼 수 없다면 에러를 던진다. 배열과 스택 포인터(`top`)로 구현하는 스택에서는 꺼낸 내용을 지우지 않고 스택 포인터만 옮겨도 상관없다.

```typescript
class Stack<T> {
  // ...

  pop(): T {
    if (this.isEmpty()) {
      throw new RangeError();
    }

    this.top -= 1;
    return this.contents[this.top];
  }
}
```

### 배열을 이용한 스택 전체 코드

```typescript
class Stack<T> {
  private contents: T[];
  private capacity: number;
  private top: number;

  constructor(capacity: number) {
    this.contents = [];
    this.capacity = capacity;
    this.top = 0;
  }

  isEmpty(): boolean {
    return top === 0;
  }

  isFull(): boolean {
    return top === capacity;
  }

  push(value: T): void {
    if (this.isFull()) {
      throw new RangeError();
    }

    this.contents[this.top] = value;
    this.top += 1;
  }

  pop(): T {
    if (this.isEmpty()) {
      throw new RangeError();
    }

    this.top -= 1;
    return this.contents[this.top];
  }
}
```

-----

## Linked List를 이용한 스택

### Linked List를 이용한 스택의 설계

배열을 이용한 스택에서는 배열의 끝 부분에서 `push`, `pop` 동작이 이루어졌다. Linked list에서는 시간 복잡도를 줄이기 위해 리스트의 시작 부분에서 `push`, `pop` 동작을 수행한다.

### Linked List를 이용한 스택의 초기화

Linked list를 초기화한다.

```typescript
class Stack<T> {
  private list: LinkedList<T>;

  constructor() {
    this.list = new LinkedList();
  }
}
```

### Linked List를 이용한 스택의 `isEmpty` 메서드

`isEmpty` 메서드는 리스트의 내용이 비었는지 검사하여 boolean 값을 반환한다. 여기서는 용량이 무한한 스택을 구현하므로 `isFull` 메서드는 필요하지 않다.

```typescript
class Stack<T> {
  // ...

  isEmpty(): boolean {
    return this.list.isEmpty();
  }
}
```

### Linked List를 이용한 스택의 `push` 메서드

주어진 값을 가진 노드를 리스트의 맨 앞에 추가한다.

```typescript
class Stack<T> {
  // ...

  push(value: T): void {
    this.list.insert(0, value);
  }
}
```

### Linked List를 이용한 스택의 `pop` 메서드

리스트의 맨 앞 노드의 값을 가져오고, 그 노드를 삭제한 뒤, 가져온 값을 반환한다.

```typescript
class Stack<T> {
  // ...

  pop(): T {
    const value: T = this.list.getValue(0);
    this.list.remove(0);
    return value;
  }
}
```

### Linked List를 이용한 스택 전체 코드

```typescript
class Stack<T> {
  private list: LinkedList<T>;

  constructor() {
    this.list = new LinkedList();
  }

  isEmpty(): boolean {
    return this.list.isEmpty();
  }

  push(value: T): void {
    this.list.insert(0, value);
  }

  pop(): T {
    const value: T = this.list.getValue(0);
    this.list.remove(0);
    return value;
  }
}
```
