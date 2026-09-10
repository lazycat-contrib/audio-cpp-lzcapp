# audio.cpp for LazyCat

[上游项目](https://github.com/0xShug0/audio.cpp)的 Vulkan WebUI 懒猫应用包，包含模型下载与管理功能，仅发布喵喵商店。

- 支持 amd64，要求懒猫系统 1.5.0+、支持 Vulkan 的 GPU 及可用的 `/dev/dri`。GPU 运行效果须在目标微服验证。
- 服务入口为 8080，保留懒猫登录鉴权；不开放匿名管理 API。
- 模型目录：`/lzcapp/var/models` → `/app/models`。缓存：`/lzcapp/cache` → `/cache`。模型由应用用户共享，随应用数据管理。
- WebUI 可下载模型、上传音频及下载生成结果，已包含懒猫文件选择器注入脚本。
- 启用 `application.gpu_accel`，并在 `lzc-build.yml` 的 `compose_override.services.audio.devices` 显式映射 `/dev/dri:/dev/dri`。容器以 `0:0` 运行，以访问 GPU 和可写模型目录，避免写死宿主机 render/video GID；未启用 privileged。
- 启动命令：`server --ui --ui-management --host 0.0.0.0 --port 8080 --backend vulkan`。

## 构建与更新

```sh
mkdir -p dist
lzc-cli project release -o dist/application.lpk
lzc-cli lpk info dist/application.lpk
```

初始镜像为 `ghcr.io/0xshug0/audio.cpp:full-vulkan-20260909-05f9c5d`，运行镜像使用 `ghcr.1ms.run` 加速源。自动检查会校验源镜像与加速源的 amd64 digest 一致。

每天检查 `full-vulkan-YYYYMMDD-<commit>` 标签，按镜像创建时间选择最新构建。镜像 tag 保持原样；LPK 版本映射为 `YYYY.M.D+<commit>.lzc1`，例如 `2026.9.9+05f9c5d.lzc1`，以区分同一天的多个构建；`lzc1` 表示显式 GPU 设备映射的打包修订。禁止版本降级。手动运行 `build` 可构建、发布当前固定版本；`auto` 会检查上游。

工作流使用 `ca-x/lazycat-github-action`，创建带版本号的 GitHub Release LPK，再发布至喵喵商店；官方商店关闭。输出在 `dist/`，不会打包进 `content/`。

需要组织或仓库 Secrets：`APPSTORE_URL`、`APPSTORE_TOKEN`。`APP_ID`（应用专属覆盖）和 `PRIVATE_STORE_GROUP_CODES`（私有分组）按需配置。组织 Secrets 必须授权本仓库；同名仓库 Secrets 优先于组织配置。未指定 APP_ID 时按包名或准确应用名 `audio.cpp` 查找或创建。

图标由用户提供的 AUDIO.png 缩放为 512×512 PNG。文件选择器脚本来自 https://developer.lazycat.cloud/lazycat-injects/lzc-file-chooser-inject.js 。上游采用 Apache-2.0 许可证。
