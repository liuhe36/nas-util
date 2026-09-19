# nas-util

NAS 上用的工具镜像:yt-dlp + ffmpeg,支持 Intel 核显硬件编解码(VAAPI / QSV)。

镜像:`shadowdk/nas-util:latest`,另有按构建日期打的标签(如 `shadowdk/nas-util:20260919`)可用来回滚。
推送到 master 会自动构建发布;只想更新 yt-dlp 时,在 Actions 页面手动 Run workflow 即可。

## 运行

```sh
docker run -dit --init --name nas-util --device /dev/dri \
  -e HOME=/config -e YTDLP_PROXY=http://代理地址:端口 \
  -v /你的下载目录:/data -v /你的配置目录:/config shadowdk/nas-util
docker exec -it nas-util sh
```

- `--device /dev/dri` 把核显交给容器,不加就只能软件编码。
- 用普通用户跑(`--user`)时要再加 `--group-add <宿主机 render 组的 gid>`。
- 镜像里已带 deno,YouTube 的 JS 验证靠它。

## 下载命令

只给链接就能下,链接可以给多个,后面还能跟任何 yt-dlp 参数:

| 命令 | 做什么 |
|---|---|
| `dl 链接` | 最高画质 + 最佳音质,直连(B 站等国内站) |
| `pdl 链接` | 同上,走 `YTDLP_PROXY` 环境变量里的代理(YouTube 等) |
| `dla 链接` | 只要音频,直连 |
| `pdla 链接` | 只要音频,走代理 |

- 存到 `/data/web-DL/<站点>/`,音频在其下的 `audio/`;播放列表、频道、合集再建一层子目录。
- 画质:在最高分辨率里挑视频码率最高的流,所以一般不会挑到 AV1。B 站的 AV1 码率只有 AVC 的四分之一,9 代核显也解不了 AV1。
- `/config/cookies.txt` 存在就自动带上(B 站 1080P 高码率以上要大会员 cookies)。
- 下载在后台跑,关掉终端或 Ctrl+C 都不会中断,日志在 `/config/logs`,保留 30 天;要停就 `pkill yt-dlp`。

## 检查硬件加速

```sh
vainfo --display drm --device /dev/dri/renderD128
ffmpeg -hwaccel vaapi -hwaccel_device /dev/dri/renderD128 -hwaccel_output_format vaapi -i in.mp4 -c:v hevc_vaapi out.mp4
ffmpeg -init_hw_device qsv=hw:/dev/dri/renderD128 -hwaccel qsv -hwaccel_output_format qsv -c:v h264_qsv -i in.mp4 -c:v hevc_qsv out.mp4
```

在 i3-9300T(UHD 630)+ fnOS 上实测过:VAAPI 和 QSV 的 H.264 / HEVC 硬解硬编、HEVC 10bit 硬解都能用。
