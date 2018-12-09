# 2.3 EurekaServer的REST API简介

### 2.3.1 REST API列表

|       操作       |                 http动作                 |          描述           |
|:----------------:|:----------------------------------------:|:-----------------------:|
| 注册新的应用实例 |        POST /eureka/apps/{appsId}        | 可以输入xml或json的body |
|   注销应用实例   | DELETE /eureka/apps/{appId}/{instanceId} |                         |
| 发送心跳   |   |   |
