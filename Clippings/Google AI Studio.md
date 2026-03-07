---
title: "Google AI Studio"
source: "https://aistudio.google.com/prompts/1ebL1EApFKaiNA77IcaFZUb42A-upvDG2"
author:
  - "[[AI Studio]]"
published:
created: 2026-03-07
description: "The fastest path from prompt to production with Gemini"
tags:
  - "clippings"
---
这是一个完整的**一键全自动安装与配置脚本**。

这个脚本会自动执行以下操作：
1.  **清理旧版本**：自动停止并清理可能存在的旧 Clash 进程。
2.  **安装核心**：下载 Clash 二进制文件并移动到系统标准目录 `/usr/local/bin`。
3.  **配置环境**：创建配置文件目录，下载您的订阅配置，并自动下载 GeoIP 数据库（防止启动报错）。
4.  **设置开机自启**：配置 Systemd 服务，确保服务器重启后 Clash 自动运行。
5.  **注入全局命令**：创建系统级的 `tizi` 和 `untizi` 命令，支持**持久化状态保存**（修改 `/etc/environment`），确保所有终端、后台任务都受控。

---

### 使用方法

1.  在服务器上创建一个文件，例如 `install_clash.sh`：
    ```bash
    nano install_clash.sh
    ```
2.  **完整复制**下面的代码内容，粘贴进去。
3.  保存退出 (`Ctrl+O` -> `Enter` -> `Ctrl+X`)。
4.  赋予脚本执行权限并运行：
    ```bash
    chmod +x install_clash.sh
    sudo ./install_clash.sh
    ```

---

### 脚本内容 (Copy This)

```bash
#!/bin/bash

# ==============================================================================
# Linux Clash 一键安装配置脚本 (持久化版)
# 功能：自动安装、配置 Systemd 服务、设置系统级全局代理开关 (tizi/untizi)
# ==============================================================================

# 定义下载链接 (根据您提供的链接)
CLASH_URL="https://3psjw.big-files.make-w0rld-static.club:8000/file/ikuuu-static-release/clash-linux/clash-linux-1.0.1/clash-linux-amd64.gz"
CONFIG_URL="https://ug0qi.no-mad-world.club/link/aPRIQmhP41q3My0D?clash=3"
# 使用公共镜像加速下载 GeoIP 数据库，避免启动时下载失败
MMDB_URL="https://cdn.jsdelivr.net/gh/Dreamacro/maxmind-geoip@master/Country.mmdb"

# 颜色定义
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
NC='\033[0m' # No Color

echo -e "${YELLOW}[*] 正在检查 root 权限...${NC}"
if [[ $EUID -ne 0 ]]; then
   echo -e "${RED}[!] 请使用 root 权限运行此脚本 (sudo ./install_clash.sh)${NC}" 
   exit 1
fi

echo -e "${YELLOW}[*] 步骤 1/5: 清理旧版本与进程...${NC}"
systemctl stop clash 2>/dev/null
systemctl disable clash 2>/dev/null
rm -f /usr/local/bin/clash
rm -rf /etc/clash
rm -f /etc/systemd/system/clash.service
# 清理旧的 profile 配置
rm -f /etc/profile.d/clash_control.sh
sed -i '/proxy/d; /tizi/d; /untizi/d' ~/.bashrc

echo -e "${YELLOW}[*] 步骤 2/5: 下载并安装 Clash 核心...${NC}"
mkdir -p /tmp/clash_install
cd /tmp/clash_install

echo "正在下载 Clash 二进制文件..."
wget -q --show-progress -O clash.gz "$CLASH_URL"
if [ $? -ne 0 ]; then echo -e "${RED}[!] Clash 下载失败，请检查网络。${NC}"; exit 1; fi

gzip -d clash.gz
chmod +x clash
mv clash /usr/local/bin/
echo -e "${GREEN}[+] Clash 安装成功: /usr/local/bin/clash${NC}"

echo -e "${YELLOW}[*] 步骤 3/5: 下载配置文件与依赖库...${NC}"
mkdir -p /etc/clash

echo "正在下载订阅配置文件..."
wget -q --show-progress -O /etc/clash/config.yaml "$CONFIG_URL"
if [ $? -ne 0 ]; then echo -e "${RED}[!] 配置文件下载失败。${NC}"; exit 1; fi

echo "正在下载 GeoIP 数据库 (Country.mmdb)..."
wget -q --show-progress -O /etc/clash/Country.mmdb "$MMDB_URL"

echo -e "${YELLOW}[*] 步骤 4/5: 配置 Systemd 开机自启服务...${NC}"
cat > /etc/systemd/system/clash.service <<EOF
[Unit]
Description=Clash Daemon
After=network.target

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/bin/clash -d /etc/clash

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable clash >/dev/null 2>&1
systemctl start clash

# 检查服务状态
sleep 2
if systemctl is-active --quiet clash; then
    echo -e "${GREEN}[+] Clash 服务已启动并设置开机自启。${NC}"
else
    echo -e "${RED}[!] Clash 服务启动失败，请检查日志: journalctl -u clash -e${NC}"
    exit 1
fi

echo -e "${YELLOW}[*] 步骤 5/5: 注入系统级全局控制命令 (tizi/untizi)...${NC}"
# 创建全局 profile 脚本
cat > /etc/profile.d/clash_control.sh << 'EOF'
# ================= Clash 全局控制脚本 =================

# 定义开启函数
function tizi() {
    # 1. 写入系统级环境变量文件 (/etc/environment) 实现持久化
    # 先清理旧的，防止重复堆叠
    sudo sed -i '/http_proxy/d; /https_proxy/d; /all_proxy/d; /no_proxy/d' /etc/environment
    
    # 写入新的
    echo 'http_proxy="http://127.0.0.1:7890"' | sudo tee -a /etc/environment >/dev/null
    echo 'https_proxy="http://127.0.0.1:7890"' | sudo tee -a /etc/environment >/dev/null
    echo 'all_proxy="socks5://127.0.0.1:7891"' | sudo tee -a /etc/environment >/dev/null
    echo 'no_proxy="localhost,127.0.0.1,::1"' | sudo tee -a /etc/environment >/dev/null

    # 2. 立即在当前 Shell 生效
    export http_proxy="http://127.0.0.1:7890"
    export https_proxy="http://127.0.0.1:7890"
    export all_proxy="socks5://127.0.0.1:7891"
    export no_proxy="localhost,127.0.0.1,::1"

    echo -e "\033[0;32m[OK] 系统全局梯子已【开启】！\033[0m"
    echo "状态已保存到 /etc/environment，所有新终端和后台服务将默认走代理。"
}

# 定义关闭函数
function untizi() {
    # 1. 从系统级环境变量文件中移除
    sudo sed -i '/http_proxy/d; /https_proxy/d; /all_proxy/d; /no_proxy/d' /etc/environment

    # 2. 立即清除当前 Shell 的变量
    unset http_proxy https_proxy all_proxy no_proxy

    echo -e "\033[0;33m[OK] 系统全局梯子已【关闭】！\033[0m"
    echo "服务器已恢复直连模式。"
}

# 3. 自动加载逻辑：每次打开新终端时，检查 /etc/environment 是否有代理配置
# 如果有，则自动 export 到当前会话，确保状态同步
if grep -q "http_proxy" /etc/environment; then
    export http_proxy="http://127.0.0.1:7890"
    export https_proxy="http://127.0.0.1:7890"
    export all_proxy="socks5://127.0.0.1:7891"
    export no_proxy="localhost,127.0.0.1,::1"
fi
EOF

# 清理临时文件
rm -rf /tmp/clash_install

echo -e "\n${GREEN}======================================================${NC}"
echo -e "${GREEN}   Clash 安装配置全部完成！   ${NC}"
echo -e "${GREEN}======================================================${NC}"
echo -e "请执行命令 \033[1;33msource /etc/profile\033[0m 使命令立刻生效，或重新登录终端。"
echo -e ""
echo -e "使用说明："
echo -e "  输入 \033[1;32mtizi\033[0m   : 开启全局代理 (持久保存，重启有效)"
echo -e "  输入 \033[1;33muntizi\033[0m : 关闭全局代理 (恢复直连)"
echo -e ""
echo -e "验证方式："
echo -e "  curl -I https://www.google.com"
echo -e "======================================================"
```

---

### 脚本执行后的效果：

1.  **Clash 核心**：已经作为后台服务在跑了，开机也会自己跑。
2.  **默认状态**：刚安装完默认是不走代理的（除非您手动去改配置）。
3.  **开启代理**：输入 `tizi`。脚本会把代理地址写进 `/etc/environment`。这意味着：
    *   **当前窗口**：马上能翻墙。
    *   **新开窗口**：马上能翻墙。
    *   **后台程序/定时任务**：也能翻墙。
    *   **重启服务器**：重启后，因为 `/etc/environment` 还在，所以依然保持翻墙状态。
4.  **关闭代理**：输入 `untizi`。脚本会把 `/etc/environment` 里的代理配置抹掉，服务器彻底恢复直连。

**特别提醒：**
脚本跑完最后，记得执行一次屏幕上提示的命令 `source /etc/profile`，或者直接**断开 SSH 重新连接**，这样 `tizi` 命令才会出现。