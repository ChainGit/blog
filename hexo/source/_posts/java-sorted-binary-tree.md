---
title: Java实现自定义排序二叉树
date: 2017/09/22
categories:
- 学习
tags:
- 学习
- Java
- 数据结构
---

Java实现自定义排序二叉树
=======================
二叉树一直是数据结构中重头，这里我实现了二叉排序树（又叫二叉搜索树，二叉查找树，BST），和二叉树的前序遍历，中序遍历，后序遍历，层次遍历，镜像二叉树。

二叉查找树(Binary Search Tree)，它对于大多数情况下的查找和插入在效率上来说是没有问题的，但是它在最差的情况下效率比较低。平衡查找树(Balanced Search Tree)能够使得一棵具有N个节点的树中，该树的高度能够维持在lgN左右，这样就能保证只需要lgN次比较操作就可以查找到想要的值，比如AVL树、2-3查找树(2-3 Search Tree)、红黑树、B树、B+树、B*树、Trie树（字典树）等。

树的演变很多，应用也很广泛。

集合框架[源码](https://github.com/ChainGit/data-structure-learn/tree/master/Data-Structure-Test)。

2017年9月21日，更新：增加查找二叉排序树中最接近k的节点的值。

2017年9月22日，更新：增加二叉树的删除节点。

2017年9月23日，更新：增加镜像二叉树。

2017年9月30日，更新：增加二叉树遍历的循环做法，镜像二叉树的循环做法。

## 二叉树的抽象类

二叉树的抽象类如下：
```java
package com.chain.algorithm.test.day01;

import java.util.List;

public abstract class AbstractBinaryTree<T> implements BinaryTreeInter<T> {

	/**
	 * 构建排序二叉树
	 * 
	 */
	public abstract void add(T node);

	/**
	 * 删除二叉树的节点
	 * 
	 * @return
	 */
	public abstract boolean delete(T value);

	public abstract int size();

	public abstract boolean isEmpty();

	public abstract void clear();

	/**
	 * 广度优先遍历
	 * 
	 */
	public abstract List<T> bfsToList();

	/**
	 * 中序遍历
	 * 
	 * @return
	 */
	public abstract List<T> inOrderToList();

	/**
	 * 前序遍历
	 * 
	 * @return
	 */
	public abstract List<T> preOrderToList();

	/**
	 * 后序遍历
	 * 
	 * @return
	 */
	public abstract List<T> postOrderToList();

	/**
	 * 中序遍历(循环)
	 * 
	 * @return
	 */
	public abstract List<T> inOrderLoopToList();

	/**
	 * 前序遍历(循环)
	 * 
	 * @return
	 */
	public abstract List<T> preOrderLoopToList();

	/**
	 * 后序遍历(循环)
	 * 
	 * @return
	 */
	public abstract List<T> postOrderLoopToList();

}
```

## 二叉树的接口

BinaryTreeInterface：
```java
package com.chain.algorithm.test.day01;

public interface BinaryTreeInter<T> {

	/**
	 * 镜像二叉树
	 */
	public void mirror();

	/**
	 * 镜像二叉树(循环)
	 */
	public void mirrorLoop();

}
```

## 排序二叉树的实现

内部包含一个树节点的内部类。

```java
package com.chain.algorithm.test.day01;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.LinkedList;
import java.util.List;
import java.util.Map;

/**
 * 二叉树
 * 
 * @author Chain
 *
 * @param <T>
 */
public class BinaryTree<T> extends AbstractBinaryTree<T> implements BinaryTreeInter<T> {

	private BinaryTreeNode<T> rootNode;

	private int count;

	/**
	 * 构建排序二叉树
	 * 
	 */
	@Override
	public void add(T value) {
		if (rootNode == null) {
			rootNode = new BinaryTreeNode<>(value);
			count++;
			return;
		}

		BinaryTreeNode<T> node = new BinaryTreeNode<>(value);

		BinaryTreeNode<T> currentNode = rootNode;
		BinaryTreeNode<T> lastNode = currentNode;
		boolean flag = true;
		while (currentNode != null) {
			lastNode = currentNode;
			if (currentNode.compareTo(node) >= 0) {
				currentNode = currentNode.getLeft();
				flag = true;
			} else {
				currentNode = currentNode.getRight();
				flag = false;
			}
		}

		if (currentNode == null) {
			if (flag) {
				lastNode.setLeft(node);
			} else {
				lastNode.setRight(node);
			}
		}

		count++;
	}

	@Override
	public int size() {
		return count;
	}

	@Override
	public boolean isEmpty() {
		if (rootNode == null)
			return true;
		return false;
		// return count == 0;
	}

	@Override
	public void clear() {
		rootNode = null;
		count = 0;
	}

	/**
	 * 广度优先遍历
	 * 
	 */
	@Override
	public List<T> bfsToList() {
		List<T> lst = new ArrayList<>();
		if (isEmpty())
			return lst;

		AbstractQueue<BinaryTreeNode<T>> queue = new LinkedQueue<>();
		// AbstractQueue<BinaryTreeNode<T>> queue = new ArrayQueue<>();
		queue.push(rootNode);

		getLeftAndRightThenPushToQueue(lst, queue);

		return lst;
	}

	private void getLeftAndRightThenPushToQueue(List<T> lst, AbstractQueue<BinaryTreeNode<T>> queue) {
		int size = size();
		for (int i = 0; i < size; i++) {
			BinaryTreeNode<T> parentNode = queue.pull();
			lst.add(parentNode.getValue());
			BinaryTreeNode<T> leftNode = parentNode.getLeft();
			BinaryTreeNode<T> rightNode = parentNode.getRight();
			if (leftNode != null)
				queue.push(leftNode);
			if (rightNode != null)
				queue.push(rightNode);
		}
		if (!queue.isEmpty())
			getLeftAndRightThenPushToQueue(lst, queue);
	}

	protected static class BinaryTreeNode<T> implements Comparable<BinaryTreeNode<T>> {

		private T value;
		private BinaryTreeNode<T> left;
		private BinaryTreeNode<T> right;

		public BinaryTreeNode(T value) {
			super();
			this.value = value;
		}

		public T getValue() {
			return value;
		}

		public void setValue(T value) {
			this.value = value;
		}

		public BinaryTreeNode<T> getLeft() {
			return left;
		}

		public void setLeft(BinaryTreeNode<T> left) {
			this.left = left;
		}

		public BinaryTreeNode<T> getRight() {
			return right;
		}

		public void setRight(BinaryTreeNode<T> right) {
			this.right = right;
		}

		@Override
		public int hashCode() {
			final int prime = 31;
			int result = 1;
			result = prime * result + ((value == null) ? 0 : value.hashCode());
			return result;
		}

		@Override
		public boolean equals(Object obj) {
			if (obj == this)
				return true;
			if (obj == null)
				return false;
			if (obj.getClass() != BinaryTreeNode.class)
				return false;
			BinaryTreeNode node = (BinaryTreeNode) obj;
			return this.compareTo(node) == 0;
		}

		@Override
		public int compareTo(BinaryTreeNode<T> other) {
			if (other == null)
				throw new RuntimeException("node is null");

			T otherValue = other.getValue();

			if (otherValue instanceof Integer && value instanceof Integer)
				return ((Integer) value).compareTo((Integer) otherValue);

			return 0;
		}

		@Override
		public String toString() {
			return value.toString();
		}

	}

	@Override
	public List<T> inOrderToList() {
		List<T> lst = new ArrayList<>();
		inOrder(rootNode, lst);
		return lst;
	}

	private void inOrder(BinaryTreeNode<T> node, List<T> lst) {
		if (node == null)
			return;

		inOrder(node.getLeft(), lst);
		lst.add(node.getValue());
		inOrder(node.getRight(), lst);
	}

	@Override
	public List<T> preOrderToList() {
		List<T> lst = new ArrayList<>();
		preOrder(rootNode, lst);
		return lst;
	}

	private void preOrder(BinaryTreeNode<T> node, List<T> lst) {
		if (node == null)
			return;

		lst.add(node.getValue());
		preOrder(node.getLeft(), lst);
		preOrder(node.getRight(), lst);
	}

	@Override
	public List<T> postOrderToList() {
		List<T> lst = new ArrayList<>();
		postOrder(rootNode, lst);
		return lst;
	}

	private void postOrder(BinaryTreeNode<T> node, List<T> lst) {
		if (node == null)
			return;

		postOrder(node.getLeft(), lst);
		postOrder(node.getRight(), lst);
		lst.add(node.getValue());
	}

	@Override
	public boolean delete(T value) {
		// 二叉树删除节点总共有四种情况，需要考虑进根节点
		BinaryTreeNode<T> currentNode = rootNode;
		BinaryTreeNode<T> parentNode = rootNode;
		// 如果为空树
		if (currentNode == null || value == null)
			return false;

		BinaryTreeNode<T> findNode = new BinaryTreeNode<>(value);

		// 类似插入节点，从根节点依次往下找，找到这个节点或直到为空
		// 往左为true
		boolean flag = true;
		while (true) {
			// 往左走
			if (currentNode.compareTo(findNode) > 0) {
				parentNode = currentNode;
				currentNode = currentNode.getLeft();
				flag = true;
			}
			// 往右走
			else if (currentNode.compareTo(findNode) < 0) {
				parentNode = currentNode;
				currentNode = currentNode.getRight();
				flag = false;
			}
			// 相等
			else {
				break;
			}
			// 没有找到该节点
			if (currentNode == null) {
				return false;
			}
		}

		// System.out.println(currentNode.value);
		// System.out.println(parentNode.value);

		// 如果找到的节点为叶子节点，即没有左右孩子
		if (currentNode.getLeft() == null && currentNode.getRight() == null) {
			if (currentNode == rootNode) {
				rootNode = null;
			}
			// 是父节点的左孩子
			else if (flag) {
				parentNode.setLeft(null);
			}
			// 是父节点的右孩子
			else if (!flag) {
				parentNode.setRight(null);
			}
		}
		// 如果找到的节点有右孩子
		else if (currentNode.getLeft() == null && currentNode.getRight() != null) {
			BinaryTreeNode<T> right = currentNode.getRight();
			if (currentNode == rootNode) {
				rootNode = right;
			} else if (flag) {
				parentNode.setLeft(right);
			} else if (!flag) {
				parentNode.setRight(right);
			}
		}
		// 如果找到的节点有左孩子
		else if (currentNode.getLeft() != null && currentNode.getRight() == null) {
			BinaryTreeNode<T> left = currentNode.getLeft();
			if (currentNode == rootNode) {
				rootNode = left;
			} else if (flag) {
				parentNode.setLeft(left);
			} else if (!flag) {
				parentNode.setRight(left);
			}
		}
		// 如果找到的节点有两个孩子，找到中序后继节点
		else {
			BinaryTreeNode<T> successorNode = getSuccessor(currentNode);

			// 然后将currentNode替换成successorNode
			if (currentNode == rootNode) {
				rootNode = successorNode;
			} else if (flag) {
				parentNode.setLeft(successorNode);
			} else if (!flag) {
				parentNode.setRight(successorNode);
			}

			// 只可能有这种情况
			successorNode.setLeft(currentNode.getLeft());
		}

		count--;

		return true;
	}

	// 找到中序后继节点：在剩余子树中找出所有比被删除节点的值 大 的所有数，并在这些数中找出一个 最小 的数来
	private BinaryTreeNode<T> getSuccessor(BinaryTreeNode<T> deleteNode) {
		BinaryTreeNode<T> currentNode = deleteNode;
		// 右节点不为空，比要删除节点大的都在右子树
		BinaryTreeNode<T> successorNode = currentNode.getRight();
		BinaryTreeNode<T> successorParentNode = successorNode;
		while (currentNode != null) {
			successorParentNode = successorNode;
			successorNode = currentNode;
			// 最小的节点又都在左子树
			currentNode = currentNode.getLeft();
		}
		// 如果中序后继节点不是要删除的节点的右孩子
		if (successorNode != deleteNode.getRight()) {
			// 中序后继节点只可能有右孩子，将其放到中序后继的位置
			successorParentNode.setLeft(successorNode.getRight());
			// 中继后继的节点的右孩子（肯定存在）连接到要删除的节点的右孩子上（也肯定存在）防止出现游离状态而被回收
			successorNode.setRight(deleteNode.getRight());
		}
		return successorNode;
	}

	@Override
	public void mirror() {
		BinaryTreeNode<T> currentNode = rootNode;

		mirrorFun(currentNode);
	}

	private void mirrorFun(BinaryTreeNode<T> currentNode) {
		if (currentNode == null)
			return;

		// 交换当前节点的左右孩子节点，无论是否为空
		BinaryTreeNode<T> tempNode = currentNode.getLeft();
		currentNode.setLeft(currentNode.getRight());
		currentNode.setRight(tempNode);

		// 递归继续交换
		mirrorFun(currentNode.getLeft());
		mirrorFun(currentNode.getRight());
	}

	@Override
	public List<T> inOrderLoopToList() {
		List<T> lst = new ArrayList<>();
		inOrderLoop(lst);
		return lst;
	}

	// 递归也是栈操作，那么循环做法其实是手动栈，不会存在栈溢出的问题
	private void inOrderLoop(List<T> lst) {
		LinkedList<BinaryTreeNode<T>> stack = new LinkedList<>();
		BinaryTreeNode<T> currentNode = rootNode;
		while (currentNode != null || !stack.isEmpty()) {
			if (currentNode != null) {
				stack.push(currentNode);
				currentNode = currentNode.getLeft();
			} else {
				currentNode = stack.pop();
				lst.add(currentNode.getValue());
				currentNode = currentNode.getRight();
			}
		}
	}

	@Override
	public List<T> preOrderLoopToList() {
		List<T> lst = new ArrayList<>();
		preOrderLoop(lst);
		return lst;
	}

	private void preOrderLoop(List<T> lst) {
		LinkedList<BinaryTreeNode<T>> stack = new LinkedList<>();
		BinaryTreeNode<T> currentNode = rootNode;
		while (currentNode != null || !stack.isEmpty()) {
			if (currentNode != null) {
				stack.push(currentNode);
				lst.add(currentNode.getValue());
				currentNode = currentNode.getLeft();
			} else {
				currentNode = stack.pop();
				currentNode = currentNode.getRight();
			}
		}
	}

	@Override
	public List<T> postOrderLoopToList() {
		List<T> lst = new ArrayList<>();
		postOrderLoop(lst);
		return lst;
	}

	// 用于后序遍历的循环做法的标志
	// 1：访问过左节点 2：访问过右节点
	private static final int LEFT_VISITED = 1;
	// 后序遍历的情况下，如果为2则代表也访问过了左节点
	private static final int RIGHT_VISITED = 2;

	// 后序遍历的循环做法复杂一些，需要对原来的树的节点加上访问标记
	private void postOrderLoop(List<T> lst) {
		LinkedList<BinaryTreeNode<T>> stack = new LinkedList<>();
		BinaryTreeNode<T> currentNode = rootNode;
		Map<BinaryTreeNode<T>, Integer> flags = new HashMap<>(count);
		// 先访问左节点 ，再访问右节点，最后访问根节点
		while (currentNode != null || !stack.isEmpty()) {
			// 访问左节点
			while (currentNode != null) {
				stack.push(currentNode);
				flags.put(currentNode, LEFT_VISITED);
				currentNode = currentNode.getLeft();
			}
			// 左右子树均访问完毕，此时输出子树的根节点
			while (!stack.isEmpty() && (flags.get(stack.peek()) == RIGHT_VISITED)) {
				BinaryTreeNode<T> node = stack.pop();
				lst.add(node.getValue());
			}
			// 访问右节点
			if (!stack.isEmpty()) {
				currentNode = stack.peek();
				flags.put(currentNode, RIGHT_VISITED);
				currentNode = currentNode.getRight();
			}
		}
	}

	// 利用自定义栈代替程序栈
	@Override
	public void mirrorLoop() {
		if (rootNode == null)
			return;
		LinkedList<BinaryTreeNode<T>> stack = new LinkedList<>();
		stack.push(rootNode);
		BinaryTreeNode<T> currentNode = null;
		while (!stack.isEmpty()) {
			currentNode = stack.pop();

			BinaryTreeNode<T> tempNode = currentNode.getLeft();
			currentNode.setLeft(currentNode.getRight());
			currentNode.setRight(tempNode);

			BinaryTreeNode<T> leftNode = currentNode.getLeft();
			if (leftNode != null)
				stack.push(leftNode);

			BinaryTreeNode<T> rightNode = currentNode.getRight();
			if (rightNode != null)
				stack.push(rightNode);
		}

	}

}
```

## 测试代码

2017年9月21日，更新：增加查找二叉排序树中最接近k的节点的值。

2017年9月22日，更新：增加二叉树的删除节点。

2017年9月23日，更新：增加镜像二叉树。

2017年9月30日，更新：增加二叉树遍历的循环做法，镜像二叉树的循环做法。

源码[传送门](https://github.com/ChainGit/algorithm-learn/tree/master/chain/test01)

```java
package com.chain.algorithm.test.day01;

import java.util.List;

import org.junit.Test;

public class TreeTest {

	@Test
	public void testBinaryTree() {
		AbstractBinaryTree<Integer> tree = new BinaryTree<>();

		System.out.println(tree.size());
		System.out.println(tree.isEmpty());

		System.out.println(tree.bfsToList());
		System.out.println(tree.preOrderToList());
		System.out.println(tree.inOrderToList());
		System.out.println(tree.postOrderToList());

		System.out.println(tree.preOrderLoopToList());
		System.out.println(tree.inOrderLoopToList());
		System.out.println(tree.postOrderLoopToList());

		tree.add(32);
		tree.add(28);
		tree.add(56);
		tree.add(14);
		tree.add(25);
		tree.add(51);
		tree.add(68);
		tree.add(78);
		tree.add(44);
		tree.add(41);
		tree.add(40);
		tree.add(42);
		tree.add(72);
		tree.add(79);

		System.out.println(tree.size());
		System.out.println(tree.isEmpty());

		System.out.println(tree.bfsToList());
		System.out.println(tree.preOrderToList());
		System.out.println(tree.inOrderToList());
		System.out.println(tree.postOrderToList());

		// 二叉树的中序遍历输出的即是已经排序好的数组
		// 要求：查找二叉排序树的最接近k的节点

		int k = 24;
		List<Integer> list = tree.inOrderToList();

		int value = getClosest(list, k);
		System.out.println(value);

		// 测试二叉树的删除
		System.out.println(tree.delete(51));

		System.out.println(tree.size());
		System.out.println(tree.isEmpty());

		System.out.println(tree.bfsToList());
		System.out.println(tree.preOrderToList());
		System.out.println(tree.inOrderToList());
		System.out.println(tree.postOrderToList());

		// 镜像二叉树
		// 由顶而上，有大到小，利用递归实现
		tree.mirror();

		System.out.println(tree.size());
		System.out.println(tree.isEmpty());

		System.out.println(tree.bfsToList());
		System.out.println(tree.preOrderToList());
		System.out.println(tree.inOrderToList());
		System.out.println(tree.postOrderToList());

		// 镜像二叉树循环做法
		tree.mirrorLoop();

		System.out.println(tree.size());
		System.out.println(tree.isEmpty());

		System.out.println(tree.bfsToList());
		System.out.println(tree.preOrderToList());
		System.out.println(tree.inOrderToList());
		System.out.println(tree.postOrderToList());

		// 树的递归的循环做法
		System.out.println();
		System.out.println(tree.preOrderLoopToList());
		System.out.println(tree.inOrderLoopToList());
		System.out.println(tree.postOrderLoopToList());
	}

	// 查找二叉排序树的最接近k的节点
	private int getClosest(List<Integer> list, int k) {
		int size = list.size();
		for (int i = 0; i < size; i++) {
			int current = list.get(i);
			if (current == k) {
				return current;
			} else if (current > k) {
				if (i > 0) {
					int before = list.get(i - 1);
					int d1 = current - k;
					int d2 = k - before;
					// 如果相等的话就返回排在前面的值
					return d2 > d1 ? current : before;
				} else {
					return current;
				}
			}
		}
		return list.get(size - 1);
	}

}
```

测试结果：
```java
0
true
[]
[]
[]
[]
[]
[]
[]
14
false
[32, 28, 56, 14, 51, 68, 25, 44, 78, 41, 72, 79, 40, 42]
[32, 28, 14, 25, 56, 51, 44, 41, 40, 42, 68, 78, 72, 79]
[14, 25, 28, 32, 40, 41, 42, 44, 51, 56, 68, 72, 78, 79]
[25, 14, 28, 40, 42, 41, 44, 51, 72, 79, 78, 68, 56, 32]
25
true
13
false
[32, 28, 56, 14, 44, 68, 25, 41, 78, 40, 42, 72, 79]
[32, 28, 14, 25, 56, 44, 41, 40, 42, 68, 78, 72, 79]
[14, 25, 28, 32, 40, 41, 42, 44, 56, 68, 72, 78, 79]
[25, 14, 28, 40, 42, 41, 44, 72, 79, 78, 68, 56, 32]
13
false
[32, 56, 28, 68, 44, 14, 78, 41, 25, 79, 72, 42, 40]
[32, 56, 68, 78, 79, 72, 44, 41, 42, 40, 28, 14, 25]
[79, 78, 72, 68, 56, 44, 42, 41, 40, 32, 28, 25, 14]
[79, 72, 78, 68, 42, 40, 41, 44, 56, 25, 14, 28, 32]
13
false
[32, 28, 56, 14, 44, 68, 25, 41, 78, 40, 42, 72, 79]
[32, 28, 14, 25, 56, 44, 41, 40, 42, 68, 78, 72, 79]
[14, 25, 28, 32, 40, 41, 42, 44, 56, 68, 72, 78, 79]
[25, 14, 28, 40, 42, 41, 44, 72, 79, 78, 68, 56, 32]

[32, 28, 14, 25, 56, 44, 41, 40, 42, 68, 78, 72, 79]
[14, 25, 28, 32, 40, 41, 42, 44, 56, 68, 72, 78, 79]
[25, 14, 28, 40, 42, 41, 44, 72, 79, 78, 68, 56, 32]
```

部分内容摘自[伯乐在线](http://blog.jobbole.com/tag/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/)。

