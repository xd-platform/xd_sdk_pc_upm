# XDGSDK-PC-6.0

## SDK 下载

[下载地址](https://github.com/xd-platform/xd_sdk_pc_upm/releases/download/6.8.2/xd-pc-sdk.unitypackage)

## 添加依赖

### UPM 依赖

```json
"com.leancloud.storage": "https://github.com/leancloud/csharp-sdk-upm.git#storage-1.0.3",
"com.leancloud.realtime": "https://github.com/leancloud/csharp-sdk-upm.git#realtime-1.0.3",
"com.taptap.tds.bootstrap": "https://github.com/TapTap/TapBootstrap-Unity.git#3.16.5",
"com.taptap.tds.common": "https://github.com/TapTap/TapCommon-Unity.git#3.16.5",
"com.taptap.tds.login": "https://github.com/taptap/TapLogin-Unity.git#3.16.5",
"com.taptap.tds.tapdb": "https://github.com/TapTap/TapDB-Unity.git#3.16.5",
"com.unity.textmeshpro": "3.0.6",
```

### PC 浏览器插件

SDK 中使用到了付费 PC WebView 插件：[3D WebView for Windows and macOS](https://assetstore.unity.com/packages/tools/gui/3d-webview-for-windows-and-macos-web-browser-154144)
内部游戏可以联系 TDS 获得；外部游戏则需要自行购买。将插件导入到 Unity 工程即可。

### Steam 插件

XD SDK 使用 [Steamworks.NET](https://steamworks.github.io/) 支持 Steam 授权，开发者可以自行下载 Steamworks.NET SDK 到工程。

XD SDK 通过 `XD_STEAM_SUPPORT` 宏确定是否要通过 Steam SDK 授权，即声明 `XD_STEAM_SUPPORT` 宏，则认为是 Steam 包，通过 Steam SDK 授权；否则通过 Web 获得 Steam 授权。

## 配置信息

与移动端一致，在 Resources 目录下放置 XDConfig.json，只要内容包括：

```json
{
  "client_id": "d4bjgwom9zk84wk",
  "region_type": "CN",
  "game_name": "XD-CN Demo",
  "app_id": "1010",
  "tapsdk": {
    "client_id": "Wzy7xYhKtYdnLUXevV",
    "client_token": "ocXvBmR1EecL7CtmFEugoC016QKiptDC0vS5pMIf",
    "server_url": "https://jfqhf3x9.cloud.tds1.tapapis.cn",
    "permissions": [
      "public_profile",
      "user_friends"
    ]
  },
  "logout_url": "https://logout-xdsdk.xd.cn/",
  "web_pay_url": "https://sdkpay-test.xd.cn/sdk",
  "report_url": "https://www.xd.com/service",
}
```

### Steam 配置

如果需要支持 Steam 主机登录，需要将 steam_appid.txt 放置 Unity 工程根目录下（其中只包括 Steam app id）。

## 日志调试

```cs
XDLogger.LogDelegate = (level, message) => {
    switch (level) {
        case XDLogLevel.Debug:
            Debug.Log($"[DEBUG] {message}");
            break;
        case XDLogLevel.Warn:
            Debug.LogWarning($"[WARN] {message}");
            break;
        case XDLogLevel.Error:
            Debug.LogError($"[ERROR] {message}");
            break;
        default:
            break;
    }
};
```

## 多语言

### 支持语言种类

```cs
public enum LanguageType {
    ZH_CN = 0,  // 简体中文
    ZH_TW = 1,  // 繁体中文
    EN = 2,     // 英文
    TH = 3,     // 泰文
    ID = 4,     // 印尼文
    KR = 5,     // 韩语
    JP = 6,     // 日语
    DE = 7,     // 德语
    FR = 8,     // 法语
    PT = 9,     // 葡萄牙语
    ES = 10,    // 西班牙语
    TR = 11,    // 土耳其语
    RU = 12,    // 俄罗斯语
}
```

### 设置语言

```cs
XDSDK.SetLanguage(languageType);
```

## 初始化

```cs
try {
    await XDSDK.Init();
    ResultText.text = "初始化成功";
} catch (Exception e) {
    ResultText.text = $"初始化失败：{e}";
}
```

## 获取 SDK 回调

```cs
XDSDK.UserStatusChangeDelegate = new UserStatusChangeDelegate {
    // 绑定成功回调
    OnBind = loginType => {
        ResultText.text = $"绑定成功: {loginType}";
    },
    // 解绑成功回调
    OnUnbind = loginType => {
        ResultText.text = $"解绑成功: {loginType}";
    },
    // 登出回调
    OnLogout = () => {
        ResultText.text = "退出登录";
    },
    // 登出后签署协议回调
    OnProtocolAgreedAfterLogout = () => {
        ResultText.text = "退出登录后签署协议";
    }
};
```

## 登录

### 支持登录类型

```cs
public enum LoginType { 
    Default     = -1,   // 自动登录，以上次登录成功的信息登录
    Guest       = 0,    // 游客登录
    Apple       = 2,    // 苹果登录
    Google      = 3,    // Google 登录
    TapTap      = 5,    // Tap 登录
    Steam       = 19,   // Steam 登录
}
```

### 登录

```cs
try {
    XDUser user = await XDSDK.LoginByType(LoginType.TapTap);
    ResultText.text = $"Tap登录成功：{user.NickName} userId: {user.UserId} kid: {user.AccessToken.Kid}";
} catch (Exception e) {
    ResultText.text = $"登录失败：{e}";
}
```

### 主机登录

目前只支持 Steam 登录

```cs
try {
    XDUser user = await XDSDK.LoginByConsole();
    ResultText.text = $"Steam 主机登录成功：{user.NickName} userId: {user.UserId} kid: {user.AccessToken.Kid}";
} catch (Exception e) {
    ResultText.text = $"Steam 主机登录失败：code: {e}";
}
```

### 建议用法

- 先自动登录，如果自动登录失败，则调用 Tap 或游客登录
- 为保证登录效率，TDSUser 采用本地创建的方式，如需获取 TDSUser 信息，需要再次拉取

```cs
TDSUser current = await TDSUser.GetCurrent();
await current.Fetch();
```

## 获取用户信息

```cs
try {
    XDUser user = await XDSDK.GetUserInfo();
    ResultText.text = $"获取成功：{user.NickName} userId: {user.UserId} kid: {user.AccessToken.Kid}";
} catch (Exception e) {
    ResultText.text = $"获取失败：code: {e}";
}
```

## 打开账号中心

```cs
XDSDK.OpenUserCenter();
```

## 支付

### 支付

国内目前支持微信和支付宝，采用内置浏览器的 Web 支付方式；海外目前支持跳转官网浏览器支付方式

```cs
try {
    await XDSDK.PayWithWeb(serverId, roleId, productId, 
        orderId: orderId,
        productName: productName,
        payAmount: payAmount,
        extras: extras);
    XDLogger.Debug("支付完成");
} catch (TaskCanceledException) {
    XDLogger.Debug("取消支付");
} catch (Exception e) {
    XDLogger.Debug($"支付异常: {e}");
}
```

## 打开客服中心

```cs
XDSDK.OpenCustomerCenter(serverId, roleId, roleName);
```

## 是否初始化

```cs
XDSDK.IsInitialized;
```

## 获取版本号

```cs
XDSDK.SDKVersion;
```

## 退出登录

```cs
XDSDK.Logout();
```

## Windows 平台网页登录唤醒

参考 [文档](https://developer.taptap.com/docs/sdk/taptap-login/guide/start/#windows-%E5%B9%B3%E5%8F%B0)

## 注意

PC SDK 中使用到了付费的 WebView 插件，Vuplex 目录只允许在公司内使用，切勿传播。
