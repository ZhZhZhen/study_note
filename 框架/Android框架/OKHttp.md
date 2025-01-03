# OKHttp
- 基本使用
    - 1、创建OKHttpClient和Request
    - 2、使用OKHttpClient.newCall(Request request)返回一个Call。其中newCall为Call.Factory的接口实现，该方法具体实现为返回一个RealCall，该类为Call接口唯一实现。
    - 3、使用Call进行同步或异步请求。execute()/enqueue()
- RealCall
    - Call接口的唯一实现。构造方法中传入OKHttpClient和Request。请求的具体实现者
    - 1、execute()：同步请求，该方法将请求将RealCall加入 runningSyncCalls并调用getResponseWithInterceptorChain()获取Response，同步请求无数量限制
    - 2、enqueue()：异步请求，该方法需要传入Callback回调，该回调被传入AsyncCall中，并加入readyAsyncCalls中。之后会调用promoteAndExecute()循环判断runningAsyncCalls列表中的AsyncCall判断异步请求数是否大于64（超出则break），异步请求中相同host请求数是否大于5（超出则continue），未超过则会收集这些AsyncCall，加入到runningAsyncCalls。并循环将收集的AsyncCall加入线程池中， ()会调用getResponseWithInterceptorChain() [AsyncCall.run]("http://AsyncCall.run")
    - 3、getResponseWithInterceptorChain()：请求的核心方法。该方法首先添加自定义拦截器，然后添加5个自带拦截器（这5个拦截器是请求的步骤实现，具体功能在拦截器中），之后会创建RealInterceptorChain用于责任链以此调用拦截器，每个拦截器会调用通过Chain调用下一级拦截器，并等待其返回，之后处理结果返回给上级拦截器
    - 链式调用：实现至RealInterceptorChain.proceed()：该方法通过index取下一个拦截器，并创建下一个RealInterceptChain next，调用interceptor.intercept(next)。在拦截器的intercept()方法中通常会获取chain中的request做拦截处理，然后使用next.proceed(request)进行下一层链的调用。之后对该方法返回的Response做处理返回给上一层
    - 提及的对象说明
        - readyAsyncCalls：存放未开始运作的异步请求（由Dispatcher管理，下二同）
        - runningAsyncCalls：存放开始运作的异步请求
        - runningSyncCalls：存放开始运作的同步请求
        - AsyncCall：异步请求所使用的包装类，其中含有RealCall，和Callback
- 拦截器
    - RetryAndFollowUpInterceptor：用于处理网络异常的重连，和返回3XX时的重定向(会自动获取响应体的头部Location组成新的Request)，次数上限为20。
    - BridgeInterceptor：改成用于将用户的请求和服务器请求进行相互转化。请求时，将Request中的配置信息转化为对应的请求头，并补齐默认请求头；响应时，对Response内容进行解压处理
    - CacheInterceptor：用于缓存响应信息，并根据缓存策略决定返回缓存Response还是进行下一层拦截器的调用（即获取网络Response）。请求时，根据缓存策略/网络情况/缓存情况决定是否直接返回缓存的Response还是请求下一个拦截器；响应时，对获取的网络Response进行缓存。（缓存策略被实现写死，只需要服务端修改响应的响应头即可，存储策略采用DiskLruCache）
    - ConnectInterceptor：通过RealCall.initExchange()建立连接，并返回ExChange用于下一拦截器，ExChange通过ExChangeCodec来完成Http协议的解析（就是通过输入输出流读写数据），ExChange则是对ExChangeCodec的封装(类似工具类)。ExChangeCodec内部则持有了开启连接的RealConnection
    - CallServerInterceptor：通过上层传递的ExChange.ExchangeCodec写入请求头/请求体。并通过ExchangeCodec读取相应头/响应体，然后返回Response给上层
- 连接步骤
    - 第四个拦截器会通过RealCall.initExchange()建立连接，该方法会返回ExChange。其中会最终通过ExChangeFinder.findConnection()返回RealConnection，并通过RealConnection获取ExChangeCodec用于编码/解码HTTP。
    - findConnection():这个方法是复用RealConnection的主要逻辑实现，RealConnection内部有Socket，用于连接的类
        - 1、RealCall的RealConnection可用时(判断其Socket有无关闭，流是否关闭，无关闭则可用)返回。
        - 2、否则查找连接池，循环RealConnection集，找出可复用的RealConnection，如果RealConnection.calls超出限制，或域名不同时，则无法复用。这边会查找两次，第一次使用ip，第二次使用域名映射集List再次查找
        - 3、否则新建一个并调用其connect()方法连接，该方法最终使内部Socket连接并获取流。之后会再次查找连接池中复用的RealConnection，如果有复用的就关闭刚刚创建的，没复用的加入连接池
            - RealConnection.connect()中：1、调用connectSocket()，使用socket.connet()创建连接（三次握手过程被Socket封装了），然后获取了其输入流输出流(用了kotlin的扩展函数来生成OutputStreamSink和InputStreamSource)。2、establishProtocol()开始后续的http2.0协议或https协议tls连接建立。3、如果无tls连接，则调用startHttp2()。4、如果有tls连接则调用connectTls()，方法内使用handShake()使用默认的SSLSocketFactory来验证证书，使用默认HostnameVerifier验证证书host是否和请求地址host一致。（我们可以自定义SSLSocketFactory和HostnameVerifier来做全信任），之后调用startHttp2()
            - 如果需要信任非认证机构的证书，需要提前存储证书，然后自定义SSLSocketFactory并设置给OKHttpClient
        - 23步骤成功后，会将RealCall加入到RealConnection.calls中（这是一个弱引用集合，用于清理时的闲置判断），同时RealCall的成员变量赋值RealConnection
- 缓存步骤
    - 先获取CacheStrategy来获取缓存策略：
        - 1、无缓存直接进行网络请求。
        - 2、CacheResponse中Cache-Control的字段为max-age=xxx，若过期则网络请求，否则使用缓存。
        - 3、CacheResponse中Cache-Control的字段为no-cache，则查看是否有Last-Modified,Etag信息，有则构建请求并添加请求头<if-ModiFied-Since,Last-Modified><If-None-Match,Etag>，该情况下服务端返回304则使用原缓存，服务端返回200则会额外返回新数据。无则进行网络请求
        - 返回new CacheStrategy(newRequest,cacheResponse)
    - 根据CacheStrategy中的request和cacheResponse进行操作：
        - 1、request和response都为null，创建请求错误Response返回上层拦截器
        - 2、request为null，cacheResponse不为null，返回给上传拦截器
        - 3、将request交给下层拦截器请求。返回Response时，判断有cacheResponse(使用Last-Modifier，Etag情况)，并且响应码为304则返回cacheResponse给上层拦截器。否则都是返回新的Response给上层拦截器，并使用DiskLruCache缓存新Response
- 连接池RealConnectionPool
    - 连接池的闲置RealConnection数为5，定时5分钟清理一次，通过TaskRunner(最终由线程池运行)来定期执行cleanup()方法
    - cleanup()：1、循环RealConnection集合。判断是否闲置并计数，并找出闲置时间最长的RealConnection。2、判断闲置数是否超出5，闲置时间是否大于5分钟是则清理闲置最长的RealConnection，并关闭其Socket连接，返回0。3、否则如果闲置数>0，但没有触发清除则说明闲置时间未超过5分钟，则计算差值返回。4、否则认为没有闲置，则返回5分钟。5、否则返回-1     （上述返回值为纳秒）cleanup()方法的返回值代表该方法下次被执行的时间间隔，如果返回不为-1就会重新加入TaskRunner下次继续执行
    - 如何判断闲置：根据RealConnection.calls的内容数是否大于0判断。当findConnection()方法中找到RealConnection时，会添加RealCall的包装弱引用至该集合。这是一个弱引用集合，在连接池清理时，会判断弱引用get是否为null，如果集合中的弱引用get为null则该弱引用会被移除，之后返回calls.size判断是否闲置