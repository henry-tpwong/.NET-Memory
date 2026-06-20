# .NET Memory Performance Analysis

<p align="center">
  <i>了解何時需要關注 .NET Memory 效能，以及在需要時應採取的措施</i>
</p>

---

## 關於本文

原作者為 **[Maoni Stephens](https://github.com/Maoni0)**（Microsoft .NET GC 團隊首席工程師），是一份長達 **~2900 行** 的 .NET Memory 效能分析權威指南，原文託管於 [mem-doc](https://github.com/Maoni0/mem-doc)。

本 Repo 將原文翻譯為繁體中文，並做了以下整理：

- 所有標題加上**結構化編號**（`一、` → `1.` → `I.` → `a.` → `a.`）
- **35 張圖片**統一收納至 `images/` 目錄
- 加入 **19 張 Mermaid 示意圖**（VA 狀態、GC heap segment、OOM 升級鏈、Free List 生命週期等）
- 原文內容**一字未刪**，僅補充新手導向說明與圖解
- 專業名詞保留英文（Memory、Heap、Allocation、Process、Pause、Bookkeeping 等）

---

## 目錄結構

```
doc/
├── NETMemoryPerformanceAnalysis.md    # 主文件
├── AGENTS.md                          # Agent 操作指引
├── images/                            # 35 張示意圖 + 5 張 Mermaid 匯出圖
├── .NETMemoryPerformanceAnalysis.md   # 原始英文版（唯讀）
├── .NETMemoryPerformanceAnalysis.zh-CN.md  # 舊版中文翻譯（唯讀）
└── README.md
```

---

## 章節總覽

| # | 章節 | 內容摘要 |
|---|------|----------|
| 1 | **本文目的** | 適用對象：.NET Framework / .NET Core 開發者 |
| 2 | **本文狀態** | 版本歷史、貢獻指引 |
| 3 | **如何看待性能分析工作** | 效能分析的偵探思維——不靠猜測，靠 Measure |
| 4 | **選擇正確的性能分析方法** | 定義效能目標、理解 GC 只是框架的一環 |
| 5 | **Memory 基礎知識** | OS 虛擬記憶體、VA 狀態圖、GC 觸發機制、Allocation Budget、分代回收、LOH、碎片化（Compacting vs Sweeping）、GC heap segment 結構、Bookkeeping 結構、OOM 升級鏈、STW 暫停分析、Server GC vs Workstation GC |
| 6 | **何時需要擔心** | Top GC 指標、% Pause Time vs % CPU Time、Tail Latency、Memory Footprint |
| 7 | **選擇正確工具並解讀數據** | PerfView 深度教學、GCStats 視圖、Allocation 分析、長時間暫停診斷、大型 heap 分析 |
| 8 | **性能問題的明確跡象** | 暫停過長、隨機長暫停、大量 Full Blocking GC |
| 9 | **幫助我們調試性能問題** | Runtime 版本、環境資訊、PerfView 收集指令 |
| 10 | **常見問題** | 升級後 memory 增加、GC 未回收 object、OS 差異、% Time in GC counter 誤解、有效求助技巧 |

---

## 涵蓋的核心概念

| 概念 | 說明 | 對應章節 |
|------|------|----------|
| **Virtual Memory / VA States** | VMM、Free/Reserved/Committed、碎片化 | 5 |
| **Generational GC** | gen0 / gen1 / gen2 分代回收機制 | 5 |
| **Allocation Budget** | 預算如何動態調整、如何決定 GC 頻率 | 5 |
| **LOH (Large Object Heap)** | ≥85KB object 直接進 LOH，只在 gen2 GC 回收 | 5 |
| **Compacting vs Sweeping** | 壓縮 vs 不壓縮的取捨、Free List 生命週期 | 5 |
| **GC Heap Segment 結構** | ephemeral segment 不變式、segment 增長與釋放 | 5 |
| **GC Bookkeeping 結構** | Card Table、Brick Table、Object Header、BGC Metadata | 5 |
| **OOM 升級鏈** | gen0 → gen1 → BGC → Full Blocking GC → OOM | 5 |
| **Ephemeral GC vs BGC vs Full Blocking GC** | 三種 GC 的觸發條件、暫停長度、回收範圍 | 5 |
| **Server GC vs Workstation GC** | 多 heap 並行 vs 單 heap、committed memory 取捨 | 5 |
| **Pinning & Demotion** | pinned object 降級到 gen0 的機制與代價 | 5 |
| **Finalizers** | 三階段生命週期（Allocation → Collection → Running） | 5 |
| **Thread Suspension** | GC pause 中非 GC 工作的部分、Safe Point | 5 |
| **Managed Memory Leak** | 定義（user root 持續引用更多 object）與診斷 | 7 |
| **% Pause Time vs % CPU Time in GC** | 兩個容易混淆的 GC 計數器 | 6 |
| **PerfView 實戰** | GCStats、Allocation Tick、Stop Trigger、Diff 比較 | 7 |

---

## 適合誰閱讀

| 你的情況 | 建議起點 |
|----------|----------|
| 完全沒做過效能分析 | 從第 1 章開始，第 5 章是核心基礎 |
| 有分析經驗，想深入 GC | 跳到 [第 5 章](./NETMemoryPerformanceAnalysis.md#五memory-基礎知識) |
| 需要快速解決一次性問題 | 跳到 [第 6 章](./NETMemoryPerformanceAnalysis.md#六何時需要擔心) |
| 專注 GC 分析的效能工程師 | 先讀 [第 5 章](./NETMemoryPerformanceAnalysis.md#五memory-基礎知識)，再查 [第 7 章](./NETMemoryPerformanceAnalysis.md#七選擇正確工具並解讀數據) |
| 有經驗、想解決特定問題 | 直接查 [第 7 章診斷段落](./NETMemoryPerformanceAnalysis.md#七選擇正確工具並解讀數據) 或 [第 10 章 FAQ](./NETMemoryPerformanceAnalysis.md#十常見問題) |

---

## 工具指引

文中主要使用 **[PerfView](https://github.com/microsoft/perfview/)** 進行分析，涵蓋以下實用指令：

- 收集 GC 頂層指標：`perfview /GCCollectOnly /AcceptEULA /nogui collect`
- 收集 Allocation Tick：`PerfView.exe /nogui /accepteula /KernelEvents=Process+Thread+ImageLoad /ClrEvents:GC+Stack collect`
- 自動觸發停止條件：`/StopOnGCOverMSec:15`、`/StopOnGCSuspendOverMSec:200`
- 比較兩個 trace：PerfView Diff → With Baseline
- Linux 等效指令：`dotnet trace collect --profile gc-collect`

---

## 相關資源

- [原始文件 (Maoni0/mem-doc)](https://github.com/Maoni0/mem-doc)
- [PerfView 專案](https://github.com/microsoft/perfview/)
- [.NET GC 官方文件](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/)
- [GC 效能基礎架構部落格](https://devblogs.microsoft.com/dotnet/gc-perf-infrastructure-part-1/)
- [Server GC 與 Workstation GC 的中間地帶](https://devblogs.microsoft.com/dotnet/middle-ground-between-server-and-workstation-gc/)
- [Maoni Stephens 的 GC 部落格](https://devblogs.microsoft.com/dotnet/author/maoni/)

---

## 致謝

感謝 **Maoni Stephens** 撰寫這份詳盡的 .NET GC 效能分析指南，以及 .NET GC 團隊持續改進 runtime 的努力。

文件翻譯與格式整理由 [OpenCode](https://github.com/anomalyco/opencode) 協助完成。

---

*最後更新：2026-06-20*
