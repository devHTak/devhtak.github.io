---
layout: post
title: 스터디 할래 4. (과제)자료구조
summary: 백기선님과 스터디 할래
author: devhtak
date: '2021-01-13 20:41:00 +0900'
category: Java Study
---

#### LinkedList를 구현하세요.

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

#### Stack을 구현하세요.

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

#### Queue를 구현하세요.

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

#### Binary Search Tree를 구현하시오.

- Binary Tree
  - 자식 노드가 최대 2개인 노드로 구성된 트리
    
- Binary Search Tree
  - Binary Tree에 left child < root < right child 로 구성되어야 한다.
  
- tree 순회
  - inorder : left -> root -> right 순으로 순회
  - preorder : root -> left -> right 순으로 순회
  - postorder : left -> right -> root 순으로 순회
  - root위치에 따라 in/pre/post가 나뉜다.
  
- dfs와 bfs
  - dfs: 깊이 우선 탐색으로 stack을 이용한다.
  - bfs: 너비 우선 탐색으로 queue를 이용한다.
  
- Binary Search Tree 구현
```java
public class BinarySearchTree {	
    private int index;
    private Node root;
    public BinarySearchTree() {
        root = null;
        index = 0;
    }
    public BinarySearchTree(int data) {
        root = new Node(data);
        index = 0;
    }
    public void insert(int data) {
        if(this.root == null) {
            this.root = new Node(data);
            return;
        }
        Node node = root;
        Node prev = new Node();
        while(node != null) {
            prev = node;
            if(node.getValue() > data) {
                node = node.getLeft();
            } else {
                node = node.getRight();
            }
        }
        if(node == prev.getLeft()) {
            prev.setLeft(new Node(data));
        } else {
            prev.setRight(new Node(data));
        }
    }

    public void remove(int data) {
        Node node = root;
        Node prev = new Node();
        while(node != null) {
            if(node.getValue() == data) {
                break;
            } else if(node.getValue() < data) {
                prev = node;
                node = node.getRight();
            } else {
                prev = node;
                node = node.getLeft();
            }
        }

        if(node.getLeft() == null && node.getRight() == null) {
            // 1. 지우고자 하는 노드에 자식이 없는 경우 -> 그냥 삭제
            if(node == root) {
                root = null;
            } else if(node == prev.getLeft()) {
                prev.setLeft(null);
            } else {
                prev.setRight(null);
            }
        } else if(node.getLeft() == null) { 
            // 2. 지우고자 하는 노드에 자식이 오른쪽에만 있는 경우 -> 해당 자식과 교체
            if(node == root) {
                root = node.getRight();
            } else if(node == prev.getLeft()) {
                prev.setLeft(node.getRight());
            } else {
                prev.setRight(node.getRight());
            }
        } else if(node.getRight() == null) { 
            // 2. 지우고자 하는 노드에 자식이 왼쪽에만 있는 경우 -> 해당 자식과 교체
            if(node == root) {
                root = node.getLeft();
            } else if(node == prev.getLeft()) {
                prev.setLeft(node.getLeft());
            } else {
                prev.setRight(node.getLeft());
            }
        } else { 
            // 3. 지우고자 하는 노드에 자식이 2개인 경우 -> 오른쪽 자식에 가장 왼쪽 자식을 대체하자
            // 오른쪽 자식에 가장 왼쪽 자식 찾기
            Node next = node.getRight();
            Node nextPrev = node;
            while(next.getLeft() != null) {
                nextPrev = next;
                next = next.getLeft();
            }
            // 대체할 노드가 삭제할 노드의 바로 오른쪽 노드가 아닌 경우,
            // 대체할 노드의 오른쪽 노드를 대체할 노드의 부모 왼쪽 노드로 대체하고, 대체할 노드의 오른쪽 자식을 삭제할 노드의 오른쪽 자식으로 세팅
            if(next != node.getRight()) {
                nextPrev.setLeft(next.getRight());
                next.setRight(node.getRight());
            }

            // 삭제할 대상과 대체
            if(node == root) {
                root = next;
            } else if(node == prev.getLeft()) {
                prev.setLeft(next);
            } else {
                prev.setRight(next);
            }
            // 삭제할 노드의 왼쪽, 오른쪽 자식은 그대로 이어준다.
            next.setLeft(node.getLeft());
        }		
    }

    public void dfs(Node root) {
        // inorder와 동일하기 때문에 구현하지 않음
    }

    public int[] bfs(int size) {
        int[] traversal = new int[size];
        this.index = 0;
        List<Node> queue = new ArrayList<>(); 
        queue.add(root);

        while(!queue.isEmpty()) {
            Node next = queue.get(0); queue.remove(0);			
            traversal[index++] = next.getValue();

            if(next.getLeft() != null) {
                queue.add(next.getLeft());
            }

            if(next.getRight() != null) {
                queue.add(next.getRight());
            }			
        }

        return traversal;
    }

    public void inOrder(Node node, int[] traversal) {
        // left -> root -> right;
        if(node == null) {
            return;
        }

        inOrder(node.getLeft(), traversal);
        traversal[index++] = node.getValue(); 
        inOrder(node.getRight(), traversal);
    }

    public void preOrder(Node node, int[] traversal) {
        // root -> left -> right;
        if(node == null)
            return;

        traversal[index++] = node.getValue();
        preOrder(node.getLeft(), traversal);
        preOrder(node.getRight(), traversal);
    }

    public void postOrder(Node node, int[] traversal) {
        // left -> right -> root
        if(node == null)
            return;
        postOrder(node.getLeft(), traversal);
        postOrder(node.getRight(), traversal);
        traversal[index++] = node.getValue();		
    }

    public int[] getInOrder(int size) {
        this.index = 0;
        int[] traversal = new int[size];
        inOrder(root, traversal);

        return traversal;
    }

    public int[] getPreOrder(int size) {
        this.index = 0;
        int[] traversal = new int[size];
        preOrder(root, traversal);

        return traversal;
    }

    public int[] getPostOrder(int size) {
        this.index = 0;
        int[] traversal = new int[size];
        postOrder(root, traversal);

        return traversal;
    }
}
```

- BinarySearchTree 테스트
```java
public class BinaryTreeTest {	
    private BinarySearchTree binarySearchTree;
    private int[] inorder = {1, 2, 3, 4, 5, 6, 7};
    private int[] preorder = {4, 2, 1, 3, 6, 5, 7};
    private int[] postorder = {1, 3, 2, 5, 7, 6, 4};	
    private int[] bfs = {4, 2, 6, 1, 3, 5, 7};
    private int[] bfsAfterRemove = {5, 3, 6, 7};

    @BeforeEach
    public void beforeEach() {
        this.binarySearchTree = new BinarySearchTree();
        this.binarySearchTree.insert(4);
        this.binarySearchTree.insert(2);
        this.binarySearchTree.insert(1);
        this.binarySearchTree.insert(3);
        this.binarySearchTree.insert(6);
        this.binarySearchTree.insert(5);
        this.binarySearchTree.insert(7);		
    }

    @Test
    public void removeTest() {
        // leaf node 삭제
        binarySearchTree.remove(1);

        // 자식이 1개인 node 삭제
        binarySearchTree.remove(2);

        // 자식이 2개인 node 삭제
        binarySearchTree.remove(4);

        int[] traversal = binarySearchTree.bfs(4);
        for(int i = 0; i < traversal.length; i++) {
            assertEquals(bfsAfterRemove[i], traversal[i]);
        }
    }

    @Test
    public void inOrderTest() {
        int[] traversal = this.binarySearchTree.getInOrder(inorder.length);

        for(int i = 0; i < traversal.length; i++) {
            assertEquals(inorder[i], traversal[i]);
        }
    }

    @Test
    public void preOrderTest() {
        int[] traversal = this.binarySearchTree.getPreOrder(preorder.length);

        for(int i = 0; i < traversal.length; i++) {
            assertEquals(preorder[i], traversal[i]);
        }
    }

    @Test
    public void postOrderTest() {
        int[] traversal = this.binarySearchTree.getPostOrder(postorder.length);

        for(int i = 0; i < traversal.length; i++) {
            assertEquals(postorder[i], traversal[i]);
        }
    }

    @Test
    public void dfsTest() {
        //inorder와 동일하기 때문에 구현하지 않음
    }

    @Test
    public void bfsTest() {
        int[] traversal = this.binarySearchTree.bfs(bfs.length);
        for(int i = 0; i < traversal.length; i++) {
            assertEquals(bfs[i], traversal[i]);
        }
    }
}
```
