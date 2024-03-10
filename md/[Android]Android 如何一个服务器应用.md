# [Android]Android 制作一个HTTP服务器应用

## 思考过程

HTTP 服务器就像Apache 服务器那样的，不过不需要做那么多的功能。

既然是HTTP 服务器那就应该遵守HTTP 协议呗，不过学习网络编程时用的时socket，并且以为socket 是一个协议，那这个HTTP 协议也应该有一个类似`HTTP` 和 `ServerHTTP` 的类呗。

但是，不是的，查了HTTP 之后，好多的文章都在说socket 不是一个协议，只是编程接口，只有HTTP 才是协议。嗯，好吧。而且还说的是socket是传输层的，负责信息传输，HTTP 是应用层的，负责决定发送什么样的内容。哦，那也就是说应该使用socket 实现HTTP 协议。

发送数据使用socket，被发送的数据要遵守HTTP 的规则，毕竟是应用层，我们也可以不遵守（不过不遵守浏览器就不干了）。

## 编码过程

1. 先创建一个空的Android 项目。

2. 在`MainActivity` 类中添加如下代码。

    ```java

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    ServerSocket serverSocket = new ServerSocket(8080);
                    Socket socket = serverSocket.accept();
                    socket.getOutputStream().write(("HTTP/1.1 200 OK\n" +
                            "Date: "+new Date().toString()+"\n" +
                            "Content-Type: text/html; charset=UTF-8\n" +
                            "\n" +
                            "<html>\n" +
                            "      <head></head>\n" +
                            "      <body>\n" +
                            "            <p>hello world</p>\n" +
                            "      </body>\n" +
                            "</html>").getBytes(StandardCharsets.UTF_8));

                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    ```

    代码非常简单，socket 都没有关闭，仅仅演示这个功能。如果添加对浏览器请求进行处理的功能，然后传回相应的信息，然后丢掉这个`socket` 对象，好像就是一个服务器了哦。

    刚开始在`write` 函数里只输出“hello world”，无效，浏览器加载失败，想到需要添加一些信息，就像做web开发时的response一样。就是现在的这个样子，最后面是html代码。想要了解更多信息建议找更多的资料。

    时间那里不知道有没有严格限制，反正现在在浏览器中还没有问题。

    现在想想浏览器也不难啊，只要我能把内容渲染出来🤩。

## 后续

1. 为了获取request 数据，使用InputStream 进行读取，但是出现了一个问题
    虽说我在做这个的时候有点主观想法吧，用了一个while 循环，就像读取文件似的，然后浏览器就开始没有响应了，去掉循环就没有问题了，request 对象长度400多，用1024 字节数组读取没有什么问题，但是还是不太理解。后面增加的想法：可能这个流怎么读都不说自己的流到头了，毕竟还要上传文件呢。

    最后，这个文章后续可能会更新，比如遇到了什么坑。

2. 如果仅支持HTTP 的话，服务器无法主动传送数据给浏览器，为了能够主动传数据，再增开一个`WebSocket` 的服务器，用的是MDN 提供给的demo代码

    ```javascript
        const socket = new WebSocket('ws://' + location.hostname + ':8081');
        // Connection opened
        socket.addEventListener('open', function(event) {
            console.log('open', event);
        });
        // Listen for messages
        socket.addEventListener('message', function(event) {
            console.log('Message from server ', event);
        });
        socket.addEventListener('close', function(e) {
            console.log('close', e);
        });
        socket.addEventListener("error", function(e) {
            console.log(e);
        });

    ```

    上面的代码和原来的稍有不同。
    服务器的代码就不贴了，因为有问题。问题是，连接是能够连接上，但是连接上之后节立马断开，找了很多问答资料都解决不了，所以换了一个开源的集成的java 库，这个库虽然不会立马断开，但是有时候根本连接不上，有时候一刷新就会连接失败，后来想着这个功能我是玩不通了，就想着连接不上的时候给用户提示，就添加了`try catch`，发现基本上连接都可以成功，不再失败了，虽然问题解决但也是一个谜题（我用的开源库是`Java-WebSocket`，用起来很无脑，不贴代码了）。

    还有就是`try catch` 其实用的不对，因为这个好像不是同步代码，是捕获不到的。

    还有要注意的是socket 回调函数的`event` 里的错误码，基本上不要参考这个玩意，很可能只会浪费自己的时间，当然也有可能是我没有找对方法。
