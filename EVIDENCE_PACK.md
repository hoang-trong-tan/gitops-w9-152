# Evidence Pack: Canary Deployment & Observability Project

Dự án triển khai mô hình **Progressive Delivery** sử dụng Argo Rollouts, Prometheus và Alertmanager.

---

## 1. GitOps Workflow (Git Sync)
*Mục tiêu: Chứng minh cấu hình được quản lý hoàn toàn qua Git và Argo CD tự động đồng bộ.*

- **Screenshot yêu cầu:** Ảnh chụp giao diện Argo CD hiển thị trạng thái `Synced` và `Healthy` của các ứng dụng `api` và `kube-prometheus-stack`.
![Gitops](images/ArgoCD%20+%20Prometheus%20sync.png)
---

## 2. Canary Deployment & Auto-Abort (Bản lỗi tự quay xe)
*Mục tiêu: Chứng minh hệ thống tự động phát hiện lỗi và Rollback.*

- **Kịch bản:** Sửa `ERROR_RATE` thành `0.6` và `VERSION` thành `v3` trên Git, đẩy lên và theo dõi Rollout.
- **Screenshot yêu cầu:**
    1. Ảnh chụp `kubectl argo rollouts get rollout api -n demo --watch` hiển thị trạng thái `Degraded` khi lỗi bị phát hiện.
    2. Ảnh chụp bảng AnalysisRun kết thúc với trạng thái `Failed` do tỷ lệ lỗi cao.
![Canary-Auto]()

---

## 3. Observability & SLO Alerts (Cảnh báo qua Email)
*Mục tiêu: Chứng minh hệ thống giám sát hoạt động và gửi cảnh báo khi vi phạm SLO.*

- **Kịch bản:** Khi Canary lỗi, Prometheus ghi nhận vi phạm ngưỡng `APISuccessRateLow` và gửi email.
- **Screenshot yêu cầu:**
    1. Ảnh chụp danh sách Alert trên Alertmanager (`http://localhost:9093`) hiển thị trạng thái `Firing`.
    2. Ảnh chụp email nhận được từ hệ thống cảnh báo (chứa thông báo `APISuccessRateLow`).
[CHÈN ẢNH TẠI ĐÂY]

---

## 4. Rollback via Git Revert (< 5 phút)
*Mục tiêu: Chứng minh khả năng phản ứng nhanh với sự cố thông qua Git revert.*

- **Kịch bản:** Giả sử bạn vừa đẩy bản lỗi `v2` và muốn quay về `v1` bằng Git.
- **Screenshot yêu cầu:**
    1. Ảnh chụp lịch sử Git với commit `revert`.
    2. Ảnh chụp giao diện Argo CD đang trong quá trình đồng bộ (Syncing) sau khi thực hiện `git revert`.
    3. Ảnh chụp Rollout đã quay về bản ổn định (stable) cũ.
[CHÈN ẢNH TẠI ĐÂY]

---

## 5. Cấu hình Kỹ thuật (Configuration Documentation)

### AnalysisTemplate Query
Đoạn code dùng để đánh giá Canary:
```yaml
(sum(rate(flask_http_request_total{namespace="demo", status="200"}[2m])) / sum(rate(flask_http_request_total{namespace="demo"}[2m]))) or on() vector(1)
```

### Prometheus SLO Rule
Ngưỡng cảnh báo cho SLO:
```yaml
- alert: APISuccessRateLow
  expr: (sum(rate(flask_http_request_total{namespace="demo", status="200"}[2m])) / sum(rate(flask_http_request_total{namespace="demo"}[2m]))) < 0.95
  for: 1m
```

---
*Người thực hiện: [Tên của bạn]*
*Dự án: GitOps Progressive Delivery*
