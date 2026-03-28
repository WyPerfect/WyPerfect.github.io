# RESTful学习

## 简介

RESTful是满足REST设计风格的API接口，REST（Representational State Transfer）指表现层状态转移。

简单一句话：

用 URL 定位资源，用 HTTP 方法表示操作，用 HTTP 状态码表示结果。

它不是协议、不是标准、不是框架，只是一套接口设计风格 / 最佳实践。

## 理解

1. 一切皆资源

- 用户、文章、订单、评论、文件…… 都是资源
- 资源用 URL 唯一标识
- URL 只代表 “资源在哪里”，不代表 “要对它做什么”

2. 用 HTTP 方法表达动作

不把操作写在 URL 里，而是用标准 HTTP 动词：

表格

| 方法 | 含义 | 典型场景 | 幂等？ |
| --- | --- | --- | --- |
| GET | 查询、获取 | 查列表、查详情 | 是 |
| POST | 新增、提交 | 创建资源 | 否 |
| PUT | 全量更新 | 替换整个资源 | 是 |
| PATCH | 局部更新 | 只改某个字段 | 否 |
| DELETE | 删除 | 删除资源 | 是 |

3. 无状态（stateless）

- 服务器不保存客户端状态
- 每次请求必须带全必要信息（token、参数等）
- 好处：易扩展、易负载均衡、不粘会话

4. 统一接口

- URL 规范
- HTTP 方法规范
- 状态码规范
- 返回格式规范（JSON）

5. 可缓存

利用 HTTP 缓存机制，提升性能。

## URL设计规范

### 基本规则

1. 用名词复数，不用动词
2. 层级用 / 分隔
3. 小写、短横线分隔，不用下划线
4. 不暴露文件后缀（.php/.html）

### 反例（不 RESTful）

/getUser
/addArticle
/deleteOrder
/api.php?act=login&uid=123

### 正例（RESTful）

GET    /users          # 用户列表
GET    /users/123      # 单个用户
POST   /users          # 创建用户
PUT    /users/123      # 全量更新用户
PATCH  /users/123      # 部分更新用户
DELETE /users/123      # 删除用户

### 关联资源写法

GET /users/123/orders    # 用户123的订单
GET /articles/456/comments # 文章456的评论

### 分页、筛选、排序（用查询参数）

GET /users?page=1&size=10
GET /users?gender=male&age=20
GET /users?sort=created_at,desc

## HTTP 状态码使用规范

RESTful 非常依赖状态码，前端可以根据状态码直接处理

### 常用分类

- 1xx：信息（一般不用）
- 2xx：成功
- 3xx：重定向
- 4xx：客户端错（参数错、权限不足、不存在）
- 5xx：服务端错（服务器挂了、BUG）

### 最常用状态码

- 200 OK：查询 / 修改成功
- 201 Created：创建成功（POST）
- 204 No Content：删除成功（无返回体）
- 400 Bad Request：参数错误
- 401 Unauthorized：未登录
- 403 Forbidden：登录了但没权限
- 404 Not Found：资源不存在
- 405 Method Not Allowed：方法不支持
- 422 Unprocessable Entity：校验失败
- 500 Internal Server Error：服务器错误

## 请求与返回格式

### 请求

- GET：参数在 URL Query
- POST/PUT/PATCH：参数在 Body（JSON）
- 认证：Header 带 Authorization: Bearer token

### 返回统一结构

示例：

```json
{
  "code": 200,
  "msg": "成功",
  "data": {
    "id": 123,
    "name": "张三"
  }
}
```

列表：

```json
{
  "code": 200,
  "msg": "成功",
  "data": {
    "list": [...],
    "total": 100,
    "page": 1,
    "size": 10
  }
}
```

## RESTful 典型接口示例

### 用户接口

GET    /users         用户列表
GET    /users/1       用户详情
POST   /users         新建用户
PUT    /users/1       全量更新
PATCH  /users/1       只改昵称/状态
DELETE /users/1       删除用户

### 文章+评论

GET    /articles
GET    /articles/100
POST   /articles
PUT    /articles/100
DELETE /articles/100

GET    /articles/100/comments
POST   /articles/100/comments

## 容易混淆的点

### PUT vs PATCH

- PUT：整体替换，客户端传所有字段
- PATCH：只传要改的字段

### 是不是必须严格 REST？

不是，REST 是风格，不是强制标准。

实际项目中：

- 登录、支付、导出等 RPC 风格接口很常见
- 只要整体统一、清晰、易维护，就是好接口

## RESTful 优点

- 接口语义清晰，一看就懂
- 前后端分离友好
- 易于缓存、易于扩展
- 标准化，团队协作成本低
- 适合微服务、开放 API

## REST 不是什么？

- 不是必须用 HTTPS
- 不是必须用 JSON
- 不是必须用 Java/Spring
- 不是不能加自定义逻辑
- 不是接口必须完美设计才叫 RESTful

## Java 后端（Spring Boot）

Spring Boot 天然支持 RESTful：

- @RestController
- @GetMapping / @PostMapping / @PutMapping / @DeleteMapping
- @PathVariable 获取路径参数
- @RequestParam 查询参数
- @RequestBody 接收 JSON

示例：

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.getById(id);
}

@PostMapping("/users")
public User create(@RequestBody User user) {
    return userService.save(user);
}
```

### Spring Boot + RESTful 风格增删改查完整示例

使用：Spring Web + Lombok + 模拟数据（无数据库也能跑）

1. 项目依赖（pom.xml）

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

2. 实体类User.java

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Long id;
    private String username;
    private Integer age;
}
```

3. 统一返回结果 R.java

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class R<T> {
    private int code;
    private String msg;
    private T data;

    public static <T> R<T> ok(T data) {
        return new R<>(200, "success", data);
    }

    public static <T> R<T> fail(String msg) {
        return new R<>(500, msg, null);
    }
}
```

4. Service 层（模拟数据）

#### UserService.java

```java
import java.util.List;

public interface UserService {
    List<User> list();
    User getById(Long id);
    void add(User user);
    void update(User user);
    void delete(Long id);
}
```

#### UserServiceImpl.java

```java
import org.springframework.stereotype.Service;
import java.util.*;
import java.util.concurrent.atomic.AtomicLong;

@Service
public class UserServiceImpl implements UserService {

    // 模拟数据库
    private static final Map<Long, User> userMap = new HashMap<>();
    private static final AtomicLong idGenerator = new AtomicLong(1);

    static {
        userMap.put(idGenerator.get(), new User(idGenerator.getAndIncrement(), "张三", 20));
        userMap.put(idGenerator.get(), new User(idGenerator.getAndIncrement(), "李四", 25));
    }

    @Override
    public List<User> list() {
        return new ArrayList<>(userMap.values());
    }

    @Override
    public User getById(Long id) {
        return userMap.get(id);
    }

    @Override
    public void add(User user) {
        long id = idGenerator.getAndIncrement();
        user.setId(id);
        userMap.put(id, user);
    }

    @Override
    public void update(User user) {
        userMap.put(user.getId(), user);
    }

    @Override
    public void delete(Long id) {
        userMap.remove(id);
    }
}
```

#### Controller（RESTful 核心）

```java
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    /**
     * 查询所有用户
     */
    @GetMapping
    public R<List<User>> list() {
        return R.ok(userService.list());
    }

    /**
     * 根据ID查询单个用户
     */
    @GetMapping("/{id}")
    public R<User> getById(@PathVariable Long id) {
        return R.ok(userService.getById(id));
    }

    /**
     * 新增用户
     */
    @PostMapping
    public R<Void> add(@RequestBody User user) {
        userService.add(user);
        return R.ok(null);
    }

    /**
     * 修改用户
     */
    @PutMapping
    public R<Void> update(@RequestBody User user) {
        userService.update(user);
        return R.ok(null);
    }

    /**
     * 删除用户
     */
    @DeleteMapping("/{id}")
    public R<Void> delete(@PathVariable Long id) {
        userService.delete(id);
        return R.ok(null);
    }
}
```

#### 启动类

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RestfulDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(RestfulDemoApplication.class, args);
    }
}
```

#### 接口清单

| 请求方法 | 接口地址 | 功能 |
| --- | --- | --- |
| GET | /users | 查询用户列表 |
| GET | /users/1 | 查询 ID=1 的用户 |
| POST | /users | 新增用户 |
| PUT | /users | 修改用户 |
| DELETE | /users/1 | 删除 ID=1 的用户 |

#### 测试示例（JSON）

新增 POST /users

```json
{
  "username": "Tom",
  "age": 22
}
```

修改 PUT /users

```json
{
  "id": 3,
  "username": "Tom Updated",
  "age": 23
}
```
