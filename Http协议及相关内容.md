#Http协议
####超文本传输协议（HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。
 
Http版本区别
---------------
 * 1.1
  * 默认持久连接
  * 支持缓存
  * 支持管道方式发送多个请求
 * 2.0
  * 多路复用允许同时通过单一的HTTP/2连接发起多重的请求-响应消息
    * 单连接多资源的方式，减少服务端的链接压力，内存占用更少，连接吞吐量更大
    * 由于TCP连接的减少而使网络拥塞情况得以改善，同时![慢启动](http://blog.csdn.net/wykwdy007/article/details/6720254)时间的减少，使拥塞和丢包恢复速度更快
  * 头部压缩(HPACK算法)
  * 对请求划分优先级
  * 服务器推送流(Server Push)

Http请求方式
--------------
 * Get:请求获取Requeat-URI所标识得资源
 * POST:在Request-URI所标识得资源后附加新的数据
 * HEAD,PUT,DELETE,TRACE,CONNECT,OPTIONS
 
Http协议特点
--------------
* 支持C/S模式
* 简单快速:客户向服务器请求服务时，只需传送请求方法和路径。
* 灵活:HTTP允许传输任意类型的数据对象
* 无连接
* 无状态
 
Request Headers和Response Headers
-----------------------------------
 ![常用HTTP响应头和请求头信息对照表](http://tools.jb51.net/table/http_header)
 
 ![image](https://github.com/shishengjia/Android-NetworkArchitecture-Design/blob/master/Headers.jpg)
 
 其中Status Code:200 表示成功
 
 **响应码**
  * 100-199 信息提示
  * 200-299 成功
  * 300-399 重定向
  * 400-499 客户端错误
  * 500-599 服务器错误
 
 User-Agent等其他请求头都可以自定义，完成特定的功能
 
 ```java
 public class HeadHttp {
    public static void main(String args[]) {
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder().
                url("https://www.baidu.com").addHeader("User-Agent", "from shishengjia").build();
        try {
            Response response = client.newCall(request).execute();
            Headers headers = response.headers();
            if(response.isSuccessful()){
                for(int i=0;i<headers.size();i++)
                    System.out.println(headers.name(i)+":"+headers.value(i));//打印响应头
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
 ```
 这个示例中通过addHeader()方法可以自定义请求头
 
Http同步请求和异步请求
--------------------------
 
同步——使用者通过单个线程调用服务；该线程发送请求，在服务运行时阻塞，并且等待响应。<br>
异步——使用者通过两个线程调用服务；一个线程发送请求，而另一个单独的线程接收响应。<br>
在同步请求/响应通信模型中，总是浏览器（与 Web 服务器、应用服务器或 Web 应用程序相对）发起请求（通过 Web 用户）。接着，Web 服务器、应用服务器或  Web 应用程序响应进入的请求。在处理同步请求/响应对期间，用户不能继续使用浏览器。<br>
在异步请求/响应通信模型中，浏览器（通过 Web 用户）到 Web 服务器、应用服务器或 Web 应用程序的通信（以及反过来）是解耦的。在异步请求/响应对的处理中，Web 用户在当前异步请求被处理时还可以继续使用浏览器。一旦异步请求处理完成，异步响应就被通信（从 Web 服务器、应用服务器或 Web 应用程序）回客户机页面。典型情况下，在这个过程中，调用对 Web 用户没有影响；他们不需要等候响应。

**示例代码**

```java
/**
 * Created by shi on 16/11/2016.
 */

public class testOKHttp {

    //同步请求
    public static void sendSyncRequest(String url) {
        //请求前获取当前线程ID
        System.out.println(Thread.currentThread().getId());
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder().url(url).build();
        try {
            Response response = client.newCall(request).execute();
            if (response.isSuccessful()) {
                //请求后获取当前线程ID
                System.out.print(Thread.currentThread().getId());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //异步请求
    public static void sendAsyncRequest(String url) {
        //请求前获取当前线程ID
        System.out.println(Thread.currentThread().getId());
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder().url(url).build();
        client.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                System.out.println("Error");
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                //请求后获取当前线程ID
                System.out.println(Thread.currentThread().getId());
            }
        });
    }

    public static void main(String[] args) {
//        sendSyncRequest("https://www.baidu.com");
        sendAsyncRequest("https://www.baidu.com");

    }
}
```
**运行结果**

同步请求:线程ID都为1，表示此时只有一个线程。<br>
异步请求:一个线程ID为1，另一个线程ID为12，表示此时有两个线程。<br>

Http缓存机制
------------
缓存图解<br>
![缓存图解](https://github.com/shishengjia/Android-NetworkArchitecture-Design/blob/master/Http%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6.png)

缓存相关字段
 
 * Expires(Htttp1.0)指示相应内容过期的时间，格林威治时间GMT ，是绝对时间，所以客户端与服务端的时间可能会有偏差
 * Cache-Control(Http1.0)指定一段时间后过期，解决Expires的不足
 * Last-Modified记录最后修改数据的时间，服务端和客户端相比较来判断缓存是否过期
 * ETag将服务端和客户端的内容做编码处理分别生成一个加密值，比较是否相同来判断是否需要重新加载数据
 * Date返回服务端的时间
 * if-Modified-Since客户端存取的该资源最后一次修改的时间，同Last-Modified
 * if-None-Match同ETag
 
 
Http中与下载有关的几个字段
----------------------------
 * Transfer-Encoding:chunked 表明传输方式是一段一段的发出，像一般网站的主页会这样设置，因为不知道文件大小，所以无法通过多线程来下载
 * Content-Length 一般静态文件如图片，静态js文件等会在Response Headers中指示，标明文件长度
 * Range 在request中指定，表明该次请求所下载文件的长度，从0开始
```java
    public static void main(String args[]){
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder().
                addHeader("Range","bytes=0-2048"). //截取指定长度的内容
                url("http://t1.zngirls.com/gallery/19705/19815/003.jpg").
                build();
        try {
            Response response = client.newCall(request).execute();
            //如果响应头没有content-length字段或者content-length=0，则返回-1.
            System.out.println("content-length: "+response.body().contentLength());
            if(response.isSuccessful()){
                Headers headers = response.headers();
                //打印响应头信息
                for(int i=0;i<headers.size();i++)
                    System.out.println(headers.name(i)+" : "+headers.value(i));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
