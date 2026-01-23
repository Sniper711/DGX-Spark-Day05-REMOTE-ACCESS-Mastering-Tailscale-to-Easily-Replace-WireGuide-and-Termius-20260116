Forget about my (Day01A)+(Day01B)+(Day03) Guilds. This (Day05) Tailscale Guide officially from NVIDIA can replace them all, and is amazingly easy. Thanks NVIDIA. 

<sub><sup>忘掉我前面這三篇文章吧 DGX Spark : [第01天A: 外網遠端操控 指南](https://github.com/Sniper711/DGX-Spark-Day01A-Remote-Access-from-Internet-Guide-20251220A/blob/main/DGX%20Spark%20(%E7%AC%AC01%E5%A4%A9A)%20%E5%A4%96%E7%B6%B2%E9%81%A0%E7%AB%AF%E6%93%8D%E6%8E%A7%20%E6%8C%87%E5%8D%97%2020251220A.md) 與 [第01天B: 同子網內網操控 指南](https://github.com/Sniper711/DGX-Spark-Day01B-Local-Access-from-Same-Subnet-Guide-20251220B/blob/main/DGX%20Spark%20(%E7%AC%AC01%E5%A4%A9B)%EF%BC%9A%E5%90%8C%E5%AD%90%E7%B6%B2%E5%85%A7%E7%B6%B2%E6%93%8D%E6%8E%A7%20%E6%8C%87%E5%8D%97%2020251220B.md) 兩種 Server/Client 連線方式，以及 [第03天: DGX Spark 現可支援 平板與手機 遠端存取](https://github.com/Sniper711/DGX-Spark-Day03-DGX-Spark-Now-Accessible-on-Tablets-and-Mobile-Devices-20260102/blob/main/DGX%20Spark%20(%E7%AC%AC03%E5%A4%A9)%20DGX%20Spark%20%E7%8F%BE%E5%8F%AF%E6%94%AF%E6%8F%B4%20%E5%B9%B3%E6%9D%BF%E8%88%87%E6%89%8B%E6%A9%9F%20%E9%81%A0%E7%AB%AF%E5%AD%98%E5%8F%96%2020260102.md)。因為跨 mobile+desktop 裝置同步 Termius 設定是需要付費的。</sup></sub>
<sub><sup> 以下，我們改學會 Tailscale 輕鬆取代 Wireguard+Termius，這是來自 NVIDIA 官方推薦的方法，而且免費。希望我的經驗能給你參考。</sup></sub>

# DGX Spark (第05天) 學會用 Tailscale 輕鬆取代 Wireguard+Termius 20260116
## 🟩 中文版
> ## 適用情境 與 優點
> **用 Tailscale 輕鬆取代 Wireguard+Termius，免費，容易維護**
> - **只安裝一套 Tailscale 軟體**
>   - Tailscale 安裝不像 WireGuard 分 Server/Client 設定，也不用管理 WireGuard Server/Client 四隻 公鑰/私鑰，避免鑰匙流出風險。
>   - Tailscale VPN 連進來只通內網有安裝 Tailscale 的設備，還有一層保護；而 WireGuard VPN 連進內網就直接通內網所有設備。
>   - (註：若有需要，Tailscales 還支持安裝 subnet router 後，也能進內網直接通內網所有設備。)
>   - Tailscale 不需要另外的 SSH 軟體，例如 Termius。
>   - Tailscale SHH 讓瀏覽器能直接開 terminal，超方便。 
>   - Tailscale 自動NAT穿透/打洞不需設定 也不需另外的 Port Forwarding 軟體，例如Termius。
>   - Tailscale 多人團隊管理超強。
>   - Tailscale 安裝更簡單，只需 5 分鐘。
>   - (註：若有需要，Tailscale 還支持自架 Headscale 後，就能保護 用戶帳號綁定與登入資訊，也保護 誰連誰，何時連，從哪個IP 等等的 Metadata)
> - **避開 Termius 跨 mobile+desktop 裝置設定同步 需要收費**
>   - 避免 Termius 只要 跨 mobile+desktop 裝置設定同步 (例:一台平板 + 一台DGX Spark) 需要付費一年台幣1800。
>   - 改用 Tailscale 多達3人且100個裝置 仍然免費。
> - **使用 Tailscale 派發的 100.x.x.x IP 互連**
>   - 安裝 Tailscale 的設備彼此使用 Tailscale 派發的 100.x.x.x IP 互連。使用上不分內網與外網。
>   - 不需要申請對外固定IP. 尤其當你把 DGX Spark 放公司與學校時，無法要求公司與學校為你申請一個固定IP.
>   - 完全沒公網 IP、沒開任何端口、IP 每天變，也能穩定連上。將來公司與學校經常換內網的網通設備與網路拓樸也不怕，Tailscale 維護成本低。


<img width="1493" height="304" alt="截圖 2026-01-23 10 52 19" src="https://github.com/user-attachments/assets/b0c19936-80bd-4640-b345-92343f0cdd9f" />



---


## 目錄 Table of Contents
- [1. 先設定Router (必做)](#1-先設定router-必做)
  - [1.1 確認網路拓樸](#11-確認網路拓樸)
  - [1.2 確認已關閉 Router VPN](#12-確認已關閉-router-vpn)
  - [1.3 設定 Port Forward (Router)](#13-設定-port-forward-router)

- [2. DGX Spark (Server) 確認實體網卡名稱 (非常重要)，與 安裝 WireGuard](#2-dgx-spark-server-確認實體網卡名稱-非常重要與-安裝-wireguard)
  - [2.1 確認實體網卡名稱](#21-確認實體網卡名稱)
  - [2.2 安裝 WireGuard VPN](#22-安裝-wireguard-vpn)

- [3. DGX Spark (Server) 建立 WireGuard 目錄與金鑰](#3-dgx-spark-server-建立-wireguard-目錄與金鑰)
  - [3.0 進入 root shell（非常重要）](#30-進入-root-shell非常重要)
  - [3.1 建立目錄](#31-建立目錄)
  - [3.2 產生金鑰（仍在 root shell內, 看到 promot 類似-rootspark-xxxx）](#32-產生金鑰仍在-root-shell內-看到-promot-類似-rootspark-xxxx)
  - [3.3 檢查金鑰](#33-檢查金鑰)

- [4. DGX Spark (Server) 建立 WireGuard Server 設定檔](#4-dgx-spark-server-建立-wireguard-server-設定檔)
  - [4.0 說明 路徑 與 檔名](#40-說明-路徑-與-檔名)
  - [4.1 建立設定檔 `wg-dgx.conf`](#41-建立設定檔-wg-dgxconf)
  - [4.2 用nano編輯 `wg-dgx-spark.conf` 文件 (複製貼上)](#42-用nano編輯-wg-dgx-sparkconf-文件-複製貼上)
  - [4.3 Server/Client PublicKey/PrivateKey 速查對照表](#43-serverclient-publickeyprivatekey-速查對照表)

- [5. DGX Spark (Server) 啟用 IPv4 Forward（必做，只需一次）](#5-dgx-spark-server-啟用-ipv4-forward必做只需一次)
  - [5.1 立即生效](#51-立即生效)
  - [5.2 設為永久](#52-設為永久)
  - [5.3 驗證](#53-驗證)

- [6. DGX Spark (Server) 啟動 WireGuard Server](#6-dgx-spark-server-啟動-wireguard-server)
  - [6.1 檢查 systemd (系統守護進程) 狀態](#61-檢查-systemd-系統守護進程-狀態)
  - [6.2 檢查 WireGuard 狀態](#62-檢查-wireguard-狀態)

- [7. Mac/PC (Client) 建立 WinGuard Client 設定檔](#7-macpc-client-建立-winguard-client-設定檔)
  - [7.0 說明](#70-說明)
  - [7.1 建立設定檔](#71-建立設定檔)
  - [7.2 用nano編輯 `wg-mac-pc.conf` 文件 (複製貼上)](#72-用nano編輯-wg-mac-pcconf-文件-複製貼上)
  - [7.3 Server/Client PublicKey/PrivateKey 速查對照表](#73-serverclient-publickeyprivatekey-速查對照表)

- [8. Mac/PC (Client) 從外網 VPN 連回家裡的 DGX Spark (Server)](#8-macpc-client-從外網-vpn-連回家裡的-dgx-spark-server)
  - [8.1 安裝 WireGuard VPN](#81-安裝-wireguard-vpn)
  - [8.2 導入 WireGuard client 設定檔，與 啟動 VPN (一定要用外網)](#82-導入-wireguard-client-設定檔與-啟動-vpn-一定要用外網)
  - [8.3 三種方法測試從Client連回Server的 `VPN通道` (一定要用外網)](#83-三種方法測試從client連回server的-vpn通道-一定要用外網)

- [9. 從 Mac/PC (Client) 登入/命令 DGX Spark (Server)](#9-從-macpc-client-登入命令-dgx-spark-server)
  - [9.1 SSH指令 從 Mac/PC (Client) 登入 DGX Spark (Server) ](#91-ssh指令-從-macpc-client-登入-dgx-spark-server)
  - [9.2 兩種範例方法 從 Mac/PC (Client) 命令 DGX Spark (Server)](#92-兩種範例方法-從-macpc-client-命令-dgx-spark-server)

---

## 1. 先設定Router (必做) 

### 1.1 確認網路拓樸
- 確認在 DGX Spark 的前端，只能有唯一的一台 Router：
  - 此 Router 直接接源頭 光纖Modem
  - 此 Router 用 PPPoE 登入電信公司，需有 固定 Public IP (x.x.x.x)
- 在 DGX Spark 的前端，家裡沒有其他的 Router (若有, 把其他 Router 都設在 AP Mode 當 Switch 用)
  - 這樣 DGX Spark 無論接在哪裡，都等於接在唯一的一台 Router 後面
- DGX Spark 的同一層與更後層的網路，可以有其他的 Router

### 1.2 確認已關閉 Router VPN
- 登入 Router
- VPN Server / VPN Client：全部關閉
- 不保留任何帳號、設定

### 1.3 設定 Port Forward (Router)
先找出 DGX Spark 內網IP位址
- 登入 Router 設定主畫面，看到裝置 spark-xxxx 內網 IP 位址 (192.168.x.x) 的值，記錄起來。

用 PPPoE 登入電信公司，取得事先已申請的固定 Public IP (通常在 Router 設定 `主畫面` -> `WAN` -> `WAN Setting` )
- WAN Connection Type：選 PPPoE
- PPPoE Setting：
  - Address Mode = Dynamic IP
  - User Name = 電信公司給你的固定 Public IP 登入帳號
  - Password = 電信公司給你的固定 Public IP 登入密碼
  - Operation Mode = Keep Alive
- Save 存檔 


依此設定 Port Forward (通常在 Router 設定 `主畫面` -> `WAN` -> `Port Forwarding` )
| 欄位 | 值 |
|------|------|
| Rule Name | wireguard |
| Protocal | UDP |
| Public Port | 51820 |
| Private Port | 51820 |
| Private IP | DGX Spark 內網IP (填入一個 192.168.x.x 的 DGX Spark 內網IP值) |
| Inbound Filter | Allow All |
| 啟用 | Yes |

原理說明
- Router 將外網 `UDP:51820`
- 轉送到 DGX Spark `UDP:51820`
- 對應 `ListenPort = 51820`

快速自我檢查
- Port Forward 啟用
- 無其他 VPN 佔用該 Port
- Public / Private Port 一致

---

## 2. DGX Spark (Server) 確認實體網卡名稱 (非常重要)，與 安裝 WireGuard

### 2.1 確認實體網卡名稱
```
ip link
```

記下實體網卡名稱 `(exxxxx 六碼英數字)`，例如：
```
exxxxx (我這台DGX Spark的實體網卡名稱範例，是6碼英數字)
```
- 以下指令基於此 `exxxxx` 六碼英數字的實體網卡名稱範例，設定 WineGuard Server 設定檔
    - 若改硬體，則本篇文章的指令需跟著改
    - 例，若加購第二台 DGX Spark，這名稱可能不同，必須重新確認。

### 2.2 安裝 WireGuard VPN
```
sudo apt update
sudo apt install -y wireguard
```

驗證版本（必做）
```
wg --version
```

---

## 3. DGX Spark (Server) 建立 WireGuard 目錄與金鑰

### 3.0 進入 root shell（非常重要）
```
sudo -i
```

你會看到 prompt 類似：
```
root@spark-xxxx:~#
```

為什麼要先 sudo -i
- /etc 是 root-only 可寫目錄，cd 不會繼承 sudo
- umask 是 shell built-in，不能用 sudo umask
- 避免兩個常見錯誤：
  - cd: Permission denied
  - sudo: umask: command not found


### 3.1 建立目錄

說明
- 取路徑名為：
  - `/etc/wireguard`
  - 以下指令基於此 WireGuard Server 設定檔路徑名
    - 若改路徑名則本篇文章的指令需跟著改
```
mkdir -p /etc/wireguard
cd /etc/wireguard
umask 077
```

### 3.2 產生金鑰（仍在 root shell內, 看到 promot 類似 `root@spark-xxxx:~#`）
```
wg genkey | tee server_private.key | wg pubkey > server_public.key
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

為什麼這樣寫是「正確且唯一建議」
- `tee private.key`：確保安全，私鑰直接寫進.key檔案，不會出現在終端機的 shell歷史紀錄
- `> public.key`：避免誤用 `tee` 生成導致覆蓋錯誤

### 3.3 檢查金鑰
```
ls -l /etc/wireguard/
```

應看到
```
server_private.key
server_public.key
client_private.key
client_public.key
```

---

## 4. DGX Spark (Server) 建立 WireGuard Server 設定檔

### 4.0 說明 路徑 與 檔名
- 取路徑名為：
  - `/etc/wireguard`
  - 以下指令基於此 WireGuard Server 設定檔 路徑名
    - 若改路徑名則本篇文章的指令需跟著改
- 取檔名為：
  - `wg-dgx-spark.conf`
  - 以下指令基於此 WireGuard Server 設定檔 檔名
    - 若改檔名則本篇文章的指令需跟著改

### 4.1 建立設定檔 `wg-dgx.conf`
- 在 DGX Spark Server 的 `/etc/wireguard` 目錄下, 建立設定檔
```
nano /etc/wireguard/wg-dgx-spark.conf
```

### 4.2 用nano編輯 `wg-dgx-spark.conf` 文件 (複製貼上)
```
[Interface]
# WireGuard Server 內部 IP（VPN 網段）
Address = 10.100.0.1/24

# WireGuard 監聽Port（需與 Router 的 Port Forward 設定一致）
ListenPort = 51820

# Server PrivateKey
# 查詢方式（所有的 Keys 都只能在 DGX Spark 下指令查詢）：
#   sudo cat /etc/wireguard/server_private.key
# 記住：Server PrivateKey 必須存放在 DGX Spark（Server）
#
# 把 <server_private_key> 包含括弧刪掉, 置換成 Server PrivateKey 的值
PrivateKey = <server_private_key>

# 啟用 NAT + Forward
# 讓 VPN Client 可存取 DGX Spark 內網與外網
#
# wg-dgx-spark = WireGuard 介面名稱（與檔名一致）
#
# <exxxxx>       = DGX Spark 實體網卡名稱（需依實機確認）
# 把 <exxxxx> 包含括弧刪掉, 置換成 DGX Spark 實體網卡名稱的值（需依實機確認，步驟 1.1 有指令能查出來）
PostUp   = iptables -A FORWARD -i wg-dgx-spark -j ACCEPT; iptables -t nat -A POSTROUTING -o <exxxxx> -j MASQUERADE
PostDown = iptables -D FORWARD -i wg-dgx-spark -j ACCEPT; iptables -t nat -D POSTROUTING -o <exxxxx> -j MASQUERADE

[Peer]
# Client PublicKey
# 查詢方式（所有的 Keys 都只能在 DGX Spark 下指令查詢）：
#   sudo wg show wg-dgx-spark
#   (*輸出一段文字，看 peer 後面的字串才是 public key)
# 或：
#   sudo cat /etc/wireguard/client_public.key
#
# 記住：Client PublicKey 必須存放在 DGX Spark（Server）
#
# 把 <client_public_key> 包含括弧刪掉, 置換成 Client PublicKey 的值
PublicKey = <client_public_key>

# WireGuard Client 在 VPN 內的固定 IP
AllowedIPs = 10.100.0.2/32
```

結束時，

按 `Ctrl+O` 存檔，

按 `Enter` 確認覆寫檔案，

按 `Ctrl+X` 離開 nano 編輯器。

### 4.3 Server/Client PublicKey/PrivateKey 速查對照表
| Key 是誰的 | 在 DGX Spark Server 機上 用什麼指令查 Key | Key 要貼到哪裡 |
|------|------|------|
| Server PublicKey | cat server_public.key 或 wg show wg-dgx-spark public-key | Client 機的 wg-mac-pc.conf --> 貼在 [Peer] PublicKey 底下 | 
| Server PrivateKey | cat server_private.key | Server 機的 wg-dgx-spark.conf --> 貼在 [Interface] PrivateLeyKey 底下 |
| Client PublicKey | cat client_public.key 或 wg show wg-dgx-spark --> 看 peer: 後面有顯示 | Server 機的 wg-dgx-spark.conf --> 貼在 [Peer] PublicKey 底下 |
| Client PrivateKey | cat client_private.key | Client 機的 wg-mac-pc.conf --> 貼在 [Interface] PrivateKey 底下 |
- 只有 DGX Spark Server 機上能查所有的 Server/Client Key
- DGX Spark Server 機的 wg-dgx-spark.conf 檔案，貼 Server PrivateKey 與 Client PublicKey (交錯，仔細想想合理)
- Mac/PC Client 機的 wg-mac-pc.conf 檔案，貼 Client PrivateKey 與 Server PublicKey (交錯，仔細想想合理)

---

## 5. DGX Spark (Server) 啟用 IPv4 Forward（必做，只需一次）
沒有這一步，VPN 會連得上，但流量無法轉送

### 5.1 立即生效
```
sysctl -w net.ipv4.ip_forward=1
```

### 5.2 設為永久
```
echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/99-wireguard.conf
sysctl --system
```

### 5.3 驗證
```
sysctl net.ipv4.ip_forward
```
應顯示
```
net.ipv4.ip_forward = 1
```

---

 
## 6. DGX Spark (Server) 啟動 WireGuard Server
```
systemctl enable wg-quick@wg-dgx-spark
systemctl start wg-quick@wg-dgx-spark
```

### 6.1 檢查 systemd (系統守護進程) 狀態
```
systemctl status wg-quick@wg-dgx-spark
```
應該看到：
- 一長串包括 `Active: active` 的文字
這次使用 `q` 離開 (不要按Ctrl+C)

### 6.2 檢查 WireGuard 狀態
```
wg
```
應該看到：
- interface: wg-dgx-spark
- listening port: 51820
- peer: 後面有顯示 (這是 Client Public Key)
- AllowedIPs = 10.100.0.2/32

---

## 7. Mac/PC (Client) 建立 WinGuard Client 設定檔

剛剛完成 Server端 DGX Spark 的設定。

現在開始 Client端 Mac/PC 的設定。

### 7.0 說明
- 取相對路徑名為：
  - `~/vpn/wireguard` (這是相對路徑，我們用相對路徑下指令)
    - 在Mac的絕對路徑會是 `User/<Your PC Name>/vpn/wireguard`，絕對路徑會隨著每台 Mac 的 PC Name而不同
  - 以下指令基於此 WireGuard Client 設定檔 相對路徑名
    - 若改相對路徑名則本篇文章的指令需跟著改
- 取檔名為：
  - `wg-mac-pc.conf`
  - 以下指令基於此 WireGuard Client 設定檔 檔名
    - 若改檔名則本篇文章的指令需跟著改

### 7.1 建立設定檔
- 在 Mac/PC Client 的 `~/vpn/wireguard` 目錄下, 建立設定檔 `wg-mac-pc.conf`
```
nano ~/vpn/wireguard/wg-mac-pc.conf
```

### 7.2 用nano編輯 `wg-mac-pc.conf` 文件 (複製貼上)
```
[Interface]
# Client PrivateKey
# 查詢方式（所有的 Keys 都只能在 DGX Spark 下指令查詢）：
#   sudo cat /etc/wireguard/client_private.key
# 記住：Client PrivateKey 必須存放在 Mac-PC（Client）
#
# 把 <client_private_key> 包含括弧刪掉, 置換成 Client PrivateKey 的值
PrivateKey = <client_private_key>

# WireGuard Client 在 VPN 內的固定 IP
# 必須與 DGX Spark（Server）wg-dgx-spark.conf 中的 AllowedIPs 對齊
Address = 10.100.0.2/32

# VPN 啟用後使用的 DNS
# 可使用中華電信與 Google DNS，或未來改為內網 DNS
# 建議換成你常用的 DNS
DNS = 168.95.192.1, 8.8.8.8

[Peer]
# Server PublicKey
# 查詢方式（所有的 Keys 都只能在 DGX Spark 下指令查詢）：
#   sudo wg show wg-dgx-spark
# 記住：Server PublicKey 必須存放在 Mac-PC（Client）
#
# 把 <server_public_key> 包含括弧刪掉, 置換成 Server Public 的值
PublicKey = <server_public_key>

# WireGuard Server 對外連線位址
# 使用 DGX Spark 的外網固定 IP
# Port 需同時符合：
#   - DGX Spark wg-dgx-spark.conf 中的 ListenPort
#   - Router WAN Port Forwarding 設定
#
# 把 <DGX_SPARK_PUBLIC_IP> 包含括弧刪掉, 置換成 DGX Spark 固定 Public IP (x.x.x.x) 的值
Endpoint = <DGX_SPARK_PUBLIC_IP>:51820

# Full Tunnel：所有流量經由 VPN
AllowedIPs = 0.0.0.0/0

# NAT 穿透保活（keepalive, 是 Client 在 NAT 後方時必備）
PersistentKeepalive = 25
```
結束時，

按 `Ctrl+O` 存檔，

按 `Enter` 確認覆寫檔案，

按 `Ctrl+X` 離開 nano 編輯器。

### 7.3 Server/Client PublicKey/PrivateKey 速查對照表
| Key 是誰的 | 在 DGX Spark Server 機上 用什麼指令查 | Key 要貼到哪裡 |
|------|------|------|
| Server PublicKey | cat server_public.key 或 wg show wg-dgx-spark public-key | Client 機的 wg-mac-pc.conf --> 貼在 [Peer] PublicKey 底下 | 
| Server PrivateKey | cat server_private.key | Server 機的 wg-dgx-spark.conf --> 貼在 [Interface] PrivateLeyKey 底下 |
| Client PublicKey | cat client_public.key 或 wg show wg-dgx-spark --> 看 peer: 後面有顯示 | Server 機的 wg-dgx-spark.conf --> 貼在 [Peer] PublicKey 底下 |
| Client PrivateKey | cat client_private.key | Client 機的 wg-mac-pc.conf --> 貼在 [Interface] PrivateKey 底下 |
- 只有 DGX Spark Server 機上能查所有的 Server/Client Key
- DGX Spark Server 機的 wg-dgx-spark.conf 檔案，貼 Server PrivateKey 與 Client PublicKey (交錯，仔細想想合理)
- Mac/PC Client 機的 wg-mac-pc.conf 檔案，貼 Client PrivateKey 與 Server PublicKey (交錯，仔細想想合理)

---

## 8 Mac/PC (Client) 從外網 VPN 連回家裡的 DGX Spark (Server)
**秘訣**：手機開熱點，分享網路給 Mac/PC (Client) 連上。以模擬外網環境。

**在 Mac/PC (Client) 上：** (一定要用外網)
確認 Mac/PC 沒有連接家裡內網，此時：
- Mac/PC 拔掉有線網路
- Mac/PC 連接手機熱點

### 8.1 安裝 WireGuard VPN
- macOS : App Store / wireguard.com 兩種下載安裝方式
- Windows / Linux : wireguard.com 一種下載安裝方式

### 8.2 導入 WireGuard client 設定檔，與 啟動 VPN (一定要用外網)
**重要**：繼續以下步驟之前，確保你的 Mac/PC 已經在外網環境

- 開啟 WireGuard
- 按左下角 `+` 號，接著選 `Import Tunnel(s) from File`
- 從相對路徑 `~/vpn/wireguard` 選取檔案 `wg-mac-pc.conf`，按 `Import`
- 視窗顯示內容無誤的話，按 `啟動`

### 8.3 三種方法測試從Client連回Server的 `VPN通道` (一定要用外網)：

#### 方法一，ping 大多數家裡 Router 的默認網關：
若 VPN通道 工作正常，正確結果應該是能成功
```
ping 10.100.0.1
```
或
```
ping 10.10.0.1
```

#### 方法二，ping 家裡 DGX Spark 的內網 IP：
若 VPN通道 工作正常，正確結果應該是能成功
```
# 把 <192.168.x.x> 包含括弧刪掉, 置換成 DGX Spark 內網 IP 位址 (192.168.x.x) 的值
ping <192.168.x.x>
```

#### 方法三，查詢家裡 Router 對外的 固定 Public IP位址：
若 VPN通道 工作正常，正確結果應該是：能 顯示 固定 Public IP 的值 x.x.x.x
```
curl ifconfig.me
```
應該看到這三個方法都成功。

---

## 9. 從 Mac/PC (Client) 登入/命令 DGX Spark (Server) 

### 9.1 SSH指令 從 Mac/PC (Client) 登入 DGX Spark (Server) 
**SSH** 超簡單一行指令 登入 DGX Spark Server (需輸入 DGX Spark 開機密碼)

###### 在 Mac/PC Client 上執行命令 執行後，會**看到終端機的命令提示字元變化**，從 Mac/PC Client機的 <本機用戶>@<本機名稱>%，變成 DGX Server 機的 <server機用戶>@Spark-xxxx:$，表示已登入。

<sub><sup>＊重開機之後，只要 Mac/PC (Client) 執行步驟9.1這一行SHH指令，超級簡單。</sup></sub>

```
# 把 <DGX Spark username> 包含括弧刪掉, 置換成 DGX Spark 開機後登入的 username
# 把 <192.168.x.x> 包含括弧刪掉, 置換成 DGX Spark 內網 IP 位址 (192.168.x.x) 的值
ssh <DGX Spark username>@<192.168.x.x>
```

### 9.2 兩種範例方法 從 Mac/PC (Client) 命令 DGX Spark (Server)
**從 Mac/PC (Client) 下命令**

方法一，命令 DGX Spark (Server) 每秒監控一次 GPU 溫度 (GPU Temp欄位) 、GPU使用率 (GPU-Util欄位)
```
watch -n 1 nvidia-smi
```

方法二，命令 DGX Spark (Server) 每秒監控一次 系統記憶體的 總量 (total欄位)、當前使用量 (used欄位)

```
watch -n 1 free -h
```
應該看到這兩個方法都成功。

---

# **恭喜你！從此你能從任何地方連回你心愛的 DGX Spark 了！**
<sub><sup>＊重開機之後，只要 Mac/PC (Client) 執行步驟9.1這一行SHH指令，超級簡單。</sup></sub>

---
