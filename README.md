# sepolianode
Build your own sepolia node, including EL and CL
Geth + Lighthouse

# 安装依赖
sudo apt update && sudo apt install -y curl wget unzip git build-essential

sudo apt update

sudo apt install cmake

sudo apt install -y clang libclang-dev llvm-dev


# 安装 Geth
sudo add-apt-repository -y ppa:ethereum/ethereum

sudo apt update

sudo apt install -y geth

# 初始化 Geth Sepolia 网络
cat <<EOF > ~/.geth_sepolia.sh

#!/bin/bash

geth \\

  --sepolia \\
  
  --syncmode=snap \\
  
  --cache=12288 \\
  
  --http \\
  
  --http.api "eth,net,web3,txpool" \\
  
  --http.addr "0.0.0.0" \\
  
  --http.port 8545 \\
  
  --authrpc.addr localhost \\
  
  --authrpc.port 8551 \\
  
  --authrpc.vhosts=* \\
  
  --authrpc.jwtsecret /root/jwt.hex \\
  
  --datadir /root/.ethereum \\
  
  --port 30303
  
EOF


chmod +x ~/.geth_sepolia.sh

# 生成 JWT secret（用于共识层通信）
openssl rand -hex 32 > /root/jwt.hex

# 后台启动 Geth
tmux new-session -d -s geth "~/.geth_sepolia.sh"

# 安装 Rust（用于编译 Lighthouse）
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y

source "$HOME/.cargo/env"

# 安装 Lighthouse
git clone https://github.com/sigp/lighthouse.git
cd lighthouse
make
sudo cp target/release/lighthouse /usr/local/bin/
cd ..
mkdir -p ~/.lighthouse

# 创建 Lighthouse 启动脚本
cat <<EOF > ~/.lighthouse_sepolia.sh

#!/bin/bash

lighthouse bn \\

  --network sepolia \\
  
  --datadir /root/.lighthouse \\
  
  --execution-endpoint http://127.0.0.1:8551 \\
  
  --jwt-secrets /root/jwt.hex \\
  
  --checkpoint-sync-url https://sepolia.checkpoint-sync.ethpandaops.io \\
  
  --port 9000 \\
  
  --http \\
  
  --http-address 0.0.0.0 \\
  
  --http-port 5052
  
EOF

chmod +x ~/.lighthouse_sepolia.sh

# 后台启动 Lighthouse
tmux new-session -d -s lighthouse "~/.lighthouse_sepolia.sh"

使用 tmux attach -t geth / lighthouse 查看日志



