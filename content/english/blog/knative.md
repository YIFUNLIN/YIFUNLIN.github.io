---
title: "Knative 深度解析：在 Kubernetes 上實現 Serverless"
date: 2025-08-05T02:27:00+08:00
draft: false

# post thumb
# image: "images/featured-post/post-3.jpg"

# meta description
description: "本文深入介紹 Knative 的核心組件 Serving 和 Eventing，探討其如何在 Kubernetes 之上構建一個強大的 Serverless 平台，實現事件驅動架構和請求驅動的自動擴展。"

# taxonomies
categories: 
  - "Kubernetes"
tags:
  - "Knative"
  - "Serverless"
  - "K8s"
  - "Event-Driven"

# post type
type: "featured"
---

在了解了 Kubernetes 的基礎架構後，這次我們來探討一個建立在其之上的強大 Serverless 框架：Knative。它將開發者從繁瑣的基礎設施管理中解放出來，專注於程式碼本身。

---

## 什麼是 Knative？

Knative 是一個開源的 Kubernetes 附加元件 (Add-on)，用於建構、部署和管理現代化的 Serverless 工作負載。它並非要取代 Kubernetes，而是**擴展** Kubernetes，提供了一組更高階的抽象化原語 (Primitives)，旨在標準化和簡化 Serverless 應用程式的開發與部署流程。

其核心價值主張是：**讓開發者只關心業務邏輯，而將服務的自動擴展 (Auto-scaling)、網路路由和事件觸發等複雜性交給平台處理。**

---

## Knative 的核心組件

Knative 主要由兩個獨立且可插拔的核心組件構成：

### 1. Serving (服務供給)
Knative Serving 的目標是部署和服務 Serverless 應用程式和函式。它最引人注目的特性是**請求驅動 (Request-Driven)** 的計算模型，能夠根據流量需求自動擴展 Pod 數量，甚至在沒有流量時**縮容至零 (Scale to Zero)**，從而極大地節省了運算資源成本。

**主要特性：**
- **快速部署與版本管理**: 每次對應用程式的程式碼或配置進行更改時，都會創建一個不可變的**修訂版本 (Revision)**，方便進行版本追蹤和快速回滾。
- **流量分割 (Traffic Splitting)**: 能夠精確控制流向不同修訂版本的流量比例，輕鬆實現金絲雀部署 (Canary Deployments) 和藍綠部署 (Blue/Green Deployments)。
- **基於請求的自動擴展**: 內建 KPA (Knative Pod Autoscaler) 元件，可根據每秒處理的併發請求數來自動增減 Pod 實例。

**核心資源 (CRDs):**
- **Service (`ksvc`)**: 這是開發者主要操作的頂層資源，它自動管理一個應用的 `Route` 和 `Configuration`，簡化了部署流程。
- **Route**: 負責將網路端點 (URL) 映射到一個或多個 `Revision`，並定義流量分配策略。
- **Configuration**: 定義了應用程式的期望狀態，包括容器映像檔、環境變數等。每次 `Configuration` 的變更都會觸發一個新的 `Revision`。
- **Revision**: `Configuration` 的一個不可變的時間點快照。每個 `Revision` 都代表了一份特定的程式碼和配置組合。

### 2. Eventing (事件驅動)
Knative Eventing 提供了一套標準化的基礎設施，用於建構事件驅動架構。它旨在實現服務間的**鬆耦合 (Loosely Coupled)**，讓事件的生產者 (Producers) 和消費者 (Consumers) 彼此獨立，無需直接感知對方的存在。

**工作流程：**
它採用了發布/訂閱 (Publish/Subscribe) 模型，事件從**來源 (Source)** 進入系統，被發送到一個稱為**代理 (Broker)** 的事件中心，然後**觸發器 (Trigger)** 根據過濾規則將事件分發給一個或多個目標**消費者 (Sink)**。

**核心資源 (CRDs):**
- **Source**: 將外部系統的事件導入 Knative 的橋樑。例如，`ApiServerSource` 可以監聽 Kubernetes API 事件，`KafkaSource` 可以從 Kafka 主題中拉取訊息。
- **Broker**: 作為事件的匯集中心和訊息通道，接收事件並將其暫存，等待分發。
- **Trigger**: 將 `Broker` 與事件消費者連接起來，並可以定義過濾規則 (Filter)，只有符合條件的事件才會被傳遞給目標。
- **Sink**: 事件的最終目的地，可以是任何可接收 HTTP 請求的資源，最常見的就是一個 Knative Service。

---

## Knative 與 Kubernetes 的關係

理解它們的關係至關重要：Knative **將 Kubernetes 的能力進行了封裝和簡化**。

當您創建一個 Knative `Service` 時，Knative 的控制器 (Controller) 會在背景將這個高階定義轉換成一系列底層的 Kubernetes 資源，例如：
- `Deployment`: 用於管理 Pod。
- `Service` (K8s Service): 用於叢集內部網路。
- `Ingress`: 用於外部流量接入。
- `HorizontalPodAutoscaler` (HPA): 用於標準的 CPU/記憶體擴展。
- `Knative Pod Autoscaler` (KPA): 用於基於請求的擴展。

開發者只需維護一份簡單的 Knative YAML 文件，Knative 就會自動處理這些複雜的底層資源配置和生命週期管理。

---

## Knative 的依賴與生態

-   **服務網格 (Service Mesh)**: Knative Serving 的許多進階網路功能（如流量分割）依賴於底層的服務網格，例如 **Istio**, **Linkerd**, 或 **Contour**。Istio 是最常見的選擇。
-   **CI/CD (持續整合/持續部署)**: Knative 最初的 `Build` 組件已被獨立出來，發展成為一個更通用的 CI/CD 專案 **Tekton**。Tekton 專注於在 Kubernetes 上構建、測試和部署應用，與 Knative 能夠完美整合。

---

## 總結：為何要用 Knative？

-   **提升開發者體驗**: 將基礎設施複雜性抽象化，讓開發者能更專注於業務邏輯。
-   **極致的成本效益**: 「縮容至零」特性確保閒置的應用不消耗任何計算資源。
-   **標準化與可移植性**: 基於 Kubernetes，不被任何雲端廠商鎖定，可在任何標準的 K8s 叢集上運行。
-   **強大的事件驅動能力**: 為構建現代化的異步、鬆耦合微服務架構提供了堅實的基礎。

總之，Knative 結合了 Serverless 的簡易性、事件驅動的靈活性以及 Kubernetes 的強大功能和可控性，是雲端原生時代下構建現代化應用的理想平台。