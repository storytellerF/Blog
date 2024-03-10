# [Android]Android 制作一个HTTP服务器应用

## 思考过程

HTTP 服务器就像Apache 服务器那样的，不过不需要做那么多的功能。

既然是HTTP 服务器那就应该遵守HTTP 协议呗，不过学习网络编程时用的时socket，并且以为socket 是一个协议，那这个HTTP 协议也应该有一个类似`HTTP` 和 `ServerHTTP` 的类呗。

但是，不是的，查了HTTP 之后，好多的文章都在说socket 不是一个协议，只是编程接口，只有HTTP 才是协议。嗯，好吧。而且还说的是socket是传输层的，负责信息传输，HTTP 是应用层的，负责决定发送什么样的内容。哦，那也就是说应该使用socket 实现HTTP 协议。

发送数据使用socket，被发送的数据要遵守HTTP 的规则（毕竟是应用层，我们也可以不遵守不过不遵守浏览器就不干了，然后我们在自己写一个浏览器吧：））。

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

## 后续

1. 为了获取request 数据，使用InputStream 进行读取，但是出现了一个问题
    虽说我在做这个的时候有点主观想法吧，用了一个while 循环，就像读取文件似的，然后浏览器就开始没有响应了，去掉循环就没有问题了，request 对象长度400多，用1024 字节数组读取没有什么问题。

    后来用的的办法是用Scanner 获取请求头，毕竟真的遇到了一次读取的内容超过1204；

    ```java
    Scanner scanner=new Scanner(inputStream);
    String request=scanner.useDelimiter("\\r\\n\\r\\n").next();
    ```

    因为一个请求头的结尾就是`\r\n`。

    但是后面实现文件上传功能时，发现这个获取请求头的方法就不行了，比如浏览器上传的文件数据是

    >12345

    获取到的数据却是

    >2345

    具体原因不明，可能Scanner 自己往后面移动了，所以这个过程也需要自己完成。

    ```java
    private String getRequest(Socket socket) throws IOException {
        InputStream inputStream = socket.getInputStream();
        socket.setSoTimeout(5000);
        byte[] bytes = new byte[1024];
        byte[] stop = new byte[4];
        stop[0] = '\r';
        stop[1] = '\n';
        stop[2] = '\r';
        stop[3] = '\n';
        int stopIndex = 0;
        int index = 0;
        while (true) {
            byte temp = (byte) inputStream.read();
            if (temp == -1) {
                break;
            }
            if (stop[stopIndex] == temp) {
                stopIndex++;
                if (stopIndex == 4) {
                    break;
                }
            } else {
                stopIndex = 0;
            }
            bytes[index++] = temp;
        }
        return new String(bytes, 0, index);
    }
    ```

2. 如果仅支持HTTP 的话，服务器无法主动传送数据给浏览器，为了能够主动传数据，再增开一个`WebSocket` 的服务器，用的是MDN 提供的demo代码

    浏览器端的代码

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
    服务器连接是能够连接上，但是连接上之后节立马断开，找了很多问答资料都解决不了，所以换了一个开源的集成的java 库，这个库虽然不会立马断开，但是有时候根本连接不上，有时候一刷新就会连接失败。

    还有要注意的是socket 回调函数的`event` 里的错误码，基本上不要参考这个玩意，很可能只会浪费自己的时间。

    然后我就放弃了使用第三方库的想法，决定自己解决这个问题。

    需要解决的问题就是连接之后立马关闭的问题。

    首先我以为是因为一个循环结束后，对象被回收了，就开了线程，发现不行。然后在一个循环结束时查看`Socket`的关闭状态，是关闭状态，然后就想到了流的问题，把关闭输入流的代码去除（原MDN 上的代码就是关闭了流的，被误导了），socket 就不再关闭，问题解决。[MDN Writing a WebSocket server in Java
    ](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_a_WebSocket_server_in_Java#First_steps)

3. 然后是WebSocket通信

    浏览器端发送一个“hello” 的字符串，在服务端获得数据是
    `-127 -123 79 -45 -102 62 39 -74 -10 82 32`，直接转换成字符串没有任何意义，都是一些乱码，因为发过来的是帧数据 [MDN Writing WebSocket servers
](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_servers)

    ```html
        Frame format:  
    ​​
        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
        +-+-+-+-+-------+-+-------------+-------------------------------+
        |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
        |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
        |N|V|V|V|       |S|             |   (if payload len==126/127)   |
        | |1|2|3|       |K|             |                               |
        +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
        |     Extended payload length continued, if payload len == 127  |
        + - - - - - - - - - - - - - - - +-------------------------------+
        |                               |Masking-key, if MASK set to 1  |
        +-------------------------------+-------------------------------+
        | Masking-key (continued)       |          Payload Data         |
        +-------------------------------- - - - - - - - - - - - - - - - +
        :                     Payload Data continued ...                :
        + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
        |                     Payload Data continued ...                |
        +---------------------------------------------------------------+
    ```

    每一个都是一位，一个字节是八位，所以读取第一个字节就是“fin rsv1，rsv2，rsv3 opcode”，下一个字节就是“mask payload len”了，后面“payload Data”是我们发送的信息（上面有一个方框什么都没有写其实数据依然是Extend payload length continued），剩下的内容MDN 上写的很清楚，如果想要了解更多信息，点击上面的链接。

    浏览器接受数据没有问题，浏览器已经帮我们处理完了，主要处理服务器端。发送到浏览器的数据应该按照此格式发送。

4. 上传文件

    开始想用apache的开源库获取文件，但是失败了，要么文件不全，要么就完全为空，还是自己写。

    文件上传请求头的部分内容

    >contentType:multipart/form-data; boundary=----WebKitFormBoundaryHpUAY0qCryu0Oc7o

    我们需要获取`boundary` 后面的数据，主要作用是标识文件在流中的范围。当然了，这个`boundary`是会变的，每次都要重新获取。

    请求头之后就是发送的数据（以一个文件为例）

    第一行

    >------WebKitFormBoundaryHpUAY0qCryu0Oc7o

    第二行

    >Content-Disposition: form-data; name="file"; filename="ic_excel.xml" filename:ic_excel.xml

    第三行

    >Content-Type: text/xml

    第四行

    >\r\n

    倒数第四行

    >\r\n

    倒数第三行

    >------WebKitFormBoundaryHpUAY0qCryu0Oc7o

    倒数第二行

    >ic_excel.xml

    倒数第一行

    >------WebKitFormBoundaryHpUAY0qCryu0Oc7o--

    也就是说我们需要按照行的方式来读取，以免错过这些`boundary`，又不能够使用Java提供的`readLine`，在二进制文件几乎没有换行，我们的字节数组是盛不下一行的。

    ```java
    private LineData getLineData(InputStream inputStream, int capacity) {
        byte[] bytes = new byte[capacity];
        int index = 0;
        try {
            while (true) {
                int current = inputStream.read();
                if (current == '\r') {
                    //检查下一个是否是\n
                    int next = inputStream.read();
                    bytes[index++] = (byte) current;
                    bytes[index++] = (byte) next;
                    if (next == '\n') {
                        //是\n，是一个crlf换行，退出循环
                        break;
                    } else if (next == -1) {
                        //因为是-1，所以退出while 不会添加换行，基本不会出现这种情况，可以根据自己需要抛出异常
                        index--;
                        break;
                    }  //current 是正常的一个\r

                } else if (current == -1) {//因为是-1，所以退出while 不会添加换行，基本不会出现这种情况，可以根据自己需要抛出异常
                    break;
                } else
                    bytes[index++] = (byte) current;
                if (index >= capacity - 1) {//不够下一次的\r\n了
                    break;
                }
            }
            return new LineData(bytes, index);
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    static class LineData {
        byte[] bytes;
        String string;
        int length;

        LineData(byte[] bytes, int length) {
            this.bytes = bytes;
            this.string = new String(bytes, 0, length);
            this.length = length;
        }
    }
    ```

    最后一个`boundary`(倒数第三行)的上面是一个换行（还要注意哦，所有的换行都是`\r\n`），所以在我们不知道下一行是不是`boundary`时，是不可以贸然写入文件的。

    ```java
        LineData lastLine = null;
        while (true) {
            LineData temp = getLineData(inputStream, capacity);
            if (temp == null) {//出现异常
                break;
            }
            if (temp.string.contains(first)) {
                //上一行(lastLine)是crlf，这样就没有输出这个内容就退出了
                break;
            }
            if (lastLine != null) {
                bufferedOutputStream.write(lastLine.bytes, 0, lastLine.length);
            }
            lastLine = temp;
        }
    ```

    文件读取完毕之后还要检查后面的`boundary`，如果后面的内容不是我们预期的那样，这个文件基本就是错误的了。解决办法将这几行数据作为文件内容，继续读取，知道下一组`boundary`结束组合。

    还可以根据文件类型，设定字节数组，比如文本文件，每行一百的都是足够的，如果是二进制文件，需要使用更长的数组。

5. WebSocket 建立连接时，需要进行Base64运算，

    ```java
    java.util.Base64.getEncoder().encodeToString();
    ```

    这段代码没有问题，但是有sdk版本限制，另一个可选的项是`android.util.Base64`，但是出错了。

    `java.util.Base64.getEncoder`的实现是返回

    ```JAVA
    static final Encoder RFC4648 = new Encoder(false, null, -1, true);
    ```

    ```java
    private Encoder(boolean isURL, byte[] newline, int linemax, boolean doPadding) {
            this.isURL = isURL;
            this.newline = newline;
            this.linemax = linemax;
            this.doPadding = doPadding;
        }
    ```

    `android.util.Base64`也有flag，

    ```java
    /**
     * Default values for encoder/decoder flags.
     */
    public static final int DEFAULT = 0;

    /**
     * Encoder flag bit to omit the padding '=' characters at the end
     * of the output (if any).
     */
    public static final int NO_PADDING = 1;

    /**
     * Encoder flag bit to omit all line terminators (i.e., the output
     * will be on one long line).
     */
    public static final int NO_WRAP = 2;

    /**
     * Encoder flag bit to indicate lines should be terminated with a
     * CRLF pair instead of just an LF.  Has no effect if {@code
     * NO_WRAP} is specified as well.
     */
    public static final int CRLF = 4;

    /**
     * Encoder/decoder flag bit to indicate using the "URL and
     * filename safe" variant of Base64 (see RFC 3548 section 4) where
     * {@code -} and {@code _} are used in place of {@code +} and
     * {@code /}.
     */
    public static final int URL_SAFE = 8;

    /**
     * Flag to pass to {@link Base64OutputStream} to indicate that it
     * should not close the output stream it is wrapping when it
     * itself is closed.
     */
    public static final int NO_CLOSE = 16;
    ```

    并且Android 明确说`Base64.DEFAULT`是`RFC2045`

    ```JAVA
        static final Encoder RFC4648 = new Encoder(false, null, -1, true);
        static final Encoder RFC2045 = new Encoder(false, CRLF, MIMELINEMAX, true);  
    ```

    区别在`newline`和`linemax`，所以我们传入`Base64.NO_WARP`即可。
