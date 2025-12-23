# 一键登录（简化版）

## 1. 背景
用户注册/登录时需验证手机号。  
系统优先尝试「一键登录」（基于运营商本机号认证），**失败后降级到短信类通道**（SMS / TTS / FlashCall），具体通道由风控、国家策略和通道可用性决定。

> 💡 说明：一键登录本身**不发送短信**，而是通过运营商网络直接验证本机号码。

## 2. 核心流程
```mermaid
sequenceDiagram
    participant Client as Client
    participant Server as Server
    participant Gateway as Gateway
    participant Carrier as Carrier

    Client->>Server: 1. get_started 请求
    Server->>Gateway: 2. 申请认证 URI
    Gateway->>Carrier: 3. 运营商黑盒处理
    Carrier-->>Gateway: 4. 返回 target URI
    Gateway-->>Server: 5. 返回 URI
    Server-->>Client: 6. 返回 URI 给客户端

    Client->>Server: 7. 访问 target URI（触发验证）
    Server->>Gateway: 8. 查询验证状态
    Gateway->>Carrier: 9. 确认用户已授权
    Carrier-->>Gateway: 10. 返回 200（已验证）
    Gateway-->>Server: 11. 返回 200
    Server-->>Client: 12. 返回成功

    Client->>Server: 13. 调用 phone_login / cookie_login 完成登录
```

## 3. 关键说明
一键登录成功：直接拿到本机手机号，无需用户输入验证码。    
一键登录失败（如无 SIM 卡、用户拒授权）：前端自动降级到短信验证码流程。     
整个过程对用户透明，仅需点击“一键登录”并确认授权弹窗。
