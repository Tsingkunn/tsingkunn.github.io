## 单列集合
[[接口与抽象类|接口]]: Collection --> 子接口: List, Set
###### List
> 集合有序可重复
1. ArrayList: 数组实现.添加,是删除元素效率低,查找快
+ 构造方法: 默认大小是10
+ 遍历集合的方式
	+ for
	+ for-each
	+ iterator
+ ***不要用for-each的方式进行集合的添加与删除操作!***
1. LinkedList: 链表实现.添加,删除元素效率高,查找慢

###### Set
> 集合无序不重复
1. HashSet
2. TreeSet
## 双列集合
[[接口与抽象类|接口]]接口: Mapper
#### Map
1. HashMap
2. TreeMap
## 集合的简单实现
#### MyArrayList
###### 实现类
```java
import java.io.Serializable;  
import java.util.*;  
import java.util.function.Consumer;  
import java.util.function.Predicate;  
  
// Iterable<E> 可以实现迭代  
// Cloneable 可以使用.clone()方法;标记型接口,内部没有需要实现的抽象方法  
// Serializable可序列化;标记型接口,内部没有需要实现的抽象方法  
public class MyArrayList<E> implements Iterable<E>, Cloneable, Serializable {  
// Object空数组  
private static final Object[] EMPTY_LIST = {};  
  
	// 数组初始化默认大小 10
	private static final int DEFAULT_SIZE = 5;  
	  
	// 记录数组大小  
	private int size;  
	  
	// values 是一个动态数组,有些数据可能是空值,为null  
	// 没有序列化的必要  
	private transient Object[] values;  
	  
	// 空参构造函数,默认初始数组大小为10  
	public MyArrayList() {  
		values = new Object[DEFAULT_SIZE];  
		this.size = 0;  
	}  
	  
	// 一参构造函数,初始化时给定默认数组的大小值  
	public MyArrayList(int size) {  
		if (size < 0) {  
			throw new IllegalArgumentException("Index Out Bounds: " + size);  
		} else {  
			values = new Object[size];  
			this.size = 0;  
		}  
	}  
	  
	// 链式添加方法.  
	public MyArrayList<E> add(E e) {  
		// 静态方法Objects.requireNonNull是不能添加null元素进来  
		Objects.requireNonNull(e);  
		addValue(e);  
		return this;  
	}  
	  
	// 普通的添加方法,添加到结合的最后一位,注意不是内部实现数组的最后一位  
	public void endAdd(E e) {  
		// 静态方法Objects.requireNonNull是不能添加null元素进来  
		Objects.requireNonNull(e);  
		addValue(e);  
	}  
	  
	// 删除,无参删除,默认删除最后一位,并返回删除的元素  
	@SuppressWarnings("unchecked")  
	public E remove() {  
		Object obj;  
		if (size > 0) {  
			obj = values[size - 1];  
			delValue(size - 1);  
		} else {  
			throw new IllegalArgumentException("Index error.");  
		}  
		return (E) obj;  
	}  
	  
	// 根据下标删除元素,并返回删除的元素  
	@SuppressWarnings("unchecked")  
	public E remove(int index) {  
		// 静态方法Objects.checkIndex是检测下标是否越界的.  
		Objects.checkIndex(index, this.size);  
		Object obj = this.values[index];  
		  
		delValue(index);  
		return (E) obj;  
	}  
	  
	// 根据对象来删除元素,删除第一次遇到该值的元素  
	public boolean removeBy(E e) {  
		for (int index = 0; index < size; index++) {  
			if (values[index].equals(e)) {  
				delValue(index);  
				return true;  
			}  
		}  
		return false;  
	}  
	  
	// 断言式删除,根据给定条件来删除指定的全部元素  
	public void removeIf(Predicate<? extends E> predicate) {  
		for (int index = 0; index < size; index++) {  
			// 根据输入的断言执行判断  
			if (predicate.test(valueAt(values, index))) {  
			// 删除元素  
			delValue(index);  
			// 每次删除一个元素后都要调整下标  
			index--;  
			}  
		}  
	}  
	  
	// 更改指定下标元素的值,  
	public void set(int index, E e) {  
		// 禁止插入null元素  
		Objects.requireNonNull(e);  
		// 扩容  
		values = Arrays.copyOf(values, size + 1);  
		// 移动元素,然后size自增  
		System.arraycopy(values, index, values, index + 1, size++ - index);  
		// 插值  
		values[index] = e;  
	}  
	  
	// 获取指定下标的值  
	public E get(int index) {  
		// 判断下标是否越界  
		Objects.checkIndex(index, size);  
		// 返回值  
		return valueAt(values, index);  
	}  
	  
	// 清空数组,集合的长度变为0,是一个size为0的Object数组  
	public void clear() {  
		// values = {}  
		values = EMPTY_LIST;  
		// size 也要初始为 0
		size = 0;  
	}  
	  
	// 获取集合的元素个数  
	public int size() {  
		return size;  
	}  
	  
	// 如果size是0,则是一个空集合.  
	public boolean isEmpty() {  
		// size 是 0 就是空集合  
		return size == 0;  
	}  
	  
	// 返回遇到给定的第一个元素的下标  
	public int indexOf(E e) {  
		int index = 0;  
		// 如果e是null 但是我们已经禁止插入或修改为null了,所以没啥用.  
		if (e == null) {  
			for (; index < size; index++) {  
				if (values[index] == null) {  
					return index;  
				}  
			}  
		} else {  
			// 这个就是遍历,找到值  
			for (Object value : values) {  
				if (value.equals(e)) {  
					return index;  
				}  
					index++;  
			}  
		}  
		return -1;  
	}  
	  
	// 普通的删除某位的元素  
	private void delValue(int index) {  
		System.arraycopy(this.values, index + 1, this.values, index, size - index - 1);  
		this.values[size = size - 1] = null;  
	}  
	  
	// 末尾追加  
	private void addValue(E e) {  
		if (this.values.length == this.size) {  
		// 扩容  
		this.values = grow(size);  
		}  
		// 添加元素  
		this.values[size++] = e;  
	}  
	  
	// 某一个元素地址的值  
	@SuppressWarnings("unchecked")  
	private static <E> E valueAt(Object[] objects, int index) {  
		return (E) objects[index];  
	}  
	  
	// 数组的扩容  
	private Object[] grow(int size) {  
		if (size == 0) {  
			return new Object[DEFAULT_SIZE];  
		} else {  
			return Arrays.copyOf(values, values.length << 1);  
		}  
	}  
	  
	// 重写toString  
	@Override  
	public String toString() {  
		StringBuilder str = new StringBuilder("[");  
		for (int i = 0; i < size; i++) {  
			if (i < size - 1) {  
				str.append(values[i]).append(", ");  
			} else {  
				str.append(values[i]);  
			}  
		}  
		str.append("]");  
		return str.toString();  
	}  
	  
	// 迭代器,重写了hasNext, next, remove  
	@Override  
	@SuppressWarnings("unchecked")  
	public Iterator<E> iterator() {  
		return new Iterator<>() {  
			private int cursor = -1;  
			  
			@Override  
			public boolean hasNext() {  
				return cursor + 1 < size;  
			}  
			  
			@Override  
			public E next() {  
				// next 就是游标下移动.默认是-1,所以开始刚进来就++,编程0,从第一位开始  
				cursor++;  
				// 返回当前值  
				return (E) values[cursor];  
			}  
			  
			@Override  
			public void remove() {  
				// 简单的删除当前元素,就是数组中元素的移动  
				delValue(cursor);  
				// 不要忘了让游标归位  
				cursor--;  
			}  
		};  
	}  
	  
	// 重写foreach,实现每隔一个消费一个  
	@Override  
	public void forEach(Consumer<? super E> action) {  
		Objects.requireNonNull(action);  
		// 这里让 i 每次自增2  
		for (int i = 0; i < size; i = i + 2) {  
			// 将元素消费掉,accept()内的元素就会被消费掉  
			// 这里的 valueAt(values, i) 就是 (E) values[i] ,获取数组中的元素,转换成传进来的泛型类型  
			action.accept(valueAt(values, i));  
		}  
	}  
	  
	// 重写hashCode()方法  
	@Override  
	public int hashCode() {  
		Object[] es = values;  
		  
		if (size > es.length) {  
			throw new ConcurrentModificationException();  
		}  
		int hashCode = 1;  
		for (int i = 0; i < size; i++) {  
			Object obj = es[i];  
			hashCode = 31 * hashCode + (obj == null ? 0 : obj.hashCode());  
		}  
		return hashCode;  
	}  
	  
	// 重写equals()方法  
	@Override  
	public boolean equals(Object obj) {  
		// 地址相同,直接返回true  
		if (this == obj) {  
			return true;  
		}  
		// 类型不同,直接返回false  
		if (!(obj instanceof MyArrayList<?>)) {  
			return false;  
		}  
		// 长度不同,直接返回false  
		if (((MyArrayList<?>) obj).size != size) {  
			return false;  
		}  
		// 数组长度  
		boolean equal = true;  
		// 当前数组  
		final Object[] es = this.values;  
		// 参数数组,要比较的数组  
		final Object[] esOther = ((MyArrayList<?>) obj).values;  
		  
		// 如果有一个元素不同,将equal标记为false,break.  
		for (int i = 0; i < size; i++) {  
			if (!es[i].equals(esOther[i])) {  
				equal = false;  
				break;  
			}  
		}  
		// 返回是否相等的结果  
		return equal;  
	}  
	  
	// 浅克隆  
	// 所谓浅克隆,就是新建了一个引用,指向了  
	@Override  
	@SuppressWarnings("unchecked")  
	protected MyArrayList<E> clone() throws CloneNotSupportedException {  
		return (MyArrayList<E>) super.clone();  
	}  
}
```
###### 测试类
```java
import java.util.ArrayList;  
import java.util.Iterator;  
  
public class MyArrayListTest {  
	public static void main(String[] args) {  
		MyArrayList<Integer> myArrayList1 = new MyArrayList<>();  
		  
		// 初始化数组,字符串 1~9
		for (int i = 0; i < 10; i++) {  
			myArrayList1.add(i);  
		}  
		//打印集合1的初始值  
		System.out.println("集合1的初始值: \t\t" + myArrayList1);  
		  
		// 迭代器  
		// hasNext() / next() / remove()  
		// 验证迭代器,遇到3跳出循环  
		Iterator<Integer> it = myArrayList1.iterator();  
		System.out.print("遇3删除并跳出迭代器循环:");  
		found:  
		{  
			while (it.hasNext()) {  
				int num = it.next();  
				if (num == 3) {  
					// 重写了迭代器的remove  
					it.remove();  
					// break的标签写法  
					break found;  
				} else {  
					System.out.print(num + " ");  
				}  
			}  
		}  
		System.out.println("\n迭代器删除了'3'\t\t" + myArrayList1);  
		  
		// 删除  
		// 删除最后一个  
		Integer num1 = myArrayList1.remove();  
		System.out.println("删除最后一个元素:\t\t" + num1);  
		  
		// 根据下标删除  
		Integer num2 = myArrayList1.remove(3);  
		System.out.println("删除下标[3]的元素: \t" + num2);  
		  
		// 根据对象删除  
		boolean isRemove = myArrayList1.removeBy(6);  
		System.out.println("删除值'6'是否成功: \t" + (isRemove ? "是" : "否"));  
		  
		// 处理后的集合数据  
		System.out.println("集合1数组长度是:\t\t" + myArrayList1.size());  
		System.out.println("集合1数组元素是:\t\t" + myArrayList1);  
		  
		// 初始化第二个数组,指定初始化数组大小为20  
		MyArrayList<Integer> myArrayList2 = new MyArrayList<>(20);  
		myArrayList2.add(0).add(1).add(2).add(5).add(7).endAdd(8);  
		  
		// 打印初始值  
		System.out.println("集合2的元素:\t\t\t" + myArrayList2);  
		  
		// 验证equals()与hashCode()函数是否匹配  
		System.out.println("两个集合是否相等:\t\t" + ((myArrayList2.equals(myArrayList1)) ? "是" : "否"));  
		System.out.println("集合1的hashCode():\t" + myArrayList1.hashCode());  
		System.out.println("集合2的hashCode():\t" + myArrayList2.hashCode());  
		  
		// 插入'12'在下标0的位置  
		myArrayList1.set(0, 12);  
		System.out.println("集合1[0]位置插入 12:\t" + myArrayList1);  
		  
		// 查找元素'12'第一次出现时的下标  
		int index = myArrayList1.indexOf(12);  
		System.out.println("集合1 12 的下标:\t\t" + index);  
		  
		  
		// 添加重复元素,验证removeIf函数是否有效  
		for (int i = 0; i < 10; i++) {  
			myArrayList1.add(1);  
		}  
		System.out.println("集合1数组元素是:\t\t" + myArrayList1);  
		myArrayList1.removeIf(integer -> integer == 1);  
		System.out.println("集合1断言等于1的都删除:\t" + myArrayList1);  
		  
		// 清空数组  
		myArrayList1.clear();  
		System.out.println("清空集合: \t\t\t" + myArrayList1);  
		  
		for (int i = 0; i < 10; i++) {  
			myArrayList1.endAdd(i);  
		}  
		// 打印  
		// 重写了foreach,  
		System.out.println("集合1数组元素是:\t\t" + myArrayList1);  
		System.out.print("集合1重写了foreach打印:");  
		myArrayList1.forEach(System.out::print);   
	}  
}
```

###### 输出
```html
集合1的初始值: 		[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
遇3删除并跳出迭代器循环:0 1 2 
迭代器删除了'3'		[0, 1, 2, 4, 5, 6, 7, 8, 9]
删除最后一个元素:		9
删除下标[3]的元素: 	4
删除值'6'是否成功: 	是
集合1数组长度是:		6
集合1数组元素是:		[0, 1, 2, 5, 7, 8]
集合2的元素:			[0, 1, 2, 5, 7, 8]
两个集合是否相等:		是
集合1的hashCode():	888491814
集合2的hashCode():	888491814
集合1[0]位置插入 12:	[12, 0, 1, 2, 5, 7, 8]
集合1 12 的下标:		0
集合1数组元素是:		[12, 0, 1, 2, 5, 7, 8, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
集合1断言等于1的都删除:	[12, 0, 2, 5, 7, 8]
清空集合: 			[]
集合1数组元素是:		[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
集合1重写了foreach打印:02468
```

#### MyLinkedList
###### 实现类
```java
import java.io.Serializable;  
import java.util.Iterator;  
import java.util.Objects;  
import java.util.function.Consumer;  
import java.util.function.Predicate;  
  
public class MyLinkedList<E> implements Iterable<E>, Cloneable, Serializable {  
  
	private transient int size = 0;  
	private transient Node<E> head;  
	private transient Node<E> last;  
	  
	public MyLinkedList() {  
	}  
	  
	public void addFirst(E e) {  
		linkNode(head, e, 0);  
	}  
	  
	public void addLast(E e) {  
		linkNode(last, e, size);  
	}  
	  
	public void add(E e) {  
		linkNode(last, e, size);  
	}  
	  
	public void add(int index, E e) {  
		checkIndex(index, size);  
		linkNode(getNode(index), e, index);  
	}  
	  
	public boolean removeFirst() {  
		unlinkNode(head);  
		return true;  
	}  
	  
	public boolean removeLast() {  
		unlinkNode(last);  
		return true;  
	}  
	  
	public E remove(int index) {  
		checkIndex(index, size);  
		E e = null;  
		Node<E> node = head;  
		for (int i = 0; i < size; i++) {  
			if (i == index) {  
				e = unlinkNode(node);  
				return e;  
			}  
			node = node.next;  
		}  
		return e;  
	}  
	  
	public boolean removeBy(E e) {  
		Node<E> node = getNode(e);  
		unlinkNode(node);  
		return true;  
	}  
	  
	public boolean removeIf(Predicate<E> predicate) {  
		Node<E> node = head;  
		while (node != null) {  
			if (predicate.test(node.item)) {  
				unlinkNode(node);  
			}  
			node = node.next;  
		}  
		return true;  
	}  
	  
	public boolean set(int index, E e) {  
		checkIndex(index, size);  
		Node<E> node = getNode(index);  
		node.item = e;  
		return true;  
	}  
	  
	public E get(int index) {  
		checkIndex(index, size);  
		return getNode(index).item;  
	}  
	  
	public void clear() {  
		for (int i = size; i > 0; i--) {  
			unlinkNode(head);  
		}  
	}  
	  
	public boolean offerFirst(E e) {  
		addFirst(e);  
		return true;  
	}  
	  
	public boolean offerLast(E e) {  
		addLast(e);  
		return true;  
	}  
	  
	public E peekFirst() {  
		return head.item;  
	}  
	  
	public E peekLast() {  
		return last.item;  
	}  
	  
	public E pollFirst() {  
		return unlinkNode(head);  
	}  
	  
	public E pollLast() {  
		return unlinkNode(last);  
	}  
	  
	public void push(E e) {  
		addFirst(e);  
	}  
	  
	public E pop() {  
		checkIndex(0, size);  
		return remove(0);  
	}  
	  
	public E element() {  
		if (head == null) 
			return null;  
		return getNode(0).item;  
	}  
	  
	public int indexOf(E e) {  
		return getIndex(e);  
	}  
	  
	public int size() {  
		return size;  
	}  
	  
	  
	// 根据节点删除元素,删除成功返回元素值  
	private E unlinkNode(Node<E> node) {  
		// 不接受null  
		Objects.requireNonNull(node);  
		  
		// 存返回值  
		E e = node.item;  
		// 当前节点的上一个节点  
		Node<E> prev = node.prev;  
		// 当前节点的下一个节点  
		Node<E> next = node.next;  
		// 删除当前节点元素的数据数据  
		node.item = null;  
		  
		// 如果就只有这么一个元素  
		if (prev == null && next == null) {  
				// 那么删除了这个元素,就是空链表,头,尾,全空.  
				head = last = null;  
			// 如果他在链表头  
			} else if (prev == null) {  
				// 下一个变成头  
				head = next;  
				// 让下一个的前一个节点是空.  
				next.prev = null;  
			// 如果再链表尾  
			} else if (next == null) {  
				// 那么他前一个版链表尾部  
				last = prev;  
				prev.next = null;  
			} else {  
				// 拼接节点  
				prev.next = next;  
				next.prev = prev;  
				// 清空该节点  
				node.prev = null;  
				node.next = null;  
			}  
		size--;  
		return e;  
	}  
	  
	private void linkNode(Node<E> node, E e, int index) {  
		  
		// 下标是0,头插法  
		if (index == 0) {  
			// 获取当前头部节点信息,储存到temp  
			Node<E> temp = head;  
			// 创建新节点,prev指向空,next是原来的head节点  
			Node<E> newNode = new Node<>(null, e, temp);  
			// 新节点newNode为新的head节点  
			head = newNode;  
			// 如果是第一次创建,也要初始化last节点的值,让last指向刚刚新创建的节点  
			if (temp == null) {  
				last = newNode;  
			} else {  
				// 如果链表里有节点,让原来的head的节点的prev指向新建的newNode节点  
				temp.prev = newNode;  
			}  
		} else if (index == size) {  
			// 保存当前节点  
			Node<E> temp = last;  
			// 新建节点newNode,prev指向原来的last,next指向空  
			Node<E> newNode = new Node<>(last, e, null);  
			// 新节点newNode为新的last节点  
			last = newNode;  
			// 如果是第一次创建,也要初始化head节点的值,让head指向刚刚新创建的节点  
			if (temp == null) {  
				head = newNode;  
			} else {  
				// 如果链表里有节点,让原来的last节点的next指向新建节点newNode节点  
				temp.next = newNode;  
			}  
		} else {  
			// 如果是根据下标插值,先新建节点,  
			// 其prev指向原下标节点node的prev  
			// 其next指向原下标节点node  
			Node<E> newNode = new Node<>(node.prev, e, node);  
			// 让node的前一个节点的值的下一个节点指向newNode  
			// 一定先让原节点的前一个节点的next先指向newNode.  
			node.prev.next = newNode;  
			// 让node的prev指向newNode  
			node.prev = newNode;  
		}  
		size++;  
	}  
	  
	private Node<E> getNode(int index) {  
		Node<E> node = head;  
		for (int i = 0; i < size; i++) {  
			if (i == index) {  
				return node;  
			}  
			node = node.next;  
		}  
		return node;  
	}  
	  
	private Node<E> getNode(E e) {  
		Node<E> node = head;  
		for (int i = 0; i < size; i++) {  
			if (Objects.equals(e, node.item)) {  
				return node;  
			}  
			node = node.next;  
		}  
		return node;  
	}  
	  
	private int getIndex(E e) {  
		Node<E> node = head;  
		for (int i = 0; i < size; i++) {  
			if (Objects.equals(e, node.item)) {  
				return i;  
			}  
			node = node.next;  
		}  
		return -1;  
	}  
	  
	private void checkIndex(int index, int size) {  
		if (index < 0 || index > size) {  
			throw new IndexOutOfBoundsException();  
		}  
	}  
	  
	@Override  
	public Iterator<E> iterator() {  
		return new Iterator<>() {  
			private Node<E> node = new Node<>(null, null, head);  
		  
			@Override  
			public boolean hasNext() {  
				return node.next != null;  
			}  
			  
			@Override  
			public E next() {  
				node = node.next;  
				return node.item;  
			}  
			  
			@Override  
			public void remove() {  
				// 这里需要新开辟一块空间,用来存储node的节点.  
				// 当执行unlinkNode是,node的所用内容都将变为null  
				// 当执行完毕之后将temp交给node节点.  
				// 为什么交给了当前的节点,这个就相当于 游标--,的操作,就像最开始赋值是head的头一个  
				// 因为进入next就直接node=next节点了,所以要给node赋值下一个节点是原来未删除时node的节点,只需要保留node.next就好  
				Node<E> temp = new Node<>(null, null, node.next);  
				unlinkNode(node);  
				node = temp;  
			}  
		};  
	}  
	  
	// 重写foreach  
	@Override  
	public void forEach(Consumer<? super E> action) {  
		Objects.requireNonNull(action);  
		Node<E> temp = last;  
		while (temp != null) {  
			action.accept(temp.item);  
			temp = temp.prev;  
		}  
	}  
	  
	@Override  
	@SuppressWarnings("unchecked")  
	public MyLinkedList<E> clone() throws CloneNotSupportedException {  
		return (MyLinkedList<E>) super.clone();  
	}  
	  
	@Override  
	public String toString() {  
		StringBuilder sb = new StringBuilder("[");  
		if (head == null) {  
			sb.append("]");  
		} else {  
			Node<E> node = head;  
			while (node != last) {  
			sb.append(node.item).append(", ");  
			node = node.next;  
		}  
			sb.append(last.item).append("]");  
		}  
		return sb.toString();  
	}  
	  
	private static class Node<E> {  
		E item;  
		Node<E> prev;  
		Node<E> next;  
		  
		public Node(Node<E> prev, E item, Node<E> next) {  
			this.prev = prev;  
			this.item = item;  
			this.next = next;  
		}  
		  
		@Override  
		public String toString() {  
			return "[" + item + "]";  
		}  
	}  
}
```
###### 测试类
```java
package practice.array;  
  
import java.util.Iterator;  
import java.util.LinkedList;  
import java.util.List;  
  
public class MyListedListTest {  
	public static void main(String[] args) {  
		System.out.println("链表模拟-------------------------------------------");  
		MyLinkedList<Integer> list = new MyLinkedList<>();  
		for (int i = 0; i < 10; i++) {  
			list.add(i);  
		}  
		// 链表初始值  
		System.out.println("链表的值为:\t\t\t" + list);  
		System.out.println("链表长度为:\t\t\t" + list.size());  
		  
		// 头删除  
		boolean b1 = list.removeFirst();  
		System.out.println("删除头元素:\t\t\t" + (b1 ? "成功" : "失败"));  
		  
		// 尾删除  
		boolean b2 = list.removeLast();  
		System.out.println("删除尾元素:\t\t\t" + (b2 ? "成功" : "失败"));  
		  
		// 打印结果  
		System.out.println("链表的值为:\t\t\t" + list);  
		  
		// 删除下标'5'的元素  
		Integer i1 = list.remove(5);  
		System.out.println("删除下标'5'的元素:\t\t" + (i1));  
		  
		// 打印结果  
		System.out.println("链表的值为:\t\t\t" + list);  
		  
		// 删除数值'7'的元素  
		boolean b4 = list.removeBy(7);  
		System.out.println("删除数值'7'的元素:\t\t" + (b4 ? "成功" : "失败"));  
		  
		list.add(3, 18);  
		list.add(6, 18);  
		list.add(0, 18);  
		// 打印结果  
		System.out.println("链表随机插入值'18'*3:\t" + list);  
		boolean b5 = list.removeIf(value -> value == 18);  
		System.out.println("断言值为'18'全部删除:\t" + (b5 ? "成功" : "失败") + "\t 链表值:" + list);  
		// 链表操作之后长度确认  
		System.out.println("链表长度为:\t\t\t" + list.size());  
		  
		// 修改下标[3]的元素,更改其值为99  
		boolean b6 = list.set(3, 99);  
		System.out.println("修改[3]的值为99:\t\t" + (b6 ? "成功" : "失败") + " 队列值为:" + list);  
		  
		// '3'的下标是  
		int index = list.indexOf(3);  
		System.out.println("值'3'的下标是:\t\t" + index);  
		  
		// 最后结果  
		System.out.println("链表的值为:\t\t\t" + list);  
		  
		// 模拟双向队列  
		System.out.println("双向队列模拟-------------------------------------------");  
		// 头入队  
		boolean b7 = list.offerFirst(99);  
		System.out.println("队列头插'99':\t\t" + (b7 ? "成功" : "失败") + " 队列值为:" + list);  
		  
		// 尾入队  
		boolean b8 = list.offerLast(98);  
		System.out.println("队列尾插'98':\t\t" + (b8 ? "成功" : "失败") + " 队列值为:" + list);  
		  
		// 检索头尾  
		Integer i2 = list.peekFirst();  
		System.out.println("队列头部检索值为:\t" + i2 + "\t队列值为:" + list);  
		Integer i3 = list.peekLast();  
		System.out.println("队列尾部检索值为:\t" + i3 + "\t队列值为:" + list);  
		  
		// 头尾出队列  
		Integer i4 = list.pollFirst();  
		System.out.println("头部出队列值:\t\t" + i4 + "\t队列值为:" + list);  
		Integer i5 = list.pollLast();  
		System.out.println("尾部出队列值:\t\t" + i5 + "\t队列值为:" + list);  
		  
		System.out.println("模拟栈-------------------------------------");  
		  
		// 入栈  
		list.push(66);  
		list.push(66);  
		list.push(66);  
		list.push(66);  
		System.out.println("连续四次入栈'66',值为:\t" + list);  
		  
		// 出栈  
		list.pop();  
		list.pop();  
		Integer i6 = list.pop();  
		System.out.println("第三次出栈值为:\t" + i6 + "\t栈值为:" + list);  
		  
		// 查询栈顶元素  
		Integer i7 = list.element();  
		System.out.println("查询栈顶元素值为:\t" + i7 + "\t栈值为:" + list);  
		  
		//重写foreach  
		System.out.print("重写for-each,从尾至头遍历: ");  
		list.forEach(v -> System.out.print(v + " "));  
		System.out.println();  
		  
		System.out.println("链表元素:\t" + list);  
		System.out.println("链表长度:\t" + list.size());  
		  
		list.addLast(99);  
		list.addLast(99);  
		list.addLast(99);  
		list.addLast(99);  
		list.addLast(99);  
		System.out.println("链表元素:\t" + list);  
		// 迭代器  
		Iterator<Integer> it = list.iterator();  
		// 迭代器迭代输出  
		while (it.hasNext()) {  
			int num = it.next();  
			if (num == 99) {  
				it.remove();  
			}  
		}  
		  
		System.out.println("\n利用迭代器删除元素'99' ↓");  
		System.out.println("链表元素:\t" + list);  
		System.out.println("链表长度:\t" + list.size());  
		  
		list.clear();  
		System.out.println("清空元素:\t" + list);  
	}  
}
```

###### 输出
```html
链表模拟-------------------------------------------
链表的值为:			[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
链表长度为:			10
删除头元素:			成功
删除尾元素:			成功
链表的值为:			[1, 2, 3, 4, 5, 6, 7, 8]
删除下标'5'的元素:	6
链表的值为:			[1, 2, 3, 4, 5, 7, 8]
删除数值'7'的元素:	成功
链表随机插入值'18'*3:[18, 1, 2, 3, 18, 4, 5, 18, 8]
断言值为'18'全部删除:成功	 链表值:[1, 2, 3, 4, 5, 18, 8]
链表长度为:			7
修改[3]的值为99:		成功 队列值为:[1, 2, 3, 99, 5, 18, 8]
值'3'的下标是:		2
链表的值为:			[1, 2, 3, 99, 5, 18, 8]
双向队列模拟-------------------------------------------
队列头插'99':		成功 队列值为:[99, 1, 2, 3, 99, 5, 18, 8]
队列尾插'98':		成功 队列值为:[99, 1, 2, 3, 99, 5, 18, 8, 98]
队列头部检索值为:	99	 队列值为:[99, 1, 2, 3, 99, 5, 18, 8, 98]
队列尾部检索值为:	98	 队列值为:[99, 1, 2, 3, 99, 5, 18, 8, 98]
头部出队列值:		99 	 队列值为:[1, 2, 3, 99, 5, 18, 8, 98]
尾部出队列值:		98	 队列值为:[1, 2, 3, 99, 5, 18, 8]
模拟栈-------------------------------------
连续四次入栈'66':	    栈值为:[66, 66, 66, 66, 1, 2, 3, 99, 5, 18, 8]
第三次出栈值为:	    66	栈值为:[66, 1, 2, 3, 99, 5, 18, 8]
查询栈顶元素值为:	66	栈值为:[66, 1, 2, 3, 99, 5, 18, 8]
重写for-each,从尾至头遍历: 8 18 5 99 3 2 1 66 
链表元素:	[66, 1, 2, 3, 99, 5, 18, 8]
链表长度:	8
链表元素:	[66, 1, 2, 3, 99, 5, 18, 8, 99, 99, 99, 99, 99]
利用迭代器删除元素'99'  ↓
链表元素:	[66, 1, 2, 3, 5, 18, 8]
链表长度:	7
清空元素:	[]
```

#### MyHashMap
###### 实现类
```java
```
###### 测试类
```java

```
