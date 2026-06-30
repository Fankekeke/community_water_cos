![](https://img.shields.io/badge/License-MIT-blue.svg)

![](https://img.shields.io/badge/Version-1.0.0-brightgreen.svg)

![](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)

# 药店药品调配系统 (Pharmacy Drug Dispensing System)

一套专为现代化连锁及单体药店设计的全链路数字化管理系统。本系统集成了**多角色权限管理、进销存一体化、电子处方流转、财务薪资结算**等核心业务模块，旨在帮助药店实现精细化运营，提升处方调配效率与库存周转率。


#### 安装环境
**️ 注意**：请确保您的开发环境已安装以下依赖。

JAVA 环境 

Node.js环境 [https://nodejs.org/en/] 选择14.17

Yarn 打开cmd， 输入npm install -g yarn !!!必须安装完毕nodejs

Mysql 数据库 [https://blog.csdn.net/qq_40303031/article/details/88935262] 一定要把账户和密码记住

redis

Idea 编译器 [https://blog.csdn.net/weixin_44505194/article/details/104452880]

WebStorm OR VScode 编译器 [https://www.jianshu.com/p/d63b5bae9dff]

#### 采用技术及功能

后端：SpringBoot、MybatisPlus、MySQL、Redis、
前端：Vue、Apex、Antd、Axios

平台前端：vue(框架) + vuex(全局缓存) + rue-router(路由) + axios(请求插件) + apex(图表)  + antd-ui(ui组件)

平台后台：springboot(框架) + redis(缓存中间件) + shiro(权限中间件) + mybatisplus(orm) + restful风格接口 + mysql(数据库)

开发环境：windows10 or windows7 ， vscode or webstorm ， idea + lambok

---

## 核心特性

- **多门店/多角色协同**：完美支持 管理员、药店、员工、用户 四大角色，数据权限严格隔离。
- **全生命周期药品管理**：涵盖采购、入库、库存调配、预警及出库全流程。
- **电子处方流转**：支持在线处方审核、调配记录与追溯，保障用药安全。
- **多维度数据看板**：提供销售排行、库存统计、员工绩效等可视化报表。
- **业财一体化**：打通在线支付、缴费记录与员工薪资发放链路。

---

## ️ 功能模块概览

### ️ 管理员

全局统筹与底层数据维护，掌控系统运行命脉。

- **数据看板**：首页订单看板、销售统计、销售排行
- **组织与人事**：员工管理、职位变动、薪资发放、用户/管理员管理
- **药品与库存**：药品管理、电子处方、库存统计、库房预警、药店库存、库存调整/调配
- **订单与财务**：订单管理/详情、订单评价、配送信息、缴费记录
- **供应链**：供应商管理、药品采购、采购物流、药店管理、公告管理

### 药店

门店日常运营与人员调度中心。

- **门店看板**：商家订单看板
- **人员管理**：员工管理、岗位调整、员工薪资、员工留言
- **业务处理**：订单管理/详情、电子处方
- **库存流转**：药店库存、库存调整、出入库详情、药店信息

### 员工

一线销售与调配人员的高效工作台。

- **个人看板**：员工销售看板
- **业务执行**：药品记录、电子处方处理、订单管理
- **个人事务**：我的信息、我的薪资、我的留言

### 用户

便捷的 C 端购药与健康管理入口。

- **购药体验**：药品购买、在线支付、药品处方查看
- **订单服务**：我的订单、订单评价、缴费记录
- **个人设置**：个人信息管理

---

## 项目结构

```text
pharmacy-dispensing-system/
├── backend/            # 后端 API 服务
│   ├── src/
│   │   ├── controllers/  # 控制器
│   │   ├── models/       # 数据模型
│   │   ├── routes/       # 路由
│   │   └── services/     # 业务逻辑
│   └── package.json
├── frontend/           # 前端 Web 应用
│   ├── src/
│   │   ├── views/      # 页面视图
│   │   ├── components/ # 公共组件
│   │   └── store/      # 状态管理
│   └── package.json
├── docs/               # 项目文档与 API 接口说明
├── LICENSE
└── README.md
```
---


#### 前台启动方式
安装所需文件 yarn install 
运行 yarn run dev

#### 默认后台账户密码
[管理员]
admin
1234qwer

[商家管理员]
shangjia
1234qwer

[用户]
fank
1234qwer

#### 项目截图

|  |  |
|---------------------|---------------------|
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/0a434def-3738-466e-8f82-b2092cde0c39.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/199caebe-c7ec-431b-bdac-26f643b5fad5.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/e96504ec-7a4c-404f-af46-b3d0a957b3fa.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/93df1307-f85d-4bff-b8ee-86210dd766b4.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/ca0f79dd-3feb-42df-b404-5d22ef1db7a2.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/79c34183-cd3a-4945-a932-f4826892fcc5.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/c5f55665-e349-44b4-af82-52173a2aea44.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/36e7671c-8089-45db-ac8e-191513a6265d.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/c2c0e9f8-13f9-41bb-a83a-d498d422f226.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/28fc6704-c8d6-4121-af4f-c57bd2b0525a.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/ba905605-3956-43c9-bd1c-418012a1c533.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/19d5c93b-7190-423d-83d0-b5b3a22d1906.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/b68046a2-0d95-4dc7-a11b-d495ff39af90.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/010bd7ec-86a1-4a76-ab70-7bc379d84493.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/a8bba953-3994-41b1-b762-c3db8ad61387.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/8c37d64d-e729-4d09-9548-4bfab331a894.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/49260112-d9f5-4f61-9dd1-d24f198915ad.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/6ab683ed-2544-471a-a526-ca2c96e892bc.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/6186a2b8-d76e-405b-b3d5-bb44f7d1bd6e.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/4d3cc96b-84e6-450d-98c0-8d24ab6a65a0.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/519a5e02-03ef-48cd-89a6-858ef60a6b8c.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/3fbfba72-2298-4d3c-9904-cd150be5c425.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/471be86f-9fd7-4d59-a49a-44ed40e03c94.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/2bfd9cc2-ea57-4032-b45f-a9880f2b2ce2.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/309cd591-11b5-48e9-922b-4e730548e943.png) | ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/0df655e6-a5f2-4110-8751-4b90a48d2f09.png) |
| ![](https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/282db690-6b37-4415-8b4e-97956828e1a5.png) |  |

#### 演示视频

暂无

#### 获取方式

Email: fan1ke2ke@gmail.com

WeChat: `Storm_Berserker`

`附带部署与讲解服务，因为要恰饭资源非免费，伸手党勿扰，谢谢理解😭`

> 1.项目纯原创，不做二手贩子 2.一次购买终身有效 3.项目讲解持续到答辩结束 4.非常负责的答辩指导 5.**黑奴价格**

> 项目部署调试不好包退！功能逻辑没讲明白包退！

#### 其它资源

[2025年-答辩顺利通过-客户评价🍜](https://berserker287.github.io/2025/06/18/2025%E5%B9%B4%E7%AD%94%E8%BE%A9%E9%A1%BA%E5%88%A9%E9%80%9A%E8%BF%87/)

[2024年-答辩顺利通过-客户评价👻](https://berserker287.github.io/2024/06/06/2024%E5%B9%B4%E7%AD%94%E8%BE%A9%E9%A1%BA%E5%88%A9%E9%80%9A%E8%BF%87/)

[2023年-答辩顺利通过-客户评价🐢](https://berserker287.github.io/2023/06/14/2023%E5%B9%B4%E7%AD%94%E8%BE%A9%E9%A1%BA%E5%88%A9%E9%80%9A%E8%BF%87/)

[2022年-答辩通过率100%-客户评价🐣](https://berserker287.github.io/2022/05/25/%E9%A1%B9%E7%9B%AE%E4%BA%A4%E6%98%93%E8%AE%B0%E5%BD%95/)

[毕业答辩导师提问的高频问题](https://berserker287.github.io/2023/06/13/%E6%AF%95%E4%B8%9A%E7%AD%94%E8%BE%A9%E5%AF%BC%E5%B8%88%E6%8F%90%E9%97%AE%E7%9A%84%E9%AB%98%E9%A2%91%E9%97%AE%E9%A2%98/)

[50个高频答辩问题-技术篇](https://berserker287.github.io/2023/06/13/50%E4%B8%AA%E9%AB%98%E9%A2%91%E7%AD%94%E8%BE%A9%E9%97%AE%E9%A2%98-%E6%8A%80%E6%9C%AF%E7%AF%87/)

[计算机毕设答辩时都会问到哪些问题？](https://www.zhihu.com/question/31020988)

[计算机专业毕业答辩小tips](https://zhuanlan.zhihu.com/p/145911029)

#### 接JAVAWEB毕设，纯原创，价格公道，诚信第一

`网站建设、小程序、H5、APP、各种系统 选题+开题报告+任务书+程序定制+安装调试+项目讲解+论文+答辩PPT`

More info: [悲伤的橘子树](https://berserker287.github.io/)

<p><img align="center" src="https://fank-bucket-oss.oss-cn-beijing.aliyuncs.com/img/%E5%90%88%E4%BD%9C%E7%89%A9%E6%96%99%E6%A0%B7%E5%BC%8F%20(3).png" alt="fankekeke" /></p>
