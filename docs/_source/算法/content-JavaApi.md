# 目录

- [ArrayList](_source/算法/JavaApiArrayList.md)
- [LinkedList](_source/算法/JavaApiLinkedList.md)
- [Stack](_source/算法/JavaApiStack.md)
- [HashMap](_source/算法/JavaApiHashMap.md)
- [Collection](_source/算法/JavaApiCollection.md)

| API  | ArrayList                     | LinkedList（`头指针`）                                                                                 | Stack         | HashMap                     |
| :--- | :---------------------------- | :----------------------------------------------------------------------------------------------------- | :------------ | :-------------------------- |
| 增加 | add(`index`, element)         | 队列：offer(element)【队尾加入】<br><br> 栈：push(element)<br><br>  链表尾 add(element)                | push(element) | put(key, value)             |
| 删除 | remove(`index`)               | 队列：poll()【队头删除】<br><br> 栈：pop()<br><br> 删除链表头 remove()<br><br> 删除链表尾 removeLast() | pop()         | remove(key)                 |
| 修改 | set(`index`, element)         | <br>set(`index`, element)<br><br>                                                                      | /             | `put`(key, value)           | `put`(key, value) |
| 查询 | get(`index`)                  | `getFirst`()<br><br> `getLast`() <br><br> get(index)                                                   | peek()        | `get`(key)                  |
| 遍历 | foreach <br><br> for + size() | foreach <br><br> for + size()                                                                          | /             | entrySet()<br><br> keySet() |