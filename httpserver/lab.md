# 实验记录

1. 运行python代码

```python
# -*- coding: utf-8 -*-

import sys
import cgi
from http.server import HTTPServer, BaseHTTPRequestHandler


class MyHTTPRequestHandler(BaseHTTPRequestHandler):
    field_name = 'a'
    form_html = \
        '''
        <html>
        <body>
        <form method='post' enctype='multipart/form-data'>
        <input type='text' name='%s'>
        <input type='submit'>
        </form>
        </body>
        </html>
        ''' % field_name

    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        try:
            file = open("."+self.path, "rb")
        except FileNotFoundError as e:
            print(e)
            self.wfile.write(self.form_html.encode())
        else:
            content = file.read()
            self.wfile.write(content)

    def do_POST(self):
        form_data = cgi.FieldStorage(
            fp=self.rfile,
            headers=self.headers,
            environ={
                'REQUEST_METHOD': 'POST',
                'CONTENT_TYPE': self.headers['Content-Type'],
            })
        fields = form_data.keys()
        if self.field_name in fields:
            input_data = form_data[self.field_name].value
            file = open("."+self.path, "wb")
            file.write(input_data.encode())

        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(b"<html><body>OK</body></html>")


class MyHTTPServer(HTTPServer):
    def __init__(self, host, port):
        print("run app server by python!")
        HTTPServer.__init__(self,  (host, port), MyHTTPRequestHandler)


if '__main__' == __name__:
    server_ip = "0.0.0.0"
    server_port = 8080
    if len(sys.argv) == 2:
        server_port = int(sys.argv[1])
    if len(sys.argv) == 3:
        server_ip = sys.argv[1]
        server_port = int(sys.argv[2])
    print("App server is running on http://%s:%s " % (server_ip, server_port))

    server = MyHTTPServer(server_ip, server_port)
    server.serve_forever()

```

2. 运行时访问网站

   代码跑起来以后，就可以在浏览器中访问 http://127.0.0.1:8080/a.html

   <img src="pic\1.png" style="zoom:50%;" />

3. 出错参考

   出错在于MyHTTPRequestHandler部分，参考下面的网站

   > https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods

   直接在浏览器中输入链接时，浏览器拿到地址以后，默认是采用GET方式向服务器发送请求。

   从服务器获取数据，使用get方面，向服务器提交数据，使用post方法。

4. 在23行下断点，重新运行，刷新网站后会失败，也就是代码命中中断

   <img src="pic\2.png" style="zoom:80%;" />

5. 提交数据，页面会出现“OK”

   44-47行处理了用户提交，写入文件。在 49-52行返回。

   <img src="pic\5.png" style="zoom:80%;" />

6. 如果填入“123”，就会多一个a.html文件，将“123”存在里面

   然后你下次再访问 http://127.0.0.1:8080/a.html 时，在浏览器地址栏里回车，由于这个时候a.htm已经存在了所以是运行的31-33行的else部分，直接把文件内容会写给浏览器。这里时在简化模拟用户提交数据-存入数据-其他用户获取这个数据的过程。这里有就XSS漏洞了。

   <img src="pic\6.png" style="zoom:80%;" />

7. 将a.html改为访问b.html，又会出现那个默认的form。如果这时我们输入

```
<html><body><script>alert('XSS')</script></form></body></html>
```

这段内容就会被写入b.html,然后在访问b.html的时候，整个页面被载入 script在浏览器上执行.也就是“用户提交的数据被执行了”

理论上，任何的js都是可以被执行的。js可以实现非常丰富的功能。

<img src="pic\7.png" style="zoom:80%;" />

<img src="pic\8.png" style="zoom:80%;" />

8. 比如在 c.html里填入

```
<html><body><script>window.location.href='http://by.cuc.edu.cn'</script></form></body></html>
```

下次再访问c.html的时候。页面跳转了。这就是 window.location.href='http://by.cuc.edu.cn' 这段脚本的功能。

<img src="pic\9.png" style="zoom:80%;" />

TIPS:不要用刷新,刷新是重复上一次的POST请求

9. 如果大家在浏览器中访问http://127.0.0.1:8080/httpserver.py   在开发者工具的sources中，是我们完整的源代码。

   <img src="pic\10.png" style="zoom:80%;" />

   由于服务器没有做任何过滤，只要是存在的文件，就发送给客户端，源代码文件也发送给了客户端。

   现在黑客可以知道整个后台的逻辑了。如果还有一些配置文件，比如数据库地址和访问口令等。

10. 更严重的是，黑客甚至可以注入后端代码。由于我们是回写到文件，你可以构造一个http post请求，把httpserver.py文件改写了。但是构造这个请求用浏览器就不行了，需要采用curl等更基础的工具裸写post请求发送给服务器的.

    在调试工具的 elements处，由于后台只处理名为a的表单项，所以我们需要把input的name改为a

    再从提交内容处输入hahah，会发现py文件发生了改变

    <img src="pic\11.png" style="zoom:80%;" />

<img src="pic\12.png" style="zoom:80%;" />