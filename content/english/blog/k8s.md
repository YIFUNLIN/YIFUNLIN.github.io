---
title: "Kubernetes (K8s) 入門筆記"
date: 2025-08-05T01:59:00+08:00
draft: false

# post thumb
# image: "images/featured-post/post-2.jpg"

# meta description
description: "這是一份關於 Kubernetes (K8s) 的入門學習筆記，從核心元件到整體架構，幫助您快速理解 K8s 的運作原理。"

# taxonomies
categories: 
  - "Kubernetes"
tags:
  - "Kubernetes"
  - "K8s"
  - "Docker"

# post type
type: "featured"
---
今天要分享的是我先前整理的 Kubernetes (K8s) 入門筆記，希望能幫助大家快速上手這個強大的容器編排工具。

---

## K8s 核心元件

-   **Node**: 一個 Node 就是一台實體伺服器或虛擬機，是 K8s 工作負載的運行基礎，可運行一個或多個 Pod。

-   **Pod**: K8s 中最小的調度和部署單位。它為一個或多個容器提供了一個共享的運行環境，包含共享的儲存（Volumes）和網路資源。雖然一個 Pod 可以包含多個容器，但通常情況下一個 Pod 只運行一個主容器。

-   **Sidecar**: 這是一種將主容器和輔助容器放在同一個 Pod 中的模式。Sidecar 容器用來擴展或增強主容器的功能，例如日誌收集、監控、配置管理等，而不需要修改主應用程式的程式碼。

-   **Service (svc)**: 將一組 Pod 封裝成一個單一且穩定的服務端點。Service 提供了一個統一的入口來訪問這組 Pod，並具備負載平衡的功能。它可以分為對內的 ClusterIP 服務和對外的 NodePort、LoadBalancer 服務。

-   **Ingress**: 作為叢集內服務的對外入口，負責管理從外部訪問叢集內部服務的規則。透過 Ingress，您可以配置不同的 HTTP/HTTPS 路由規則，將外部請求根據域名或路徑轉發到叢集內不同的 Service 上。

-   **ConfigMap 和 Secret**: 用於將配置資訊和敏感資料（如密碼、API 金鑰）與應用程式映像檔解耦。這使得配置的修改不需要重新編譯和部署應用程式，提高了靈活性和安全性。

-   **Volume**: 解決了容器內數據持久化的問題。它可以將數據掛載到本地磁碟或遠端儲存空間（如 NFS、Ceph 或雲端儲存），確保即使 Pod 重啟或被刪除，數據依然存在。

-   **Deployment 和 StatefulSet**:
    * **Deployment**: 用於部署和管理「無狀態」應用程式。它能管理 Pod 和 ReplicaSet，並提供副本控制、滾動更新（rolling update）和回滾（rollback）等功能，以實現高可用性。
    * **StatefulSet**: 用於部署和管理「有狀態」應用程式（如資料庫）。它能確保 Pod 擁有穩定且唯一的網路標識符和持久化儲存，並保證部署和擴展的順序性。

---

## K8s 架構：Master-Worker

K8s 採用 Master-Worker (或稱 Control Plane-Node) 的架構。

### Worker Node (工作節點) 架構
每個 Worker Node 都包含三個核心組件：

-   **Container Runtime**: 運行容器的基礎軟體，負責拉取容器映像檔、創建、啟動和停止容器。常見的 Runtime 有 Docker、containerd 和 CRI-O。
-   **Kubelet**: 每個節點上的代理程式，負責管理該節點上的 Pod，確保容器按照 Pod 的規格（PodSpec）運行。它會定期與 API Server 通信，接收指令並回報節點狀態。
-   **Kube-proxy**: 負責為 Service 提供網路代理和負載平衡。它會在每個節點上維護網路規則，將發往 Service 的流量高效地路由到後端正確的 Pod 中。

### Master Node (控制平面) 架構
Master Node 是 K8s 叢集的大腦，負責管理和決策，主要由以下四個元件組成：

-   **kube-apiserver**: K8s 的 API 伺服器，是整個叢集的統一入口。所有元件之間的通信都通過 API Server 進行。它負責處理 REST 請求、驗證和處理數據，並將結果存儲到 etcd 中。同時，它也負責認證、授權和訪問控制。

-   **etcd**: 一個高可用的鍵值（Key-Value）儲存系統，用於保存整個叢集的所有狀態數據，例如 Pod、Service、Node 的配置和狀態資訊。etcd 是叢集的單一事實來源（single source of truth），是整個系統的數據儲存中心。

-   **Scheduler (調度器)**: 負責監控新創建的 Pod，並根據資源需求、策略和限制，將它們分配到最合適的 Worker Node 上運行。

-   **Controller Manager (控制器管理器)**: 運行多個控制器進程，這些控制器負責監控叢集狀態，並努力將當前狀態驅動到期望狀態。例如，當一個 Node 故障時，Node Controller 會及時發現並處理；當 Deployment 的副本數不足時，Replication Controller 會啟動新的 Pod。

-   **Cloud Controller Manager (雲端控制器管理器)**: (可選) 如果您在雲端供應商（如 GCP, AWS, Azure）上運行 K8s，這個組件會負責與雲端平台的 API 進行交互，管理雲端特有的資源，如負載平衡器、儲存卷等。

---

## Deployment 與 ReplicaSet 的關係

在 K8s 中，我們通常不直接操作 Pod 或 ReplicaSet，而是通過更高層級的 Deployment 來管理。

`Deployment` --(管理)--> `ReplicaSet` --(管理)--> `Pod`

-   **ReplicaSet**: 確保在任何時候都有指定數量的 Pod 副本在運行。
-   **Deployment**: 管理 ReplicaSet 的版本和更新。當您更新 Deployment 的配置時（例如，更新容器映像檔），它會創建一個新的 ReplicaSet，並以滾動更新的方式逐步將 Pod 從舊版本遷移到新版本，實現平滑升級。

我們只需要定義期望的狀態（例如，"我需要這個應用程式的3個副本運行最新版本"），K8s 就會自動完成所有底層的創建、更新和管理工作。

---

## Service 的其他類型

除了常見的 `NodePort`，Service 還有其他類型：

-   **LoadBalancer**: 將服務暴露到外部的雲端負載平衡器上，通常在雲端環境中使用。
-   **ExternalName**: 將服務通過 CNAME 記錄映射到一個外部域名，用於在叢集內部訪問外部服務。
-   **Headless**: 不會分配 ClusterIP，主要用於服務發現，允許您直接解析到後端所有 Pod 的 IP 地址。

---

## K3s & Minikube

-   **K3s**: 一個輕量級、經過 CNCF 認證的 Kubernetes 發行版，適用於邊緣計算、物聯網和資源受限的環境。
-   **Minikube**: 一個可以在本地輕鬆運行單節點 Kubernetes 叢集的工具，非常適合學習和開發測試。

---

## 學習資源

1.  **[30 分鐘 Docker 入門](https://www.youtube.com/watch?v=Ozb9mZg7MVM)**
    * 這部影片介紹了 Docker 的核心概念，包括它與傳統虛擬機的區別，以及鏡像（Image）、容器（Container）和倉庫（Repository）等關鍵元素。影片還通過一個 Node.js 應用程式實例，演示了如何編寫 Dockerfile、構建鏡像和運行容器，並介紹了 Docker Compose 在多容器應用管理中的應用。

2.  **[Kubernetes 一小時輕鬆入門](https://www.youtube.com/watch?v=SL83f7Nzxr0)**
    * 這部影片旨在幫助初學者快速掌握 K8s 的核心概念和架構，包括 Node、Pod、Service 等。影片詳細介紹了 Master-Worker 架構中各組件的功能，並指導如何使用 Minikube 或 K3s 在本地搭建環境，以及 `kubectl` 的基本操作。

3.  **[Ithome 鐵人賽好文：入門 Kubernetes 到考取 CKA 證照](https://ithelp.ithome.com.tw/users/20168692/ironman/7376?page=1)**
    * 這是一系列非常詳盡的部落格文章，從基礎概念到實戰操作，內容涵蓋廣泛，是深入學習 K8s 並準備 CKA 認證的絕佳資源。

http://googleusercontent.com/youtube_content/0 http://googleusercontent.com/youtube_content/1