---
layout: post
title: 스터디 할래 4. (과제)자료구조
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-01-13 20:41:00 +0900'
category: Java Study
---

#### 과제 0. JUnit 5 학습하세요.

- 2월 중 TDD 실전법과 도구, 백기선님 강좌로 학습 예정

#### 과제 1. live-study 대시 보드를 만드는 코드를 작성하세요.

#### 과제 2. LinkedList를 구현하세요.

- ListNode 구현
  ```java
  public class ListNode {

      public int data;
      public ListNode next;

      public ListNode() {}
      public ListNode(int data) {
          this.data = data;
          this.next = null;
      }

      public ListNode add(ListNode head, ListNode nodeToAdd, int position) {
          ListNode node = head;
          for(int i = 0; i < position - 1; i++) {
              node = node.next;
          }

          nodeToAdd.next = node.next;
          node.next = nodeToAdd;

          return node;
      }

      public ListNode remove(ListNode head, int position) {
          ListNode node = head;

          // head를 삭제하는 경우
          if(position == 0) {
              ListNode deleteNode = node;
              head = node.next;
              deleteNode.next = null;
              return deleteNode;
          }

          for(int i = 0; i < position - 1; i++) {
              node = node.next;
          }

          ListNode deleteNode = node.next;
          node.next = deleteNode.next;

          return deleteNode;
      }

      public boolean contains(ListNode head, ListNode nodeToCheck) {
          ListNode node = head;

          while(node != null) {
              if(node.data == nodeToCheck.data) 
                  return true;
              node = node.next;
          }
          return false;
        }
  }
  ```
  
- ListNode 테스트

  ```java
  public class ListNodeTest {	
      private ListNode listNode;

      @BeforeEach
      public void beforeEach() {
          listNode = new ListNode(1);
          ListNode head = listNode;
          for(int i = 2; i < 5; i++) {
              ListNode nextNode = new ListNode(i);
              listNode.next = nextNode;
              listNode = listNode.next;
          }
          listNode = head;
      }

      @AfterEach
      public void afterEach() {
          while(listNode != null) {
              ListNode next = listNode.next;
              listNode = null;
              listNode = next;
          }
      }

      @Test
      public void showAll() {
          int i = 1;
          while(listNode != null) {
              assertEquals(i, listNode.data);
              i += 1;
              listNode = listNode.next;
          }
          assertEquals(i, 5);
      }

      @Test
      public void addTest() {
          ListNode newNode = new ListNode(5);
          listNode.add(listNode, newNode, 4);

          int i = 1;
          while(listNode != null) {
              assertEquals(i, listNode.data);
              i += 1;
              listNode = listNode.next;
          }
          assertEquals(i, 6);
      }

      @Test
      public void removeTest() {
          for(int i = 3; i >= 0; i--) {
              ListNode node = listNode.remove(listNode, i);
              assertEquals(node.data, i + 1);
          }
      }

      @Test
      public void containsTest() {
          for(int i = 1; i < 5; i++) {
              ListNode nodeToCheck = new ListNode(i);
              assertTrue(listNode.contains(listNode, nodeToCheck));
          }
      }
  }
  ```

#### 과제 3, 4. Stack을 구현하세요.

- 배열로 구현한 ArrayStack
  
  ```java
  public class ArrayStack {
      int maxSize;
      int index;
      int[] array;

      public ArrayStack() {
          this.maxSize = 10;
          this.index = 0;
          array = new int[maxSize];
      }

      public ArrayStack(int maxSize) {
          this.maxSize = maxSize;
          this.index = 0;
          array = new int[maxSize];
      }

      public void push(int data) {
          // 무제한으로 늘리자
          if(index >= maxSize) {
              maxSize += 5;
              int[] newArray = new int[maxSize];
              for(int i = 0; i < index; i++) {
                  newArray[i] = this.array[i];
              }
              this.array = newArray;
          }

          array[index++] = data;
      }

      public int pop() {
          if(index == -1) 
             return -1;

          int returnValue = this.array[--index];
          return returnValue;
      }
  }
  ```
  
- ArrayStack 테스트

  ```java
  public class ArrayStackTest {	
      @Test
      public void pushTest() {
          ArrayStack stack = new ArrayStack(5);

          for(int i = 1; i < 12; i++) {
              stack.push(i);
          }
          assertEquals(stack.index, 11);
          assertEquals(stack.maxSize, 15);
      }

      @Test
      public void popTest() {
          ArrayStack stack = new ArrayStack(10);

          for(int i = 1; i < 12; i++) {
              stack.push(i);
          }

          for(int i = 11; i > 0; i--) {
              assertEquals(stack.pop(), i);
          }
      }
  }
  
  ```

- ListNode로 구현한 ListStack

  ```java
  public class ListStack {	
      private ListNode head;
      private int size;

      public ListStack() {
          head = new ListNode();
          size = 0;
      }

      public void push(int nextValue) {
          if(size == 0) {
              head = new ListNode(nextValue);
              size += 1;
              return;
          } 
          ListNode nextToAdd = new ListNode(nextValue);
          head.add(head, nextToAdd, this.size++);
      }

      public int pop() {
          if(this.size() == 0) {
              return -1;
          }
          ListNode returnValue = head.remove(this.head, --this.size);
          return returnValue.data;
      }

      public int size() {
          return this.size;
      }
  }
  ```

- ListStack 테스트

  ```java
  public class ListStackTest {	
      @Test
      public void pushTest() {
          ListStack stack = new ListStack();

          for(int i = 1; i < 12; i++) {
              stack.push(i);
          }
          assertEquals(stack.size(), 11);
      }

      @Test
      public void popTest() {
          ListStack stack = new ListStack();

          for(int i = 1; i < 12; i++) {
              stack.push(i);
          }

          for(int i = 11; i > 0; i--) {
              assertEquals(stack.pop(), i);
          }
          assertEquals(stack.pop(), -1);
      }
  }
  ```

#### 과제 5. Queue를 구현하세요.

- 배열로 구현한 ArrayQueue

  ```java
  public class ArrayQueue {
      private int[] array;
      private int first;
      private int last;
      private int maxSize;

      public ArrayQueue() {
          maxSize = 10;
          array = new int[maxSize];
          first = 0;
          last = 0;
      }

      public void add(int num) {
          // 강제로 크기 변환
          if(this.last >= this.maxSize) {
              maxSize += 5;
              int[] newArray = new int[maxSize];
              int j = 0;
              for(int i = first; i < last; i++) {
                  newArray[j++] = array[i];
              }
              first = 0;
              last = j;
              this.array = newArray;
          }
          array[last++] = num;        
      }

      public int get() {
          if(first >= last)
              return -1;
          int returnValue = array[first];
          array[first++] = 0;
          return returnValue;
      }

      public boolean contain(int data) {
          for(int i = first; i < last; i++) {
            if(array[i] == data)
              return true;
          }
          return false;
      }

      public int size() {
          return last - first;
      }
  }
  ```
  
- ArrayQueue 테스트

  ```java
  public class ArrayQueueTest {
      @Test
      public void addTest() {
          ArrayQueue queue = new ArrayQueue();
          for(int i = 0; i < 12; i++) {
              queue.add(i + 1);
          }
          assertEquals(queue.size(), 12);
      }

      @Test
      public void getTest() {
          ArrayQueue queue = new ArrayQueue();
          for(int i = 0; i < 6; i++) {
              queue.add(i + 1);
          }
          assertEquals(queue.size(), 6);

          for(int i = 0; i < 6; i++) {
              assertEquals(queue.get(), (i+1));
          }
          assertEquals(queue.size(), 0);

          for(int i = 0; i < 6; i++) {
              queue.add(i + 1);
          }
          assertEquals(queue.size(), 6);

          for(int i = 0; i < 6; i++) {
              assertEquals(queue.get(), (i+1));
          }
          assertEquals(queue.size(), 0);
      }

      @Test
      public void containTest() {
          ArrayQueue queue = new ArrayQueue();
          for(int i = 0; i < 6; i++) {
              queue.add(i + 1);
          }
          assertEquals(queue.size(), 6);

          for(int i = 0; i< 6; i++) {
              assertTrue(queue.contain((i+1)));
          }
      }
  }
  ```
  
- ListNode로 구현한 ListQueue

  ```java
  public class ListQueue {	
      private ListNode head;
      private int index;
      public ListQueue() {
          head = new ListNode();
          index = 0;
      }
      public void add(int data) {
          ListNode now = new ListNode(data);
          head.add(head, now, ++index);
      }
      public int get() {
          if(index == 0) {
              return -1;
          }
          ListNode deleteNode = head.remove(head, 1);
          index -= 1;
          return deleteNode.data;
      }
      public boolean contain(int data) {
          ListNode node = head;

          while(node != null) {
              if(node.data == data)
                  return true;
              node = node.next;			
          }

          return false;
      }
      public int size() {
          return index;
      }
  }
  ```

- ListQueue 테스트

  ```java
  public class ListQueueTest {
      @Test
      public void addTest() {
          ListQueue queue = new ListQueue();
          for(int i = 0; i < 12; i++) {
              queue.add(i + 1);
          }
          assertEquals(queue.size(), 12);
      }
      @Test
      public void getTest() {
          ListQueue queue = new ListQueue();
          for(int i = 0; i < 6; i++) {
              queue.add(i + 1);
          }
          assertEquals(queue.size(), 6);

          for(int i = 0; i < 6; i++) {
              assertEquals(queue.get(), (i+1));
          }
          assertEquals(queue.size(), 0);
      }
      @Test
      public void containTest() {
          ListQueue queue = new ListQueue();
          for(int i = 0; i < 6; i++) {
              queue.add(i + 1);
          }
          assertEquals(queue.size(), 6);

          for(int i = 0; i< 6; i++) {
              assertTrue(queue.contain((i+1)));
          }
      }
  }
  ```
