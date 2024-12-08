# 前端 BFF 多包架构应用使用文档

## 写在前面

- 本文后面的例子代码可能在项目master分支还没有，如果没有，请拉分支2478-demo查看，后面会合到master
- 本文档就是项目中的md，文档原文[链接](https://git.dustess.com/dustess-f2e/cfx-proxy-monorepo/-/tree/2478-demo)

## 目录

- [前端 BFF 多包架构应用使用文档](#前端-bff-多包架构应用使用文档)
  - [写在前面](#写在前面)
  - [目录](#目录)
  - [简介](#简介)
  - [项目地址](#项目地址)
  - [准备](#准备)
    - [技术栈涉及](#技术栈涉及)
    - [运行环境](#运行环境)
    - [推荐安装 VSCode 插件](#推荐安装-vscode-插件)
  - [项目整体结构](#项目整体结构)
  - [代码风格、约定和注意事项](#代码风格约定和注意事项)
    - [代码风格](#代码风格)
      - [文件/文件夹命名](#文件文件夹命名)
      - [controller 命名](#controller-命名)
      - [业务实体模块组织形式](#业务实体模块组织形式)
    - [约定](#约定)
      - [依赖关系](#依赖关系)
    - [其他](#其他)
      - [注意事项](#注意事项)
  - [本地开发](#本地开发)
    - [安装依赖](#安装依赖)
    - [启动](#启动)
  - [示例](#示例)
    - [如何写一个接口](#如何写一个接口)
      - [1 确认接口属于的业务实体](#1-确认接口属于的业务实体)
      - [2 编写 bff 接口代码](#2-编写-bff-接口代码)
        - [2.1 编写 controller](#21-编写-controller)
        - [2.2 编写 service](#22-编写-service)
        - [2.3 组合 module](#23-组合-module)
        - [2.4 app 引入模块](#24-app-引入模块)
      - [3 写接口文档](#3-写接口文档)
      - [4 写单元测试](#4-写单元测试)
      - [5 提测/上线](#5-提测上线)
  - [数据 mock](#数据-mock)
  - [其他](#其他-1)
    - [接口域名新增规则](#接口域名新增规则)
      - [webgateway接口](#webgateway接口)
      - [非webgateway接口](#非webgateway接口)
        - [1. 需要先在项目env文件中新增环境变量](#1-需要先在项目env文件中新增环境变量)
        - [2. 生产环境读取的环境变量不在这里，但是变量名称都相同，值如何查看，请见步骤](#2-生产环境读取的环境变量不在这里但是变量名称都相同值如何查看请见步骤)
    - [常用命令](#常用命令)
      - [生成 webapi service](#生成-webapi-service)
      - [新增应用](#新增应用)
      - [创建 Library](#创建-library)
      - [Unit Test](#unit-test)
      - [Lint](#lint)
      - [Build](#build)
    - [Nest 相关工具](#nest-相关工具)
    - [查看代码影响范围](#查看代码影响范围)
  - [写在最后](#写在最后)

## 简介

前端 BFF 应用,采用多包架构模式, 架构设计方案见方案文档: [【企微文档】前端多包架构 BFF 方案](https://doc.weixin.qq.com/doc/w3_AVkAEQalABE88Bp1YAfTMiGBCJbMY?scode=ACUArAcBAAkYfYZtPcAVkAEQalABE)

## 项目地址

[https://git.dustess.com/dustess-f2e/cfx-proxy-monorepo](https://git.dustess.com/dustess-f2e/cfx-proxy-monorepo)，下文的 demo 也在项目中

## 准备

### 技术栈涉及

- 装饰器 [TS](https://www.typescriptlang.org/docs/handbook/decorators.html)
- Nx [NX 文档](https://nx.dev/getting-started/intro)
- nestjs [英文](https://docs.nestjs.com/) [中文](https://docs.nestjs.cn/)

### 运行环境

- Nodejs : > 20.0.0
- npm 源: <https://npm.dustess.com>
- Nx : > 16.10.0 [NX 文档](https://nx.dev/getting-started/intro)
- pnpm : > 8.5.0

注:禁止使用 npm/yarn

### 推荐安装 VSCode 插件

使用 vscode 插件 NXConsole 可视化来执行.
![Nx Console](https://raw.githubusercontent.com/nrwl/nx-console/master/static/nx-console-light.png)
插件下载地址: [VSCode 插件](https://marketplace.visualstudio.com/items?itemName=nrwl.angular-console)

## 项目整体结构
>
> 主要列出需要大家需要关注的项目目录，其他的项目配置相关的文件、目录省略

```bash
# 当前应用
root
└── apps
    ├── active-proxy # 独立运营活动,2C的内容
    ├── desktop-proxy # PC端应用,包含浏览器端和企微内置浏览器
    ├── mini-proxy # 小程序应用,和uni-app同构的h5
    ├── mobile-proxy # 移动端应用
    └── sidebar-proxy # 侧边栏应用
 └── libs
     #一般情况下, 所有的业务逻辑和api的实现都在Library实现, 为了区别业务逻辑和公用逻辑分logic和shared库
     ├── logic # 业务的实现,代码组织方式按照业务实体实现即可
         └──src
            ├── customer # eg.联系人对应的相关实现,实现controller和service,对外暴露module
            └── xxx 其他
     └── shared # 共享方法，公共包,被apps加载
         └──src
            ├── api-service # 依赖的后端请求
                └── customer # eg.客户相关后端服务
            ├── webapi-service # 调用 pnpm webapi 自动生成的 webapi serivce
            ├── config # 配置
            ├── controller # 公共控制器，例如全局继承的 /healthy
            ├── decorator # 公共装饰器，例如用于包装swagger文档的ApiWrappedResponse
            ├── enums # 暂时无用，公共枚举
            ├── http # bff端，公共请求库，用于请求后端服务
            ├── interceptor # 公共拦截器
            ├── log # 日志系统
            ├── middleware # 公共中间件
            ├── service # 公共的service
            ├── types # 类型定义
            └── utils # 公共方法
```

## 代码风格、约定和注意事项

### 代码风格

#### 文件/文件夹命名

以-分割，单词为维度
eg.

```bash
├── a-folder
├── b-folder
├── c-folder
```

#### controller 命名

小驼峰

```typescript
@Controller('demo')
@ApiTags('demo')
export class DemoController {
  constructor(private demoService: DemoService) {}

  //小驼峰
  @Post('/myDemoApi')
  postCustomerDetailDemo(@Body() searchDto: SearchDto): Promise<any> {
    return this.demoService.getCombineData(searchDto);
  }
}
```

#### 业务实体模块组织形式

同一实体模块应当放到同一文件夹下放，文件夹内部包含一个整体的模块
包括 module、controller、service、spec,文件名统一为 index,通过后缀来区分功能
接口类型，同一用文件夹 dto 包裹，定义接口的请求、返回类型
若后续模块变的更复杂，比如 customer 下面又能包括很多模块，则继续在当前模块文件夹下新增 module 文件夹，来管理下级 module
eg.

```bash
logic
└── src
    └── customer
        ├── module #若有
            ├── sub-module-a
            └── sub-module-b
        ├── dto
           ├── response.dto.ts
           ├── request.dto.ts
           └── index.ts
        ├── index.module.ts
        ├── index.controller.ts
        ├── index.controller.spec.ts
        ├── index.service.ts
        ├── index.service.spec.ts
        └── index.ts
```

### 约定

#### 依赖关系

- apps 可以依赖 libs 下 logic 和 shared
- logic 可以依赖 logic 下模块和 shared
- shared 只能依赖 shared 下模块

### 其他

#### 注意事项

- 所有的 library 均不需要实现 publish 逻辑,均在项目内直接依赖引用
- Library 无法独立运行,需要在 apps 中引用

## 本地开发

### 安装依赖

```bash
# 安装全部依赖
pnpm install
# 安装指定依赖
pnpm add <pkg-name>
```

### 启动

启动本地开发环境 ,本地默认启动 3000 端口, 可以使用环境变量注入其他端口
> 命令行启动

```bash
# Use pnpm
pnpm nx serve <app-name> #默认启动dev环境，意思就是请求的接口为dev的后端接口
#or
pnpm nx serve <app-name> -c <env> # env可选为dev/test/qa/tencent

# Use nx , Need install Nx to global
nx serve <app-name> #默认启动dev环境，意思就是请求的接口为dev的后端接口
#or
nx serve <app-name> -c <env> # env可选为dev/test/qa/tencent
```

> 使用 vscode 插件启动

![示例](https://cf-cdn.dustess.com/image-host/21/a70dd670-f887-46d2-b4bc-f567015252ba.png)

## 示例

### 如何写一个接口

示例：如何写一个 bff 接口，从开发到上线, **要运行体验 demo，请启动 desktop-proxy**

> 前置流程，在 dx 上面的操作，略。请知晓，bff 的 dx 操作流程与其他纯前端项目完全相同

#### 1 确认接口属于的业务实体

eg.侧边栏有一个查询联系人详情的接口（内部可能聚合了三个接口）。则该接口应该属于侧边栏，对应 apps/sidebar-proxy。
如果该接口在服务商后台又被用到了，则该接口也属于 apps/desktop-proxy。
所以这个服务需要在两个应用出口中被暴露。

#### 2 编写 bff 接口代码

以下实现一个组合两个后端接口的 demo

##### 2.1 编写 controller

nestjs controller 文档 [跳转](https://docs.nestjs.com/controllers)
> controller 负责处理传入的请求和向客户端返回响应，不应该包含业务逻辑，业务处理逻辑应该放在 service 中处理
路由命名约定，小驼峰

先实现一个纯返回字符串的 post 接口

```typescript
// 完整代码见当前项目 libs/logic/demo/module/demo1/index.controller.ts

@Controller('/demo1')
@ApiTags('demo')
export class Demo1Controller {

  @Post('/getCombineData')
  getCombineData() {
    return '返回数据';
  }
}
```

##### 2.2 编写 service

接口的详细业务逻辑都放在 service 中，多个 service 可以进行组合

继续以上的 demo，在上方 demo 的基础上，我们去聚合后端接口，并且返回接口数据

```typescript
// 完整代码见当前项目 libs/logic/demo/module/demo2/index.controller.ts
// 添加一个新的路由
@Controller('/demo2')
export class Demo2Controller {

  constructor(private demo2Service: Demo2Service) {}

  @Post('/getCombineData')
  getCombineData(@Body() searchDto: SearchDto): Promise<any> {
    return this.demo2Service.getCombineData(searchDto);
  }
}

```

```typescript
// 完整代码见当前项目 libs/logic/demo/module/demo2/index.controller.ts
// 在以上代码基础上，封装业务逻辑，实现具体业务，这里demo展示聚合两个后端接口然后返回
@Injectable()
export class DemoService {
  // 这里依赖另一个service，service提供的就是具体的http调用
  constructor(private demoApiService : DemoApiService){}

  async getCombineData(searchDto: SearchDto) {
    // 下面是接口请求
    const [res1, res2] = await Promise.all([
      this.demoApiService.getGroupTag<SearchDto['api1']>(searchDto.api1),
      this.demoApiService.getMenu<SearchDto['api2']>(searchDto.api2)
    ]);

    return {
      data: {
        data1: res1.data,
        data2: res2.data
      }
    };
  }
}
```

##### 2.3 组合 module

通过 module，可以方便的导出整个模块

```typescript
// 完整代码见当前项目 libs/logic/demo/index.module.ts
import { Module } from '@nestjs/common';
import { Demo1Module } from './demo1';
import { Demo2Module } from './demo2';

@Module({
  imports: [Demo1Module, Demo2Module],
  controllers: [],
  providers: []
})
export class DemoModule {}

```

##### 2.4 app 引入模块

```typescript
// apps/desktop-proxy/app/src/app/app.module.ts
@Module({
  imports: [
    DemoModule
  ]
})
```

#### 3 写接口文档

nestjs 项目中集成了@nestjs/swagger，详情[请见](https://docs.nestjs.com/openapi/introduction)
> ApiWrappedResponse为项目中封装的装饰器，会自动将msg，code等参数包装在swagger文档中

eg.单纯为了掩饰如何写文档，与刚才实现的 demo 接口没关系

```typescript
// 完整代码见当前项目 libs/logic/demo/module/module/demo3/index.controller.ts
@Controller('/demo3')
@ApiTags('demo')
export class Demo3Controller {

  @Post('/showSwaggerDoc')
  @HttpCode(200)
  @ApiOperation({
    summary: '演示swagger文档编写方法',
    description: '演示swagger文档编写方法'
  })
  @ApiWrappedResponse(ResponseDto)
  async getCustomerDetailDemo(@Body() body: PaginationDto) {
    return '数据返回';
  }
}
```

#### 4 写单元测试

单元测试详细功能，[参考文档](https://docs.nestjs.com/fundamentals/testing)
eg.单纯为了演示单测，与刚才实现的 demo 接口没关系

```typescript
// 完整代码见当前项目 libs/logic/demo/module/module/demo4
```

运行demo单元测试请运行

```bash
pnpm nx test  --testFile demo4
# or
nx test logic --testFile demo4
```

运行全部整个项目请使用命令
[Unit Test](#unit-test)

#### 5 提测/上线

没有yapi项目权限请找**李霖**添加

略，dx 正常流程

## 数据 mock

详见
简单概括如下：
使用 yapi 的 mock 能力

怎么推送 yapi：

```bash
# 以desktop为例
# 启动 app
nx serve desktop-proxy
# 运行脚本 根据交互选择你的 app，完成后会自动推送
npm run publish:yapi
# 默认本地端口为 3000，请根据本地启动的端口更改端口参数
npm run publish:yapi -p=3001
```

示例：
以demo3为例，运行完以上命令之后，远程swagger文档内容已经生成
![示例](https://cf-cdn.dustess.com/image-host/21/918f656d-0133-47a5-863f-a6fcb3a135c3.png)
使用时候只需要请求mock地址就行

## 其他

### 接口域名新增规则

由于本地开发和发到各个环境，dev、qa、生产的时候，需要bff层在调用后端服务时候使用内网域名，所以需要对内网域名进行环境变量配置，但本地开发时候不支持内网域名，所以会有两套环境变量，项目中的env文件定义（本地开发），远程的dconf配置（生产），本地的env文件需要开发者在用到的时候进行添加(非webgateway接口)，并且告知对应环境的内网域名，让#李泽宇进行dconf配置

#### webgateway接口

不用管，统一使用已经封装好的webgateway请求方法就行

#### 非webgateway接口

需要新增域名地址，并且在调用接口的时候需要传递新增的环境变量名，以使用到mall-config-ms服务为例，示例如下：

##### 1. 需要先在项目env文件中新增环境变量

```bash
# .env.dev
MALL_CONFIG_MS_API=https://mk-dev.dustess.com
# 其他环境.env.test/.env.qa/.env.tencent为其对应的外网地址
```

##### 2. 生产环境读取的环境变量不在这里，但是变量名称都相同，值如何查看，请见步骤

- 确认调用的接口属于哪个后端服务
- 登录吉利，查看内网域名，不同环境的内网域名不一样，需要都进行提供
![示例](https://cf-cdn.dustess.com/image-host/26/a72e20dd-d1fc-4bcc-b91e-f205c8fc3dfb.png)
- 让#泽宇帮忙配一配
- 代码调用时候传入对应域名变量

```typescript
this.httpService.http({
  method: 'GET',
  baseURL: this.configService.get<string>('MALL_CONFIG_MS_API'),
  url: '/mall-config-ms/v1/config-account/code?code=mall_decorate_setting'
});
```

### 常用命令

#### 生成 webapi service

> 代码生成能力正在试用中使用前先联系一下： @李泽宇

在项目根目录下配置 `webApiConfig.yaml`

```yaml
apiApps:
    - appName: crm-customer   # 应用名称
      services:
        - serviceName: FieldsConfigService   # 服务名
        # vernsion: latest   # 当前pb版本号，需询问后端，不填则默认拉取最新版本
    - appName: xxx
      services:
        - serviceName: xxx2
        # vernsion: latest
        - serviceName: xxx2
        # vernsion: latest
        - serviceName: xxx3
        # vernsion: latest
...
```

执行代码生成命令

```shell
pnpm webapi
```

#### 新增应用

一般情况下不会增加. 如有需要增加新的应用,请发送邮件到<liupan1@dustess.com>申请

``` bash
# Use pnpm
pnpm dlx nx g @nx/nest:app app-name --directory=apps/app-name --e2eTestRunner=none --projectNameAndRootFormat=as-provided

# Use Nx , Need install Nx to global
nx g @nx/nest:app app-name --directory=apps/app-name --e2eTestRunner=none --projectNameAndRootFormat=as-provided
```

说明:
![命名说明](https://cf-cdn.dustess.com/image-host/26/23a9a0bd-d7bf-4a25-bd5c-577f5a06fdc0.png)

#### 创建 Library

备注: 建议使用 VSCode 对应的 Nx 插件可视化生成

```bash
# logic library
## Use pnpm
pnpm dlx nx g @nx/nest:lib lib-name --directory=libs/logic/lib-name --controller=true --importPath=@cfx/proxy/logic/lib-name --projectNameAndRootFormat=as-provided --service=true --simpleName=true
##  Use nx , Need install Nx to global
nx g @nx/nest:lib lib-name --directory=libs/logic/lib-name --controller=true --importPath=@cfx/proxy/logic/lib-name --projectNameAndRootFormat=as-provided --service=true --simpleName=true

# shared nest library
## Use pnpm
pnpm dlx nx g @nx/nest:lib lib-name --directory=libs/shared/lib-name --importPath=@cfx/proxy/shared/lib-name --projectNameAndRootFormat=as-provided --service=true --simpleName=true
##  Use nx , Need install Nx to global
nx g @nx/nest:lib lib-name --directory=libs/shared/lib-name --controller=true --importPath=@cfx/proxy/shared/lib-name --projectNameAndRootFormat=as-provided --service=true --simpleName=true

# shared js library
## Use pnpm
pnpm dlx nx g @nx/js:lib lib-name --directory=libs/shared/lib-name --importPath=@cfx/proxy/shared/lib-name --unitTestRunner=jest --pascalCaseFiles=true --projectNameAndRootFormat=as-provided --simpleName=true
##  Use nx , Need install Nx to global
nx g @nx/js:lib lib-name --directory=libs/shared/lib-name --importPath=@cfx/proxy/shared/lib-name --unitTestRunner=jest --pascalCaseFiles=true --projectNameAndRootFormat=as-provided --simpleName=true
```

#### Unit Test

```bash
# Use pnpm
pnpm nx test <app-name>
# Use nx , Need install Nx to global
nx test <app-name>
```

#### Lint

```bash
# Use pnpm
pnpm nx lint <app-name>
# Use nx , Need install Nx to global
nx lint <app-name>
```

#### Build

```bash
# Use pnpm
pnpm nx build <app-name>
# Use nx , Need install Nx to global
nx build <app-name>
```

### Nest 相关工具

Nest 相关代码生成工具,参考插件: [@nx/nest](https://nx.dev/nx-api/nest)

默认情况下: 请直接增加 nest lib, 如要在某一 lib 下增加 controller,module,service 请手动添加. 原因:

- 采用了 app 应用和 logic 分离的方式实现代码的组织, 自动化增加的内容不会生成到 libs 下
- logic 下每一个实体实现的 module 可以引用多个 controller 和 service 的注入, 如果归属该实体,请直接手动添加,如果没有,请增加 libs
- 针对组合实体的内容,请使用共享原则,请参考示例说明

### 查看代码影响范围

NX 提供了对应的依赖影响范围关系, 在代码发生变更后,不确定那些项目受到影响,请运行影响检查命令

```bash
# Use pnpm
pnpm nx graph < --files=xxxx >
# or
pnpm nx graph --affected < --files=xxxx >
# Use nx , Need install Nx to global
nx graph < --files=xxxx>
# or
nx graph --affected < --files=xxxx >
```

该命令会在本地启动一个网页服务,在浏览器中打开工作区的项目图，并突出显示受更改代码发生变更的文件影响的项目
eg.

```bash
nx graph --affected --files=libs/logic/customer/index.controller.ts
```

得到结果
![结果](https://cf-cdn.dustess.com/image-host/21/09ff81b3-b1b9-4bdd-ae08-ff542edf1589.png)

参考: [NX graph](https://nx.dev/nx-api/nx/documents/dep-graph)

## 写在最后

好了，看完了本文，以及包含的需要了解的东西，你应该已经会了
