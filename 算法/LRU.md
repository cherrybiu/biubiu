>  运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制 。
> 实现 LRUCache 类：

LRUCache(int capacity) 以正整数作为容量 capacity 初始化 LRU 缓存
int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
void put(int key, int value) 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。  

​																																										`--leecode`

- **时间复杂度为 O(n)**

```javascript

/**
 * @param {number} capacity
 */
var LRUCache = function(capacity) {
    this.capacity = capacity;
    this.keys = [];
    this.cache = Object.create(null);
};

/**
 * @param {number} key
 * @return {number}
 */
LRUCache.prototype.get = function(key) {
    if (this.keys.indexOf(key) > -1) {
        this.keys = this.keys.filter((value) => key !== value);
        this.keys.push(key)
        return this.cache[key];
    } else {
        return -1;
    }
};

/**
 * @param {number} key
 * @param {number} value
 * @return {void}
 */
LRUCache.prototype.put = function(key, value) {
    if (this.keys.indexOf(key) > -1) {
        this.keys = this.keys.filter((v) => key !== v);
        this.keys.push(key)
        this.cache[key] = value;
    } else {
        this.keys.push(key)
        this.cache[key] = value;
    }

    if (this.keys.length > this.capacity && key !== this.keys[0]) {
        delete this.cache[this.keys[0]];
        this.keys.shift();
    }
};

var lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4

```



- **时间复杂度为O(1) : 结合hashMap和链表方式**

```javascript
class ListNode {
    constructor(key, value) {
        this.key = key;
        this.value = value;
        this.prev = null;
        this.next = null;
    }
}

class LRUCache {
    constructor(capacity) {
        this.max = capacity;
        this.hashTable = {};
        this.count = 0;
        this.dommyHead = new ListNode();
        this.dommyTail = new ListNode();
        this.dommyHead.next = this.dommyTail;
        this.dommyTail.prev = this.dommyHead;
    }

    get(key) {
        let node = this.hashTable[key];
        if (node == null) {
            return -1;
        } else {
            this.removeFromList(node);
            this.addToHead(node);
            return node.value
        }
    }

    put(key, value) {
        var node = this.hashTable[key];
        if (node == null) {
            let newNode = new ListNode(key, value);
            this.count ++;
            this.hashTable[key] = newNode;
            this.addToHead(newNode);
            if (this.count > this.max) {
                this.removeTail();
            }
        } else {
            node.value = value;
            this.removeFromList(node);
            this.addToHead(node);
        }
    }

    removeTail() {
        let tail = this.dommyTail.prev;
        this.removeFromList(tail);
        delete this.hashTable[tail.key];
        this.count --;
    }

    removeFromList(node) {
        let prevNode = node.prev;
        let nextNode = node.next;
        prevNode.next = nextNode;
        nextNode.prev = prevNode;
    }

    addToHead(node) {
        node.next = this.dommyHead.next;
        node.prev = this.dommyHead;
        node.next.prev = node;
        this.dommyHead.next = node;
    }
}
```



leecode示例:

```makefile
输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```

