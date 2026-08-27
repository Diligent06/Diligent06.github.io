# Docker 镜像创建与上传完整流程

本文档记录如何将当前机器的根文件系统制作成 Docker 镜像归档，并上传至私有 Registry。文档同时给出本次受限 Kubernetes 环境下实际验证成功的方案，以及普通 Docker 主机上的标准方案。

## 1. 最终产物

本次已经成功生成并上传以下镜像：

```text
registry.hd-02.alayanew.com:8443/
alayanew-6ace96f5-0166-4a81-b99b-218fb6b85db8/
delta_d1:v1
```

完整镜像地址：

```text
registry.hd-02.alayanew.com:8443/alayanew-6ace96f5-0166-4a81-b99b-218fb6b85db8/delta_d1:v1
```

远端 Registry manifest digest：

```text
sha256:213d0e8892fa2927ef420a79056c7c37f81c7e7b714c3664efc6ad30cfc297e1
```

镜像平台：

```text
OS: linux
Architecture: amd64
```

本地归档文件：

```text
/root/nas/rongpeng/images/Delta_D1_v1.tar
```

归档大小：

```text
57,833,676,800 bytes，约 53.86 GiB
```

归档 SHA-256：

```text
620d7f9b373ef91f147f26eca476dc8abd2131afe2635fd9608b3a24f1029888
```

## 2. 环境限制

当前机器实际上运行在一个受限 Kubernetes 容器中，缺少以下能力：

```text
CAP_SYS_ADMIN
unshare namespace 权限
```

因此会出现以下限制：

- Docker daemon 可以启动并响应镜像 API。
- `docker import` 或 `docker load` 在提交 layer 时可能报错：

  ```text
  unshare: operation not permitted
  ```

- 不适合直接使用 `docker run` 启动容器。
- 可以直接构造标准 Docker archive，再使用 Skopeo 上传。

因此，本环境中推荐：

```text
根文件系统 → Docker archive → Skopeo → Registry
```

而不是：

```text
根文件系统 → docker import/load → docker tag → docker push
```

## 3. 检查磁盘占用

检查根分区：

```bash
df -h /
```

按目录检查实际占用：

```bash
du -x -h --max-depth=1 / 2>/dev/null | sort -hr | head -20
```

同时比较逻辑文件大小：

```bash
du -x -h --apparent-size --max-depth=1 / 2>/dev/null \
  | sort -hr \
  | head -20
```

本次主要空间来源为：

| 目录 | 大约大小 | 主要内容 |
|---|---:|---|
| `/workspace` | 28.3 GiB | Anaconda 和训练环境 |
| `/opt` | 9.9 GiB | 基础 Conda、NVIDIA 工具 |
| `/usr` | 8.9 GiB | 系统库和本地软件 |
| `/root` | 5.9 GiB | VS Code Server、用户配置等 |

Docker archive 的大小可能高于 `df` 显示的根分区已用空间。这是因为当前根文件系统使用 overlay：`df` 主要反映可写层占用，而导出时会将基础镜像只读层和当前修改层合并展开。

## 4. 清理不需要的缓存

### 4.1 清理 Conda package cache

先检查是否存在环境文件软链接到 package cache：

```bash
find /workspace/anaconda3/envs \
  -xdev \
  -type l \
  -lname '/workspace/anaconda3/pkgs/*' \
  -print
```

如果没有输出，可以清理 package cache：

```bash
/workspace/anaconda3/bin/conda clean --force-pkgs-dirs --yes
```

该操作：

- 不会删除已有 Conda 环境中的实际文件。
- 不会删除训练代码或 checkpoint。
- 会导致以后安装或回滚软件包时重新下载。

清理后验证环境：

```bash
/workspace/anaconda3/envs/sam_3d_body/bin/python \
  -c 'import sys, torch; print(sys.executable, torch.__version__)'

/workspace/anaconda3/envs/sam3d_mhr_pytorch/bin/python \
  -c 'import sys, torch; print(sys.executable, torch.__version__)'
```

### 4.2 清理 VS Code Server

不要在 VS Code Remote 或 Codex 正在运行时直接删除整个目录：

```text
/root/.vscode-server
```

直接删除可能导致当前连接、终端、扩展或 Codex 会话中断。

先检查正在运行的版本：

```bash
ps -eo pid,etime,stat,args \
  | grep -E '[v]scode-server|[c]ode-server|[r]emote-cli'
```

检查已安装的 Server 版本：

```bash
du -sh /root/.vscode-server/cli/servers/* 2>/dev/null | sort -hr
```

只删除没有任何进程使用的旧版本。扩展安装包缓存通常也可以删除：

```text
/root/.vscode-server/data/CachedExtensionVSIXs
```

不要删除正在使用的 Server 版本、已安装扩展或当前 Codex 进程。

本次清理结果：

```text
根分区使用率：74% → 65%
根分区可用空间：13 GiB → 18 GiB
.vscode-server：5.8 GiB → 2.6 GiB
```

> 注意：本次 `v1` 镜像是在清理之前生成的，因此已经上传的 `v1` 仍然包含原来的缓存。需要重新生成 `v2` 才能获得清理后的较小镜像。

## 5. 安装 Docker

检查安装结果：

```bash
docker --version
docker compose version
docker buildx version
```

本次安装版本：

```text
Docker Engine 29.7.2
Docker Compose v5.5.0
Docker Buildx v0.36.1
```

## 6. 将 Docker 数据目录放到 NAS

检查当前 Docker daemon 的数据目录：

```bash
docker info --format '{{.DockerRootDir}}'
```

本次设置为：

```text
/root/nas/rongpeng/images/docker-data
```

因此，理论上执行 `docker load` 时，解包后的镜像会保存到 NAS，而不是默认的 `/var/lib/docker`。

本次受限环境使用以下 daemon 参数：

```bash
dockerd \
  --host=unix:///tmp/docker-delta.sock \
  --pidfile=/tmp/dockerd-delta.pid \
  --data-root=/root/nas/rongpeng/images/docker-data \
  --exec-root=/tmp/docker-delta-exec \
  --storage-driver=vfs \
  --iptables=false \
  --ip6tables=false \
  --bridge=none \
  --ip-forward=false \
  --ip-masq=false
```

这些参数主要用于绕过当前 Kubernetes 容器缺少网络管理能力的问题。普通 Docker 主机不建议照搬 `vfs` 和禁用网络的参数。

## 7. 创建 Docker archive

本次使用的构建脚本：

```text
/root/nas/rongpeng/images/build_delta_d1_archive.sh
```

运行：

```bash
bash /root/nas/rongpeng/images/build_delta_d1_archive.sh
```

脚本执行以下操作：

1. 将根文件系统导出为一个未压缩 rootfs layer。
2. 排除 NAS、数据集、代码目录和虚拟文件系统。
3. 计算 rootfs layer SHA-256。
4. 生成 Docker image config JSON。
5. 生成 `manifest.json` 和 `repositories`。
6. 封装为标准 Docker archive。
7. 生成归档 SHA-256 文件。
8. 验证 archive 成员和元数据。

主要排除目录：

```text
/proc
/sys
/dev
/run
/tmp
/root/nas
/root/Codes
/root/Datasets
/root/Checkpoints
/var/lib/docker
/var/lib/containerd
/root/.vscode-server/data/logs
/root/.codex/ipc
/root/.codex/tmp
```

输出文件：

```text
/root/nas/rongpeng/images/Delta_D1_v1.tar
/root/nas/rongpeng/images/Delta_D1_v1.tar.sha256
/root/nas/rongpeng/images/Delta_D1_v1.inspect.json
```

### 7.1 推荐在 tmux 中运行

```bash
tmux new-session -s delta_d1_image

bash /root/nas/rongpeng/images/build_delta_d1_archive.sh
```

脱离 tmux：

```text
Ctrl+B，然后按 D
```

重新进入：

```bash
tmux attach -t delta_d1_image
```

## 8. 校验本地归档

执行：

```bash
cd /root/nas/rongpeng/images
sha256sum -c Delta_D1_v1.tar.sha256
```

期望输出：

```text
/root/nas/rongpeng/images/Delta_D1_v1.tar: OK
```

检查 Docker archive 结构：

```bash
tar -tf /root/nas/rongpeng/images/Delta_D1_v1.tar | head
```

使用 Skopeo 验证镜像元数据：

```bash
skopeo inspect \
  docker-archive:/root/nas/rongpeng/images/Delta_D1_v1.tar:delta_d1:v1
```

## 9. 安装 Skopeo

在 Ubuntu 上安装：

```bash
apt-get update
apt-get install -y skopeo
```

检查版本：

```bash
skopeo --version
```

Skopeo 可以直接从 Docker archive 向 Registry 上传，不需要先执行 `docker load`，因此不会在 DockerRootDir 中再解包一份约 54 GiB 的镜像。

## 10. 登录 Registry

使用交互式密码输入，避免密码写入命令行或 shell history：

```bash
skopeo login \
  --username delta-surongpeng \
  registry.hd-02.alayanew.com:8443
```

出现以下提示后输入密码：

```text
Password:
```

成功后显示：

```text
Login Succeeded!
```

不要使用以下形式，因为密码可能出现在 shell history 或进程参数中：

```bash
# 不推荐
skopeo login --password 'PASSWORD' ...
```

## 11. 直接从归档推送到 Registry

执行：

```bash
skopeo copy \
  docker-archive:/root/nas/rongpeng/images/Delta_D1_v1.tar:delta_d1:v1 \
  docker://registry.hd-02.alayanew.com:8443/alayanew-6ace96f5-0166-4a81-b99b-218fb6b85db8/delta_d1:v1
```

成功时输出类似：

```text
Getting image source signatures
Copying blob ... done
Copying config ... done
Writing manifest to image destination
Storing signatures
```

### 11.1 在 tmux 中登录并推送

```bash
tmux new-session -s registry_push

skopeo login \
  --username delta-surongpeng \
  registry.hd-02.alayanew.com:8443

skopeo copy \
  docker-archive:/root/nas/rongpeng/images/Delta_D1_v1.tar:delta_d1:v1 \
  docker://registry.hd-02.alayanew.com:8443/alayanew-6ace96f5-0166-4a81-b99b-218fb6b85db8/delta_d1:v1
```

## 12. 验证远端镜像

```bash
skopeo inspect \
  docker://registry.hd-02.alayanew.com:8443/alayanew-6ace96f5-0166-4a81-b99b-218fb6b85db8/delta_d1:v1
```

仅输出主要字段：

```bash
skopeo inspect \
  --format 'Name={{.Name}} Digest={{.Digest}} Architecture={{.Architecture}} OS={{.Os}} Created={{.Created}}' \
  docker://registry.hd-02.alayanew.com:8443/alayanew-6ace96f5-0166-4a81-b99b-218fb6b85db8/delta_d1:v1
```

本次验证结果：

```text
Digest=sha256:213d0e8892fa2927ef420a79056c7c37f81c7e7b714c3664efc6ad30cfc297e1
Architecture=amd64
OS=linux
```

## 13. 在普通 Docker 主机上加载归档

先确认 DockerRootDir：

```bash
docker info --format '{{.DockerRootDir}}'
```

加载镜像：

```bash
docker load -i /root/nas/rongpeng/images/Delta_D1_v1.tar
```

查看镜像：

```bash
docker image inspect delta_d1:v1
docker image ls delta_d1:v1
```

运行：

```bash
docker run --rm -it delta_d1:v1
```

当前 Kubernetes 容器可能因缺少 `unshare/CAP_SYS_ADMIN` 而无法完成 `docker load` 或 `docker run`。归档应当在具备完整 Docker 权限的主机上加载。

## 14. 普通 Docker 主机的标准构建和上传流程

如果项目具备 Dockerfile，推荐使用标准镜像构建流程，而不是快照整个根文件系统。

### 14.1 构建镜像

```bash
docker build -t delta_d1:v1 .
```

### 14.2 登录 Registry

```bash
docker login \
  registry.hd-02.alayanew.com:8443 \
  --username delta-surongpeng
```

### 14.3 添加远端标签

```bash
docker tag delta_d1:v1 \
  registry.hd-02.alayanew.com:8443/alayanew-6ace96f5-0166-4a81-b99b-218fb6b85db8/delta_d1:v1
```

### 14.4 推送镜像

```bash
docker push \
  registry.hd-02.alayanew.com:8443/alayanew-6ace96f5-0166-4a81-b99b-218fb6b85db8/delta_d1:v1
```

### 14.5 验证 manifest

```bash
docker manifest inspect \
  registry.hd-02.alayanew.com:8443/alayanew-6ace96f5-0166-4a81-b99b-218fb6b85db8/delta_d1:v1
```

## 15. 常见问题

### 15.1 为什么镜像约 54 GiB？

因为本次打包的是完整合并根文件系统，包含：

- 两个完整 Conda/PyTorch 环境；
- CUDA、NVIDIA 和系统库；
- `/opt/conda`；
- VS Code Server；
- 基础镜像只读层展开后的内容。

Docker archive 中的 rootfs layer 默认未压缩，因此归档大小接近所有文件的逻辑大小总和。

### 15.2 为什么 `df` 只有约 36 GiB，但镜像约 54 GiB？

`df` 统计 overlay 文件系统实际占用块；导出镜像时会将基础只读层和当前可写层合并展开，所以两者不能直接比较。

### 15.3 `docker load` 会把镜像加载到哪里？

由以下命令的结果决定：

```bash
docker info --format '{{.DockerRootDir}}'
```

当前机器为：

```text
/root/nas/rongpeng/images/docker-data
```

### 15.4 如何避免 `docker load` 占用额外空间？

直接使用：

```bash
skopeo copy docker-archive:... docker://...
```

这样不需要将归档解包到 DockerRootDir。

### 15.5 如何进一步缩小镜像？

建议：

1. 在构建镜像之前清理 Conda package cache。
2. 排除未使用的 VS Code Server。
3. 只保留实际需要的 Conda 环境。
4. 使用 Dockerfile 构建，而不是快照整个运行环境。
5. 使用多阶段构建，仅复制运行时依赖。
6. 将模型、数据集和 checkpoint 放在外部 NAS，通过 volume 挂载。
7. 将不需要的编译器、开发头文件和调试工具从运行时镜像中移除。

### 15.6 为什么不应直接把密码写进命令？

密码可能出现在：

- shell history；
- 进程参数；
- tmux capture 输出；
- 自动化日志。

应当使用交互式密码输入，或使用安全的凭据文件/凭据管理服务。

## 16. 推荐的后续版本流程

已经上传的 `v1` 是缓存清理前生成的版本。若要制作更小的版本，建议：

1. 保留云端 `v1` 作为可回滚版本。
2. 在清理后的系统上重新构建归档。
3. 使用新标签，例如 `delta_d1:v2`。
4. 校验本地归档 SHA-256。
5. 使用 Skopeo 上传 `v2`。
6. 使用 `skopeo inspect` 验证远端 digest。
7. 完成验证后再决定是否删除 NAS 上的旧归档。

建议不要覆盖已经推送的 `v1` 标签，以便后续复现和回滚。
