<sub><sup>**Forget my previous 3 articles** on the two Server/Client connection methods for DGX Spark: [Day01A: Remote Access from Internet Guide](https://github.com/Sniper711/DGX-Spark-Day01A-Remote-Access-from-Internet-Guide-20251220A/blob/main/DGX%20Spark%20(Day01A)%20Remote%20Access%20from%20Internet%20Guide%2020251220A.md) and [Day01B: Local Access from Same Subnet Guide](https://github.com/Sniper711/DGX-Spark-Day01B-Local-Access-from-Same-Subnet-Guide-20251220B/blob/main/DGX%20Spark%20(Day01B)%EF%BC%9ALocal%20Access%20from%20Same%20Subnet%20Guide%2020251220B.md), and the article [Day03: DGX Spark Now Accessible on Tablets and Mobile Devices](https://github.com/Sniper711/DGX-Spark-Day03-DGX-Spark-Now-Accessible-on-Tablets-and-Mobile-Devices-20260102/blob/main/DGX%20Spark%20(Day03)%20DGX%20Spark%20Now%20Accessible%20on%20Tablets%20and%20Mobile%20Devices%2020260102.md), because syncing Termius settings across mobile+desktop devices requires Termius PRO subscription payment! </sup></sub>

<sub><sup> Here, let's master Tailscale to Easily Replace WireGuide+Termius. This is a method officially recommended by NVIDIA, and it's free. I hope my experience can serve as a reference for you.</sup></sub>

<sub><sup>**忘掉我前面這三篇文章吧** DGX Spark : [第01天A: 外網遠端操控 指南](https://github.com/Sniper711/DGX-Spark-Day01A-Remote-Access-from-Internet-Guide-20251220A/blob/main/DGX%20Spark%20(%E7%AC%AC01%E5%A4%A9A)%20%E5%A4%96%E7%B6%B2%E9%81%A0%E7%AB%AF%E6%93%8D%E6%8E%A7%20%E6%8C%87%E5%8D%97%2020251220A.md) 與 [第01天B: 同子網內網操控 指南](https://github.com/Sniper711/DGX-Spark-Day01B-Local-Access-from-Same-Subnet-Guide-20251220B/blob/main/DGX%20Spark%20(%E7%AC%AC01%E5%A4%A9B)%EF%BC%9A%E5%90%8C%E5%AD%90%E7%B6%B2%E5%85%A7%E7%B6%B2%E6%93%8D%E6%8E%A7%20%E6%8C%87%E5%8D%97%2020251220B.md) 兩種 Server/Client 連線方式，以及 [第03天: DGX Spark 現可支援 平板與手機 遠端存取](https://github.com/Sniper711/DGX-Spark-Day03-DGX-Spark-Now-Accessible-on-Tablets-and-Mobile-Devices-20260102/blob/main/DGX%20Spark%20(%E7%AC%AC03%E5%A4%A9)%20DGX%20Spark%20%E7%8F%BE%E5%8F%AF%E6%94%AF%E6%8F%B4%20%E5%B9%B3%E6%9D%BF%E8%88%87%E6%89%8B%E6%A9%9F%20%E9%81%A0%E7%AB%AF%E5%AD%98%E5%8F%96%2020260102.md)。因為跨 mobile+desktop 裝置同步 Termius 設定是需要付費的。</sup></sub>

<sub><sup> 以下，我們改學會 Tailscale 輕鬆取代 WireGuard+Termius，這是來自 NVIDIA 官方推薦的方法，而且免費。希望我的經驗能給你參考。</sup></sub>

---
# DGX Spark (Day05) Mastering Tailscale to Easily Replace WireGuide+Termius 20260116 🟩 [English](https://github.com/Sniper711/DGX-Spark-Day05-Mastering-Tailscale-to-Easily-Replace-WireGuide-and-Termius-20260116/blob/main/DGX%20Spark%20(Day05)%20Mastering%20Tailscale%20to%20Easily%20Replace%20WireGuide+Termius%2020260116.md)
# DGX Spark (第05天) 學會用 Tailscale 輕鬆取代 WireGuard+Termius 20260116 🟩 [中文版](https://github.com/Sniper711/DGX-Spark-Day05-Mastering-Tailscale-to-Easily-Replace-WireGuide-and-Termius-20260116/blob/main/DGX%20Spark%20(%E7%AC%AC05%E5%A4%A9)%20%E5%AD%B8%E6%9C%83%E7%94%A8%20Tailscale%20%E8%BC%95%E9%AC%86%E5%8F%96%E4%BB%A3%20WireGuard%2BTermius%2020260116.md)


> ## Scenarios & Advantages
> **Use Tailscale to Easily Replace WireGuard + Termius, Free, and Easy to Maintain**
> - **Simply install one piece of software: Tailscale**
>   - Tailscale installation doesn't differentiate between Server/Client configurations like WireGuard, and there's no need to manage four public/private keys for WireGuard Server/Client, avoiding the risk of key leakage.
>   - Tailscale VPN connects only to devices on the internal network that have Tailscale installed, providing an extra layer of protection; whereas WireGuard VPN directly accesses all devices on the internal network once connected.
>   - (Note: If needed, Tailscale also supports installing a subnet router to allow direct access to all devices on the intranet.)
>   - Tailscale doesn't require additional SSH software, such as Termius.
>   - Tailscale SSH allows opening a terminal directly in the browser, which is super convenient.
>   - Tailscale automatically handles NAT traversal and hole punching and doesn't need additional Port Forwarding software, like Termius.
>   - Tailscale excels in multi-user team management.
>   - Tailscale installation is a breeze — just 5 minutes.
>   - (Note: If needed, Tailscale also supports self-hosting Headscale to protect user account binding and login info, as well as to safeguard metadata like who connects to whom, when, from which IP, etc.)
> - **Avoid Termius cross mobile+desktop devices requires Termius PRO subscription payment**
>   - Avoid shelling out ~$60 USD per year for Termius's cross-device sync (e.g., between 1 tablet and 1 DGX Spark).
>   - Tailscale offers free support for up to 3 users and 100 devices.
> - **Use Tailscale-Assigned 100.x.x.x IPs for Interconnection**
>   - Devices running Tailscale connect to each other via the 100.x.x.x IPs it assigns, blurring the lines between internal and external networks.
>   - No need to get a fixed external IP—especially handy when your DGX Spark is at a company or school, where you can't request one.
>   - It works reliably even without a public IP, open ports, or stable IPs that change daily. Future changes in network gear or topology at work or school won't faze it—Tailscale keeps maintenance costs low.
<img width="1350" height="544" alt="截圖 2026-01-23 14 14 51" src="https://github.com/user-attachments/assets/ac3e79a6-da56-4334-8312-59edeefcb62a" />

### Congratulations - Now you can connect to your DGX Spark from any device and unleash its GPU power to run all kinds of AI applications smoothly!
<br>


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

### 恭喜你！從此你能用任何裝置連上 DGX Spark，用 DGX Spark 的 GPU 算力暢跑各種 AI 應用了！
<br>

<img width="1330" height="414" alt="Day05 Mastering Tailscale to easily replace WireGuard+Termius" src="https://github.com/user-attachments/assets/2e658507-d100-4be5-b25c-7dd4399039a1" />
![Uploading Day05 Mastering Tailscale (IP) to easily replace WireGuard+Termius.png…]()

<img width="893" height="736" alt="截圖 2026-01-23 14 17 00" src="https://github.com/user-attachments/assets/71700ff9-819e-48c3-9708-7328615d7542" />

---

## 喜歡這個專案嗎？ 如果對您有幫助，請給一個 ★ Star 吧！這對我非常重要！

## If you find this project helpful, please give it a Star ★! Your support means a lot to me!

---
Davis Lin (Sniper711) .
Unauthorized article copying, distribution, or modification is prohibited.
