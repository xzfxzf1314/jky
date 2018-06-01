# 京客云redisjar包版本变迁内容


## 1.1版本

1. 初始化jar包，添加过滤器等功能，详情配置请见文档

## 1.2版本

1. 修改过滤器返回的数据结构，为了登录认证系统解析通用
2. 登录接口接受推广系统的字段，不过滤userID字段，返回前端时加上sessionID字段
3. redis里不在保存userID和userType字段。具体协议由推广系统和前端约定

## 1.3版本

1. head里增加platform字段(排除 auth/sendSms, auth/register, auth/getAuthImage三个接口)。
2. 对web类型的终端不在做多设备登录的判断，但是mac字段要传。默认值为空。其他终端类型需要传递mac值。

![](http://ohwrspy13.bkt.clouddn.com/18-5-30/47621177.jpg)

## 1.4版本

1.  对web类型的终端不在做多设备登录的判断，但是mac字段要传。默认值为**web-default-mac**。