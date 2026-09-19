# nas-util

NAS 上用的工具镜像:yt-dlp + ffmpeg,支持 Intel 核显硬件编解码(VAAPI / QSV)。

镜像:`shadowdk/nas-util:latest`,另有按构建日期打的标签(如 `shadowdk/nas-util:20260919`)可用来回滚。
推送到 master 会自动构建发布;只想更新 yt-dlp 时,在 Actions 页面手动 Run workflow 即可。

## 运行

```sh
docker run -it --rm --device /dev/dri -v /你的下载目录:/data shadowdk/nas-util
```

- `--device /dev/dri` 把核显交给容器,不加就只能软件编码。
- 容器里用 root 跑就行;换成普通用户要再加 `--group-add <宿主机 render 组的 gid>`。
- 下 YouTube 要走代理:`yt-dlp --proxy http://代理地址:端口 ...`。镜像里已带 deno,YouTube 的 JS 验证靠它。

## 检查硬件加速

```sh
vainfo --display drm --device /dev/dri/renderD128
ffmpeg -hwaccel vaapi -hwaccel_device /dev/dri/renderD128 -hwaccel_output_format vaapi -i in.mp4 -c:v hevc_vaapi out.mp4
ffmpeg -init_hw_device qsv=hw:/dev/dri/renderD128 -hwaccel qsv -hwaccel_output_format qsv -c:v h264_qsv -i in.mp4 -c:v hevc_qsv out.mp4
```

在 i3-9300T(UHD 630)+ fnOS 上实测过:VAAPI 和 QSV 的 H.264 / HEVC 硬解硬编、HEVC 10bit 硬解都能用。
