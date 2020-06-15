### 构建镜像

\+ 进入 my-gin-blog 根目录

\````

docker build -t my-gin-blog-docker .

\````



\### 验证镜像

\````

[root@localhost my-gin-blog]# docker images

REPOSITORY           TAG                 IMAGE ID            CREATED                  SIZE

golang               latest              272e3f68338f        Less than a second ago   803MB

my-gin-blog-docker   latest              80a0a9a255ac        16 seconds ago           1.11GB

\````



\### 创建并运行一个容器

\````

[root@localhost my-gin-blog]# docker run -p 8080:8080 my-gin-blog-docker

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.

 \- using env:	export GIN_MODE=release

 \- using code:	gin.SetMode(gin.ReleaseMode)



[GIN-debug] GET    /auth                     --> github.com/stone955/my-gin-blog/router/api.GetAuth (3 handlers)

[GIN-debug] GET    /swagger/*any             --> github.com/swaggo/gin-swagger.CustomWrapHandler.func1 (3 handlers)

[GIN-debug] GET    /api/v1/tags              --> github.com/stone955/my-gin-blog/router/api/v1.GetTags (4 handlers)

[GIN-debug] GET    /api/v1/tags/:id          --> github.com/stone955/my-gin-blog/router/api/v1.GetTag (4 handlers)

[GIN-debug] POST   /api/v1/tags              --> github.com/stone955/my-gin-blog/router/api/v1.AddTag (4 handlers)

[GIN-debug] PUT    /api/v1/tags/:id          --> github.com/stone955/my-gin-blog/router/api/v1.EditTag (4 handlers)

[GIN-debug] DELETE /api/v1/tags/:id          --> github.com/stone955/my-gin-blog/router/api/v1.DeleteTag (4 handlers)

[GIN-debug] GET    /api/v1/articles          --> github.com/stone955/my-gin-blog/router/api/v1.GetArticles (4 handlers)

[GIN-debug] GET    /api/v1/articles/:id      --> github.com/stone955/my-gin-blog/router/api/v1.GetArticle (4 handlers)

[GIN-debug] POST   /api/v1/articles          --> github.com/stone955/my-gin-blog/router/api/v1.AddArticle (4 handlers)

[GIN-debug] PUT    /api/v1/articles/:id      --> github.com/stone955/my-gin-blog/router/api/v1.EditArticle (4 handlers)

[GIN-debug] DELETE /api/v1/articles/:id      --> github.com/stone955/my-gin-blog/router/api/v1.DeleteArticle (4 handlers)

2020/01/05 09:48:30 Actual pid is 1

\````



\### 验证容器实例

\````

[root@localhost ~]# docker ps

CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS                    NAMES

5b25ed2ed214        my-gin-blog-docker   "./my-gin-blog"     15 minutes ago      Up 15 minutes       0.0.0.0:8080->8080/tcp   romantic_sutherland

\````



\### 删除镜像

\````

\# 查看 container

docker ps

\# 查看 关联的容器

docker ps -a

\# 停止运行中的 container

docker stop 5b25ed2ed214

\# 删除 container

docker rm 5b25ed2ed214

\# 删除 image

docker rmi 80a0a9a255ac

\````



\### 构建 scratch 镜像

Scratch镜像，简洁、小巧，基本是个空镜像



\#### 修改 Dockerfile

\````

FROM scratch



WORKDIR /app/my-gin-blog

COPY . /app/my-gin-blog



EXPOSE 8080

CMD ["./my-gin-blog"]

\````



\#### 编译可执行文件

\````

CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o my-gin-blog .

\````



\#### 构建镜像

\````

docker build -t my-gin-blog-docker-scratch .

\````



\#### 运行镜像