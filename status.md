# 2026-11-19 第二版：频道扩容 + 画质升级

- 容器：`iptv-api` 运行中（host 网络，端口 8080），配置 `config/user_config.ini`
- 上游源：6 个
  1. plsy1/iptv `unicast/unicast-aptv.m3u`（山东联通单播 RTSP，经 ghfast.top 代理）
  2. suxuang/myIPTV ipv6
  3. inkss gist ipv6
  4. vbskycn/iptv iptv4
  5. Kimentanm/aptv iptv
  6. fanmingming/live ipv6
- 输出：**93 条 m3u / 51 个频道**，其中 54 条为 `rtsp://`（山东联通原码率）
- 频道覆盖：
  - 央视：CCTV-1 ~ CCTV-15、CCTV-17、CCTV-5+（缺 CCTV-16）
  - 卫视：山东、北京、湖南、浙江、东方、江苏、安徽、天津、辽宁、深圳、东南、湖北、广东、黑龙江、贵州、河北、重庆、海南、四川、江西、广西、吉林、云南、新疆、甘肃、青海、西藏、宁夏、陕西、内蒙古、山西、三沙（共 32+）
  - 地方：广州综合
- 画质策略：`open_filter_resolution` + `min_resolution=1920x1080`；`open_filter_speed` + `min_speed=1.0`（4K 要求 2.0 MiB/s）；`sort_by=resolution,speed`
- RTSP 方案：`config/whitelist.txt` 写入 50 个频道 × 山东联通 RTSP 地址，免测速直接置顶输出
  - 手工 RTSP DESCRIBE 抽验：54 条中 53 条返回 `200 OK`（1 条 412，APTV 仍可尝试）
  - IPTV 服务器需 TCP 传输；APTV 若播不动，在设置里把 RTSP 传输设为 TCP

## 已知缺口

- CCTV-16（奥运）、CETV-1~4：上游订阅源无匹配，需后续专门找源补入 whitelist
- RTSP 白名单条目不做测速，失效时依赖每日巡检发现并替换

## 自动维护

- 容器定时重测：每日 06:00 / 18:00（Asia/Shanghai）
- 外部每日巡检：07:30（日报）/ 19:30（仅异常）
- 订阅地址（APTV）：`http://192.168.31.231:8080/m3u`
