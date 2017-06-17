<!-- TOC -->

- [1. API blueprint](#1-api-blueprint)
    - [1.1. 概要](#11-概要)
    - [1.2. Tutorial](#12-tutorial)
        - [1.2.1. format](#121-format)
        - [1.2.2. Resource Groups](#122-resource-groups)
        - [1.2.3. Resource](#123-resource)
        - [1.2.4. Actions](#124-actions)
        - [1.2.5. URI Template](#125-uri-template)
        - [1.2.6. URI Parameters](#126-uri-parameters)
        - [1.2.7. actions](#127-actions)
        - [1.2.8. Response without a body](#128-response-without-a-body)
    - [1.3. API Blueprint Language](#13-api-blueprint-language)
        - [1.3.1. Blueprint section](#131-blueprint-section)
            - [1.3.1.1. section结构](#1311-section结构)
            - [1.3.1.2. 关键字](#1312-关键字)
            - [1.3.1.3. 标识符](#1313-标识符)
            - [1.3.1.4. 描述](#1314-描述)
            - [1.3.1.5. 嵌套的section](#1315-嵌套的section)
        - [1.3.2. section参考](#132-section参考)
            - [1.3.2.1. 摘要](#1321-摘要)
                - [1.3.2.1.1. Named section](#13211-named-section)
                - [1.3.2.1.2. Asset section](#13212-asset-section)
                - [1.3.2.1.3. Payload section](#13213-payload-section)
        - [1.3.3. section基础](#133-section基础)
            - [1.3.3.1. Metadata section](#1331-metadata-section)
            - [1.3.3.2. API 名称和概览部分](#1332-api-名称和概览部分)
            - [1.3.3.3. Resource group section](#1333-resource-group-section)
            - [1.3.3.4. Resource section](#1334-resource-section)
            - [1.3.3.5. Resource model section](#1335-resource-model-section)
            - [1.3.3.6. Schema section](#1336-schema-section)
            - [1.3.3.7. Action section](#1337-action-section)
            - [1.3.3.8. Request section](#1338-request-section)
            - [1.3.3.9. Response section](#1339-response-section)
            - [1.3.3.10. URI parameters section](#13310-uri-parameters-section)
            - [1.3.3.11. Attributes Section](#13311-attributes-section)
            - [1.3.3.12. Headers section](#13312-headers-section)
            - [1.3.3.13. Body section](#13313-body-section)
            - [1.3.3.14. Data Structures section](#13314-data-structures-section)
            - [1.3.3.15. Relation section](#13315-relation-section)
    - [1.4. 附录](#14-附录)
        - [1.4.1. URI 模板](#141-uri-模板)
    - [1.5. 解析工具 aglio](#15-解析工具-aglio)

<!-- /TOC -->

# 1. API blueprint

## 1.1. 概要

* [官网](https://apiblueprint.org/)
* [Tutorial](https://apiblueprint.org/documentation/tutorial.html)
* [渲染工具-aglio](https://github.com/danielgtaylor/aglio)
* [语法文档](https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md)
* [示例](https://github.com/apiaryio/api-blueprint/tree/master/examples)
* [MSON Specification](https://github.com/apiaryio/mson/blob/master/MSON%20Specification.md)
* [MockServer-drakov](https://github.com/Aconex/drakov)


## 1.2. Tutorial

### 1.2.1. format

```md
FORMAT: 1A

# Polls

Polls is a simple API allowing consumers to view polls and vote in them.

这里展示一些列表：
* list item 1
* list item 2
* list item 3
```

`FORMAT` 关键字指示 API blueprint 的版本是 1A.

第一个标题是 API 的名字，上面是 `Polls`， 后面是 API 的描述, 描述中可以使用 markdown 语法。


### 1.2.2. Resource Groups

在标题行的开头使用 `Group` 关键字表示相关联的一组资源。

```apib
# Group Questions

Resources related to questions in the API.
```

### 1.2.3. Resource

定义在 Resource Group 下面, 在标题末尾的方括号内填写资源的 URI。

```md
## Questions Collection [/questions]
```

### 1.2.4. Actions

定义在 Resource 下面，标题后的方括号内填写动作( HTTP 方法).

```md
### List All Questions [GET]
```

一个 Action 至少要有一个 Response，Response 包含一个状态码，响应体可选。
Response 是一个列表项，列表项可以使用 `+`, `*` 或 `-`.

```md
+ Response 200 (application/json)

        [
            {
                "question": "Favourite programming language?",
                "published_at": "2014-11-11T08:40:51.620Z",
                "url": "/questions/1",
                "choices": [
                    {
                        "choice": "Swift",
                        "url": "/questions/1/choices/1",
                        "votes": 2048
                    }, {
                        "choice": "Python",
                        "url": "/questions/1/choices/2",
                        "votes": 1024
                    }, {
                        "choice": "Objective-C",
                        "url": "/questions/1/choices/3",
                        "votes": 512
                    }, {
                        "choice": "Ruby",
                        "url": "/questions/1/choices/4",
                        "votes": 256
                    }
                ]
            }
        ]
```
状态码后面的 media 类型会生成一个 `Content-Type` http 头，这个不是必须指定的。
response 的响应体缩进8个空格或者2个tab。

下面的例子展示如何创建一个问题：

```md
### Create a New Question [POST]

You may create your own question using this action. It takes a JSON object
containing a question and a collection of answers in the form of choices.

+ question (string) - The question
+ choices (array[string]) - A collection of choices.
```

请求时使用 JSON 格式：

```md
+ Request (application/json)

            {
                "question": "Favourite programming language?",
                "choices": [
                    "Swift",
                    "Python",
                    "Objective-C",
                    "Ruby"
                ]
            }
```

返回 201 状态码以及 http 头和 body:

```md
+ Response 201 (application/json)

    + Headers

            Location: /questions/1

    + Body

                {
                    "question": "Favourite programming language?",
                    "published_at": "2014-11-11T08:40:51.620Z",
                    "url": "/questions/1",
                    "choices": [
                        {
                            "choice": "Swift",
                            "url": "/questions/1/choices/1",
                            "votes": 0
                        }, {
                            "choice": "Python",
                            "url": "/questions/1/choices/2",
                            "votes": 0
                        }, {
                            "choice": "Objective-C",
                            "url": "/questions/1/choices/3",
                            "votes": 0
                        }, {
                            "choice": "Ruby",
                            "url": "/questions/1/choices/4",
                            "votes": 0
                        }
                    ]
                }
```

下一个资源是 "Question"，表示一个单独的问题：

```md
## Question [/questions/{question_id}]
```

### 1.2.5. URI Template

URI 中使用变量，比如上例中的 `question_id`。

### 1.2.6. URI Parameters

```md
+ Parameters
    + question_id (number) - ID of the Question in the form of an integer
```

### 1.2.7. actions

```md
### View a Questions Detail [GET]

+ Response 200 (application/json)

            {
                "question": "Favourite programming language?",
                "published_at": "2014-11-11T08:40:51.620Z",
                "url": "/questions/1",
                "choices": [
                    {
                        "choice": "Swift",
                        "url": "/questions/1/choices/1",
                        "votes": 2048
                    }, {
                        "choice": "Python",
                        "url": "/questions/1/choices/2",
                        "votes": 1024
                    }, {
                        "choice": "Objective-C",
                        "url": "/questions/1/choices/3",
                        "votes": 512
                    }, {
                        "choice": "Ruby",
                        "url": "/questions/1/choices/4",
                        "votes": 256
                    }
                ]
            }
```

### 1.2.8. Response without a body

```md
### Delete [DELETE]

+ Response 204
```










## 1.3. API Blueprint Language

FROM https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md

API blueprint 是一个面向文档的web API设计语言。基于 Markdown 语法。

一个 API blueprint 文档使用全部或部分 Markdown 文档来描述 web API. 文档按结构分成几个逻辑章节。

文档结构：

![](api_blueprint/doc_structure.png)

### 1.3.1. Blueprint section

`section` 表示 API 蓝图的逻辑单元，比如一个API预览，一组资源或者一个资源定义。

通常，section使用一个markdown实体中的关键字来定义，根据section的类型，关键字可以作为一个markdown标题或者一个列表项。

section定义中可以还会包含额外的变量组件，比如标识符或者额外的修饰符。

NOTE: 有两个特殊的section是根据他们的位置而不是关键字来识别的：Metadata section 和 api名称和简介部分。

例子：标题形式的section

```
# <keyword>

 ...

# <keyword>

 ...
```

或者：

```
<keyword>
=========

...

<keyword>
=========

...
```

例子：列表形式的section:

```
+ <keyword>

 ...

+ <keyword>

 ...
```

除了 `+`， 还可以使用 `-` 或者 `*`:

```
* <keyword>

...

- <keyword>

...
```

#### 1.3.1.1. section结构

通常，一个由关键字定义的section包含标识符(name)，描述，嵌套的section，或者特定格式的内容。

例子：标题形式：

```
# <keyword> <identifier>

<description>

<specific content>

<nested sections>
```

例子：列表形式：

```
+ <keyword> <identifier>

    <description>

    <specific content>

    <nested sections>
```

#### 1.3.1.2. 关键字

section中的关键字有以下几个

**标题关键字**

* Group
* Data Structures
* HTTP methods(e.g. GET, POST, PUT, DELETE...)
* URI模板(e.g. `/resource/{id}`)
* HTTP方法和URI模板组合(e.g. `GET /resource/{id}`)

**列表关键字**

* Request
* Response
* Body
* Schema
* Model
* Header & Headers
* Parameter & Parameters
* Values
* Attribute & Attributes
* Relation

NOTE: 尽量避免在其他 markdown 标题或者列表中使用这些关键字。
NOTE: 除了HTTP方法关键字外其他关键字是大小写敏感的。

#### 1.3.1.3. 标识符

section中可能或者必须包含一个标识符。标识符是任意非空字符的组合，除了 `[`, `]`, `(`, `)` 和换行符。标识符中不能包含关键字。

例子：

```
Adam's Message 42
my-awesome-message_2
```

#### 1.3.1.4. 描述

描述是section定义后面的任意的markdown格式的内容。

描述中可以适应任意的markdown标题或者列表，只有不与保留关键字冲突即可。

#### 1.3.1.5. 嵌套的section

根据section的类型来嵌套section，比如增加标题级别或者增加列表项的缩进。

section开始处和紧跟着的下一个同等级的section之间的任意内容都属于该section.

例子：嵌套的标题形式的section:

```
# <section definition>

 ...

## <nested section definition>

 ...
```

例子：嵌套的列表形式的section:

```
+ <section definition>

     ...

    + <nested section definition>

     ...
```

### 1.3.2. section参考

#### 1.3.2.1. 摘要

##### 1.3.2.1.1. Named section

定义为一个关键字紧跟着可选的名称：

```
# <keyword> <identifier>
+ <keyword> <identifier>
```

这是基本形式，由标识符、描述、嵌套section或者特定格式的内容组成。

例子：

```
# <keyword> Section Name
This the `Section Name` description.

- one
- **two**
- three

<nested sections> |  <formatted content>
```

##### 1.3.2.1.2. Asset section

仅由关键字来定义。

```
+ <keyword>
```

API blueprint中原子数据的基本section。内容是格式化的代码块。

例子：

```
+ <keyword>

        {
            "message": "Hello"
        }
```

例子：Fenced code blocks

```
+ <keyword>

    ```
    {
        "message": "Hello"
    }
    ```
```

##### 1.3.2.1.3. Payload section

由一个在列表项中的关键字来定义，关键字后可以跟一个标识符，还可能包含由括号括起来的media类型。

```
+ <keyword> <identifier> (<media type>)
```

表示HTTP请求或响应的负载信息。由可选的http头形式的meta信息和可选的http body形式的内容组成。

此外，负载(payload)还包含它本身的描述，消息体属性的描述，消息体验证模式的描述。

负载可能与其media类型相关联。负载的media类型表示以HTTP的`Content-Type`头形式接收或发送的元数据。指定一个负载时应包含嵌套的Body section.

这个section应包含以下嵌套section中至少一个：
* 0-1 Headers section
* 0-1 Attributes section
* 0-1 Body section
* 0-1 Schema section

如果没有嵌套的section，负载的内容会被考虑作为Body section.

**Body, Schema 和 Attributes section的关系**

每个body, schema和attributes section描述一个信息负载的body.这些描述应该是一致的，相互不能冲突。当有多个body描述时应该按照以下优先级：

1. 对于解析消息体模式
    1. Schema section
    2. Attributes section
    3. Body section
2. 对于解析消息体例子
    1. Body section
    2. Attributes section
    3. Schema section

**引用**

除了提供payload section的内容外，还可以引用一个model payload section:

```md
[<identifier>][]
```

例子：

```
+ <keyword> Payload Name (application/json)

    This the `Payload Name` description.

    + Headers

     ...

    + Body

     ...

    + Schema

    ...
```

引用model payload:

```
+ <keyword> Payload Name

    [Resource model identifier][]
```


### 1.3.3. section基础

#### 1.3.3.1. Metadata section

键值对。键值之间用 `:` 分隔。每个键值对一行。在文档开头定义，结束于第一个不能解析为键值对的markdown元素。

元数据的键值对是作用于工具的，关于支持的键的列表需要查询相关工具的文档。

例子：

```
FORMAT: 1A
HOST: http://blog.acme.com
```

#### 1.3.3.2. API 名称和概览部分

第一个markdown标题会被当做名称和概览的部分，除非这个标题表示其他的section定义。（表示这个section可有可无）

例子：

```
# Basic ACME Blog API
Welcome to the **ACME Blog** API. This API provides access to the **ACME
Blog** service.
```

#### 1.3.3.3. Resource group section

以 `Group` 关键字开头，后面跟组的名称。

```
# Group <identifier>
```

代表一组资源，可能会包含一个或多个嵌套的Resource section.

例子：

```
# Group Blog Posts

## Resource 1 [/resource1]

 ...

# Group Authors
Resources in this groups are related to **ACME Blog** authors.

## Resource 2 [/resource2]

 ...
```


#### 1.3.3.4. Resource section

URI模板形式：

```
# <URI template>
```

或者 URI 模板前加上标识符：

```
# <identifier> [<URI template>]
```

或者 URI 模板前添加 HTTP 方法：

```
# <HTTP request method> <URI template>
```

或者标识符加 http 方法加uri模板：

```
# <identifier> [<HTTP request method> <URI template>]
```

API 资源由它的URI指定，或者由匹配URI模板的资源集合指定。

这个section至少应包含一个嵌套的 Action section，可能包含以下嵌套的 section:
* 0-1 URI parameters section
    在 Resource section范围内定义的URI模板适用于任何嵌套的action section，除非为Action定义了模板。
* 0-1 Attribute section
    在 Resource section 范围内定义的属性表示资源属性。如果资源使用一个名字定义属性，那么这些属性可能引用了属性section。
* 0-1 Model section
* 额外的 Action section

例子：

```
# Blog Posts [/posts/{id}]
Resource representing **ACME Blog** posts.

# /posts/{id}

# GET /posts/{id}
```


#### 1.3.3.5. Resource model section

使用 `Model` 关键字定义，后面可跟media类型。

```
+ Model (<media type>)
```

在这个section中定义的payload可以使用父section的标识符在任意响应或请求部分引用。

例子：

```
# My Resource [/resource]

+ Model (text/plain)

        Hello World

## Retrieve My Resource [GET]

+ Response 200

    [My Resource][]
```

#### 1.3.3.6. Schema section

```
+ Schema
```

为http消息体指定一个有效的模式。

例子：

下面的例子使用Body提供一个`application/json`的例子，使用Schema提供一个JSON模式描述所有可能的有效的payload.

```
## Retrieve a Message [GET]

+ Response 200 (application/json)
    + Body

            {"message": "Hello world!"}

    + Schema

            {
                "$schema": "http://json-schema.org/draft-04/schema#",
                "type": "object",
                "properties": {
                    "message": {
                        "type": "string"
                    }
                }
            }
```

#### 1.3.3.7. Action section

使用http请求方法定义：

```
## <HTTP request method>
```

或者http方法前添加action标识符：

```
## <identifier> [<HTTP request method>]
```

或者标识符+请求方法+uri模板：

```
## <identifier> [<HTTP request method> <URI template>]
```

至少定义一个HTTP事务，可能有多个事务例子。

可以包含一个嵌套的URI参数部分，作用域仅在相应的action内。

可以包含一个嵌套的参数部分，用于定义输入（请求）参数。除非另有说明，否则这些参数应该被每一个Request部分继承。

应该包含至少一个嵌套的Response部分，以及可能包含额外的嵌套Request和Response部分。

嵌套的Request和Response部分可能被排序成组，每个组代表一个事务示例。第一个事务示例组从第一个嵌套的Request或Response部分开始，后续组从响应部分后的第一个嵌套Request部分开始。

一个事务示例中的多个Request和Response嵌套部分应具有不同的标识符。

例子：

```m
# Blog Posts [/posts{?limit}]
 ...

## Retrieve Blog Posts [GET]
Retrieves the list of **ACME Blog** posts.

+ Parameters
    + limit (optional, number) ... Maximum number of posts to retrieve

+ Response 200

        ...

## Create a Post [POST]

+ Attributes

        ...

+ Request

        ...

+ Response 201

        ...

## Delete a Post [DELETE /posts/{id}]

+ Parameters
    + id (string) ... Id of the post

+ Response 204
```

例子：多个事务的示例：

```m
# Resource [/resource]
## Create Resource [POST]

+ request A

        ...

+ response 200

        ...

+ request B

        ...

+ response 200

        ...

+ response 500

        ...

+ request C

        ...

+ request D

        ...

+ response 200

        ...
```

上例展示了3个事务：
1. request A, response 200
2. request B, response 200 and 500
3. request C and D, response 200


#### 1.3.3.8. Request section

关键字 `Request`，后跟可选的标识符：

```m
+ Request <identifier> (<Media Type>)
```

例子：

```m
+ Request Create Blog Post (application/json)

        { "message" : "Hello World." }
```


#### 1.3.3.9. Response section

关键字 `Response`，后面应该使用HTTP状态码作为标识符：

```
+ Response <HTTP status code> (<Media Type>)
```

例子：

```m
+ Response 201 (application/json)

            { "message" : "created" }
```


#### 1.3.3.10. URI parameters section

关键字 `Parameters`，markdown列表的形式：

```m
+ Parameters
```

在父section的范围内来讨论 URI parameters.

这个section只能作为嵌套的列表项。不能包含其他的元素。一个列表项只能描述一个 URI parameter。

```m
+ <parameter name>: `<example value>` (<type> | enum[<type>], required | optional) - <description>

    <additional description>

    + Default: `<default value>`

    + Members
        + `<enumeration value 1>`
        + `<enumeration value 2>`
        ...
        + `<enumeration value N>`
```

* `<parameter name>` 是写在 Resource section的URI中的参数名（比如 "id"）.
* `<description>` 可选，markdown格式.
* `<additional description>` 额外的可选的markdown格式的描述。
* `<default value>` 可选，参数的默认值。
* `<example value>` 可选，参数值的示例，比如 1234.
* `<type>` 可选，期望的类型（比如："number", "string", "boolean"），"string" 是默认值。
* Members 是可选的，列举出可能的值。如果有Members，那么 `<type>` 需要用 `enum[]` 包含。例如，如果提供了枚举值，参数类型是 number, 那么应该使用 `enum[number]` 而不是 `number`。
* `<enumeration value n>` 列举枚举类型的一个元素。
* required 必选，默认值。
* optional 可选。

NOTE: 这个section应该只列举出在URI模板中指定的参数，不必把所有的参数都列出。

示例：

```
# GET /posts/{id}
```

```
+ Parameters
    + id - Id of a post.
```

```
+ Parameters
    + id (number) - Id of a post.
```

```
+ Parameters
    + id: `1001` (number, required) - Id of a post.
```

```
+ Parameters
    + id: `1001` (number, optional) - Id of a post.
        + Default: `20`
```

```
+ Parameters
    + id (enum[string])

        Id of a Post

        + Members
            + `A`
            + `B`
            + `C`
```


#### 1.3.3.11. Attributes Section

```
+ Attributes <Type Definition>
```

`<Type Definition>` 是要描述的数据结构的类型定义。如果没有指定，将假定一个对象基本类型。细节请看[MSON Type Definition](https://github.com/apiaryio/mson/blob/master/MSON%20Specification.md#35-type-definition)

示例：

```
+ Attributes (object)
```

使用markdown语法描述一个数据结构。基于父section，数据结构被描述为以下之一：
* 资源数据属性（Resource section）
* Action请求属性（Action section）
* 负载消息体属性（Payload section）

定义在这个部分的数据结构可能会引用定义在其他数据结构部分的结构，以及由命名资源属性描述定义的任何数据结构(见下文).

**资源属性**

如果数据结构定义为命名资源，那么其他数据结构可以使用资源名引用它。

示例：

```
# Blog Post [/posts/{id}]
Resource representing **ACME Blog** posts.

+ Attributes
    + id (number)
    + message (string) - The blog post article
    + author: john@appleseed.com (string) - Author of the blog post
```

NOTE: 这个数据结构可以被这样引用：

```
+ Attributes (Blog Post)
```

**Action属性**

如果定义了，Action section下的所有Request都继承这些属性，除非另有指定。

示例：

```
## Create a Post [POST]

+ Attributes
    + message (string) - The blog post article
    + author: john@appleseed.com (string) - Author of the blog post

+ Request (application/json)

+ Request (application/yaml)

+ Response 201
```

**Payload属性**

不必把所有属性都描述。如果描述了一个属性，那么这个属性应该出现在Body示例中（如果有Body部分的话）。

如果定义了，Body部分可能被省略了，那么示例应该从属性描述中产生。

如果没有Schema部分的话，消息体属性的描述可能被用于描述消息体的校验。如果有Schema的话，属性描述应该和Schema一致。

```
## Retrieve a Post [GET]

+ Response 200 (application/json)

    + Attributes (object)
        + message (string) - Message to the world

    + Body

            { "message" : "Hello World." }
```


#### 1.3.3.12. Headers section

```
+ Headers
```

指定父payload部分的HTTP消息头。语法期望是下面这样的预格式化代码块：

```
<HTTP header name>: <value>
```

示例：

```
+ Headers

        Accept-Charset: utf-8
        Connection: keep-alive
        Content-Type: multipart/form-data, boundary=AaB03x
```


#### 1.3.3.13. Body section

指定payload部分的消息体。

示例：

```
+ Body

        {
            "message": "Hello"
        }
```


#### 1.3.3.14. Data Structures section

定义在这个section的数据结构可以被用于任何属性section。类似地，定义在属性section的任何数据结构也可以被用于数据结构定义中。

示例：

```
# Data Structures

## Message (object)

+ text (string) - text of the message
+ author (Author) - author of the message

## Author (object)

+ name: John
+ email: john@appleseed.com
```

示例：在资源中复用数据结构：

```
# User [/user]

+ Attributes (Author)

# Data Structures

## Author (object)

+ name: John
+ email: john@appleseed.com
```

示例：在数据结构中复用定义的资源：

```
# User [/user]

+ Attributes
    + name: John
    + email: john@appleseed.com

# Data Structures

## Author (User)
```


#### 1.3.3.15. Relation section

格式：

```
+ Relation: <link relation identifier>
```

这个section为给定的Action指定连接关系类型，如RFC 5988指出的那样。

NOTE: 在文档中，每个资源的连接关系标识符应该是唯一的。

示例：

```
# Task [/tasks/{id}]

+ Parameters
    + id

## Retrieve Task [GET]

+ Relation: task
+ Response 200

        { ... }

## Delete Task [DELETE]

+ Relation: delete
+ Response 204
```







## 1.4. 附录

### 1.4.1. URI 模板

一个简单的URI路径段像这样：

```
/path/to/resources/42
```

**模板变量**

变量名是大小写敏感的。允许的变量名只能包含以下几种：
* ASCII字符（`a-z`, `A-Z`）
* 数字（`0-9`）
* `_`
* [百分比编码字符](https://en.wikipedia.org/wiki/Percent-encoding)
* `.`

多个变量之间用逗号分隔，逗号前后不能有空格。变量必须包含在`{}`中，前后也不能有空格。

**操作符**

括号中第一个变量前可能有一个操作符。允许的操作符如下：
* `#` 片段标识符操作符
* `+` 保留值操作符
* `?` 表单式查询操作符
* `&` 表单式查询连接操作符

示例：

```
{var}
{var1,var2,var3}
{#var}
{+var}
{?var}
{?var1,var2}
{?%24var}
{&var}
```

下面的字符在变量名中属于保留字符：

`:` / `/` / `?` / `#` / `[` / `]` / `@` / `!` / `$` / `&` / `'` / `(` / `)` / `*` / `+` / `,` / `;` / `=`

**路径分段变量**

没有任何操作符：

```
/path/to/resources/{var}
```

如果 `var := 42`，那么应该是 `/path/to/resources/42`.

**片段标识符变量**

```
/path/to/resources/42{#var}
```

如果 `var := my_id`,那么`/path/to/resources/42#my_id`.

**包含保留字符的变量**

```
/path/{+var}/42
```

如果 `var := to/resources`, 那么 `/path/to/resources/42`.


**表单式查询操作符**

```
/path/to/resources/{varone}{?vartwo}
```

如果 `varone := 42` 并且 `vartwo = hello`, 那么 `/path/to/resources/42?vartwo=hello`.

```
/path/to/resources/{varone}?path=test{&vartwo,varthree}
```

如果 `varone := 42, vartwo = hello, varthree = 1024`, 那么 `/path/to/resources/42?path=test&vartwo=hello&varthree=1024`.





## 1.5. 解析工具 aglio

```bash
sudo npm install aglio
```

可能会出现 `read ECONNRESET` 错误，根据 [stack overflow](http://stackoverflow.com/questions/18419144/npm-not-working-read-econnreset) 上的解答，执行下面的命令：

```bash
npm config set registry http://registry.npmjs.org/
```

npm 会请求 http 而不是 https 了。