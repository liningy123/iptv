# APTV 直播源自动维护

Apple TV 上的 APTV 订阅自家 NAS 生成的直播源，全天自动刷新。

## 订阅地址（APTV 里填这个）

```
http://192.168.31.231:8080/m3u
```

## 架构

```
群晖 DS920+ (192.168.31.231)
  └─ Container Manager 项目: iptv-api (host 网络)
       ├─ 每天 06:00 / 18:00 自动拉取上游直播源 → ffmpeg 测速 → 生成 m3u
       └─ 输出: /m3u (全部)  /ipv4/m3u  /ipv6/m3u  /epg/epg.gz
Apple TV (192.168.31.12)
  └─ APTV 订阅上面的 m3u，家里局域网直连，不走外网
```

## 上游项目

- 镜像：`docker.1ms.run/guovern/iptv-api:latest`（Docker Hub 官方镜像的国内代理）
- 项目：https://github.com/Guovin/iptv-api

## 文件说明

| 文件 | 作用 |
|---|---|
| `docker-compose.yml` | NAS 上项目的容器定义（host 网络 + 两个绑定挂载） |
| `config/user_config.ini` | 覆盖配置：IPv6 支持、自动更新时段、排序、自动禁用失效源 |
| `config/subscribe.txt` | 直播源订阅列表（实测可用的 5 个上游） |
| `status.md` | 每日巡检归档（频道数、可用率、异常记录） |

## 维护方式

1. NAS 容器自带定时器，每天 06:00、18:00 各重新拉源 + 测速一次，无需人工干预。
2. `open_auto_disable_source = True`：连续失效的上游源会被自动停用，避免拖慢生成。
3. 本地每日巡检任务（07:30 / 19:30）：检查通道数、可用率、抽样验证 30 条链接，异常时才告警。
4. 需要换源时，只改 `config/subscribe.txt`，重启容器即可。

## 注意

- NAS 必须固定 IP（目前 192.168.31.231），否则 APTV 订阅地址会失效。
- 订阅地址是局域网直连，只在家里网络可用，不做公网暴露。
