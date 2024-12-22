# Bundle
- Android传递存储数据的容器，实现了Parcelable接口，可用于跨进程传输
- 内部使用ArrayMap<String,Object>来存储数据
- *跨进程时Parcel支持基本数据，ArrayMap，ArraySet，Map，IBinder的携带。同时从Parcel读取数据的顺序应该与写入顺序一致，否则错误偏移就读不到对应数据了