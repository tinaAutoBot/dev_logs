# OpenClaw 多代理系統 (Multi-Agent) 深度研究報告

**研究日期：** 2026-04-07  
**研究目標：** 深入分析 OpenClaw 的 multiAgentGroups 相關設定，理解多代理架構的設計理念、實作細節與最佳實務。

---

## 目錄

1. [執行摘要](#執行摘要)
2. [架構概覽](#架構概覽)
3. [核心概念](#核心概念)
4. [配置結構](#配置結構)
5. [綁定與路由機制](#綁定與路由機制)
6. [沙箱隔離與工具策略](#沙箱隔離與工具策略)
7. [最佳實務模式](#最佳實務模式)
8. [實際案例研究](#實際案例研究)
9. [常見問題與除錯](#常見問題與除錯)
10. [結論與建議](#結論與建議)

---

## 執行摘要

**重要發現：** OpenClaw 的配置系統中並不存在名為 `multiAgentGroups` 的頂層設定選項。經過深入檢查 config schema 與官方文件，多代理功能是透過以下三個核心配置區塊協作實現：

1. **`agents`** - 定義代理實體（id、workspace、model、tools、sandbox）
2. **`bindings`** - 定義路由規則，將訊息導向對應的代理
3. **`channels`** - 定義通道帳號，可與代理綁定

這是一個**去中心化的多代理架構**，而非分組管理架構。每個代理是獨立運作的「大腦」，透過綁定規則決定誰該處理哪些訊息。

---

## 架構概覽

### 單一 Gateway 多代理模型

```
┌─────────────────────────────────────────────────────────────┐
│                     OpenClaw Gateway                          │
│                                                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  Agent:main │  │Agent:family │  │ Agent:work  │         │
│  │             │  │             │  │             │         │
│  │ workspace/  │  │workspace-   │  │workspace-   │         │
│  │ SOUL.md     │  │ family/     │  │ work/       │         │
│  │ sessions/   │  │ sessions/   │  │ sessions/   │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│         ▲                ▲                ▲                  │
│         │                │                │                  │
│  ┌──────┴────────────────┴────────────────┴──────┐         │
│  │                  Bindings                       │         │
│  │  (路由規則: channel + accountId + peer → agent) │         │
│  └─────────────────────────────────────────────────┘         │
│                           ▲                                  │
│                           │                                  │
│  ┌────────────────────────┴────────────────────────┐        │
│  │                   Channels                        │        │
│  │  WhatsApp │ Telegram │ Discord │ Slack │ ...    │        │
│  └──────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

### 關鍵設計理念

| 特性 | 說明 |
|------|------|
| **完整隔離** | 每個代理擁有獨立的 workspace、sessions、auth profiles |
| **決定性路由** | 綁定規則精確、可預測，most-specific-wins |
| **單一程序** | 所有代理運行於同一 Gateway process，資源效率高 |
| **彈性工具策略** | 每個代理可有獨立的工具允許/拒絕清單 |

---

## 核心概念

### 什麼是「一個代理」？

一個代理 (Agent) 是一個**完整作用域的「大腦」**，擁有：

```
~/.openclaw/
├── workspace-<agentId>/        # 工作目錄
│   ├── AGENTS.md               # 行為規則
│   ├── SOUL.md                 # 人格設定
│   ├── USER.md                 # 使用者資訊
│   ├── MEMORY.md               # 長期記憶
│   ├── memory/                 # 每日記憶
│   └── skills/                 # 專屬技能
│
└── agents/<agentId>/
    ├── agent/                  # 代理狀態目錄
    │   └── auth-profiles.json  # 認證配置
    └── sessions/               # 對話歷史
```

### Agent ID 與 Session Key

- **`agentId`**：代理的唯一識別碼（如 `main`, `work`, `family`）
- **Session Key 格式**：`agent:<agentId>:<mainKey>`
- **Main Session**：直接對話的預設 session，key 為 `main`

### 認證隔離

```
每個代理讀取自己的認證檔案：
~/.openclaw/agents/<agentId>/agent/auth-profiles.json

⚠️ 認證不會自動共享！
如需共享，需手動複製 auth-profiles.json 到另一個代理的 agentDir。
```

---

## 配置結構

### 完整 Schema 結構

```json5
{
  "agents": {
    "defaults": {
      // 代理的預設設定（可被 list 中的個別代理覆蓋）
      "model": { "primary": "string" },
      "workspace": "string",
      "sandbox": {
        "mode": "off" | "all" | "non-main",
        "scope": "agent" | "session" | "shared"
      },
      "tools": { "allow": [], "deny": [] },
      "thinkingDefault": "on" | "off" | "stream",
      "contextTokens": 128000,
      "heartbeat": { "intervalMs": 1800000 },
      // ... 更多設定
    },
    
    "list": [
      {
        "id": "string (required)",         // 代理 ID
        "default": true,                   // 是否為預設代理
        "name": "string",                  // 顯示名稱
        "workspace": "string",             // 工作目錄路徑
        "agentDir": "string",              // 代理狀態目錄
        "model": { "primary": "string" },  // 代理專屬模型
        "thinkingDefault": "string",
        "reasoningDefault": "string",
        "fastModeDefault": true,
        "skills": ["skill1", "skill2"],    // 技能過濾清單
        "memorySearch": { "enabled": true },
        "humanDelay": { "minMs": 1000 },
        "heartbeat": { "intervalMs": 1800000 },
        "identity": { "name": "string" },
        "groupChat": {
          "mentionPatterns": ["@agent1", "@agentBot"]
        },
        "subagents": {
          "allow": ["agent2", "agent3"],
          "defaultModel": "string"
        },
        "sandbox": {
          "mode": "off" | "all" | "non-main",
          "scope": "agent" | "session" | "shared",
          "docker": {
            "setupCommand": "string"
          }
        },
        "params": {                        // 傳遞給模型的額外參數
          "temperature": 0.7
        },
        "tools": {
          "profile": "string",             // 工具設定檔名稱
          "allow": ["read", "exec"],
          "deny": ["browser", "gateway"]
        },
        "runtime": {
          "kind": "embedded" | "acp"
        }
      }
    ]
  }
}
```

### 關鍵配置屬性詳解

#### `agents.defaults` vs `agents.list[]`

| 層級 | 用途 | 覆蓋規則 |
|------|------|----------|
| `defaults` | 定義所有代理的基準設定 | 作為 fallback |
| `list[].*` | 定義個別代理的覆蓋設定 | 完全覆蓋 defaults 中對應的屬性 |

**範例：**

```json5
{
  "agents": {
    "defaults": {
      "sandbox": { "mode": "non-main" },
      "tools": { "allow": ["read", "write", "exec"] }
    },
    "list": [
      {
        "id": "main",
        "sandbox": { "mode": "off" },  // 覆蓋 defaults.mode
        // tools 繼承 defaults.tools
      },
      {
        "id": "public",
        // sandbox 繼承 defaults.sandbox (non-main)
        "tools": { "allow": ["read"] }  // 完全覆蓋 defaults.tools
      }
    ]
  }
}
```

#### `sandbox.mode` 選項

| 值 | 行為 |
|----|------|
| `"off"` | 永不沙箱化，直接在 host 執行 |
| `"all"` | 永遠沙箱化，所有操作在 Docker 容器中 |
| `"non-main"` | 僅非 main session 沙箱化（group/channel 會被沙箱化） |

⚠️ **常見陷阱：** `"non-main"` 是基於 `session.mainKey`（預設 `"main"`），而非 `agentId`。Group/channel sessions 有自己的 key，所以會被沙箱化。

#### `sandbox.scope` 選項

| 值 | 行為 |
|----|------|
| `"agent"` | 每個代理一個專屬容器 |
| `"session"` | 每個 session 一個容器（預設） |
| `"shared"` | 所有代理共用一個容器 |

---

## 綁定與路由機制

### 綁定結構

```json5
{
  "bindings": [
    {
      "agentId": "string",          // 目標代理 ID
      "match": {
        "channel": "string",        // 通道類型 (whatsapp, telegram, discord...)
        "accountId": "string",      // 帳號 ID（可用 "*" 表示所有帳號）
        "peer": {                   // 特定對象
          "kind": "direct" | "group" | "channel",
          "id": "string"
        },
        "guildId": "string",        // Discord guild
        "teamId": "string",         // Slack team
        "roles": ["role1"]          // Discord role 過濾
      }
    }
  ]
}
```

### 路由優先順序（Most-Specific-Wins）

綁定是**決定性**的，優先順序如下：

1. **`peer` 完全匹配**（特定 DM/Group/Channel）
2. **`parentPeer` 匹配**（thread 繼承）
3. **`guildId + roles` 匹配**（Discord）
4. **`guildId` 匹配**（Discord）
5. **`teamId` 匹配**（Slack）
6. **`accountId` 匹配**
7. **`accountId: "*"` 匹配**（通道層級 fallback）
8. **預設代理**（`agents.list[].default: true` 或第一個条目或 `main`）

**同步匹配規則：** 如果一個綁定設定了多個 match 欄位（如 `peer` + `guildId`），所有欄位都必須匹配（AND 語意）。

### 綁定範例

#### WhatsApp 多帳號對多代理

```json5
{
  "agents": {
    "list": [
      { "id": "home", "default": true },
      { "id": "work" }
    ]
  },
  "bindings": [
    { "agentId": "home", "match": { "channel": "whatsapp", "accountId": "personal" } },
    { "agentId": "work", "match": { "channel": "whatsapp", "accountId": "biz" } },
    // 特定群組路由到 work（放在前面，優先匹配）
    {
      "agentId": "work",
      "match": {
        "channel": "whatsapp",
        "accountId": "personal",
        "peer": { "kind": "group", "id": "1203630...@g.us" }
      }
    }
  ]
}
```

#### WhatsApp 單一帳號多使用者 DM 分流

```json5
{
  "agents": {
    "list": [
      { "id": "alex", "workspace": "~/.openclaw/workspace-alex" },
      { "id": "mia", "workspace": "~/.openclaw/workspace-mia" }
    ]
  },
  "bindings": [
    {
      "agentId": "alex",
      "match": { "channel": "whatsapp", "peer": { "kind": "direct", "id": "+15551230001" } }
    },
    {
      "agentId": "mia",
      "match": { "channel": "whatsapp", "peer": { "kind": "direct", "id": "+15551230002" } }
    }
  ],
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551230001", "+15551230002"]
    }
  }
}
```

#### Telegram 多 Bot 多代理

```json5
{
  "bindings": [
    { "agentId": "main", "match": { "channel": "telegram", "accountId": "default" } },
    { "agentId": "alerts", "match": { "channel": "telegram", "accountId": "alerts" } }
  ],
  "channels": {
    "telegram": {
      "accounts": {
        "default": {
          "botToken": "123456:ABC...",
          "dmPolicy": "pairing"
        },
        "alerts": {
          "botToken": "987654:XYZ...",
          "dmPolicy": "allowlist",
          "allowFrom": ["tg:123456789"]
        }
      }
    }
  }
}
```

---

## 沙箱隔離與工具策略

### 沙箱配置層級

```
agents.list[].sandbox.* > agents.defaults.sandbox.*
```

| 配置 | 覆蓋規則 |
|------|----------|
| `sandbox.mode` | 代理設定覆蓋預設 |
| `sandbox.scope` | 代理設定覆蓋預設 |
| `sandbox.docker.*` | 代理設定覆蓋預設（當 scope 為 "shared" 時忽略） |
| `sandbox.browser.*` | 代理設定覆蓋預設 |
| `sandbox.prune.*` | 代理設定覆蓋預設 |

### 工具策略過濾順序

```
1. Tool profile (tools.profile 或 agents.list[].tools.profile)
   ↓
2. Provider tool profile (tools.byProvider[provider].profile)
   ↓
3. Global tool policy (tools.allow / tools.deny)
   ↓
4. Provider tool policy (tools.byProvider[provider].allow/deny)
   ↓
5. Agent-specific tool policy (agents.list[].tools.allow/deny)
   ↓
6. Agent provider policy (agents.list[].tools.byProvider[provider].allow/deny)
   ↓
7. Sandbox tool policy (tools.sandbox.tools 或 agents.list[].tools.sandbox.tools)
   ↓
8. Subagent tool policy (tools.subagents.tools)
```

**關鍵規則：** 每一層只能進一步限制工具，不能恢復已被前面層級拒絕的工具。

### 工具策略範例

#### 唯讀代理

```json5
{
  "agents": {
    "list": [{
      "id": "readonly",
      "tools": {
        "allow": ["read"],
        "deny": ["exec", "write", "edit", "apply_patch", "process"]
      }
    }]
  }
}
```

#### 安全執行代理（無檔案修改）

```json5
{
  "agents": {
    "list": [{
      "id": "safe-exec",
      "tools": {
        "allow": ["read", "exec", "process"],
        "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
      }
    }]
  }
}
```

#### 通訊專用代理

```json5
{
  "agents": {
    "list": [{
      "id": "messaging",
      "tools": {
        "sessions": { "visibility": "tree" },
        "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
        "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
      }
    }]
  }
}
```

---

## 最佳實務模式

### 何時需要多代理？

| 需求 | 建議 |
|------|------|
| 多通道但相同行為 | 單一代理 + 多通道綁定 |
| 不同工具策略（安全考量） | 多代理 + 獨立 tools allow/deny |
| 不同人格/記憶 | 多代理 + 獨立 workspace |
| 不同模型（成本���量） | 多代理 + 獨立 model 設定 |
| 多使用者隔離 | 多代理 + DM peer 綁定 |

### 配置建議

1. **保持 ID 穩定**：綁定、審批、session 路由都依賴 `agentId`
2. **最特定放前面**：綁定陣列中，把最精確的 peer 匹配放在前面
3. **明確 workspace 路徑**：避免相對路徑造成混淆
4. **避免共用 agentDir**：會導致 auth/session 衝突
5. **使用 `openclaw agents add` 精靈**：自動建立乾淨的 workspace 和 agentDir 結構

### 驗證命令

```bash
# 列出代理與綁定
openclaw agents list --bindings

# 驗證通道狀態
openclaw channels status --probe

# 檢查沙箱容器
docker ps --filter "name=openclaw-sbx-"

# 監控路由日誌
tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
```

---

## 實際案例研究

### 案例 1：家族 WhatsApp 群組機器人

**需求：** 在家族 WhatsApp 群組中運行一個嚴格限制的代理，只能讀取，不能執行命令或修改檔案。

```json5
{
  "agents": {
    "list": [
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/.openclaw/workspace-family",
        "identity": { "name": "Family Bot" },
        "groupChat": {
          "mentionPatterns": ["@family", "@familybot"]
        },
        "sandbox": { "mode": "all", "scope": "agent" },
        "tools": {
          "allow": ["read", "sessions_list", "sessions_history"],
          "deny": ["write", "edit", "exec", "browser", "gateway", "cron"]
        }
      }
    ]
  },
  "bindings": [{
    "agentId": "family",
    "match": {
      "channel": "whatsapp",
      "peer": { "kind": "group", "id": "120363999999999999@g.us" }
    }
  }]
}
```

### 案例 2：個人 + 工作雙代理

**需求：** 個人助理使用 OpenAI GPT-5.2，工作助理使用 Claude Opus，分別綁定到不同的 Telegram bot。

```json5
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/.openclaw/workspace-personal",
        "model": { "primary": "openai/gpt-5.2" },
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "name": "Work Assistant",
        "workspace": "~/.openclaw/workspace-work",
        "model": { "primary": "anthropic/claude-opus-4-6" },
        "sandbox": { "mode": "non-main" }
      }
    ]
  },
  "bindings": [
    { "agentId": "personal", "match": { "channel": "telegram", "accountId": "personal" } },
    { "agentId": "work", "match": { "channel": "telegram", "accountId": "work" } }
  ],
  "channels": {
    "telegram": {
      "accounts": {
        "personal": { "botToken": "PERSONAL_BOT_TOKEN" },
        "work": { "botToken": "WORK_BOT_TOKEN" }
      }
    }
  }
}
```

### 案例 3：Solo Founder 多代理團隊（4 Agents 協作）

**架構：**
- **Milo (Team Lead)**: Claude Opus，負責策略規劃
- **Josh (Business)**: Claude Sonnet，負責商業分析
- **Marketing**: Gemini，負責行銷研究
- **Dev**: Claude Opus / Codex，負責開發

**特色：**
- 透過單一 Telegram 群組控制，使用 @mention 觸發特定代理
- 共用記憶（GOALS.md, DECISIONS.md）+ 私有記憶
- 定時任務：每日 morning standup、metrics pull、content ideation

**配置片段：**

```json5
{
  "agents": {
    "list": [
      {
        "id": "milo",
        "name": "Milo - Team Lead",
        "model": { "primary": "anthropic/claude-opus-4-6" },
        "workspace": "~/.openclaw/workspace-milo",
        "groupChat": { "mentionPatterns": ["@milo"] }
      },
      {
        "id": "josh",
        "name": "Josh - Business",
        "model": { "primary": "anthropic/claude-sonnet-4-6" },
        "workspace": "~/.openclaw/workspace-josh",
        "groupChat": { "mentionPatterns": ["@josh"] }
      },
      {
        "id": "marketing",
        "name": "Marketing",
        "model": { "primary": "google/gemini-2.0-flash" },
        "workspace": "~/.openclaw/workspace-marketing",
        "groupChat": { "mentionPatterns": ["@marketing"] }
      },
      {
        "id": "dev",
        "name": "Dev",
        "model": { "primary": "anthropic/claude-opus-4-6" },
        "workspace": "~/.openclaw/workspace-dev",
        "tools": {
          "allow": ["read", "write", "edit", "exec", "process"],
          "deny": ["browser", "gateway"]
        },
        "groupChat": { "mentionPatterns": ["@dev"] }
      }
    ]
  }
}
```

---

## 常見問題與除錯

### Q1: 設定了 `sandbox.mode: "all"` 但代理未被沙箱化

**原因：** 可能有 `agents.defaults.sandbox.mode` 或綁定解析錯誤。

**解決：**
```bash
# 檢查日誌
tail -f ~/.openclaw/logs/gateway.log | grep "sandbox"

# 明確設定代理的 sandbox.mode
```

### Q2: 工具仍可用於 deny 清單

**原因：** 工具過濾是累積的，每一層只能進一步限制。

**解決：**
```bash
# 檢查過濾鏈
tail -f ~/.openclaw/logs/gateway.log | grep "tools filtering"

# 確認沒有上層 allow 該工具
```

### Q3: 容器未按代理隔離

**原因：** `scope` 預設為 `"session"`。

**解決：**
```json5
{
  "sandbox": { "scope": "agent" }
}
```

### Q4: 代理間無法通訊

**原因：** `tools.agentToAgent` 預設關閉。

**解決：**
```json5
{
  "tools": {
    "agentToAgent": {
      "enabled": true,
      "allow": ["agent1", "agent2"]
    }
  }
}
```

### Q5: DM 分流失效，所有訊息都到同一個代理

**原因：** 綁定順序問題，或 peer ID 格式錯誤。

**解決：**
```bash
# 列出綁定
openclaw agents list --bindings

# 確保 peer binding 在 channel-wide binding 之前
# 確認 peer ID 格式正確（E.164 格式，如 "+15551234567"）
```

---

## 結論與建議

### 核心發現

1. **無 `multiAgentGroups` 配置**：OpenClaw 採用 `agents` + `bindings` 的扁平化架構，而非分組管理架構。

2. **完整隔離**：每個代理是獨立的「大腦」，擁有獨立的 workspace、sessions、auth、tools、memory。

3. **決定性路由**：綁定規則精確、可預測，最特定匹配優先。

4. **彈性安全策略**：支援多層次的沙箱與工具限制，可實現細緻的權限控制。

### 適用場景

| 場景 | 推薦配置 |
|------|----------|
| 個人多通道助理 | 單一代理 + 多通道綁定 |
| 家用共享（多人隔離） | 多代理 + DM peer 綁定 |
| 工作與個人分離 | 多代理 + 不同 channel/account |
| 安全限制的公用機器人 | 獨立代理 + 嚴格 tools deny + sandbox |
| 專案團隊協作 | 多代理 + shared memory + 定時任務 |

### 下一步行動

1. **評估需求**：是否真的需要多代理？單一代理是否足夠？
2. **規劃架構**：繪製代理、通道、綁定的關係圖。
3. **使用精靈**：`openclaw agents add <id>` 減少配置錯誤。
4. **測試驗證**：使用 `openclaw agents list --bindings` 確認配置。
5. **監控日誌**：特別注意 routing、sandbox、tools 相關訊息。

---

## 參考資源

- [OpenClaw Multi-Agent Routing 官方文件](https://docs.openclaw.ai/concepts/multi-agent)
- [Multi-Agent Sandbox & Tools 配置](https://docs.openclaw.ai/tools/multi-agent-sandbox-tools)
- [Sandboxing 完整文件](https://docs.openclaw.ai/gateway/sandboxing)
- [Sandbox vs Tool Policy vs Elevated](https://docs.openclaw.ai/gateway/sandbox-vs-tool-policy-vs-elevated)
- [OpenClaw Configuration Reference](https://docs.openclaw.ai/gateway/configuration-reference)
- [Lumadock: OpenClaw Multi-Agent Setup Tutorial](https://lumadock.com/tutorials/openclaw-multi-agent-setup)

---

**報告撰寫者：** Tina ✨  
**最後更新：** 2026-04-07