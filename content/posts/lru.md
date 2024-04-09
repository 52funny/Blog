---
title: "LRU Cache 算法"
date: 2024-04-08T18:49:51+08:00
draft: false
tags: ['算法']
---



LRU 是缓存替换机制中的一个算法。由于计算机存储空间大小是 **固定** 的，并不是无限大小的空间，无法容纳所有的数据，所以当有新的数据要被置换到 **缓存** 时，必须采用一定的原则来取代适当的数据。此原则就是 **缓存淘汰算法**。

<!--more-->



## 缓存淘汰算法

传统的淘汰算法有以下算法：

+ 先进先出算法（FIFO）：最先进入的内容作为替换对象
+ 最少使用算法（LFU）：最久没有访问的内容作为替换对象
+ 最近最少使用算法（LRU）：最近最少使用的内容作为替换对象
+ 非最近使用算法（NMRU）：在最近没有使用的内容中随机选择一个作为替换对象

**LRU**（Least recently used） 算法根据内容的历史访问次数来进行淘汰数据，其核心思想是「如果一个内容最近被访问过，那么将来被访问的几率也会很高。」

我们可以使用双端链表来进行模拟数据，然后根据内容数据的访问来删除一些数据。

1. 当有新的内容数据要 **插入** 或者 **更新** 的时候，我们将其插入到链表头部
2. 当每次内容被 **命中** 的时候，则将内容移动到链表头部
3. 当 **链表满** 的时候，将链表尾部的内容进行丢弃



[146.LRU 缓存]( https://leetcode.cn/problems/lru-cache/) 让我们实现一个 LRU 的模拟。我们可以使用双向链表来实现 **快速插入** 节点到头部和 **快速删除** 尾部节点。同时我们要要实现将链表中间的元素 **快速插入** 到头部，所以我们要记住每一个中间节点的指针，我们可以使用 Hash 表来记住每一个节点的指针，这样我们就可以快速找到每一个指针的地址了。

代码实现如下：

```cpp
#include <cstddef>
#include <iterator>
#include <unordered_map>

struct Node {
    int key;
    int val;
    Node *prev;
    Node *next;

  public:
    Node() : key(0), val(0), prev(nullptr), next(nullptr) {}
    Node(int key, int val) : key(key), val(val), prev(nullptr), next(nullptr) {}
};
class LRUCache {
    Node *head;
    Node *tail;
    std::unordered_map<int, Node *> h;
    int capacity;
    int now;
    void pushHead(Node *n) {
        Node *next = head->next;
        head->next = n;
        n->prev = head;
        n->next = next;
        next->prev = n;
    }
    void remove(Node *n) {
        Node *prev = n->prev;
        Node *next = n->next;
        prev->next = next;
        next->prev = prev;
    }

  public:
    LRUCache(int capacity) : capacity(capacity) {
        head = new Node();
        tail = new Node();
        head->next = tail;
        tail->prev = head;
        now = 0;
    }

    int get(int key) {
        if (h.count(key) == 0) {
            return -1;
        }
        remove(h[key]);
        pushHead(h[key]);
        return h[key]->val;
    }
    void put(int key, int val) {
        if (h.count(key) > 0) {
            h[key]->val = val;
            remove(h[key]);
            pushHead(h[key]);
            return;
        }
        if (now >= capacity) {
            Node *last = tail->prev;
            remove(last);
            h.erase(last->key);
            delete last;
            now--;
        }
        Node *n = new Node(key, val);
        pushHead(n);
        now++;
        h[key] = n;
    }
};

```



# 参考内容

[https://en.wikipedia.org/wiki/Cache_replacement_policies](https://en.wikipedia.org/wiki/Cache_replacement_policies)

[https://zhuanlan.zhihu.com/p/34989978](https://zhuanlan.zhihu.com/p/34989978)
