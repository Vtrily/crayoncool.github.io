# 队列（数据结构与算法笔记）


队列在线程池等有限资源池中的应用

<!--more-->

## 队列

我们可以把队列比作是一个排队买东西的场景，先来的先买，后来的排队等待。

队列特性：先进先出

队列是一种操作受限的线性表数据结构

### 基于数组实现的队列

我们可以使用数组来实现一个顺序队列，利用两个指针分别指向队列头和队列尾。入队时增加队列尾，出队时增加队列头。我们看一下简单的实现

```java
public class ArrayQueue {
    private String[] items;
    private int capacity = 0;
    private int head = 0;
    private int tail = 0;
    public ArrayQueue(int capacity) {
        this.items = new String[capacity];
        this.capacity = capacity;
    }

    // 入队
    public boolean enqueue(String item) {
        if (tail == capacity) {
            return false;
        }
        items[tail] = item;
        tail++;
        return true;
    }

    // 出队
    public String dequeue() {
        if (head == tail) return null;
        String ret = items[head];
        head++;
        return ret;
    }
}

```

使用数组实现队列时，当head == tail队列为空，tail == capacity队列为满


以上代码在频繁的进行入队出队操作时，当tail移动到最后面时，数组还有空间但是已经不能进行入队操作。我们这时可以引入数组的扩容机制中的数据搬移来进行优化。当tail指针到队尾并且head不等于0时，我们进行一次数据搬移。

```java
public boolean enqueueSwapData(String item) {
    if (tail == capacity) {
        if (head == 0) {
            return false;
        }
        for (int i = head; i < tail; i++) {
            items[i - head] = items[i];
        }
        tail -= head;
        tail++;
    }
    items[tail] = item;
    tail++;
    return true;
}
```


### 基于链表实现的队列


```java
public class LinkedListQueue {

    private Node head;
    private Node tail;

    // 入队
    public void enqueue(String item) {
        if (tail == null) {
            Node newNode = new Node(item, null);
            head = newNode;
            tail = newNode;
        } else {
            tail.nextNode = new Node(item, null);
            tail = tail.nextNode;
        }
    }
    // 出队
    public String dequeue() {
        if (head == null) {
            return null;
        }
        String value = head.data;
        head = head.nextNode;
        if (head == null) {
            tail = null;
        }
        return value;
    }

    public static class Node {
        private String data;
        private Node nextNode;

        public Node(String data, Node nodeData) {
            this.data = data;
            this.nextNode = nodeData;
        }
    }
}
```

### 循环队列

用数组来实现队列时，会存在数据搬移的操作。我们可以使用循环队列我们可以避免数据搬移操作。

我们来实现循环队列时，需要特别注意队空和队满的判断条件。

