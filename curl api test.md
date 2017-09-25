## HTTP curl Restful api

启动 [json-server](https://github.com/AaronFlower/restful-json-server) 来测试下：

### 1. OPTIONS 方法查看服务器支持什么方法

OPTIONS请求方法的主要用途有两个：

- 获取服务器支持的HTTP请求方法；也是黑客经常使用的方法。
- 用来检查服务器的性能。例如：AJAX进行跨域请求时的预检，需要向另外一个域名的资源发送一个HTTP OPTIONS请求头，用以判断实际发送的请求是否安全。

```bash
$ curl -X OPTIONS -I localhost:3000 # -X GET/POST/DELETE/OPTIONS/PUT/HEAD 
HTTP/1.1 204 No Content
X-Powered-By: Express
Vary: Origin, Access-Control-Request-Headers
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE
Content-Length: 0
Date: Mon, 25 Sep 2017 07:15:54 GMT
Connection: keep-alive
```

### 2. GET API 测试

- 简单的获取信息及关系信息

```
# -X GET 默认使用 GET  方法测试
# 获取整个 db
curl localhost:3000/db
# 获取 users 
curl localhost:3000/users
curl localhost:3000/users/1
# 获取 companies
curl localhost:3000/companies
curl localhost:3000/companies/2
# 获取隶属于 company 2 的用户
curl localhost:3000/companies/2/users # 根据 company 的主键去获取 users 的资源。内部是怎么实现的那？

```

- 使用过滤条件

  **注意:** [Percent-encoding (百分号编码) ](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters), 百分号编码（Percent-encoding)，也称 URL-encoding, 即一个“%”后面跟着两个表示该字节值的十六进制字母，字母一律采用大写形式。

  保留的关键字：

  | `!`   | `#`   | `$`   | `&`   | `'`   | `(`   | `)`   | `*`   | `+`   | `,`   | `/`   | `:`   | `;`   | `=`   | `?`   | `@`   | `[`   | `]`   |
  | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
  | `%21` | `%23` | `%24` | `%26` | `%27` | `%28` | `%29` | `%2A` | `%2B` | `%2C` | `%2F` | `%3A` | `%3B` | `%3D` | `%3F` | `%40` | `%5B` | `%5D` |

```
# 根据 name 来查询公司
curl localhost:3000/companies?name=apple
curl localhost:3000/companies?name=Microsoft&name=Apple # 用 curl 命令会忽略重复的参数
# 其实不是忽略了重复的参数，而是 「&」对于 shell 来说是保留关键字，直接写会被当成后台任务来执行的。所以
# 应该转义下 %26, 或者用单引号引用下 
curl 'localhost:3000/companies?name=Microsoft&name=Apple'
或
curl -v  -G localhost:3000/companies -d name=Google -d name=Microsoft
```

**说明**： 在通过 curl 来实现 GET 请求，指定多个参数时，可能需要 URL-encoding. 如果使用 ` -d, --data, --data-binary or --data-urlencode` 来指定多个参数时，curl 默认会使用 `POST` 方法，如果在使用这些标识后还要使用 `GET` 方法来使用，可以使用 -G / —get 来完成。如下：

```bash
curl -v  -G localhost:3000/companies -d name=Google -d name=Microsoft
*   Trying ::1...
* TCP_NODELAY set
* Connection failed
* connect to ::1 port 3000 failed: Connection refused
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 3000 (#0)
> GET /companies?name=Google&name=Microsoft HTTP/1.1
> Host: localhost:3000
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200 OK
< X-Powered-By: Express
< Vary: Origin, Accept-Encoding
< Access-Control-Allow-Credentials: true
< Cache-Control: no-cache
< Pragma: no-cache
< Expires: -1
< X-Content-Type-Options: nosniff
< Content-Type: application/json; charset=utf-8
< Content-Length: 747
< ETag: W/"2eb-Zp9wehQJ85/2xF5qJ7zloXp5Kn8"
< Date: Mon, 25 Sep 2017 08:40:57 GMT
< Connection: keep-alive
<
[
  {
    "id": "2",
    "name": "Microsoft",
    "description": "Microsoft Corporation, incorporated on September 22, 1993, is a technology company. The Company develops, licenses, and supports a range of software products, services and devices. The Company's segments include Productivity and Business Processes, Intelligent Cloud and More Personal Computing"
  },
  {
    "id": "3",
    "name": "Google",
    "description": "Google was founded by Larry Page and Sergey Brin while they were students at Stanford University. The company was officially launched in September, 1998 in a friend's garage. ... The official mission statement of the company is to “organize the world's information and make it universally accessible and useful"
  }
* Connection #0 to host localhost left intact
]
```

- 范围

```
# 查询年龄在 34 以上的 user
curl -G localhost:3000/users -d age_gte=34
# [33~34]
curl -G localhost:3000/users -d age_lte=34 -d age_gte=33
```

- 分页

```
curl -G localhost:3000/companies -d _page=1 -d _limit=2
curl -G localhost:3000/users -d _page=2 -d _limit=3
```

- 排序

```
curl -G localhost:3000/companies -d _sort=name -d _order=asc
```

- 全文检索

```
curl -G localhost:3000/companies?q=com
或
curl -G localhost:3000/companies -d q=com
```

### 3. POST API 测试

 进行操作一般都需要使用 JSON, 所以需要指定下 "Content-Type: application/json", 具体的 MIME Type 可以参考 [Media-type wiki](https://en.wikipedia.org/wiki/Media_type).

- 添加一个 user

```bash
curl -X POST -H "Content-Type: application/json" -d '{"firstName": "Eason", "lastName": "Chan"}' localhost:3000/users
# 返回结果
{                                                                                                              
  "firstName": "Eason",                                                                                        
  "lastName": "Chan",                                                                                          
  "id": "S1oc5BIoZ"                                                                                            
}  
```

### 4. PATCH API 测试 

- 更新 user , 需要指定 id

```shell
curl -X PATCH localhost:3000/users/S1oc5BIoZ -H "Content-Type: application/json" -d '{"age": "18"}' 
```

### 5. DELETE API 测试

```shell
curl -X DELETE localhost:3000/users/S1oc5BIoZ
curl -X DELETE localhost:3000/users/SyqVFSUob
```