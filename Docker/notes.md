## Docker

- [安裝](#安裝)
- [Dockerfile](#Dockerfile)
- [docker-compose](#docker-compose)
- [常用 CLI](#常用CLI)

<br/>

### 安裝

#### 1. 產生 Docker 的 apt 存儲庫

```sh
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

#### 2. 安裝最新版

```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 2-1. 安裝指定版本

- 列出可用版本

```sh
apt-cache madison docker-ce | awk '{ print $3 }'
```

- 設置版本參數

```sh
VERSION_STRING=5:24.0.0-1~ubuntu.22.04~jammy
```

- 安裝

```sh
sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

- 檢查安裝是否正常

```sh
sudo docker run hello-world
```

#### 3. 安裝 docker compose（可選）

```sh
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

- 驗證安裝

```sh
docker compose version
```

<br/>

### Dockerfile

- 一般範例

```dockerfile
FROM alpine:3.19

COPY --from=builder /app /app

WORKDIR /app

CMD ["./server"]
```

<br/>

- 於 Docker 內編譯後執行（Go）

```dockerfile
FROM golang:1.22-alpine as builder

WORKDIR /app

COPY . .
RUN go build -o main main.go

FROM alpine:3.19

COPY --from=builder /app /app

WORKDIR /app

CMD ["./server"]
```

<br/>

- 指令清單

| 指令          | 說明                                                                                                                                                                                                                                  | 範例                                                                                |
| :------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :---------------------------------------------------------------------------------- |
| `ADD`         | 進階版 `COPY` 。不實用。僅在需要自動解壓縮時可以考慮使用。                                                                                                                                                                            | `ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /`                               |
| `ARG`         | 構建階段使用的變數，容器運行時不會存在，作用域僅到下個 `FROM` 前，與 `ENV` 的主要差異在於 `ARG` 限定了作用域，影響的層數較少，所以可以更精細的控制緩存。                                                                              | `ARG USERNAME=user`                                                                 |
| `CMD`         | 容器啟動時要執行的命令。若存在多個只會執行最後被宣告的那道指令。                                                                                                                                                                      | `CMD ["./main"]`                                                                    |
| `COPY`        | 將構建上下文目錄中的源路徑的文件或目錄複製到鏡向內的指定路徑。                                                                                                                                                                        | `COPY . .`                                                                          |
| `ENTRYPOINT`  | 功能同 `CMD`，但當使用 `ENTRYPOINT` 時，`CMD` 的功能將不再是直接執行，而是做為 `ENTRYPOINT` 的參數執行，也可以在 CLI 執行 `docker run <映像名稱> <參數> `。                                                                           | `ENTRYPOINT [ "go", "run", "./main.go" ]`                                           |
| `ENV`         | 定義環境變數，可用於 dockerfile 內部以及容器運行時提供給應用。                                                                                                                                                                        | `ENV VERSION=1.0`                                                                   |
| `EXPOSE`      | 聲明容器打算使用的端口，僅用於聲明。映射還是由 CLI 的 `-p <主機端口>:<容器端口>` 指定。                                                                                                                                               | `EXPOSE <端口1> [<端口2>...]`                                                       |
| `FROM`        | 建立基底映像。                                                                                                                                                                                                                        | `FROM alpine:3.19`, `FROM golang:1.22-alpine as <name>`                             |
| `HEALTHCHECK` | 設置一個固定周期互叫容器內應用的指令用於確認應用是否正常運行。**※範例是每 5 秒檢查一次網頁伺服器響應狀態，若超過 3 秒沒響應就會將容器狀態設置為 `unhealthy`，並退出容器。**                                                           | `HEALTHCHECK --interval=5s --timeout=3s CMD curl -fs http://localhost/ \|\| exit 1` |
| `LABEL`       | 替映像添加元數據。                                                                                                                                                                                                                    | `LABEL doc="https://example.com/doc"`                                               |
| `MAINTAINER`  | 指定映像的維護者，可用 `LABEL` 取代。                                                                                                                                                                                                 | `MAINTAINER MMQ`                                                                    |
| `ONBUILD`     | 指定未來套用此映象為基底時要執行的指令（不影響當前要構建的映象）。                                                                                                                                                                    | `ONBUILD COPY ./package.json /app`                                                  |
| `RUN`         | 映像生成時要執行的指令。                                                                                                                                                                                                              | `RUN [ "npm", "i" ]`                                                                |
| `SHELL`       | 指定構建映像時，執行 `RUN`, `ENTRYPOINT`, `CMD` 要使用的終端。                                                                                                                                                                        | `SHELL ["/bin/sh", "-cex"]`                                                         |
| `STOPSIGNAL`  | 設置當執行 `docker stop` 時， docker 向容器應用發射終止訊號的類型（預設：SIGTERM），讓應用能夠捕獲終止訊號做出對應的行為。訊號類型可以是編號也可以是訊號名稱。也可以使用 `docker kill --signal=SIGTERM foo` 來覆蓋 dockerfil 的設定。 | `STOPSIGNAL SIGTERM`                                                                |
| `USER`        | 指定要切換的用戶名稱（不使用 `root`），這個指令僅做切換，若該用戶組不存在會報錯。                                                                                                                                                     | `USER <用户名>[:<用户组>]`                                                          |
| `VOLUME`      | 替容器生成匿名卷，用於持久化資料。若要映射到宿主目錄（取消匿名），需要在運行 `docker run` 時，加上參數 `-v <宿主目錄>:<容器目錄>`                                                                                                     | `VOLUME <路徑>`, `VOLUME ["<路徑1>", "<路徑2>", ...]` `                             |
| `WORKDIR`     | 指定容器的工作目錄。                                                                                                                                                                                                                  | `WORKDIR /app`                                                                      |

<br/>

### docker-compose

**※基本上 docker-compose.yml 內的設定會直接覆蓋掉 Dockerfile 內重複的設定，僅有 ENV 指令，若有重複的鍵值，會在構建階段使用 ENV 指令，在容器運行階段使用 docker-compose.yml 內的 environment。**

<br/>

- 範例

```yml
version: "3.8"
services:
  app-redis:
    image: redis:latest
    container_name: app-redis
    user: root
    # ports:
    # - "6379:6379"
    restart: unless-stopped

  app-psql:
    image: postgres:alpine3.19
    container_name: app-psql
    user: root
    environment:
      POSTGRES_PASSWORD: "PA$$W0RD"
      POSTGRES_USER: root
    # ports:
    # - "5432:5432"
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./pgdata:/var/lib/postgresql/data
    restart: unless-stopped

  myapp:
    build: .
    container_name: myapp
    depends_on:
      - app-redis
      - app-psql
    ports:
      - "54321:12345"
    restart: unless-stopped
```

<br/>

- 常用字段說明

> version

指定 Compose 文件格式的版本。

<br/>

> services

定義應用程序中的服務，一個服務代表一個容器。
每個服務都擁有自己的配置。

<br/>

> build

指定該服務的映像的構建上下文。
可以是一個路徑或超連結，設置為 `.` 代表使用當前目錄。
如果設置了 `build` 標籤，Compose 將使用當前目錄下的 Dockerfile 構建映像。
若 `services` 的各服務區塊下使用 `build` ，可以再使用 `image` 來指定構建出的映象檔名稱，若不存在 `image` ，則會使用 `<項目名>_<服務名>` 作為映像名稱。

<br/>

```yml
build:
  context: ./path/to/dockerdir # dockerfile 目錄
  dockerfile: Dockerfile # 指定的 dockerfile 名稱，若不存在此字段會使用預設的 "Dockerfile"
  args:
    - VERSION=1.0 # 建構階段的環境變數
image: ${DOCKER_HUB_USER}/myapp:${VERSION} # myregistry.azurecr.io/myapp:latest
```

<br/>

> image

指定從哪個映像啟動容器。
如果設置了 `build` 標籤，則 `image` 標籤的功能變成了構建完成後要使用的映像名稱。

<br/>

```yml
image: postgres:alpine3.19
```

<br/>

> restart

定義容器因人為或各種因素退出後的重啟策略。
可選值有 `no`、`always`、`on-failure` 和 `unless-stopped`。

`no`：不重啟。
`always`：無論退出因素都不停地嘗試重啟。
`on-failure`：僅在因容器異常導致退出後重啟。
`unless-stopped`：除非人為停止，否則嘗試重啟。

<br/>

> ports

設置端口映射，用於指定要暴露哪些容器內的端口映射到宿主端口上。

```yml
ports:
  - "80:80" # 暴露容器 80 端口並映射到宿主的 80 端口
  - "12345:54321" # 暴露容器 12345 端口並映射到宿主的 54321 端口
```

<br/>

> networks

- 在 `services` 外宣告用於定義 docker 網路。
  預設容器接使用 `bridge` 模式並且同一份 docker-compose 檔案內的服務預設會在同一個網路下。

<br/>

```yml
networks:
  my_docker_network:
    name: my_docker_network
    driver: bridge # 指定網路驅動模式
    enable_ipv6: true # 用於啟用 ipv6
    external: true # 指定此網路由外部維護，Compose 不會創建此網路，若不存在會報錯
    ipam:
      driver: default # 指定 IP 管理與分配的驅動類型
      config:
        - subnet: 172.28.0.0/16 # 指定子網網段
          gateway: 172.28.0.1 # 指定該網段所使用的網關位址
```

其中 `dirver` 指定網路驅動模式，可選值：

`bridge`：預設值，創建虛擬網路，並可透過暴露容器端口映射到宿主端口來對外服務。
`overlay`：跨越多個 docker 主機的集群網路。
`host`：直接與宿主使用相同網路。
`none`：禁用所有網路功能。

<br/>

- 在 `services` 各區塊內宣告用於指定服務要使用的 docker 網路。

<br/>

```yml
services:
  myapp:
    networks:
      - my_docker_network
        - ipv4_address: 172.28.0.2 # 可以指定要使用的 docker 網路 ip 位址，但不建議。
      - other_network
```

<br/>

> volumes

- 在 `services` 外宣告用於定義資料卷

```yml
volumes:
  example: # 這是資料卷的引用名稱，用於在服務或其他位置引用該資料卷
    driver_opts: # 設置用於創建該資料卷的驅動程序選項
      name: "my-app-data" # 資料卷的名稱
      type: "nfs" # 使用的驅動程序類型
      o: "addr=10.40.0.199,nolock,soft,rw" # 傳遞給驅動程序的選項
      device: ":/docker/example" # 本地或遠端的宿主目錄路徑，用於建構要被容器掛載的資料卷。
      external: true # 指定此資料卷由外部維護，Compose 不會嘗試創建，若不存在會報錯
      labels: # metadata
        - "com.example.description=Database volume"
        - "com.example.department=IT/Ops"
        - "com.example.label-with-empty-value"
```

<br/>

- 在 `services` 各區塊內宣告用於將宿主上的資料卷或目錄掛載到容器中。

```yml
services:
  myapp:
    image: example/app
    volumes:
      - example:/etc/data # <volume 引用名稱>:<容器目錄>
```

<br/>

> environment

設置環境變量。

<br/>

```yml
environment:
  POSTGRES_PASSWORD: "PA$$W0RD"
  POSTGRES_USER: root
```

<br/>

> depends_on

設置服務之間的依賴關係。
確保被依賴的服務在當前服務之前啟動以滿足程序運行需求。

<br/>

```yml
depends_on:
  - app-redis
  - app-psql
```

<br/>

### 常用 CLI

- Docker 指令:

  > 建構映像檔

  `docker build -t <image_name>:<tag> .`

  `-t`：標記映像檔的名稱和標籤
  `.`：表示使用當前目錄下的 Dockerfile 進行構建
  `--no-cache`：不使用緩存構建映像

    <br/>

  > 啟動容器

  `docker run <image_name>:<tag>`

  `-d`：在背景執行
  `-p <宿主端口:容器端口>`：端口映射
  `--network <network_name>`：指定 docker 網路

    <br/>

  > 停止指定的容器

  `docker stop <container_id/name>`

    <br/>

  > 移除指定的容器

  `docker rm <container_id/name>`

    <br/>

  > 列出映像

  `docker image ls`

  <br/>

  > 列出容器

  `docker ps`

    <br/>

  > 查看指定容器 log

  `docker logs <container_id/name>`

    <br/>
    
  > 移除指定的映像檔

  `docker rmi <image_id/name>:<tag>`

    <br/>

  > 建立一個新的 Docker 網路

  `docker network create <network_name>`

    <br/>

- Docker Compose 指令:

  > （多）容器同步啟動

  根據當前目錄下的 docker-compose.yml 建立並啟動服務

  `docker-compose up`

  <br/>

  `-d`：在背景執行
  `--build`：在啟動前重新建置映像檔

  <br/>

  > （多）容器同步停止

  停止並移除當前正在運行的服務

  `docker-compose down`

  <br/>

  `--rmi`：移除指定映像檔，`all` 刪除全部由該 Compose 創建的所有映像。
  `-v`：移除匹配的匿名數據卷

  <br/>

  > 列出由 Compose 管理的容器

  `docker-compose ps`

  <br/>

  > 查看由 Compose 管理的指定服務的 log

  `docker-compose logs <service_name>`
