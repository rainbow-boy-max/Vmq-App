# 修改说明（Modifications）

本项目 fork 自 <https://github.com/shinian-a/Vmq-App>（Apache License 2.0）。
按 Apache-2.0 第 4(b) 条要求，此文件声明本仓库相对上游的修改内容。

## 2026-10-30 · 心跳链路修复

文件：`app/src/main/java/com/shinian/pay/service/PayNotificationListenerService.java`

1. **心跳不再复用 OkHttp 连接池**（`ConnectionPool(0, 1, MILLISECONDS)`）。
   原因：心跳每 50 秒一次，而 OkHttp 默认连接池保活 5 分钟，等于一直复用同一条
   TCP；手机长期在线时该连接会被运营商 NAT / 路由器静默回收（不回 RST），
   复用时请求已写出却读不到响应，而 OkHttp 对「请求已发出之后的读超时」不重试，
   直接判定失败。表现为日志里的 `错误详情：timeout`。
2. **心跳地址由 `http://` 改为 `https://`**，避开 301 跳转，每轮少一次往返。
3. **新增 `heartbeatOnce()`**：连接超时 5s / 读超时 15s，失败后 2s、5s 快速重试，
   替代原先「失败就干等 50 秒」的做法。
4. **心跳成功也落日志**：连接恢复时记一条「心跳已恢复正常」，之后每连续成功 60 次
   （约 50 分钟）记一条，便于确认链路状态；错误改为按错误内容去重（30 分钟一次），
   不再用「日志面板文本 contains」判断——原逻辑会吞掉绝大多数失败记录。
5. **`initAppHeart()` 的重复启动判断改为 `newThread.isAlive()`**，线程死掉后可重建
   （原实现只看引用，线程一旦死亡永远不会再启动）。

## 2026-10-30 · 构建配置

- `app/build.gradle`：`versionCode 13 → 131`，`versionName 3.0 → 3.1.1-fix`
- `app/build.gradle`：签名改为仓库内自签 PKCS12 `app/keystore/keystore.p12`
  （`storeType "pkcs12"`），口令由 CI Secrets 注入
- `.github/workflows/android.yml`：`KEYSTORE_DEFAULT_PATH` 指向 `keystore.p12`

## 许可声明

上游与本仓库均遵循 Apache License 2.0，`LICENSE` 文件保持原样未作改动，
所有上游版权声明均予保留。本仓库与上游作者无隶属关系，不代表上游立场。
