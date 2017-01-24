#### 前言

这里介绍描述 HTTP+JSON API 的一种设计模式, 主要参考了一些知名平台的 API 设计指引, 具体见最后的参考资源.

本项目/系统/平台等 API 必须遵守此规范, 若有任何疑问和问题,请及时提出,我们会及时改进.

我们的目的/原则:

- 保持一致性，专注业务逻辑同时避免过度设计。
- 持续改进，尝试找出一种良好的、一致的、显而易见的 API 设计方法，而并不是所谓的"最终/理想模式"。

以下将分章节进行描述.

#### 基础

##### 隔离关注点

设计时通过将请求和响应之间的不同部分隔离来让事情变得简单。保持简单的规则让我们能更关注在一些更大的更困难的问题上。

请求和响应将解决一个特定的资源或集合。使用路径（path）来表明身份，body来传输内容（content）还有头信息（header）来传递元数据（metadata）。
查询参数 (query) 同样可以用来传递头信息的内容，但头信息是首选，因为他们更灵活、更能传达不同的信息。

##### URL Schema

URL 统一遵循: `{schema}://{host}:{port}/api/{version}/{the_actual_api_path}/?{query_string}`

- `schema`: `http`|`https`, 默认为 `http`; 如有必要, 所有访问API的行为都走 `https`
- `host`: 主机名; 必须为域名，严禁使用ip
- `port`: 端口号, 默认为 `80`
- `version`: 接口版本号, 默认为 `v1`
- `the_actual_api_path`: 具体接口的路径
- `query_string`: 查询参数

#### Request

##### Content-Type

请求 header 中 `Content-Type` 为 `application/json`

##### `body` 使用 `JSON` 格式

在 `PUT/PATCH/POST` 请求的正文（request bodies）中使用 `JSON` 格式数据

示例:

            {
                "id": 16784,
                "name": "Lorem Ipsum",
                "age": null,
                "relatives": [], // no relatives
                "address": null,
                "empty_object": {}
            }

- 更新字段时,必须设置正确的取值
- 如果要重置某个字段为空, 则取值为 `null`
    - 包括 int, number, string
- 对于 array 类型的字段, 如果要重置为空, 则取值为 `[]` 而非 `null`
- 对于 object 类型的字段, 如果要重置为空, 则取值为 `{}` 而非 `null`


##### API 授权认证

API 接口统一使用 `HTTP Authentication Header` 方式进行认证

- 用户登录/修改密码后, 会返回授权的 `token`, 此 `token` 标记用户身份
- 若无特殊说明所有需要授权访问的接口请求时必须在 Request 的 Header 添加:

            Authorization: {the_token}

- 否则, 会返回 `401 Unauthorized`


##### Pagination 分页
Request：所有分页请求均遵循以下规则

- page: 分页的页码,即请求所有数据页中的第几页
- per_page: 分页的大小,即每页的数据量,默认为15; 客户端可以根据需求自定义,但最大不能超过 100
- 分页参数通过 query 的形式传递, 如: `/the_actual_api_path/?page=2&per_page=25`

Response：分页元信息请求响应时, 分页元信息在 Response Header 中返回.
Header中的分页元信息:  
- X-Total: 数据总记录数
- X-Per-Page: 分页时每页大小

##### i18n 与 locale

本项目客户端以及服务端默认语言为简体中文.

客户端在发起请求时, 通过 `HTTP Header` `X-LAIKA-Locale` 的方式, 来指定目标语言, 例如:

        X-LAIKA-Locale: {the_language_locale}

目前仅支持简体中文和英文:

- 简体中文: `zh-CN`, 默认语言
- 英文: `en`

另, 服务端暂时不会记录用户选择的语言, 客户端需自行处理.


##### 自定义 User-Agent

`User-Agent` 作为 `HTTP` 标准头之一，主要用来识别用户代理， 统计、跟踪用户行为，以及为特定用户代理定制响应。
它不是 `HTTP` 请求必须包含的字段，但是用户代理请求时应该包含这个字段，该字段包含了多个产品等标记的信息。

现定义 LaiKa 产品 `User-Agent` 规范如下:

- 客户端 在通用的`User-Agent` 中额外追加 `LAIKA-{AppName}/{version}`, 示例如下

    - 客户端: `LAIKA/1.0.0`

另, 因为浏览器目前对自定义 User-Agent (HTTP 的标准头部) 支持不足, 将来会允许修改. 因此我们添加自定义Header: `X-LAIKA-App`, 取值同上.

综上, iOS/Android/Web添加 Header 规则 如下:

- 客户端(iOS/Android):
    - 追加 `LAIKA/1.0.0` 到 `User-Agent` 中;
    - 同时添加 `X-LAIKA-App`: `LAIKA/1.0.0` (方便服务端统一处理);

参考

- https://github.com/whatwg/fetch/issues/37

##### 搜索

所有涉及到搜索的地方，遵循以下规则:

- `/the/url/?q=query`，其中 `q` 为搜索条件
    - 若 `q=keyword`，则对该 `resource` 的默认字段搜索，如搜索球员，则为对名字搜索, 具体见接口定义
    - 若 `q=field:keyword`，则为对该 `resource` 的指定字段 `field` 进行搜索，默认为正则匹配
    - 若 `q=field1:keyword AND field2:keyword`，则对该 `resource` 的两个字段进行搜索，且同时满足两个条件
    - 若 `q=field1:keyword OR field2:keyword`，则对该 `resource` 的两个字段进行搜索，满足任意一个条件即可
- 暂时仅支持以上 URI Search 规则，更复杂的搜索需要搜索引擎如 `Solr`、`ElasticSearch` 支持，若有需求再更新
    - [ElasticSearch - URI Search](https://www.elastic.co/guide/en/elasticsearch/reference/1.6/search-uri-request.html)
    - [ElasticSearch - Request Body Search](https://www.elastic.co/guide/en/elasticsearch/reference/1.6/search-request-body.html)
    - [SolrQuerySyntax - Solr Wiki](http://wiki.apache.org/solr/SolrQuerySyntax)

##### 排序

所有涉及到排序的地方，遵循以下规则: `/the/url/?sort=field1:desc,field2:asc`

- `sort` 为排序的参数，可以一个，或多个 field:order 组合，使用 `,` 分割
- `desc`: 降序
- `asc`: 升序

注意:

- 对于列表数据, 默认按照 `id` 降序排列
- 对于搜索, 默认按照匹配度降序排列


#### Response

##### `body` 使用 `JSON` 格式

所有请求响应有返回数据时 body 均为 `JSON` 格式.

如果一个属性是可选的或者包含空值或null值，会考虑从JSON中去掉该属性，除非它的存在有很强的语义原因。

            {
              "volume": 10,

              // 即使 "balance" 属性值是零, 它也应当被保留,
              // 因为 "0" 表示 "均衡"
              // "-1" 表示左倾斜和"＋1" 表示右倾斜
              "balance": 0,

              // "currentlyPlaying" 是null的时候可被移除
              // "currentlyPlaying": null
            }

##### 下载文件

返回的响应头包含信息为：
`Content-Disposition: attachment; filename="%E7%90%83%E5%91%98%E6%95%B0%E6%8D%AE.xlsx"`

这里的 filename 是 URL Encode 过的值。客户端要想拿到预期的文件名，需要先 URL Decode。

参考：http://www.url-encode-decode.com/

##### 状态码

所有请求均会返回合适的状态码

为每一次的响应返回合适的HTTP状态码。 好的响应应该使用如下的状态码:

- 200: GET请求成功，及DELETE或PATCH同步请求完成，或者PUT同步更新一个已存在的资源
- 201: POST 同步请求完成，或者PUT同步创建一个新的资源
- 202: POST，PUT，DELETE，或PATCH请求接收，将被异步处理
- 204: DELETE 请求成功, 无内容返回
- 206: GET 请求成功，但是只返回一部分

使用身份认证（authentication）和授权（authorization）错误码时需要注意：

- 401 Unauthorized: 用户未认证，请求失败
- 403 Forbidden: 用户无权限访问该资源，请求失败

当用户请求错误时，提供合适的状态码可以提供额外的信息：

- 400 Bad request: 请求错误服务端无法正确解析
- 404 Not found: 资源不存在
- 422 Unprocessable Entity: 请求被服务器正确解析，但是包含无效字段
- 429 Too Many Requests: 因为访问频繁，你已经被限制访问，稍后重试
- 500 Internal Server Error: 服务器错误，确认状态并报告问题

对于用户错误和服务器错误情况状态码，参考： [HTTP response code spec](https://tools.ietf.org/html/rfc7231#section-6)

##### 错误信息

请求失败时, 错误信息以统一的 `JSON` 格式返回


            {
               "error": "简短的错误提示",
               "message": "友好的错误提示",
               "errcode": 400,
               "errors": {
                    "email":  [
                        "Email 不能为空",
                        "Email 格式不正确"
                    ]
               }
            }


其中, 字段描述如下:

- `error`: string 类型; 简短的错误提示
- `message`: string 类型; 友好的错误提示, 一般为请求错误时的提示文案
- `error_code`: int 类型; 具体的错误码, 一般与 response 的 status_code 一致
- `errors`: dict 类型; key 为出错的字段; value 为出错的提示, 统一为 list 类型
    - `errors` 一般如上示例, 帮助开发者或用户定位具体的错误信息
    - `errors` 可能为 {}

对于提交表单类错误, 如 `PUT`, `POST`, 数据校验失败时:

- `errcode` 统一为 422
- 返回详细的错误信息, 即 `errors`
- `errors` 数据层次结构与提交的表单一致, 方便客户端定位具体的错误位置


##### 提供标准的时间戳

为资源提供默认的创建时间 created_at 和更新时间 updated_at，例如:

            {
              ...
              "created_at": "2012-01-01T12:00:00Z",
              "updated_at": "2012-01-01T13:00:00Z",
              ...
            }

##### 使用 UTC + ISO8601 时间

在接收和返回时都只使用UTC格式。ISO8601 格式的数据，例如:


            {
                ...
                "finished_at": "2012-01-01T12:00:00Z"
                ...
            }


#### 参考资源

- [Thoughts on RESTful API Design](https://restful-api-design.readthedocs.org/en/latest/)
- [Google JSON Style Guide](https://google.github.io/styleguide/jsoncstyleguide.xml)
- [RFC 2617 - HTTP Authentication: Basic and Digest Access Authentication](https://tools.ietf.org/html/rfc2617)
- [RFC 7231 - Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231#section-6)
- [RFC 5988 - Web Linking](http://tools.ietf.org/html/rfc5988)
