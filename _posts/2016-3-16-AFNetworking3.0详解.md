## AFNetworking3.0详解
在AF中，大量用到了`GDC`和`NSURLSession`的代理方法，所以在介绍AF之前，有必要简单介绍一下GDC(grand dispatch center) 和NSURLSession.

#### 1.GDC

GDC是苹果在C层面上实现的一种消息配发机制，极大地方便开发者处理多线程和线程中的消息传递。在此之上，苹果封装了NSOperation和NSOperationQueue,其功能更为强大，但是仍然是基于GDC的。

GDC中有几个重要概念。
`dispatch_async` 异步派发
`dispatch_sync` 同步派发
`dispatch_get_main_queue` 主队列
`dispatch_queue_create(q_id)` 非主队列
`DISPATCH_QUEUE_SERIAL` 串行
`DISPATCH_QUEUE_CONCURRENT` 并行
看起来好像有点晕，一个个来解释其实也不难。
首先，`主队列`在系统中是唯一的，即iOS中常说的UI线程，其他创建的线程都是`非主队列`(所以main_queue只有get方法，而其他queue可以create)。

`主队列`一定是`串行`的，而`非主队列`可以是`串行`也可以是`并行`的，具体由创建时候传进的参数决定。

`同步派发`意味中程序要`阻塞`，等到代码块执行完毕后，继续执行后面的代码。`异步派发`意味着程序`不阻塞`，程序执行这一行后，立马执行下一句，而不是等待执行结果。

#### 2.NSURLSession
`NSURLSession`是iOS7之后的重要更新，之前一直是`NSURLConnection`
- NSURLConnection
	包含`NSURLRequest`，`NSURLResponse`， `NSURLProtocol`， `NSURLCache`， `NSHTTPCookieStorage`， `NSURLCredentialStorage`， `NSURLConnection`
	一个请求再被发送到服务器之前，会先查询缓存信息，根据策略(policy)以及可用性(availability)的不同来进行处理。
	在发送给服务器后，服务器可能会发出鉴权查询(authentication challenge)，可以由cookie和机密存储(credential storage)自动响应，也可以被委托对象响应
- NSURLSession
		与NSURLConnection的区别，是把URLConnection换成了URLSession，包括`NSURLSession`, `NSURLSessionConfiguration`,  `NSURLSessionTask`(`NSURLSessionDataTask`, `NSURLSessionUploadTask`, `NSURLSessionDownloadTask`)(NSURLSessionTask是一个抽象类，它下面的三个子类才是真正可以使用的类，其中dataTask继承于DataTask)
		与NSURLConnection相比，URLSession最直接的改进就是可以配置每个session的缓存，协议，cookie以及证书策略，甚至跨应用共享这些数据。
		`SessionTask`负责数据的加载，所有task共享其创造者`NSURLSession`这一公共委托。

#### 3.一个简单的网络请求框架

    同步
    NSData *data = [NSData dataWithContentUrl:[NSURL URLWithString:@"http://www.baidu.com"]];

    异步
    dispatch_async(dispatch_queue_create('com.domain.me'), ^{
		NSData *data = [NSData dataWithContentUrl:[NSURL URLWithString:@"http://www.baidu.com"]];
		dispatch_async(dispatch_get_main_queue(), ^{
			display_data_on_UI(data);
		});
	})


上面是一个非常简单的网络请求，这是异步网络请求的主要思想。

#### 4.基于NSURLSession的网络请求实现
NSURLSession是一个网络请求框架，其功能更为强大，它有各个阶段的代理方法，包括前期的缓存、鉴权策略，后续的请求序列化；它还可以在应用没有运行的时候进行上传下载操作；它也可以取消任务、重启任务、暂停任务、恢复任务等等。
其主要使用方式如下：

    NSURLSession = [NSURLSession sharedSession];
    NSURLDataTask *task = [session dataTaskWithURL:[NSURL URLWithString:@"http://www.baidu.com"]
    completionHandler:^(NSData *data,
		      NSURLResponse *response,
                    NSError *error) {
	                    //TODO
                    }];
    [task resume];
                  
上面是task的一个实现版本，每个task的构造方法都有包含comletionHandler block和没有包含两个版本。例如：这样两个构造方法 –dataTaskWithRequest: 和 –dataTaskWithRequest:completionHandler:。通过指定completionHandler block会创建一个隐式的delegate取代原来的delegate -- session。对于需要重写原有session task的delegate默认行为的情况，我们需要使用不含completionHandler的版本（如AFNetworking）。

#### 5.为什么使用AFNetworking
在NSURLSession面前，其实AF的优势没有之前那么大了，但是大家还是习惯使用它，因为AF封装出来的接口简单易懂，非常易用。
AFNetworking的实现：
主要两个类`AFURLSessionManager`和`AFURLSessionManagerTaskDelegate`
其他的`AFURLRequestSerialization`和`AFURLResponseSerialization`负责请求和返回体的序列化
`AFSecurityPolicy`负责安全策略
`AFHTTPSessionManager` 是咱们直接使用所接触到的类，他把以上几者都关联在一起
`AFURLSessionManager` 包含一个`session`(该session的delegate为sessionManager self)，一个`mutableDictionary`（用于保存task和taskDelegate之间的映射关系）

**请求开始时**

通过下面方法传入request以及completionHandler。

```
    - (NSURLSessionDataTask *)dataTaskWithRequest:(NSURLRequest *)request
                               uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgressBlock
                             downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgressBlock
                            completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject,  NSError * _Nullable error))completionHandler
```
这个函数是AF的核心，
1. 首先在一个串行队列中创建dataTask
2. 通过`delegateManager`alloc一个TaskDelegate，设置它的manager，completionHandler, block等
3. 在`mutablDictionary`添加task和taskDelegate的映射关系
4. 添加didResume和didSuspend的Notification Observer

**请求返回时**
	中途的各种状态，都可以通过delegate方法回调到sessionManager,由sessionManager进行具体处理。请求成功之后的回调也不例外。
```
    - (void)URLSession:(NSURLSession *)session
              task:(NSURLSessionTask *)task
didCompleteWithError:(NSError *)error
```
该delegate方法属于`NSURLSessionTaskDelegate`, 该方法通过`mutableDictionary`取出对应task的taskDelegate，调用该delegate的处理方法，然后从dictionary中移除该映射。

AFURLSessionManagerTaskDelegate中主要实现各个delegate之后的具体处理。例如：

    - (void)URLSession:(__unused NSURLSession *)session
              task:(NSURLSessionTask *)task 
              didCompleteWithError:(NSError *)errorr


该方法中主要做的事是：
1. 取出manager的responseSerializer，用于后续的返回体序列化
2. 设置userInfo的各项值，用于后续Notification传递出去
3. 判断是否有错误，如有，则dispatch到主线程执行completionHandler，并在主线程发起完成通知
4. 如果请求成功，则将结果序列化之后，dispatch到主线程执行completionHandler，并在主线程发起完成通知

#### 6. 其他的边边角角

缓存策略是通过sessionConfiguration配置的，使用NSURLCache，是系统提供的支持memory和disk的缓存机制，具体有NSURLSession自行完成
resume和suspend状态是NSURLSession自行控制切换的。

UIImageView+AFNetworking
3.0.4使用的是NSMutableDictionary, 自身管理缓存淘汰，淘汰最长时间没有使用的image
之前的采用的是NSCache，NSCache只支持memory缓存，不知道disk缓存。

对于具体的Task，只要保留了该指针，皆可通过它进行取消、暂停、恢复等操作。

