**qpy-webframework是一个能在qpy平台上运行的微型web服务器，支持websocket，HTML/python模板语言和路由**

### 一、资源

 [python入门教程](https://www.runoob.com/python3/python3-basic-syntax.html)

 [QuecPython API](https://python.quectel.com/wiki/#/)

 [TCP SERVER](http://118.89.61.11:7777/net)

 [Quecpython社区链接](https://forumschinese.quectel.com/c/function-subjects/quectpython/43)

 [环境搭建](https://python.quectel.com/doc/doc/Quick_start/zh/index.html)

### 最小应用
###### 1、创建tmain.py 文件，复制下面代码
    from usr.WebSrv import WebSrv
    @WebSrv.route('/json')
    def json(httpClient, httpResponse) :
        httpResponse.WriteResponseJSON({1:2,"test":5})#返回json字符串
    @WebSrv.route('/test-template')
    def _reader_template(httpClient, httpResponse) :
        httpResponse.render("test.html")#渲染模板，test。HTML请参照模板部分
    @WebSrv.route('/')
    def _httpHandlerTestPost(httpClient, httpResponse) :
        content = "<p>Hello, World!</p>"
        httpResponse.WriteResponseOk( headers        = None,
                                      contentType    = "text/html",
                                      contentCharset = "UTF-8",
                                      content        = content )
    srv = WebSrv(ip='0.0.0.0',port=80,templatePath="/usr/www/")
    srv.Start(threaded=True)#开启web服务器
###### 目录结构：
![这是图片](/img/img_4.png "Magic Gardens")
###### 2、烧录框架和测试文件tmain.py，并运行应用，在浏览器中打开 http://192.168.1.1/，如下
![这是图片](/img/img_1.png "Magic Gardens")
###### 在浏览器中打开 http://192.168.1.1/json，如下
![这是图片](/img/img_2.png "Magic Gardens")
###### 在浏览器中打开 http://192.168.1.1/test-template，如下
![这是图片](/img/img_3.png "Magic Gardens")

###路由使用
###### 基本使用
    @WebSrv.route('/get-test')#默认get方法
    def handlerFuncGet(httpClient, httpResponse) :
      print("In GET-TEST HTTP")
    
    @WebSrv.route('/post-test', 'POST')#post方法
    def handlerFuncPost(httpClient, httpResponse) :
      print("In POST-TEST HTTP")

###### 带变量的路由使用
    @MicroWebSrv.route('/edit/<testid>/<testpath>')
    def handlerFuncEdit(httpClient, httpResponse, routeArgs) :
      print("In EDIT HTTP variable route :")
      print(" - testid   = %s" % routeArgs['testid'])
      print(" - testpath = %s" % routeArgs['testpath'])

### websocket 使用
    def _acceptWebSocketCallback(webSocket, httpClient) :
        webSocket.RecvTextCallback   = _recvTextCallback
        webSocket.RecvBinaryCallback = _recvBinaryCallback
        webSocket.ClosedCallback     = _closedCallback
    
    def _recvTextCallback(webSocket, msg) :
        print("WS RECV TEXT : %s" % msg)
        webSocket.SendText("Reply for %s" % msg)
    
    def _recvBinaryCallback(webSocket, data) :
        print("WS RECV DATA : %s" % data)
    
    def _closedCallback(webSocket):
        print("WS CLOSED")


    srv = WebSrv()
    srv.MaxWebSocketRecvLen     = 256
    srv.WebSocketThreaded       = True
    srv.AcceptWebSocketCallback = _acceptWebSocketCallback
    srv.Start(threaded=True)

### 模板使用（不推荐使用，推荐前后端分离）


*模板是嵌入了py语句的HTMl文件，例如：


    <html>
      <head>
        <title>TEST PYHTML</title>
      </head>
      <body>
        <h1>BEGIN</h1>
        {{ py }}
          def _testFunction(x) :
            return "IN TEST FUNCTION %s" % x
        {{ end }}
        <div style="background-color: black; color: white;">
          {{ for toto in range(3) }}
            This is an HTML test...<br />
            TOTO = {{ toto + 1 }} !<br />
            {{ for toto2 in range(3) }}
              TOTO2 = {{ _testFunction(toto2) }}
            {{ end }}
            Ok good.<br />
          {{ end }}
        </div>
        {{ _testFunction(100) }}<br />
        <br />
        {{ if 2+5 < 3 }}
          IN IF (1)
        {{ elif 10+15 != 25 }}
          IN ELIF (2)
        {{ elif 10+15 == 25 }}
          IN ELIF (3)
        {{ else }}
          IN ELSE (4)
        {{ end }}
      </body>
    </html>
|指令|使用|
|--|---|
|PY|{{ py }}py 代码 {{ end }} |
|IF|{{ if  condition }} html bloc {{ end }}|
|ELIF|{{ elif MicroPython condition }} html bloc {{ end }}|
|ELSE|{{ else }} html bloc {{ end }}|
|FOR|	{{ for identifier in  iterator }} html bloc {{ end }}|
|INCLUDE|	{{ include pyhtml_filename }}|
|?|{{ MicroPython expression }}|


###### {{ py }} :
    {{ py }}
      import machine
      from utime import sleep
      test = 123
      def testFunc(x) :
        return 2 * x
    {{ end }}



######  {{ if ... }} :
    {{ if testFunc(5) <= 3 }}
      <span>titi</span>
    {{ elif testFunc(10) >= 15 }}
      <span>tata</span>
    {{ else }}
      <span>I like the number {{ test }} !</span>
    {{ end }}
    
######  {{ for ... }} :
    {{ for toto in range(testFunc(3)) }}
      <div>toto x 10 equal {{ toto * 10 }}</div>
      <hr />
    {{ end }}

######  {{ include ... }} :
    {{ include myTemplate.pyhtml }}

### 完整实例
    @WebSrv.route('/json')
    def json(httpClient, httpResponse) :
        httpResponse.WriteResponseJSON({1:2,"test":5})#返回json字符串
    
    
    @WebSrv.route('/edit/<index>')  # <IP>/edit/123           ->   args['index']=123
    @WebSrv.route('/edit/<index>/abc/<foo>')  # <IP>/edit/123/abc/bar   ->   args['index']=123  args['foo']='bar'
    @WebSrv.route('/edit')  # <IP>/edit               ->   args={}
    def _httpHandlerEditWithArgs(httpClient, httpResponse, args={}):
        content = """\
        <!DOCTYPE html>
        <html lang=en>
            <head>
                <meta charset="UTF-8" />
                <title>TEST EDIT</title>
            </head>
            <body>
        """
        content += "<h1>EDIT item with {} variable arguments</h1>" \
            .format(len(args))
    
        if 'index' in args:
            content += "<p>index = {}</p>".format(args['index'])
    
        if 'foo' in args:
            content += "<p>foo = {}</p>".format(args['foo'])
    
        content += """
            </body>
        </html>
        """
        httpResponse.WriteResponseOk(headers= None,
                                      contentType= "text/html",
                                      contentCharset= "UTF-8",
                                      content       = content )
    
    
    
    @WebSrv.route('/test', 'POST')
    def _httpHandlerTestPost(httpClient, httpResponse) :
        formData  = httpClient.ReadRequestPostedFormData()
        firstname = formData["firstname"]
        lastname  = formData["lastname"]
        content   = """\
        <!DOCTYPE html>
        <html lang=en>
            <head>
                <meta charset="UTF-8" />
                <title>TEST POST</title>
            </head>
            <body>
                <h1>TEST POST</h1>
                Firstname = %s<br />
                Lastname = %s<br />
            </body>
        </html>
        """ % ( WebSrv.HTMLEscape(firstname),
                WebSrv.HTMLEscape(lastname) )
        httpResponse.WriteResponseOk( headers        = None,
                                      contentType    = "text/html",
                                      contentCharset = "UTF-8",
                                      content        = content )
    
    
    
    def _acceptWebSocketCallback(webSocket, httpClient) :

        print("WS ACCEPT")
        webSocket.RecvTextCallback   = _recvTextCallback
        webSocket.RecvBinaryCallback = _recvBinaryCallback
        webSocket.ClosedCallback     = _closedCallback
    
    def _recvTextCallback(webSocket, msg) :
        print("WS RECV TEXT : %s" % msg)
        webSocket.SendText("Reply for %s" % msg)
    
    def _recvBinaryCallback(webSocket, data) :
        print("WS RECV DATA : %s" % data)
    
    def _closedCallback(webSocket):
        print("WS CLOSED")
    
    
    
    srv = WebSrv(ip='0.0.0.0',port=8888,templatePath="/usr/www/", staticPath="/usr/www/", staticPrefix="/static/")
    srv.MaxWebSocketRecvLen     = 256
    srv.WebSocketThreaded       = True
    srv.AcceptWebSocketCallback = _acceptWebSocketCallback
### API
### class WebSrv(ip='0.0.0.0',port=80,templatePath="/usr/www/", staticPath="/usr/www/", staticPrefix="/static/")

&nbsp; &nbsp; <font size=3>创建web服务器</font>

* &nbsp; 参数：

|参数|解释|
|--|---|
|ip|服务器IP|
|port|监听端口|
|templatePath|模板路径|
|staticPath|静态文件路径|
|staticPrefix|静态文件url前缀|

###### WebSrv.Start(threaded=True)

&nbsp; &nbsp; <font size=3>开启服务器</font>

* &nbsp; 参数：

|参数|解释|
|--|---|
|threaded|是否运行在子线程中|
* &nbsp; 返回值：

无
    
### WebSrv.Stop()

&nbsp; &nbsp; <font size=3>关闭web服务</font>

* &nbsp; 返回值：

无

###### WebSrv.IsStarted()()

&nbsp; &nbsp; <font size=3>  检查服务器状态，True代表服务器在运行，反之False</font>

* &nbsp; 返回值：

True 服务器在运行

False 服务器停止运行

###### static WebSrv.route(url, method='GET')

&nbsp; &nbsp; <font size=3>    路由装饰器</font>

* &nbsp; 参数：

|参数|解释|
|--|---|
|url|路由 |
|method|方法|


###### WebSrv.AcceptWebSocketCallback

&nbsp; &nbsp; <font size=3>     websocket可用回调函数，例如mws.AcceptWebSocketCallback = _acptWS _acptWS(webSocket, httpClient) { }</font>


###### WebSrv.MaxWebSocketRecvLen

&nbsp; &nbsp; <font size=3>  websocket最大接受缓存默认：1024</font>


###### WebSrv.WebSocketThreaded

&nbsp; &nbsp; <font size=3>     websocket是否开启新线程</font>


### HttpClient

###### httpClient.GetServer()

&nbsp; &nbsp; <font size=3>      获取服务器实例</font>

###### httpClient.GetAddr()

&nbsp; &nbsp; <font size=3>         获取客户端地址</font>


###### httpClient.GetIPAddr()

&nbsp; &nbsp; <font size=3>         获取客户端ip</font>



###### httpClient.GetPort()

&nbsp; &nbsp; <font size=3>         获取客户端端口</font>


###### httpClient.GetRequestMethod()

&nbsp; &nbsp; <font size=3>         获取请求方法</font>


###### httpClient.GetRequestQueryString()

&nbsp; &nbsp; <font size=3>             获取请求查询字符串</font>


###### httpClient.GetRequestQueryParams()

&nbsp; &nbsp; <font size=3>            请求查询参数作为列表返回</font>

###### httpClient.GetRequestHeaders()

&nbsp; &nbsp; <font size=3>          获取请求头为列表返回</font>


###### httpClient.ReadRequestPostedFormData()

&nbsp; &nbsp; <font size=3>          获取form参数</font>



###### httpClient.ReadRequestContentAsJSON()()

&nbsp; &nbsp; <font size=3>         获取json提交数据</font>



### HttpResponse


###### httpResponse.WriteResponse(code, headers, contentType, contentCharset, content)

&nbsp; &nbsp; <font size=3>         写返回</font>



###### httpResponse.render(filepath, headers=None, vars=None,escape=True)

&nbsp; &nbsp; <font size=3>        渲染模板</font>



###### httpResponse.render(filepath, headers=None, vars=None,escape=True)

&nbsp; &nbsp; <font size=3>           渲染模板</font>



###### httpResponse.WriteResponseFile(filepath, contentType=None, headers=None)

&nbsp; &nbsp; <font size=3>               写文件</font>


###### httpResponse.WriteResponseFileAttachment(filepath, attachmentName, headers=None)

&nbsp; &nbsp; <font size=3>                写附件</font>



###### httpResponse.WriteResponseJSON(obj=None, headers=None)

&nbsp; &nbsp; <font size=3>                 返回json</font>


###### httpResponse.WriteResponseRedirect(location)

&nbsp; &nbsp; <font size=3>                重定向</font>


######  httpResponse.WriteResponseError(code)

&nbsp; &nbsp; <font size=3>                 定义错误码</font>



###### httpResponse.WriteResponseBadRequest()


###### httpResponse.WriteResponseForbidden()


###### httpResponse.WriteResponseNotFound()



###### httpResponse.WriteResponseMethodNotAllowed()



###### httpResponse.WriteResponseInternalServerError()



###### httpResponse.WriteResponseNotImplemented()

### WebSocket


###### websocket.RecvTextCallback

&nbsp; &nbsp; <font size=3>                  接收文本消息回调：websocket.RecvTextCallback = func(webSocket, msg)</font>



###### websocket.RecvTextCallback

&nbsp; &nbsp; <font size=3>                   接收二进制消息回调：websocket.RecvBinaryCallback = func(webSocket, msg)</font>





###### websocket.ClosedCallback

&nbsp; &nbsp; <font size=3>                      连接关闭回调：websocket.ClosedCallback = func(webSocket)</font>





###### websocket.SendText(msg)

&nbsp; &nbsp; <font size=3>              发送文本</font>



###### websocket.SendBinary(msg)

&nbsp; &nbsp; <font size=3>              发送二进制</font>



###### websocket.Close()

&nbsp; &nbsp; <font size=3>              关闭连接</font>



###### websocket.IsClosed()

&nbsp; &nbsp; <font size=3>                  获取连接状态</font>