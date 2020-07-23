# Map在遍历过程中删除元素

HashMap之快速失败
为什么HashMap通过迭代器自身的remove或add方法就不会出现迭代器失败？

HashMap所有集合类视图所返回迭代器都是快速失败（fast-fail）的。

在HashMap中，有一个变量modCount来指示集合被修改的次数。在创建Iterator迭代器的时候，会给这个变量赋值给expectedModCount。当集合方法修改集合元素时，例如集合的remove()方法时，此时会修改modCount值，但不会同步修改expectedModCount值。当使用迭代器遍历元素操作时，会首先对比expectedModCount与modCount是否相等。如果不相等，则马上抛出java.util.ConcurrentModificationException异常。而通过Iterator的remove()方法移除元素时，会同时更新expectedModCount的值，将modCount的值重新赋值给expectedModCount，这样下一次遍历时，就不会发抛出ava.util.ConcurrentModificationException异常。

具体从代码来看。

首先看HashMap的remove方法：

public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

 * ```java
 /**

- Implements Map.remove and related methods
  *
- @param hash hash for key
- @param key the key
- @param value the value to match if matchValue, else ignored
- @param matchValue if true only remove if value is equal
- @param movable if false do not move other nodes while removing
- @return the node, or null if none
  */
  final Node<K,V> removeNode(int hash, Object key, Object value,
                         boolean matchValue, boolean movable) {
  Node<K,V>[] tab; Node<K,V> p; int n, index;
  if ((tab = table) != null && (n = tab.length) > 0 &&
      (p = tab[index = (n - 1) & hash]) != null) {
      Node<K,V> node = null, e; K k; V v;
      if (p.hash == hash &&
          ((k = p.key) == key || (key != null && key.equals(k))))
          node = p;
      else if ((e = p.next) != null) {
          if (p instanceof TreeNode)
              node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
          else {
              do {
                  if (e.hash == hash &&
                      ((k = e.key) == key ||
                       (key != null && key.equals(k)))) {
                      node = e;
                      break;
                  }
                  p = e;
              } while ((e = e.next) != null);
          }
      }
      if (node != null && (!matchValue || (v = node.value) == value ||
                           (value != null && value.equals(v)))) {
          if (node instanceof TreeNode)
              ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
          else if (node == p)
              tab[index] = node.next;
          else
              p.next = node.next;
          //在真正移除节点时，修改modCount的值
          ++modCount;
          --size;
          afterNodeRemoval(node);
          return node;
      }
  }
  return null;
  }

 /**

- The number of times this HashMap has been structurally modified
- Structural modifications are those that change the number of mappings in
- the HashMap or otherwise modify its internal structure (e.g.,
- rehash).  This field is used to make iterators on Collection-views of
- the HashMap fail-fast.  (See ConcurrentModificatio
nException).
  */
transient int modCount;
```

//指示HashMap被修改的次数当通过remove移除HashMap中的一个元素时，会修改modCount值，其他修改HashMap集合的方法也会修改modCount值。该值在创建迭代器的时候，会赋值给expectedModCount，在迭代器工作的时候，会判定检查modCount值是否修改了。如果该值被修改了，则抛出ConcurrentModificationException异常。HashMap的Value迭代器实现如下：

```java
final class ValueIterator extends HashIterator
    implements Iterator<V> {
    public final V next() { return nextNode().value; }
}

abstract class HashIterator {
    Node<K,V> next;        // next entry to return
    Node<K,V> current;     // current entry
    int expectedModCount;  // for fast-fail
    int index;             // current slot

HashIterator() {
    //在构造迭代器的时候，将modCount值赋值给expectedModCount
    expectedModCount = modCount;
    Node<K,V>[] t = table;
    current = next = null;
    index = 0;
    if (t != null && size > 0) { // advance to first entry
        do {} while (index < t.length && (next = t[index++]) == null);
    }
}

public final boolean hasNext() {
    return next != null;
}

final Node<K,V> nextNode() {
    Node<K,V>[] t;
    Node<K,V> e = next;
    //在获取下一个节点前，先判定modCount值是否修改，如果被修改了则抛出ConcurrentModificationException异常，从前面可以知道，当修改了HashMap的时候，都会修改modCount值。
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    if (e == null)
        throw new NoSuchElementException();
    if ((next = (current = e).next) == null && (t = table) != null) {
        do {} while (index < t.length && (next = t[index++]) == null);
    }
    return e;
}

//迭代器的删除操作，会重新给exceptedModCount赋值，因此不会导致fast-fail
public final void remove() {
    Node<K,V> p = current;
    if (p == null)
        throw new IllegalStateException();
    //先判定modCount值是否被修改了
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    current = null;
    K key = p.key;
    removeNode(hash(key), key, null, false, false);
    //将modCount值重新赋值给expectedModCount，这样下次迭代时，不会出现fast-fail
    expectedModCount = modCount;
}
```
}

可以看到，通过迭代器remove一个元素，虽然会改变modCount值，但会将modCount值重新赋值为expectedModCount，这样下次再迭代时，不会出现fast-fail。而通过HashMap的remove方法会修改modCount值，但不会更新迭代器的expectedModCount值，所以迭代器在迭代操作时，会抛出ConcurrentModificationException异常。

如果从结构上对映射进行修改，除非通过迭代器自身的 remove 或 add 方法，其他任何时间任何方式的修改，迭代器都将抛出 ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败。注意，迭代器的快速失败行为不能得到保证，一般来说，存在不同步的并发修改时，不能作出任何坚决的保证。快速失败迭代器尽最大努力抛出 ConcurrentModificationException。
