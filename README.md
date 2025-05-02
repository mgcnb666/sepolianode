sudo apt update


sudo apt install apt-transport-https ca-certificates curl software-properties-common


curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg


sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"


sudo apt update


sudo apt install docker-ce docker-ce-cli containerd.io




sudo usermod -aG docker ${USER}


sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose



sudo chmod +x /usr/local/bin/docker-compose


# 链接到 /usr/bin



sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose





mkdir sepolia-node


cd sepolia-node



mkdir execution consensus shared-data # 创建数据和共享文件目录

# 生成 JWT 秘密文件

openssl rand -hex 32 > ./shared-data/jwtsecret

# 设置文件权限（可选但推荐）

chmod 600 ./shared-data/jwtsecret


docker-compose up -d


docker-compose logs -f

