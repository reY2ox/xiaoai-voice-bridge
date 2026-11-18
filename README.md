# xiaoai-voice-bridge

把小爱音箱接进 OpenClaw 的语音桥。**上游 forks：** [ZhengXieGang/Xiaoai-Claw-Addon](https://github.com/ZhengXieGang/Xiaoai-Claw-Addon)（AGPL-3.0-or-later）。

原项目功能是完整的，但对 2026 年的小米接口有几处不适配，装上去会卡在 `-127 client withdraw`，登录成功也拉不到设备。这个 fork 把踩到的坑修在源码里，而不是打补丁。

## 修了什么

| # | 问题 | 修法 |
|---|---|---|
| 1 | `MIIO_USER_AGENT` 是 2020 年的 `iOS-14.4 / Mi Home 6.0.103`，小米已下线该客户端，所有 `api.io.mi.com` 请求返回 `-127 client withdraw` | 换成仍在服务的安卓 UA，并动态带 `deviceId`：`Android-7.1.1-1.0.0-ONEPLUS A3010-136-<deviceId> APP/xiaomi.smarthome APPV/62830` |
| 2 | `isAuthErrorPayload` 不认 `-127`，遇到就把错误直接抛给用户，不重登也不重试 | `-127` 与 `withdraw` 纳入鉴权错误，触发重新登录后重试 |
| 3 | 重登后立刻重试仍会被拒 | 读小米返回的 `result.wait_second`，先等待（3–10 秒）再重试 |
| 4 | python helper 在还没有账号会话时也执行登录，新建米家会话把刚拿到的 token 踢下线 | 只在已有 `xiaomiio` 会话时走 python helper |
| 5 | 二次验证风控下 `micoapi` 密码登录拿不到 serviceToken 直接失败 | 新增回退：先完成 `xiaomiio` 登录拿 `passToken`，再由 python helper 换 `micoapi` token |

## 构建

```bash
npm ci
npm run build        # tsc -> dist/
```

要求 Node 22.22.3+ / 24.15+，OpenClaw 2026.03.24+。

## 安装到 OpenClaw

```bash
openclaw plugins install /path/to/xiaoai-voice-bridge
node scripts/configure-openclaw-install.mjs
# 然后重启 Gateway（插件在内存里，reload 在网关进程内会被拒）
```

首次使用需要小米账号登录：`xiaoai_login_begin` 拿登录入口，扫码/账号密码，账号开了二次验证就按短信验证码走。

## 验证

```bash
xiaoai_get_status     # 初始化状态应为「已就绪」，目标设备应为 小爱音箱Pro (LX06)
xiaoai_speak text="连接测试"
```

## 已知限制

- 小米接口对第三方客户端时常调整，登录态偶发失效，重登即可
- 同一时刻音箱只能播一路：AirPlay 投送 / DLNA 点歌 / 语音播报互斥
- 语音识别文本只能走小米云端，本地 miIO 拿不到

## 许可

AGPL-3.0-or-later，与原项目一致。上游版权归 ZhengXieGang；本 fork 的改动同样以 AGPL 发布。见 `LICENSE` 与 `NOTICE`。
