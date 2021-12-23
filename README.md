# docker-whale

Run me : `docker run -p80:80 ushamandya/whale-example`

Build me: `docker build -t ushamandya/whale-example .`


学习目标:

1. 以Docker项目为例子来配置GitHub Actions
2. 设置GitHub Actions workflow
3. 优化您的工作流以减少拉取请求的数量和总生成时间
4. 仅将特定版本推送到Docker Hub

### 创建 token

1. Add your Docker ID as a secret to GitHub. Navigate to your GitHub repository and click Settings > Secrets > New secret.
2. Create a new secret with the name DOCKER_HUB_USERNAME and your Docker ID as value.
3. Create a new Personal Access Token (PAT). To create a new token, go to Docker Hub Settings and then click New Access Token.
4. Let’s call this token simplewhaleci.
5. Now, add this Personal Access Token (PAT) as a second secret into the GitHub secrets UI with the name DOCKER_HUB_ACCESS_TOKEN.

### 设置 github 工作流

1. 操作使我们能够使用存储在 GitHub 存储库中的机密登录到 Docker Hub。
2. 构建和推送操作。


(1)命名工作流：

```
name: CI to Docker Hub
```

(2)设置何时运行此工作流

项目主分支的每次推送执行此操作
```
on:
  push:
    branches: [ main ]
```

(3)指定我们实际想要在我们的操作中发生的事情

我们将添加我们的构建版本并选择它在最新的可用 Ubuntu 实例上运行：

```
jobs:

  build:
    runs-on: ubuntu-latest
```

(4)添加所需的步骤。

第一, 切换分支

第二, 使用我们的 PAT 和用户名登录到 Docker Hub

第三, Builder，该动作通过一个简单的 Buildx 动作在幕后使用 BuildKit

```
    steps:

      - name: Check Out Repo 
        uses: actions/checkout@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

      - name: Set up Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v1

      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          context: ./
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKER_HUB_USERNAME }}/simplewhale:latest

      - name: Image digest
        run: echo ${{ steps.docker_build.outputs.digest }}
```
现在，让工作流第一次运行，然后调整 Dockerfile 以确保 CI 正在运行并推送新的图像更改：


原文地址: https://docs.docker.com/language/nodejs/configure-ci-cd/
