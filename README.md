# .NET Memory Performance Analysis

<p align="center">
  <i>了解何時需要關注 .NET 記憶體效能，以及在需要時應採取的措施</i>
</p>

---

## 關於本文

本文原作者為 **[Maoni Stephens](https://github.com/Maoni0)**（Microsoft .NET GC 團隊首席工程師），是一份長達 **2300 行、170 KB** 的 .NET 記憶體效能分析權威指南，原文託管於 [mem-doc](https://github.com/Maoni0/mem-doc)。

本 Repo 將原文翻譯為繁體中文，並做了以下整理：

- 所有 H1~H4 標題加上**結構化編號**（`一、` → `1.` → `I.` → `a.`）
- **35 張圖片**統一收納至 `images/` 目錄
- 原文內容**一字未刪**
- FAQ 段落合併為單一標題，提升可讀性

---

## 目錄結構

```
.NET Memory/
├── NETMemoryPerformanceAnalysis.md   # 主文件
├── images/                           # 35 張示意圖（GC heap、PerfView 操作、事件序等）
└── README.md
```

---

## 章節總覽

| # | 章節 | 重點內容 |
|---|------|----------|
| 1 | **Purpose of this document** | 適用對象：.NET Framework / .NET Core 開發者 |
| 2 | **State of this document** | 版本歷史、貢獻指引（歡迎 Linux 內容貢獻） |
| 3 | **How to read this document** | 依經驗程度提供 5 種閱讀路徑 |
| 4 | **Contents** | 完整目錄、內部錨點連結 |
| 5 | **How to think about performance work** | 效能分析的偵探思維、不靠猜測靠測量 |
| 6 | **Picking the right approaches** | 定義效能目標、理解 GC 只是框架的一環 |
| 7 | **Memory 基礎知識** | OS 虛擬記憶體、GC 運作原理、Allocation Budget、分代回收、LOH、Pinning、Finalizers |
| 8 | **Know when to worry** | Top GC 指標、% Pause time、Tail latency、Memory footprint |
| 9 | **選擇正確工具並解讀數據** | PerfView 深度教學、收集 GC 指標、分析長時間暫停原因 |
| 10 | **性能問題的明確跡象** | 暫停過長、隨機長暫停、大量 Full Blocking GC |
| 11 | **幫助我們調試性能問題** | 如何向 GC 團隊提供有效的診斷資訊 |
| 12 | **常見問題** | 升級後記憶體增加、GC 未回收物件、OS 差異、% Time in GC 誤解 |

---

## 涵蓋的核心概念

| 概念 | 說明 |
|------|------|
| **Generational GC** | Gen0 / Gen1 / Gen2 分代回收機制 |
| **Allocation Budget** | 分配預算如何決定 GC 觸發時機 |
| **LOH (Large Object Heap)** | 大型物件堆的設計與陷阱 |
| **Background GC vs Blocking GC** | 並行 vs 完全暫停的取捨 |
| **Server GC vs Workstation GC** | 伺服器 / 桌面應用的 GC 模式選擇 |
| **Pinning & Demotion** | 記憶體釘選、碎片化與降級機制 |
| **Finalizers** | 終結器的效能成本與替代方案 |
| **Managed Memory Leak** | 受控記憶體洩漏的定義與診斷 |
| **Thread Suspension** | GC 暫停中非 GC 工作的部分 |
| **PerfView 實戰** | 收集 trace、GCStats 視圖、Stop Trigger 觸發條件 |

---

## 適合誰閱讀

| 你的情況 | 建議起點 |
|----------|----------|
| 完全沒做過效能分析 | 從第 1 章開始 |
| 有分析經驗，想深入 GC | 跳到 [第 7 章](./NETMemoryPerformanceAnalysis.md#7-memory-基礎知識) |
| 需要快速解決一次性問題 | 跳到 [第 8 章](./NETMemoryPerformanceAnalysis.md#8-know-when-to-worry) |
| 專注 GC 分析的效能工程師 | 先讀 [第 7 章](./NETMemoryPerformanceAnalysis.md#7-memory-基礎知識)，再查 [第 9 章](./NETMemoryPerformanceAnalysis.md#9-選擇正確工具並解讀數據) |
| 有經驗、想解決特定問題 | 直接查 [第 10 章](./NETMemoryPerformanceAnalysis.md#10-性能問題的明確跡象) |

---

## 工具指引

文中主要使用 **[PerfView](https://github.com/microsoft/perfview/)** 進行分析，涵蓋以下實用指令：

- 收集 GC 頂層指標：`perfview /GCCollectOnly /AcceptEULA /nogui collect`
- 收集 Allocation Tick：`PerfView.exe /nogui /accepteula /KernelEvents=Process+Thread+ImageLoad /ClrEvents:GC+Stack ...`
- 自動觸發停止條件：`/StopOnGCOverMSec:15` 等
- Linux 等效指令：`dotnet trace collect --profile gc-collect`

---

## 相關資源

- [原始文件 (Maoni0/mem-doc)](https://github.com/Maoni0/mem-doc)
- [PerfView 專案](https://github.com/microsoft/perfview/)
- [.NET GC 官方文件](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/)
- [GC 效能基礎架構部落格 (Part 1)](https://devblogs.microsoft.com/dotnet/gc-perf-infrastructure-part-1/)
- [Server GC 與 Workstation GC 的中間地帶](https://devblogs.microsoft.com/dotnet/middle-ground-between-server-and-workstation-gc/)
- [Maoni Stephens 的 GC 部落格](https://devblogs.microsoft.com/dotnet/author/maoni/)

---

## 致謝

感謝 **Maoni Stephens** 撰寫這份詳盡的 .NET GC 效能分析指南，以及 .NET GC 團隊持續改進 runtime 的努力。

文件格式整理由 [opencode](https://github.com/anomalyco/opencode) 協助完成。

---

*最後更新：2026-05-30*
