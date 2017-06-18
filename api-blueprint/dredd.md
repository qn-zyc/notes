# install

```bash
npm install -g dredd
```

# Quickstart

1. 编写 api 文档，比如 api blueprint 或者 swagger.
2. 实现 api. 比如 api blueprint 可以使用 drakov 工具。
3. 使用 dredd 测试：

    ```bash
    $ dredd api-description.apib http://127.0.0.1:3000  # API Blueprint
    $ dredd api-description.yml http://127.0.0.1:3000  # Swagger
    ```

# How It Works

## 执行时的生命周期

1. Load and parse API description documents
    * Report parse errors and warnings
2. Pre-run API description check
    * Missing example values for URI template parameters
    * Required parameters present in URI
    * Report non-parseable JSON bodies
    * Report invalid URI parameters
    * Report invalid URI templates
3. Compile HTTP transactions from API description documents
    * Inherit headers
    * Inherit parameters
    * Expand URI templates with parameters
4. Load hooks
5. Test run
    * Report test run start
    * Run beforeAll hooks
    * For each compiled transaction:
        * Report test start
        * Run beforeEach hook
        * Run before hook
        * Send HTTP request
        * Receive HTTP response
        * Run beforeEachValidation hook
        * Run beforeValidation hook
        * Perform validation
        * Run after hook
        * Run afterEach hook
        * Report test end with result for in-progress reporting
    * Run afterAll hooks
6. Report test run end with result statistics


## 自动化期望

利用 [Gavel.js](https://github.com/apiaryio/gavel.js) 基于API描述的例子自动生成HTTP响应的期望。想了解更多查看 [Gavel](https://www.relishapp.com/apiary/gavel/docs) 的规则。

### 响应头的期望

* 所有声明在API描述中的 headers 必须出现在响应中。
* header 的名称大小写敏感。
* Only values of headers significant for content negotiation are validated.
* 其他所有 headers 的值都可以不同。

### 响应体的期望

如果响应体是 JSON，Dredd 只校验结构。其他格式的响应体都当做纯文本来校验。

Dredd 使用从 API 描述中推测的 JSON Schema 来校验结构。有效的 JSON Schema 从下列几处获取（优先级从高到低）：

#### API Blueprint

1. `+ Schema` 部分 - 提供的特制的 JSON Schema 将被采用(Draft [v4](https://tools.ietf.org/html/draft-zyp-json-schema-04) 和 [v3](https://tools.ietf.org/html/draft-zyp-json-schema-03))。
2. `+ Attributes` 部分，使用 MSON 描述的数据结构 - API Blueprint 解析器自动从 MSON 生成 JSON Schema.
3. `+ Body` 部分，包含 JSON payload 的样本 - Gavel.js, 在 Dredd 中负责校验，自动推断一些基本的期望。

### Gavel的期望

* 样本中出现的所有 JSON 的关键字，不管什么级别，都必须出现在响应的 JSON 中。
* 响应中 JSON 的值必须是 JSON 原始(primitive)值。
* JSON 值可以不同。
* 数组可以有多余的元素，数组元素的类型或结构不会校验。
* 纯文本必须完全匹配。

### 特定的期望

可以在 hook 中添加自己特定的期望。示例可以查看如果使用 [Chai.js断言](http://dredd.readthedocs.io/en/latest/hooks/#using-chai-assertions)


## 让你的 API 描述可测试
