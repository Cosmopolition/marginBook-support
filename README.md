# marginBook-support

MarginBook 的**隐私政策**与**技术支持**静态站点。

- 站点源码属于**独立 Git 仓库**，通过 GitHub Pages 发布。
- 纯静态：无框架、无构建步骤。根目录三页按 `navigator.language` 分流到对应语言目录。
- 四种语言：简体中文 / English / 日本語 / 한국어。

---

## 线上地址（App Store Connect 里填这两条）

| 用途 | URL |
|---|---|
| Privacy Policy URL | `https://cosmopolition.github.io/marginBook-support/privacy.html` |
| Support URL | `https://cosmopolition.github.io/marginBook-support/support.html` |

> ★ 根目录的 `privacy.html` / `support.html` 会**按访问者的浏览器语言**自动跳到
> `zh/` `en/` `ja/` `ko/` 里的对应页面 ⇒ App Store Connect 里只需要填**这一条根路径**，
> 不用为四种语言各建一条 URL。
>
> ★ 商店描述正文末尾也用这两条根路径（读者点进去时按浏览器语言自动进对应语言）。

---

## 目录结构

```
support/
├── index.html          # 根：按浏览器语言 → <lang>/support.html
├── privacy.html        # 根：按浏览器语言 → <lang>/privacy.html
├── support.html        # 根：按浏览器语言 → <lang>/support.html
├── css/
│   └── style.css       # 共享样式（配色取 App 的 Theme token，跟随系统深浅色）
├── en/  ja/  ko/  zh/
│   ├── privacy.html
│   └── support.html
└── README.md
```

---

## 部署步骤（GitHub Pages）

1. 仓库设为 **public**。
2. `Settings → Pages → Build and deployment → Source` 选 `Deploy from a branch`，
   分支选 `main`、目录选 `/ (root)` → **Save**。
3. 等 1～2 分钟，站点在 `https://cosmopolition.github.io/marginBook-support/` 上线。
4. 到 App Store Connect：
   - App 详情页 → **Privacy Policy URL** 填 `https://cosmopolition.github.io/marginBook-support/privacy.html`
   - App 详情页 → **Support URL** 填 `https://cosmopolition.github.io/marginBook-support/support.html`

> ⚠️ **Pages 没打开之前，根目录那三页会显示 404** —— 因为它们是纯静态文件，
> 必须有 GitHub Pages 才会被服务出来。先开 Pages，再去商店填 URL。

---

## 本地预览

```bash
cd support
python3 -m http.server 8000
# 打开 http://localhost:8000/            → 根分流页
#    http://localhost:8000/zh/privacy.html
#    http://localhost:8000/en/support.html
```

> 根分流页依赖 `navigator.language`，本地预览时可以直接打开 `<lang>/` 下的页面看具体语言。

---

## 结构守门人（改完必跑）

四语页面的结构必须 1:1（章节数 / 层级 / 表格行数 / FAQ 条数与顺序），
跨语言的关键事实串（两个 URL、邮箱、日期、版本号）也必须一致：

```bash
node ~/.minimax/skills/app-support-site/scripts/check_structure.mjs \
  --site <本目录> --base zh --langs zh,en,ja,ko --facts "2026-11-24,1.0.0"
```

退出码 0 = 全绿。**不许靠肉眼过。**

---

## 事实来源：每条「我们不做什么」的代码依据

隐私政策里每一句「我们不做 X」都对应一条可查的代码事实，改 App 时记得回头对：

| 政策里的说法 | 代码依据 |
|---|---|
| 不申请任何系统权限 | `apps/marginbook/App/Resources/Info.plist` 里**没有任何一条** `NS*UsageDescription` |
| 不做追踪 | `Info.plist` 的 `NSPrivacyTracking = false`、`NSPrivacyTrackingDomains = []` |
| 不收集数据 | `App/Resources/PrivacyInfo.xcprivacy` 的 `NSPrivacyCollectedDataTypes` 是空数组 |
| App 代码里没有网络请求 | `App/` 与 `Packages/LedgerCore/` 下**没有一处** `URLSession` / `URLRequest` / `http` |
| 没有第三方 SDK | `project.yml` 的 `packages:` 只有本地路径包 `LedgerCore` |
| 唯一联网是购买与恢复购买 | `App/Services/PurchaseManager.swift`：`AppStorePurchase.product(for:)`、`product.purchase()`、`Transaction.currentEntitlements` |
| 不碰照片 / 不读 EXIF | `App/` 下没有 `PhotosPicker` / `UIImagePickerController` / `UIImage` / EXIF 相关调用 |
| 数据存在沙盒 | `App/MarginBookApp.swift` 的 `ModelConfiguration(isStoredInMemoryOnly: false)`；`App/Models/LedgerRecord.swift` 的 `dateYear/dateMonth/dateDay` |
| 设置与买断状态存 UserDefaults | `App/Models/AppSettings.swift` 的 `Keys`（`settings.*` 九个键） |
| 导出产物落在临时目录 | `App/Services/ExportService.swift` 写 `FileManager.default.temporaryDirectory`，`SettingsView` 用系统分享面板交出去 |
| 记录与 CSV 永远免费 | `Packages/LedgerCore/Sources/LedgerCore/Paywall.swift`：`freeEntryLimit = nil`、`availableExports` 未买断只给 `.csv` |
| 买断解锁三样 | 同上 + `SettingsLayout.sections(isUnlocked:)`：未买断时设置页**根本没有**「记账」这一整组 |

> ★ **恢复购买也会联网**（`Transaction.currentEntitlements` 要与商店沙盒通信）。
> 所以政策里写的是「购买或恢复购买时」，不是「只有购买时」。

---

## 改这份站点时要注意的两件事

1. **站点政策与 App 内的隐私文案是两处。**
   App 内那一句在 `apps/marginbook/App/Resources/<lang>.lproj/Localizable.strings` 的
   `data_privacy_body`，用户在「设置 › 数据」里能看到。口径不一致时，审核员对照商店描述与
   App 内文案会发现矛盾 —— 改一处要同步另一处。

2. **事实不许改。** 价格、数量、日期、邮箱、URL、文件路径一律照抄；
   技术名词（`URLSession` / `PhotosPicker` / `StoreKit` / `EXIF` / `UserDefaults` 等）保留原文不翻译。
