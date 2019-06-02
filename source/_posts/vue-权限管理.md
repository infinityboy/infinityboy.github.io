---
title: vue 权限管理
date: 2019-06-01 21:21:55
tags: vue
categories: 
- web前端
---

>本篇博客仅以个人学习记录使用
>参照开源项目地址：https://github.com/zhangdaiscott/jeecg-boot
>对比参照：https://pro.loacg.com/docs/getting-started

### 产生背景

首先，权限管理并不是随着vue或者react或者angular等技术出现才产生的。早期的web是十分简单的，随着互联网的发展，人们越来越多的使用互联网，使用web作为系统管理，web需求变得越来越复杂，当一个系统需要区分不同的账号，不同的角色，分别有不同的操作使用权限和查看权限的时候，就需要做权限管理，简单的举例说就是区分识别普通用户和vip用户(人民币玩家)。

### 权限种类

权限控制大体上分为单账号->单角色/单账号->多角色。单账号单角色更简单一些只需要使用账号就可以区分开，单账号多角色还需要针对该账号存在的不同角色再去做区分匹配。同时对于权限控制在页面上也会有更加细粒度的控制，除了浏览权限之外，还有按钮控制权限，还有数据权限。当然权限最大的还是超级管理员，这个人可以肆意妄为。。。

### 区别

随着单页应用和前后端分离，权限管理的实现方式出现了一些变化，但是思路上大致都是相同的。以往的区分方式是使用session id去匹配出不同的账号和角色，返回相应的展示，这个时候是由服务端去做的，也是根据url地址区分不同的页面，这是多页时代，随着react,angular和vue的兴起，单页应用越来越流行起来，这个时候就不能通过url地址去区分了，因为这个时候的单页应用是前后端分离的，后端不存在路由跳转了，后端只有接口了，所有的路由跳转都是通过前端来实现的，在前端中，前端拥有了自己的路由，也就是前端路由，前端路由去控制跳转，而这个跳转实际上不过是不同模块和组件的切换而已，这个切换的状态会展现在url地址栏当中，这个时候的url也就不同于过去的url了，这个时候的url不负责请求资源，只是作为跳转路径存在，而请求资源都是通过Ajax或者Fetch去实现。因此采用单页应用，由前端渲染的开发方式也就可以不需要使用session了，这时我们便通过Token令牌的方式去更好的实现权限区分(当然使用Token替代session主要原因并不是因为单页应用前端渲染，而是因为不再需要服务端维护session)，以往时跳转是在服务端实现的，但是现在是在客户端实现的。前端还需要存储服务端返回的Token,每次执行请求时需要带上这个Token去做验证，同时后端还应设置Token的过期时间，Token过期，返回过期标识，前端跳出进行重新登录。

### 登录/注销，Token验证及存储实现思路

一般登陆的时候，像后端发送请求，后端会返回token令牌，然后前端将token存存储到localstorage当中(这样方便在登陆之前下获取一次本地token当本地token有效时，直接根据token像后端请求权限路由)。并且在登陆之前还会先有一个全局验证的步骤，而且一般token采用jwt标准生成。

### 全局权限验证

全局验证思路主要是通过router.beforeEach这个方法，在路由初始化第一次访问路由的时候就开始验证了，具体逻辑看以下代码:

```
import Vue from 'vue'
import router from './router'
import store from './store'
import NProgress from 'nprogress' // progress bar
import 'nprogress/nprogress.css' // progress bar style
// import notification from 'ant-design-vue/es/notification'
import { ACCESS_TOKEN } from '@/store/mutation-types'
import { generateIndexRouter } from "@/utils/util"

NProgress.configure({ showSpinner: false }) // NProgress Configuration

const whiteList = ['/user/login', '/user/register', '/user/register-result'] // no redirect whitelist

router.beforeEach((to, from, next) => {
  NProgress.start() // start progress bar

  if (Vue.ls.get(ACCESS_TOKEN)) {
    /* has token */
    if (to.path === '/user/login') {
      next({ path: '/dashboard/workplace' })
      NProgress.done()
    } else {
      if (store.getters.permissionList.length === 0) {
        store.dispatch('GetPermissionList').then(res => {
              const menuData = res.result.menu;
              console.log(res.message)
              if (menuData === null || menuData === "" || menuData === undefined) {
                return;
              }
              let constRoutes = [];
              constRoutes = generateIndexRouter(menuData);
              // 添加主界面路由
              store.dispatch('UpdateAppRouter',  { constRoutes }).then(() => {
                // 根据roles权限生成可访问的路由表
                // 动态添加可访问路由表
                router.addRoutes(store.getters.addRouters)
                const redirect = decodeURIComponent(from.query.redirect || to.path)
                if (to.path === redirect) {
                  // hack方法 确保addRoutes已完成 ,set the replace: true so the navigation will not leave a history record
                  next({ ...to, replace: true })
                } else {
                  // 跳转到目的路由
                  next({ path: redirect })
                }
              })
            })
          .catch(() => {
           /* notification.error({
              message: '系统提示',
              description: '请求用户信息失败，请重试！'
            })*/
            // 未请求到权限强制退出重新登录
            store.dispatch('Logout').then(() => {
              next({ path: '/user/login', query: { redirect: to.fullPath } })
            })
          })
      } else {
        next()
      }
    }
  } else {
    if (whiteList.indexOf(to.path) !== -1) {
      // 在免登录白名单，直接进入
      next()
    } else {
      next({ path: '/user/login', query: { redirect: to.fullPath } })
      NProgress.done() // if current page is login will not trigger afterEach hook, so manually handle it
    }
  }
})

router.afterEach(() => {
  NProgress.done() // finish progress bar
})

```

### 路由表实现思路

既然路由跳转是在客户端实现的，那么路由表自然也就是在客户端咯？？答案是，这也不一定。前端路由是一定要有的，只不过这时在实现上有区别，第一种:前端会保存一份完整的路由表，通过请求后端，由后端返回标志，前端去筛选这份完整的前端路由，找出想要的部分来当做该权限下的路由表使用。第二种：前端只保留基本路由，其他的路由，全部由后端动态的生成，然后传给前端。第一种方式在简单的场景下可以采用，这样不必占用过多的服务器资源，前端路由完全放在前端去维护，更加方便前端去更改，当然在面对复杂的权限管理时问题也会变的十分显著。第二种方式也就是我们所说的动态路由了，这种实现方式完全由后端动态生成返回给前端，前端根据这份路由表去实现权限控制。采用这种方式也更加灵活，无论是简单的权限管理还是复杂的权限管理都可以直接采用这种方式去实现，前端也不需要去维护路由表，这时需要做的就是和后端约定好路由规则，根据需求动态生成路由表，传给前端就可以了。这份路由表当中包括了前端的路由跳转以及权限控制信息。一份完整的路由表可能是这样：
```
{
    "success": true,
    "message": "查询成功",
    "code": 200,
    "result": {
        "allAuth": [
            {
                "action": "user:form:phone",
                "describe": "手机号禁用",
                "type": "2",
                "status": "1"
            },
            {
                "action": "user:add",
                "describe": "添加用户按钮",
                "type": "1",
                "status": "1"
            }
        ],
        "auth": [
            {
                "action": "user:form:phone",
                "describe": "手机号禁用",
                "type": "2"
            },
            {
                "action": "user:add",
                "describe": "添加用户按钮",
                "type": "1"
            }
        ],
        "menu": [
            {
                "redirect": null,
                "path": "/dashboard/analysis",
                "component": "dashboard/Analysis",
                "route": "1",
                "meta": {
                    "keepAlive": "true",
                    "icon": "home",
                    "title": "首页"
                },
                "name": "dashboard-analysis",
                "id": "9502685863ab87f0ad1134142788a385"
            },
            {
                "redirect": null,
                "path": "/report",
                "component": "layouts/RouteView",
                "route": "1",
                "children": [
                    {
                        "path": "/report/ArchivesStatisticst",
                        "component": "jeecg/report/ArchivesStatisticst",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "布局统计报表"
                        },
                        "name": "report-ArchivesStatisticst",
                        "id": "2aeddae571695cd6380f6d6d334d6e7d"
                    },
                    {
                        "path": "/report/ViserChartDemo",
                        "component": "jeecg/report/ViserChartDemo",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "ViserChartDemo"
                        },
                        "name": "report-ViserChartDemo",
                        "id": "020b06793e4de2eee0007f603000c769"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "bar-chart",
                    "title": "统计报表"
                },
                "name": "report",
                "id": "f0675b52d89100ee88472b6800754a08"
            },
            {
                "redirect": null,
                "path": "/isystem",
                "component": "layouts/RouteView",
                "route": "1",
                "children": [
                    {
                        "path": "/isystem/user",
                        "component": "system/UserList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "用户管理"
                        },
                        "name": "isystem-user",
                        "id": "3f915b2769fc80648e92d04e84ca059d"
                    },
                    {
                        "path": "/isystem/depart",
                        "component": "system/DepartList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "部门管理"
                        },
                        "name": "isystem-depart",
                        "id": "45c966826eeff4c99b8f8ebfe74511fc"
                    },
                    {
                        "path": "/isps/userAnnouncement",
                        "component": "system/UserAnnouncementList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "我的消息"
                        },
                        "name": "isps-userAnnouncement",
                        "id": "53a9230444d33de28aa11cc108fb1dba"
                    },
                    {
                        "path": "/system/departUserList",
                        "component": "system/DepartUserList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "我的部门"
                        },
                        "name": "system-departUserList",
                        "id": "5c2f42277948043026b7a14692456828"
                    },
                    {
                        "path": "/isystem/role",
                        "component": "system/RoleList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "角色管理"
                        },
                        "name": "isystem-role",
                        "id": "e8af452d8948ea49d37c934f5100ae6a"
                    },
                    {
                        "path": "/isystem/permission",
                        "component": "system/PermissionList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "菜单管理"
                        },
                        "name": "isystem-permission",
                        "id": "54dd5457a3190740005c1bfec55b1c34"
                    },
                    {
                        "path": "/isystem/dict",
                        "component": "system/DictList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "数据字典"
                        },
                        "name": "isystem-dict",
                        "id": "f1cb187abf927c88b89470d08615f5ac"
                    },
                    {
                        "path": "/isystem/annountCement",
                        "component": "system/SysAnnouncementList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "系统通告"
                        },
                        "name": "isystem-annountCement",
                        "id": "e08cb190ef230d5d4f03824198773950"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "setting",
                    "title": "系统管理"
                },
                "name": "isystem",
                "id": "d7d6e2e4e2934f2c9385a623fd98c6f3"
            },
            {
                "redirect": null,
                "path": "/dashboard3",
                "component": "layouts/RouteView",
                "route": "1",
                "children": [
                    {
                        "path": "/monitor",
                        "component": "layouts/RouteView",
                        "route": "1",
                        "children": [
                            {
                                "path": "/monitor/redis/info",
                                "component": "modules/monitor/RedisInfo",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "Redis监控"
                                },
                                "name": "monitor-redis-info",
                                "id": "8d1ebd663688965f1fd86a2f0ead3416"
                            },
                            {
                                "path": "/monitor/TomcatInfo",
                                "component": "modules/monitor/TomcatInfo",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "Tomcat信息"
                                },
                                "name": "monitor-TomcatInfo",
                                "id": "024f1fd1283dc632458976463d8984e1"
                            },
                            {
                                "path": "/monitor/SystemInfo",
                                "component": "modules/monitor/SystemInfo",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "服务器信息"
                                },
                                "name": "monitor-SystemInfo",
                                "id": "8b3bff2eee6f1939147f5c68292a1642"
                            },
                            {
                                "path": "/monitor/JvmInfo",
                                "component": "modules/monitor/JvmInfo",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "JVM信息"
                                },
                                "name": "monitor-JvmInfo",
                                "id": "d07a2c87a451434c99ab06296727ec4f"
                            },
                            {
                                "path": "/monitor/HttpTrace",
                                "component": "modules/monitor/HttpTrace",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "请求追踪"
                                },
                                "name": "monitor-HttpTrace",
                                "id": "fc810a2267dd183e4ef7c71cc60f4670"
                            },
                            {
                                "path": "/monitor/Disk",
                                "component": "modules/monitor/DiskMonitoring",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "磁盘监控"
                                },
                                "name": "monitor-Disk",
                                "id": "97c8629abc7848eccdb6d77c24bb3ebb"
                            }
                        ],
                        "meta": {
                            "keepAlive": "true",
                            "title": "性能监控"
                        },
                        "name": "monitor",
                        "id": "700b7f95165c46cc7a78bf227aa8fed3"
                    },
                    {
                        "path": "/isystem/log",
                        "component": "system/LogList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "日志管理"
                        },
                        "name": "isystem-log",
                        "id": "58857ff846e61794c69208e9d3a85466"
                    },
                    {
                        "path": "/sys/dataLog-list",
                        "component": "system/DataLogList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "数据日志"
                        },
                        "name": "sys-dataLog-list",
                        "id": "841057b8a1bef8f6b4b20f9a618a7fa6"
                    },
                    {
                        "path": "5f22d2592b01c9e964efe70040162b83",
                        "component": "layouts/IframePageView",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "SQL监控",
                            "url": "{{ window._CONFIG['domianURL'] }}/druid/"
                        },
                        "name": "{{ window._CONFIG['domianURL'] }}-druid-",
                        "id": "aedbf679b5773c1f25e9f7b10111da73"
                    },
                    {
                        "path": "5388c8fe3cd402f07a4bc39dddc61b30",
                        "component": "layouts/IframePageView",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "在线文档",
                            "url": "{{ window._CONFIG['domianURL'] }}/swagger-ui.html#/"
                        },
                        "name": "{{ window._CONFIG['domianURL'] }}-swagger-ui.html#-",
                        "id": "2dbbafa22cda07fa5d169d741b81fe12"
                    },
                    {
                        "path": "/isystem/QuartzJobList",
                        "component": "system/QuartzJobList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "定时任务"
                        },
                        "name": "isystem-QuartzJobList",
                        "id": "b1cb0a3fedf7ed0e4653cb5a229837ee"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "dashboard",
                    "title": "系统监控"
                },
                "name": "dashboard3",
                "id": "08e6b9dc3c04489c8e1ff2ce6f105aa4"
            },
            {
                "redirect": null,
                "path": "/message",
                "component": "layouts/RouteView",
                "route": "1",
                "children": [
                    {
                        "path": "/modules/message/sysMessageList",
                        "component": "modules/message/SysMessageList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "消息管理"
                        },
                        "name": "modules-message-sysMessageList",
                        "id": "944abf0a8fc22fe1f1154a389a574154"
                    },
                    {
                        "path": "/modules/message/sysMessageTemplateList",
                        "component": "modules/message/SysMessageTemplateList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "模板管理"
                        },
                        "name": "modules-message-sysMessageTemplateList",
                        "id": "f780d0d3083d849ccbdb1b1baee4911d"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "message",
                    "title": "消息中心"
                },
                "name": "message",
                "id": "5c8042bd6c601270b2bbd9b20bccc68b"
            },
            {
                "redirect": null,
                "path": "/jeecg",
                "component": "layouts/RouteView",
                "route": "1",
                "children": [
                    {
                        "path": "/jeecg/jeecgDemoList",
                        "component": "jeecg/JeecgDemoList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "单表模型示例"
                        },
                        "name": "DemoList",
                        "id": "4148ec82b6acd69f470bea75fe41c357"
                    },
                    {
                        "path": "/jeecg/TableExpandeSub",
                        "component": "jeecg/TableExpandeSub",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "内嵌Table"
                        },
                        "name": "jeecg-TableExpandeSub",
                        "id": "4356a1a67b564f0988a484f5531fd4d9"
                    },
                    {
                        "path": "/jeecg/ImagPreview",
                        "component": "jeecg/ImagPreview",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "图片预览"
                        },
                        "name": "jeecg-ImagPreview",
                        "id": "655563cd64b75dcf52ef7bcdd4836953"
                    },
                    {
                        "path": "/jeecg/SelectDemo",
                        "component": "jeecg/SelectDemo",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "常用选择组件"
                        },
                        "name": "jeecg-SelectDemo",
                        "id": "9a90363f216a6a08f32eecb3f0bf12a3"
                    },
                    {
                        "path": "/jeecg/tablist/JeecgOrderDMainList",
                        "component": "jeecg/tablist/JeecgOrderDMainList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "一对多Tab示例"
                        },
                        "name": "jeecg-tablist-JeecgOrderDMainList",
                        "id": "6ad53fd1b220989a8b71ff482d683a5a"
                    },
                    {
                        "path": "/jeecg/JeecgOrderMainList",
                        "component": "jeecg/JeecgOrderMainList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "一对多示例"
                        },
                        "name": "jeecg-JeecgOrderMainList",
                        "id": "fb367426764077dcf94640c843733985"
                    },
                    {
                        "path": "/jeecg/JeecgTreeTable",
                        "component": "jeecg/JeecgTreeTable",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "异步树列表Demo"
                        },
                        "name": "jeecg-JeecgTreeTable",
                        "id": "0620e402857b8c5b605e1ad9f4b89350"
                    },
                    {
                        "path": "/jeecg/JeecgOrderMainListForJEditableTable",
                        "component": "jeecg/JeecgOrderMainListForJEditableTable",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "一对多JEditable"
                        },
                        "name": "jeecg-JeecgOrderMainListForJEditableTable",
                        "id": "c431130c0bc0ec71b0a5be37747bb36a"
                    },
                    {
                        "path": "/jeecg/jeecgPdfView",
                        "component": "jeecg/JeecgPdfView",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "PDF预览"
                        },
                        "name": "jeecg-jeecgPdfView",
                        "id": "e1979bb53e9ea51cecc74d86fd9d2f64"
                    },
                    {
                        "path": "/jeecg/PrintDemo",
                        "component": "jeecg/PrintDemo",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "打印测试"
                        },
                        "name": "jeecg-PrintDemo",
                        "id": "e6bfd1fcabfd7942fdd05f076d1dad38"
                    },
                    {
                        "path": "/jeecg/imgDragSort",
                        "component": "jeecg/ImgDragSort",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "图片拖拽排序"
                        },
                        "name": "jeecg-imgDragSort",
                        "id": "265de841c58907954b8877fb85212622"
                    },
                    {
                        "path": "/jeecg/helloworld",
                        "component": "jeecg/helloworld",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "helloworld"
                        },
                        "name": "jeecg-helloworld",
                        "id": "339329ed54cf255e1f9392e84f136901"
                    },
                    {
                        "path": "/jeecg/imgTurnPage",
                        "component": "jeecg/ImgTurnPage",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "图片翻页"
                        },
                        "name": "jeecg-imgTurnPage",
                        "id": "58b9204feaf07e47284ddb36cd2d8468"
                    },
                    {
                        "path": "bfa89e563d9509fbc5c6503dd50faf2e",
                        "component": "layouts/IframePageView",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "百度",
                            "url": "http://www.baidu.com"
                        },
                        "name": "http@--www.baidu.com",
                        "id": "a400e4f4d54f79bf5ce160ae432231af"
                    },
                    {
                        "path": "/jeecg/splitPanel",
                        "component": "jeecg/SplitPanel",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "分屏"
                        },
                        "name": "jeecg-splitPanel",
                        "id": "3fac0d3c9cd40fa53ab70d4c583821f8"
                    },
                    {
                        "path": "/jeecg/InterfaceTest",
                        "component": "jeecg/InterfaceTest",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "数据回执模拟"
                        },
                        "name": "jeecg-InterfaceTest",
                        "id": "c6cf95444d80435eb37b2f9db3971ae6"
                    },
                    {
                        "path": "/jeecg/JEditableTable",
                        "component": "jeecg/JeecgEditableTableExample",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "JEditableTable示例"
                        },
                        "name": "jeecg-JEditableTable",
                        "id": "7960961b0063228937da5fa8dd73d371"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "qrcode",
                    "title": "常见案例"
                },
                "name": "jeecg",
                "id": "2a470fc0c3954d9dbb61de6d80846549"
            },
            {
                "redirect": null,
                "path": "/result",
                "component": "layouts/PageView",
                "route": "1",
                "children": [
                    {
                        "path": "/result/success",
                        "component": "result/Success",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "成功"
                        },
                        "name": "result-success",
                        "id": "00a2a0ae65cdca5e93209cdbde97cbe6"
                    },
                    {
                        "path": "/result/fail",
                        "component": "result/Error",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "失败"
                        },
                        "name": "result-fail",
                        "id": "13212d3416eb690c2e1d5033166ff47a"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "check-circle-o",
                    "title": "结果页"
                },
                "name": "result",
                "id": "2e42e3835c2b44ec9f7bc26c146ee531"
            },
            {
                "redirect": null,
                "path": "/profile",
                "component": "layouts/RouteView",
                "route": "1",
                "children": [
                    {
                        "path": "/profile/basic",
                        "component": "profile/basic/Index",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "基础详情页"
                        },
                        "name": "profile-basic",
                        "id": "cc50656cf9ca528e6f2150eba4714ad2"
                    },
                    {
                        "path": "/profile/advanced",
                        "component": "profile/advanced/Advanced",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "高级详情页"
                        },
                        "name": "profile-advanced",
                        "id": "b3c824fc22bd953e2eb16ae6914ac8f9"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "profile",
                    "title": "详情页"
                },
                "name": "profile",
                "id": "4875ebe289344e14844d8e3ea1edd73f"
            },
            {
                "redirect": null,
                "path": "/exception",
                "component": "layouts/RouteView",
                "route": "1",
                "children": [
                    {
                        "path": "/exception/403",
                        "component": "exception/403",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "403"
                        },
                        "name": "exception-403",
                        "id": "65a8f489f25a345836b7f44b1181197a"
                    },
                    {
                        "path": "/exception/404",
                        "component": "exception/404",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "404"
                        },
                        "name": "exception-404",
                        "id": "d2bbf9ebca5a8fa2e227af97d2da7548"
                    },
                    {
                        "path": "/exception/500",
                        "component": "exception/500",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "500"
                        },
                        "name": "exception-500",
                        "id": "b4dfc7d5dd9e8d5b6dd6d4579b1aa559"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "warning",
                    "title": "异常页"
                },
                "name": "exception",
                "id": "c65321e57b7949b7a975313220de0422"
            },
            {
                "redirect": "/list/query-list",
                "path": "/list",
                "component": "layouts/PageView",
                "route": "1",
                "children": [
                    {
                        "path": "/list/query-list",
                        "component": "list/TableList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "查询表格"
                        },
                        "name": "list-query-list",
                        "id": "418964ba087b90a84897b62474496b93"
                    },
                    {
                        "path": "/list/edit-table",
                        "component": "list/TableInnerEditList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "内联编辑表格"
                        },
                        "name": "list-edit-table",
                        "id": "ae4fed059f67086fd52a73d913cf473d"
                    },
                    {
                        "path": "/list/user-list",
                        "component": "list/UserList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "用户列表"
                        },
                        "name": "list-user-list",
                        "id": "05b3c82ddb2536a4a5ee1a4c46b5abef"
                    },
                    {
                        "path": "/list/role-list",
                        "component": "list/RoleList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "角色列表"
                        },
                        "name": "list-role-list",
                        "id": "4f84f9400e5e92c95f05b554724c2b58"
                    },
                    {
                        "path": "/list/permission-list",
                        "component": "list/PermissionList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "权限列表"
                        },
                        "name": "list-permission-list",
                        "id": "73678f9daa45ed17a3674131b03432fb"
                    },
                    {
                        "path": "/list/basic-list",
                        "component": "list/StandardList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "标准列表"
                        },
                        "name": "list-basic-list",
                        "id": "f23d9bfff4d9aa6b68569ba2cff38415"
                    },
                    {
                        "path": "/list/card",
                        "component": "list/CardList",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "卡片列表"
                        },
                        "name": "list-card",
                        "id": "7ac9eb9ccbde2f7a033cd4944272bf1e"
                    },
                    {
                        "path": "/list/search",
                        "component": "list/search/SearchLayout",
                        "route": "1",
                        "children": [
                            {
                                "path": "/list/search/article",
                                "component": "list/TableList",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "搜索列表（文章）"
                                },
                                "name": "list-search-article",
                                "id": "078f9558cdeab239aecb2bda1a8ed0d1"
                            },
                            {
                                "path": "/list/search/application",
                                "component": "list/TableList",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "搜索列表（应用）"
                                },
                                "name": "list-search-application",
                                "id": "200006f0edf145a2b50eacca07585451"
                            },
                            {
                                "path": "/list/search/project",
                                "component": "list/TableList",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "搜索列表（项目）"
                                },
                                "name": "list-search-project",
                                "id": "de13e0f6328c069748de7399fcc1dbbd"
                            }
                        ],
                        "meta": {
                            "keepAlive": "true",
                            "title": "搜索列表"
                        },
                        "name": "list-search",
                        "id": "fb07ca05a3e13674dbf6d3245956da2e"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "table",
                    "title": "列表页"
                },
                "name": "list",
                "id": "540a2936940846cb98114ffb0d145cb8"
            },
            {
                "redirect": null,
                "path": "/account",
                "component": "layouts/RouteView",
                "route": "1",
                "children": [
                    {
                        "path": "/account/center",
                        "component": "account/center/Index",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "个人中心"
                        },
                        "name": "account-center",
                        "id": "d86f58e7ab516d3bc6bfb1fe10585f97"
                    },
                    {
                        "path": "/account/settings/base",
                        "component": "account/settings/Index",
                        "route": "1",
                        "children": [
                            {
                                "path": "/account/settings/base",
                                "component": "account/settings/BaseSetting",
                                "route": "1",
                                "hidden": true,
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "基本设置"
                                },
                                "name": "account-settings-base",
                                "id": "1367a93f2c410b169faa7abcbad2f77c"
                            },
                            {
                                "path": "/account/settings/binding",
                                "component": "account/settings/Binding",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "账户绑定"
                                },
                                "name": "account-settings-binding",
                                "id": "4f66409ef3bbd69c1d80469d6e2a885e"
                            },
                            {
                                "path": "/account/settings/custom",
                                "component": "account/settings/Custom",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "个性化设置"
                                },
                                "name": "account-settings-custom",
                                "id": "882a73768cfd7f78f3a37584f7299656"
                            },
                            {
                                "path": "/account/settings/security",
                                "component": "account/settings/Security",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "安全设置"
                                },
                                "name": "account-settings-security",
                                "id": "ec8d607d0156e198b11853760319c646"
                            },
                            {
                                "path": "/account/settings/notification",
                                "component": "account/settings/Notification",
                                "route": "1",
                                "meta": {
                                    "keepAlive": "true",
                                    "title": "新消息通知"
                                },
                                "name": "account-settings-notification",
                                "id": "fedfbf4420536cacc0218557d263dfea"
                            }
                        ],
                        "meta": {
                            "keepAlive": "true",
                            "title": "个人设置"
                        },
                        "name": "account-settings-base",
                        "id": "6e73eb3c26099c191bf03852ee1310a1",
                        "alwaysShow": true
                    },
                    {
                        "path": "/dashboard/workplace",
                        "component": "dashboard/Workplace",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "工作台"
                        },
                        "name": "dashboard-workplace",
                        "id": "8fb8172747a78756c11916216b8b8066"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "user",
                    "title": "个人页"
                },
                "name": "account",
                "id": "717f6bee46f44a3897eca9abd6e2ec44"
            },
            {
                "redirect": null,
                "path": "/form",
                "component": "layouts/PageView",
                "route": "1",
                "children": [
                    {
                        "path": "/form/base-form",
                        "component": "form/BasicForm",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "基础表单"
                        },
                        "name": "form-base-form",
                        "id": "277bfabef7d76e89b33062b16a9a5020"
                    },
                    {
                        "path": "/form/step-form",
                        "component": "form/stepForm/StepForm",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "分步表单"
                        },
                        "name": "form-step-form",
                        "id": "6531cf3421b1265aeeeabaab5e176e6d"
                    },
                    {
                        "path": "/form/advanced-form",
                        "component": "form/advancedForm/AdvancedForm",
                        "route": "1",
                        "meta": {
                            "keepAlive": "true",
                            "title": "高级表单"
                        },
                        "name": "form-advanced-form",
                        "id": "e5973686ed495c379d829ea8b2881fc6"
                    }
                ],
                "meta": {
                    "keepAlive": "true",
                    "icon": "form",
                    "title": "表单页"
                },
                "name": "form",
                "id": "e3c13679c73a4f829bcff2aba8fd68b1"
            }
        ]
    },
    "timestamp": 1558921538231
}
```

完整字段可能长这样：
```
/**
 * 路由配置说明：
 * 建议：sider menu 请不要超过三级菜单，若超过三级菜单，则应该设计为顶部主菜单 配合左侧次级菜单
 *
 **/
 {
  redirect: noredirect,
  name: 'router-name',
  hidden: true,
  meta: {
    title: 'title',
    icon: 'a-icon',
    keepAlive: true,
    hiddenHeaderContent: true,
  }
}   
```

下面就对字段进行详细说明：

{ Route } 对象

| 参数  |  说明  |  类型  |  默认值  |
| :---: | :---: | :---: | :---: |
|  hidden  |  控制路由是否显示在sidebar  |  boolean  |  false  |
|  redirect  |  重定向地址，访问这个路由时，自动进行重定向  |  string  |  -  |
|  name  |  路由名称，建议设置，且不能重名  |  string  |  —  |
|  meta  |  路由元信息(路由附带扩展信息)  |  object  |  {}  |

{ Meta } 路由元信息对象

| 参数  |  说明  |  类型  |  默认值  |
| :---: | :---: | :---: | :---: |
| title  |  路由标题，用于显示面包屑，页面标题*推荐设置  |  string  |  -  |
| icon  |  路由在menu上显示的图标  |  string  |  -  |
| keepAlive  |  缓存该路由  |  boolean  |  false  |
| hiddenHeaderContent  |  *特殊 隐藏 PageHeader 组件中的页面带的 面包屑和页面标题栏  |  boolean  |  false  |
| permission  |  与项目提供的权限拦截匹配的权限，如果不匹配，则会被禁止访问该路由页面  |  array  |  []  |

路由例子

```
const asyncRouterMap = [
  {
    path: '/',
    name: 'index',
    component: BasicLayout,
    meta: { title: '首页' },
    redirect: '/dashboard/analysis',
    children: [
      {
        path: '/dashboard',
        component: Layout,
        name: 'dashboard',
        redirect: '/dashboard/workplace',
        meta: {title: '仪表盘', icon: 'dashboard', permission: ['dashboard']},
        children: [
          {
            path: '/dashboard/analysis',
            name: 'Analysis',
            component: () => import('@/views/dashboard/Analysis'),
            meta: {title: '分析页', permission: ['dashboard']}
          },
          {
            path: '/dashboard/monitor',
            name: 'Monitor',
            hidden: true,
            component: () => import('@/views/dashboard/Monitor'),
            meta: {title: '监控页', permission: ['dashboard']}
          },
          {
            path: '/dashboard/workplace',
            name: 'Workplace',
            component: () => import('@/views/dashboard/Workplace'),
            meta: {title: '工作台', permission: ['dashboard']}
          }
        ]
      },

      // result
      {
        path: '/result',
        name: 'result',
        component: PageView,
        redirect: '/result/success',
        meta: { title: '结果页', icon: 'check-circle-o', permission: [ 'result' ] },
        children: [
          {
            path: '/result/success',
            name: 'ResultSuccess',
            component: () => import(/* webpackChunkName: "result" */ '@/views/result/Success'),
            // 该页面隐藏面包屑和页面标题栏
            meta: { title: '成功', hiddenHeaderContent: true, permission: [ 'result' ] }
          },
          {
            path: '/result/fail',
            name: 'ResultFail',
            component: () => import(/* webpackChunkName: "result" */ '@/views/result/Error'),
            // 该页面隐藏面包屑和页面标题栏
            meta: { title: '失败', hiddenHeaderContent: true, permission: [ 'result' ] }
          }
        ]
      },
      ...
    ]
  },
]
```

### 按钮级权限控制

按钮的权限控制通过注册一个指令去实现验证，但是单单靠一个指令，有时候并不能完全满足，所以还会需要配合v-if

```
//v-has 
<a-button @click="handleAdd" v-has="'user:add'" type="primary" icon="plus">添加用户</a-button>
```
```
import { USER_AUTH,SYS_BUTTON_AUTH } from "@/store/mutation-types"

const hasPermission = {
    install (Vue, options) {
        console.log(options);
          Vue.directive('has', {
            inserted: (el, binding, vnode)=>{
                console.log("页面权限控制----");
                //节点权限处理，如果命中则不进行全局权限处理
                if(!filterNodePermission(el, binding, vnode)){
                  filterGlobalPermission(el, binding, vnode);
                }
            }
          });
    }
};

/**
 * 全局权限控制
 */
export function filterNodePermission(el, binding, vnode) {
  console.log("页面权限--NODE--");

  var permissionList = [];
  try {
    var obj = vnode.context.$props.formData;
    if (obj) {
      let bpmList = obj.permissionList;
      for (var bpm of bpmList) {
        if(bpm.type != '2') {
          permissionList.push(bpm);
        }
      }
    }
  } catch (e) {
    //console.log("页面权限异常----", e);
  }
  if (permissionList === null || permissionList === "" || permissionList === undefined||permissionList.length<=0) {
    //el.parentNode.removeChild(el)
    return false;
  }
  let permissions = [];
  for (var item of permissionList) {
    if(item.type != '2') {
      permissions.push(item.action);
    }
  }
  //console.log("页面权限----"+permissions);
  //console.log("页面权限----"+binding.value);
  if (!permissions.includes(binding.value)) {
    //el.parentNode.removeChild(el)
    return false;
  }else{
    for (var item2 of permissionList) {
      if(binding.value === item2.action){
        return true;
      }
    }
  }
  return false;
}

/**
 * 全局权限控制
 */
export function filterGlobalPermission(el, binding, vnode) {
  console.log("页面权限--Global--");

  var permissionList = [];
  var allPermissionList = [];

  //let authList = Vue.ls.get(USER_AUTH);
  let authList = JSON.parse(sessionStorage.getItem(USER_AUTH) || "[]");
  for (var auth of authList) {
    if(auth.type != '2') {
      permissionList.push(auth);
    }
  }
  //console.log("页面权限--Global--",sessionStorage.getItem(SYS_BUTTON_AUTH));
  let allAuthList = JSON.parse(sessionStorage.getItem(SYS_BUTTON_AUTH) || "[]");
  for (var gauth of allAuthList) {
    if(gauth.type != '2') {
      allPermissionList.push(gauth);
    }
  }
  //设置全局配置是否有命中
  var invalidFlag = false;//无效命中
  if(allPermissionList != null && allPermissionList != "" && allPermissionList != undefined && allPermissionList.length > 0){
    for (var itemG of allPermissionList) {
      if(binding.value === itemG.action){
        if(itemG.status == '0'){
          invalidFlag = true;
          break;
        }
      }
    }
  }
  if(invalidFlag){
    return;
  }
  if (permissionList === null || permissionList === "" || permissionList === undefined||permissionList.length<=0) {
    el.parentNode.removeChild(el);
    return;
  }
  let permissions = [];
  for (var item of permissionList) {
    if(item.type != '2'){
      permissions.push(item.action);
    }
  }
  if (!permissions.includes(binding.value)) {
    el.parentNode.removeChild(el);
  }
}

export default hasPermission;
```



