**参考博客**
[理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)
[RESTful API 最佳实践](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)

[toc]

# 概述
**RESTful API**，全称为**Representational State Transfer**，即表现层状态转化。

### 资源
表现层状态转化，省略的主语即为**资源**。网络互动（包括但不仅限于API请求）会产生一系列动作，而资源就是动作操作的抽象实体（包含相关数据）。比如用户注册，注册是动作，用户就是资源；（展示）好友列表，查询是动作，好友列表就是资源；等等。
### 表现层
不同类型的资源甚至完全相同的资源，也可能有不同的表现形式。

不同类型的资源，例如一段文本，可能有txt、HTML、XML等不同的表现方式；完全相同的资源，需要的数据也可能不同，比如需要相同用户的名字或email等。
### 状态转化
客户端与服务端交互是通过HTTP协议的，HTTP是一个**无状态**协议，因此客户端的状态信息都保存在服务端上。那么客户端与服务端的交互，就是通过HTTP协议来让服务端发生“**状态转化**”，这种状态转化是建立再表现层上的，因此叫做表现层状态转化。

---

# URL设计
### 动词+宾语
RESTful API的核心思想就是，客户端发出的数据操作指令都是“动词+宾语”的结构。比如```GET /users```这个命令，```GET```是动词，```users```是宾语。
动词通常就是HTTP的五种方法，对映CURD操作。
|HTTP方法|CURD操作|
|-|-|
|GET|读取（Read）
|POST|新建（Create）
|PUT|更新（Update）
|PATCH|更新（Update），通常是部分更新
|DELETE|删除（Delete）

>- 根据HTTP规范，动词一律大写。
>- 需要强制在URL中增加版本号```/v1/users```。
### 动词的覆盖
有些客户端只能用```GET```和```POST```两种方法，因此客户端需要通过HTTP的```X-HTTP-Method-Override```属性告诉用哪个动词（```PUT```，```PATCH```，```DELETE```）替换```POST```方法。
```
POST /api/Users/4 HTTP/1.1  
X-HTTP-Method-Override: PUT
```
### 宾语
宾语必须是名词！且最好为复数！
```
GET /users/2
```
同时要避免多级URL，除了第一级，其他级别都用查询字符串表达。
```
GET /users/12/phone=2
GET /users/12?phone=2  //better
```

---

# 状态码
客户端的每一次请求，服务器必须回应HTTP状态码和数据两部分内容。
HTTP状态码（[wiki](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)）是一个三位数，分成5个类别：
>- 1xx：相关信息
>- 2xx：操作成功
>- 3xx：重定向，请求的资源位置发生变化
>- 4xx：客户端发送的请求有错误
>- 5xx：服务器错误

### 验证、授权和限流
- 验证失败返回**401 Unauthorized**
- 授权失败返回**403 Forbidden**
- 超过流量请求返回**429 Too many requests**
>限流通过在响应头中增加以下属性来告知客户端使用情况：
>- X-RateLimit-Limit：用户每个小时允许发送请求的最大值
>- X-RateLimit-Remaining：当前时间窗口剩下的可用请求数目
>- X-RateLimit-Rest：时间窗口重置的时候，到这个时间点可用的请求数量就会变成 X-RateLimit-Limit 的值
>
>没有登录用户通过IP来判断，登录用户则通过用户信息判断。

---

# 服务器回应
### 数据格式
客户端请求HTTP头的```ACCEPT```属性应该设置为```application/json```，且服务器响应HTTP头的```Content-Type```属性也应该设置为```application/json```。
### 信息明确
当请求发生错误时，服务器除了要返回合适的状态码外，还需要再HTTP响应正文提供有用的错误提示和详细的描述。
```
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid payoad.",
  "detail": {
     "yearOfBirth": "This field is required."
  }
}
```
>属性应该使用**小驼峰命名法**。
### 提供链接
在响应参数增加**links**参数，来提供可动态变化的其他不同功能的API。数据分页即可通过此方式提供上一页、下一页的链接。
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "In progress",
   "links": {[
    { "rel":"cancel", "method": "delete", "href":"/api/status/12345" } ,
    { "rel":"edit", "method": "put", "href":"/api/status/12345" }
  ]}
}
```