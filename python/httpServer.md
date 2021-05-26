# Python http服务器

python可以很方便的启动一个http服务器.

## 命令行方式
一个简单的静态http服务器（只有下载文件的功能）
```bash
python -m http.server [port] -d [directory] -b [bind_address]
```

## 简单静态http服务器
```python3
import http.server
import socketserver

PORT = 8000

Handler = http.server.SimpleHTTPRequestHandler

with socketserver.TCPServer(("", PORT), Handler) as httpd:
    print("serving at port", PORT)
    httpd.serve_forever()
```

## 简单自定义http服务器
下面实现一个简单的echo server

```python3
from http.server import BaseHTTPRequestHandler, HTTPServer


class EchoHandler(BaseHTTPRequestHandler):

    def do_POST(self):
        content_length = int(self.headers["Content-Length"])
        request_body = self.rfile.read(content_length)
        print(request_body.decode("utf-8"))
        self._set_response()
        self.wfile.write("Hello World".encode("utf-8"))

    def do_GET(self):
        self._set_response()
        self.wfile.write("Hello World".encode("utf-8"))

    def _set_response(self):
        self.send_response(200)
        self.send_header("Content-Type", "text/html")
        self.end_headers()


if __name__ == "__main__":
    _port = 8500
    httpd = HTTPServer(("", _port), EchoHandler)
    print("Starting http at %s" % _port)
    httpd.serve_forever()
```
