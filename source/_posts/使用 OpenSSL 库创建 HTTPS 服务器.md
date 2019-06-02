title: 使用 OpenSSL 库创建 HTTPS 服务器
urlname: 使用 OpenSSL 库创建 HTTPS 服务器
date: 2018/03/18 18:16:17
categories:
- 实用技巧
tags:
- Rust
- TLS
- OpenSSL

---
![](https://image.covertness.me/openssl_https_server/https.png)

网上现存的大部分 OpenSSL 示例都是通过 C/C++ 编写的，本文通过更加现代的 Rust 语言来实现一个简单的 HTTPS 服务器，程序更加简洁精炼。
<!-- more -->
随着大家对网络安全的日益重视，大部分 Web 应用都开始使用 HTTPS 取代 HTTP 以提供更安全的服务。 OpenSSL 是一个封装了很多安全加密算法的类库，使用它只需要几行代码就可以给现有的 Web 应用加上安全保护。

### 生成加密所需的公钥和私钥
```bash
$ openssl req -x509 -newkey rsa:2048 -days 3650 -nodes \
    -keyout server-key.pem -out server-cert.pem
```
其中 server-key.pem 存储了私钥信息， server-cert.pem 存储的是公钥信息。

### 在项目中添加 OpenSSL 依赖，在 `Cargo.toml` 文件中添加如下信息即可
```toml
[dependencies]
openssl = "0.10.5"
```

### 实现代码
```rust
extern crate openssl;

use openssl::ssl::{Error, SslAcceptor, SslFiletype, SslMethod, SslStream};
use std::net::{TcpListener, TcpStream};
use std::sync::Arc;
use std::thread;
use std::str;

fn main() {
    let mut acceptor = SslAcceptor::mozilla_intermediate(SslMethod::tls()).unwrap();
    acceptor
        .set_private_key_file("server-key.pem", SslFiletype::PEM)
        .unwrap();
    acceptor
        .set_certificate_chain_file("server-cert.pem")
        .unwrap();
    acceptor.check_private_key().unwrap();
    let acceptor = Arc::new(acceptor.build());

    let listener = TcpListener::bind("0.0.0.0:8443").unwrap();

    println!("server is up");

    fn write_response(stream: &mut SslStream<TcpStream>) -> Result<usize, Error> {
        let mut len = 0;
        len += stream.ssl_write(b"HTTP/1.1 200 OK\r\n")?;
        len += stream.ssl_write(b"Content-Length: 10\r\n")?;
        len += stream.ssl_write(b"Content-Type: text/html\r\n\r\n")?;
        len += stream.ssl_write(b"hello rust")?;
        Ok(len)
    }

    fn handle_requset(stream: &mut SslStream<TcpStream>) -> Result<usize, Error> {
        let mut buffer = [0; 100];
        stream.ssl_read(&mut buffer)?;

        println!("request: {:?}", str::from_utf8(&buffer).unwrap());

        write_response(stream)
    }

    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                let acceptor = acceptor.clone();
                thread::spawn(move || {
                    let mut stream = acceptor.accept(stream).unwrap();

                    println!("new client");

                    match handle_requset(&mut stream) {
                        Ok(len) => {
                            println!("response success body len {}", len)
                        }
                        Err(e) => {
                            println!("response error {:?}", e)
                        }
                    }
                });
            }
            Err(e) => {
                println!("stream error {:?}", e)
            }
        }
    }
}
```
这段代码逻辑比较简单，所实现的功能仅是对所有请求都返回 `hello rust` ，但所有数据已经完成了对应的加解密操作，下面通过浏览器来验证下。

### 启动这个程序
```bash
$ cargo run
```

### 使用浏览器访问 `https://127.0.0.1:8443`
![](https://image.covertness.me/openssl_https_server/snap.png)
> Tips: 由于之前生成的公钥并没有交由第三方 CA 授权，所以浏览器会认为此证书无效，这里无需理会这个错误继续访问即可。

可以看到浏览器完整地显示了程序返回的 `hello rust` ，对应程序后台也打印了完整的 HTTP 请求头。
