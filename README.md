# Sepolia Node Setup with Docker Compose

本指南介绍如何使用 Docker Compose 在 Ubuntu 上设置一个支持 EIP-4884 的 Sepolia RPC 节点和 Beacon API。

## 先决条件

- Ubuntu 操作系统
- 已安装 Docker 和 Docker Compose

## 安装 Docker 和 Docker Compose

1. 更新包索引并安装 Docker：
   ```bash
   sudo apt update
   sudo apt install docker.io
   ```

2. 启动 Docker 并设置为开机启动：
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

3. 安装 Docker Compose：
   ```bash
   sudo apt install docker-compose
   ```

## 设置项目目录

1. 创建项目目录并进入：
   ```bash
   mkdir sepolia-node
   cd sepolia-node
   ```

2. 创建数据和共享文件目录：
   ```bash
   mkdir execution consensus shared-data
   ```

3. 生成 JWT 秘密文件：
   ```bash
   openssl rand -hex 32 > ./shared-data/jwtsecret
   ```

4. 设置文件权限（可选但推荐）：
   ```bash
   chmod 600 ./shared-data/jwtsecret
   ```

## Docker Compose 配置

在项目目录中创建 `docker-compose.yml` 文件，内容如下：

```yaml
services:
  geth:
    image: ethereum/client-go:stable
    container_name: sepolia-geth
    command:
      - --sepolia
      - --syncmode=snap
      - --datadir=/data
      - --http
      - --http.addr=0.0.0.0
      - --http.port=8545
      - --http.api=eth,net,web3,debug
      - --authrpc.addr=0.0.0.0
      - --authrpc.port=8551
      - --authrpc.vhosts=*
      - --authrpc.jwtsecret=/shared/jwtsecret
      - --metrics.expensive
    volumes:
      - ./execution:/data
      - ./shared-data/jwtsecret:/shared/jwtsecret:ro
    ports:
      - "8545:8545"
      - "8551:8551"
      - "30303:30303"
      - "30303:30303/udp"
    restart: unless-stopped
    networks:
      - sepolia-net

  prysm:
    image: gcr.io/prysmaticlabs/prysm/beacon-chain:latest
    container_name: sepolia-prysm
    command:
      - --sepolia
      - --datadir=/data
      - --execution-endpoint=http://geth:8551
      - --jwt-secret=/shared/jwtsecret
      - --accept-terms-of-use
      - --http-host=0.0.0.0
      - --http-port=3500
      - --http-modules=prysm,eth
      - --checkpoint-sync-url=https://checkpoint-sync.sepolia.ethpandaops.io
    volumes:
      - ./consensus:/data
      - ./shared-data/jwtsecret:/shared/jwtsecret:ro
    ports:
      - "3500:3500"
      - "13000:13000"
      - "13000:13000/udp"
    restart: unless-stopped
    depends_on:
      - geth
    networks:
      - sepolia-net

networks:
  sepolia-net:
    driver: bridge
```

## 启动服务

在项目目录中运行以下命令启动服务：

```bash
docker-compose up -d
```

## 查看日志

使用以下命令查看服务日志：

```bash
docker-compose logs -f
```

## 注意事项

- 确保所有服务在同一网络中运行。
- 确保 JWT 秘密文件路径正确，并且两个服务都能访问。
- 检查防火墙设置，确保允许必要的端口。

通过这些步骤，你可以成功在 Ubuntu 上使用 Docker Compose 运行一个支持 EIP-4884 的 Sepolia RPC 节点和 Beacon API。
