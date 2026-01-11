## **详细的原理和注意事项请查看**
- [OIDC协议单点登录](https://www.kancloud.cn/zlt2000/microservices-platform/2278851)

- [单点登出详解](https://www.kancloud.cn/zlt2000/microservices-platform/2539642)



## oauth-center数据库执行以下sql
```sql
alter table oauth_client_details add support_id_token tinyint(1) DEFAULT 1 COMMENT '是否支持id_token';
alter table oauth_client_details add id_token_validity int(11) DEFAULT 60 COMMENT 'id_token有效期';

update oauth_client_details set additional_information = '{"LOGOUT_NOTIFY_URL_LIST":"http://127.0.0.1:8082/logoutNotify"}'
where client_id = 'webApp';
```

## 启动以下服务

1. zlt-uaa：统一认证中心
2. user-center：用户服务
3. sc-gateway：api网关
4. oidc-sso：单点登录demo(webApp应用)
5. web-sso：单点登录demo(app应用)

使用 OIDC 的 ID Token 做认证，Access Token 仅用于授权访问 API
Access Token 的“身份信息”不可信，泄露 能冒充用户登录 能直接调用所有 API  ，身份 + 权限 同时沦陷
而 OIDC 的核心设计就是：
把“你是谁”从 Access Token 中剥离出来，形成一个不可混用的认证凭证
ID Token 的强约束
1必须是 JWT
2必须签名
3必须校验：
 iss
 aud
 exp
 nonce
4audience 是 客户端自己
👉 这才是“我确认你就是你”

一个类比（非常好记）
酒吧入场证 Access Token
身份证 ID Token


Access Token 即使包含用户信息，也只是授权载体，规范不保证其结构与身份语义；
登录必须使用 OIDC 的 ID Token 或服务端 Session，Access Token 不应承担认证职责。
Access Token ->访问资源
ID Token ->身份认证   必须签名



## 测试步骤

1. 登录app应用：
    通过地址 http://127.0.0.1:8081 先登录app应用
2. 访问app应用(单点成功)：
   在浏览器打开一个新的页签(共享session)，通过地址 http://127.0.0.1:8082/ 访问webApp应用，单点登录成功显示当前登录用户名、应用id、token信息
3. 再次切换回app应用 http://127.0.0.1:8081 页面点击登出按钮，登出app应用
4. 回来webApp应用 http://127.0.0.1:8082 刷新页面，返回登录页面（单点登出成功）