# 系统设计

## 一、技术选型

本系统的项目后端基于 Spring Boot、Spring Cloud Alibaba 框架，数据库使用 MySQL,缓存中间件使用 Redis,数据操作框架使用 Mybatis-Plus，前端基于 Vue2 进行开发，使用 Ajax 与后端进行交互，做到真正的前后端分离开发，程序评测运行使用开源的Go-Judge 安全沙盒保证程序的高性能判题和系统的 安全防护。使用Docker 和 Docker-Compose 进行服务编排与部署，做到真正的一键化极易部署。

**前端技术：**

!!! tip

    - 技术以Vue2为主，element-ui为主要的UI框架
    - 支持手机端，响应式布局
    - 以CodeMirror作为在线代码编辑器
    - 以Mavon-Editor作为富文本编辑器
    - 以Vxe-Table作为表格组件



**后端技术：**

!!! tip

    *hoj-backend（数据服务）*

    - 主体Web框架技术以SpringBoot为主
    - 以Nacos为分布式注册中心及分布式配置中心，支持配置文件动态刷新。
    - 以Mybatis-Plus为数据库中间件，负责数据实体类与数据库数据的转化与获取。
    - 以Shiro为安全框架，支持用户角色权限管理，支持token刷新
    - 以Redis作为数据缓存和使用list作为等待评测队列



**评测端技术：**

!!! tip

    *hoj-judgeserver（评测服务）*
    - 主体Web框架技术以SpringBoot为主
    - 以Mybatis-Plus为数据库中间件，负责数据实体类与数据库数据的转化与获取。
    - 将服务注册到Nacos，以供hoj-backend进行调度，同时获取到Nacos上的配置

- 本地评测：
  - 主流程：调用SandBox（Go-Judge）进行评测，将对应结果写回数据库
  - 功能：支持普通评测、特殊评测、交互评测、在线调试
- 远程评测：
  - 提交流程：通过爬虫技术将代码提交到HDU、Codeforces、POJ等平台，获取提交id
  - 获取结果：根据提交id多次轮询获取最终的评测结果，将对应结果写回数据库



## 二、整体架构

!!! info

    本系统是基于前后端分离、分布式架构搭建的，用户通过浏览器进行访问，请求发送到Nginx被代理转发到Vue项目生成的静态文件，Vue项目的后端数据请求则再次通过Nginx进行转发到后端业务服务，由后端业务服务查询MySQL数据、Redis缓存进行业务处理生成所需JSON数据返回给前端，由Vue进行渲染展示给用户。



此外，如果用户提交了评测请求，则业务服务将通过Nacos查询健康可用的评测服务实例，将请求发送给指定的评测服务，评测服务则将进行评测操作，调用安全沙盒，编译运行用户代码，跑各个评测点数据得出最终结果，写回数据库。整个系统各个服务都是在Ubuntu系统下基于Docker来搭建的，保证各个服务之间互不干扰。

![系统架构图](../../resource/img/sys_architecture.png)

## 三、功能介绍

| 模块      |                   功能介绍                   |
| ------- | :--------------------------------------: |
| 首页      |       展示网站轮播图、公告栏、近期比赛栏、最近7天做题排名栏        |
| 题目      | 提供展示题目列表页，可以根据题库、难度、标签等进行筛选查询，同时展示各个题目的做题情况；提供展示题目详情页，可以看到题目内容，在线编写代码，查看当前用户对于该题的历史提交记录 |
| 训练      | 提供展示训练列表页，可以根据权限、分类进行筛选查询，进入指定训练页，可以查看到训练的介绍、训练的题目单以及训练记录单 |
| 比赛      | 提供展示类型为ACM和OI的比赛列表页，可以根据类型与比赛状态进行筛选查询，同时展示比赛的标题、时间、时长、类型、参赛人数等信息；进入指定比赛，可以看到比赛题目、比赛提交列表、排行榜、比赛公告、评论，以及比赛管理员可以选择重新评测某个比赛题目、提供现场打印代码的功能 |
| 评测      |  用户可以看到所有的提交记录列表，可以通过状态、题目ID、提交者进行筛选过滤   |
| 排名      | 分为ACM排行榜和OI排行榜，分别根据对应ACM题目和OI题目提交情况对用户进行排名展示 |
| 讨论      | 提供讨论列表页，可以看到各个分类的讨论帖子，同时也可以发布用户自己的讨论；点击指定的讨论帖子，即可看到帖子详情，在下方可以对讨论进行评论以及回复他人的评论 |
| 团队      | 团队可以看做是一个独立的小HOJ，里面包含了现有HOJ的题目、训练、比赛、评测、讨论、公告、排名等功能，其中各个团队的数据与HOJ主站的数据已做了完全隔离，各个团队可以自定义属于自己的题目、比赛、训练等，其中目前支持团队中的题目申请公开到主题库 |
| 关于      |          介绍本网站的开发者和各个编程语言的编译介绍           |
| 个人信息与设置 | 用户拥有自己的个人主页，可以展示用户的学校、名称、个人简介、做题数和得分等信息；同时登陆状态可以在右上角进入个人设置，可以修改密码和邮箱，以及各种个人信息 |
| 登录与注册   |  用户可以通过点击导航栏右上角选择登录、注册、重置密码，即可完成对用户的鉴权   |
| 站内消息    | 提供消息自动查询，每两分钟更新最新消息来提示用户，进入消息中心，可以看到评论我的、回复我的、收到的赞、系统通知、我的消息等模块的消息 |