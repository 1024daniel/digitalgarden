---
{"dg-publish":true,"permalink":"/CloudNative/docker/docker build/更改基础镜像用户id/","noteIcon":"3"}
---

#dockerfile #docker
```dockerfile
# 使用基础镜像
FROM base-image:latest

# 切换到 root 用户
USER root

# 修改用户和组的 UID 和 GID
RUN groupmod -g 1001 HwHiAiUser && \
    usermod -u 1001 -g 1001 HwHiAiUser

# 修复整个文件系统中属于旧 UID/GID 的文件权限
RUN find / -path /proc -prune -o -user 1000 -exec chown -h 1001 {} + && \
    find / -path /proc -prune -o -group 1000 -exec chgrp -h 1001 {} +

# 切换回非 root 用户（如果需要）
USER HwHiAiUser

# 继续其他构建步骤

```