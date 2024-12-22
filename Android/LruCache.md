# LruCache
- 使用时通过构造函数指定size，通过重写sizeOf()来计算每个成员的size。使用类似于使用Map(因为原理是利用了LinkedHashMap)
- 原理：利用LinkedHashMap来维护最近使用的逻辑，通过维护size来维护容量。 每次增加数据时，会通过trimToSize()来调整大小。相应操作代码块有synchronized修饰，所以是线程安全。
- put()：通过使用LinkedHashMap(下面用Map表示)存储数据，并增加size；如果key原来存在，则相应原元value的size减少。最后调用trimToSize()调整大小
- trimToSize()：内部维持一个循环，循环直到size<maxSize时退出，通过Map取最不常使用的值进行删除并相应减少size（最不常用的会处于LinkedHashMap的链头）
- get()：1、通过Map查找，找到就返回；2、没找到，如果重写了create()返回对象，就会重新map.put()插入并维护size。(create()不处于同步块中，因此可能会有多线程问题，此时如果map.put()遇到冲突值，会重新map.put()原值，弃掉新值)