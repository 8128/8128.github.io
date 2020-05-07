---
title: RESTful API 设计指南
comments: true
toc: true
toc_number: false
copyright: true
mathjax: false
katex: false
hide: false
date: 2020-04-23 15:32:03
tags:
categories:
description:
cover: https://raw.githubusercontent.com/8128/PicGo/master/20200423153556.png
top_img:
keywords:
---

原文地址：

- [阮一峰](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

- [tailnode](https://segmentfault.com/a/1190000008997508)

网络应用程序，分为前端和后端两个部分。当前的发展趋势，就是前端设备层出不穷（手机、平板、桌面电脑、其他专用设备......）。

因此，必须有一种统一的机制，方便不同的前端设备与后端进行通信。这导致API构架的流行，甚至出现["API First"](http://www.google.com.hk/search?q=API+first)的设计思想。[RESTful API](http://en.wikipedia.org/wiki/Representational_state_transfer)是目前比较成熟的一套互联网应用程序的API设计理论。我以前写过一篇[《理解RESTful架构》](http://www.ruanyifeng.com/blog/2011/09/restful.html)，探讨如何理解这个概念。

今天，我将介绍RESTful API的设计细节，探讨如何设计一套合理、好用的API。我的主要参考了两篇文章（[1](http://codeplanet.io/principles-good-restful-api-design/)，[2](https://bourgeois.me/rest/)）。

![](https://raw.githubusercontent.com/8128/PicGo/master/20200423153556.png)

## 协议

API与用户的通信协议，总是使用[HTTPs协议](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)。

## 域名

应该尽量将API部署在专用域名之下。

> ```javascript
> https://api.example.com
> ```

如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下。

> ```javascript
> https://example.org/api/
> ```

## 版本（Versioning）

应该将API的版本号放入URL。

> ```javascript
> https://api.example.com/v1/
> ```

另一种做法是，将版本号放在HTTP头信息中，但不如放入URL方便和直观。[Github](https://developer.github.com/v3/media/#request-specific-version)采用这种做法。

## 路径（Endpoint）

路径又称"终点"（endpoint），表示API的具体网址。

在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。

举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样。

> - https://api.example.com/v1/zoos
> - https://api.example.com/v1/animals
> - https://api.example.com/v1/employees

## API命名规则

> 翻译自 [API Design Guide - Naming Conventions](https://cloud.google.com/apis/design/naming_convention)

为了在长期和大量使用的 API 中提供统一的开发体验，API 中的所有名字 **应该（should）** :

- 简单
- 直观
- 一致

此文章讨论了接口、资源、集合、方法和消息的名字。

因为很多开发者的母语并不是英语，这些命名约定通过鼓励使用简单、统一的短词汇来命名方法和资源来保证大部分开发者能够容易理解 API。

- API 中的名字 **应该（should）** 使用美式英语，例如：license（不是 licence），color（不是 colour）。
- 使用常用的简短形式或缩写，例如： API 比 Application Programming Interface 更好。
- 尽量使用直接、熟悉的术语，例如：描述对资源的删除（销毁）时，delete 比 erase 更好。
- 对相同的概念使用相同的名字或术语，包括在 API 中共享的概念。
- 避免名字重用，对不同的概念要使用不同名字。
- 避免使用在 API 上下文中会造成混乱的过于通用的名字，它们会导致对 API 概念的误解。应该选择能够精确描述概念的名字。这对于定义一阶 API 元素的名称尤其重要。因为名字与上下文相关，所以并没有明确的名字黑名单。Instance, info 和 service 是会产生问题的名字。应该选择能够明确表达出 API 概念（例：instance 表示什么的实例？）并且容易与其他相关概念有区分（例：alert 的意思是规则，信号还是通知？）的名字。
- 谨慎使用会与常用编程语言中的关键字有冲突的名字。

### 产品名

产品名是指 API 的产品营销名称，例如 *Google Calendar API*。在 API、UI、文档、服务条款、结账单和商业合同中使用的产品名称 **应该（should）** 一致。

Google API **必须（must）** 使用 *Google* 作为产品名的前缀，除非它们像 *Gmail*, *Nest*, *Youtube* 这种有不同的品牌。一般来说产品名 **应该（should）** 由产品和市场部门决定。

下表列出了所有相关 API 名称及其一致性的示例，有关各自名称及其约定的更多详细信息，请继续往下看。

| API 名 | 示例 |
| - | - |
| 产品名 | Google Calendar API |
| 服务名 | calendar.googleapis.com |
| 包名 | google.calendar.v3 |
| 接口名 | google.calendar.v3.CalendarService |
| 资源目录 | //google/calendar/v3 |
| API 名 | calendar |

### 服务名

服务名 **应该（should）** 是一个能够被解析为一个或多个网络地址的合法 [DNS 名字](http://www.ietf.org/rfc/rfc1035.txt)。公有 Google API 的服务名遵循如下模式：`xxx.googleapis.com`。例如：谷歌日历的服务名是 `calendar.googleapis.com`。

如果一个 API 由多个服务组成，它们 **应该（should）** 以能够提高可发现性的方法命名。一种方法是为这些服务名使用相同的前缀。例如服务 `build.googleapis.com` 和 `buildresults.googleapis.com` 都是 Google Build API 的一部分。

### 包名

在 .proto 文件中定义的包名 **应该（should）** 与产品名和服务名相同。有版本号的包名 **必须（must）** 以版本号结尾。例如：

```
// Google Calendar API
package google.calendar.v3;
```

不与服务直接关联的抽象 API，例如 Google Watcher API **应该（should）** 使用与产品名相同的 proto 包名：

```
// Google Watcher API
package google.watcher.v1;
```

在 .proto 文件中指定的 Java 包名 **必须（must）** 符合标准 Java 包名的前缀（`com.`, `edu.`, `net.` 等）。例如：

```
package google.calendar.v3;

// 指定 Java 包的名字，使用标准前缀 "com."
option java_package = "com.google.calendar.v3";
```

### 集合 ID

[集合 ID](http://tailnode.tk/2017/03/google-api-design-guide/resource-names/#collectionid) **应该（should）** 使用美式英语的、复数形式的、首字母小写的驼峰命名法，例如：`events`, `children` 或 `deletedEvents`。

### 接口名

为了避免和形如 `pubsub.googleapis.com` 的[资源名](http://tailnode.tk/2017/04/google-api-design-guide/naming-conventions/#service_name)混淆，术语 *接口名* 指的是在 .proto 文件中定义 `service` 时使用的名字：

```
// Library is the interface name.
service Library {
  rpc ListBooks(...) returns (...);
  rpc ...
}
```

你可以认为 *服务名* 是指一组 API 的具体实现，*接口名* 是指一个 API 的抽象定义。

接口名称 **应该（should）** 使用直观的名词，如 Calendar 或 Blob。**不应该（should not）** 与编程语言中已有的任何概念或运行时库冲突（如：File）。

当 *接口名* 与 API 中其他名字冲突时，应该为其加上前缀（如 `Api` 或 `Service`）来进行区分。

### 方法名

在其 IDL 规范中，服务 **可以（may）** 定义与集合和资源上的方法相对应的一个或多个RPC方法。方法名 **应该（should）** 遵循像 `VerbNoun` 这样首字母大写的驼峰命名法的命名约定，其中名词（Noun）通常是资源类型。

| 动词（Verb） | 名词（Noun） | 方法名 | 请求信息 | 响应信息 |
| -| -| - | - | - |
| List | Book | ListBooks | ListBooksRequest | ListBooksResponse |
| Get | Book | GetBook | GetBookRequest | Book |
| Create | Book | CreateBook | CreateBookRequest | Book |
| Update | Book | UpdateBook | UpdateBookRequest | Book |
| Rename | Book | RenameBook | RenameBookRequest | RenameBookResponse |
| Delete | Book | DeleteBook | DeleteBookRequest | google.protobuf.Empty |

### 消息名

RPC 方法的请求与响应消息 **应该（should）** 以方法名分别加上 `Request` 和 `Response` 的方式命名。除非方法的请求或响应类型如下：

- 空消息（使用 `google.protobuf.Empty`）
- 资源类型
- 代表一种操作的资源

### 枚举名

枚举类型的名字 **必须（must）** 使用首字母大写的驼峰命名法。

枚举值 **必须（must）** 以下划线分隔且字母全部大写的方式来命名（例如：CAPITALIZED_NAMES_WITH_UNDERSCORES）。每个枚举值 **必须（must）** 以分号而不是逗号结尾。第一个值 **应该（should）** 命名为 ENUM_TYPE_UNSPECIFIED，用于当没有明确指定枚举值时返回。

```
enum FooBar {
  // 第一个表示默认值，并且一定等于 0
  FOO_BAR_UNSPECIFIED = 0;
  FIRST_VALUE = 1;
  SECOND_VALUE = 2;
}
```

### 字段名

在 .proto 文件中定义的字段名 **必须（must）** 以下划线分隔且字母全部小写的方式来命名（例如：lower_case_underscore_separated_names）。这些名字会遵守各编程语言的命名约定来映射到生成的代码中。

#### 重复字段名（Repeated field）

API 中的重复字段 **必须（must）** 使用合适的复数形式。这符合现有 Google API 的惯例以及外部开发人员的通常期望。

#### 时间和间隔

**应该（should）** 使用 `google.protobuf.Timestamp`并且字段名 **应该（should）** 以 `time` 结尾来表示独立于任一时区的时间点。例如 `start_time`, `end_time`。

如果表示一个活动的时间，字段名 **应该（should）** 使用 `verb_time` 格式，如 `create_time`, `update_time`。避免使用动词的过去时形式，如 `created_time` 或 `last_updated_time`。

**应该（should）** 使用 `google.protobuf.Duration` 来表示一个时间段。

```
message FlightRecord {
  google.protobuf.Timestamp takeoff_time = 1;
  google.protobuf.Duration flight_duration = 2;
}
```

如果因为遗留系统或兼容原因要使用整形来表示时间相关的字段（包含墙上时间、时间段、延迟），字段名 **必须（must）** 有如下形式：

```
xxx_{time|duration|delay|latency}_{seconds|millis|micros|nanos}
message Email {
  int64 send_time_millis = 1;
  int64 receive_time_millis = 2;
}
```

如果因为遗留系统或兼容原因要使用字符串来表示时间戳，字段名 **不应该（should not）** 包含任何单位后缀。**应该（should）** 使用形如 `2014-07-30T10:43:17Z` 的 RFC 3339 格式。

#### 日期与时间

**应该（should）** 使用 `google.type.Date` 并且字段名以 `_date` 结尾来表示独立于时区与时间的日期。如果必须以字符串表示，应该使用形如 YYYY-MM-DD 的 ISO 8601 日期格式（例如 2014-07-30）。

**应该（should）** 使用 `google.type.TimeOfDay` 并且字段名以 `_time` 结尾来表示独立于时区与日期的时间。如果必须以字符串表示，应该使用形如 HH:MM:SS[.FFF] 的 ISO 8601 的 24 小时时间格式（例如 14:55:01.672）。

```
message StoreOpening {
  google.type.Date opening_date = 1;
  google.type.TimeOfDay opening_time = 2;
}
```

#### 数量

以整形表示的数量 **必须（must）** 包含单位。

```
xxx_{bytes|width_pixels|meters}
```

如果表示物品的数量，字段名 **应该（should）** 使用 `_count` 做为后缀，如 `node_count`。

#### List 过滤字段

如果 `List` 方法支持过滤资源，包含过滤表达式的字段 **应该（should）** 命名为 `filter`。例：

```
message ListBooksRequest {
  // 父资源名
  string parent = 1;

  // 过滤表达式
  string filter = 2;
}
```

#### List 响应

`List` 方法的响应消息中包含资源列表的字段名字 **必须（must）** 是资源名的复数形式。例如，方法 `CalendarApi.ListEvents()` **必须（must）** 定义响应消息 `ListEventsResponse`，此消息中包含名为 `events` 的重复字段来用于返回资源列表。

```
service CalendarApi {
  rpc ListEvents(ListEventsRequest) returns (ListEventsResponse) {
    option (google.api.http) = {
      get: "/v3/{parent=calendars/*}/events";
    };
  }
}

message ListEventsRequest {
  string parent = 1;
  int32 page_size = 2;
  string page_token = 3;
}

message ListEventsResponse {
  repeated Event events = 1;
  string next_page_token = 2;
}
```

### 驼峰命名法

除了字段名和枚举值，在 .proto 文件中的所有定义 **必须（must）** 使用首字母大写的[驼峰命名法](https://google.github.io/styleguide/javaguide.html#s5.3-camel-case)。

### 名字缩写

对于像 `config` 和 `spec` 这种被软件工程师熟知的缩写，在 API 定义中 **应该（should）** 使用缩写而不是其完整形式，这会使代码便于读写。在正式的文档中 **应该（should）** 使用其完整形式。例如：

- config (configuration)
- id (identifier)
- spec (specification)
- stats (statistics)

## HTTP动词

对于资源的具体操作类型，由HTTP动词表示。

常用的HTTP动词有下面五个（括号里是对应的SQL命令）。

> - GET（SELECT）：从服务器取出资源（一项或多项）。
> - POST（CREATE）：在服务器新建一个资源。
> - PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
> - PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
> - DELETE（DELETE）：从服务器删除资源。

还有两个不常用的HTTP动词。

> - HEAD：获取资源的元数据。
> - OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。

下面是一些例子。

> - GET /zoos：列出所有动物园
> - POST /zoos：新建一个动物园
> - GET /zoos/ID：获取某个指定动物园的信息
> - PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
> - PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
> - DELETE /zoos/ID：删除某个动物园
> - GET /zoos/ID/animals：列出某个指定动物园的所有动物
> - DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

## 过滤信息（Filtering）

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。

下面是一些常见的参数。

> - ?limit=10：指定返回记录的数量
> - ?offset=10：指定返回记录的开始位置。
> - ?page=2&per_page=100：指定第几页，以及每页的记录数。
> - ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
> - ?animal_type_id=1：指定筛选条件

参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，GET /zoo/ID/animals 与 GET /animals?zoo_id=ID 的含义是相同的。

## 状态码（Status Codes）

服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）。

> - 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
> - 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
> - 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
> - 204 NO CONTENT - [DELETE]：用户删除数据成功。
> - 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
> - 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
> - 403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
> - 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
> - 406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
> - 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
> - 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
> - 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。

状态码的完全列表参见[这里](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。

## 错误处理（Error handling）

如果状态码是4xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。

> ```javascript
> {
>     error: "Invalid API key"
> }
> ```

## 返回结果

针对不同操作，服务器向用户返回的结果应该符合以下规范。

> - GET /collection：返回资源对象的列表（数组）
> - GET /collection/resource：返回单个资源对象
> - POST /collection：返回新生成的资源对象
> - PUT /collection/resource：返回完整的资源对象
> - PATCH /collection/resource：返回完整的资源对象
> - DELETE /collection/resource：返回一个空文档

## Hypermedia API

RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。

比如，当用户向api.example.com的根目录发出请求，会得到这样一个文档。

> ```javascript
> {"link": {
>   "rel":   "collection https://www.example.com/zoos",
>   "href":  "https://api.example.com/zoos",
>   "title": "List of zoos",
>   "type":  "application/vnd.yourformat+json"
> }}
> ```

上面代码表示，文档中有一个link属性，用户读取这个属性就知道下一步该调用什么API了。rel表示这个API与当前网址的关系（collection关系，并给出该collection的网址），href表示API的路径，title表示API的标题，type表示返回类型。

Hypermedia API的设计被称为[HATEOAS](http://en.wikipedia.org/wiki/HATEOAS)。Github的API就是这种设计，访问[api.github.com](https://api.github.com/)会得到一个所有可用API的网址列表。

> ```javascript
> {
>   "current_user_url": "https://api.github.com/user",
>   "authorizations_url": "https://api.github.com/authorizations",
>   // ...
> }
> ```

从上面可以看到，如果想获取当前用户的信息，应该去访问[api.github.com/user](https://api.github.com/user)，然后就得到了下面结果。

> ```javascript
> {
>   "message": "Requires authentication",
>   "documentation_url": "https://developer.github.com/v3"
> }
> ```

上面代码表示，服务器给出了提示信息，以及文档的网址。

## 其他

（1）API的身份认证应该使用[OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)框架。

（2）服务器返回的数据格式，应该尽量使用JSON，避免使用XML。

（完）