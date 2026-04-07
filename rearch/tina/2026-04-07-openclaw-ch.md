# OpenClaw 開發歷史深度研究報告

**研究日期：** 2026-04-07  
**研究主題：** OpenClaw 發展歷史與高光時刻  
**研究範圍：** 官方網站、GitHub Repo、Telegram/Discord Channel 更新

---

## 目錄

1. [專案概述](#專案概述)
2. [起源故事](#起源故事)
3. [命名演變史](#命名演變史)
4. [重大里程碑](#重大里程碑)
5. [Telegram Channel 更新高光](#telegram-channel-更新高光)
6. [Discord Channel 更新高光](#discord-channel-更新高光)
7. [安全事件記錄](#安全事件記錄)
8. [創作者與團隊](#創作者與團隊)
9. [未來展望](#未來展望)
10. [參考來源](#參考來源)

---

## 專案概述

**OpenClaw** 是一個開源 AI 代理框架，以多通道訊息整合為核心特色。專案允許用戶透過 Telegram、Discord、WhatsApp、Signal 等多種通訊平台與 AI 代理互動。

### 核心數據
- **GitHub Stars：** 346,000+ (截至 2026年4月)
- **暴露實例：** 135,000+ (遍佈 82 國)
- **ClawHub Skills：** 2,857+ 個社群技能
- **支援平台：** Telegram、Discord、WhatsApp、Signal、Matrix、Feishu、Slack 等

### 技術棧
- Node.js v24+
- Multi-channel AI Gateway
- Plugin 架構
- Context Engine (記憶系統)

---

## 起源故事

### 2025年11月：週末專案的誕生

一切始於 2025 年 11 月，奧地利開發者 **Peter Steinberger**（PSPDFKit 創始人）在一個週末打造了一個名為 **「WhatsApp Relay」** 的小專案。

> *"這個龍蝦開始接管世界" — Peter Steinberger*

最初的構想很簡單：建立一個 WhatsApp 閘道器，讓 AI 助理能透過即時通訊平台與用戶互動。Peter 將其命名為 **Clawd** — 這是對 Anthropic Claude 的致敬（結合龍蝦意象），同時體現專案的核心精神。

### 第一個里程碑：從玩具到工具

專案發布後迅速引起社群關注。開發者發現這個「玩具」有著實際生產力的潛力：

- **2025年12月3日「目錄傾印事件」：** 早期版本曾意外在群組對話中分享整個目錄結構，暴露了權限控管的需求
- **機器人購物清單事件：** 從玩笑到實際研究 Boston Dynamics Spot ($74,500)、Unitree G1 EDU ($40,000)，最終訂購了 Reachy Mini

---

## 命名演變史

OpenClaw 經歷了開源史上最快的「三重命名」過程：

### 時間軸

| 時期 | 名稱 | 期間 |
|------|------|------|
| 2025年11月 - 2026年1月27日 | **Clawdbot** | ~2個月 |
| 2026年1月27日 - 1月30日 | **Moltbot** | 僅3天 |
| 2026年1月30日 - 至今 | **OpenClaw** | 正式名稱 |

### 第一次改名：Clawdbot → Moltbot

**原因：** Anthropic 發出商標侵權警告信，認為「Clawd」與「Claude」過於相似。

Peter 在收到郵件後迅速回應，採用生物學靈感：

> *"龍蝦會蛻殼（molt）來成長，所以就叫 Moltbot 吧！"*

### 第二次改名：Moltbot → OpenClaw

**原因：** Moltbot 發音不順口 + 重大安全事件後的品牌重塑需求。

**2026年1月30日「最終型態」行動：**
- 凌晨 4 點 GMT，團隊再次集結
- 3 小時內完成：
  - GitHub 重命名為 `github.com/openclaw/openclaw`
  - X (Twitter) 帳號 `@openclaw` 獲得金色認證
  - npm 套件發布
  - 文檔遷移至 `docs.openclaw.ai`
  - 90 分鐘內 20 萬+ 瀏覽量

### 「搶註者」風波

命名變更過程中出現了戲劇性的搶註事件：

1. **Twitter/X 帳號搶註：** 機器人在改名瞬間搶走 `@openclaw`，並發布加密貨幣錢包地址
2. **GitHub 帳號失誤：** Peter 誤將個人 GitHub 帳號改名，`steipete` 在幾分鐘內被搶註
3. **「帥氣 Molty」事件：** AI 生成圖示時產生了「龍蝦身上人臉」的怪異圖片，加密騙子迅速將其變成迷因
4. **假開發者詐騙：** 騙子創建假 GitHub 個人檔案聲稱「OpenClaw 工程主管」

### 社群誕生的新傳統

- *"The claw is the law"* 🤠
- *"Yee-claw"*
- *"Claw abiding citizens"*
- *"Clawntroversy"*

---

## 重大里程碑

### 版本演進總覽

| 版本 | 發布日期 | 重要性 |
|------|----------|--------|
| v2026.1.x | 2026年1月 | 安全修補、基礎功能 |
| v2026.2.x | 2026年2月 | SSRF 強化、安全邊界 |
| v2026.3.x | 2026年3月 | Task Flow、新平台 |
| v2026.4.x | 2026年4月 | 影片/音樂生成 |

---

### v2026.2.x（2026年2月）：安全強化期

**重點更新：**

- **v2026.2.12 (2月13日)：**
  - SSRF 防護強化
  - Cron 排程器大改版
  - 瀏覽器/網頁內容視為不受信任來源

- **v2026.2.15：**
  - 多 DM 支援
  - Android 秘密管理改進
  - 瀏覽器控制優化

- **v2026.2.23：**
  - HSTS 標頭支援
  - 配置脫敏行為
  - Gateway 信任和 Webhook 認證漏洞修補
  - 模型控制 token 從用戶文本中移除

---

### v2026.3.x（2026年3月）：功能擴展期

**v2026.3.22（3月23日）：重大更新**

- **45+ 新功能**
- **13 個破壞性變更**
- **82 個錯誤修復**
- **20 個安全修補**
- **15 個效能改進**

**核心新增：**

1. **Chrome DevTools MCP attach mode**
2. **批量瀏覽器操作**
3. **Docker 時區覆蓋**
4. **QQ Bot 支援**
5. **Matrix 串流**

**v2026.3.31：統一後台任務管理**

- Task Flow 基礎架構
- 受管對同步模式
- 持久化流程狀態追蹤
- `openclaw flows` 檢查/恢復原語

---

### v2026.4.x（2026年4月）：多媒體時代

**v2026.4.5（4月6日）：最大功能更新**

#### 🎬 內建影片生成
- **video_generate 工具：** 代理可直接建立影片
- **支援供應商：**
  - xAI (grok-imagine-video)
  - Alibaba Model Studio Wan
  - Runway
- 配置後直接在回覆中返回生成的媒體

#### 🎵 內建音樂生成
- **music_generate 工具：**
  - Google Lyria
  - MiniMax
  - Comfy 工作流程支援
- 非同步任務追蹤
- 完成後自動送達音訊

#### 🌐 多語言控制介面
支援語言：
- 簡體中文
- 繁體中文
- 巴西葡萄牙語
- 德語
- 西班牙語
- 日語
- 韓語
- 法語
- 土耳其語
- 印尼語
- 波蘭語
- 烏克蘭語

#### 🧠 記憶做夢系統（實驗性）

- **三階段睡眠模式：**
  - Light（淺睡）
  - Deep（深睡）
  - REM（快速動眼）
- **Dreams UI**
- **/dreaming 指令**
- 加權短期記憶晉升
- 多語言概念標記

#### 🔐 執行批准強化

- **iOS：** APNs 通用批准通知
- **Matrix：** 原生執行批准提示
- **before_tool_call hooks：** 允許暂停工具執行並請求用戶批准

#### 📦 新供應商

- Qwen
- Fireworks AI
- StepFun
- MiniMax TTS
- Ollama Web Search
- Amazon Bedrock Mantle

---

## Telegram Channel 更新高光

### 功能支援演進

| 時間 | 功能 | 說明 |
|------|------|------|
| 2026年1月 | Inline Buttons | 執行批准按鈕支援 |
| 2026年2月 | Topic Routing | 主題路由功能 |
| 2026年3月 | Media Download | 媒體下載 SSRF 防護 |
| 2026年4月 | Native Command Menu | 原生命令選單 |

### 🔥 Telegram 高光時刻

#### 1. Inline Button for Exec Approvals (Issue #22078)

**問題：** Discord 已支援按鈕式執行批准，但 Telegram 僅提供純文字，用戶需手動輸入 `/approve allow-once|allow-always|deny`。

**解決方案：** 新增 Telegram inline button 支援，讓執行批准更直觀。

```yaml
# 配置範例
channels:
  telegram:
    accounts:
      main:
        capabilities:
          inlineButtons: "all"  # 或 "allowlist"
```

#### 2. Session Transcript Path 錯誤修復 (Issue #15451)

**v2026.2.12 問題：** 多代理 Telegram 訊息被靜默丟棄。

**原因：** session path validation 過度嚴格。

**修復：** 2026.2.14 版本中放寬驗證邏輯。

#### 3. Media Download SSRF 問題 (Issue #57452)

**問題：** Telegram CDN 檔案伺服器解析到 RFC 2544 基準範圍 IP 時，SSRF 檢查會阻擋下載。

**對比：** Discord 已設定 `allowRfc2544BenchmarkRange: true`。

**影響：** 大檔案下載失敗。

#### 4. Telegram Native Command Menu (v2026.4.5)

新功能：
- 長描述自動修剪
- 子 100 命令集符合 Telegram payload 限制
- 更多 / 指令可見

### Telegram 配置範例

```yaml
channels:
  telegram:
    accounts:
      main:
        token: "YOUR_BOT_TOKEN"
        capabilities:
          inlineButtons: "all"  # 啟用 inline buttons
        onboarding:
          silentWelcome: false
        contextVisibility: "allowlist"
```

---

## Discord Channel 更新高光

### 功能支援演進

| 時間 | 功能 | 說明 |
|------|------|------|
| 2026年1月 | Button Exec Approvals | 首個支援的平台 |
| 2026年2月 | Thread Context | 執行緒上下文過濾 |
| 2026年3月 | Block Streaming | 區塊串流支援 |
| 2026年4月 | Voice Timeout Split | 語音連接優化 |

### 🔥 Discord 高光時刻

#### 1. 首創 Button-Based Exec Approvals

Discord 是第一個支援互動式執行批准的平台：

- 無需手動輸入 `/approve` 指令
- 一鍵允許/拒絕
- 支援 `allow-once` / `allow-always` / `deny`

#### 2. Plugin State Corruption (Issue #12196)

**問題：** Discord plugin 運行時損壞，錯誤訊息：`Cannot read properties of undefined (reading '0')`。

**觸發條件：**
- 新增 API 供應商（Moonshot, ElevenLabs 等）
- 長時間運行

**解決：** 與 Telegram plugin 共享的底層 plugin 框架修復。

#### 3. Duplicate Proactive Messages (Issue #40545)

**問題：** 當 Discord 和 Telegram 同時啟用時，cron/heartbeat 主動訊息重複送達。

**原因：** 多通道路由決策邏輯歧異。

**預期行為：** 每個主動更新只應送達一次。

#### 4. v2026.4.5 Discord 優化

- REST、webhook、監控流量保持在配置的代理上
- 保留 component-only 媒體發送
- 遵守 `@everyone` 和 `@here` 提及閘門
- ACK 反應保持在活躍帳號上
- 分離語音 connect/playback 超時

### Discord 配置範例

```yaml
channels:
  discord:
    accounts:
      main:
        token: "YOUR_BOT_TOKEN"
        applicationId: "YOUR_APP_ID"
        guilds:
          - id: "YOUR_GUILD_ID"
        capabilities:
          reactions: true
          threads: true
        contextVisibility: "allowlist"
```

---

## 安全事件記錄

OpenClaw 的發展過程伴隨著多起安全事件，這些事件深刻影響了專案的安全設計。

### CVE 統計（2026年）

| CVE ID | CVSS | 發布日期 | 說明 |
|--------|------|----------|------|
| CVE-2026-21852 | 5.3 | 2026年2月 | API key 竊取 |
| CVE-2026-25253 | 8.8 | 2026年2月 | 一鍵 RCE |
| CVE-2026-28460 | 高 | 2026年3月 | 允許清單繞過 |
| CVE-2026-32922 | 高 | 2026年3月 | 權限提升 |

### 🔴 ClawHavoc 供應鏈攻擊（2026年2月13日）

**概要：**
- **341+ 惡意技能** 透過 ClawHub 市場分發
- 部署 **Atomic Stealer (AMOS)** infostealer
- **Vidar infostealer** 變種針對 OpenClaw 代理身份

**攻擊手法：**
- 惡意 repository 配置檔
- 遠端程式碼執行
- API key 竊取

**影響：**
- 1,184 個惡意技能被發現（後續調查）
- 攻擊者從針對運行時，轉向針對生態系

### 🔴 CVE-2026-25253：一鍵 RCE

**嚴重性：** CVSS 8.8

**漏洞：** 透過惡意連結可劫持 OpenClaw AI 助理。

**修補版本：** v2026.1.29

**官方回應：**
> *"OpenClaw 的核心安全聲明之一 — 人在迴路中批准模式 — 存在根本性的實作缺口。"*

### 🔴 2026年3月的「CVE 洪水」

**Nine CVEs in Four Days**

在 2026 年 3 月的一週內，連續披露 9 個 CVE：

- CVE-2026-28460：使用 shell 換行字元繞過允許清單
- 其他漏洞涉及不同的安全邊界攻破

**教訓：**
- 人機互動批准機制需要更嚴格的實作
- 多層防禦的重要性

### 安全回應措施

OpenClaw 團隊採取了多項安全強化措施：

1. **v2026.2.x 安全強化：**
   - SSRF 防護
   - HSTS 標頭
   - 配置脫敏
   - 瀏覽器/網頁內容預設不受信任

2. **v2026.4.x 安全改進：**
   - 保留限制性 plugin-only 工具允許清單
   - /allowlist 新增/移除需要 owner 權限
   - before_tool_call hooks 當機時安全關閉
   - 瀏覽器 SSRF 重定向繞過早期阻擋
   - 非互動 auth-choice 推論僅限於捆綁和已信任 plugins

3. **VirusTotal 合作夥伴關係：**
   - 2026年2月宣布
   - 加強惡意程式碼偵測

---

## 創作者與團隊

### Peter Steinberger

- **身份：** PSPDFKit 創始人（經營13年）
- **國籍：** 奧地利
- **社群：** @steipete (GitHub/X)

### 2026年2月：加入 OpenAI

Peter 於 2026 年 2 月宣布加入 OpenAI：

> *"我是一個建構者。我已經玩過建立公司的遊戲，傾注了 13 年的生命並學到了很多。我想做的是改變世界，而不是建立一個大公司。與 OpenAI 合作是將這一切帶給每個人的最快方式。"*

### OpenClaw 基金會

為確保 OpenClaw 保持開源並獨立發展：

- OpenAI 承諾讓 Peter 投入時間於專案
- OpenAI 已成為專案贊助商
- 正在建立獨立基金會
- 將繼續支援更多模型和公司

### 社群英雄（The Claw Crew）

根據官方 Lore，以下人物在「最終型態」遷移中做出重要貢獻：

- **ELU：** 創建了「THE CLAW IS THE LAW」西部旗幟 logo
- **Whurley (William Hurley)：** 量子運算先驅，製作 ASCII 藝術
- **Onur：** GitHub 管理，首位佩戴 affiliate 徽章
- **Shadow：** Discord vanity 安全，清除惡意軟體

---

## 未來展望

### Molty 的未來（官方路线圖）

根據 OpenClaw Lore，Molty 的未來可能包括：

- 🦿 **腿：** Reachy Mini 已訂購！
- 👂 **耳朵：** Brabble 語音守護程序開發中
- 🏠 **智慧家居控制：** KNX + openhue 整合
- 🌍 **世界統治：** 延伸目標

### 技術路線

1. **ACP (Agent Communication Protocol) 標準化**
2. **更多捆綁供應商支援**
3. **增強記憶系統**
4. **多媒體生成工作流程**

### ClawHub 生態系

- 持續擴展 Skills 市場
- 更嚴格的安全審核
- 社群驅動的技能開發

---

## 參考來源

### 官方資源

- [OpenClaw 官網](https://openclaw.ai/)
- [GitHub Repository](https://github.com/openclaw/openclaw)
- [官方文檔](https://docs.openclaw.ai/)
- [OpenClaw Lore](https://docs.openclaw.ai/start/lore)
- [ClawHub Skills Market](https://clawhub.ai)

###社群

- [Discord](https://discord.com/invite/clawd)
- [Reddit r/openclaw](https://www.reddit.com/r/openclaw/)

### 新聞報導

- CNBC: "From Clawdbot to Moltbot to OpenClaw" (2026-02-02)
- Mashable: "Clawdbot changes name to Moltbot" (2026-01-30)
- Peter Steinberger Blog: "OpenClaw, OpenAI and the future"

### 安全研究

- CrowdStrike: ClawHavoc 攻擊分析
- Kaspersky: "Key OpenClaw risks, Clawdbot, Moltbot" (2026-02-24)
- NVD: CVE-2026-25253, CVE-2026-21852, CVE-2026-28460, CVE-2026-32922

### GitHub Issues 參考

- #22078: Telegram inline button support
- #15451: Multi-agent Telegram messages dropped
- #57452: Telegram media download SSRF
- #12196: Discord plugin state corruption
- #40545: Duplicate proactive messages

---

*本報告由 Tina ✨ 於 2026年4月7日編製*

*"The claw is the law"* 🦞