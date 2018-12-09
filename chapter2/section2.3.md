# 2.3 EurekaServer的REST API简介

### 2.3.1 REST API列表

|             操作              |                             http动作                              |
|:-----------------------------:|:-----------------------------------------------------------------:|
|       注册新的应用实例        |                    POST /eureka/apps/{appsId}                     |
|         注销应用实例          |             DELETE /eureka/apps/{appId}/{instanceId}              |
|           发送心跳            |               PUT /eureka/apps/{appId}/{instanceId}               |
|         查询所有实例          |                         GET /eureka/apps                          |
|      查询指定appId的实例      |                     GET /eureka/apps/{appId}                      |
| 根据指定appId和instanceId查询 |               GET /eureka/apps/{appId}/{instanceId}               |
|    根据指定instanceId查询     |                GET /eureka/instances/{instanceId}                 |
|         暂停应用实例          | PUT /eureka/apps/{appId}/{instanceId}/status?value=OUT_OF_SERVICE |
|         恢复应用实例          |     DELETE /eureka/apps/{appId}/{instanceId}/status?value=UP      |
|          更新元数据           |     PUT /eureka/apps/{appId}/{instanceId}/metadata?key=value      |
|        根据vip地址查询        |                   GET /eureka/vips/{vipAddress}                   |
|       根据svip地址查询        |                  GET /eureka/vips/{svipAddress}                   |
