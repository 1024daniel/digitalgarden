---
{"dg-publish":true,"permalink":"/CloudNative/docker/dockerfile/","noteIcon":"3"}
---

#docker #dockerfile


### `BUILD`指令

构建中需要联网安装包的话需要设置代理，代理可以通过参数传入:
```sh
docker build \ 
--build-arg http_proxy=$http_proxy
--build-arg https_proxy=$http_proxy
...

```


在 Dockerfile 中，`ENV` 和 `ARG` 是两种用于定义变量的指令，它们在功能和用途上有一些区别。

### `ENV` 指令


> [!NOTE] 注意
> ENV指令不能直接用~或者$HOME，Dockerfile 的 ENV 只将字符串设置为环境变量，不会执行 Shell 解析

- **功能**:
  - `ENV` 指令用于设置环境变量，这些变量将在容器运行时可用，并且可以被容器内的应用程序或命令使用。
  
- **语法**:
  ```dockerfile
  ENV <key> <value>
  ENV <key1>=<value1> <key2>=<value2> ...
  ```

- **示例**:
  ```dockerfile
  ENV APP_ENV production
  ENV DB_HOST=db.example.com DB_PORT=5432
  ```

- **特点**:
  - `ENV` 设置的环境变量在容器启动时会被加载，并且可以在 Dockerfile 中的后续指令中使用。
  - 运行时可以通过 `-e` 参数或者 `docker run` 命令的 `--env` 选项来覆盖这些环境变量的值。

### `ARG` 指令

- **功能**:
  - `ARG` 指令用于定义构建参数，这些参数在构建镜像的过程中传递给 Docker 构建引擎，但在容器运行时不可用。

- **语法**:
  ```dockerfile
  ARG <name>[=<default value>]
  ```

- **示例**:
  ```dockerfile
  ARG BUILD_VERSION=latest
  ```

- **特点**:
  - `ARG` 用于在构建时传递参数，比如可以在构建过程中指定版本号或者其他需要在构建时确定的值。
  - `ARG` 定义的参数在构建过程中可以被 Dockerfile 内的指令使用，但在容器运行时不会存在。
  - 可以通过 `docker build` 命令的 `--b
  - uild-arg` 选项来传递构建参数的值，例如 `docker build --build-arg BUILD_VERSION=1.0 .`

区别总结

- `ENV` 用于设置在容器运行时可用的环境变量，可以影响容器内部的应用程序行为。
- `ARG` 用于在构建过程中传递参数，但这些参数在容器运行时不可用，主要用于构建镜像时的配置和条件。

总的来说，`ENV` 是用于定义容器运行时的环境变量，而 `ARG` 是用于在构建时传递参数给 Docker 构建引擎。



### `RUN`指令


> [!NOTE] 注意
> RUN默认是使用/bin/sh，没法直接使用source，需要替换成`.`(dot)
