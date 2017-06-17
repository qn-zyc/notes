# install

```bash
npm install -g dredd
```

# Quickstart

1. 编写 api 文档，比如 api blueprint 或者 swagger.
2. 实现 api. 比如 api blueprint 可以使用 drakov 工具。
3. 使用 dredd 测试：

    ```
    $ dredd api-description.apib http://127.0.0.1:3000  # API Blueprint
    $ dredd api-description.yml http://127.0.0.1:3000  # Swagger
    ```

# How It Works

执行步骤：

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


