# 前言

1. 图片的来源分为 [Pixiv](https://www.pixiv.net/) 和 [X](https://x.com/)，版权也归作品的作者所有，API 仅储存了作品的基本信息，不提供图片的代理或储存服务
2. 为了保证图片质量，库中的图片大部分由本人添加，绝非爬虫自动爬取的产物，所有大部分图都是根据我 xp 来的，如果不合您的胃口那真是抱歉了
3. 这些色图是广义上的色图，所以你也许会觉得图不够色，这是正常的，你就当成是抽卡好了【**可能下一发出出金了呢**】
4. 不接受任何索要数据库的请求
5. 使用本 API 是你个人的行为，产生的任何问题都与我无关
6. 注意节制，身体是自己的

## 相关拓展服务

### [来点图](https://setu.mossia.top)

可以给 API 内的图进行评分的一个小东西，现阶段只接入了 [LinuxDo](https://linux.do)登录，需要有其账号才能使用

### 随机一图

可以随机获取 API 内的一张图
* [Pixiv版本](https://rand.mossia.top)
* [Pixiv版本(R-18)](https://rand-r18.mossia.top)
* [X版本](https://rand-x.mossia.top)

# API

## Pixiv 图源

### GET 请求

```http
https://api.mossia.top/duckMo
```

### POST 请求

**传参格式： `Content-Type: application/json`**

```http
https://api.mossia.top/duckMo
```

#### 请求

|    参数名    |  数据类型  |     默认值     |                        说明                        |
| :----------: | :--------: | :------------: | :------------------------------------------------: |
|    `num`     |   `int`    |      `1`       |        一次返回的结果数量，范围为 `1` 到 20        |
|    `pid`     |  `int[]`   |                |                返回指定 `pid` 图片                 |
|    `uid`     |  `int[]`   |                |             返回指定 `uid` 作者的作品              |
|   `author`   |  `string`  |                |              根据作者名称模糊搜索作品              |
|   `proxy`    |  `string`  |  `i.pixiv.re`  |     设置图片地址所使用的在线反代服务，详见下文     |
|   `aiType`   |   `int`    |                |       ai 类型 0-未知[旧画作] \| 1-否 \| 2-是       |
|  `r18Type`   |   `int`    |                |                r18 类型 0-不是 1-是                |
| `dateAfter`  |   `int`    |                | 返回在这个时间及以后上传的作品；时间戳，单位为毫秒 |
| `dateBefore` |   `int`    |                | 返回在这个时间及以前上传的作品；时间戳，单位为毫秒 |
|  `sizeList`  | `string[]` | `["original"]` |          返回指定图片规格的地址，详见下文          |

#### sizeList

以下为五种规格的示例

| 规格       | 地址                                                                                                                                                                                                 |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `original` | [https://i.pixiv.re/img-original/img/2022/07/07/00/00/07/99549235_p0.png](https://i.pixiv.re/img-original/img/2022/07/07/00/00/07/99549235_p0.png)                                                   |
| `regular`  | [https://i.pixiv.re/img-master/img/2022/07/07/00/00/07/99549235_p0_master1200.jpg](https://i.pixiv.re/img-master/img/2022/07/07/00/00/07/99549235_p0_master1200.jpg)                                 |
| `small`    | [https://i.pixiv.re/c/540x540_70/img-master/img/2022/07/07/00/00/07/99549235_p0_master1200.jpg](https://i.pixiv.re/c/540x540_70/img-master/img/2022/07/07/00/00/07/99549235_p0_master1200.jpg)       |
| `thumb`    | [https://i.pixiv.re/c/250x250_80_a2/img-master/img/2022/07/07/00/00/07/99549235_p0_square1200.jpg](https://i.pixiv.re/c/250x250_80_a2/img-master/img/2022/07/07/00/00/07/99549235_p0_square1200.jpg) |
| `mini`     | [https://i.pixiv.re/c/48x48/img-master/img/2022/07/07/00/00/07/99549235_p0_square1200.jpg](https://i.pixiv.re/c/48x48/img-master/img/2022/07/07/00/00/07/99549235_p0_square1200.jpg)                 |

##### proxy

由于 P 站资源域名`i.pximg.net`具有防盗链措施，不含`www.pixiv.net` referrer 的请求均会 403，所以如果需要直接在网页上展示或在客户端上直接下载必须依靠反代服务

### 响应

| 字段名    | 数据类型  | 说明         |
| --------- | --------- | ------------ |
| `errCode` | `string`  | 状态码       |
| `success` | `boolean` | 请求是否成功 |
| `message` | `boolean` | 响应提示     |
| `data`    | `vo[]`    | 色图数组     |

#### vo

| 字段名        | 数据类型  | 说明                                                             |
| ------------- | --------- | ---------------------------------------------------------------- |
| `pid`         | `int`     | 作品 pid                                                         |
| `uid`         | `int`     | 作者 uid                                                         |
| `title`       | `string`  | 作品标题                                                         |
| `author`      | `string`  | 作者名                                                           |
| `width`       | `int`     | 原图宽度 px                                                      |
| `height`      | `int`     | 原图高度 px                                                      |
| `ext`         | `string`  | 图片扩展名                                                       |
| `aiType`      | `number`  | 是否是 AI 作品，`0` 未知（旧画作或字段未更新），`1` 不是，`2` 是 |
| `pcreateDate` | `int`     | 作品创建日期；时间戳，单位为毫秒                                 |
| `puploadDate` | `int`     | 作品上传日期；时间戳，单位为毫秒                                 |
| `tagsList`    | `tagVO[]` | 作品标签                                                         |
| `urlsList`    | `urlVO[]` | 作品 url                                                         |

#### tagVO

| 字段名    | 数据类型 | 说明       |
| --------- | -------- | ---------- |
| `tagName` | `string` | 标签名     |
| `tagEn`   | `string` | 标签名翻译 |

#### urlVO

| 字段名    | 数据类型 | 说明     |
| --------- | -------- | -------- |
| `urlSize` | `string` | 图片规格 |
| `url`     | `string` | 图片链接 |

## X 图源

### GET 请求

```http
https://api.mossia.top/duckMo/x
```

### POST 请求

**传参格式： `Content-Type: application/json`**

```http
https://api.mossia.top/duckMo/x
```

#### 请求

|    参数名    | 数据类型 | 默认值 |                        说明                        |
| :----------: | :------: | :----: | :------------------------------------------------: |
|    `num`     |  `int`   |  `1`   |        一次返回的结果数量，范围为 `1` 到 20        |
| `dateAfter`  |  `int`   |        | 返回在这个时间及以后上传的作品；时间戳，单位为毫秒 |
| `dateBefore` |  `int`   |        | 返回在这个时间及以前上传的作品；时间戳，单位为毫秒 |

### 响应

| 字段名    | 数据类型  | 说明         |
| --------- | --------- | ------------ |
| `errCode` | `string`  | 状态码       |
| `success` | `boolean` | 请求是否成功 |
| `message` | `boolean` | 响应提示     |
| `data`    | `vo[]`    | 色图数组     |

#### vo

| 字段名        | 数据类型 | 说明                             |
| ------------- | -------- | -------------------------------- |
| `url`         | `int`    | 帖子地址                         |
| `pictureUrl`  | `int`    | 图片地址                         |
| `xCreateDate` | `int`    | 作品创建日期；时间戳，单位为毫秒 |
