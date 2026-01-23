<sub><sup>**忘掉我前面這三篇文章吧** DGX Spark : [第01天A: 外網遠端操控 指南](https://github.com/Sniper711/DGX-Spark-Day01A-Remote-Access-from-Internet-Guide-20251220A/blob/main/DGX%20Spark%20(%E7%AC%AC01%E5%A4%A9A)%20%E5%A4%96%E7%B6%B2%E9%81%A0%E7%AB%AF%E6%93%8D%E6%8E%A7%20%E6%8C%87%E5%8D%97%2020251220A.md) 與 [第01天B: 同子網內網操控 指南](https://github.com/Sniper711/DGX-Spark-Day01B-Local-Access-from-Same-Subnet-Guide-20251220B/blob/main/DGX%20Spark%20(%E7%AC%AC01%E5%A4%A9B)%EF%BC%9A%E5%90%8C%E5%AD%90%E7%B6%B2%E5%85%A7%E7%B6%B2%E6%93%8D%E6%8E%A7%20%E6%8C%87%E5%8D%97%2020251220B.md) 兩種 Server/Client 連線方式，以及 [第03天: DGX Spark 現可支援 平板與手機 遠端存取](https://github.com/Sniper711/DGX-Spark-Day03-DGX-Spark-Now-Accessible-on-Tablets-and-Mobile-Devices-20260102/blob/main/DGX%20Spark%20(%E7%AC%AC03%E5%A4%A9)%20DGX%20Spark%20%E7%8F%BE%E5%8F%AF%E6%94%AF%E6%8F%B4%20%E5%B9%B3%E6%9D%BF%E8%88%87%E6%89%8B%E6%A9%9F%20%E9%81%A0%E7%AB%AF%E5%AD%98%E5%8F%96%2020260102.md)。因為跨 mobile+desktop 裝置同步 Termius 設定是需要付費的。</sup></sub>

<sub><sup> 以下，我們改學會 Tailscale 輕鬆取代 WireGuard+Termius，這是來自 NVIDIA 官方推薦的方法，而且免費。希望我的經驗能給你參考。</sup></sub>

# DGX Spark (第05天) 學會用 Tailscale 輕鬆取代 WireGuard+Termius 20260116
## 🟩 中文版
> ## 適用情境 與 優點
> **用 Tailscale 輕鬆取代 WireGuard+Termius，免費，容易維護**
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
<img width="648" height="531" alt="截圖 2026-01-23 14 05 17" src="https://github.com/user-attachments/assets/39948f26-0e75-427c-9cbb-c6f9c0a5e202" />


---

## NVIDIA 官網 - [在您的 Spark 上設定 Tailscale](https://build.nvidia.com/spark/tailscale/instructions)
### 依照 NVIDIA 官網指南的步驟完成安裝。

---

# **恭喜你！從此你能用任何裝置連上 DGX Spark，用 DGX Spark 的 GPU 算力暢跑各種應用了！**

---
