---
title: FinalShell离线激活持久化配置
date: 2026-09-11
categories:
  - 杂文
  - Windows
tags:
  - FinalShell
  - SSH
  - hosts
  - 离线激活
---

# 问题现象

每次打开 FinalShell 都要手动进行一次离线激活，激活后当次可以使用专业版功能，但关闭软件重新打开后又恢复为未激活状态，需要再次走离线激活流程，非常烦人。

# 根因分析

FinalShell 在每次启动时会尝试连接官方验证服务器校验授权状态。涉及的主要域名包括：

- `hostbuf.com` / `www.hostbuf.com`（官网）
- `youtusoft.com` / `www.youtusoft.com`（软件开发商）
- `dkys.org`
- `tcpspeed.com`
- `wn1998.com` / `www.wn1998.com` / `pwlt.wn1998.com`
- `backup.www.hostbuf.com`

当这些域名可以正常解析并连通时，软件会向服务器发起授权校验请求。服务器检测到当前使用的是"离线激活"而非"在线授权"后，会将激活状态重置为未激活，导致每次启动都需要重新激活。

**本质原因：** 离线激活的状态信息保存在本地，但软件启动时优先向远程服务器校验，服务器端不认可离线激活的结果，直接覆盖本地状态。

# 解决方案

在系统 hosts 文件中将上述验证域名全部指向 `127.0.0.1`，使软件无法连接验证服务器，从而保留本地离线激活状态。

## 操作步骤

### 1. 编辑 hosts 文件

hosts 文件路径：

```
C:\Windows\System32\drivers\etc\hosts
```

以管理员身份打开文本编辑器（推荐 Notepad++ 或直接用管理员权限的记事本），在文件末尾追加以下内容：

```
# === FinalShell offline activation persistence ===
127.0.0.1 www.youtusoft.com
127.0.0.1 youtusoft.com
127.0.0.1 hostbuf.com
127.0.0.1 www.hostbuf.com
127.0.0.1 dkys.org
127.0.0.1 tcpspeed.com
127.0.0.1 www.wn1998.com
127.0.0.1 wn1998.com
127.0.0.1 pwlt.wn1998.com
127.0.0.1 backup.www.hostbuf.com
```

### 2. 验证 hosts 是否生效

打开命令提示符（cmd），执行以下命令：

```
ping dkys.org
ping hostbuf.com
ping youtusoft.com
```

如果返回 `127.0.0.1`，说明 hosts 配置已生效：

```
正在 Ping dkys.org [127.0.0.1] 具有 32 字节的数据:
来自 127.0.0.1 的回复: 字节=32 时间<1ms TTL=128
```

### 3. 执行离线激活

1. 打开 FinalShell
2. 在登录界面随便输入账号密码，点击"离线激活"
3. 复制显示的机器码
4. 使用激活码生成工具（Java 算法）根据机器码生成激活码
5. 将生成的激活码粘贴到软件中，点击激活

### 4. 验证持久化效果

关闭 FinalShell，重新打开。如果不再弹出激活提示，且专业版功能正常可用，说明配置成功。

# 离线激活码生成原理

FinalShell 的离线激活码基于机器码通过 MD5 哈希算法生成，核心逻辑如下：

```java
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class FinalShellKeygen {

    public static void main(String[] args) {
        String machineCode = "替换为你的机器码";
        generateKey(machineCode);
    }

    public static void generateKey(String hardwareId) {
        String proKey = transform(61305 + hardwareId + 8552);
        String pfKey = transform(2356 + hardwareId + 13593);
        System.out.println("Pro 版激活码: " + proKey);
        System.out.println("专业版激活码: " + pfKey);
    }

    public static String transform(String str) {
        String md5 = hashMD5(str);
        return md5.substring(8, 24);
    }

    public static String hashMD5(String str) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] digest = md.digest(str.getBytes());
            StringBuilder sb = new StringBuilder();
            for (byte b : digest) {
                sb.append(String.format("%02x", b));
            }
            return sb.toString();
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }
}
```

**算法要点：**
- 固定盐值 `61305`、`8552`、`2356`、`13593` 与机器码拼接
- 对拼接结果做 MD5 哈希
- 截取哈希值的第 8~24 位（16 个字符）作为最终激活码

也可以使用在线 Java 运行环境直接执行：[https://c.runoob.com/compile/10/](https://c.runoob.com/compile/10/)

# 常见问题排查

如果配置 hosts 后仍然每次要求激活，按以下顺序排查：

| 排查项 | 检查方法 | 解决方案 |
|--------|----------|----------|
| hosts 未生效 | `ping dkys.org` 看是否返回 127.0.0.1 | 刷新 DNS 缓存：`ipconfig /flushdns` |
| 杀毒软件拦截 | 检查杀毒软件是否恢复了 hosts 文件 | 将 hosts 加入杀毒软件白名单 |
| 权限不足 | 检查 hosts 文件修改时间是否更新 | 以管理员身份重新编辑保存 |
| 系统时间异常 | 检查系统时间是否准确 | 同步网络时间（NTP） |
| JDK 版本变动 | 更换过 Java 运行环境 | 重新获取机器码并生成新激活码 |

# 注意事项

- 建议购买官方正版授权以获得最佳体验和云端同步功能
- 修改 hosts 文件需要管理员权限
- 如果 FinalShell 升级到新版本，可能需要重新激活一次
- 系统更新后 hosts 文件可能被重置，需检查是否还在
