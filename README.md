## HttpRunner demo 使用说明

### demo使用

#### 安装HttpRunner

> pip install httprunner

#### clone demo到本地

> git clone https://github.com/BiaoLiu/httprunner-demo.git

#### 进入到demo目录

> cd xxx/httprunner-demo

#### 运行测试用例

> hrun testsuites/manager/ad/ad_testsuite.yml



### HttpRunner说明

#### 创建api目录下的测试用例

api目录存放api接口，request参数具体对应python的requests库的参数
validate为接口返回结果的校验器

例如，创建api/manager/ad_list.json，参数定义完成后，可使用：

> hrun api/manager/ad/ad_list.json 

进行单个接口请求测试



#### 创建testcases目录下的测试用例

如果多个接口间有依赖，如 详情接口与更新接口 依赖于新增接口（详情接口与更新接口都需要使用到 新增接口创建完数据生成的主键id），
使用extract参数，就可从新增接口的 HTTP 请求的响应结果中提取参数，并保存到参数变量中（例如welcomeId），后续测试用例可通过$welcomeId的形式进行引用

例如，创建testcases/manager/ad_testcase.yml，参数定义完成后，可使用：

> hrun testcases/manager/ad/ad_testcase.yml

进行有序接口请求测试



#### 创建testsuites目录下的测试用例集

在自动化测试中，经常会遇到如下场景：

- 测试搜索功能，只有一个搜索输入框，但有 10 种不同类型的搜索关键字；
- 测试账号登录功能，需要输入用户名和密码，按照等价类划分后有 20 种组合情况。

如果需要使用动态参数运行测试用例，则需要使用testsuite的parameters参数。

##### 动态参数的几种形式

###### 独立参数 & 直接指定参数列表

对于参数列表比较小的情况，最简单的方式是直接在 YAML/JSON 中指定参数列表内容。

例如，对于独立参数 user_id，参数列表为 [1001, 1002, 1003, 1004]，那么就可以按照如下方式进行配置：

```
config:
    name: testcase description

testcases:
    create user:
        testcase: demo-quickstart-6.yml
        parameters:
            user_id: [1001, 1002, 1003, 1004]
```

进行该配置后，测试用例在运行时就会对 user_id 实现数据驱动，即分别使用 [1001, 1002, 1003, 1004] 四个值运行测试用例。

###### 关联参数 & 直接指定参数列表

对于具有关联性的多个参数，例如 username 和 password，那么就可以按照如下方式进行配置：

```
config:
    name: "demo"

testcases:
    testcase1_name:
        testcase: /path/to/testcase1
        parameters:
            username-password:
                - ["user1", "111111"]
                - ["user2", "222222"]
                - ["user3", "333333"]
```

进行该配置后，测试用例在运行时就会对 username 和 password 实现数据驱动，即分别使用 `{"username": "user1", "password": "111111"}`、`{"username": "user2", "password": "222222"}`、`{"username": "user3", "password": "333333"}` 运行 3 次测试，并且保证参数值总是成对使用。

其他的参数形式，具体参考官方文档：

[httprunner 参数化数据驱动](https://cn.httprunner.org/prepare/parameters/)



例如，创建testsuites/manager/ad/ad_testsuite.yml，参数定义完成后，可使用：

> hrun testsuites/manager/ad/ad_testsuite.yml

进行接口的多种参数请求测试



#### 测试报告

使用 HttpRunner 执行完自动化测试后，会在当前路径的 reports 目录下生成一份 HTML 格式的测试报告。



#### 参考文档

[HttpRunner官方文档](