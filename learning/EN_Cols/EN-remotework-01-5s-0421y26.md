# EN-remotework-01-5s-0421y26

## Remote Work English for Mobile Developers (iOS/Android)

**Target:** 遠距工作的手機開發者，需要清晰溝通需求  
**Focus:** 5 個最常用的英文工作用語

---

## 1. "I'll look into this and get back to you."

**中文：** 我會調查一下，然後回覆你。

**使用場景：**
- 有人回報 bug 或提出問題，你需要時間調查
- Promise/Async 問題需要 debug
- 不確定問題根因時

**例句：**
> Q: "The app crashes on iOS 17 when opening the camera."
> A: "Thanks for reporting. **I'll look into this and get back to you** by EOD."

---

## 2. "Could you clarify the expected behavior?"

**中文：** 可以請你說明預期的行為嗎？

**使用場景：**
- Requirement 不夠清楚
- UI/UX 規格需要確認
- 避免「做錯重做」的情況

**例句：**
> "The button should scroll with the content, but stay fixed at the top."
> "**Could you clarify the expected behavior?** Should it disappear when scrolling down and reappear when scrolling up, or stay visible the whole time?"

---

## 3. "I'm blocked on [X], need your input."

**中文：** 我卡在 [X] 了，需要你的 input。

**使用場景：**
- Daily standup 報告障礙
- 等待 API、Design Review、或其他團隊的回應
- Remote 工作中主動尋求幫助

**例句：**
> "**I'm blocked on the API endpoint** for user authentication. The backend team hasn't deployed the new version yet."

---

## 4. "Let me sync with the team and circle back."

**中文：** 讓我跟團隊同步一下，再回覆你。

**使用場景：**
- 需要和其他 iOS/Android 開發者討論
- 需要 PM 或 Designer 確認
- 無法當場給答案時

**例句：**
> Q: "Can we ship this feature next week?"
> A: "**Let me sync with the team and circle back.** There are some platform-specific considerations we need to discuss."

---

## 5. "This is ready for review."

**中文：** 這個準備好可以 review 了。

**使用場景：**
- PR (Pull Request) 提交後通知團隊
- Code Review 請求
- QA 測試準備

**例句：**
> "**This is ready for review.** I've implemented the new login flow for both iOS and Android. The PR is linked in the ticket."

---

## 完整對話範例

### 對話 1：Bug 回報與調查

**情境：** QA 在 Slack 回報一個 iOS crash，你需要時間調查。

> **QA:** Hey, I found a critical bug. The app crashes on iOS 17 when opening the camera from the profile page.
>
> **You:** Thanks for reporting. Can you share the crash log and the steps to reproduce?
>
> **QA:** Sure, here's the crash log. Steps: 1) Open app 2) Go to profile 3) Tap camera icon 4) App crashes immediately.
>
> **You:** Got it. **I'll look into this and get back to you by EOD.** This might be related to the new iOS 17 privacy permissions.
>
> **QA:** Sounds good. Let me know if you need more info.
>
> *(Later that day...)*
>
> **You:** @QA I found the issue. We're missing the new `NSCameraSecure主题活动AccessDescription` key in Info.plist for iOS 17. I've pushed a fix to the `fix/camera-crash-ios17` branch. Can you test the new build?
>
> **QA:** Testing now... Looks good! No crash on iOS 17 anymore.

---

### 對話 2：需求確認

**情境：** PM 給了一個模糊的 UI 需求，你需要釐清細節。

> **PM:** For the new search feature, the search bar should be sticky at the top, but also scroll with the content on the home screen.
>
> **You:** **Could you clarify the expected behavior?** I want to make sure I understand correctly.
>
> **PM:** Hmm, let me think...
>
> **You:** Here are some options:
> 1. The search bar stays at the top always (like Instagram)
> 2. It scrolls away when scrolling down, and reappears when scrolling up ( like Google Maps)
> 3. It's part of the content and scrolls away completely
>
> Which one matches your vision?
>
> **PM:** Option 2! Like Google Maps. When the user scrolls down to browse content, the search bar hides to give more space. When they scroll up, it reappears.
>
> **You:** Perfect. One more question - should this behavior be the same on Android and iOS?
>
> **PM:** Yes, consistent across both platforms. Thanks for asking!

---

### 對話 3：Standup 報告障礙

**情境：** Daily standup 會議中，你被 API 問題卡住。

> **Scrum Master:** Okay, let's go around. Alex, what did you work on yesterday and what's your plan for today?
>
> **You:** Yesterday I finished the user profile UI for iOS. Today I planned to integrate the profile API, but **I'm blocked on the API endpoint for avatar upload.** The backend team mentioned it would be ready yesterday, but it's still returning 404.
>
> **Scrum Master:** Got it. @Backend Lead, can you provide an update?
>
> **Backend Lead:** Sorry for the delay. We had some issues with the image compression service. The endpoint should be deployed in the next 2 hours.
>
> **You:** Thanks for the update. In the meantime, I'll work on the offline caching feature so I'm not sitting idle.
>
> **Scrum Master:** Good plan. Let us know if you're still blocked after lunch.

---

### 對話 4：跨團隊協調

**情境：** PM 問你下週能否交付某個功能，但你需要跟團隊討論。

> **PM:** Hey, the client is asking if we can ship the push notification feature by next Friday. Do you think that's feasible?
>
> **You:** That's a tight timeline. **Let me sync with the team and circle back.** There are some platform-specific considerations we need to discuss:
> - iOS needs APNs certificate setup
> - Android needs FCM configuration
> - We also need to decide on the backend architecture
>
> **PM:** Sure, when can you give me an answer?
>
> **You:** I'll discuss it in our sync meeting this afternoon and get back to you by tomorrow morning.
>
> *(Next morning...)*
>
> **You:** @PM I've discussed with the team. Next Friday is doable for basic push notifications, but we'll need to:
> 1. Cut scope to MVP (no rich media in v1)
> 2. Prioritize this over the settings redesign
> 3. Get the APNs and FCM credentials from the client by Wednesday
>
> Does that work?
>
> **PM:** Let me confirm with the client. I'll get back to you.

---

### 對話 5：Code Review 請求

**情境：** 你完成了功能的實作，準備提交 PR。

> **You:** @team **This is ready for review.** I've implemented the new login flow for both iOS and Android.
>
> PR links:
> - iOS: #PR-234
> - Android: #PR-235
>
> Key changes:
> - Biometric authentication (Face ID / Fingerprint)
> - Token refresh logic
> - Error handling for network issues
>
> **Senior Dev:** I'll review the iOS PR. @Junior, can you take the Android one?
>
> **Junior:** Sure, I'll review today.
>
> *(After review...)*
>
> **Senior Dev:** Left some comments on the iOS PR. Mainly about the error handling in the token refresh - we should add retry logic. Otherwise looks good!
>
> **You:** Thanks! I'll add the retry logic and update the PR. Should be ready for merge by EOD.
>
> **Senior Dev:** Great. Once both PRs are approved, we can merge and deploy to staging.

---

## 額外：Mobile Development 常用術語

| 術語 | 說明 |
|------|------|
| **PR (Pull Request)** | 程式碼合併請求 |
| **Code Review** | 程式碼審查 |
| **QA (Quality Assurance)** | 品質保證測試 |
| **Deploy / Release** | 部署 / 上架 |
| **Hotfix** | 緊急修補 |
| **Build** | 編譯版本 |
| **APK / IPA** | Android / iOS 安裝檔 |
| **Crash log** | 當機記錄 |
| **Feature flag** | 功能開關（可動態啟用/停用功能）|
| **Backlog** | 待辦事項清單 |

---

## 使用建議

1. **非同步溝通 (Async communication)** 是 Remote 工作的常態
   - 用 "I'll get back to you" 買時間思考
   - 用 "Let me sync with the team" 避免承諾過快

2. **確認需求** 比「假設正確」更重要
   - 問清楚 expected behavior
   - Mobile 開發跨平台差異大（iOS/Android），先確認再實作

3. **主動回報** 是信任的基礎
   - 卡住要說 "I'm blocked on..."
   - 完成要說 "This is ready for review"

---

## Summary

| # | Phrase | 用途 |
|---|--------|------|
| 1 | I'll look into this and get back to you. | 調查後回覆 |
| 2 | Could you clarify the expected behavior? | 確認需求 |
| 3 | I'm blocked on [X], need your input. | 回報障礙 |
| 4 | Let me sync with the team and circle back. | 團隊同步後回覆 |
| 5 | This is ready for review. | 通知完成待審 |

---

*Created: 2026-04-21 by Tina*
*Reference: Remote work communication best practices for software developers*
