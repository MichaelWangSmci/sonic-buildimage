# SONiC CPO 最新發展追蹤

> GitHub snapshot：2026-07-15 01:00 UTC  
> 搜尋範圍：`sonic-net` 主要 repos（SONiC、platform-common、platform-daemons、buildimage、utilities、mgmt、linux-kernel），包含 PR、issue、review、checks、linked work 與 default-branch code。

## 摘要

- 追蹤工作項目：約 25
- 存在 draft、conflict、CI 或 review 阻塞：約 16
- 活躍硬體線：
  - Bailly
  - Davisson（common library only；尚無公開 vendor consumer）
  - NVIDIA / Mellanox

已落地的關鍵規格與 release 節點：

- [SONiC #2211](https://github.com/sonic-net/SONiC/pull/2211)：`cpo.json` port mapping HLD 已合併
- [utilities #4647](https://github.com/sonic-net/sonic-utilities/pull/4647)：ELS CLI show 已合併
- [buildimage #28311](https://github.com/sonic-net/sonic-buildimage/pull/28311)：202605 已消費 Bailly CMIS common backport（#689）

仍未形成可依序合併的完整實作鏈：

- 通用 topology：[buildimage #28077](https://github.com/sonic-net/sonic-buildimage/pull/28077)、[platform-common #700](https://github.com/sonic-net/sonic-platform-common/pull/700) 名稱已對齊 HLD，但分別卡在 CI / draft
- `xcvrd`：[SONiC #2444](https://github.com/sonic-net/SONiC/pull/2444)、[daemons #843](https://github.com/sonic-net/sonic-platform-daemons/pull/843) 仍為 draft
- NVIDIA Joint Mode：[buildimage #28324](https://github.com/sonic-net/sonic-buildimage/pull/28324) 仍使用 `optical_devices.json`
- Bailly 202605 後續：[buildimage #28109](https://github.com/sonic-net/sonic-buildimage/pull/28109)、[#28140](https://github.com/sonic-net/sonic-buildimage/pull/28140) 已 approved 且 CI 全過，待 merge

## Cross-repository interface mismatch

[SONiC #2211](https://github.com/sonic-net/SONiC/pull/2211) 已將 `cpo.json`、`get_cpo_data()`、`construct_cpo_devices` 訂為正式規格。

通用實作鏈名稱已對齊，但尚未合併：

- [sonic-buildimage #28077](https://github.com/sonic-net/sonic-buildimage/pull/28077) — `get_cpo_data()` 解析 `cpo.json`
- [sonic-platform-common #700](https://github.com/sonic-net/sonic-platform-common/pull/700) — `ChassisBase.construct_cpo_devices(cpo_data)`

仍未對齊的節點：

- [sonic-buildimage #28324](https://github.com/sonic-net/sonic-buildimage/pull/28324) — Mellanox draft 仍新增 / 讀取 `optical_devices.json`

這些 PR 尚未形成介面一致、可以依序合併的完整實作鏈。

## 主要技術方向

### Port model 正在取代平台特例

規格端已落地：

- `cpo.json`（HwSKU 優先、platform fallback）
- Sparse `_cpo_list` / `_sfp_list`
- Vendor hook：`construct_cpo_devices`

實作端仍卡在 #28077、#700 未合併，以及 NVIDIA 仍用舊檔名。

### `xcvrd` 尚未收斂成單一路線

三套重疊設計仍在：

- [SONiC #2273](https://github.com/sonic-net/SONiC/pull/2273)：共用 CMIS manager（Joint Mode）
- [SONiC #2391](https://github.com/sonic-net/SONiC/pull/2391)：`TRANSCEIVER_ELS_*` tables
- [SONiC #2444](https://github.com/sonic-net/SONiC/pull/2444)：獨立 CPO tasks（07-14 仍有開放 review）

[sonic-platform-daemons #843](https://github.com/sonic-net/sonic-platform-daemons/pull/843) 仍是 draft 骨架：merge conflict、Azure fail。作者已說明 `CpoManagerTask` 粒度與 logical interface 相同；`skip_cpo_mgr`、非 EEPROM→DB DOM 路徑仍待 #2444 收斂。

### Mellanox/NVIDIA 仍是第三條活躍硬體線

平台、vendor CLI、vModule telemetry 草稿仍同時存在，且尚未可合併：

- [buildimage #28324](https://github.com/sonic-net/sonic-buildimage/pull/28324)
- [buildimage #28325](https://github.com/sonic-net/sonic-buildimage/pull/28325)
- [platform-common #708](https://github.com/sonic-net/sonic-platform-common/pull/708)
- [utilities #4673](https://github.com/sonic-net/sonic-utilities/pull/4673)

### Bailly 202605：common backport 已被 image 消費

- [platform-common #689](https://github.com/sonic-net/sonic-platform-common/pull/689)：已合併
- [buildimage #28311](https://github.com/sonic-net/sonic-buildimage/pull/28311)：已合併；`202605` 的 `src/sonic-platform-common` = `357115bf`（#689）
- 仍待落地：[buildimage #28109](https://github.com/sonic-net/sonic-buildimage/pull/28109)、[#28140](https://github.com/sonic-net/sonic-buildimage/pull/28140)

## 依賴鏈與目前斷點

### 通用 port topology

[SONiC #2211](https://github.com/sonic-net/SONiC/pull/2211) ✅ Merged  
→ [sonic-buildimage #28077](https://github.com/sonic-net/sonic-buildimage/pull/28077) ❌ Open（CI fail、review required）  
→ [sonic-platform-common #700](https://github.com/sonic-net/sonic-platform-common/pull/700) ❌ Draft（Azure fail）

名稱已一致；合併順序仍是 28077 → 700（700 依賴 `device_info.get_cpo_data()`）。

### `xcvrd` runtime

[platform-daemons #823](https://github.com/sonic-net/sonic-platform-daemons/pull/823) ✅  
→ [SONiC #2444](https://github.com/sonic-net/SONiC/pull/2444) ❌ Draft  
→ [platform-daemons #843](https://github.com/sonic-net/sonic-platform-daemons/pull/843) ❌ Draft / conflict / CI fail

### NVIDIA Joint Mode

[SONiC #2273](https://github.com/sonic-net/SONiC/pull/2273) ❌  
→ [buildimage #28324](https://github.com/sonic-net/sonic-buildimage/pull/28324) ❌（仍 `optical_devices.json`）  
→ [platform-common #708](https://github.com/sonic-net/sonic-platform-common/pull/708) ❌  
→ [utilities #4673](https://github.com/sonic-net/sonic-utilities/pull/4673) ❌

### Bailly 202605

[platform-common #689](https://github.com/sonic-net/sonic-platform-common/pull/689) ✅  
→ [buildimage #28311](https://github.com/sonic-net/sonic-buildimage/pull/28311) ✅  
→ [buildimage #28109](https://github.com/sonic-net/sonic-buildimage/pull/28109) ❌ Approved、CI pass、`Cherry Pick Conflict_202605`  
→ [buildimage #28140](https://github.com/sonic-net/sonic-buildimage/pull/28140) ❌ Approved、CI pass、待 merge

## Work tracker

| GitHub item | 層次 | 狀態 | 最後更新 | 目前訊號 |
|---|---|---|---|---|
| [SONiC #2152](https://github.com/sonic-net/SONiC/pull/2152) — CPO support in SONiC | Design | Open、review required、DCO fail | 06-02 | 整體 CPO HLD；明確不進 202605。 |
| [SONiC #2207](https://github.com/sonic-net/SONiC/pull/2207) — ELSFP memory map HLD | Design | Open、approved、CI pass | 05-27 | HLD 未合併；相關 common 實作多已進主線。 |
| [SONiC #2211](https://github.com/sonic-net/SONiC/pull/2211) — Port mapping for CPO | Design | **Merged** | 07-13 | `cpo.json` 正式規格；下游實作仍未合併。 |
| [SONiC #2273](https://github.com/sonic-net/SONiC/pull/2273) — Joint Mode | Design | Open、DCO fail | 06-25 | CPO manager 是否分離仍有未解 threads。 |
| [SONiC #2295](https://github.com/sonic-net/SONiC/pull/2295) — Bailly CPO doc | Design | Open、no review | 06-25 | 實作多已合併，HLD 仍缺人工 review。 |
| [SONiC #2317](https://github.com/sonic-net/SONiC/pull/2317) — Align transceiver CLI | CLI / Test | Open、approved | 05-26 | CPO 是否用專用 debug tool 仍有爭議。 |
| [SONiC #2391](https://github.com/sonic-net/SONiC/pull/2391) — CPO DOM / API refactor | Runtime | Draft | 06-17 | `TRANSCEIVER_ELS_*`；與 #2273、#2444 重疊。 |
| [SONiC #2444](https://github.com/sonic-net/SONiC/pull/2444) — xcvrd CPO HLD | Runtime | Draft | 07-14 | tshalvi：非 EEPROM DOM、`skip_cpo_mgr`；prgeor：`cpo_list`/`sfp_list` 分離。 |
| [daemons #843](https://github.com/sonic-net/sonic-platform-daemons/pull/843) — Initial CPO in xcvrd | Runtime | Draft、conflict、Azure fail | 07-14 | 骨架；task 粒度已說明為 logical interface。 |
| [common #700](https://github.com/sonic-net/sonic-platform-common/pull/700) — ChassisBase CPO | Platform | Draft、Azure fail | 07-13 | 已用 `construct_cpo_devices` + `get_cpo_data()`。 |
| [common #708](https://github.com/sonic-net/sonic-platform-common/pull/708) — NVIDIA vModule stats | Platform | Draft、Azure fail | 07-13 | Review 指出可能含無關 CMIS page 變更。 |
| [common #693](https://github.com/sonic-net/sonic-platform-common/pull/693) — Bailly RLM refactor | Platform | Open、conflicting | 06-23 | 疑似被 #690 取代；不宜當 active line。 |
| [buildimage #28077](https://github.com/sonic-net/sonic-buildimage/pull/28077) — Parse `cpo.json` | Platform | Open、CI fail | 07-13 | 介面已對齊 HLD；Azure / dualtor kvm 阻擋。 |
| [buildimage #28324](https://github.com/sonic-net/sonic-buildimage/pull/28324) — Mellanox Joint Mode | Platform | Draft、conflict、CI fail | 07-10 | 仍用 `optical_devices.json`。 |
| [buildimage #28325](https://github.com/sonic-net/sonic-buildimage/pull/28325) — Mellanox vendor CLI | CLI / Test | Draft、CI/DCO fail | 07-09 | 與 #708、#4673 同工作流。 |
| [utilities #4615](https://github.com/sonic-net/sonic-utilities/pull/4615) — Bailly RLM DOM | CLI / Test | Draft、conflict | 06-24 | 未完成 review。 |
| [utilities #4647](https://github.com/sonic-net/sonic-utilities/pull/4647) — ELS CLI show | CLI / Test | **Merged** | 07-13 | ELS DOM CLI 進 utilities master。 |
| [utilities #4673](https://github.com/sonic-net/sonic-utilities/pull/4673) — Generic CPO CLI | CLI / Test | Draft、coverage fail | 07-10 | 通用 sfputil/sfpshow CPO commands。 |
| [mgmt #25881](https://github.com/sonic-net/sonic-mgmt/pull/25881) — CPO SFP API test | CLI / Test | **Closed（未合併）** | 07-14 | CHANGES_REQUESTED 後關閉；202605 request 仍在 label。 |
| [common #689](https://github.com/sonic-net/sonic-platform-common/pull/689) — Bailly CMIS 202605 | Release | Merged | 07-08 | common-side backport。 |
| [buildimage #28311](https://github.com/sonic-net/sonic-buildimage/pull/28311) — 202605 pointer | Release | **Merged** | 07-14 | image 已消費 #689。 |
| [buildimage #28109](https://github.com/sonic-net/sonic-buildimage/pull/28109) — Bailly import path | Release | Open、approved、CI pass | 07-14 | 待 merge；`Cherry Pick Conflict_202605`。 |
| [buildimage #28140](https://github.com/sonic-net/sonic-buildimage/pull/28140) — cpoutil 202605 | Release | Open、approved、CI pass | 07-14 | 待 merge。 |
| [common #677](https://github.com/sonic-net/sonic-platform-common/issues/677) | Adjacent | Issue open | 05-26 | CMIS memory maps by revision；非 CPO 專屬。 |
| [buildimage #26243](https://github.com/sonic-net/sonic-buildimage/issues/26243) | Adjacent | Issue open | 05-11 | 一般 CMIS firmware timing；非 CPO 專屬。 |

## 已合併的基礎能力

| Repo | PR | 合併日期 | 已落地能力 |
|---|---|---|---|
| SONiC | [#2183](https://github.com/sonic-net/SONiC/pull/2183) | 2026-05-11 | CMIS banking HLD |
| SONiC | [#2211](https://github.com/sonic-net/SONiC/pull/2211) | 2026-07-13 | CPO port mapping HLD；`cpo.json` 正式規格 |
| sonic-linux-kernel | [#473](https://github.com/sonic-net/sonic-linux-kernel/pull/473) | 2026-05-04 | optoe 支援超過 8 lanes 的 CMIS banks |
| sonic-platform-common | [#632](https://github.com/sonic-net/sonic-platform-common/pull/632)、[#640](https://github.com/sonic-net/sonic-platform-common/pull/640)、[#668](https://github.com/sonic-net/sonic-platform-common/pull/668) | 03-16～05-20 | Bank parameter、完整 banking 與 `max_banks_size` |
| sonic-platform-common | [#667](https://github.com/sonic-net/sonic-platform-common/pull/667)、[#678](https://github.com/sonic-net/sonic-platform-common/pull/678)、[#684](https://github.com/sonic-net/sonic-platform-common/pull/684) | 05-26～06-08 | CMIS page refactor、ELSFP memory map 與 API |
| sonic-platform-common | [#676](https://github.com/sonic-net/sonic-platform-common/pull/676)、[#682](https://github.com/sonic-net/sonic-platform-common/pull/682) | 2026-06-10 | CPO identifier 與 CpoBase/OE/ELSFP API factory |
| sonic-platform-common | [#672](https://github.com/sonic-net/sonic-platform-common/pull/672) | 2026-06-05 | Bailly CMIS/API base |
| sonic-platform-common | [#692](https://github.com/sonic-net/sonic-platform-common/pull/692) | 2026-06-26 | Broadcom Davisson TH6 CPO |
| sonic-platform-common | [#690](https://github.com/sonic-net/sonic-platform-common/pull/690) | 2026-07-07 | Bailly RLM DOM 與 joint-mode paths |
| sonic-platform-common | [#689](https://github.com/sonic-net/sonic-platform-common/pull/689) | 2026-07-08 | Bailly CMIS common-side 202605 backport |
| sonic-platform-daemons | [#823](https://github.com/sonic-net/sonic-platform-daemons/pull/823) | 2026-06-10 | `xcvrd` CMIS state machine 接受 CPO identifier |
| sonic-buildimage | [#24371](https://github.com/sonic-net/sonic-buildimage/pull/24371)、[#24439](https://github.com/sonic-net/sonic-buildimage/pull/24439) | 2025-11 | Mellanox CPO platform API 與 shared label-port mapping |
| sonic-buildimage | [#26998](https://github.com/sonic-net/sonic-buildimage/pull/26998) | 2026-05-13 | Micas M2-W6940-128X1-FR4 初始 CPO platform tree |
| sonic-buildimage | [#27517](https://github.com/sonic-net/sonic-buildimage/pull/27517) | 2026-06-03 | Broadcom Bailly TH5 platform |
| sonic-buildimage | [#27808](https://github.com/sonic-net/sonic-buildimage/pull/27808) | 2026-06-25 | Bailly RLM DOM、laser info 與 cpoutil |
| sonic-buildimage | [#28118](https://github.com/sonic-net/sonic-buildimage/pull/28118) | 2026-06-27 | master 引入 Davisson #692 與最新 SFF-8024 codes |
| sonic-buildimage | [#28292](https://github.com/sonic-net/sonic-buildimage/pull/28292) | 2026-07-09 | master 傳播 Bailly RLM / joint-mode (#690) |
| sonic-buildimage | [#28311](https://github.com/sonic-net/sonic-buildimage/pull/28311) | 2026-07-14 | 202605 消費 Bailly CMIS common backport (#689) |
| sonic-utilities | [#4333](https://github.com/sonic-net/sonic-utilities/pull/4333)、[#4447](https://github.com/sonic-net/sonic-utilities/pull/4447) | 03-15～04-10 | CPO EEPROM 識別與 page-failure 容錯 |
| sonic-utilities | [#4647](https://github.com/sonic-net/sonic-utilities/pull/4647) | 2026-07-13 | ELS CLI show（sfputil / sfpshow） |
| sonic-mgmt | [#22892](https://github.com/sonic-net/sonic-mgmt/pull/22892)、[#23609](https://github.com/sonic-net/sonic-mgmt/pull/23609)、[#23508](https://github.com/sonic-net/sonic-mgmt/pull/23508) | 2026-04 | CPO presence 與 reboot-cause tests |

## Recent GitHub activity

| 時間 | 項目 | 事件 | 實質影響 |
|---|---|---|---|
| 07-13 16:26 | [SONiC #2211](https://github.com/sonic-net/SONiC/pull/2211) | Merged | 通用 `cpo.json` 規格落地。 |
| 07-13 16:26 | [utilities #4647](https://github.com/sonic-net/sonic-utilities/pull/4647) | Merged | ELS CLI 進主線。 |
| 07-13 | [mgmt #25881](https://github.com/sonic-net/sonic-mgmt/pull/25881) | Closed | CPO platform API test 未合併即關閉。 |
| 07-13 18:00 | [common #700](https://github.com/sonic-net/sonic-platform-common/pull/700) | Sync | Hook/檔名對齊 HLD。 |
| 07-13 23:05 | [buildimage #28077](https://github.com/sonic-net/sonic-buildimage/pull/28077) | CI regress | 名稱對齊後 CI 轉紅。 |
| 07-14 05:24 | [daemons #843](https://github.com/sonic-net/sonic-platform-daemons/pull/843) | Review | 確認 CPO manager 以 logical interface 為粒度。 |
| 07-14 09:13 | [buildimage #28311](https://github.com/sonic-net/sonic-buildimage/pull/28311) | Merged | 202605 image 吃到 #689。 |
| 07-14 | [buildimage #28109](https://github.com/sonic-net/sonic-buildimage/pull/28109) / [#28140](https://github.com/sonic-net/sonic-buildimage/pull/28140) | Merge pressure | 兩者 approved + CI green，等待維護者合併。 |
| 07-14 17:09 | [SONiC #2444](https://github.com/sonic-net/SONiC/pull/2444) | Review | runtime 設計方向持續收斂中。 |

## 容易誤讀的項目

- 通用鏈（#2211 / #28077 / #700）**介面名稱已對齊**；不要再把三者都標成 `optical_devices.json` 分叉。剩餘分叉主要在 Mellanox #28324。
- [common #700](https://github.com/sonic-net/sonic-platform-common/pull/700) 的部分舊 review 文字仍提到 `optical_devices.json` / `construct_optical_devices`；以目前 PR head code 為準。
- [mgmt #25881](https://github.com/sonic-net/sonic-mgmt/pull/25881) 是 closed-unmerged，不是完成。
- [common #693](https://github.com/sonic-net/sonic-platform-common/pull/693) 仍不宜視為健康 active line。
- utilities #4647 已在 utilities master；buildimage 是否已把該 submodule pointer 推進需另看後續 automerge PR（目前 master utilities SHA 已包含 #4647 之後的 commits）。

---

# Davisson TH6 CPO Support 現況

> GitHub snapshot：2026-07-15  
> 核心 PR：[sonic-platform-common #692](https://github.com/sonic-net/sonic-platform-common/pull/692)

## 結論

Davisson TH6 的 common API、factory 與 memory map 已合併，也已由 `sonic-buildimage/master` 引入。  
公開程式碼中**找不到** vendor platform 建立：

```python
CpoHardwareInfo(oe_id=OeId.BROADCOM_DAVISSON, ...)
```

`BROADCOM_DAVISSON` code search（sonic-net）只命中 `sonic-platform-common` 的 enum / factory / unit tests。

因此現況是：

> Davisson 的 common library support 已完成，但從平台 topology、物件建立、`xcvrd` runtime、DOM、CLI 到實體硬體驗證的 end-to-end 路徑尚未完成。

## 相關通用進度與 Davisson 缺口

| 通用進度 | 對 Davisson 的意義 |
|---|---|
| SONiC #2211 已合併 | 可依正式 `cpo.json` 規格做 topology |
| buildimage #28077 / common #700 對齊但未合併 | platform consumer 仍缺通用 parser / Chassis hook |
| utilities #4647 已合併 | ELS CLI 通用能力可用；尚非 Davisson-specific wiring |
| xcvrd #2444 / #843 未合併 | runtime / DOM 路徑仍不可用 |
| 無公開 Davisson vendor / HLD / 202605 backport PR | SKU wiring 與 release 路徑尚未啟動 |

## 已完成的基礎工作

### 通用 CPO API

[platform-common #682](https://github.com/sonic-net/sonic-platform-common/pull/682)：`CpoBase`、`OeBase`、`ElsfpBase`、`CpoHardwareInfo`、factories、API cache / EEPROM wiring。

### CMIS / ELSFP memory map

[#667](https://github.com/sonic-net/sonic-platform-common/pull/667)、[#678](https://github.com/sonic-net/sonic-platform-common/pull/678)、[#684](https://github.com/sonic-net/sonic-platform-common/pull/684)、[linux-kernel #473](https://github.com/sonic-net/sonic-linux-kernel/pull/473)。

### Davisson TH6 CPO Support #692

- 合併：2026-06-26  
- Merge commit：`bc607da86d6012315547072bdae92d57df8d66b0`  
- `OeId.BROADCOM_DAVISSON = 1`（process-local factory key）  
- `DavissonTh6OeApi` / `DavissonTh6ElsfpApi` + `0xB0`–`0xB5` page remap  
- Joint Mode：`elsfp_id=None`，ELSFP 映到 OE CMIS address space  
- Unit tests：factory、memmap、bank、EEPROM R/W、cache  
- Buildimage 整合：[buildimage #28118](https://github.com/sonic-net/sonic-buildimage/pull/28118)（已合併）

### 相關 TH6 platform groundwork

[buildimage #27734](https://github.com/sonic-net/sonic-buildimage/pull/27734) 的 Micas TH6 平台（M2-W6950-128OC、M2-W6951-64HC-CP）仍**沒有**公開 `BROADCOM_DAVISSON` wiring；只能當 TH6 groundwork，不能證明 Davisson CPO 已接上。

本 repo（buildimage）目前可見的 `cpo.json` / `get_cpo_json_data()` 路徑屬於 **Bailly / Micas FR4**（`m2-w6940-128x1-fr4`），不是 Davisson consumer。

## 目前缺少的部分（更新後）

1. **Vendor platform consumer** — 仍未找到  
2. **Davisson `cpo.json` topology** — 仍未找到；通用規格 (#2211) 已就緒，parser (#28077) 未合併  
3. **Chassis object construction** — 通用 hook (#700) 未合併；亦無 Davisson 實作  
4. **`xcvrd` runtime** — #2444 / #843 未合併  
5. **Composite CPO API 完整化** — `do_fiber_check` / `tx_disable` 等仍 TODO  
6. **DOM / Redis publication** — 仍在 #2391 / #2444  
7. **CLI** — 通用 ELS CLI (#4647) 已合併；Davisson-specific / 完整 CPO CLI (#4673) 未完成  
8. **實體硬體 register validation** — 仍只有 unit tests  
9. **Davisson-specific HLD** — 仍未找到  
10. **202605 Davisson common API backport** — 仍未找到（#28311 只帶 Bailly #689，不含 #692）  
11. **Hardware ID factory 擴充性** — centralized `OeId` registry 問題仍在；#2211 review 曾提 vendor 自帶 `XcvrApiConfig`，尚未收斂

## 建議的完成順序（微調）

1. 確認 Davisson vendor platform 與對應 TH6 SKU  
2. 等或協助合併通用鏈：#28077 → #700  
3. 建立 Davisson `cpo.json` topology  
4. platform code 建立 `CpoHardwareInfo(BROADCOM_DAVISSON, ...)`  
5. EEPROM / `0xB0`–`0xB5` smoke + multi-bank / setpoint / alarm tests  
6. 完成 `xcvrd` CPO manager 與 DOM（#2444 → #843）  
7. 完成通用 CPO CLI（#4673）並驗證 Davisson 欄位  
8. 補 Davisson-specific HLD 與 release validation  
9. 決定是否需要把 #692 backport 到 202605

## 整體成熟度

| 層次 | 狀態 |
|---|---|
| Common CPO base API | 已合併 |
| Davisson hardware ID / OE / ELSFP / memmap | 已合併 |
| Unit tests / buildimage master integration | 已完成 |
| 通用 `cpo.json` HLD | **已合併（新）** |
| 通用 parser / Chassis hook | 對齊中、**未合併** |
| TH6 platform groundwork | 已合併，與 Davisson wiring 關係未明確 |
| Vendor platform consumer | 未找到 |
| Davisson `cpo.json` topology | 未找到 |
| `xcvrd` runtime / DOM / CLI（完整） | 未完成 |
| 實體硬體 validation / Davisson HLD | 未公開 |
| 202605 Davisson common backport | 未找到 |

## Sources

- `sonic-net/SONiC`
- `sonic-net/sonic-platform-common`
- `sonic-net/sonic-platform-daemons`
- `sonic-net/sonic-buildimage`
- `sonic-net/sonic-utilities`
- `sonic-net/sonic-mgmt`
- `sonic-net/sonic-linux-kernel`
