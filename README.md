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



## HttpRunner 说明

### 创建API请求描述

API请求描述文档存放在api目录下，示例如下：

`api/manager/ad/create_ad.json`

```
{
  "base_url": "${ENV(BASE_URL)}",
  "name": "create ad welcome",
  "variables": {
    "userId": 1,
    "resType": "PIC"
  },
  "request": {
    "url": "/ad/welcomes",
    "method": "POST",
    "headers": {
      "Content-Type": "application/json"
    },
    "json": {
      "userId": "$userId",
      "title": "测试的欢迎页10",
      "resType": "$resType",
      "resUrl": "https://www.sina.com",
      "platform": "web",
      "stopSeconds": 60,
      "link": {
        "linkId": "",
        "linkModule": "textbook",
        "linkPageType": "list"
      },
      "linkUrl": "https://www.baidu.com",
      "isActive": false,
      "welcomeNo": 99
    }
  },
  "validate": [
    {
      "eq": [
        "status_code",
        200
      ]
    },
    {
      "eq": [
        "content.ecode",
        0
      ]
    }
  ]
}
```

参数说明：   

- request   
  request的参数指定接口请求url、method以及请求的数据等，具体对应Python的Requests库的参数   
- variables   
  如果在接口描述中存在变量引用的情况，在variables中对参数进行定义
- validate   
  validate为接口返回结果的校验器

例如，创建api/manager/ad_list.json，参数定义完成后，可使用：

> hrun api/manager/ad/ad_list.json 

进行单个接口请求测试



### 创建测试用例(testcase)

测试用例存放在testcases目录下

概念：

- 测试用例（testcase）应该是完整且独立的，每条测试用例应该是都可以独立运行的
- 测试用例是测试步骤（teststep）的 有序 集合，每一个测试步骤对应一个 API 的请求描述

在一个测试用例中，如果多个接口间有依赖，如 详情接口与更新接口 依赖于新增接口（详情接口与更新接口都需要使用到 新增接口创建完数据生成的主键id），使用teststeps定义接口请求顺序，同时使用extract参数，就可从新增接口的 HTTP 请求的响应结果中提取参数，并保存到参数变量中（例如welcomeId），后续测试步骤可通过$welcomeId的形式进行引用

示例如下：

`testcases/manager/ad/ad_testcase.yml`

```
config:
    name: "ad testcase"
    variables:
        username: ${ENV(USERNAME)}
        password: ${ENV(PASSWORD)}
    base_url: ${ENV(BASE_URL)}

teststeps:
-
    name: ad list
    api: api/manager/ad/ad_list.json
    variables:
        pageNo: 1
        pageSize: 10
    validate:

- eq: ["status_code", 200]
- eq: ["content.ecode",0]
  -
      name: create ad welcome
      api: api/manager/ad/create_ad.json #引用API请求
      extract:  # 保存新增数据的wecomeId
- welcomeId: content.data.welcomeId
  date:
- eq: ["status_code", 200]
- eq: ["content.ecode",0]
  -
      name: ad welcome detail
      api: api/manager/ad/ad_detail.json  #请具体查看，其中url引用了welcomeId
      validate:
- eq: ["status_code", 200]
- eq: ["content.ecode",0]
  -
      name: update ad welcome
      api: api/manager/ad/update_ad.json
      validate:
- eq: ["status_code", 200]
- eq: ["content.ecode",0]
  -
      name: delete ad welcome
      api: api/manager/ad/delete_ad.json
      validate:
- eq: ["status_code", 200]
- eq: ["content.ecode",0]
```

例如，创建testcases/manager/ad_testcase.yml，参数定义完成后，可使用：

> hrun testcases/manager/ad/ad_testcase.yml

运行测试用例



### 创建测试用例集(testsuite)

测试用例集存放在testcases目录下

概念：

- 测试用例集（testsuite）是测试用例（testcase）的 无序 集合，集合中的测试用例应该都是相互独立，不存在先后依赖关系的；如果确实存在先后依赖关系，那就需要在测试用例中完成依赖的处理

使用testsuite不是必须的，使用testsuite主要有2个好处：

1. 当测试用例数量比较多以后，为了方便管理和实现批量运行，通常需要使用测试用例集来对测试用例进行组织。
2. 参数化请求

#### 参数化请求

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

[HttpRunner 参数化数据驱动](https://cn.httprunner.org/prepare/parameters/)



示例如下：

`testsuites/manager/ad/ad_testsuite.yml`

```
config:
    name: "ad testsuite"
    base_url: ${ENV(BASE_URL)}

testcases:
-
    name: ad welcome testcases
    testcase: testcases/manager/ad/ad_testcase.yml
    parameters:
        pageNo-pageSize:
            - [1,10]
            - [2,1]
            - [3,1]
        userId: [101, 102, 103]
        resType: ["PIC", "VID","TEST"]
```



例如，创建testsuites/manager/ad/ad_testsuite.yml，参数定义完成后，可使用：

> hrun testsuites/manager/ad/ad_testsuite.yml

对测试用例进行批量运行，以及进行接口的多种参数请求测试



### 测试报告

使用 HttpRunner 执行完自动化测试后，会在当前路径的 reports 目录下生成一份 HTML 格式的测试报告。

![](http://file.wpcenter.cn/b67d58e6-414d-4e0f-9234-cfb6aa677e1c.png)

如果用例测试失败，点击Detail中log，查看日志信息
![](http://file.wpcenter.cn/Snipaste_2019-07-26_15-22-31.png)



### 参考文档

[HttpRunner官方文档](https://cn.httprunner.org)