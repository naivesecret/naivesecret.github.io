---
title: Win10 配置 Clion + WSL2 环境
date: 2022-01-18 12:00:58
tags: 新知
categories:
  - [Linux]
  - [WSL]
---
## 安装 WSL2
暂略...

## 在 WSL2 中配置并运行 ssh-server
``` bash
wget https://raw.githubusercontent.com/JetBrains/clion-wsl/master/ubuntu_setup_env.sh
bash ubuntu_setup_env.sh
```
*这个脚本需要改动， SSHD_LISTEN_ADDRESS 改为`0.0.0.0`， SSHD_PORT 改为自己的端口号*
``` bash
#!/bin/bash
set -e

SSHD_LISTEN_ADDRESS=0.0.0.0

SSHD_PORT=2222
SSHD_FILE=/etc/ssh/sshd_config
SUDOERS_FILE=/etc/sudoers
  
# 0. update package lists
sudo apt-get update

# 0.1. reinstall sshd (workaround for initial version of WSL)
sudo apt remove -y --purge openssh-server
sudo apt install -y openssh-server

# 0.2. install basic dependencies
sudo apt install -y cmake ninja-build gcc clang gdb valgrind build-essential

# 1.1. configure sshd
sudo cp $SSHD_FILE ${SSHD_FILE}.`date '+%Y-%m-%d_%H-%M-%S'`.back
sudo sed -i '/^Port/ d' $SSHD_FILE
sudo sed -i '/^ListenAddress/ d' $SSHD_FILE
sudo sed -i '/^UsePrivilegeSeparation/ d' $SSHD_FILE
sudo sed -i '/^PasswordAuthentication/ d' $SSHD_FILE
echo "# configured by CLion"      | sudo tee -a $SSHD_FILE
echo "ListenAddress ${SSHD_LISTEN_ADDRESS}"	| sudo tee -a $SSHD_FILE
echo "Port ${SSHD_PORT}"          | sudo tee -a $SSHD_FILE
echo "UsePrivilegeSeparation no"  | sudo tee -a $SSHD_FILE
echo "PasswordAuthentication yes" | sudo tee -a $SSHD_FILE
# 1.2. apply new settings
sudo service ssh --full-restart
  
# 2. autostart: run sshd 
sed -i '/^sudo service ssh --full-restart/ d' ~/.bashrc
echo "%sudo ALL=(ALL) NOPASSWD: /usr/sbin/service ssh --full-restart" | sudo tee -a $SUDOERS_FILE
cat << 'EOF' >> ~/.bashrc
sshd_status=$(service ssh status)
if [[ $sshd_status = *"is not running"* ]]; then
  sudo service ssh --full-restart
fi
EOF
  

# summary: SSHD config info
echo 
echo "SSH server parameters ($SSHD_FILE):"
echo "ListenAddress ${SSHD_LISTEN_ADDRESS}"
echo "Port ${SSHD_PORT}"
echo "UsePrivilegeSeparation no"
echo "PasswordAuthentication yes"
```

使用如下命令增加脚本运行权限并运行：
``` bash
sudo chmod +x ubuntu_setup_env.sh
bash ubuntu_setup_env.sh
```

然后在 WSL 环境下运行（端口号根据脚本内的 `SSHD_PORT` 进行设置）：
``` bash
ssh username@localhost -p2222
```
## 在 Clion 内配置 `Toolchains`